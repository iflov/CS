---
tags:
  - database
  - sql
  - join
  - query
  - relational
created: 2025-01-08
updated: 2025-01-08
---

# JOIN 연산

> [!info] 개요
> JOIN은 두 개 이상의 테이블에서 관련된 데이터를 결합하여 조회하는 SQL의 핵심 기능입니다. 다양한 JOIN 유형과 최적화 기법을 학습합니다.

## 📑 목차

- [[#🎯 JOIN 기본 개념]]
- [[#🔗 INNER JOIN]]
- [[#⬅️ LEFT JOIN]]
- [[#➡️ RIGHT JOIN]]
- [[#🔄 FULL OUTER JOIN]]
- [[#✖️ CROSS JOIN]]
- [[#🔁 SELF JOIN]]
- [[#⚡ JOIN 최적화]]
- [[#📚 참고자료]]

---

## 🎯 JOIN 기본 개념

### JOIN이란?

> [!note] JOIN 정의
> JOIN은 관계형 데이터베이스에서 정규화된 테이블들을 연결하여 필요한 정보를 조합하는 연산입니다.

### JOIN 종류 개요

```sql
-- 샘플 테이블 생성
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id INT
);

CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50)
);

-- 샘플 데이터
INSERT INTO employees VALUES 
(1, '김철수', 10),
(2, '이영희', 20),
(3, '박민수', 10),
(4, '정지원', NULL),  -- 부서 미배정
(5, '최현우', 30);    -- 존재하지 않는 부서

INSERT INTO departments VALUES 
(10, '개발팀'),
(20, '마케팅팀'),
(40, '인사팀');  -- 직원이 없는 부서
```

---

## 🔗 INNER JOIN

### INNER JOIN 개념

> [!tip] INNER JOIN
> 두 테이블에서 조인 조건을 만족하는 행만 반환합니다.

```sql
-- INNER JOIN 기본 문법
SELECT 
    e.emp_name,
    d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;

-- 결과: 김철수-개발팀, 이영희-마케팅팀, 박민수-개발팀
-- 제외: 정지원(부서 NULL), 최현우(부서 30 없음), 인사팀(직원 없음)

-- 다른 문법 (WHERE 절 사용)
SELECT 
    e.emp_name,
    d.dept_name
FROM employees e, departments d
WHERE e.dept_id = d.dept_id;
```

### 다중 테이블 INNER JOIN

```sql
-- 3개 이상 테이블 조인
CREATE TABLE projects (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(100),
    dept_id INT
);

SELECT 
    e.emp_name,
    d.dept_name,
    p.project_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
INNER JOIN projects p ON d.dept_id = p.dept_id
WHERE p.status = 'active';

-- 조인 조건과 필터 조건 구분
SELECT 
    e.emp_name,
    d.dept_name
FROM employees e
INNER JOIN departments d 
    ON e.dept_id = d.dept_id      -- 조인 조건
    AND d.location = 'Seoul'      -- 조인 시 필터
WHERE e.salary > 50000;           -- 결과 필터
```

---

## ⬅️ LEFT JOIN

### LEFT JOIN (LEFT OUTER JOIN)

> [!note] LEFT JOIN
> 왼쪽 테이블의 모든 행과 오른쪽 테이블의 매칭되는 행을 반환합니다.

```sql
-- LEFT JOIN: 모든 직원 + 부서 정보
SELECT 
    e.emp_name,
    COALESCE(d.dept_name, '부서 미배정') as dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;

-- 결과: 모든 직원 포함 (정지원-부서 미배정, 최현우-NULL)

-- LEFT JOIN으로 부서 없는 직원 찾기
SELECT 
    e.emp_name,
    e.emp_id
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL;
-- 결과: 정지원(NULL), 최현우(존재하지 않는 부서)
```

### LEFT JOIN 활용 패턴

```sql
-- 주문하지 않은 고객 찾기
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) as order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id
HAVING COUNT(o.order_id) = 0;

-- 리뷰 없는 상품 포함 모든 상품 조회
SELECT 
    p.product_id,
    p.product_name,
    AVG(r.rating) as avg_rating,
    COUNT(r.review_id) as review_count
FROM products p
LEFT JOIN reviews r ON p.product_id = r.product_id
GROUP BY p.product_id;
```

---

## ➡️ RIGHT JOIN

### RIGHT JOIN (RIGHT OUTER JOIN)

> [!tip] RIGHT JOIN
> 오른쪽 테이블의 모든 행과 왼쪽 테이블의 매칭되는 행을 반환합니다.

```sql
-- RIGHT JOIN: 모든 부서 + 직원 정보
SELECT 
    COALESCE(e.emp_name, '직원 없음') as emp_name,
    d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;

-- 결과: 모든 부서 포함 (인사팀-직원 없음)

-- RIGHT JOIN은 LEFT JOIN으로 대체 가능
SELECT 
    COALESCE(e.emp_name, '직원 없음') as emp_name,
    d.dept_name
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id;
```

---

## 🔄 FULL OUTER JOIN

### FULL OUTER JOIN

> [!warning] FULL OUTER JOIN
> MySQL은 FULL OUTER JOIN을 직접 지원하지 않습니다. UNION을 사용해 구현합니다.

```sql
-- MySQL에서 FULL OUTER JOIN 구현
SELECT 
    e.emp_name,
    d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
UNION
SELECT 
    e.emp_name,
    d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;

-- PostgreSQL, SQL Server는 지원
-- SELECT * FROM employees e
-- FULL OUTER JOIN departments d ON e.dept_id = d.dept_id;
```

---

## ✖️ CROSS JOIN

### CROSS JOIN (Cartesian Product)

> [!danger] CROSS JOIN 주의
> 두 테이블의 모든 행을 조합합니다. 결과 행 수 = 테이블1 행 수 × 테이블2 행 수

```sql
-- CROSS JOIN: 모든 조합
SELECT 
    e.emp_name,
    d.dept_name
FROM employees e
CROSS JOIN departments d;
-- 결과: 5명 × 3부서 = 15개 행

-- 실용적인 CROSS JOIN 예제: 날짜 캘린더 생성
CREATE TABLE months (month_num INT);
CREATE TABLE days (day_num INT);

INSERT INTO months VALUES (1),(2),(3),(4),(5),(6),(7),(8),(9),(10),(11),(12);
INSERT INTO days VALUES (1),(2),(3),...,(31);

SELECT 
    CONCAT(2024, '-', LPAD(m.month_num, 2, '0'), '-', LPAD(d.day_num, 2, '0')) as date
FROM months m
CROSS JOIN days d
WHERE (m.month_num IN (1,3,5,7,8,10,12) AND d.day_num <= 31)
   OR (m.month_num IN (4,6,9,11) AND d.day_num <= 30)
   OR (m.month_num = 2 AND d.day_num <= 29);
```

---

## 🔁 SELF JOIN

### SELF JOIN

> [!example] SELF JOIN
> 같은 테이블을 두 번 참조하여 조인합니다.

```sql
-- 직원-관리자 관계
CREATE TABLE employees_hierarchy (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    manager_id INT
);

-- SELF JOIN: 직원과 관리자 정보
SELECT 
    e1.emp_name as employee,
    e2.emp_name as manager
FROM employees_hierarchy e1
LEFT JOIN employees_hierarchy e2 ON e1.manager_id = e2.emp_id;

-- 같은 부서 직원 쌍 찾기
SELECT 
    e1.emp_name as employee1,
    e2.emp_name as employee2,
    e1.dept_id
FROM employees e1
INNER JOIN employees e2 
    ON e1.dept_id = e2.dept_id 
    AND e1.emp_id < e2.emp_id;  -- 중복 제거

-- 계층 구조 쿼리 (3단계)
SELECT 
    e1.emp_name as level1,
    e2.emp_name as level2,
    e3.emp_name as level3
FROM employees_hierarchy e1
LEFT JOIN employees_hierarchy e2 ON e2.manager_id = e1.emp_id
LEFT JOIN employees_hierarchy e3 ON e3.manager_id = e2.emp_id
WHERE e1.manager_id IS NULL;  -- 최상위부터 시작
```

---

## ⚡ JOIN 최적화

### JOIN 성능 최적화

> [!tip] JOIN 최적화 전략
> 1. 적절한 인덱스 생성
> 2. JOIN 순서 최적화
> 3. 필요한 컬럼만 SELECT
> 4. WHERE 조건 활용

```sql
-- 인덱스 생성
CREATE INDEX idx_emp_dept ON employees(dept_id);
CREATE INDEX idx_dept_id ON departments(dept_id);

-- 효율적인 JOIN
EXPLAIN SELECT 
    e.emp_name,
    d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE d.location = 'Seoul';  -- 작은 결과셋부터 필터

-- 비효율적인 JOIN
SELECT * 
FROM large_table1 l1
CROSS JOIN large_table2 l2  -- 카테시안 곱
WHERE l1.id = l2.id;  -- 나중에 필터

-- 효율적으로 수정
SELECT * 
FROM large_table1 l1
INNER JOIN large_table2 l2 ON l1.id = l2.id;
```

### JOIN 알고리즘

```sql
-- Nested Loop Join
-- 작은 테이블을 외부 루프로
SELECT /*+ USE_NL(e d) */ 
    e.*, d.*
FROM employees e, departments d
WHERE e.dept_id = d.dept_id;

-- Hash Join (MySQL 8.0+)
-- 동등 조인에 효율적
SELECT /*+ HASH_JOIN(e d) */
    e.*, d.*
FROM employees e, departments d
WHERE e.dept_id = d.dept_id;

-- Sort Merge Join
-- 정렬된 데이터에 효율적
SELECT /*+ USE_MERGE(e d) */
    e.*, d.*
FROM employees e, departments d
WHERE e.dept_id = d.dept_id;
```

### Anti-Join 패턴

```sql
-- NOT EXISTS (효율적)
SELECT *
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);

-- LEFT JOIN + IS NULL (효율적)
SELECT c.*
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;

-- NOT IN (비효율적, NULL 주의)
SELECT *
FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id FROM orders
    WHERE customer_id IS NOT NULL
);
```

### Semi-Join 패턴

```sql
-- EXISTS (효율적)
SELECT *
FROM products p
WHERE EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.product_id
    AND oi.order_date >= '2024-01-01'
);

-- IN with subquery
SELECT *
FROM products
WHERE product_id IN (
    SELECT DISTINCT product_id
    FROM order_items
    WHERE order_date >= '2024-01-01'
);
```

---

## 📚 참고자료

### 내부 문서
- [[03_sql-crud-operations|SQL CRUD 작업]]
- [[05_relationships-and-erd|관계와 ERD]]
- [[07_advanced-sql-queries|고급 SQL 쿼리]]
- [[08_database-index|데이터베이스 인덱스]]

### 외부 리소스
- [Visual JOIN Explanations](https://www.codeproject.com/Articles/33052/Visual-Representation-of-SQL-Joins)
- [MySQL JOIN Optimization](https://dev.mysql.com/doc/refman/8.0/en/join-optimization.html)
- [PostgreSQL JOIN Types](https://www.postgresql.org/docs/current/queries-table-expressions.html)

### 추가 학습 주제
- Natural JOIN
- Using 절
- Lateral JOIN
- Window Functions와 JOIN
- Recursive CTE와 계층 쿼리

---

> [!quote]
> "JOIN은 관계형 데이터베이스의 강력함을 보여주는 핵심 기능입니다. 올바른 JOIN 사용은 복잡한 데이터 관계를 우아하게 표현할 수 있게 합니다."