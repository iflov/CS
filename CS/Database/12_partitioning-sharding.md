---
tags:
  - database
  - partitioning
  - sharding
  - scaling
  - distributed
created: 2025-01-08
updated: 2025-01-08
---

# 파티셔닝과 샤딩

> [!info] 개요
> 파티셔닝과 샤딩은 대용량 데이터를 효율적으로 관리하고 성능을 향상시키기 위한 데이터 분할 기법입니다. 수평/수직 분할 전략과 구현 방법을 학습합니다.

## 📑 목차

- [[#🎯 파티셔닝 개념]]
- [[#📊 파티셔닝 유형]]
- [[#🔧 파티션 관리]]
- [[#🌐 샤딩 개념]]
- [[#🔑 샤드 키 전략]]
- [[#⚡ 샤딩 구현]]
- [[#⚠️ 샤딩 트레이드오프]]
- [[#📚 참고자료]]

---

## 🎯 파티셔닝 개념

### 파티셔닝이란?

> [!note] Partitioning 정의
> 파티셔닝은 논리적으로 하나인 큰 테이블을 물리적으로 여러 작은 단위로 분할하는 기법입니다.

### 파티셔닝의 장점

| 장점 | 설명 |
|------|------|
| **성능 향상** | 파티션 프루닝으로 스캔 범위 축소 |
| **관리 용이성** | 파티션 단위로 백업, 복구, 유지보수 |
| **가용성 향상** | 파티션 단위 장애 격리 |
| **대용량 데이터 처리** | 병렬 처리 가능 |

### 수평 vs 수직 파티셔닝

```sql
-- 수평 파티셔닝 (Horizontal Partitioning)
-- 행 단위로 분할
-- 원본 테이블
CREATE TABLE orders (
    order_id INT,
    order_date DATE,
    customer_id INT,
    amount DECIMAL(10,2)
);

-- 파티셔닝 후
-- orders_2024_01, orders_2024_02, ... (월별 분할)

-- 수직 파티셔닝 (Vertical Partitioning)
-- 열 단위로 분할
-- 원본 테이블: users (id, name, email, profile_image, bio)
-- 분할 후:
-- users_basic (id, name, email)
-- users_profile (id, profile_image, bio)
```

---

## 📊 파티셔닝 유형

### RANGE 파티셔닝

> [!tip] 범위 기반 분할
> 날짜, 숫자 등 연속적인 값의 범위로 분할

```sql
-- 날짜 기반 RANGE 파티셔닝
CREATE TABLE sales (
    sale_id INT,
    sale_date DATE,
    product_id INT,
    amount DECIMAL(10,2)
)
PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- 월별 RANGE 파티셔닝
CREATE TABLE logs (
    log_id INT,
    log_date DATETIME,
    message TEXT
)
PARTITION BY RANGE (TO_DAYS(log_date)) (
    PARTITION p202401 VALUES LESS THAN (TO_DAYS('2024-02-01')),
    PARTITION p202402 VALUES LESS THAN (TO_DAYS('2024-03-01')),
    PARTITION p202403 VALUES LESS THAN (TO_DAYS('2024-04-01'))
);
```

### LIST 파티셔닝

```sql
-- 지역별 LIST 파티셔닝
CREATE TABLE customers (
    customer_id INT,
    name VARCHAR(50),
    region VARCHAR(20)
)
PARTITION BY LIST COLUMNS(region) (
    PARTITION p_asia VALUES IN ('Korea', 'Japan', 'China'),
    PARTITION p_europe VALUES IN ('UK', 'France', 'Germany'),
    PARTITION p_americas VALUES IN ('USA', 'Canada', 'Brazil'),
    PARTITION p_others VALUES IN ('Australia', 'Others')
);
```

### HASH 파티셔닝

```sql
-- HASH 파티셔닝 (균등 분산)
CREATE TABLE users (
    user_id INT,
    username VARCHAR(50),
    email VARCHAR(100)
)
PARTITION BY HASH(user_id)
PARTITIONS 8;  -- 8개 파티션으로 균등 분산

-- KEY 파티셔닝 (MySQL 특화)
CREATE TABLE sessions (
    session_id VARCHAR(32),
    user_id INT,
    created_at TIMESTAMP
)
PARTITION BY KEY(session_id)
PARTITIONS 4;
```

### COMPOSITE 파티셔닝

```sql
-- 복합 파티셔닝 (RANGE-HASH)
CREATE TABLE transactions (
    trans_id INT,
    trans_date DATE,
    account_id INT,
    amount DECIMAL(10,2)
)
PARTITION BY RANGE (YEAR(trans_date))
SUBPARTITION BY HASH(account_id)
SUBPARTITIONS 4 (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

---

## 🔧 파티션 관리

### 파티션 추가/삭제

```sql
-- 파티션 추가
ALTER TABLE sales ADD PARTITION (
    PARTITION p2025 VALUES LESS THAN (2026)
);

-- 파티션 삭제
ALTER TABLE sales DROP PARTITION p2022;

-- 파티션 재구성
ALTER TABLE sales REORGANIZE PARTITION p_future INTO (
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- 파티션 통합
ALTER TABLE sales REORGANIZE PARTITION p202401, p202402, p202403 INTO (
    PARTITION p2024q1 VALUES LESS THAN (TO_DAYS('2024-04-01'))
);
```

### 파티션 프루닝

```sql
-- 파티션 프루닝 확인
EXPLAIN PARTITIONS 
SELECT * FROM sales 
WHERE sale_date BETWEEN '2024-01-01' AND '2024-03-31';
-- partitions: p2024  (다른 파티션은 스캔하지 않음)

-- 파티션 정보 확인
SELECT 
    partition_name,
    table_rows,
    data_length,
    index_length
FROM information_schema.partitions
WHERE table_name = 'sales'
    AND table_schema = 'mydb';
```

---

## 🌐 샤딩 개념

### 샤딩이란?

> [!warning] Sharding 정의
> 샤딩은 데이터를 여러 물리적 데이터베이스 서버에 수평 분할하는 기법입니다.

### 샤딩 vs 파티셔닝

| 구분 | 파티셔닝 | 샤딩 |
|------|---------|------|
| **범위** | 단일 DB 서버 내 | 여러 DB 서버 |
| **트랜잭션** | 지원 | 제한적 |
| **JOIN** | 가능 | 어려움 |
| **확장성** | 수직적 확장 | 수평적 확장 |
| **복잡도** | 낮음 | 높음 |

### 샤딩 아키텍처

```
애플리케이션
     ↓
[샤드 라우터/프록시]
     ↓
샤드 맵핑 테이블
     ↓
┌─────┬─────┬─────┐
│Shard│Shard│Shard│
│  1  │  2  │  3  │
└─────┴─────┴─────┘
```

---

## 🔑 샤드 키 전략

### 샤드 키 선택 기준

> [!tip] 좋은 샤드 키의 조건
> - 균등한 데이터 분산
> - 쿼리 패턴과 일치
> - 변경이 거의 없음
> - 높은 카디널리티

### 해시 기반 샤딩

```python
# Python 예제: 해시 샤딩
import hashlib

class HashSharding:
    def __init__(self, shard_count):
        self.shard_count = shard_count
        self.shards = [f"shard_{i}" for i in range(shard_count)]
    
    def get_shard(self, shard_key):
        # 일관된 해시 생성
        hash_value = int(hashlib.md5(
            str(shard_key).encode()
        ).hexdigest(), 16)
        
        # 샤드 선택
        shard_index = hash_value % self.shard_count
        return self.shards[shard_index]
    
    def add_user(self, user_id, user_data):
        shard = self.get_shard(user_id)
        # shard 서버에 데이터 저장
        print(f"User {user_id} → {shard}")

# 사용 예제
sharding = HashSharding(4)
sharding.add_user(12345, {"name": "김철수"})  # → shard_1
sharding.add_user(67890, {"name": "이영희"})  # → shard_3
```

### 범위 기반 샤딩

```sql
-- 범위 샤딩 맵핑 테이블
CREATE TABLE shard_map (
    range_start INT,
    range_end INT,
    shard_id INT,
    server_host VARCHAR(100),
    PRIMARY KEY (range_start)
);

INSERT INTO shard_map VALUES
(0, 999999, 1, 'shard1.example.com'),
(1000000, 1999999, 2, 'shard2.example.com'),
(2000000, 2999999, 3, 'shard3.example.com');

-- 샤드 찾기 함수
DELIMITER //
CREATE FUNCTION get_shard(user_id INT)
RETURNS VARCHAR(100)
DETERMINISTIC
BEGIN
    DECLARE shard_host VARCHAR(100);
    SELECT server_host INTO shard_host
    FROM shard_map
    WHERE user_id >= range_start 
      AND user_id <= range_end;
    RETURN shard_host;
END//
DELIMITER ;
```

### 지역 기반 샤딩

```python
# 지역별 샤딩
class GeoSharding:
    def __init__(self):
        self.region_shards = {
            'KR': 'asia-shard-1.example.com',
            'JP': 'asia-shard-1.example.com',
            'CN': 'asia-shard-2.example.com',
            'US': 'us-shard-1.example.com',
            'EU': 'eu-shard-1.example.com'
        }
    
    def get_shard(self, country_code):
        return self.region_shards.get(
            country_code, 
            'default-shard.example.com'
        )
```

---

## ⚡ 샤딩 구현

### 애플리케이션 레벨 샤딩

```python
# Python: 애플리케이션 레벨 샤딩
import mysql.connector
from mysql.connector import pooling

class ShardedDatabase:
    def __init__(self, shard_configs):
        self.pools = {}
        self.shard_count = len(shard_configs)
        
        # 각 샤드별 연결 풀 생성
        for i, config in enumerate(shard_configs):
            self.pools[i] = pooling.MySQLConnectionPool(
                pool_name=f"shard_{i}",
                pool_size=5,
                **config
            )
    
    def get_shard_id(self, key):
        return hash(key) % self.shard_count
    
    def execute_on_shard(self, key, query, params=None):
        shard_id = self.get_shard_id(key)
        pool = self.pools[shard_id]
        
        conn = pool.get_connection()
        cursor = conn.cursor()
        
        try:
            cursor.execute(query, params)
            if query.strip().upper().startswith('SELECT'):
                result = cursor.fetchall()
                return result
            else:
                conn.commit()
                return cursor.lastrowid
        finally:
            cursor.close()
            conn.close()
    
    def cross_shard_query(self, query, params=None):
        """모든 샤드에서 쿼리 실행 후 결과 병합"""
        all_results = []
        
        for shard_id, pool in self.pools.items():
            conn = pool.get_connection()
            cursor = conn.cursor()
            
            try:
                cursor.execute(query, params)
                results = cursor.fetchall()
                all_results.extend(results)
            finally:
                cursor.close()
                conn.close()
        
        return all_results

# 사용 예제
shard_configs = [
    {'host': 'shard0.example.com', 'database': 'mydb'},
    {'host': 'shard1.example.com', 'database': 'mydb'},
    {'host': 'shard2.example.com', 'database': 'mydb'}
]

db = ShardedDatabase(shard_configs)

# 특정 샤드에 쿼리
user_id = 12345
result = db.execute_on_shard(
    user_id,
    "SELECT * FROM users WHERE user_id = %s",
    (user_id,)
)

# 모든 샤드 검색
all_users = db.cross_shard_query(
    "SELECT COUNT(*) FROM users"
)
```

### Vitess를 이용한 샤딩

```yaml
# Vitess 설정 예제
keyspaces:
  - name: user_keyspace
    sharding_column_name: user_id
    sharding_column_type: UINT64
    vindexes:
      hash:
        type: hash
    tables:
      users:
        column_vindexes:
          - column: user_id
            name: hash
```

---

## ⚠️ 샤딩 트레이드오프

### 샤딩의 문제점

> [!danger] 샤딩 복잡성
> 샤딩은 많은 이점이 있지만 복잡성도 크게 증가합니다.

### JOIN의 어려움

```python
# Cross-shard JOIN 구현 (비효율적)
def cross_shard_join(db, user_id, order_table):
    # 1. 사용자 정보 조회
    user = db.execute_on_shard(
        user_id,
        "SELECT * FROM users WHERE user_id = %s",
        (user_id,)
    )[0]
    
    # 2. 모든 샤드에서 주문 조회
    orders = db.cross_shard_query(
        f"SELECT * FROM {order_table} WHERE user_id = %s",
        (user_id,)
    )
    
    # 3. 애플리케이션에서 JOIN
    result = []
    for order in orders:
        result.append({
            'user': user,
            'order': order
        })
    
    return result
```

### 트랜잭션 처리

```python
# 분산 트랜잭션 (2PC - Two Phase Commit)
class DistributedTransaction:
    def __init__(self, sharded_db):
        self.db = sharded_db
        self.prepared_shards = []
    
    def execute(self, operations):
        try:
            # Phase 1: Prepare
            for op in operations:
                shard_id = self.db.get_shard_id(op['key'])
                # 각 샤드에 PREPARE 전송
                self.prepared_shards.append(shard_id)
            
            # Phase 2: Commit
            for shard_id in self.prepared_shards:
                # 각 샤드에 COMMIT 전송
                pass
                
        except Exception as e:
            # Rollback all shards
            for shard_id in self.prepared_shards:
                # 각 샤드에 ROLLBACK 전송
                pass
            raise e
```

### 리샤딩

```python
# 리샤딩 (Resharding) 전략
class ReshardingStrategy:
    def __init__(self, old_shard_count, new_shard_count):
        self.old_count = old_shard_count
        self.new_count = new_shard_count
    
    def get_new_shard(self, key):
        """일관된 해싱으로 리샤딩 최소화"""
        # Virtual nodes를 사용한 일관된 해싱
        import hashlib
        
        virtual_nodes = 150
        ring = {}
        
        for shard in range(self.new_count):
            for vnode in range(virtual_nodes):
                hash_key = f"{shard}:{vnode}"
                hash_val = int(hashlib.md5(
                    hash_key.encode()
                ).hexdigest(), 16)
                ring[hash_val] = shard
        
        key_hash = int(hashlib.md5(
            str(key).encode()
        ).hexdigest(), 16)
        
        # 가장 가까운 노드 찾기
        for hash_val in sorted(ring.keys()):
            if key_hash <= hash_val:
                return ring[hash_val]
        
        return ring[sorted(ring.keys())[0]]
```

### 샤딩 모범 사례

```sql
-- 1. AUTO_INCREMENT 충돌 방지
-- 샤드 1
SET @@auto_increment_increment = 3;
SET @@auto_increment_offset = 1;

-- 샤드 2
SET @@auto_increment_increment = 3;
SET @@auto_increment_offset = 2;

-- 2. 글로벌 유니크 ID 사용
-- UUID 또는 Snowflake ID 사용
CREATE TABLE users (
    user_id BINARY(16) PRIMARY KEY,  -- UUID
    username VARCHAR(50)
);

-- 3. 샤드 키 변경 방지
-- 샤드 키는 불변으로 설계
-- 변경이 필요하면 데이터 마이그레이션 필요
```

---

## 📚 참고자료

### 내부 문서
- [[08_database-index|데이터베이스 인덱스]]
- [[11_database-replication|데이터베이스 복제]]
- [[13_database-scaling|데이터베이스 확장]]

### 외부 리소스
- [MySQL Partitioning](https://dev.mysql.com/doc/refman/8.0/en/partitioning.html)
- [Vitess Documentation](https://vitess.io/docs/)
- [Consistent Hashing](https://www.toptal.com/big-data/consistent-hashing)
- [Database Sharding Explained](https://www.mongodb.com/features/database-sharding-explained)

### 추가 학습 주제
- Consistent Hashing
- Virtual Sharding
- Cross-shard Aggregation
- Distributed SQL (NewSQL)
- Data Migration Strategies

---

> [!quote]
> "샤딩은 무한한 확장성을 제공하지만, 복잡성이라는 대가를 치러야 합니다. 샤딩을 도입하기 전에 다른 최적화 방법을 먼저 고려하세요."