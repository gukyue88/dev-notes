---
layout: single
title: "[image-sensor] libcamera/rpi agc 코드 분석"
categories: hardware/image-sensor
---

## 주요 코드 : agc_channel.cpp

#### channel의 의미

- rpi.channel에는 normal AGC, short HDR, long HDR, night mode 채널이 들어있음

#### AgcChannel::process()

```cpp
void AgcChannel::process(StatisticsPtr &stats, DeviceStatus const &deviceStatus,
			 Metadata *imageMetadata,
			 const AgcChannelTotalExposures &channelTotalExposures)
{
	frameCount_++;

    // 노출 계산 전, AE 루프가 시작될 때마다 변경된 설정 값이 있는지 확인하여 최신 설정 모드 정보를 불러와 반영하는 단계
	housekeepConfig();

    // 현재 exposureTime, analogueGain, totalExposureNoDG(`exposureTime * analogueGain`)을 가져옴
	fetchCurrentExposure(deviceStatus);

	// 매핑 함수로 lux to targetY를 계산하고, 해당 targetY를 만족시키기 위한 gain을 계산하여 반환함
	double gain, targetY;
	computeGain(stats, imageMetadata, gain, targetY);

	// 위에서 계산된 gain을 기반으로 최종 target_.totalExposure에 current_.exposureTime * current_.analogueGain * gain 값을 대입
	computeTargetExposure(gain);

	// `target_.totalExposure`가 적용될 때, 너무 급격하게 변하는 것을 방지하고, 노출의 안전성 확보를 위해 `filtered_.totalExposure`, speed, stableRegion을 조정
	filterExposure();

	// 각 채널별 Constraints를 순회하며, 다른 채널의 Contraint도 모두 만족할 수 있는 `filtered_.totalExposure`를 정함
	bool channelBound = applyChannelConstraints(channelTotalExposures);

    // AWB에 의한 색상 왜곡 방지를 위한 dg를 계산하고, 해당 dg로 인해 하이라이트가 포화될 위험에 있을 때 dg를 줄여 빠른 노출 감소를 통해 desaturate를 유도
	bool desaturate = applyDigitalGain(gain, targetY, channelBound);

    // 순수 아날로그 노출값(filtered\_.totalExposureNoDG)를 flickerMode까지 고려하여, 현재 exposureMode의 exposureTime배열과 gain배열을 참고하여 exposureTime과 analogueGain로 분배함
	divideUpExposure();

	// imageMetadata의 `agc.status`에 totalExposureValue(filtered), targetExposureValue(target_), exposure, exposureTime, analogueGain을 새김
	writeAndFinish(imageMetadata, desaturate);
}
```

#### AgcChannel::housekeepConfig()

> 노출 계산 전, AE 루프가 시작될 때마다 변경된 설정 값이 있는지 확인하여 최신 설정 모드 정보를 불러와 반영하는 단계

```cpp
void AgcChannel::housekeepConfig()
{
	/* First fetch all the up-to-date settings, so no one else has to do it. */
	status_.ev = ev_;
	status_.fixedExposureTime = limitExposureTime(fixedExposureTime_);
	status_.fixedAnalogueGain = fixedAnalogueGain_;
	status_.flickerPeriod = flickerPeriod_;
    status_.meteringMode = meteringModeName_; // 측광 모드 (어떤 영역에서 측광할 것인가)
    status_.exposureMode = exposureModeName_; // 조리개 우선, 셔터 스피드 우선, 수동 모드, 자동 모드
    status_.constraintMode = constraintModeName_; // 하이라이트와 쉐도우를 제외한 주요 영역을 사용하기 위한 설정 (예. qLo : 10%, qHi : 90%) 이 들어있음, 위 아래 부분을 잘라 평균을 왜곡하는 부분을 방지함
}
```

#### AgcChannel::fetchCurrentExposure

> 현재 exposureTime, analogueGain, totalExposureNoDG(`exposureTime * analogueGain`)을 가져옴

```cpp
void AgcChannel::fetchCurrentExposure(DeviceStatus const &deviceStatus)
{
	current_.exposureTime = deviceStatus.exposureTime;
	current_.analogueGain = deviceStatus.analogueGain;
	current_.totalExposure = 0s; /* this value is unused */
	current_.totalExposureNoDG = current_.exposureTime * current_.analogueGain;
}
```

