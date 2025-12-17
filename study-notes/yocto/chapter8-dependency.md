# Chapter 8. 의존성

- 소프트웨어 패키지들은 제대로 빌드되고 실행되기 위해서 다른 패키지나 라이브러리의 지원이 필요함

## 1. 의존성의 종류

- Yocto에서 사용하는 의존성에는 **"빌드 의존성"**과 **"실행시간 의존성"**이 있음
- 실행시간 의존성 : bitbake가 타깃 시스템에 설치될 모든 패키지들이 제대로 실행될 수 있도록 패키지들 간의 의존 관계를 패키지 설치시에 확인함
  - 어떤 패키지가 아직 설치되지 않은 다른 패키지를 필요로 한다면 bitbkae는 의존성에 따라 사전 설치돼야 하느 패키지를 먼저 설치함
- 빌드 의존성 : 소프트웨어 패키지가 빌드될 때 특정 라이브러리를 사용해야한다면 bitbkae가 사전 설치돼야 하는 라이브러리를 생성하는 패키지를 먼저 빌드함

#### 빌드 의존성

- 소프트웨어 패키지가 성공적으로 빌드되기 위해 다른 패키지의 헤더 파일, 정적 라이브러리 등이 필요한 경우가 있음
- 빌드 의존성은 패키지를 빌드하기 위해 필요한 헤더 파일과 정적 라이브러리를 만들어내는 다른 레시피에 대한 의존성을 의미함
- 빌드 의존성이 필요한 레시피는 `DEPENDS`라는 변수에 의존성을 제공하는 레시피의 이름 또는 의존성을 제공하는 레시피의 `PROVIDES` 변수의 값을 할당하면 됨

nano editor 예제 중 DEPENDS 부분

```conf
DEPENDS = "ncurses"
```

- 위 내용은 bitbake 내부적으로 nano editor 레시피 파일의 `do_prepare_recipe_sysroot` 태스크가 ncurses 레시피의 `do_populate_sysroot` 태스크에 의존하고 인식함

변수 플래그를 통한 빌드 시간 의존성

```conf
do_prepare_recipe_sysroot[deptask] = "do_populate_sysroot"
```

- 변수 플래그 개념은 아직 배우진 않았지만, `do_prepare_recipe_sysroot` 태스크가 `do_populate_sysroot` 태스크에 의존성을 가지고 있다는 표현임
- 즉, `do_prepare_recipe_sysroot` 태스크가 실행되기 전에 의존성이 있는 레시피의 `do_populate_sysroot` 태스크가 완료 되어야한다는 뜻임

#### do_prepare_recipe_sysroot 태스크

- log.taskorder 파일에 보면 `do_prepare_recipe_sysroot` 태스크를 볼 수 있음
- 이 태스크가 실행되면 nano/6.04-r0 디렉터리 아래 `recipe-sysroot`와 `recipe-sysroot-native` 디렉터리가 생성되도록 되어있음
- 여기서 sysroot 디렉터리란 헤더와 라이브러리를 찾기 위한 루트 디렉터리로 간주되는 디렉터리임.
- 레시피 하나당 하나의 sysroot를 가짐
- recipe-sysroot
  - 타깃 시스템에서 사용하는 헤더, 라이브러리들이 포함
  - `STAGING_DIR_TARGET` 변수에 할당되어있음
- recipe-sysroot-native
  - 빌드 도구들(크로스 컴파일에서 사용되는 컴파일러, 링커, 빌드 스크립트 등)이 포함
  - `STAGING_DIR_NATIVE` 변수에 할당 되어있음

#### sysroot-providers 디렉터리

- recipe-sysroot 아래 sysroot-providers 디리터리가 존재함
- 의존성을 다룰 때 꼭 알아야 하는 디렉터리임
- `DEPENDS` 변수에 할당된 ncurses 이름을 가진 텍스트 파일을 볼 수 있고, DEPENDS 변수에 할당되진 않았지만 빌드 의존성을 가진 패키지(gcc-runtime, glibc, libgcc 등)을 볼 수 있음
- ncurses 패키지는 ncurses 레시피 빌드가 완료돼야 생성됨
- `do_populate_sysroot` 태스크는 `do_install` 태스크 실행 후 설치된 파일들을 의존성을 가진 다른 레시피가 사용할 수 있도록 sysroot 디렉터리에 복사함

