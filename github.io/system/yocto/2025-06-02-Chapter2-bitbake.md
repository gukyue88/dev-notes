# bitbake

## bitbake란?

- 문서 정의 : 임베디드 리눅스 크로스 컴파일을 위한 배포와 패키지에 초점을 맞춘 make와 유사한 빌드 툴
- 파이썬 및 쉘 스크립트 혼합 코드를 분석하는 작업 스케쥴러, 임베디드 리눅스의 크로스 컴파일을 위한 패키지 및 관련 파일을 빌드하는데 사용되는 툴이라고 문서에 정의됨
- 일단, C나 C++ 빌드를 위해 Windows에서 사용하는 Visual Studio와 같은 툴과 같은 빌드 도구 정도로 이해하고 넘어가자.
- poky 안에 포함된 빌드 도구
- Yocto 버전과 일치하는 bitbake 소스가 필요
- bitbake 소스 다운로드하면, bin 디렉토리 내에 bitbake 실행 파일이 존재함을 확인할 수 있음

## 메타데이터

- bitbake는 빌드 툴로 빌드 수행을 위해 메타데이터를 사용함
- 메타데이터는 소프트웨어를 어떻게 빌드할지, 빌드하려는 소프트웨어들 간에 어떤 의존성이 있는지를 기술함
- 메타데이터에는 크게 `변수` + `실행 가능함 함수(쉘 함수 또는 파이썬 함수) 또는 태스크`로 2가지 종류가 있음
- 메타데이터 파일에는 다섯 종류에 파일이 있음
- 실제 개념은 다르지만 `메타데이터 파일`을 `메타데이터`라고 부르는 경우가 많고, 이 책에서도 그렇게 부를꺼임

#### 메타데이터 파일과 파일 내 포함된 metadata

```
- 환경설정 파일(.conf) : metadata 변수
- 레시피 파일(.bb) : metadata 변수 + metadata 함수/task
- 클래스 파일(.bbclass) : metadata 변수 + metadata 함수/task
- 레시피 확장 파일(.bbappend) : metadata 변수 + metadata 함수/task
- 인클루드 파일(.inc) : metadata 변수 + metadata 함수/task
```

##### 메타데이터 파일과 bitbake

```
메타 데이터 파일들 -> bitbake -> 루트 파일 시스템 이미지 / 커널 이미지 / 부트로더 이미지
```

#### 메타 데이터 파일 상세

- 환경 설정 파일 (.conf)
  - 변수들의 집합
  - 여기에 선언된 변수는 전역 변수의 특징을 가짐 (다른 메타데이터 파일에서 사용할 수 있음)
- 레시피 파일 (.bb)
  - 빌드 내용 기술 (소프트웨어를 어디에서 다운로드 받을지, 받은 소프트웨어를 어떻게 빌드할지, 빌드된 산출물이 어디에 위치해야하는지 등)
  - bitbake가 실행할 수 있는 함수들이 정의될 수 있음 (Yocto에서는 이러한 함수를 태스크라고 부름)
- 클래스 파일 (.bbclass)
  - 빌드를 위해 사용되는 기능들을 추상화해놓은 파일
  - 레시피 파일들이 `inherit`라는 지시자를 사용해 클래스 파일을 상속받아 클래스 파일의 기능을 사용함
  - 여기에 선언된 변수, 함수는 해당 클래스 파일을 상속한 레시피에서만 사용 가능
- 레시피 확장 파일 (.bbappend)
  - 레시피 파일에서 선언된 변수나 태스크를 재정의
  - 이 파일은 비슷한 기능을 하는 메타데이터끼리 모아 놓은 레이어라는 개념을 알아야 설명 가능
- 인클루드 파일 (.inc)
  - 클래스 파일처럼 메타데이터를 다른 메타데이터 파일과 공유할 수 있도록 해주는 파일
  - 클래스 파일은 공식적인 기능을 제공하는 반면, 인클루드 파일은 비공식적인 내용을 공유할 때 사용함

## bitbake 문법 첫 번째

- bitbake는 변수에 대한 타입이 없음
- bitbake는 변수 이름에 크게 제약을 두지 않으나, **모든 변수 이름을 대문자로 시작**하도록 함
- bitbake는 모든 변수를 문자열로 인식하기 때문에, 변수에 값을 할당할 때에는 큰 따옴표나 작은 따옴표를 사용해 값을 할당함 (`BITBAKE_VAR = "value"`)
- 메타데이터 파일 내 주석은 `#`을 사용하고, 사용하지 않는 한 줄의 맨 앞에서 시작해야 함

