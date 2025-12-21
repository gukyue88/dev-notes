# Yocto 주요 개념

## Yocto란?

- 특정 하드웨어(보드)에 딱 맞는 최적화된 리눅스 배포판을 직접 빌드하기 위한 툴, 프로세스, 템플릿 및 개발 방법을 제공하는 프로젝트

## 범용 리눅스 vs Yocto 기반 리눅스

- 범용 리눅스(Ubuntu, Fedora 등)
  - 타겟 : 불특정 다수 하드웨어(테스크탑, 서버등)
  - 혹시 몰라 많은 드라이버, 서비스, 라이브러리를 포함함
  - 상대적으로 무겁고, 부팅속도가 느림
- Yocto 기반 리눅스
  - 타겟 : 특정 하드웨어(임베디드 기기)
  - 해당 보드에 필요한 드라이버, 서비스, 라이브러리만 포함
  - 가볍고, 부팅속도가 빠름

## Yocto 프로젝트는 단일 프로젝트가 아니다

> Yocto 프로젝트에는 단일 프로젝트가 아니고 안에 다양한 프로젝트들이 존재함

#### Yocto 프로젝트 내 주요 프로젝트

- bitbake : 레시피를 읽어서 소스코드를 다운로드하고 컴파일 하는 실행 엔진 (실제 요리를 하는 요리사 역할)
  - 파이썬 및 셀 스크립트 혼합 코드를 분석하는 작업 스케쥴러 + 크로스 컴파일을 위한 패키지 및 관련 파일을 빌드하는데 사용되는 툴
  - Poky에서 사용되지만, 단독으로도 사용 가능함
- metadata : 어떤 소스를 어디서 가져오고, 어떤 의존성을 가지며, 어떻게 빌드해라 라는 지시서 (요리법이 적힌 레시피 역할)
  - 크게 '변수' 와 '태스크'로 두 종류로 나뉨
- Poky : bitbake와 샘플용 metadata를 제공하는 프로젝트

#### metadata 파일

metadata 파일이란 metadata(변수, 태스크)를 기술하는 파일이며, 5 종류의 파일이 있음

- 환경 설정 파일(.conf) : 변수만 존재, 여기에 기술된 변수는 전역 속성을 가짐, (bitbake의 변수는 시스템 환경 변수와는 별개로 운영됨)
- 레시피 파일(.bb) : 빌드 전반에 걸친 내용(소프트웨어 출처, 빌드 방법, 빌드 이미지 출력 path 등) 기술
- 클래스 파일(.bbclass) : 빌드에 사용되는 기능을 추상화 해놓은 파일, 레시피에서 `inherit`를 사용해 클래스 파일을 상속 받아 사용
- 레시피 확장 파일(.bbappend) : 레시피 파일에 선언된 metadata를 재정의
- 인클루드 파일(.inc) : 클래스 파일과 같은 동작, 단, 인클루드 파일은 비공식적인 내용을 공유할 때 사용

