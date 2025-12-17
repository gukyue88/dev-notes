# Chapter 9. 패키지 그룹 및 빌드 환경 구축

- 패키지 그룹을 학습하고, 기존 예제의 이미지 레시피인 core-image-minmal.bb에 패키지 그룹으로 만든 예제 패키지를 넣어보자
- 10장 부터는 레이어를 계층별로 쌓아 올리면서 학습을 진행할 예정
  - 커스텀 레이어를 생성하면서 Yocto에서 제공하는 기본 빌드 스크립트를 가져가기엔 제약사항이 있음
  - 커스텀 빌드 스크립트를 만들어 앞으로 진행할 예제들을 쉽게 빌드할 수 있도록 환경을 구축할 예정

## 1. IMAGE_INSTALL, IMAGE_FEATURES 변수

> `IMAGE_INSTALL` : 루트 파일 시스템에 설치할 패키지를 나열한 변수

- 우리는 루트 파일 시스템 이비지를 생성하는 레시피 파일로 core-image-minimal.bb 파일을 사용해 왔음
- 5장에서 루트 파일 시스템 이미지에 우리가 만든 패키지를 넣으려고 `IMAGE_INSTALL += "hello nano"`를 함
- 참고로 이제부터 언급되는 "이미지" == "루트 파일 시스템" 임

> 이미지 == `IMAGE_INSTALL` 변수에 할당된 패키지들 + `IMAGE_FEATURES` 변수에 할당된 features

- `IMAGE_FEATURES` : 루트 파일 시스템을 생성하는 대상 이미지 레시피(core-image-minimal.bb)에서 상속한 기반 이미지 클래스에 따라 할당할 수 있는 값들이 결정됨(?)
- 즉, 이미지 클래스는 미리 정의된 기능 목록을 갖고 있으므로 `IMAGE_FEATURES` 변수에 기능 목록 중에서 필요한 기능을 할당함

> `IMAGE_FEATURES` : 이미지에 추가되는 기능들을 레시피에서 추가
> local.conf 와 같은 환경 설정 파일에 추가할 때는 `EXTRA_IMAGE_FEATURES` 변수를 사용. (`EXTRA_IMAGE_FEATURES`는 최종적으로 `IMAGE_FEATURES`변수에 추가됨)

#### 이미지 클래스에서 미리 정의된 기능 목록 중 몇가지

- image.bbclass
  - debug-tweaks : 비밀번호 없이 SSH 로그인 가능 (개발시 주로 사용)
  - package-management : 루트 파일 시스템을 만들 경우, `PACKAGE_CLASSES` 변수에 설정된 패키지 관리 유형에 따른 패키지 관리 시스템 설치
  - read-only-rootfs : 읽기 전용으로 루트 파일 시스템을 생성
  - splash : 부팅 시 텍스트 기반 메시지가 아닌 스플래시 스크린이 동작하도록 함

> 정리해보면 `IMAGE_FEATURES` 변수는 사용하려는 이미지의 특정한 features를 활성화하도록 만드는데 사용함

#### splash 기능 추가해보기

poky_src/build/conf/local.conf

```conf
EXTRA_IMAGE_FEATURES +=  "splash"
```

```bash
bitbake core-image-minmal -C rootfs
runqemu
```

위와 같이 진행하면 splash 화면이 출력되는 것을 볼 수 있음. (단, nographic 옵션을 뺴야함)

#### IMAGE_INSTALL과 IMAGE_FEATURES 변수의 차이

- 루트 파일 시스템은 `IMAGE_FEATURES`와 `IMAGE_INSTALL` 두 변수로 구성됨
- `IMAGE_FEATURES += "splash"`와 `IMAGE_INSTALL += "splash"`의 차이는?
  - `IMAGE_FEATURES`에 할당되는 값은 사전에 정의된 기능 목록에 있음.
    - 즉, 사전 정의된 기능 목록에 splash가 있어하함
    - `IMAGE_FEATURES` 변수는 `FEATURE_PACKAGES` 변수와 함께 사용됨
      - `FEATURE_PACKAGES_feature1 = "package1 package2"`
      - 위 예제의 의미는 `IMAGE_FEATURES`변수에 할당된 값에 "feature1"이 포함되어 있다면 `FEATURES_PACKAGES` 변수에 package1, package2를 포함하게 된다는 의미
      - `IMAGE_FEATURS += "splash"`는 `FEATURE_PACKAGES_splash = "${SPLASH}"` 구문이 처리되게 하고, 루트 파일 시스템에 psplash 레시피에서 만들어지는 패키지가 설치됨

