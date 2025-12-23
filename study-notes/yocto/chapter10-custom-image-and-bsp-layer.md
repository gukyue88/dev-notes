# Chapter 10. Poky 배포를 기반으로한 커스텀 이미지, BSP 레이어 작성

'great'라는 이름을 가진 가상의 타깃 시스템을 만들자

#### 시스템 구성

- 바닐라 커널 버전 5.4, u-boot
- 머신 이름 : great
- 배포 이름 : great-distro
- 이미지 지원 기능 : splash, great 계정 및 group 계정 추가, password 지원, 기존에 작성했던 nano, hello 패키지 추가 등

#### great 시스템 전체 구조

- 실습에서 만들어 갈 레이어
  - 커스터머 레이어 (./meta-myproject)
  - 배포 레이어 (./meta-great)
  - bsp 레이어 (./meta-great-bsp)
- 기본적으로 poky에서 제공하는 레이어
  - 포키 참조 배포 레이어 (./meta-poky)
  - oe-core (./meta)

## 1. 커스텀 이미지 레시피 생성

- 이전까지는 poky에서 제공한 'core-image-minimal.bb' 이미지 레시피 파일을 확장해 루트 파일 시스템을 구성함
- 이 이미지는 오픈임베디드 코어에서 제공한 것으로 사전에 만들어진 이미지 레시피 파일임
- 여기에서는 커스텀 이미지 레시피를 생성해보자
- 커스텀 이미지 레시피를 포함하는 메타 레이어는 기존에 추가한 'meta-great'을 그대로 사용하고, 커스텀 이미지를 생성하는 이미지 레시피 파일로 'great.bb'를 만들 것임
- 생성할 이미지 타입과 크기 및 사전에 생성된 패키지 그룹을 포함할 클래스 파일도 함께 생성

#### meta-great 디렉터리의 전체 구조

```
- poky_src/poky
    - meta-great
        - classes
            - great-base-image.bbclass
        - conf
            - layser.conf
        - recipes-core
            - images
                - core-image-minimal.bbappend
                - great-image.bb
            - packagesgroups
                - packagegroup-great.bb
        - template
            - bblayers.conf.sample
            - conf-notes.txt
            - local.conf.sample
```

#### meta-great/conf/layer.conf

```conf
BBPATH =. "${LAYERDIR}:"
BBFILES += "${LAYERDIR}/recipes*/*/*.bb"
BBFILES += "${LAYERDIR}/recipes*/*/*.bbappend"
BBFILE_COLLECTIONS += "great"
BBFILE_PATTERN_great = "^${LAYERDIR}/"
BBFILE_PRIORITY_great = "11"
LAYERDEPENDS_gret = "core"
LAYERSERIES_COMPAT_great = "${LAYERSERIES_COMPAT_core}"
```

- great 레이어는 core(오픈임베디드 코어) 레이어에 의존성을 가짐

#### meta-great/classes/great-base-image.bbclass

```conf
inherit core-image
IMAGE_FSTYPES = " tar.bz2 ext4"
IMAGE_ROOTFS_SIZE = "10240"
IMAGE_ROOTFS_EXTRA_SIZE = "10240"
IMAGE_ROOTFS_ALIGNMENT = "1024"
CORE_IMAGE_BASE_IONSTALL = "\
    packagegroup-core-boot \
    packagegroup-base-extended \
    ${CORE_IMAGE_EXTRA_INSTALLL} \
"
```

- `IMAGE_FSTYPES` : tar.bz2와 ext4 타입의 루트 파일 시스템 2개 생성
- `IMAGE_ROOTFS_SIZE` : 루트 파일 시스템 이미지 (KB), 기본값이고 빌드시 커질 수 있음
- `IMAGE_ROOTFS_EXTRA_SIZE` : 루트 파일 시스템에 추가적인 공간 (KB)
- `IMAGE_ROOTFS_ALIGNMENT` : 루트 파일 시스템 이미지 크기를 이 변수의 배수로 맞춤

