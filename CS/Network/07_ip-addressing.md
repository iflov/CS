---
title: IP 주소 체계
created: 2024-12-31
updated: 2024-12-31
tags:
  - CS
  - Network
  - IP
  - IPv4
  - IPv6
  - Subnet
  - NAT
category: Network
status: 완료
---

# 🌐 IP 주소 체계 (IP Addressing)

## 📌 개요

IP(Internet Protocol) 주소는 네트워크상의 각 장치를 **고유하게 식별**하는 주소입니다. IPv4에서 IPv6로 전환되고 있으며, 서브넷과 NAT를 통해 효율적으로 주소를 관리합니다.

> [!info] 핵심 개념
> - **IPv4**: 32비트 주소 체계 (약 43억 개)
> - **IPv6**: 128비트 주소 체계 (거의 무한)
> - **서브넷**: 네트워크를 작은 단위로 분할
> - **NAT**: 사설 IP와 공인 IP 간 변환

## 📚 목차

1. [IPv4 주소 체계](#ipv4-주소-체계)
2. [IPv6 주소 체계](#ipv6-주소-체계)  
3. [서브넷팅 (Subnetting)](#서브넷팅-subnetting)
4. [NAT와 사설 IP](#nat와-사설-ip)
5. [IP 라우팅](#ip-라우팅)
6. [실제 구현 예시](#실제-구현-예시)
7. [참고 자료](#참고-자료)

## IPv4 주소 체계

### 구조
- **32비트** (4바이트)
- **점 십진 표기법**: `192.168.1.1`
- **총 주소 개수**: 2^32 = 4,294,967,296개
- **2011년 고갈**: 새로운 공인 IP 할당 어려움

### IP 클래스

| 클래스 | 첫 번째 옥텟 | 네트워크 비트 | 호스트 비트 | 주소 범위 | 용도 |
|--------|-------------|--------------|------------|-----------|------|
| **A** | 0-127 | 8 | 24 | 0.0.0.0 - 127.255.255.255 | 대규모 네트워크 |
| **B** | 128-191 | 16 | 16 | 128.0.0.0 - 191.255.255.255 | 중규모 네트워크 |
| **C** | 192-223 | 24 | 8 | 192.0.0.0 - 223.255.255.255 | 소규모 네트워크 |
| **D** | 224-239 | - | - | 224.0.0.0 - 239.255.255.255 | 멀티캐스트 |
| **E** | 240-255 | - | - | 240.0.0.0 - 255.255.255.255 | 예약/실험용 |

### 특수 IP 주소
- **127.0.0.1**: 루프백 (localhost)
- **0.0.0.0**: 모든 인터페이스
- **255.255.255.255**: 브로드캐스트
- **169.254.0.0/16**: Link-local (APIPA)

## IPv6 주소 체계

### 구조
- **128비트** (16바이트)
- **콜론 16진수 표기법**: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- **압축 표기**: `2001:db8:85a3::8a2e:370:7334`
- **총 주소 개수**: 2^128 ≈ 3.4 × 10^38개

### IPv6 주소 유형
- **Unicast**: 단일 인터페이스
- **Multicast**: 다중 인터페이스
- **Anycast**: 가장 가까운 인터페이스

### IPv4 vs IPv6 비교

| 특징 | IPv4 | IPv6 |
|------|------|------|
| 주소 길이 | 32 bits | 128 bits |
| 주소 표현 | 십진수 | 16진수 |
| 헤더 크기 | 20-60 bytes | 40 bytes (고정) |
| 체크섬 | 있음 | 없음 (상위 계층 의존) |
| 브로드캐스트 | 지원 | 미지원 (멀티캐스트로 대체) |
| IPSec | 선택적 | 필수 |

## 서브넷팅 (Subnetting)

### 개념
- 큰 네트워크를 **작은 네트워크로 분할**
- IP 주소의 **효율적 사용**
- 브로드캐스트 도메인 축소

### 서브넷 마스크
```
IP 주소:      192.168.1.100
서브넷 마스크: 255.255.255.0  (/24)
네트워크 부분: 192.168.1.0
호스트 부분:   0.0.0.100
```

### CIDR (Classless Inter-Domain Routing)
- 클래스 없는 주소 할당
- **슬래시 표기법**: `192.168.1.0/24`
- 유연한 서브넷 크기

## NAT와 사설 IP

### 사설 IP 대역
- **Class A**: 10.0.0.0 - 10.255.255.255 (10.0.0.0/8)
- **Class B**: 172.16.0.0 - 172.31.255.255 (172.16.0.0/12)
- **Class C**: 192.168.0.0 - 192.168.255.255 (192.168.0.0/16)

### NAT (Network Address Translation)
- **목적**: 공인 IP 절약, 보안 향상
- **방식**: 사설 IP ↔ 공인 IP 변환
- **종류**:
  - **Static NAT**: 1:1 매핑
  - **Dynamic NAT**: 동적 매핑
  - **PAT (Port Address Translation)**: 포트 기반 매핑

## IP 라우팅

### 라우팅 테이블
```
Destination     Gateway         Interface
0.0.0.0         192.168.1.1     eth0        (기본 게이트웨이)
192.168.1.0/24  0.0.0.0         eth0        (로컬 네트워크)
10.0.0.0/8      192.168.1.254   eth0        (다른 네트워크)
```

### 라우팅 과정
1. 목적지 IP 확인
2. 라우팅 테이블 조회
3. 최적 경로 선택
4. 다음 홉으로 전달

## 실제 구현 예시

### TypeScript로 IP 주소 처리 구현

```typescript
// IP 주소 유틸리티 클래스
class IPAddress {
  private octets: number[] = [];
  private version: 4 | 6;

  constructor(address: string) {
    if (address.includes('.')) {
      this.version = 4;
      this.parseIPv4(address);
    } else if (address.includes(':')) {
      this.version = 6;
      this.parseIPv6(address);
    } else {
      throw new Error('Invalid IP address format');
    }
  }

  // IPv4 파싱
  private parseIPv4(address: string): void {
    const parts = address.split('.');
    if (parts.length !== 4) {
      throw new Error('Invalid IPv4 address');
    }
    
    this.octets = parts.map(part => {
      const num = parseInt(part, 10);
      if (num < 0 || num > 255) {
        throw new Error('Invalid octet value');
      }
      return num;
    });
  }

  // IPv6 파싱 (간소화)
  private parseIPv6(address: string): void {
    // IPv6 파싱 로직 (실제로는 더 복잡)
    console.log(`Parsing IPv6: ${address}`);
  }

  // IP를 32비트 정수로 변환 (IPv4)
  toInt32(): number {
    if (this.version !== 4) {
      throw new Error('Only IPv4 addresses can be converted to Int32');
    }
    
    return (this.octets[0] << 24) |
           (this.octets[1] << 16) |
           (this.octets[2] << 8) |
           this.octets[3];
  }

  // 32비트 정수를 IP로 변환
  static fromInt32(num: number): IPAddress {
    const octets = [
      (num >>> 24) & 0xFF,
      (num >>> 16) & 0xFF,
      (num >>> 8) & 0xFF,
      num & 0xFF
    ];
    return new IPAddress(octets.join('.'));
  }

  // IP 클래스 확인 (IPv4)
  getClass(): 'A' | 'B' | 'C' | 'D' | 'E' | null {
    if (this.version !== 4) return null;
    
    const firstOctet = this.octets[0];
    if (firstOctet <= 127) return 'A';
    if (firstOctet <= 191) return 'B';
    if (firstOctet <= 223) return 'C';
    if (firstOctet <= 239) return 'D';
    return 'E';
  }

  // 사설 IP 여부 확인
  isPrivate(): boolean {
    if (this.version !== 4) return false;
    
    // 10.0.0.0/8
    if (this.octets[0] === 10) return true;
    
    // 172.16.0.0/12
    if (this.octets[0] === 172 && 
        this.octets[1] >= 16 && 
        this.octets[1] <= 31) return true;
    
    // 192.168.0.0/16
    if (this.octets[0] === 192 && 
        this.octets[1] === 168) return true;
    
    return false;
  }

  // 루프백 주소 여부
  isLoopback(): boolean {
    if (this.version === 4) {
      return this.octets[0] === 127;
    }
    // IPv6 루프백: ::1
    return false;
  }

  toString(): string {
    if (this.version === 4) {
      return this.octets.join('.');
    }
    // IPv6 문자열 변환
    return '';
  }
}

// 서브넷 계산 클래스
class SubnetCalculator {
  private ip: IPAddress;
  private prefixLength: number;

  constructor(cidr: string) {
    const [ipStr, prefix] = cidr.split('/');
    this.ip = new IPAddress(ipStr);
    this.prefixLength = parseInt(prefix, 10);
  }

  // 서브넷 마스크 생성
  getSubnetMask(): IPAddress {
    const mask = (0xFFFFFFFF << (32 - this.prefixLength)) >>> 0;
    return IPAddress.fromInt32(mask);
  }

  // 네트워크 주소 계산
  getNetworkAddress(): IPAddress {
    const ipInt = this.ip.toInt32();
    const maskInt = (0xFFFFFFFF << (32 - this.prefixLength)) >>> 0;
    const networkInt = ipInt & maskInt;
    return IPAddress.fromInt32(networkInt);
  }

  // 브로드캐스트 주소 계산
  getBroadcastAddress(): IPAddress {
    const ipInt = this.ip.toInt32();
    const maskInt = (0xFFFFFFFF << (32 - this.prefixLength)) >>> 0;
    const broadcastInt = (ipInt & maskInt) | (~maskInt >>> 0);
    return IPAddress.fromInt32(broadcastInt);
  }

  // 호스트 수 계산
  getHostCount(): number {
    return Math.pow(2, 32 - this.prefixLength) - 2; // 네트워크와 브로드캐스트 제외
  }

  // 사용 가능한 IP 범위
  getUsableRange(): { first: IPAddress; last: IPAddress } {
    const networkInt = this.getNetworkAddress().toInt32();
    const broadcastInt = this.getBroadcastAddress().toInt32();
    
    return {
      first: IPAddress.fromInt32(networkInt + 1),
      last: IPAddress.fromInt32(broadcastInt - 1)
    };
  }

  // VLSM (Variable Length Subnet Masking) 계산
  static calculateVLSM(
    networkCIDR: string,
    requirements: Array<{ name: string; hosts: number }>
  ): Array<{ name: string; subnet: string; hosts: number }> {
    // 요구사항을 호스트 수 기준으로 내림차순 정렬
    requirements.sort((a, b) => b.hosts - a.hosts);
    
    const results: Array<{ name: string; subnet: string; hosts: number }> = [];
    let currentNetwork = new SubnetCalculator(networkCIDR);
    let currentIP = currentNetwork.getNetworkAddress().toInt32();
    
    for (const req of requirements) {
      // 필요한 비트 수 계산
      const bitsNeeded = Math.ceil(Math.log2(req.hosts + 2));
      const prefixLength = 32 - bitsNeeded;
      
      // 서브넷 생성
      const subnetIP = IPAddress.fromInt32(currentIP);
      const subnet = `${subnetIP.toString()}/${prefixLength}`;
      
      results.push({
        name: req.name,
        subnet: subnet,
        hosts: Math.pow(2, bitsNeeded) - 2
      });
      
      // 다음 서브넷 시작 주소 계산
      currentIP += Math.pow(2, bitsNeeded);
    }
    
    return results;
  }
}

// NAT 구현
class NAT {
  private natTable: Map<string, string> = new Map();
  private publicIP: string;
  private portMapping: Map<number, { privateIP: string; privatePort: number }> = new Map();
  private nextPort: number = 50000;

  constructor(publicIP: string) {
    this.publicIP = publicIP;
  }

  // Static NAT (1:1 매핑)
  addStaticMapping(privateIP: string, publicIP: string): void {
    this.natTable.set(privateIP, publicIP);
    console.log(`Static NAT: ${privateIP} <-> ${publicIP}`);
  }

  // Dynamic NAT (PAT - Port Address Translation)
  translateOutgoing(privateIP: string, privatePort: number): { ip: string; port: number } {
    const mappingKey = `${privateIP}:${privatePort}`;
    
    // 기존 매핑 확인
    for (const [publicPort, mapping] of this.portMapping) {
      if (mapping.privateIP === privateIP && mapping.privatePort === privatePort) {
        return { ip: this.publicIP, port: publicPort };
      }
    }
    
    // 새 매핑 생성
    const publicPort = this.nextPort++;
    this.portMapping.set(publicPort, { privateIP, privatePort });
    
    console.log(`PAT: ${privateIP}:${privatePort} -> ${this.publicIP}:${publicPort}`);
    return { ip: this.publicIP, port: publicPort };
  }

  // Incoming 변환
  translateIncoming(publicPort: number): { ip: string; port: number } | null {
    const mapping = this.portMapping.get(publicPort);
    if (!mapping) return null;
    
    return { ip: mapping.privateIP, port: mapping.privatePort };
  }

  // NAT 테이블 출력
  printNATTable(): void {
    console.log('=== NAT Table ===');
    console.log('Static Mappings:');
    for (const [priv, pub] of this.natTable) {
      console.log(`  ${priv} <-> ${pub}`);
    }
    
    console.log('Dynamic Mappings (PAT):');
    for (const [pubPort, mapping] of this.portMapping) {
      console.log(`  ${mapping.privateIP}:${mapping.privatePort} -> ${this.publicIP}:${pubPort}`);
    }
  }
}

// 라우팅 테이블 구현
class RoutingTable {
  private routes: Array<{
    destination: string;
    mask: string;
    gateway: string;
    interface: string;
    metric: number;
  }> = [];

  // 라우트 추가
  addRoute(
    destination: string,
    mask: string,
    gateway: string,
    iface: string,
    metric: number = 1
  ): void {
    this.routes.push({
      destination,
      mask,
      gateway,
      interface: iface,
      metric
    });
    
    // 메트릭 기준으로 정렬
    this.routes.sort((a, b) => a.metric - b.metric);
  }

  // 최적 경로 찾기
  findRoute(destIP: string): { gateway: string; interface: string } | null {
    const dest = new IPAddress(destIP);
    const destInt = dest.toInt32();
    
    for (const route of this.routes) {
      const routeDest = new IPAddress(route.destination);
      const routeMask = new IPAddress(route.mask);
      const maskInt = routeMask.toInt32();
      
      // 마스크 적용하여 네트워크 비교
      if ((destInt & maskInt) === (routeDest.toInt32() & maskInt)) {
        return {
          gateway: route.gateway,
          interface: route.interface
        };
      }
    }
    
    return null;
  }

  // 라우팅 테이블 출력
  print(): void {
    console.log('=== Routing Table ===');
    console.log('Destination      Mask             Gateway         Interface  Metric');
    console.log('─'.repeat(70));
    
    for (const route of this.routes) {
      console.log(
        `${route.destination.padEnd(16)} ` +
        `${route.mask.padEnd(16)} ` +
        `${route.gateway.padEnd(16)} ` +
        `${route.interface.padEnd(10)} ` +
        `${route.metric}`
      );
    }
  }
}

// 사용 예제
function demonstrateIPAddressing() {
  console.log('=== IP Address Examples ===');
  
  // IP 주소 생성 및 분석
  const ip1 = new IPAddress('192.168.1.100');
  console.log(`IP: ${ip1.toString()}`);
  console.log(`Class: ${ip1.getClass()}`);
  console.log(`Private: ${ip1.isPrivate()}`);
  console.log(`As Int32: ${ip1.toInt32()}`);
  
  console.log('\n=== Subnet Calculation ===');
  
  // 서브넷 계산
  const subnet = new SubnetCalculator('192.168.1.0/24');
  console.log(`Network: ${subnet.getNetworkAddress().toString()}`);
  console.log(`Broadcast: ${subnet.getBroadcastAddress().toString()}`);
  console.log(`Subnet Mask: ${subnet.getSubnetMask().toString()}`);
  console.log(`Host Count: ${subnet.getHostCount()}`);
  
  const range = subnet.getUsableRange();
  console.log(`Usable Range: ${range.first.toString()} - ${range.last.toString()}`);
  
  console.log('\n=== VLSM Calculation ===');
  
  // VLSM 계산
  const vlsmRequirements = [
    { name: 'Sales', hosts: 50 },
    { name: 'Engineering', hosts: 25 },
    { name: 'HR', hosts: 10 },
    { name: 'Management', hosts: 5 }
  ];
  
  const vlsmResults = SubnetCalculator.calculateVLSM('192.168.1.0/24', vlsmRequirements);
  for (const result of vlsmResults) {
    console.log(`${result.name}: ${result.subnet} (${result.hosts} hosts)`);
  }
  
  console.log('\n=== NAT Example ===');
  
  // NAT 설정
  const nat = new NAT('203.0.113.1');
  
  // Static NAT
  nat.addStaticMapping('192.168.1.10', '203.0.113.10');
  
  // Dynamic NAT (PAT)
  const translated1 = nat.translateOutgoing('192.168.1.100', 80);
  const translated2 = nat.translateOutgoing('192.168.1.101', 443);
  
  nat.printNATTable();
  
  console.log('\n=== Routing Table ===');
  
  // 라우팅 테이블
  const routingTable = new RoutingTable();
  routingTable.addRoute('0.0.0.0', '0.0.0.0', '192.168.1.1', 'eth0', 10);
  routingTable.addRoute('192.168.1.0', '255.255.255.0', '0.0.0.0', 'eth0', 1);
  routingTable.addRoute('10.0.0.0', '255.0.0.0', '192.168.1.254', 'eth0', 5);
  
  routingTable.print();
  
  // 라우팅 테스트
  const route = routingTable.findRoute('8.8.8.8');
  if (route) {
    console.log(`\nRoute for 8.8.8.8: via ${route.gateway} on ${route.interface}`);
  }
}

// demonstrateIPAddressing();
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **IPv4**: 32비트, 고갈 상태, NAT로 연명
> 2. **IPv6**: 128비트, 충분한 주소 공간
> 3. **서브넷팅**: 효율적인 IP 관리와 네트워크 분할
> 4. **NAT**: 사설 IP로 공인 IP 절약
> 5. **라우팅**: 최적 경로로 패킷 전달

## 참고 자료

### 📖 추가 학습
- [IP Address - Wikipedia](https://en.wikipedia.org/wiki/IP_address)
- [IPv6 - Wikipedia](https://en.wikipedia.org/wiki/IPv6)
- [Subnet Calculator](https://www.subnet-calculator.com/)

### 🔗 관련 문서
- [[08_network-components|네트워크 구성요소]]
- [[03_tcp-ip-model|TCP/IP 4계층 모델]]
- [[10_firewall-dns|방화벽과 DNS]]