#### AgcChannel::computeGain

> 매핑 함수로 lux to targetY를 계산하고, 해당 targetY를 만족시키기 위한 gain을 계산하여 반환함

```cpp
void AgcChannel::computeGain(StatisticsPtr &statistics, Metadata *imageMetadata,
			     double &gain, double &targetY)
{
	struct LuxStatus lux = {};
	lux.lux = 400; /* default lux level to 400 in case no metadata found */
	if (imageMetadata->get("lux.status", lux) != 0)
		LOG(RPiAgc, Warning) << "No lux level found";
	const Histogram &h = statistics->yHist;
	double evGain = status_.ev * config_.baseEv; // 사용자 보정 EV 값 반영

    /*
     * 자연스러운 노출을 위해, 단순한 노출 공식을 따르지 않고, 측정된 lux에 따른 목표 targetY를 다르게 함
     * yTarget.domain().clamp() 함수를 통해 lux 너무 낮거나 높게 설정된 경우 자름
     * yTarget.eval() 함수가 clamp된 lux 값을 넣어 사용자가 만족할 만한 이미지 밝기(제조사 노출 전략)을 얻어냄
     * targetY * evGain은 사용자 보정 EV 값을 반영한 targetY
     * targetY * evGain과 EvGainYTargetLimit를 비교해 최소값을 최종 targetY로 결정
     */
	targetY = config_.yTarget.eval(config_.yTarget.domain().clamp(lux.lux));
	targetY = std::min(EvGainYTargetLimit, targetY * evGain);

	/*
     * 필요한 gain을 구하는 부분이며, gain은 단순히 "targetY / 현재 평균 휘도"로만 구해지지 않음
     * 1.센서의 HDR 처리나 포화된 하이라이트 영역이 포함된 경우, 게인과 이미지 밝기가 선형적이지 않음 (computeInitalY)
     *   - 게인이 2배가 된다고 해서, 이미지 밝기가 2배가 되는 것이 아님
     *   - gain이 적용되었을 때, 포화되는 영역을 계산하여 평균 휘도를 구하게 됨
     * 2. 게인 값을 점진적으로 조정하여 targetY에 부드럽게 수렴하도록 도와, 노출이 한 번에 급변하는 것을 방지 (min(10.0 , targetY/initialY))
	 */
	gain = 1.0;
	for (int i = 0; i < 8; i++) {
        // initialY : gain을 적용했을 때, 정규화된 평균 휘도 값
		double initialY = computeInitialY(statistics, awb_, meteringMode_->weights, gain);
        // targetY / initialY == 필요한 gain
        // + 0.001은 initalY가 0인 경우, 0으로 나누는 오류를 방지하기 위해 넣은 것
        // min(10.0, 필요한 gain)은 너무 급작스럽게 gain이 올라가는 것을 방지하기 위해 10.0으로 상한선을 둔 것임
		double extraGain = std::min(10.0, targetY / (initialY + .001));
		gain *= extraGain;
		LOG(RPiAgc, Debug) << "Initial Y " << initialY << " target " << targetY
				   << " gives gain " << gain;
		if (extraGain < 1.01) /* 오차범위인 1% 이내로 gain이 차이나면 루프를 끝냄 */
			break;
	}

    // 다양한 제약 조건(constraintMode_)를 순회하며 최종 gain을 선택
    // 위에서 계산된 기본 gain 값은 targetY에만 맞추어져 있음
    // LOWER Bound : 최소 gain을 의미, gain이 너무 낮으면, 제약조건으로 계산된 새로운 gain을 채택
    // - gain이 너무 낮아지면, 노출시간이 너무 길어져 프레임속도 유지가 어려움
    // UPPER Bound : 최대 gain을 의미, gain이 너무 크면, 제약조건으로 계산된 새로운 gain을 채택
    // - gain이 너무 높아지면 노이즈가 심해짐
	for (auto &c : *constraintMode_) {
		double newTargetY;
        // newGain : qLo, qHi를 제외한 주요 영역 기준 targetY를 위한 단순 계산된 gain 값
		double newGain = constraintComputeGain(c, h, lux.lux, evGain, newTargetY);
        // constraintMode가 LOWER 한계에 관한 것이고, newGain이 위에서 계산한 gain보다 크다면 newGain을 채택함
		if (c.bound == AgcConstraint::Bound::LOWER && newGain > gain) {
			LOG(RPiAgc, Debug) << "Lower bound constraint adopted";
			gain = newGain;
			targetY = newTargetY;
        // constraintMode가 UPPER 한계에 관한 것이고, newGain이 위에서 계산한 gain보다 작다면 newGain을 채택함
		} else if (c.bound == AgcConstraint::Bound::UPPER && newGain < gain) {
			LOG(RPiAgc, Debug) << "Upper bound constraint adopted";
			gain = newGain;
			targetY = newTargetY;
		}
	}
	LOG(RPiAgc, Debug) << "Final gain " << gain << " (target_Y " << targetY << " ev "
			   << status_.ev << " base_ev " << config_.baseEv
			   << ")";
}
```

