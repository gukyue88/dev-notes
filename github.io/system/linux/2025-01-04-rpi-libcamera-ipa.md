---
layout: single
title: "[linux][라즈베리파이 libcamera Doc] ipa"
categories: system/linux
published: false
---

repository : [raspberrypi/libcamera](https://github.com/raspberrypi/libcamera)
doc : libcamera/doc/ipa.rst

# IPA 개발자 가이드

- IPA 모듈은 이미지 처리 알고리즘(Image Processing Algorithm) 모듈임
- 이 모듈은 파이프라인 핸들러가 이미지 처리에 사용할 수 있는 기능을 제공함
- 이 가이드는 IPA 인터페이스의 정의와 파이프라인 핸들러와 IPA 간의 연결 방법을 다룸

## IPA 인터페이스와 프로토콜

- IPA 인터페이스는 파이프라인 핸들러와 IPA 간의 인터페이스를 정의함
- 구체적으로, 파이프라인 핸들러가 호출할 수 있도록 IPA가 노출하는 함수와,
- IPA로부터 비동기적으로 데이터를 수신하기 위해 파이프라인 핸들러가 연결할 수 있는 신호를 정의함
- 또한, 파이프라인 핸들러와 IPA가 서로 전달할 수 있는 사용자 정의 데이터 구조를 포함함
- 동일한 IPA 인터페이스를 여러 하드웨어 플랫폼의 파이프라인 핸들러와 함께 사용하는 것이 가능함
- 일반적으로 이러한 경우, 이 플랫폼들은 공통된 하드웨어 ISP(Image Signal Processor) 파이프라인을 가짐
- 예를 들어, rkisp1 파이프라인 핸들러는 RK3399와 i.MX8MP가 동일한 ISP를 통합하고 있기 때문에 둘 다 지원함
- 그러나 i.MX8MP는 더 복잡한 카메라 파이프라인을 가지고 있어, 미래에는 전용 파이프라인 핸들러가 필요할 수 있음
- ISP가 RK3399와 동일하므로, 동일한 IPA 인터페이스를 두 파이프라인 핸들러 모두에 사용할 수 있음
- 빌드 파일은 :ref:`compiling-section`에 자세히 설명된 대로 파이프라인 핸들러에서 IPA 인터페이스 이름으로의 매핑을 제공함
- IPA 프로토콜은 주어진 IPA 호출에 대한 IPA의 예상 응답(들)에 대한 파이프라인 핸들러와 IPA 간의 약속을 의미함
- 이 프로토콜은 코드 내에서 명시적으로 선언될 필요는 없지만, 하나의 파이프라인 핸들러에 대해 여러 IPA 구현이 있을 수 있으므로 문서화되어야 함
- libcamera의 설계의 일환으로, IPA는 별도의 프로세스로 격리되거나, 동일한 프로세스 내의 다른 스레드에서 libcamera와 함께 실행될 수 있음
- IPA가 격리되었는지 여부에 따라 파이프라인 핸들러와 IPA는 작동을 변경할 필요가 없지만, 격리 가능성을 염두에 두어야 함
- 따라서 그들 사이에 전달되는 모든 데이터는 직렬화 가능해야 하며, `mojo Interface Definition Language`\_ (IDL)에서 별도로 정의되어야 함
- 코드 생성기는 이 정의에 해당하는 헤더와 직렬화기를 생성함
- 모든 인터페이스는 mojom 파일에 정의되며 다음을 포함함
  - 파이프라인 핸들러가 IPA에서 호출할 수 있는 함수
  - IPA가 내보낼 수 있는 파이프라인 핸들러의 신호
  - 파이프라인 핸들러와 IPA 사이에 전달될 사용자 정의 구조체
- 특정 파이프라인 핸들러의 모든 IPA 모듈은 동일한 IPA 인터페이스를 사용함
- 따라서 IPA 인터페이스 정의는 파이프라인 핸들러 작성자가 파이프라인 핸들러와 IPA 간의 상호 작용을 어떻게 설계하는지에 따라 작성
- 함수, 신호, 그리고 모든 사용자 정의 구조체를 포함한 전체 IPA 인터페이스는 `include/libcamera/ipa/` 아래의 `{interface_name}.mojom`이라는 파일에 정의되어야 함

## Namespacing

- 다른 IPA 인터페이스와 libcamera에 의해 정의된 데이터 유형 간의 이름 충돌을 피하기 위해, 각 IPA 인터페이스는 자체 네임스페이스에 정의되어야 함
- 네임스페이스는 `모조(mojo)`의 `모듈(module)` 지시어로 지정됨
- 이는 모조 데이터 정의 파일에서 주석이 아닌 첫 번째 줄이어야 함
- 예를 들어, 라즈베리 파이 IPA 인터페이스는 다음을 사용함

```
module ipa.rpi;
```

## Data containers

- Pipeline 핸들러와 IPA 간에 전달되는 데이터는 직렬화를 지원해야 하므로, 모든 사용자 정의 데이터 컨테이너는 모조 IDL(Interface Definition Language)로 정의되어야 함
- 다음은 인터페이스 정의에서 지원되며, 함수 매개변수 유형 또는 구조체 필드 유형으로 사용될 수 있는 **libcamera 객체** 목록임
  - libcamera.ControlInfoMap
  - libcamera.ControlList
  - libcamera.FileDescriptor
  - libcamera.IPABuffer
  - libcamera.IPACameraSensorInfo
  - libcamera.IPASettings
  - libcamera.IPAStream
  - libcamera.Point
  - libcamera.Rectangle
  - libcamera.Size
  - libcamera.SizeRange
- 이러한 객체를 사용하려면, 모조 데이터 정의 파일에 core.mojom이 포함되어야 함

```
import "include/libcamera/ipa/core.mojom";
```

- 다른 사용자 정의 구조체도 정의하고 사용할 수 있음
- 사용 전에 반드시 정의해야 하는 요구사항은 없음
- **열거형(enums)**과 **구조체(structs)**가 모두 지원됨
- 다음은 플래그(flags)로 사용하기 위한 열거형 정의의 예시임

```
struct ConfigInput {
        uint32 op;
        uint32 transform;
        libcamera.FileDescriptor lsTableHandle;
        int32 lsTableHandleStatic = -1;
        map<uint32, libcamera.IPAStream> streamConfig;
        array<libcamera.IPABuffer> buffers;
};
```

- 이 예시에는 몇 가지 특별한 점이 있음
- 우선, **FileDescriptor** 데이터 타입을 사용함
- 이 타입은 IPC 경계(IPA가 격리된 프로세스에 있을 때)를 넘어 파일 디스크립터가 올바르게 변환되도록 보장하기 위해 사용됨
- 따라서 만약 파일 디스크립터가 변환되지 않고 전송되어야 한다면(예를 들어, IPA가 파이프라인 핸들러가 소유한 파일 디스크립터 중 어느 것에 대해 조치를 취해야 하는지 알려주기 위함), **일반 int32 타입**에 있어야 함
- 이 예시는 또한 구조체 필드에 기본값(default values)이 할당될 수 있다는 것을 보여줌
- `lsTableHandleStatic`에 할당된 값이 그 예임
- 이는 구조체가 기본 생성자로 생성될 때 해당 필드가 가질 값임
- **배열(arrays)**과 **맵(maps)**도 지원됨
- 이들은 각각 C++의 `vector`와 `map`으로 변환됨
- 배열과 맵의 멤버는 포함(embedded)되며, `const`가 될 수 없음
- 참고로, 모조(mojo)가 지원하는 널(nullable) 필드, 고정 길이 배열(static-length arrays), 핸들(handles), 그리고 유니온(unions)은 **저희 코드 생성기에서는 지원되지 않음.**

## main IPA Interface

- IPA 인터페이스는 두 부분으로 나뉨
- **메인 IPA 인터페이스**는 파이프라인 핸들러가 IPA에서 호출할 수 있는 함수를 기술하고,
- **이벤트 IPA 인터페이스**는 IPA가 방출할 수 있는, 파이프라인 핸들러가 수신하는 신호를 기술함
- 둘 다 정의되어야 함
- 이 섹션은 메인 IPA 인터페이스에 초점을 맞춥니다.
- 메인 인터페이스는 `IPA{interface_name}Interface`로 명명되어야 함
- 파이프라인 핸들러가 IPA에서 호출할 수 있는 함수는 **동기식(synchronous)** 또는 **비동기식(asynchronous)**일 수 있음
- 동기식 함수는 IPA가 함수에서 반환할 때까지 반환하지 않지만, 비동기식 함수는 IPA가 반환하기를 기다리지 않고 즉시 반환함
- 최소한 다음 세 가지 함수는 반드시 존재해야 합니다 (그리고 구현되어야 합니다):
  - `init();`
  - `start();`
  - `stop();`
- 이 세 함수는 모두 동기식입니다. `start()`와 `init()`의 매개변수는 사용자 정의가 가능함
- `init()` : IPA 인터페이스를 초기화. 이 함수는 `IPAInterface`의 다른 어떤 함수보다 먼저 호출되어야 함
- `stop()`는 카메라가 중지되었음을 IPA 모듈에 알림. IPA 모듈은 `start()`에서 준비된 리소스를 해제해야 함
- `configure()` 함수는 권장됨. 예를 들어, IPA가 사용할 모든 `ControlInfoMap` 인스턴스는 `configure` 시점에 파이프라인 핸들러로부터 IPA로 전송되어야 함
- 모든 입력 매개변수는 산술 유형을 제외하고는 `const` 참조가 되며, 산술 유형은 값으로 전달됨
- 출력 매개변수는 포인터가 되지만, 첫 번째 출력 매개변수가 `int32`이거나 기본 출력 매개변수가 하나만 있는 경우에는 일반적인 반환값이 됨
- `const`는 배열과 맵 내에서는 허용되지 않음. 모조(mojo) 배열은 C++의 `std::vector<>`가 됨
- 기본적으로, 메인 인터페이스에 정의된 모든 함수는 동기식임
- 이는 IPC(즉, 격리된 IPA)의 경우, 반환값 또는 출력 매개변수가 준비될 때까지 함수 호출이 반환되지 않음을 의미함
- 비동기식 함수를 지정하기 위해 `[async]` 속성을 사용할 수 있음
- 비동기식 함수는 IPC의 경우 호출이 즉시 반환되어야 하므로, 반환값이나 출력 매개변수를 가질 수 없음
- IPA가 격리된 상태로 실행되지 않을 수도 있음
- 이 경우, `start()`가 호출될 때까지 IPA 스레드는 존재하지 않음
- 이는 격리가 없는 경우 `start()` 전에 비동기 호출을 할 수 없음을 의미함
- IPA 인터페이스는 격리 여부에 관계없이 동일해야 하므로, 격리된 경우에도 동일한 제한이 적용되며, `start()` 전에 호출될 모든 함수는 동기식이어야 함
- 또한, `start()` 이후 `stop()` 이전에 이루어지는 모든 호출은 비동기식이어야 함
- 이는 파이프라인 핸들러의 실시간 성능 저하를 피하기 위함임
- 파이프라인 핸들러가 IPA로부터 어떤 데이터를 원한다면, IPA는 이 데이터를 이벤트를 통해 비동기적으로 반환해야 힘 ("이벤트 IPA 인터페이스" 참조).
- 다음은 메인 인터페이스 정의의 예시임

```
interface IPARPiInterface {
        init(libcamera.IPASettings settings, string sensorName)
                => (int32 ret, bool metadataSupport);
        start() => (int32 ret);
        stop();

        configure(libcamera.IPACameraSensorInfo sensorInfo,
                    map<uint32, libcamera.IPAStream> streamConfig,
                    map<uint32, libcamera.ControlInfoMap> entityControls,
                    ConfigInput ipaConfig)
                => (int32 ret, ConfigOutput results);

        mapBuffers(array<IPABuffer> buffers);
        unmapBuffers(array<uint32> ids);

        [async] signalStatReady(uint32 bufferId);
        [async] signalQueueRequest(libcamera.ControlList controls);
        [async] signalIspPrepare(ISPConfig data);
};
```

- 요구되는 함수는 처음 세 가지임.
- 함수는 `stop()`, `mapBuffers()`, `unmapBuffers()`와 같이 반환값이 없을 수 있음
- 앞서 설명했듯이, 비동기 함수인 경우 반환값이 **반드시 없어야** 함.

## Event IPA Interface

- 이벤트 IPA 인터페이스는 IPA가 내보낼 수 있는, 파이프라인 핸들러가 수신하는 신호를 기술함
- 이 인터페이스는 반드시 정의되어야 함.
- 만약 이벤트 함수가 없다면 비어 있을 수 있음
- 이러한 신호 방출은 요청 데이터 준비와 같은 특정 이벤트를 파이프라인 핸들러에 알리기 위한 것이며, IPA가 카메라 파이프라인을 직접 구동하는 데 **사용되어서는 안 됨**.
- 이벤트 인터페이스는 `IPA{interface_name}EventInterface`로 명명되어야 함
- 이벤트 인터페이스에 정의된 함수는 암묵적으로 비동기식임
- 따라서 어떠한 값도 반환할 수 없음. `[async]` 태그를 명시적으로 지정할 필요는 없음
- 이벤트 인터페이스에 정의된 함수는 IPA 인터페이스에서 신호(signals)가 됨
- IPA는 신호를 내보낼 수 있으며, 파이프라인 핸들러는 이에 슬롯(slots)을 연결할 수 있음
- 다음은 이벤트 인터페이스 정의의 예시임

```
pipeline_ipa_mojom_mapping = [
    'rpi/vc4': 'raspberrypi.mojom',
]
```

- 이는 모조(mojo) 데이터 정의 파일이 컴파일되도록 합니다. 구체적으로, 다음의 다섯 가지 파일을 생성함
  - 사용자 정의 데이터 구조와 전체 IPA 인터페이스를 설명하는 헤더 파일 (경로: `{$build_dir}/include/libcamera/ipa/{interface}_ipa_interface.h`)
  - 사용자 정의 데이터 구조의 직렬화/역직렬화를 구현하는 직렬화기 파일 (경로: `{$build_dir}/include/libcamera/ipa/{interface}_ipa_serializer.h`)
  - 특수화된 IPA 프록시를 설명하는 프록시 헤더 파일 (경로: `{$build_dir}/include/libcamera/ipa/{interface}_ipa_proxy.h`)
  - IPA 프록시를 구현하는 프록시 소스 파일 (경로: `{$build_dir}/src/libcamera/proxy/{interface}_ipa_proxy.cpp`)
  - IPA 프록시의 다른 한쪽 끝을 구현하는 프록시 워커 소스 파일 (경로: `{$build_dir}/src/libcamera/proxy/worker/{interface}_ipa_proxy_worker.cpp`)
- IPA 프록시는 파이프라인 핸들러와 IPA 사이의 계층 역할을 하며, 스레딩과 격리(isolation)를 투명하게 처리함
- 파이프라인 핸들러와 IPA는 인터페이스 헤더와 프록시 헤더만 필요로 함
- 직렬화기는 프록시 내부에서만 사용됨

## custom data structures 사용하기

mojo 데이터 정의 파일에 정의된 사용자 정의 데이터 구조를 사용하려면, 다음 헤더 파일을 포함해야 함

```cpp
#include <libcamera/ipa/{interface_name}_ipa_interface.h>
```

- 모조(mojo) 구조체의 POD 타입(Plain Old Data types)은 간단히 C++ 대응 타입으로 변환됨
- 예를 들어, 모조의 `uint32`는 C++에서 `uint32_t`가 됨
- 모조의 **맵(map)**은 C++의 `std::map`이 되고,
- 모조의 **배열(array)**은 C++의 `std::vector`가 됨
- 맵과 벡터의 모든 멤버는 내장(embedded)되며, 포인터가 아님
- 멤버는 `const`가 될 수 없음
- 구조체의 모든 필드 이름은 데이터 정의 파일에 정의된 것과 정확히 같은 방식으로 C++에서 사용될 수 있음
- 예를 들어, 모조 파일에 다음과 같이 정의된 구조체가 있다고 가정해 보자

```
struct SensorConfig {
    uint32 gainDelay = 1;
    uint32 exposureDelay;
    uint32 sensorMetadata;
};
```

이 모조 파일 내용은 아래의 cpp로 바뀜

```cpp
struct SensorConfig {
    uint32_t gainDelay;
    uint32_t exposureDelay;
    uint32_t sensorMetadata;
};
```

- 생성된 구조체에는 두 가지 생성자가 있음
- 모든 필드를 기본값으로 채우는 생성자와 모든 필드에 대한 값을 받는 두 번째 생성자임
- 기본값 생성자는 지정된 기본값이 있는 경우 해당 값으로 필드를 채움
- 위 예시에서 `gainDelay_`는 1로 초기화됨
- 기본값이 지정되지 않은 경우, 0으로 채워짐(FileDescriptor 타입의 경우 -1).
- 이 생성된 구조체의 모든 필드와 생성자/소멸자는 **public**임

## IPA interface 사용하기 (pipeline handler)

- IPA를 파이프라인 핸들러에서 사용하려면 다음 헤더들이 필요함 (예시: 라즈베리 파이).

```cpp
#include <libcamera/ipa/raspberrypi_ipa_interface.h>
#include <libcamera/ipa/raspberrypi_ipa_proxy.h>
```

- 첫 번째 헤더 파일은 사용자 정의 데이터 구조 정의와 전체 IPA 인터페이스(메인 및 이벤트 IPA 인터페이스 모두 포함) 정의를 담고 있음
- 이 헤더 파일의 이름은 여기서는 `raspberrypi.mojom`이었던 모조(mojom) 파일 이름에서 유래함
- 두 번째 헤더 파일은 특수화된 IPA 프록시 정의를 포함함
- 이 프록시는 완전한 IPA 인터페이스를 노출함
- 이 섹션에서는 이를 어떻게 사용하는지 알아보겠음
- 파이프라인 핸들러에서, 우리는 먼저 특수화된 IPA 프록시를 생성해야 함
- 파이프라인 핸들러의 관점에서, 이 객체가 바로 IPA임
- 이를 위해, 우리는 IPAManager를 호출함

```cpp
std::unique_ptr<ipa::rpi::IPAProxyRPi> ipa_ =
        IPAManager::createIPA<ipa::rpi::IPAProxyRPi>(pipe_, 1, 1);
```

- mojo 데이터 정의 파일의 "네임스페이스 지정" 섹션에서 정의한 네임스페이스로부터 `ipa::rpi` 네임스페이스가 생성됨
- 프록시의 이름인 `IPAProxyRPi`는 "메인 IPA 인터페이스" 섹션에서 메인 IPA 인터페이스에 부여된 이름인 `IPARPiInterface`에서 유래함
- `IPAManager::createIPA`의 반환값은 **널 포인터(nullptr)가 아닌지 확인**하기 위해 오류를 검사해야 함
- 그 후, IPA를 초기화하기 전에, **이벤트 IPA 인터페이스**에 정의된 모든 IPA 신호에 슬롯을 연결해야 함

```cpp
ipa_->statsMetadataComplete.connect(this, &RPiCameraData::statsMetadataComplete);
ipa_->runIsp.connect(this, &RPiCameraData::runIsp);
ipa_->embeddedComplete.connect(this, &RPiCameraData::embeddedComplete);
ipa_->setIsp.connect(this, &RPiCameraData::setIsp);
ipa_->setStaggered.connect(this, &RPiCameraData::setStaggered);
```

- 슬롯 함수는 이벤트 IPA 인터페이스의 함수 정의를 기반으로 함수 서명을 가짐
- 모든 POD(Plain Old Data) 타입은 그대로(C++ 버전, 예: `uint32` -> `uint32_t`) 사용되며, 모든 구조체는 `const` 참조가 됨
- 예를 들어, 이벤트 IPA 인터페이스의 다음 항목의 경우:

```
statsMetadataComplete(uint32 bufferId, ControlList controls);
```

- 신호에 연결될 함수는 다음과 같은 함수 시그니처를 가져야 함

```cpp
void statsMetadataComplete(uint32_t bufferId, const ControlList &controls);
```

- 슬롯을 신호에 연결한 후, IPA는 (이전의 메인 인터페이스 정의 예시를 사용하여) 초기화되어야 함

```cpp
IPASettings settings{};
bool metadataSupport;
int ret = ipa_->init(settings, "sensor name", &metadataSupport);
```

- 여기부터 메인 IPA 인터페이스에 정의된 어떤 IPA 함수라도 일반 멤버 함수처럼 호출할 수 있음
- 예를 들어 (이전 메인 인터페이스 정의 예시에 기반하여):

```cpp
ipa_->start();
int ret = ipa_->configure(sensorInfo_, streamConfig, entityControls, ipaConfig, &result);
ipa_->signalStatReady(RPi::BufferMask::STATS | static_cast<unsigned int>(index));
```

- 비동기 함수로 지정된 모든 함수는 `start()` 전에 호출되면 **안 된다는** 것을 기억해야함
- `init()`와 `configure()` 함수를 보면, 첫 번째 출력 매개변수는 `int32`이므로 직접 반환되는 반면, 다른 출력 매개변수는 포인터 기반의 출력 매개변수라는 점을 알 수 있음

## IPA interface 사용하기 (IPA Module)

- Raspberry Pi를 예로 들어 IPA 모듈을 구현하는 데 필요한 헤더 파일은 다음과 같음

```cpp
#include <libcamera/ipa/raspberrypi_ipa_interface.h>
```

- 이 헤더는 사용자 정의 데이터 구조 정의와 전체 IPA 인터페이스(메인 및 이벤트 IPA 인터페이스 모두 포함) 정의를 담고 있음
- 헤더 파일의 이름은 여기서는 `raspberrypi.mojom`이었던 모조(mojom) 파일 이름에서 유래함
- IPA 모듈은 해당 헤더에 정의된 IPA 인터페이스 클래스를 구현해야 함
- 이 예시의 경우, `ipa::rpi::IPARpiInterface`임
- `ipa::rpi` 네임스페이스는 모조 데이터 정의 파일의 "네임스페이스 지정" 섹션에서 정의한 네임스페이스에서 비롯됨
- 인터페이스 이름은 메인 IPA 인터페이스에 부여된 이름과 동일함
- 함수 시그니처 규칙은 파이프라인 핸들러 측의 슬롯(slots)과 동일함
- POD(Plain Old Data)는 값으로 전달되고, 구조체는 `const` 참조로 전달됨
- 메인 IPA 인터페이스의 경우, 출력 값도 허용되므로(동기 호출에 한함) 출력 매개변수가 있을 수 있음
- 첫 번째 출력 매개변수가 POD이면 값으로 반환되고, 그렇지 않으면 출력 매개변수 포인터로 반환됨
- 두 번째 및 그 외 다른 출력 매개변수도 출력 매개변수 포인터로 반환됨
- 예를 들어, 메인 IPA 인터페이스 정의의 다음 함수 사양의 경우:

```
configure(libcamera.IPACameraSensorInfo sensorInfo,
            uint32 exampleNumber,
            map<uint32, libcamera.IPAStream> streamConfig,
            map<uint32, libcamera.ControlInfoMap> entityControls,
            ConfigInput ipaConfig)
=> (int32 ret, ConfigOutput results);
```

- 다음과 같은 함수 시그니처를 가진 함수를 구현해야 함

```cpp
int configure(const IPACameraSensorInfo &sensorInfo,
                uint32_t exampleNumber,
                const std::map<unsigned int, IPAStream> &streamConfig,
                const std::map<unsigned int, ControlInfoMap> &entityControls,
                const ipa::rpi::ConfigInput &data,
                ipa::rpi::ConfigOutput *response);
```

- 첫 번째 출력 매개변수가 int32이기 때문에 반환 값은 int가 됨
- 나머지 출력 매개변수(이 경우에는 response뿐)는 출력 매개변수 포인터가 됨
- 비(非) POD(Plain Old Data) 입력 매개변수는 const 참조가 되고, POD 입력 매개변수는 값으로 전달됨
- start() 이후 stop() 이전에 언제든지(보통은 IPA 호출에 대한 응답으로만) IPA는 신호를 내보내서 파이프라인 핸들러에게 데이터를 전송할 수 있음
- 이러한 신호는 C++ IPA 인터페이스 클래스(생성되고 포함된 헤더 파일에 있음)에 정의되어 있음
- 예를 들어, 이벤트 IPA 인터페이스에 정의된 다음 함수에 대해:

```
statsMetadataComplete(uint32 bufferId, libcamera.ControlList controls);
```

- 신호를 내보내는 방법은 다음과 같음

```cpp
statsMetadataComplete.emit(bufferId & RPi::BufferMask::ID, libcameraMetadata_);
```
