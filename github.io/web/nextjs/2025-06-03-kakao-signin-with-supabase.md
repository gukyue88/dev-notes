---
layout: single
title: "[Next.js] supabase & Kakao Signin"
categories: web/nextjs
---

## supabase & 카카오 개발자 연동

#### [supabase](https://supabase.com/)의 카카오 콜백 URI 값 확인

- `SUPABASE_KAKAO_CALLBACK_URI` : Authentication > Sign In / Providers > Kakao > Callback URL

#### supbase redirect URLs 등록

> 이 부분은 나중에 production 상황에서 필요한 부분이나 미리 등록

- `VERCEL_URL` : vercel > project 선택 > domains
- supabase > 프로젝트 선택 > Authentication > URL Configuration > Redirect URLs > Add URL
  - `http://localhost:3000/**`
  - `{VERCEL_URL}/**`

#### 카카오 개발자 사이트 프로젝트 생성 및 설정

- 프로젝트 생성 : 앱이름(이 부분이 나중에 인증시 사용자에게 보여질 이름으로 생각됨), 회사명, 카테고리 선택 후 저장
- 비즈앱 전환
  - 프로젝트 > 비즈니스 > 앱아이콘 등록 (아무거나)
  - 프로젝트 > 비즈니스 > 개인 개발자 비즈 앱 전환 > 목적 : 이메일 필수 로 해서 신청
- 앱 키 확인
  - `KAKAO_REST_API_KEY` : 프로젝트 > 앱 키 > REST API 키
- 카카오 로그인 설정 :
  - 프로젝트 > 카카오 로그인 > 활성화 상태 On
  - 프로젝트 > 카카오 로그인 > Redirect URI : supabase의 `SUPABASE_KAKAO_CALLBACK_URI`를 등록
  - 프로젝트 > 동의항목 (카카오 로그인)
    - 닉네임, 프로필 사진, 카카오 계정등 > 설정 > 필수 동의 > 동의목적 작성 > 저장
  - `KAKAO_CLIENT_SECRET` : 프로젝트 > 보안 (카카오 로그인) > 코드 생성

#### supabase Authentication > Sign In / Providers > Kakao 설정

- Kakao enabled : true
- REST API Key : `KAKAO_REST_API_KEY`
- Client Secret Code : `KAKAO_CLIENT_SECRET`
- Save 클릭

## 기존 코드 삭제

- app/auth
- app/protected
- app/opengraph-image.png
- app/twitter-image.png
- components

## 코드 추가 및 수정

#### lib/supabase/client.ts

```ts
import { createBrowserClient } from "@supabase/ssr";

if (!process.env.NEXT_PUBLIC_SUPABASE_URL) {
  throw new Error("Missing env var: NEXT_PUBLIC_SUPABASE_URL");
}
if (!process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY) {
  throw new Error("Missing env var: NEXT_PUBLIC_SUPABASE_ANON_KEY");
}

export const supabase = createBrowserClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);
```

- 클라이언트 사이드의 supabase client를 싱글톤 패턴으로 사용하기위해 변경함
- 서버 사이드의 supabase client의 경우, 매 요청마다 client을 만드는 것이 일반적이라고 함. (추가 공부 필요)

#### components/KakaoSignInButton.tsx & app/auth/callback/route.ts

components/KakaoSignInButton.tsx

```tsx
"use client";

import { supabase } from "@/lib/supabase/client";
import { Session } from "@supabase/supabase-js";
import { Dispatch, SetStateAction, useEffect, useState } from "react";

const signInWithKakao = async () => {
  const { data, error } = await supabase.auth.signInWithOAuth({
    provider: "kakao",
    options: {
      redirectTo: process.env.NEXT_PUBLIC_VERCEL_URL
        ? `https://${process.env.NEXT_PUBLIC_VERCEL_URL}/auth/callback`
        : "http://localhost:3000/auth/callback",
    },
  });
  console.log("data : ", data);
  console.log("error : ", error);
};

const signOut = async (
  setSession: Dispatch<SetStateAction<Session | null>>
) => {
  await supabase.auth.signOut();
  setSession(null);
  console.log("signOut()");
};

export default function KakaoSignInButton() {
  const [session, setSession] = useState<Session | null>(null);

  useEffect(() => {
    const loadSession = async () => {
      const {
        data: { session },
        error,
      } = await supabase.auth.getSession();

      setSession(session);
      console.log(session?.user);
      console.log("error : ", error);
    };
    loadSession();
  }, []);

  return (
    <button
      onClick={
        session
          ? () => {
              signOut(setSession);
            }
          : signInWithKakao
      }
    >
      {session ? "로그아웃" : "카카오 로그인"}
    </button>
  );
}
```

app/auth/callback/route.ts

```ts
import { NextResponse } from "next/server";
import { createClient } from "@/lib/supabase/server";

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url);
  const code = searchParams.get("code"); // if "next" is in param, use it as the redirect URL
  const next = searchParams.get("next") ?? "/";

  if (code) {
    const supabase = await createClient();
    const { error } = await supabase.auth.exchangeCodeForSession(code);
    if (!error) {
      // origin : "http://localhost:3000", next : ""
      return NextResponse.redirect(`${origin}${next}`);
    }
  } // return the user to an error page with instructions

  return NextResponse.redirect(`${origin}/auth/error`);
}
```

- 카카오 로그인 버튼을 누르면 `supabase.auth.signInWithOAuth()` 함수가 호출됨.
- 브라우저에서 supabase에 카카오 로그인을 요청하게 됨.
- supabase는 브라우저를 카카오 서버로 리디렉션하여 로그인을 진행하도록 함.
- 카카오 서버는 인증이 끝나면 등록된 callback url (supabase)로 리디렉션하도록 함.
- 그럼 브라우저가 supabase의 callback uri로 리디렉션을 함.
- supabase는 `supabase.auth.signInWithOAuth()` 호출시 넣어준 options의 redirectTo(내 next.js 서비스)로 리디렉션 하도록 함.
- 브라우저는 내 서비스의 `/auth/callback`으로 redirection을 하게됨.
- 그럼 내 서비스의 `/auth/callback/route.ts`가 실행되고, 인증이 잘되었으면 브라우저를 `{url}/` (Index 페이지)로 이동시킴
- 그럼 브라우저가 내 서비스의 인덱스 페이지를 다시 요청하게 된것이니, KakaoSignInButton의 useEffect(()=>{},[]) 부분도 다시 불리게 됨.
- 그럼, 이때는 loadSession()에서 유저 정보를 얻을 수 있어서 유저정보를 콘솔로 찍게됨

> redirectTo를 `/auth/callback`이 아닌 `/`로 하면 더 간단한거 아닌가? `/auth/callback/router.ts`에서 인증 결과 확인 후 올바른 경우에만 인덱스 페이지로 넘어가고, 그렇지 않은 경우 error 페이지를 보이기위해 이렇게 처리한 것 같다

#### app/auth/error/page.tsx

```tsx
export default async function ErrorPage() {
  return (
    <main className="min-h-screen flex flex-col items-center">
      <div>Error Page</div>
    </main>
  );
}
```

- 카카오 인증에 실패했을 경우, 오게될 페이지 (route.ts에서 인증 실패시 /auth/error로 리디렉션)

#### app/page.tsx

```tsx
import KakaoSignInButton from "@/components/KakaoSignInButton";

export default function IndexPage() {
  return (
    <main className="min-h-screen flex flex-col items-center">
      <div>Index Page</div>
      <KakaoSignInButton />
    </main>
  );
}
```
