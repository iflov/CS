---
tags:
  - database
  - rdbms
  - relational-database
  - sql
  - table
created: 2025-01-09
updated: 2025-01-09
aliases:
  - 관계형 데이터베이스
  - RDBMS
  - Relational Database
description: 관계형 데이터베이스(RDBMS)의 기본 개념, 테이블 구조, 관계형 모델의 용어 정리
status: published
category: tutorial
---

# 관계형 데이터베이스 (RDBMS) 기초

> [!info] 개요
> 관계형 데이터베이스(RDBMS)는 데이터를 테이블 형태로 저장하고 관리하는 데이터베이스 시스템입니다. 테이블 간의 관계를 통해 복잡한 데이터 구조를 효율적으로 표현합니다.

## 📑 목차

- [[#🎯 RDBMS란?]]
- [[#📊 테이블 구조와 구성 요소]]
- [[#🔑 관계형 모델의 핵심 개념]]
- [[#📝 RDBMS 용어 정리]]
- [[#🏗️ 스키마와 인스턴스]]
- [[#💼 주요 RDBMS 제품]]
- [[#⚡ RDBMS의 장단점]]
- [[#📚 참고자료]]

---

## 🎯 RDBMS란?

> [!note] 정의
> RDBMS(Relational Database Management System)는 관계형 모델을 기반으로 하는 데이터베이스 관리 시스템입니다. 모든 데이터를 2차원 테이블(표) 형태로 저장합니다.

### 관계형 모델의 탄생

**E.F. Codd의 관계형 모델 (1970)**
- IBM 연구원 Edgar F. Codd가 제안
- 수학적 집합론과 관계 대수에 기반
- 데이터 독립성과 일관성 보장

### RDBMS의 핵심 특징

1. **테이블 기반 구조**
   - 모든 데이터를 행(Row)과 열(Column)로 구성된 테이블에 저장
   - 각 테이블은 고유한 이름을 가짐

2. **관계(Relationship) 표현**
   - Foreign Key를 통한 테이블 간 연결
   - 참조 무결성 보장

3. **SQL 사용**
   - 표준화된 질의 언어
   - 선언적 언어로 "무엇을" 원하는지 기술

---

## 📊 테이블 구조와 구성 요소

### 테이블의 구조

```
┌─────────────────────────────────────────┐
│              Relation (테이블)           │
├─────────────────────────────────────────┤
│     Attribute (속성/컬럼)                │
│  ┌──────┬──────┬──────┬──────┬──────┐  │
│  │  ID  │ Name │ Age  │ City │ ...  │  │ ← 스키마
│  ├──────┼──────┼──────┼──────┼──────┤  │
│  │  1   │ 김철수 │  25  │ 서울  │ ...  │  │ ← Tuple
│  │  2   │ 이영희 │  30  │ 부산  │ ...  │  │   (행/레코드)
│  │  3   │ 박민수 │  28  │ 대구  │ ...  │  │
│  └──────┴──────┴──────┴──────┴──────┘  │
│           ↑                              │
│        Domain                           │
│     (값의 범위)                          │
└─────────────────────────────────────────┘
```

### 구성 요소 상세

> [!important] 테이블 구성 요소
> 각 구성 요소는 관계형 모델에서 특별한 의미를 가집니다.

**1. Relation (관계/테이블)**
- 동일한 속성을 공유하는 튜플들의 집합
- 수학적 관계(relation)에서 유래

**2. Attribute (속성/컬럼)**
- 테이블의 열을 구성
- 각 속성은 고유한 이름과 도메인을 가짐

**3. Tuple (튜플/행/레코드)**
- 하나의 개체(entity)를 표현
- 속성값들의 순서쌍

**4. Domain (도메인)**
- 속성이 가질 수 있는 값의 집합
- 예: Age 속성의 도메인 = {0 이상의 정수}

**5. Cardinality (카디널리티)**
- 테이블의 튜플 개수
- 테이블의 크기를 나타냄

**6. Degree (차수)**
- 테이블의 속성 개수
- 테이블의 복잡도를 나타냄

---

## 🔑 관계형 모델의 핵심 개념

### Primary Key (기본키)

> [!tip] Primary Key의 특징
> - **유일성**: 각 행을 고유하게 식별
> - **NOT NULL**: NULL 값 불가
> - **불변성**: 한번 설정되면 변경 최소화

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,  -- 기본키
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE
);
```

### Foreign Key (외래키)

> [!example] Foreign Key 사용 예시
> ```sql
> CREATE TABLE orders (
>     order_id INT PRIMARY KEY,
>     customer_id INT,
>     order_date DATE,
>     FOREIGN KEY (customer_id) 
>         REFERENCES customers(customer_id)
> );
> ```

**참조 무결성 제약조건**:
1. 참조하는 테이블의 Primary Key 값만 가능
2. NULL이거나 참조 테이블에 존재하는 값
3. 부모 레코드 삭제 시 처리 방법 지정 가능

---

## 📝 RDBMS 용어 정리

### SQL 용어 vs 관계형 모델 용어

| SQL 용어 | 관계형 데이터베이스 용어 | 설명 |
|---------|------------------------|------|
| Table | Relation (관계) | 데이터를 저장하는 2차원 구조 |
| Row | Tuple (튜플) | 하나의 레코드/항목 |
| Column | Attribute (속성) | 데이터의 특정 필드 |
| Primary Key | Primary Key | 행을 고유하게 식별하는 키 |
| Foreign Key | Foreign Key | 다른 테이블 참조 키 |
| View | Derived Relation | 가상 테이블 |
| Result Set | Result Relation | 쿼리 결과 |

### 관계 대수 (Relational Algebra)

> [!note] 관계 대수란?
> 관계형 데이터베이스의 이론적 기초. 테이블을 조작하는 수학적 연산의 집합.

**기본 연산**:
- **Selection (σ)**: 조건에 맞는 행 선택
- **Projection (π)**: 특정 열만 선택
- **Union (∪)**: 두 테이블 합집합
- **Difference (−)**: 차집합
- **Cartesian Product (×)**: 카테시안 곱
- **Join (⋈)**: 조인 연산

```sql
-- Selection: WHERE 절
SELECT * FROM students WHERE age > 20;

-- Projection: SELECT 절의 컬럼 지정
SELECT name, age FROM students;

-- Join: JOIN 연산
SELECT * FROM students 
JOIN enrollments ON students.id = enrollments.student_id;
```

---

## 🏗️ 스키마와 인스턴스

### 스키마 (Schema)

> [!info] 스키마의 3단계 구조
> 데이터베이스의 논리적 구조를 정의

```
┌─────────────────────────┐
│   External Schema       │ ← 사용자/응용프로그램 관점
│   (View Level)          │   여러 개 존재 가능
├─────────────────────────┤
│   Conceptual Schema     │ ← 전체 데이터베이스 구조
│   (Logical Level)       │   하나만 존재
├─────────────────────────┤
│   Internal Schema       │ ← 물리적 저장 구조
│   (Physical Level)      │   하나만 존재
└─────────────────────────┘
```

**1. 외부 스키마 (External Schema)**
- 개별 사용자 관점
- View를 통해 구현
- 보안과 사용자 편의성 제공

**2. 개념 스키마 (Conceptual Schema)**
- 전체 데이터베이스의 논리적 구조
- 테이블, 관계, 제약조건 정의
- DBA가 관리

**3. 내부 스키마 (Internal Schema)**
- 물리적 저장 구조
- 인덱스, 파일 구조, 저장 방식
- DBMS가 관리

### 인스턴스 (Instance)

특정 시점의 데이터베이스 상태:
- 스키마는 구조, 인스턴스는 실제 데이터
- 시간에 따라 변화
- 스키마는 고정, 인스턴스는 동적

---

## 💼 주요 RDBMS 제품

### 오픈소스 RDBMS

> [!success] 무료 라이선스
> 복잡한 기능은 직접 개발하여 사용 가능

| 제품 | 특징 | 주요 사용처 |
|------|------|------------|
| **MySQL** | - 가장 널리 사용되는 오픈소스 DB<br>- 웹 애플리케이션에 최적화<br>- Oracle이 인수 | 웹 서비스, 스타트업 |
| **PostgreSQL** | - 가장 발전된 오픈소스 DB<br>- 확장성과 표준 준수<br>- JSON, 지리정보 지원 | 복잡한 쿼리, GIS |
| **MariaDB** | - MySQL 포크<br>- MySQL 호환성<br>- 활발한 커뮤니티 | MySQL 대체제 |
| **SQLite** | - 서버리스, 파일 기반<br>- 임베디드 시스템<br>- 설정 불필요 | 모바일 앱, 임베디드 |

### 상용 RDBMS

> [!warning] 유료 라이선스
> 성능 튜닝이 잘 되어있고 기능이 풍부하지만 비용이 높음

| 제품 | 특징 | 주요 사용처 |
|------|------|------------|
| **Oracle** | - 엔터프라이즈 표준<br>- 강력한 기능과 성능<br>- 높은 라이선스 비용 | 대기업, 금융권 |
| **MS SQL Server** | - Windows 환경 최적화<br>- .NET 통합 우수<br>- BI 도구 포함 | Windows 기반 기업 |
| **IBM DB2** | - 메인프레임 지원<br>- 높은 안정성<br>- AI 기능 통합 | 대규모 기업 |

---

## ⚡ RDBMS의 장단점

### 장점

> [!success] RDBMS의 강점
> 1. **데이터 무결성**: 제약조건과 트랜잭션으로 데이터 일관성 보장
> 2. **표준화된 SQL**: 범용적인 질의 언어
> 3. **ACID 속성**: 트랜잭션 안정성
> 4. **정규화**: 중복 최소화와 이상현상 방지
> 5. **성숙한 기술**: 오랜 역사와 검증된 안정성
> 6. **강력한 도구**: 다양한 관리/개발 도구

### 단점

> [!failure] RDBMS의 한계
> 1. **확장성 제한**: 수평적 확장(Scale-out) 어려움
> 2. **스키마 경직성**: 구조 변경 어려움
> 3. **복잡한 JOIN**: 성능 저하 가능성
> 4. **빅데이터 처리**: 대용량 비정형 데이터 처리 한계
> 5. **높은 비용**: 상용 제품의 라이선스 비용
> 6. **객체-관계 불일치**: ORM 필요

### RDBMS vs NoSQL 선택 기준

| 고려사항 | RDBMS 선택 | NoSQL 선택 |
|---------|-----------|-----------|
| 데이터 구조 | 정형화된 구조 | 비정형/반정형 |
| 일관성 요구 | ACID 필수 | Eventually Consistent 허용 |
| 확장성 | 수직 확장 가능 | 수평 확장 필요 |
| 트랜잭션 | 복잡한 트랜잭션 | 단순 읽기/쓰기 |
| 쿼리 복잡도 | 복잡한 JOIN 필요 | 단순 조회 위주 |

---

## 📚 참고자료

### 내부 문서
- [[01_database-fundamentals|데이터베이스 기초]]
- [[03_sql-crud-operations|SQL CRUD 작업]]
- [[04_keys-and-constraints|키와 제약조건]]
- [[05_relationships-and-erd|관계와 ERD]]

### 외부 자료
- [Relational Model - Wikipedia](https://en.wikipedia.org/wiki/Relational_model)
- [E.F. Codd's Original Paper (1970)](https://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf)
- [SQL Standard Documentation](https://www.iso.org/standard/63555.html)

### 추천 학습 경로
1. **기초**: 테이블 구조와 Primary Key 이해
2. **중급**: 정규화와 JOIN 연산 학습
3. **고급**: 인덱스 최적화와 쿼리 튜닝

---

> [!quote]
> "관계형 모델의 힘은 그 단순함에 있다. 테이블이라는 직관적인 구조로 복잡한 현실 세계를 모델링할 수 있다." - E.F. Codd