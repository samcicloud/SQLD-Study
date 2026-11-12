# SQLD 명령어 암기 치트시트 (D-2)

> Oracle 기준. `→` 오른쪽이 결과. 결과를 손으로 가리고 맞혀 보세요.
> 예시 데이터: `EMP`(14명), `DEPT`(10, 20, 30, 40번 부서). 40번 부서는 사원 없음.

---

## 0. 명령어 분류 (한 줄 암기)

| 분류 | 명령어 | 외우는 법 |
|---|---|---|
| DDL | CREATE, ALTER, DROP, RENAME, TRUNCATE | 구조를 바꿈 (Oracle: **자동 커밋**) |
| DML | SELECT, INSERT, UPDATE, DELETE, MERGE | 데이터를 바꿈 (롤백 가능) |
| DCL | GRANT, REVOKE | 권한 |
| TCL | COMMIT, ROLLBACK, SAVEPOINT | 트랜잭션 |

⚠️ TRUNCATE는 **DDL**, DELETE는 **DML**

---

## 1. SELECT 골격과 실행 순서

```sql
SELECT   DEPTNO, COUNT(*) AS CNT   -- ⑤
FROM     EMP                       -- ①
WHERE    SAL > 1000                -- ②
GROUP BY DEPTNO                    -- ③
HAVING   COUNT(*) >= 3             -- ④
ORDER BY CNT DESC;                 -- ⑥
```

**F-W-G-H-S-O** (프-웨-그-해-셀-오)

| 위치 | SELECT 별칭 사용 | 집계 함수 사용 |
|---|---|---|
| WHERE | ❌ | ❌ |
| GROUP BY | ❌ | ❌ |
| HAVING | ❌ | ⭕ |
| ORDER BY | ⭕ | ⭕ |

---

## 2. 연산자

```sql
WHERE SAL BETWEEN 1000 AND 2000   -- 1000 이상 2000 이하 (경계 포함)
WHERE ENAME LIKE 'S%'             -- S로 시작
WHERE ENAME LIKE '_A%'            -- 두 번째 글자가 A
WHERE ENAME LIKE '%\_%' ESCAPE '\'  -- _ 문자 자체를 찾을 때
WHERE DEPTNO IN (10, 20)
WHERE COMM IS NULL                -- ⭕
WHERE COMM = NULL                 -- ❌ 항상 0건
```

- 우선순위: `()` > `NOT` > 비교 > `AND` > `OR`
- `NOT IN (10, NULL)` → **항상 0건**
- `IN (10, NULL)` → 10인 행만 나옴 (NULL은 무시)

---

## 3. 단일행 함수 (입력 → 결과)

### 문자

| 함수 | 결과 |
|---|---|
| `SUBSTR('SQL Expert', 5, 3)` | `'Exp'` (5번째부터 3글자) |
| `SUBSTR('ABCDE', -2)` | `'DE'` (뒤에서 2번째부터 끝까지) |
| `INSTR('HELLO', 'L')` | `3` (처음 나오는 위치) |
| `LENGTH('SQL ')` | `4` (공백 포함) |
| `LPAD('7', 3, '0')` | `'007'` |
| `RPAD('A', 3, '*')` | `'A**'` |
| `TRIM('  A  ')` | `'A'` |
| `LTRIM('xxAxx', 'x')` | `'Axx'` |
| `RTRIM('xxAxx', 'x')` | `'xxA'` |
| `TRIM('x' FROM 'xxAxx')` | `'A'` |
| `REPLACE('ABCABC', 'A', 'Z')` | `'ZBCZBC'` |
| `REPLACE('ABC', 'A')` | `'BC'` (세 번째 인자 생략 시 삭제) |
| `CONCAT('A', 'B')` | `'AB'` (인자 2개만) |
| `'A' \|\| 'B' \|\| 'C'` | `'ABC'` |
| `UPPER('sql')` / `LOWER('SQL')` | `'SQL'` / `'sql'` |
| `INITCAP('sql dev')` | `'Sql Dev'` |

### 숫자

