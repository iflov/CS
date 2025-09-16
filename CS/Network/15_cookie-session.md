---
title: 쿠키와 세션
created: 2025-01-03
updated: 2025-01-03
tags:
  - CS
  - Network
  - HTTP
  - Cookie
  - Session
  - JWT
  - Authentication
category: Network
status: 완료
---

# 🍪 쿠키와 세션 (Cookie and Session)

## 📌 개요

HTTP는 무상태(Stateless) 프로토콜이므로 각 요청은 독립적입니다. 쿠키와 세션은 이런 HTTP의 한계를 극복하고 상태를 유지하기 위한 메커니즘입니다. JWT는 토큰 기반의 현대적인 인증 방식입니다.

> [!info] 핵심 개념
> - **쿠키**: 클라이언트에 저장되는 작은 데이터
> - **세션**: 서버에 저장되는 사용자 상태
> - **JWT**: JSON Web Token, 자체 포함 토큰
> - **상태 관리**: Stateless HTTP에서 상태 유지

## 📚 목차

1. [HTTP의 무상태성](#http의-무상태성)
2. [쿠키 (Cookie)](#쿠키-cookie)
3. [세션 (Session)](#세션-session)
4. [쿠키 vs 세션](#쿠키-vs-세션)
5. [JWT (JSON Web Token)](#jwt-json-web-token)
6. [토큰 기반 인증](#토큰-기반-인증)
7. [보안 고려사항](#보안-고려사항)
8. [실제 구현 예시](#실제-구현-예시)
9. [참고 자료](#참고-자료)

## HTTP의 무상태성

### Stateless 문제
```
요청 1: 로그인 (user: alice, pass: secret)
서버: OK, 로그인 성공

요청 2: 프로필 조회
서버: 누구세요? 🤔 (이전 요청 기억 못함)
```

### 상태 유지 필요성
- 로그인 상태 유지
- 장바구니 정보 저장
- 사용자 설정 기억
- 세션 추적

## 쿠키 (Cookie)

### 쿠키란?
- 서버가 클라이언트에 저장하는 작은 데이터
- 최대 4KB 크기
- key=value 형식
- 매 요청마다 자동 전송

### 쿠키 설정
```http
# 서버 응답
HTTP/1.1 200 OK
Set-Cookie: sessionId=abc123; Path=/; HttpOnly
Set-Cookie: theme=dark; Max-Age=3600
Set-Cookie: language=ko; Expires=Wed, 09 Jun 2025 10:18:14 GMT

# 클라이언트 요청
GET /profile HTTP/1.1
Cookie: sessionId=abc123; theme=dark; language=ko
```

### 쿠키 속성
- **Domain**: 쿠키가 전송될 도메인
- **Path**: 쿠키가 전송될 경로
- **Expires/Max-Age**: 만료 시간
- **Secure**: HTTPS에서만 전송
- **HttpOnly**: JavaScript 접근 차단
- **SameSite**: CSRF 공격 방지

### 쿠키 종류
1. **Session Cookie**: 브라우저 종료 시 삭제
2. **Persistent Cookie**: 만료일까지 유지
3. **First-party Cookie**: 현재 도메인
4. **Third-party Cookie**: 다른 도메인

## 세션 (Session)

### 세션이란?
- 서버에 저장되는 사용자 정보
- 세션 ID로 식별
- 보안에 민감한 정보 저장 가능

### 세션 동작 과정
```
1. 클라이언트 → 로그인 요청
2. 서버 → 세션 생성, 세션 ID 발급
3. 서버 → Set-Cookie: sessionId=xyz789
4. 클라이언트 → Cookie: sessionId=xyz789
5. 서버 → 세션 ID로 사용자 식별
```

### 세션 저장소
- **메모리**: 빠르지만 재시작 시 손실
- **파일**: 영구 저장 가능
- **데이터베이스**: 확장 가능
- **Redis/Memcached**: 고성능 캐시

## 쿠키 vs 세션

| 구분 | 쿠키 | 세션 |
|------|------|------|
| 저장 위치 | 클라이언트 | 서버 |
| 보안 | 상대적 취약 | 상대적 안전 |
| 용량 | 4KB | 제한 없음 |
| 속도 | 빠름 | 서버 조회 필요 |
| 서버 부담 | 낮음 | 높음 |
| 확장성 | 좋음 | 세션 클러스터링 필요 |

## JWT (JSON Web Token)

### JWT 구조
```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### JWT 구성 요소
1. **Header**: 알고리즘, 토큰 타입
2. **Payload**: 실제 데이터 (Claims)
3. **Signature**: 검증용 서명

### JWT 특징
- **자체 포함(Self-contained)**: 필요한 모든 정보 포함
- **무상태(Stateless)**: 서버 저장소 불필요
- **확장 가능**: 마이크로서비스에 적합
- **표준화**: RFC 7519

## 토큰 기반 인증

### Access Token & Refresh Token
```
Access Token: 짧은 유효기간 (15분)
Refresh Token: 긴 유효기간 (30일)

1. 로그인 → Access + Refresh 토큰 발급
2. API 요청 시 Access Token 사용
3. Access Token 만료 → Refresh Token으로 재발급
4. Refresh Token 만료 → 다시 로그인
```

### 토큰 저장 위치
- **localStorage**: XSS 취약
- **sessionStorage**: 탭 닫으면 삭제
- **Cookie (httpOnly)**: XSS 방어, CSRF 취약
- **Memory + Refresh in Cookie**: 균형잡힌 접근

## 보안 고려사항

### 쿠키 보안
- **HttpOnly**: XSS 공격 방지
- **Secure**: HTTPS 전송만
- **SameSite**: CSRF 공격 방지
- **암호화**: 민감한 데이터 암호화

### 세션 보안
- **세션 고정 공격**: 로그인 시 세션 ID 재생성
- **세션 하이재킹**: HTTPS 사용
- **세션 타임아웃**: 자동 로그아웃

### JWT 보안
- **Secret Key 관리**: 안전한 키 저장
- **토큰 탈취**: HTTPS 필수
- **Payload 암호화**: 민감 정보 보호
- **토큰 무효화**: 블랙리스트 관리

## 실제 구현 예시

### TypeScript로 쿠키, 세션, JWT 구현

```typescript
import * as crypto from 'crypto';
import { EventEmitter } from 'events';

// ===== 쿠키 관리 =====
class CookieManager {
  // 쿠키 파싱
  static parse(cookieString: string): Map<string, string> {
    const cookies = new Map<string, string>();
    
    if (!cookieString) return cookies;
    
    cookieString.split(';').forEach(cookie => {
      const [key, value] = cookie.trim().split('=');
      if (key && value) {
        cookies.set(key, decodeURIComponent(value));
      }
    });
    
    return cookies;
  }
  
  // 쿠키 직렬화
  static serialize(
    name: string, 
    value: string, 
    options: CookieOptions = {}
  ): string {
    let cookie = `${name}=${encodeURIComponent(value)}`;
    
    if (options.maxAge) {
      cookie += `; Max-Age=${options.maxAge}`;
    }
    
    if (options.expires) {
      cookie += `; Expires=${options.expires.toUTCString()}`;
    }
    
    if (options.path) {
      cookie += `; Path=${options.path}`;
    }
    
    if (options.domain) {
      cookie += `; Domain=${options.domain}`;
    }
    
    if (options.secure) {
      cookie += '; Secure';
    }
    
    if (options.httpOnly) {
      cookie += '; HttpOnly';
    }
    
    if (options.sameSite) {
      cookie += `; SameSite=${options.sameSite}`;
    }
    
    return cookie;
  }
}

// 쿠키 옵션 인터페이스
interface CookieOptions {
  maxAge?: number;
  expires?: Date;
  path?: string;
  domain?: string;
  secure?: boolean;
  httpOnly?: boolean;
  sameSite?: 'Strict' | 'Lax' | 'None';
}

// ===== 세션 관리 =====
class SessionManager {
  private sessions: Map<string, SessionData>;
  private config: SessionConfig;
  
  constructor(config: SessionConfig = {}) {
    this.sessions = new Map();
    this.config = {
      maxAge: config.maxAge || 3600000, // 1시간
      rolling: config.rolling || false,
      secure: config.secure || false,
      httpOnly: config.httpOnly || true,
      sameSite: config.sameSite || 'Lax'
    };
    
    // 만료된 세션 정리
    setInterval(() => this.cleanup(), 60000); // 1분마다
  }
  
  // 세션 생성
  create(data: any = {}): string {
    const sessionId = this.generateSessionId();
    const session: SessionData = {
      id: sessionId,
      data: data,
      createdAt: Date.now(),
      lastAccessed: Date.now(),
      maxAge: this.config.maxAge!
    };
    
    this.sessions.set(sessionId, session);
    return sessionId;
  }
  
  // 세션 조회
  get(sessionId: string): SessionData | null {
    const session = this.sessions.get(sessionId);
    
    if (!session) return null;
    
    // 만료 확인
    if (this.isExpired(session)) {
      this.destroy(sessionId);
      return null;
    }
    
    // Rolling 옵션: 접근 시마다 만료시간 연장
    if (this.config.rolling) {
      session.lastAccessed = Date.now();
    }
    
    return session;
  }
  
  // 세션 업데이트
  update(sessionId: string, data: any): boolean {
    const session = this.get(sessionId);
    
    if (!session) return false;
    
    session.data = { ...session.data, ...data };
    session.lastAccessed = Date.now();
    
    return true;
  }
  
  // 세션 삭제
  destroy(sessionId: string): boolean {
    return this.sessions.delete(sessionId);
  }
  
  // 세션 ID 생성
  private generateSessionId(): string {
    return crypto.randomBytes(32).toString('hex');
  }
  
  // 만료 확인
  private isExpired(session: SessionData): boolean {
    return Date.now() - session.lastAccessed > session.maxAge;
  }
  
  // 만료된 세션 정리
  private cleanup(): void {
    for (const [id, session] of this.sessions) {
      if (this.isExpired(session)) {
        this.sessions.delete(id);
      }
    }
  }
}

// 세션 데이터 인터페이스
interface SessionData {
  id: string;
  data: any;
  createdAt: number;
  lastAccessed: number;
  maxAge: number;
}

// 세션 설정 인터페이스
interface SessionConfig {
  maxAge?: number;
  rolling?: boolean;
  secure?: boolean;
  httpOnly?: boolean;
  sameSite?: 'Strict' | 'Lax' | 'None';
}

// ===== JWT 관리 =====
class JWTManager {
  private secret: string;
  
  constructor(secret: string) {
    this.secret = secret;
  }
  
  // JWT 생성
  sign(payload: any, options: JWTOptions = {}): string {
    const header = {
      alg: 'HS256',
      typ: 'JWT'
    };
    
    const now = Math.floor(Date.now() / 1000);
    
    // 표준 클레임 추가
    const claims = {
      ...payload,
      iat: now,
      exp: options.expiresIn ? now + options.expiresIn : undefined,
      iss: options.issuer,
      sub: options.subject,
      aud: options.audience
    };
    
    // Base64 인코딩
    const encodedHeader = this.base64UrlEncode(JSON.stringify(header));
    const encodedPayload = this.base64UrlEncode(JSON.stringify(claims));
    
    // 서명 생성
    const signature = this.createSignature(encodedHeader, encodedPayload);
    
    return `${encodedHeader}.${encodedPayload}.${signature}`;
  }
  
  // JWT 검증
  verify(token: string): any {
    const parts = token.split('.');
    
    if (parts.length !== 3) {
      throw new Error('Invalid token format');
    }
    
    const [encodedHeader, encodedPayload, signature] = parts;
    
    // 서명 검증
    const expectedSignature = this.createSignature(encodedHeader, encodedPayload);
    
    if (signature !== expectedSignature) {
      throw new Error('Invalid signature');
    }
    
    // Payload 디코딩
    const payload = JSON.parse(this.base64UrlDecode(encodedPayload));
    
    // 만료 확인
    if (payload.exp && payload.exp < Math.floor(Date.now() / 1000)) {
      throw new Error('Token expired');
    }
    
    return payload;
  }
  
  // 서명 생성
  private createSignature(header: string, payload: string): string {
    const data = `${header}.${payload}`;
    const signature = crypto
      .createHmac('sha256', this.secret)
      .update(data)
      .digest('base64');
    
    return this.base64UrlEscape(signature);
  }
  
  // Base64 URL 인코딩
  private base64UrlEncode(str: string): string {
    const base64 = Buffer.from(str).toString('base64');
    return this.base64UrlEscape(base64);
  }
  
  // Base64 URL 디코딩
  private base64UrlDecode(str: string): string {
    const base64 = this.base64UrlUnescape(str);
    return Buffer.from(base64, 'base64').toString();
  }
  
  // Base64 URL 이스케이프
  private base64UrlEscape(str: string): string {
    return str.replace(/\+/g, '-').replace(/\//g, '_').replace(/=/g, '');
  }
  
  // Base64 URL 언이스케이프
  private base64UrlUnescape(str: string): string {
    str += new Array(5 - str.length % 4).join('=');
    return str.replace(/-/g, '+').replace(/_/g, '/');
  }
}

// JWT 옵션 인터페이스
interface JWTOptions {
  expiresIn?: number; // 초 단위
  issuer?: string;
  subject?: string;
  audience?: string;
}

// ===== 인증 미들웨어 =====
class AuthMiddleware {
  private sessionManager: SessionManager;
  private jwtManager: JWTManager;
  
  constructor(sessionManager: SessionManager, jwtManager: JWTManager) {
    this.sessionManager = sessionManager;
    this.jwtManager = jwtManager;
  }
  
  // 세션 기반 인증
  sessionAuth(req: any, res: any, next: Function): void {
    const cookies = CookieManager.parse(req.headers.cookie || '');
    const sessionId = cookies.get('sessionId');
    
    if (!sessionId) {
      res.statusCode = 401;
      res.end('Unauthorized');
      return;
    }
    
    const session = this.sessionManager.get(sessionId);
    
    if (!session) {
      res.statusCode = 401;
      res.end('Session expired');
      return;
    }
    
    req.session = session;
    next();
  }
  
  // JWT 기반 인증
  jwtAuth(req: any, res: any, next: Function): void {
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      res.statusCode = 401;
      res.end('No token provided');
      return;
    }
    
    const token = authHeader.substring(7);
    
    try {
      const payload = this.jwtManager.verify(token);
      req.user = payload;
      next();
    } catch (error: any) {
      res.statusCode = 401;
      res.end(error.message);
    }
  }
}

// ===== Refresh Token 관리 =====
class RefreshTokenManager {
  private refreshTokens: Map<string, RefreshTokenData>;
  private jwtManager: JWTManager;
  
  constructor(jwtManager: JWTManager) {
    this.refreshTokens = new Map();
    this.jwtManager = jwtManager;
  }
  
  // 토큰 쌍 생성
  async createTokenPair(userId: string): Promise<TokenPair> {
    // Access Token (15분)
    const accessToken = this.jwtManager.sign(
      { userId, type: 'access' },
      { expiresIn: 900 } // 15분
    );
    
    // Refresh Token (30일)
    const refreshToken = crypto.randomBytes(32).toString('hex');
    const refreshData: RefreshTokenData = {
      token: refreshToken,
      userId: userId,
      createdAt: Date.now(),
      expiresAt: Date.now() + 30 * 24 * 60 * 60 * 1000 // 30일
    };
    
    this.refreshTokens.set(refreshToken, refreshData);
    
    return {
      accessToken,
      refreshToken,
      expiresIn: 900
    };
  }
  
  // Access Token 갱신
  async refresh(refreshToken: string): Promise<string | null> {
    const tokenData = this.refreshTokens.get(refreshToken);
    
    if (!tokenData) return null;
    
    // 만료 확인
    if (Date.now() > tokenData.expiresAt) {
      this.refreshTokens.delete(refreshToken);
      return null;
    }
    
    // 새 Access Token 발급
    const accessToken = this.jwtManager.sign(
      { userId: tokenData.userId, type: 'access' },
      { expiresIn: 900 }
    );
    
    return accessToken;
  }
  
  // Refresh Token 폐기
  revoke(refreshToken: string): boolean {
    return this.refreshTokens.delete(refreshToken);
  }
  
  // 사용자의 모든 토큰 폐기
  revokeAll(userId: string): number {
    let count = 0;
    for (const [token, data] of this.refreshTokens) {
      if (data.userId === userId) {
        this.refreshTokens.delete(token);
        count++;
      }
    }
    return count;
  }
}

// 토큰 쌍 인터페이스
interface TokenPair {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
}

// Refresh Token 데이터
interface RefreshTokenData {
  token: string;
  userId: string;
  createdAt: number;
  expiresAt: number;
}

// ===== 보안 유틸리티 =====
class SecurityUtils {
  // CSRF 토큰 생성
  static generateCSRFToken(): string {
    return crypto.randomBytes(32).toString('hex');
  }
  
  // CSRF 토큰 검증
  static verifyCSRFToken(token: string, expectedToken: string): boolean {
    return crypto.timingSafeEqual(
      Buffer.from(token),
      Buffer.from(expectedToken)
    );
  }
  
  // Rate Limiting
  static createRateLimiter(
    maxRequests: number, 
    windowMs: number
  ): (identifier: string) => boolean {
    const requests = new Map<string, number[]>();
    
    return (identifier: string): boolean => {
      const now = Date.now();
      const userRequests = requests.get(identifier) || [];
      
      // 윈도우 밖의 요청 제거
      const validRequests = userRequests.filter(
        time => now - time < windowMs
      );
      
      if (validRequests.length >= maxRequests) {
        return false; // 제한 초과
      }
      
      validRequests.push(now);
      requests.set(identifier, validRequests);
      return true;
    };
  }
}

// ===== 사용 예제 =====
class AuthenticationDemo {
  static demonstrateCookie(): void {
    console.log('=== Cookie Demo ===\n');
    
    // 쿠키 생성
    const cookie = CookieManager.serialize('user', 'alice', {
      maxAge: 3600,
      httpOnly: true,
      secure: true,
      sameSite: 'Strict'
    });
    
    console.log('Set-Cookie:', cookie);
    
    // 쿠키 파싱
    const cookies = CookieManager.parse('user=alice; theme=dark; lang=ko');
    console.log('Parsed cookies:', Object.fromEntries(cookies));
  }
  
  static demonstrateSession(): void {
    console.log('\n=== Session Demo ===\n');
    
    const sessionManager = new SessionManager({
      maxAge: 1800000, // 30분
      rolling: true
    });
    
    // 세션 생성
    const sessionId = sessionManager.create({
      userId: 'user123',
      username: 'alice',
      role: 'admin'
    });
    
    console.log('Session created:', sessionId);
    
    // 세션 조회
    const session = sessionManager.get(sessionId);
    console.log('Session data:', session?.data);
    
    // 세션 업데이트
    sessionManager.update(sessionId, { lastPage: '/dashboard' });
    console.log('Session updated');
    
    // 세션 삭제
    sessionManager.destroy(sessionId);
    console.log('Session destroyed');
  }
  
  static demonstrateJWT(): void {
    console.log('\n=== JWT Demo ===\n');
    
    const jwtManager = new JWTManager('my-secret-key-2024');
    
    // JWT 생성
    const token = jwtManager.sign(
      {
        userId: 'user123',
        username: 'alice',
        role: 'admin'
      },
      {
        expiresIn: 3600, // 1시간
        issuer: 'myapp.com'
      }
    );
    
    console.log('JWT Token:');
    const parts = token.split('.');
    console.log('  Header:', parts[0]);
    console.log('  Payload:', parts[1]);
    console.log('  Signature:', parts[2]);
    
    // JWT 검증
    try {
      const decoded = jwtManager.verify(token);
      console.log('\nDecoded payload:', decoded);
    } catch (error: any) {
      console.error('Verification failed:', error.message);
    }
  }
  
  static async demonstrateRefreshToken(): Promise<void> {
    console.log('\n=== Refresh Token Demo ===\n');
    
    const jwtManager = new JWTManager('my-secret-key-2024');
    const refreshManager = new RefreshTokenManager(jwtManager);
    
    // 토큰 쌍 생성
    const tokens = await refreshManager.createTokenPair('user123');
    console.log('Token pair created:');
    console.log('  Access Token:', tokens.accessToken.substring(0, 20) + '...');
    console.log('  Refresh Token:', tokens.refreshToken);
    console.log('  Expires in:', tokens.expiresIn, 'seconds');
    
    // Access Token 갱신
    await new Promise(resolve => setTimeout(resolve, 1000));
    const newAccessToken = await refreshManager.refresh(tokens.refreshToken);
    console.log('\nNew Access Token:', newAccessToken?.substring(0, 20) + '...');
    
    // 토큰 폐기
    refreshManager.revoke(tokens.refreshToken);
    console.log('Refresh token revoked');
  }
  
  static demonstrateSecurity(): void {
    console.log('\n=== Security Features Demo ===\n');
    
    // CSRF 토큰
    const csrfToken = SecurityUtils.generateCSRFToken();
    console.log('CSRF Token:', csrfToken);
    
    const isValid = SecurityUtils.verifyCSRFToken(csrfToken, csrfToken);
    console.log('CSRF Valid:', isValid);
    
    // Rate Limiting
    const rateLimiter = SecurityUtils.createRateLimiter(5, 60000); // 5 requests per minute
    
    console.log('\nRate Limiting (5 requests/minute):');
    for (let i = 1; i <= 7; i++) {
      const allowed = rateLimiter('user123');
      console.log(`  Request ${i}: ${allowed ? 'Allowed' : 'Blocked'}`);
    }
  }
}

// 실행 함수
async function demonstrateCookieAndSession() {
  console.log('=== Cookie, Session, and JWT Demonstration ===\n');
  
  AuthenticationDemo.demonstrateCookie();
  AuthenticationDemo.demonstrateSession();
  AuthenticationDemo.demonstrateJWT();
  await AuthenticationDemo.demonstrateRefreshToken();
  AuthenticationDemo.demonstrateSecurity();
}

// 실행
// demonstrateCookieAndSession().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **쿠키**: 클라이언트 저장, 4KB 제한, 자동 전송
> 2. **세션**: 서버 저장, 세션 ID로 식별, 보안 우수
> 3. **JWT**: 자체 포함 토큰, 무상태, 확장 가능
> 4. **Refresh Token**: Access Token 갱신용 장기 토큰
> 5. **보안**: HttpOnly, Secure, SameSite, CSRF 토큰

## 참고 자료

### 📖 추가 학습
- [HTTP State Management - RFC 6265](https://tools.ietf.org/html/rfc6265)
- [JWT - RFC 7519](https://tools.ietf.org/html/rfc7519)
- [OWASP Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [MDN - HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

### 🔗 관련 문서
- [[12_http-protocol|HTTP 프로토콜]]
- [[13_https-tls|HTTPS와 TLS]]
- [[18_api-rest-graphql|API와 REST/GraphQL]]
