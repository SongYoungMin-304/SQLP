# SQLP

SQLP 준비

### SQL

→ Structured Query Language

→ 구조적 질의 언어

사용자 → (sql) → 옵티마이저 → (실행계획) → 프로시저

SQL 실행

1) SQL 파싱

- 파싱트리 생성
- SYNTAX 체크 : 문법적 오류 확인
- SEMANNTIC 체크 : 의미 상 오류 확인(존재하지 않는 테이블 OR 컬럼, 권한 존재 확인)

2) SQL 최적화

- SQL 옵티마이저는 미리 수집한 시스템 및 오브젝트 통계정보를 바탕으로 다향한 실행경로를 생성해서 비교한 후 가장 효울적인 하나를 선택한다.
- 성능을 결정하는 핵심적인 엔진

3) 로우 소스 생성 

- SQL 옵티마이저가 선택한 실행결로를 실제 실행 가능한 코드 또는 프로시저 형태로 포맷팅
- 로우 소스 생성기 가 그 역할을 맡음

※ SQL 옵티마이저

→ 사용자가 원하는 작업을 가장 효율적으로 수행할 수 있는 최적의 데이터 엑세스 경로를 선택해주는 DBMS의 핵심엔진이다.

1) 사용자로부터 전달 받은 쿼리를 수행하는 데 후보군이 될 만한 실행 계획을 찾아냄

2) 데이터 딕션너리에 미리 수집해 놓은 정보로  각 실행 계획의 예상 비용을 산정한다.

3) 최저 비용을 나타내는 실행 계획을 선택한다.

### 오라클 힌트 목록

**1. 최적화 목표**

- /*+ALL_LOWS */: 전체 처리속도 최적화
- /*+FIRST_ROWS(N) */ : 최초 N건 응답속도 최적화

**2.액세스 방식**

- /*+FULL */ :인덱스 타지말고 바로 테이블 풀스캔으로 접근해라
- /*+INDEX */: 인덱스를 타라
- /*+INDEX_DESC */: 인덱스를 ORDER BY DESC 역순으로 타라 (시간, 결과값등 최근인것 혹은 MAX값 구할때 좋음)
- /*+INDEX_FFS */: INDEX FAST FULL SCAN으로 타라
- /*+INDEX_SS */: INDEX SKIP SCAN

**3. 조인순서**

- /*+ORDERED */: FROM절에 나열된 테이블 순서대로 조인해라
- /*+LEADING */: 내가 힌트절에 열거한 테이블 순서대로 조인해라 EX: /*+ LEADING (A B C)*/

-- A,B,C 순서대로 조인하세요

- /*+SWAP_JOIN_INPUTS */: 해시조인의 경우, BUILD INPUT를 명시적으로 선택

EX: /*+ SWAP_JOIN_INPUTS(A)*/

--해시조인의경우 BUILD INPUT과 PROBE에 대한 순서를 정할 수 있다.

**4. 조인방식**

- /*+USE_NL */ :NL(NESTED LOOP - 중첩루프)방식 조인 유도
- /*+USE_MERGE */: 소트머지 조인으로 유도
- /*+USE_HASH */: 해시조인으로 유도
- /*+NL_SJ */: NL SEMI조인으로 유도
- /*+MERGE_SJ */: 소트머지 세미조인으로 유도
- /*+HASH_SJ */: 해시 세미조인으로 유도

**5. 서브쿼리팩토링**

- /*+MATERIALIZE */: WITH문으로 정의한 집합을 물리적으로 생성하도록 유도

EX) WITH /*+ MATERIALIZE*/ T AS (SELECT ...)

- /*+INLINE */: WITH문으로 정의한 집합을 물리적으로 생성하지않고 INLINE 처리하도록 유도

EX)WITH /*+ INLINE*/ T AS (SELECT ...)

**6.쿼리변환**

