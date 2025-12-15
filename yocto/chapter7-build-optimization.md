# Chapter 7. 유용한 오픈임베디드 코어 클래스 기능을 사용한 빌드 최적화

- GNU Autotools : GNU 빌드 시스템으로 Unix 기반 시스템에서 소스코드를 빌드할때 도움을 주는 빌드 툴
- Autotools는 서로 다른 환경(플랫폼, 머신)에도 불구하고 개발자가 차이점을 이해하지 않아도 빌드가 가능하도록 환경 설정을 해주는 유용한 툴
- 오픈임베디드 빌드 시스템 내에서는 Autotools를 다루는 함수들이 클래스 파일에 존재하여, 세세하게 Autotools를 다루는 방법을 알 필요는 없음
- nano editor 코드를 받아 Autotools로 빌드해보자
  - externalsrc.bbclass 클래스 파일을 사용해 변경하고자 하는 소스를 로컬에 저장해 편집할 수 있도록 해보자(?)
    - 코드를 수정했는데, 다시 소스를 새롭게 받아와서 수정한 소스가 사라지지 않도록 해보자

## 1. Autotools를 이용한 nano editor 빌드

- 소스 파일이 한 두개 일 때는 컴파일하고 설치하는 것이 쉬우나, 많은 소스 파일을 컴파일하고 설치하는 것은 매우 어려움
- 또, 세상에는 다양한 아키텍처를 가진 머신이 존재하고, 아키텍처에 따라 컴파일러도 바뀌어야 함
- 소프트웨어 패키지를 컴파일하고 설치하는데 있어서 위 불편함을 최소화할 수 있도록 Autotools 빌드 시스템을 사용함

#### Autotools는 autoconf, automake, libtool 도구들로 구성된 빌드 툴임

- autoconf : configure.ac를 파싱 -> configure 스크립트 생성 및 실행 -> 최종 Makefile 생성
- automake : Makefile.am을 파싱 -> Makefile.in 생성 -> configure 파일에서 Makefile 생성 시 참고
- libtool : 라이브러리 생성 처리

#### nano editoc 소스 다운로드 및 md5sum 값 구하기

```bash
wget https://www.nano-editor.org/dist/v65/nano-6.0.tar.gz
md5sum nano-6.0.tar.gz
tar -xvf nano-6.0tar.gz
cd nano-6.0
md5sum COPYING
md5sum COPYING.DOC
```

#### meta-nano-editor 레이어 만들고 빌드 해보기

```
- poky_src/poky
    - meta-nano-editor
        - conf
            - layer.conf
        - recipes-nano
            - nano_6.0.bb
```

poky_src/poky/meta-nano-editor/conf/layer.conf

```conf
BBPATH =. "${LAYERDIR}"
BBFILES += "${LAYERDIR}/recipes*/*.bb"
BBFILE_COLLECTIONS += "nano-editor"
BBFILE_PATTERN_nano-editor = "^${LAYERDIR}/"
BBFILE_PRIORITY_nano-editor = "10"
LAYERSERIES_COMPAT_nano-editor = "${LAYERSERIES_COMPAT_core}"
```

poky_src/poky/meta-nano-editor/recipes-nano/nano_6.0.bb

```conf
DESCRIPTION = "Nano editor example"
LICENSE = "GPLv3"
LIC_FILES_CHKSUM = "file://COPYING;md5=... \
                    file://COPYING.DOC;md4=..."
SRC_URI = "https://www.nano-editor.org/dist/v6/nano-6.0.tar.gz"
SRC_URI[md5sum] = "..."
DEPENDS = "ncurses"

inherit gettext pkgconfig autotools
```

- bitbake는 checksum을 이용해 다운로드 파일을 검증함. (md5 나 sha256 중 하나는 반드시 있어야 함)
- 앞서 구한 md5 내용을 넣어준다.
- `DEPENDS` : 의존성을 나타내며, ncurses라는 패키지와 의존성을 가졌고, ncurses라는 패키지가 앞서 필요하다는 것을 bitbake에게 알려줘서 패키지간의 빌드 우선순위를 정하게 함
- `inherit gettext pkgconfig autotools` : 오픈임베디드 코어에서 제공하는 클래스 파일을 사용해 Autotools의 기능을 사용하는 것
  - Yocto에서 제공하는 기능을 사용하고자 할 때는 단순히 Yocto에서 제공하는 클래스 파일을 상속하는 것만으로도 기능을 사용하는데 충분(?)

