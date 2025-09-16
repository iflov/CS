---
tags:
  - database
  - sql
  - stored-procedures
  - triggers
  - functions
  - procedures
  - database-programming
created: 2025-01-08
updated: 2025-01-08
---

# 저장 프로시저와 함수 (Stored Procedures & Functions)

> [!info] 개요
> 저장 프로시저(Stored Procedure)는 데이터베이스에 저장되어 실행되는 SQL 코드의 집합입니다. 복잡한 비즈니스 로직을 데이터베이스 레벨에서 처리하여 성능을 향상시키고 코드 재사용성을 높일 수 있습니다.

## 📑 목차

- [[#⚡ 빠른 시작]]
- [[#🔧 핵심 개념]]
- [[#💡 실전 예제]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## ⚡ 빠른 시작

### 저장 프로시저 기본

> [!example] 간단한 저장 프로시저
> ```sql
> -- MySQL 저장 프로시저 생성
> DELIMITER $$
> CREATE PROCEDURE GetUserById(
>     IN userId INT,
>     OUT userName VARCHAR(100)
> )
> BEGIN
>     SELECT name INTO userName
>     FROM users
>     WHERE id = userId;
> END$$
> DELIMITER ;
> 
> -- 프로시저 호출
> CALL GetUserById(1, @name);
> SELECT @name;
> ```

### TypeScript에서 저장 프로시저 호출

```typescript
import mysql from 'mysql2/promise';

interface DatabaseConfig {
  host: string;
  user: string;
  password: string;
  database: string;
}

class StoredProcedureManager {
  private connection: mysql.Connection | null = null;

  constructor(private config: DatabaseConfig) {}

  async connect(): Promise<void> {
    this.connection = await mysql.createConnection(this.config);
  }

  // 저장 프로시저 호출 래퍼
  async callProcedure<T>(
    procedureName: string,
    params: any[] = []
  ): Promise<T[]> {
    if (!this.connection) {
      throw new Error('Database not connected');
    }

    const placeholders = params.map(() => '?').join(', ');
    const query = `CALL ${procedureName}(${placeholders})`;
    
    try {
      const [results] = await this.connection.execute(query, params);
      return results as T[];
    } catch (error) {
      console.error(`Error calling procedure ${procedureName}:`, error);
      throw error;
    }
  }

  // 트랜잭션 내에서 프로시저 실행
  async executeInTransaction<T>(
    callback: (connection: mysql.Connection) => Promise<T>
  ): Promise<T> {
    if (!this.connection) {
      throw new Error('Database not connected');
    }

    await this.connection.beginTransaction();
    
    try {
      const result = await callback(this.connection);
      await this.connection.commit();
      return result;
    } catch (error) {
      await this.connection.rollback();
      throw error;
    }
  }

  async disconnect(): Promise<void> {
    if (this.connection) {
      await this.connection.end();
      this.connection = null;
    }
  }
}

// 비즈니스 로직 레이어
class UserService {
  constructor(private dbManager: StoredProcedureManager) {}

  async getUserWithOrders(userId: number): Promise<any> {
    interface UserOrder {
      user_id: number;
      user_name: string;
      order_id: number;
      order_date: Date;
      total_amount: number;
    }

    const results = await this.dbManager.callProcedure<UserOrder>(
      'GetUserWithOrders',
      [userId]
    );

    // 결과 가공
    if (results.length === 0) {
      return null;
    }

    const user = {
      id: results[0].user_id,
      name: results[0].user_name,
      orders: results.map(row => ({
        id: row.order_id,
        date: row.order_date,
        totalAmount: row.total_amount
      }))
    };

    return user;
  }

  async createUserWithProfile(
    userData: {
      name: string;
      email: string;
      age: number;
      address: string;
    }
  ): Promise<number> {
    return await this.dbManager.executeInTransaction(async (conn) => {
      // 복잡한 비즈니스 로직을 저장 프로시저로 처리
      const [result]: any = await conn.execute(
        'CALL CreateUserWithProfile(?, ?, ?, ?, @newUserId)',
        [userData.name, userData.email, userData.age, userData.address]
      );
      
      const [rows]: any = await conn.execute('SELECT @newUserId as userId');
      return rows[0].userId;
    });
  }
}
```

---

## 🔧 핵심 개념

### 저장 프로시저 vs 함수

> [!note] 주요 차이점
> - **저장 프로시저**: 값을 반환하지 않거나 여러 값을 OUT 파라미터로 반환
> - **함수**: 단일 값을 반환하며 SELECT 문에서 직접 사용 가능
> - **트리거**: 특정 이벤트(INSERT, UPDATE, DELETE)에 자동 실행

### 저장 프로시저 구성 요소

```sql
-- 복잡한 비즈니스 로직 프로시저
DELIMITER $$
CREATE PROCEDURE ProcessMonthlyReport(
    IN reportMonth INT,
    IN reportYear INT,
    OUT totalRevenue DECIMAL(10,2),
    OUT totalOrders INT,
    OUT topCustomerId INT
)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE customerId INT;
    DECLARE customerRevenue DECIMAL(10,2);
    DECLARE maxRevenue DECIMAL(10,2) DEFAULT 0;
    
    -- 커서 선언
    DECLARE customer_cursor CURSOR FOR
        SELECT customer_id, SUM(amount) as revenue
        FROM orders
        WHERE MONTH(order_date) = reportMonth
        AND YEAR(order_date) = reportYear
        GROUP BY customer_id;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    -- 총 매출과 주문 수 계산
    SELECT 
        SUM(amount),
        COUNT(*)
    INTO totalRevenue, totalOrders
    FROM orders
    WHERE MONTH(order_date) = reportMonth
    AND YEAR(order_date) = reportYear;
    
    -- 최고 매출 고객 찾기
    OPEN customer_cursor;
    
    read_loop: LOOP
        FETCH customer_cursor INTO customerId, customerRevenue;
        
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        IF customerRevenue > maxRevenue THEN
            SET maxRevenue = customerRevenue;
            SET topCustomerId = customerId;
        END IF;
    END LOOP;
    
    CLOSE customer_cursor;
    
    -- 로그 테이블에 기록
    INSERT INTO report_logs (report_type, month, year, generated_at)
    VALUES ('MONTHLY_REVENUE', reportMonth, reportYear, NOW());
END$$
DELIMITER ;
```

### TypeScript 저장 프로시저 관리 시스템

```typescript
// 저장 프로시저 메타데이터 관리
interface ProcedureMetadata {
  name: string;
  parameters: {
    name: string;
    type: 'IN' | 'OUT' | 'INOUT';
    dataType: string;
  }[];
  description: string;
  version: string;
}

class ProcedureRegistry {
  private procedures: Map<string, ProcedureMetadata> = new Map();
  
  register(metadata: ProcedureMetadata): void {
    this.procedures.set(metadata.name, metadata);
  }
  
  validateCall(name: string, params: any[]): boolean {
    const metadata = this.procedures.get(name);
    if (!metadata) {
      throw new Error(`Procedure ${name} not registered`);
    }
    
    const inParams = metadata.parameters.filter(p => 
      p.type === 'IN' || p.type === 'INOUT'
    );
    
    if (params.length !== inParams.length) {
      throw new Error(
        `Parameter count mismatch for ${name}. ` +
        `Expected ${inParams.length}, got ${params.length}`
      );
    }
    
    return true;
  }
  
  getDocumentation(name: string): string {
    const metadata = this.procedures.get(name);
    if (!metadata) {
      return 'Procedure not found';
    }
    
    const params = metadata.parameters
      .map(p => `${p.name} ${p.type} ${p.dataType}`)
      .join(', ');
    
    return `
Procedure: ${metadata.name}
Version: ${metadata.version}
Description: ${metadata.description}
Parameters: ${params}
    `.trim();
  }
}

// 프로시저 버전 관리
class ProcedureVersionManager {
  private versions: Map<string, string[]> = new Map();
  
  async deployProcedure(
    connection: mysql.Connection,
    procedureName: string,
    procedureSQL: string,
    version: string
  ): Promise<void> {
    // 기존 프로시저 백업
    await this.backupProcedure(connection, procedureName);
    
    // 새 버전 배포
    await connection.execute(`DROP PROCEDURE IF EXISTS ${procedureName}`);
    await connection.execute(procedureSQL);
    
    // 버전 기록
    const versions = this.versions.get(procedureName) || [];
    versions.push(version);
    this.versions.set(procedureName, versions);
    
    // 메타데이터 테이블 업데이트
    await connection.execute(
      `INSERT INTO procedure_versions (name, version, deployed_at, sql_text)
       VALUES (?, ?, NOW(), ?)`,
      [procedureName, version, procedureSQL]
    );
  }
  
  private async backupProcedure(
    connection: mysql.Connection,
    procedureName: string
  ): Promise<void> {
    const [rows]: any = await connection.execute(
      `SHOW CREATE PROCEDURE ${procedureName}`
    );
    
    if (rows.length > 0) {
      const backupName = `${procedureName}_backup_${Date.now()}`;
      const createSQL = rows[0]['Create Procedure']
        .replace(procedureName, backupName);
      
      await connection.execute(createSQL);
    }
  }
  
  async rollback(
    connection: mysql.Connection,
    procedureName: string,
    targetVersion: string
  ): Promise<void> {
    const [rows]: any = await connection.execute(
      `SELECT sql_text FROM procedure_versions 
       WHERE name = ? AND version = ?`,
      [procedureName, targetVersion]
    );
    
    if (rows.length > 0) {
      await connection.execute(`DROP PROCEDURE IF EXISTS ${procedureName}`);
      await connection.execute(rows[0].sql_text);
    }
  }
}
```

---

## 💡 실전 예제

### 재고 관리 시스템

```typescript
// 재고 관리 저장 프로시저 래퍼
class InventoryProcedures {
  constructor(private db: StoredProcedureManager) {}
  
  // 재고 차감 프로시저
  async processOrder(
    orderId: number,
    items: { productId: number; quantity: number }[]
  ): Promise<boolean> {
    return await this.db.executeInTransaction(async (conn) => {
      for (const item of items) {
        const [result]: any = await conn.execute(
          'CALL DeductInventory(?, ?, @success, @message)',
          [item.productId, item.quantity]
        );
        
        const [status]: any = await conn.execute(
          'SELECT @success as success, @message as message'
        );
        
        if (!status[0].success) {
          throw new Error(
            `Inventory deduction failed: ${status[0].message}`
          );
        }
      }
      
      // 주문 상태 업데이트
      await conn.execute(
        'CALL UpdateOrderStatus(?, "PROCESSED")',
        [orderId]
      );
      
      return true;
    });
  }
  
  // 재고 보충 프로시저
  async restockProducts(
    restockData: {
      productId: number;
      quantity: number;
      supplierId: number;
      cost: number;
    }[]
  ): Promise<number[]> {
    const restockIds: number[] = [];
    
    await this.db.executeInTransaction(async (conn) => {
      for (const data of restockData) {
        await conn.execute(
          'CALL RestockProduct(?, ?, ?, ?, @restockId)',
          [data.productId, data.quantity, data.supplierId, data.cost]
        );
        
        const [result]: any = await conn.execute(
          'SELECT @restockId as restockId'
        );
        
        restockIds.push(result[0].restockId);
      }
    });
    
    return restockIds;
  }
}

// 성능 모니터링 프로시저
class PerformanceMonitor {
  private metrics: Map<string, number[]> = new Map();
  
  async monitorProcedure<T>(
    procedureName: string,
    executor: () => Promise<T>
  ): Promise<T> {
    const startTime = performance.now();
    
    try {
      const result = await executor();
      const duration = performance.now() - startTime;
      
      // 메트릭 수집
      const metrics = this.metrics.get(procedureName) || [];
      metrics.push(duration);
      this.metrics.set(procedureName, metrics);
      
      // 임계값 초과 시 경고
      if (duration > 1000) {
        console.warn(
          `Slow procedure execution: ${procedureName} took ${duration}ms`
        );
      }
      
      return result;
    } catch (error) {
      const duration = performance.now() - startTime;
      console.error(
        `Procedure ${procedureName} failed after ${duration}ms`,
        error
      );
      throw error;
    }
  }
  
  getStatistics(procedureName: string): {
    count: number;
    avg: number;
    min: number;
    max: number;
    p95: number;
  } | null {
    const metrics = this.metrics.get(procedureName);
    if (!metrics || metrics.length === 0) {
      return null;
    }
    
    const sorted = [...metrics].sort((a, b) => a - b);
    const sum = sorted.reduce((a, b) => a + b, 0);
    const p95Index = Math.floor(sorted.length * 0.95);
    
    return {
      count: sorted.length,
      avg: sum / sorted.length,
      min: sorted[0],
      max: sorted[sorted.length - 1],
      p95: sorted[p95Index]
    };
  }
}
```

### 트리거 구현

```sql
-- 감사(Audit) 트리거
DELIMITER $$
CREATE TRIGGER audit_user_changes
AFTER UPDATE ON users
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (
        table_name,
        record_id,
        action,
        old_value,
        new_value,
        changed_by,
        changed_at
    ) VALUES (
        'users',
        NEW.id,
        'UPDATE',
        JSON_OBJECT('name', OLD.name, 'email', OLD.email),
        JSON_OBJECT('name', NEW.name, 'email', NEW.email),
        USER(),
        NOW()
    );
END$$
DELIMITER ;

-- 재고 자동 알림 트리거
DELIMITER $$
CREATE TRIGGER low_stock_alert
AFTER UPDATE ON inventory
FOR EACH ROW
BEGIN
    IF NEW.quantity < NEW.min_stock_level AND OLD.quantity >= OLD.min_stock_level THEN
        INSERT INTO notifications (
            type,
            message,
            product_id,
            created_at,
            status
        ) VALUES (
            'LOW_STOCK',
            CONCAT('Product ', NEW.product_name, ' is low on stock: ', NEW.quantity),
            NEW.product_id,
            NOW(),
            'PENDING'
        );
    END IF;
END$$
DELIMITER ;
```

---

## ⚠️ 주의사항

### 저장 프로시저 사용 시 고려사항

> [!warning] 주의점
> - **디버깅 어려움**: 데이터베이스 내부 로직 디버깅이 애플리케이션보다 어려움
> - **버전 관리**: 소스 코드와 별도로 관리되어 동기화 문제 발생 가능
> - **이식성**: 데이터베이스 벤더별로 문법이 다름
> - **성능**: 복잡한 로직은 애플리케이션 서버에서 처리하는 것이 더 효율적일 수 있음

### 보안 고려사항

```typescript
// SQL 인젝션 방지를 위한 프로시저 호출
class SecureProcedureExecutor {
  private allowedProcedures: Set<string> = new Set([
    'GetUserById',
    'CreateOrder',
    'UpdateInventory'
  ]);
  
  async execute(
    connection: mysql.Connection,
    procedureName: string,
    params: any[]
  ): Promise<any> {
    // 화이트리스트 검증
    if (!this.allowedProcedures.has(procedureName)) {
      throw new Error(`Procedure ${procedureName} is not allowed`);
    }
    
    // 파라미터 타입 검증
    this.validateParameters(procedureName, params);
    
    // 안전한 실행
    const placeholders = params.map(() => '?').join(', ');
    const query = `CALL ${procedureName}(${placeholders})`;
    
    return await connection.execute(query, params);
  }
  
  private validateParameters(procedureName: string, params: any[]): void {
    // 각 프로시저별 파라미터 검증 로직
    switch (procedureName) {
      case 'GetUserById':
        if (params.length !== 1 || typeof params[0] !== 'number') {
          throw new Error('Invalid parameters for GetUserById');
        }
        break;
      // ... 다른 프로시저들
    }
  }
}
```

---

## 📚 참고자료

### 관련 문서
- [[07_advanced-sql-queries|고급 SQL 쿼리]]
- [[08_database-index|데이터베이스 인덱스]]
- [[17_database-security|데이터베이스 보안]]

### 외부 리소스
- [MySQL Stored Procedures](https://dev.mysql.com/doc/refman/8.0/en/stored-routines.html)
- [PostgreSQL Functions](https://www.postgresql.org/docs/current/sql-createfunction.html)
- [SQL Server Stored Procedures](https://docs.microsoft.com/en-us/sql/relational-databases/stored-procedures/)

### 모범 사례
1. **명명 규칙**: sp_[동작]_[대상] (예: sp_get_user)
2. **에러 처리**: 항상 예외 처리 포함
3. **문서화**: 각 프로시저의 목적과 파라미터 문서화
4. **테스트**: 단위 테스트 작성
5. **모니터링**: 실행 시간과 리소스 사용량 추적

---

> [!quote]
> "저장 프로시저는 강력한 도구이지만, 적절한 상황에서만 사용해야 합니다. 비즈니스 로직의 복잡도와 유지보수성을 고려하여 신중히 선택하세요."