- /*+ MEERGE */: 뷰 머징 유도
- /*+NO_MERGE */: 뷰 머징 방지
- /*+UNNEST */: 서브쿼리 UNNESTING 유도
- /*+NO_UNNEST */: 서브쿼리 UNNESTING 방지
- /*+PUSH_PRED */: 조인조건 PUSHDOWN 유도
- /*+NO_PUSH_PRED */: 조인조건 PUSHDOWN 방지
- /*+USE_CONCAT */: OR 또는 IN-LIST조건을 OR-EXPANSION으로 유도
- /*+NO_EXPAND */: OR 또는 IN-LIST 조건에 대한 OR-EXPANSION방지

**7.병렬처리**

- /*+PARALLEL */: 테이블 스캔, DML 병렬방식으로 처리하도록 할때 사용.. 단일 대형 테이블의 접근시 정말 많이 쓴다.

EX) /*+ PARALLEL(T1 4)*/

- /*+PARALLEL_INDEX */: 인덱스 스캔을 병렬방식으로 처리하도록 유도
- /*+PQ_DISTRIBUTE */: 병렬수행시 데이터 분배방식 결정

EX) PQ_DISTRIBUTE(T1 HASH(--BUILD INPUT) HASH(--PROBE TABLE))

**8. 그외 기타**

- /*+APPEND*/: DIRECT PATH INSERT유도로 INSERT 문에 주로 많이 쓴다
- /*+DRIVING_SITE */: DB LINK REMOTE쿼리에 대한 최적화 및 실행 주체 지정 (LOCAL 또는 REMOTE)
- /*+PUSH_SUBQ */: 서브쿼리를 가급적 빨리 필터링하도록 유도
- /*+NO_PUSH_SUBQ */: 서브쿼리를 가급적 늦게 필터링 하도록 유도

### 소프트 파싱 VS 하드파싱

소프트 파싱 : 캐시에서 찾아서 곧바로 실행

하드 파싱 : 캐시에서 찾는 데 실패, 최적화 & 로우 소스 생성

```mermaid
flowchart LR
node(사용자)
n2(SQL파싱)
n3(캐시에 존재)
n4(실행)
n5(최적화)
n6(로우소스생성)
node --> n2 --> n3 --> |Y|n4
n3 --> |N|n5 --> n6
```

※ 옵티마이저가 사용하는 정보

1) 테이블, 컬럼, 인덱스 구조에 관한 기본정보

2) 오브젝트 통계 : 테이블 통계, 인덱스 통계

3) 시스템 통계 : cpu속도, I/O 속도 등

→ 어렵게 생성한 프로시저를 한번만 사용하고 버린다면 비효율**(라이브러리 캐시가 필요한 이유)**

```sql
SELECT * FROM CUSTOMER WHERE LOGIN_ID ='oraking';
SELECT * FROM CUSTOMER WHERE LOGIN_ID ='oraking1';
SELECT * FROM CUSTOMER WHERE LOGIN_ID ='oraking2';
SELECT * FROM CUSTOMER WHERE LOGIN_ID ='oraking3';
-> 하드파싱

SELECT * FROM CUSTOMER WHERE LOGIN_ID = :?
-> 한번만 하드 파싱하고 소프트 파싱으로 처리 됨
```

### 데이터 저장 구조 및 I/O 매커니즘

