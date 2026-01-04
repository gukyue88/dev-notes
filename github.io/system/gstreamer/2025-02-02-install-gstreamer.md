---
layout: single
title: "[GStreamer] 설치 (on Windows)"
categories: system/gstreamer
---

- [installing on windows](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html?gi-language=c)
- [download](https://gstreamer.freedesktop.org/download/#windows)

## GStreamer 바이너리 다운로드 및 설치

GStreamer 바이너리에는 3가지 종류의 파일 세트가 존재함

1. 런타임 파일(runtime installer) : gstreamer 앱 구동만 필요한 경우

- GStreamer 앱을 실행하는데 필요함
- 대부분의 경우 이 파일들을 애플리케이션과 함께 배포하거나, 아래에 나오는 설치 프로그램을 이용해 배포함

2. 개발 파일(development installer) : 개발환경에 설정하여 gstreamer 관련 소스코드 작성할 때

- GStreamer 앱을 빌드할 때, 필요한 추가 파일들
- 헤더 파일, 라이브러리(.lib/.a)등, 개발 환경에 포함시켜야 함

3. 머지 모듈 파일 : MSI 설치 파일에 GStreamer를 포함하고 싶을 때만

- windows 환경에서 설치 프로그램(.msi)와 함께 GStreamer 바이너리를 배포하는 경우에 사용
- 일반적인 배포 방식이 아니라, MSI 설치 파일에 직접 포함시키고 싶을 때 선택적으로 사용함

> 대부분의 경우 1번과 2번을 모두 받아 설치하면 됨

#### 환경변수 설정

- 설정 > 시스템 > 고급 시스템 설정 > 환경 변수 > 사용자 변수(또는 시스템 변수) > Path > 편집 > 찾아보기
- gstreamer 설치 경로 안에 bin 폴더 (예. `{설치경로}/gstreamer/1.0/msvc_x86)64/bin`)
  - 이 bin 폴더 안에 dll들과 앱 (예. gst-launch-1.0.exe, gst-inspect-1.0.exe 등드이 들어있음

```bash
gst-launch-1.0 --version
```
