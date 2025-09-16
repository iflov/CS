---
tags:
  - database
  - sql
  - crud
  - dml
  - query
created: 2025-01-09
updated: 2025-01-09
aliases:
  - SQL CRUD
  - 데이터 조작어
  - DML
description: SQL의 기본 CRUD(Create, Read, Update, Delete) 작업과 데이터 조작 언어(DML)에 대한 가이드
status: published
category: tutorial
---

# SQL CRUD 작업

> [!info] 개요
> CRUD는 데이터베이스의 기본적인 데이터 처리 기능을 의미합니다. Create(생성), Read(읽기), Update(수정), Delete(삭제)의 약자로, 대부분의 애플리케이션이 필요로 하는 핵심 기능입니다.

## 📑 목차

- [[#🎯 CRUD란?]]
- [[#📝 CREATE - INSERT 문]]
- [[#🔍 READ - SELECT 문]]
- [[#✏️ UPDATE - UPDATE 문]]
- [[#🗑️ DELETE - DELETE 문]]
- [[#🎨 고급 쿼리 기법]]
- [[#⚡ 성능 최적화]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## 🎯 CRUD란?

> [!note] CRUD 정의
> 대부분의 컴퓨터 소프트웨어가 가지는 기본적인 데이터 처리 기능입니다. 
> 데이터베이스뿐만 아니라 RESTful API, 파일 시스템 등에서도 동일한 개념이 사용됩니다.

### CRUD와 SQL 매핑

| CRUD 작업 | SQL 명령어 | HTTP Method | 설명 |
|----------|-----------|-------------|------|
| **Create** | INSERT | POST | 새로운 데이터 생성 |
| **Read** | SELECT | GET | 데이터 조회 |
| **Update** | UPDATE | PUT/PATCH | 기존 데이터 수정 |
| **Delete** | DELETE | DELETE | 데이터 삭제 |

### SQL 명령어 분류

```
SQL 명령어
├── DDL (Data Definition Language)
│   ├── CREATE TABLE
│   ├── ALTER TABLE
│   └── DROP TABLE
├── DML (Data Manipulation Language) ← CRUD
│   ├── INSERT
│   ├── SELECT
│   ├── UPDATE
│   └── DELETE
├── DCL (Data Control Language)
│   ├── GRANT
│   └── REVOKE
└── TCL (Transaction Control Language)
    ├── COMMIT
    ├── ROLLBACK
    └── SAVEPOINT
```

---

## 📝 CREATE - INSERT 문

> [!info] INSERT 문
> 테이블에 새로운 레코드를 추가하는 명령어입니다.

### 기본 문법

```sql
-- 모든 컬럼에 값 입력
INSERT INTO 테이블명 
VALUES (값1, 값2, 값3, ...);

-- 특정 컬럼만 지정하여 입력
INSERT INTO 테이블명 (컬럼1, 컬럼2, 컬럼3) 
VALUES (값1, 값2, 값3);

-- 여러 행 한번에 입력
INSERT INTO 테이블명 (컬럼1, 컬럼2) 
VALUES 
    (값1, 값2),
    (값3, 값4),
    (값5, 값6);
```

### 실제 예시

> [!example] INSERT 사용 예시
> ```sql
> -- 학생 테이블에 데이터 추가
> INSERT INTO students (student_id, name, age, major) 
> VALUES (1001, '김철수', 22, '컴퓨터공학');
> 
> -- 여러 학생 한번에 추가
> INSERT INTO students (student_id, name, age, major) 
> VALUES 
>     (1002, '이영희', 21, '경영학'),
>     (1003, '박민수', 23, '전자공학'),
>     (1004, '정수진', 20, '컴퓨터공학');
> 
> -- 다른 테이블에서 데이터 복사
> INSERT INTO graduated_students 
> SELECT * FROM students 
> WHERE graduation_year = 2024;
> ```

### Bulk Insert (대량 삽입)

> [!tip] 대량 데이터 삽입 최적화
> 많은 양의 데이터를 삽입할 때는 다음 방법들을 사용합니다:

```sql
-- 1. 트랜잭션으로 묶기
START TRANSACTION;
INSERT INTO users VALUES (...);
INSERT INTO users VALUES (...);
-- ... 수천 개의 INSERT
COMMIT;

-- 2. CSV 파일 로드 (MySQL)
LOAD DATA INFILE '/path/to/file.csv'
INTO TABLE users
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;

-- 3. 다중 값 INSERT
INSERT INTO users (name, email) VALUES
    ('user1', 'user1@email.com'),
    ('user2', 'user2@email.com'),
    -- ... 최대 1000개 정도
    ('user1000', 'user1000@email.com');
```

---

## 🔍 READ - SELECT 문

> [!info] SELECT 문
> 데이터베이스에서 가장 많이 사용되는 명령어로, 데이터를 조회합니다.

### 기본 문법

```sql
SELECT [DISTINCT] 컬럼1, 컬럼2, ...
FROM 테이블명
[WHERE 조건]
[GROUP BY 컬럼]
[HAVING 그룹조건]
[ORDER BY 컬럼 [ASC|DESC]]
[LIMIT 개수];
```

### SELECT 절의 다양한 활용

> [!example] 기본 조회
> ```sql
> -- 모든 컬럼 조회
> SELECT * FROM students;
> 
> -- 특정 컬럼만 조회
> SELECT name, age, major FROM students;
> 
> -- 별칭(Alias) 사용
> SELECT 
>     name AS '학생이름',
>     age AS '나이',
>     major AS '전공'
> FROM students;
> 
> -- 중복 제거
> SELECT DISTINCT major FROM students;
> ```

### WHERE 절 - 조건 필터링

```sql
-- 비교 연산자
SELECT * FROM students WHERE age > 20;
SELECT * FROM students WHERE major = '컴퓨터공학';
SELECT * FROM students WHERE age BETWEEN 20 AND 25;

-- IN 연산자
SELECT * FROM students 
WHERE major IN ('컴퓨터공학', '전자공학', '소프트웨어공학');

-- LIKE 연산자 (패턴 매칭)
SELECT * FROM students WHERE name LIKE '김%';  -- 김으로 시작
SELECT * FROM students WHERE email LIKE '%@gmail.com';  -- gmail 사용자
SELECT * FROM students WHERE phone LIKE '010-____-____';  -- 전화번호 패턴

-- NULL 체크
SELECT * FROM students WHERE advisor_id IS NULL;
SELECT * FROM students WHERE advisor_id IS NOT NULL;

-- 복합 조건
SELECT * FROM students 
WHERE age >= 20 AND major = '컴퓨터공학' 
   OR (age < 20 AND gpa > 3.5);
```

### 집계 함수

> [!tip] 주요 집계 함수
> ```sql
> -- COUNT: 개수 세기
> SELECT COUNT(*) FROM students;
> SELECT COUNT(DISTINCT major) FROM students;
> 
> -- SUM: 합계
> SELECT SUM(credits) FROM courses;
> 
> -- AVG: 평균
> SELECT AVG(age) FROM students;
> 
> -- MAX/MIN: 최대/최소값
> SELECT MAX(gpa), MIN(gpa) FROM students;
> 
> -- GROUP BY와 함께 사용
> SELECT major, COUNT(*) as student_count, AVG(gpa) as avg_gpa
> FROM students
> GROUP BY major
> HAVING COUNT(*) > 10;  -- 10명 이상인 전공만
> ```

### ORDER BY - 정렬

```sql
-- 오름차순 정렬 (기본값)
SELECT * FROM students ORDER BY name;
SELECT * FROM students ORDER BY name ASC;

-- 내림차순 정렬
SELECT * FROM students ORDER BY age DESC;

-- 다중 정렬
SELECT * FROM students 
ORDER BY major ASC, gpa DESC, name ASC;

-- NULL 값 처리
SELECT * FROM students 
ORDER BY advisor_id NULLS FIRST;  -- NULL을 먼저
```

---

## ✏️ UPDATE - UPDATE 문

> [!warning] UPDATE 주의사항
> WHERE 절을 빠뜨리면 모든 레코드가 수정됩니다!

### 기본 문법

```sql
UPDATE 테이블명
SET 컬럼1 = 값1,
    컬럼2 = 값2,
    ...
WHERE 조건;
```

### UPDATE 예시

> [!example] UPDATE 사용 예시
> ```sql
> -- 단일 레코드 수정
> UPDATE students 
> SET age = 23 
> WHERE student_id = 1001;
> 
> -- 여러 컬럼 동시 수정
> UPDATE students 
> SET age = age + 1, 
>     year = year + 1
> WHERE status = 'active';
> 
> -- 조건부 수정 (CASE WHEN)
> UPDATE students 
> SET scholarship = 
>     CASE 
>         WHEN gpa >= 4.0 THEN 'full'
>         WHEN gpa >= 3.5 THEN 'half'
>         ELSE 'none'
>     END
> WHERE year = 4;
> 
> -- 다른 테이블 참조하여 수정
> UPDATE students s
> SET s.advisor_name = (
>     SELECT p.name 
>     FROM professors p 
>     WHERE p.id = s.advisor_id
> );
> ```

### 안전한 UPDATE 패턴

```sql
-- 1. 먼저 SELECT로 확인
SELECT * FROM students 
WHERE major = '컴퓨터공학' AND year = 3;

-- 2. 트랜잭션 사용
START TRANSACTION;
UPDATE students SET year = 4 
WHERE major = '컴퓨터공학' AND year = 3;
-- 결과 확인
SELECT * FROM students WHERE major = '컴퓨터공학';
-- 문제없으면 COMMIT, 문제있으면 ROLLBACK
COMMIT;  -- 또는 ROLLBACK;
```

---

## 🗑️ DELETE - DELETE 문

> [!danger] DELETE 주의사항
> WHERE 절 없이 실행하면 테이블의 모든 데이터가 삭제됩니다!
> 중요한 데이터는 반드시 백업 후 삭제하세요.

### 기본 문법

```sql
DELETE FROM 테이블명
WHERE 조건;
```

### DELETE 예시

> [!example] DELETE 사용 예시
> ```sql
> -- 단일 레코드 삭제
> DELETE FROM students 
> WHERE student_id = 1001;
> 
> -- 조건에 맞는 여러 레코드 삭제
> DELETE FROM students 
> WHERE graduation_year = 2020 AND status = 'graduated';
> 
> -- 서브쿼리 사용
> DELETE FROM enrollments 
> WHERE student_id IN (
>     SELECT student_id 
>     FROM students 
>     WHERE status = 'expelled'
> );
> ```

### DELETE vs TRUNCATE vs DROP

| 명령어 | 설명 | 롤백 가능 | 속도 | 사용 시기 |
|--------|------|----------|------|----------|
| DELETE | 특정 행 삭제 | 가능 | 느림 | 조건부 삭제 |
| TRUNCATE | 모든 행 삭제 | 불가능 | 빠름 | 테이블 초기화 |
| DROP | 테이블 자체 삭제 | 불가능 | 빠름 | 테이블 제거 |

```sql
-- DELETE: 로그 남김, 느림, 롤백 가능
DELETE FROM students;

-- TRUNCATE: 로그 최소, 빠름, 롤백 불가
TRUNCATE TABLE students;

-- DROP: 테이블 구조까지 삭제
DROP TABLE students;
```

---

## 🎨 고급 쿼리 기법

### CASE WHEN 구문

> [!tip] 조건부 로직
> ```sql
> SELECT 
>     name,
>     age,
>     CASE 
>         WHEN age < 20 THEN '10대'
>         WHEN age < 30 THEN '20대'
>         WHEN age < 40 THEN '30대'
>         ELSE '40대 이상'
>     END AS age_group
> FROM users;
> ```

### COALESCE - NULL 처리

```sql
-- NULL을 기본값으로 대체
SELECT 
    name,
    COALESCE(phone, email, 'No Contact') AS contact
FROM users;

-- NULL을 0으로 처리하여 계산
SELECT 
    product,
    COALESCE(quantity, 0) * COALESCE(price, 0) AS total
FROM orders;
```

### 서브쿼리

```sql
-- WHERE 절 서브쿼리
SELECT * FROM students
WHERE age > (SELECT AVG(age) FROM students);

-- FROM 절 서브쿼리 (인라인 뷰)
SELECT * FROM (
    SELECT major, COUNT(*) as cnt
    FROM students
    GROUP BY major
) AS major_counts
WHERE cnt > 10;

-- SELECT 절 서브쿼리 (스칼라 서브쿼리)
SELECT 
    name,
    (SELECT COUNT(*) FROM enrollments e 
     WHERE e.student_id = s.student_id) AS course_count
FROM students s;
```

---

## ⚡ 성능 최적화

### 쿼리 성능 개선 팁

> [!success] 최적화 체크리스트
> 1. **SELECT * 피하기**: 필요한 컬럼만 선택
> 2. **인덱스 활용**: WHERE 절에 사용되는 컬럼에 인덱스
> 3. **LIMIT 사용**: 필요한 만큼만 가져오기
> 4. **JOIN 최소화**: 필요한 경우만 JOIN
> 5. **서브쿼리 → JOIN**: 가능하면 JOIN으로 변경

### EXPLAIN - 실행 계획 분석

```sql
-- 쿼리 실행 계획 확인
EXPLAIN SELECT * FROM students WHERE major = '컴퓨터공학';

-- 상세 실행 계획 (MySQL)
EXPLAIN ANALYZE SELECT * FROM students WHERE major = '컴퓨터공학';
```

---

## ⚠️ 주의사항

> [!warning] 흔한 실수들
> 1. **WHERE 절 누락**: UPDATE/DELETE 시 전체 데이터 변경
> 2. **트랜잭션 미사용**: 중요한 작업은 트랜잭션으로 보호
> 3. **SQL Injection**: 사용자 입력 직접 연결 금지
> 4. **N+1 문제**: 반복문 내 쿼리 실행
> 5. **대소문자 구분**: 데이터베이스별로 다름

### SQL Injection 방지

```sql
-- 위험한 방법 (절대 사용 금지!)
String query = "SELECT * FROM users WHERE name = '" + userName + "'";

-- 안전한 방법 (Prepared Statement)
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM users WHERE name = ?"
);
pstmt.setString(1, userName);
```

---

## 📚 참고자료

### 내부 문서
- [[02_rdbms-basics|RDBMS 기초]]
- [[04_keys-and-constraints|키와 제약조건]]
- [[07_database-index|데이터베이스 인덱스]]
- [[09_join-operations|JOIN 연산]]

### 외부 자료
- [W3Schools SQL Tutorial](https://www.w3schools.com/sql/)
- [SQL Style Guide](https://www.sqlstyle.guide/)
- [Use The Index, Luke!](https://use-the-index-luke.com/)

### 연습 사이트
- [SQLiteOnline](https://sqliteonline.com/)
- [HackerRank SQL](https://www.hackerrank.com/domains/sql)
- [LeetCode Database](https://leetcode.com/problemset/database/)

---

> [!quote]
> "좋은 SQL을 작성하는 것은 단순히 문법을 아는 것이 아니라, 데이터베이스가 어떻게 동작하는지 이해하는 것이다."