![image](https://user-images.githubusercontent.com/56577599/232319149-e651c7b0-ab25-4251-a0cd-da002304dec8.png)


```mermaid
flowchart LR

n1(테이블스페이스)
n2(세그먼트)
n3(익스텐트)
n4(블록)
n5(데이터파일)

n1 --> n2 --> n3 --> n4
n2 --> n5--> n1
n3 --> n5
```

- 블록 : 데이터를 읽고 쓰는 단위
- 엑스턴트 : 공간을 확장하는 단위, 연속된 블록 집합
- 세그먼트 : 데이터 자장공간이 필요한 오브젝트
- 테이블스페이스 : 세그먼트를 담는 콘테이너
- 데이터 파일 : 디스크 상의 물리적인 os 파일

※블록: DBMS가 데이터를 읽고 쓰는 단위(한 블록에 10ROW 가 있다면 1 ROW를 읽어도 한블록을 읽음)

### 시퀀셜 엑세스 VS 랜덤 엑세스

시퀀셜엑세스 : 논리적 또는 물리적으로 연결된 순서에 따라 차례대로 블록을 읽는 방식

랜덤엑세스 : 논리적 물리적 순서를 따르지 않고 한블로씩 접근

### 논리적 I/O VS 물리적 I

논리적 블록 I/O : SQL을 처리하는 과정에서 발생한 총 블럭 I/O

물리적 블록 I/O : 디스크에서 발생한 총 블록 I/O

```mermaid
flowchart LR

n1(Server Process)
n2(DB 버퍼 캐시)
n3(Server Process)
n4(DB File)

n1 <--> |논리적블록I/O|n2 --> |물리적블록I/O| n3

n2 -->|물리적블록I/O| n4
```

버퍼캐시 히트율(BCHR)

: (캣기에서 찾은 블록 수 / 총 읽은 블록 수) * 100

: (논리적 I/O - 물리적 I/O ) / 논리적I/O * 100

: (1 - (물리적 I/O) / (논리적 I/O) * 100

히트율이 높아야함 !

**SQL 튜닝을 통해서 논리적 I/O 블록의 갯수를 줄이는 것이 튜닝**

| TABLE FULL SCAN | INDEX RANGE SCAN |
| --- | --- |
| MULTI BLOCKED I/O 방식 | SINGLE BLOCKED I/O 방식 |
| 수레에 데이터가 많다 | 수레에 데이터가 적다. |
|  | 반복적인 I/O |



# 인덱스 구조 및탐색

홍길동 학생을 찾는 방법

1) 모든 교실을 돌면서 찾기

2) 학생 명부를 조회해 홍길동 학생이 있는 교실만 찾아냄

### 인덱스를 통한 데이터 탐색 과정

1) 수직적 탐색 + 수평적 탐색을 통한 인덱스 탐색 진행

2) 인덱스 탐색에서 나온 rowId를 통해서 테이블 랜덤 엑세스 

### 인덱스 원리

(SELECT * FROM 고객 WHERE 고객명 = ‘이재희’)

1) 수직적 탐색 + 수평적 탐색을 통한 인덱스 탐색 진행

![image](https://user-images.githubusercontent.com/56577599/232505207-b54c52f4-8fef-424f-b945-0262737ae8e2.png)



1) ROOT 에서 “서” 로 확인을 해봤을 때 이재희는 서 보다 뒤에 있다. 

→ 따라서 서 뒤에 있는 BRANCH 로 이동

2) BRANCH 에서 계속 스캔을 하다가 이재희보다 뒤에 있는 이재홍을 확인

→ 해당 LEAF로 내려간다.

**—> 여기까지가 수직적 탐색**

3) 이재희가 있는 곳에서 계속 조건에 만족하는 이재희를 찾는다.

**—> 여기가 수평적 탐색**

4) 인덱스 탐색을 통해서 가져온 ROW ID를 통해서 테이블에서 데이터를 가져온다.

**—> 랜덤 액세스**  

### **Index Range Scan**

- 스캔 지점에 시작점과 종료점이 있어야함
- 인덱스 컬럼이 가공되면 사용 못함(EX) 5월생의 데이터를 가져온다고 했을 때 인덱스의 시작점과 종료점을 알수 없기 때문(날짜 순으로 정렬)
- IN 조건절에 대해서는 IN-LIST Iterator 방식을 사용(IN-LIST 갯수만큰 INDEX RANGE SCAN 반복, UNION ALL 처럼)
- 인덱스의 선두컬럼이 가공되지 않으면 무조건 RANGE SCAN 이 가능한다.
- 선두 컬럼만 가공되지 않는다고 하더라도 문제가 없는 건 아님(범위가 그만큼 안 줄어들기 때문)
- 인덱스는 정렬이 되어있다. (인덱스 스캔을 할 수 있는 이유)
→ 따라서 인덱스를 사용하면 ORDER BY 를 하더라도 SORT 연산을 하지 않는다.