image.bbclass 파일에서 splash 처리

```conf
SPLASH ?= "psplash"
FEATURE_PACKAGES_splash = "${SPLASH}"
```

- `IMAGE_INSTALL`변수에 "splash" 값을 넣는 경우는 원칙적으로 문제가 되지는 않음
  - 그러나 splash라는 이름을 가진 레시피 파일이나 `PROVIDES`를 통해 제공된 이름이 없기 때문에 빌드 자체가 실패함
  - 굳이 넣고자 하면 `IMAGE_INSTALL += "psplash"`와 같이 넣어주면 문제는 되지 않음
  - 다만, 이런 방법은 오픈임베디드 빌드 시스템에서 권장하는 형태가 아님

#### 결론

- 루트 파일 시스템을 만들어 내는 이미지 레시피들(core-image-minimal등)은 상속한 기본 클래스에 따라 미리 정의된 기능 목록이 존재함
- `IMAGE_FEATURES`는 이 기능 목록들 중에서 사용하고자 하는 기능을 추가하는 변수임
- `IMAGE_INSTALL`은 레시피에 의해 만들어지는 패키지를 루트 파일 시스템에 추가하고자 할 때 사용하는 변수임

## 2. 패키지 그룹

> 패키지 그룹 : 이미지에 포함될 수 있는 패키지들의 집합

- 기존 예제에서 core-image-minimal의 최종 산출물인 루트 파일 시스템에 nano, hello 실행 파일을 포함시키려고 `IMAGE_INSTALL` 변수에 패키지 이름을 추가했었음
- 실제로 `IMAGE_INSTALL` 변수는 루트 파일 시스템을 확장하는 데 사용함
- 패키지 그룹도 동작은 이것과 동일함. 다만, 각각의 패키지를 추가하는 것이 아니라, 그룹화한 여러 개의 패키지를 한 번에 추가한다는 점이 다름
- 패키지 그룹은 "packagegroups.bbclass"라는 클래스 파일을 상속함으로써 사용됨
- 패키지 그룹을 사용하려고 만든 레시피 파일은 여러 패키지들을 그룹핑해 의존성만 부여함
  - 다른 레시피 파인 처럼 어떤 것을 빌드하거나, 결과물을 만들지는 않음
- 패키디 그룹의 레시피 파일 이름은 `packagegroup-<name>.bb`와 같이 지어짐

#### core-image-minimal 이미지에 패키지 그룹을 통한 패키지들 추가 예제

- 가칭 great이라는 타깃 시스템을 개발한다고 하자
- 이 시스템 개발을 위해 먼저 루트 파일 시스템 이미지를 만들 메타 레이어로 "meta-great"이라는 디렉터리를 생성하자
- 그리고 이전에 수행했던 hello, nano 패키지들을 패키지 그룹으로 만들어 core-image-minimal 이미지에 추가하자

poky_src/poky/meta-great 디렉터리 구조

```
- meta-great
    - conf
        - layer.conf
    - recipes-core
        - image
            - core-image-minmal-bbappend
        - pacakgegroups
            - packagegroup-great.bb
```

poky_src/poky/meta-great/conf/layer.conf

```conf
BBPATH =. "${LAYERDIR}:"
BBFIELS += "${LAYERDIR}/recipes*/*/*.bb"
BBFILES += "${LAYERDIR}/recipes*/*/*.bbappend"
BBFILES_COLLECTIONS += "great"
BBFILE_PATTERN_great = "^${LAYERDIR}"
BBFILE_PRIORITY_great = "11"
LAYERSERIES_COMPAT_great = "${LAYERSERIES_COMPAT_core}"
```

- BBFILE_PRIORITY : 새로 만든 레이어가 최종적으로 반영돼야 할 메타데이터들을 갖고 있을 거기 때문에 레이어의 우선순위를 기존에 존재하던 레이어들 보다 큰 11로 할당

poky_src/poky/meta-grea/recipes-core/packagegroups/packagegroup.bb

```conf
DESCRIPTION = "this package group is great's packages"
inherit packagegroup

PACKAGE_ARCH = "${MACHINE_ARCH}"
RDEPENDS_${PN} = "\
                hello \
                nano \
                "
```

- hello, nano 패키지를 실행 시간 의존성으로 추가
- `PACAKAGE_ARCH` : 빌드 후 생성된 이미지가 어떤 아키텍처로 생성되어야 하는가?
  - 패키지가 빌드 대상인 특정 머신에 의존적인 경우 추가
  - 빌드 대상인 머신과는 관계없이 모든 아키텍처에 적용되는 패키지라면? `inherit allarch`를 사용
  - poky_src/build/tmp/work 디렉터리를 가보면 머신에 특화된 애들, 모든 곳에서 사용되는 패키지에 따라 디렉터리가 구분됨

