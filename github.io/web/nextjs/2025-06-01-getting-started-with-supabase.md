---
layout: single
title: "[Next.js] supabase와 함께 시작하기"
categories: web/nextjs
---

## Step 1. 프로젝트 생성

#### 프로젝트 생성

```bash
npx create-next-app -e with-supabase
```

- repository 이름이 vercel 배포시 domain이 됨을 감안하고 작명 필요

#### 프로젝트 로컬 실행

```bash
npm run dev
```

## Step 2. 프로젝트 배포

[Vercel](https://vercel.com/)

#### Vercel의 장점

- git에 push 하면 자동으로 변경사항을 감지하고, 자동으로 빌드 및 배포가 이루어짐.
- CDN을 통해 전세계적으로 콘텐츠를 빠르게 제공함
- 서버리스 함수를 지원하여 백엔드 로직을 구현할 수 있게 해줌
- secret code등도 환경변수로 서버에 넣어서 사용 가능

#### Vercel의 단점

- 광고를 달기 위해서는 plan에 가입해야함 (상업적사용)

#### Vercel 초기 설정

- signup시, Continue with GitHub 선택
  - verification code가 안보이면 **스팸메시자함 확인**
- 홈 > Import Git Repository > Install
  - Only select repositories
    - repository 선택
  - Install 클릭
- 홈 > Import Git Repository > 프로젝트 > Import
  - Environment Variables : API 키등 환경변수 등록
  - Deploy 클릭

> 나중에 새로운 repository를 추가할 때는 다른 화면으로 들어가야할 것처럼 보임. (바로 보이진않음)

#### 도메인 확인

- Vercel > 프로필 > Dashboard > 프로젝트 선택 > Domains

## Step 3. supabase 프로젝트 생성

- supabase > new project
  - Project name : 프로젝트 이름
  - Database Password : Generate a password 사용 (크게 쓸 일 없고, 나중에 변경 가능)
  - Region : 서비스 지역 선택

## Step 4. 환경변수 설정

#### 로컬 서버 환경변수 설정

파일 이름 변경 후 내용 수정 : .env.example -> .env.local

```
NEXT_PUBLIC_SUPABASE_URL={supabse > 프로젝트 > 설정 > Data API > Project URL > URL}
NEXT_PUBILC_SUPABASE_ANON_KEY={supabse > 프로젝트 > 설정 > API Keys > anon}
```

#### vercel 환경변수 설정

> 개발 중에는 필요하지 않지만, 배포를 위해 미리 등록함

- vercel > 프로젝트 > Settings > Environment Variables
- Production & Preview 선택 ( Development 해제)
- Sensitive : Enabled
- 위의 환경변수(NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBILC_SUPABASE_ANON_KEY) 추가
- Save 클릭
- Redeploy 클릭

## Step 5. 서버 재시작 및 브로우저 새로고침

```bash
npm run dev
```

- 서버 재시작 후 브라우저 새로고침을 하면 페이지가 변경됨

## 참고자료

- [Use supabase with Next.js - supabase DOCS](https://supabase.com/docs/guides/getting-started/quickstarts/nextjs)
