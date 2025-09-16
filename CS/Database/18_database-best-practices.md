---
tags:
  - database
  - best-practices
  - design-patterns
  - optimization
  - maintenance
  - monitoring
  - architecture
created: 2025-01-08
updated: 2025-01-08
---

# 데이터베이스 모범 사례 (Database Best Practices)

> [!info] 개요
> 데이터베이스 설계, 개발, 운영에서 검증된 모범 사례들을 종합적으로 다룹니다. 이러한 원칙들을 따르면 확장 가능하고, 유지보수가 쉬우며, 성능이 뛰어난 데이터베이스 시스템을 구축할 수 있습니다.

## 📑 목차

- [[#⚡ 빠른 시작]]
- [[#🔧 설계 모범 사례]]
- [[#💡 개발 모범 사례]]
- [[#🚀 운영 모범 사례]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## ⚡ 빠른 시작

### 핵심 원칙 체크리스트

> [!tip] 10가지 핵심 원칙
> - [ ] **정규화와 역정규화 균형**: 3NF를 기본으로 하되 성능을 위해 선택적 역정규화
> - [ ] **인덱스 전략**: 쿼리 패턴 분석 후 최적화된 인덱스 설계
> - [ ] **명명 규칙 일관성**: 테이블, 컬럼, 인덱스 명명 규칙 표준화
> - [ ] **트랜잭션 관리**: ACID 원칙 준수와 적절한 격리 수준 설정
> - [ ] **백업 전략**: 정기적인 백업과 복구 테스트
> - [ ] **모니터링**: 성능 메트릭 실시간 모니터링
> - [ ] **보안**: 최소 권한 원칙과 데이터 암호화
> - [ ] **문서화**: 스키마, API, 운영 절차 문서화
> - [ ] **버전 관리**: 스키마 변경 이력 관리
> - [ ] **테스트**: 단위 테스트와 통합 테스트 구현

### TypeScript 베스트 프랙티스 프레임워크

```typescript
import mysql from 'mysql2/promise';
import { z } from 'zod';

// 1. 연결 풀 관리
class DatabaseConnectionPool {
  private static instance: DatabaseConnectionPool;
  private pool: mysql.Pool;

  private constructor(config: mysql.PoolOptions) {
    this.pool = mysql.createPool({
      ...config,
      waitForConnections: true,
      connectionLimit: 10,
      maxIdle: 10,
      idleTimeout: 60000,
      queueLimit: 0,
      enableKeepAlive: true,
      keepAliveInitialDelay: 0
    });
  }

  static getInstance(config?: mysql.PoolOptions): DatabaseConnectionPool {
    if (!DatabaseConnectionPool.instance) {
      if (!config) throw new Error('Config required for first initialization');
      DatabaseConnectionPool.instance = new DatabaseConnectionPool(config);
    }
    return DatabaseConnectionPool.instance;
  }

  async getConnection(): Promise<mysql.PoolConnection> {
    return await this.pool.getConnection();
  }

  async execute<T>(
    query: string,
    params?: any[]
  ): Promise<T[]> {
    const [rows] = await this.pool.execute(query, params);
    return rows as T[];
  }

  async transaction<T>(
    callback: (connection: mysql.PoolConnection) => Promise<T>
  ): Promise<T> {
    const connection = await this.getConnection();
    
    try {
      await connection.beginTransaction();
      const result = await callback(connection);
      await connection.commit();
      return result;
    } catch (error) {
      await connection.rollback();
      throw error;
    } finally {
      connection.release();
    }
  }

  async close(): Promise<void> {
    await this.pool.end();
  }
}

// 2. 스키마 검증
const UserSchema = z.object({
  id: z.number().positive(),
  username: z.string().min(3).max(50),
  email: z.string().email(),
  created_at: z.date(),
  updated_at: z.date()
});

type User = z.infer<typeof UserSchema>;

// 3. Repository 패턴
abstract class BaseRepository<T> {
  constructor(
    protected db: DatabaseConnectionPool,
    protected tableName: string,
    protected schema: z.ZodSchema<T>
  ) {}

  async findById(id: number): Promise<T | null> {
    const rows = await this.db.execute<T>(
      `SELECT * FROM ${this.tableName} WHERE id = ?`,
      [id]
    );
    
    if (rows.length === 0) return null;
    
    return this.schema.parse(rows[0]);
  }

  async findAll(
    limit: number = 100,
    offset: number = 0
  ): Promise<T[]> {
    const rows = await this.db.execute<T>(
      `SELECT * FROM ${this.tableName} LIMIT ? OFFSET ?`,
      [limit, offset]
    );
    
    return rows.map(row => this.schema.parse(row));
  }

  async create(data: Partial<T>): Promise<number> {
    const validated = this.schema.partial().parse(data);
    const columns = Object.keys(validated);
    const values = Object.values(validated);
    const placeholders = columns.map(() => '?').join(', ');
    
    const [result] = await this.db.execute<any>(
      `INSERT INTO ${this.tableName} (${columns.join(', ')})
       VALUES (${placeholders})`,
      values
    );
    
    return result.insertId;
  }

  async update(id: number, data: Partial<T>): Promise<boolean> {
    const validated = this.schema.partial().parse(data);
    const updates = Object.keys(validated)
      .map(key => `${key} = ?`)
      .join(', ');
    const values = [...Object.values(validated), id];
    
    const [result] = await this.db.execute<any>(
      `UPDATE ${this.tableName} SET ${updates} WHERE id = ?`,
      values
    );
    
    return result.affectedRows > 0;
  }

  async delete(id: number): Promise<boolean> {
    const [result] = await this.db.execute<any>(
      `DELETE FROM ${this.tableName} WHERE id = ?`,
      [id]
    );
    
    return result.affectedRows > 0;
  }
}

// 4. 쿼리 빌더
class QueryBuilder<T> {
  private selectClause: string[] = ['*'];
  private whereConditions: string[] = [];
  private joinClauses: string[] = [];
  private orderByClause?: string;
  private limitValue?: number;
  private offsetValue?: number;
  private params: any[] = [];

  constructor(private tableName: string) {}

  select(...columns: string[]): this {
    this.selectClause = columns;
    return this;
  }

  where(condition: string, value?: any): this {
    this.whereConditions.push(condition);
    if (value !== undefined) {
      this.params.push(value);
    }
    return this;
  }

  join(
    type: 'INNER' | 'LEFT' | 'RIGHT',
    table: string,
    on: string
  ): this {
    this.joinClauses.push(`${type} JOIN ${table} ON ${on}`);
    return this;
  }

  orderBy(column: string, direction: 'ASC' | 'DESC' = 'ASC'): this {
    this.orderByClause = `${column} ${direction}`;
    return this;
  }

  limit(value: number): this {
    this.limitValue = value;
    return this;
  }

  offset(value: number): this {
    this.offsetValue = value;
    return this;
  }

  build(): { query: string; params: any[] } {
    let query = `SELECT ${this.selectClause.join(', ')} FROM ${this.tableName}`;
    
    if (this.joinClauses.length > 0) {
      query += ` ${this.joinClauses.join(' ')}`;
    }
    
    if (this.whereConditions.length > 0) {
      query += ` WHERE ${this.whereConditions.join(' AND ')}`;
    }
    
    if (this.orderByClause) {
      query += ` ORDER BY ${this.orderByClause}`;
    }
    
    if (this.limitValue !== undefined) {
      query += ` LIMIT ${this.limitValue}`;
    }
    
    if (this.offsetValue !== undefined) {
      query += ` OFFSET ${this.offsetValue}`;
    }
    
    return { query, params: this.params };
  }
}
```

---

## 🔧 설계 모범 사례

### 1. 스키마 설계 원칙

> [!note] 설계 지침
> - **적절한 데이터 타입 선택**: 저장 공간과 성능 최적화
> - **제약조건 활용**: CHECK, UNIQUE, NOT NULL 적극 활용
> - **관계 명확화**: 외래 키로 참조 무결성 보장
> - **시간 데이터 표준화**: UTC 사용, 타임존 별도 저장

```typescript
// 스키마 마이그레이션 관리
class SchemaMigrationManager {
  private migrations: Map<number, Migration> = new Map();

  constructor(private db: DatabaseConnectionPool) {}

  // 마이그레이션 테이블 생성
  async initialize(): Promise<void> {
    await this.db.execute(`
      CREATE TABLE IF NOT EXISTS schema_migrations (
        version INT PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        execution_time_ms INT,
        checksum VARCHAR(64)
      )
    `);
  }

  // 마이그레이션 등록
  register(migration: Migration): void {
    this.migrations.set(migration.version, migration);
  }

  // 마이그레이션 실행
  async migrate(): Promise<void> {
    const executed = await this.getExecutedMigrations();
    const pending = Array.from(this.migrations.entries())
      .filter(([version]) => !executed.includes(version))
      .sort(([a], [b]) => a - b);

    for (const [version, migration] of pending) {
      await this.executeMigration(version, migration);
    }
  }

  private async executeMigration(
    version: number,
    migration: Migration
  ): Promise<void> {
    const startTime = Date.now();
    
    await this.db.transaction(async (connection) => {
      // 마이그레이션 실행
      await migration.up(connection);
      
      // 기록
      const executionTime = Date.now() - startTime;
      const checksum = this.calculateChecksum(migration.sql);
      
      await connection.execute(
        `INSERT INTO schema_migrations (version, name, execution_time_ms, checksum)
         VALUES (?, ?, ?, ?)`,
        [version, migration.name, executionTime, checksum]
      );
    });

    console.log(`✅ Migration ${version}: ${migration.name} completed`);
  }

  private async getExecutedMigrations(): Promise<number[]> {
    const rows = await this.db.execute<{ version: number }>(
      'SELECT version FROM schema_migrations'
    );
    
    return rows.map(row => row.version);
  }

  private calculateChecksum(sql: string): string {
    const crypto = require('crypto');
    return crypto.createHash('sha256').update(sql).digest('hex');
  }

  // 롤백 기능
  async rollback(targetVersion?: number): Promise<void> {
    const executed = await this.getExecutedMigrations();
    const toRollback = targetVersion 
      ? executed.filter(v => v > targetVersion)
      : [Math.max(...executed)];

    for (const version of toRollback.sort((a, b) => b - a)) {
      const migration = this.migrations.get(version);
      if (!migration) {
        throw new Error(`Migration ${version} not found`);
      }

      await this.db.transaction(async (connection) => {
        await migration.down(connection);
        await connection.execute(
          'DELETE FROM schema_migrations WHERE version = ?',
          [version]
        );
      });

      console.log(`↩️ Rolled back migration ${version}: ${migration.name}`);
    }
  }
}

interface Migration {
  version: number;
  name: string;
  sql: string;
  up: (connection: mysql.PoolConnection) => Promise<void>;
  down: (connection: mysql.PoolConnection) => Promise<void>;
}

// 마이그레이션 예제
const migration001: Migration = {
  version: 1,
  name: 'create_users_table',
  sql: `
    CREATE TABLE users (
      id INT AUTO_INCREMENT PRIMARY KEY,
      username VARCHAR(50) UNIQUE NOT NULL,
      email VARCHAR(255) UNIQUE NOT NULL,
      password_hash VARCHAR(255) NOT NULL,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
      updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
      INDEX idx_username (username),
      INDEX idx_email (email),
      INDEX idx_created_at (created_at)
    )
  `,
  up: async (connection) => {
    await connection.execute(migration001.sql);
  },
  down: async (connection) => {
    await connection.execute('DROP TABLE IF EXISTS users');
  }
};
```

### 2. 인덱스 설계 전략

```typescript
// 인덱스 분석 및 추천 시스템
class IndexAdvisor {
  constructor(private db: DatabaseConnectionPool) {}

  // 슬로우 쿼리 분석
  async analyzeSlowQueries(
    threshold: number = 1000 // ms
  ): Promise<Array<{
    query: string;
    executionTime: number;
    recommendations: string[];
  }>> {
    const slowQueries = await this.db.execute<any>(`
      SELECT 
        query,
        execution_time_ms,
        rows_examined,
        rows_sent
      FROM performance_schema.events_statements_history_long
      WHERE execution_time_ms > ?
      ORDER BY execution_time_ms DESC
      LIMIT 100
    `, [threshold]);

    const recommendations = [];
    
    for (const query of slowQueries) {
      const explain = await this.explainQuery(query.query);
      const suggestion = this.generateIndexRecommendation(explain);
      
      recommendations.push({
        query: query.query,
        executionTime: query.execution_time_ms,
        recommendations: suggestion
      });
    }

    return recommendations;
  }

  // 쿼리 실행 계획 분석
  private async explainQuery(query: string): Promise<any[]> {
    try {
      return await this.db.execute(`EXPLAIN ${query}`);
    } catch (error) {
      console.error('Failed to explain query:', error);
      return [];
    }
  }

  // 인덱스 추천 생성
  private generateIndexRecommendation(explain: any[]): string[] {
    const recommendations: string[] = [];

    for (const row of explain) {
      // 풀 테이블 스캔 감지
      if (row.type === 'ALL') {
        recommendations.push(
          `Consider adding index on ${row.table} for WHERE clause columns`
        );
      }

      // 파일 정렬 감지
      if (row.Extra && row.Extra.includes('Using filesort')) {
        recommendations.push(
          `Consider adding index for ORDER BY columns on ${row.table}`
        );
      }

      // 임시 테이블 사용 감지
      if (row.Extra && row.Extra.includes('Using temporary')) {
        recommendations.push(
          `Consider optimizing GROUP BY or DISTINCT on ${row.table}`
        );
      }

      // 인덱스 미사용 감지
      if (!row.key) {
        recommendations.push(
          `No index used for ${row.table}. Consider adding appropriate index`
        );
      }
    }

    return recommendations;
  }

  // 중복 인덱스 찾기
  async findDuplicateIndexes(): Promise<Map<string, string[]>> {
    const indexes = await this.db.execute<any>(`
      SELECT 
        TABLE_NAME,
        INDEX_NAME,
        GROUP_CONCAT(COLUMN_NAME ORDER BY SEQ_IN_INDEX) as columns
      FROM information_schema.STATISTICS
      WHERE TABLE_SCHEMA = DATABASE()
      GROUP BY TABLE_NAME, INDEX_NAME
    `);

    const duplicates = new Map<string, string[]>();

    // 동일한 컬럼 조합을 가진 인덱스 찾기
    const indexMap = new Map<string, string[]>();
    
    for (const index of indexes) {
      const key = `${index.TABLE_NAME}:${index.columns}`;
      
      if (indexMap.has(key)) {
        const existing = indexMap.get(key)!;
        duplicates.set(index.TABLE_NAME, [
          ...existing,
          index.INDEX_NAME
        ]);
      } else {
        indexMap.set(key, [index.INDEX_NAME]);
      }
    }

    return duplicates;
  }

  // 사용되지 않는 인덱스 찾기
  async findUnusedIndexes(days: number = 30): Promise<any[]> {
    return await this.db.execute(`
      SELECT 
        object_schema,
        object_name,
        index_name
      FROM performance_schema.table_io_waits_summary_by_index_usage
      WHERE index_name IS NOT NULL
        AND count_star = 0
        AND object_schema = DATABASE()
    `);
  }
}
```

---

## 💡 개발 모범 사례

### 1. 쿼리 최적화

```typescript
// 쿼리 최적화 유틸리티
class QueryOptimizer {
  constructor(private db: DatabaseConnectionPool) {}

  // N+1 문제 해결
  async fetchUsersWithPosts(): Promise<any[]> {
    // ❌ Bad: N+1 쿼리
    // const users = await db.execute('SELECT * FROM users');
    // for (const user of users) {
    //   user.posts = await db.execute(
    //     'SELECT * FROM posts WHERE user_id = ?',
    //     [user.id]
    //   );
    // }

    // ✅ Good: JOIN 사용
    const result = await this.db.execute(`
      SELECT 
        u.id as user_id,
        u.username,
        u.email,
        p.id as post_id,
        p.title,
        p.content,
        p.created_at as post_created_at
      FROM users u
      LEFT JOIN posts p ON u.id = p.user_id
      ORDER BY u.id, p.created_at DESC
    `);

    // 결과 재구성
    const usersMap = new Map<number, any>();
    
    for (const row of result) {
      if (!usersMap.has(row.user_id)) {
        usersMap.set(row.user_id, {
          id: row.user_id,
          username: row.username,
          email: row.email,
          posts: []
        });
      }

      if (row.post_id) {
        usersMap.get(row.user_id)!.posts.push({
          id: row.post_id,
          title: row.title,
          content: row.content,
          created_at: row.post_created_at
        });
      }
    }

    return Array.from(usersMap.values());
  }

  // 페이징 최적화
  async optimizedPagination(
    page: number,
    pageSize: number,
    lastId?: number
  ): Promise<any[]> {
    // ❌ Bad: OFFSET 사용 (대용량 데이터에서 느림)
    // return await db.execute(
    //   'SELECT * FROM posts ORDER BY id LIMIT ? OFFSET ?',
    //   [pageSize, (page - 1) * pageSize]
    // );

    // ✅ Good: Keyset pagination
    if (lastId) {
      return await this.db.execute(
        `SELECT * FROM posts 
         WHERE id > ? 
         ORDER BY id 
         LIMIT ?`,
        [lastId, pageSize]
      );
    } else {
      return await this.db.execute(
        'SELECT * FROM posts ORDER BY id LIMIT ?',
        [pageSize]
      );
    }
  }

  // 배치 작업 최적화
  async batchInsert<T>(
    tableName: string,
    data: T[],
    batchSize: number = 1000
  ): Promise<void> {
    if (data.length === 0) return;

    const columns = Object.keys(data[0]);
    const columnString = columns.join(', ');

    for (let i = 0; i < data.length; i += batchSize) {
      const batch = data.slice(i, i + batchSize);
      
      const placeholders = batch
        .map(() => `(${columns.map(() => '?').join(', ')})`)
        .join(', ');
      
      const values = batch.flatMap(row => 
        columns.map(col => (row as any)[col])
      );

      await this.db.execute(
        `INSERT INTO ${tableName} (${columnString}) VALUES ${placeholders}`,
        values
      );
    }
  }

  // 쿼리 캐싱
  private queryCache = new Map<string, { data: any; timestamp: number }>();
  private cacheTTL = 60000; // 1분

  async cachedQuery<T>(
    key: string,
    query: string,
    params: any[] = [],
    ttl: number = this.cacheTTL
  ): Promise<T[]> {
    const cached = this.queryCache.get(key);
    
    if (cached && Date.now() - cached.timestamp < ttl) {
      return cached.data;
    }

    const result = await this.db.execute<T>(query, params);
    
    this.queryCache.set(key, {
      data: result,
      timestamp: Date.now()
    });

    // 캐시 크기 제한
    if (this.queryCache.size > 100) {
      const oldestKey = Array.from(this.queryCache.entries())
        .sort(([, a], [, b]) => a.timestamp - b.timestamp)[0][0];
      this.queryCache.delete(oldestKey);
    }

    return result;
  }
}
```

### 2. 에러 처리 및 재시도

```typescript
// 강건한 에러 처리
class ResilientDatabase {
  private retryConfig = {
    maxRetries: 3,
    initialDelay: 100,
    maxDelay: 5000,
    backoffMultiplier: 2
  };

  constructor(private db: DatabaseConnectionPool) {}

  // 재시도 로직
  async executeWithRetry<T>(
    operation: () => Promise<T>,
    context: string
  ): Promise<T> {
    let lastError: Error | null = null;
    let delay = this.retryConfig.initialDelay;

    for (let attempt = 1; attempt <= this.retryConfig.maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error: any) {
        lastError = error;

        // 재시도 가능한 에러인지 확인
        if (!this.isRetryableError(error)) {
          throw error;
        }

        if (attempt < this.retryConfig.maxRetries) {
          console.log(
            `Retry ${attempt}/${this.retryConfig.maxRetries} for ${context} after ${delay}ms`
          );
          
          await this.sleep(delay);
          delay = Math.min(
            delay * this.retryConfig.backoffMultiplier,
            this.retryConfig.maxDelay
          );
        }
      }
    }

    throw new Error(
      `Operation failed after ${this.retryConfig.maxRetries} retries: ${lastError?.message}`
    );
  }

  private isRetryableError(error: any): boolean {
    const retryableCodes = [
      'ETIMEDOUT',
      'ECONNREFUSED',
      'ECONNRESET',
      'ER_LOCK_DEADLOCK',
      'ER_LOCK_WAIT_TIMEOUT'
    ];

    return retryableCodes.includes(error.code);
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  // 서킷 브레이커 패턴
  private circuitState: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  private failureCount = 0;
  private successCount = 0;
  private lastFailureTime?: number;
  private readonly failureThreshold = 5;
  private readonly successThreshold = 3;
  private readonly timeout = 60000; // 1분

  async executeWithCircuitBreaker<T>(
    operation: () => Promise<T>
  ): Promise<T> {
    // 서킷이 열려있는지 확인
    if (this.circuitState === 'OPEN') {
      if (Date.now() - (this.lastFailureTime || 0) > this.timeout) {
        this.circuitState = 'HALF_OPEN';
        this.successCount = 0;
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await operation();

      // 성공 처리
      if (this.circuitState === 'HALF_OPEN') {
        this.successCount++;
        if (this.successCount >= this.successThreshold) {
          this.circuitState = 'CLOSED';
          this.failureCount = 0;
        }
      } else {
        this.failureCount = 0;
      }

      return result;
    } catch (error) {
      // 실패 처리
      this.failureCount++;
      this.lastFailureTime = Date.now();

      if (this.failureCount >= this.failureThreshold) {
        this.circuitState = 'OPEN';
      }

      throw error;
    }
  }
}
```

---

## 🚀 운영 모범 사례

### 1. 모니터링 및 알림

```typescript
// 데이터베이스 모니터링 시스템
class DatabaseMonitor {
  private metrics: Map<string, any[]> = new Map();
  private alerts: Array<{
    condition: (metrics: any) => boolean;
    message: string;
    severity: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  }> = [];

  constructor(private db: DatabaseConnectionPool) {
    this.setupDefaultAlerts();
  }

  // 기본 알림 설정
  private setupDefaultAlerts(): void {
    // 연결 풀 사용률
    this.addAlert(
      metrics => metrics.connectionPoolUsage > 90,
      'Connection pool usage above 90%',
      'HIGH'
    );

    // 슬로우 쿼리
    this.addAlert(
      metrics => metrics.slowQueryCount > 10,
      'High number of slow queries detected',
      'MEDIUM'
    );

    // 디스크 사용률
    this.addAlert(
      metrics => metrics.diskUsage > 85,
      'Disk usage above 85%',
      'HIGH'
    );

    // 복제 지연
    this.addAlert(
      metrics => metrics.replicationLag > 5000,
      'Replication lag exceeds 5 seconds',
      'CRITICAL'
    );
  }

  // 메트릭 수집
  async collectMetrics(): Promise<Record<string, any>> {
    const metrics: Record<string, any> = {};

    // 연결 정보
    const [connections] = await this.db.execute<any>(`
      SHOW STATUS WHERE Variable_name IN (
        'Threads_connected',
        'Max_used_connections',
        'Aborted_connects'
      )
    `);

    connections.forEach((conn: any) => {
      metrics[conn.Variable_name] = parseInt(conn.Value);
    });

    // 쿼리 성능
    const [queryStats] = await this.db.execute<any>(`
      SHOW STATUS WHERE Variable_name IN (
        'Slow_queries',
        'Questions',
        'Com_select',
        'Com_insert',
        'Com_update',
        'Com_delete'
      )
    `);

    queryStats.forEach((stat: any) => {
      metrics[stat.Variable_name] = parseInt(stat.Value);
    });

    // InnoDB 메트릭
    const [innodbStats] = await this.db.execute<any>(`
      SHOW STATUS WHERE Variable_name LIKE 'Innodb_%'
    `);

    innodbStats.forEach((stat: any) => {
      metrics[stat.Variable_name] = stat.Value;
    });

    // 테이블 크기
    const [tableSize] = await this.db.execute<any>(`
      SELECT 
        SUM(data_length + index_length) / 1024 / 1024 as total_size_mb,
        SUM(data_length) / 1024 / 1024 as data_size_mb,
        SUM(index_length) / 1024 / 1024 as index_size_mb
      FROM information_schema.tables
      WHERE table_schema = DATABASE()
    `);

    metrics.totalSizeMB = tableSize[0].total_size_mb;
    metrics.dataSizeMB = tableSize[0].data_size_mb;
    metrics.indexSizeMB = tableSize[0].index_size_mb;

    // 메트릭 저장
    this.storeMetrics(metrics);

    // 알림 확인
    this.checkAlerts(metrics);

    return metrics;
  }

  // 메트릭 저장
  private storeMetrics(metrics: Record<string, any>): void {
    const timestamp = Date.now();
    
    for (const [key, value] of Object.entries(metrics)) {
      if (!this.metrics.has(key)) {
        this.metrics.set(key, []);
      }
      
      const history = this.metrics.get(key)!;
      history.push({ value, timestamp });
      
      // 24시간 이상 된 데이터 제거
      const cutoff = timestamp - 24 * 60 * 60 * 1000;
      const filtered = history.filter(m => m.timestamp > cutoff);
      this.metrics.set(key, filtered);
    }
  }

  // 알림 추가
  addAlert(
    condition: (metrics: any) => boolean,
    message: string,
    severity: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL'
  ): void {
    this.alerts.push({ condition, message, severity });
  }

  // 알림 확인
  private checkAlerts(metrics: Record<string, any>): void {
    for (const alert of this.alerts) {
      if (alert.condition(metrics)) {
        this.sendAlert(alert.message, alert.severity);
      }
    }
  }

  // 알림 전송
  private sendAlert(message: string, severity: string): void {
    console.log(`[${severity}] ${message}`);
    // 실제 구현: 이메일, Slack, PagerDuty 등
  }

  // 성능 리포트 생성
  generatePerformanceReport(): {
    summary: Record<string, any>;
    trends: Record<string, any>;
    recommendations: string[];
  } {
    const summary: Record<string, any> = {};
    const trends: Record<string, any> = {};
    const recommendations: string[] = [];

    // 현재 메트릭 요약
    for (const [key, history] of this.metrics.entries()) {
      if (history.length > 0) {
        const current = history[history.length - 1].value;
        const values = history.map(h => h.value);
        
        summary[key] = {
          current,
          avg: values.reduce((a, b) => a + b, 0) / values.length,
          min: Math.min(...values),
          max: Math.max(...values)
        };

        // 추세 분석
        if (history.length > 10) {
          const recent = values.slice(-10);
          const older = values.slice(-20, -10);
          
          if (older.length > 0) {
            const recentAvg = recent.reduce((a, b) => a + b, 0) / recent.length;
            const olderAvg = older.reduce((a, b) => a + b, 0) / older.length;
            const change = ((recentAvg - olderAvg) / olderAvg) * 100;
            
            trends[key] = {
              direction: change > 0 ? 'increasing' : 'decreasing',
              changePercent: Math.abs(change)
            };
          }
        }
      }
    }

    // 권장사항 생성
    if (summary.Slow_queries?.current > 10) {
      recommendations.push('Optimize slow queries or add appropriate indexes');
    }

    if (summary.Threads_connected?.current > summary.Max_used_connections?.current * 0.8) {
      recommendations.push('Consider increasing max_connections');
    }

    if (summary.totalSizeMB?.current > 10000) {
      recommendations.push('Consider archiving old data or partitioning large tables');
    }

    return { summary, trends, recommendations };
  }
}
```

### 2. 백업 및 복구

```typescript
// 백업 관리 시스템
class BackupManager {
  constructor(
    private db: DatabaseConnectionPool,
    private config: {
      backupPath: string;
      retentionDays: number;
      compression: boolean;
    }
  ) {}

  // 전체 백업
  async fullBackup(): Promise<string> {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const filename = `backup_full_${timestamp}.sql`;
    const filepath = `${this.config.backupPath}/${filename}`;

    // mysqldump 실행 (실제 구현에서는 child_process 사용)
    const command = this.buildBackupCommand(filepath);
    
    console.log(`Starting full backup to ${filepath}`);
    // await exec(command);

    if (this.config.compression) {
      // await exec(`gzip ${filepath}`);
      return `${filepath}.gz`;
    }

    return filepath;
  }

  // 증분 백업 (바이너리 로그 기반)
  async incrementalBackup(lastPosition: string): Promise<string> {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const filename = `backup_incr_${timestamp}.binlog`;
    const filepath = `${this.config.backupPath}/${filename}`;

    // mysqlbinlog 실행
    const command = `mysqlbinlog --start-position=${lastPosition} > ${filepath}`;
    
    console.log(`Starting incremental backup from position ${lastPosition}`);
    // await exec(command);

    return filepath;
  }

  // 백업 검증
  async verifyBackup(backupFile: string): Promise<boolean> {
    try {
      // 테스트 데이터베이스에 복원 시도
      const testDb = 'backup_verification_test';
      
      await this.db.execute(`CREATE DATABASE IF NOT EXISTS ${testDb}`);
      
      // 백업 파일 복원
      const command = `mysql ${testDb} < ${backupFile}`;
      // await exec(command);

      // 기본 검증 쿼리 실행
      await this.db.execute(`USE ${testDb}`);
      const [tables] = await this.db.execute<any>('SHOW TABLES');
      
      // 정리
      await this.db.execute(`DROP DATABASE ${testDb}`);

      return tables.length > 0;
    } catch (error) {
      console.error('Backup verification failed:', error);
      return false;
    }
  }

  // 복구 프로세스
  async restore(backupFile: string, targetDb?: string): Promise<void> {
    const database = targetDb || 'restored_db';
    
    console.log(`Starting restoration to ${database}`);
    
    // 데이터베이스 생성
    await this.db.execute(`CREATE DATABASE IF NOT EXISTS ${database}`);
    
    // 백업 파일 복원
    const command = `mysql ${database} < ${backupFile}`;
    // await exec(command);
    
    console.log(`Restoration completed to ${database}`);
  }

  // 백업 파일 관리
  async cleanupOldBackups(): Promise<number> {
    const fs = require('fs').promises;
    const path = require('path');
    
    const files = await fs.readdir(this.config.backupPath);
    const now = Date.now();
    const maxAge = this.config.retentionDays * 24 * 60 * 60 * 1000;
    let deletedCount = 0;

    for (const file of files) {
      const filepath = path.join(this.config.backupPath, file);
      const stats = await fs.stat(filepath);
      
      if (now - stats.mtime.getTime() > maxAge) {
        await fs.unlink(filepath);
        deletedCount++;
        console.log(`Deleted old backup: ${file}`);
      }
    }

    return deletedCount;
  }

  private buildBackupCommand(outputFile: string): string {
    const options = [
      '--single-transaction', // InnoDB 일관성
      '--routines',           // 저장 프로시저 포함
      '--triggers',           // 트리거 포함
      '--events',            // 이벤트 포함
      '--add-drop-table',    // DROP TABLE 문 포함
      '--create-options',    // 전체 CREATE 옵션
      '--extended-insert',   // 확장 INSERT 문
      '--lock-tables=false', // 테이블 잠금 방지
      '--quick',            // 대용량 테이블 최적화
      '--set-gtid-purged=OFF' // GTID 문제 방지
    ];

    return `mysqldump ${options.join(' ')} > ${outputFile}`;
  }
}
```

---

## ⚠️ 주의사항

### 안티패턴 피하기

> [!danger] 피해야 할 패턴
> - **SELECT \***: 필요한 컬럼만 명시적으로 선택
> - **인덱스 없는 JOIN**: JOIN 컬럼에는 항상 인덱스 필요
> - **과도한 정규화**: 성능을 위해 적절한 역정규화 고려
> - **트랜잭션 남용**: 필요한 경우에만 트랜잭션 사용
> - **준비되지 않은 문자열 연결**: SQL 인젝션 위험
> - **무제한 결과**: 항상 LIMIT 사용
> - **동기 처리**: 비동기 처리로 동시성 향상

### 체크리스트

| 영역 | 체크 항목 | 완료 |
|------|-----------|------|
| 설계 | 적절한 정규화 | ☐ |
| 설계 | 인덱스 전략 | ☐ |
| 설계 | 명명 규칙 | ☐ |
| 개발 | Prepared Statement | ☐ |
| 개발 | 에러 처리 | ☐ |
| 개발 | 트랜잭션 관리 | ☐ |
| 운영 | 백업 전략 | ☐ |
| 운영 | 모니터링 | ☐ |
| 운영 | 보안 설정 | ☐ |
| 운영 | 문서화 | ☐ |

---

## 📚 참고자료

### 관련 문서
- [[09_normalization|정규화]]
- [[08_database-index|데이터베이스 인덱스]]
- [[13_database-scaling|데이터베이스 스케일링]]
- [[17_database-security|데이터베이스 보안]]

### 외부 리소스
- [High Performance MySQL](https://www.oreilly.com/library/view/high-performance-mysql/9781492080503/)
- [Database Reliability Engineering](https://www.oreilly.com/library/view/database-reliability-engineering/9781491925935/)
- [SQL Performance Explained](https://sql-performance-explained.com/)

### 도구 및 라이브러리
1. **모니터링**: Prometheus + Grafana, DataDog, New Relic
2. **백업**: Percona XtraBackup, MySQL Enterprise Backup
3. **마이그레이션**: Flyway, Liquibase, TypeORM Migrations
4. **성능 분석**: pt-query-digest, MySQL Workbench
5. **ORM**: TypeORM, Prisma, Sequelize

---

> [!quote]
> "데이터베이스 모범 사례는 단순한 가이드라인이 아니라, 수년간의 경험과 실패에서 얻은 지혜입니다. 이를 따르면 안정적이고 확장 가능한 시스템을 구축할 수 있습니다."