poky_src/poky/meta-great/recipes-core/image/core-image-minmal.bbappend

```conf
IMAGE_INSTALL += "packagegroup-great"
```

poky_src/build/conf/bblays.conf

```conf
BBLAYRES ?= " \
            ...
            /home/gukyue88/poky_src/poky/meta-great \
            "
```

- 기존 예제에서는 새로운 패키지를 만들 때 마다 각각의 메타 레이어를 생성 + 각각의 메타 레이어 내에서 생성된 패키지를 이미지를 생성하는 레시피 확장 파일을 통해 추가함
- 여기에서는 이 패키지들을 패키지 그룹으로 묶어 새로 생성된 meta-great 레이어 아래 루트 팡리 시스템을 생성하는 레시피 확장 파일인 core-image-minimal.bbappend에 추가할 것임
- 따라서 meta-hello, meta-nano-editor 레이어들에서 core-image-minmal.bbappend 레시피 확작 파일에 패키지 추가를 위해 넣은 부분을 삭제한다

poky_src/poky/meta-hello/recipes-core/images/core-image-minimal.bbappend

```conf
# IMAGE_INSATALL_append = " hello"
```

poky_src/poky/meta-nano-editor/recipes-core/images/core-image-minmal.bbappend

```conf
# IMAGE_INSTALL_append = " nano"
```

빌드해보자

```bash
bitbake core-image-minimal
runqemu core-image-minmal nographic
```

- 시스템이 부팅되고 로그인한 후 hello, nano 애플리케이션을 실행해보면 정상적으로 실행됨

```bash
# -g 옵션 : 지정한 레시피 파일에 대해 의존성을 가진 모든 패키지들을 pn-buildlist 파일로 출력
bitbake -g great-image
```

- pn-buildlist 파일을 확인해보면 hello, nano 패키지가 설치된 것을 확인할 수 있음

#### 대체 패키지 생성에 따른 의존성 처리

- 8장에서 실행 시간 의존성을 학습했고, 패키지 그룹은 실행 시간 의존성을 이용해 빌드 진행시 미리 의존성이 걸린 패키지를 루트 파일 시스템에 설치한다는 것을 알게됨
- 여기에서는 대체 패키지가 생성됐을 때, 실행 시간 의존성을 사용하는 방법에 대해 배우자
- 기존에 실습했던 hello 패키지를 대체하는 newhello 라는 패키지를 생성하자.
  - 기존의 코드는 수정하지도 삭제하지도 않는 조건 하에 newhello 패키지를 hello 대체 패키지로 만들자

hello 레시피를 위한 작업 디렉터리인 recipes-hello 디렉터리를 복사하여 recipes-newhello라는 디렉터리를 만들자

poky/meta-hello/recipes-newhello 구조

```
- newhello.bb
- source
    - COPYING
    - hello.service
    - newhello.c
```

poky_rsrc/poky/meta-hello/recipes-newhello/newhello.bb

```conf
...
SRC_URI = "file://newhello.c"

...

do_compile() {
    ${CC} newhello.c ${LDFLAGS} -o hello
}

...

RREPLACES_${PN} = "hello"
RPROVIDES_${PN} = "hello"
RCONFILTS_${PN} = "hello"
```

- SRC_URI : hello.c 파일을 newhello.c 파일로 교체
- do_compile : newhello.c 를 컴파일하되 이전과 동일한 hello로 이름을 유지
- RREPLACES, RPROVIDES, RCONFILCTS 변수를 사용해 대체되는 기존 패키지를 사용하지 않도록 하고, 다른 패키지와의 호환성도 보장한다

poky_src/poky/meta-hello/recipes-newhello/source/newhello.c

```c
...
printf("New hello world!\n");
```

poky_src/build/conf/local.conf

```conf
...
BBMASK = "meta-hello/recipes-hello"
```

- bitbake가 기존 hello 레시피 파일들을 처리하지 못하도록 함

```bash
bitbake newhello
bitgbake core-image-minimal -C rootfs
runqemu nographic

# 로그인 후 hello 실행
hello
```

- 실행 결과를 보면 newhello가 실행된 것을 볼 수 있음

## 3. 미리 정의된 패키지 그룹

