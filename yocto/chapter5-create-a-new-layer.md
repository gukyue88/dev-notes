# Chapter 5. 새로운 레이어를 만들고 레시피 생성

- 새로운 레이어를 만들어 보고 레이어 내에 간단한 레시피 파일을 작성해보자
- 몇 가지 문법을 간단한 예제와 함께 익히고 "hello world!" 애플리케이션을 만들어 보자

## 1. 문법을 실습할 예제 작성

- 간단한 예제를 만들어보자
- bitbake의 예제와 비슷하나, 앞서 만든 예제는 Poky를 받지 않은 상태에서 bitbake만 사용한 것
- 여기서는 Poky 내에 새로운 레이어를 생성하고 예제 레시피를 만들어서 진행한다는 것

```bash
# poky 작업폴더로 이동
cd poky_src

mkdir poky/meta-hello
mkdir poky/meta-hello/conf
mkdir poky/meta-hello/recipes-hello

touch poky/meta-hello/conf/layer.conf
touch poky/meta-hello/recipes-hello/hello.bb
```

#### poky/meta-hello/conf/layer.conf

```conf
BBPATH =. "${LAYERDIR}:"
BBFILES += "${LAYERDIR}/recipes*/*.bb"
BBFILE_COLLECTIONS += "hello"
BBFILE_PATTERN_hello = "^${LAYERDIR}/"
BBFILE_PRIORITY_hello = "10"
LAYERSERIES_COMPAT_hello = "${LAYERsERIES_COMPAT_core}"
```

#### poky/meta-hello/recipes-hello/hello.bb

```
DESCRIPTION = "Simple hello example"
LICENSE = "CLOSED"

do_printhello() {
    bbwarn "hello world!"
}
addtask do_printhello after do_compile before do_install
```

- `SUMMARY` : 패키지에 대한 간단한 소개, 80자 제한
- `DESCRIPTION` : 패키지에 대한 자세한 설명, 여러줄 가능
- `AUTHOR` : 패키지 저자의 이름과 이메일 주소
- `HOMEPAGE` : 소프트웨어 패키지가 제공되는 주소, http:// 로 시작되는 URL
- `BUGTRACKER` : 버그 추적 시스템의 주소, http:// 로 시작되는 URL

#### poky_src/build/conf/bblayers.conf

- bitbake는 빌드 진행 시 환경 설정 파일인 bblayers.conf 내의 `BBLAYERS` 변수를 통해 레이어들을 파악함
- 새로 추가된 레이어인 meta-hello를 bitbake가 알 수 있도록 추가

```conf
BBLAYERS ?= " \
    ...
    /home/gukyue88/poky_src/poky/meta-hello \
```

#### 레이어 정상 추가 확인

```bash
bitbake-layers show-layers
```

#### 실행

```bash
bitbake hello

# 만약 bitbake 명령을 찾을 수 없다고 나온다면?
# source poky/oe-init-build-env
```

## 2. bitbake 문법 두 번째

#### 변수의 범위

- 환경 설정 파일(.conf)에서 정의된 변수는 전역변수
  - 전역변수는 모든 레시피 파일(.bb, .bbappend)에서 사용이 가능함
- 레시피 내에서 선언된 변수는 해당 레시피에서만 사용 가능

#### 문법

- `=` : 변수 할당
- `?=` : default 할당 연산자, 변수가 이전에 정의되지 않은 경우에 값을 할당
- `??=` : 약한 default 할당 연산자, 변수가 이전에 정의되지 않은 경우 값을 할당, 다만 `??=` 가 중복될 경우 뒤에 값을 사용
- `:=` : 즉시 할당, `${변수명}`는 원래 변수가 실제 사용되는 순간에 확장(대입)이 되는데, `:=`를 사용하게 되면 바로 확장(대입)이 일어남
- `+=` : 후입, 원래 값에 공백을 추가하면서 뒤쪽에 값을 추가함
- `=+` : 선입, 원래 값에 공백을 추가하면서 앞쪽에 값을 추가함
- `.=` : 공백없는 후입, 원래 값에 공백 없이 뒤쪽에 값을 추가함
- `=.` : 공백없는 선입, 원래 값에 공백 없이 앞쪽에 값을 추가함

```bash
VAR1 = "A"
VAR2 = "${VAR1}B"
VAR3 := "${VAR2}C"

do_printhello() {
    bbwarn "VAR3: ${VAR3}"
}
add task do_printhello after do_compile before do_install
```

- 위 출력 결과는 "VAR3 : BC"가 됨
- 변수의 확장은 변수가 실제 사용되는 순간임
- VAR2는 아직 사용되기 전이라서 변수의 확장이 일어나지 않음. 즉, VAR2 = "B"인 상태임
- `:=`를 사용해 VAR3에서 VAR2를 즉시 할당하면, 확장되지 않은 상태의 VAR2 내용이 반영되어 VAR3에 할당됨

#### 주의

- 전역 변수를 태스크 내에서 변경해도 다른 태스크에 영향을 주지 않는다.
  - bitbake는 여러 메타데이터를 바탕으로 레시피의 특정 태스크를 다시 구성함
  - 이때 태스크 내의 변수는 이미 확장된 상태로 셀 스크립트에 기술됨