![image](https://user-images.githubusercontent.com/56577599/232505363-38e08ce9-4c24-46e0-b954-dd18322e6b21.png)


### **Index Full scan**

- 수직점 탐색을 하지 않고 수평적 탐색만 진행
- ename, sal 순으로 인덱스가 생성이 되었을 때 조회 조건이 sal 만 있다면 range 시작 지점과 종료지점을 정의하기가 어려워서 전체를 스캔한다.
- 즉 선두컬럼이 조회 조건에 없는 경우이다.
- 인덱스 전체 스캔
- 데이터가 적을 경우 효율적이지만 데이터가 많은 경우 그냥 테이블 full scan 이 좋다.(row id 로 테이블에 접근하는 건이 엄청 많을 거기 때문에,,)

![image](https://user-images.githubusercontent.com/56577599/232505403-21f44dbe-38ab-4f2e-9b42-2827ed5ba487.png)


![image](https://user-images.githubusercontent.com/56577599/232505436-d9d8c304-00c0-49d5-8d70-750feac9af4f.png)


### **Index Unique scan**

- 수직점 탐색으로만 데이터를 찾는 스캔 방식(= 조건으로 unique 인덱스)

![image](https://user-images.githubusercontent.com/56577599/232505474-46b8307d-27a9-4926-a662-d78efbeb202d.png)


### **Index Skip scan**

- 인덱스의 선두 컬럼이 조회 조건에 없구 데이터가 별로 없을 때는 index full scan
- 인덱스의 선두 컬럼이 조회 조건에 없구 데이터가 많은 경우 table full scan
- **인덱스의 선두 컬럼이 조건절에 없어도 인덱스를 활용하는 새로운 스캔 방식
(선두 컬럼의 Distinct Value 개수가 적고 후행 컬럼의 Distinct Value 개수가 많을 때 유용)**

![image](https://user-images.githubusercontent.com/56577599/232505513-4a1c702b-1510-4b5b-ac74-8100f977bd22.png)


![image](https://user-images.githubusercontent.com/56577599/232505544-b0e71340-31f8-4cb4-ae85-32b0792e3355.png)


```sql
select * from 사원 where 성별 = '남' and 연봉 between 2000 and 4000
```

1) 해당 쿼리를 한다면 3번 블럭을 수직적 탐색을 통해서 가져와서 rowid로 테이블을 접근한다.

```sql
select * from 사원 where 연봉 between 2000 and 4000
```

1) 해당 쿼리를 한다면 1번 블록(처음), 3번 블록 6번 블록, 7번 블록, 10번 블록(마지막)을 가져와서 rowId로 테이블에 접근한다.
**→ 특정 블록만 가져온다**

### **Index Fast Full scan**

- Multiblock I/O 방식으로 스캔해서 INDEX FULL SCAN 보다 빠름
(동시에 여러개의 브랜치에서 리프에 접근한다.)

| Index Full Scan | Index Fast Full Scan |
| --- | --- |
| 인덱스 구조를 따라 스캔 | 세그먼트 전체를 스캔 |
| 결과 집합 순서를 보장 | 결과집합 순서 보장 안됨 |
| Single Block I/O | MultiBlock I/O |
| 병렬 스캔 불가 | 병렬 스캔 가능 |
| 인덱스가 포함되지 않은 컬럼 조회 시에도 사용 가능 | 인덱스에 포함된 컬럼으로만 조회할 때 사용 가능 |

### **Index Range Scan Descending**

- 뒤에서 앞으로 스캔하는 것만 다름(내림차순)

![image](https://user-images.githubusercontent.com/56577599/232505608-24bb3da0-85f5-4db7-a9c8-714ae1793c4f.png)
