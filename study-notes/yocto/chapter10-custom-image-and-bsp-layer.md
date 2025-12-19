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

## 2. BSP 레이어

## 3. bitbake 문법 세 번째

## 4. 커스텀 BSP 레이어 만들기

## 5. 요약
