---
tags:
  - database
  - index
  - performance
  - b-tree
  - optimization
created: 2025-01-08
updated: 2025-01-08
---

# 데이터베이스 인덱스

> [!info] 개요
> 인덱스는 데이터베이스 검색 속도를 향상시키는 자료구조입니다. 적절한 인덱스 설계는 쿼리 성능을 수십 배에서 수백 배까지 향상시킬 수 있습니다.

## 📑 목차

- [[#🎯 인덱스 개념]]
- [[#🌳 인덱스 자료구조]]
- [[#📊 인덱스 종류]]
- [[#⚡ 인덱스 설계 전략]]
- [[#🔍 EXPLAIN 분석]]
- [[#💡 인덱스 최적화]]
- [[#⚠️ 인덱스 주의사항]]
- [[#📚 참고자료]]

---

## 🎯 인덱스 개념

### 인덱스란?

> [!note] 인덱스 정의
> 인덱스는 책의 색인과 같이 데이터를 빠르게 찾을 수 있도록 돕는 데이터베이스 객체입니다.

### 인덱스의 장단점

| 구분 | 내용 |
|------|------|
| **장점** | • 검색 속도 향상<br>• ORDER BY, GROUP BY 성능 개선<br>• JOIN 성능 향상<br>• UNIQUE 제약 강제 |
| **단점** | • 추가 저장 공간 필요<br>• INSERT, UPDATE, DELETE 성능 저하<br>• 인덱스 유지관리 비용<br>• 잘못된 인덱스는 오히려 성능 저하 |

### 인덱스 동작 원리

```sql
-- 인덱스 없이 검색 (Full Table Scan)
SELECT * FROM users WHERE email = 'user@example.com';
-- 100만 건 테이블: 100만 건 모두 확인

-- 인덱스 있을 때 (Index Seek)
CREATE INDEX idx_email ON users(email);
SELECT * FROM users WHERE email = 'user@example.com';
-- B-Tree 높이가 3이면: 3번의 I/O로 검색 완료
```

---

## 🌳 인덱스 자료구조

### B-Tree 인덱스

> [!tip] B-Tree 특징
> - 균형 트리 구조로 O(log n) 검색 성능
> - 범위 검색에 효율적
> - 대부분의 RDBMS에서 기본 인덱스 구조

```
        [50]
       /    \
    [20,30]  [70,80]
   /   |   \    |   \
[10] [25] [35] [60] [75,85,90]
```

### B-Tree 인덱스 특성

```sql
-- B-Tree가 효율적인 경우
SELECT * FROM products WHERE price > 100 AND price < 500;  -- 범위 검색
SELECT * FROM users WHERE name LIKE 'Kim%';  -- 전방 일치
SELECT * FROM orders ORDER BY order_date;  -- 정렬

-- B-Tree가 비효율적인 경우
SELECT * FROM users WHERE name LIKE '%Kim%';  -- 중간 일치
SELECT * FROM products WHERE price * 1.1 > 100;  -- 함수 적용
```

### Hash 인덱스

> [!note] Hash 인덱스 특징
> - O(1) 검색 성능
> - 동등 비교(=)만 지원
> - 범위 검색 불가
> - Memory 스토리지 엔진에서 주로 사용

```sql
-- Hash 인덱스가 효율적인 경우
SELECT * FROM users WHERE user_id = 12345;  -- 동등 비교

-- Hash 인덱스 사용 불가
SELECT * FROM users WHERE user_id > 10000;  -- 범위 검색 불가
SELECT * FROM users ORDER BY user_id;  -- 정렬 불가
```

---

## 📊 인덱스 종류

### 클러스터드 인덱스

> [!warning] 클러스터드 인덱스
> - 테이블당 하나만 존재
> - 실제 데이터가 인덱스 순서로 정렬
> - InnoDB에서는 Primary Key가 클러스터드 인덱스

```sql
-- 클러스터드 인덱스 (Primary Key)
CREATE TABLE users (
    user_id INT PRIMARY KEY,  -- 클러스터드 인덱스
    email VARCHAR(100),
    name VARCHAR(50)
);

-- 데이터가 user_id 순서로 물리적으로 저장됨
```

### 논클러스터드 인덱스

```sql
-- 논클러스터드 인덱스 (Secondary Index)
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_name ON users(name);

-- 별도의 인덱스 구조에서 데이터 위치를 가리킴
```

### 복합 인덱스

```sql
-- 복합 인덱스 생성
CREATE INDEX idx_category_price ON products(category, price);

-- 효율적인 쿼리 (인덱스 활용)
SELECT * FROM products WHERE category = 'Electronics';  -- ✅
SELECT * FROM products WHERE category = 'Electronics' AND price > 100;  -- ✅

-- 비효율적인 쿼리 (인덱스 미활용)
SELECT * FROM products WHERE price > 100;  -- ❌ 첫 번째 컬럼 없음
```

### 커버링 인덱스

```sql
-- 커버링 인덱스: 쿼리에 필요한 모든 컬럼을 포함
CREATE INDEX idx_covering ON orders(customer_id, order_date, total_amount);

-- 인덱스만으로 쿼리 처리 (테이블 접근 불필요)
SELECT customer_id, order_date, total_amount
FROM orders
WHERE customer_id = 1001;
```

### 유니크 인덱스

```sql
-- 유니크 인덱스
CREATE UNIQUE INDEX idx_unique_email ON users(email);

-- 중복 방지
INSERT INTO users (email) VALUES ('user@example.com');  -- 성공
INSERT INTO users (email) VALUES ('user@example.com');  -- 실패 (중복)
```

---

## ⚡ 인덱스 설계 전략

### 인덱스 선택 기준

> [!tip] 인덱스 생성 고려사항
> 1. **카디널리티**: 중복이 적은 컬럼 우선
> 2. **선택도**: 전체 대비 필터링 비율
> 3. **사용 빈도**: 자주 사용되는 조건
> 4. **정렬 요구사항**: ORDER BY, GROUP BY

### 카디널리티와 선택도

```sql
-- 카디널리티 확인
SELECT 
    COUNT(DISTINCT gender) as gender_cardinality,  -- 낮음 (2)
    COUNT(DISTINCT email) as email_cardinality,     -- 높음 (유니크)
    COUNT(DISTINCT birth_date) as birth_cardinality -- 중간
FROM users;

-- 선택도 계산
SELECT 
    COUNT(DISTINCT column_name) / COUNT(*) as selectivity
FROM table_name;
-- 선택도가 높을수록(1에 가까울수록) 인덱스 효율적
```

### 복합 인덱스 설계

```sql
-- 복합 인덱스 컬럼 순서 원칙
-- 1. 동등 조건 컬럼 먼저
-- 2. 정렬 컬럼
-- 3. 범위 조건 컬럼 마지막

-- 최적화된 복합 인덱스
CREATE INDEX idx_optimal ON orders(
    status,        -- 동등 조건 (WHERE status = 'pending')
    customer_id,   -- 동등 조건 (AND customer_id = 1001)
    order_date     -- 범위 조건 (AND order_date > '2024-01-01')
);

-- 쿼리
SELECT * FROM orders
WHERE status = 'pending'
  AND customer_id = 1001
  AND order_date > '2024-01-01'
ORDER BY order_date;
```

---

## 🔍 EXPLAIN 분석

### EXPLAIN 사용법

```sql
-- 쿼리 실행 계획 확인
EXPLAIN SELECT * FROM users WHERE email = 'user@example.com';

-- 상세 실행 계획
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'user@example.com';
```

### EXPLAIN 출력 해석

| 컬럼 | 설명 | 주요 값 |
|------|------|---------|
| **type** | 접근 방식 | const > eq_ref > ref > range > index > ALL |
| **possible_keys** | 사용 가능한 인덱스 | 인덱스 목록 |
| **key** | 실제 사용된 인덱스 | 인덱스명 또는 NULL |
| **rows** | 예상 검색 행 수 | 작을수록 효율적 |
| **Extra** | 추가 정보 | Using index (커버링), Using filesort (정렬) |

### 실행 계획 예제

```sql
-- 인덱스 사용 확인
EXPLAIN SELECT * FROM products 
WHERE category = 'Electronics' 
  AND price > 100;

-- 결과 해석
+----+-------------+----------+-------+----------------------+-------------+---------+------+------+-------------+
| id | select_type | table    | type  | possible_keys        | key         | key_len | ref  | rows | Extra       |
+----+-------------+----------+-------+----------------------+-------------+---------+------+------+-------------+
|  1 | SIMPLE      | products | range | idx_category_price   | idx_category_price | 8  | NULL | 1000 | Using where |
+----+-------------+----------+-------+----------------------+-------------+---------+------+------+-------------+
```

---

## 💡 인덱스 최적화

### 인덱스 힌트

```sql
-- 인덱스 강제 사용
SELECT * FROM users USE INDEX (idx_email)
WHERE email = 'user@example.com';

-- 인덱스 무시
SELECT * FROM users IGNORE INDEX (idx_email)
WHERE email = 'user@example.com';

-- 인덱스 강제 (FORCE)
SELECT * FROM users FORCE INDEX (idx_email)
WHERE email = 'user@example.com';
```

### 인덱스 통계 업데이트

```sql
-- 인덱스 통계 분석
ANALYZE TABLE users;

-- 인덱스 재구성
OPTIMIZE TABLE users;

-- 인덱스 통계 확인
SHOW INDEX FROM users;
```

### 불필요한 인덱스 제거

```sql
-- 사용되지 않는 인덱스 찾기
SELECT 
    s.index_name,
    s.table_name,
    s.cardinality,
    s.avg_row_length
FROM information_schema.statistics s
LEFT JOIN performance_schema.table_io_waits_summary_by_index_usage u
    ON s.table_schema = u.object_schema
    AND s.table_name = u.object_name
    AND s.index_name = u.index_name
WHERE u.count_star = 0
    AND s.table_schema = 'your_database';

-- 중복 인덱스 찾기
SELECT 
    redundant_index.table_name,
    redundant_index.index_name as redundant,
    dominant_index.index_name as dominant
FROM (
    SELECT * FROM information_schema.statistics
    WHERE table_schema = 'your_database'
) redundant_index
JOIN (
    SELECT * FROM information_schema.statistics
    WHERE table_schema = 'your_database'
) dominant_index
ON redundant_index.table_name = dominant_index.table_name
    AND redundant_index.column_name = dominant_index.column_name
    AND redundant_index.seq_in_index = dominant_index.seq_in_index
    AND redundant_index.index_name != dominant_index.index_name;
```

---

## ⚠️ 인덱스 주의사항

### 인덱스가 사용되지 않는 경우

> [!danger] 인덱스 무효화 상황
> - 함수나 연산자 적용
> - 타입 불일치
> - NULL 비교
> - 부정 조건 (!=, NOT IN)
> - LIKE '%pattern'

```sql
-- ❌ 인덱스 사용 불가
SELECT * FROM users WHERE YEAR(created_at) = 2024;  -- 함수 적용
SELECT * FROM products WHERE price * 1.1 > 100;     -- 연산
SELECT * FROM users WHERE email != 'test@test.com'; -- 부정 조건
SELECT * FROM users WHERE name LIKE '%kim';         -- 후방 일치

-- ✅ 인덱스 사용 가능하도록 수정
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
SELECT * FROM products WHERE price > 100 / 1.1;
SELECT * FROM users WHERE email = 'test@test.com' OR email IS NULL;
SELECT * FROM users WHERE name LIKE 'kim%';
```

### 과도한 인덱스의 문제

```sql
-- 테이블에 너무 많은 인덱스가 있으면:
-- 1. INSERT/UPDATE/DELETE 성능 저하
-- 2. 저장 공간 증가
-- 3. 옵티마이저 혼란

-- 인덱스 개수 확인
SELECT 
    table_name,
    COUNT(*) as index_count
FROM information_schema.statistics
WHERE table_schema = 'your_database'
GROUP BY table_name
ORDER BY index_count DESC;
```

---

## 📚 참고자료

### 내부 문서
- [[04_keys-and-constraints|키와 제약조건]]
- [[07_advanced-sql-queries|고급 SQL 쿼리]]
- [[13_database-scaling|데이터베이스 확장]]

### 외부 리소스
- [Use The Index, Luke!](https://use-the-index-luke.com/)
- [MySQL Index Documentation](https://dev.mysql.com/doc/refman/8.0/en/optimization-indexes.html)
- [PostgreSQL Index Types](https://www.postgresql.org/docs/current/indexes-types.html)

### 추가 학습 주제
- Full-Text 인덱스
- Spatial 인덱스
- Bitmap 인덱스
- Functional 인덱스
- Partial 인덱스

---

> [!quote]
> "인덱스는 데이터베이스 성능의 열쇠입니다. 하지만 모든 열쇠가 모든 자물쇠에 맞는 것은 아닙니다. 적절한 인덱스 설계가 중요합니다."