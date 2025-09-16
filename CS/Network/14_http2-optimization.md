---
title: HTTP/2와 최적화
created: 2025-01-03
updated: 2025-01-03
tags:
  - CS
  - Network
  - HTTP2
  - Performance
  - Optimization
  - Multiplexing
category: Network
status: 완료
---

# ⚡ HTTP/2와 최적화 (HTTP/2 and Optimization)

## 📌 개요

HTTP/2는 2015년에 표준화된 HTTP/1.1의 후속 버전으로, 웹 페이지 로딩 속도를 크게 개선했습니다. 바이너리 프로토콜, 멀티플렉싱, 서버 푸시 등의 기능으로 성능을 향상시켰습니다.

> [!info] 핵심 개념
> - **멀티플렉싱**: 단일 연결로 다중 스트림
> - **서버 푸시**: 요청 전 리소스 전송
> - **헤더 압축**: HPACK 압축
> - **바이너리 프로토콜**: 효율적 파싱

## 📚 목차

1. [HTTP/1.x의 한계](#http1x의-한계)
2. [HTTP/2의 주요 기능](#http2의-주요-기능)
3. [멀티플렉싱](#멀티플렉싱)
4. [서버 푸시](#서버-푸시)
5. [헤더 압축 (HPACK)](#헤더-압축-hpack)
6. [스트림 우선순위](#스트림-우선순위)
7. [HTTP/1.x 최적화 기법](#http1x-최적화-기법)
8. [HTTP/2 최적화 전략](#http2-최적화-전략)
9. [실제 구현 예시](#실제-구현-예시)
10. [참고 자료](#참고-자료)

## HTTP/1.x의 한계

### 1. Head-of-Line Blocking
```
요청1 ────────────────→ 응답1 대기...
요청2 (블로킹됨)
요청3 (블로킹됨)
```

### 2. 연결당 하나의 요청
- 동시 요청 시 다중 TCP 연결 필요
- 연결 생성 오버헤드
- 브라우저 연결 제한 (보통 6개)

### 3. 헤더 중복
```http
GET /style.css HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0...
Accept: text/css
Cookie: session=abc123...  ← 매번 전송

GET /script.js HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0...  ← 똑같은 헤더 반복
Accept: application/javascript
Cookie: session=abc123...  ← 또 전송
```

## HTTP/2의 주요 기능

### 바이너리 프레이밍
```
HTTP/1.x (텍스트):
GET /index.html HTTP/1.1\r\n
Host: example.com\r\n
\r\n

HTTP/2 (바이너리 프레임):
+-----------------------------------------------+
|                Length (24)                   |
+---------------+---------------+---------------+
|   Type (8)    |   Flags (8)  |
+-+-------------+---------------+-------------------------------+
|R|                 Stream Identifier (31)                     |
+=+=============================================================+
|                   Frame Payload (0...)                      ...
+---------------------------------------------------------------+
```

### 프레임 타입
- **DATA**: 메시지 바디
- **HEADERS**: 헤더 정보
- **PRIORITY**: 스트림 우선순위
- **RST_STREAM**: 스트림 종료
- **SETTINGS**: 연결 설정
- **PUSH_PROMISE**: 서버 푸시 예고
- **PING**: 연결 확인
- **GOAWAY**: 연결 종료 예고
- **WINDOW_UPDATE**: 흐름 제어
- **CONTINUATION**: 헤더 연속

## 멀티플렉싱

### 개념
```
HTTP/1.1:
연결1: [요청A] ──────→ [응답A]
연결2: [요청B] ──────→ [응답B]
연결3: [요청C] ──────→ [응답C]

HTTP/2:
연결1: [A][B][C] ──→ [B][A][C] (인터리빙)
```

### 스트림과 메시지
```
Connection (TCP)
    ├── Stream 1 (HTML)
    │   ├── HEADERS frame
    │   └── DATA frames
    ├── Stream 3 (CSS)
    │   ├── HEADERS frame
    │   └── DATA frames
    └── Stream 5 (JS)
        ├── HEADERS frame
        └── DATA frames
```

## 서버 푸시

### 동작 방식
```
클라이언트: GET /index.html
서버: 
  1. index.html 응답
  2. PUSH_PROMISE: /style.css
  3. PUSH_PROMISE: /script.js
  4. style.css 전송 (요청 전에!)
  5. script.js 전송 (요청 전에!)
```

### 푸시 캐시
- 브라우저가 거부 가능 (RST_STREAM)
- 캐시된 리소스 재사용
- 동일 출처 정책 적용

## 헤더 압축 (HPACK)

### HPACK 알고리즘
1. **정적 테이블**: 자주 사용되는 헤더 (61개)
2. **동적 테이블**: 연결별 헤더 저장
3. **허프만 인코딩**: 문자열 압축

### 압축 예시
```
첫 요청:
:method: GET (정적 테이블 #2)
:path: /index.html (허프만 인코딩)
:scheme: https (정적 테이블 #7)
user-agent: Chrome/90 (동적 테이블 저장)

두 번째 요청:
:method: GET (인덱스 2)
:path: /style.css (허프만 인코딩)
:scheme: https (인덱스 7)
user-agent: (인덱스 62 - 이미 저장됨)
```

## 스트림 우선순위

### 우선순위 트리
```
          stream 0 (root)
              │
          stream 1 (HTML)
          weight: 256
              │
    ┌─────────┴─────────┐
stream 3 (CSS)      stream 5 (JS)
weight: 128         weight: 128
    │
stream 7 (Image)
weight: 32
```

### 가중치와 의존성
- **의존성**: 부모 스트림 완료 후 처리
- **가중치**: 1-256, 리소스 할당 비율

## HTTP/1.x 최적화 기법

### 1. 도메인 샤딩
```javascript
// 여러 도메인으로 병렬 다운로드
<img src="//cdn1.example.com/img1.jpg">
<img src="//cdn2.example.com/img2.jpg">
<img src="//cdn3.example.com/img3.jpg">
```

### 2. 스프라이팅
```css
/* 여러 이미지를 하나로 합침 */
.icon-home { 
  background: url(sprite.png) 0 0; 
}
.icon-user { 
  background: url(sprite.png) -20px 0; 
}
```

### 3. 번들링
```javascript
// 여러 JS/CSS 파일을 하나로
// bundle.js = file1.js + file2.js + file3.js
<script src="bundle.min.js"></script>
```

### 4. 인라이닝
```html
<!-- 작은 리소스는 직접 포함 -->
<style>
  body { margin: 0; padding: 0; }
</style>
```

## HTTP/2 최적화 전략

### HTTP/2에서 변경된 최적화
1. **도메인 샤딩 불필요**: 단일 연결로 충분
2. **번들링 재고려**: 캐싱 효율 vs 압축
3. **스프라이팅 불필요**: 멀티플렉싱으로 해결
4. **서버 푸시 활용**: 중요 리소스 먼저 전송

### 새로운 최적화 기법
```javascript
// 1. 리소스 힌트
<link rel="preconnect" href="https://api.example.com">
<link rel="dns-prefetch" href="https://cdn.example.com">
<link rel="preload" href="/font.woff2" as="font">
<link rel="prefetch" href="/next-page.html">

// 2. 우선순위 힌트
<img src="hero.jpg" importance="high">
<img src="footer.jpg" importance="low">

// 3. 조건부 푸시
Link: </style.css>; rel=preload; as=style; nopush
```

## 실제 구현 예시

### TypeScript로 HTTP/2 서버 구현

```typescript
import * as http2 from 'http2';
import * as fs from 'fs';
import * as path from 'path';
import { EventEmitter } from 'events';

// HTTP/2 프레임 타입
enum FrameType {
  DATA = 0x0,
  HEADERS = 0x1,
  PRIORITY = 0x2,
  RST_STREAM = 0x3,
  SETTINGS = 0x4,
  PUSH_PROMISE = 0x5,
  PING = 0x6,
  GOAWAY = 0x7,
  WINDOW_UPDATE = 0x8,
  CONTINUATION = 0x9
}

// 스트림 상태
enum StreamState {
  IDLE,
  RESERVED_LOCAL,
  RESERVED_REMOTE,
  OPEN,
  HALF_CLOSED_LOCAL,
  HALF_CLOSED_REMOTE,
  CLOSED
}

// HTTP/2 서버 구현
class HTTP2Server extends EventEmitter {
  private server: http2.Http2SecureServer | null = null;
  private streams: Map<number, StreamInfo> = new Map();
  private pushCache: Map<string, Buffer> = new Map();
  
  constructor() {
    super();
  }
  
  // 서버 시작
  start(port: number, keyPath: string, certPath: string): void {
    const options = {
      key: fs.readFileSync(keyPath),
      cert: fs.readFileSync(certPath),
      
      // HTTP/2 설정
      allowHTTP1: false,
      
      // 세션 설정
      settings: {
        enablePush: true,
        initialWindowSize: 65535,
        maxFrameSize: 16384,
        maxConcurrentStreams: 100,
        maxHeaderListSize: 8192
      }
    };
    
    this.server = http2.createSecureServer(options);
    
    this.server.on('stream', (stream, headers) => {
      this.handleStream(stream, headers);
    });
    
    this.server.on('session', (session) => {
      this.handleSession(session);
    });
    
    this.server.listen(port, () => {
      console.log(`HTTP/2 Server listening on port ${port}`);
      this.emit('listening', port);
    });
  }
  
  // 세션 처리
  private handleSession(session: http2.ServerHttp2Session): void {
    console.log('New HTTP/2 session established');
    
    // 세션 설정
    session.settings({
      enablePush: true,
      initialWindowSize: 65535,
      maxFrameSize: 16384
    });
    
    // 세션 이벤트
    session.on('error', (err) => {
      console.error('Session error:', err);
    });
    
    session.on('goaway', () => {
      console.log('Session goaway');
    });
    
    session.on('close', () => {
      console.log('Session closed');
    });
    
    // PING 프레임 처리
    session.ping((err, duration) => {
      if (!err) {
        console.log(`PING acknowledged in ${duration}ms`);
      }
    });
  }
  
  // 스트림 처리
  private handleStream(
    stream: http2.ServerHttp2Stream,
    headers: http2.IncomingHttpHeaders
  ): void {
    const streamId = stream.id;
    const path = headers[':path'] as string;
    const method = headers[':method'] as string;
    
    console.log(`Stream ${streamId}: ${method} ${path}`);
    
    // 스트림 정보 저장
    this.streams.set(streamId, {
      id: streamId,
      path: path,
      method: method,
      state: StreamState.OPEN,
      priority: stream.priority
    });
    
    // 라우팅
    if (path === '/') {
      this.serveIndex(stream);
    } else if (path.endsWith('.css')) {
      this.serveStatic(stream, path, 'text/css');
    } else if (path.endsWith('.js')) {
      this.serveStatic(stream, path, 'application/javascript');
    } else if (path.endsWith('.jpg') || path.endsWith('.png')) {
      this.serveImage(stream, path);
    } else {
      this.send404(stream);
    }
    
    // 스트림 이벤트
    stream.on('error', (err) => {
      console.error(`Stream ${streamId} error:`, err);
    });
    
    stream.on('close', () => {
      this.streams.delete(streamId);
    });
  }
  
  // 인덱스 페이지 제공 (서버 푸시 포함)
  private serveIndex(stream: http2.ServerHttp2Stream): void {
    const html = `
      <!DOCTYPE html>
      <html>
        <head>
          <title>HTTP/2 Demo</title>
          <link rel="stylesheet" href="/style.css">
          <script src="/script.js"></script>
        </head>
        <body>
          <h1>HTTP/2 Server Push Demo</h1>
          <img src="/image.jpg" alt="Demo">
        </body>
      </html>
    `;
    
    // 서버 푸시
    this.pushResource(stream, '/style.css', 'text/css');
    this.pushResource(stream, '/script.js', 'application/javascript');
    
    // HTML 응답
    stream.respond({
      ':status': 200,
      'content-type': 'text/html; charset=utf-8',
      'cache-control': 'no-cache'
    });
    
    stream.end(html);
  }
  
  // 서버 푸시
  private pushResource(
    stream: http2.ServerHttp2Stream,
    path: string,
    contentType: string
  ): void {
    if (!stream.pushAllowed) {
      console.log('Push not allowed');
      return;
    }
    
    stream.pushStream(
      { ':path': path },
      (err, pushStream) => {
        if (err) {
          console.error('Push error:', err);
          return;
        }
        
        console.log(`Pushing ${path}`);
        
        // 푸시 스트림에 응답
        pushStream.respond({
          ':status': 200,
          'content-type': contentType,
          'cache-control': 'max-age=3600'
        });
        
        // 실제 파일 내용 (시뮬레이션)
        const content = this.generateContent(path, contentType);
        pushStream.end(content);
      }
    );
  }
  
  // 정적 파일 제공
  private serveStatic(
    stream: http2.ServerHttp2Stream,
    filepath: string,
    contentType: string
  ): void {
    const content = this.generateContent(filepath, contentType);
    
    stream.respond({
      ':status': 200,
      'content-type': contentType,
      'cache-control': 'max-age=3600',
      'content-length': content.length
    });
    
    stream.end(content);
  }
  
  // 이미지 제공
  private serveImage(
    stream: http2.ServerHttp2Stream,
    filepath: string
  ): void {
    const ext = path.extname(filepath).slice(1);
    const contentType = `image/${ext}`;
    
    // 실제로는 파일 읽기
    const imageBuffer = Buffer.from([0xFF, 0xD8, 0xFF]); // JPEG 헤더
    
    stream.respond({
      ':status': 200,
      'content-type': contentType,
      'cache-control': 'max-age=86400',
      'content-length': imageBuffer.length
    });
    
    stream.end(imageBuffer);
  }
  
  // 404 응답
  private send404(stream: http2.ServerHttp2Stream): void {
    stream.respond({
      ':status': 404,
      'content-type': 'text/plain'
    });
    stream.end('Not Found');
  }
  
  // 콘텐츠 생성 (시뮬레이션)
  private generateContent(filepath: string, contentType: string): string {
    if (filepath === '/style.css') {
      return `
        body {
          font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
          margin: 0;
          padding: 20px;
          background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
          color: white;
        }
        h1 { font-size: 3em; text-shadow: 2px 2px 4px rgba(0,0,0,0.3); }
      `;
    } else if (filepath === '/script.js') {
      return `
        console.log('HTTP/2 Server Push - Script loaded!');
        document.addEventListener('DOMContentLoaded', () => {
          console.log('Page fully loaded with HTTP/2');
          
          // Performance timing
          const perfData = window.performance.timing;
          const pageLoadTime = perfData.loadEventEnd - perfData.navigationStart;
          console.log('Page load time:', pageLoadTime, 'ms');
        });
      `;
    }
    return '';
  }
}

// 스트림 정보 인터페이스
interface StreamInfo {
  id: number;
  path: string;
  method: string;
  state: StreamState;
  priority?: {
    exclusive: boolean;
    parent: number;
    weight: number;
  };
}

// HPACK 헤더 압축 구현
class HPACK {
  private staticTable: Map<number, [string, string]>;
  private dynamicTable: Array<[string, string]>;
  private maxDynamicTableSize: number = 4096;
  
  constructor() {
    this.staticTable = this.initStaticTable();
    this.dynamicTable = [];
  }
  
  // 정적 테이블 초기화
  private initStaticTable(): Map<number, [string, string]> {
    const table = new Map<number, [string, string]>();
    
    // HTTP/2 정적 테이블 (일부)
    table.set(1, [':authority', '']);
    table.set(2, [':method', 'GET']);
    table.set(3, [':method', 'POST']);
    table.set(4, [':path', '/']);
    table.set(5, [':path', '/index.html']);
    table.set(6, [':scheme', 'http']);
    table.set(7, [':scheme', 'https']);
    table.set(8, [':status', '200']);
    table.set(13, [':status', '404']);
    table.set(14, [':status', '500']);
    table.set(15, ['accept-charset', '']);
    table.set(16, ['accept-encoding', 'gzip, deflate']);
    // ... 총 61개
    
    return table;
  }
  
  // 헤더 인코딩
  encode(headers: Array<[string, string]>): Buffer {
    const encoded: number[] = [];
    
    for (const [name, value] of headers) {
      // 정적 테이블 확인
      let found = false;
      
      for (const [index, [sName, sValue]] of this.staticTable) {
        if (name === sName && value === sValue) {
          // 인덱스 헤더 (1바이트)
          encoded.push(0x80 | index);
          found = true;
          break;
        }
      }
      
      if (!found) {
        // 동적 테이블 확인
        const dynIndex = this.dynamicTable.findIndex(
          ([dName, dValue]) => dName === name && dValue === value
        );
        
        if (dynIndex !== -1) {
          // 동적 테이블 인덱스 (정적 테이블 크기 + 동적 인덱스)
          const index = 62 + dynIndex;
          encoded.push(0x80 | index);
        } else {
          // 리터럴 헤더 (새로운 이름-값)
          this.encodeLiteral(encoded, name, value);
          
          // 동적 테이블에 추가
          this.addToDynamicTable([name, value]);
        }
      }
    }
    
    return Buffer.from(encoded);
  }
  
  // 리터럴 인코딩
  private encodeLiteral(
    encoded: number[],
    name: string,
    value: string
  ): void {
    // 인덱싱 있는 리터럴 헤더
    encoded.push(0x40); // 01000000
    
    // 이름 길이와 값
    const nameBytes = Buffer.from(name);
    encoded.push(nameBytes.length);
    encoded.push(...nameBytes);
    
    // 값 길이와 값
    const valueBytes = Buffer.from(value);
    encoded.push(valueBytes.length);
    encoded.push(...valueBytes);
  }
  
  // 동적 테이블 추가
  private addToDynamicTable(entry: [string, string]): void {
    this.dynamicTable.unshift(entry);
    
    // 크기 제한
    while (this.getTableSize() > this.maxDynamicTableSize) {
      this.dynamicTable.pop();
    }
  }
  
  // 테이블 크기 계산
  private getTableSize(): number {
    return this.dynamicTable.reduce((size, [name, value]) => {
      return size + name.length + value.length + 32;
    }, 0);
  }
  
  // 헤더 디코딩
  decode(data: Buffer): Array<[string, string]> {
    const headers: Array<[string, string]> = [];
    let i = 0;
    
    while (i < data.length) {
      const byte = data[i];
      
      if (byte & 0x80) {
        // 인덱스 헤더
        const index = byte & 0x7F;
        const entry = this.getTableEntry(index);
        if (entry) {
          headers.push(entry);
        }
        i++;
      } else if (byte & 0x40) {
        // 인덱싱 있는 리터럴
        i++; // 플래그 바이트 건너뛰기
        const [name, value, nextIndex] = this.decodeLiteral(data, i);
        headers.push([name, value]);
        this.addToDynamicTable([name, value]);
        i = nextIndex;
      } else {
        // 기타 타입 (간단 구현)
        i++;
      }
    }
    
    return headers;
  }
  
  // 테이블 항목 가져오기
  private getTableEntry(index: number): [string, string] | null {
    if (index <= 61) {
      return this.staticTable.get(index) || null;
    } else {
      const dynIndex = index - 62;
      return this.dynamicTable[dynIndex] || null;
    }
  }
  
  // 리터럴 디코딩
  private decodeLiteral(
    data: Buffer,
    start: number
  ): [string, string, number] {
    // 이름 길이
    const nameLen = data[start];
    const name = data.slice(start + 1, start + 1 + nameLen).toString();
    
    // 값 길이
    const valueStart = start + 1 + nameLen;
    const valueLen = data[valueStart];
    const value = data.slice(
      valueStart + 1,
      valueStart + 1 + valueLen
    ).toString();
    
    const nextIndex = valueStart + 1 + valueLen;
    return [name, value, nextIndex];
  }
}

// 성능 벤치마크
class HTTP2Benchmark {
  // HTTP/1.1 vs HTTP/2 비교
  static async compareProtocols(): Promise<void> {
    console.log('\n=== HTTP/1.1 vs HTTP/2 Performance ===\n');
    
    // 시뮬레이션: 10개 리소스 로드
    const resources = [
      'index.html', 'style.css', 'script.js',
      'image1.jpg', 'image2.jpg', 'image3.jpg',
      'font1.woff', 'font2.woff',
      'api/data1.json', 'api/data2.json'
    ];
    
    // HTTP/1.1 시뮬레이션 (6개 연결)
    console.log('HTTP/1.1 (6 connections):');
    const http1Start = Date.now();
    await this.simulateHTTP1(resources);
    const http1Time = Date.now() - http1Start;
    console.log(`Total time: ${http1Time}ms\n`);
    
    // HTTP/2 시뮬레이션 (1개 연결)
    console.log('HTTP/2 (1 connection, multiplexing):');
    const http2Start = Date.now();
    await this.simulateHTTP2(resources);
    const http2Time = Date.now() - http2Start;
    console.log(`Total time: ${http2Time}ms\n`);
    
    const improvement = ((http1Time - http2Time) / http1Time * 100).toFixed(1);
    console.log(`HTTP/2 is ${improvement}% faster`);
  }
  
  private static async simulateHTTP1(resources: string[]): Promise<void> {
    const maxConnections = 6;
    const queue = [...resources];
    const active: Promise<void>[] = [];
    
    while (queue.length > 0 || active.length > 0) {
      while (active.length < maxConnections && queue.length > 0) {
        const resource = queue.shift()!;
        const promise = this.fetchResource(resource, 50 + Math.random() * 100);
        active.push(promise);
      }
      
      if (active.length > 0) {
        const completed = await Promise.race(active.map((p, i) => 
          p.then(() => i)
        ));
        active.splice(completed, 1);
      }
    }
  }
  
  private static async simulateHTTP2(resources: string[]): Promise<void> {
    // 모든 리소스 병렬 요청 (멀티플렉싱)
    await Promise.all(
      resources.map(r => this.fetchResource(r, 30 + Math.random() * 50))
    );
  }
  
  private static async fetchResource(
    name: string,
    delay: number
  ): Promise<void> {
    await new Promise(resolve => setTimeout(resolve, delay));
    console.log(`  Loaded: ${name} (${delay.toFixed(0)}ms)`);
  }
}

// 최적화 비교
class OptimizationComparison {
  static demonstrate(): void {
    console.log('\n=== HTTP/1.x vs HTTP/2 Optimization ===\n');
    
    console.log('HTTP/1.x Optimizations:');
    console.log('  ✓ Domain Sharding (cdn1, cdn2, cdn3)');
    console.log('  ✓ Resource Bundling (bundle.js)');
    console.log('  ✓ Image Sprites');
    console.log('  ✓ Inline Critical CSS');
    console.log('  → Complex build process');
    console.log('  → Cache invalidation issues\n');
    
    console.log('HTTP/2 Optimizations:');
    console.log('  ✗ Domain Sharding (unnecessary)');
    console.log('  △ Resource Bundling (case by case)');
    console.log('  ✗ Image Sprites (multiplexing handles it)');
    console.log('  ✓ Server Push for critical resources');
    console.log('  → Simpler build process');
    console.log('  → Better cache granularity\n');
    
    console.log('New Best Practices:');
    console.log('  • Use single domain');
    console.log('  • Smaller, cacheable resources');
    console.log('  • Resource hints (preload, prefetch)');
    console.log('  • Priority hints');
    console.log('  • Conditional server push');
  }
}

// 사용 예제
async function demonstrateHTTP2() {
  console.log('=== HTTP/2 and Optimization Demo ===\n');
  
  // HPACK 압축 데모
  console.log('=== HPACK Compression ===');
  const hpack = new HPACK();
  
  const headers: Array<[string, string]> = [
    [':method', 'GET'],
    [':path', '/'],
    [':scheme', 'https'],
    ['user-agent', 'Chrome/90'],
    ['accept', 'text/html']
  ];
  
  const encoded = hpack.encode(headers);
  console.log('Original headers:', headers);
  console.log('Encoded size:', encoded.length, 'bytes');
  console.log('Compression ratio:', 
    (1 - encoded.length / JSON.stringify(headers).length) * 100, '%\n');
  
  // 프로토콜 비교
  await HTTP2Benchmark.compareProtocols();
  
  // 최적화 전략 비교
  OptimizationComparison.demonstrate();
}

// 실행
// demonstrateHTTP2().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **멀티플렉싱**: 단일 연결로 다중 요청/응답
> 2. **서버 푸시**: 요청 전에 리소스 전송
> 3. **HPACK**: 헤더 압축으로 오버헤드 감소
> 4. **바이너리 프로토콜**: 효율적 파싱과 처리
> 5. **최적화 전략 변경**: 번들링/샤딩 → 개별 리소스

## 참고 자료

### 📖 추가 학습
- [HTTP/2 - RFC 7540](https://tools.ietf.org/html/rfc7540)
- [HTTP/2 Explained](https://http2-explained.haxx.se/)
- [Web Fundamentals - HTTP/2](https://developers.google.com/web/fundamentals/performance/http2)
- [HPACK - RFC 7541](https://tools.ietf.org/html/rfc7541)

### 🔗 관련 문서
- [[12_http-protocol|HTTP 프로토콜]]
- [[13_https-tls|HTTPS와 TLS]]
- [[17_realtime-communication|실시간 통신]]
