---
title: 포트와 소켓
created: 2024-12-31
updated: 2024-12-31
tags:
  - CS
  - Network
  - Port
  - Socket
  - TCP
  - UDP
category: Network
status: 완료
---

# 🔌 포트와 소켓 (Port and Socket)

## 📌 개요

포트(Port)는 네트워크 통신의 **논리적 종단점**이며, 소켓(Socket)은 프로세스가 네트워크 통신을 위해 사용하는 **인터페이스**입니다. IP 주소가 컴퓨터를 찾는다면, 포트는 그 컴퓨터 내의 특정 프로세스를 찾습니다.

> [!info] 핵심 개념
> - **포트**: 0-65535 범위의 논리적 주소
> - **소켓**: IP + 포트 조합의 통신 엔드포인트
> - **Well-known Ports**: 0-1023 (시스템 예약)
> - **소켓 프로그래밍**: 네트워크 애플리케이션 개발 기법

## 📚 목차

1. [포트 (Port)](#포트-port)
2. [소켓 (Socket)](#소켓-socket)
3. [포트와 소켓의 관계](#포트와-소켓의-관계)
4. [소켓 프로그래밍](#소켓-프로그래밍)
5. [실제 구현 예시](#실제-구현-예시)
6. [참고 자료](#참고-자료)

## 포트 (Port)

### 포트 범위
- **총 범위**: 0 - 65535 (2^16 - 1)
- **Well-known Ports**: 0 - 1023 (시스템/관리자 권한 필요)
- **Registered Ports**: 1024 - 49151 (등록된 서비스)
- **Dynamic/Private Ports**: 49152 - 65535 (임시 할당)

### 주요 Well-known Ports

| 포트 | 프로토콜 | 서비스 | 설명 |
|------|---------|--------|------|
| 20/21 | TCP | FTP | 파일 전송 (데이터/제어) |
| 22 | TCP | SSH | 보안 셸 |
| 23 | TCP | Telnet | 원격 접속 |
| 25 | TCP | SMTP | 메일 전송 |
| 53 | TCP/UDP | DNS | 도메인 이름 해석 |
| 67/68 | UDP | DHCP | IP 자동 할당 |
| 80 | TCP | HTTP | 웹 서비스 |
| 110 | TCP | POP3 | 메일 수신 |
| 143 | TCP | IMAP | 메일 수신 |
| 443 | TCP | HTTPS | 보안 웹 서비스 |
| 3306 | TCP | MySQL | 데이터베이스 |
| 3389 | TCP | RDP | 원격 데스크톱 |
| 5432 | TCP | PostgreSQL | 데이터베이스 |
| 6379 | TCP | Redis | 캐시 서버 |
| 8080 | TCP | HTTP-Alt | 대체 웹 포트 |
| 27017 | TCP | MongoDB | NoSQL 데이터베이스 |

### 포트의 특징
- **유일성**: 동일 시점에 하나의 프로세스만 특정 포트 사용
- **양방향**: 송신/수신 모두 가능
- **프로토콜별 독립**: TCP 80번과 UDP 80번은 별개

## 소켓 (Socket)

### 소켓의 정의
- **네트워크 소켓**: 프로세스 간 통신(IPC)의 종단점
- **구성**: `<Protocol, Local IP, Local Port, Remote IP, Remote Port>`
- **유일성**: 5-tuple로 고유하게 식별

### 소켓 타입

#### 1. TCP 소켓 (SOCK_STREAM)
- 연결 지향
- 신뢰성 보장
- 순서 보장
- 양방향 통신

#### 2. UDP 소켓 (SOCK_DGRAM)
- 비연결성
- 신뢰성 미보장
- 빠른 전송
- 브로드캐스트/멀티캐스트 지원

#### 3. Raw 소켓 (SOCK_RAW)
- 프로토콜 헤더 직접 제어
- 관리자 권한 필요
- 패킷 스니핑, 커스텀 프로토콜

## 포트와 소켓의 관계

```
포트: 문의 번호 (아파트 호수)
소켓: 실제 열린 문 (거주자가 사용 중인 호실)

IP 주소 + 포트 번호 = 소켓 주소
192.168.1.100:80 = 웹 서버 소켓 주소
```

### 관계 다이어그램
```
Application Layer
    ↓
Process A  Process B  Process C
    ↓         ↓         ↓
Socket 1   Socket 2   Socket 3
    ↓         ↓         ↓
Port 80    Port 443   Port 3000
    ↓         ↓         ↓
    Transport Layer (TCP/UDP)
           ↓
      Network Layer (IP)
```

### 비유로 이해하기
- **IP 주소**: 아파트 건물 주소
- **포트**: 호실 번호
- **소켓**: 실제로 사람이 살고 있는 호실
- **패킷**: 우편물

## 소켓 프로그래밍

### 소켓 생명주기

#### TCP 서버
1. `socket()` - 소켓 생성
2. `bind()` - 포트 바인딩
3. `listen()` - 연결 대기
4. `accept()` - 연결 수락
5. `send()/recv()` - 데이터 송수신
6. `close()` - 소켓 닫기

#### TCP 클라이언트
1. `socket()` - 소켓 생성
2. `connect()` - 서버 연결
3. `send()/recv()` - 데이터 송수신
4. `close()` - 소켓 닫기

#### UDP 통신
1. `socket()` - 소켓 생성
2. `bind()` - 포트 바인딩 (선택적)
3. `sendto()/recvfrom()` - 데이터 송수신
4. `close()` - 소켓 닫기

## 실제 구현 예시

### TypeScript로 포트와 소켓 구현

```typescript
import * as net from 'net';
import * as dgram from 'dgram';
import { EventEmitter } from 'events';

// 포트 관리 클래스
class PortManager {
  private usedPorts: Set<number> = new Set();
  private reservedPorts: Map<number, string> = new Map();

  constructor() {
    this.initializeWellKnownPorts();
  }

  private initializeWellKnownPorts(): void {
    const wellKnownPorts: Array<[number, string]> = [
      [20, 'FTP-DATA'],
      [21, 'FTP'],
      [22, 'SSH'],
      [23, 'TELNET'],
      [25, 'SMTP'],
      [53, 'DNS'],
      [80, 'HTTP'],
      [110, 'POP3'],
      [143, 'IMAP'],
      [443, 'HTTPS'],
      [3306, 'MySQL'],
      [3389, 'RDP'],
      [5432, 'PostgreSQL'],
      [6379, 'Redis'],
      [8080, 'HTTP-ALT'],
      [27017, 'MongoDB']
    ];
    
    for (const [port, service] of wellKnownPorts) {
      this.reservedPorts.set(port, service);
    }
  }

  // 포트 할당
  allocatePort(port?: number): number | null {
    if (port) {
      // 특정 포트 요청
      if (this.isAvailable(port)) {
        this.usedPorts.add(port);
        console.log(`Port ${port} allocated`);
        return port;
      }
      console.log(`Port ${port} is not available`);
      return null;
    } else {
      // 동적 포트 할당
      return this.allocateDynamicPort();
    }
  }

  // 동적 포트 할당
  private allocateDynamicPort(): number {
    const minDynamic = 49152;
    const maxDynamic = 65535;
    
    for (let port = minDynamic; port <= maxDynamic; port++) {
      if (this.isAvailable(port)) {
        this.usedPorts.add(port);
        console.log(`Dynamic port ${port} allocated`);
        return port;
      }
    }
    
    throw new Error('No available dynamic ports');
  }

  // 포트 해제
  releasePort(port: number): void {
    this.usedPorts.delete(port);
    console.log(`Port ${port} released`);
  }

  // 포트 사용 가능 여부
  isAvailable(port: number): boolean {
    if (port < 0 || port > 65535) return false;
    if (this.usedPorts.has(port)) return false;
    if (port < 1024 && !this.hasPrivilege()) return false;
    return true;
  }

  // 권한 확인 (시뮬레이션)
  private hasPrivilege(): boolean {
    return process.getuid ? process.getuid() === 0 : false;
  }

  // Well-known 포트 조회
  getServiceByPort(port: number): string | undefined {
    return this.reservedPorts.get(port);
  }

  // 포트 상태 출력
  printPortStatus(): void {
    console.log('=== Port Status ===');
    console.log('Used Ports:', Array.from(this.usedPorts).sort((a, b) => a - b));
    console.log('Available Dynamic Ports:', 65535 - 49152 + 1 - this.usedPorts.size);
  }
}

// 소켓 정보 클래스
class SocketInfo {
  protocol: 'TCP' | 'UDP';
  localAddress: string;
  localPort: number;
  remoteAddress?: string;
  remotePort?: number;
  state: 'CLOSED' | 'LISTENING' | 'CONNECTING' | 'CONNECTED' | 'CLOSING';

  constructor(
    protocol: 'TCP' | 'UDP',
    localAddress: string,
    localPort: number
  ) {
    this.protocol = protocol;
    this.localAddress = localAddress;
    this.localPort = localPort;
    this.state = 'CLOSED';
  }

  // 5-tuple 문자열
  toString(): string {
    if (this.remoteAddress && this.remotePort) {
      return `${this.protocol} ${this.localAddress}:${this.localPort} <-> ${this.remoteAddress}:${this.remotePort}`;
    }
    return `${this.protocol} ${this.localAddress}:${this.localPort} (${this.state})`;
  }
}

// TCP 서버 소켓 구현
class TCPServerSocket extends EventEmitter {
  private server: net.Server | null = null;
  private clients: Map<string, net.Socket> = new Map();
  private portManager: PortManager;
  private socketInfo: SocketInfo | null = null;

  constructor(portManager: PortManager) {
    super();
    this.portManager = portManager;
  }

  // 서버 시작
  listen(port: number, host: string = '0.0.0.0', callback?: () => void): void {
    const allocatedPort = this.portManager.allocatePort(port);
    if (!allocatedPort) {
      throw new Error(`Port ${port} is not available`);
    }

    this.socketInfo = new SocketInfo('TCP', host, allocatedPort);
    this.socketInfo.state = 'LISTENING';

    this.server = net.createServer((socket) => {
      this.handleClient(socket);
    });

    this.server.listen(allocatedPort, host, () => {
      console.log(`TCP Server listening on ${host}:${allocatedPort}`);
      this.emit('listening', allocatedPort);
      if (callback) callback();
    });

    this.server.on('error', (err) => {
      console.error(`Server error: ${err}`);
      this.portManager.releasePort(allocatedPort);
      this.emit('error', err);
    });

    this.server.on('close', () => {
      this.socketInfo!.state = 'CLOSED';
      this.emit('close');
    });
  }

  // 클라이언트 처리
  private handleClient(socket: net.Socket): void {
    const clientId = `${socket.remoteAddress}:${socket.remotePort}`;
    this.clients.set(clientId, socket);
    
    console.log(`Client connected: ${clientId}`);
    console.log(`Active connections: ${this.clients.size}`);
    this.emit('connection', socket);

    socket.on('data', (data) => {
      console.log(`Received from ${clientId}: ${data.toString().trim()}`);
      this.emit('data', data, clientId);
      
      // Echo back
      socket.write(`Echo: ${data}`);
    });

    socket.on('close', () => {
      console.log(`Client disconnected: ${clientId}`);
      this.clients.delete(clientId);
      this.emit('disconnect', clientId);
    });

    socket.on('error', (err) => {
      console.error(`Client error ${clientId}: ${err.message}`);
      this.clients.delete(clientId);
    });
  }

  // 모든 클라이언트에게 브로드캐스트
  broadcast(message: string): void {
    const data = Buffer.from(message);
    for (const [id, socket] of this.clients) {
      socket.write(data);
      console.log(`Broadcast to ${id}`);
    }
  }

  // 특정 클라이언트에게 전송
  sendTo(clientId: string, message: string): void {
    const socket = this.clients.get(clientId);
    if (socket) {
      socket.write(message);
      console.log(`Sent to ${clientId}: ${message}`);
    }
  }

  // 서버 종료
  close(): void {
    if (this.server) {
      // 모든 클라이언트 연결 종료
      for (const [_, socket] of this.clients) {
        socket.destroy();
      }
      this.clients.clear();
      
      this.server.close();
      if (this.socketInfo) {
        this.portManager.releasePort(this.socketInfo.localPort);
        this.socketInfo.state = 'CLOSED';
      }
      console.log('TCP Server closed');
    }
  }

  // 서버 정보
  getInfo(): SocketInfo | null {
    return this.socketInfo;
  }
}

// TCP 클라이언트 소켓 구현
class TCPClientSocket extends EventEmitter {
  private socket: net.Socket | null = null;
  private socketInfo: SocketInfo | null = null;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 3;

  // 서버 연결
  connect(host: string, port: number, callback?: () => void): void {
    this.socket = new net.Socket();
    
    this.socket.connect(port, host, () => {
      this.socketInfo = new SocketInfo(
        'TCP',
        this.socket!.localAddress || '',
        this.socket!.localPort || 0
      );
      this.socketInfo.remoteAddress = host;
      this.socketInfo.remotePort = port;
      this.socketInfo.state = 'CONNECTED';
      
      console.log(`Connected to ${host}:${port}`);
      console.log(`Local endpoint: ${this.socketInfo.localAddress}:${this.socketInfo.localPort}`);
      
      this.emit('connect');
      if (callback) callback();
    });

    this.socket.on('data', (data) => {
      console.log(`Received: ${data.toString().trim()}`);
      this.emit('data', data);
    });

    this.socket.on('close', () => {
      console.log('Connection closed');
      if (this.socketInfo) {
        this.socketInfo.state = 'CLOSED';
      }
      this.emit('close');
      
      // 자동 재연결 로직
      if (this.reconnectAttempts < this.maxReconnectAttempts) {
        this.reconnectAttempts++;
        console.log(`Reconnecting... (${this.reconnectAttempts}/${this.maxReconnectAttempts})`);
        setTimeout(() => this.connect(host, port), 2000);
      }
    });

    this.socket.on('error', (err) => {
      console.error(`Connection error: ${err.message}`);
      if (this.socketInfo) {
        this.socketInfo.state = 'CLOSED';
      }
      this.emit('error', err);
    });

    this.socket.on('timeout', () => {
      console.log('Connection timeout');
      this.socket!.destroy();
    });
  }

  // 데이터 전송
  send(data: string): boolean {
    if (this.socket && this.socketInfo?.state === 'CONNECTED') {
      this.socket.write(data);
      console.log(`Sent: ${data.trim()}`);
      return true;
    } else {
      console.error('Not connected');
      return false;
    }
  }

  // 연결 종료
  close(): void {
    if (this.socket) {
      this.socket.destroy();
      if (this.socketInfo) {
        this.socketInfo.state = 'CLOSED';
      }
      console.log('Client socket closed');
    }
  }

  // 소켓 정보
  getInfo(): SocketInfo | null {
    return this.socketInfo;
  }
}

// UDP 소켓 구현
class UDPSocket extends EventEmitter {
  private socket: dgram.Socket | null = null;
  private socketInfo: SocketInfo | null = null;
  private portManager: PortManager;

  constructor(portManager: PortManager) {
    super();
    this.portManager = portManager;
  }

  // UDP 소켓 생성 및 바인딩
  bind(port?: number, host: string = '0.0.0.0', callback?: () => void): void {
    this.socket = dgram.createSocket('udp4');
    
    const allocatedPort = this.portManager.allocatePort(port);
    if (!allocatedPort) {
      throw new Error(`Port ${port || 'dynamic'} is not available`);
    }

    this.socketInfo = new SocketInfo('UDP', host, allocatedPort);
    this.socketInfo.state = 'LISTENING';

    this.socket.bind(allocatedPort, host, () => {
      console.log(`UDP Socket bound to ${host}:${allocatedPort}`);
      this.emit('listening', allocatedPort);
      if (callback) callback();
    });

    this.socket.on('message', (msg, rinfo) => {
      console.log(`UDP received from ${rinfo.address}:${rinfo.port}: ${msg.toString().trim()}`);
      this.emit('message', msg, rinfo);
    });

    this.socket.on('error', (err) => {
      console.error(`UDP error: ${err}`);
      this.portManager.releasePort(allocatedPort);
      this.emit('error', err);
    });

    this.socket.on('close', () => {
      if (this.socketInfo) {
        this.socketInfo.state = 'CLOSED';
      }
      this.emit('close');
    });
  }

  // UDP 데이터 전송
  send(message: string, port: number, address: string, callback?: () => void): void {
    if (!this.socket) {
      throw new Error('Socket not initialized');
    }

    const data = Buffer.from(message);
    this.socket.send(data, port, address, (err) => {
      if (err) {
        console.error(`Send error: ${err}`);
        this.emit('error', err);
      } else {
        console.log(`UDP sent to ${address}:${port}: ${message.trim()}`);
        if (callback) callback();
      }
    });
  }

  // 브로드캐스트
  broadcast(message: string, port: number): void {
    if (!this.socket) {
      throw new Error('Socket not initialized');
    }

    this.socket.setBroadcast(true);
    this.send(message, port, '255.255.255.255');
    console.log(`Broadcast message sent to port ${port}`);
  }

  // 멀티캐스트 가입
  joinMulticast(multicastAddr: string): void {
    if (!this.socket) {
      throw new Error('Socket not initialized');
    }

    this.socket.addMembership(multicastAddr);
    console.log(`Joined multicast group: ${multicastAddr}`);
  }

  // 멀티캐스트 탈퇴
  leaveMulticast(multicastAddr: string): void {
    if (!this.socket) {
      throw new Error('Socket not initialized');
    }

    this.socket.dropMembership(multicastAddr);
    console.log(`Left multicast group: ${multicastAddr}`);
  }

  // 소켓 닫기
  close(): void {
    if (this.socket && this.socketInfo) {
      this.portManager.releasePort(this.socketInfo.localPort);
      this.socket.close();
      this.socketInfo.state = 'CLOSED';
      console.log('UDP socket closed');
    }
  }

  // 소켓 정보
  getInfo(): SocketInfo | null {
    return this.socketInfo;
  }
}

// 포트 스캐너
class PortScanner {
  // TCP 포트 스캔
  async scanTCPPort(host: string, port: number, timeout: number = 1000): Promise<boolean> {
    return new Promise((resolve) => {
      const socket = new net.Socket();
      
      const timer = setTimeout(() => {
        socket.destroy();
        resolve(false);
      }, timeout);

      socket.on('connect', () => {
        clearTimeout(timer);
        socket.destroy();
        resolve(true);
      });

      socket.on('error', () => {
        clearTimeout(timer);
        resolve(false);
      });

      socket.connect(port, host);
    });
  }

  // 포트 범위 스캔
  async scanRange(
    host: string,
    startPort: number,
    endPort: number,
    timeout: number = 500
  ): Promise<number[]> {
    const openPorts: number[] = [];
    
    console.log(`Scanning ${host} ports ${startPort}-${endPort}...`);
    
    for (let port = startPort; port <= endPort; port++) {
      const isOpen = await this.scanTCPPort(host, port, timeout);
      if (isOpen) {
        openPorts.push(port);
        const service = this.getServiceName(port);
        console.log(`Port ${port}: OPEN${service ? ` (${service})` : ''}`);
      }
    }
    
    return openPorts;
  }

  // Well-known 포트 스캔
  async scanWellKnown(host: string): Promise<Map<number, string>> {
    const results = new Map<number, string>();
    const wellKnownPorts = [
      21, 22, 23, 25, 53, 80, 110, 143, 443, 
      3306, 3389, 5432, 6379, 8080, 27017
    ];
    
    console.log(`Scanning well-known ports on ${host}...`);
    
    for (const port of wellKnownPorts) {
      const isOpen = await this.scanTCPPort(host, port);
      if (isOpen) {
        const service = this.getServiceName(port);
        results.set(port, service);
        console.log(`Port ${port}: OPEN (${service})`);
      }
    }
    
    return results;
  }

  private getServiceName(port: number): string {
    const services: { [key: number]: string } = {
      20: 'FTP-DATA',
      21: 'FTP',
      22: 'SSH',
      23: 'Telnet',
      25: 'SMTP',
      53: 'DNS',
      67: 'DHCP',
      68: 'DHCP',
      80: 'HTTP',
      110: 'POP3',
      143: 'IMAP',
      443: 'HTTPS',
      3306: 'MySQL',
      3389: 'RDP',
      5432: 'PostgreSQL',
      6379: 'Redis',
      8080: 'HTTP-ALT',
      27017: 'MongoDB'
    };
    
    return services[port] || 'Unknown';
  }
}

// 소켓 통계 클래스
class SocketStatistics {
  private stats = {
    tcpConnections: 0,
    udpSockets: 0,
    bytesReceived: 0,
    bytesSent: 0,
    packetsReceived: 0,
    packetsSent: 0,
    errors: 0
  };

  private connectionHistory: Array<{
    timestamp: Date;
    type: 'TCP' | 'UDP';
    action: 'CONNECT' | 'DISCONNECT';
    address: string;
  }> = [];

  // TCP 연결 추가
  addTCPConnection(address: string): void {
    this.stats.tcpConnections++;
    this.connectionHistory.push({
      timestamp: new Date(),
      type: 'TCP',
      action: 'CONNECT',
      address
    });
  }

  // TCP 연결 제거
  removeTCPConnection(address: string): void {
    if (this.stats.tcpConnections > 0) {
      this.stats.tcpConnections--;
      this.connectionHistory.push({
        timestamp: new Date(),
        type: 'TCP',
        action: 'DISCONNECT',
        address
      });
    }
  }

  // UDP 소켓 추가
  addUDPSocket(address: string): void {
    this.stats.udpSockets++;
    this.connectionHistory.push({
      timestamp: new Date(),
      type: 'UDP',
      action: 'CONNECT',
      address
    });
  }

  // UDP 소켓 제거
  removeUDPSocket(address: string): void {
    if (this.stats.udpSockets > 0) {
      this.stats.udpSockets--;
      this.connectionHistory.push({
        timestamp: new Date(),
        type: 'UDP',
        action: 'DISCONNECT',
        address
      });
    }
  }

  // 데이터 전송 기록
  recordDataTransfer(bytes: number, direction: 'in' | 'out'): void {
    if (direction === 'in') {
      this.stats.bytesReceived += bytes;
      this.stats.packetsReceived++;
    } else {
      this.stats.bytesSent += bytes;
      this.stats.packetsSent++;
    }
  }

  // 에러 기록
  recordError(): void {
    this.stats.errors++;
  }

  // 통계 출력
  printStats(): void {
    console.log('\n=== Socket Statistics ===');
    console.log(`TCP Connections: ${this.stats.tcpConnections}`);
    console.log(`UDP Sockets: ${this.stats.udpSockets}`);
    console.log(`Bytes Received: ${this.formatBytes(this.stats.bytesReceived)}`);
    console.log(`Bytes Sent: ${this.formatBytes(this.stats.bytesSent)}`);
    console.log(`Packets Received: ${this.stats.packetsReceived}`);
    console.log(`Packets Sent: ${this.stats.packetsSent}`);
    console.log(`Errors: ${this.stats.errors}`);
  }

  // 연결 이력 출력
  printHistory(limit: number = 10): void {
    console.log('\n=== Connection History ===');
    const recent = this.connectionHistory.slice(-limit);
    
    for (const entry of recent) {
      console.log(
        `[${entry.timestamp.toISOString()}] ` +
        `${entry.type} ${entry.action} ${entry.address}`
      );
    }
  }

  private formatBytes(bytes: number): string {
    if (bytes < 1024) return `${bytes} B`;
    if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(2)} KB`;
    return `${(bytes / (1024 * 1024)).toFixed(2)} MB`;
  }
}

// 통합 네트워크 애플리케이션 예제
class NetworkApplication {
  private portManager: PortManager;
  private statistics: SocketStatistics;
  private tcpServer: TCPServerSocket | null = null;
  private tcpClient: TCPClientSocket | null = null;
  private udpSocket: UDPSocket | null = null;

  constructor() {
    this.portManager = new PortManager();
    this.statistics = new SocketStatistics();
  }

  // TCP 에코 서버 시작
  async startTCPEchoServer(port: number = 8080): Promise<void> {
    console.log('\n=== Starting TCP Echo Server ===');
    
    this.tcpServer = new TCPServerSocket(this.portManager);
    
    this.tcpServer.on('connection', (socket) => {
      const address = `${socket.remoteAddress}:${socket.remotePort}`;
      this.statistics.addTCPConnection(address);
    });

    this.tcpServer.on('disconnect', (clientId) => {
      this.statistics.removeTCPConnection(clientId);
    });

    this.tcpServer.on('data', (data) => {
      this.statistics.recordDataTransfer(data.length, 'in');
      this.statistics.recordDataTransfer(data.length, 'out'); // Echo
    });

    this.tcpServer.listen(port);
  }

  // TCP 클라이언트 연결
  async connectTCPClient(host: string, port: number): Promise<void> {
    console.log('\n=== Connecting TCP Client ===');
    
    this.tcpClient = new TCPClientSocket();
    
    this.tcpClient.on('data', (data) => {
      this.statistics.recordDataTransfer(data.length, 'in');
    });

    this.tcpClient.connect(host, port, () => {
      this.statistics.addTCPConnection(`${host}:${port}`);
      
      // 테스트 메시지 전송
      this.tcpClient!.send('Hello from client!');
      this.statistics.recordDataTransfer(17, 'out');
    });
  }

  // UDP 서비스 시작
  async startUDPService(port: number = 8081): Promise<void> {
    console.log('\n=== Starting UDP Service ===');
    
    this.udpSocket = new UDPSocket(this.portManager);
    
    this.udpSocket.on('message', (msg, rinfo) => {
      this.statistics.recordDataTransfer(msg.length, 'in');
      
      // Echo back
      this.udpSocket!.send(
        `Echo: ${msg}`,
        rinfo.port,
        rinfo.address
      );
      this.statistics.recordDataTransfer(msg.length + 6, 'out');
    });

    this.udpSocket.bind(port, '0.0.0.0', () => {
      this.statistics.addUDPSocket(`0.0.0.0:${port}`);
    });
  }

  // 포트 스캔 실행
  async performPortScan(host: string = 'localhost'): Promise<void> {
    console.log('\n=== Port Scanning ===');
    
    const scanner = new PortScanner();
    const openPorts = await scanner.scanRange(host, 8080, 8090);
    
    console.log(`\nOpen ports found: ${openPorts.join(', ')}`);
  }

  // 종료
  shutdown(): void {
    console.log('\n=== Shutting down ===');
    
    if (this.tcpServer) {
      this.tcpServer.close();
    }
    
    if (this.tcpClient) {
      this.tcpClient.close();
    }
    
    if (this.udpSocket) {
      this.udpSocket.close();
    }
    
    this.statistics.printStats();
    this.statistics.printHistory();
    this.portManager.printPortStatus();
  }
}

// 메인 실행 함수
async function demonstratePortsAndSockets() {
  console.log('=== Port and Socket Demonstration ===');
  
  const app = new NetworkApplication();
  
  try {
    // TCP 서버 시작
    await app.startTCPEchoServer(8080);
    
    // 잠시 대기
    await new Promise(resolve => setTimeout(resolve, 100));
    
    // TCP 클라이언트 연결
    await app.connectTCPClient('localhost', 8080);
    
    // UDP 서비스 시작
    await app.startUDPService(8081);
    
    // 포트 스캔
    await app.performPortScan();
    
    // 3초 후 종료
    setTimeout(() => {
      app.shutdown();
    }, 3000);
    
  } catch (error) {
    console.error('Error:', error);
    app.shutdown();
  }
}

// 실행 (주석 해제하여 사용)
// demonstratePortsAndSockets().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **포트**: 0-65535 범위, Well-known(0-1023) 주의
> 2. **소켓**: IP + 포트 = 통신 엔드포인트
> 3. **5-tuple**: Protocol, Local IP/Port, Remote IP/Port로 유일 식별
> 4. **TCP 소켓**: 연결 지향, listen() → accept() → send/recv
> 5. **UDP 소켓**: 비연결, bind() → sendto/recvfrom

## 참고 자료

### 📖 추가 학습
- [Network Socket - Wikipedia](https://en.wikipedia.org/wiki/Network_socket)
- [List of TCP and UDP port numbers](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)
- [Socket Programming Guide](https://beej.us/guide/bgnet/)
- [TCP/IP 소켓 프로그래밍](http://www.ktword.co.kr/abbr_view.php?m_temp1=280)

### 🔗 관련 문서
- [[05_tcp-udp|TCP와 UDP 프로토콜]]
- [[06_tcp-handshake|TCP Handshake]]
- [[10_firewall-dns|방화벽과 DNS]]