#### computeInitialY

> gain에 따른 histogram이나 AgcRegion의 RGB합을 통해 평균 휘도를 구하고, 이를 0~1 사이의 값을 갖도록 정규화한 뒤 반환

```cpp
static double computeInitialY(StatisticsPtr &stats, AwbStatus const &awb,
			      std::vector<double> &weights, double gain)
{
	constexpr uint64_t maxVal = 1 << Statistics::NormalisationFactorPow2;

    // AGC region stats가 없고, Y histogram만 존재하는 경우
    // 이미지의 Y값 평균을 계산하여 return 함
	if (!stats->agcRegions.numRegions() && stats->yHist.bins()) {
		/*
		 * When the gain is applied to the histogram, anything below minBin
		 * will scale up directly with the gain, but anything above that
		 * will saturate into the top bin.
		 */
		auto &hist = stats->yHist;
        // hist.bins() : 분포를 나누는 구간의 개수, 예) 256개
        // minBin : gain을 적용했을 때, 픽셀이 포화되지 않고 정상적으로 측정될 수 있는 최대 구간 (bin index)
        // hist.bins()가 256이고, gain이 2.0이라면 minBin은 128이됨
        // 128 아래로는 픽셀이 포화되지 않고, 128 이상부터는 포화된 픽셀임을 의미함
		double minBin = std::min(1.0, 1.0 / gain) * hist.bins();
        // 픽셀이 포화되지 않은 영역의 Y값의 평균을 계산함 (픽셀이 포화될 수 있는 영역은 제외)
		double binMean = hist.interBinMean(0.0, minBin);
        // 픽셀이 포화되지 않은 영역의 픽셀 개수
		double numUnsaturated = hist.cumulativeFreq(minBin);
        // 픽셀이 포화되지 않은 영역의 평균에 gain을 곱하고, 픽셀 갯수를 곱해 픽셀의 총 휘도 합을 구함
		double ySum = binMean * gain * numUnsaturated;
		/* And add the ones that will saturate. */
        // 전체 픽셀 개수에서 포화되지 않은 영역의 픽셀 개수를 빼서 포화될 픽셀 개수를 구함
        // 그리고 해당 픽셀들은 모두 가장 높은 휘도값(예. 256 == 전체 구간의 갯수 == hist.bins()) 가진다고 가정하여
        // 포화될 픽셀의 휘도 합을 총 휘도 합에 더함
		ySum += (hist.total() - numUnsaturated) * hist.bins();
        // 평균 휘도 = 총 휘도 합 / 픽셀 전체 개수
        // 정규화 : 평균 휘도 / 전체 구간의 개수 => 일관성 있게 사용하고자 0 ~ 1 사이의 표준화된 범위로 변환
		return ySum / hist.total() / hist.bins();
	}

    // 영역별 가중치의 개수와 agc 영역의 개수가 일치하는지 확인
	ASSERT(weights.size() == stats->agcRegions.numRegions());

    // weights는 이미 IPA(Image Processing Alogrithm)에 의해 통계 데이터에 직접 적용된 상태임, 그래서 아래 코드에는 weights 말고 gain만 사용됨
    // agcRegions를 순회하면서 각 RGB의 합인 rSum, gSum, bSum에 gain을 곱해 새로운 sum.r(), sum.g(), sum.b()를 구함
    // 이때, xSum * gain이 포화 상태를 넘어갈 경우, 포화된 값을 사용함
    // maxVal - 1 : 센서가 측정할 수 있는 최대 코드 값 (4096 - 1)
    // region.counted : 현재 영역에 포함된 픽셀의 총 개수
	RGB<double> sum{ 0.0 };
	double pixelSum = 0;
	for (unsigned int i = 0; i < stats->agcRegions.numRegions(); i++) {
		auto &region = stats->agcRegions.get(i);
		sum.r() += std::min<double>(region.val.rSum * gain, (maxVal - 1) * region.counted);
		sum.g() += std::min<double>(region.val.gSum * gain, (maxVal - 1) * region.counted);
		sum.b() += std::min<double>(region.val.bSum * gain, (maxVal - 1) * region.counted);
		pixelSum += region.counted;
	}

    // pixelSum : region.counted의 누적이 0일 경우, 0을 반환
	if (pixelSum == 0.0) {
		LOG(RPiAgc, Warning) << "computeInitialY: pixelSum is zero";
		return 0;
	}

    // agc의 통계가 WB 이전이라면, sum에 RGB gain을 곱함
	if (stats->agcStatsPos == Statistics::AgcStatsPos::PreWb)
		sum *= RGB<double>{ { awb.gainR, awb.gainG, awb.gainB } };

    // RGB 색상의 합을 휘도 값의 합으로 변환
	double ySum = ipa::rec601LuminanceFromRGB(sum);

    // 평균 휘도 : 총 휘도 합 / 픽셀 전체 개수
    // 정규화 : 평균 휘도 / (1 << 16) => 센서가 16비트로 고정되었거나 16비트 스케일로 정규화 처리되고 있음을 의미
	return ySum / pixelSum / (1 << 16);
}
```

