---
layout: single
title: "[electron] Addon & Node-API"
categories: application/electron
---

#### 핵심 요약

> Node.js에서 처리하기 힘든 고성능 작업은 C/C++ 코드로 작성함.
> Addon : Node.js에서 사용할 수 있는 C/C++ 코드로 작성된 동적 링크 라이브러리
> Node-API : Addon을 구현하기 위한 방법 중 하나, Node.js 버전 호환성 지원

#### 핵심 원리

- Node.js 자체가 C++로 만들어져있음.
- Addon을 구현할 수 있는 개발환경을 Node.js에서 공식 제공함.

## Addon

- Addon 이란, Node.js에서 사용할 수 있는 C/C++로 작성된 동적 링크 라이브러리임.
- `require` 함수를 통해 Node.js의 모듈로서 로드됨.
- `.node` 확장자를 가지는데, 실질적으로 Windows에서는 `.dll` 파일이고, Linux 계열에서는 `.so` 파일임.
- C/C++ 라이브러리와 js 사이의 인터페이스를 제공함.

#### Addon을 구현하는 3가지 방법

1. 직접 V8, libuv, Node.js 라이브러리를 사용하는 방법 (`node.h` 사용)
2. 1번 보단 좀 더 추상화된 Native Abstractions for Node.js를 사용하는 방법 (`nan.h` 사용)
3. Node-API를 사용하는 방법 (`node_api.h` 또는 `napi.h` 사용)

> 1번과 2번은 결국 Node.js 버전별로 빌드가 필요하고, 일반적인 상황에서는 Node-API를 사용하는 것을 추천

- 단, Node-API는 모든 기능을 지원하지는 않으므로, 특정 기능이 꼭 필요한 경우 1번과 2번 방법으로 구현할 필요가 있을 수 있음

#### node-gyp

- Addon 코드는 `node-gyp`에 의해 `{모듈명}.node` 파일로 컴파일 됨
- `npm install -g node-gyp`를 통해 설치
- `node-gyp configure` : `node-gyp`가 `binding.gyp` 파일의 내용을 참고하여 `build` 폴더 안에 현재 플랫폼에 맞는 프로젝트 빌드 파일을 생성함
  - Windows 플랫폼의 경우 `vcxproj` 파일을, Unix 계열 플랫폼의 경우 `Makefile`을 `build` 폴더 안에 생성함
- `node-gyp build` : `build/Release` 폴더 안에 `{모듈명}.node` 파일이 생성됨
  - 실질적으로 `.node` 파일은 Windows 플랫폼의 경우 `.dll`, Unix 계열 플랫폼의 경우 `.so` 파일임

#### npm install & node-gyp

- `npm`(Node.js 패키지 관리자)은 내부적으로 특정 버전의 `node-gyp`를 포함하고 있긴함.
- 개발자가 `npm` 내부의 `node-gyp`를 직접 사용할 수는 없음.
- `npm` 내부의 `node-gyp`는 `npm install` 명령을 통해 애드온을 컴파일하고 설치하는 용도로 설계됨.
- `npm install` 이라고 하는 명령어가, 이 `node-gyp configure && node-gyp build`를 내부적으로 실행하는거임.
- addon 개발자가 코드를 배포하고, 코드를 받아 우리 Node.js 버전에 맞게 컴파일해서 사용하는 메커니즘임.

## Node-API

- 네이티브 애드온을 개발하기 위한 방법 중 하나
- Javascript 런타임(예. v8)과 독립적임
- Node-API은 ABI(Application Binary Interface) 안성을 제공함.
- 특정 버전의 Node.js에서 컴파일된 애드온이 다른 버전에서 재컴파일 없이 동작가능

> 애드온이 js엔진 변경사항에 영향을 받지 않도록 보호하는 역할

#### node_api.h vs napi.h

Node.js Native Addon 개발에 사용되는 node_api.h와 napi.h는 사실상 같은 기능을 제공함.

> 결론적으로, node_api.h는 Node-API의 기반이 되는 원시적인 C 인터페이스를 제공하고, napi.h는 그 위에 node-addon-api라는 C++ 래퍼를 얹어 개발자 편의성을 극대화한 버전임. 특별한 이유가 없다면 node-addon-api를 사용하자!

node_api.h vs napi.h의 차이

| 특징        | node_api.h                                | napi.h                                 |
| ----------- | ----------------------------------------- | -------------------------------------- |
| 제공 주체   | Node.js 런타임 자체                       | node-addon-api npm 패키지              |
| 목적        | Node-API의 C 인터페이스 직접 사용         | Node-API를 위한 C++ 래퍼 라이브러리    |
| 코드 스타일 | C 스타일의 함수 호출 (napi_create_string) | C++ 객체 및 메서드 (Napi::String::New) |

어떤 것을 사용해야 하나?

- 대부분의 경우, napi.h를 통해 node-addon-api를 사용하는 것을 강력히 권장함.
  - C++ 친화적: C++ 래퍼를 통해 더 간결하고 직관적인 C++ 코드를 작성할 수 있음.
  - 편의성: 메모리 관리, 예외 처리 등 많은 번거로운 작업을 node-addon-api가 대신 처리해 줌
  - 하위 호환성: node-addon-api는 내부적으로 Node-API 버전을 관리하여, 구형 Node.js 버전에서도 안정적으로 동작하도록 도와줌.
  - 생산성: 더 빠르게 Node.js Native Addon을 개발하고 유지보수할 수 있음.
- node_api.h를 직접 사용하는 경우는 매우 제한적임.
  - node-addon-api의 오버헤드조차 허용할 수 없는 극도로 성능에 민감한 경우 (거의 없음).
  - C 스타일 코드로만 작성해야 하는 특정 제약이 있는 경우.
  - Node-API의 내부 동작 방식을 아주 깊게 이해하고 싶을 때.

```cpp
//  node-addon-api 코드 (napi.h)
Napi::String Method(const Napi::CallbackInfo& info) {
    Napi::Env env = info.Env();
    return Napi::String::New(env, "world");
}

// 실제 사용 코드 (Node-API 호출)
napi_value Method(napi_env env, napi_callback_info info) {
    napi_value result;
    napi_create_string_utf8(env, "world", NAPI_AUTO_LENGTH, &result);
    return result;
}
```

#### Node-API의 특징

- `status = napi_func(&out_param);`
  - 상태 코드 반환 : 모든 Node-API 호출은 napi_status 타입의 상태 코드를 반환함.
  - 출력 매개변수 사용 : Node-API의 반환 값은 출력 매개변수(out parameter)를 통해 전달됨
- JavaScript 값 추상화 : 모든 Javasecript 값은 napi_value 값으로 추상화 됨(상위 부모가 napi_value 라는 의미인 것 같음)
- ECMA-262 표준과 매핑 : Node-API는 JavaScript의 개념과 연산을 ECMA-262 언어 사양에 정의된 아이디어와 매핑함.
- C++ 래퍼 : node-addon-api : Node-API는 기본적으로 C API 이지만, `node-addon-api`라는 C++ 래퍼 모듈이 제공됨.

#### 참고자료

- nodejs.org/api/addons.html
- nodejs.org/api/n-api.html
