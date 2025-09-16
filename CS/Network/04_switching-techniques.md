---
title: 스위칭 기법
created: 2024-12-31
updated: 2024-12-31
tags:
  - CS
  - Network
  - Switching
  - Packet
  - Circuit
  - Multiplexing
category: Network
status: 완료
---

# 🔄 스위칭 기법 (Switching Techniques)

## 📌 개요

스위칭은 네트워크에서 데이터를 **출발지에서 목적지까지 전달**하는 기법입니다. 크게 회선 교환(Circuit Switching)과 패킷 교환(Packet Switching)으로 나뉘며, 각각의 장단점에 따라 적절한 용도로 사용됩니다.

> [!info] 핵심 개념
> - **Circuit Switching**: 전용 회선을 할당받아 연결 유지
> - **Packet Switching**: 데이터를 작은 패킷으로 나누어 전송
> - **Multiplexing**: 하나의 채널로 여러 신호를 동시 전송

## 📚 목차

1. [Circuit Switching (회선 교환)](#circuit-switching-회선-교환)
2. [Packet Switching (패킷 교환)](#packet-switching-패킷-교환)
3. [Message Switching (메시지 교환)](#message-switching-메시지-교환)
4. [Multiplexing (다중화)](#multiplexing-다중화)
5. [실제 구현 예시](#실제-구현-예시)
6. [참고 자료](#참고-자료)

## Circuit Switching (회선 교환)

### 특징
- **전용 회선** 할당으로 연결 지속
- **일정한 품질** 보장
- 연결 후 추가 관리 불필요
- **대표 예시**: 전화 시스템

### 장점 ✅
- 일정한 대역폭과 지연 시간
- 데이터 순서 보장
- 실시간 통신에 적합

### 단점 ❌
- 회선 낭비 (사용하지 않을 때도 점유)
- 초기 연결 설정 시간 필요
- 확장성 제한

### 동작 과정
1. **연결 설정**: 전용 경로 확립
2. **데이터 전송**: 확립된 경로로만 전송
3. **연결 해제**: 통신 종료 후 회선 반환

## Packet Switching (패킷 교환)

### 특징
- 데이터를 작은 **패킷**으로 분할
- 각 패킷은 **독립적으로** 전송
- 패킷 헤더에 목적지 정보 포함
- **대표 예시**: 인터넷

### 장점 ✅
- 효율적인 대역폭 활용
- 여러 경로 활용 가능 (다중 경로)
- 오류 발생 시 해당 패킷만 재전송

### 단점 ❌
- 패킷 순서 보장 필요
- 각 패킷마다 헤더 오버헤드
- 가변적인 지연 시간

### 패킷 구조
```
┌─────────┬──────────┐
│ Header  │ Payload  │
├─────────┼──────────┤
│ 출발지   │          │
│ 목적지   │  데이터   │
│ 순서번호  │          │
└─────────┴──────────┘
```

## Message Switching (메시지 교환)

### 특징
- **전체 메시지**를 하나로 전송
- Store-and-Forward 방식
- 패킷 교환의 전신

### 장점 ✅
- 구현이 단순
- 메시지 무결성 유지

### 단점 ❌
- 큰 메시지 전송 시 긴 대기 시간
- 중간 노드에 큰 저장 공간 필요
- 오류 시 전체 메시지 재전송

## Multiplexing (다중화)

### FDM (Frequency Division Multiplexing)
- **주파수**를 나누어 동시 전송
- 아날로그 신호에 주로 사용
- 예: 라디오 방송, 케이블 TV

### TDM (Time Division Multiplexing)
- **시간**을 나누어 순차 전송
- 디지털 신호에 적합
- 예: 디지털 전화 시스템

### WDM (Wavelength Division Multiplexing)
- **파장**을 나누어 전송
- 광섬유 통신에 사용
- 높은 대역폭 제공

## 실제 구현 예시

### TypeScript로 스위칭 기법 구현

```typescript
// 패킷 인터페이스
interface Packet {
  id: number;
  source: string;
  destination: string;
  sequenceNumber: number;
  totalPackets: number;
  data: Buffer;
  timestamp: number;
}

// 메시지 인터페이스
interface Message {
  id: string;
  source: string;
  destination: string;
  data: string;
}

// Circuit Switching 구현
class CircuitSwitch {
  private circuits: Map<string, {
    path: string[];
    bandwidth: number;
    established: Date;
  }> = new Map();

  // 회선 설정
  establishCircuit(
    source: string,
    destination: string,
    bandwidth: number
  ): string {
    const circuitId = `${source}-${destination}-${Date.now()}`;
    
    // 경로 찾기 (간소화)
    const path = this.findPath(source, destination);
    
    // 회선 할당
    this.circuits.set(circuitId, {
      path: path,
      bandwidth: bandwidth,
      established: new Date()
    });
    
    console.log(`Circuit established: ${circuitId}`);
    return circuitId;
  }

  // 회선 해제
  releaseCircuit(circuitId: string): void {
    if (this.circuits.has(circuitId)) {
      this.circuits.delete(circuitId);
      console.log(`Circuit released: ${circuitId}`);
    }
  }

  // 데이터 전송
  transmit(circuitId: string, data: Buffer): void {
    const circuit = this.circuits.get(circuitId);
    if (!circuit) {
      throw new Error('Circuit not found');
    }
    
    // 전용 회선으로 데이터 전송
    console.log(`Transmitting ${data.length} bytes over circuit ${circuitId}`);
  }

  private findPath(source: string, destination: string): string[] {
    // 간단한 경로 찾기 (실제로는 라우팅 알고리즘 사용)
    return [source, 'switch1', 'switch2', destination];
  }
}

// Packet Switching 구현
class PacketSwitch {
  private packetBuffer: Map<string, Packet[]> = new Map();
  private packetId = 0;
  private maxPacketSize = 1500; // MTU

  // 메시지를 패킷으로 분할
  fragmentMessage(message: Message): Packet[] {
    const data = Buffer.from(message.data);
    const packets: Packet[] = [];
    const totalPackets = Math.ceil(data.length / this.maxPacketSize);
    
    for (let i = 0; i < totalPackets; i++) {
      const start = i * this.maxPacketSize;
      const end = Math.min(start + this.maxPacketSize, data.length);
      const chunk = data.slice(start, end);
      
      packets.push({
        id: this.packetId++,
        source: message.source,
        destination: message.destination,
        sequenceNumber: i,
        totalPackets: totalPackets,
        data: chunk,
        timestamp: Date.now()
      });
    }
    
    return packets;
  }

  // 패킷 전송
  sendPacket(packet: Packet): void {
    // 라우팅 테이블 조회
    const nextHop = this.routePacket(packet);
    
    console.log(
      `Routing packet ${packet.id} ` +
      `(${packet.sequenceNumber}/${packet.totalPackets}) ` +
      `to ${nextHop}`
    );
    
    // 버퍼에 저장 (시뮬레이션)
    const key = `${packet.source}-${packet.destination}`;
    if (!this.packetBuffer.has(key)) {
      this.packetBuffer.set(key, []);
    }
    this.packetBuffer.get(key)!.push(packet);
  }

  // 패킷 재조립
  reassemblePackets(source: string, destination: string): Buffer | null {
    const key = `${source}-${destination}`;
    const packets = this.packetBuffer.get(key);
    
    if (!packets || packets.length === 0) {
      return null;
    }
    
    // 첫 패킷의 전체 개수 확인
    const totalPackets = packets[0].totalPackets;
    
    if (packets.length < totalPackets) {
      return null; // 아직 모든 패킷이 도착하지 않음
    }
    
    // 순서대로 정렬
    packets.sort((a, b) => a.sequenceNumber - b.sequenceNumber);
    
    // 데이터 재조립
    const buffers = packets.map(p => p.data);
    const reassembled = Buffer.concat(buffers);
    
    // 버퍼 정리
    this.packetBuffer.delete(key);
    
    return reassembled;
  }

  private routePacket(packet: Packet): string {
    // 간단한 라우팅 (실제로는 라우팅 테이블 사용)
    return `router-${packet.destination}`;
  }
}

// Message Switching 구현
class MessageSwitch {
  private messageQueue: Message[] = [];
  private storage = new Map<string, Message>();

  // Store-and-Forward
  storeAndForward(message: Message): void {
    // 메시지 저장
    this.storage.set(message.id, message);
    console.log(`Message ${message.id} stored`);
    
    // 큐에 추가
    this.messageQueue.push(message);
    
    // 전송 처리
    this.processQueue();
  }

  private processQueue(): void {
    while (this.messageQueue.length > 0) {
      const message = this.messageQueue.shift()!;
      
      // 다음 홉으로 전송
      const nextHop = this.findNextHop(message.destination);
      this.forwardMessage(message, nextHop);
      
      // 저장소에서 제거
      this.storage.delete(message.id);
    }
  }

  private findNextHop(destination: string): string {
    // 라우팅 로직
    return `node-${destination}`;
  }

  private forwardMessage(message: Message, nextHop: string): void {
    console.log(`Forwarding message ${message.id} to ${nextHop}`);
  }
}

// Multiplexing 구현
class Multiplexer {
  // FDM (Frequency Division Multiplexing)
  fdmMultiplex(
    signals: Array<{ frequency: number; data: Buffer }>
  ): Buffer {
    // 각 신호를 다른 주파수 대역에 할당
    const multiplexed = Buffer.alloc(
      signals.reduce((sum, s) => sum + s.data.length, 0)
    );
    
    let offset = 0;
    signals.forEach(signal => {
      // 주파수 변조 (시뮬레이션)
      console.log(`FDM: Signal at ${signal.frequency} Hz`);
      signal.data.copy(multiplexed, offset);
      offset += signal.data.length;
    });
    
    return multiplexed;
  }

  // TDM (Time Division Multiplexing)
  tdmMultiplex(
    channels: Buffer[],
    slotDuration: number
  ): Buffer[] {
    const frames: Buffer[] = [];
    const maxLength = Math.max(...channels.map(c => c.length));
    
    // 각 시간 슬롯에 채널 할당
    for (let time = 0; time < maxLength; time += slotDuration) {
      const frame = Buffer.alloc(channels.length * slotDuration);
      
      channels.forEach((channel, index) => {
        const start = time;
        const end = Math.min(time + slotDuration, channel.length);
        
        if (start < channel.length) {
          const data = channel.slice(start, end);
          data.copy(frame, index * slotDuration);
        }
      });
      
      frames.push(frame);
    }
    
    return frames;
  }

  // WDM (Wavelength Division Multiplexing)
  wdmMultiplex(
    opticalSignals: Array<{ wavelength: number; data: Buffer }>
  ): Buffer {
    // 광섬유에서 다른 파장으로 전송
    console.log('WDM Multiplexing:');
    opticalSignals.forEach(signal => {
      console.log(`  λ = ${signal.wavelength} nm: ${signal.data.length} bytes`);
    });
    
    // 모든 신호를 합성 (시뮬레이션)
    return Buffer.concat(opticalSignals.map(s => s.data));
  }
}

// 성능 비교 클래스
class SwitchingComparison {
  // 지연 시간 비교
  static compareLatency(): void {
    console.log('=== Latency Comparison ===');
    console.log('Circuit Switching: 고정 (낮음)');
    console.log('Packet Switching: 가변 (중간)');
    console.log('Message Switching: 높음');
  }

  // 대역폭 활용도 비교
  static compareBandwidthUtilization(): void {
    console.log('=== Bandwidth Utilization ===');
    console.log('Circuit Switching: 낮음 (전용 회선)');
    console.log('Packet Switching: 높음 (공유)');
    console.log('Message Switching: 중간');
  }

  // 적합한 용도
  static compareUseCases(): void {
    console.log('=== Best Use Cases ===');
    console.log('Circuit Switching: 실시간 통신 (전화, 화상 회의)');
    console.log('Packet Switching: 데이터 통신 (인터넷, 이메일)');
    console.log('Message Switching: 대용량 파일 전송');
  }
}

// 사용 예제
function demonstrateSwitching() {
  console.log('=== Circuit Switching Demo ===');
  const circuitSwitch = new CircuitSwitch();
  const circuitId = circuitSwitch.establishCircuit('NodeA', 'NodeB', 1000);
  circuitSwitch.transmit(circuitId, Buffer.from('Hello via Circuit'));
  circuitSwitch.releaseCircuit(circuitId);

  console.log('\n=== Packet Switching Demo ===');
  const packetSwitch = new PacketSwitch();
  const message: Message = {
    id: 'msg-001',
    source: 'ClientA',
    destination: 'ServerB',
    data: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. '.repeat(100)
  };
  
  const packets = packetSwitch.fragmentMessage(message);
  console.log(`Message fragmented into ${packets.length} packets`);
  
  // 패킷 전송 (순서 섞어서)
  const shuffled = [...packets].sort(() => Math.random() - 0.5);
  shuffled.forEach(packet => packetSwitch.sendPacket(packet));
  
  // 재조립
  const reassembled = packetSwitch.reassemblePackets('ClientA', 'ServerB');
  if (reassembled) {
    console.log(`Message reassembled: ${reassembled.length} bytes`);
  }

  console.log('\n=== Message Switching Demo ===');
  const messageSwitch = new MessageSwitch();
  messageSwitch.storeAndForward(message);

  console.log('\n=== Multiplexing Demo ===');
  const multiplexer = new Multiplexer();
  
  // FDM
  const fdmSignals = [
    { frequency: 100, data: Buffer.from('Channel 1') },
    { frequency: 200, data: Buffer.from('Channel 2') },
    { frequency: 300, data: Buffer.from('Channel 3') }
  ];
  multiplexer.fdmMultiplex(fdmSignals);
  
  // TDM
  const tdmChannels = [
    Buffer.from('Data from User 1'),
    Buffer.from('Data from User 2'),
    Buffer.from('Data from User 3')
  ];
  const frames = multiplexer.tdmMultiplex(tdmChannels, 4);
  console.log(`TDM: Created ${frames.length} frames`);

  console.log('\n=== Performance Comparison ===');
  SwitchingComparison.compareLatency();
  console.log();
  SwitchingComparison.compareBandwidthUtilization();
  console.log();
  SwitchingComparison.compareUseCases();
}

// demonstrateSwitching();
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **Circuit Switching**: 전화처럼 **전용 회선**, 일정한 품질
> 2. **Packet Switching**: 인터넷의 기반, **효율적** 대역폭 활용
> 3. **Message Switching**: Store-and-Forward, 현재는 거의 사용 안 함
> 4. **Multiplexing**: FDM(주파수), TDM(시간), WDM(파장) 분할
> 5. 각 방식은 **용도에 따라** 선택 (실시간 vs 효율성)

## 참고 자료

### 📖 추가 학습
- [Circuit Switching vs Packet Switching](http://www.rfwireless-world.com/Terminology/circuit-switching-vs-packet-switching.html)
- [Packet Switching vs Message Switching](http://www.rfwireless-world.com/Terminology/packet-switching-vs-message-switching.html)
- [Multiplexing - Wikipedia](https://en.wikipedia.org/wiki/Multiplexing)

### 🔗 관련 문서
- [[05_tcp-udp|TCP와 UDP 프로토콜]]
- [[03_tcp-ip-model|TCP/IP 4계층 모델]]
- [[08_network-components|네트워크 구성요소]]
