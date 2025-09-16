---
tags:
  - database
  - mysql
  - storage-engine
  - innodb
  - myisam
  - memory
  - performance
created: 2025-01-08
updated: 2025-01-08
---

# MySQL 스토리지 엔진 (Storage Engines)

> [!info] 개요
> MySQL은 플러그형 스토리지 엔진 아키텍처를 제공하여 다양한 용도에 맞는 스토리지 엔진을 선택할 수 있습니다. 각 엔진은 고유한 특성과 장단점을 가지며, 애플리케이션의 요구사항에 따라 적절한 엔진을 선택하는 것이 중요합니다.

## 📑 목차

- [[#⚡ 빠른 시작]]
- [[#🔧 주요 스토리지 엔진]]
- [[#💡 실전 예제]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## ⚡ 빠른 시작

### 스토리지 엔진 확인 및 설정

> [!example] 기본 명령어
> ```sql
> -- 사용 가능한 스토리지 엔진 확인
> SHOW ENGINES;
> 
> -- 테이블의 스토리지 엔진 확인
> SHOW TABLE STATUS WHERE Name = 'users';
> 
> -- 테이블 생성 시 엔진 지정
> CREATE TABLE users (
>     id INT PRIMARY KEY,
>     name VARCHAR(100)
> ) ENGINE=InnoDB;
> 
> -- 기존 테이블 엔진 변경
> ALTER TABLE users ENGINE=MyISAM;
> ```

### TypeScript에서 스토리지 엔진 관리

```typescript
import mysql from 'mysql2/promise';

interface StorageEngineInfo {
  engine: string;
  support: 'YES' | 'NO' | 'DEFAULT' | 'DISABLED';
  comment: string;
  transactions: 'YES' | 'NO';
  xa: 'YES' | 'NO';
  savepoints: 'YES' | 'NO';
}

interface TableInfo {
  name: string;
  engine: string;
  rowFormat: string;
  rows: number;
  avgRowLength: number;
  dataLength: number;
  indexLength: number;
}

class MySQLEngineManager {
  private connection: mysql.Connection | null = null;

  constructor(private config: mysql.ConnectionOptions) {}

  async connect(): Promise<void> {
    this.connection = await mysql.createConnection(this.config);
  }

  // 사용 가능한 스토리지 엔진 조회
  async getAvailableEngines(): Promise<StorageEngineInfo[]> {
    if (!this.connection) throw new Error('Not connected');

    const [rows] = await this.connection.execute<any[]>('SHOW ENGINES');
    
    return rows.map(row => ({
      engine: row.Engine,
      support: row.Support,
      comment: row.Comment,
      transactions: row.Transactions,
      xa: row.XA,
      savepoints: row.Savepoints
    }));
  }

  // 테이블 정보 조회
  async getTableInfo(tableName: string): Promise<TableInfo | null> {
    if (!this.connection) throw new Error('Not connected');

    const [rows] = await this.connection.execute<any[]>(
      'SHOW TABLE STATUS WHERE Name = ?',
      [tableName]
    );

    if (rows.length === 0) return null;

    const row = rows[0];
    return {
      name: row.Name,
      engine: row.Engine,
      rowFormat: row.Row_format,
      rows: row.Rows,
      avgRowLength: row.Avg_row_length,
      dataLength: row.Data_length,
      indexLength: row.Index_length
    };
  }

  // 테이블 엔진 변경
  async changeTableEngine(
    tableName: string,
    newEngine: 'InnoDB' | 'MyISAM' | 'Memory' | 'CSV'
  ): Promise<void> {
    if (!this.connection) throw new Error('Not connected');

    // 변경 전 백업 권장
    console.log(`Warning: Changing engine for table ${tableName} to ${newEngine}`);
    
    await this.connection.execute(
      `ALTER TABLE ?? ENGINE = ${newEngine}`,
      [tableName]
    );
  }

  // 엔진별 최적화된 테이블 생성
  async createOptimizedTable(
    tableName: string,
    engine: string,
    columns: Array<{ name: string; type: string; constraints?: string }>
  ): Promise<void> {
    if (!this.connection) throw new Error('Not connected');

    const columnDefs = columns
      .map(col => `${col.name} ${col.type} ${col.constraints || ''}`)
      .join(', ');

    let engineSpecificOptions = '';
    
    // 엔진별 최적화 옵션
    switch (engine) {
      case 'InnoDB':
        engineSpecificOptions = `
          ROW_FORMAT=DYNAMIC
          KEY_BLOCK_SIZE=8
        `;
        break;
      case 'MyISAM':
        engineSpecificOptions = `
          PACK_KEYS=1
          DELAY_KEY_WRITE=1
        `;
        break;
      case 'Memory':
        engineSpecificOptions = `
          MAX_ROWS=1000000
        `;
        break;
    }

    const query = `
      CREATE TABLE ${tableName} (
        ${columnDefs}
      ) ENGINE=${engine}
      ${engineSpecificOptions}
    `;

    await this.connection.execute(query);
  }

  async disconnect(): Promise<void> {
    if (this.connection) {
      await this.connection.end();
      this.connection = null;
    }
  }
}
```

---

## 🔧 주요 스토리지 엔진

### InnoDB (기본 엔진)

> [!note] InnoDB 특징
> - **트랜잭션 지원**: ACID 완벽 지원
> - **외래 키 제약**: 참조 무결성 보장
> - **행 수준 잠금**: 높은 동시성
> - **충돌 복구**: 자동 복구 메커니즘
> - **MVCC**: 다중 버전 동시성 제어

```typescript
// InnoDB 최적화 관리자
class InnoDBOptimizer {
  constructor(private connection: mysql.Connection) {}

  // InnoDB 버퍼 풀 상태 확인
  async getBufferPoolStatus(): Promise<{
    totalPages: number;
    freePages: number;
    dirtyPages: number;
    hitRate: number;
  }> {
    const [rows] = await this.connection.execute<any[]>(`
      SELECT 
        VARIABLE_NAME,
        VARIABLE_VALUE
      FROM information_schema.SESSION_STATUS
      WHERE VARIABLE_NAME LIKE 'Innodb_buffer_pool%'
    `);

    const stats = rows.reduce((acc, row) => {
      acc[row.VARIABLE_NAME] = parseInt(row.VARIABLE_VALUE) || 0;
      return acc;
    }, {} as any);

    const reads = stats.Innodb_buffer_pool_reads || 0;
    const readRequests = stats.Innodb_buffer_pool_read_requests || 0;
    const hitRate = readRequests > 0 
      ? ((readRequests - reads) / readRequests) * 100 
      : 0;

    return {
      totalPages: stats.Innodb_buffer_pool_pages_total || 0,
      freePages: stats.Innodb_buffer_pool_pages_free || 0,
      dirtyPages: stats.Innodb_buffer_pool_pages_dirty || 0,
      hitRate: Math.round(hitRate * 100) / 100
    };
  }

  // InnoDB 설정 최적화
  async optimizeSettings(config: {
    bufferPoolSize?: string;
    logFileSize?: string;
    flushMethod?: string;
    filePerTable?: boolean;
  }): Promise<void> {
    const settings = [];

    if (config.bufferPoolSize) {
      settings.push(`SET GLOBAL innodb_buffer_pool_size = ${config.bufferPoolSize}`);
    }

    if (config.logFileSize) {
      settings.push(`SET GLOBAL innodb_log_file_size = ${config.logFileSize}`);
    }

    if (config.flushMethod) {
      settings.push(`SET GLOBAL innodb_flush_method = '${config.flushMethod}'`);
    }

    if (config.filePerTable !== undefined) {
      settings.push(`SET GLOBAL innodb_file_per_table = ${config.filePerTable ? 1 : 0}`);
    }

    for (const setting of settings) {
      await this.connection.execute(setting);
    }
  }

  // 테이블 통계 분석
  async analyzeTable(tableName: string): Promise<void> {
    await this.connection.execute(`ANALYZE TABLE ${tableName}`);
  }

  // 테이블 최적화
  async optimizeTable(tableName: string): Promise<void> {
    await this.connection.execute(`OPTIMIZE TABLE ${tableName}`);
  }
}
```

### MyISAM

> [!note] MyISAM 특징
> - **빠른 읽기**: 읽기 전용 또는 읽기 위주 작업에 최적
> - **테이블 수준 잠금**: 동시성 제한
> - **트랜잭션 미지원**: ACID 미지원
> - **전체 텍스트 인덱스**: FULLTEXT 인덱스 지원
> - **압축 테이블**: 읽기 전용 압축 테이블 지원

```typescript
// MyISAM 특화 기능
class MyISAMManager {
  constructor(private connection: mysql.Connection) {}

  // MyISAM 테이블 압축
  async compressTable(tableName: string): Promise<void> {
    // myisampack 유틸리티를 통한 압축 (외부 명령)
    const { exec } = require('child_process');
    const util = require('util');
    const execPromise = util.promisify(exec);

    try {
      // 테이블 플러시
      await this.connection.execute(`FLUSH TABLES ${tableName}`);
      
      // 압축 실행 (서버에서 직접 실행 필요)
      await execPromise(`myisampack /var/lib/mysql/${tableName}`);
      
      // 인덱스 재구축
      await this.connection.execute(`REPAIR TABLE ${tableName}`);
    } catch (error) {
      console.error('Compression failed:', error);
      throw error;
    }
  }

  // MyISAM 키 캐시 최적화
  async optimizeKeyCache(
    cacheName: string,
    size: string
  ): Promise<void> {
    // 키 캐시 생성
    await this.connection.execute(
      `SET GLOBAL ${cacheName}.key_buffer_size = ${size}`
    );
  }

  // 전체 텍스트 검색 인덱스 생성
  async createFullTextIndex(
    tableName: string,
    columns: string[]
  ): Promise<void> {
    const columnList = columns.join(', ');
    await this.connection.execute(
      `ALTER TABLE ${tableName} ADD FULLTEXT(${columnList})`
    );
  }

  // 전체 텍스트 검색
  async fullTextSearch(
    tableName: string,
    columns: string[],
    searchTerm: string
  ): Promise<any[]> {
    const columnList = columns.join(', ');
    const [rows] = await this.connection.execute(
      `SELECT *, MATCH(${columnList}) AGAINST(? IN NATURAL LANGUAGE MODE) AS relevance
       FROM ${tableName}
       WHERE MATCH(${columnList}) AGAINST(? IN NATURAL LANGUAGE MODE)
       ORDER BY relevance DESC`,
      [searchTerm, searchTerm]
    );
    
    return rows;
  }
}
```

### Memory (HEAP)

> [!note] Memory 엔진 특징
> - **메모리 저장**: 모든 데이터를 RAM에 저장
> - **빠른 액세스**: 매우 빠른 읽기/쓰기
> - **휘발성**: 서버 재시작 시 데이터 손실
> - **고정 길이**: VARCHAR를 CHAR로 저장
> - **임시 테이블**: 세션 임시 데이터에 적합

```typescript
// Memory 엔진 관리
class MemoryEngineManager {
  constructor(private connection: mysql.Connection) {}

  // 임시 캐시 테이블 생성
  async createCacheTable(
    tableName: string,
    ttlSeconds: number = 3600
  ): Promise<void> {
    // 캐시 테이블 생성
    await this.connection.execute(`
      CREATE TABLE IF NOT EXISTS ${tableName} (
        cache_key VARCHAR(255) PRIMARY KEY,
        cache_value TEXT,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        expires_at TIMESTAMP
      ) ENGINE=Memory
    `);

    // TTL 기반 자동 삭제 이벤트 생성
    await this.connection.execute(`
      CREATE EVENT IF NOT EXISTS clean_${tableName}
      ON SCHEDULE EVERY 1 MINUTE
      DO DELETE FROM ${tableName}
      WHERE expires_at < NOW()
    `);
  }

  // 캐시 데이터 저장
  async setCache(
    tableName: string,
    key: string,
    value: any,
    ttlSeconds: number = 3600
  ): Promise<void> {
    const jsonValue = JSON.stringify(value);
    
    await this.connection.execute(
      `INSERT INTO ${tableName} (cache_key, cache_value, expires_at)
       VALUES (?, ?, DATE_ADD(NOW(), INTERVAL ? SECOND))
       ON DUPLICATE KEY UPDATE
       cache_value = VALUES(cache_value),
       expires_at = VALUES(expires_at)`,
      [key, jsonValue, ttlSeconds]
    );
  }

  // 캐시 데이터 조회
  async getCache(
    tableName: string,
    key: string
  ): Promise<any | null> {
    const [rows] = await this.connection.execute<any[]>(
      `SELECT cache_value FROM ${tableName}
       WHERE cache_key = ? AND expires_at > NOW()`,
      [key]
    );

    if (rows.length === 0) return null;

    try {
      return JSON.parse(rows[0].cache_value);
    } catch {
      return rows[0].cache_value;
    }
  }

  // 메모리 사용량 확인
  async getMemoryUsage(): Promise<{
    totalMemory: number;
    usedMemory: number;
    tables: Array<{ name: string; size: number }>;
  }> {
    const [tables] = await this.connection.execute<any[]>(`
      SELECT 
        TABLE_NAME,
        (DATA_LENGTH + INDEX_LENGTH) as size
      FROM information_schema.TABLES
      WHERE ENGINE = 'MEMORY'
      AND TABLE_SCHEMA = DATABASE()
    `);

    const totalUsed = tables.reduce((sum, table) => sum + table.size, 0);

    const [maxHeap] = await this.connection.execute<any[]>(
      `SHOW VARIABLES LIKE 'max_heap_table_size'`
    );

    return {
      totalMemory: parseInt(maxHeap[0].Value),
      usedMemory: totalUsed,
      tables: tables.map(t => ({
        name: t.TABLE_NAME,
        size: t.size
      }))
    };
  }
}
```

---

## 💡 실전 예제

### 하이브리드 스토리지 전략

```typescript
// 엔진별 테이블 분류 및 관리
class HybridStorageStrategy {
  private innodb: InnoDBOptimizer;
  private myisam: MyISAMManager;
  private memory: MemoryEngineManager;

  constructor(private connection: mysql.Connection) {
    this.innodb = new InnoDBOptimizer(connection);
    this.myisam = new MyISAMManager(connection);
    this.memory = new MemoryEngineManager(connection);
  }

  // 트랜잭션이 필요한 핵심 테이블
  async createTransactionalTable(
    tableName: string,
    schema: string
  ): Promise<void> {
    await this.connection.execute(`
      CREATE TABLE ${tableName} (
        ${schema}
      ) ENGINE=InnoDB
      ROW_FORMAT=DYNAMIC
      DEFAULT CHARSET=utf8mb4
      COLLATE=utf8mb4_unicode_ci
    `);

    // 외래 키 제약 추가
    console.log(`Created transactional table: ${tableName}`);
  }

  // 읽기 전용 로그 테이블
  async createLogTable(
    tableName: string
  ): Promise<void> {
    await this.connection.execute(`
      CREATE TABLE ${tableName} (
        id BIGINT AUTO_INCREMENT PRIMARY KEY,
        log_level ENUM('DEBUG', 'INFO', 'WARN', 'ERROR') NOT NULL,
        message TEXT NOT NULL,
        context JSON,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        KEY idx_created_at (created_at),
        KEY idx_log_level (log_level)
      ) ENGINE=MyISAM
      PACK_KEYS=1
    `);

    console.log(`Created log table: ${tableName}`);
  }

  // 세션 데이터용 메모리 테이블
  async createSessionTable(): Promise<void> {
    await this.connection.execute(`
      CREATE TABLE sessions (
        session_id VARCHAR(128) PRIMARY KEY,
        user_id INT NOT NULL,
        data JSON,
        last_activity TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
        KEY idx_user_id (user_id),
        KEY idx_last_activity (last_activity)
      ) ENGINE=Memory
      MAX_ROWS=100000
    `);

    console.log('Created memory-based session table');
  }

  // 자동 엔진 선택 로직
  async recommendEngine(requirements: {
    transactions: boolean;
    foreignKeys: boolean;
    fullTextSearch: boolean;
    volatileData: boolean;
    writeIntensive: boolean;
    dataSize: 'small' | 'medium' | 'large';
  }): Promise<string> {
    // 트랜잭션이나 외래 키가 필요하면 InnoDB
    if (requirements.transactions || requirements.foreignKeys) {
      return 'InnoDB';
    }

    // 휘발성 데이터이고 작은 크기면 Memory
    if (requirements.volatileData && requirements.dataSize === 'small') {
      return 'Memory';
    }

    // 전체 텍스트 검색이 필요하고 쓰기가 적으면 MyISAM
    if (requirements.fullTextSearch && !requirements.writeIntensive) {
      return 'MyISAM';
    }

    // 기본적으로 InnoDB 권장
    return 'InnoDB';
  }
}

// 성능 벤치마크
class StorageEngineBenchmark {
  constructor(private connection: mysql.Connection) {}

  async runBenchmark(
    tableName: string,
    engine: string,
    operations: number = 10000
  ): Promise<{
    engine: string;
    insertTime: number;
    selectTime: number;
    updateTime: number;
    deleteTime: number;
  }> {
    // 테스트 테이블 생성
    await this.connection.execute(`
      CREATE TEMPORARY TABLE ${tableName}_bench (
        id INT AUTO_INCREMENT PRIMARY KEY,
        data VARCHAR(255),
        value INT,
        INDEX idx_value (value)
      ) ENGINE=${engine}
    `);

    const results = {
      engine,
      insertTime: 0,
      selectTime: 0,
      updateTime: 0,
      deleteTime: 0
    };

    // INSERT 벤치마크
    const insertStart = Date.now();
    for (let i = 0; i < operations; i++) {
      await this.connection.execute(
        `INSERT INTO ${tableName}_bench (data, value) VALUES (?, ?)`,
        [`data_${i}`, Math.floor(Math.random() * 1000)]
      );
    }
    results.insertTime = Date.now() - insertStart;

    // SELECT 벤치마크
    const selectStart = Date.now();
    for (let i = 0; i < operations / 10; i++) {
      await this.connection.execute(
        `SELECT * FROM ${tableName}_bench WHERE value = ?`,
        [Math.floor(Math.random() * 1000)]
      );
    }
    results.selectTime = Date.now() - selectStart;

    // UPDATE 벤치마크
    const updateStart = Date.now();
    for (let i = 0; i < operations / 10; i++) {
      await this.connection.execute(
        `UPDATE ${tableName}_bench SET value = ? WHERE id = ?`,
        [Math.floor(Math.random() * 1000), Math.floor(Math.random() * operations)]
      );
    }
    results.updateTime = Date.now() - updateStart;

    // DELETE 벤치마크
    const deleteStart = Date.now();
    for (let i = 0; i < operations / 10; i++) {
      await this.connection.execute(
        `DELETE FROM ${tableName}_bench WHERE id = ?`,
        [i]
      );
    }
    results.deleteTime = Date.now() - deleteStart;

    // 정리
    await this.connection.execute(`DROP TABLE ${tableName}_bench`);

    return results;
  }

  // 여러 엔진 비교
  async compareEngines(
    engines: string[] = ['InnoDB', 'MyISAM', 'Memory']
  ): Promise<void> {
    console.log('Storage Engine Performance Comparison');
    console.log('=====================================');

    for (const engine of engines) {
      try {
        const result = await this.runBenchmark(`test_${engine}`, engine, 1000);
        
        console.log(`\n${engine}:`);
        console.log(`  Insert: ${result.insertTime}ms`);
        console.log(`  Select: ${result.selectTime}ms`);
        console.log(`  Update: ${result.updateTime}ms`);
        console.log(`  Delete: ${result.deleteTime}ms`);
        console.log(`  Total: ${
          result.insertTime + result.selectTime + 
          result.updateTime + result.deleteTime
        }ms`);
      } catch (error) {
        console.log(`\n${engine}: Not available or error occurred`);
      }
    }
  }
}
```

---

## ⚠️ 주의사항

### 엔진 선택 가이드

> [!warning] 중요 고려사항
> - **데이터 무결성**: 금융 데이터나 중요 정보는 반드시 InnoDB 사용
> - **성능 vs 안정성**: MyISAM은 빠르지만 충돌 시 데이터 손실 위험
> - **메모리 제한**: Memory 엔진은 RAM 크기에 제한됨
> - **마이그레이션**: 엔진 변경 시 다운타임과 데이터 변환 시간 고려

### 엔진별 제한사항

| 기능 | InnoDB | MyISAM | Memory |
|------|---------|---------|---------|
| 트랜잭션 | ✅ | ❌ | ❌ |
| 외래 키 | ✅ | ❌ | ❌ |
| 행 수준 잠금 | ✅ | ❌ | ❌ |
| 충돌 복구 | ✅ | ⚠️ | ❌ |
| FULLTEXT | ✅ | ✅ | ❌ |
| 압축 | ✅ | ✅ | ❌ |
| 데이터 영속성 | ✅ | ✅ | ❌ |

---

## 📚 참고자료

### 관련 문서
- [[08_database-index|데이터베이스 인덱스]]
- [[13_database-scaling|데이터베이스 스케일링]]
- [[18_database-best-practices|데이터베이스 모범 사례]]

### 외부 리소스
- [MySQL Storage Engines](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)
- [InnoDB Storage Engine](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html)
- [Percona Storage Engines](https://www.percona.com/doc/percona-server/)

### 권장 설정
1. **일반 애플리케이션**: InnoDB (기본값 유지)
2. **로그 시스템**: MyISAM 또는 Archive
3. **세션 관리**: Memory 또는 Redis
4. **분석 시스템**: ColumnStore 또는 ClickHouse
5. **캐시 레이어**: Memory 엔진 또는 Memcached

---

> [!quote]
> "올바른 스토리지 엔진 선택은 데이터베이스 성능과 안정성의 기초입니다. 각 엔진의 특성을 이해하고 애플리케이션 요구사항에 맞게 선택하세요."