poky_src/build/conf/bblayers.conf

```conf
...
BBLAYERS ?= " \
            ...
            /home/gukyue88/poky_src/poky/meta-nano-editor
            "
```

#### 빌드

```bash
bitbake nano
# poky_src/build/tmp/work/core2-64-poky-linux/nano/6.0-r0/image/usr/bin에 nano 실행파일이 생성됨
```

> nano editor 소스 빌드를 위해 한 일 : 소스의 위치를 레시피 파일에 추가 + Autotools 클래스 파일을 상속받은 것
> 나머지는 bitbkake가 Autotools 클래스를 비롯한 메타데이터들을 이용해 우리가 생성하지 않은 여러 태스크를 실행하고 알아서 최종 바이너리까지 생성함

#### nano editor 실행

- nano editor 바이너리를 루트 파일 시스템에 넣어 최종 타깃에서 결과를 확인해보자

poky_src/poky/meta-nano-editor/recipes-core/images/core-image-minimal.bbappend

```conf
IMAGE_INSTALLL += "nano"
```

poky_src/poky/meta-nono-editor/conf/layer.conf

```conf
...
BBFILES += "${LAYERDIR}/recipes*/*/*.bbappend"
...
```

```bash
bitbake nano -c clean all && bitbake nano
bitbake core-image-minimal -C rootfs
runqemu core-image-minimal nographic

# 로그인 후 nano 실행
```

## 2. 빌드 히스토리

- 빌드 히스토리 : Yocto에서 제공하는 이미지의 변경점을 추적하고 변경된 이미지가 수행한 빌드 절차를 이전과 비교할 수 있는 기능
- 빌드 히스토리 기능을 사용하려면 local.confg 파일에 buildhistroy.bbcalss 클래스 파일과 관련 변수 몇 개를 추가해야함

poky_src/build/conf/local.conf

```conf
...
INHERIT += "buildhistory"
BUILDHISTORY_COMMIT = "1"
BUILDHISTORY_COMMIT_AUTHOR = "gukyue <gukyue88@gmail.com>"
BUILDHSITORY_DIR = "${TOPDIR}/buildhistory"
BUILDHISTORY_IMAGE_FILES = "/etc/passwd /etc/group"
```

> 위에처럼 하면 buildhsitory.bbclss 클래스를 상속해 활성화 시킬 수 있음

- BUILDHISTORY_COMMIT : git repository에 빌드 히스토리를 커밋 하도록 함
- BUILDHSITORY_COMMIT_AUTHOR : repository에 커밋할 때 제공하는 사용자의 이름
- BUILDHISTORY_DIR : buildhistory.bbclass가 빌드히스토리를 저장하는 디렉터리의 경로
- BUILDHISTORY_IMAGE_FILES : 루트 파일 시스템에 설치된 특정 파일에 대한 경로를 할당해 내용을 추적할 수 있도록 함(?)
  - 기본값으로 `/etc/passwd`, `/etc/group`를 가짐
  - 사용자 및 그룹 항목의 변경을 모니터링 하겠다는 의미

#### inherit vs INHERIT

- inherit : 클래스를 상속받고자 하는 레시피 파일에서 사용, 사용범위가 레시피 파일 내로 제한
- INHERIT : 환경 설정 파일에서 사용, 환경 설정 파일은 전역적인 성격을 띔, 즉 모든 레시피 파일들이 클래스를 상속받게 함

#### 재빌드

```bash
bitbake hello -c cleanall && bitbake hello
bitbake hello -c cleanall && bitbake nano
bitbake core-image-minimal -C rootfs
```

- 빌드가 완료되면 build 디렉터리 아래 `buildhistory` 디렉터리가 생성된 것을 볼 수 있음

#### buildhistory

buildhistory 디렉터리 아래의 파일 중 2개의 중요한 파일을 살펴보자

- Image-info.txt : 루트 파일 시스템 이미지 내용의 변경 추적에 사용, 내용 : 이미지 생성을 위해 사용된 클래스 목록과 패키지들 등
- build-id.txt : 이미지 생성을 위해 사용된 빌드 환경에 대한 정보 제공

#### core-image-minimal 이미지 레시피에 usbutils 패키지를 설치해보자