- 앞에서는 패키지 그룹을 만들고, 이를 `IMAGE_INSTALL` 변수에 할당하여 루트 파일 시스템 이미지에 해당 패키지들을 추가함
- 오픈임베디드 코어에는 이미지들에서 사용할 수 있도록 사전에 만들어진 공용의 패키지 그룹들이 존재함
- 우리가 사용하고 있는 core-image-minmal.bb 레시피 파일 같은 경우도 미리 정의된 패키지 그룹을 사용함
- core-image-minimal.bb 내부 : `IMAGE_INSTALL = "packagegroup-core-boot ${CORE_IMAGE_EXTRA_INSTALL}"`
- packagegroup-core-boot 패키지 그룹은 콘솔을 갖는 부팅이 가능한 이미지를 생성하는데 필요한 최소한의 패키지를 제공함
- packagegroup-core-boot.bb 패키지 그룹 레시피 파일을 열어보면, 패키지를 제공하지도 설치하지도 않음. 다만, 실행 시간 의존성만 줘 각각의 패키지가 루트 파일 시스템이 만들어질 때 이미지에 함께 설치되도록 함
- 자주 사용되는 중요한 패키지 그룹 레시피 파일들은 "poky/meta/recipes-core/packagegroups/"에 정의됨

## 4. 커스텀 빌드 스크립트를 통한 빌드 환경 구축

- 간단한 빌드 스크립트를 만들어 보자
- 기존에 빌드 환경을 초기화할 때는 `oe-init-build-env` 스크립트를 실행했음
- 그럼, 기본적으로 build 디렉터리가 생성되고 내부에 몇 가지 환경 설정 파일과 디렉터리가 만들어짐
- build 디렉터리는 bitbake가 빌드 작업을 수행할 때 산출물들이 만들어지는 곳임
- 앞으로 실습을 진행하면서 machine layer, distribution layer를 만들 것임
- 이에 따라 빌드의 산출물을 따로 구분해 저장해야하기 때문에 빌드 디렉터리를 서로 다른 이름으로 만듦
- 또한 빌드 디렉터리 아래 환경 설정 파일들 (local.conf, bblayers.conf)도 그때마다 바뀌어야함
- 이런 작업들을 수행하기 위해 따로 빌드 스크립트를 만들어 관리할 것임

간단한 빌드 스크립트를 만들어 보자.

#### TEMPLATECONF 변수

- `TEMPLATECONF` : 빌드에 필요한 환경 설정 파일을 어디서 복사해 올지 지정해주는 변수
- 기본적으로 "poky/meta-poky/conf" 디렉터리를 가리킴

#### oe-init-build-env 스크립트의 동작

- `oe-init-build-env` 스크립트를 실행하면 위 디렉터리 아래 bblayer.conf.sample, local.conf.sample 파일이 생기는데 이를 "build/conf" 아래 복사함 (복사를 진행하면 .sample이 삭제됨)
- bblayers.conf.sample 파일에서 기술된 placeholder인 OEROOT 변수는 오픈 임베디드 빌드시스템 특정 파일 절대 경로로 대체
  - bblayer.conf.sample 파일은 build/conf 아래로 복사되면서 OEROOT 변수의 값이 `oe-init-build-env` 스크립트의 절대 경로로 바뀜
- build 디렉터릴를 포함한 소스 전체를 다른 사람에게 배포할 경우 문제되는 것 중 하나느 bblayers.conf 파일이 가지고 있는 `BBLAYERS` 변수가 가지고 있는 절대 경로 (각 PC마다 달라짐)임
- `TEMPLATECONF`라는 변수는 `oe-init-build-env` 스크립트에서 인클루드 하고 있는 poky/scripts/oe-setup-builddir 파일 내에서 처리되는 변수임
- 이 변수는 기본 값으로 poky/meta-poky/conf 디렉터리에 아래 파일들을 build/conf 디렉터리 아래 복사함

oe-setup-builddir

```conf
OECORELAYERCONF="${TEMPLATECONF}/bblayers.conf.sample"
OECORELAYERCONF="${TEMPLATECONF}/local.conf.sample"
OECORELAYERCONF="${TEMPLATECONF}/conf.notes.txt"
```

- 위 스크립트를 보면 복사되는 파일은 bblyaers.conf.sample, local.conf.sample, conf.notes.txt 총 3개임
- conf-notes.txt 안에는 `oe-init-build-env` 스크립트를 실행했을 때 화면에 출력되는 안내 문구들이 들어가 있음

#### 커스텀 빌드 스크립트 작성