| 함수 | 결과 |
|---|---|
| `ROUND(123.456, 1)` | `123.5` |
| `ROUND(123.456, -1)` | `120` (일의 자리에서 반올림) |
| `ROUND(125, -1)` | `130` |
| `TRUNC(123.456, 1)` | `123.4` (버림) |
| `TRUNC(129, -1)` | `120` |
| `CEIL(1.1)` / `CEIL(-1.5)` | `2` / `-1` (올림: 크거나 같은 최소 정수) |
| `FLOOR(1.9)` / `FLOOR(-1.5)` | `1` / `-2` (내림: 작거나 같은 최대 정수) |
| `MOD(10, 3)` | `1` |
| `ABS(-5)` / `SIGN(-5)` | `5` / `-1` |

### 날짜

| 식 | 결과 |
|---|---|
| `SYSDATE + 1` | 내일 같은 시각 |
| `SYSDATE + 1/24` | 1시간 뒤 |
| `SYSDATE + 1/24/60` | 1분 뒤 |
| `날짜2 - 날짜1` | 두 날짜 사이 **일수** |
| `ADD_MONTHS(DATE '2026-01-31', 1)` | `2026-02-28` (말일 보정) |
| `MONTHS_BETWEEN(날짜2, 날짜1)` | 개월 수 |
| `LAST_DAY(DATE '2026-02-10')` | `2026-02-28` |
| `TRUNC(SYSDATE)` | 오늘 00:00:00 |
| `EXTRACT(YEAR FROM SYSDATE)` | 연도 숫자 |
| `TO_CHAR(SYSDATE, 'YYYY-MM-DD HH24:MI:SS')` | 날짜 → 문자 |
| `TO_DATE('20261114', 'YYYYMMDD')` | 문자 → 날짜 |

### NULL 함수

| 함수 | 결과 | 외우는 법 |
|---|---|---|
| `NVL(NULL, 0)` | `0` | NULL이면 대체 |
| `NVL(5, 0)` | `5` | |
| `NVL2(NULL, 'Y', 'N')` | `'N'` | NULL 아니면 2번째, NULL이면 3번째 |
| `NVL2(5, 'Y', 'N')` | `'Y'` | |
| `NULLIF(10, 10)` | `NULL` | 같으면 NULL |
| `NULLIF(10, 20)` | `10` | 다르면 첫 번째 |
| `COALESCE(NULL, NULL, 3, 4)` | `3` | 처음 나오는 NOT NULL |
| `100 + NULL` | `NULL` | NULL 연산은 무조건 NULL |
| `'A' \|\| NULL` | `'A'` | ⚠️ Oracle 문자 연결은 NULL 무시 |

### CASE / DECODE

```sql
CASE WHEN SAL >= 3000 THEN 'A' WHEN SAL >= 2000 THEN 'B' ELSE 'C' END
CASE DEPTNO WHEN 10 THEN '회계' WHEN 20 THEN '연구' END   -- ELSE 없으면 NULL
DECODE(DEPTNO, 10, '회계', 20, '연구', '기타')
```

---

## 4. 집계 함수 (EMP의 COMM = 300, 500, 1400, 0, 나머지 10명 NULL)

| 식 | 결과 |
|---|---|
| `COUNT(*)` | `14` (NULL 포함 행 수) |
| `COUNT(COMM)` | `4` (NULL 제외) |
| `SUM(COMM)` | `2200` |
| `AVG(COMM)` | `550` (2200 / **4**) |
| `SUM(COMM) / COUNT(*)` | `157.14...` (2200 / **14**) |
| `AVG(NVL(COMM, 0))` | `157.14...` |
| `COUNT(DISTINCT DEPTNO)` | `3` |

- 결과 행이 0건일 때: `COUNT(*)` → `0`, `SUM` / `MAX` / `AVG` → `NULL`
- GROUP BY 없이 집계 함수를 쓰면 결과는 항상 **1행** (0건이어도 1행)

---

## 5. 조인 문법