## bitbake 환경설정

- 편의를 위해 `$PATH`에 bitbake 실행 파일이 들어 있는 `bin` 디렉토리를 추가
- 참고로 PATH 변수에 기술된 각각의 경로들은 앞에 위치한 경로가 뒤에 위치한 경로보다 우선순위가 높다

```bash
bitbake --version
```

## bitbake로 "Hello! bitbake world!" 출력

```bash
cd /home/gukyue88
mkdir bitbake_test
export BBPATH=/home/gukyue88/bitbake_test/ # bitbake가 bitbake_test 디렉토리
```

- `BBPATH` : bitbake 실행파일은 `BBPATH` 변수에 저장된 경로들 아래의 `conf 디렉토리`에서 `.conf` 파일들을 찾고, `classes 디렉토리`에서 `.bbclass` 파일들을 찾음

#### bitbake_test 폴더 구조

- bitbake_test
  - conf
    - bblayers.conf
    - bitbake.conf
  - mylayer
    - conf
      - layer.conf
    - hello.bb
  - classes
    - base.bbclass

> hello.bb는 bitbake가 수행하길 원하는 태스크가 있는 파일이고, 나머지는 bitbake가 동작하는 데 필요한 최소한의 파일들

#### bblayers.conf

```conf
BBLAYERS ?= " \
    /home/gukyue88/bitbake_test/mylayer \
"
```