> bitbake가 metadata 파일 안의 metadata를 가지고 이미지(루트 파일 시스템 이미지, 커널 이미지, 부트로더 이미지지를 만들어 냄

## 레이어

- Yocto 프로젝트에서 기능을 **모듈화** 해서 관리하는 단위
- 그림이 그려진 투명한 비닐(layer)을 겹쳐서, 완성된 그림(리눅스 이미지)를 만드는 개념
  - 모듈화 뿐만 아니라 계층적 구조를 통한 overlay를 통해 원본은 보존하고, 수정사항은 layer로 관리하는 방식을 사용
- 독립성, 재사용성이 높아짐
- 보통 레이어의 이름은 `meta-`로 시작함

> 같은 레시피가 여러 레이어에 있다면, 레이어 우선순위에 따라 높은 순위의 레이어가 낮은 순위의 레이어를 덮어씀

#### 관습적인 레이어의 분류

- base layer : Yocto 프로젝트에서 가장 기본적인 빌드 규칙과 핵심 소프트웨어를 포함 (예. meta, meta-poky 등)
- bsp layer : 특정 보드를 동작하기 위한 커널과 드라이버 설정을 포함, 하드웨어 제조사가 제공 (예. meta-raspberrypi, meta-intel, meta-imx 등)
- software/feature layer : 특정 프로그램이나 프레임워크에 대한 레이어, (예. meta-python, meta-qt5, meta-ros 등)
- custom layer : 사용자가 프로젝트 고유의 설정이나 앱을 위해 직접 추가하는 레이어 (예. meta-my-project)

## Datastore

- bitbake가 실행되는 동안, 모든 변수는 Datastore라는 가상의 공간에 저장됨
- bitbake는 Datastore의 변수를 업데이트 해가면서 실행됨
  1. 초기화 : 시스템 환경 변수 중 일부 (예. BBPATH 등)을 가져와 기반 변수를 설정함
  2. 구성 파일 파싱: bitbake.conf, base.bbclass, local.conf 등을 읽으며 Datastore에 전역 변수들을 채움
  3. 레시피 파싱 : 특정 소프트웨어를 빌드할 때, 해당 .bb 파일의 내용을 읽어 Datastore의 내용을 해당 레시피 전용으로 확장함
- Datastore는 단순한 key, value 쌍은 아님
  - 계층적 구조 : 레시피에서 설정한 값들이 층층이 쌓여있는 형태로, 그 중 유효한 최종값을 결정함
  - 파이썬 기반 : 파이썬 객체로 구현되어있고, 레시피 안의 파이썬 코드에서 `d.getVar({name})` 형태로 접근 가능

## bitbake 파싱 순서

```
# 시스템 환경 변수
export BBPATH=/home/gukyue88/bitbake_test

# /home/gukyue88/bitbake_test
- bitbake_test
  - conf
    - bblayers.conf : BBLAYERS(layer들의 경로, 여기에서는 mylayer 경로가 추가)
    - bitbake.conf : PN, TMPDIR, CACHE, T, B등 정의
  - mylayer : mylayer(custom layer) 용 디렉토리
    - conf
      - layer.conf : BBFILES += "${LAYERDIR}/*.bb", BBPATH .= ":${LAYERDIR}"
    - hello.bb : mylayer의 레시피 파일, PN, PV, DESCRIPTION, do_build 태스크 정의
  - classes
    - base.bbclass : `addtask do_build`
```

#### 1. 환경 설정 파일 분석

1. bitbake가 시스템 환경 변수 중 일부 (예. `BBPATH`등)를 가져와 Datastore를 초기화함. (앞으로 과정에서도 계속해서 Datastore를 업데이트 함)
2. `BBPATH`에 저장된 경로들 아래의 conf 디렉터리에서 'bblayers.conf' 환경 설정 파일을 검색함
3. 'bblayers.conf' 파일의 `BBLAYERS` 변수로 부터 현재 어떤 레이어들이 존재하는지 파악함. (여기에서는 mylayer 존재 파악)
4. 위에서 파악한 레이어(여기에서는 mylayer)에 가서 'conf/layer.conf' 파일을 찾아 분석함
5. layer.conf의 `BBFILES`로 해당 레이어의 레시피 파일 위치 확인 + `BBPATH`에 현재 레이어 디렉터리도 추가한 것을 반영
6. `BBPATH` 경로의 conf 디렉터리 내 bitbake.conf 파일 확인 및 분석

#### 2. 클래스 파일 및 레시피 파일 분석

1. `BBPATH` 경로의 classes 디렉터리에서 base.bbclass를 포함한 클래스 파일 (.bbclass) 확인 및 분석
2. 분석된 환경 설정 파일 및 클래스 파일을 기반으로 해당 레이어의 레시피 파일 (여기에서는 hello.bb)를 분석

> 분석 과정에서 클래스 파일에서 정의된 태스크와 레시피 파일에서 정의된 태스크는 태스크 체인으로 형성돼 실행순서가 정해짐

## Poky
