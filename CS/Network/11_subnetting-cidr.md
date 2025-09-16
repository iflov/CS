---
title: 서브네팅과 CIDR
created: 2025-01-03
updated: 2025-01-03
tags:
  - CS
  - Network
  - Subnetting
  - CIDR
  - IP
  - Subnet-Mask
category: Network
status: 완료
---

# 🔀 서브네팅과 CIDR (Subnetting and CIDR)

## 📌 개요

서브네팅(Subnetting)은 하나의 네트워크를 여러 개의 작은 네트워크로 분할하는 기술이며, CIDR(Classless Inter-Domain Routing)는 클래스 없이 유연하게 IP 주소를 할당하는 방법입니다. 이를 통해 IP 주소를 효율적으로 관리하고 라우팅 테이블 크기를 줄일 수 있습니다.

> [!info] 핵심 개념
> - **서브네팅**: 네트워크를 논리적으로 분할
> - **서브넷 마스크**: 네트워크와 호스트 부분 구분
> - **CIDR**: 클래스리스 주소 할당 (/표기법)
> - **VLSM**: 가변 길이 서브넷 마스킹

## 📚 목차

1. [IP 주소의 구성](#ip-주소의-구성)
2. [서브넷과 서브네팅](#서브넷과-서브네팅)
3. [CIDR 표기법](#cidr-표기법)
4. [서브넷 계산](#서브넷-계산)
5. [VLSM과 슈퍼네팅](#vlsm과-슈퍼네팅)
6. [실제 구현 예시](#실제-구현-예시)
7. [참고 자료](#참고-자료)

## IP 주소의 구성

### IPv4 기본 구조
```
IP 주소 = Network ID + Host ID

예: 192.168.1.100
    │     │
    │     └─ Host ID (특정 호스트 식별)
    └─────── Network ID (네트워크 식별)
```

### 클래스별 구분
- **Class A**: 첫 8비트가 Network ID (0으로 시작)
- **Class B**: 첫 16비트가 Network ID (10으로 시작)
- **Class C**: 첫 24비트가 Network ID (110으로 시작)
- **Class D**: 멀티캐스트 (1110으로 시작)
- **Class E**: 예약됨 (1111으로 시작)

## 서브넷과 서브네팅

### 서브네팅의 필요성
1. **효율적인 IP 관리**: 필요한 만큼만 IP 할당
2. **브로드캐스트 도메인 축소**: 네트워크 부하 감소
3. **보안 향상**: 망 분리를 통한 접근 제어
4. **라우팅 효율성**: 계층적 네트워크 구성

### 서브넷 마스크
```
IP 주소:      11000000.10101000.00000001.01100100 (192.168.1.100)
서브넷 마스크: 11111111.11111111.11111111.00000000 (255.255.255.0)
              └─────── Network ────────┘└─ Host ─┘
```

### 서브네팅 전후 비교
```
서브네팅 전 (Class B):
172.16.0.0/16 → 65,534개 호스트 (하나의 큰 네트워크)

서브네팅 후:
172.16.0.0/24  → 254개 호스트
172.16.1.0/24  → 254개 호스트
172.16.2.0/24  → 254개 호스트
... (256개의 작은 네트워크)
```

## CIDR 표기법

### CIDR란?
- **Classless Inter-Domain Routing**
- 클래스 개념 없이 네트워크 비트 수를 명시
- 슬래시 표기법 사용: `IP주소/프리픽스`

### CIDR 표기 예시
```
192.168.1.0/24
├─ IP 주소: 192.168.1.0
└─ 프리픽스: 24비트가 네트워크 부분

/24 = 255.255.255.0 (서브넷 마스크)
/25 = 255.255.255.128
/26 = 255.255.255.192
/27 = 255.255.255.224
/28 = 255.255.255.240
/29 = 255.255.255.248
/30 = 255.255.255.252
/31 = 255.255.255.254
/32 = 255.255.255.255 (단일 호스트)
```

## 서브넷 계산

### 계산 공식
- **서브넷 수**: 2^(빌린 비트 수)
- **호스트 수**: 2^(호스트 비트 수) - 2
- **블록 크기**: 256 - 서브넷 마스크의 마지막 옥텟

### 계산 예제
```
예제: 218.128.32.0/24를 60개 호스트씩 4개 서브넷으로 분할

1. 필요 호스트 비트: 2^6 = 64 > 60 ✓ (6비트 필요)
2. 네트워크 비트: 32 - 6 = 26비트 (/26)
3. 각 서브넷:
   - 218.128.32.0/26   (0-63)
   - 218.128.32.64/26  (64-127)
   - 218.128.32.128/26 (128-191)
   - 218.128.32.192/26 (192-255)
```

## VLSM과 슈퍼네팅

### VLSM (Variable Length Subnet Masking)
- 서브넷마다 다른 크기 할당
- IP 주소 낭비 최소화
- 효율적인 주소 공간 활용

### 슈퍼네팅 (Route Aggregation)
- 여러 작은 네트워크를 하나로 통합
- 라우팅 테이블 크기 감소
- CIDR의 역방향 개념

```
예: 4개의 Class C 네트워크 통합
192.168.0.0/24
192.168.1.0/24  → 192.168.0.0/22 (슈퍼넷)
192.168.2.0/24
192.168.3.0/24
```

## 실제 구현 예시

### TypeScript로 서브네팅과 CIDR 구현

```typescript
// IP 주소 유틸리티 클래스
class IPAddress {
  private octets: number[];
  
  constructor(ip: string) {
    this.octets = ip.split('.').map(n => parseInt(n));
    this.validate();
  }
  
  private validate(): void {
    if (this.octets.length !== 4) {
      throw new Error('Invalid IP address format');
    }
    for (const octet of this.octets) {
      if (octet < 0 || octet > 255) {
        throw new Error(`Invalid octet value: ${octet}`);
      }
    }
  }
  
  toInt32(): number {
    return this.octets.reduce((acc, octet, i) => {
      return acc | (octet << ((3 - i) * 8));
    }, 0) >>> 0;
  }
  
  static fromInt32(num: number): IPAddress {
    const octets = [
      (num >>> 24) & 0xFF,
      (num >>> 16) & 0xFF,
      (num >>> 8) & 0xFF,
      num & 0xFF
    ];
    return new IPAddress(octets.join('.'));
  }
  
  toString(): string {
    return this.octets.join('.');
  }
  
  toBinary(): string {
    return this.octets
      .map(o => o.toString(2).padStart(8, '0'))
      .join('.');
  }
}

// CIDR 클래스
class CIDR {
  private baseIP: IPAddress;
  private prefixLength: number;
  
  constructor(cidr: string) {
    const [ip, prefix] = cidr.split('/');
    this.baseIP = new IPAddress(ip);
    this.prefixLength = parseInt(prefix);
    
    if (this.prefixLength < 0 || this.prefixLength > 32) {
      throw new Error(`Invalid prefix length: ${this.prefixLength}`);
    }
  }
  
  getSubnetMask(): IPAddress {
    const mask = (0xFFFFFFFF << (32 - this.prefixLength)) >>> 0;
    return IPAddress.fromInt32(mask);
  }
  
  getNetworkAddress(): IPAddress {
    const ipInt = this.baseIP.toInt32();
    const maskInt = this.getSubnetMask().toInt32();
    return IPAddress.fromInt32(ipInt & maskInt);
  }
  
  getBroadcastAddress(): IPAddress {
    const networkInt = this.getNetworkAddress().toInt32();
    const hostBits = 32 - this.prefixLength;
    const broadcastInt = networkInt | ((1 << hostBits) - 1);
    return IPAddress.fromInt32(broadcastInt);
  }
  
  getFirstHost(): IPAddress {
    const networkInt = this.getNetworkAddress().toInt32();
    return IPAddress.fromInt32(networkInt + 1);
  }
  
  getLastHost(): IPAddress {
    const broadcastInt = this.getBroadcastAddress().toInt32();
    return IPAddress.fromInt32(broadcastInt - 1);
  }
  
  getTotalHosts(): number {
    return Math.pow(2, 32 - this.prefixLength);
  }
  
  getUsableHosts(): number {
    const total = this.getTotalHosts();
    return total > 2 ? total - 2 : 0;
  }
  
  contains(ip: string): boolean {
    const testIP = new IPAddress(ip);
    const testInt = testIP.toInt32();
    const networkInt = this.getNetworkAddress().toInt32();
    const broadcastInt = this.getBroadcastAddress().toInt32();
    return testInt >= networkInt && testInt <= broadcastInt;
  }
  
  getInfo(): object {
    return {
      cidr: `${this.baseIP.toString()}/${this.prefixLength}`,
      network: this.getNetworkAddress().toString(),
      broadcast: this.getBroadcastAddress().toString(),
      subnetMask: this.getSubnetMask().toString(),
      firstHost: this.getFirstHost().toString(),
      lastHost: this.getLastHost().toString(),
      totalHosts: this.getTotalHosts(),
      usableHosts: this.getUsableHosts()
    };
  }
}

// 서브네팅 계산기
class SubnetCalculator {
  // 네트워크를 동일 크기로 분할
  static divideEqually(cidr: string, subnetCount: number): CIDR[] {
    const network = new CIDR(cidr);
    const bitsNeeded = Math.ceil(Math.log2(subnetCount));
    const newPrefix = network['prefixLength'] + bitsNeeded;
    
    if (newPrefix > 30) {
      throw new Error('Cannot create subnets: prefix too long');
    }
    
    const subnets: CIDR[] = [];
    const networkInt = network.getNetworkAddress().toInt32();
    const subnetSize = Math.pow(2, 32 - newPrefix);
    
    for (let i = 0; i < Math.pow(2, bitsNeeded); i++) {
      const subnetInt = networkInt + (i * subnetSize);
      const subnetIP = IPAddress.fromInt32(subnetInt);
      subnets.push(new CIDR(`${subnetIP.toString()}/${newPrefix}`));
    }
    
    return subnets.slice(0, subnetCount);
  }
  
  // VLSM 계산
  static vlsmAllocation(
    networkCidr: string,
    requirements: Array<{name: string, hosts: number}>
  ): Array<{name: string, subnet: CIDR, allocated: number}> {
    // 요구사항을 호스트 수 기준 내림차순 정렬
    requirements.sort((a, b) => b.hosts - a.hosts);
    
    const allocations: Array<{name: string, subnet: CIDR, allocated: number}> = [];
    let currentNetwork = new CIDR(networkCidr);
    let currentIP = currentNetwork.getNetworkAddress().toInt32();
    
    for (const req of requirements) {
      // 필요한 호스트 비트 계산 (+2는 네트워크와 브로드캐스트)
      const bitsNeeded = Math.ceil(Math.log2(req.hosts + 2));
      const prefixLength = 32 - bitsNeeded;
      
      // 서브넷 생성
      const subnetIP = IPAddress.fromInt32(currentIP);
      const subnet = new CIDR(`${subnetIP.toString()}/${prefixLength}`);
      
      allocations.push({
        name: req.name,
        subnet: subnet,
        allocated: subnet.getUsableHosts()
      });
      
      // 다음 서브넷 시작 주소
      currentIP += Math.pow(2, bitsNeeded);
    }
    
    return allocations;
  }
  
  // 서브넷 요약 (슈퍼네팅)
  static summarize(cidrs: string[]): CIDR | null {
    if (cidrs.length === 0) return null;
    
    const networks = cidrs.map(c => new CIDR(c));
    const ips = networks.map(n => n.getNetworkAddress().toInt32());
    
    // 모든 IP를 이진수로 변환하여 공통 프리픽스 찾기
    let commonBits = 0;
    for (let bit = 31; bit >= 0; bit--) {
      const mask = 1 << bit;
      const firstBit = ips[0] & mask;
      
      if (ips.every(ip => (ip & mask) === firstBit)) {
        commonBits++;
      } else {
        break;
      }
    }
    
    const summaryPrefix = 32 - Math.ceil(Math.log2(
      Math.max(...ips) - Math.min(...ips) + 1
    ));
    
    const summaryIP = IPAddress.fromInt32(Math.min(...ips));
    return new CIDR(`${summaryIP.toString()}/${summaryPrefix}`);
  }
}

// 서브넷 시각화
class SubnetVisualizer {
  static visualizeBinary(cidr: string): void {
    const network = new CIDR(cidr);
    const ip = network.getNetworkAddress();
    const mask = network.getSubnetMask();
    
    console.log('=== Binary Visualization ===');
    console.log(`IP Address:    ${ip.toBinary()}`);
    console.log(`Subnet Mask:   ${mask.toBinary()}`);
    console.log(`               ${'▲'.repeat(network['prefixLength'])}${'▼'.repeat(32 - network['prefixLength'])}`);
    console.log(`               Network (${network['prefixLength']} bits) | Host (${32 - network['prefixLength']} bits)`);
  }
  
  static visualizeRange(cidr: string): void {
    const network = new CIDR(cidr);
    const info = network.getInfo() as any;
    
    console.log('\n=== IP Range Visualization ===');
    console.log('┌─────────────────────────────┐');
    console.log(`│ CIDR: ${info.cidr.padEnd(21)} │`);
    console.log('├─────────────────────────────┤');
    console.log(`│ Network:   ${info.network.padEnd(17)} │`);
    console.log(`│ First IP:  ${info.firstHost.padEnd(17)} │`);
    console.log(`│ ...        ${info.usableHosts} usable IPs`.padEnd(29) + ' │');
    console.log(`│ Last IP:   ${info.lastHost.padEnd(17)} │`);
    console.log(`│ Broadcast: ${info.broadcast.padEnd(17)} │`);
    console.log('└─────────────────────────────┘');
  }
  
  static visualizeSubnets(parentCidr: string, subnets: CIDR[]): void {
    console.log('\n=== Subnet Division ===');
    console.log(`Parent: ${parentCidr}`);
    console.log('│');
    
    subnets.forEach((subnet, i) => {
      const info = subnet.getInfo() as any;
      const isLast = i === subnets.length - 1;
      const prefix = isLast ? '└─' : '├─';
      
      console.log(`${prefix} ${info.cidr} (${info.usableHosts} hosts)`);
      
      if (!isLast) {
        console.log('│');
      }
    });
  }
}

// 실습 예제
class NetworkDesignExample {
  static designCorporateNetwork(): void {
    console.log('=== 기업 네트워크 설계 예제 ===\n');
    console.log('시나리오: 회사에 172.16.0.0/16 네트워크 할당');
    console.log('요구사항:');
    console.log('- 본사: 500명');
    console.log('- 지사1: 200명');
    console.log('- 지사2: 100명');
    console.log('- 서버팜: 50대');
    console.log('- 관리네트워크: 20대\n');
    
    const requirements = [
      { name: '본사', hosts: 500 },
      { name: '지사1', hosts: 200 },
      { name: '지사2', hosts: 100 },
      { name: '서버팜', hosts: 50 },
      { name: '관리네트워크', hosts: 20 }
    ];
    
    const allocations = SubnetCalculator.vlsmAllocation(
      '172.16.0.0/16',
      requirements
    );
    
    console.log('=== VLSM 할당 결과 ===');
    console.log('부서명         | 서브넷           | 할당 IP | 요청 IP');
    console.log('─'.repeat(60));
    
    for (const alloc of allocations) {
      const info = alloc.subnet.getInfo() as any;
      const req = requirements.find(r => r.name === alloc.name)!;
      console.log(
        `${alloc.name.padEnd(13)} | ` +
        `${info.cidr.padEnd(16)} | ` +
        `${alloc.allocated.toString().padEnd(7)} | ` +
        `${req.hosts}`
      );
    }
    
    // 할당된 서브넷 시각화
    console.log('\n=== 할당된 서브넷 상세 ===');
    for (const alloc of allocations) {
      const info = alloc.subnet.getInfo() as any;
      console.log(`\n[${alloc.name}]`);
      console.log(`  네트워크: ${info.network}`);
      console.log(`  사용범위: ${info.firstHost} - ${info.lastHost}`);
      console.log(`  서브넷마스크: ${info.subnetMask}`);
    }
  }
  
  static demonstrateSubnetting(): void {
    console.log('\n=== 서브네팅 실습 ===\n');
    
    // 기본 서브넷 정보
    const cidr = '192.168.1.0/24';
    console.log(`원본 네트워크: ${cidr}`);
    
    const network = new CIDR(cidr);
    SubnetVisualizer.visualizeBinary(cidr);
    SubnetVisualizer.visualizeRange(cidr);
    
    // 4개로 분할
    console.log('\n=== 4개 서브넷으로 분할 ===');
    const subnets = SubnetCalculator.divideEqually(cidr, 4);
    SubnetVisualizer.visualizeSubnets(cidr, subnets);
    
    // IP 포함 여부 테스트
    console.log('\n=== IP 포함 여부 테스트 ===');
    const testIPs = ['192.168.1.50', '192.168.1.100', '192.168.1.200', '192.168.2.1'];
    
    for (const ip of testIPs) {
      for (const subnet of subnets) {
        if (subnet.contains(ip)) {
          const info = subnet.getInfo() as any;
          console.log(`${ip} → ${info.cidr}`);
          break;
        }
      }
    }
  }
  
  static demonstrateSupernetting(): void {
    console.log('\n=== 슈퍼네팅 실습 ===\n');
    
    const networks = [
      '192.168.0.0/24',
      '192.168.1.0/24',
      '192.168.2.0/24',
      '192.168.3.0/24'
    ];
    
    console.log('개별 네트워크:');
    networks.forEach(n => console.log(`  - ${n}`));
    
    const summary = SubnetCalculator.summarize(networks);
    if (summary) {
      console.log(`\n요약된 네트워크: ${summary.getInfo().cidr}`);
      
      const info = summary.getInfo() as any;
      console.log(`총 IP 수: ${info.totalHosts}`);
      console.log(`범위: ${info.network} - ${info.broadcast}`);
    }
  }
}

// 메인 실행 함수
function demonstrateSubnettingAndCIDR(): void {
  console.log('=== 서브네팅과 CIDR 종합 실습 ===\n');
  
  // 1. 기업 네트워크 설계
  NetworkDesignExample.designCorporateNetwork();
  
  // 2. 서브네팅 실습
  NetworkDesignExample.demonstrateSubnetting();
  
  // 3. 슈퍼네팅 실습
  NetworkDesignExample.demonstrateSupernetting();
}

// 실행
// demonstrateSubnettingAndCIDR();
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **서브네팅**: 큰 네트워크를 작은 네트워크로 분할
> 2. **CIDR**: /표기법으로 유연한 주소 할당
> 3. **서브넷 마스크**: 네트워크와 호스트 구분
> 4. **VLSM**: 가변 길이로 효율적 IP 활용
> 5. **계산 공식**: 2^n (서브넷 수), 2^n-2 (호스트 수)

## 참고 자료

### 📖 추가 학습
- [Subnetting - Wikipedia](https://en.wikipedia.org/wiki/Subnetwork)
- [CIDR - RFC 4632](https://tools.ietf.org/html/rfc4632)
- [Visual Subnet Calculator](https://www.davidc.net/sites/default/subnets/subnets.html)
- [VLSM Tutorial](http://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/13788-3.html)

### 🔗 관련 문서
- [[07_ip-addressing|IP 주소 체계]]
- [[08_network-components|네트워크 구성요소]]
- [[10_firewall-dns|방화벽과 DNS]]