#### 정리

- nano editor 빌드 진행시 `do_prepare_recipe_sysroot` 태스크를 실행하기 전, ncurses 레시비 빌드 진행
- ncurses 레시피의 빌드 진행중 `do_popluate_sysroot` 태스크를 실행하면 sysroot 디렉터리에 빌드 결과물들이 생성됨
- 이 겳과불들은 다시 nano 레시피의 sysroot 디렉터리로 복사
- 이후 nano 레시피의 나머지 태스크가 실행되고 최종 이미지가 생성됨

> 그럼 8-4 보면 더 명확하게 정리되어 있음

#### 실행 시간 의존성

- 실행 시간 의존성은 소프트웨어 패키지가 정상적으로 작동하기 위해 다른 패키지가 필요한 경우를 나타냄
- 빌드하는 과정과는 별개로, 소프트웨어 실행되는 동안 필요로 하는 외부 패키지들과 의존성을 의미함
- 이 의존성은 소프트웨어 패키지를 실행하는 시점에서 라이브러리, 모듈, 실행 파일등을 포함함
- 실행 시간 의존성이 필요한 레시피는 `RDEPENDS_${PN}`이라는 변수에 의존성을 제공하는 패키지의 이름을 넣거나, 의존성을 제공하는 패키지 레시피의 `RPROVIDES` 변수의 값을 할당하면 됨
  - 주의 점은 `RDPENSDS`는 `RDEPENDS_${PN}`으로 표현해야 하며, 레시피의 이름이 아닌 패키지의 이름이라는 것임
- 아직 패키지의 개념을 배우지 않았기 떄문에, 패키지 == 레시피 빌드의 결과물 정도로 알아두자
- bitbkae는 실행 시간 의존성을 표현하는데 `RDEPENDS`와 `RRECOMMENDS` 변수를 사용함
- `RDEPENDS_${PN} = <package name>`은 bitbkae 내부적으로 `do_build[rdeptask] = "do_package_write_xxx"`로 표현됨
  - `do_build` 태스크가 `RDEPENDS_${PN}` 변수에 할당된 패키지를 생성하는 레시피의 `do_package_write_xxx` 태스크에 의존성을 갖고 있다는 것을 의미
  - 즉, `do_build` 태스크가 실행 완료되기 전 실행 시간 의존성을 갖고 있는 각각의 패키지들을 생성하는 레시피의 `do_package_write_xxx` 태스크가 완료되어야 함
- `do_build` 태스크는 태스크 체인 중 가장 마지막에 위치한 태스크임. 실제 내용은 없을 수 있고, 태스크 체인 구성을 위해 존재하는 태스크임

#### 패키징과 태스크

- 아직 배우지는 않았지만, `RDEPENDS` 변수는 패키징 단계에서 사용됨
- 일반 레시피와 패키징 레시피는 다른 태스크들을 가짐
  - 일반 레시피's task들 : do_configure, do_compile, do_install등
  - 패키징 레시피's task들 : do_rootfs, do_image, do_image_complete등
- 패키징을 수행하는 레시피는 파일 시스템 이미지를 생성하는 core-image-minimal.bb와 같은 레시피를 의한함
- 패키징 레시피's task의 주요 복적은 **루트 파일 시스템**을 생성하는 것이고, 이 태스크 체인 끝에 `do_build` 태스크가 존재함
- 결론적으로 `do_build` 태스크와 실행 시간 의존성을 가졌다는 것 == 실행 시간 의존성을 필요로 하는 레시피의 빌드와 상관없이 최종 루트 파일 시스템 이미지가 생성되기 전, 즉, `do_rootfs` 태스크 실행 이전에 의존 관계에 있는 레시피의 `do_package_write_xxx`가 실행돼야 한다는 것을 뜻함.
- 쉽게 말하면 최종 루트 파일 시스템 이미지가 생성되기 전에 의존 관계에 있는 패키지가 생성돼 루트 파일 시스템에 추가돼야 한다는 것임

#### 대체 패키지 생성

