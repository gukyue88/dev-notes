# Chapter 4. 빌드 속도 개선을 위한 작업들

- `bitbake core-image-minimal` 명령어로 루트 파일 시스템 이미지를 생성해보면 빌드하는데 오랜 시간이 걸림
- 시간이 많이 걸리는 이유 2가지
  - 외부 저장소에서 소스 fetch를 하는 과정
  - 빌드를 진행하는 과정

> 1. 로컬 소스 저장소를 구축하여 외부 저장소로부터의 다운로드 시간을 단축하자!
> 2. Yocto에서 제공하는 공유 상태 캐시 (Shared State Cache)라는 캐시 저장소를 구축해 빌드 시간을 줄이자!

- 로컬 소스 저장소 == Yocto에서는 `mirror` 라고 부름
- 미러를 생성하면 팀원 각자가 외부 네트워크에 접근해 소스를 받아오는 부하를 최소화할 수 있음

## 1. 소스 받기

#### 오픈임베디드 빌드 시스템은 어떻게 필요한 소스를 받아오나? (소스 fetch 방법)

```
Source Mirrors ------------> | ------------------ |
                             |                    |
Upstream Project Release --> |                    |
                             | download directory | --> bitbake
Local Projects ------------> |                    |
                             |                    |
SCMs (optional) -----------> | ------------------ |
```

- Source Mirrors : 외부에 archive 파일 형태로 저장된 저장소 + Local에 archive 파일 형태로 저장된 저장소
- Upstream Project Release : tarball과 같은 압축 파일로 돼 있는 software package 들 (xxx.tar.gz, xxx.tar.bz2 등)
- Local Projects : Local에 저장된 source package 들
- SCMs (optional) : SCM으로 저장된 software package 들 (Git, Subversion 등)
- downlaod directory : 받은 소스를 저장하는 역할

> 소프트웨어 스택 빌드를 위한 소프트웨어 패키지들은 웹사이트, 파일 서버, SCM(Git, Subversion 같은 버전관리 시스템)을 통해 제공받아야함

#### SRC_URI 변수

- 레시피 파일 (.bb, .bbappend)에서는 `SRC_URI` 변수에 URI 값을 넣어 소스를 받을 위치를 정의함
- 예. `/poky/'meta/recipes-kernel/linux/linux-yocto_5.4.bb` 파일에 기술된 `SRC_URI`를 보면 커널 소스를 Git으로 부터 받아오도록 되어있음.

#### DL_DIR 변수

- 받은 소스는 `DL_DIR` 변수가 가리키는 경로에 저장됨
- `DL_DIR` 변수의 기본 값은 `${TOPDIR}/downloads`임
  - 여기서 `${TOPDIR}`은 build 디렉터리임

```bash
# core-image-minimal의 DL_DIR 값 확인
bitbake-getvar -r core-image-minmal DL_DIR
```

#### build/download 디렉터리 내부 내용

```
tzdata2022d.tar.gz
tzdata2022d.tar.gz.done
unifdef-2.12.tar.xz
unifdef-2.12.tar.xz.done
...
uninative
git2
```

- tar.gz & tar.xz 파일들은 소스가 전부 다운로드 되었다는 표시로 .done 파일이 생성됨
- uninative : C 라이브러인 glibc, 호스트의 glibc 버전이 아닌 Yocto 프로젝트에서 배포하는 glibc 버전으로 컴파일된다는 의미
- git2 : git을 통해 소스를 받는 경우 git2 디렉터리에 받은 소스가 저장됨
  - git2 디렉터리 내 파일들도 다운로드 되면 .done 파일이 생성됨

> 빌드 과정에서 `.done` 파일이 확인되면, 레시피에서 지정한 `S` 변수가 가리키는 위치로 받은 소스가 복사됨

#### do_fetch 태스크가 소스를 가져오는 과정

