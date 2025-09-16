---
tags:
  - database
  - keys
  - constraints
  - primary-key
  - foreign-key
  - data-integrity
created: 2025-01-08
updated: 2025-01-08
---

# 키(Keys)와 제약조건(Constraints)

> [!info] 개요
> 데이터베이스에서 키는 레코드를 고유하게 식별하고, 제약조건은 데이터 무결성을 보장합니다. 이 문서에서는 다양한 키의 종류와 제약조건의 역할을 학습합니다.

## 📑 목차

- [[#🔑 키의 종류]]
- [[#⚡ Primary Key]]
- [[#🔗 Foreign Key]]
- [[#💎 기타 키 유형]]
- [[#🛡️ 제약조건]]
- [[#💡 실전 예제]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## 🔑 키의 종류

> [!note] 키(Key)란?
> 키는 테이블의 레코드를 고유하게 식별하거나 테이블 간의 관계를 설정하는 데 사용되는 속성 또는 속성의 집합입니다.

### 키의 분류

| 키 종류 | 설명 | 특징 |
|--------|------|------|
| **Super Key** | 유일성을 만족하는 속성들의 집합 | 최소성 보장 안 됨 |
| **Candidate Key** | 유일성과 최소성을 만족하는 키 | 기본키 후보 |
| **Primary Key** | 선택된 후보키 | NULL 불가, 유일성 보장 |
| **Alternate Key** | 선택되지 않은 후보키 | 보조 식별자 |
| **Foreign Key** | 다른 테이블의 기본키를 참조 | 관계 설정 |
| **Composite Key** | 2개 이상의 컬럼으로 구성된 키 | 복합 유일성 |

---

## ⚡ Primary Key

### 기본키의 특징

> [!tip] Primary Key 원칙
> 1. **유일성(Uniqueness)**: 중복된 값 불허
> 2. **NOT NULL**: NULL 값 불허
> 3. **불변성(Immutability)**: 한 번 설정되면 변경 최소화
> 4. **단일성**: 테이블당 하나만 존재

### Primary Key 생성

```sql
-- 테이블 생성 시 지정
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE
);

-- 복합 기본키
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- ALTER TABLE로 추가
ALTER TABLE employees
ADD PRIMARY KEY (employee_id);
```

### Auto Increment

```sql
-- MySQL
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);

-- PostgreSQL
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);
```

---

## 🔗 Foreign Key

### 외래키의 역할

> [!example] Foreign Key 개념
> 외래키는 참조 무결성(Referential Integrity)을 보장하며, 테이블 간의 관계를 정의합니다.

### Foreign Key 생성

```sql
-- 테이블 생성 시 외래키 지정
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) 
    REFERENCES customers(customer_id)
);

-- 명시적 제약조건 이름 지정
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    CONSTRAINT fk_customer
    FOREIGN KEY (customer_id) 
    REFERENCES customers(customer_id)
    ON DELETE CASCADE
    ON UPDATE CASCADE
);
```

### 참조 동작 옵션

| 옵션 | ON DELETE | ON UPDATE | 설명 |
|------|-----------|-----------|------|
| **CASCADE** | 부모 삭제 시 자식도 삭제 | 부모 수정 시 자식도 수정 | 연쇄 작업 |
| **SET NULL** | 부모 삭제 시 자식을 NULL로 | 부모 수정 시 자식을 NULL로 | NULL 설정 |
| **SET DEFAULT** | 부모 삭제 시 기본값으로 | 부모 수정 시 기본값으로 | 기본값 설정 |
| **RESTRICT** | 자식이 있으면 삭제 불가 | 자식이 있으면 수정 불가 | 엄격한 제한 |
| **NO ACTION** | RESTRICT와 동일 | RESTRICT와 동일 | 표준 SQL |

---

## 💎 기타 키 유형

### Unique Key

```sql
-- UNIQUE 제약조건
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    social_security_number VARCHAR(20) UNIQUE
);

-- 복합 UNIQUE 키
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    semester VARCHAR(20),
    UNIQUE KEY unique_enrollment (student_id, course_id, semester)
);
```

### Candidate Key 예제

```sql
-- students 테이블의 후보키들
CREATE TABLE students (
    student_id INT,          -- 후보키 1
    social_number VARCHAR(20), -- 후보키 2  
    email VARCHAR(100),       -- 후보키 3
    name VARCHAR(50),
    PRIMARY KEY (student_id),  -- 선택된 기본키
    UNIQUE (social_number),    -- 대체키
    UNIQUE (email)            -- 대체키
);
```

---

## 🛡️ 제약조건

### 제약조건의 종류

> [!warning] 데이터 무결성 보장
> 제약조건은 데이터의 정확성과 일관성을 유지하는 규칙입니다.

| 제약조건 | 설명 | 예제 |
|----------|------|------|
| **NOT NULL** | NULL 값 불허 | `name VARCHAR(50) NOT NULL` |
| **UNIQUE** | 중복 값 불허 | `email VARCHAR(100) UNIQUE` |
| **PRIMARY KEY** | 기본키 지정 | `id INT PRIMARY KEY` |
| **FOREIGN KEY** | 외래키 지정 | `FOREIGN KEY (dept_id) REFERENCES departments(id)` |
| **CHECK** | 조건 검사 | `age INT CHECK (age >= 18)` |
| **DEFAULT** | 기본값 설정 | `status VARCHAR(20) DEFAULT 'active'` |

### 제약조건 구현 예제

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) CHECK (price > 0),
    stock_quantity INT DEFAULT 0,
    category_id INT,
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_category
    FOREIGN KEY (category_id) 
    REFERENCES categories(category_id)
);

-- 제약조건 추가
ALTER TABLE products
ADD CONSTRAINT check_stock 
CHECK (stock_quantity >= 0);

-- 제약조건 삭제
ALTER TABLE products
DROP CONSTRAINT check_stock;
```

---

## 💡 실전 예제

### 전자상거래 데이터베이스 설계

> [!example] 실무 적용 사례

```sql
-- 카테고리 테이블
CREATE TABLE categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(50) NOT NULL UNIQUE,
    parent_category_id INT,
    FOREIGN KEY (parent_category_id) 
    REFERENCES categories(category_id)
);

-- 제품 테이블
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_code VARCHAR(20) UNIQUE NOT NULL,
    product_name VARCHAR(100) NOT NULL,
    category_id INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL CHECK (price > 0),
    stock INT DEFAULT 0 CHECK (stock >= 0),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) 
    REFERENCES categories(category_id)
);

-- 고객 테이블
CREATE TABLE customers (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_email (email)
);

-- 주문 테이블
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    order_number VARCHAR(20) UNIQUE NOT NULL,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10, 2) NOT NULL,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') 
           DEFAULT 'pending',
    FOREIGN KEY (customer_id) 
    REFERENCES customers(customer_id)
);

-- 주문 상세 테이블 (복합키 사용)
CREATE TABLE order_details (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,
    discount_percent DECIMAL(5, 2) DEFAULT 0 
                    CHECK (discount_percent >= 0 AND discount_percent <= 100),
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) 
    REFERENCES orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) 
    REFERENCES products(product_id)
);
```

---

## ⚠️ 주의사항

> [!danger] 키 설계 시 주의점
> - 기본키는 비즈니스 로직과 독립적인 값 사용 권장
> - 외래키 순환 참조 방지
> - 복합키는 최소한의 컬럼으로 구성
> - NULL 가능한 컬럼은 기본키로 사용 불가

### 안티패턴 예제

```sql
-- ❌ 잘못된 예: 변경 가능한 비즈니스 값을 기본키로 사용
CREATE TABLE users (
    email VARCHAR(100) PRIMARY KEY,  -- 이메일은 변경될 수 있음
    name VARCHAR(50)
);

-- ✅ 올바른 예: 불변의 대리키 사용
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(50)
);
```

---

## 📚 참고자료

### 내부 문서
- [[01_database-fundamentals|데이터베이스 기초]]
- [[02_rdbms-basics|RDBMS 기본 개념]]
- [[05_relationships-and-erd|관계와 ERD]]

### 외부 리소스
- [MySQL Constraints Documentation](https://dev.mysql.com/doc/refman/8.0/en/constraints.html)
- [PostgreSQL Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)
- [Database Design Best Practices](https://www.oracle.com/database/what-is-database-design/)

### 추가 학습
- 인덱스와 키의 관계
- 파티션 키
- 샤딩 키
- 분산 데이터베이스의 키 전략

---

> [!quote]
> "좋은 키 설계는 데이터베이스 성능과 무결성의 기초입니다. 처음부터 올바른 키를 선택하는 것이 나중의 많은 문제를 예방합니다."