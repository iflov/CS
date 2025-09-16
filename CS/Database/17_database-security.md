---
tags:
  - database
  - security
  - authentication
  - encryption
  - sql-injection
  - access-control
  - audit
created: 2025-01-08
updated: 2025-01-08
---

# 데이터베이스 보안 (Database Security)

> [!info] 개요
> 데이터베이스 보안은 민감한 데이터를 보호하고 무단 액세스를 방지하는 포괄적인 접근 방식입니다. SQL 인젝션 방지, 접근 제어, 암호화, 감사 등 다양한 보안 계층을 통해 데이터를 안전하게 보호합니다.

## 📑 목차

- [[#⚡ 빠른 시작]]
- [[#🔧 핵심 보안 영역]]
- [[#💡 실전 예제]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## ⚡ 빠른 시작

### SQL 인젝션 방지

> [!danger] SQL 인젝션 위험
> ```typescript
> // ❌ 위험한 코드 - SQL 인젝션 취약점
> const getUserBad = async (userId: string) => {
>   const query = `SELECT * FROM users WHERE id = ${userId}`;
>   // userId = "1 OR 1=1" 입력 시 모든 사용자 정보 노출
>   return await connection.execute(query);
> };
> 
> // ✅ 안전한 코드 - Prepared Statement 사용
> const getUserSafe = async (userId: string) => {
>   const query = 'SELECT * FROM users WHERE id = ?';
>   return await connection.execute(query, [userId]);
> };
> ```

### TypeScript 보안 레이어 구현

```typescript
import mysql from 'mysql2/promise';
import crypto from 'crypto';
import bcrypt from 'bcrypt';

interface SecurityConfig {
  encryptionKey: string;
  saltRounds: number;
  maxLoginAttempts: number;
  lockoutDuration: number; // minutes
}

class DatabaseSecurity {
  private connection: mysql.Connection | null = null;
  private encryptionKey: Buffer;

  constructor(
    private config: mysql.ConnectionOptions,
    private securityConfig: SecurityConfig
  ) {
    this.encryptionKey = Buffer.from(securityConfig.encryptionKey, 'hex');
  }

  async connect(): Promise<void> {
    // SSL/TLS 연결 강제
    this.connection = await mysql.createConnection({
      ...this.config,
      ssl: {
        rejectUnauthorized: true
      }
    });
  }

  // 입력 검증 및 살균
  sanitizeInput(input: string): string {
    // 특수 문자 이스케이프
    return input
      .replace(/['"\\]/g, '\\$&')
      .replace(/[\x00-\x1f\x7f]/g, '') // 제어 문자 제거
      .trim();
  }

  // 파라미터화된 쿼리 실행
  async executeSecureQuery<T>(
    query: string,
    params: any[] = []
  ): Promise<T[]> {
    if (!this.connection) throw new Error('Not connected');

    // 파라미터 검증
    const sanitizedParams = params.map(param => {
      if (typeof param === 'string') {
        return this.sanitizeInput(param);
      }
      return param;
    });

    try {
      const [rows] = await this.connection.execute(query, sanitizedParams);
      return rows as T[];
    } catch (error) {
      // 에러 로깅 (민감한 정보 제외)
      console.error('Query execution failed:', {
        query: query.replace(/\?/g, '?'), // 파라미터 값 숨김
        error: error.message
      });
      throw new Error('Database operation failed');
    }
  }

  // 데이터 암호화
  encrypt(text: string): string {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(
      'aes-256-cbc',
      this.encryptionKey,
      iv
    );
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return iv.toString('hex') + ':' + encrypted;
  }

  // 데이터 복호화
  decrypt(encryptedText: string): string {
    const parts = encryptedText.split(':');
    const iv = Buffer.from(parts[0], 'hex');
    const encrypted = parts[1];
    
    const decipher = crypto.createDecipheriv(
      'aes-256-cbc',
      this.encryptionKey,
      iv
    );
    
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }

  // 비밀번호 해싱
  async hashPassword(password: string): Promise<string> {
    return await bcrypt.hash(password, this.securityConfig.saltRounds);
  }

  // 비밀번호 검증
  async verifyPassword(
    password: string,
    hash: string
  ): Promise<boolean> {
    return await bcrypt.compare(password, hash);
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

## 🔧 핵심 보안 영역

### 1. 접근 제어 (Access Control)

> [!note] 권한 관리 원칙
> - **최소 권한 원칙**: 필요한 최소한의 권한만 부여
> - **역할 기반 접근 제어**: RBAC 구현
> - **세분화된 권한**: 테이블/컬럼 수준 권한 설정

```typescript
// 역할 기반 접근 제어 시스템
class RBACManager {
  constructor(private connection: mysql.Connection) {}

  // 역할 생성
  async createRole(
    roleName: string,
    permissions: string[]
  ): Promise<void> {
    // 역할 생성
    await this.connection.execute(
      'INSERT INTO roles (name, created_at) VALUES (?, NOW())',
      [roleName]
    );

    // 권한 할당
    for (const permission of permissions) {
      await this.connection.execute(
        `INSERT INTO role_permissions (role_name, permission)
         VALUES (?, ?)`,
        [roleName, permission]
      );
    }
  }

  // 사용자에게 역할 할당
  async assignRole(
    userId: number,
    roleName: string
  ): Promise<void> {
    await this.connection.execute(
      `INSERT INTO user_roles (user_id, role_name, assigned_at)
       VALUES (?, ?, NOW())`,
      [userId, roleName]
    );
  }

  // 권한 확인
  async checkPermission(
    userId: number,
    permission: string
  ): Promise<boolean> {
    const [rows] = await this.connection.execute<any[]>(
      `SELECT COUNT(*) as count
       FROM user_roles ur
       JOIN role_permissions rp ON ur.role_name = rp.role_name
       WHERE ur.user_id = ? AND rp.permission = ?`,
      [userId, permission]
    );

    return rows[0].count > 0;
  }

  // 동적 SQL 권한 생성
  generateGrants(
    username: string,
    database: string,
    table: string,
    privileges: string[]
  ): string {
    const privString = privileges.join(', ');
    return `GRANT ${privString} ON ${database}.${table} TO '${username}'@'%'`;
  }

  // 권한 감사
  async auditPermissions(): Promise<any[]> {
    const [rows] = await this.connection.execute(`
      SELECT 
        u.username,
        GROUP_CONCAT(DISTINCT ur.role_name) as roles,
        GROUP_CONCAT(DISTINCT rp.permission) as permissions
      FROM users u
      LEFT JOIN user_roles ur ON u.id = ur.user_id
      LEFT JOIN role_permissions rp ON ur.role_name = rp.role_name
      GROUP BY u.id
    `);

    return rows;
  }
}
```

### 2. 데이터 암호화

```typescript
// 필드 수준 암호화 관리
class FieldEncryption {
  private algorithm = 'aes-256-gcm';
  private keyDerivation = 'pbkdf2';

  constructor(private masterKey: string) {}

  // 데이터 키 생성
  generateDataKey(): {
    encryptedKey: string;
    plainKey: Buffer;
  } {
    const plainKey = crypto.randomBytes(32);
    const cipher = crypto.createCipher('aes-256-cbc', this.masterKey);
    
    let encryptedKey = cipher.update(plainKey.toString('hex'), 'utf8', 'hex');
    encryptedKey += cipher.final('hex');
    
    return { encryptedKey, plainKey };
  }

  // 민감한 필드 암호화
  async encryptField(
    data: string,
    dataKey: Buffer
  ): Promise<{
    encrypted: string;
    authTag: string;
    iv: string;
  }> {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(this.algorithm, dataKey, iv);
    
    let encrypted = cipher.update(data, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = (cipher as any).getAuthTag().toString('hex');
    
    return {
      encrypted,
      authTag,
      iv: iv.toString('hex')
    };
  }

  // 민감한 필드 복호화
  async decryptField(
    encryptedData: string,
    authTag: string,
    iv: string,
    dataKey: Buffer
  ): Promise<string> {
    const decipher = crypto.createDecipheriv(
      this.algorithm,
      dataKey,
      Buffer.from(iv, 'hex')
    );
    
    (decipher as any).setAuthTag(Buffer.from(authTag, 'hex'));
    
    let decrypted = decipher.update(encryptedData, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}

// 투명한 데이터 암호화 (TDE) 시뮬레이션
class TransparentDataEncryption {
  private encryptionManager: FieldEncryption;
  private dataKeys: Map<string, Buffer> = new Map();

  constructor(
    private connection: mysql.Connection,
    masterKey: string
  ) {
    this.encryptionManager = new FieldEncryption(masterKey);
  }

  // 암호화된 테이블 생성
  async createEncryptedTable(
    tableName: string,
    columns: Array<{
      name: string;
      type: string;
      encrypted?: boolean;
    }>
  ): Promise<void> {
    const columnDefs = columns.map(col => {
      if (col.encrypted) {
        // 암호화된 필드는 더 큰 크기로 저장
        return `${col.name}_encrypted VARBINARY(512),
                ${col.name}_auth_tag VARCHAR(32),
                ${col.name}_iv VARCHAR(32)`;
      }
      return `${col.name} ${col.type}`;
    }).join(', ');

    await this.connection.execute(`
      CREATE TABLE ${tableName} (
        id INT AUTO_INCREMENT PRIMARY KEY,
        ${columnDefs},
        data_key_id VARCHAR(64)
      )
    `);

    // 테이블용 데이터 키 생성
    const { encryptedKey, plainKey } = this.encryptionManager.generateDataKey();
    this.dataKeys.set(tableName, plainKey);

    // 키 관리 테이블에 저장
    await this.connection.execute(
      `INSERT INTO encryption_keys (table_name, encrypted_key, created_at)
       VALUES (?, ?, NOW())`,
      [tableName, encryptedKey]
    );
  }

  // 암호화하여 데이터 삽입
  async insertEncrypted(
    tableName: string,
    data: Record<string, any>,
    encryptedFields: string[]
  ): Promise<void> {
    const dataKey = this.dataKeys.get(tableName);
    if (!dataKey) throw new Error('Data key not found');

    const columns: string[] = [];
    const values: any[] = [];
    const placeholders: string[] = [];

    for (const [field, value] of Object.entries(data)) {
      if (encryptedFields.includes(field)) {
        const { encrypted, authTag, iv } = 
          await this.encryptionManager.encryptField(
            JSON.stringify(value),
            dataKey
          );
        
        columns.push(
          `${field}_encrypted`,
          `${field}_auth_tag`,
          `${field}_iv`
        );
        values.push(encrypted, authTag, iv);
        placeholders.push('?', '?', '?');
      } else {
        columns.push(field);
        values.push(value);
        placeholders.push('?');
      }
    }

    await this.connection.execute(
      `INSERT INTO ${tableName} (${columns.join(', ')})
       VALUES (${placeholders.join(', ')})`,
      values
    );
  }
}
```

### 3. 감사 및 모니터링

```typescript
// 데이터베이스 감사 시스템
class AuditSystem {
  constructor(private connection: mysql.Connection) {}

  // 감사 로그 테이블 생성
  async createAuditTables(): Promise<void> {
    // 액세스 로그
    await this.connection.execute(`
      CREATE TABLE IF NOT EXISTS audit_access_log (
        id BIGINT AUTO_INCREMENT PRIMARY KEY,
        user_id INT,
        username VARCHAR(100),
        ip_address VARCHAR(45),
        action VARCHAR(50),
        table_name VARCHAR(100),
        record_id INT,
        query_text TEXT,
        success BOOLEAN,
        error_message TEXT,
        timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        INDEX idx_user_timestamp (user_id, timestamp),
        INDEX idx_action (action),
        INDEX idx_table (table_name)
      )
    `);

    // 데이터 변경 로그
    await this.connection.execute(`
      CREATE TABLE IF NOT EXISTS audit_data_changes (
        id BIGINT AUTO_INCREMENT PRIMARY KEY,
        table_name VARCHAR(100),
        record_id INT,
        operation ENUM('INSERT', 'UPDATE', 'DELETE'),
        old_values JSON,
        new_values JSON,
        changed_by INT,
        changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        INDEX idx_table_record (table_name, record_id),
        INDEX idx_changed_by (changed_by)
      )
    `);

    // 보안 이벤트 로그
    await this.connection.execute(`
      CREATE TABLE IF NOT EXISTS audit_security_events (
        id BIGINT AUTO_INCREMENT PRIMARY KEY,
        event_type VARCHAR(50),
        severity ENUM('LOW', 'MEDIUM', 'HIGH', 'CRITICAL'),
        user_id INT,
        ip_address VARCHAR(45),
        details JSON,
        detected_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        INDEX idx_severity (severity),
        INDEX idx_event_type (event_type)
      )
    `);
  }

  // 액세스 로깅
  async logAccess(
    userId: number,
    username: string,
    ipAddress: string,
    action: string,
    tableName: string,
    recordId: number | null,
    queryText: string,
    success: boolean,
    errorMessage?: string
  ): Promise<void> {
    await this.connection.execute(
      `INSERT INTO audit_access_log
       (user_id, username, ip_address, action, table_name, record_id, 
        query_text, success, error_message)
       VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)`,
      [userId, username, ipAddress, action, tableName, recordId,
       queryText, success, errorMessage || null]
    );
  }

  // 의심스러운 활동 감지
  async detectSuspiciousActivity(
    userId: number,
    timeWindow: number = 60 // minutes
  ): Promise<any[]> {
    const [suspicious] = await this.connection.execute<any[]>(`
      SELECT 
        user_id,
        COUNT(*) as attempt_count,
        COUNT(DISTINCT table_name) as tables_accessed,
        SUM(CASE WHEN success = false THEN 1 ELSE 0 END) as failed_attempts,
        GROUP_CONCAT(DISTINCT action) as actions
      FROM audit_access_log
      WHERE user_id = ?
        AND timestamp > DATE_SUB(NOW(), INTERVAL ? MINUTE)
      GROUP BY user_id
      HAVING attempt_count > 100
         OR failed_attempts > 10
         OR tables_accessed > 20
    `, [userId, timeWindow]);

    // 의심스러운 활동 발견 시 보안 이벤트 기록
    for (const activity of suspicious) {
      await this.logSecurityEvent(
        'SUSPICIOUS_ACTIVITY',
        'HIGH',
        userId,
        '',
        activity
      );
    }

    return suspicious;
  }

  // 보안 이벤트 로깅
  async logSecurityEvent(
    eventType: string,
    severity: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL',
    userId: number,
    ipAddress: string,
    details: any
  ): Promise<void> {
    await this.connection.execute(
      `INSERT INTO audit_security_events
       (event_type, severity, user_id, ip_address, details)
       VALUES (?, ?, ?, ?, ?)`,
      [eventType, severity, userId, ipAddress, JSON.stringify(details)]
    );

    // CRITICAL 이벤트는 즉시 알림
    if (severity === 'CRITICAL') {
      await this.sendSecurityAlert(eventType, details);
    }
  }

  // 보안 알림 전송
  private async sendSecurityAlert(
    eventType: string,
    details: any
  ): Promise<void> {
    // 실제 구현에서는 이메일, SMS, Slack 등으로 알림
    console.error(`🚨 CRITICAL SECURITY EVENT: ${eventType}`, details);
  }

  // 감사 보고서 생성
  async generateAuditReport(
    startDate: Date,
    endDate: Date
  ): Promise<{
    summary: any;
    topUsers: any[];
    failedAttempts: any[];
    securityEvents: any[];
  }> {
    // 요약 통계
    const [summary] = await this.connection.execute<any[]>(`
      SELECT 
        COUNT(*) as total_accesses,
        COUNT(DISTINCT user_id) as unique_users,
        COUNT(DISTINCT table_name) as tables_accessed,
        SUM(CASE WHEN success = false THEN 1 ELSE 0 END) as failed_attempts
      FROM audit_access_log
      WHERE timestamp BETWEEN ? AND ?
    `, [startDate, endDate]);

    // 가장 활발한 사용자
    const [topUsers] = await this.connection.execute<any[]>(`
      SELECT 
        user_id,
        username,
        COUNT(*) as access_count
      FROM audit_access_log
      WHERE timestamp BETWEEN ? AND ?
      GROUP BY user_id, username
      ORDER BY access_count DESC
      LIMIT 10
    `, [startDate, endDate]);

    // 실패한 시도
    const [failedAttempts] = await this.connection.execute<any[]>(`
      SELECT 
        user_id,
        username,
        action,
        table_name,
        error_message,
        timestamp
      FROM audit_access_log
      WHERE success = false
        AND timestamp BETWEEN ? AND ?
      ORDER BY timestamp DESC
      LIMIT 100
    `, [startDate, endDate]);

    // 보안 이벤트
    const [securityEvents] = await this.connection.execute<any[]>(`
      SELECT *
      FROM audit_security_events
      WHERE detected_at BETWEEN ? AND ?
      ORDER BY severity DESC, detected_at DESC
    `, [startDate, endDate]);

    return {
      summary: summary[0],
      topUsers,
      failedAttempts,
      securityEvents
    };
  }
}
```

---

## 💡 실전 예제

### 종합 보안 시스템 구현

```typescript
// 통합 보안 관리자
class IntegratedSecurityManager {
  private dbSecurity: DatabaseSecurity;
  private rbac: RBACManager;
  private audit: AuditSystem;
  private loginAttempts: Map<string, number> = new Map();

  constructor(
    private connection: mysql.Connection,
    private config: SecurityConfig
  ) {
    this.dbSecurity = new DatabaseSecurity(
      {} as any,
      config
    );
    this.rbac = new RBACManager(connection);
    this.audit = new AuditSystem(connection);
  }

  // 안전한 로그인 처리
  async secureLogin(
    username: string,
    password: string,
    ipAddress: string
  ): Promise<{
    success: boolean;
    userId?: number;
    token?: string;
    message: string;
  }> {
    // 로그인 시도 횟수 확인
    const attempts = this.loginAttempts.get(username) || 0;
    
    if (attempts >= this.config.maxLoginAttempts) {
      await this.audit.logSecurityEvent(
        'ACCOUNT_LOCKED',
        'HIGH',
        0,
        ipAddress,
        { username, attempts }
      );
      
      return {
        success: false,
        message: 'Account locked due to too many failed attempts'
      };
    }

    try {
      // 사용자 조회 (prepared statement)
      const [users] = await this.connection.execute<any[]>(
        'SELECT id, username, password_hash FROM users WHERE username = ?',
        [username]
      );

      if (users.length === 0) {
        this.loginAttempts.set(username, attempts + 1);
        await this.audit.logAccess(
          0, username, ipAddress, 'LOGIN', 'users', null,
          'Login attempt', false, 'User not found'
        );
        return { success: false, message: 'Invalid credentials' };
      }

      const user = users[0];
      
      // 비밀번호 검증
      const isValid = await this.dbSecurity.verifyPassword(
        password,
        user.password_hash
      );

      if (!isValid) {
        this.loginAttempts.set(username, attempts + 1);
        await this.audit.logAccess(
          user.id, username, ipAddress, 'LOGIN', 'users', user.id,
          'Login attempt', false, 'Invalid password'
        );
        return { success: false, message: 'Invalid credentials' };
      }

      // 로그인 성공
      this.loginAttempts.delete(username);
      
      // 세션 토큰 생성
      const token = crypto.randomBytes(32).toString('hex');
      
      // 세션 저장
      await this.connection.execute(
        `INSERT INTO sessions (user_id, token, ip_address, created_at, expires_at)
         VALUES (?, ?, ?, NOW(), DATE_ADD(NOW(), INTERVAL 1 HOUR))`,
        [user.id, token, ipAddress]
      );

      await this.audit.logAccess(
        user.id, username, ipAddress, 'LOGIN', 'users', user.id,
        'Login successful', true
      );

      return {
        success: true,
        userId: user.id,
        token,
        message: 'Login successful'
      };

    } catch (error) {
      await this.audit.logSecurityEvent(
        'LOGIN_ERROR',
        'MEDIUM',
        0,
        ipAddress,
        { username, error: error.message }
      );
      
      throw error;
    }
  }

  // 권한 기반 쿼리 실행
  async executeWithPermission(
    userId: number,
    permission: string,
    query: string,
    params: any[]
  ): Promise<any> {
    // 권한 확인
    const hasPermission = await this.rbac.checkPermission(
      userId,
      permission
    );

    if (!hasPermission) {
      await this.audit.logAccess(
        userId, '', '', 'QUERY', '', null,
        query, false, 'Permission denied'
      );
      
      throw new Error('Permission denied');
    }

    // 쿼리 실행
    try {
      const result = await this.dbSecurity.executeSecureQuery(
        query,
        params
      );
      
      await this.audit.logAccess(
        userId, '', '', 'QUERY', '', null,
        query, true
      );
      
      return result;
    } catch (error) {
      await this.audit.logAccess(
        userId, '', '', 'QUERY', '', null,
        query, false, error.message
      );
      
      throw error;
    }
  }

  // 데이터 마스킹
  maskSensitiveData(
    data: any[],
    sensitiveFields: string[]
  ): any[] {
    return data.map(row => {
      const maskedRow = { ...row };
      
      for (const field of sensitiveFields) {
        if (maskedRow[field]) {
          const value = String(maskedRow[field]);
          if (field.includes('email')) {
            // 이메일 마스킹: u***@domain.com
            const [local, domain] = value.split('@');
            maskedRow[field] = local[0] + '***@' + domain;
          } else if (field.includes('phone')) {
            // 전화번호 마스킹: ***-****-1234
            maskedRow[field] = '***-****-' + value.slice(-4);
          } else if (field.includes('ssn') || field.includes('card')) {
            // SSN/카드번호 마스킹: ****-****-****-1234
            maskedRow[field] = '****-****-****-' + value.slice(-4);
          } else {
            // 기본 마스킹: 첫 글자와 마지막 글자만 표시
            maskedRow[field] = value[0] + '***' + value.slice(-1);
          }
        }
      }
      
      return maskedRow;
    });
  }
}

// 보안 정책 시행
class SecurityPolicyEnforcer {
  constructor(private connection: mysql.Connection) {}

  // 비밀번호 정책 검증
  validatePasswordPolicy(password: string): {
    valid: boolean;
    errors: string[];
  } {
    const errors: string[] = [];
    
    if (password.length < 12) {
      errors.push('Password must be at least 12 characters');
    }
    if (!/[A-Z]/.test(password)) {
      errors.push('Password must contain uppercase letters');
    }
    if (!/[a-z]/.test(password)) {
      errors.push('Password must contain lowercase letters');
    }
    if (!/[0-9]/.test(password)) {
      errors.push('Password must contain numbers');
    }
    if (!/[!@#$%^&*]/.test(password)) {
      errors.push('Password must contain special characters');
    }
    
    return {
      valid: errors.length === 0,
      errors
    };
  }

  // 정기적인 비밀번호 변경 강제
  async enforcePasswordRotation(
    maxAgeDays: number = 90
  ): Promise<number[]> {
    const [users] = await this.connection.execute<any[]>(`
      SELECT id, username, email
      FROM users
      WHERE password_changed_at < DATE_SUB(NOW(), INTERVAL ? DAY)
    `, [maxAgeDays]);

    const expiredUsers = users.map(u => u.id);
    
    // 비밀번호 변경 필요 플래그 설정
    if (expiredUsers.length > 0) {
      await this.connection.execute(
        `UPDATE users
         SET password_expired = true
         WHERE id IN (${expiredUsers.map(() => '?').join(',')})`,
        expiredUsers
      );
    }
    
    return expiredUsers;
  }

  // 세션 타임아웃 관리
  async cleanupExpiredSessions(): Promise<number> {
    const [result] = await this.connection.execute<any>(
      `DELETE FROM sessions
       WHERE expires_at < NOW()`
    );
    
    return result.affectedRows;
  }
}
```

---

## ⚠️ 주의사항

### 보안 체크리스트

> [!warning] 필수 보안 조치
> - [ ] 모든 입력값 검증 및 살균
> - [ ] Prepared Statement 사용
> - [ ] 최소 권한 원칙 적용
> - [ ] 민감한 데이터 암호화
> - [ ] SSL/TLS 연결 사용
> - [ ] 정기적인 보안 패치
> - [ ] 감사 로깅 활성화
> - [ ] 백업 암호화
> - [ ] 접근 제어 목록 관리
> - [ ] 정기적인 보안 감사

### 일반적인 취약점과 대응

| 취약점 | 위험도 | 대응 방법 |
|--------|--------|-----------|
| SQL 인젝션 | 높음 | Prepared Statement, 입력 검증 |
| 권한 상승 | 높음 | RBAC, 최소 권한 원칙 |
| 데이터 노출 | 높음 | 암호화, 마스킹, 접근 제어 |
| 무차별 대입 | 중간 | 계정 잠금, 속도 제한 |
| 세션 하이재킹 | 중간 | 세션 타임아웃, IP 검증 |
| 감사 우회 | 낮음 | 트리거 기반 로깅 |

---

## 📚 참고자료

### 관련 문서
- [[15_stored-procedures|저장 프로시저와 함수]]
- [[18_database-best-practices|데이터베이스 모범 사례]]

### 외부 리소스
- [OWASP Database Security](https://owasp.org/www-project-database-security/)
- [MySQL Security Guide](https://dev.mysql.com/doc/refman/8.0/en/security.html)
- [PostgreSQL Security](https://www.postgresql.org/docs/current/security.html)

### 보안 표준
1. **PCI DSS**: 카드 데이터 보안 표준
2. **GDPR**: 개인정보 보호 규정
3. **HIPAA**: 의료 정보 보호법
4. **SOC 2**: 서비스 조직 통제
5. **ISO 27001**: 정보보안 관리체계

---

> [!quote]
> "데이터베이스 보안은 다층 방어 전략이 필요합니다. 한 가지 보안 조치만으로는 충분하지 않으며, 여러 보안 계층을 통해 심층 방어를 구축해야 합니다."