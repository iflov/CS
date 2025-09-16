---
title: HTTP 프로토콜
created: 2025-01-03
updated: 2025-01-03
tags:
  - CS
  - Network
  - HTTP
  - Protocol
  - REST
  - API
category: Network
status: 완료
---

# 🌐 HTTP 프로토콜 (HyperText Transfer Protocol)

## 📌 개요

HTTP(HyperText Transfer Protocol)는 웹의 기초가 되는 애플리케이션 계층 프로토콜입니다. 클라이언트-서버 모델을 기반으로 하며, 요청-응답 구조를 통해 하이퍼텍스트와 다양한 리소스를 전송합니다.

> [!info] 핵심 개념
> - **Stateless**: 각 요청은 독립적
> - **Request-Response**: 클라이언트 요청, 서버 응답
> - **메서드**: GET, POST, PUT, DELETE 등
> - **상태 코드**: 200, 404, 500 등

## 📚 목차

1. [HTTP 기본 구조](#http-기본-구조)
2. [HTTP 메서드](#http-메서드)
3. [HTTP 헤더](#http-헤더)
4. [HTTP 상태 코드](#http-상태-코드)
5. [HTTP 바디와 콘텐츠 타입](#http-바디와-콘텐츠-타입)
6. [REST API](#rest-api)
7. [HTTP 성능 최적화](#http-성능-최적화)
8. [실제 구현 예시](#실제-구현-예시)
9. [참고 자료](#참고-자료)

## HTTP 기본 구조

### HTTP 메시지 구조
```
┌─────────────┐     Request      ┌─────────────┐
│   Client    │ ───────────────> │   Server    │
│  (Browser)  │                  │ (Web Server)│
└─────────────┘ <─────────────── └─────────────┘
                    Response
```

### Request 구조
```http
POST /api/users HTTP/1.1          ← Start Line
Host: api.example.com              ← Headers
Content-Type: application/json
Authorization: Bearer token123

{"name": "John", "age": 30}       ← Body
```

### Response 구조
```http
HTTP/1.1 200 OK                   ← Status Line
Content-Type: application/json     ← Headers
Cache-Control: no-cache

{"id": 1, "status": "created"}    ← Body
```

### HTTP 특징
1. **Stateless**: 서버는 이전 요청을 기억하지 않음
2. **Connectionless**: 요청-응답 후 연결 종료 (HTTP/1.0)
3. **ASCII 기반**: 사람이 읽을 수 있는 텍스트 형식
4. **유연성**: 다양한 데이터 타입 전송 가능

## HTTP 메서드

### 주요 메서드

| 메서드 | 설명 | 멱등성 | 안전성 | 캐시 가능 |
|--------|------|--------|--------|-----------|
| **GET** | 리소스 조회 | O | O | O |
| **POST** | 리소스 생성 | X | X | △ |
| **PUT** | 리소스 전체 수정 | O | X | X |
| **PATCH** | 리소스 부분 수정 | X | X | X |
| **DELETE** | 리소스 삭제 | O | X | X |
| **HEAD** | 헤더만 조회 | O | O | O |
| **OPTIONS** | 허용 메서드 조회 | O | O | X |
| **CONNECT** | 프록시 터널링 | X | X | X |
| **TRACE** | 경로 추적 | O | O | X |

### RESTful API 설계
```
GET    /users       # 모든 사용자 조회
GET    /users/123   # 특정 사용자 조회
POST   /users       # 새 사용자 생성
PUT    /users/123   # 사용자 전체 수정
PATCH  /users/123   # 사용자 부분 수정
DELETE /users/123   # 사용자 삭제
```

## HTTP 헤더

### 일반 헤더
- **Date**: 메시지 생성 날짜
- **Connection**: 연결 관리 (keep-alive, close)
- **Cache-Control**: 캐싱 정책
- **Transfer-Encoding**: 메시지 인코딩 방식

### 요청 헤더
- **Host**: 요청 호스트 (필수)
- **User-Agent**: 클라이언트 정보
- **Accept**: 수신 가능한 미디어 타입
- **Accept-Language**: 선호 언어
- **Accept-Encoding**: 수신 가능한 인코딩
- **Authorization**: 인증 토큰
- **Cookie**: 쿠키 데이터
- **Referer**: 이전 페이지 URL

### 응답 헤더
- **Server**: 서버 소프트웨어 정보
- **Set-Cookie**: 쿠키 설정
- **Location**: 리다이렉트 위치
- **ETag**: 리소스 버전 식별
- **Access-Control-Allow-Origin**: CORS 정책

### 엔티티 헤더
- **Content-Type**: 바디 미디어 타입
- **Content-Length**: 바디 길이
- **Content-Encoding**: 바디 인코딩
- **Content-Language**: 바디 언어
- **Last-Modified**: 마지막 수정 시간

## HTTP 상태 코드

### 1xx - 정보성
- **100 Continue**: 계속 진행
- **101 Switching Protocols**: 프로토콜 전환
- **102 Processing**: 처리 중

### 2xx - 성공
- **200 OK**: 정상 처리
- **201 Created**: 리소스 생성됨
- **202 Accepted**: 요청 수락됨
- **204 No Content**: 콘텐츠 없음
- **206 Partial Content**: 부분 콘텐츠

### 3xx - 리다이렉션
- **301 Moved Permanently**: 영구 이동
- **302 Found**: 임시 이동
- **303 See Other**: 다른 URI 참조
- **304 Not Modified**: 수정되지 않음
- **307 Temporary Redirect**: 임시 리다이렉트
- **308 Permanent Redirect**: 영구 리다이렉트

### 4xx - 클라이언트 오류
- **400 Bad Request**: 잘못된 요청
- **401 Unauthorized**: 인증 필요
- **403 Forbidden**: 접근 금지
- **404 Not Found**: 리소스 없음
- **405 Method Not Allowed**: 허용되지 않은 메서드
- **408 Request Timeout**: 요청 시간 초과
- **409 Conflict**: 충돌
- **429 Too Many Requests**: 요청 과다

### 5xx - 서버 오류
- **500 Internal Server Error**: 내부 서버 오류
- **501 Not Implemented**: 구현되지 않음
- **502 Bad Gateway**: 잘못된 게이트웨이
- **503 Service Unavailable**: 서비스 이용 불가
- **504 Gateway Timeout**: 게이트웨이 시간 초과

## HTTP 바디와 콘텐츠 타입

### MIME 타입
```
Content-Type: type/subtype[;parameter]

예시:
- text/html; charset=UTF-8
- application/json
- image/png
- multipart/form-data; boundary=----
```

### 주요 Content-Type
- **text/plain**: 일반 텍스트
- **text/html**: HTML 문서
- **application/json**: JSON 데이터
- **application/xml**: XML 데이터
- **application/x-www-form-urlencoded**: 폼 데이터
- **multipart/form-data**: 파일 업로드
- **image/jpeg, image/png**: 이미지
- **video/mp4**: 비디오
- **application/octet-stream**: 바이너리 데이터

## REST API

### REST 원칙
1. **Client-Server**: 클라이언트와 서버 분리
2. **Stateless**: 무상태성
3. **Cacheable**: 캐시 가능
4. **Uniform Interface**: 일관된 인터페이스
5. **Layered System**: 계층화된 시스템
6. **Code on Demand**: 코드 온 디맨드 (선택적)

### REST API 설계 모범 사례
```
# 명사 사용 (동사 X)
GET /users         ✓
GET /getUsers      ✗

# 복수형 사용
/users/123         ✓
/user/123          ✗

# 계층 관계 표현
/users/123/posts   ✓

# 필터링
/users?status=active&sort=name

# 버전 관리
/api/v1/users
```

## HTTP 성능 최적화

### Keep-Alive (Persistent Connection)
```
Connection: keep-alive
Keep-Alive: timeout=5, max=100
```

### 캐싱
```
Cache-Control: public, max-age=3600
ETag: "abc123"
If-None-Match: "abc123"
```

### 압축
```
Accept-Encoding: gzip, deflate
Content-Encoding: gzip
```

### Pipelining
- 응답을 기다리지 않고 여러 요청 전송
- HTTP/1.1에서 지원하지만 문제가 많아 잘 사용 안 함

## 실제 구현 예시

### TypeScript로 HTTP 서버와 클라이언트 구현

```typescript
import * as http from 'http';
import * as https from 'https';
import { URL } from 'url';
import { EventEmitter } from 'events';

// HTTP 메시지 인터페이스
interface HTTPMessage {
  method?: string;
  url?: string;
  version: string;
  headers: Map<string, string>;
  body: Buffer | string;
}

// HTTP 파서 클래스
class HTTPParser {
  // HTTP 메시지 파싱
  static parseRequest(data: string): HTTPMessage {
    const lines = data.split('\r\n');
    const [method, url, version] = lines[0].split(' ');
    
    const headers = new Map<string, string>();
    let bodyStart = 0;
    
    for (let i = 1; i < lines.length; i++) {
      if (lines[i] === '') {
        bodyStart = i + 1;
        break;
      }
      const [key, ...valueParts] = lines[i].split(':');
      headers.set(key.toLowerCase(), valueParts.join(':').trim());
    }
    
    const body = lines.slice(bodyStart).join('\r\n');
    
    return {
      method,
      url,
      version,
      headers,
      body
    };
  }
  
  // HTTP 응답 파싱
  static parseResponse(data: string): HTTPMessage {
    const lines = data.split('\r\n');
    const [version, statusCode, ...statusText] = lines[0].split(' ');
    
    const headers = new Map<string, string>();
    let bodyStart = 0;
    
    for (let i = 1; i < lines.length; i++) {
      if (lines[i] === '') {
        bodyStart = i + 1;
        break;
      }
      const [key, ...valueParts] = lines[i].split(':');
      headers.set(key.toLowerCase(), valueParts.join(':').trim());
    }
    
    const body = lines.slice(bodyStart).join('\r\n');
    
    return {
      version,
      headers,
      body
    };
  }
  
  // 요청 문자열 생성
  static buildRequest(
    method: string,
    url: string,
    headers: Map<string, string>,
    body?: string
  ): string {
    let request = `${method} ${url} HTTP/1.1\r\n`;
    
    headers.forEach((value, key) => {
      request += `${key}: ${value}\r\n`;
    });
    
    request += '\r\n';
    
    if (body) {
      request += body;
    }
    
    return request;
  }
  
  // 응답 문자열 생성
  static buildResponse(
    statusCode: number,
    statusText: string,
    headers: Map<string, string>,
    body?: string
  ): string {
    let response = `HTTP/1.1 ${statusCode} ${statusText}\r\n`;
    
    headers.forEach((value, key) => {
      response += `${key}: ${value}\r\n`;
    });
    
    response += '\r\n';
    
    if (body) {
      response += body;
    }
    
    return response;
  }
}

// HTTP 서버 구현
class SimpleHTTPServer extends EventEmitter {
  private server: http.Server;
  private routes: Map<string, Map<string, Function>>;
  private middlewares: Function[];
  
  constructor() {
    super();
    this.routes = new Map();
    this.middlewares = [];
    this.server = http.createServer(this.handleRequest.bind(this));
  }
  
  // 미들웨어 추가
  use(middleware: Function): void {
    this.middlewares.push(middleware);
  }
  
  // 라우트 등록
  route(method: string, path: string, handler: Function): void {
    if (!this.routes.has(method)) {
      this.routes.set(method, new Map());
    }
    this.routes.get(method)!.set(path, handler);
  }
  
  // 편의 메서드
  get(path: string, handler: Function): void {
    this.route('GET', path, handler);
  }
  
  post(path: string, handler: Function): void {
    this.route('POST', path, handler);
  }
  
  put(path: string, handler: Function): void {
    this.route('PUT', path, handler);
  }
  
  delete(path: string, handler: Function): void {
    this.route('DELETE', path, handler);
  }
  
  // 요청 처리
  private async handleRequest(
    req: http.IncomingMessage,
    res: http.ServerResponse
  ): Promise<void> {
    console.log(`${req.method} ${req.url}`);
    
    // 요청 바디 읽기
    const body = await this.readBody(req);
    
    // 요청 객체 확장
    const request = {
      method: req.method!,
      url: req.url!,
      headers: req.headers,
      body: body,
      params: {} as any,
      query: this.parseQuery(req.url!)
    };
    
    // 응답 객체 확장
    const response = {
      statusCode: 200,
      headers: new Map<string, string>(),
      
      status: function(code: number) {
        res.statusCode = code;
        return this;
      },
      
      set: function(key: string, value: string) {
        res.setHeader(key, value);
        return this;
      },
      
      json: function(data: any) {
        res.setHeader('Content-Type', 'application/json');
        res.end(JSON.stringify(data));
      },
      
      send: function(data: string) {
        res.end(data);
      }
    };
    
    // 미들웨어 실행
    for (const middleware of this.middlewares) {
      await middleware(request, response, () => {});
    }
    
    // 라우트 매칭
    const handler = this.findRoute(request.method, request.url);
    
    if (handler) {
      await handler(request, response);
    } else {
      res.statusCode = 404;
      res.end('Not Found');
    }
  }
  
  // 바디 읽기
  private readBody(req: http.IncomingMessage): Promise<string> {
    return new Promise((resolve) => {
      let body = '';
      req.on('data', chunk => body += chunk.toString());
      req.on('end', () => resolve(body));
    });
  }
  
  // 쿼리 파라미터 파싱
  private parseQuery(url: string): any {
    const urlObj = new URL(url, 'http://localhost');
    const query: any = {};
    
    urlObj.searchParams.forEach((value, key) => {
      query[key] = value;
    });
    
    return query;
  }
  
  // 라우트 찾기
  private findRoute(method: string, url: string): Function | null {
    const methodRoutes = this.routes.get(method);
    if (!methodRoutes) return null;
    
    const path = url.split('?')[0];
    
    // 정확한 매칭
    if (methodRoutes.has(path)) {
      return methodRoutes.get(path)!;
    }
    
    // 패턴 매칭 (간단한 구현)
    for (const [pattern, handler] of methodRoutes) {
      if (this.matchPath(pattern, path)) {
        return handler;
      }
    }
    
    return null;
  }
  
  // 경로 매칭
  private matchPath(pattern: string, path: string): boolean {
    // 간단한 와일드카드 매칭
    const regex = new RegExp('^' + pattern.replace(/\*/g, '.*') + '$');
    return regex.test(path);
  }
  
  // 서버 시작
  listen(port: number, callback?: () => void): void {
    this.server.listen(port, () => {
      console.log(`HTTP Server listening on port ${port}`);
      if (callback) callback();
    });
  }
  
  // 서버 종료
  close(): void {
    this.server.close();
  }
}

// HTTP 클라이언트 구현
class SimpleHTTPClient {
  private defaultHeaders: Map<string, string>;
  
  constructor() {
    this.defaultHeaders = new Map([
      ['User-Agent', 'SimpleHTTPClient/1.0'],
      ['Accept', '*/*']
    ]);
  }
  
  // GET 요청
  async get(url: string, headers?: any): Promise<any> {
    return this.request('GET', url, { headers });
  }
  
  // POST 요청
  async post(url: string, data?: any, headers?: any): Promise<any> {
    return this.request('POST', url, { data, headers });
  }
  
  // PUT 요청
  async put(url: string, data?: any, headers?: any): Promise<any> {
    return this.request('PUT', url, { data, headers });
  }
  
  // DELETE 요청
  async delete(url: string, headers?: any): Promise<any> {
    return this.request('DELETE', url, { headers });
  }
  
  // 범용 요청 메서드
  private async request(
    method: string,
    url: string,
    options: {
      data?: any,
      headers?: any
    } = {}
  ): Promise<any> {
    return new Promise((resolve, reject) => {
      const urlObj = new URL(url);
      const isHttps = urlObj.protocol === 'https:';
      const client = isHttps ? https : http;
      
      const reqOptions = {
        hostname: urlObj.hostname,
        port: urlObj.port || (isHttps ? 443 : 80),
        path: urlObj.pathname + urlObj.search,
        method: method,
        headers: {
          ...Object.fromEntries(this.defaultHeaders),
          ...options.headers
        }
      };
      
      // Content-Type 설정
      if (options.data) {
        if (typeof options.data === 'object') {
          reqOptions.headers['Content-Type'] = 'application/json';
        } else {
          reqOptions.headers['Content-Type'] = 'text/plain';
        }
      }
      
      const req = client.request(reqOptions, (res) => {
        let data = '';
        
        res.on('data', chunk => data += chunk);
        res.on('end', () => {
          const response = {
            status: res.statusCode,
            statusText: res.statusMessage,
            headers: res.headers,
            data: this.parseBody(data, res.headers['content-type'])
          };
          
          if (res.statusCode! >= 200 && res.statusCode! < 300) {
            resolve(response);
          } else {
            reject(response);
          }
        });
      });
      
      req.on('error', reject);
      
      // 바디 전송
      if (options.data) {
        const body = typeof options.data === 'object' 
          ? JSON.stringify(options.data)
          : options.data;
        req.write(body);
      }
      
      req.end();
    });
  }
  
  // 응답 바디 파싱
  private parseBody(data: string, contentType?: string): any {
    if (!contentType) return data;
    
    if (contentType.includes('application/json')) {
      try {
        return JSON.parse(data);
      } catch {
        return data;
      }
    }
    
    return data;
  }
}

// REST API 서버 예제
class RESTAPIServer {
  private server: SimpleHTTPServer;
  private db: Map<string, any>;
  private idCounter: number;
  
  constructor() {
    this.server = new SimpleHTTPServer();
    this.db = new Map();
    this.idCounter = 1;
    this.setupMiddlewares();
    this.setupRoutes();
  }
  
  // 미들웨어 설정
  private setupMiddlewares(): void {
    // CORS 미들웨어
    this.server.use((req: any, res: any, next: Function) => {
      res.set('Access-Control-Allow-Origin', '*');
      res.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
      res.set('Access-Control-Allow-Headers', 'Content-Type, Authorization');
      next();
    });
    
    // JSON 파싱 미들웨어
    this.server.use((req: any, res: any, next: Function) => {
      if (req.headers['content-type'] === 'application/json' && req.body) {
        try {
          req.body = JSON.parse(req.body);
        } catch (e) {
          res.status(400).json({ error: 'Invalid JSON' });
          return;
        }
      }
      next();
    });
    
    // 로깅 미들웨어
    this.server.use((req: any, res: any, next: Function) => {
      console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
      next();
    });
  }
  
  // 라우트 설정
  private setupRoutes(): void {
    // 모든 리소스 조회
    this.server.get('/api/users', (req: any, res: any) => {
      const users = Array.from(this.db.values());
      res.json(users);
    });
    
    // 특정 리소스 조회
    this.server.get('/api/users/*', (req: any, res: any) => {
      const id = req.url.split('/').pop();
      const user = this.db.get(id);
      
      if (user) {
        res.json(user);
      } else {
        res.status(404).json({ error: 'User not found' });
      }
    });
    
    // 리소스 생성
    this.server.post('/api/users', (req: any, res: any) => {
      const user = {
        id: this.idCounter++,
        ...req.body,
        createdAt: new Date().toISOString()
      };
      
      this.db.set(user.id.toString(), user);
      res.status(201).json(user);
    });
    
    // 리소스 수정
    this.server.put('/api/users/*', (req: any, res: any) => {
      const id = req.url.split('/').pop();
      const user = this.db.get(id);
      
      if (user) {
        const updated = {
          ...user,
          ...req.body,
          id: user.id,
          updatedAt: new Date().toISOString()
        };
        this.db.set(id, updated);
        res.json(updated);
      } else {
        res.status(404).json({ error: 'User not found' });
      }
    });
    
    // 리소스 삭제
    this.server.delete('/api/users/*', (req: any, res: any) => {
      const id = req.url.split('/').pop();
      
      if (this.db.has(id)) {
        this.db.delete(id);
        res.status(204).send('');
      } else {
        res.status(404).json({ error: 'User not found' });
      }
    });
  }
  
  // 서버 시작
  start(port: number): void {
    this.server.listen(port, () => {
      console.log(`REST API Server running on port ${port}`);
    });
  }
}

// 사용 예제
async function demonstrateHTTP() {
  console.log('=== HTTP Protocol Demonstration ===\n');
  
  // REST API 서버 시작
  const apiServer = new RESTAPIServer();
  apiServer.start(3000);
  
  // 클라이언트 생성
  const client = new SimpleHTTPClient();
  
  // API 테스트
  setTimeout(async () => {
    try {
      // POST - 사용자 생성
      console.log('Creating users...');
      const user1 = await client.post('http://localhost:3000/api/users', {
        name: 'Alice',
        email: 'alice@example.com'
      });
      console.log('Created:', user1.data);
      
      const user2 = await client.post('http://localhost:3000/api/users', {
        name: 'Bob',
        email: 'bob@example.com'
      });
      console.log('Created:', user2.data);
      
      // GET - 모든 사용자 조회
      console.log('\nFetching all users...');
      const allUsers = await client.get('http://localhost:3000/api/users');
      console.log('All users:', allUsers.data);
      
      // GET - 특정 사용자 조회
      console.log('\nFetching user 1...');
      const user = await client.get('http://localhost:3000/api/users/1');
      console.log('User 1:', user.data);
      
      // PUT - 사용자 수정
      console.log('\nUpdating user 1...');
      const updated = await client.put('http://localhost:3000/api/users/1', {
        name: 'Alice Smith'
      });
      console.log('Updated:', updated.data);
      
      // DELETE - 사용자 삭제
      console.log('\nDeleting user 2...');
      await client.delete('http://localhost:3000/api/users/2');
      console.log('Deleted user 2');
      
      // GET - 최종 확인
      console.log('\nFinal users list:');
      const finalUsers = await client.get('http://localhost:3000/api/users');
      console.log('Users:', finalUsers.data);
      
    } catch (error: any) {
      console.error('Error:', error.data || error.message);
    }
    
    process.exit(0);
  }, 1000);
}

// 실행
// demonstrateHTTP().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **Stateless**: HTTP는 상태를 유지하지 않음
> 2. **Request-Response**: 클라이언트 요청, 서버 응답 구조
> 3. **메서드**: GET(조회), POST(생성), PUT(수정), DELETE(삭제)
> 4. **상태 코드**: 2xx(성공), 3xx(리다이렉션), 4xx(클라이언트 오류), 5xx(서버 오류)
> 5. **REST**: 리소스 중심의 API 설계 원칙

## 참고 자료

### 📖 추가 학습
- [HTTP - MDN](https://developer.mozilla.org/ko/docs/Web/HTTP)
- [RFC 7230 - HTTP/1.1](https://tools.ietf.org/html/rfc7230)
- [REST API Tutorial](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)

### 🔗 관련 문서
- [[13_https-tls|HTTPS와 TLS]]
- [[14_http2-optimization|HTTP/2와 최적화]]
- [[15_cookie-session|쿠키와 세션]]