#### 연산자 적용 순위

1. metadata
2. =, :=, ?=, ??=, +=, =+, .=, =.
3. `_append`, `_prepend`, `_remove`

## 3. hello 애플리케이션 레시피 작성

- 조금 구색을 갖춘 레시피 파일을 만들어 보자
- bitbake로 간단한 c 파일을 빌드하고, 빌드 결과로 나온 이미지를 루트 파일 시스템에 넣어 수행해본다
- 위에서 했던 예제를 계속 끌고 감

```bash
mkdir poky/meta-hello/recipes-hello/source
touch poky/meta-hello/recipes-hello/source/hello.c
touch poky/meta-hello/recipes-hello/source/COPYING # 라이센스 파일
```

#### hello.c

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    int i = 0;
    while (i < 10) {
        printf("Hello world!\n");
        sleep(1);
    }
    return 0;
}
```

#### COPYING

- 테스트 용도로 만든 라이센스 파일이기 때문에 아무 내용이나 입력
- 새로운 레시피 파일이 생성되면 적절한 라이선스와 라이선스 설명 파일에 대한 checksum을 지정하는 것이 필수이기 때문에 COPYING 파일을 생성

```
EXMAPLE LICENSE FILE

This is example license file.
```

#### hello.bb

```bash
DSCRIPTION = "Simple helloworld application example"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://COPYING;md5=80cae152345134tedfg234534t..."

SRC_URI = "file://hello.c"
SRC_URI_append = " file://COPYING"
S = "${WORKDIR}"

do_compile() {
    ${CC} hello.c ${LDFLAGS} -o hello
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 hello ${D}${bindir}
}

FILEEXTRAPATHS_preappend := "${THISDIR}/source:"
FILES_${PN} += "${bindir}/hello"
```

- `SRC_URI` : fetch 위치
- `FILESEXTRAPATHS_prepend := "${THISDIR}/source"` : URI는 상대 경로, 절대 경로가 다 가능한데 상대 경로를 얻으려면 파일이 위치하는 경로를 알려줘야함
  - THISDIR 은 bitbake가 현재 파싱하는 파일이 위치하고 있는 디렉터리
  - FILESEXTRAPATHS 변수는 FILESPATH를 확장하고, "poky/meta/classes/base.bbclass" 클래스 파일에 정의되어 있음
  - 오픈임베디드 시스템이 패치 및 파일을 검색할 때 사용하는 디렉터리 리스트들을 가지고 있음
  - `bitbake-getvar -r hello FILESPATH`를 하면 FILESPATH 변수의 내용을 볼 수 있음
  - 오픈임베디드 빌드 시스템은 이 경로에서 SRC_URI에 추가한 파일이나 패치를 찾음
  - SRC_URI를 추가할 떄, 이 경로들에 포함이 안된 파일을 추고 하고 싶다면 FILESEXTRAPATHS를 활용함
- `LIC_FILES_CHKSUM` : `md5sum COPYING`을 통해 값을 얻을 수 있음
- `WORKDIR` : bitbake가 주어진 레시피를 빌드하면서 생성된 각종 정보를 저장하는 디렉터리의 경로
- `S` : bitbkae가 빌드 시 압축된 소스를 해제해 위치시키는 디렉터리이자, SRC_URI에서 로컬 소스 지정 시 해당 소스가 빌드를 진행할 때 복사돼 위치하는 디렉터리
- `do_compile`, `do_install` : base.bbclass에서 사전 정의된 태스크이고,서여기에서는 해당 태스크들을 override 해서 사용한 것
- `install -d xxx` : xxx 디렉터리를 생성
- `install -m 0755 hello xxx` : hello 실행 파일 권한을 0755로 주고, hello 파일을 xxx 디렉터리로 복사하겠다는 뜻
- `${bindir}` : 사전에 정의된 디렉터리 이름으로 `/usr/bin`을 나타냄, 이외 sbindir, libdir, libexecdir, sysconfdir, datadir, mandir, includedir등이 있다
- `${D}` : 빌드의 결과물로 생성된 바이너리가 위치하는 경로, `${WORKDIR}/image` 값을 가짐
  - 프로그램 파일은 일반적으로 다음과 같은 위치에 설치됨
    - 사용자 프로그램 -> `/usr/bin`
    - 시스템 관리 프로그램 -> `/usr/sbin`
    - 라이브러리 -> `/usr/lib`
    - 환경 설정 파일 -> `/etc`

#### 빌드

```bash
bitbake hello -c cleanall
bitbake hello
```

- `bitbake hello` 명령은 앞선 예제처럼 `addtask`를 통한 실행해야 하는 태스크를 지정하지 않음
- 태스크를 지정하지 않은 경우 기본 태스크를 수행함
- `BB_DEFAULT_TASK` 라는 변수에 기본태스크가 담겨있고, "build"를 기본값으로 가짐
- 기본 태스크인 build를 실행한다는 것은 빌드를 끝까지 진행한다는 뜻임

## 4. 라이선스

## 5. 레시피 확장 파일

## 6. BBFILE_COLLECTIONS, BBFILE_PATTERN 변수의 역할

## 7. 요약s
