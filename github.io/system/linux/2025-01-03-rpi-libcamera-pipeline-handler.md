---
layout: single
title: "[linux][라즈베리파이 libcamera Doc] pipeline-handler"
categories: system/linux
published: false
---

repository : [raspberrypi/libcamera](https://github.com/raspberrypi/libcamera)
doc : libcamera/doc/pipeline-handler.rst

# Pilpeline Handler 작성자 가이드

- pipeline handlers는 device-specific hardware configuration 를 위한 추상화 계층임
- pipeline handlers는 v4l2와 media controller kernel interface를 통해 하드웨어를 제어
- 파이프라인의 캡처 컴포넌트와 ISP를 직업 제어하기 위한 내무 API를 구현함

## 필수 지식 : 시스템 아키텍처

- pipeline handler는 이미지 획득 및 transformation pipeline을 구성하고 관리함
- 이미지 획득 및 transformation pipeline은 데이터/제어 버스를 통해 시스템에 연결된 이미지 소스와 결합된 시스템 주변장치(peripherals)에 의해 구현됨
  - 여기서 이미지 소스는 카메라를, 데이터 버스는 mipi csi data lane을 제어 버스는 I2C를 의미하는 것 같음
  - 그럼 카메라와 결합된 시스템 주변 장치는 무엇을 말하는 것일까? video card 같은 것이 달려있는걸 의미하는 것 같기도 함
- 주변장치(peripherals) 존재여부, 개수, 특성은 시스템 설계 및 타겟 플랫폼의 제품 통합 방식(?)에 따라 달라짐

## 3가지 시스템 구성 요소

#### 1. Input ports

- 외부장치와의 인터페이스
- 여기서 외부장치(주로 이미지 센서)는 physical 버스에서 다른 시스템 주변장치(peripherals)가 접근할 수 있는 영역으로 데이터를 전송함
- Input port는 입력 이미지 형식과 크기에 맞게 구성되어야함.
- 또, input port는 수신된 이미지에 대해 일반적으로 자르기/스케일링 및 일부 포맷 변환과 같은 기번 변환을 선택적으로 적용할 수 있음
- libcamera가 주로 타게팅하는 시스템에 대한 산업 표준은 MIPI CSI-2 사양을 준수하는 수신기를 가지는 것임.
- 이는 MIPI D-PHY나 MIPI C-PHY와 같은 호환 가능한 물리 계층 상에 구현됨.
- 다른 설계도 가능하지만, 덜 일반적임

#### 2. Image Signal Processor (ISP)

- 이미지 스트림에 디지털 변환을 적용하는 미디어 특화 프로세서
- ISP는 SoC 내부에 시스템 주변장치(Peripherals)로 통합되거나, 버스를 통해 AP에 연결된 독립형 칩으로 존재할 수 있음
- libcamera에서 사용하는 대부분의 하드웨어는 대부분 SoC 내부에 통합된 ISP를 활용함.
- 하지만, 외부 ISP 침이나 GPU 또는 FPGAS IP프블록과 같은 다른 시스템 자원을 사용하도록 구성할 수 있음
- ISP는 소프트웨어 프로그래밍 인터페이스를 제공함.
- 이 SW Programming Interface는 "이미지 변환 파이프라인"을 구성하는 여러 processing 블록의 구성을 하게해 줌.
- ISP는 메타데이터 + processed image 스트림을 생성함
- 메타데이터는 해당 output frames를 생성하기 위해 적용된 처리 과정이 적혀있음

#### 3. Camera Sensor

- 렌즈 + 이미지센서를 통합한 디지털 컴포넌트
- SoC 이미지 수신 포트와 인터페이스함
- 현재 시스템 구성에 적합한 형식과 크기의 이미지를 생성하도록 프로그래밍됨
- ISP 통합된 카메라 모듈은 시스템에 이미지를 전달하기 전 이미지 처리를 하기도 함
- ISP 프로세서가 시스템에서는 일반적으로 Raw Bayer 형식의 이미지를 생성하는 카메라 센서를 사용함.

## 파이프 라인의 역할

- 시스템 구성요소 (Input ports, ISP, Camera Sensor)를 인터페이스함 (연결)
- 시스템에 사용 가능한 카메라 장치를 감지 및 관련 이미지 스트림 셋과 함께 등록
- 애플리케이션이 요청한 구성을 충족할 수 있도록, 시스템 리소스(메모리, 공유 컴포넌트 등)을 할당하여 이미지 획득 및 처리 파이프 라인 구성
- 이미지 획득 및 처리 시작/정지
- 애플리케이션이 요청한 세팅 및 libcaemra의 이미지 처리 알고리즘에 의해 계산된 세팅을 하드웨어 디바이스에 적용
- 새로운 이미지의 availability를 애플리케이션에게 알리고, 올바른 위치에 이미지를 전달

## 사전 지식 : libcamera architecture

파이프라인 핸들러는 위에 설명된 기능을 구현하기 위해 아래의 libcamera 클래스들을 활용함.

#### MediaDevice

- kernel media controller device 와 이것과 연결된 객체들과 관련된 클래스
- kernel media controller device : v4l2의 서브시스템에서 제공하는 기능, 이미지센서, 비디오 인코더/디코더와 같은 여러 미디어 하드웨어 구성요소를 관리하고 제어
- 연결된 객체들 : MediaDevice에 연결된 하드웨어 구성요소를 의미, MediaDevice는 메인 컨트롤러에 해당되며, 센서, 렌즈, ISP등이 연결된 객체가 됨
- MediaDevice는 이러한 모든 구성 요소를 단일 인터페이스로 통합하여 제어하는 역할임

#### DeviceEnumerator

- 시스템에 연결된 모든 미디어 장치와 미디어 엔티티를 찾아내고 목록화함
- 미디어 장치 : 카메라, 비디오 캡처장치
- 미디어 엔티티 : 미디어 파이프라인을 구성하는 개별요소, 이미지센서, 렌즈, ISP등이
- DeviceEnumeratorsms 미디어 장치를 발견할 때마다, 해당 장치를 대표하는 MediaDevice 클래스의 새로운 객체를 생성함

#### DeviceMatch

- 미디어 장치 검색 패턴을 정의하는 클래스
- 엔티티의 이름이나 다른 속성을 사용하여 원하는 장치를 찾을 때 활용

#### V4L2VideoDevice

- v4l2 video device node path(`/dev/video`)를 사용하여 구성된 v4l2 video device의 인스턴스를 모델링\

#### V4L2SubDevice

- v4l2 device의 하드웨어 컴포넌트를 모델링하는 sub-devices에 대한 API를 제공

#### CameraSensor

- v4l2 subdevice kernel API의 복잡한 세부사항을 숨기고 센서 정보를 캐싱하여 카메라 센서 제어를 추상화

#### Camera::Private

- 파이프라인 핸들러가 각 `Camera` 인스턴스에 연결하는 device-specific 데이터를 나타냄

#### StreamConfiguration

- 카메라가 생성하는 이미지 스트림의 configuration을 모델링
- 이미지 스트림의 format과 size를 알려줌

#### CameraConfiguration

- 카메라의 현재 설정을 나타내는 클래스
- 이 설정에는 캡처 세션에서 활성화된 각 스트림에 대한 StreamConfiguration 목록이 포함됨
- validater 과정을 거친 뒤, 카메라에 적용됨

#### IPAInterface

- Image Processing Algorithim 모듈에 대한 인터페이스
- 이미지 처리 파이프라인의 튜닝 매개변수를 계산하는 역할을 수행함

#### ControlList

- 제어 항목(control items)의 목록
- `Control<>` 인스턴스 또는 숫자 인덱스로 접근 가능
- 애플리케이션과 IPA가 이미지 스트림의 매개변수를 변경하는데 사용되는 값을 담고 있음
- 또한, 캡처된 이미지와 관련된 메타데이터를 애플리케이션에 반환하고 IPA와 공유하는데 사용됨
- 시스템 초기화시 열거된 변경 불가능한 카메라 특성을 알리는 역할도 함

## 파이프라인 핸들러 생성하기

이 가이드는 v4l2 가상 비디오 테스트 드라이버(vivid)를 지원하는 간단한 파이프라인 핸들러인 'Vivid"를 만드는 단계를 안내함.

- vivid test driver를 사용하려면, vivid 커널 모듈이 load 되어 있는지 확인해야함

```sh
modprobe vivid
```

#### 스켈레톤 파일 구조 생성

- 새 파이프라인 핸들러를 추가하려면, `src/libcamera/pipeline/` 디렉토리 안에 디렉토리를 만들어야 함
- 생성되는 디렉토리 이름은 파이프라인 이름과 일치 (이 경우에는 `src/libcamera/pipeline/vivid`)
- 새 디렉토리 안에 `meson.build` 파일(libcamera 빌드 시스템과 통합)과 파이프라인 이름과 일치하는 `vivid.cpp` 파일 추가
- `meson.build` 파일에 `vivid.cpp` 파일을 build source 로 추가하려면, 전역 meson `libcamera_internal_sources` 변수에 넣어줌

```
libcamera_internal_sources += files([
    'vivid.cpp',
])
```

- libcamera 유저는 libcamera를 빌드할 때 `pipelines` 옵션을 사용하여 파이프라인을 선택적으로 활성화할 수 있음
- 예를 들어, IPU3, UVC, VIVID 파이프라인만 활성화하려면, 빌드 디렉터리를 생성할 때, `~Dpipelines`와 함께 쉽표로 구분된 목록으로 ㅣ정

```
meson build -Dpipelines=ipu3,uvcvideo,vivid
```

- 추가적인 빌드 디렉토리 configuration에 관해서는 `Meson build configuration 문서` 참조
- 이 옵션의 리스트에 새로운 파이프라인 핸들러를 추가하기 위해서, 해당 디렉토리 이름을 최상위 `meson_options.txt`파일의 libcamera build options에 추가

```
option('pipelines',
        type : 'array',
        choices : ['ipu3', 'rkisp1', 'rpi/pisp', 'rpi/vc4', 'simple', 'uvcvideo', 'vimc', 'vivid'],
        description : 'Select which pipeline handlers to include')
```

- `vivid.cpp` 파일 안에 파이프라인 핸들러를 `libcamera` 네임스페이스에 추가
- `PipelineHandler`를 상속한 `PipelineHandlerVivid`라는 클래스를 정의
- override된 클래스 멤버 구현

```cpp
namespace libcamera {

class PipelineHandlerVivid : public PipelineHandler
{
public:
       PipelineHandlerVivid(CameraManager *manager);

       CameraConfiguration *generateConfiguration(Camera *camera,
       Span<const StreamRole> roles) override;
       int configure(Camera *camera, CameraConfiguration *config) override;

       int exportFrameBuffers(Camera *camera, Stream *stream,
       std::vector<std::unique_ptr<FrameBuffer>> *buffers) override;

       int start(Camera *camera, const ControlList *controls) override;
       void stopDevice(Camera *camera) override;

       int queueRequestDevice(Camera *camera, Request *request) override;

       bool match(DeviceEnumerator *enumerator) override;
};

PipelineHandlerVivid::PipelineHandlerVivid(CameraManager *manager)
       : PipelineHandler(manager)
{
}

CameraConfiguration *PipelineHandlerVivid::generateConfiguration(Camera *camera,
                                                                 Span<const StreamRole> roles)
{
       return nullptr;
}

int PipelineHandlerVivid::configure(Camera *camera, CameraConfiguration *config)
{
       return -1;
}

int PipelineHandlerVivid::exportFrameBuffers(Camera *camera, Stream *stream,
                                             std::vector<std::unique_ptr<FrameBuffer>> *buffers)
{
       return -1;
}

int PipelineHandlerVivid::start(Camera *camera, const ControlList *controls)
{
       return -1;
}

void PipelineHandlerVivid::stopDevice(Camera *camera)
{
}

int PipelineHandlerVivid::queueRequestDevice(Camera *camera, Request *request)
{
       return -1;
}

bool PipelineHandlerVivid::match(DeviceEnumerator *enumerator)
{
       return false;
}

REGISTER_PIPELINE_HANDLER(PipelineHandlerVivid, "vivid")

} /* namespace libcamera */
```

> `REGISTER_PIPELINE_HANDLER`를 통해 `PipelineHandler`의 서브클래스를 반드시 등록해야 함!

- 이 `REGISTER_PIPELINE_HANDLER` 매크로는 글로벌 심볼을 생성하여 클래스가 참조할 수 있게 하며, 장치를 시도해 매칭할 수 있도록 해줌
- `"vivid"`라는다자열은 파이프라인에 할당된 이름이며, 실제 소스 트리의 파이프라인 하위 디렉터리 이름과 일치함
- 개발중 파이프라인 핸들러를 디버깅하고 테스트
  - 사용자가 `LOG_DEFINE_CATEGORY` 매크로를 통해 파이프라인 핸들러에 대한 로그 메시지 카테고리를 정의할 수 있음
  - `LIBCAMERA_LOG_LEVELS` 환경 변수를 통해 유저가 원하는 로그를 뽑을 수 있음

```cpp
#include <libcamera/base/log.h>
#include "libcamera/internal/pipeline_handler.h"

LOG_DEFINE_CATEGORY(VIVID)
```

- 빌드

```sh
meson build
ninja -C build
```

- libcamera code base를 빌드하고, 빌드 시스템이 새로운 파이프 라인 핸들러를 찾았는지 확인하려면 아래 내용 실행

```sh
LIBCAMERA_LOG_LEVELS=Camera:0 ./build/src/cam/cam -l
```

- 그럼 결과는 아래와 같을 것임

```sh
DEBUG Camera camera_manager.cpp:148 Found registered pipeline handler 'PipelineHandlerVivid'
```

## Maching devices

- 각 파이프라인 핸들러는 `DviceMatch`를 시스템의 `DeviceEnumerator`와 매칭하여, 현재 시스템 configuration에 대해 테스트함
- 이 테스트가 끝나면 요청된 모든 컴포넌트가 시스템에 등록되었음을 확인하고, 파이프라인 핸들러가 초기화 될 수 있도록 허용함.
- 파이프라인의 main entry point는 `match()` 클래스 멤버 함수임
- `CameraManager`가 `start()` 함수를 이용해 시작되면,
- 등록된 모든 파이프라인 핸들러는 순차적으로 호출되며,
- 시스템에서 찾은 모든 장치에 대한 enumerator와 함께 각 핸들러의 `match`함수가 호출됨
- `match` 함수는 파이프라인이 지원하는 적절한 장치들이 `DeviceEnumerator`에 있는지 확인하는 역할을 함
- 장치가 일치하면 `true`, 그렇지 않으면 `false`를 return함
- 이 작업을 수행하려면 `MediaController` 장치의 이름을 사용해, `DeviceMatch` 클래스를 만들어야 함.
- `DeviceMatch`dp `.add()` 함수를 사용해 특정 미디어 엔티티를 추가함으로써 검색을 더욱 구체화할 수 있음
- 이 예제는 `vivid`와 일치하는 검색 패턴을 사용하지만, 새로운 파이프라인 핸들러를 개발할 때는, 이 값을 해당 device 에 맞게 변경해야 함

`PipelineHandlerVivid::match` 함수 내용 수정

```cpp
DeviceMatch dm("vivid");
dm.add("vivid-000-vid-cap");
return false; // Prevent infinite loops for now
```

- 디바이스 매칭 기준이 정의되면, `aquireMediaDevice` 함수를 사용하여 일치하는 미디어 컨트롤러 장치에 대한 `exclusive acess` 권한을 획득하려고 시도함
- 만약 이 함수가 이미 일치된 장치에 접근을 시도하면 `false`를 반환함.

```cpp
MediaDevice *media = acquireMediaDevice(enumerator, dm);
if (!media)
        return false;
```

파이프라인 핸들러에 디바이스 열거 기능을 위한 include 항목 추가

```cpp
#include "libcamera/internal/device_enumerator.h"
```

- 아직 libcamera가 애플리케이션에 보고할 카메라를 만드는 코드를 추가하지 않은 단계임
- 이 상태에서 파이프라인 핸들러가 장치를 성공적으로 매칭하는지 테스트 필요
- 임시 유효성 검사단계로, 디버그 프린트를 추가하여 확인

```cpp
LOG(VIVID, Debug) << "Vivid Device Identified";
```

- `PipelineHandlerVivid::match`함수의 최종 return 문 앞에, 파이프라인 핸들러가 `MediaDevice`와 `MediaEntity` 이름을 성공적으로 매칭할 때 위 내용을 추가.
- 다시 빌드하고 실행하여 파이프라인 핸들러가 장치를 매칭하고 찾는지 테스트

```sh
ninja -C build
LIBCAMERA_LOG_LEVELS=Pipeline,VIVID:0 ./build/src/cam/cam -l
```

그러면 출력이 아래와 같아야 함

```sh
DEBUG VIVID vivid.cpp:74 Vivid Device Identified
```

## camera devices 생성

- 파이프라인 핸들러가 실행 중인 시스템과 성공적으로 매칭되면
- 필요한 모든 `V4L2VideoDevice`, `V4L2Subdevice`, `CameraSensor` 하드웨어 추상화 클래스의 인스턴스를 생성하여 초기화할 수 있음
- 파이프라인 핸들러가 ISP를 지원하는 경우, 카메라 장치 생성 전, IPA 모듈을 초기화할 수 있음
- 이미지 `Stream`은 알려진 format과 size의 이미지와 데이터 시퀀스를 나타냄
- 또, `Stream`은 애플리케이션이 접근 가능한 메모리 위치에 저장됨
- 일반적인 스트림의 예시로 ISP 처리된 출력물과 수신기 포트 출력에서 캡처된 원본 이미지가 있음
- 파이프라인 핸들러는 카메라와 관련된 `Stream` set을 정의하는 역할을 함
- 각 카메라는 `Camera::Private` 클래스를 사용하여 나타내지는 instance-specific 데이터를 가지고 있음
- 이는 파이프라인 핸들러의 특정 요구 사항에 맞게 확장될 수 있음
- 향후 등록할 카메라를 지원하기 위해, 우리 파이프라인 핸들러에 맞게 구현할 수 있는 `Camera::Private` 클래스를 생성해야 함
- 파이프라인 핸들러 클래스 정의 앞에 `Camera::Private`를 상속받는 `VividCameraPrivate` 클래스를 정의

```cpp
class VividCameraData : public Camera::Private
{
public:
      VividCameraData(PipelineHandler *pipe, MediaDevice *media)
            : Camera::Private(pipe), media_(media), video_(nullptr)
      {
      }

      ~VividCameraData()
      {
            delete video_;
      }

      int init();
      void bufferReady(FrameBuffer *buffer);

      MediaDevice *media_;
      V4L2VideoDevice *video_;
      Stream stream_;
};
```

- 여기 예시의 파이프라인 핸들러는 `VividCameraData` 클래스 멤버로 표현되는 단일 비디오 장치를 다루고, 하나의 스트림을 지원함
- 더 복잡한 파이프라인 핸들러는 여러 비디오 장치와 서브 장치로 구성된 카메라, 혹은 이미지 캡처 파이프라인의 여러 구성 요소를 나타내는 카메라 마다의 여러 스트림을 등록할 수 있음
- 맞춤형 `PipelineHandler`를 개발할 때는 이 모든 구성 요소를 `Camera::Private` 파생 클래스에 포함시켜야 함
- 우리가 만든 예시인 `VividCameraData` 에서는 파이프라인 핸들러로부터 객체를 준비하기 위한 `init()` 함수를 구현함
- 하지만, `Camera::Private` 클래스에는 초기화를 위한 인터페이스가 명시되어있지 않으며, 파이프라인 핸들러는 각자의 필요에 따라 이를 관리할 수 있음
- `Camera::Private`를 상속받은 클래스는 해당 파이프라인 핸들러에 의해서만 사용됨
- `Camera::Private` 클래스는 각 카메라 인스턴스에 필요한 컨텍스트를 저장하며, 일반적으로 캡처 파이프라인에서 사용되는 모든 장치를 open 하는 역할을 담당함
- 이제 예시 파이프라인 핸들러의 `init` 함수를 구현하여 미디어 엔티티로부터 새로운 V4L2 비디오 장치를 생성할 수 있음
- 이는 `MediaDevice`의 `MediaDevice::getEntityByName` 함수를 사용하여 지정할 수 있음
- 우리의 예시는 단순한 Vivid 테스트 장치를 기반으로 하므로, 장치에서 `vivid-000-vid-cap`이라고 명명된 단일 캡처 장치만 열면 됨

```cpp
int VividCameraData::init()
{
      video_ = new V4L2VideoDevice(media_->getEntityByName("vivid-000-vid-cap"));
      if (video_->open())
            return -ENODEV;

      return 0;
}
```

The VividCameraData should be created and initialised before we move on to
register a new Camera device so we need to construct and initialise our
VividCameraData after we have identified our device within
PipelineHandlerVivid::match(). The VividCameraData is wrapped by a
std::unique_ptr to help manage the lifetime of the instance.

If the camera data initialization fails, return `false` to indicate the
failure to the `match()` function and prevent retrying of the pipeline
handler.

- 카메라 장치를 새로 등록하기 전에 `VividCameraData`를 생성하고 초기화 해야함
- 따라서 `PipelineHandlerVivid::match()` 내에서 장치 확인 후, `VividCameraData`를 구성하고 초기화 해야함
- `VividCameraData` 인스턴스는 수명 관리를 돕기 위해 `std::unique_ptr`로 감싸짐
- 만약 카메라 데이터 초기화에 실패하면 `match()`함수에서 `false`를 반환해야함

```cpp
std::unique_ptr<VividCameraData> data = std::make_unique<VividCameraData>(this, media);

if (data->init())
        return false;
```

Once the camera data has been initialized, the Camera device instances and the
associated streams have to be registered. Create a set of streams for the
camera, which for this device is only one. You create a camera using the static
`Camera::create`_ function, passing the Camera::Private instance, the id of the
camera, and the streams available. Then register the camera with the pipeline
handler and camera manager using `registerCamera`_.

Finally with a successful construction, we return 'true' indicating that the
PipelineHandler successfully matched and constructed a device.

- 카메라 데이터가 초기화되면, 카메라 장치 인스턴스와 연관된 스트림들을 등록해야함
- 이 장치의 경우 단 하나의 스트림만 있으므로, 이를 위한 스트림 집합을 생성함
- 그런 다음, 정적 함수인 `Camera::create`를 사용하여 `Camera::Private` 인스턴스, 카메라 ID, 그리고 사용가능한 스트림들을 전달해 카메라를 생성함.
- 이후 `registerCamera`를 사용하여 파이프라인 핸들러와 카메라 매니저에 카메라를 등록함
- 이러한 구성이 성공적으로 완료되면, `true`를 반환하여 파이프라인 핸들러 장치가 성공적으로 매칭하고 초기화 되었음을 알림

```cpp
std::set<Stream *> streams{ &data->stream_ };
std::shared_ptr<Camera> camera = Camera::create(std::move(data), data->video_->deviceName(), streams);
registerCamera(std::move(camera));

return true;
```

최종 `match()` 함수

```cpp
bool PipelineHandlerVivid::match(DeviceEnumerator *enumerator)
{
  DeviceMatch dm("vivid");
  dm.add("vivid-000-vid-cap");

  MediaDevice *media = acquireMediaDevice(enumerator, dm);
  if (!media)
    return false;

  std::unique_ptr<VividCameraData> data = std::make_unique<VividCameraData>(this, media);

  /* Locate and open the capture video node. */
  if (data->init())
    return false;

  /* Create and register the camera. */
  std::set<Stream *> streams{ &data->stream_ };
  std::shared_ptr<Camera> camera = Camera::create(std::move(data), data->video_->deviceName(), streams);
  registerCamera(std::move(camera));

  return true;
}
```

- 파이프라인 핸들러 전체에서 커스텀 `VividCameraData`를 자주 사용하게 됨
- 따라서 `Camera::Private` 인스턴스로 부터 `VividCameraData` 인스턴스를 가져오고 캐스팅하는 헬퍼함수를 추가

```cpp
private:
    VividCameraData *cameraData(Camera *camera)
    {
            return static_cast<VividCameraData *>(camera->_d());
    }
```

지금 단계에서는 카메라 인터페이스와 장치 상호 작용 인터페이스를 제공하기 위해 새로운 include를 추가

```cpp
#include <libcamera/camera.h>
#include "libcamera/internal/media_device.h"
#include "libcamera/internal/v4l2_videodevice.h"
```

## Registering controls and properties

- libcamera의 `controls framework`는 애플리케이션이 매 프레임별 streams capture parameter를 설정할 수 있도록 함
- `Camera` 장치의 변경 불가능한 속성을 알리는데도 사용됨
- libcameradml controls 및 properties는 YAML 형식으로 정의됨
- YAML 형식은 자동으로 문서 및 인터페이스를 생성하기 위해 처리됨
- controls는 `src/libcamera/control_ids_core.yaml` 파일에 정의됨
- camera properties는 `src/libcamera/property_ids_core.yaml` 파일에 정의됨
- 파이프라인 핸들러는 애플리케이션이 설정할 수 있는 컨트롤 목록과 변경 불가능한 카메라 속성 목록을 선택적으로 등록할 수 잇음
- 이들은 모두 카메라에 특화된 값으로, `Camera::Private`의 멤버인 `Camera::Private::controlInfo_`와 `Camera::Private::properties_`필드에 표현됨
- `controlInfo_` 필드는 `ControlId` 인스턴스의 맵을 나타냄
- 각 인스턴스에는 해당 컨트롤이 지원하는 유효한 값의 한계가 연결됨
- 더 자세한 사항은 `ControlInfoMap` 클래스 문서 참고
- 파이프라인 핸들러는 튜닝 가능한 장치 및 IPA 매개변수를 애플리케이션에 노출하기 위해 컨트롤을 등록함.
- 예시 파이프라인 핸들러는 지원되는 각 V4L2 컨트롤에 대해 연관된 값을 가진 ControlId 인스턴스를 등록함으로써 비디오 장치의 기본적인 컨트롤만 노출하지만, 이는 V4L2 컨트롤을 libcamera ControlID에 매핑하는 방법을 보여줌
- `VividCameraData::init()` 함수에 다음 코드를 추가하여 `VividCameraData` 클래스의 초기화(initialization)를 완료하고 컨트롤을 초기화하자
- 더 복잡한 컨트롤 설정의 경우 별도의 함수로 분리할 수도 있지만, 지금은 `VividCameraData`의 `init` 함수 내에서 소수의 컨트롤을 인라인으로 초기화함.

```cpp
/* Initialise the supported controls. */
const ControlInfoMap &controls = video_->controls();
ControlInfoMap::Map ctrls;

for (const auto &ctrl : controls) {
        const ControlId *id;
        ControlInfo info;

        switch (ctrl.first->id()) {
        case V4L2_CID_BRIGHTNESS:
                id = &controls::Brightness;
                info = ControlInfo{ { -1.0f }, { 1.0f }, { 0.0f } };
                break;
        case V4L2_CID_CONTRAST:
                id = &controls::Contrast;
                info = ControlInfo{ { 0.0f }, { 2.0f }, { 1.0f } };
                break;
        case V4L2_CID_SATURATION:
                id = &controls::Saturation;
                info = ControlInfo{ { 0.0f }, { 2.0f }, { 1.0f } };
                break;
        default:
                continue;
        }

        ctrls.emplace(id, info);
}

controlInfo_ = ControlInfoMap(std::move(ctrls), controls::controls);
```

- `properties_` 필드는 변경 불가능한 값과 관련된 ControlId 인스턴스 목록임
- 이 값들은 애플리케이션이 시스템 내 카메라 장치를 식별하는 데 사용할 수 있는 정적인 특성들을 나타냄.
- properties는 비디오 장치와 카메라 센서의 V4L2 컨트롤 값을 검사하여 등록될 수 있음 (예: 카메라의 위치와 방향을 검색하는 경우)
- 또는 다른 변경 불가능한 특성들을 표현하는 데 사용될 수 있음.
- 예시 파이프라인 핸들러에는 등록된 속성이 없지만, libcamera 코드 베이스에서 예시를 찾아볼 수 있음.

이 시점에서 컨트롤 처리를 위해 다음 인클루드들을 파일 맨 위에 추가해야 함

```cpp
#include <libcamera/controls.h>
#include <libcamera/control_ids.h>
```

## specific controls & properties

- 벤더(vendor) 고유의 컨트롤과 속성은 별도의 YAML 파일에 정의되어야 함
- `include/libcamera/meson.build`에 파이프라인 핸들러와 파일의 매핑을 정의함으로써 빌드에 포함됨.
- 이러한 YAML 파일은 src/libcamera 디렉터리에 위치함
- 예를 들어, PiSP 파이프라인 핸들러를 위한 라즈베리 파이 벤더 컨트롤 파일을 추가하는 것은 다음 매핑을 통해 수행됨

```meson
controls_map = {
  'controls': {
      'draft': 'control_ids_draft.yaml',
      'libcamera': 'control_ids_core.yaml',
      'rpi/pisp': 'control_ids_rpi.yaml',
  },

  'properties': {
      'draft': 'property_ids_draft.yaml',
      'libcamera': 'property_ids_core.yaml',
  }
}
```

- 위에서 언급된 파이프라인 핸들러 이름은 meson 빌드 설정에 명시된 파이프라인 핸들러 옵션 문자열과 일치해야 함
- 벤더(vendor) 고유의 컨트롤과 속성은 YAML 파일에 `vendor: <vendor_string>` 태그를 반드시 포함해야 함.
- 모든 고유한 벤더 태그는 `src/libcamera/control_ranges.yaml` 파일에서 고유하고 중복되지 않는 예약된 컨트롤 ID 범위를 정의해야 함.
- 예를 들어, 다음 블록은 `rpi` 벤더 태그를 가진 벤더 고유의 컨트롤을 정의함

```yaml
vendor: rpi
controls:
  - PispConfigDumpFile:
      type: string
      description: |
        Triggers the Raspberry Pi PiSP pipeline handler to generate a JSON
        formatted dump of the Backend configuration to the filename given by the
        value of the control.
```

- 컨트롤은 벤더 고유의 네임스페이스인 `libcamera::controls::rpi`에서 생성됨
- 추가로, 애플리케이션이 이 컨트롤들의 사용 가능 여부를 테스트할 수 있도록 `#define LIBCAMERA_HAS_RPI_VENDOR_CONTROLS`가 제공됨

## default configuration 생성

- 카메라 장치와 연관된 스트림들이 등록되면, 애플리케이션은 프레임 캡처 세션을 준비하기 위해 카메라를 획득하고 설정하는 절차를 진행할 수 있음.
- 애플리케이션은 원하는 이미지 크기와 픽셀 형식을 담은 `StreamConfiguration` 인스턴스를 활성화할 각 스트림에 할당함으로써 요청된 설정을 명시함
- 이 스트림 설정들은 파이프라인 핸들러가 검사하고 지원 가능한 설정으로 조정하기 위해 유효성을 검사하는 `CameraConfiguration`에 묶이게 됨.
- 이 과정에는 장치의 기능에 맞도록 포맷이나 이미지 크기 또는 정렬을 조정하는 작업이 포함될 수 있음
- 애플리케이션은 유효성 검사 단계를 반복하며, 애플리케이션의 요구에 적합한 유효한 `StreamConfigurations`가 반환될 때까지 매개변수를 조정할 수 있음
- 파이프라인 핸들러가 유효한 카메라 설정을 받으면, 이미지 스트림 설정을 사용하여 하드웨어 장치에 설정을 적용할 수 있음
- 이러한 설정 및 유효성 검사 프로세스는 공통 기본 구현 및 인터페이스에서 파생된 또 다른 파이프라인 특정 클래스로 관리됨
- 예시 파이프라인 핸들러에서 유효성 검사를 지원하기 위해, 기본 `CameraConfiguration` 클래스에서 파생된 `VividCameraConfiguration`이라는 새 클래스를 만들어 `PipelineHandlerVivid` 클래스 내에서 구현하고 사용하자
- 파생된 `CameraConfiguration` 클래스는 스트림 설정 검사 및 조정이 발생하는 기본 클래스의 `validate()` 함수를 오버라이드(override)해야 함

```cpp
class VividCameraConfiguration : public CameraConfiguration
{
public:
        VividCameraConfiguration();

        Status validate() override;
};

VividCameraConfiguration::VividCameraConfiguration()
        : CameraConfiguration()
{
}
```

- 응용 프로그램은 `Camera::generateConfiguration()` 함수를 호출하여 CameraConfiguration 인스턴스를 생성
- 이 함수는 오버라이드된 `PipelineHandler::generateConfiguration()` 함수의 파이프라인 구현으로 이어짐
- 구성(Configurations)은 StreamRole 인스턴스 목록을 받아 생성됨.
- libcamera는 이 StreamRole을 응용 프로그램이 카메라를 사용하려는 미리 정의된 방식(전체 목록은 StreamRole API 문서 참조)으로 활용함.
- 이는 응용 프로그램이 스트림을 어떻게 사용할지에 대한 선택적 힌트이며, 파이프라인 핸들러는 요청된 각 역할에 대한 이상적인 구성을 반환해야 함.
- 파이프라인 핸들러 generateConfiguration 구현에서 return nullptr;를 제거하고, CameraConfiguration 파생 클래스의 새 인스턴스를 생성하여 이를 기본 클래스 포인터에 할당해야 함

```cpp
auto config = std::make_unique<VividCameraConfiguration>();
```

- Pipeline 핸들러 코드 경로에서만 특정 파이프라인의 `CameraConfiguration`을 생성할 수 있음.
- 애플리케이션은 또한 빈 구성을 생성하고 원하는 스트림 구성을 수동으로 추가할 수 있음.
- 파이프라인은 요청된 역할이 없을 경우 빈 구성을 반환함으로써 이를 허용해야 함
- `PipelineHandlerVivid`에서 이를 지원하려면, `generateConfiguration`에서 `CameraConfiguration`이 생성된 후에 다음 확인 코드를 추가해야함.

```cpp
if (roles.empty())
        return config;
```

- 생산용 파이프라인 핸들러는 카메라 장치가 지원하는 모든 적절한 스트림 역할에 대해 `StreamConfiguration`을 생성해야 gka.
- 이 간단한 예시(하나의 스트림만 존재)의 경우, 파이프라인 핸들러는 기본 `V4L2VideoDevice`로부터 추론된 동일한 구성을 항상 반환함
- 아래에 그 방법이 나와 있지만, 더 복잡한 예시를 살펴보려면 IPU3, RKISP1 및 라즈베리 파이용으로 더 많은 기능이 구현된 파이프라인들을 검토하는 것이 좋음
- `StreamConfiguration`을 생성하려면, 스트림의 출력으로 지원되는 픽셀 포맷과 프레임 크기 목록이 필요함.
- 기본 출력 장치에서 지원하는 `V4LPixelFormat` 및 `SizeRange` 맵을 가져올 수 있지만, 파이프라인 핸들러는 이를 애플리케이션에 전달하기 위해 `libcamera::PixelFormat` 타입으로 변환해야 함.
- 아래 코드 예시를 `generateConfiguration` 구현에 이어서 추가

```cpp
std::map<V4L2PixelFormat, std::vector<SizeRange>> v4l2Formats =
        data->video_->formats();
std::map<PixelFormat, std::vector<SizeRange>> deviceFormats;

for (auto &[v4l2PixelFormat, sizes] : v4l2Formats) {
        PixelFormat pixelFormat = v4l2PixelFormat.toPixelFormat();
        if (pixelFormat.isValid())
                deviceFormats.try_emplace(pixelFormat, std::move(sizes));
}
```

- `StreamFormats` 클래스는 스트림이 지원할 수 있는 픽셀 포맷과 프레임 크기에 대한 정보를 담고 있음
- 이 클래스는 픽셀 포맷을 기준으로 크기 정보를 그룹화함
- 아래 코드는 `StreamFormats` 클래스를 사용하여 지원되는 모든 픽셀 포맷과 그에 연관된 프레임 크기 목록을 나타냄
- 그런 다음, 애플리케이션이 단일 스트림을 구성하는 데 사용할 수 있는 정보를 모델링하기 위해 지원되는 `StreamConfiguration`을 생성함
- 이를 지원하기 위해 다음 코드를 계속해서 추가

```cpp
StreamFormats formats(deviceFormats);
StreamConfiguration cfg(formats);
```

- 지원되는 `StreamFormats` 목록 외에도, `StreamConfiguration`은 초기화된 기본 구성을 제공해야 함.
- 이는 임의적일 수 있지만, 사용 사례에 따라 센서 출력과 일치하는 출력을 선택하거나 하드웨어에서 더 높은 성능을 제공할 수 있는 픽셀 포맷을 선호할 수 있음
- `bufferCount`는 이 스트림에서 기능적이고 연속적인 처리를 지원하는 데 필요한 버퍼의 개수를 나타냄

```cpp
cfg.pixelFormat = formats::BGR888;
cfg.size = { 1280, 720 };
cfg.bufferCount = 4;
```

- 생성된 `StreamConfiguration`을 `CameraConfiguration`에 추가하고, 애플리케이션에 반환하기 전에 반드시 유효성을 검사해야 함.
- 이 코드에서는 지원되는 스트림이 하나뿐이므로 단 하나의 `StreamConfiguration`만 추가함
- 하지만 더 많은 스트림을 처리할 수 있는 장치에서는 각 지원 역할에 대해 `StreamConfiguration`을 추가해야 함.
- `generateConfiguration`의 구현을 완성하기 위해 다음 코드를 추가

```cpp
config->addConfiguration(cfg);

config->validate();

return config;
```

- 카메라 구성을 검증하기 위해, 파이프라인 핸들러는 파생 클래스에서 `CameraConfiguration::validate()` 함수를 구현하여 모든 연관 스트림 구성을 검사하고, 유효하게 만들기 위해 필요한 조정을 한 후, 유효성 검사 상태를 반환해야 함.
- 변경 사항이 있으면 구성은 **`Adjusted`**로 표시되지만, 요청된 구성이 지원되지 않고 조정될 수 없는 경우에는 **`Invalid`**로 거부되어 표시됨
- 유효성 검사 단계는 요청된 구성이 모든 플랫폼별 제약 조건을 준수하는지 확인함
- 가장 간단한 예시는 요청된 이미지 형식이 지원되는지, 이미지 정렬 제약 조건이 준수되는지 확인하는 것임
- `validate()`의 파이프라인 핸들러별 구현은 수신된 모든 구성 매개변수를 검사해야 하며, 응용 프로그램이 구성이 생성된 후에도 요청된 스트림 매개변수를 자유롭게 변경할 수 있으므로, 그 내용이 정확하다고 가정해서는 안 됨
- 이 예시 파이프라인 핸들러는 더 간단하므로, 실제 사례를 보려면 더 복잡한 구현들을 살펴보자
- 파일에 다음 함수 구현을 추가

```cpp
CameraConfiguration::Status VividCameraConfiguration::validate()
{
        Status status = Valid;

        if (config_.empty())
              return Invalid;

        if (config_.size() > 1) {
              config_.resize(1);
              status = Adjusted;
        }

        StreamConfiguration &cfg = config_[0];

        const std::vector<libcamera::PixelFormat> &formats = cfg.formats().pixelformats();
        if (std::find(formats.begin(), formats.end(), cfg.pixelFormat) == formats.end()) {
              cfg.pixelFormat = formats[0];
              LOG(VIVID, Debug) << "Adjusting format to " << cfg.pixelFormat.toString();
              status = Adjusted;
        }

        cfg.bufferCount = 4;

        return status;
}
```

- `PixelFormat` 타입을 다루고 있으므로, 코드를 다시 빌드하고 테스트하기 전에 인클루드 섹션에 `#include <libcamera/formats.h>`를 추가해야 함

```sh
ninja -C build
LIBCAMERA_LOG_LEVELS=Pipeline,VIVID:0 ./build/src/cam/cam -c vivid -I
```

- 새로운 파이프라인 핸들러의 기능과 구성이 생성되었음을 보여주는 다음 출력이 나타나야 함

```sh
Using camera vivid
0: 1280x720-BGR888
* Pixelformat: NV21 (320x180)-(3840x2160)/(+0,+0)
- 320x180
- 640x360
- 640x480
- 1280x720
- 1920x1080
- 3840x2160
* Pixelformat: NV12 (320x180)-(3840x2160)/(+0,+0)
- 320x180
- 640x360
- 640x480
- 1280x720
- 1920x1080
- 3840x2160
* Pixelformat: BGRA8888 (320x180)-(3840x2160)/(+0,+0)
- 320x180
- 640x360
- 640x480
- 1280x720
- 1920x1080
- 3840x2160
* Pixelformat: RGBA8888 (320x180)-(3840x2160)/(+0,+0)
- 320x180
- 640x360
- 640x480
- 1280x720
- 1920x1080
- 3840x2160
```

## Configuring a device

- 컨피규레이션이 생성되고 (선택적으로 수정 및 재검증된 후), 파이프라인 핸들러에는 애플리케이션이 하드웨어 장치에 컨피규레이션을 적용할 수 있도록 하는 함수가 필요함.
- `PipelineHandler::configure()` 함수는 유효한 `CameraConfiguration`을 받아서, 해당 매개변수를 사용하여 원하는 속성을 가진 스트리밍 세션을 위해 장치를 준비하는 등, 설정을 하드웨어 장치에 적용함
- 임시로 만들어진 PipelineHandlerVivid::configure 함수의 내용을 다음 코드로 대체하여 카메라 데이터와 스트림 컨피규레이션을 가져옴
- 이 파이프라인 핸들러는 단일 스트림만 지원하므로, 카메라 컨피규레이션에서 첫 번째 StreamConfiguration을 직접 가져옴
- 여러 스트림을 가진 파이프라인 핸들러는 각 StreamConfiguration을 검사하고 그에 따라 시스템을 구성해야 함

```cpp
VividCameraData *data = cameraData(camera);
StreamConfiguration &cfg = config->at(0);
int ret;
```

- Vivid 캡처 장치는 V4L2 비디오 장치이므로, 우리는 fourcc와 크기 속성을 가진 `V4L2DeviceFormat`을 사용하여 캡처 장치 노드에 직접 적용함.
- fourcc 속성은 `V4L2PixelFormat`으로, `libcamera::PixelFormat`과는 다름
- 포맷을 변환하려면 다중 평면 포맷에 대한 평면 구성에 대한 지식이 필요하므로, 포맷이 적용될 `V4L2VideoDevice` 인스턴스가 제공하는 헬퍼 함수인 `V4L2VideoDevice::toV4L2PixelFormat()`을 사용하여 명시적으로 변환해야 함
- 위 코드 아래에 다음 코드를 추가

```cpp
V4L2DeviceFormat format = {};
format.fourcc = data->video_->toV4L2PixelFormat(cfg.pixelFormat);
format.size = cfg.size;
```

- V4L2VideoDevice::setFormat()` 함수를 사용하여 위에서 정의한 비디오 장치 포맷을 설정함
- 커널 드라이버가 포맷을 조정했는지 확인해야 함.
- 만약 조정했다면 파이프라인 핸들러가 유효성 검사 단계를 제대로 처리하지 못한 것이므로, 설정 작업 또한 실패해야 함.

```cpp
ret = data->video_->setFormat(&format);
if (ret)
      return ret;

if (format.size != cfg.size ||
      format.fourcc != data->video_->toV4L2PixelFormat(cfg.pixelFormat))
      return -EINVAL;
```

- 마지막으로, 스트림의 상태를 반영하는 스트림별 데이터를 저장하고 설정해야 함
- StreamConfiguration::setStream 함수를 사용하여 설정을 스트림과 연결하고, 필요에 따라 개별 스트림 설정 멤버의 값을 설정

```cpp
cfg.setStream(&data->stream_);
cfg.stride = format.planes[0].bpl;

return 0;
```

## device controls 초기화

- 파이프라인 핸들러는 시스템 설정 시점에 비디오 장치와 카메라 센서 컨트롤을 선택적으로 초기화하여, 정상적인 값으로 기본 설정될 수 있도록 함.
- 장치 컨트롤 처리는 다시 libcamera의 **controls framework**를 사용해 수행됨
- 이 섹션은 컨트롤의 초기값을 커널 드라이버가 정의한 **Vivid Controls**와 일치하도록 설정하기 때문에 Vivid에 특히 해당됨
- 이 코드는 사용자의 파이프라인 핸들러에 필요하지 않지만, 파이프라인 핸들러가 필요로 할 수 있는 기능을 어떻게 구현하는지 보여주는 예시로 포함됨
- 우리는 편의를 위해 파일 상단에 몇 가지 정의를 추가해야 합니다. 이 정의들은 커널 소스에서 직접 가져온 것임

```cpp
#define VIVID_CID_VIVID_BASE            (0x00f00000 | 0xf000)
#define VIVID_CID_VIVID_CLASS           (0x00f00000 | 1)
#define VIVID_CID_TEST_PATTERN          (VIVID_CID_VIVID_BASE  + 0)
#define VIVID_CID_OSD_TEXT_MODE         (VIVID_CID_VIVID_BASE  + 1)
#define VIVID_CID_HOR_MOVEMENT          (VIVID_CID_VIVID_BASE  + 2)
```

- 우리는 이제 V4L2 컨트롤 ID를 사용하여 `ControlList` 클래스로 컨트롤 목록을 준비하고, `ControlList::set()` 함수를 사용해 이 컨트롤들을 설정할 수 있음.
- 파이프라인의 `configure` 함수에서, 포맷이 설정되고 확인된 후에 다음 코드를 추가하여 `ControlList`를 초기화하고 장치에 적용하자

```cpp
ControlList controls(data->video_->controls());
controls.set(VIVID_CID_TEST_PATTERN, 0);
controls.set(VIVID_CID_OSD_TEXT_MODE, 0);

controls.set(V4L2_CID_BRIGHTNESS, 128);
controls.set(V4L2_CID_CONTRAST, 128);
controls.set(V4L2_CID_SATURATION, 128);

controls.set(VIVID_CID_HOR_MOVEMENT, 5);

ret = data->video_->setControls(&controls);
if (ret) {
      LOG(VIVID, Error) << "Failed to set controls: " << ret;
      return ret < 0 ? ret : -EINVAL;
}
```

- 이 컨트롤들은 VIVID가 기본 테스트 패턴을 사용하고, 모든 화면 표시 텍스트를 활성화하며, 적절한 밝기, 대비, 채도 값을 설정하도록 구성함.
- 개별 컨트롤을 설정하려면 **`controls.set`** 함수를 사용함

## Buffer Handling 및 stream control

- 시스템이 요청된 매개변수로 설정되면, 이제 애플리케이션은 Camera 장치로부터 프레임 캡처를 시작할 수 있음
- libcamera는 Request 인스턴스를 Camera 객체에 큐잉(queueing)하여 프레임별 요청 캡처 모델을 구현함
- 애플리케이션이 캡처 요청을 제출하기 전에, 캡처 파이프라인은 요청이 들어오는 즉시 프레임을 전달할 수 있도록 준비되어야 함
- 메모리는 초기화되어야 하고, 장치들이 이미지를 생성할 준비를 마친 상태에서 사용 가능해야 함
- 캡처 세션이 끝나면, 할당된 메모리를 정리하고 하드웨어 장치를 정지시키기 위해 Camera 장치도 중지되어야 함.
- 파이프라인 핸들러는 이러한 목적을 위해 **start()**와 stopDevice() 두 함수를 구현함
- start() 단계에서 이루어지는 메모리 초기화는 비디오 장치가 dma-buf 파일 디스크립터로 내보내진 메모리 버퍼를 사용할 수 있도록 구성하는 역할을 함
- 파이프라인 핸들러의 관점에서, 애플리케이션 측 스트림을 제공하는 비디오 장치는 항상 메모리 임포터(importer) 역할을 하며, V4L2 용어로는 V4L2_MEMORY_DMABUF 메모리 타입의 버퍼를 사용함
- libcamera는 exportFrameBuffers 함수와 FrameBufferAllocator 클래스를 통해 메모리를 할당하고 애플리케이션으로 내보내는 API도 제공함
- libcamera에서 비디오 메모리 버퍼는 FrameBuffer 클래스로 표현됨
- FrameBuffer 인스턴스는 캡처 요청의 일부인 각 **Stream**과 연결되어야 함
- 파이프라인 핸들러는 작업에 필요한 dma-buf 파일 디스크립터를 가져와 캡처 장치를 준비해야 함
- 이 작업은 V4L2VideoDevice API를 사용해 수행되며, 이 API는 비디오 장치를 적절히 준비시키는 importBuffers() 함수를 제공함
- start() 함수 구현을 시작하려면, 임시 버전을 다음 코드로 대체

```cpp
VividCameraData *data = cameraData(camera);
unsigned int count = data->stream_.configuration().bufferCount;

int ret = data->video_->importBuffers(count);
if (ret < 0)
      return ret;

return 0;
```

- 시작 단계에서 파이프라인 핸들러는 이미지 캡처 파이프라인의 여러 구성 요소(예: CSI-2 수신기와 ISP 입력 사이) 간에 데이터를 전송하는 데 필요한 모든 내부 버퍼 풀을 할당함
- 이 예시 파이프라인은 내부 풀이 필요하지 않지만, 더 복잡한 파이프라인 핸들러 예시는 libcamera 코드 베이스에서 찾을 수 있음
- 애플리케이션은 시스템의 다른 곳에서 메모리를 할당하는 대신 비디오 장치에 할당된 메모리를 사용하고 싶을 수 있음
- libcamera는 이러한 작업을 돕기 위해 `FrameBufferAllocator` 클래스를 제공함
- `FrameBufferAllocator`는 비디오 장치 내의 스트림을 위한 메모리를 예약하고 이를 dma-buf 파일 디스크립터로 내보냄
- 이 시점부터, 할당된 `FrameBuffer`들은 요청 내의 스트림 인스턴스들과 연결되며, 다른 곳에서 할당된 것처럼 파이프라인 핸들러에 의해 정확히 동일한 방식으로 가져와짐
- 파이프라인 핸들러는 `exportFrameBuffers` 함수를 구현하여 `FrameBufferAllocator` 작업을 지원함
- 이 함수는 스트림과 연관된 비디오 장치에 메모리를 할당하고 이를 내보냄
- 이러한 처리를 위해 `exportFrameBuffers` 임시 함수를 다음 코드로 구현

```cpp
unsigned int count = stream->configuration().bufferCount;
VividCameraData *data = cameraData(camera);

return data->video_->exportBuffers(count, buffers);
```

- 메모리 설정이 완료되면, 캡처 작업을 준비하기 위해 비디오 장치를 시작할 수 있습니다. 다음 코드를 추가하여 `start` 함수의 구현을 완성

```cpp
ret = data->video_->streamOn();
if (ret < 0) {
      data->video_->releaseBuffers();
      return ret;
}

return 0;
```

- 이 함수는 `streamOn` 함수를 사용하여 스트림과 연결된 비디오 장치를 시작함
- 호출이 실패하면 오류 값이 호출자에게 전파되고, `releaseBuffers` 함수는 장치를 일관된 상태로 남겨두기 위해 모든 버퍼를 해제함.
- 파이프라인 핸들러가 이미지 처리 알고리즘이나 다른 장치를 사용한다면, 그것들도 중지해야 함
- 물론, 장치에서 스트리밍을 중지하는 해당 작업도 처리해야 함
- `streamOff` 함수로 스트림을 중지하고 모든 버퍼를 해제하기 위해 `stopDevice()` 함수에 다음을 추가

```cpp
VividCameraData *data = cameraData(camera);
data->video_->streamOff();
data->video_->releaseBuffers();
```

## Queuing requests between applications and hardware

- libcamera는 애플리케이션이 카메라 장치에 큐(queue)에 넣는 캡처 요청을 기반으로 하는 스트리밍 모델을 구현함.
- 각 요청은 적어도 하나의 `Stream` 인스턴스와 연관된 `FrameBuffer` 객체를 포함합니다.
- 애플리케이션이 캡처 요청을 보내면, 파이프라인 핸들러는 활성화된 스트림에서 프레임을 생성하기 위해 어떤 비디오 장치에 버퍼를 제공해야 하는지 식별함.
- 이 예시 파이프라인 핸들러는 유일하게 지원되는 스트림에서 `findBuffer` 헬퍼 함수를 사용하여 버퍼를 식별하고, `V4L2VideoDevice`가 제공하는 `queueBuffer` 함수로 해당 버퍼를 캡처 장치에 직접 큐에 넣음.
- `queueRequestDevice`의 임시 내용을 다음으로 대체

```cpp
VividCameraData *data = cameraData(camera);
FrameBuffer *buffer = request->findBuffer(&data->stream_);
if (!buffer) {
      LOG(VIVID, Error)
              << "Attempt to queue request with invalid stream";

      return -ENOENT;
}

int ret = data->video_->queueBuffer(buffer);
if (ret < 0)
      return ret;

return 0;
```

## Processing controls

- 캡처 요청에는 스트림과 메모리 버퍼뿐만 아니라, 선택적으로 애플리케이션이 스트리밍 매개변수를 수정하기 위해 설정한 컨트롤 목록이 포함될 수 있음
- 애플리케이션은 `Registering controls and properties` 섹션에서 설명한 대로, 초기화 단계에서 파이프라인 핸들러가 등록한 컨트롤을 설정할 수 있음
- `queueRequestDevice` 함수 위에 `processControls` 함수를 구현하여, 각 요청과 함께 수신된 컨트롤 목록을 반복 처리하고 컨트롤 값을 검사
- 컨트롤은 설정되기 전에 libcamera의 컨트롤 범위 정의와 장치 상의 해당 값 사이에서 변환이 필요할 수 있음

```cpp
int PipelineHandlerVivid::processControls(VividCameraData *data, Request *request)
{
      ControlList controls(data->video_->controls());

      for (auto it : request->controls()) {
              unsigned int id = it.first;
              unsigned int offset;
              uint32_t cid;

              if (id == controls::Brightness) {
                    cid = V4L2_CID_BRIGHTNESS;
                    offset = 128;
              } else if (id == controls::Contrast) {
                    cid = V4L2_CID_CONTRAST;
                    offset = 0;
              } else if (id == controls::Saturation) {
                    cid = V4L2_CID_SATURATION;
                    offset = 0;
              } else {
                    continue;
              }

              int32_t value = std::lround(it.second.get<float>() * 128 + offset);
              controls.set(cid, std::clamp(value, 0, 255));
      }

      for (const auto &ctrl : controls)
              LOG(VIVID, Debug)
                    << "Setting control " << utils::hex(ctrl.first)
                    << " to " << ctrl.second.toString();

      int ret = data->video_->setControls(&controls);
      if (ret) {
              LOG(VIVID, Error) << "Failed to set controls: " << ret;
              return ret < 0 ? ret : -EINVAL;
      }

      return ret;
}
```

- `processControls` 함수는 요청 처리 시 내부 헬퍼 함수로만 사용되므로, `PipelineHandlerVivid` 클래스의 private 멤버 내에 해당 함수 프로토타입을 선언해야 함

```cpp
private:
    int processControls(VividCameraData *data, Request *request);
```

- **파이프라인 핸들러**는 요청에 포함된 컨트롤을 해당 하드웨어 장치에 적용하는 역할을 함
- 이는 캡처 장치에 직접 적용하거나, 필요한 경우 V4L2 서브 장치에 직접 컨트롤을 설정함으로써 이루어질 수 있음
- 각 파이프라인 핸들러는 자신이 지원하는 장치에 컨트롤을 적용하는 올바른 절차를 파악하고 있어야 함
- 이 예시 파이프라인 핸들러는 각 요청에 대해 `queueRequestDevice` 함수를 실행하는 동안 컨트롤을 적용하며, 캡처 노드를 통해 컨트롤을 캡처 장치에 적용함
- `queueRequestDevice` 함수에서 내용을 수정

```cpp
int ret = data->video_->queueBuffer(buffer);
if (ret < 0)
    return ret;
```

위 코드를 아래 코드로 대체

```cpp
int ret = processControls(data, request);
if (ret < 0)
    return ret;

ret = data->video_->queueBuffer(buffer);
if (ret < 0)
    return ret;
```

- 컨트롤 값 변환 연산을 지원하기 위해 다음 include 지시문을 추가해야 함

```cpp
#include <cmath>
```

## Frame completion and event handling

- libcamera는 이벤트를 처리하기 위해 이벤트 소스와 콜백을 연결하는 **시그널과 슬롯(signals and slots)** 메커니즘을 구현함
- **요약하자면,** `Slot`은 `Signal`에 연결될 수 있으며, **시그널이 발생하면 연결된 슬롯의 실행이 트리거됨**
- libcamera 구현에 대한 자세한 설명은 `libcamera Signal and Slot` 클래스 문서를 참조하세요.
- 새로운 프레임 및 데이터의 가용성을 애플리케이션에 알리기 위해, `Camera` 장치는 애플리케이션이 프레임 완료 이벤트에 대한 알림을 받을 수 있도록 두 개의 `Signal`을 노출함
- **`bufferComplete`** 시그널: 요청의 일부인 단일 `Stream`의 완료 이벤트를 애플리케이션에 보고하는 역할을 함
- **`requestComplete`** 시그널: 요청의 일부로 제출된 모든 스트림과 데이터의 완료를 알림
- 이 메커니즘을 통해 부분적인 요청 완료를 구현할 수 있으므로, 애플리케이션은 모든 버퍼가 준비될 때까지 기다리지 않고도 단일 스트림과 관련된 완료된 버퍼를 검사할 수 있음
- `bufferComplete`와 `requestComplete` 시그널은 파이프라인 핸들러로부터 받은 알림에 따라 `Camera` 장치에 의해 발생됨
- 파이프라인 핸들러는 버퍼 및 요청 완료 상태를 추적함
- 단일 버퍼 완료 알림은 파이프라인 핸들러가 버퍼를 큐에 넣었던 캡처 장치의 `bufferReady` 시그널을 완료된 프레임 처리를 담당하는 멤버 함수 슬롯에 연결함으로써 구현됨
- 버퍼가 준비되면, 파이프라인 핸들러는 `PipelineHandler` 기본 클래스의 `completeBuffer` 함수를 사용하여 해당 버퍼의 완료를 `Camera`에 전파해야 함
- 요청에 의해 참조된 모든 버퍼가 완료되면, 파이프라인 핸들러는 다시 `PipelineHandler` 기본 클래스의 `completeRequest` 함수를 사용하여 `Camera`에 알려야 함
- `PipelineHandler` 클래스 구현은 요청 완료 알림이 제출된 순서와 동일한 순서로 애플리케이션에 전달되도록 보장함
- `int VividCameraData::init()` 함수로 돌아가, 닫는 `return 0;` 위에 다음 코드를 추가하여 파이프라인 핸들러의 `bufferReady` 함수를 V4L2 장치 버퍼 시그널에 연결하자

```cpp
video_->bufferReady.connect(this, &VividCameraData::bufferReady);
```

- `VividCameraData::init()` 구현 뒤에 일치하는 `VividCameraData::bufferReady` 함수를 생성해야함
- `bufferReady` 함수는 `request` 함수를 사용하여 버퍼로부터 요청을 가져온 후, 버퍼와 요청이 완료되었음을 `Camera`에 알림.
- 이 간단한 파이프라인 핸들러에는 스트림이 하나뿐이므로 요청을 즉시 완료
- 여러 스트림을 지원하는 보다 복잡한 이벤트 처리 예시는 libcamera 코드베이스에서 찾을 수 있음
- 여러 스트림을 지원하는 경우 : 뷰파인더스트림/캡처스트림, 하나의 센서에서 여러 출력을 제공, ISP에서 여러 출력을 제공

```cpp
void VividCameraData::bufferReady(FrameBuffer *buffer)
{
      Request *request = buffer->request();

      pipe_->completeBuffer(request, buffer);
      pipe_->completeRequest(request);
}
```

## Testing a pipeline handler

- 파이프라인 핸들러를 빌드한 후, 코드베이스를 다시 빌드하고 `cam` 및 `qcam` 유틸리티를 모두 사용하여 파이프라인을 통한 캡처를 테스트할 수 있음

```sh
ninja -C build
./build/src/cam/cam -c vivid -C5
```

- 파이프라인 핸들러가 장치를 감지하고 입력을 캡처할 수 있는지 테스트하려면 위 명령어를 실행
- 그러면 픽셀 포맷에 대한 (많은) 정보가 출력된 후 프레임 데이터 캡처가 시작되며, 다음과 같은 출력이 나타나야 함

```
user@dev:/home/libcamera$ ./build/src/cam/cam -c vivid -C5
[42:34:08.573066847] [186470]  INFO IPAManager ipa_manager.cpp:136 libcamera is not installed. Adding '/home/libcamera/build/src/ipa' to the IPA search path
[42:34:08.575908115] [186470]  INFO Camera camera_manager.cpp:287 libcamera v0.0.11+876-7b27d262
[42:34:08.610334268] [186471]  INFO IPAProxy ipa_proxy.cpp:122 libcamera is not installed. Loading IPA configuration from '/home/libcamera/src/ipa/vimc/data'
Using camera vivid
[42:34:08.618462130] [186470]  WARN V4L2 v4l2_pixelformat.cpp:176 Unsupported V4L2 pixel format Y10
... <remaining Unsupported V4L2 pixel format warnings can be ignored>
[42:34:08.619901297] [186470]  INFO Camera camera.cpp:793 configuring streams: (0) 1280x720-BGR888
Capture 5 frames
fps: 0.00 stream0 seq: 000000 bytesused: 2764800
fps: 4.98 stream0 seq: 000001 bytesused: 2764800
fps: 5.00 stream0 seq: 000002 bytesused: 2764800
fps: 5.03 stream0 seq: 000003 bytesused: 2764800
fps: 5.03 stream0 seq: 000004 bytesused: 2764800
```

- 이는 파이프라인 핸들러가 프레임을 성공적으로 캡처하고 있음을 보여주지만, 시각적 출력을 보고 이미지가 올바르게 처리되고 있는지 확인하는 것이 도움이 됨
- libcamera 프로젝트는 또한 시각적 검사를 위해 프레임을 창에 렌더링하는 Qt 기반 애플리케이션을 구현함

```sh
./build/src/qcam/qcam -c vivid
```

---

## 개인 정리

#### 파이프라인 핸들러 생성 과정

src/libcamera/camera_manager.cpp

```cpp
// 1. 애플리케이션이 CameraManager 인스턴스를 생성하고, init()이 호출됨 (init 호출 부분은 잘 안보임)
// DeviceEnumerator 생성
// cretePipelineHandlers() 함수 호출
int CameraManager::Private::init()
{
	enumerator_ = DeviceEnumerator::create();
	createPipelineHandlers();
}

// 2. createPipelineHandlers()
// LIBCAMERA_PIPELINES_MATCH_LIST 환경변수를 읽어 순회하며,
// 해당 파이프라인의 대한 factory를 pipelineFactoryMatch()함수에 넣어 호출함
void CameraManager::Private::createPipelineHandlers()
{
	const char *pipesList =
		utils::secure_getenv("LIBCAMERA_PIPELINES_MATCH_LIST");
  for (const auto &pipeName : utils::split(pipesList, ",")) {
    const PipelineHandlerFactoryBase *factory;
    factory = PipelineHandlerFactoryBase::getFactoryByName(pipeName);
    pipelineFactoryMatch(factory);
  }
}

// 3. pipelineFactoryMatch
// 해당 파이프라인 핸들러의 match() 함수가 true를 반환할 때까지 loop
void CameraManager::Private::pipelineFactoryMatch(const PipelineHandlerFactoryBase *factory)
{
	CameraManager *const o = LIBCAMERA_O_PTR();

	/* Provide as many matching pipelines as possible. */
	while (1) {
		std::shared_ptr<PipelineHandler> pipe = factory->create(o);
		if (!pipe->match(enumerator_.get()))
			break;
	}
}
```

#### match 과정

src/libcamera/pipeline/vivid/vivid.cpp

```cpp
// DeviceMatch 객체를 만들고 "vivid-000-vid-cap" 캡처장치 이름 추가
// acquireMediaDevice 함수에 CameraManager에서 넘겨받은 enumerator와 생성한 DeviceMatch를 넘겨주어 호출함.
// return된 MediaDevice와 함께 VividCameraData 생성
// VividCameraData + stream set + device name으로 Camera 생성
// registerCamera를 통해 생성된 카메라 등록
bool PipelineHandlerVivid::match(DeviceEnumerator *enumerator)
{
  DeviceMatch dm("vivid");
  dm.add("vivid-000-vid-cap");

  MediaDevice *media = acquireMediaDevice(enumerator, dm);
  std::unique_ptr<VividCameraData> data = std::make_unique<VividCameraData>(this, media);

  /* Locate and open the capture video node. */
  if (data->init())
    return false;

  /* Create and register the camera. */
  std::set<Stream *> streams{ &data->stream_ };
  std::shared_ptr<Camera> camera = Camera::create(std::move(data), data->video_->deviceName(), streams);
  registerCamera(std::move(camera));

  return true;
}
```

#### 주요 구조

- `CameraManager`도 `Extensible` 패턴을 사용하여 `CameraManager::Private`을 가짐
  - `CameraManager` 안에는 아래의 것들이 들어있음
    - `DeviceEnumerator`
    - `IPAManager`
    - `ProcessManager`

> Camera는 인터페이스이고, 실제 구현은 PipelineHandler에서 이루어지는 구조임