poky_src-build/conf/local.conf

```conf
...
CORE_IMAGE_EXTRA_INSTALL += "usbutils"
```

```bash
bitbake core-image-minimal
buildhistory-diff
```

- buildhsitory-diff를 사용하면, usbutils를 추가 설치 하기 전과 후의 buildhistory의 diff를 확인할 수 있음

## 3. rm-work를 통한 디스크 크기 절감

#### rm-work

- rm-work : 오픈임베디드 빌드 시스템이 빌드 작업을 끝내고 작업한 파일들을 삭제하는 태스크
- rm_work.bbclass 클래스가 이 태스크를 제공함

#### rm-work 세팅

poky_src/poky/meta-nano-editor/appends/nano_6.0.bbappend

```conf
inherit rm_work
```

- nano_6.0 레시피에서만 rm_work 클래스를 상속받으려고 inherit 지시자를 사용함

poky_src/poky/meta-nano-editor/conf/layser.conf

```conf
...
BBFILES += "${LAYERDIR}/appends/*.bbappend"
```

#### rm-work 확인

```bash
# rm_work가 적용되기 전 nano editor 빌드 작업 디렉터리 크기 확인
du -sh 6.0-r0

# nano 재빌드
bitbake nano -c cleanall && bitbake nano

# rm_work가 적용된 후 nano editor 빌드 작업 디렉터리 크기 확인
du -sh 6.0-r0
```

## 4. externalsrc를 이용한 외부 소스로부터 소스 빌드

- 소스 코드를 받는 방법에 따라 로컬에 위차하는 장소가 달라짐
  - tarball 과 같은 압축 파일은 압축이 풀리면서 : `${WORKDIR}/${BP}`
  - git으로 부터 소스를 받은 경우 : `${WORKDIR}/git`
  - 로컬에서 직접 사용하는 경우 : `${WORKDIR}`

#### nano editor 소스 변경시 문제

1. 레시피의 내용이 업데이트 된 경우 do_fetch 태스크부터 다시 수행하거나 fetch와 같은 명령이 수행되면 변경한 코드가 모두 사라짐
2. 소스 코드가 바뀐 경우 do_unpack 태스크부터 다시 수행하기 때문에 수정한 코드를 보존하면서 빌드에 적용하려면 `bitbake -C compile`과 같이 명령을 입력해야함
3. 코드 수정 후 빌드를 수행하면 필요 없는 태스크들이 추가로 수행되기 때문에 빌드 시간이 오래 걸림
4. build 디렉터리를 삭제하는 경우, 변경한 코드가 사라짐

> 이 문제를 해결할 수 있는 방법 중 하나는 externalsrc 클래스임

#### externalsrc 클래스 적용

```bash
cd poky_src

# 소스를 로컬에 담을 디렉터리 생성
mkdir source
mkdir source/nano

# build 디렉터리의 nano editor 소스 코드를 새로 생성한 디렉터리로 복사
cp -r build/tmp/work/core2-64-poky-linux/nano/6.0-r0/nano-6.0/* source/nano/
```

nano_6.0.bbappend 파일 수정

```conf
# Inhrit rm_work
inherit externalsrc
EXTERNALSRC = "${COREBASE}/../source/nano"
```

- rm_work와 externalsrc 클래스 기능은 상충되기 떄문에 rm_work 클래스를 주석처리함

```bash
bitbake nano -c cleanall && bitbake nano
```

- 빌드후 기존 소스코드가 있던 곳을 가보면 소스코드가 없어진 것을 볼 수 있음
- log.task_order 파일을 열어보면 fetch, unpack, patch 태스크 실행이 생략된 것을 볼 수 있음

## 5. 요약

- Poky가 포함하고 있는 meta 디렉터리는 오픈임베디드 코어라고 불리며, 수많은 유용한 메타데이터를 가짐
- 특히, 오픈임베디드 코어에 포함된 각각의 클래스 파일들은 특정 기능을 포함하고 있음
- 여기에서는 systemd, Autotools, buildhistory, rm-workd, externalsrc 클래스 파일들에 대해 다룸
- 이런 클래스 파일들은 단순히 레시피 파일에서 상속만 함으로써 그 기능을 활성화 시킬 수 있음
- 이는 추상화에 따른 단순화의 예로 볼 수 있음
