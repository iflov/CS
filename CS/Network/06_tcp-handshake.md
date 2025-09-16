---
title: TCP Handshake
created: 2024-12-31
updated: 2024-12-31
tags:
  - CS
  - Network
  - TCP
  - Handshake
  - Connection
category: Network
status: 완료
---

# 🤝 TCP Handshake

## 📌 개요

TCP Handshake는 신뢰성 있는 연결을 설정하고 종료하는 과정입니다. **3-way handshake**로 연결을 맺고, **4-way handshake**로 연결을 종료합니다. 이를 통해 양쪽이 통신 준비가 되었음을 확인합니다.

> [!info] 핵심 개념
> - **3-way Handshake**: 연결 설정 (SYN → SYN-ACK → ACK)
> - **4-way Handshake**: 연결 종료 (FIN → ACK → FIN → ACK)
> - **Sequence Number**: 데이터 순서 보장을 위한 번호

## 📚 목차

1. [3-Way Handshake (연결 설정)](#3-way-handshake-연결-설정)
2. [4-Way Handshake (연결 종료)](#4-way-handshake-연결-종료)
3. [TCP 플래그와 상태](#tcp-플래그와-상태)
4. [시퀀스 번호와 확인 번호](#시퀀스-번호와-확인-번호)
5. [실제 구현 예시](#실제-구현-예시)
6. [참고 자료](#참고-자료)

## 3-Way Handshake (연결 설정)

### 과정

```
Client                          Server
  |                                |
  |  1. SYN (SEQ=x)              →|
  |     "연결하고 싶어요"           |
  |                                |
  |← 2. SYN-ACK (SEQ=y, ACK=x+1)  |
  |     "좋아요, 저도 연결할게요"     |
  |                                |
  |  3. ACK (ACK=y+1)            →|
  |     "확인했어요"                |
  |                                |
  |      [Connection Established]  |
```

### 단계별 설명

1. **SYN (Synchronize)**
   - 클라이언트가 연결 요청
   - 초기 시퀀스 번호(ISN) 전송
   - 상태: `CLOSED` → `SYN_SENT`

2. **SYN-ACK (Synchronize-Acknowledge)**
   - 서버가 연결 수락
   - 서버의 ISN 전송 + 클라이언트 SYN 확인
   - 상태: `LISTEN` → `SYN_RECEIVED`

3. **ACK (Acknowledge)**
   - 클라이언트가 서버의 SYN 확인
   - 연결 설정 완료
   - 상태: `SYN_SENT` → `ESTABLISHED`

## 4-Way Handshake (연결 종료)

### 과정

```
Client                          Server
  |                                |
  |  1. FIN (SEQ=x)              →|
  |     "연결 종료하고 싶어요"        |
  |                                |
  |← 2. ACK (ACK=x+1)             |
  |     "알겠어요, 잠시만요"          |
  |                                |
  |← 3. FIN (SEQ=y)                |
  |     "저도 종료할게요"            |
  |                                |
  |  4. ACK (ACK=y+1)            →|
  |     "확인했어요"                |
  |                                |
  |     [Connection Closed]        |
```

### 단계별 설명

1. **FIN (Finish)**
   - 클라이언트가 종료 요청
   - 상태: `ESTABLISHED` → `FIN_WAIT_1`

2. **ACK**
   - 서버가 종료 요청 확인
   - 상태: `ESTABLISHED` → `CLOSE_WAIT`

3. **FIN**
   - 서버가 종료 준비 완료
   - 상태: `CLOSE_WAIT` → `LAST_ACK`

4. **ACK**
   - 클라이언트가 최종 확인
   - 상태: `FIN_WAIT_2` → `TIME_WAIT` → `CLOSED`

## TCP 플래그와 상태

### TCP 플래그 (6 bits)

| 플래그 | 이름 | 설명 |
|--------|------|------|
| **URG** | Urgent | 긴급 데이터 포함 |
| **ACK** | Acknowledgment | 확인 번호 유효 |
| **PSH** | Push | 즉시 전달 요청 |
| **RST** | Reset | 연결 강제 종료 |
| **SYN** | Synchronize | 연결 시작 |
| **FIN** | Finish | 연결 종료 |

### TCP 상태 전이

```
         CLOSED
            ↓
         LISTEN
         ↙    ↘
   SYN_RECV  SYN_SENT
         ↘    ↙
      ESTABLISHED
       ↙        ↘
  FIN_WAIT_1  CLOSE_WAIT
       ↓          ↓
  FIN_WAIT_2  LAST_ACK
       ↓          ↓
   TIME_WAIT   CLOSED
       ↓
    CLOSED
```

## 시퀀스 번호와 확인 번호

### Sequence Number (SEQ)
- 송신한 데이터의 **바이트 번호**
- 초기값은 랜덤 (보안상 이유)
- 데이터 전송 시 증가

### Acknowledgment Number (ACK)
- 다음에 받을 것으로 **예상되는 바이트 번호**
- SEQ + 데이터 길이 + 1
- 수신 확인 용도

## 실제 구현 예시

### TypeScript로 TCP Handshake 구현

```typescript
// TCP 플래그 열거형
enum TCPFlags {
  URG = 0x20,
  ACK = 0x10,
  PSH = 0x08,
  RST = 0x04,
  SYN = 0x02,
  FIN = 0x01
}

// TCP 상태 열거형
enum TCPState {
  CLOSED = 'CLOSED',
  LISTEN = 'LISTEN',
  SYN_SENT = 'SYN_SENT',
  SYN_RECEIVED = 'SYN_RECEIVED',
  ESTABLISHED = 'ESTABLISHED',
  FIN_WAIT_1 = 'FIN_WAIT_1',
  FIN_WAIT_2 = 'FIN_WAIT_2',
  CLOSE_WAIT = 'CLOSE_WAIT',
  CLOSING = 'CLOSING',
  LAST_ACK = 'LAST_ACK',
  TIME_WAIT = 'TIME_WAIT'
}

// TCP 세그먼트 인터페이스
interface TCPSegment {
  sourcePort: number;
  destPort: number;
  sequenceNumber: number;
  acknowledgmentNumber: number;
  flags: number;
  windowSize: number;
  data?: Buffer;
}

// TCP 연결 클래스
class TCPConnection {
  private state: TCPState = TCPState.CLOSED;
  private sequenceNumber: number;
  private acknowledgmentNumber: number;
  private initialSequenceNumber: number;
  private remoteSequenceNumber: number = 0;
  
  constructor(private localPort: number, private remotePort: number) {
    // 초기 시퀀스 번호를 랜덤으로 생성 (보안)
    this.initialSequenceNumber = Math.floor(Math.random() * 0xFFFFFFFF);
    this.sequenceNumber = this.initialSequenceNumber;
    this.acknowledgmentNumber = 0;
  }

  // 3-Way Handshake - 클라이언트
  async connect(remoteAddress: string): Promise<void> {
    console.log(`[${this.state}] Initiating connection to ${remoteAddress}:${this.remotePort}`);
    
    // Step 1: Send SYN
    const synSegment = this.createSegment(TCPFlags.SYN);
    this.state = TCPState.SYN_SENT;
    console.log(`[${this.state}] → SYN sent (SEQ=${synSegment.sequenceNumber})`);
    
    // Simulate sending and receiving
    const synAck = await this.sendAndWaitForResponse(synSegment, TCPFlags.SYN | TCPFlags.ACK);
    
    // Step 2: Receive SYN-ACK
    if (synAck) {
      this.remoteSequenceNumber = synAck.sequenceNumber;
      this.acknowledgmentNumber = synAck.sequenceNumber + 1;
      console.log(`[${this.state}] ← SYN-ACK received (SEQ=${synAck.sequenceNumber}, ACK=${synAck.acknowledgmentNumber})`);
      
      // Step 3: Send ACK
      this.sequenceNumber++;
      const ackSegment = this.createSegment(TCPFlags.ACK);
      this.state = TCPState.ESTABLISHED;
      await this.send(ackSegment);
      console.log(`[${this.state}] → ACK sent (ACK=${ackSegment.acknowledgmentNumber})`);
      console.log(`[${this.state}] ✅ Connection established!`);
    }
  }

  // 3-Way Handshake - 서버
  async listen(): Promise<void> {
    this.state = TCPState.LISTEN;
    console.log(`[${this.state}] Listening on port ${this.localPort}`);
    
    // Simulate receiving SYN
    const syn = await this.waitForSegment(TCPFlags.SYN);
    
    if (syn) {
      // Step 1: Receive SYN
      this.remoteSequenceNumber = syn.sequenceNumber;
      this.acknowledgmentNumber = syn.sequenceNumber + 1;
      console.log(`[${this.state}] ← SYN received (SEQ=${syn.sequenceNumber})`);
      
      // Step 2: Send SYN-ACK
      const synAckSegment = this.createSegment(TCPFlags.SYN | TCPFlags.ACK);
      this.state = TCPState.SYN_RECEIVED;
      await this.send(synAckSegment);
      console.log(`[${this.state}] → SYN-ACK sent (SEQ=${synAckSegment.sequenceNumber}, ACK=${synAckSegment.acknowledgmentNumber})`);
      
      // Step 3: Receive ACK
      const ack = await this.waitForSegment(TCPFlags.ACK);
      if (ack) {
        this.sequenceNumber++;
        this.state = TCPState.ESTABLISHED;
        console.log(`[${this.state}] ← ACK received (ACK=${ack.acknowledgmentNumber})`);
        console.log(`[${this.state}] ✅ Connection accepted!`);
      }
    }
  }

  // 4-Way Handshake - 연결 종료 시작
  async close(): Promise<void> {
    if (this.state !== TCPState.ESTABLISHED) {
      console.log(`Cannot close: connection not established`);
      return;
    }

    console.log(`[${this.state}] Initiating connection close`);
    
    // Step 1: Send FIN
    const finSegment = this.createSegment(TCPFlags.FIN | TCPFlags.ACK);
    this.state = TCPState.FIN_WAIT_1;
    await this.send(finSegment);
    console.log(`[${this.state}] → FIN sent (SEQ=${finSegment.sequenceNumber})`);
    
    // Step 2: Receive ACK
    const ack = await this.waitForSegment(TCPFlags.ACK);
    if (ack) {
      this.state = TCPState.FIN_WAIT_2;
      console.log(`[${this.state}] ← ACK received (ACK=${ack.acknowledgmentNumber})`);
      
      // Step 3: Receive FIN
      const fin = await this.waitForSegment(TCPFlags.FIN);
      if (fin) {
        console.log(`[${this.state}] ← FIN received (SEQ=${fin.sequenceNumber})`);
        this.acknowledgmentNumber = fin.sequenceNumber + 1;
        
        // Step 4: Send ACK
        const finalAck = this.createSegment(TCPFlags.ACK);
        await this.send(finalAck);
        console.log(`[${this.state}] → ACK sent (ACK=${finalAck.acknowledgmentNumber})`);
        
        // Enter TIME_WAIT
        this.state = TCPState.TIME_WAIT;
        console.log(`[${this.state}] Entering TIME_WAIT (2 MSL)`);
        
        // Wait 2 MSL (Maximum Segment Lifetime)
        await this.wait2MSL();
        
        this.state = TCPState.CLOSED;
        console.log(`[${this.state}] ✅ Connection closed`);
      }
    }
  }

  // 세그먼트 생성
  private createSegment(flags: number, data?: Buffer): TCPSegment {
    return {
      sourcePort: this.localPort,
      destPort: this.remotePort,
      sequenceNumber: this.sequenceNumber,
      acknowledgmentNumber: this.acknowledgmentNumber,
      flags: flags,
      windowSize: 65535,
      data: data
    };
  }

  // 세그먼트 전송 시뮬레이션
  private async send(segment: TCPSegment): Promise<void> {
    // 실제로는 네트워크로 전송
    await this.simulateNetworkDelay();
  }

  // 응답 대기 시뮬레이션
  private async sendAndWaitForResponse(
    segment: TCPSegment, 
    expectedFlags: number
  ): Promise<TCPSegment | null> {
    await this.send(segment);
    return this.waitForSegment(expectedFlags);
  }

  // 특정 플래그의 세그먼트 대기
  private async waitForSegment(expectedFlags: number): Promise<TCPSegment | null> {
    // 실제로는 소켓에서 수신 대기
    await this.simulateNetworkDelay();
    
    // 시뮬레이션: 예상된 세그먼트 반환
    return this.createSegment(expectedFlags);
  }

  // 네트워크 지연 시뮬레이션
  private async simulateNetworkDelay(): Promise<void> {
    await new Promise(resolve => setTimeout(resolve, Math.random() * 100 + 50));
  }

  // 2 MSL 대기
  private async wait2MSL(): Promise<void> {
    const MSL = 2000; // 2초 (실제로는 2분)
    await new Promise(resolve => setTimeout(resolve, 2 * MSL));
  }

  // 상태 확인
  getState(): TCPState {
    return this.state;
  }

  // RST로 강제 종료
  async reset(): Promise<void> {
    const rstSegment = this.createSegment(TCPFlags.RST);
    await this.send(rstSegment);
    this.state = TCPState.CLOSED;
    console.log(`[${this.state}] Connection reset`);
  }
}

// Half-Close 구현
class TCPHalfClose {
  private canSend: boolean = true;
  private canReceive: boolean = true;

  // 송신만 종료
  shutdownSend(): void {
    this.canSend = false;
    console.log('Shutdown send: can still receive data');
  }

  // 수신만 종료
  shutdownReceive(): void {
    this.canReceive = false;
    console.log('Shutdown receive: can still send data');
  }

  // 상태 확인
  getStatus(): { canSend: boolean; canReceive: boolean } {
    return {
      canSend: this.canSend,
      canReceive: this.canReceive
    };
  }
}

// TCP 옵션 처리
class TCPOptions {
  // MSS (Maximum Segment Size) 협상
  static negotiateMSS(localMSS: number, remoteMSS: number): number {
    return Math.min(localMSS, remoteMSS);
  }

  // Window Scaling
  static calculateWindowScale(desiredWindow: number): number {
    let scale = 0;
    let window = 65535;
    
    while (window < desiredWindow && scale < 14) {
      scale++;
      window = 65535 << scale;
    }
    
    return scale;
  }

  // Selective ACK (SACK)
  static generateSACK(receivedBlocks: Array<[number, number]>): Buffer {
    const sack = Buffer.allocUnsafe(receivedBlocks.length * 8);
    
    receivedBlocks.forEach(([left, right], index) => {
      sack.writeUInt32BE(left, index * 8);
      sack.writeUInt32BE(right, index * 8 + 4);
    });
    
    return sack;
  }
}

// 시나리오 테스트
class HandshakeScenarios {
  // 정상적인 연결
  static async normalConnection(): Promise<void> {
    console.log('=== Normal Connection Scenario ===\n');
    
    const client = new TCPConnection(54321, 80);
    const server = new TCPConnection(80, 54321);
    
    // 병렬로 서버 리스닝과 클라이언트 연결 시작
    await Promise.all([
      server.listen(),
      client.connect('192.168.1.1')
    ]);
    
    // 데이터 전송 시뮬레이션
    console.log('\n--- Data Transfer ---');
    console.log('Sending HTTP request...');
    console.log('Receiving HTTP response...');
    
    // 연결 종료
    console.log('\n--- Connection Closing ---');
    await client.close();
  }

  // SYN Flood 공격 시뮬레이션
  static async synFloodAttack(): Promise<void> {
    console.log('=== SYN Flood Attack Scenario ===\n');
    
    const attackers: TCPConnection[] = [];
    
    // 여러 개의 SYN만 보내고 ACK는 보내지 않음
    for (let i = 0; i < 5; i++) {
      const attacker = new TCPConnection(50000 + i, 80);
      attackers.push(attacker);
      
      // SYN만 보내고 중단
      console.log(`Attacker ${i}: Sending SYN from port ${50000 + i}`);
      // ACK를 보내지 않아 서버는 SYN_RECEIVED 상태에 머물게 됨
    }
    
    console.log('\nServer resources exhausted - SYN_RECEIVED queue full');
  }

  // Connection Reset
  static async connectionReset(): Promise<void> {
    console.log('=== Connection Reset Scenario ===\n');
    
    const connection = new TCPConnection(54321, 80);
    
    // 연결 시도 중 RST
    console.log('Attempting connection...');
    console.log('Unexpected error occurred!');
    await connection.reset();
  }

  // TIME_WAIT 상태 처리
  static demonstrateTIMEWAIT(): void {
    console.log('=== TIME_WAIT State ===\n');
    console.log('Purpose of TIME_WAIT:');
    console.log('1. Ensure the remote TCP received the final ACK');
    console.log('2. Allow old duplicate segments to expire');
    console.log('3. Prevent port reuse issues');
    console.log(`\nDuration: 2 * MSL (typically 2 * 2 minutes = 4 minutes)`);
  }
}

// 메인 실행
async function main() {
  // 정상 연결 시나리오
  await HandshakeScenarios.normalConnection();
  
  console.log('\n' + '='.repeat(50) + '\n');
  
  // SYN Flood 공격 시나리오
  await HandshakeScenarios.synFloodAttack();
  
  console.log('\n' + '='.repeat(50) + '\n');
  
  // Connection Reset 시나리오
  await HandshakeScenarios.connectionReset();
  
  console.log('\n' + '='.repeat(50) + '\n');
  
  // TIME_WAIT 설명
  HandshakeScenarios.demonstrateTIMEWAIT();
  
  // TCP 옵션 예제
  console.log('\n=== TCP Options ===');
  const mss = TCPOptions.negotiateMSS(1460, 1400);
  console.log(`Negotiated MSS: ${mss} bytes`);
  
  const scale = TCPOptions.calculateWindowScale(1048576);
  console.log(`Window Scale: ${scale} (window = 65535 << ${scale})`);
}

// main().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **3-Way Handshake**: SYN → SYN-ACK → ACK로 연결 설정
> 2. **4-Way Handshake**: FIN → ACK → FIN → ACK로 연결 종료
> 3. **Sequence Number**: 데이터 순서와 신뢰성 보장
> 4. **TIME_WAIT**: 늦은 패킷 처리를 위한 대기 상태
> 5. **보안**: SYN Flood 등의 공격 고려 필요

## 참고 자료

### 📖 추가 학습
- [TCP 3-Way Handshake - Wikipedia](https://en.wikipedia.org/wiki/Handshaking)
- [TCP Connection Termination](https://www.geeksforgeeks.org/tcp-connection-termination/)
- [네트워크 쉽게 이해하기 - TCP Handshake](http://mindnet.tistory.com/entry/네트워크-쉽게-이해하기-22편-TCP-3-WayHandshake-4-WayHandshak)

### 🔗 관련 문서
- [[05_tcp-udp|TCP와 UDP 프로토콜]]
- [[09_port-socket|포트와 소켓]]
- [[03_tcp-ip-model|TCP/IP 4계층 모델]]