1. `SRC_URI` 변수에 정의된 소스가 로컬에 다운로드 되어있는지 확인한다. 이때 확인하는 디렉터리가 `DL_DIR` 변수(기본 : `build/downloads`)에 저장된 경로이다.
2. `DL_DIR` 디렉터리에서 찾았는데 없다면 `PREMIRRORS` 변수에 지정된 자체 mirror 또는 mirror site로부터 다시 다운로드 받으려고 시도한다.
3. `PREMIRRORS`에서도 다운로드 받을 수 없다면 업스트림(upstream) 소스, 즉 `SRC_URI`에 정의된 곳으로터부터 다운로드를 시도한다.
4. 업스트림 소스에서도 다운로드 받을 수 없다면 `MIRRORS` 변수에 기술된 mirror 사이트에서 다시 다운로드가 가능한지 확인한다.
5. `MIRRORS`에서도 다운받을 수 없다면 bitbake는 결국 오류를 출력한다.

> 소스 다운로드에 성공했다면, 다음 단계로 do_unpack 태스크에서 `S` 변수가 가리키는 디렉터리로 받은 소스를 복사한다.
> `S` 변수는 압축 해제, 패치, 컴파일이 진행되는 디렉터리임

## 2. 자신만의 소스 저장소 PREMIRRORS 구성

- 빌드에 필요한 소스들을 저장하는 저장소를 만든다면 팀 프로젝트의 경우, 팀원들 각각이 외부 네트워크에 접근하는 것을 최소화할 수 있음

```bash
# 빌드를 생략하고 소스만 받는 명령어
bitbake core-image-minimal --runall=fetch
```

- `--runall==<task>`는 bitbake가 레시피를 수행하면서 `do_<task>` 태스까까지만 수행한다는 뜻임

#### BB_GENERATE_MIRROR_TARBALLS

```conf
# poky_src/build/conf/local.conf
# compress tarballs for mirrors
BB_GENERATE_MIRROR_TARBALLS = "1"
```

- git에서 받은 파일들은 git2 디렉토리에 들어가는데, 이렇게하면 git2 디렉터리에 들어가지 않고 downloads 디렉터리에 `tar.gz`로 압축되어 저장됨
- 이를 테스트 해보려면, 앞선 빌드 과정에서 소스패치기 이미 되어있는 상태이기 때문에, 지우고 강제로 다시 fetch를 진햐해야함

```bash
rm -rf downloads
bitbake core-image-minimal --runall=fetch -f
```

- 위와 같이 진행하면 이제 downloads/git2에 저장되지 않고 downloads 디렉터리에 `.tar.gz`로 압축되어 저장됨

#### 자체 소스 미러 구축

```bash
cd poky_src

# downloads 디렉터리에서 `.done` 파일을 모두 삭제
rm -rf build/downloads/*.done

# git2 디렉터리 삭제
rm -rf build/downloads/git2

# source-mirrors 디렉터리 생성
mkdir source-mirrors

# downloads 디렉터리 내 파일을 source-mirrors로 복사
cp -r build/downloads/* source-mirrors/
```

build/conf/local.conf 파일 수정

```conf
# Specify own PREMIRRORS location

# own-mirrors 클래스 파일을 INHERIT 변수에 할당
# INHERIT 변수는 상속받고자 하는 클래스 파일의 이름을 넣는 변수
# 이 변수에 할당된 클래스 파일은 모든 레시피에서 전역적으로 상속됨
INHERIT += "own-mirrors"

# SOURCE_MIRROR_URL은 own-mirrors.bbclass 클래스 파일에서 선언된 변수
# 이 변수 자체는 PREMIRRORS의 디렉터리 경로를 할당 받음
# 자체 PREMIRRORS는 앞에서 생성한 source-mirrors 디렉터리이기 때문에 이 절대 경로를 file:// 프로토콜과 함께 할당
# ${COREBASE} 변수는 meta 디렉터리의 부모 디렉터리 경로(poky 디렉터리의 절대 경로)가 담겨있음
SOURCE_MIRROR_URL = "file://${COREBASE}/../source-mirrors"
```

```bash
# PREMIRRORS가 정상 동작하는지 확인
rm -rf build/downloads
bitbake core-image-minimal -f --runall=fetch
```

위와 같이 실행하면 로컬에 있는 PREMIRRORS를 사용하기 때문에 매우 빠르게 소스 fetch가 끝남

> 참고 : GitHub나 bitbucket 같은 깃 저장소 호스팅을 지원하는 서비스가 사내에 존재하면 PREMIRRORS를 깃의 리포지터리로 만들어 공유

## 3. 자신만의 공유 상태 캐시 (Shared State Cache) 생성

