---
tags:
  - database
  - replication
  - high-availability
  - scaling
  - master-slave
created: 2025-01-08
updated: 2025-01-08
---

# 데이터베이스 복제(Replication)

> [!info] 개요
> 데이터베이스 복제는 마스터 DB의 데이터를 하나 이상의 슬레이브 DB로 복사하는 기술입니다. 고가용성, 읽기 성능 향상, 백업 등 다양한 목적으로 사용됩니다.

## 📑 목차

- [[#🎯 복제 개념]]
- [[#🏗️ 복제 아키텍처]]
- [[#⚙️ 복제 메커니즘]]
- [[#📊 복제 구성]]
- [[#⚡ 읽기 분산]]
- [[#⚠️ 복제 지연]]
- [[#🔄 Multi-Master]]
- [[#📚 참고자료]]

---

## 🎯 복제 개념

### 복제란?

> [!note] Replication 정의
> 복제는 마스터 데이터베이스의 변경사항을 하나 이상의 슬레이브 데이터베이스에 자동으로 동기화하는 프로세스입니다.

### 복제의 장점

| 장점 | 설명 |
|------|------|
| **읽기 성능 향상** | 읽기 작업을 여러 슬레이브로 분산 |
| **고가용성** | 마스터 장애 시 슬레이브로 전환 |
| **백업** | 슬레이브에서 백업 수행으로 마스터 부하 감소 |
| **지리적 분산** | 여러 지역에 복제본 배치 |
| **분석 워크로드 분리** | 분석 쿼리를 슬레이브에서 실행 |

### 복제의 단점

```sql
-- 복제의 한계
-- 1. 쓰기 성능은 개선되지 않음 (마스터만 쓰기)
-- 2. 복제 지연 발생 가능
-- 3. 데이터 일관성 문제 (Eventually Consistent)
-- 4. 설정 및 관리 복잡도 증가
```

---

## 🏗️ 복제 아키텍처

### Master-Slave 구조

> [!tip] 기본 복제 구조
> 하나의 마스터가 쓰기를 담당하고, 여러 슬레이브가 읽기를 담당합니다.

```
         [Master]
         /   |   \
        /    |    \
   [Slave1] [Slave2] [Slave3]
```

### 복제 토폴로지

```sql
-- 1. 단순 Master-Slave
Master → Slave1
      → Slave2
      → Slave3

-- 2. 계층적 복제 (Cascading)
Master → Slave1 → Slave2
                → Slave3

-- 3. 원형 복제 (Circular)
Master1 → Master2 → Master3 → Master1

-- 4. 스타 토폴로지
        Slave1
           ↑
    Master → Slave2
           ↓
        Slave3
```

---

## ⚙️ 복제 메커니즘

### 바이너리 로그

> [!warning] Binary Log
> MySQL 복제의 핵심은 바이너리 로그입니다. 모든 변경 쿼리가 기록됩니다.

```sql
-- 바이너리 로그 설정 (my.cnf)
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW  -- STATEMENT, ROW, MIXED
expire_logs_days = 7
max_binlog_size = 100M

-- 바이너리 로그 확인
SHOW BINARY LOGS;
SHOW MASTER STATUS;

-- 특정 바이너리 로그 내용 보기
SHOW BINLOG EVENTS IN 'mysql-bin.000001' LIMIT 10;
```

### 복제 프로세스

```sql
-- 1. 마스터에서 변경 발생
INSERT INTO users (name, email) VALUES ('김철수', 'kim@example.com');

-- 2. 바이너리 로그에 기록
-- mysql-bin.000001 파일에 이벤트 저장

-- 3. 슬레이브 I/O Thread가 바이너리 로그 읽기
-- 릴레이 로그(relay log)에 저장

-- 4. 슬레이브 SQL Thread가 릴레이 로그 실행
-- 슬레이브 데이터베이스에 변경 적용
```

### 복제 형식

```sql
-- STATEMENT 기반 복제
-- SQL 문장 자체를 복제
SET binlog_format = 'STATEMENT';
-- 장점: 로그 크기 작음
-- 단점: 비결정적 함수 문제 (NOW(), RAND())

-- ROW 기반 복제
-- 변경된 행 데이터를 복제
SET binlog_format = 'ROW';
-- 장점: 정확한 복제
-- 단점: 로그 크기 큼

-- MIXED 복제
-- 상황에 따라 STATEMENT/ROW 자동 선택
SET binlog_format = 'MIXED';
```

---

## 📊 복제 구성

### 마스터 설정

```sql
-- 1. 마스터 서버 설정 (my.cnf)
[mysqld]
server-id = 1
log_bin = mysql-bin
binlog_do_db = mydb  -- 특정 DB만 복제

-- 2. 복제 사용자 생성
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
FLUSH PRIVILEGES;

-- 3. 마스터 상태 확인
SHOW MASTER STATUS;
-- File: mysql-bin.000001
-- Position: 154
```

### 슬레이브 설정

```sql
-- 1. 슬레이브 서버 설정 (my.cnf)
[mysqld]
server-id = 2
relay_log = relay-bin
read_only = 1  -- 읽기 전용

-- 2. 마스터 정보 설정
CHANGE MASTER TO
    MASTER_HOST='master_ip',
    MASTER_USER='repl_user',
    MASTER_PASSWORD='password',
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=154;

-- 3. 복제 시작
START SLAVE;

-- 4. 복제 상태 확인
SHOW SLAVE STATUS\G
-- Slave_IO_Running: Yes
-- Slave_SQL_Running: Yes
-- Seconds_Behind_Master: 0
```

### GTID 기반 복제

```sql
-- GTID (Global Transaction ID) 설정
-- my.cnf
[mysqld]
gtid_mode = ON
enforce_gtid_consistency = ON

-- GTID 기반 복제 설정
CHANGE MASTER TO
    MASTER_HOST='master_ip',
    MASTER_USER='repl_user',
    MASTER_PASSWORD='password',
    MASTER_AUTO_POSITION=1;

-- GTID 확인
SHOW GLOBAL VARIABLES LIKE 'gtid_executed';
```

---

## ⚡ 읽기 분산

### 읽기/쓰기 분리

> [!example] Read/Write Split
> 애플리케이션에서 읽기와 쓰기를 분리하여 부하를 분산합니다.

```python
# Python 예제: 읽기/쓰기 분리
import mysql.connector
from mysql.connector import pooling

# 마스터 연결 풀 (쓰기용)
master_config = {
    'host': 'master.example.com',
    'user': 'app_user',
    'password': 'password',
    'database': 'mydb'
}
master_pool = pooling.MySQLConnectionPool(
    pool_name="master_pool",
    pool_size=5,
    **master_config
)

# 슬레이브 연결 풀 (읽기용)
slave_configs = [
    {'host': 'slave1.example.com'},
    {'host': 'slave2.example.com'},
    {'host': 'slave3.example.com'}
]

slave_pools = []
for i, config in enumerate(slave_configs):
    pool = pooling.MySQLConnectionPool(
        pool_name=f"slave_pool_{i}",
        pool_size=5,
        user='app_user',
        password='password',
        database='mydb',
        **config
    )
    slave_pools.append(pool)

# 쓰기 작업
def write_data(query, params):
    conn = master_pool.get_connection()
    cursor = conn.cursor()
    cursor.execute(query, params)
    conn.commit()
    cursor.close()
    conn.close()

# 읽기 작업 (라운드 로빈)
import random
def read_data(query, params):
    pool = random.choice(slave_pools)
    conn = pool.get_connection()
    cursor = conn.cursor()
    cursor.execute(query, params)
    result = cursor.fetchall()
    cursor.close()
    conn.close()
    return result
```

### ProxySQL을 이용한 읽기 분산

```sql
-- ProxySQL 설정
-- 1. 서버 그룹 설정
INSERT INTO mysql_servers(hostgroup_id, hostname, port, weight) VALUES
(0, 'master.example.com', 3306, 1000),  -- 쓰기 그룹
(1, 'slave1.example.com', 3306, 900),   -- 읽기 그룹
(1, 'slave2.example.com', 3306, 900),
(1, 'slave3.example.com', 3306, 800);

-- 2. 쿼리 규칙 설정
INSERT INTO mysql_query_rules(rule_id, match_pattern, destination_hostgroup) VALUES
(1, '^SELECT.*', 1),     -- SELECT는 읽기 그룹으로
(2, '^INSERT|UPDATE|DELETE.*', 0);  -- DML은 쓰기 그룹으로

LOAD MYSQL SERVERS TO RUNTIME;
LOAD MYSQL QUERY RULES TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;
SAVE MYSQL QUERY RULES TO DISK;
```

---

## ⚠️ 복제 지연

### 복제 지연 원인

> [!danger] Replication Lag
> 마스터와 슬레이브 간 데이터 동기화 지연

```sql
-- 복제 지연 확인
SHOW SLAVE STATUS\G
-- Seconds_Behind_Master: 10  -- 10초 지연

-- 복제 지연 원인
-- 1. 네트워크 지연
-- 2. 슬레이브 하드웨어 성능 부족
-- 3. 대용량 트랜잭션
-- 4. 슬레이브의 과도한 읽기 부하
-- 5. 단일 스레드 SQL 실행 (MySQL 5.6 이전)
```

### 복제 지연 해결

```sql
-- 1. 병렬 복제 활성화 (MySQL 5.7+)
SET GLOBAL slave_parallel_workers = 4;
SET GLOBAL slave_parallel_type = 'LOGICAL_CLOCK';

-- 2. 슬레이브 성능 튜닝
SET GLOBAL innodb_flush_log_at_trx_commit = 2;
SET GLOBAL sync_binlog = 0;

-- 3. 읽기 부하 분산
-- 지연이 큰 슬레이브는 읽기에서 제외

-- 4. 반동기 복제 (Semi-Sync Replication)
-- 플러그인 설치
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

-- 마스터 설정
SET GLOBAL rpl_semi_sync_master_enabled = 1;
SET GLOBAL rpl_semi_sync_master_timeout = 1000;  -- 1초

-- 슬레이브 설정
SET GLOBAL rpl_semi_sync_slave_enabled = 1;
```

### 복제 지연 모니터링

```sql
-- 복제 지연 모니터링 쿼리
SELECT 
    TIMESTAMPDIFF(SECOND, 
        TIMESTAMP(SUBSTRING_INDEX(SUBSTRING_INDEX(
            (SELECT processlist_info 
             FROM performance_schema.threads 
             WHERE name = 'thread/sql/slave_sql'), 
        'timestamp=', -1), ' ', 1)),
        NOW()) AS replication_delay_seconds;

-- 복제 상태 상세 확인
SELECT * FROM performance_schema.replication_applier_status;
SELECT * FROM performance_schema.replication_connection_status;
```

---

## 🔄 Multi-Master

### Multi-Master 복제

> [!note] Multi-Master Replication
> 여러 노드가 동시에 읽기/쓰기를 처리할 수 있는 구조

```sql
-- Galera Cluster (MySQL/MariaDB)
-- 동기식 복제, 모든 노드가 동일한 데이터 보유

-- 설정 예제 (my.cnf)
[mysqld]
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_address=gcomm://192.168.1.1,192.168.1.2,192.168.1.3
wsrep_cluster_name='my_cluster'
wsrep_node_address='192.168.1.1'
wsrep_sst_method=rsync
binlog_format=ROW
```

### 충돌 해결

```sql
-- Multi-Master에서 충돌 가능성
-- 1. 동일 행 동시 수정
-- 2. AUTO_INCREMENT 충돌
-- 3. Unique 제약 위반

-- AUTO_INCREMENT 충돌 방지
-- 서버 1
SET @@auto_increment_increment = 3;
SET @@auto_increment_offset = 1;
-- ID: 1, 4, 7, 10...

-- 서버 2
SET @@auto_increment_increment = 3;
SET @@auto_increment_offset = 2;
-- ID: 2, 5, 8, 11...

-- 서버 3
SET @@auto_increment_increment = 3;
SET @@auto_increment_offset = 3;
-- ID: 3, 6, 9, 12...
```

---

## 📚 참고자료

### 내부 문서
- [[06_transaction-acid|트랜잭션과 ACID]]
- [[12_partitioning-sharding|파티셔닝과 샤딩]]
- [[13_database-scaling|데이터베이스 확장]]

### 외부 리소스
- [MySQL Replication Documentation](https://dev.mysql.com/doc/refman/8.0/en/replication.html)
- [PostgreSQL Streaming Replication](https://www.postgresql.org/docs/current/warm-standby.html)
- [Galera Cluster Documentation](https://galeracluster.com/library/documentation/)

### 추가 학습 주제
- Group Replication
- Delayed Replication
- Point-in-Time Recovery
- Cross-Region Replication
- Change Data Capture (CDC)

---

> [!quote]
> "복제는 데이터베이스 고가용성과 성능 향상의 핵심입니다. 하지만 복제 지연과 데이터 일관성 사이의 균형을 찾는 것이 중요합니다."