---
title: TCP/IP 4계층 모델
created: 2024-12-31
updated: 2024-12-31
tags:
  - CS
  - Network
  - TCP/IP
  - Protocol
  - Internet
category: Network
status: 완료
---

# 🌍 TCP/IP 4계층 모델 (TCP/IP Model)

## 📌 개요

TCP/IP 모델은 인터넷의 실제 구현에 사용되는 **실용적인 네트워크 모델**입니다. OSI 7계층을 단순화하여 4개 계층으로 구성했으며, 현재 인터넷의 표준 프로토콜 스택으로 사용됩니다.

> [!info] 핵심 개념
> - **Internet Protocol Suite**: TCP/IP 프로토콜 집합의 공식 명칭
> - **실용성 중심**: OSI보다 단순하고 구현 중심적
> - **계층 통합**: OSI의 상위 3계층을 Application으로 통합

## 📚 목차

1. [TCP/IP vs OSI 비교](#tcpip-vs-osi-비교)
2. [TCP/IP 4계층 구조](#tcpip-4계층-구조)
3. [각 계층별 프로토콜](#각-계층별-프로토콜)
4. [실제 구현 예시](#실제-구현-예시)
5. [참고 자료](#참고-자료)

## TCP/IP vs OSI 비교

```
OSI 7계층                    TCP/IP 4계층
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
7. Application    ┐
6. Presentation   ├─────→  4. Application
5. Session        ┘
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
4. Transport      ─────→  3. Transport
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
3. Network        ─────→  2. Internet (Network)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
2. Data Link      ┐
1. Physical       ┴─────→  1. Network Interface
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## TCP/IP 4계층 구조

### 4️⃣ Application Layer (응용 계층)
- **역할**: 사용자 애플리케이션과 직접 상호작용
- **OSI 대응**: Application + Presentation + Session
- **프로토콜**: 
  - **HTTP/HTTPS**: 웹 통신
  - **FTP**: 파일 전송
  - **SMTP/POP3/IMAP**: 이메일
  - **DNS**: 도메인 이름 해석
  - **DHCP**: 동적 IP 할당
  - **SSH/Telnet**: 원격 접속
- **데이터 단위**: Message

### 3️⃣ Transport Layer (전송 계층)
- **역할**: 프로세스 간 신뢰성 있는 통신
- **OSI 대응**: Transport Layer
- **프로토콜**: 
  - **TCP**: 신뢰성, 연결 지향
  - **UDP**: 비연결성, 빠른 전송
- **데이터 단위**: Segment (TCP), Datagram (UDP)
- **주요 기능**: 포트 번호, 오류 제어, 흐름 제어

### 2️⃣ Internet Layer (인터넷 계층)
- **역할**: 네트워크 간 데이터 전송 및 라우팅
- **OSI 대응**: Network Layer
- **프로토콜**: 
  - **IP (IPv4/IPv6)**: 주소 지정 및 라우팅
  - **ICMP**: 오류 보고 및 진단
  - **ARP**: IP → MAC 주소 변환
  - **IGMP**: 멀티캐스트 그룹 관리
- **데이터 단위**: Packet
- **주요 기능**: IP 주소, 라우팅, 패킷 분할/재조립

### 1️⃣ Network Interface Layer (네트워크 인터페이스 계층)
- **역할**: 물리적 네트워크와의 인터페이스
- **OSI 대응**: Physical + Data Link Layer
- **프로토콜**: 
  - **Ethernet**: 유선 LAN
  - **WiFi (802.11)**: 무선 LAN
  - **PPP**: Point-to-Point 연결
- **데이터 단위**: Frame
- **주요 기능**: MAC 주소, 프레이밍, 오류 검출

## 각 계층별 프로토콜

### Application Layer 프로토콜 상세

| 프로토콜 | 포트 | 용도 | 특징 |
|---------|------|------|------|
| HTTP | 80 | 웹 페이지 전송 | 상태 비저장 |
| HTTPS | 443 | 보안 웹 통신 | TLS/SSL 암호화 |
| FTP | 20/21 | 파일 전송 | 데이터/제어 분리 |
| SSH | 22 | 보안 원격 접속 | 암호화 터널 |
| SMTP | 25 | 메일 발송 | 텍스트 기반 |
| DNS | 53 | 도메인 이름 해석 | UDP 사용 |
| DHCP | 67/68 | IP 자동 할당 | 브로드캐스트 |

## 실제 구현 예시

### TypeScript로 TCP/IP 스택 구현

```typescript
// TCP/IP 계층 인터페이스
interface TCPIPLayer {
  name: string;
  processOutgoing(data: Buffer): Buffer;
  processIncoming(data: Buffer): Buffer;
}

// 패킷 구조체
interface IPPacket {
  version: number;
  sourceIP: string;
  destIP: string;
  protocol: number;
  ttl: number;
  data: Buffer;
}

interface TCPSegment {
  sourcePort: number;
  destPort: number;
  sequenceNumber: number;
  acknowledgmentNumber: number;
  flags: {
    SYN: boolean;
    ACK: boolean;
    FIN: boolean;
    RST: boolean;
    PSH: boolean;
    URG: boolean;
  };
  windowSize: number;
  data: Buffer;
}

// TCP/IP 스택 구현
class TCPIPStack {
  private layers: TCPIPLayer[] = [];

  constructor() {
    this.layers = [
      new NetworkInterfaceLayer(),
      new InternetLayer(),
      new TransportLayer(),
      new ApplicationLayer()
    ];
  }

  // 데이터 전송 (아래로)
  send(data: string): Buffer {
    let buffer = Buffer.from(data);
    
    // Application → Network Interface
    for (let i = this.layers.length - 1; i >= 0; i--) {
      buffer = this.layers[i].processOutgoing(buffer);
    }
    
    return buffer;
  }

  // 데이터 수신 (위로)
  receive(data: Buffer): string {
    let buffer = data;
    
    // Network Interface → Application
    for (let i = 0; i < this.layers.length; i++) {
      buffer = this.layers[i].processIncoming(buffer);
    }
    
    return buffer.toString();
  }
}

// Application Layer 구현
class ApplicationLayer implements TCPIPLayer {
  name = 'Application';

  processOutgoing(data: Buffer): Buffer {
    // HTTP 요청 생성 예시
    const httpRequest = 
      `GET / HTTP/1.1\r\n` +
      `Host: example.com\r\n` +
      `User-Agent: TypeScript-Client\r\n` +
      `Accept: text/html\r\n` +
      `\r\n` +
      data.toString();
    
    return Buffer.from(httpRequest);
  }

  processIncoming(data: Buffer): Buffer {
    // HTTP 응답 파싱
    const response = data.toString();
    const bodyStart = response.indexOf('\r\n\r\n') + 4;
    return Buffer.from(response.substring(bodyStart));
  }
}

// Transport Layer (TCP) 구현
class TransportLayer implements TCPIPLayer {
  name = 'Transport';
  private sequenceNumber = 1000;
  private acknowledgmentNumber = 0;

  processOutgoing(data: Buffer): Buffer {
    const segment: TCPSegment = {
      sourcePort: 54321,
      destPort: 80,
      sequenceNumber: this.sequenceNumber,
      acknowledgmentNumber: this.acknowledgmentNumber,
      flags: {
        SYN: false,
        ACK: false,
        FIN: false,
        RST: false,
        PSH: true,
        URG: false
      },
      windowSize: 65535,
      data: data
    };

    // TCP 헤더 생성 (간소화)
    const header = this.createTCPHeader(segment);
    this.sequenceNumber += data.length;
    
    return Buffer.concat([header, data]);
  }

  processIncoming(data: Buffer): Buffer {
    // TCP 헤더 파싱 (간소화)
    const headerSize = 20;
    const header = data.slice(0, headerSize);
    const payload = data.slice(headerSize);
    
    // 시퀀스 번호 확인 등의 처리
    const segment = this.parseTCPHeader(header);
    this.acknowledgmentNumber = segment.sequenceNumber + payload.length;
    
    return payload;
  }

  private createTCPHeader(segment: TCPSegment): Buffer {
    const header = Buffer.alloc(20);
    
    // Source Port
    header.writeUInt16BE(segment.sourcePort, 0);
    // Destination Port
    header.writeUInt16BE(segment.destPort, 2);
    // Sequence Number
    header.writeUInt32BE(segment.sequenceNumber, 4);
    // Acknowledgment Number
    header.writeUInt32BE(segment.acknowledgmentNumber, 8);
    // Data Offset and Flags
    const dataOffset = 5 << 4; // 20 bytes = 5 * 4
    const flags = this.encodeFlags(segment.flags);
    header.writeUInt16BE(dataOffset | flags, 12);
    // Window Size
    header.writeUInt16BE(segment.windowSize, 14);
    // Checksum (0 for simplicity)
    header.writeUInt16BE(0, 16);
    // Urgent Pointer
    header.writeUInt16BE(0, 18);
    
    return header;
  }

  private parseTCPHeader(header: Buffer): TCPSegment {
    return {
      sourcePort: header.readUInt16BE(0),
      destPort: header.readUInt16BE(2),
      sequenceNumber: header.readUInt32BE(4),
      acknowledgmentNumber: header.readUInt32BE(8),
      flags: this.decodeFlags(header.readUInt16BE(12) & 0x3F),
      windowSize: header.readUInt16BE(14),
      data: Buffer.alloc(0)
    };
  }

  private encodeFlags(flags: TCPSegment['flags']): number {
    let result = 0;
    if (flags.URG) result |= 0x20;
    if (flags.ACK) result |= 0x10;
    if (flags.PSH) result |= 0x08;
    if (flags.RST) result |= 0x04;
    if (flags.SYN) result |= 0x02;
    if (flags.FIN) result |= 0x01;
    return result;
  }

  private decodeFlags(value: number): TCPSegment['flags'] {
    return {
      URG: !!(value & 0x20),
      ACK: !!(value & 0x10),
      PSH: !!(value & 0x08),
      RST: !!(value & 0x04),
      SYN: !!(value & 0x02),
      FIN: !!(value & 0x01)
    };
  }
}

// Internet Layer (IP) 구현
class InternetLayer implements TCPIPLayer {
  name = 'Internet';
  private identification = 1;

  processOutgoing(data: Buffer): Buffer {
    const packet: IPPacket = {
      version: 4,
      sourceIP: '192.168.1.100',
      destIP: '93.184.216.34',
      protocol: 6, // TCP
      ttl: 64,
      data: data
    };

    const header = this.createIPHeader(packet);
    return Buffer.concat([header, data]);
  }

  processIncoming(data: Buffer): Buffer {
    // IP 헤더 파싱
    const headerSize = 20;
    const header = data.slice(0, headerSize);
    const payload = data.slice(headerSize);
    
    const packet = this.parseIPHeader(header);
    
    // TTL 확인
    if (packet.ttl <= 0) {
      throw new Error('TTL exceeded');
    }
    
    return payload;
  }

  private createIPHeader(packet: IPPacket): Buffer {
    const header = Buffer.alloc(20);
    
    // Version and Header Length
    header.writeUInt8((packet.version << 4) | 5, 0);
    // Type of Service
    header.writeUInt8(0, 1);
    // Total Length
    header.writeUInt16BE(20 + packet.data.length, 2);
    // Identification
    header.writeUInt16BE(this.identification++, 4);
    // Flags and Fragment Offset
    header.writeUInt16BE(0x4000, 6); // Don't Fragment
    // TTL
    header.writeUInt8(packet.ttl, 8);
    // Protocol
    header.writeUInt8(packet.protocol, 9);
    // Header Checksum (0 for simplicity)
    header.writeUInt16BE(0, 10);
    // Source IP
    this.writeIPAddress(header, 12, packet.sourceIP);
    // Destination IP
    this.writeIPAddress(header, 16, packet.destIP);
    
    return header;
  }

  private parseIPHeader(header: Buffer): IPPacket {
    const versionAndLength = header.readUInt8(0);
    return {
      version: versionAndLength >> 4,
      sourceIP: this.readIPAddress(header, 12),
      destIP: this.readIPAddress(header, 16),
      protocol: header.readUInt8(9),
      ttl: header.readUInt8(8),
      data: Buffer.alloc(0)
    };
  }

  private writeIPAddress(buffer: Buffer, offset: number, ip: string): void {
    const parts = ip.split('.').map(Number);
    parts.forEach((part, index) => {
      buffer.writeUInt8(part, offset + index);
    });
  }

  private readIPAddress(buffer: Buffer, offset: number): string {
    const parts: number[] = [];
    for (let i = 0; i < 4; i++) {
      parts.push(buffer.readUInt8(offset + i));
    }
    return parts.join('.');
  }
}

// Network Interface Layer 구현
class NetworkInterfaceLayer implements TCPIPLayer {
  name = 'Network Interface';

  processOutgoing(data: Buffer): Buffer {
    // Ethernet Frame 생성
    const frame = Buffer.alloc(14 + data.length + 4);
    
    // Destination MAC (6 bytes)
    frame.write('AABBCCDDEEFF', 0, 'hex');
    // Source MAC (6 bytes)
    frame.write('112233445566', 6, 'hex');
    // EtherType (2 bytes) - IPv4
    frame.writeUInt16BE(0x0800, 12);
    // Payload
    data.copy(frame, 14);
    // FCS (4 bytes) - simplified
    frame.writeUInt32BE(this.calculateCRC(data), 14 + data.length);
    
    return frame;
  }

  processIncoming(data: Buffer): Buffer {
    // Ethernet Frame 파싱
    const payload = data.slice(14, data.length - 4);
    const fcs = data.readUInt32BE(data.length - 4);
    
    // CRC 검증
    if (this.calculateCRC(payload) !== fcs) {
      throw new Error('Frame check sequence failed');
    }
    
    return payload;
  }

  private calculateCRC(data: Buffer): number {
    // 간단한 CRC 계산 (실제로는 CRC32 사용)
    let crc = 0;
    for (let i = 0; i < data.length; i++) {
      crc = (crc + data[i]) % 0xFFFFFFFF;
    }
    return crc;
  }
}

// DNS 클라이언트 예시
class DNSClient {
  private cache: Map<string, string> = new Map();

  async resolve(domain: string): Promise<string> {
    // 캐시 확인
    if (this.cache.has(domain)) {
      return this.cache.get(domain)!;
    }

    // DNS 쿼리 생성
    const query = this.createDNSQuery(domain);
    
    // UDP를 통해 DNS 서버로 전송 (포트 53)
    // 실제 구현에서는 dgram 모듈 사용
    const response = await this.sendDNSQuery(query);
    
    // 응답 파싱
    const ip = this.parseDNSResponse(response);
    
    // 캐시 저장
    this.cache.set(domain, ip);
    
    return ip;
  }

  private createDNSQuery(domain: string): Buffer {
    const query = Buffer.alloc(512);
    let offset = 0;
    
    // Transaction ID
    query.writeUInt16BE(Math.floor(Math.random() * 65536), offset);
    offset += 2;
    
    // Flags (Standard Query)
    query.writeUInt16BE(0x0100, offset);
    offset += 2;
    
    // Questions
    query.writeUInt16BE(1, offset);
    offset += 2;
    
    // Answer RRs
    query.writeUInt16BE(0, offset);
    offset += 2;
    
    // Authority RRs
    query.writeUInt16BE(0, offset);
    offset += 2;
    
    // Additional RRs
    query.writeUInt16BE(0, offset);
    offset += 2;
    
    // Query section
    const labels = domain.split('.');
    for (const label of labels) {
      query.writeUInt8(label.length, offset++);
      query.write(label, offset);
      offset += label.length;
    }
    query.writeUInt8(0, offset++); // Root label
    
    // Type A (IPv4 address)
    query.writeUInt16BE(1, offset);
    offset += 2;
    
    // Class IN (Internet)
    query.writeUInt16BE(1, offset);
    offset += 2;
    
    return query.slice(0, offset);
  }

  private async sendDNSQuery(query: Buffer): Promise<Buffer> {
    // 실제 구현에서는 UDP 소켓 사용
    // 여기서는 시뮬레이션
    return Buffer.from('mock DNS response');
  }

  private parseDNSResponse(response: Buffer): string {
    // DNS 응답 파싱 (간소화)
    // 실제로는 복잡한 파싱 필요
    return '93.184.216.34';
  }
}

// 사용 예제
async function main() {
  // TCP/IP 스택 초기화
  const stack = new TCPIPStack();
  
  // HTTP 요청 전송
  console.log('=== Sending HTTP Request ===');
  const request = 'GET /index.html HTTP/1.1\r\nHost: example.com\r\n\r\n';
  const transmitted = stack.send(request);
  console.log('Transmitted bytes:', transmitted.length);
  
  // DNS 해석
  const dns = new DNSClient();
  const ip = await dns.resolve('example.com');
  console.log(`DNS Resolution: example.com -> ${ip}`);
  
  // 응답 시뮬레이션
  const response = Buffer.from(
    'HTTP/1.1 200 OK\r\n' +
    'Content-Type: text/html\r\n' +
    '\r\n' +
    '<html><body>Hello, TCP/IP!</body></html>'
  );
  
  // 네트워크 시뮬레이션...
}

// main().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **실용성**: OSI보다 단순하고 실제 구현 중심
> 2. **4계층 구조**: Application, Transport, Internet, Network Interface
> 3. **핵심 프로토콜**: TCP/IP가 인터넷의 기반
> 4. **유연성**: 계층 간 명확한 인터페이스로 독립적 발전 가능
> 5. **현재 표준**: 전 세계 인터넷 통신의 표준 모델

## 참고 자료

### 📖 추가 학습
- [Internet Protocol Suite - Wikipedia](https://en.wikipedia.org/wiki/Internet_protocol_suite)
- [TCP/IP Model vs OSI Model](http://fiberbit.com.tw/tcpip-model-vs-osi-model/)
- [TCP/IP 네트워크 프로그래밍](https://www.joinc.co.kr/w/Site/Network_Programing/Documents/IntroTCPIP)

### 🔗 관련 문서
- [[02_osi-7-layer|OSI 7계층 모델]]
- [[05_tcp-udp|TCP와 UDP 프로토콜]]
- [[07_ip-addressing|IP 주소 체계]]