#### constraintComputeGain

> 히스토그램에서 주요 영역의 평균 휘도를 계산하여, 해당 constraint에서의 gain 값인 'targetY / 평균 휘도'를 단순 계산하여 반환

```cpp
static double constraintComputeGain(AgcConstraint &c, const Histogram &h, double lux,
				    double evGain, double &targetY)
{
    // 현재 lux와 evGain(사용자 설정 ev)를 통해 targetY를 결정
    // 여기서 targetY는 정규화된 target Y임
	targetY = c.yTarget.eval(c.yTarget.domain().clamp(lux));
	targetY = std::min(EvGainYTargetLimit, targetY * evGain);

    // 히스토그램 중 qLo, qHi를 잘라 평균읠 왜곡하는 것을 방지 (예 : 10% ~ 90% 영역만 사용)
    // iqm : 해당 영역의 평균 휘도
	double iqm = h.interQuantileMean(c.qLo, c.qHi);

    // 정규화된 targetY * 최대 스케일 => 역정규화 (센서 데이터 스케일)
    // gain = 센서 데이터 스케일의 targatY / 현재 측정 평균 휘도
	return (targetY * h.bins()) / iqm;
}
```

#### AgcChannel::computeTargetExposure

> gain을 통해 `target_.totalExposure`를 계산해 냄

- 일반적인 경우, `target_.totalExposure = current_.exposureTime * current_.analogueGain * gain` 을 계산됨

```cpp
void AgcChannel::computeTargetExposure(double gain)
{
	if (status_.fixedExposureTime && status_.fixedAnalogueGain) {
        // EIT와 AnalogueGain이 모두 고정인 경우, 특정 색상 채널이 너무 낮아 색상 왜곡이 발생할 수 있음
        // AWB 게인 중 가장 작은 gain을 찾아 이 gain의 역수(1/minColourGain)만큼 디지털 게인이 최소한 적용될 수 있도록 totalExposure를 강제 설정함
		double minColourGain = std::min({ awb_.gainR, awb_.gainG, awb_.gainB, 1.0 });
		ASSERT(minColourGain != 0.0);
		target_.totalExposure =
			status_.fixedExposureTime * status_.fixedAnalogueGain / minColourGain;
	} else {
        // EIT나 AnalogueGain 둘 중 하나라도 고정 되지 않은 경우(자동인 경우)
        // target_.totalExposure = current_.totalExposureNoDG * gain; 가 됨
        // current_.totalExposureNoDG = current_.exposureTime * current_.analogueGain;임
        target_.totalExposure = current_.totalExposureNoDG * gain;

		// ExposureMode에 따른 maxTotalExposure를 구함
		Duration maxExposureTime = status_.fixedExposureTime
					 ? status_.fixedExposureTime
					 : exposureMode_->exposureTime.back();
		maxExposureTime = limitExposureTime(maxExposureTime);
		Duration maxTotalExposure =
			maxExposureTime *
			(status_.fixedAnalogueGain != 0.0
				 ? status_.fixedAnalogueGain
				 : exposureMode_->gain.back());

        // 최종적으로 계산된 totalExposure와 ExposureMode에 따른 maxTotalExposure 중 작은 것을 최종 target_.totalExposure로 선정함
		target_.totalExposure = std::min(target_.totalExposure, maxTotalExposure);
	}
}
```

