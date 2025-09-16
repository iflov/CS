---
tags:
  - database
  - scaling
  - performance
  - caching
  - optimization
created: 2025-01-08
updated: 2025-01-08
---

# 데이터베이스 확장(Scaling)

> [!info] 개요
> 데이터베이스 확장은 증가하는 부하와 데이터를 처리하기 위한 전략입니다. 수직적/수평적 확장, 캐싱, 연결 풀링 등 다양한 확장 기법을 학습합니다.

## 📑 목차

- [[#🎯 확장 전략]]
- [[#⬆️ 수직적 확장]]
- [[#➡️ 수평적 확장]]
- [[#💾 캐싱 전략]]
- [[#🔄 Connection Pooling]]
- [[#📊 로드 밸런싱]]
- [[#⚡ 성능 모니터링]]
- [[#📚 참고자료]]

---

## 🎯 확장 전략

### 확장의 필요성

> [!note] 왜 확장이 필요한가?
> - 사용자 증가로 인한 부하 증가
> - 데이터 양의 지속적인 증가
> - 응답 시간 요구사항
> - 고가용성 요구

### 확장 방식 비교

| 구분 | 수직적 확장 (Scale-Up) | 수평적 확장 (Scale-Out) |
|------|------------------------|-------------------------|
| **방법** | 서버 성능 향상 | 서버 대수 증가 |
| **비용** | 지수적 증가 | 선형적 증가 |
| **한계** | 하드웨어 한계 존재 | 이론적 무한 확장 |
| **복잡도** | 낮음 | 높음 |
| **다운타임** | 필요 | 불필요 |

---

## ⬆️ 수직적 확장

### 하드웨어 업그레이드

```typescript
// TypeScript: 성능 메트릭 모니터링
interface ServerMetrics {
  cpu: {
    usage: number;      // 0-100%
    cores: number;
    threads: number;
  };
  memory: {
    total: number;      // GB
    used: number;       // GB
    cached: number;     // GB
  };
  disk: {
    iops: number;       // I/O per second
    throughput: number; // MB/s
    latency: number;    // ms
  };
}

class PerformanceMonitor {
  private metrics: ServerMetrics[] = [];
  
  async collectMetrics(): Promise<ServerMetrics> {
    // 실제로는 시스템 메트릭 수집
    return {
      cpu: {
        usage: Math.random() * 100,
        cores: 16,
        threads: 32
      },
      memory: {
        total: 64,
        used: 45,
        cached: 15
      },
      disk: {
        iops: 10000,
        throughput: 500,
        latency: 0.5
      }
    };
  }
  
  needsUpgrade(): boolean {
    const recent = this.metrics.slice(-100);
    const avgCpu = recent.reduce((sum, m) => sum + m.cpu.usage, 0) / recent.length;
    const avgMemory = recent.reduce((sum, m) => sum + (m.memory.used / m.memory.total), 0) / recent.length;
    
    return avgCpu > 80 || avgMemory > 0.85;
  }
}
```

### 데이터베이스 튜닝

```sql
-- MySQL 성능 튜닝 파라미터
-- my.cnf 설정

[mysqld]
# 메모리 설정
innodb_buffer_pool_size = 48G  # 전체 RAM의 70-80%
innodb_buffer_pool_instances = 8
innodb_log_file_size = 2G
innodb_flush_log_at_trx_commit = 2

# 연결 설정
max_connections = 500
thread_cache_size = 50
table_open_cache = 4000

# 쿼리 캐시 (MySQL 5.7까지)
query_cache_type = 1
query_cache_size = 256M

# 임시 테이블
tmp_table_size = 512M
max_heap_table_size = 512M
```

---

## ➡️ 수평적 확장

### Read Replica 구성

```typescript
// TypeScript: 읽기/쓰기 분리 구현
import { Pool, PoolConfig } from 'pg';

interface DatabaseConfig {
  master: PoolConfig;
  replicas: PoolConfig[];
}

class DatabaseCluster {
  private master: Pool;
  private replicas: Pool[] = [];
  private currentReplica = 0;
  
  constructor(config: DatabaseConfig) {
    // 마스터 연결
    this.master = new Pool(config.master);
    
    // 레플리카 연결
    config.replicas.forEach(replicaConfig => {
      this.replicas.push(new Pool(replicaConfig));
    });
  }
  
  // 쓰기 작업
  async write(query: string, params?: any[]): Promise<any> {
    const client = await this.master.connect();
    try {
      const result = await client.query(query, params);
      return result.rows;
    } finally {
      client.release();
    }
  }
  
  // 읽기 작업 (라운드 로빈)
  async read(query: string, params?: any[]): Promise<any> {
    const replica = this.getNextReplica();
    const client = await replica.connect();
    try {
      const result = await client.query(query, params);
      return result.rows;
    } finally {
      client.release();
    }
  }
  
  private getNextReplica(): Pool {
    const replica = this.replicas[this.currentReplica];
    this.currentReplica = (this.currentReplica + 1) % this.replicas.length;
    return replica;
  }
  
  // 건강 체크
  async healthCheck(): Promise<Map<string, boolean>> {
    const health = new Map<string, boolean>();
    
    // 마스터 체크
    try {
      await this.master.query('SELECT 1');
      health.set('master', true);
    } catch {
      health.set('master', false);
    }
    
    // 레플리카 체크
    for (let i = 0; i < this.replicas.length; i++) {
      try {
        await this.replicas[i].query('SELECT 1');
        health.set(`replica_${i}`, true);
      } catch {
        health.set(`replica_${i}`, false);
      }
    }
    
    return health;
  }
}

// 사용 예제
const dbCluster = new DatabaseCluster({
  master: {
    host: 'master.db.example.com',
    database: 'myapp',
    user: 'dbuser',
    password: 'password',
    max: 20
  },
  replicas: [
    {
      host: 'replica1.db.example.com',
      database: 'myapp',
      user: 'dbuser',
      password: 'password',
      max: 20
    },
    {
      host: 'replica2.db.example.com',
      database: 'myapp',
      user: 'dbuser',
      password: 'password',
      max: 20
    }
  ]
});

// 쓰기 작업
await dbCluster.write(
  'INSERT INTO users (name, email) VALUES ($1, $2)',
  ['김철수', 'kim@example.com']
);

// 읽기 작업
const users = await dbCluster.read('SELECT * FROM users WHERE active = true');
```

---

## 💾 캐싱 전략

### Redis 캐싱 구현

```typescript
// TypeScript: Redis 캐싱 레이어
import Redis from 'ioredis';
import { createHash } from 'crypto';

interface CacheOptions {
  ttl?: number;  // Time to live in seconds
  prefix?: string;
}

class CacheLayer {
  private redis: Redis;
  private defaultTTL = 3600; // 1 hour
  
  constructor(redisUrl: string) {
    this.redis = new Redis(redisUrl);
  }
  
  // 캐시 키 생성
  private generateKey(query: string, params?: any[]): string {
    const hash = createHash('md5');
    hash.update(query);
    if (params) {
      hash.update(JSON.stringify(params));
    }
    return `sql:${hash.digest('hex')}`;
  }
  
  // 캐시 가져오기
  async get<T>(key: string): Promise<T | null> {
    const data = await this.redis.get(key);
    if (data) {
      return JSON.parse(data);
    }
    return null;
  }
  
  // 캐시 저장
  async set(key: string, value: any, ttl?: number): Promise<void> {
    const expiry = ttl || this.defaultTTL;
    await this.redis.setex(key, expiry, JSON.stringify(value));
  }
  
  // 캐시 무효화
  async invalidate(pattern: string): Promise<void> {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
  
  // Look-Aside 캐싱 패턴
  async lookAside<T>(
    key: string,
    fetcher: () => Promise<T>,
    ttl?: number
  ): Promise<T> {
    // 캐시 확인
    let data = await this.get<T>(key);
    
    if (data === null) {
      // 캐시 미스: 데이터 조회
      data = await fetcher();
      // 캐시 저장
      await this.set(key, data, ttl);
    }
    
    return data;
  }
  
  // Write-Through 캐싱 패턴
  async writeThrough<T>(
    key: string,
    value: T,
    writer: (value: T) => Promise<void>,
    ttl?: number
  ): Promise<void> {
    // 데이터베이스 쓰기
    await writer(value);
    // 캐시 업데이트
    await this.set(key, value, ttl);
  }
  
  // Write-Behind 캐싱 패턴 (비동기)
  async writeBehind<T>(
    key: string,
    value: T,
    writer: (value: T) => Promise<void>,
    ttl?: number
  ): Promise<void> {
    // 캐시 먼저 업데이트
    await this.set(key, value, ttl);
    
    // 비동기로 데이터베이스 업데이트
    setImmediate(() => {
      writer(value).catch(err => {
        console.error('Write-behind failed:', err);
        // 에러 처리 로직
      });
    });
  }
}

// 사용 예제
class UserService {
  private cache: CacheLayer;
  private db: DatabaseCluster;
  
  constructor(cache: CacheLayer, db: DatabaseCluster) {
    this.cache = cache;
    this.db = db;
  }
  
  async getUser(userId: number) {
    const key = `user:${userId}`;
    
    return this.cache.lookAside(
      key,
      async () => {
        const [user] = await this.db.read(
          'SELECT * FROM users WHERE id = $1',
          [userId]
        );
        return user;
      },
      300 // 5분 TTL
    );
  }
  
  async updateUser(userId: number, data: any) {
    const key = `user:${userId}`;
    
    await this.cache.writeThrough(
      key,
      data,
      async (userData) => {
        await this.db.write(
          'UPDATE users SET name = $1, email = $2 WHERE id = $3',
          [userData.name, userData.email, userId]
        );
      }
    );
    
    // 관련 캐시 무효화
    await this.cache.invalidate(`user:list:*`);
  }
}
```

### Memcached vs Redis

```typescript
// TypeScript: 캐시 추상화
interface CacheProvider {
  get(key: string): Promise<any>;
  set(key: string, value: any, ttl?: number): Promise<void>;
  delete(key: string): Promise<void>;
  flush(): Promise<void>;
}

class MemcachedProvider implements CacheProvider {
  // Memcached 구현
  async get(key: string): Promise<any> {
    // Memcached는 단순 key-value
    return null;
  }
  
  async set(key: string, value: any, ttl?: number): Promise<void> {
    // 최대 1MB 제한
  }
  
  async delete(key: string): Promise<void> {
    // 단순 삭제
  }
  
  async flush(): Promise<void> {
    // 전체 플러시
  }
}

class RedisProvider implements CacheProvider {
  // Redis 구현 - 더 많은 기능
  async get(key: string): Promise<any> {
    // Redis는 다양한 데이터 구조 지원
    return null;
  }
  
  async set(key: string, value: any, ttl?: number): Promise<void> {
    // 크기 제한 없음
  }
  
  async delete(key: string): Promise<void> {
    // 패턴 매칭 삭제 가능
  }
  
  async flush(): Promise<void> {
    // 데이터베이스별 플러시
  }
  
  // Redis 특화 기능
  async lpush(key: string, value: any): Promise<void> {}
  async zadd(key: string, score: number, value: any): Promise<void> {}
  async publish(channel: string, message: any): Promise<void> {}
}
```

---

## 🔄 Connection Pooling

### 연결 풀 구현

```typescript
// TypeScript: Connection Pool 구현
interface PoolOptions {
  min: number;
  max: number;
  idleTimeoutMillis: number;
  connectionTimeoutMillis: number;
}

class ConnectionPool<T> {
  private connections: T[] = [];
  private availableConnections: T[] = [];
  private waitingQueue: Array<(conn: T) => void> = [];
  private options: PoolOptions;
  
  constructor(
    private createConnection: () => Promise<T>,
    private destroyConnection: (conn: T) => Promise<void>,
    options: Partial<PoolOptions> = {}
  ) {
    this.options = {
      min: options.min || 2,
      max: options.max || 10,
      idleTimeoutMillis: options.idleTimeoutMillis || 30000,
      connectionTimeoutMillis: options.connectionTimeoutMillis || 5000
    };
    
    this.initialize();
  }
  
  private async initialize() {
    // 최소 연결 수만큼 생성
    for (let i = 0; i < this.options.min; i++) {
      const conn = await this.createConnection();
      this.connections.push(conn);
      this.availableConnections.push(conn);
    }
  }
  
  async acquire(): Promise<T> {
    // 사용 가능한 연결이 있으면 반환
    if (this.availableConnections.length > 0) {
      return this.availableConnections.pop()!;
    }
    
    // 최대 연결 수에 도달하지 않았으면 새로 생성
    if (this.connections.length < this.options.max) {
      const conn = await this.createConnection();
      this.connections.push(conn);
      return conn;
    }
    
    // 대기
    return new Promise((resolve) => {
      const timeout = setTimeout(() => {
        const index = this.waitingQueue.indexOf(resolve);
        if (index !== -1) {
          this.waitingQueue.splice(index, 1);
          throw new Error('Connection timeout');
        }
      }, this.options.connectionTimeoutMillis);
      
      this.waitingQueue.push((conn: T) => {
        clearTimeout(timeout);
        resolve(conn);
      });
    });
  }
  
  release(connection: T): void {
    // 대기 중인 요청이 있으면 할당
    if (this.waitingQueue.length > 0) {
      const waiter = this.waitingQueue.shift()!;
      waiter(connection);
    } else {
      // 풀에 반환
      this.availableConnections.push(connection);
    }
  }
  
  async destroy(): Promise<void> {
    // 모든 연결 종료
    await Promise.all(
      this.connections.map(conn => this.destroyConnection(conn))
    );
    this.connections = [];
    this.availableConnections = [];
  }
  
  getPoolStats() {
    return {
      total: this.connections.length,
      available: this.availableConnections.length,
      inUse: this.connections.length - this.availableConnections.length,
      waiting: this.waitingQueue.length
    };
  }
}

// HikariCP 스타일 설정
class HikariStylePool {
  private config = {
    minimumIdle: 10,
    maximumPoolSize: 20,
    connectionTimeout: 30000,
    idleTimeout: 600000,
    maxLifetime: 1800000,
    leakDetectionThreshold: 60000
  };
  
  async getConnection() {
    // Leak detection
    const start = Date.now();
    const conn = await this.pool.acquire();
    
    // Proxy로 감싸서 leak detection
    return new Proxy(conn, {
      get: (target, prop) => {
        if (prop === 'close') {
          return () => {
            const duration = Date.now() - start;
            if (duration > this.config.leakDetectionThreshold) {
              console.warn(`Possible connection leak detected: ${duration}ms`);
            }
            this.pool.release(conn);
          };
        }
        return target[prop];
      }
    });
  }
}
```

---

## 📊 로드 밸런싱

### 데이터베이스 프록시

```typescript
// TypeScript: ProxySQL 스타일 로드 밸런서
interface ServerConfig {
  host: string;
  port: number;
  weight: number;
  maxConnections: number;
}

interface QueryRule {
  pattern: RegExp;
  targetGroup: 'master' | 'replica';
  cache?: boolean;
  cacheTTL?: number;
}

class DatabaseProxy {
  private servers: Map<string, ServerConfig[]> = new Map();
  private queryRules: QueryRule[] = [];
  private connections: Map<string, any[]> = new Map();
  
  constructor() {
    this.servers.set('master', []);
    this.servers.set('replica', []);
  }
  
  addServer(group: 'master' | 'replica', config: ServerConfig) {
    const servers = this.servers.get(group) || [];
    servers.push(config);
    this.servers.set(group, servers);
  }
  
  addQueryRule(rule: QueryRule) {
    this.queryRules.push(rule);
    // 패턴 우선순위로 정렬
    this.queryRules.sort((a, b) => {
      if (a.pattern.source.includes('INSERT|UPDATE|DELETE')) return -1;
      if (b.pattern.source.includes('INSERT|UPDATE|DELETE')) return 1;
      return 0;
    });
  }
  
  private selectServer(group: string): ServerConfig | null {
    const servers = this.servers.get(group);
    if (!servers || servers.length === 0) return null;
    
    // Weighted round-robin
    const totalWeight = servers.reduce((sum, s) => sum + s.weight, 0);
    let random = Math.random() * totalWeight;
    
    for (const server of servers) {
      random -= server.weight;
      if (random <= 0) {
        return server;
      }
    }
    
    return servers[0];
  }
  
  async executeQuery(query: string, params?: any[]): Promise<any> {
    // 쿼리 규칙 매칭
    let targetGroup: 'master' | 'replica' = 'replica';
    let shouldCache = false;
    let cacheTTL = 0;
    
    for (const rule of this.queryRules) {
      if (rule.pattern.test(query)) {
        targetGroup = rule.targetGroup;
        shouldCache = rule.cache || false;
        cacheTTL = rule.cacheTTL || 0;
        break;
      }
    }
    
    // 서버 선택
    const server = this.selectServer(targetGroup);
    if (!server) {
      throw new Error(`No available server in group: ${targetGroup}`);
    }
    
    // 쿼리 실행
    console.log(`Routing to ${targetGroup}: ${server.host}:${server.port}`);
    
    // 실제 구현에서는 연결 풀에서 연결을 가져와 실행
    return { server, query, params };
  }
  
  // 헬스 체크
  async healthCheck(): Promise<Map<string, boolean>> {
    const health = new Map<string, boolean>();
    
    for (const [group, servers] of this.servers) {
      for (const server of servers) {
        const key = `${group}:${server.host}:${server.port}`;
        try {
          // 실제로는 SELECT 1 쿼리 실행
          health.set(key, true);
        } catch {
          health.set(key, false);
        }
      }
    }
    
    return health;
  }
}

// 사용 예제
const proxy = new DatabaseProxy();

// 서버 추가
proxy.addServer('master', {
  host: 'master.db.example.com',
  port: 3306,
  weight: 1000,
  maxConnections: 100
});

proxy.addServer('replica', {
  host: 'replica1.db.example.com',
  port: 3306,
  weight: 900,
  maxConnections: 200
});

proxy.addServer('replica', {
  host: 'replica2.db.example.com',
  port: 3306,
  weight: 800,
  maxConnections: 200
});

// 쿼리 규칙 추가
proxy.addQueryRule({
  pattern: /^(INSERT|UPDATE|DELETE)/i,
  targetGroup: 'master'
});

proxy.addQueryRule({
  pattern: /^SELECT.*FROM\s+users/i,
  targetGroup: 'replica',
  cache: true,
  cacheTTL: 300
});

// 쿼리 실행
await proxy.executeQuery('SELECT * FROM users WHERE id = ?', [1]);
await proxy.executeQuery('UPDATE users SET name = ? WHERE id = ?', ['김철수', 1]);
```

---

## ⚡ 성능 모니터링

### 모니터링 메트릭

```typescript
// TypeScript: 성능 모니터링 시스템
interface QueryMetrics {
  query: string;
  duration: number;
  rows: number;
  timestamp: Date;
}

interface PerformanceMetrics {
  qps: number;          // Queries per second
  avgLatency: number;   // ms
  p95Latency: number;   // 95 percentile
  p99Latency: number;   // 99 percentile
  errorRate: number;    // percentage
  activeConnections: number;
  slowQueries: QueryMetrics[];
}

class PerformanceMonitor {
  private queryMetrics: QueryMetrics[] = [];
  private slowQueryThreshold = 1000; // 1초
  
  recordQuery(query: string, duration: number, rows: number) {
    const metric: QueryMetrics = {
      query,
      duration,
      rows,
      timestamp: new Date()
    };
    
    this.queryMetrics.push(metric);
    
    // 슬로우 쿼리 로깅
    if (duration > this.slowQueryThreshold) {
      console.warn(`Slow query detected: ${duration}ms`, query);
    }
    
    // 메모리 관리: 최근 10000개만 유지
    if (this.queryMetrics.length > 10000) {
      this.queryMetrics = this.queryMetrics.slice(-10000);
    }
  }
  
  getMetrics(): PerformanceMetrics {
    const now = new Date();
    const oneMinuteAgo = new Date(now.getTime() - 60000);
    
    // 최근 1분간의 메트릭
    const recentMetrics = this.queryMetrics.filter(
      m => m.timestamp > oneMinuteAgo
    );
    
    // QPS 계산
    const qps = recentMetrics.length / 60;
    
    // 지연 시간 계산
    const latencies = recentMetrics.map(m => m.duration).sort((a, b) => a - b);
    const avgLatency = latencies.reduce((sum, l) => sum + l, 0) / latencies.length || 0;
    const p95Index = Math.floor(latencies.length * 0.95);
    const p99Index = Math.floor(latencies.length * 0.99);
    
    // 슬로우 쿼리
    const slowQueries = recentMetrics
      .filter(m => m.duration > this.slowQueryThreshold)
      .sort((a, b) => b.duration - a.duration)
      .slice(0, 10);
    
    return {
      qps,
      avgLatency,
      p95Latency: latencies[p95Index] || 0,
      p99Latency: latencies[p99Index] || 0,
      errorRate: 0, // 실제로는 에러 추적 필요
      activeConnections: 0, // 연결 풀에서 가져옴
      slowQueries
    };
  }
  
  // Prometheus 형식 메트릭 출력
  getPrometheusMetrics(): string {
    const metrics = this.getMetrics();
    
    return `
# HELP db_qps Queries per second
# TYPE db_qps gauge
db_qps ${metrics.qps}

# HELP db_latency_avg Average query latency in milliseconds
# TYPE db_latency_avg gauge
db_latency_avg ${metrics.avgLatency}

# HELP db_latency_p95 95th percentile query latency
# TYPE db_latency_p95 gauge
db_latency_p95 ${metrics.p95Latency}

# HELP db_latency_p99 99th percentile query latency
# TYPE db_latency_p99 gauge
db_latency_p99 ${metrics.p99Latency}

# HELP db_slow_queries Number of slow queries
# TYPE db_slow_queries counter
db_slow_queries ${metrics.slowQueries.length}
    `.trim();
  }
}

// EXPLAIN 분석
class QueryAnalyzer {
  async analyzeQuery(query: string): Promise<any> {
    // EXPLAIN 실행
    const explainQuery = `EXPLAIN ANALYZE ${query}`;
    
    // 결과 분석
    return {
      query,
      executionPlan: '...',
      estimatedCost: 100,
      actualTime: 50,
      rowsExamined: 1000,
      indexUsed: true,
      suggestions: [
        'Consider adding index on column X',
        'Query uses filesort, consider ORDER BY optimization'
      ]
    };
  }
}
```

---

## 📚 참고자료

### 내부 문서
- [[08_database-index|데이터베이스 인덱스]]
- [[11_database-replication|데이터베이스 복제]]
- [[12_partitioning-sharding|파티셔닝과 샤딩]]

### 외부 리소스
- [High Performance MySQL](https://www.oreilly.com/library/view/high-performance-mysql/9781492080503/)
- [Database Reliability Engineering](https://www.oreilly.com/library/view/database-reliability-engineering/9781491925935/)
- [ProxySQL Documentation](https://proxysql.com/documentation/)
- [Redis Documentation](https://redis.io/documentation)

### 추가 학습 주제
- Query Optimization
- Database Monitoring Tools
- Auto-scaling Strategies
- Cloud Database Services
- Database Migration Strategies

---

> [!quote]
> "확장성은 처음부터 고려해야 하지만, 과도한 최적화는 오히려 독이 될 수 있습니다. 측정하고, 분석하고, 필요한 만큼만 확장하세요."