- 오픈임베디드 코어에는 kdb라는 패키지가 있음.
- 과거에는 console-tools 패키지를 사용했으나, 현재는 kdb 패키지가 대체함
- 이처럼 패키지가 대체되고 패키지의 이름이 바꾸는 경우 아래와 같이 처리함
  - console-tools 패키지와 실행 시간 의존성을 가진 다른 패키지의 레시피 파일은 건드리지 않는다.
  - `PPROVIDES` : 패키지 이름의 별칭을 만든다
  - `RCONFLICTS` : 현재 패키지가 설치될 때 충돌하는 것으로 알려진 패키지
  - `RREPLACES` : 대체되기 전의 패키지 이름을 넣어 패키지가 대체되었다는 것을 알려줌

poky/meta/recipes-core/kdb/kdb_2.2.0.bb

```conf
# 대체되기 전 패키지 이름을 넣어 패키지가 대체되었음을 알려줌
RREPLACE_${PN} = "console-tools"

# 별칭을 기존과 동일하게 유지하여, 다른 레시피에서 기존에 실행시간 의존성을 위해 사용한 패키지 이름을 수정하지 않아도 되게 함
RPROVIDES_${PN} = "console-tools"

# kdb 패키지가 console-tools를 대체했기 때문에 충돌이 일어나지 않도록 설정
RCONFLICTS_${PN} = "console-tools"
```

## 2. 의존성을 제공하는 레시피의 PROVIDES 변수

- 레시피 파일은 `PROVIDES`라는 변수에 별칭과 같은 이름을 할당해 그 이름을 다른 레시피에 알려줌
- 어떤 레시피에 대한 의존성 표시를 위해서 해당 레시피의 `PROVIDES` 이름을 `DEPENDS` 변수에 넣으면 됨
- bitbake는 두가지 종류의 `PROVIDES` 변수 표현 방식을 제공함
  1. 레시피 파일 이름으로부터 PROVIDES 변수의 값 할당 받기
  - 패키지 관련 변수인 PN(Package name), PV(Package version), PR(Package revision)은 따로 지정하지 않는 경우 레시피 파일로 부터 얻어냄
  - `PROVIDES` 변수도 따로 레시피 파일에 정의하지 않으면 `PN` 변수 값을 따라가도록 되어 있음
  2. 명시적으로 `PROVIDES` 변수 값 할당
  - `PROVIDES =+ "nano_alias"` 와 같이 명시적으로 추가할 수 있음

#### 다중 패키지, 다중 버전을 위힌 virtual PROVIDER

- 여러 레시피에서 `virtual/xxx` 동일한 값으로 `PROVIDES` 변수의 값을 설정하고,
- 이를 사용할 레시피에서 `PREFERRED_PROVIDER_virtual/xxx = "실제 레시피명"` 형태로 고를 수 있음
- 다중 버전을 가진 패키지의 경우 `PREFERRED_VERSION_레시피명 = "버전명"` 형태로 고를 수 있음

## 3. 요약

- 오픈임베디드 빌드 시스템은 레시피들 간 또는 패키지들 간에 의존성을 지정할 수 있음
- bitbake가 시작되면 의존성을 만족하는 순서로 레시피나 패키지들을 생성함
- 의존성은 빌드 의존성과 실행 시간 의존성으로 분류됨
- 빌드 의존성은 빌드 중에 필요한 의존성으로 `DEPENDS` 변수를 통해 빌드 의존성을 나타냄
- 실행 시간 의존성은 실제 패키지가 동작시에 필요한 의존성으로 `RDEPENDS` 변수를 통해 실행 시간 의존성을 나탄냄
  - 특정 패키지가 동작하려면 다른 패키지가 앞서 설치되어야 하는 것도 실행 시간 의존성 예 중 하나임
- 특정한 동일 기능을 하는 레시피가 서로 다른 버전이나 다른 이름으로 존재할 수 있음
  - 이런 경우 각 레시피는 `PROVIDES` 변수에 `virtual/xxx` 형태로 동일한 이름을 할당함
  - 그리고 빌드시 `PREFERRED_PROVIDER` 변수를 통해 실제 빌드돼야 하는 레시피를 지정함
  - 서로 다른 버번의 레시피가 존재하는 경우 `PREFERRED_VERSION` 변수를 통해 실제 빌드가 돼야하는 레시피를 지정함
