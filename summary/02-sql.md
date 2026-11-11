# 2과목. SQL 기본과 활용 (요약)

> SQLD는 주로 **Oracle** 문법 기준. SQL Server와 다른 부분은 따로 표시

## 1장. SQL 기본

### SQL 종류

| 분류 | 명령어 |
|---|---|
| DDL (정의) | CREATE, ALTER, DROP, RENAME, TRUNCATE |
| DML (조작) | SELECT, INSERT, UPDATE, DELETE, MERGE |
| DCL (제어) | GRANT, REVOKE |
| TCL (트랜잭션) | COMMIT, ROLLBACK, SAVEPOINT |

> SELECT를 DQL로 따로 분류하기도 함

### SELECT 실행 순서 (매우 중요)

```
5 SELECT
1 FROM
2 WHERE
3 GROUP BY
4 HAVING
6 ORDER BY
```
- WHERE에서는 SELECT 별칭을 쓸 수 없음. ORDER BY에서는 별칭 사용 가능
- WHERE에는 집계 함수 사용 불가 → HAVING 사용

### 연산자
- 우선순위: `()` → `NOT` → 비교 연산자 → `AND` → `OR`
- `BETWEEN a AND b`: a 이상 b 이하
- `LIKE`: `%` 0개 이상 문자, `_` 정확히 1개 문자
- `IN (a, b)`: a 또는 b
- ⚠️ `NOT IN` 목록에 **NULL이 있으면 결과가 0건**
- 문자열 연결: Oracle `||`, SQL Server `+`

### 단일행 함수

| 종류 | 함수 |
|---|---|
| 문자 | LOWER, UPPER, LENGTH(SQL Server: LEN), SUBSTR(SQL Server: SUBSTRING), CONCAT, LTRIM, RTRIM, TRIM, LPAD, RPAD, REPLACE, INSTR |
| 숫자 | ABS, SIGN, MOD, ROUND, TRUNC, CEIL(SQL Server: CEILING), FLOOR |
| 날짜 | SYSDATE(SQL Server: GETDATE()), ADD_MONTHS, EXTRACT, MONTHS_BETWEEN |
| 변환 | TO_CHAR, TO_NUMBER, TO_DATE / SQL Server: CAST, CONVERT |

- `ROUND(15.67, 1)` = 15.7, `TRUNC(15.67, 1)` = 15.6, `CEIL(15.1)` = 16, `FLOOR(15.9)` = 15
- 날짜 연산: 날짜 + 1 = 하루 뒤, 날짜 + 1/24 = 1시간 뒤, 날짜 - 날짜 = 일수
- 암시적 형변환이 일어나면 인덱스를 못 탈 수 있음

### NULL 관련 함수 (자주 출제)

| 함수 | 결과 |
|---|---|
| `NVL(a, b)` | a가 NULL이면 b (SQL Server: `ISNULL`) |
| `NVL2(a, b, c)` | a가 NULL이 아니면 b, NULL이면 c |
| `NULLIF(a, b)` | a = b이면 NULL, 아니면 a |
| `COALESCE(a, b, c, ...)` | 처음으로 NULL이 아닌 값 |

### CASE / DECODE
```sql
CASE WHEN SAL >= 3000 THEN 'HIGH'
     WHEN SAL >= 1000 THEN 'MID'
     ELSE 'LOW' END

DECODE(DEPTNO, 10, 'A', 20, 'B', 'ETC')   -- Oracle 전용
```
- ELSE를 생략하면 해당 없는 경우 NULL

### 집계 함수와 GROUP BY
- `COUNT(*)`: NULL 포함 전체 행 수 / `COUNT(컬럼)`: NULL 제외
- SUM, AVG, MAX, MIN: NULL 제외하고 계산
  - ⚠️ `AVG(COMM)` ≠ `SUM(COMM) / COUNT(*)` (분모가 다름)
- 결과 행이 하나도 없을 때: `COUNT` → 0, `SUM`/`MAX` 등 → NULL
- GROUP BY를 쓰면 SELECT에는 **GROUP BY 컬럼 또는 집계 함수만** 올 수 있음
- GROUP BY 절에서는 SELECT 별칭 사용 불가

### ORDER BY
- 기본 ASC. 컬럼명, 별칭, 컬럼 위치 번호 사용 가능
- Oracle: NULL이 가장 큰 값 → ASC면 맨 뒤 (`NULLS FIRST/LAST`로 조정)
- SELECT에 없는 컬럼으로도 정렬 가능 (단, GROUP BY가 있으면 GROUP BY 컬럼이나 집계만)