> 공유 상태 캐시 == 빌드 시간을 절감할 수 있는 방법

- bitbake는 레시피의 각 태스크 수행 시 signature 값을 만듦 (어떤 책에서는 signature를 checksum이라고도 표현함)
- 이 signature와 함께 빌드의 결과를 object 형태인 공유 상태 캐시를 만들어 특정 디렉터리에 저장함
- 이미 빌드 과정에서 실행이 완료된 태스크를 다시 수행하려고 할 때, bitbake는 앞서 계산된 signature 값과 현재 signature 값을 비교함
  - 값이 동일하거나 이미 수행된 object가 존재하고 재사용이 가능하다고 판단되면 해당 태스크를 건너뜀
  - 기존과 대비해 입력이 변경되지 않으면 signature값이 변경되지 않기 때문임
  - signature 값이 동일하다는 것은 이미 수행돼 공유 상태 캐시에 저장된 태스크의 결과를 그대로 사용해도 된다는 뜻
  - signature 값이 동일해 공유 상태 캐시에서 수행된 태스크의 결과를 그대로 가져오는 태스크를 setscene 태스크라고 함
  - setscene 태스크는 `do_<taskname>_setscene`과 같은 형식의 이름을 가짐
- bitbake가 특정 레시피를 빌드(fetch 부터 install까지) 하기 전 공유 상태 캐시를 먼저 확인해 재활용이 가능한 태스크가 존재하면 setscene 태스크를 실행해 결과를 가져옴

> 그렇다고 모든 태스크가 setscene 태스크를 갖고 있지는 않음

#### build/sstate-cache

- build/sstate-cache 디렉터리는 공유 상태 캐시가 저장되는 곳임
- 이 경로를 지정하는 변수는 `SSTATE_DIR` 임

```bash
# SSTATE_DIR 확인
bitbake-getvar -r core-image-minimal SSTATE_DIR
```

#### 테스트

```bash
rm build/tmp
bitbake core-image-minimal
```

- tmp 디렉터리는 오픈임베디드 빌드 시스템이 빌드 결과물을 저장하는 디렉터리
  - 참고 : `TMPDIR` 변수로 지정
- tmp 디렉터리를 지운 상태에서 빌드를 진행해보면, 기존에 수행한 레시피들의 태스크들은 실행하지 않기 때문에 빌드가 금방 끝남
- 중간 중간에 `xxx_setscene` 태스크가 실행되는 것을 볼 수 있음
- `Sstate summary: xxx(94% match xx)`는 sstate-cache 디렉터리를 확인해보니 빌드 과정에서 수행하려는 레시피들의 태스크 시그니처 값이 94% 일치한다는 것을 의미함

#### 자체 공유 상태 캐시 저장소 구축

- sstate-cache를 로컬이 아닌 외부 서버에도 구축 가능하고, 그럼 팀원들의 빌드 시간을 줄일 수 있음
- 자세한 내용은 참조
- 공유 상태 캐시를 깊게 이해하기 위해서는 좀 더 많은 기반 지식들이 필요함
- 일단, 공유 상태 캐시를 사용하면 빌드 시간을 줄일 수 있는 정도로만 알고 넘어가자

## 4. 요약

- 오픈임제디드 빌드 시스템은 빌드에 필요한 소스를 빌드할 때마다 실시간으로 받아옴
- 각 소스들은 웹사이트, 파일 서버 등 다양한 경로에서 받아올 수 있음
- 그러기 위해서는 http, https, ftp, sftp 등 다양한 프로토콜의 지원이 필요함
- 어떤 소프트웨어 패키지들은 git과 같은 SCM(버전 관리 시스템)을 통해 소프트웨어를 제공받아야 함
- 오픈임베디드 빌드 시스템은 이런 모든 것들을 자동화해 제공하는 빌드 시스템임
- 리눅스 소프트웨어 스택을 구성하고 있는 많은 소프트웨어를 fetch 하는 데는 많은 시간이 소모됨
- `PREMIRRORS`를 구성하면 소스 fetch에 걸리는 시간을 줄일 수 있음
- 받은 소프트웨어들을 빌드하는 데도 시간이 많이 소모됨
- 우리만의 공유 상태 캐시 저장소를 만들어 팀원들의 빌드 시간을 단축시킬 수 있음
