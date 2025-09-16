---
tags:
  - database
  - normalization
  - database-design
  - normal-form
  - denormalization
created: 2025-01-08
updated: 2025-01-08
---

# 정규화(Normalization)

> [!info] 개요
> 정규화는 데이터 중복을 최소화하고 데이터 무결성을 보장하기 위한 데이터베이스 설계 기법입니다. 이상 현상을 방지하고 효율적인 데이터 구조를 만드는 과정을 학습합니다.

## 📑 목차

- [[#🎯 정규화 개념]]
- [[#⚠️ 이상 현상]]
- [[#1️⃣ 제1정규형]]
- [[#2️⃣ 제2정규형]]
- [[#3️⃣ 제3정규형]]
- [[#🔄 BCNF]]
- [[#📊 고급 정규형]]
- [[#⬇️ 역정규화]]
- [[#📚 참고자료]]

---

## 🎯 정규화 개념

### 정규화란?

> [!note] 정규화 정의
> 정규화는 관계형 데이터베이스에서 중복을 최소화하고 데이터 무결성을 향상시키기 위해 데이터를 구조화하는 프로세스입니다.

### 정규화의 목적

| 목적 | 설명 |
|------|------|
| **데이터 중복 최소화** | 저장 공간 절약, 일관성 유지 |
| **이상 현상 방지** | 삽입, 갱신, 삭제 이상 제거 |
| **데이터 무결성 보장** | 데이터 정확성과 일관성 유지 |
| **확장성 향상** | 구조 변경 용이성 증가 |

### 함수적 종속성

```sql
-- 함수적 종속성: X → Y
-- X가 결정되면 Y도 유일하게 결정됨

-- 예: student_id → student_name
-- 학번이 결정되면 학생 이름도 유일하게 결정됨

-- 완전 함수적 종속
-- (student_id, course_id) → grade
-- 학번과 과목 코드가 모두 있어야 성적이 결정됨

-- 부분 함수적 종속
-- (student_id, course_id) → student_name
-- 학번만으로도 학생 이름이 결정됨 (부적절)
```

---

## ⚠️ 이상 현상

### 이상 현상의 종류

> [!warning] 정규화되지 않은 테이블의 문제점
> 정규화되지 않은 테이블에서 발생하는 데이터 조작 시 문제점들

```sql
-- 비정규화된 테이블 예제
CREATE TABLE student_courses (
    student_id INT,
    student_name VARCHAR(50),
    student_dept VARCHAR(50),
    course_id VARCHAR(10),
    course_name VARCHAR(100),
    professor VARCHAR(50),
    grade VARCHAR(2)
);

-- 샘플 데이터
INSERT INTO student_courses VALUES
(1001, '김철수', '컴퓨터공학', 'CS101', '데이터베이스', '이교수', 'A'),
(1001, '김철수', '컴퓨터공학', 'CS102', '알고리즘', '박교수', 'B+'),
(1002, '이영희', '컴퓨터공학', 'CS101', '데이터베이스', '이교수', 'A+');
```

### 삽입 이상 (Insertion Anomaly)

```sql
-- 문제: 아직 수강신청을 하지 않은 학생 정보를 추가할 수 없음
-- course_id, course_name 등이 NULL이 되어야 함
INSERT INTO student_courses (student_id, student_name, student_dept)
VALUES (1003, '박민수', '컴퓨터공학');  -- 불완전한 데이터
```

### 갱신 이상 (Update Anomaly)

```sql
-- 문제: 학생의 학과가 변경되면 여러 행을 수정해야 함
-- 일부만 수정하면 데이터 불일치 발생
UPDATE student_courses 
SET student_dept = '소프트웨어학과'
WHERE student_id = 1001;  -- 2개 행 모두 수정 필요
```

### 삭제 이상 (Deletion Anomaly)

```sql
-- 문제: 과목 수강을 취소하면 학생 정보도 함께 삭제됨
DELETE FROM student_courses 
WHERE student_id = 1002 AND course_id = 'CS101';
-- 이영희가 CS101만 수강한다면 학생 정보 전체가 사라짐
```

---

## 1️⃣ 제1정규형

### 제1정규형 (1NF) 조건

> [!tip] 1NF 요구사항
> - 각 컬럼의 값은 원자값(Atomic Value)이어야 함
> - 반복 그룹이 없어야 함
> - 모든 행은 유일해야 함

### 1NF 위반 예제

```sql
-- ❌ 1NF 위반: 다중값 저장
CREATE TABLE students_bad (
    student_id INT PRIMARY KEY,
    name VARCHAR(50),
    phone_numbers VARCHAR(200)  -- '010-1234-5678, 02-123-4567'
);

-- ❌ 1NF 위반: 반복 그룹
CREATE TABLE orders_bad (
    order_id INT,
    customer VARCHAR(50),
    product1 VARCHAR(50),
    quantity1 INT,
    product2 VARCHAR(50),
    quantity2 INT,
    product3 VARCHAR(50),
    quantity3 INT
);
```

### 1NF 적용

```sql
-- ✅ 1NF 적용: 별도 테이블로 분리
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE student_phones (
    student_id INT,
    phone_number VARCHAR(20),
    phone_type VARCHAR(20),
    PRIMARY KEY (student_id, phone_number),
    FOREIGN KEY (student_id) REFERENCES students(student_id)
);

-- ✅ 1NF 적용: 반복 그룹 제거
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer VARCHAR(50),
    order_date DATE
);

CREATE TABLE order_items (
    order_id INT,
    product VARCHAR(50),
    quantity INT,
    PRIMARY KEY (order_id, product),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

---

## 2️⃣ 제2정규형

### 제2정규형 (2NF) 조건

> [!note] 2NF 요구사항
> - 1NF를 만족해야 함
> - 부분 함수적 종속이 없어야 함
> - 모든 비주요 속성이 기본키에 완전 함수적 종속

### 2NF 위반 예제

```sql
-- ❌ 2NF 위반: 부분 함수적 종속 존재
CREATE TABLE order_details_bad (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),  -- product_id에만 종속
    product_price DECIMAL(10,2), -- product_id에만 종속
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
-- 문제: product_name과 product_price는 product_id에만 종속됨
```

### 2NF 적용

```sql
-- ✅ 2NF 적용: 테이블 분리
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    product_price DECIMAL(10,2)
);

CREATE TABLE order_details (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

---

## 3️⃣ 제3정규형

### 제3정규형 (3NF) 조건

> [!tip] 3NF 요구사항
> - 2NF를 만족해야 함
> - 이행적 함수 종속이 없어야 함
> - 비주요 속성 간에 함수적 종속 관계가 없어야 함

### 3NF 위반 예제

```sql
-- ❌ 3NF 위반: 이행적 함수 종속
CREATE TABLE employees_bad (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id INT,
    dept_name VARCHAR(50),    -- dept_id → dept_name
    dept_location VARCHAR(50)  -- dept_id → dept_location
);
-- 문제: emp_id → dept_id → dept_name (이행적 종속)
```

### 3NF 적용

```sql
-- ✅ 3NF 적용: 테이블 분리
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50),
    dept_location VARCHAR(50)
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);
```

---

## 🔄 BCNF

### Boyce-Codd 정규형

> [!warning] BCNF 조건
> - 3NF를 만족해야 함
> - 모든 결정자가 후보키여야 함

### BCNF 위반 예제

```sql
-- ❌ BCNF 위반 예제
CREATE TABLE course_instructors (
    student_id INT,
    course_id VARCHAR(10),
    instructor VARCHAR(50),
    PRIMARY KEY (student_id, course_id)
);
-- 문제: instructor → course_id (강사가 한 과목만 가르침)
-- instructor는 결정자이지만 후보키가 아님

-- ✅ BCNF 적용
CREATE TABLE courses (
    course_id VARCHAR(10) PRIMARY KEY,
    instructor VARCHAR(50)
);

CREATE TABLE enrollments (
    student_id INT,
    course_id VARCHAR(10),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

---

## 📊 고급 정규형

### 제4정규형 (4NF)

> [!note] 4NF - 다치 종속성 제거
> 다치 종속성(MVD)이 없어야 함

```sql
-- ❌ 4NF 위반: 다치 종속성
CREATE TABLE person_hobbies_languages (
    person_id INT,
    hobby VARCHAR(50),
    language VARCHAR(50),
    PRIMARY KEY (person_id, hobby, language)
);
-- 문제: 취미와 언어가 독립적인데 함께 저장

-- ✅ 4NF 적용: 독립적인 관계 분리
CREATE TABLE person_hobbies (
    person_id INT,
    hobby VARCHAR(50),
    PRIMARY KEY (person_id, hobby)
);

CREATE TABLE person_languages (
    person_id INT,
    language VARCHAR(50),
    PRIMARY KEY (person_id, language)
);
```

### 제5정규형 (5NF)

```sql
-- 5NF: 조인 종속성 제거
-- 모든 조인 종속성이 후보키를 통해서만 성립
-- 실무에서는 거의 고려하지 않음
```

---

## ⬇️ 역정규화

### 역정규화란?

> [!tip] 역정규화 (Denormalization)
> 성능 향상을 위해 의도적으로 정규화 원칙을 위반하는 것

### 역정규화 기법

```sql
-- 1. 테이블 통합
-- 정규화된 테이블
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(50)
);

-- 역정규화: 자주 조인되는 테이블 통합
CREATE TABLE orders_denormalized (
    order_id INT PRIMARY KEY,
    customer_id INT,
    customer_name VARCHAR(50),  -- 중복 저장
    order_date DATE
);

-- 2. 계산된 컬럼 추가
CREATE TABLE orders_with_total (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2)  -- 계산값 저장
);

-- 3. 중복 컬럼 추가
CREATE TABLE products_with_category (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_id INT,
    category_name VARCHAR(50)  -- 조인 회피용 중복
);
```

### 역정규화 트레이드오프

| 장점 | 단점 |
|------|------|
| 조회 성능 향상 | 데이터 중복으로 저장 공간 증가 |
| 조인 감소 | 데이터 일관성 유지 어려움 |
| 집계 연산 최적화 | INSERT/UPDATE 성능 저하 |
| 복잡한 쿼리 단순화 | 데이터 무결성 관리 복잡 |

### 역정규화 사용 시나리오

```sql
-- 리포팅 테이블
CREATE TABLE sales_summary (
    date DATE,
    product_id INT,
    total_quantity INT,
    total_revenue DECIMAL(10,2),
    avg_price DECIMAL(10,2),
    PRIMARY KEY (date, product_id)
);

-- 캐시 테이블
CREATE TABLE user_stats_cache (
    user_id INT PRIMARY KEY,
    total_orders INT,
    total_spent DECIMAL(10,2),
    last_order_date DATE,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 트리거를 이용한 동기화
DELIMITER //
CREATE TRIGGER update_user_stats
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    INSERT INTO user_stats_cache (user_id, total_orders, total_spent)
    VALUES (NEW.customer_id, 1, NEW.total_amount)
    ON DUPLICATE KEY UPDATE
        total_orders = total_orders + 1,
        total_spent = total_spent + NEW.total_amount,
        last_order_date = NEW.order_date,
        updated_at = NOW();
END//
DELIMITER ;
```

---

## 📚 참고자료

### 내부 문서
- [[01_database-fundamentals|데이터베이스 기초]]
- [[05_relationships-and-erd|관계와 ERD]]
- [[08_database-index|데이터베이스 인덱스]]

### 외부 리소스
- [Database Normalization](https://en.wikipedia.org/wiki/Database_normalization)
- [Normalization Tutorial](https://www.studytonight.com/dbms/database-normalization.php)
- [When to Denormalize](https://www.vertabelo.com/blog/denormalization-when-why-and-how/)

### 추가 학습 주제
- Domain-Key Normal Form (DKNF)
- 정규화 자동화 도구
- NoSQL에서의 정규화
- 마이크로서비스 환경에서의 데이터 정규화

---

> [!quote]
> "정규화는 데이터베이스 설계의 기초입니다. 하지만 맹목적인 정규화보다는 비즈니스 요구사항에 맞는 적절한 수준의 정규화가 중요합니다."