### 조인
- **EQUI 조인:** `=` 조건 / **Non-EQUI 조인:** `BETWEEN`, `>` 등
- 테이블 N개를 조인하려면 조인 조건이 최소 **N-1개**

### 표준(ANSI) 조인

| 조인 | 특징 |
|---|---|
| `INNER JOIN ... ON` | 조건에 맞는 행만 |
| `NATURAL JOIN` | 이름이 같은 모든 컬럼으로 등가 조인. ⚠️ 그 컬럼에 **테이블 별칭 접두사 사용 불가** |
| `JOIN ... USING (컬럼)` | 지정 컬럼으로 등가 조인. ⚠️ USING 컬럼에 **접두사 사용 불가** |
| `CROSS JOIN` | 카티션 곱 (M × N 행) |
| `LEFT / RIGHT / FULL OUTER JOIN` | 기준 쪽 행은 조건이 안 맞아도 모두 출력 |

- Oracle 아우터 조인 `(+)`: **데이터가 없는 쪽**에 붙임
  `WHERE E.DEPTNO(+) = D.DEPTNO` → D 기준 LEFT OUTER JOIN과 같음

---

## 2장. SQL 활용

### 서브쿼리

| 위치 | 이름 | 특징 |
|---|---|---|
| SELECT 절 | 스칼라 서브쿼리 | **한 행, 한 컬럼**만 반환 |
| FROM 절 | 인라인 뷰 | 임시 테이블처럼 사용, ORDER BY 가능 |
| WHERE / HAVING 절 | 중첩 서브쿼리 | 조건으로 사용 |

| 반환 결과 | 사용 연산자 |
|---|---|
| 단일 행 | `=`, `<`, `>`, `<=`, `>=`, `<>` |
| 다중 행 | `IN`, `ANY`(`SOME`), `ALL`, `EXISTS` |
| 다중 컬럼 | `(A, B) IN (SELECT A, B ...)` |

- **연관(Correlated) 서브쿼리:** 서브쿼리 안에서 메인쿼리 컬럼을 사용
- `> ANY` = 최솟값보다 크면 / `> ALL` = 최댓값보다 크면
- `EXISTS`는 결과가 있는지만 확인 (true / false)
- 단일 행 연산자에 여러 행이 반환되면 **에러**

### 뷰 (View)
- 장점: 독립성(테이블 구조가 바뀌어도 응용 프로그램 유지), 편리성(복잡한 쿼리 단순화), 보안성(필요한 컬럼만 노출)
- 실제 데이터를 저장하지 않음 (Materialized View 제외)

### 집합 연산자

| 연산자 | 결과 |
|---|---|
| `UNION` | 합집합, **중복 제거** |
| `UNION ALL` | 합집합, 중복 포함 (정렬 작업 없어 가장 빠름) |
| `INTERSECT` | 교집합 |
| `MINUS` (SQL Server: `EXCEPT`) | 차집합 |

- 위아래 SELECT의 **컬럼 수와 타입**이 맞아야 함
- 결과 컬럼명은 **첫 번째 SELECT** 기준
- ORDER BY는 **맨 마지막에 한 번**만

### 그룹 함수 (소계, 총계)

| 함수 | `(A, B)`를 넣었을 때 집계 단위 |
|---|---|
| `ROLLUP(A, B)` | (A, B), (A), 전체 → **인자 순서 중요** |
| `CUBE(A, B)` | (A, B), (A), (B), 전체 → 모든 조합 |
| `GROUPING SETS(A, B)` | (A), (B) → 각각 따로, 순서 무관, 전체 합계 없음 |

- `GROUPING(컬럼)`: 해당 컬럼이 소계/총계로 집계된 행이면 **1**, 아니면 0
- 활용: `CASE WHEN GROUPING(DEPTNO) = 1 THEN '합계' END`

### 윈도우 함수
```sql
함수() OVER (PARTITION BY 그룹 ORDER BY 정렬 [ROWS | RANGE BETWEEN ... AND ...])
```

| 분류 | 함수 |
|---|---|
| 순위 | RANK, DENSE_RANK, ROW_NUMBER |
| 집계 | SUM, AVG, MAX, MIN, COUNT |
| 행 순서 | FIRST_VALUE, LAST_VALUE, LAG(이전 행), LEAD(다음 행) |
| 비율 | RATIO_TO_REPORT, PERCENT_RANK, CUME_DIST, NTILE |