#### AgcChannel::filterExposure

> `target_.totalExposure`가 적용될 때, 너무 급격하게 변하는 것을 방지하고, 노출의 안전성 확보를 위해 `filtered_.totalExposure`, speed, stableRegion을 조정

- speed : 얼마나 빠르게 target\_.totalExposure에 도달할 것인가
- stableRegion : totalExposure가 stableRegion 사이에 있을 경우, Exposure를 변경시키지 않기 위해 설정하는 안정 영역

```cpp
void AgcChannel::filterExposure()
{
	double speed = config_.speed;
	double stableRegion = config_.stableRegion;

    // 고정 노출 (수동 모드) 이거나, 초기 시작 frame인 경우 노출을 바로 잡아야함
	if ((status_.fixedExposureTime && status_.fixedAnalogueGain) ||
	    frameCount_ <= config_.startupFrames) {
		speed = 1.0; // 바로 적용
		stableRegion = 0.0; // 안정 영역 무시
    }

	if (!filtered_.totalExposure) {
        // filtered_totalExposure가 없는 경우, target_.totalExposure로 초기화
		filtered_.totalExposure = target_.totalExposure;
	} else if (filtered_.totalExposure * (1.0 - stableRegion) < target_.totalExposure &&
		   filtered_.totalExposure * (1.0 + stableRegion) > target_.totalExposure) {
        // target_.totalExposure가 안정 영역에 있는 경우, 아무것도 하지 않음
        // 노출이 미세하게 떨리는 것을 방지함
	} else {
		/*
		 * If close to the result go faster, to save making so many
		 * micro-adjustments on the way. (Make this customisable?)
		 */
         // filtered_.totalExposure 가 target_.totalExposure의 20% 범위로 들어오면 speed를 높임
         // (speed가 0~1 사이의 값이기 때문에 루트를 씌우면 오히려 커지게 됨)
         // 목표에 근접했을 때, 남은 미세 조정을 빠르게 완료하기 위함
		if (filtered_.totalExposure < 1.2 * target_.totalExposure &&
		    filtered_.totalExposure > 0.8 * target_.totalExposure)
			speed = sqrt(speed);

        // 현재 speed로 target_.totalExposure에서 filtered_.totalExposure를 계산해냄
		filtered_.totalExposure = speed * target_.totalExposure +
					  filtered_.totalExposure * (1.0 - speed);
	}
	LOG(RPiAgc, Debug) << "After filtering, totalExposure " << filtered_.totalExposure
			   << " no dg " << filtered_.totalExposureNoDG;
}
```

#### AgcChannel::applyChannelConstraints

> 각 채널별 Constraints를 순회하며, 다른 채널의 Contraint도 모두 만족할 수 있는 `filtered_.totalExposure`를 정함