```sql
-- Oracle 조인
SELECT E.ENAME, D.DNAME FROM EMP E, DEPT D WHERE E.DEPTNO = D.DEPTNO;

-- ANSI 조인
SELECT E.ENAME, D.DNAME FROM EMP E INNER JOIN DEPT D ON E.DEPTNO = D.DEPTNO;
SELECT ENAME, DNAME, DEPTNO FROM EMP JOIN DEPT USING (DEPTNO);  -- DEPTNO에 접두사 ❌
SELECT ENAME, DNAME, DEPTNO FROM EMP NATURAL JOIN DEPT;         -- DEPTNO에 접두사 ❌
SELECT * FROM EMP CROSS JOIN DEPT;                              -- 14 × 4 = 56행

-- OUTER 조인 (40번 부서까지 나오게)
SELECT D.DNAME, E.ENAME FROM DEPT D LEFT OUTER JOIN EMP E ON D.DEPTNO = E.DEPTNO;
SELECT D.DNAME, E.ENAME FROM DEPT D, EMP E WHERE D.DEPTNO = E.DEPTNO(+);  -- 같은 결과
```

- `(+)`는 **데이터가 모자라는 쪽**(NULL로 채워질 쪽)에 붙임
- `FULL OUTER JOIN` = LEFT 결과 ∪ RIGHT 결과 (Oracle `(+)`로는 양쪽 불가)
- Non-EQUI 조인: `ON E.SAL BETWEEN S.LOSAL AND S.HISAL`

---

## 6. 서브쿼리

```sql
-- 단일행: = < > 사용
SELECT * FROM EMP WHERE SAL > (SELECT AVG(SAL) FROM EMP);

-- 다중행: IN, ANY, ALL, EXISTS 사용
SELECT * FROM EMP WHERE SAL IN (SELECT MAX(SAL) FROM EMP GROUP BY DEPTNO);

-- 연관 서브쿼리: 안쪽에서 바깥 컬럼(E.DEPTNO) 사용
SELECT * FROM EMP E
WHERE SAL > (SELECT AVG(SAL) FROM EMP WHERE DEPTNO = E.DEPTNO);

-- EXISTS: 존재 여부만
SELECT * FROM DEPT D WHERE EXISTS (SELECT 1 FROM EMP E WHERE E.DEPTNO = D.DEPTNO);

-- 스칼라 서브쿼리 (SELECT 절, 1행 1열)
SELECT ENAME, (SELECT DNAME FROM DEPT D WHERE D.DEPTNO = E.DEPTNO) FROM EMP E;

-- 인라인 뷰 (FROM 절)
SELECT * FROM (SELECT DEPTNO, AVG(SAL) AVG_SAL FROM EMP GROUP BY DEPTNO) WHERE AVG_SAL > 2000;
```

| 연산 | 의미 |
|---|---|
| `> ANY (100, 200)` | 100보다 크면 (**최솟값**보다 큼) |
| `> ALL (100, 200)` | 200보다 크면 (**최댓값**보다 큼) |
| `< ANY` | 최댓값보다 작음 |
| `< ALL` | 최솟값보다 작음 |
| `= ANY` | `IN`과 같음 |

---

## 7. 집합 연산자

```sql
SELECT DEPTNO FROM EMP
UNION            -- 중복 제거 → 10, 20, 30
SELECT DEPTNO FROM DEPT
ORDER BY 1;      -- ORDER BY는 맨 마지막에 한 번
```

| 연산 | `{1,2,2,3}` ? `{2,3,4}` |
|---|---|
| UNION | 1, 2, 3, 4 |
| UNION ALL | 1, 2, 2, 3, 2, 3, 4 (7개) |
| INTERSECT | 2, 3 |
| MINUS | 1 |

---

## 8. 그룹 함수 (소계/총계): 결과 행 수 맞히기

```sql
SELECT DEPTNO, JOB, SUM(SAL) FROM EMP GROUP BY ROLLUP(DEPTNO, JOB);
```

| 함수 | 집계 단위 | EMP 결과 행 수 |
|---|---|---|
| `GROUP BY DEPTNO, JOB` | (부서, 직무) | 9 |
| `ROLLUP(DEPTNO, JOB)` | (부서, 직무) + (부서) + (전체) | 9 + 3 + 1 = **13** |
| `ROLLUP(JOB, DEPTNO)` | (직무, 부서) + (직무) + (전체) | 9 + 5 + 1 = **15** |
| `CUBE(DEPTNO, JOB)` | (부서, 직무) + (부서) + (직무) + (전체) | 9 + 3 + 5 + 1 = **18** |
| `GROUPING SETS(DEPTNO, JOB)` | (부서) + (직무) | 3 + 5 = **8** |

