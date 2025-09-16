---
tags:
  - database
  - sql
  - query
  - group-by
  - aggregate
  - advanced-sql
created: 2025-01-08
updated: 2025-01-08
---

# 고급 SQL 쿼리

> [!info] 개요
> GROUP BY, HAVING, 집계함수, 서브쿼리 등 고급 SQL 쿼리 기법을 학습합니다. 실무에서 자주 사용되는 복잡한 쿼리 패턴과 최적화 방법을 다룹니다.

## 📑 목차

- [[#📊 집계함수]]
- [[#🔄 GROUP BY]]
- [[#⚡ HAVING 절]]
- [[#📅 날짜 함수]]
- [[#🎯 서브쿼리]]
- [[#🔗 집합 연산]]
- [[#💡 고급 쿼리 패턴]]
- [[#📝 실전 예제]]
- [[#📚 참고자료]]

---

## 📊 집계함수

### 주요 집계함수

> [!note] 집계함수 종류
> 집계함수는 여러 행을 하나의 결과로 집계하는 함수입니다.

| 함수 | 설명 | 예제 |
|------|------|------|
| **COUNT()** | 행의 개수 | `COUNT(*)`, `COUNT(DISTINCT column)` |
| **SUM()** | 합계 | `SUM(price)` |
| **AVG()** | 평균 | `AVG(score)` |
| **MAX()** | 최대값 | `MAX(date)` |
| **MIN()** | 최소값 | `MIN(salary)` |
| **GROUP_CONCAT()** | 문자열 연결 | `GROUP_CONCAT(name)` |
| **STDDEV()** | 표준편차 | `STDDEV(value)` |
| **VARIANCE()** | 분산 | `VARIANCE(value)` |

### 집계함수 사용 예제

```sql
-- 기본 집계함수
SELECT 
    COUNT(*) as total_orders,
    COUNT(DISTINCT customer_id) as unique_customers,
    SUM(total_amount) as revenue,
    AVG(total_amount) as avg_order_value,
    MAX(order_date) as last_order,
    MIN(order_date) as first_order
FROM orders
WHERE status = 'completed';

-- NULL 처리와 집계
SELECT 
    COUNT(*) as total_rows,           -- NULL 포함
    COUNT(phone) as with_phone,       -- NULL 제외
    COUNT(DISTINCT phone) as unique_phones
FROM customers;

-- 조건부 집계
SELECT 
    COUNT(CASE WHEN status = 'active' THEN 1 END) as active_count,
    SUM(CASE WHEN category = 'premium' THEN price ELSE 0 END) as premium_revenue
FROM products;
```

---

## 🔄 GROUP BY

### GROUP BY 기본

> [!tip] GROUP BY 원칙
> SELECT 절에 나타나는 모든 컬럼은 집계함수이거나 GROUP BY 절에 포함되어야 합니다.

```sql
-- 카테고리별 상품 통계
SELECT 
    category,
    COUNT(*) as product_count,
    AVG(price) as avg_price,
    MIN(price) as min_price,
    MAX(price) as max_price
FROM products
GROUP BY category
ORDER BY product_count DESC;

-- 다중 컬럼 GROUP BY
SELECT 
    YEAR(order_date) as year,
    MONTH(order_date) as month,
    status,
    COUNT(*) as order_count,
    SUM(total_amount) as total_revenue
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date), status
ORDER BY year, month, status;

-- WITH ROLLUP: 소계와 총계 추가
SELECT 
    COALESCE(category, '전체') as category,
    COALESCE(subcategory, '소계') as subcategory,
    COUNT(*) as count,
    SUM(price) as total_price
FROM products
GROUP BY category, subcategory WITH ROLLUP;
```

### GROUP BY 최적화

```sql
-- 인덱스를 활용한 GROUP BY
CREATE INDEX idx_category_price ON products(category, price);

-- 효율적인 GROUP BY
SELECT category, COUNT(*), AVG(price)
FROM products
GROUP BY category;  -- 인덱스 활용

-- 비효율적인 GROUP BY
SELECT UPPER(category), COUNT(*)
FROM products
GROUP BY UPPER(category);  -- 함수 적용으로 인덱스 미활용
```

---

## ⚡ HAVING 절

### HAVING vs WHERE

> [!warning] HAVING과 WHERE의 차이
> - **WHERE**: 그룹화 전 개별 행 필터링
> - **HAVING**: 그룹화 후 그룹 필터링

```sql
-- HAVING 사용 예제
SELECT 
    customer_id,
    COUNT(*) as order_count,
    SUM(total_amount) as total_spent
FROM orders
WHERE status = 'completed'  -- 그룹화 전 필터
GROUP BY customer_id
HAVING COUNT(*) >= 5        -- 그룹화 후 필터
   AND SUM(total_amount) > 1000
ORDER BY total_spent DESC;

-- 복잡한 HAVING 조건
SELECT 
    product_id,
    AVG(rating) as avg_rating,
    COUNT(*) as review_count
FROM reviews
GROUP BY product_id
HAVING AVG(rating) >= 4.0
   AND COUNT(*) >= 10  -- 평점 4.0 이상, 리뷰 10개 이상
ORDER BY avg_rating DESC, review_count DESC;
```

---

## 📅 날짜 함수

### 주요 날짜 함수

```sql
-- 날짜 차이 계산
SELECT 
    customer_id,
    first_order_date,
    last_order_date,
    DATEDIFF(last_order_date, first_order_date) as customer_lifetime_days,
    DATEDIFF(CURDATE(), last_order_date) as days_since_last_order
FROM (
    SELECT 
        customer_id,
        MIN(order_date) as first_order_date,
        MAX(order_date) as last_order_date
    FROM orders
    GROUP BY customer_id
) customer_stats;

-- 날짜 포맷팅과 추출
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') as month,
    YEAR(order_date) as year,
    QUARTER(order_date) as quarter,
    WEEK(order_date) as week_number,
    DAYNAME(order_date) as day_name,
    COUNT(*) as order_count
FROM orders
GROUP BY DATE_FORMAT(order_date, '%Y-%m')
ORDER BY month;

-- 날짜 연산
SELECT 
    order_date,
    DATE_ADD(order_date, INTERVAL 7 DAY) as expected_delivery,
    DATE_SUB(order_date, INTERVAL 1 MONTH) as one_month_ago,
    LAST_DAY(order_date) as month_end,
    DATE_FORMAT(order_date, '%Y-%m-01') as month_start
FROM orders;
```

### COALESCE와 NULL 처리

```sql
-- COALESCE: 첫 번째 NULL이 아닌 값 반환
SELECT 
    customer_id,
    COALESCE(phone, email, 'No Contact') as primary_contact,
    COALESCE(preferred_name, first_name, 'Customer') as display_name
FROM customers;

-- IFNULL과 NULLIF
SELECT 
    product_id,
    IFNULL(discount_price, regular_price) as final_price,
    NULLIF(quantity, 0) as valid_quantity  -- 0을 NULL로 변환
FROM products;

-- NULL 처리와 집계
SELECT 
    category,
    AVG(COALESCE(rating, 3)) as avg_rating,  -- NULL을 3으로 대체
    COUNT(rating) as rated_count,
    COUNT(*) - COUNT(rating) as unrated_count
FROM products
GROUP BY category;
```

---

## 🎯 서브쿼리

### 서브쿼리 유형

> [!example] 서브쿼리 분류
> - **스칼라 서브쿼리**: 단일 값 반환
> - **인라인 뷰**: FROM 절의 서브쿼리
> - **상관 서브쿼리**: 외부 쿼리 참조

```sql
-- 스칼라 서브쿼리
SELECT 
    product_name,
    price,
    (SELECT AVG(price) FROM products) as avg_price,
    price - (SELECT AVG(price) FROM products) as price_diff
FROM products;

-- IN 서브쿼리
SELECT *
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id
    FROM orders
    WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
);

-- EXISTS 서브쿼리
SELECT c.*
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
    AND o.status = 'pending'
);

-- 인라인 뷰 (FROM 절 서브쿼리)
SELECT 
    category,
    avg_price,
    product_count
FROM (
    SELECT 
        category,
        AVG(price) as avg_price,
        COUNT(*) as product_count
    FROM products
    GROUP BY category
) category_stats
WHERE avg_price > 100
ORDER BY avg_price DESC;
```

### 상관 서브쿼리

```sql
-- 각 카테고리에서 평균 이상 가격의 상품
SELECT p1.*
FROM products p1
WHERE price > (
    SELECT AVG(p2.price)
    FROM products p2
    WHERE p2.category = p1.category
);

-- 각 고객의 최근 주문
SELECT o1.*
FROM orders o1
WHERE order_date = (
    SELECT MAX(o2.order_date)
    FROM orders o2
    WHERE o2.customer_id = o1.customer_id
);

-- 순위 계산 (MySQL 8.0 이전)
SELECT 
    product_id,
    price,
    (SELECT COUNT(DISTINCT price) 
     FROM products p2 
     WHERE p2.price > p1.price) + 1 as price_rank
FROM products p1
ORDER BY price_rank;
```

---

## 🔗 집합 연산

### UNION과 UNION ALL

```sql
-- UNION: 중복 제거
SELECT customer_id, 'order' as source
FROM orders
WHERE order_date >= '2024-01-01'
UNION
SELECT customer_id, 'review' as source
FROM reviews
WHERE review_date >= '2024-01-01';

-- UNION ALL: 중복 포함 (더 빠름)
SELECT product_id, quantity, 'sale' as type
FROM sales
UNION ALL
SELECT product_id, quantity, 'return' as type
FROM returns;

-- 복잡한 UNION 쿼리
(SELECT 
    'High Value' as customer_type,
    customer_id,
    total_spent
FROM (
    SELECT customer_id, SUM(total_amount) as total_spent
    FROM orders
    GROUP BY customer_id
    HAVING SUM(total_amount) > 10000
) high_value
LIMIT 10)
UNION ALL
(SELECT 
    'Frequent' as customer_type,
    customer_id,
    total_spent
FROM (
    SELECT customer_id, SUM(total_amount) as total_spent
    FROM orders
    GROUP BY customer_id
    HAVING COUNT(*) > 20
) frequent
LIMIT 10)
ORDER BY customer_type, total_spent DESC;
```

---

## 💡 고급 쿼리 패턴

### Window 함수 (MySQL 8.0+)

```sql
-- ROW_NUMBER, RANK, DENSE_RANK
SELECT 
    product_id,
    category,
    price,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as row_num,
    RANK() OVER (PARTITION BY category ORDER BY price DESC) as rank,
    DENSE_RANK() OVER (PARTITION BY category ORDER BY price DESC) as dense_rank
FROM products;

-- 누적 합계와 이동 평균
SELECT 
    order_date,
    total_amount,
    SUM(total_amount) OVER (ORDER BY order_date) as cumulative_total,
    AVG(total_amount) OVER (
        ORDER BY order_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7days
FROM orders;

-- LAG와 LEAD
SELECT 
    month,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY month) as prev_month_revenue,
    LEAD(revenue, 1) OVER (ORDER BY month) as next_month_revenue,
    revenue - LAG(revenue, 1) OVER (ORDER BY month) as month_over_month_change
FROM monthly_revenue;
```

### Common Table Expression (CTE)

```sql
-- WITH 절 사용
WITH customer_stats AS (
    SELECT 
        customer_id,
        COUNT(*) as order_count,
        SUM(total_amount) as total_spent,
        AVG(total_amount) as avg_order_value
    FROM orders
    GROUP BY customer_id
),
customer_segments AS (
    SELECT 
        customer_id,
        CASE 
            WHEN total_spent > 10000 THEN 'VIP'
            WHEN total_spent > 5000 THEN 'Gold'
            WHEN total_spent > 1000 THEN 'Silver'
            ELSE 'Bronze'
        END as segment
    FROM customer_stats
)
SELECT 
    segment,
    COUNT(*) as customer_count,
    AVG(cs.total_spent) as avg_spent
FROM customer_segments seg
JOIN customer_stats cs ON seg.customer_id = cs.customer_id
GROUP BY segment
ORDER BY avg_spent DESC;
```

---

## 📝 실전 예제

### 커플 관계 데이터베이스 분석

> [!example] 실무 시나리오 기반 쿼리

```sql
-- 1. 각 커플의 평균 평점
SELECT 
    relationship_id,
    AVG(score) as avg_rating,
    COUNT(*) as rating_count
FROM ratings
GROUP BY relationship_id
ORDER BY avg_rating DESC;

-- 2. 평가자 2명 이상인 커플 중 최고 평점
SELECT 
    r.relationship_id,
    s1.name as person1,
    s2.name as person2,
    AVG(rt.score) as avg_rating,
    COUNT(rt.student_id) as evaluator_count
FROM relationships r
JOIN students s1 ON r.sender = s1.student_id
JOIN students s2 ON r.receiver = s2.student_id
LEFT JOIN ratings rt ON r.id = rt.relationship_id
GROUP BY r.relationship_id
HAVING COUNT(rt.student_id) >= 2
ORDER BY avg_rating DESC
LIMIT 1;

-- 3. 현재 사귀는 커플들의 평균 평점
SELECT 
    r.relationship_id,
    AVG(rt.score) as avg_rating,
    DATEDIFF(COALESCE(r.broke_at, CURDATE()), r.started_at) as days_together
FROM relationships r
LEFT JOIN ratings rt ON r.id = rt.relationship_id
WHERE r.broke_at IS NULL
GROUP BY r.relationship_id;

-- 4. N다리 걸친 학생 찾기 (N > 1)
SELECT 
    student_id,
    name,
    active_relationships
FROM (
    SELECT 
        s.student_id,
        s.name,
        (
            SELECT COUNT(*)
            FROM relationships r
            WHERE (r.sender = s.student_id OR r.receiver = s.student_id)
            AND r.broke_at IS NULL
        ) as active_relationships
    FROM students s
) student_relationships
WHERE active_relationships > 1;

-- 5. 연애 경험 통계
SELECT 
    s.student_id,
    s.name,
    COUNT(DISTINCT CASE WHEN r.sender = s.student_id THEN r.id END) as sent_count,
    COUNT(DISTINCT CASE WHEN r.receiver = s.student_id THEN r.id END) as received_count,
    COUNT(DISTINCT r.id) as total_relationships
FROM students s
LEFT JOIN relationships r ON s.student_id IN (r.sender, r.receiver)
GROUP BY s.student_id
ORDER BY total_relationships DESC;
```

### 성능 최적화된 쿼리

```sql
-- 인덱스 활용 최적화
CREATE INDEX idx_order_customer_date ON orders(customer_id, order_date);
CREATE INDEX idx_order_status_date ON orders(status, order_date);

-- 최적화된 월별 매출 집계
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') as month,
    COUNT(*) as order_count,
    SUM(total_amount) as revenue
FROM orders
WHERE status = 'completed'
    AND order_date >= '2024-01-01'
GROUP BY DATE_FORMAT(order_date, '%Y-%m')
ORDER BY month;

-- 서브쿼리 최적화 (JOIN으로 변경)
-- 비효율적
SELECT *
FROM products
WHERE category_id IN (
    SELECT category_id
    FROM categories
    WHERE parent_id = 1
);

-- 효율적
SELECT p.*
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
WHERE c.parent_id = 1;
```

---

## 📚 참고자료

### 내부 문서
- [[03_sql-crud-operations|SQL CRUD 작업]]
- [[08_database-index|데이터베이스 인덱스]]
- [[10_join-operations|JOIN 연산]]

### 외부 리소스
- [MySQL Aggregate Functions](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html)
- [Window Functions in MySQL](https://dev.mysql.com/doc/refman/8.0/en/window-functions.html)
- [SQL Query Optimization](https://use-the-index-luke.com/)

### 추가 학습 주제
- Query Execution Plan 분석
- Index Hints 사용법
- Partitioned Tables에서의 집계
- Parallel Query Execution

---

> [!quote]
> "복잡한 쿼리를 작성하는 능력은 데이터를 통찰력으로 변환하는 핵심 기술입니다. 하지만 항상 가독성과 유지보수성을 고려해야 합니다."