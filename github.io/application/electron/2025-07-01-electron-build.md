---
layout: single
title: "[electron] electron 빌드"
categories: application/electron
---

`npx create-next-app --example with-electron-typescript with-electron-typescript-app`으로 만들어진 프로젝트의 빌드 관련 내용 정리

#### package.json

```json
"script" : {
    "dist" : "npm run build && electron-builder",
    "build" : "npm run build-renderer && npm run build-electron",
    "build-renderer" : "next build renderer && next export renderer",
    "build-electron" : "tsc -p electron-src"
}
```

- `npm run dist`로 빌드 명령으로 빌드 가능
- `npm run dist` == `npm run build-renderer` + `npm run build-electron`
  - `npm run build-renderer` == `next build renderer && next export renderer`
    - 넥스트 앱의 renderer를 빌드하고 export함
  - `npm run build-electron` == `tsc -p electron-src`
    - tsc(타입스크립트 컴파일러러가 `tsconfg.json`파일 기반으로 빌드 작업을 진행함
    - tsconfg.json 파일을 보니 exclude node_modules가 존재함. 이걸 이용해볼 수도 있을듯

```json
"build": {
    "extraResources": {
        "./lib"
    }
}
```

- `extraResources`로 등록된 애들은 `build/win-unpacked/resources/` 경로로 들어가는 것 같음.