- 공식: `ROLLUP(A, B)` → 인자 n개면 **n+1가지** 그룹 / `CUBE(A, B)` → **2ⁿ가지**
- `GROUPING(JOB)` → JOB이 소계로 합쳐진 행이면 `1`
- `ROLLUP(A, (B, C))` → (A, B, C), (A), () : 괄호로 묶으면 한 덩어리

---

## 9. 윈도우 함수

```sql
SELECT ENAME, DEPTNO, SAL,
       RANK()       OVER (ORDER BY SAL DESC) AS R,
       DENSE_RANK() OVER (ORDER BY SAL DESC) AS DR,
       ROW_NUMBER() OVER (ORDER BY SAL DESC) AS RN,
       SUM(SAL)     OVER (PARTITION BY DEPTNO) AS 부서합계,
       SUM(SAL)     OVER (ORDER BY SAL)       AS 누적합계,
       LAG(SAL)     OVER (ORDER BY SAL)       AS 이전행,
       LEAD(SAL, 1, 0) OVER (ORDER BY SAL)    AS 다음행_없으면0
FROM EMP;
```

| SAL | RANK | DENSE_RANK | ROW_NUMBER |
|---|---|---|---|
| 5000 | 1 | 1 | 1 |
| 3000 | 2 | 2 | 2 |
| 3000 | 2 | 2 | 3 |
| 2975 | **4** | **3** | 4 |

| 윈도우 범위 | 의미 |
|---|---|
| `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | 처음부터 현재 행까지 (누적) |
| `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` | 앞 1행 + 현재 + 뒤 1행 |
| `ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING` | 현재부터 끝까지 |
| `RANGE BETWEEN 100 PRECEDING AND 100 FOLLOWING` | 현재 **값** ±100 범위 |
| ORDER BY만 있고 범위 생략 | `RANGE UNBOUNDED PRECEDING ~ CURRENT ROW` → **누적합** (같은 값은 같이 더해짐) |
| ORDER BY도 없음 | 파티션 **전체** |

- `LAG(컬럼, n, 기본값)`: n행 앞, 없으면 기본값 (생략 시 NULL)
- `NTILE(4)`: 4등분해서 그룹 번호
- `RATIO_TO_REPORT(SAL)`: 합계 대비 비율 (합 = 1)
- `CUME_DIST`: 누적 분포 (0 초과 ~ 1) / `PERCENT_RANK`: (순위-1)/(행수-1) (0 ~ 1)

---

## 10. Top N

```sql
-- ⭕ 정렬 후 자르기
SELECT * FROM (SELECT * FROM EMP ORDER BY SAL DESC) WHERE ROWNUM <= 3;
-- ❌ 자른 뒤 정렬 (랜덤 3명을 정렬한 것)
SELECT * FROM EMP WHERE ROWNUM <= 3 ORDER BY SAL DESC;
-- Oracle 12c+
SELECT * FROM EMP ORDER BY SAL DESC FETCH FIRST 3 ROWS ONLY;
SELECT * FROM EMP ORDER BY SAL DESC FETCH FIRST 3 ROWS WITH TIES;  -- 동점 포함
-- SQL Server
SELECT TOP(3) WITH TIES * FROM EMP ORDER BY SAL DESC;
```

| 조건 | 결과 |
|---|---|
| `ROWNUM = 1` | 1건 |
| `ROWNUM <= 3` | 3건 |
| `ROWNUM = 2` | **0건** |
| `ROWNUM > 1` | **0건** |

---

## 11. 계층형 질의

```sql
SELECT LEVEL, LPAD(' ', 2*(LEVEL-1)) || ENAME AS 조직도,
       CONNECT_BY_ROOT ENAME AS 루트,
       CONNECT_BY_ISLEAF AS 리프여부,
       SYS_CONNECT_BY_PATH(ENAME, '/') AS 경로
