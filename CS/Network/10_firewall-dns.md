---
title: 방화벽과 DNS
created: 2024-12-31
updated: 2024-12-31
tags:
  - CS
  - Network
  - Firewall
  - DNS
  - Security
  - Domain
category: Network
status: 완료
---

# 🛡️ 방화벽과 DNS (Firewall and DNS)

## 📌 개요

방화벽(Firewall)은 네트워크 보안의 첫 번째 방어선이며, DNS(Domain Name System)는 인터넷의 전화번호부 역할을 합니다. 방화벽이 문지기라면, DNS는 주소록입니다.

> [!info] 핵심 개념
> - **방화벽**: 네트워크 트래픽 필터링 및 제어
> - **DNS**: 도메인 이름을 IP 주소로 변환
> - **보안 규칙**: Inbound/Outbound 트래픽 제어
> - **DNS 계층**: Root → TLD → Authoritative

## 📚 목차

1. [방화벽 (Firewall)](#방화벽-firewall)
2. [방화벽 규칙과 정책](#방화벽-규칙과-정책)
3. [DNS (Domain Name System)](#dns-domain-name-system)
4. [DNS 동작 과정](#dns-동작-과정)
5. [실제 구현 예시](#실제-구현-예시)
6. [참고 자료](#참고-자료)

## 방화벽 (Firewall)

### 정의와 목적
- **정의**: 신뢰할 수 있는 내부 네트워크와 신뢰할 수 없는 외부 네트워크 사이의 보안 장벽
- **목적**: 
  - 비인가 접근 차단
  - 악성 트래픽 필터링
  - 네트워크 정책 시행
  - 로깅 및 감사

### 방화벽 유형

#### 1. 패킷 필터링 방화벽
- OSI Layer 3-4에서 동작
- IP, 포트, 프로토콜 기반 필터링
- Stateless: 각 패킷 독립적 검사

#### 2. Stateful 방화벽
- 연결 상태 추적
- 세션 테이블 유지
- 더 정교한 필터링

#### 3. 응용 계층 방화벽 (Proxy)
- OSI Layer 7에서 동작
- 애플리케이션 프로토콜 이해
- 콘텐츠 기반 필터링

#### 4. 차세대 방화벽 (NGFW)
- 통합 보안 기능
- DPI (Deep Packet Inspection)
- IPS/IDS 통합

## 방화벽 규칙과 정책

### 규칙 구성 요소
```
Action | Protocol | Source IP | Source Port | Dest IP | Dest Port | Direction
───────┼──────────┼───────────┼─────────────┼─────────┼───────────┼───────────
ALLOW  | TCP      | Any       | Any         | 10.0.0.1| 80        | Inbound
DENY   | UDP      | 192.168.x | Any         | Any     | 53        | Outbound
```

### 기본 정책
- **Default Deny**: 명시적 허용 외 모두 차단 (보안 우선)
- **Default Allow**: 명시적 차단 외 모두 허용 (편의 우선)

### 규칙 우선순위
1. 더 구체적인 규칙 우선
2. Deny 규칙이 Allow보다 우선
3. 순서대로 매칭 (First Match)

## DNS (Domain Name System)

### DNS 계층 구조

```
                    . (Root)
                   /    |    \
              .com    .org    .kr
              /  \      |      |
         google  naver  wiki  co.kr
            |      |     |      |
          www    mail  en   example
```

### DNS 서버 유형

#### 1. Root Name Server
- 최상위 서버 (전 세계 13개 세트)
- TLD 서버 정보 제공

#### 2. TLD (Top Level Domain) Server
- .com, .org, .kr 등 관리
- 권한 있는 네임서버 정보 제공

#### 3. Authoritative Name Server
- 실제 도메인-IP 매핑 보유
- 최종 답변 제공

#### 4. Recursive Resolver
- 클라이언트 요청 대행
- 캐싱 기능

## DNS 동작 과정

### DNS 조회 과정
1. **로컬 캐시 확인**: 브라우저 → OS 캐시
2. **Recursive Resolver 질의**: ISP DNS 서버
3. **Root 서버 질의**: TLD 서버 주소 획득
4. **TLD 서버 질의**: Authoritative 서버 주소 획득
5. **Authoritative 서버 질의**: 최종 IP 주소 획득
6. **응답 및 캐싱**: 결과 저장 및 반환

### DNS 레코드 타입

| 타입 | 설명 | 예시 |
|------|------|------|
| **A** | IPv4 주소 | example.com → 93.184.216.34 |
| **AAAA** | IPv6 주소 | example.com → 2606:2800:220:1:248:1893:25c8:1946 |
| **CNAME** | 별칭 | www.example.com → example.com |
| **MX** | 메일 서버 | mail.example.com (우선순위: 10) |
| **TXT** | 텍스트 정보 | SPF, DKIM 검증 |
| **NS** | 네임서버 | ns1.example.com |
| **PTR** | 역방향 조회 | 93.184.216.34 → example.com |

## 실제 구현 예시

### TypeScript로 방화벽과 DNS 구현

```typescript
// 방화벽 규칙 인터페이스
interface FirewallRule {
  id: number;
  priority: number;
  action: 'ALLOW' | 'DENY';
  protocol: 'TCP' | 'UDP' | 'ICMP' | 'ANY';
  source: {
    ip: string;
    port?: number | 'ANY';
  };
  destination: {
    ip: string;
    port?: number | 'ANY';
  };
  direction: 'INBOUND' | 'OUTBOUND' | 'BOTH';
  enabled: boolean;
  description?: string;
}

// 패킷 인터페이스
interface Packet {
  protocol: 'TCP' | 'UDP' | 'ICMP';
  source: { ip: string; port?: number };
  destination: { ip: string; port?: number };
  direction: 'INBOUND' | 'OUTBOUND';
  data?: Buffer;
  timestamp: Date;
}

// 방화벽 구현
class Firewall {
  private rules: FirewallRule[] = [];
  private defaultPolicy: 'ALLOW' | 'DENY' = 'DENY';
  private logs: Array<{
    timestamp: Date;
    packet: Packet;
    action: 'ALLOWED' | 'DENIED';
    rule?: FirewallRule;
  }> = [];
  private connectionTable: Map<string, {
    state: 'ESTABLISHED' | 'RELATED' | 'NEW';
    created: Date;
    lastSeen: Date;
  }> = new Map();

  constructor(defaultPolicy: 'ALLOW' | 'DENY' = 'DENY') {
    this.defaultPolicy = defaultPolicy;
    this.initializeDefaultRules();
  }

  // 기본 규칙 초기화
  private initializeDefaultRules(): void {
    // 루프백 허용
    this.addRule({
      id: 1,
      priority: 1,
      action: 'ALLOW',
      protocol: 'ANY',
      source: { ip: '127.0.0.1', port: 'ANY' },
      destination: { ip: '127.0.0.1', port: 'ANY' },
      direction: 'BOTH',
      enabled: true,
      description: 'Allow loopback'
    });

    // Established 연결 허용
    this.addRule({
      id: 2,
      priority: 2,
      action: 'ALLOW',
      protocol: 'TCP',
      source: { ip: 'ANY', port: 'ANY' },
      destination: { ip: 'ANY', port: 'ANY' },
      direction: 'INBOUND',
      enabled: true,
      description: 'Allow established connections'
    });

    // DHCP 허용
    this.addRule({
      id: 3,
      priority: 10,
      action: 'ALLOW',
      protocol: 'UDP',
      source: { ip: 'ANY', port: 68 },
      destination: { ip: 'ANY', port: 67 },
      direction: 'OUTBOUND',
      enabled: true,
      description: 'Allow DHCP'
    });

    // DNS 허용
    this.addRule({
      id: 4,
      priority: 11,
      action: 'ALLOW',
      protocol: 'UDP',
      source: { ip: 'ANY', port: 'ANY' },
      destination: { ip: 'ANY', port: 53 },
      direction: 'OUTBOUND',
      enabled: true,
      description: 'Allow DNS'
    });
  }

  // 규칙 추가
  addRule(rule: FirewallRule): void {
    this.rules.push(rule);
    this.rules.sort((a, b) => a.priority - b.priority);
    console.log(`Firewall rule added: ${rule.description || `Rule ${rule.id}`}`);
  }

  // 규칙 제거
  removeRule(id: number): void {
    this.rules = this.rules.filter(rule => rule.id !== id);
    console.log(`Firewall rule ${id} removed`);
  }

  // 패킷 필터링
  filterPacket(packet: Packet): boolean {
    // Stateful 검사
    if (this.isEstablishedConnection(packet)) {
      this.log(packet, 'ALLOWED');
      return true;
    }

    // 규칙 매칭
    for (const rule of this.rules) {
      if (!rule.enabled) continue;
      
      if (this.matchRule(rule, packet)) {
        this.log(packet, rule.action === 'ALLOW' ? 'ALLOWED' : 'DENIED', rule);
        
        if (rule.action === 'ALLOW') {
          this.trackConnection(packet);
        }
        
        return rule.action === 'ALLOW';
      }
    }

    // 기본 정책
    this.log(packet, this.defaultPolicy === 'ALLOW' ? 'ALLOWED' : 'DENIED');
    return this.defaultPolicy === 'ALLOW';
  }

  // 규칙 매칭
  private matchRule(rule: FirewallRule, packet: Packet): boolean {
    // 방향 확인
    if (rule.direction !== 'BOTH' && rule.direction !== packet.direction) {
      return false;
    }

    // 프로토콜 확인
    if (rule.protocol !== 'ANY' && rule.protocol !== packet.protocol) {
      return false;
    }

    // 소스 IP 확인
    if (rule.source.ip !== 'ANY' && !this.matchIP(rule.source.ip, packet.source.ip)) {
      return false;
    }

    // 소스 포트 확인
    if (rule.source.port !== 'ANY' && rule.source.port !== packet.source.port) {
      return false;
    }

    // 목적지 IP 확인
    if (rule.destination.ip !== 'ANY' && !this.matchIP(rule.destination.ip, packet.destination.ip)) {
      return false;
    }

    // 목적지 포트 확인
    if (rule.destination.port !== 'ANY' && rule.destination.port !== packet.destination.port) {
      return false;
    }

    return true;
  }

  // IP 매칭 (CIDR 지원)
  private matchIP(pattern: string, ip: string): boolean {
    if (pattern === ip) return true;
    
    // CIDR 표기법 처리
    if (pattern.includes('/')) {
      return this.matchCIDR(pattern, ip);
    }
    
    // 와일드카드 처리
    if (pattern.includes('*')) {
      const regex = new RegExp(pattern.replace(/\*/g, '.*'));
      return regex.test(ip);
    }
    
    return false;
  }

  // CIDR 매칭
  private matchCIDR(cidr: string, ip: string): boolean {
    const [network, prefixLength] = cidr.split('/');
    const prefix = parseInt(prefixLength, 10);
    
    const networkInt = this.ipToInt(network);
    const ipInt = this.ipToInt(ip);
    const mask = (0xFFFFFFFF << (32 - prefix)) >>> 0;
    
    return (networkInt & mask) === (ipInt & mask);
  }

  private ipToInt(ip: string): number {
    const parts = ip.split('.');
    return parts.reduce((acc, part, index) => {
      return acc + (parseInt(part, 10) << ((3 - index) * 8));
    }, 0) >>> 0;
  }

  // Stateful 연결 추적
  private isEstablishedConnection(packet: Packet): boolean {
    const key = this.getConnectionKey(packet);
    const conn = this.connectionTable.get(key);
    
    if (conn && conn.state === 'ESTABLISHED') {
      conn.lastSeen = new Date();
      return true;
    }
    
    return false;
  }

  private trackConnection(packet: Packet): void {
    const key = this.getConnectionKey(packet);
    this.connectionTable.set(key, {
      state: 'NEW',
      created: new Date(),
      lastSeen: new Date()
    });
  }

  private getConnectionKey(packet: Packet): string {
    return `${packet.protocol}:${packet.source.ip}:${packet.source.port}-${packet.destination.ip}:${packet.destination.port}`;
  }

  // 로깅
  private log(packet: Packet, action: 'ALLOWED' | 'DENIED', rule?: FirewallRule): void {
    this.logs.push({
      timestamp: new Date(),
      packet,
      action,
      rule
    });
  }

  // 로그 출력
  printLogs(limit: number = 10): void {
    console.log('=== Firewall Logs ===');
    const recent = this.logs.slice(-limit);
    
    for (const log of recent) {
      console.log(
        `[${log.timestamp.toISOString()}] ` +
        `${log.action} ${log.packet.protocol} ` +
        `${log.packet.source.ip}:${log.packet.source.port} -> ` +
        `${log.packet.destination.ip}:${log.packet.destination.port} ` +
        `(${log.rule ? log.rule.description || `Rule ${log.rule.id}` : 'Default Policy'})`
      );
    }
  }

  // 규칙 출력
  printRules(): void {
    console.log('=== Firewall Rules ===');
    console.log('Pri | Action | Protocol | Source         | Destination    | Direction | Description');
    console.log('─'.repeat(90));
    
    for (const rule of this.rules) {
      if (!rule.enabled) continue;
      
      console.log(
        `${rule.priority.toString().padEnd(3)} | ` +
        `${rule.action.padEnd(6)} | ` +
        `${rule.protocol.padEnd(8)} | ` +
        `${rule.source.ip}:${rule.source.port || 'ANY'}`.padEnd(14) + ' | ' +
        `${rule.destination.ip}:${rule.destination.port || 'ANY'}`.padEnd(14) + ' | ' +
        `${rule.direction.padEnd(9)} | ` +
        `${rule.description || ''}`
      );
    }
    
    console.log(`\nDefault Policy: ${this.defaultPolicy}`);
  }
}

// DNS 레코드 인터페이스
interface DNSRecord {
  name: string;
  type: 'A' | 'AAAA' | 'CNAME' | 'MX' | 'TXT' | 'NS' | 'PTR';
  value: string;
  ttl: number;
  priority?: number; // MX 레코드용
}

// DNS 캐시 엔트리
interface CacheEntry {
  record: DNSRecord;
  expires: Date;
}

// DNS 서버 구현
class DNSServer {
  private records: Map<string, DNSRecord[]> = new Map();
  private cache: Map<string, CacheEntry> = new Map();
  private forwarders: string[] = ['8.8.8.8', '8.8.4.4']; // Google DNS
  private queryLog: Array<{
    timestamp: Date;
    query: string;
    type: string;
    result: string;
    cached: boolean;
  }> = [];

  constructor() {
    this.initializeRecords();
  }

  // 기본 레코드 초기화
  private initializeRecords(): void {
    // 루트 서버
    this.addRecord({
      name: '.',
      type: 'NS',
      value: 'a.root-servers.net',
      ttl: 3600000
    });

    // 로컬 도메인
    this.addRecord({
      name: 'localhost',
      type: 'A',
      value: '127.0.0.1',
      ttl: 86400
    });

    // 예제 도메인
    this.addRecord({
      name: 'example.local',
      type: 'A',
      value: '192.168.1.100',
      ttl: 3600
    });

    this.addRecord({
      name: 'www.example.local',
      type: 'CNAME',
      value: 'example.local',
      ttl: 3600
    });
  }

  // 레코드 추가
  addRecord(record: DNSRecord): void {
    if (!this.records.has(record.name)) {
      this.records.set(record.name, []);
    }
    
    this.records.get(record.name)!.push(record);
    console.log(`DNS record added: ${record.name} ${record.type} ${record.value}`);
  }

  // DNS 조회
  async resolve(domain: string, type: DNSRecord['type'] = 'A'): Promise<string | null> {
    console.log(`DNS Query: ${domain} (${type})`);
    
    // 캐시 확인
    const cached = this.checkCache(domain, type);
    if (cached) {
      this.logQuery(domain, type, cached, true);
      return cached;
    }

    // 로컬 레코드 확인
    const local = this.lookupLocal(domain, type);
    if (local) {
      this.cacheResult(domain, type, local);
      this.logQuery(domain, type, local, false);
      return local;
    }

    // CNAME 처리
    const cname = this.lookupLocal(domain, 'CNAME');
    if (cname) {
      return this.resolve(cname, type);
    }

    // 재귀 조회 (시뮬레이션)
    const result = await this.recursiveResolve(domain, type);
    if (result) {
      this.cacheResult(domain, type, result);
      this.logQuery(domain, type, result, false);
    }
    
    return result;
  }

  // 캐시 확인
  private checkCache(domain: string, type: string): string | null {
    const key = `${domain}:${type}`;
    const entry = this.cache.get(key);
    
    if (entry && entry.expires > new Date()) {
      console.log(`Cache HIT: ${domain}`);
      return entry.record.value;
    }
    
    if (entry) {
      this.cache.delete(key);
    }
    
    return null;
  }

  // 로컬 조회
  private lookupLocal(domain: string, type: DNSRecord['type']): string | null {
    const records = this.records.get(domain);
    
    if (!records) return null;
    
    const record = records.find(r => r.type === type);
    return record ? record.value : null;
  }

  // 재귀 조회 (시뮬레이션)
  private async recursiveResolve(domain: string, type: DNSRecord['type']): Promise<string | null> {
    console.log(`Forwarding query to ${this.forwarders[0]}`);
    
    // 시뮬레이션: 실제로는 DNS 프로토콜 사용
    await new Promise(resolve => setTimeout(resolve, 100));
    
    // 시뮬레이션 결과
    if (domain === 'google.com' && type === 'A') {
      return '142.250.185.46';
    }
    if (domain === 'example.com' && type === 'A') {
      return '93.184.216.34';
    }
    
    return null;
  }

  // 결과 캐싱
  private cacheResult(domain: string, type: DNSRecord['type'], value: string, ttl: number = 3600): void {
    const key = `${domain}:${type}`;
    this.cache.set(key, {
      record: {
        name: domain,
        type,
        value,
        ttl
      },
      expires: new Date(Date.now() + ttl * 1000)
    });
  }

  // 쿼리 로깅
  private logQuery(query: string, type: string, result: string, cached: boolean): void {
    this.queryLog.push({
      timestamp: new Date(),
      query,
      type,
      result,
      cached
    });
  }

  // 역방향 조회
  async reverseLookup(ip: string): Promise<string | null> {
    // IP를 역방향 도메인으로 변환
    const parts = ip.split('.');
    const reverseDomain = `${parts.reverse().join('.')}.in-addr.arpa`;
    
    return this.resolve(reverseDomain, 'PTR');
  }

  // 캐시 정리
  cleanCache(): void {
    const now = new Date();
    let removed = 0;
    
    for (const [key, entry] of this.cache) {
      if (entry.expires < now) {
        this.cache.delete(key);
        removed++;
      }
    }
    
    console.log(`DNS cache cleaned: ${removed} entries removed`);
  }

  // 캐시 상태 출력
  printCache(): void {
    console.log('=== DNS Cache ===');
    console.log('Domain                Type  Value                TTL');
    console.log('─'.repeat(70));
    
    for (const [key, entry] of this.cache) {
      const remaining = Math.floor((entry.expires.getTime() - Date.now()) / 1000);
      console.log(
        `${entry.record.name.padEnd(20)} ` +
        `${entry.record.type.padEnd(5)} ` +
        `${entry.record.value.padEnd(20)} ` +
        `${remaining}s`
      );
    }
  }

  // 쿼리 로그 출력
  printQueryLog(limit: number = 10): void {
    console.log('=== DNS Query Log ===');
    const recent = this.queryLog.slice(-limit);
    
    for (const log of recent) {
      console.log(
        `[${log.timestamp.toISOString()}] ` +
        `${log.query} (${log.type}) -> ${log.result} ` +
        `${log.cached ? '[CACHED]' : '[RESOLVED]'}`
      );
    }
  }
}

// DNS 클라이언트 구현
class DNSClient {
  private dnsServer: string;
  private timeout: number = 5000;

  constructor(dnsServer: string = '8.8.8.8') {
    this.dnsServer = dnsServer;
  }

  // 도메인 조회
  async lookup(domain: string): Promise<string | null> {
    console.log(`Looking up ${domain} via ${this.dnsServer}`);
    
    // 로컬 hosts 파일 확인 (시뮬레이션)
    const hostsEntry = this.checkHostsFile(domain);
    if (hostsEntry) {
      return hostsEntry;
    }
    
    // DNS 서버에 쿼리
    return this.queryDNSServer(domain);
  }

  // hosts 파일 확인 (시뮬레이션)
  private checkHostsFile(domain: string): string | null {
    const hosts: { [key: string]: string } = {
      'localhost': '127.0.0.1',
      'localhost.localdomain': '127.0.0.1'
    };
    
    return hosts[domain] || null;
  }

  // DNS 서버 쿼리 (시뮬레이션)
  private async queryDNSServer(domain: string): Promise<string | null> {
    // UDP 포트 53으로 쿼리 전송 (시뮬레이션)
    await new Promise(resolve => setTimeout(resolve, 50));
    
    // 시뮬레이션 응답
    const responses: { [key: string]: string } = {
      'google.com': '142.250.185.46',
      'example.com': '93.184.216.34',
      'localhost': '127.0.0.1'
    };
    
    return responses[domain] || null;
  }
}

// 통합 보안 시스템
class NetworkSecurity {
  private firewall: Firewall;
  private dnsServer: DNSServer;

  constructor() {
    this.firewall = new Firewall('DENY');
    this.dnsServer = new DNSServer();
    this.setupSecurityPolicies();
  }

  // 보안 정책 설정
  private setupSecurityPolicies(): void {
    // HTTP/HTTPS 허용
    this.firewall.addRule({
      id: 100,
      priority: 20,
      action: 'ALLOW',
      protocol: 'TCP',
      source: { ip: 'ANY', port: 'ANY' },
      destination: { ip: 'ANY', port: 80 },
      direction: 'OUTBOUND',
      enabled: true,
      description: 'Allow HTTP'
    });

    this.firewall.addRule({
      id: 101,
      priority: 21,
      action: 'ALLOW',
      protocol: 'TCP',
      source: { ip: 'ANY', port: 'ANY' },
      destination: { ip: 'ANY', port: 443 },
      direction: 'OUTBOUND',
      enabled: true,
      description: 'Allow HTTPS'
    });

    // 악성 도메인 차단
    this.blockMaliciousDomain('malware.example.com');
    this.blockMaliciousDomain('phishing.site.com');
  }

  // 악성 도메인 차단
  private blockMaliciousDomain(domain: string): void {
    // DNS 싱크홀링
    this.dnsServer.addRecord({
      name: domain,
      type: 'A',
      value: '0.0.0.0', // 블랙홀 IP
      ttl: 86400
    });
  }

  // 패킷 검사 및 필터링
  async inspectPacket(packet: Packet): Promise<boolean> {
    // 1. 방화벽 규칙 확인
    if (!this.firewall.filterPacket(packet)) {
      console.log(`Packet blocked by firewall`);
      return false;
    }

    // 2. DNS 기반 필터링 (도메인 평판 확인)
    if (packet.destination.port === 80 || packet.destination.port === 443) {
      // IP 역방향 조회로 도메인 확인
      const domain = await this.dnsServer.reverseLookup(packet.destination.ip);
      if (domain && this.isMaliciousDomain(domain)) {
        console.log(`Packet blocked: Malicious domain ${domain}`);
        return false;
      }
    }

    return true;
  }

  // 악성 도메인 확인
  private isMaliciousDomain(domain: string): boolean {
    const blacklist = ['malware.example.com', 'phishing.site.com'];
    return blacklist.includes(domain);
  }

  // 보안 상태 리포트
  generateSecurityReport(): void {
    console.log('\n=== Network Security Report ===');
    console.log(`Generated: ${new Date().toISOString()}`);
    
    console.log('\n--- Firewall Status ---');
    this.firewall.printRules();
    
    console.log('\n--- Recent Firewall Activity ---');
    this.firewall.printLogs(5);
    
    console.log('\n--- DNS Cache Status ---');
    this.dnsServer.printCache();
    
    console.log('\n--- Recent DNS Queries ---');
    this.dnsServer.printQueryLog(5);
  }
}

// 사용 예제
async function demonstrateFirewallAndDNS() {
  console.log('=== Firewall and DNS Demonstration ===\n');
  
  // 방화벽 테스트
  console.log('--- Firewall Testing ---');
  const firewall = new Firewall('DENY');
  
  // 웹 트래픽 허용 규칙 추가
  firewall.addRule({
    id: 200,
    priority: 30,
    action: 'ALLOW',
    protocol: 'TCP',
    source: { ip: '192.168.1.0/24', port: 'ANY' },
    destination: { ip: 'ANY', port: 80 },
    direction: 'OUTBOUND',
    enabled: true,
    description: 'Allow web traffic from local network'
  });
  
  // 패킷 테스트
  const packets: Packet[] = [
    {
      protocol: 'TCP',
      source: { ip: '192.168.1.100', port: 54321 },
      destination: { ip: '93.184.216.34', port: 80 },
      direction: 'OUTBOUND',
      timestamp: new Date()
    },
    {
      protocol: 'TCP',
      source: { ip: '10.0.0.50', port: 12345 },
      destination: { ip: '192.168.1.100', port: 22 },
      direction: 'INBOUND',
      timestamp: new Date()
    },
    {
      protocol: 'UDP',
      source: { ip: '192.168.1.100', port: 54321 },
      destination: { ip: '8.8.8.8', port: 53 },
      direction: 'OUTBOUND',
      timestamp: new Date()
    }
  ];
  
  for (const packet of packets) {
    const allowed = firewall.filterPacket(packet);
    console.log(
      `Packet ${packet.protocol} ` +
      `${packet.source.ip}:${packet.source.port} -> ` +
      `${packet.destination.ip}:${packet.destination.port}: ` +
      `${allowed ? 'ALLOWED' : 'DENIED'}`
    );
  }
  
  firewall.printRules();
  
  console.log('\n--- DNS Testing ---');
  const dnsServer = new DNSServer();
  
  // DNS 조회 테스트
  const domains = [
    'localhost',
    'example.local',
    'www.example.local',
    'google.com',
    'example.com'
  ];
  
  for (const domain of domains) {
    const ip = await dnsServer.resolve(domain);
    console.log(`${domain} -> ${ip || 'Not found'}`);
  }
  
  // 캐시 상태
  dnsServer.printCache();
  
  console.log('\n--- Integrated Security System ---');
  const security = new NetworkSecurity();
  
  // 보안 검사
  const testPacket: Packet = {
    protocol: 'TCP',
    source: { ip: '192.168.1.100', port: 54321 },
    destination: { ip: '93.184.216.34', port: 443 },
    direction: 'OUTBOUND',
    timestamp: new Date()
  };
  
  const allowed = await security.inspectPacket(testPacket);
  console.log(`Security check: ${allowed ? 'PASSED' : 'BLOCKED'}`);
  
  // 보안 리포트 생성
  security.generateSecurityReport();
}

// demonstrateFirewallAndDNS().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **방화벽**: 트래픽 필터링, Stateful/Stateless, 규칙 우선순위
> 2. **DNS**: 계층 구조 (Root → TLD → Authoritative)
> 3. **DNS 캐싱**: TTL 기반, 재귀 조회 최소화
> 4. **보안 정책**: Default Deny, 최소 권한 원칙
> 5. **UDP 53번 포트**: DNS는 주로 UDP 사용 (빠른 조회)

## 참고 자료

### 📖 추가 학습
- [Firewall - Wikipedia](https://en.wikipedia.org/wiki/Firewall_(computing))
- [Domain Name System - Wikipedia](https://en.wikipedia.org/wiki/Domain_Name_System)
- [How DNS Works](https://howdns.works/)

### 🔗 관련 문서
- [[09_port-socket|포트와 소켓]]
- [[07_ip-addressing|IP 주소 체계]]
- [[08_network-components|네트워크 구성요소]]
