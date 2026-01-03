# DataBase

## 개념 정리

### Key의 종류

1.  primary key : 식별할 수 있는 유일한 값, 중복 없음, Null 없음
2.  alternate key : primary key가 될 수 있는 key이지만 primary key로 지정 안된 키
3.  foreign key : 특정 테이블에 포함되어 있고, 다른 테이블의 primary key로 지정된 키
4.  composite key : 여러 열을 조합하여 primary key 역할을 할 수 있는 키

candidate key : primary key가 될 수 있는 키 - primary key + alternate key

### 자료형

-   CHAR : 고정길이 문자열 데이터 (4000bytes 까지)
-   VARCHAR2 : 가변길이 문자열 데이터 (4000bytes 까지)
-   NVARCHAR2 : 가변길이 문자열 데이터 (4000bytes 까지) - 국가별 문자 세트 데이터
-   DATE : 날짜 데이터 (세기, 연, 월, 일, 시, 분, 초)
-   NUMBER : +-38자릿수 숫자, NUMBER(P,S)로 표기시 - 전체 P자리, 소수점 S자리
-   BLOB : 대용량 이진데이터 (최대크기 4GB)
-   CLOB : 대용량 텍스트 데이터 (최대크기 4GB)
-   BFILE : 대용량 이진데이터 파일 (최대크기 4GB)

### 오라클 객체 종류

-   table : 데이터 저장 장소
-   index : 데이터의 검색효율을 높이기 위해 사용
-   view : 하나 또는 여러 개의 선별된 데이터를 논리적으로 연결해 하나의 테이블 처럼 사용
-   sequence : 일련번호 생성
-   synonym : 오라클 객체의 별칭(alias)를 지정함
-   procedure : 프로그래밍 연산 및 기능 수행이 가능 (반환값 없음)
-   function : 프로그래밍 연산 및 기능 수행이 가능 (반환값 있음)
-   package : 관련있는 procedure와 function을 보관함
-   trigger : 데이터 관련 작업의 연결 및 방지 관련 기능 제공

## DB 조회하기

### 데이터를 조회하는 3가지 방법

-   셀렉션 : 행 단위 조회
-   프로젝션 : 열 단위 조회
-   조인 : 두 개 이상의 테이블을 사용하여 조회

셀렉션 + 프로젝션 조합도 가능함

```
-- 기본 조회
SELECT 열 FROM 테이블; /* 열 : 열이름 또는 열1, 열2, 열3 또는 * (전체) */

-- 중복데이터 삭제 : DISTINCT
SELECT DISTICT 열 FROM 테이블; /* 해당 열 기준 중복 제거 */
SELECT DISTICT 열, 열2 FROM table; /* 열1 + 열2 열 기중 중복 제거 */

-- 별칭 설정 방식 4가지
-- 3번째 방식 가장 많이 사용 : 알아보기 좋고, 프로그래밍시 "" 사용이 겹치지 않음
-- 예 : 열*12 생성시
SELECT 열*12 별칭 FROM 테이블
SELECT 열*12 "별칭" FROM 테이블
SELECT 열*12 AS 별칭 FROM 테이블
SELECT 열*12 AS "별칭" FROM 테이블

-- 원하는 순서대로 정렬 : ORDER BY
-- 주의 : 많은 자원이 필요하므로, 꼭 필요한 경우가 아니라면 사용하지 않는 것이 좋음
SELECT 열 FROM 테이블 ORDER BY 열; /* 열 기준 오름차순 정렬 */
SELECT 열 FROM 테이블 ORDER BY 열 DESC; /* 열 기준 내림차순 정렬 */
SELECT 열 FROM 테이블 ORDER BY 열1 ASC, 열2 DESC; /* 열1 기준 오름차순 1순위 정렬 후, 열2 기준 내림차순 2순위 정렬 */

-- 필요한 데이터만 출력하는 : WHERE
SELECT 열 FROM 테이블 WHERE 조건식

```

### 연산자

-   AND, OR, NOT : 논리 연산
-   \*,-,/,+ : 사칙연산
-   \>, >=, <, <=, =, !=, <>, ^= : 비교연산, 같지 않다를 3가지 표현으로 사용(!=, <>, ^=), 문자열의 경우 알파벳순 비교
-   열 IN 데이터리스트 : 열의 값의 데이터 리스트 내에 있으면 True
-   열 NOT IN 데이터리스트 : 열의 값의 데이터 리스트 내에 없으면 True
-   BETwEEN A AND B : A와 B 사이의 값
-   LIKE : 문자열 비교, 와일드카드 (\_, %) 사용 가능, 와일드카드 사용시 조회 성능에 영향을 미칠 수 있음을 감안할 것!
-   IS NULL : NULL 이면 True
-   UNION : 조회한 결과 합집합
-   UNION ALL : 조회한 결과 합집합 (중복허용)
-   MINUS : 조회한 결과 차집합
-   INTERSECT : 조회한 결과 교집합