FROM EMP
START WITH MGR IS NULL           -- 시작: KING
CONNECT BY PRIOR EMPNO = MGR     -- 순방향 (위 → 아래)
ORDER SIBLINGS BY ENAME;
```

| CONNECT BY | 방향 |
|---|---|
| `PRIOR EMPNO = MGR` (PRIOR가 PK 쪽) | 순방향: 관리자 → 부하 |
| `PRIOR MGR = EMPNO` | 역방향: 부하 → 관리자 |
| `EMPNO = PRIOR MGR` | 역방향 (위와 같음) |

- 기억법: "**PRIOR 붙은 컬럼 = 이전 행의 값**". `PRIOR EMPNO = MGR` → 이전 행의 사번이 내 MGR인 행 = 부하
- `LEVEL`: 루트 1부터 / `CONNECT_BY_ISLEAF`: 자식 없으면 1
- WHERE는 전개 **후** 필터 (해당 행만 빠지고 자식은 남음)
- 순환 데이터 → `CONNECT BY NOCYCLE PRIOR ...`

---

## 12. PIVOT / 정규표현식

```sql
SELECT * FROM (SELECT DEPTNO, JOB, SAL FROM EMP)
PIVOT (SUM(SAL) FOR JOB IN ('CLERK' AS CLERK, 'MANAGER' AS MANAGER));
```

| 식 | 결과 |
|---|---|
| `REGEXP_SUBSTR('A1B22C', '[0-9]+')` | `'1'` (첫 번째 매칭) |
| `REGEXP_SUBSTR('A1B22C', '[0-9]+', 1, 2)` | `'22'` (두 번째 매칭) |
| `REGEXP_REPLACE('010-1234-5678', '[^0-9]')` | `'01012345678'` |
| `REGEXP_COUNT('banana', 'a')` | `3` |
| `REGEXP_LIKE(ENAME, '^S')` | S로 시작하는 행 |
| `REGEXP_INSTR('ABC123', '\d')` | `4` |

---

## 13. DML

```sql
INSERT INTO DEPT VALUES (50, 'IT', 'SEOUL');                 -- 모든 컬럼 순서대로
INSERT INTO DEPT (DEPTNO, DNAME) VALUES (60, 'HR');          -- 나머지는 NULL/DEFAULT
INSERT INTO EMP_BAK SELECT * FROM EMP WHERE DEPTNO = 10;     -- 서브쿼리로 여러 행
UPDATE EMP SET SAL = SAL * 1.1, COMM = 100 WHERE DEPTNO = 10;
UPDATE EMP SET SAL = 1000;                                    -- WHERE 없으면 전체
DELETE FROM EMP WHERE DEPTNO = 40;
DELETE EMP;                                                   -- Oracle은 FROM 생략 가능

MERGE INTO EMP_BAK T
USING EMP S
ON (T.EMPNO = S.EMPNO)
WHEN MATCHED THEN
  UPDATE SET T.SAL = S.SAL
WHEN NOT MATCHED THEN
  INSERT (EMPNO, ENAME, SAL) VALUES (S.EMPNO, S.ENAME, S.SAL);
