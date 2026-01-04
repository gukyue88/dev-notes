---
layout: single
title: "[linux] 라즈베리파이 libcamera AGC 코드 분석"
categories: system/linux
published: false
---

코드 : [raspberrypi/libcamera](https://github.com/raspberrypi/libcamera)

## Overview

#### 주요 객체

- CameraManager : 시스템에 연결된 Camera 장치들 enumerate 및 관리
- Camera : 물리적인 카메라 장치 추상화
- Stream : 카메라 센서의 이미지 데이터 출구
- Request : 단일 프레임 캡처를 위한 요청 객체 (FrameBuffer 및 MetaData 포함)
  - FrameBuffer : 캡처된 실제 이미지 데이터를 담는 버퍼
  - MetaData : 이미지 센서로 부터 얻은 정보와 해당 캡처를 위한 설정값(게인, 노출 시간등)을 담음
- IPA (Image Processing Algorithm) : 이미지 센서의 RAW 데이터를 사람이 볼 수 있는 최종 이미지로 가공하는 알고리즘의 집합, AGC(자동 게인 제어), AWB(자동 화이트 밸런스), NR(노이즈 제거) 등

#### 초기화 흐름

1. 애플리케이션이 CameraManager 생성하고 시스템의 카메라 검색
2. 원하는 Camera 객체를 얻고, StreamConfiguration을 통해 해상도, 픽셀형식 등 설정

#### 캡처 흐름

1.  Request 생성 : 빈 FrameBuffer를 가진 Request 객체 생성
2.  Request 제출 : Camera의 queueRequest()를 사용해 request를 제출
3.  이미지 캡처 : request를 하나 꺼내옴. 해당 request의 메타데이터에 있는 설정 값으로 이미지 센서를 세팅하고, 이미지 데이터를 캡처하여 해당 request의 FrameBuffer에 저장
4.  IPA 처리 : 캡처 완료 후, FrameBuffer를 포함한 request가 IPA 모듈로 전달며고, IPA는 데이터를 분석해 AGC, AWB등 알고리즘을 실행
5.  피드백 : IPA에서 계산된 노출 값 등이 다음 프레임의 Request에 메타데이터로 포함됨

#### lux & status

#### 전체 구조

> 일단 ipa 관련 전체 구조 파악이 좀 더 필요하고 그 다음 상세 내용 파악하자!

- vc4 : Video Core 4 칩셋 (raspberry pi 4 이전)
- vc6 : Video Core 6 (rasberry pi 4부터), 이건 코드에 직접 보이지는 않음
  - BCM2711 SoC를 지원하는 파이프라인 핸들러 안에 코드가 구현됨
  -

> Documentation/gudes/pipeline-handler.rst 를 보면 주요 구조체 같은게 보이는것 같음
> Documentation/guides/ipa.rst 도 존재함

## AGC 관련 주요 코드

- 파이프라인핸들러(pisp)가 초기화시 카메라 튜닝파일 (imx708.json)을 읽어들임
- 이 안에는 rpi.agc에 관한 내용 및 기타 튜닝 내용들이 들어있음
- rpi.channel에는 normal AGC, short HDR, long HDR, night mode 채널이 들어있음
- AgcChannel::process()가 메인인듯

#### src/ipa/rpi/controller/rpi/agc_channel.cpp

```cpp
void AgcChannel::process(StatisticsPtr &stats, DeviceStatus const &deviceStatus, Metadata *imageMetadata, const AgcChannelTotalExposures &channelTotalExposures) {
  frameCount_++;

  housekeepConfig();
  fetchCurrentExposure(deviceStatus);

  double gain, targetY;
  computeGain(stats, imageMetadata, gain, targetY);
  computeTargetExposure(gain);
  filterExposure();

  bool channelBound = applyChannelConstraints(channelTotalExposures);
  bool desaturate = applyDigitalGain(gain, targetY, channelBound);
  divideUpExposure();

  writeAndFinish(imageMetadata, desaturate);
}
```

- HousekeepConfig(): `status_` 내 값들 업데이트 (업데이트된 configuration이 있는지 보고 업데이트 하는 과정)
  - ev : 노출값
  - fixedExposureTime : 고정 노출 시간
  - fixedAnalogueGain : 고정 아날로그 게인
  - flickerPeriod : 플리커 주기
  - meteringMode : 측광모드(센터, 스팟, 매트릭스 등)
  - exposureMode : 노출모드(자동, 매뉴얼, 셔터스피드 우선 모드)
  - constraintMode : 제약모드(일반, 하이라이트:밝은부분과다노출방지모드, 쉐도우:어두운부데테일을살리는모드)
- fetchCurrentExposure(): deviceStatus의 값으로 현재 노출값(`current_`) 설정
  - `current_.exposureTime = deviceStatus.exposureTime;`
  - `current_.analogueGain = deviceStatus.analogueGain;`
  - `current_.totalExposureNoDG = deviceStatus.exposureTime * deviceStatus.analogueGain;`
- computeGain() :
  - imageMetadata의 lux.status를 확인함 (이 lux.status는 이전 프레임에서 agc가 계산해서 넣어놓았던 값)
- computeTargetExposure(): 현재 노출 대비 필요한 total gain 계산
  - 현재 프레임의 밝기(Lux)를 기반으로 **이상적인 노출량(Target Exposure)**을 계산합니다. 이 값은 다음 프레임이 가져야 할 전체적인 밝기 목표를 의미합니다.
- diviceUpExposure(): **computeTargetExposure()**에서 계산된 총 노출량을 아날로그 게인, 노출 시간, 디지털 게인 세 가지 요소로 나누는 역할을 합니다. 보통 아날로그 게인을 먼저 최대한 활용하고, 그 이후에 노출 시간을 늘리며, 마지막으로 디지털 게인을 적용하는 순서로 분할합니다.
- applyChannelConstraints(): **diviceUpExposure()**에서 계산된 게인과 노출 시간 값이 카메라 센서나 드라이버가 지원하는 최대/최소 한계를 벗어나지 않도록 보정합니다.
- applyDigitalGain(): diviceUpExposure()의 결과, 아날로그 게인만으로 목표 노출량에 도달하기 어려울 때 디지털 게인을 추가적으로 적용합니다. 디지털 게인은 노이즈를 증가시킬 수 있어 마지막에 사용됩니다.
- filterExposure(): 계산된 노출 값을 필터링하여 부드럽게 만듭니다. 이는 밝기 변화가 급격할 때 영상이 깜빡이는 플리커(Flicker) 현상을 방지하는 데 필수적인 단계입니다.
- writeAndFinish(): 위의 모든 계산이 완료된 후, 최종적으로 결정된 게인 및 노출 시간 값을 다음 프레임의 Request 객체 메타데이터에 기록하고 함수를 종료합니다.
