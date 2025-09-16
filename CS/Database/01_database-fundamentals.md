---
tags:
  - database
  - cs-fundamentals
  - dbms
  - data-management
created: 2025-01-09
updated: 2025-01-09
aliases:
  - 데이터베이스 기초
  - DB 기본 개념
  - Database Basics
description: 데이터베이스의 기본 개념, DBMS의 역할, 데이터베이스의 특징에 대한 종합 가이드
status: published
category: tutorial
---

# 데이터베이스 기초

> [!info] 개요
> 데이터베이스는 체계적으로 구조화된 데이터의 집합입니다. 이 문서에서는 데이터베이스의 기본 개념, DBMS의 역할, 그리고 데이터베이스의 핵심 특징들을 알아봅니다.

## 📑 목차

- [[#🎯 데이터베이스란?]]
- [[#🔧 DBMS (Database Management System)]]
- [[#💡 데이터베이스의 핵심 특징]]
- [[#📊 데이터베이스의 구조]]
- [[#🚀 데이터베이스 사용 이유]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## 🎯 데이터베이스란?

> [!note] 정의
> 데이터베이스(Database)는 서로 관련있는 데이터들의 정리된 집합입니다. 단순한 파일 시스템과 달리, 체계적으로 조직화되어 있어 효율적인 접근과 관리가 가능합니다.

### 핵심 개념

**데이터(Data)**
- 의미를 가지고 기록될 수 있는 모든 것
- 예: 이름, 나이, 주소, 구매 기록 등

**데이터베이스(Database)**
- 서로 관련있는 데이터들의 정리된 집합
- 중복을 최소화하고 구조화된 형태로 저장
- 여러 사용자가 동시에 접근 가능

### 데이터베이스 vs 파일 시스템

| 특징 | 파일 시스템 | 데이터베이스 |
|------|------------|-------------|
| 데이터 중복 | 높음 | 최소화 |
| 데이터 일관성 | 보장 어려움 | 제약조건으로 보장 |
| 동시 접근 | 제한적 | 다중 사용자 지원 |
| 보안 | 파일 단위 | 세밀한 권한 관리 |
| 백업/복구 | 수동 | 자동화 지원 |

---

## 🔧 DBMS (Database Management System)

> [!info] DBMS란?
> 데이터베이스를 관리하고 운영하는 소프트웨어 시스템입니다. 사용자와 데이터베이스 사이의 중개자 역할을 합니다.

### DBMS 아키텍처

```
       사용자/프로그래머
            ↕
      데이터베이스 시스템
    ┌─────────────────┐
    │  응용 프로그램/질의  │
    └────────┬────────┘
            ↕
    ┌─────────────────┐
    │      DBMS       │
    │  ┌───────────┐  │
    │  │ 질의 처리기 │  │
    │  │ SW + 디스크 │  │
    │  │ 접근 관리   │  │
    │  └─────┬─────┘  │
    └────────┼────────┘
            ↕
    ┌─────────────────┐
    │   메타 데이터    │
    │       +         │
    │  데이터베이스    │
    └─────────────────┘
```

### DBMS의 주요 기능

1. **데이터 정의 (DDL - Data Definition Language)**
   - 테이블 생성, 수정, 삭제
   - 스키마 정의

2. **데이터 조작 (DML - Data Manipulation Language)**
   - 데이터 검색, 삽입, 수정, 삭제
   - CRUD 작업 수행

3. **데이터 제어 (DCL - Data Control Language)**
   - 사용자 권한 관리
   - 보안 정책 적용

4. **트랜잭션 관리**
   - ACID 속성 보장
   - 동시성 제어

---

## 💡 데이터베이스의 핵심 특징

> [!important] 놓치기 쉬운 중요한 특징들
> 데이터베이스는 단순한 데이터 저장소가 아닙니다. 다음과 같은 고유한 특징들이 있습니다.

### 1. 자기 기술성 (Self-Describing)

데이터베이스는 데이터뿐만 아니라 데이터에 대한 데이터(메타데이터)도 함께 저장합니다.

> [!example] 메타데이터 예시
> ```sql
> -- 테이블 구조 정보
> DESCRIBE users;
> 
> -- 사용자 권한 정보
> SHOW GRANTS FOR 'username'@'localhost';
> 
> -- 인덱스 정보
> SHOW INDEX FROM table_name;
> ```

**카탈로그(Catalog)**
- 데이터베이스의 메타데이터를 저장하는 특별한 영역
- 테이블 구조, 제약조건, 사용자 정보 등 포함
- 시스템이 자동으로 관리

### 2. 프로그램-데이터 독립성

> [!tip] 독립성의 이점
> 데이터 구조가 변경되어도 응용 프로그램은 영향을 받지 않습니다.

**물리적 독립성**
- 데이터의 물리적 저장 방식 변경이 논리적 구조에 영향 없음
- 예: 인덱스 추가/삭제, 파일 재구성

**논리적 독립성**
- 논리적 구조 변경이 응용 프로그램에 최소한의 영향
- 예: 테이블에 새 컬럼 추가

### 3. 데이터 추상화 레벨

```
┌─────────────────────┐
│   외부 스키마 (View)  │ ← 사용자별 관점
├─────────────────────┤
│   개념 스키마         │ ← 전체 논리적 구조
├─────────────────────┤
│   내부 스키마         │ ← 물리적 저장 구조
└─────────────────────┘
```

---

## 📊 데이터베이스의 구조

### 테이블 기반 저장

> [!note] 왜 테이블인가?
> 사람에게 가장 직관적이고 가독성이 좋은 데이터 표현 방식이 표(테이블)이기 때문입니다.

### 데이터베이스 저장 구조 비유

**HashMap 자료구조와의 유사성**:

```python
# HashMap 구조
hashmap = {
    "key1": "value1",
    "key2": "value2"
}

# 데이터베이스 구조 (개념적)
database = {
    "primary_key1": (value1, value2, ..., valueN),  # RECORD
    "primary_key2": (value1, value2, ..., valueN)
}
```

**실제 데이터베이스 특징**:
- Primary Key를 통한 빠른 레코드 검색 (해시 또는 B-Tree 인덱스)
- 레코드는 여러 필드의 조합
- 인덱스를 통한 O(1) 또는 O(log n) 접근 시간

---

## 🚀 데이터베이스 사용 이유

### 1. 데이터 무결성 보장

> [!success] 제약조건을 통한 데이터 품질 유지
> - NOT NULL: 필수 입력 필드
> - UNIQUE: 중복 방지
> - CHECK: 유효성 검사
> - FOREIGN KEY: 참조 무결성

### 2. 동시성 제어

여러 사용자가 동시에 데이터에 접근해도 일관성 유지:
- 트랜잭션 격리 수준
- 락(Lock) 메커니즘
- MVCC (Multi-Version Concurrency Control)

### 3. 보안 및 권한 관리

```sql
-- 세밀한 권한 제어 예시
GRANT SELECT, INSERT ON database.table 
TO 'user'@'localhost';

REVOKE DELETE ON database.table 
FROM 'user'@'localhost';
```

### 4. 백업 및 복구

- 정기적인 자동 백업
- 시점 복구 (Point-in-Time Recovery)
- 트랜잭션 로그를 통한 복구

---

## ⚠️ 주의사항

> [!warning] 데이터베이스 설계 시 고려사항
> 1. **과도한 정규화 주의**: 성능 저하 가능성
> 2. **인덱스 남용 금지**: INSERT/UPDATE 성능 저하
> 3. **적절한 데이터 타입 선택**: 저장 공간과 성능에 영향
> 4. **백업 정책 수립**: 데이터 손실 방지

> [!danger] 보안 주의사항
> - 절대 프로덕션 데이터베이스 비밀번호를 코드에 하드코딩하지 마세요
> - SQL Injection 공격에 대비한 Prepared Statement 사용
> - 최소 권한 원칙 적용

---

## 📚 참고자료

### 내부 문서
- [[02_rdbms-basics|관계형 데이터베이스 기초]]
- [[03_sql-crud-operations|SQL CRUD 작업]]
- [[06_transaction-acid|트랜잭션과 ACID]]

### 외부 자료
- [Database System Concepts - Silberschatz](https://www.db-book.com/)
- [Head First SQL - O'Reilly](http://www.hanbit.co.kr/store/books/look.php?p_code=B5555904666)
- [Wikipedia - Database](https://en.wikipedia.org/wiki/Database)

### 추천 학습 자료
- **입문자**: Head First SQL (한빛미디어)
- **중급자**: Database System Concepts
- **실무자**: High Performance MySQL

---

> [!quote]
> "데이터베이스는 단순한 저장소가 아니라, 조직의 지식과 정보를 체계적으로 관리하는 핵심 인프라입니다."