```

---

## 14. TCL: 시나리오로 외우기

**문제 1. 최종적으로 남는 행은?**
```sql
INSERT ... (1);
SAVEPOINT A;
INSERT ... (2);
SAVEPOINT B;
INSERT ... (3);
ROLLBACK TO A;
COMMIT;
```
→ **1만 남음** (A 이후 2, 3 취소)

**문제 2. Oracle에서 최종 결과는?**
```sql
INSERT ... (1);
CREATE TABLE T2 (...);   -- DDL → 자동 COMMIT
ROLLBACK;
```
→ **1이 남음** (DDL이 앞의 INSERT까지 커밋함)

**문제 3.** `ROLLBACK TO B` 후 `ROLLBACK TO A`는? → 가능 (뒤 → 앞 순서). `ROLLBACK TO A` 후 `ROLLBACK TO B`는? → **에러** (B는 이미 사라짐)

| | Oracle | SQL Server |
|---|---|---|
| 기본 커밋 | 수동 (COMMIT 필요) | AUTO COMMIT |
| DDL 실행 시 | 자동 커밋 | 트랜잭션 안이면 롤백 가능 |
| SAVEPOINT 문법 | `SAVEPOINT A` / `ROLLBACK TO A` | `SAVE TRANSACTION A` / `ROLLBACK TRANSACTION A` |

---

## 15. DDL: Oracle vs SQL Server

| 작업 | Oracle | SQL Server |
|---|---|---|
| 컬럼 추가 | `ALTER TABLE T ADD (C NUMBER);` | `ALTER TABLE T ADD C INT;` |
| 컬럼 변경 | `ALTER TABLE T MODIFY (C VARCHAR2(30));` | `ALTER TABLE T ALTER COLUMN C VARCHAR(30);` |
| 컬럼 삭제 | `ALTER TABLE T DROP COLUMN C;` | 같음 |
| 컬럼 이름 변경 | `ALTER TABLE T RENAME COLUMN A TO B;` | `sp_rename 'T.A', 'B', 'COLUMN';` |
| 테이블 이름 변경 | `RENAME T TO T2;` | `sp_rename 'T', 'T2';` |
| 테이블 복사 | `CREATE TABLE T2 AS SELECT * FROM T;` | `SELECT * INTO T2 FROM T;` |
| 제약조건 추가 | `ALTER TABLE T ADD CONSTRAINT PK_T PRIMARY KEY (ID);` | 같음 |
| 제약조건 삭제 | `ALTER TABLE T DROP CONSTRAINT PK_T;` | 같음 |
| 참조 무시하고 삭제 | `DROP TABLE T CASCADE CONSTRAINTS;` | (FK 먼저 삭제) |

```sql
CREATE TABLE MEMBER (
  ID      NUMBER(5)    CONSTRAINT PK_MEMBER PRIMARY KEY,
  NAME    VARCHAR2(20) NOT NULL,
  EMAIL   VARCHAR2(50) UNIQUE,
  AGE     NUMBER(3)    CHECK (AGE BETWEEN 0 AND 150),
  JOIN_DT DATE         DEFAULT SYSDATE,
  DEPTNO  NUMBER(2)    REFERENCES DEPT(DEPTNO) ON DELETE SET NULL
);
```

| 제약 | NULL | 중복 | 테이블당 |
|---|---|---|---|
| PRIMARY KEY | ❌ | ❌ | 1개 |
| UNIQUE | ⭕ | ❌ | 여러 개 |
| NOT NULL | ❌ | ⭕ | - |

- CTAS(`CREATE TABLE AS SELECT`) → 제약조건 중 **NOT NULL만** 복사
- FK `ON DELETE CASCADE` → 부모 삭제 시 자식도 삭제 / `SET NULL` → 자식 FK를 NULL로

---

## 16. DCL

```sql
GRANT SELECT, UPDATE ON EMP TO KIM;                    -- 객체 권한
GRANT SELECT ON EMP TO KIM WITH GRANT OPTION;          -- KIM이 남에게 줄 수 있음
GRANT CREATE TABLE TO KIM WITH ADMIN OPTION;           -- 시스템 권한
REVOKE SELECT ON EMP FROM KIM;
GRANT SELECT ON EMP TO PUBLIC;                         -- 모든 사용자

