---
title: HTTPS와 TLS
created: 2025-01-03
updated: 2025-01-03
tags:
  - CS
  - Network
  - HTTPS
  - TLS
  - SSL
  - Security
  - Encryption
category: Network
status: 완료
---

# 🔒 HTTPS와 TLS (HTTPS and TLS)

## 📌 개요

HTTPS(HTTP Secure)는 TLS(Transport Layer Security)를 사용하여 HTTP 통신을 암호화한 프로토콜입니다. 기밀성, 무결성, 인증을 제공하여 안전한 웹 통신을 보장합니다.

> [!info] 핵심 개념
> - **TLS/SSL**: 전송 계층 보안 프로토콜
> - **End-to-End 암호화**: 종단 간 암호화
> - **인증서**: 서버 신원 확인
> - **CA**: 인증서 발급 기관

## 📚 목차

1. [HTTPS 개요](#https-개요)
2. [TLS/SSL 프로토콜](#tlsssl-프로토콜)
3. [TLS 핸드셰이크](#tls-핸드셰이크)
4. [암호화 알고리즘](#암호화-알고리즘)
5. [디지털 인증서](#디지털-인증서)
6. [인증 기관 (CA)](#인증-기관-ca)
7. [Let's Encrypt](#lets-encrypt)
8. [실제 구현 예시](#실제-구현-예시)
9. [참고 자료](#참고-자료)

## HTTPS 개요

### HTTP vs HTTPS
```
HTTP:  Client ----[평문]----> Server
       도청 가능 ⚠️

HTTPS: Client =====[암호화]====> Server
       도청 불가 ✅
```

### HTTPS가 제공하는 보안
1. **기밀성(Confidentiality)**: 데이터 암호화
2. **무결성(Integrity)**: 데이터 변조 방지
3. **인증(Authentication)**: 서버 신원 확인
4. **부인 방지(Non-repudiation)**: 전송 사실 부인 방지

## TLS/SSL 프로토콜

### 역사
- **SSL 1.0**: 미공개 (보안 문제)
- **SSL 2.0**: 1995년 (구식, 사용 금지)
- **SSL 3.0**: 1996년 (POODLE 공격 취약)
- **TLS 1.0**: 1999년 (SSL 3.1)
- **TLS 1.1**: 2006년
- **TLS 1.2**: 2008년 (현재 표준)
- **TLS 1.3**: 2018년 (최신, 더 빠르고 안전)

## TLS 핸드셰이크

### TLS 1.2 핸드셰이크 과정
```
Client                                  Server
  |                                        |
  |-------- 1. Client Hello -------------->|
  |         (지원 암호 스위트 목록)           |
  |                                        |
  |<------- 2. Server Hello ----------------|
  |         (선택된 암호 스위트)              |
  |                                        |
  |<------- 3. Certificate -----------------|
  |         (서버 인증서)                    |
  |                                        |
  |<------- 4. Server Key Exchange ---------|
  |         (키 교환 파라미터)                |
  |                                        |
  |<------- 5. Server Hello Done -----------|
  |                                        |
  |-------- 6. Client Key Exchange -------->|
  |         (Pre-Master Secret)             |
  |                                        |
  |-------- 7. Change Cipher Spec -------->|
  |-------- 8. Finished ------------------>|
  |                                        |
  |<------- 9. Change Cipher Spec ----------|
  |<------- 10. Finished -------------------|
  |                                        |
  |======== Application Data =============>|
```

### 핸드셰이크 단계별 설명

1. **Client Hello**
   - 클라이언트가 지원하는 TLS 버전
   - 지원 암호 스위트 목록
   - 클라이언트 랜덤 값

2. **Server Hello**
   - 선택된 TLS 버전
   - 선택된 암호 스위트
   - 서버 랜덤 값

3. **Certificate**
   - 서버의 X.509 인증서
   - 인증서 체인

4. **Key Exchange**
   - 암호화 키 생성을 위한 정보 교환
   - Pre-Master Secret 생성

## 암호화 알고리즘

### 대칭키 암호화
- **같은 키**로 암호화/복호화
- 빠른 속도
- 예: AES-128, AES-256, ChaCha20

### 비대칭키 암호화
- **공개키/개인키** 쌍 사용
- 느린 속도, 높은 보안
- 예: RSA, ECDSA, EdDSA

### 하이브리드 방식
```
1. 비대칭키로 세션키 교환
2. 대칭키(세션키)로 실제 데이터 암호화
→ 보안성 + 속도 모두 확보
```

## 디지털 인증서

### X.509 인증서 구조
```
Certificate:
    Version: 3
    Serial Number: xx:xx:xx:xx
    Issuer: CN=Let's Encrypt Authority X3
    Validity:
        Not Before: Jan 1 00:00:00 2024
        Not After: Apr 1 23:59:59 2024
    Subject: CN=example.com
    Subject Alternative Names:
        DNS:example.com
        DNS:www.example.com
    Public Key Info:
        Algorithm: RSA
        Public Key: (2048 bit)
    Signature Algorithm: SHA256withRSA
    Signature: ...
```

### 인증서 체인
```
Root CA (자체 서명)
    └─> Intermediate CA
            └─> Server Certificate (example.com)
```

## 인증 기관 (CA)

### CA의 역할
1. **신원 확인**: 도메인 소유권 검증
2. **인증서 발급**: 디지털 서명된 인증서
3. **인증서 폐기**: CRL(Certificate Revocation List) 관리

### 주요 CA
- DigiCert
- GlobalSign
- Let's Encrypt (무료)
- Comodo/Sectigo
- GoDaddy

### 인증서 검증 과정
1. 인증서 체인 확인
2. CA 신뢰 여부 확인
3. 유효 기간 확인
4. 도메인 일치 확인
5. 폐기 목록 확인

## Let's Encrypt

### 특징
- **무료**: 비용 없이 SSL/TLS 인증서 발급
- **자동화**: ACME 프로토콜로 자동 발급/갱신
- **90일 유효기간**: 자동 갱신 권장
- **와일드카드 지원**: *.example.com

### Certbot을 이용한 인증서 발급
```bash
# 설치
sudo apt-get install certbot

# 인증서 발급 (Standalone)
sudo certbot certonly --standalone -d example.com

# 인증서 발급 (Webroot)
sudo certbot certonly --webroot -w /var/www/html -d example.com

# 자동 갱신
sudo certbot renew --dry-run
```

## 실제 구현 예시

### TypeScript로 HTTPS 서버 구현

```typescript
import * as https from 'https';
import * as fs from 'fs';
import * as crypto from 'crypto';
import { EventEmitter } from 'events';

// TLS 설정 인터페이스
interface TLSConfig {
  key: string;
  cert: string;
  ca?: string;
  minVersion?: string;
  maxVersion?: string;
  ciphers?: string;
}

// HTTPS 서버 클래스
class SecureHTTPSServer extends EventEmitter {
  private server: https.Server | null = null;
  private config: TLSConfig;
  
  constructor(config: TLSConfig) {
    super();
    this.config = config;
  }
  
  // 서버 생성 및 시작
  start(port: number, hostname: string = 'localhost'): void {
    const options = {
      key: fs.readFileSync(this.config.key),
      cert: fs.readFileSync(this.config.cert),
      ca: this.config.ca ? fs.readFileSync(this.config.ca) : undefined,
      
      // TLS 버전 설정
      minVersion: this.config.minVersion || 'TLSv1.2',
      maxVersion: this.config.maxVersion || 'TLSv1.3',
      
      // 암호 스위트 설정
      ciphers: this.config.ciphers || [
        'ECDHE-RSA-AES128-GCM-SHA256',
        'ECDHE-RSA-AES256-GCM-SHA384',
        'ECDHE-RSA-CHACHA20-POLY1305',
        'DHE-RSA-AES128-GCM-SHA256',
        'DHE-RSA-AES256-GCM-SHA384'
      ].join(':'),
      
      // 보안 옵션
      honorCipherOrder: true,
      secureOptions: crypto.constants.SSL_OP_NO_TLSv1 | 
                    crypto.constants.SSL_OP_NO_TLSv1_1
    };
    
    this.server = https.createServer(options, (req, res) => {
      this.handleRequest(req, res);
    });
    
    this.server.listen(port, hostname, () => {
      console.log(`HTTPS Server running at https://${hostname}:${port}`);
      this.emit('listening', port);
    });
    
    // TLS 연결 이벤트
    this.server.on('secureConnection', (tlsSocket) => {
      console.log('TLS Connection established');
      console.log('Cipher:', tlsSocket.getCipher());
      console.log('Protocol:', tlsSocket.getProtocol());
      this.emit('tlsConnection', tlsSocket);
    });
  }
  
  // 요청 처리
  private handleRequest(req: any, res: any): void {
    // HSTS 헤더 추가
    res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
    
    // CSP 헤더
    res.setHeader('Content-Security-Policy', "default-src 'self'");
    
    // 기타 보안 헤더
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    res.setHeader('X-XSS-Protection', '1; mode=block');
    
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end(`
      <!DOCTYPE html>
      <html>
        <head><title>Secure HTTPS Server</title></head>
        <body>
          <h1>🔒 Secure Connection</h1>
          <p>This page is served over HTTPS</p>
        </body>
      </html>
    `);
  }
}

// 인증서 생성 클래스 (자체 서명 - 테스트용)
class CertificateGenerator {
  // 자체 서명 인증서 생성
  static generateSelfSigned(domain: string): { key: string; cert: string } {
    const { privateKey, publicKey } = crypto.generateKeyPairSync('rsa', {
      modulusLength: 2048,
      publicKeyEncoding: { type: 'spki', format: 'pem' },
      privateKeyEncoding: { type: 'pkcs8', format: 'pem' }
    });
    
    // 간단한 자체 서명 인증서 (실제로는 X.509 형식 필요)
    const cert = `-----BEGIN CERTIFICATE-----
Self-signed certificate for ${domain}
Public Key: ${publicKey}
Valid: ${new Date().toISOString()}
-----END CERTIFICATE-----`;
    
    return {
      key: privateKey,
      cert: cert
    };
  }
}

// TLS 클라이언트
class TLSClient {
  // HTTPS 요청
  static async request(
    url: string,
    options: https.RequestOptions = {}
  ): Promise<any> {
    return new Promise((resolve, reject) => {
      const req = https.request(url, options, (res) => {
        let data = '';
        
        // TLS 정보 출력
        const socket = res.socket as any;
        if (socket.encrypted) {
          console.log('TLS Info:');
          console.log('  Protocol:', socket.getProtocol());
          console.log('  Cipher:', socket.getCipher());
          console.log('  Certificate:', socket.getPeerCertificate());
        }
        
        res.on('data', chunk => data += chunk);
        res.on('end', () => resolve({
          statusCode: res.statusCode,
          headers: res.headers,
          data: data
        }));
      });
      
      req.on('error', reject);
      req.end();
    });
  }
}

// 인증서 검증 클래스
class CertificateValidator {
  // 인증서 체인 검증
  static validateChain(cert: any): boolean {
    // 실제 구현에서는 X.509 파싱 및 검증 필요
    const validations = {
      expired: this.checkExpiry(cert),
      domainMatch: this.checkDomain(cert),
      trusted: this.checkTrust(cert)
    };
    
    return Object.values(validations).every(v => v);
  }
  
  private static checkExpiry(cert: any): boolean {
    // 유효 기간 확인
    const now = new Date();
    const notBefore = new Date(cert.valid_from);
    const notAfter = new Date(cert.valid_to);
    return now >= notBefore && now <= notAfter;
  }
  
  private static checkDomain(cert: any): boolean {
    // 도메인 일치 확인
    // CN 또는 SAN 확인
    return true; // 간단 구현
  }
  
  private static checkTrust(cert: any): boolean {
    // CA 신뢰 확인
    // 시스템 신뢰 저장소 확인
    return true; // 간단 구현
  }
}

// MITM 공격 시뮬레이션
class MITMSimulator {
  static demonstrateAttack(): void {
    console.log('\n=== Man-in-the-Middle Attack Simulation ===\n');
    
    // HTTP 통신 (취약)
    console.log('HTTP Communication:');
    console.log('Client --> [LOGIN: user=alice, pass=secret123] --> Server');
    console.log('           ↑ MITM can read this! ⚠️\n');
    
    // HTTPS 통신 (안전)
    console.log('HTTPS Communication:');
    console.log('Client ==> [ENCRYPTED DATA: x#$@!%^&*...] ==> Server');
    console.log('           ↑ MITM sees only encrypted data ✅\n');
  }
  
  static demonstrateDNSSpoofing(): void {
    console.log('=== DNS Spoofing Attack ===\n');
    console.log('1. User requests: bank.com');
    console.log('2. MITM intercepts DNS response');
    console.log('3. MITM returns fake IP: 1.2.3.4 (attacker server)\n');
    console.log('Without HTTPS:');
    console.log('  - User connects to fake server ⚠️');
    console.log('  - Credentials stolen!\n');
    console.log('With HTTPS:');
    console.log('  - Certificate mismatch detected ✅');
    console.log('  - Browser shows warning');
    console.log('  - Connection blocked');
  }
}

// 암호화 알고리즘 벤치마크
class CryptoBenchmark {
  // AES 암호화 테스트
  static benchmarkAES(data: Buffer, iterations: number = 1000): void {
    const key = crypto.randomBytes(32); // 256-bit key
    const iv = crypto.randomBytes(16);
    
    console.log('\n=== AES-256-GCM Benchmark ===');
    const start = process.hrtime.bigint();
    
    for (let i = 0; i < iterations; i++) {
      const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
      const encrypted = Buffer.concat([
        cipher.update(data),
        cipher.final()
      ]);
      const authTag = cipher.getAuthTag();
      
      const decipher = crypto.createDecipheriv('aes-256-gcm', key, iv);
      decipher.setAuthTag(authTag);
      const decrypted = Buffer.concat([
        decipher.update(encrypted),
        decipher.final()
      ]);
    }
    
    const end = process.hrtime.bigint();
    const duration = Number(end - start) / 1_000_000; // to ms
    
    console.log(`Processed ${iterations} iterations`);
    console.log(`Data size: ${data.length} bytes`);
    console.log(`Total time: ${duration.toFixed(2)} ms`);
    console.log(`Throughput: ${(iterations * data.length / duration * 1000 / 1024 / 1024).toFixed(2)} MB/s`);
  }
  
  // RSA vs ECC 비교
  static compareAsymmetric(): void {
    console.log('\n=== RSA vs ECC Comparison ===\n');
    
    // RSA 2048
    const rsaStart = process.hrtime.bigint();
    const rsa = crypto.generateKeyPairSync('rsa', {
      modulusLength: 2048
    });
    const rsaEnd = process.hrtime.bigint();
    
    // ECC P-256
    const eccStart = process.hrtime.bigint();
    const ecc = crypto.generateKeyPairSync('ec', {
      namedCurve: 'P-256'
    });
    const eccEnd = process.hrtime.bigint();
    
    console.log('Key Generation Time:');
    console.log(`RSA-2048: ${Number(rsaEnd - rsaStart) / 1_000_000} ms`);
    console.log(`ECC-P256: ${Number(eccEnd - eccStart) / 1_000_000} ms`);
    
    // 서명 속도 테스트
    const data = crypto.randomBytes(32);
    const iterations = 100;
    
    // RSA 서명
    const rsaSignStart = process.hrtime.bigint();
    for (let i = 0; i < iterations; i++) {
      const sign = crypto.createSign('SHA256');
      sign.update(data);
      sign.end();
      sign.sign(rsa.privateKey);
    }
    const rsaSignEnd = process.hrtime.bigint();
    
    // ECC 서명
    const eccSignStart = process.hrtime.bigint();
    for (let i = 0; i < iterations; i++) {
      const sign = crypto.createSign('SHA256');
      sign.update(data);
      sign.end();
      sign.sign(ecc.privateKey);
    }
    const eccSignEnd = process.hrtime.bigint();
    
    console.log(`\nSigning Speed (${iterations} iterations):`);
    console.log(`RSA-2048: ${Number(rsaSignEnd - rsaSignStart) / 1_000_000} ms`);
    console.log(`ECC-P256: ${Number(eccSignEnd - eccSignStart) / 1_000_000} ms`);
  }
}

// 보안 헤더 미들웨어
class SecurityHeaders {
  static apply(req: any, res: any): void {
    // HSTS - HTTP Strict Transport Security
    res.setHeader(
      'Strict-Transport-Security',
      'max-age=31536000; includeSubDomains; preload'
    );
    
    // CSP - Content Security Policy
    res.setHeader(
      'Content-Security-Policy',
      "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'"
    );
    
    // 기타 보안 헤더
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'SAMEORIGIN');
    res.setHeader('X-XSS-Protection', '1; mode=block');
    res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
    res.setHeader('Permissions-Policy', 'geolocation=(), microphone=()');
  }
}

// 사용 예제
async function demonstrateHTTPS() {
  console.log('=== HTTPS and TLS Demonstration ===\n');
  
  // MITM 공격 시연
  MITMSimulator.demonstrateAttack();
  MITMSimulator.demonstrateDNSSpoofing();
  
  // 암호화 벤치마크
  const testData = crypto.randomBytes(1024); // 1KB
  CryptoBenchmark.benchmarkAES(testData);
  CryptoBenchmark.compareAsymmetric();
  
  console.log('\n=== Certificate Validation Example ===');
  const mockCert = {
    valid_from: new Date(Date.now() - 86400000).toISOString(),
    valid_to: new Date(Date.now() + 86400000).toISOString(),
    subject: { CN: 'example.com' }
  };
  
  console.log('Certificate valid:', CertificateValidator.validateChain(mockCert));
}

// 실행
// demonstrateHTTPS().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **HTTPS = HTTP + TLS**: 암호화된 HTTP 통신
> 2. **TLS 핸드셰이크**: 안전한 연결 수립 과정
> 3. **하이브리드 암호화**: 비대칭키로 키 교환, 대칭키로 데이터 암호화
> 4. **인증서**: CA가 서명한 서버 신원 증명
> 5. **Let's Encrypt**: 무료 자동화 인증서 발급

## 참고 자료

### 📖 추가 학습
- [TLS - RFC 8446](https://tools.ietf.org/html/rfc8446)
- [Let's Encrypt](https://letsencrypt.org/)
- [SSL Labs](https://www.ssllabs.com/)
- [Mozilla SSL Configuration](https://ssl-config.mozilla.org/)

### 🔗 관련 문서
- [[12_http-protocol|HTTP 프로토콜]]
- [[14_http2-optimization|HTTP/2와 최적화]]
- [[10_firewall-dns|방화벽과 DNS]]