**순위 함수 비교 (급여 300, 300, 200일 때)**

| 함수 | 결과 |
|---|---|
| RANK | 1, 1, **3** (동순위 다음 건너뜀) |
| DENSE_RANK | 1, 1, **2** (건너뛰지 않음) |
| ROW_NUMBER | 1, 2, 3 (동순위 없음) |

**윈도우 범위**
- `ROWS`: 물리적인 행 기준 / `RANGE`: 값(논리적) 기준
- `UNBOUNDED PRECEDING`(처음), `CURRENT ROW`(현재), `UNBOUNDED FOLLOWING`(끝), `n PRECEDING / FOLLOWING`
- ⚠️ ORDER BY만 있고 범위를 생략하면 기본값은 `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` → **누적 합계**가 나옴

### Top N 쿼리
```sql
-- Oracle ROWNUM: 정렬 후 자르려면 인라인 뷰 필요
SELECT * FROM (SELECT * FROM EMP ORDER BY SAL DESC) WHERE ROWNUM <= 3;

-- Oracle 12c 이상
SELECT * FROM EMP ORDER BY SAL DESC FETCH FIRST 3 ROWS ONLY;

-- SQL Server
SELECT TOP(3) * FROM EMP ORDER BY SAL DESC;   -- WITH TIES: 동점 포함
```
- ⚠️ `ROWNUM = 1`, `ROWNUM <= N`은 되지만 `ROWNUM = 2`, `ROWNUM > 1`은 **결과 0건**
- ROWNUM은 ORDER BY보다 **먼저** 부여됨

### 계층형 질의 (Oracle)
```sql
SELECT LEVEL, LPAD(' ', (LEVEL-1)*2) || ENAME
FROM EMP
START WITH MGR IS NULL              -- 루트 시작 조건
CONNECT BY PRIOR EMPNO = MGR        -- 부모 → 자식 (순방향)
ORDER SIBLINGS BY ENAME;            -- 같은 레벨끼리 정렬
```
- `CONNECT BY PRIOR 자식컬럼 = 부모컬럼` 형태면 **순방향**(위에서 아래로), 반대면 **역방향**
  - 기억법: **PRIOR가 붙은 쪽에서 반대쪽으로 이동**
- `LEVEL`: 루트 = 1
- `CONNECT_BY_ISLEAF`: 리프(자식 없음)면 1
- `CONNECT_BY_ROOT 컬럼`: 루트의 값
- `SYS_CONNECT_BY_PATH(컬럼, '/')`: 루트부터 경로
- `NOCYCLE`, `CONNECT_BY_ISCYCLE`: 순환 구조 처리
- WHERE 절은 계층 전개가 **끝난 뒤** 필터링

### 셀프 조인
- 같은 테이블을 별칭을 다르게 해서 조인 (예: 사원과 관리자)
```sql
SELECT E.ENAME AS 사원, M.ENAME AS 관리자
FROM EMP E LEFT OUTER JOIN EMP M ON E.MGR = M.EMPNO;
```

### PIVOT / UNPIVOT
- `PIVOT`: 행 → 열 (예: 부서별 행을 직무별 컬럼으로)
- `UNPIVOT`: 열 → 행

### 정규 표현식
- 함수: `REGEXP_LIKE`, `REGEXP_REPLACE`, `REGEXP_SUBSTR`, `REGEXP_INSTR`, `REGEXP_COUNT`

| 메타 문자 | 의미 |
|---|---|
| `.` | 아무 문자 1개 |
| `^` / `$` | 시작 / 끝 |
| `*` / `+` / `?` | 0회 이상 / 1회 이상 / 0 또는 1회 |
| `{m,n}` | m회 이상 n회 이하 |
| `[abc]` / `[^abc]` | 셋 중 하나 / 셋 제외 |
| `\d` / `\w` / `\s` | 숫자 / 영문자·숫자·_ / 공백 |
| `\|` | OR |

---

## 3장. 관리 구문

### DML
```sql
INSERT INTO DEPT (DEPTNO, DNAME) VALUES (50, 'IT');
UPDATE EMP SET SAL = SAL * 1.1 WHERE DEPTNO = 10;
DELETE FROM EMP WHERE EMPNO = 7369;

MERGE INTO 대상 T
USING 원본 S ON (T.ID = S.ID)
WHEN MATCHED THEN UPDATE SET T.VAL = S.VAL
WHEN NOT MATCHED THEN INSERT (ID, VAL) VALUES (S.ID, S.VAL);
```