- 앞서 만든 meta-great 레이어에 template라는 디렉터리를 만든다
- "poky_src/poky/meta-poky/conf" 디렉터리 아래 bblayers.conf.sample, local.conf.sample, conf-notes.txt 파일을 template 디렉터리 아래 복사해오자

bblayers.conf.sample 파일 수정

```conf
POKY_BBLAYERS_CONF_VERSION = "2"
BBPATH = "${TOPDIR}"
BBFILES ?= ""
BBLAYERS ?= " \
    ##OEROOT##/meta \
    ##OEROOT##/meta-poky \
    ##OEROOT##/meta-yocto-bsp \
    ##OEROOT##/meta-hello \
    ##OERROT##/meta-nano-editor \
    ##OEROOT##/meta-great \
    "
```

local.conf.sample 파일을 열어 제일 하단에 아래 내용을 추가

- 이건 책 274 페이지 리스트 9-14 참고
- 추가하는 내용은 앞서 학습한 내용으로 실습의 연속성을 위해 추가한 부분임

conf-notes.txt 파일은 oe-init-build-env 스크립트를 실행하고 나면 화면에 출력되는 안내 문구 인데 아래와 같이 수정하자

```
### Shell environment set up for builds ###
Welcome@ This is my yocto example.
You can now run 'bitbake <target>'

Commopn targets are:
    core-image-minmal

You can also run generate qemu images with a command like 'runquemu qemux86'
```

그 다음 빌드 스크립트(poky_src/buildenv.sh)를 작성하자

```bash
#!/bin/bash

function find_top_dir()
{
    local TOPDIR=poky
# move into script file path
    cd $(dirname ${BASH_SOURCE[0]})
    if [ -d $TOPDIR ]; then
        echo $(pwd)
    else
        while [ ! -d $TOPDIR ] && [ $(pwd) != "/" ];
        do
            cd ..
        done
        if [ -d $TOPDIR ]; then
            echo $(pwd)
        else
            echo "/dev/null"
        fi
    fi
}

ROOT=${find_top_dir}
export TEMPLATECONF=${ROOT}/poky/meta-great/template/
source poky/oe-init-build-env build2
```

- 스크립트 요약 : `TEMPLATECONF` 변수에 우리가 만들어 놓은 template 디렉터리를 할당한 후 oe-init-build-env 스크립트를 실행할 때 build 디렉터리 내의 conf 디렉터리 아래 관련 환경 설정 파일들이 복사되도록 하는 것임

```bash
chmod 777 buildenv.sh
source buildenv.sh
```

- 위 스크립트를 실행하면 conf-notes.txt 내용이 출력됨
- 다음 현재 작업을 진행하는 디렉터리의 위치가 build2 디렉터리로 바뀌게 됨
- build2 디렉터리를 확인해보면 bblayers.conf, local.conf가 복사된 것을 볼 수 있음
- templateconf.cfg 파일에는 현재 만들어진 환경 설정 파일을 어디서 복사했는지 기술되어 있음

```bash
bitbake core-image-minimal
```

- bitbake는 새로 빌드를 수행하며 생성되는 모든 산출물들을 build2 디렉터리에 저장함

## 5. 요약

- 9장은 10장에서 진행할 커스텀 이미지를 생성하기 위한 전 단계임
- 이미지 레시피를 통해 루트 파일 시스템에 패키지를 추가할 때는 `IMAGE_INSTALL` 변수에 설치하고자 하는 패키지 이름을 추가함
- 실제 최종 루트 파일 시스템에는 `IMAGE_FEATURES` 변수에 할당된 features도 `IMAGE_INSTALL` 변수에 할당된 패키지들과 함께 설치됨
- `IMAGE_FEATURES` 변수에서 추가한 기능들은 이미지 클래스들에서 미리 정의된 기능 목록들 중 필욯나 기능들을 추가하기 위해 사용함
- 패키지 그룹은 packagegroups.bbclass라는 클래스 파일을 상속함으로써 사용함
- 패키지 그룹 레시피 파일은 어떤 것도 빌드하거나 결과물도 만들지 않음. 다만, 여러 패키지들을 그룹핑해 실행 시간 의존성만 부여하는 특징을 가짐
- 오픈임베디드 코어에서는 이미지들에서 사용할 수 있도록 사전에 만들어진 패키지 그룹들이 존재함
- 우리는 이 패키지 그룹을 사용해 core-image-minimal이라는 이미지 레시피를 구성해봄
- 또 앞으로 실습을 위해 간단한 빌드 스크립트를 작성해봄
- 이 스크립트는 차후 machine layer와 distribution layer를 생성하면서 빌드 환경 구축을 도울 것임
