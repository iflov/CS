---
title: OSI 7계층 모델
created: 2024-12-31
updated: 2024-12-31
tags:
  - CS
  - Network
  - OSI
  - Protocol
  - Layer
category: Network
status: 완료
---

# 🎯 OSI 7계층 모델 (OSI 7 Layer Model)

## 📌 개요

OSI(Open Systems Interconnection) 7계층 모델은 네트워크 통신을 **7개의 계층**으로 나누어 표준화한 참조 모델입니다. 각 계층은 독립적인 역할을 수행하며, 계층 간 추상화를 통해 네트워크 시스템의 복잡성을 관리합니다.

> [!info] 핵심 개념
> - **계층화(Layering)**: 복잡한 시스템을 관리 가능한 단위로 분할
> - **캡슐화(Encapsulation)**: 각 계층에서 헤더 정보 추가
> - **추상화(Abstraction)**: 하위 계층의 구현 세부사항 은닉

## 📚 목차

1. [OSI 7계층 구조](#osi-7계층-구조)
2. [각 계층의 역할과 프로토콜](#각-계층의-역할과-프로토콜)
3. [계층 간 통신 과정](#계층-간-통신-과정)
4. [실제 구현 예시](#실제-구현-예시)
5. [참고 자료](#참고-자료)

## OSI 7계층 구조

### 계층별 분류

```
┌─────────────────────────────────┐
│ 7. Application Layer            │ ← UPPER LAYERS
│ 6. Presentation Layer           │   (응용 계층)
│ 5. Session Layer                │   End-to-End
├─────────────────────────────────┤
│ 4. Transport Layer              │ ← TRANSPORT SERVICE
├─────────────────────────────────┤
│ 3. Network Layer                │ ← LOWER LAYERS
│ 2. Data Link Layer              │   (하위 계층)
│ 1. Physical Layer               │   Node-to-Node
└─────────────────────────────────┘
```

## 각 계층의 역할과 프로토콜

### 1️⃣ Physical Layer (물리 계층)
- **역할**: 비트 스트림의 전기적/물리적 전송
- **기능**: 전기 신호 변환, 비트 표현 방식 정의
- **프로토콜**: Ethernet, USB, Bluetooth 물리 규격
- **장비**: 케이블, 리피터, 허브

### 2️⃣ Data Link Layer (데이터 링크 계층)
- **역할**: 신뢰성 있는 Point-to-Point 통신
- **기능**: 오류 검출/수정, 흐름 제어
- **프로토콜**: Ethernet, PPP, WiFi (802.11)
- **장비**: 스위치, 브리지
- **주소**: MAC Address

### 3️⃣ Network Layer (네트워크 계층)
- **역할**: 네트워크 간 라우팅 및 경로 설정
- **기능**: 논리적 주소 지정, 패킷 전달
- **프로토콜**: IP (IPv4/IPv6), ICMP, ARP
- **장비**: 라우터, L3 스위치
- **주소**: IP Address

### 4️⃣ Transport Layer (전송 계층)
- **역할**: End-to-End 신뢰성 있는 데이터 전송
- **기능**: 오류 제어, 흐름 제어, 분할/재조립
- **프로토콜**: TCP, UDP
- **단위**: Segment (TCP), Datagram (UDP)

### 5️⃣ Session Layer (세션 계층)
- **역할**: 세션 설정, 관리, 종료
- **기능**: 동기화, 체크포인트, 세션 복구
- **프로토콜**: NetBIOS, SSH, SQL
- **예시**: 로그인 세션 관리

### 6️⃣ Presentation Layer (표현 계층)
- **역할**: 데이터 형식 변환 및 암호화
- **기능**: 인코딩/디코딩, 암호화/복호화, 압축
- **프로토콜**: SSL/TLS, JPEG, MPEG
- **예시**: ASCII ↔ EBCDIC 변환

### 7️⃣ Application Layer (응용 계층)
- **역할**: 사용자와 직접 상호작용
- **기능**: 네트워크 서비스 제공
- **프로토콜**: HTTP, FTP, SMTP, DNS, DHCP
- **예시**: 웹 브라우저, 이메일 클라이언트

## 계층 간 통신 과정

### 데이터 캡슐화 과정

```
송신측 (Encapsulation)           수신측 (Decapsulation)
━━━━━━━━━━━━━━━━━━━━━           ━━━━━━━━━━━━━━━━━━━━━
Application   [Data]        →    Application   [Data]
     ↓                                 ↑
Presentation  [Data]        →    Presentation  [Data]
     ↓                                 ↑
Session       [Data]        →    Session       [Data]
     ↓                                 ↑
Transport     [H4|Data]     →    Transport     [H4|Data]
     ↓                                 ↑
Network       [H3|H4|Data]  →    Network       [H3|H4|Data]
     ↓                                 ↑
Data Link     [H2|H3|H4|Data|T2] → Data Link  [H2|H3|H4|Data|T2]
     ↓                                 ↑
Physical      [Bits...]     →    Physical      [Bits...]
```

## 실제 구현 예시

### TypeScript로 OSI 계층 시뮬레이션

```typescript
// OSI 계층 인터페이스
interface Layer {
  name: string;
  number: number;
  process(data: any, direction: 'send' | 'receive'): any;
}

// 프로토콜 데이터 유닛 (PDU)
interface PDU {
  header?: any;
  data: any;
  trailer?: any;
}

// OSI 모델 구현
class OSIModel {
  private layers: Map<number, Layer>;

  constructor() {
    this.layers = new Map();
    this.initializeLayers();
  }

  private initializeLayers(): void {
    // 각 계층 초기화
    this.layers.set(7, new ApplicationLayer());
    this.layers.set(6, new PresentationLayer());
    this.layers.set(5, new SessionLayer());
    this.layers.set(4, new TransportLayer());
    this.layers.set(3, new NetworkLayer());
    this.layers.set(2, new DataLinkLayer());
    this.layers.set(1, new PhysicalLayer());
  }

  // 데이터 송신 (캡슐화)
  send(data: any): any {
    let processedData = data;
    
    // 7계층부터 1계층까지 순차 처리
    for (let i = 7; i >= 1; i--) {
      const layer = this.layers.get(i)!;
      processedData = layer.process(processedData, 'send');
      console.log(`Layer ${i} (${layer.name}) - Send:`, processedData);
    }
    
    return processedData;
  }

  // 데이터 수신 (역캡슐화)
  receive(data: any): any {
    let processedData = data;
    
    // 1계층부터 7계층까지 역순 처리
    for (let i = 1; i <= 7; i++) {
      const layer = this.layers.get(i)!;
      processedData = layer.process(processedData, 'receive');
      console.log(`Layer ${i} (${layer.name}) - Receive:`, processedData);
    }
    
    return processedData;
  }
}

// Application Layer 구현
class ApplicationLayer implements Layer {
  name = 'Application';
  number = 7;

  process(data: any, direction: 'send' | 'receive'): any {
    if (direction === 'send') {
      // HTTP 헤더 추가 예시
      return {
        protocol: 'HTTP',
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          'User-Agent': 'TypeScript-Client'
        },
        body: data
      };
    } else {
      // HTTP 응답 파싱
      return data.body || data;
    }
  }
}

// Transport Layer 구현
class TransportLayer implements Layer {
  name = 'Transport';
  number = 4;
  private sequenceNumber = 0;

  process(data: any, direction: 'send' | 'receive'): any {
    if (direction === 'send') {
      // TCP 세그먼트 생성
      return {
        header: {
          sourcePort: 3000,
          destPort: 80,
          sequenceNumber: this.sequenceNumber++,
          acknowledgment: 0,
          flags: { SYN: false, ACK: false, FIN: false },
          windowSize: 65535,
          checksum: this.calculateChecksum(data)
        },
        payload: data
      };
    } else {
      // TCP 세그먼트 검증
      if (this.validateChecksum(data)) {
        return data.payload;
      }
      throw new Error('Checksum validation failed');
    }
  }

  private calculateChecksum(data: any): number {
    // 간단한 체크섬 계산 (실제로는 더 복잡함)
    const str = JSON.stringify(data);
    let sum = 0;
    for (let i = 0; i < str.length; i++) {
      sum += str.charCodeAt(i);
    }
    return sum % 65536;
  }

  private validateChecksum(segment: any): boolean {
    const calculatedChecksum = this.calculateChecksum(segment.payload);
    return calculatedChecksum === segment.header.checksum;
  }
}

// Network Layer 구현
class NetworkLayer implements Layer {
  name = 'Network';
  number = 3;

  process(data: any, direction: 'send' | 'receive'): any {
    if (direction === 'send') {
      // IP 패킷 생성
      return {
        header: {
          version: 4,
          headerLength: 20,
          totalLength: JSON.stringify(data).length + 20,
          identification: Math.floor(Math.random() * 65536),
          flags: { DF: 0, MF: 0 },
          fragmentOffset: 0,
          ttl: 64,
          protocol: 6, // TCP
          checksum: 0,
          sourceIP: '192.168.1.100',
          destinationIP: '93.184.216.34' // example.com
        },
        payload: data
      };
    } else {
      // IP 패킷 검증 및 라우팅
      if (data.header.ttl <= 0) {
        throw new Error('TTL exceeded');
      }
      return data.payload;
    }
  }
}

// Data Link Layer 구현
class DataLinkLayer implements Layer {
  name = 'Data Link';
  number = 2;

  process(data: any, direction: 'send' | 'receive'): any {
    if (direction === 'send') {
      // 이더넷 프레임 생성
      return {
        header: {
          destinationMAC: 'AA:BB:CC:DD:EE:FF',
          sourceMAC: '11:22:33:44:55:66',
          etherType: 0x0800 // IPv4
        },
        payload: data,
        trailer: {
          fcs: this.calculateFCS(data) // Frame Check Sequence
        }
      };
    } else {
      // 프레임 검증
      if (this.validateFCS(data)) {
        return data.payload;
      }
      throw new Error('FCS validation failed');
    }
  }

  private calculateFCS(data: any): number {
    // CRC32 대신 간단한 FCS 계산
    return JSON.stringify(data).length % 256;
  }

  private validateFCS(frame: any): boolean {
    const calculatedFCS = this.calculateFCS(frame.payload);
    return calculatedFCS === frame.trailer.fcs;
  }
}

// Presentation Layer 구현
class PresentationLayer implements Layer {
  name = 'Presentation';
  number = 6;

  process(data: any, direction: 'send' | 'receive'): any {
    if (direction === 'send') {
      // 데이터 압축 및 암호화 (간단한 예시)
      const jsonString = JSON.stringify(data);
      return {
        encrypted: this.simpleEncrypt(jsonString),
        compressed: jsonString.length > 100
      };
    } else {
      // 복호화 및 압축 해제
      const decrypted = this.simpleDecrypt(data.encrypted);
      return JSON.parse(decrypted);
    }
  }

  private simpleEncrypt(text: string): string {
    // Base64 인코딩 (실제로는 복잡한 암호화 사용)
    return Buffer.from(text).toString('base64');
  }

  private simpleDecrypt(encrypted: string): string {
    return Buffer.from(encrypted, 'base64').toString('utf-8');
  }
}

// Session Layer 구현
class SessionLayer implements Layer {
  name = 'Session';
  number = 5;
  private sessionId: string | null = null;

  process(data: any, direction: 'send' | 'receive'): any {
    if (direction === 'send') {
      // 세션 설정
      if (!this.sessionId) {
        this.sessionId = this.generateSessionId();
      }
      return {
        sessionId: this.sessionId,
        timestamp: Date.now(),
        data: data
      };
    } else {
      // 세션 검증
      if (data.sessionId === this.sessionId) {
        return data.data;
      }
      throw new Error('Invalid session');
    }
  }

  private generateSessionId(): string {
    return Math.random().toString(36).substr(2, 9);
  }
}

// Physical Layer 구현
class PhysicalLayer implements Layer {
  name = 'Physical';
  number = 1;

  process(data: any, direction: 'send' | 'receive'): any {
    if (direction === 'send') {
      // 비트 스트림으로 변환
      const jsonString = JSON.stringify(data);
      return this.stringToBinary(jsonString);
    } else {
      // 비트 스트림에서 데이터로 변환
      const jsonString = this.binaryToString(data);
      return JSON.parse(jsonString);
    }
  }

  private stringToBinary(str: string): string {
    return str.split('').map(char => 
      char.charCodeAt(0).toString(2).padStart(8, '0')
    ).join('');
  }

  private binaryToString(binary: string): string {
    const bytes = binary.match(/.{1,8}/g) || [];
    return bytes.map(byte => 
      String.fromCharCode(parseInt(byte, 2))
    ).join('');
  }
}

// 사용 예제
const osiModel = new OSIModel();

// 데이터 전송 시뮬레이션
const originalData = { message: "Hello, Network!" };

console.log("=== 송신 과정 (Encapsulation) ===");
const transmittedData = osiModel.send(originalData);

console.log("\n=== 전송 중... ===\n");

console.log("=== 수신 과정 (Decapsulation) ===");
const receivedData = osiModel.receive(transmittedData);

console.log("\n최종 수신 데이터:", receivedData);
```

## 구글 검색 예시 (실제 통신 과정)

1. **Physical Layer**: WiFi 전파로 신호 전송
2. **Data Link Layer**: WiFi 카드가 MAC 주소로 라우터 식별
3. **Network Layer**: IP 주소로 구글 서버 위치 파악
4. **Transport Layer**: TCP로 신뢰성 있는 연결 설정
5. **Session Layer**: 검색 세션 관리, 상태 유지
6. **Presentation Layer**: HTTPS 암호화/복호화
7. **Application Layer**: HTTP 요청/응답 처리

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **상위 3계층** (5,6,7): End-to-End 통신, 사용자 중심
> 2. **Transport Layer** (4): 신뢰성 보장의 핵심
> 3. **하위 3계층** (1,2,3): Node-to-Node 통신, 하드웨어 중심
> 4. 각 계층은 **독립적**이며 **추상화**를 통해 복잡성 관리
> 5. 실제로는 **TCP/IP 4계층**이 더 널리 사용됨

## 참고 자료

### 📖 추가 학습
- [OSI Model - Wikipedia](https://en.wikipedia.org/wiki/OSI_model)
- [List of Network Protocols (OSI model)](https://en.wikipedia.org/wiki/List_of_network_protocols_(OSI_model))
- [OSI 7 계층 상세 설명](https://brunch.co.kr/@toughrogrammer/16)

### 🔗 관련 문서
- [[03_tcp-ip-model|TCP/IP 4계층 모델]]
- [[05_tcp-udp|TCP와 UDP 프로토콜]]
- [[04_switching-techniques|스위칭 기법]]
