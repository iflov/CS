---
title: TCP와 UDP 프로토콜
created: 2024-12-31
updated: 2024-12-31
tags:
  - CS
  - Network
  - TCP
  - UDP
  - Transport
  - Protocol
category: Network
status: 완료
---

# 🔄 TCP와 UDP 프로토콜 (TCP vs UDP)

## 📌 개요

TCP(Transmission Control Protocol)와 UDP(User Datagram Protocol)는 Transport Layer의 핵심 프로토콜입니다. TCP는 **신뢰성**을 중시하고, UDP는 **속도**를 중시하는 상반된 특성을 가집니다.

> [!info] 핵심 개념
> - **TCP**: 연결 지향, 신뢰성 보장, 순서 보장
> - **UDP**: 비연결성, 빠른 전송, 실시간 통신
> - **Trade-off**: 신뢰성 vs 속도

## 📚 목차

1. [TCP (Transmission Control Protocol)](#tcp-transmission-control-protocol)
2. [UDP (User Datagram Protocol)](#udp-user-datagram-protocol)
3. [TCP vs UDP 비교](#tcp-vs-udp-비교)
4. [패킷 구조 분석](#패킷-구조-분석)
5. [실제 구현 예시](#실제-구현-예시)
6. [참고 자료](#참고-자료)

## TCP (Transmission Control Protocol)

### 핵심 특징
- **연결 지향** (Connection-Oriented)
- **신뢰성 보장** (Reliable)
- **순서 보장** (Ordered)
- **흐름 제어** (Flow Control)
- **혼잡 제어** (Congestion Control)

### TCP 헤더 구조 (20 bytes)
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if any)                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### TCP 사용 사례
- **HTTP/HTTPS**: 웹 페이지 전송
- **FTP**: 파일 전송
- **SMTP**: 이메일 전송
- **SSH**: 원격 접속
- **데이터베이스**: 쿼리 및 결과 전송

## UDP (User Datagram Protocol)

### 핵심 특징
- **비연결성** (Connectionless)
- **신뢰성 미보장** (Unreliable)
- **순서 미보장** (Unordered)
- **낮은 오버헤드** (Low Overhead)
- **빠른 전송** (Fast)

### UDP 헤더 구조 (8 bytes)
```
 0      7 8     15 16    23 24    31
+--------+--------+--------+--------+
|   Source Port   |  Destination Port |
+--------+--------+--------+--------+
|     Length      |    Checksum      |
+--------+--------+--------+--------+
|            Data (if any)           |
+------------------------------------+
```

### UDP 사용 사례
- **DNS**: 도메인 이름 조회
- **DHCP**: IP 주소 할당
- **VoIP**: 인터넷 전화
- **온라인 게임**: 실시간 액션
- **스트리밍**: 실시간 방송

## TCP vs UDP 비교

### 주요 차이점

| 특성 | TCP | UDP |
|------|-----|-----|
| **연결 설정** | 3-way handshake | 없음 |
| **신뢰성** | 보장 (재전송) | 미보장 |
| **순서** | 보장 | 미보장 |
| **속도** | 상대적 느림 | 빠름 |
| **헤더 크기** | 20 bytes | 8 bytes |
| **오버헤드** | 높음 | 낮음 |
| **흐름 제어** | 있음 | 없음 |
| **사용 예** | 파일 전송, 웹 | 스트리밍, 게임 |

## 패킷 구조 분석

### TCP 세그먼트 필드
- **Source/Destination Port**: 16 bits each
- **Sequence Number**: 32 bits (바이트 단위 순서)
- **Acknowledgment Number**: 32 bits (다음 예상 바이트)
- **Flags**: URG, ACK, PSH, RST, SYN, FIN
- **Window Size**: 수신 가능한 데이터 크기
- **Checksum**: 오류 검출

### UDP 데이터그램 필드
- **Source/Destination Port**: 16 bits each
- **Length**: 헤더 + 데이터 전체 길이
- **Checksum**: 선택적 오류 검출

## 실제 구현 예시

### TypeScript로 TCP/UDP 시뮬레이션

```typescript
import * as dgram from 'dgram'; // UDP용
import * as net from 'net'; // TCP용

// TCP 서버/클라이언트 구현
class TCPConnection {
  private sequenceNumber: number = 0;
  private acknowledgmentNumber: number = 0;
  private windowSize: number = 65535;
  private retransmissionQueue: Map<number, {
    data: Buffer;
    timestamp: number;
    retries: number;
  }> = new Map();
  private maxRetries = 3;
  private timeout = 3000; // 3초

  // TCP 세그먼트 생성
  createSegment(
    data: Buffer,
    flags: { SYN?: boolean; ACK?: boolean; FIN?: boolean; PSH?: boolean }
  ): Buffer {
    const segment = Buffer.allocUnsafe(20 + data.length);
    
    // Source Port (임의)
    segment.writeUInt16BE(12345, 0);
    // Destination Port
    segment.writeUInt16BE(80, 2);
    // Sequence Number
    segment.writeUInt32BE(this.sequenceNumber, 4);
    // Acknowledgment Number
    segment.writeUInt32BE(this.acknowledgmentNumber, 8);
    
    // Data Offset and Flags
    let offsetAndFlags = (5 << 12); // 5 * 4 = 20 bytes header
    if (flags.SYN) offsetAndFlags |= 0x02;
    if (flags.ACK) offsetAndFlags |= 0x10;
    if (flags.FIN) offsetAndFlags |= 0x01;
    if (flags.PSH) offsetAndFlags |= 0x08;
    segment.writeUInt16BE(offsetAndFlags, 12);
    
    // Window Size
    segment.writeUInt16BE(this.windowSize, 14);
    // Checksum
    segment.writeUInt16BE(this.calculateChecksum(segment), 16);
    // Urgent Pointer
    segment.writeUInt16BE(0, 18);
    
    // Data
    data.copy(segment, 20);
    
    return segment;
  }

  // 신뢰성 있는 전송
  async reliableSend(data: Buffer): Promise<void> {
    const segment = this.createSegment(data, { PSH: true, ACK: true });
    
    // 재전송 큐에 추가
    this.retransmissionQueue.set(this.sequenceNumber, {
      data: segment,
      timestamp: Date.now(),
      retries: 0
    });
    
    // 전송
    await this.transmit(segment);
    
    // ACK 대기
    await this.waitForAck(this.sequenceNumber);
    
    // 시퀀스 번호 증가
    this.sequenceNumber += data.length;
  }

  // ACK 대기 및 재전송
  private async waitForAck(seqNum: number): Promise<void> {
    return new Promise((resolve, reject) => {
      const checkInterval = setInterval(() => {
        const entry = this.retransmissionQueue.get(seqNum);
        
        if (!entry) {
          clearInterval(checkInterval);
          resolve();
          return;
        }
        
        // 타임아웃 확인
        if (Date.now() - entry.timestamp > this.timeout) {
          if (entry.retries >= this.maxRetries) {
            clearInterval(checkInterval);
            reject(new Error('Max retries exceeded'));
            return;
          }
          
          // 재전송
          console.log(`Retransmitting seq ${seqNum} (retry ${entry.retries + 1})`);
          entry.retries++;
          entry.timestamp = Date.now();
          this.transmit(entry.data);
        }
      }, 100);
    });
  }

  // ACK 수신 처리
  receiveAck(ackNum: number): void {
    // ACK된 세그먼트를 큐에서 제거
    for (const [seq, _] of this.retransmissionQueue) {
      if (seq < ackNum) {
        this.retransmissionQueue.delete(seq);
      }
    }
    
    this.acknowledgmentNumber = ackNum;
  }

  private async transmit(segment: Buffer): Promise<void> {
    // 실제 전송 시뮬레이션
    console.log(`Transmitting TCP segment: ${segment.length} bytes`);
  }

  private calculateChecksum(data: Buffer): number {
    let sum = 0;
    for (let i = 0; i < data.length - 1; i += 2) {
      sum += data.readUInt16BE(i);
    }
    if (data.length % 2 === 1) {
      sum += data[data.length - 1] << 8;
    }
    
    while (sum >> 16) {
      sum = (sum & 0xFFFF) + (sum >> 16);
    }
    
    return ~sum & 0xFFFF;
  }

  // 흐름 제어 (Sliding Window)
  updateWindow(size: number): void {
    this.windowSize = size;
    console.log(`Window size updated to ${size}`);
  }

  // 혼잡 제어 (간단한 구현)
  congestionControl(event: 'timeout' | 'ack' | 'duplicate_ack'): void {
    switch (event) {
      case 'timeout':
        // Slow Start
        this.windowSize = Math.max(1, Math.floor(this.windowSize / 2));
        console.log(`Congestion detected: window reduced to ${this.windowSize}`);
        break;
      case 'ack':
        // Additive Increase
        this.windowSize = Math.min(65535, this.windowSize + 1);
        break;
      case 'duplicate_ack':
        // Fast Recovery
        this.windowSize = Math.max(1, Math.floor(this.windowSize * 0.75));
        break;
    }
  }
}

// UDP 구현
class UDPConnection {
  private socket: dgram.Socket | null = null;

  // UDP 데이터그램 생성
  createDatagram(data: Buffer, sourcePort: number, destPort: number): Buffer {
    const header = Buffer.allocUnsafe(8);
    
    // Source Port
    header.writeUInt16BE(sourcePort, 0);
    // Destination Port
    header.writeUInt16BE(destPort, 2);
    // Length
    header.writeUInt16BE(8 + data.length, 4);
    // Checksum (optional, 0 for simplicity)
    header.writeUInt16BE(0, 6);
    
    return Buffer.concat([header, data]);
  }

  // 빠른 전송 (신뢰성 없음)
  fastSend(data: Buffer, address: string, port: number): void {
    const datagram = this.createDatagram(data, 54321, port);
    
    // 즉시 전송, ACK 대기 없음
    this.transmit(datagram, address, port);
  }

  private transmit(datagram: Buffer, address: string, port: number): void {
    console.log(`Transmitting UDP datagram: ${datagram.length} bytes to ${address}:${port}`);
    // Fire and forget
  }

  // 브로드캐스트
  broadcast(data: Buffer, port: number): void {
    const datagram = this.createDatagram(data, 54321, port);
    this.transmit(datagram, '255.255.255.255', port);
    console.log('UDP broadcast sent');
  }

  // 멀티캐스트
  multicast(data: Buffer, group: string, port: number): void {
    const datagram = this.createDatagram(data, 54321, port);
    this.transmit(datagram, group, port);
    console.log(`UDP multicast sent to ${group}`);
  }
}

// 실제 사용 예제
class NetworkDemo {
  // TCP 파일 전송 시뮬레이션
  static async tcpFileTransfer(filePath: string): Promise<void> {
    console.log('=== TCP File Transfer ===');
    const tcp = new TCPConnection();
    
    // 파일을 청크로 나누어 전송
    const fileData = Buffer.from('File content here...');
    const chunkSize = 1024;
    
    for (let i = 0; i < fileData.length; i += chunkSize) {
      const chunk = fileData.slice(i, Math.min(i + chunkSize, fileData.length));
      
      try {
        await tcp.reliableSend(chunk);
        console.log(`Sent chunk ${i / chunkSize + 1}`);
      } catch (error) {
        console.error(`Failed to send chunk: ${error}`);
        // TCP는 재전송 시도
      }
    }
  }

  // UDP 실시간 스트리밍 시뮬레이션
  static udpStreaming(): void {
    console.log('=== UDP Streaming ===');
    const udp = new UDPConnection();
    
    // 비디오 프레임 시뮬레이션
    const frameInterval = setInterval(() => {
      const frame = Buffer.from(`Frame ${Date.now()}`);
      udp.fastSend(frame, '192.168.1.100', 5000);
      
      // 프레임 드롭 가능 - UDP는 신경쓰지 않음
    }, 33); // 30 FPS
    
    // 5초 후 중지
    setTimeout(() => {
      clearInterval(frameInterval);
      console.log('Streaming stopped');
    }, 5000);
  }

  // 프로토콜 선택 가이드
  static protocolSelector(requirements: {
    reliability: boolean;
    realtime: boolean;
    ordered: boolean;
    broadcast: boolean;
  }): 'TCP' | 'UDP' {
    if (requirements.broadcast) {
      return 'UDP'; // TCP는 브로드캐스트 불가
    }
    
    if (requirements.realtime && !requirements.reliability) {
      return 'UDP'; // 실시간이 중요하고 약간의 손실 허용
    }
    
    if (requirements.reliability && requirements.ordered) {
      return 'TCP'; // 신뢰성과 순서가 중요
    }
    
    // 기본적으로 UDP (더 간단하고 빠름)
    return 'UDP';
  }
}

// 성능 비교 테스트
class PerformanceComparison {
  static async compareLatency(): Promise<void> {
    console.log('=== Latency Comparison ===');
    
    // TCP 연결 설정 시간 포함
    const tcpStart = Date.now();
    // 3-way handshake 시뮬레이션
    await new Promise(resolve => setTimeout(resolve, 50));
    const tcpSetupTime = Date.now() - tcpStart;
    
    // UDP는 연결 설정 없음
    const udpSetupTime = 0;
    
    console.log(`TCP Setup: ${tcpSetupTime}ms`);
    console.log(`UDP Setup: ${udpSetupTime}ms`);
  }

  static compareThroughput(): void {
    console.log('=== Throughput Comparison ===');
    
    const dataSize = 1000000; // 1MB
    const tcpHeaderSize = 20;
    const udpHeaderSize = 8;
    
    // TCP 오버헤드
    const tcpPackets = Math.ceil(dataSize / 1460); // MTU - TCP header
    const tcpOverhead = tcpPackets * tcpHeaderSize;
    
    // UDP 오버헤드
    const udpPackets = Math.ceil(dataSize / 1472); // MTU - UDP header
    const udpOverhead = udpPackets * udpHeaderSize;
    
    console.log(`TCP Overhead: ${tcpOverhead} bytes (${tcpPackets} packets)`);
    console.log(`UDP Overhead: ${udpOverhead} bytes (${udpPackets} packets)`);
    console.log(`Overhead Difference: ${tcpOverhead - udpOverhead} bytes`);
  }
}

// 실행 예제
async function main() {
  // 프로토콜 선택
  const webBrowsing = NetworkDemo.protocolSelector({
    reliability: true,
    realtime: false,
    ordered: true,
    broadcast: false
  });
  console.log(`Web Browsing: Use ${webBrowsing}`);
  
  const voiceCall = NetworkDemo.protocolSelector({
    reliability: false,
    realtime: true,
    ordered: false,
    broadcast: false
  });
  console.log(`Voice Call: Use ${voiceCall}`);
  
  // 성능 비교
  await PerformanceComparison.compareLatency();
  PerformanceComparison.compareThroughput();
}

// main().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **TCP**: 신뢰성이 중요한 경우 (파일, 웹, 이메일)
> 2. **UDP**: 속도가 중요한 경우 (게임, 스트리밍, VoIP)
> 3. **Trade-off**: 신뢰성 vs 속도/효율성
> 4. **헤더 크기**: TCP(20 bytes) vs UDP(8 bytes)
> 5. **용도별 선택**: 요구사항에 따라 적절한 프로토콜 선택

## 참고 자료

### 📖 추가 학습
- [TCP vs UDP - Diffen](https://www.diffen.com/difference/TCP_vs_UDP)
- [Transmission Control Protocol - Wikipedia](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)
- [User Datagram Protocol - Wikipedia](https://en.wikipedia.org/wiki/User_Datagram_Protocol)

### 🔗 관련 문서
- [[06_tcp-handshake|TCP Handshake]]
- [[03_tcp-ip-model|TCP/IP 4계층 모델]]
- [[09_port-socket|포트와 소켓]]
