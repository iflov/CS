---
title: 네트워크 구성요소
created: 2024-12-31
updated: 2024-12-31
tags:
  - CS
  - Network
  - Router
  - Switch
  - Hub
  - ARP
  - MAC
category: Network
status: 완료
---

# 🔧 네트워크 구성요소 (Network Components)

## 📌 개요

네트워크는 다양한 하드웨어와 소프트웨어 구성요소들이 유기적으로 연결되어 동작합니다. 각 구성요소는 OSI 계층에서 특정 역할을 수행하며, 데이터가 목적지까지 전달될 수 있도록 합니다.

> [!info] 핵심 개념
> - **물리적 장비**: 허브, 스위치, 라우터, 공유기
> - **주소 체계**: MAC Address (L2), IP Address (L3)
> - **프로토콜**: ARP, DHCP
> - **계층별 동작**: 각 장비가 동작하는 OSI 계층

## 📚 목차

1. [네트워크 장비](#네트워크-장비)
2. [MAC Address](#mac-address)
3. [ARP (Address Resolution Protocol)](#arp-address-resolution-protocol)
4. [공유기와 NAT](#공유기와-nat)
5. [실제 구현 예시](#실제-구현-예시)
6. [참고 자료](#참고-자료)

## 네트워크 장비

### 1️⃣ 허브 (Hub) - L1 Physical Layer
- **역할**: 전기 신호 중계
- **특징**: 
  - 모든 포트에 신호 전달 (브로드캐스트)
  - 충돌 도메인 공유
  - 보안 취약
- **현재**: 거의 사용 안 함 (스위치로 대체)

### 2️⃣ 스위치 (Switch) - L2 Data Link Layer
- **역할**: MAC 주소 기반 프레임 전달
- **특징**:
  - MAC 주소 테이블 학습
  - 충돌 도메인 분리
  - 전이중 통신 (Full-Duplex)
- **종류**:
  - **Unmanaged**: 기본 기능만
  - **Managed**: VLAN, QoS, 포트 미러링 등
  - **L3 Switch**: 라우팅 기능 포함

### 3️⃣ 라우터 (Router) - L3 Network Layer
- **역할**: IP 주소 기반 패킷 라우팅
- **특징**:
  - 라우팅 테이블 관리
  - 서로 다른 네트워크 연결
  - 브로드캐스트 도메인 분리
- **기능**:
  - 경로 결정
  - 패킷 필터링
  - NAT, DHCP

### 4️⃣ 브리지 (Bridge) - L2 Data Link Layer
- **역할**: 두 네트워크 세그먼트 연결
- **특징**:
  - MAC 주소 필터링
  - 충돌 도메인 분리
- **현재**: 스위치에 통합됨

### 장비별 비교

| 장비 | OSI 계층 | 주소 | 기능 | 충돌 도메인 |
|------|---------|------|------|------------|
| 허브 | L1 | 없음 | 신호 증폭 | 공유 |
| 스위치 | L2 | MAC | 프레임 교환 | 분리 |
| 라우터 | L3 | IP | 패킷 라우팅 | 분리 |
| L3 스위치 | L2/L3 | MAC/IP | 스위칭 + 라우팅 | 분리 |

## MAC Address

### 구조
- **48비트** (6바이트)
- **표기법**: `AA:BB:CC:DD:EE:FF`
- **구성**:
  - 상위 24비트: OUI (제조사 코드)
  - 하위 24비트: 고유 번호

### 특징
- **물리적 주소**: NIC에 고정
- **전역적 유일성**: 중복 없음 (이론상)
- **변경 불가**: 하드웨어에 내장 (소프트웨어로 스푸핑은 가능)

### 특수 MAC 주소
- **FF:FF:FF:FF:FF:FF**: 브로드캐스트
- **01:00:5E:xx:xx:xx**: IPv4 멀티캐스트
- **33:33:xx:xx:xx:xx**: IPv6 멀티캐스트

## ARP (Address Resolution Protocol)

### 동작 과정
1. **ARP Request**: "이 IP를 가진 장치의 MAC 주소는?"
2. **브로드캐스트**: 네트워크 전체에 전송
3. **ARP Reply**: 해당 장치가 자신의 MAC 응답
4. **ARP Cache**: 결과를 캐시에 저장

### ARP 테이블
```
IP Address      MAC Address        Type
192.168.1.1     AA:BB:CC:DD:EE:FF  dynamic
192.168.1.100   11:22:33:44:55:66  dynamic
192.168.1.254   FF:EE:DD:CC:BB:AA  static
```

## 공유기와 NAT

### 공유기 (Home Router)
- **통합 기능**:
  - 라우터: 라우팅
  - 스위치: 다중 이더넷 포트
  - AP: 무선 접속점
  - 방화벽: 보안
  - DHCP 서버: IP 자동 할당
  - NAT: 주소 변환

### DHCP (Dynamic Host Configuration Protocol)
- **자동 설정**:
  - IP 주소
  - 서브넷 마스크
  - 기본 게이트웨이
  - DNS 서버
- **과정**: DORA (Discover, Offer, Request, Acknowledge)

## 실제 구현 예시

### TypeScript로 네트워크 구성요소 구현

```typescript
// MAC 주소 클래스
class MACAddress {
  private octets: number[];

  constructor(mac: string) {
    this.octets = mac.split(':').map(oct => parseInt(oct, 16));
    if (this.octets.length !== 6 || this.octets.some(oct => isNaN(oct))) {
      throw new Error('Invalid MAC address format');
    }
  }

  toString(): string {
    return this.octets
      .map(oct => oct.toString(16).padStart(2, '0').toUpperCase())
      .join(':');
  }

  isBroadcast(): boolean {
    return this.octets.every(oct => oct === 0xFF);
  }

  isMulticast(): boolean {
    return (this.octets[0] & 0x01) === 0x01;
  }

  getOUI(): string {
    return this.octets.slice(0, 3)
      .map(oct => oct.toString(16).padStart(2, '0').toUpperCase())
      .join(':');
  }
}

// ARP 테이블 구현
class ARPTable {
  private entries: Map<string, {
    mac: MACAddress;
    type: 'static' | 'dynamic';
    timestamp: Date;
    ttl: number; // seconds
  }> = new Map();

  // ARP 엔트리 추가
  addEntry(
    ip: string,
    mac: MACAddress,
    type: 'static' | 'dynamic' = 'dynamic'
  ): void {
    this.entries.set(ip, {
      mac,
      type,
      timestamp: new Date(),
      ttl: type === 'static' ? Infinity : 120 // 2분
    });
    console.log(`ARP: ${ip} -> ${mac.toString()} (${type})`);
  }

  // MAC 주소 조회
  lookup(ip: string): MACAddress | null {
    const entry = this.entries.get(ip);
    
    if (!entry) return null;
    
    // TTL 확인
    if (entry.type === 'dynamic') {
      const age = (Date.now() - entry.timestamp.getTime()) / 1000;
      if (age > entry.ttl) {
        this.entries.delete(ip);
        return null;
      }
    }
    
    return entry.mac;
  }

  // ARP Request 시뮬레이션
  async request(ip: string): Promise<MACAddress | null> {
    console.log(`ARP Request: Who has ${ip}? Tell ${this.getMyIP()}`);
    
    // 브로드캐스트 시뮬레이션
    await this.broadcast(`ARP_REQUEST:${ip}`);
    
    // 응답 대기 (시뮬레이션)
    const mac = await this.waitForReply(ip);
    
    if (mac) {
      this.addEntry(ip, mac, 'dynamic');
      return mac;
    }
    
    return null;
  }

  private async broadcast(message: string): Promise<void> {
    console.log(`Broadcasting: ${message}`);
    await new Promise(resolve => setTimeout(resolve, 100));
  }

  private async waitForReply(ip: string): Promise<MACAddress | null> {
    // 시뮬레이션: 랜덤 MAC 생성
    await new Promise(resolve => setTimeout(resolve, 200));
    const randomMAC = Array.from({ length: 6 }, () => 
      Math.floor(Math.random() * 256).toString(16).padStart(2, '0')
    ).join(':');
    
    console.log(`ARP Reply: ${ip} is at ${randomMAC}`);
    return new MACAddress(randomMAC);
  }

  private getMyIP(): string {
    return '192.168.1.100'; // 시뮬레이션
  }

  // 테이블 출력
  print(): void {
    console.log('=== ARP Table ===');
    console.log('IP Address      MAC Address         Type     Age');
    console.log('─'.repeat(55));
    
    for (const [ip, entry] of this.entries) {
      const age = Math.floor((Date.now() - entry.timestamp.getTime()) / 1000);
      console.log(
        `${ip.padEnd(15)} ${entry.mac.toString().padEnd(19)} ` +
        `${entry.type.padEnd(8)} ${age}s`
      );
    }
  }
}

// 스위치 구현
class Switch {
  private macTable: Map<string, number> = new Map(); // MAC -> Port
  private ports: Array<{ device: string; mac: MACAddress }> = [];
  private vlanConfig: Map<number, Set<number>> = new Map(); // VLAN ID -> Ports

  constructor(private portCount: number) {
    for (let i = 0; i < portCount; i++) {
      this.ports[i] = { device: '', mac: new MACAddress('00:00:00:00:00:00') };
    }
  }

  // 장치 연결
  connect(port: number, device: string, mac: MACAddress): void {
    if (port >= this.portCount) {
      throw new Error(`Invalid port: ${port}`);
    }
    
    this.ports[port] = { device, mac };
    this.macTable.set(mac.toString(), port);
    console.log(`Device ${device} connected to port ${port}`);
  }

  // 프레임 전달
  forward(frame: {
    source: MACAddress;
    destination: MACAddress;
    data: Buffer;
  }): void {
    // MAC 테이블 학습
    const sourcePort = this.getPortByMAC(frame.source);
    if (sourcePort !== null) {
      this.macTable.set(frame.source.toString(), sourcePort);
    }
    
    // 목적지 확인
    if (frame.destination.isBroadcast()) {
      // 브로드캐스트: 모든 포트에 전달 (소스 제외)
      this.broadcast(frame, sourcePort);
    } else if (frame.destination.isMulticast()) {
      // 멀티캐스트: 그룹 멤버에게만 전달
      this.multicast(frame, sourcePort);
    } else {
      // 유니캐스트
      const destPort = this.macTable.get(frame.destination.toString());
      if (destPort !== undefined) {
        // 알려진 목적지: 특정 포트로 전달
        this.unicast(frame, destPort);
      } else {
        // 알 수 없는 목적지: 플러딩 (브로드캐스트)
        console.log(`Unknown destination, flooding...`);
        this.broadcast(frame, sourcePort);
      }
    }
  }

  private getPortByMAC(mac: MACAddress): number | null {
    for (let i = 0; i < this.ports.length; i++) {
      if (this.ports[i].mac.toString() === mac.toString()) {
        return i;
      }
    }
    return null;
  }

  private broadcast(
    frame: { source: MACAddress; destination: MACAddress; data: Buffer },
    excludePort: number | null
  ): void {
    for (let i = 0; i < this.ports.length; i++) {
      if (i !== excludePort && this.ports[i].device) {
        console.log(`Forwarding to port ${i} (${this.ports[i].device})`);
      }
    }
  }

  private multicast(
    frame: { source: MACAddress; destination: MACAddress; data: Buffer },
    sourcePort: number | null
  ): void {
    // IGMP Snooping 시뮬레이션
    console.log(`Multicast to group ${frame.destination.toString()}`);
  }

  private unicast(
    frame: { source: MACAddress; destination: MACAddress; data: Buffer },
    port: number
  ): void {
    console.log(`Forwarding to port ${port} (${this.ports[port].device})`);
  }

  // VLAN 설정
  configureVLAN(vlanId: number, ports: number[]): void {
    this.vlanConfig.set(vlanId, new Set(ports));
    console.log(`VLAN ${vlanId} configured on ports: ${ports.join(', ')}`);
  }

  // MAC 테이블 출력
  printMACTable(): void {
    console.log('=== MAC Address Table ===');
    console.log('MAC Address         Port  Device');
    console.log('─'.repeat(45));
    
    for (const [mac, port] of this.macTable) {
      const device = this.ports[port].device;
      console.log(`${mac.padEnd(19)} ${port.toString().padEnd(5)} ${device}`);
    }
  }
}

// 라우터 구현
class Router {
  private interfaces: Map<string, {
    ip: string;
    mac: MACAddress;
    network: string;
  }> = new Map();
  
  private routingTable: Array<{
    destination: string;
    gateway: string;
    interface: string;
    metric: number;
  }> = [];
  
  private arpTable: ARPTable = new ARPTable();

  // 인터페이스 설정
  configureInterface(name: string, ip: string, network: string): void {
    const mac = this.generateMAC();
    this.interfaces.set(name, { ip, mac, network });
    console.log(`Interface ${name}: IP=${ip}, MAC=${mac.toString()}`);
  }

  // 라우팅 테이블에 경로 추가
  addRoute(
    destination: string,
    gateway: string,
    iface: string,
    metric: number = 1
  ): void {
    this.routingTable.push({ destination, gateway, interface: iface, metric });
    this.routingTable.sort((a, b) => a.metric - b.metric);
  }

  // 패킷 라우팅
  async route(packet: {
    source: string;
    destination: string;
    ttl: number;
    data: Buffer;
  }): Promise<void> {
    console.log(`Routing packet from ${packet.source} to ${packet.destination}`);
    
    // TTL 확인
    if (packet.ttl <= 0) {
      console.log('TTL exceeded, dropping packet');
      return;
    }
    packet.ttl--;
    
    // 라우팅 테이블 조회
    const route = this.findRoute(packet.destination);
    if (!route) {
      console.log('No route to host');
      return;
    }
    
    // ARP로 다음 홉의 MAC 주소 확인
    let nextHopMAC = this.arpTable.lookup(route.gateway);
    if (!nextHopMAC) {
      nextHopMAC = await this.arpTable.request(route.gateway);
    }
    
    if (nextHopMAC) {
      console.log(`Forwarding to ${route.gateway} via ${route.interface}`);
      this.transmit(packet, nextHopMAC, route.interface);
    }
  }

  private findRoute(destination: string): {
    destination: string;
    gateway: string;
    interface: string;
    metric: number;
  } | null {
    // 최적 경로 찾기 (간소화)
    for (const route of this.routingTable) {
      if (this.matchesNetwork(destination, route.destination)) {
        return route;
      }
    }
    
    // 기본 경로
    return this.routingTable.find(r => r.destination === '0.0.0.0/0') || null;
  }

  private matchesNetwork(ip: string, network: string): boolean {
    // 네트워크 매칭 로직 (간소화)
    return ip.startsWith(network.split('/')[0].split('.').slice(0, 3).join('.'));
  }

  private transmit(
    packet: any,
    nextHopMAC: MACAddress,
    iface: string
  ): void {
    const interfaceInfo = this.interfaces.get(iface);
    if (interfaceInfo) {
      console.log(
        `Transmitting: ${interfaceInfo.mac.toString()} -> ` +
        `${nextHopMAC.toString()} on ${iface}`
      );
    }
  }

  private generateMAC(): MACAddress {
    const octets = Array.from({ length: 6 }, () =>
      Math.floor(Math.random() * 256).toString(16).padStart(2, '0')
    );
    return new MACAddress(octets.join(':'));
  }

  // 라우팅 테이블 출력
  printRoutingTable(): void {
    console.log('=== Routing Table ===');
    console.log('Destination      Gateway         Interface  Metric');
    console.log('─'.repeat(55));
    
    for (const route of this.routingTable) {
      console.log(
        `${route.destination.padEnd(16)} ${route.gateway.padEnd(16)} ` +
        `${route.interface.padEnd(10)} ${route.metric}`
      );
    }
  }
}

// DHCP 서버 구현
class DHCPServer {
  private pool: { start: string; end: string };
  private leases: Map<string, {
    ip: string;
    mac: MACAddress;
    expires: Date;
  }> = new Map();
  
  private config = {
    subnet: '192.168.1.0',
    mask: '255.255.255.0',
    gateway: '192.168.1.1',
    dns: ['8.8.8.8', '8.8.4.4'],
    leaseTime: 86400 // 24 hours
  };

  constructor(poolStart: string, poolEnd: string) {
    this.pool = { start: poolStart, end: poolEnd };
  }

  // DHCP DORA 프로세스
  async handleDHCP(message: {
    type: 'DISCOVER' | 'REQUEST' | 'RELEASE' | 'RENEW';
    mac: MACAddress;
    requestedIP?: string;
  }): Promise<void> {
    switch (message.type) {
      case 'DISCOVER':
        await this.handleDiscover(message.mac);
        break;
      case 'REQUEST':
        await this.handleRequest(message.mac, message.requestedIP);
        break;
      case 'RELEASE':
        this.handleRelease(message.mac);
        break;
      case 'RENEW':
        await this.handleRenew(message.mac);
        break;
    }
  }

  private async handleDiscover(mac: MACAddress): Promise<void> {
    console.log(`DHCP DISCOVER from ${mac.toString()}`);
    
    const offeredIP = this.findAvailableIP();
    if (offeredIP) {
      console.log(`DHCP OFFER: ${offeredIP} to ${mac.toString()}`);
      // 임시 예약
      this.reserveIP(offeredIP, mac);
    } else {
      console.log('DHCP: No available IP addresses');
    }
  }

  private async handleRequest(
    mac: MACAddress,
    requestedIP?: string
  ): Promise<void> {
    console.log(`DHCP REQUEST from ${mac.toString()} for ${requestedIP}`);
    
    if (requestedIP && this.isAvailable(requestedIP)) {
      // IP 할당
      this.leases.set(mac.toString(), {
        ip: requestedIP,
        mac: mac,
        expires: new Date(Date.now() + this.config.leaseTime * 1000)
      });
      
      console.log(`DHCP ACK: ${requestedIP} assigned to ${mac.toString()}`);
      console.log(`  Gateway: ${this.config.gateway}`);
      console.log(`  DNS: ${this.config.dns.join(', ')}`);
      console.log(`  Lease: ${this.config.leaseTime}s`);
    } else {
      console.log('DHCP NAK: Requested IP not available');
    }
  }

  private handleRelease(mac: MACAddress): void {
    const lease = this.leases.get(mac.toString());
    if (lease) {
      console.log(`DHCP RELEASE: ${lease.ip} from ${mac.toString()}`);
      this.leases.delete(mac.toString());
    }
  }

  private async handleRenew(mac: MACAddress): Promise<void> {
    const lease = this.leases.get(mac.toString());
    if (lease) {
      lease.expires = new Date(Date.now() + this.config.leaseTime * 1000);
      console.log(`DHCP RENEW: ${lease.ip} for ${mac.toString()}`);
    }
  }

  private findAvailableIP(): string | null {
    const [start1, start2, start3, start4] = this.pool.start.split('.').map(Number);
    const [end1, end2, end3, end4] = this.pool.end.split('.').map(Number);
    
    for (let i = start4; i <= end4; i++) {
      const ip = `${start1}.${start2}.${start3}.${i}`;
      if (this.isAvailable(ip)) {
        return ip;
      }
    }
    
    return null;
  }

  private isAvailable(ip: string): boolean {
    for (const [_, lease] of this.leases) {
      if (lease.ip === ip && lease.expires > new Date()) {
        return false;
      }
    }
    return true;
  }

  private reserveIP(ip: string, mac: MACAddress): void {
    // 임시 예약 (OFFER 상태)
    console.log(`Reserving ${ip} for ${mac.toString()}`);
  }

  // DHCP 리스 출력
  printLeases(): void {
    console.log('=== DHCP Leases ===');
    console.log('IP Address      MAC Address         Expires');
    console.log('─'.repeat(60));
    
    for (const [_, lease] of this.leases) {
      const remaining = Math.floor(
        (lease.expires.getTime() - Date.now()) / 1000
      );
      console.log(
        `${lease.ip.padEnd(15)} ${lease.mac.toString().padEnd(19)} ` +
        `${remaining}s`
      );
    }
  }
}

// 네트워크 시뮬레이션
async function simulateNetwork() {
  console.log('=== Network Component Simulation ===\n');
  
  // MAC 주소 테스트
  const mac1 = new MACAddress('AA:BB:CC:DD:EE:FF');
  console.log(`MAC: ${mac1.toString()}`);
  console.log(`OUI: ${mac1.getOUI()}`);
  console.log(`Broadcast: ${mac1.isBroadcast()}`);
  
  console.log('\n=== ARP Simulation ===');
  const arp = new ARPTable();
  await arp.request('192.168.1.1');
  await arp.request('192.168.1.254');
  arp.print();
  
  console.log('\n=== Switch Simulation ===');
  const switch1 = new Switch(8);
  switch1.connect(0, 'PC-1', new MACAddress('11:11:11:11:11:11'));
  switch1.connect(1, 'PC-2', new MACAddress('22:22:22:22:22:22'));
  switch1.connect(2, 'Server', new MACAddress('33:33:33:33:33:33'));
  
  // 프레임 전달 테스트
  switch1.forward({
    source: new MACAddress('11:11:11:11:11:11'),
    destination: new MACAddress('33:33:33:33:33:33'),
    data: Buffer.from('Hello Server')
  });
  
  switch1.printMACTable();
  
  console.log('\n=== Router Simulation ===');
  const router = new Router();
  router.configureInterface('eth0', '192.168.1.1', '192.168.1.0/24');
  router.configureInterface('eth1', '10.0.0.1', '10.0.0.0/8');
  
  router.addRoute('0.0.0.0/0', '192.168.1.254', 'eth0', 10);
  router.addRoute('192.168.1.0/24', '0.0.0.0', 'eth0', 1);
  router.addRoute('10.0.0.0/8', '0.0.0.0', 'eth1', 1);
  
  router.printRoutingTable();
  
  // 패킷 라우팅 테스트
  await router.route({
    source: '192.168.1.100',
    destination: '8.8.8.8',
    ttl: 64,
    data: Buffer.from('DNS Query')
  });
  
  console.log('\n=== DHCP Simulation ===');
  const dhcp = new DHCPServer('192.168.1.100', '192.168.1.200');
  
  const clientMAC = new MACAddress('AA:AA:AA:AA:AA:AA');
  await dhcp.handleDHCP({ type: 'DISCOVER', mac: clientMAC });
  await dhcp.handleDHCP({ 
    type: 'REQUEST', 
    mac: clientMAC,
    requestedIP: '192.168.1.100'
  });
  
  dhcp.printLeases();
}

// simulateNetwork().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **계층별 장비**: 허브(L1), 스위치(L2), 라우터(L3)
> 2. **MAC Address**: 물리적 주소, NIC에 고정
> 3. **ARP**: IP를 MAC으로 변환
> 4. **공유기**: 라우터 + 스위치 + AP + NAT + DHCP
> 5. **DHCP**: IP 자동 할당 (DORA 과정)

## 참고 자료

### 📖 추가 학습
- [Network Switch - Wikipedia](https://en.wikipedia.org/wiki/Network_switch)
- [Router - Wikipedia](https://en.wikipedia.org/wiki/Router_(computing))
- [ARP - Wikipedia](https://ko.wikipedia.org/wiki/주소_결정_프로토콜)

### 🔗 관련 문서
- [[07_ip-addressing|IP 주소 체계]]
- [[01_network-basics|네트워크 기초]]
- [[09_port-socket|포트와 소켓]]