#### meta-great/recipes-core/images/great-image.bb

```conf
SUMMARY = "A very small image for yocto test"
inherit great-base-image
LINGUAS_KO_KR = "ko-kr"
LINGUAS_EN_US = "en-us"
IMAGE_LINGUAS = "${LINGUAS_KO_KR} ${LINGUAS_EN_US}"
IMAGE_INSTALL += "packagegroup-great"
IMAGE_OVERHEAD_FACTOR = "1.3"

inherit extrausers

EXTRA_USERS_PARAMS = "\
    groupadd greatgroup; \
    useradd -p `openssl passwd 9876` greate; \
    useradd -g greatgroup great; \
"
```

- great-base-image 클래스 파일 상속
- 한글과 영어 지원
- 앞 예제에서 만든 packagegroup-great 패키지 포함(hello, nano 패키지)
- `IMAGE_OVERHEAD_FACTOR` : 30% 여유 공간을 추가해 이미지 생성
- extrausers.bbclass 클래스 파일을 상속 받아 사용자 및 그룹 추가

#### meta-great/template/conf-notes.txt

```
### Shell environment set up for builds. ###
Welcome! This is my yoctor example.
You can now run 'bitbake <target>'

Common targets are:
    great-image

You can also run generated qemu images with a command like 'runqemu qemux86'
```

- buildenv.sh 파일 실행시 화면 출력용 파일

#### meta-great/template/local.conf.sample

```conf
...
EXTRA_IMAGE_FEATURES += "splash"
```

- splash 기능 추가

#### 빌드

```bash
source buildenv.sh
bitbake great-image
runqemu great-image nographic
```

## 2. BSP 레이어

- 원래 BSP는 OS & 장치드라이버 & 부트로더를 의미함
- 그런데 Yocto의 BSP 레이어외 오픈임베디드 코어에서 일부 BSP 기능을 포함함
- 그럼에도 불구하고 하드웨어와 관련된 필요한 기능은 BSP 레이어에 추가해야 올바른 방법임
- Poky 소스 중 "meta-yocto-bsp"가 Poky가 제공한 BSP 레이어임.
- 우리는 QEMU를 사용하기 때문에 위 레이어가 아닌 오픈임베디드 코어를 참고하면서 진행함
- meta 레이어에서 conf 디렉터리 아래 machine 디렉터리가 오픈임베디드 코어에 포함된 BSP 레이어임
- qemu 에 대한 머신 환경 설정 파일은 `poky/meta/conf/machine/qemux86-64.conf`임
- 이 파일의 내용을 전부 다루진 않고 기본적인 변수나 사용법 정도 확인하고 넘어가자

## 3. bitbake 문법 세 번째

#### 조건부 변수 할당 (OVERRIDES)

```
OVERRIDES = "korean:american:vietnamese"

FOOD_korean = "rice"
FOOD_american = "bread"
FOOD_british = "sandwitch"
CLOTHES = "kilt"
CLOTHES_korean = "hanbok"
```

- OVERRIDES 변수는 오른쪽으로 갈수록 우선순위가 높음
- `FOOD_britsh` 구문은 실행되지 않음
- 우선 순위에 따라 `FOOD` 변수에 "bread"가 할당됨
- 우선 순위에 따라 `CLOTHES` 변수에 "hanbok"이 할당됨

> Yocto 버전에 따라 조건부 변수의 할당 방법의 차이가 있음

## 4. 커스텀 BSP 레이어 만들기

#### meta-great-bsp 디렉터리 생성

```
- meta-great-bsp
    - conf
        - layer.conf
        - machine
            - great.conf
```

#### bblayers.conf.sample

```conf
...
BBLAYERS ?= " \
    ...
    ##OEROOT##/meta-great-bsp \
"
```

- meta-great-bsp 레이어 추가

#### 환경 재설정

```bash
rm -rf build/conf
source buildenv.sh
```

#### meta-great-bsp/conf/layer.conf

```conf
참고
```

#### great.conf

```conf
참고
```

## 5. 요약
