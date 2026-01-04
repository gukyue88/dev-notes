---
layout: single
title: "[windows] Visual Studio 빌드 Release vs Debug"
categories: application/windows
---

#### Visual Studio 프로비트 빌드시 Release / Debug 차이

| 내용           | debug                     | release               |
| -------------- | ------------------------- | --------------------- |
| 최적화         | X                         | O                     |
| 디버그정보포함 | O                         | X                     |
| 라이브러리     | 디버그용 런타임라이브러리 | 일반 런타임라이브러리 |

> Debug 모드 빌드시 디버그용 런타임 라이브러리(\*d.dll)를 사용하게 되고, 사용자 PC에 Windows SDK가 설치되어 있지않으면 실행에 실패함.

- C++ Redistributable은 일반 런타임 라이브러리 설치이고, 디버그용 런타임 라이브러리 설치가 아님.