CREATE ROLE R_DEV;
GRANT CREATE SESSION, CREATE TABLE TO R_DEV;
GRANT R_DEV TO KIM;
```

| 옵션 | 대상 | 회수 시 KIM이 준 권한 |
|---|---|---|
| WITH GRANT OPTION | 객체 권한 | **같이 회수됨** (연쇄) |
| WITH ADMIN OPTION | 시스템 권한, 롤 | 회수 **안 됨** |

---

## 17. 1과목 키워드 한 줄 암기

| 주제 | 암기 |
|---|---|
| 모델링 특징 | 추상화, 단순화, 명확화 |
| 모델링 유의점 | 중복, 비유연성, 비일관성 |
| 모델링 단계 | 개념(추상, 전사) → 논리(정규화, 재사용) → 물리(성능) |
| 3층 스키마 | 외부(사용자, 뷰) / 개념(통합) / 내부(물리 저장) |
| 독립성 | 논리적 독립성(개념 변경 → 외부 영향 X), 물리적 독립성(내부 변경 → 개념 영향 X) |
| 엔터티 특징 | 업무 필요, 식별자, **인스턴스 2개 이상**, 업무 프로세스, **속성 필수**, **관계 1개 이상** |
| 엔터티 유무형 | 유형(사원), 개념(조직), 사건(주문) |
| 엔터티 발생시점 | 기본(사원, 부서), 중심(주문, 계약), 행위(주문목록, 이력) |
| 속성 특성 | 기본, 설계(코드, 일련번호), 파생(계산값) |
| 관계 구성 | 관계명, 차수(1:1, 1:M, M:N), 선택사양(필수/선택) |
| 주식별자 특징 | **유일성, 최소성, 불변성, 존재성** |
| 식별자 분류 | 대표성(주/보조), 생성(내부/외부), 수(단일/복합), 대체(본질/인조) |
| 식별 관계 | 부모 PK → 자식 **PK**, 실선 |
| 비식별 관계 | 부모 PK → 자식 **일반속성**, 점선 |
| 1NF / 2NF / 3NF | 원자값 / 부분 종속 제거 / 이행 종속 제거 |
| BCNF | 모든 결정자가 후보키 |
| 이상 현상 | 삽입 이상, 갱신 이상, 삭제 이상 |
| ACID | 원자성, 일관성, 고립성, 지속성 |
| 인조식별자 단점 | 중복 데이터 발생 가능, 불필요한 인덱스 |

---

## 18. 셀프 퀴즈 (답은 펼쳐서 확인)

<details><summary>Q1. <code>SELECT COUNT(COMM), AVG(COMM) FROM EMP</code> (COMM: 300, 500, 1400, 0, 나머지 NULL)</summary>

`4`, `550`
</details>

<details><summary>Q2. <code>NVL2(NULL, 1, 2)</code>, <code>NULLIF('A', 'A')</code>, <code>COALESCE(NULL, 0, 5)</code></summary>

`2`, `NULL`, `0`
</details>

<details><summary>Q3. <code>SUBSTR('DATABASE', 3, 4)</code></summary>

`'TABA'`
</details>

<details><summary>Q4. <code>ROUND(45.926, -1)</code>, <code>TRUNC(45.926, 2)</code></summary>

`50`, `45.92`
</details>

<details><summary>Q5. SAL이 300, 200, 200, 100일 때 RANK / DENSE_RANK / ROW_NUMBER (내림차순)</summary>

RANK: 1, 2, 2, 4 / DENSE_RANK: 1, 2, 2, 3 / ROW_NUMBER: 1, 2, 3, 4
</details>

<details><summary>Q6. <code>ROLLUP(A, B)</code>에서 A가 3종류, (A, B) 조합이 7개면 결과 행 수는?</summary>

7 + 3 + 1 = `11`
</details>

<details><summary>Q7. <code>WHERE DEPTNO NOT IN (10, NULL)</code>의 결과는?</summary>

`0건`
</details>

<details><summary>Q8. <code>SELECT * FROM EMP WHERE ROWNUM = 2</code>의 결과는?</summary>

`0건`
</details>

<details><summary>Q9. Oracle에서 INSERT → TRUNCATE TABLE 다른테이블 → ROLLBACK 하면 INSERT는?</summary>

남아 있음 (TRUNCATE는 DDL이라 자동 커밋)
</details>

<details><summary>Q10. NATURAL JOIN에서 <code>SELECT E.DEPTNO</code>라고 쓰면?</summary>

에러 (조인 컬럼에 접두사 사용 불가)
</details>

<details><summary>Q11. <code>CUBE(A, B, C)</code>는 몇 가지 그룹 조합?</summary>

2³ = `8`
</details>

<details><summary>Q12. <code>SUM(SAL) OVER (ORDER BY SAL)</code>에서 SAL이 100, 200, 200, 300이면?</summary>

100, 500, 500, 800 (RANGE 기본값이라 같은 값 200은 함께 더해짐)
</details>

<details><summary>Q13. 테이블 구조는 남기고 데이터만 전부 지우되 롤백이 필요 없다면?</summary>

`TRUNCATE TABLE`
</details>

<details><summary>Q14. A가 B에게 WITH GRANT OPTION으로 SELECT 권한을 주고, B가 C에게 줬다. A가 B의 권한을 회수하면 C는?</summary>

C의 권한도 함께 회수됨
</details>

<details><summary>Q15. 주식별자 특징 4가지는?</summary>

유일성, 최소성, 불변성, 존재성
</details>