```cpp
bool AgcChannel::applyChannelConstraints(const AgcChannelTotalExposures &channelTotalExposures)
{
	bool channelBound = false;
	LOG(RPiAgc, Debug)
		<< "Total exposure before channel constraints " << filtered_.totalExposure;

	for (const auto &constraint : config_.channelConstraints) {
		LOG(RPiAgc, Debug)
			<< "Check constraint: channel " << constraint.channel << " bound "
			<< (constraint.bound == AgcChannelConstraint::Bound::UPPER ? "UPPER" : "LOWER")
			<< " factor " << constraint.factor;
		if (constraint.channel >= channelTotalExposures.size() ||
		    !channelTotalExposures[constraint.channel]) {
			LOG(RPiAgc, Debug) << "no such channel or no exposure available- skipped";
			continue;
		}

		libcamera::utils::Duration limitExposure =
			channelTotalExposures[constraint.channel] * constraint.factor;
		LOG(RPiAgc, Debug) << "Limit exposure " << limitExposure;
		if ((constraint.bound == AgcChannelConstraint::Bound::UPPER &&
		     filtered_.totalExposure > limitExposure) ||
		    (constraint.bound == AgcChannelConstraint::Bound::LOWER &&
		     filtered_.totalExposure < limitExposure)) {
			filtered_.totalExposure = limitExposure;
			LOG(RPiAgc, Debug) << "Constraint applies";
			channelBound = true;
		} else
			LOG(RPiAgc, Debug) << "Constraint does not apply";
	}

	LOG(RPiAgc, Debug)
		<< "Total exposure after channel constraints " << filtered_.totalExposure;

	return channelBound;
}
```

#### AgcChannel::applyDigitalGain

> AWB에 의한 색상 왜곡 방지를 위한 dg를 계산하고, 해당 dg로 인해 하이라이트가 포화될 위험에 있을 때 dg를 줄여 빠른 노출 감소를 통해 desaturate를 유도

```cpp
bool AgcChannel::applyDigitalGain(double gain, double targetY, bool channelBound)
{
    // RGB 채널 중 가장 낮은 게인을 가진 채널의 정보가 디지털 노이즈 아래로 사라지는 것을 방지
    // 모든 채널의 밝기를 가장 낮은 게인을 받은 채널과 동일하게 올리는 효과
	double minColourGain = std::min({ awb_.gainR, awb_.gainG, awb_.gainB, 1.0 });
	ASSERT(minColourGain != 0.0);
	double dg = 1.0 / minColourGain;
	/*
	 * I think this pipeline subtracts black level and rescales before we
	 * get the stats, so no need to worry about it.
	 */
	LOG(RPiAgc, Debug) << "after AWB, target dg " << dg << " gain " << gain
			   << " target_Y " << targetY;

    // 이미지가 너무 밝아 하이라이트가 포화되려고 할때, 노출을 빠르게 줄여 desaturate(탈색)을 유도하여 포화상태에서 빠르게 벗어나게 하는 로직
    // desaturate 설정이 on 되어 있는 경우, channelBound가 되지 않았고, targetY가 fastReduceThreshold(포화 위험임계값)보다 높고, gain이 루트 targetY 보다 작을 때 == 이미지가 너무 밝아 노출을 줄여야할 때
    // dg = dg / config_.fastReduceThreshold 하고
    // filtered_.totalExposureNoDG = filtered_totalExposure / dg 로 설정
    // desaturate 여부를 반환
	bool desaturate = false;
	if (config_.desaturate)
		desaturate = !channelBound &&
			     targetY > config_.fastReduceThreshold && gain < sqrt(targetY);
	if (desaturate)
		dg /= config_.fastReduceThreshold;
	LOG(RPiAgc, Debug) << "Digital gain " << dg << " desaturate? " << desaturate;
	filtered_.totalExposureNoDG = filtered_.totalExposure / dg;
	LOG(RPiAgc, Debug) << "Target totalExposureNoDG " << filtered_.totalExposureNoDG;
	return desaturate;
}
```

#### AgcChannel::divideUpExposure

> 순수 아날로그 노출값(filtered\_.totalExposureNoDG)를 flickerMode까지 고려하여 exposureTime과 analogueGain로 분배함

- exposureTime \* analogueGain이 exposureValue보다 작은경우, ExposureMode의 exposureTime 배열과 gain 배열을 순차적으로 증가시키며 크거나 같도록 만듦
- exposureTime \* analogueGain이 exposureValue보다 큰 경우는 이미 전 단계에서 처리됨(?)

