# bitbake cheatsheet

## 문법

- 변수는 모두 문자열임(""나 ''로 값을 할당)
- 변수의 이름은 대문자로 시작
- 주석은 `#`을 사용. 사용하지 않는 줄에 작성해야 함.

## 명령어

```bash
# 버전 확인
bitbake --version

# 빌드 환경 설정 (build 디렉터리 내 conf 파일 및 templateconf.cfg 파일 생성)
source {script_name}

# 빌드
# -k 옵션 : 에러가 나도 끝까지 빌드
bitbake {recipe_name}

# 변수 값 확인
# -r 옵션 : 변화 추적
bitbake-getvar {recipe_name} {var_name}
```