### DELETE vs TRUNCATE vs DROP (단골 문제)

| 구분 | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| 분류 | DML | DDL | DDL |
| 삭제 대상 | 데이터 (WHERE 가능) | 데이터 전체 | 테이블 구조까지 |
| 테이블 구조 | 유지 | 유지 | 삭제 |
| 롤백 (Oracle) | 가능 | 불가 (자동 커밋) | 불가 (자동 커밋) |
| 저장 공간 | 유지 | 해제 | 해제 |

### TCL과 트랜잭션
- `COMMIT`: 확정 / `ROLLBACK`: 취소 / `SAVEPOINT sp1` + `ROLLBACK TO sp1`: 부분 취소
- **Oracle:** DML은 자동 커밋 안 됨 (직접 COMMIT 필요), **DDL을 실행하면 자동 커밋**
- **SQL Server:** 기본이 AUTO COMMIT, DDL도 트랜잭션 안에서 롤백 가능
- 같은 이름의 SAVEPOINT를 다시 만들면 이전 것은 덮어씀
- 고립성이 낮을 때 생기는 문제: Dirty Read, Non-Repeatable Read, Phantom Read

### DDL
```sql
CREATE TABLE T (
  ID   NUMBER(5)    CONSTRAINT PK_T PRIMARY KEY,
  NAME VARCHAR2(20) NOT NULL,
  AGE  NUMBER(3)    CHECK (AGE >= 0),
  DEPTNO NUMBER(2)  REFERENCES DEPT(DEPTNO) ON DELETE CASCADE
);

ALTER TABLE T ADD (EMAIL VARCHAR2(50));
ALTER TABLE T MODIFY (NAME VARCHAR2(30));      -- SQL Server: ALTER COLUMN
ALTER TABLE T DROP COLUMN EMAIL;
ALTER TABLE T RENAME COLUMN NAME TO FULL_NAME;
RENAME T TO T2;                                -- Oracle
```

**제약조건**

| 제약 | 특징 |
|---|---|
| PRIMARY KEY | UNIQUE + NOT NULL, 테이블당 1개 |
| UNIQUE | 중복 불가, **NULL 허용** |
| NOT NULL | NULL 불가 |
| CHECK | 값의 범위/조건 제한 |
| FOREIGN KEY | 다른 테이블 PK 참조. 삭제 옵션: CASCADE, SET NULL, SET DEFAULT, RESTRICT, NO ACTION |

- `CREATE TABLE 새테이블 AS SELECT ...` (CTAS): 데이터와 구조는 복사되지만 제약조건은 **NOT NULL만** 복사
- `CHAR`는 고정 길이(남는 자리 공백), `VARCHAR2`는 가변 길이

### DCL
```sql
GRANT SELECT, INSERT ON EMP TO USER1 WITH GRANT OPTION;
REVOKE SELECT ON EMP FROM USER1;

CREATE ROLE R1;
GRANT CREATE SESSION, CREATE TABLE TO R1;
GRANT R1 TO USER1;
```
- `WITH GRANT OPTION` (객체 권한): 권한을 회수하면 그 사용자가 준 권한도 **연쇄 회수**
- `WITH ADMIN OPTION` (시스템 권한): 연쇄 회수 **안 됨**
- **ROLE:** 여러 권한을 묶어서 관리 (Oracle 기본 롤: CONNECT, RESOURCE, DBA)

---

## 시험 직전 체크 (자주 나오는 함정)
- [ ] `NULL`과의 비교는 `IS NULL`. `= NULL`은 항상 결과 없음
- [ ] `NOT IN` 목록에 NULL이 있으면 결과 0건
- [ ] `COUNT(*)`와 `COUNT(컬럼)`의 차이, AVG의 분모
- [ ] SELECT 실행 순서와 별칭 사용 가능 위치
- [ ] NATURAL JOIN / USING 컬럼에 접두사 사용 불가
- [ ] ROLLUP 인자 순서에 따라 결과가 달라짐
- [ ] RANK / DENSE_RANK / ROW_NUMBER 결과 차이
- [ ] 윈도우 함수에서 범위 생략 시 누적 합계
- [ ] `ROWNUM > 1`, `ROWNUM = 2`는 0건
- [ ] 계층형 질의에서 PRIOR 위치에 따른 방향
- [ ] Oracle은 DDL 실행 시 자동 커밋 → 앞선 DML도 확정됨
- [ ] DELETE / TRUNCATE / DROP 비교