```cpp
void AgcChannel::divideUpExposure()
{
	/*
	 * Sending the fixed exposure time/gain cases through the same code may
	 * seem unnecessary, but it will make more sense when extend this to
	 * cover variable aperture.
	 */

    // 초기 exposureValue, exposureTime, analogueGain 값을 설정
	Duration exposureValue = filtered_.totalExposureNoDG;
	Duration exposureTime;
	double analogueGain;
	exposureTime = status_.fixedExposureTime ? status_.fixedExposureTime
						 : exposureMode_->exposureTime[0];
	exposureTime = limitExposureTime(exposureTime);
	analogueGain = status_.fixedAnalogueGain != 0.0 ? status_.fixedAnalogueGain
							: exposureMode_->gain[0];
	analogueGain = limitGain(analogueGain);

    // 현재 exposureTime * analogueGain이 exposureValue보다 작다면
    // exposureMode의 exposureTime 배열과 gain 배열을 순차적으로 적용하며, exposureTime * analogueGain이 exposureValue보다 크거나 같을 때까지 exposureTime과 gain을 순차적으로 늘려나감
	if (exposureTime * analogueGain < exposureValue) {
        // exposureMode의 gain 배열을 순회하며
		for (unsigned int stage = 1;
		     stage < exposureMode_->gain.size(); stage++) {
            // exposureTime을 먼저 늘려보고
			if (!status_.fixedExposureTime) {
				Duration stageExposureTime =
					limitExposureTime(exposureMode_->exposureTime[stage]);
				if (stageExposureTime * analogueGain >= exposureValue) {
                    // exposureTime * analogueGain이 exposureValue보다 크다면 순회 종료
					exposureTime = exposureValue / analogueGain;
					break;
				}
				exposureTime = stageExposureTime;
			}
            // anoglueGain을 늘려보고
			if (status_.fixedAnalogueGain == 0.0) {
                // exposureTime * analogueGain이 exposureValue보다 크다면 순회 종료
				if (exposureMode_->gain[stage] * exposureTime >= exposureValue) {
					analogueGain = exposureValue / exposureTime;
					break;
				}
				analogueGain = exposureMode_->gain[stage];
				analogueGain = limitGain(analogueGain);
			}
		}
	}
	LOG(RPiAgc, Debug)
		<< "Divided up exposure time and gain are " << exposureTime
		<< " and " << analogueGain;
	/*
	 * Finally adjust exposure time for flicker avoidance (require both
	 * exposure time and gain not to be fixed).
	 */

     // flicker방지 모드이고, exposureTime과 analogueGain이 고정이 아니라면
	if (!status_.fixedExposureTime && !status_.fixedAnalogueGain &&
	    status_.flickerPeriod) {
		int flickerPeriods = exposureTime / status_.flickerPeriod;
		if (flickerPeriods) {
            // exposureTime을 flikerPeriod의 정수 배로 맞추고
            // 해당 analogueGain을 줄어든 만큼의 ExposureTime만큼 증가시켜 밝기를 보상함
			Duration newExposureTime = flickerPeriods * status_.flickerPeriod;
			analogueGain *= exposureTime / newExposureTime;

			analogueGain = std::min(analogueGain, exposureMode_->gain.back());
			analogueGain = limitGain(analogueGain);
			exposureTime = newExposureTime;
		}
		LOG(RPiAgc, Debug) << "After flicker avoidance, exposure time "
				   << exposureTime << " gain " << analogueGain;
	}
	filtered_.exposureTime = exposureTime;
	filtered_.analogueGain = analogueGain;
}
```

#### AgcChannel::writeAndFinish

> imageMetadata의 `agc.status`에 totalExposureValue(filtered), targetExposureValue(target\_), exposure, exposureTime, analogueGain을 새김

```cpp
void AgcChannel::writeAndFinish(Metadata *imageMetadata, bool desaturate)
{
	status_.totalExposureValue = filtered_.totalExposure;
	status_.targetExposureValue = desaturate ? 0s : target_.totalExposure;
	status_.exposureTime = filtered_.exposureTime;
	status_.analogueGain = filtered_.analogueGain;
	/*
	 * Write to metadata as well, in case anyone wants to update the camera
	 * immediately.
	 */
	imageMetadata->set("agc.status", status_);
	LOG(RPiAgc, Debug) << "Output written, total exposure requested is "
			   << filtered_.totalExposure;
	LOG(RPiAgc, Debug) << "Camera exposure update: exposure time " << filtered_.exposureTime
			   << " analogue gain " << filtered_.analogueGain;
}
```
