# Chapter 6. 초기화 관리자 추가 및 로그 파일을 통한 디버깅

- hello 이미지를 systemd에 추가하자 + bitbake가 생성하는 로그파일을 통해 실패의 원인을 찾아보자

> systemd : 시스템 부팅후 가장 먼저 실행되어 다른 프로세스를 실행하는 daemon

## 1. systemd 초기화 관리자 추가

- 앞에서 만든 hello 예제는 hello라는 실행 파일을 만들어 루트 파일 시스템에 추가하고 타깃 시스템을 부팅시켜 hello를 직접 실행함
- systemd는 컴포넌트(?)들을 unit으로 다룸
- systemd는 서비스들을 병렬로 실행함
- systemd는 시스템이 시작될 떄 최소한의 서비스만 실행되고, 필요한 것은 필요시 마다 실행함
- systemd는 매우 복잡하고 많은 기능을 다루고 있어, 여기에서는 Yocto 예제를 이해할 수 있는 수준에서의 내용만 다룸

![systemd architecture](https://opensource.com/sites/default/files/uploads/systemd-architecture.png)

- systemd의 'd'는 데몬을 의미하며, 데몬은 백그라운드에서 실행되는 프로세스를 의미함
  - 다른 말로 메모리에 머물고 있으면서, 특정 요청이 오면 바로 대응할 수 있도록 대기 중인 프로세스
- 데몬은 부모 프로세스를 갖지 않으며, 대부분 프로세스 트리에서 init 바로 아래 위다함
- 즉 시스템의 첫 프로세스인 systemd의 하위 프로세스가 됨
- 이런 데몬들을 실행하고 관리해주는 데몬이 바로 systemd 임
- 기존의 init 데몬을 systemd가 대체했기 때문에 이전 init과 동일하게 PID '1'을 가짐
- 또한 부모 프로세스가 없기 때문에 PPID 또한 '1'임
- systemd는 리소스를 unit이라 불리는 단위로 관리하며 아래와 같은 타입의 유닛이 존재함
  - .service, .socket, .device, .mount, .automount, .swap, .target, .path, .timer, ...
- 여기에서는 .service 유닛에 대해서만서다룸
- .service 유닛 파일은 서비스나 애플리케이션을 서버상에서서어떻게 관리할지에 대한 정보를 가짐
- 일반적으로 .service 유닛 파일은 `/lib/systemd/system` 아래에 복사해 동작하도록 함

#### 유닛 파일 내에서의 섹션의 종류

유닛 파일 내에서 Unit, Service, Install과 같은 지시자를 섹션이라고 함

- Unit 섹션 : 최초로 동작하는 섹션, 다른 unit 간의 관계를 설정
  - Description : unit에 대한 설명
  - Documentation : 서비스에 대한 문서가 있는 URI, `systemlctl status hello` 명령어로 볼 수 있음
  - After : 현재 유닛보다 먼저 실행돼야 하는 유닛 리스트
  - Before : 현재 유닛보다 나중에 실행돼야 하는 유닛 리스트
  - Requires : 현재 유닛이 의존하고 있는 모든 유닛 리스트, 이 목록의 unit들이 실행되고 있어야함
  - Wants : Requires와 동일하나, 이 목록의 unit들의 실행 여부에 상관없음
- Service 섹션 : 이 섹션은 .service 유닛 파일에만 적용 가능함
  - Type : 서비스가 어떤 형태로 동작하는지 설정
    - simple : default 값으로, ExecStart가 기술되어야함.
    - forking : 서비스가 자식 프로세스를 생성할 때 사용
    - oneshot : systemd가 서비스가 끝날때까지 기다림, 프로세스가 오래 실행되지 않을 때 사용해야함
    - dbus : DBUS에 지정된 BusName이 준비될 때 까지 대기
    - notify : 서비스가 startup이 끝날 때 systemd에 signal을 보냄
    - idle : 모든 서비스가 실행된 이후 실행
    - ExecStart : 서비스를 시작하기 위한 전체 경로, 서비스 시작 전에 이 명령어를 실행해야 함
    - ExecStartPre : 서비스가 시작되기 전 실행할 명령을 설정
    - ExecStartPost : 서비스가 시작되고 나서 실행할 명령을 설정
    - Restart : 서비스가 종료됐을 때 자동으로 재시작 되어야 함
  - TimeoutStartSec : 서비스 시작시 대기하는 시간
- Install 섹션 : 유닛이 활성화되거나 비활성화될 때 유닛의 행동을 정의
  - WantedBy : `systemctl enable` 명령어로 유닛을 등록할 때, 등록에 필요한 유닛을 지정
  - Also : `systemctl enable` & `systemctl disable` 명령어로 유닛을 등록/해제할 때 enable, disable할 유닛을 지정
  - Alias : 유닛의 별칭

#### systemd를 통한 hello 애플리케이션 실행

poky_src/poky/meta-hello/reciepes-hello/hello.bb

```
...

SRC_URI_append = " file://COPYING"
SRC_URI_append = " file://hello.service"
inherit system

...

SYSTEMD_SERVICE_${PN} = "hello.service"
SYSTEMD_AUTO_ENABLE = "enable"

...

do_install() {
    ....
    install -d ${D}${systemd_unitdir}/system
    install -m 0644 hello.service ${D}${systemd_unitdir}/system
}

...

FILES_${PN} += "${systemd_unitdir}/system/hello.service"

```

- `systemd_unitdir` == /lib/systemd
- `systemd_system_unitdir` == /lib/systemd/system
- `systemd_user_unitdir` == /usr/lib/systemd/user

poky_src/poky/meta-hello/recipes-hello/source/hello.service

```
[Unit]
Description=Hello World startup script

[Service]
ExecStart=/usr/bin/hello

[Install]
WantedBy=multi-user.target
```

poky_src/build/conf/local.conf

```
DISTRO_FEATURES_append = " systemd"
DISTRO_FEATURES_remove = " systemvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd-compat-units"
DISTRO-FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_initscript = "systemd-compat-units"
```

- Yocto는 기본 초기화 관리자로 System V Init을 가지고 있어, 이걸 systemd로 바꾸어 주어야 함

```
bitbake hello -C fetch
bitbake core-image-minimal -C rootfs
run core-image-minimal nographic
```

- 결과를 보면, 직접 실행하지 않고도 자동으로 실행된 것을 볼 수 있음

## 2. 로그를 이용한 디버깅

- bitbake는 빌드를 진행하며 메타데이터에 추가한 디버깅 정보나 실행된 모든 명령의 산출물 그리고 오류 메시지를 기록함
- 또한 태스크 각각에 대해, 전체 빌드 절차에 대해서도 로그 파일로 기록함
- 로그 파일이 어디에 생성되고, 디버깅 시 로그 파일들을 어떻게 이용해야 하는지 알아보자

#### 쿠커의 로그 파일 생성

![bitbake application 구조](https://aosabook.org/static/yocto/aosa3.jpg)

- bitbake는 클라이언트-서버 애플리케이션임
  - 백엔드 : 서버 + 쿠커, 클라이언트 : 사용자 인터페이스용 프론트엔드 프로세스)
  - 백엔드와 프론트엔드는 IPC 를 이용해 정보를 교환함
- 클라이언트 : 빌드 출력, 빌트 상태 및 빌드 진행 상황 로깅, 비드 작업에서 이벤트를 수신하기 위한 메커니즘 제공
  - 기본적으로 사용되는 사용자 인터페이스는 bitbake의 콘솔 명령줄 인터페이스인 knotty임
- 쿠커 프로세스 : bitbake의 핵심부분, Poky 빌드 중에서 발생하는 일의 대부분이 처리되는 곳
  - 메타데이터 구문 분석을 관리하고 종속성 및 태스크 체인 생성을 시작하며 빌드를 관리
  - bitbake 실행 파일에서 쿠커를 실행하면 쿠커는 bitbake 데이터 저장소를 초기화한 후 Poky의 모든 메타데이터 파일을 구문 분석 시작함
  - 그런 다음 runqueue 객체를 만들고 빌드를 시작함
  - poky_src/build/tmp/log/cooker/qemux86-64 안에 쿠커에 의해 만들어진 로그 파일들이 담김

#### console-latest.log

- BB_VERSION : bitbake 버전
- BUILD_SYS : 호스트 빌드 시스템
- NATIVELSBSTRING : 호스트의 배포자를 식별하기 위한 문자열
  - Ubuntu 18.04가 아닌 universal이라는 값이 표기되는 이유는 호스트 배포판에서 제공하는 glibc 버전이 아닌 Yocto 프로젝트에서 배포하는 glibc 버전으로 컴파일 되기 때문임
- MACHINE : bitbake가 빌드하기 위한 타깃 머신
- DISTRO : 타깃 시스템 배포의 이름

#### 태스크 로그 파일, 태스크 스크립트 파일 및 디버깅 팁

- bitbake는 빌드 과정에서 태스크를 수행하며 그에 따른 로그 파일, 스크립트 파일을 생성함
- 태스크의 로그 파일, 스크립트 파일의 저장 위치는 `T` 변수에 할당되어 있음
- `T`는 기본 값으로 `${WORKDIR}/temp`를 가짐

```bash
bitbake-getvar -r hello T
```

- 태스크 로그 파일은 `log.do_<taskname>.<pid>`와 같은 형식을 가짐
  - pid는 bitbake가 실행한 태스크 프로세스 ID임
  - 동일한 태스크를 여러번 수행하면서, 그때만다 태스크 프로세스 ID가 바뀌어 서로 다른 로그파일이 생성됨
- 로그 파일중 `.<pid>` 확장자 없이 `log.do_<taskname>`의 형식의 파일은 심볼릭 링크 파일
  - 이 파일은 현재 수행하거나 바로 직전에 수행한 출력 로그를 가리킴
- bitbkae는 태스크 실행 중에 수행하는 명령어를 스크립트 파일로 만듦
- 이러한 스크립트 파일은 `run.do_<taskname>.<pid>`와 같은 형식을 가짐
  - 스크립트 파일은 태스크 로그 파일과 동일한 경로에 저장
  - 현재 실행 중이거나 직전에 수행된 테스크 스크립트 파일은 `run.do_<taskname>`과 같은 이름을 가짐
- `log.task_order` 파일 : 최근 실행된 프로세스 ID + 실행된 태스크의 순차적 목록
  - 파일을 보면 태스크를 수행하면서 발생한 로그가 남아있고, 빌드 에러가 발생한 경우, 에러가 발생한 태스크의 로그 파일을 열어 세부 사항을 확인함

> `run.do_<taskname>` 파일은 태크 실행 스크립트 파일이며, 이 파일은 셀 스크립트로 구성되어 있어 바로 실행이 가능함!
> 실행 스크립트 파일에 echo 등으로 로깅 데이터를 추가해 디버깅을 수행하며 도움이 됨

## 3. 요약

- 현재 대부분의 리눅스 시스템들은 System V Init 대신 systemd를 사용함
  - 병렬 실행으로 부팅 속도의 향상을 가져오는 등 향상된 다양한 기능을 제공
- 빌드가 실패했을 때, 대처할 수 있는 가장 좋은 방법은 실행 로그를 얻어 분석하는 것
- bitbake는 빌드를 진행하면서 로그 파일을 생성하기 때문에 이 로그 파일을 열어 확인해보면 문제 발생 원인을 추적할 수 있음
- 태스크 실행 스크립트 파일도 생성하기 때문에 문제가 발생한 태스크에 로그 메시지를 삽입해 재현 테스트가 가능함