- bitbake는 수행되면 제일 먼저 BBPATH의 conf 디렉토리의 `bblayers.conf`를 확인함
- `bblayers.conf`는 bitbake에게 어떤 레이어(연관된 메타데이터들을 포함하는 저장소(디렉토리)리가 존재하는지 알려줌
- `BBLAYERS` 변수가 메타데이터들이 위치하고 있는 디렉토리 경로를 저장

#### layer.conf

```conf
BBPATH .= ":${LAYERDIR}"
BBFILES += "${LAYERDIR}/*.bb"
BBFILE_COLLECTIONS += "mylayer"
BBFILE_PATTERN_mylayer := "^${LAYERDIR}/"
```

- bblayers.conf 파일로부터 알아낸 레이어들의 경로 정보를 기반으로 각 레이어 아래 conf 디렉토리 안의 `layer.conf` 파일을 확인
- 모든 레이어는 conf 디렉토리 내 `layer.conf` 파일을 가짐
- `layer.conf` 파일은 bitbake에게 현재 레이어의 경로와 레이어 내에 있는 레시피 파일들(.bb, .bbappend)의 경로를 알려줌
- `BBPATH` : bitbake가 이 변수에 저장된 경로의 하위 디렉터리 classes에서 클래스 파일 (.bbclass), 하위 디렉터리 conf에서 환경설정 파일(.conf)를 찾아냄, 이 변수는 콜론으로 구분된 디렉터리 목록을 가짐
- `BBFILES` : bitbake가 이 변수를 보고 레시피 및 레시피 확장 파일들의 위치를 알아냄
- `BBFILE_COLLECTIONS` : layer.conf 파일을 포함하고 있는 레이어의 이름, 꼭 레이어 최상위 디렉터리 이름일 필요는 없음(?)
- `BBFILE_PATTERN_<layer name>` : bitbake가 실행되면서 분석을 시작할 때 각 레이어별로 분석하게 되는데, 이 변수는 존재하는 모든 레이어의 레시피 파일이 저장된 경로를 가지고 있음
  - 저장된 경로 정보들 가운데 현재 레이어에 속한 레시피 파일들만 찾아 분석하는데 사용(?)
  - 예제에서는 mylayer라는 레이어 내부에 존재하는 레시피 파일들만 찾으려고 `BBFILE_PATTERN_mylayer`라는 변수 이름을 만듦(?)
  - `^${LAYERDIR}` 기호의 의미는 최상위 디렉터리 이름으로 시작되는 문자열들을 검색 패턴으로 사용하겠다는 것

#### bitbake.conf

```conf
PN = "${bb.parse.vars-from_file(d.getVar('FILE', False),d)[0] or 'defaultpkgname'}"
TMPDIR = "${TOPDIR}/tmp"
CACHE = "${TOPDIR}/cache"
STAMP = "${TOPDIR}/${PN}/stamps"
T = "${TOPDIR}/${PN}/work"
B = "${TOPDIR}/${PN}"
```

- bitbake는 bitbake.conf 파일로부터 기본적인 환경설정을 읽어들임
- `${TOPDIR}` 변수의 값은 bitbake_test 디렉터리임, Poky가 포함된 빌드 시스템에서는 build 디렉터리를 가리킴
- `${TMPDIR}` 변수의 값은 빌드 시스템에서 만들어진 산출물들이 위치하는 디렉터리
- PN (Package name): 레시피 파일의 이름이 여기 들어감. 예제에서 `bitbake hello`라고 하면 hello라는 이름이 PN 변수에 할당됨
  - 패키지란 어떤 일을 하는 데 필요한 소프트웨어나 프로그램을 지칭하며
  - 레시피 파일이란 실행이 가능한 메타데이터가 모인 파일임
  - 일반적으로 소프트웨어 이름과 레시피 파일 이름이 동일한 경우가 많아, 패키지 이름을 레시피 파일의 이름으로 생각해도 무방함
- CACHE : bitbake는 메타데이터를 분석하고 그 결과를 cache에 기록함. 이후 메타데이터가 변경되지 않으면 bitbake는 이 캐시에서 실제적인 메타데이터를 읽음
- STAMP : bitbake는 각각의 태스크(실행 가능한 함수)가 완료되면 stamp라는 파일을 만듦. 이 파일은 bitbake가 태스크 실행이 필요한지 결정하는데 사용
- T : bitbake가 레시피를 빌드할 때, 임시적으로 생성되는 파일(태스크 실행로그, 태스크 스크립트 파일 등)을 저장하는 경로가
- B : bitbake가 레시피 빌드 과정에서 함수를 실행하는 디렉터리를 가리킴.(?)

#### base.bbclass

```
addtask do_build
```

- 앞서 본 환경 파일(.conf)들은 변수만을 가질 수 있는 메타데이터임
- 클래스 파일은 변수와 실행 가능한 함수를 모두 가질 수 있음
- addtask라는 지시어로 bitbake에 do_build 라는 함수를 추가함
- bitbake가 실행되려면 적어도 하나의 클래스 파일이 존재해야 함

#### hello.bb

```
DESCRIPTION = "hello world example"
PN = "hello"
PV = "1"
python do_build() {
    bb.warn("Hello! bitbake world!")
}
```

- bitbake가 실행하는 주체
- bitbake는 파싱 과정에서 환경설정 파일들과 클래스 파일에서 분석한 내용을 레시피 파일에 배치하고 이를 바탕으로 만들어진 레시피 파일의 태스크를 실행함
- PN : 패키지 이름
- PV : 패키지 버전
- 레시피 파일의 이름 규칙 : `<package name>_<package version>_<package_revision>.bb`

#### 실행

```bash
bitbake hello -f
```

- 레시피 파일인 hello 파일 안에 `do_build()` 라는 태스크(실행 가능한 함수)가 있고 이를 bitbake가 실행함
- 우리가 레시피 파일을 통해 bitbake에게 할 일을 준 개념임
- 함수와 태스크의 용어가 혼동될 수 있는데, 우선은 같다고 생각하자

```bash
# bitbake를 통한 레시피 파일 실행
bitbake <recipe file name>
```

- `-c <task name>`을 활용하면 특정 task를 지정할 수 있음
- bitbake의 특징 중 하나는 이미 수행한 태스크를 다시 수행하면 이전 태스크 변경 여부와 실행 이력을 체크하여 완료된 이력이 있으면 동일 태스크를 실행하지 않도록 함
- `-f`를 활용하면 완료된 동일 태스크라도 강제로 수행하게 함

## bitbake 예제 요약

#### 환경변수 설정

```
PATH : bitbake 실행 파일 경로 등록
BBPATH : 프로젝트 경로 등록
```

#### bitbake 동작

1. BBPATH 변수에 저장된 경로 아래의 conf 디렉터리에서 bblayers.conf 파일을 찾음
2. bblayers.conf에 기술된 BBLAYERS 변수로부터 현재 어떤 레이어들이 존재하는지 확인

- 각 레이어들의 conf 디렉터리를 찾으며, 찾은 conf 디렉터리 아래에서 layer.conf 파일을 찾아 분석함
- layer.conf 파일 내용 중 `BBPATH .= ":${LAYERDIR}"` 부분에서 환경변수 BBPATH에 현재 layerdir을 추가함
  - 예제에서는 `BBPATH="/home/gukyue88/bitbake_test:/home/gukyue88/bitbake_test/mylayer"` 가 됨

3. layer.conf 파일에서 `BBFILES += "${LAYERDIR}/*.bb"`를 통해 레시피 파일들이 존재하는 경로를 알아냄
4. bitbake가 BBPATH에 할당된 경로들 아래의 conf 디렉터리 아래에서 bitbake.conf 파일을 찾음

- 예제에서 bitbake.conf 파일은 bitbake가 빌드를 진행하며 만든 산출물들이 저장되는 디렉터리 설정 밖에는 없음
- 뒤에서 배울 Poky에서 배포한 bitbake.conf 파일을 보면 수많은 변수들이 선언되고 정의돼 있음

5. bitbake는 BBPATH 변수에 저장된 경로들 아래의 classes 디렉터리에서 클래스 파일(.bbclass)를 찾아 분석하고 배치함.

- base.bbclass 파일은 필수적으로 존재해야하는 파일임

6. bitbake가 분석한 메타데이터들을 기반으로 레시피 파일을 분석함.

- 이 과정에서 클래스 파일에서 정의된 태스크와 레시피 파일에서 정의된 태스크는 태스크 체인으로 형성돼 실행순서가 정해짐
- bitbake는 태스크 체인에 따라 태스크를 순서대로 실행함

## addtask 지시어를 통한 태스크 추가

```
addtask test after do_build before do_install
```

- `addtask` 라는 지시어는 태스크를 추가하는 명령어임
- `before`와 `after` 지시어를 사용하면 태스크들 간의 실행 순서를 나타낼 수 있음
- 위 task 순서는 `do_build -> test -> do_install`
- addtask 다음에 추가되는 태스크의 이름은 `do_` 접두어를 빼고 넣어도 됨

#### hello.bb 파일 예제

```
DESCRIPTION = "hello world example"
PN = "hello"
PV = "1"

python do_build() {
  bb.warn("Hello! bitbake world!")
}
addtask build

python do_preprebuild() {
  bb.warn("Add prebuild")
}
addtask preprebuild before do_build

python do_prebuild() {
  bb.warn("Add prebuild")
}
addtask prebuild after do_preprebuild before do_build
```

- `addtask build` : `do_build` 태스크 추가
- `addtask preprebuild` : `do_build` 태스크 전 `do_preprebuild` 태스크 추가
- `addtask prebuild after do_preprebuild before do_build` : `do_build` 태스크 전, `do_preprebuild` 태스크 후, `do_prebuild` 태스크 추가
- 최종 태스크 체인 : `do_preprebuild -> do_prebuild -> do_build`

## 요약

- bitbake는 파이썬과 쉘 스크립트로 만들어진 태스크 스케쥴러로 메타데이터 파일에서 정의한 태스크를 실행함
- bitbake는 빌드를 수행할 때 메타데이터를 사용함
- 이 메타데이터는 소프트웨어를 어떻게 빌드할지, 빌드하려는 소프트웨어들 간에 어떤 의존성이 있는지 기술함
- 메타데이터에는 크게 변수 + 실행이 가능한 함수(또는 태스크) 두 종류가 있음
- 메타데이터는 다섯 종류의 파일에 기술되고 이 파일들을 메타데이터 파일이라고 함
- 메타데이터 파일은 환경설정파일, 레시피 파일, 레시피 확장 파일, 클래스 파일, 인클루드 파일이 있음
- bitbake 순서
  - 환경 설정 파일 확인
    - bblayers.conf
    - 각 layer들의 conf 디렉터리 아래 layer.conf
    - bitbake.conf
  - base.bbclass
  - 레시피 파일들 (.bb, .bbappend)
- bitbake는 요리사에 비유할 수 있음
  - parsing : 요리사가 요리책을 읽고, 어떻게 요리하는지 이해, 태스크 s순서를 정함
  - cooking : bitbake가 태스크들을 정해진 순서로 실행
- 태스크는 addtask 라는 지시를 통해 추가되며, after, before 지시어로 실행 순서를 조정할 수 있음

> bitbake는 매우 강력하고 유연한 빌드 도구로 메타데이터를 읽어 분석하고 이를 바탕으로 태스크들의 스케줄링을 관리하고 실행함
