---
title: 실시간 통신
created: 2025-01-03
updated: 2025-01-03
tags:
  - CS
  - Network
  - WebRTC
  - Real-time
  - P2P
  - Streaming
category: Network
status: 완료
---

# 📡 실시간 통신 (Real-time Communication)

## 📌 개요

실시간 통신은 지연 시간을 최소화하여 즉각적인 데이터 교환을 가능하게 하는 기술입니다. WebRTC, WebSocket, Server-Sent Events 등 다양한 기술을 통해 실시간 애플리케이션을 구현할 수 있습니다.

> [!info] 핵심 개념
> - **WebRTC**: P2P 실시간 미디어 통신
> - **WebSocket**: 양방향 실시간 통신
> - **SSE**: 서버 → 클라이언트 단방향 스트리밍
> - **Long Polling**: HTTP 기반 실시간 통신

## 📚 목차

1. [실시간 통신 개요](#실시간-통신-개요)
2. [WebRTC](#webrtc)
3. [WebSocket](#websocket)
4. [Server-Sent Events (SSE)](#server-sent-events-sse)
5. [Long Polling](#long-polling)
6. [실시간 통신 비교](#실시간-통신-비교)
7. [NAT Traversal](#nat-traversal)
8. [실제 구현 예시](#실제-구현-예시)
9. [참고 자료](#참고-자료)

## 실시간 통신 개요

### 실시간 통신이란?
- 최소 지연으로 데이터 전송
- 양방향 또는 단방향 통신
- 지속적인 연결 유지
- 이벤트 기반 아키텍처

### 사용 사례
- **화상 회의**: Zoom, Google Meet
- **실시간 채팅**: Slack, Discord
- **라이브 스트리밍**: YouTube Live, Twitch
- **온라인 게임**: 멀티플레이어 게임
- **협업 도구**: Google Docs, Figma

## WebRTC

### WebRTC란?
- Web Real-Time Communication
- P2P 미디어 스트리밍
- 플러그인 없이 브라우저에서 동작
- 오디오, 비디오, 데이터 채널 지원

### WebRTC 아키텍처
```
Peer A                          Peer B
  │                               │
  ├──── 1. Offer (SDP) ──────────>│
  │                               │
  │<──── 2. Answer (SDP) ─────────┤
  │                               │
  ├──── 3. ICE Candidates ───────>│
  │<──── 4. ICE Candidates ────────┤
  │                               │
  │═════ 5. P2P Connection ═══════│
```

### WebRTC 구성 요소
1. **MediaStream**: 오디오/비디오 스트림
2. **RTCPeerConnection**: P2P 연결
3. **RTCDataChannel**: 데이터 전송
4. **Signaling Server**: 연결 설정 중개

### SDP (Session Description Protocol)
```
v=0
o=- 123456 2 IN IP4 127.0.0.1
s=-
t=0 0
m=audio 9 UDP/TLS/RTP/SAVPF 111
a=rtpmap:111 opus/48000/2
m=video 9 UDP/TLS/RTP/SAVPF 96
a=rtpmap:96 VP8/90000
```

## WebSocket

### WebSocket이란?
- 전이중 통신 프로토콜
- HTTP 업그레이드로 시작
- 낮은 오버헤드
- 실시간 양방향 통신

### WebSocket 핸드셰이크
```http
# 클라이언트 요청
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

# 서버 응답
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### WebSocket 프레임 구조
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
|N|V|V|V|       |S|             |   (if payload len==126/127)   |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
|     Extended payload length continued, if payload len == 127 |
+ - - - - - - - - - - - - - - - +-------------------------------+
|                               |Masking-key, if MASK set to 1  |
+-------------------------------+-------------------------------+
| Masking-key (continued)       |          Payload Data         |
+-------------------------------- - - - - - - - - - - - - - - - +
:                     Payload Data continued ...                :
+ - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
|                     Payload Data continued ...                |
+---------------------------------------------------------------+
```

## Server-Sent Events (SSE)

### SSE란?
- 서버에서 클라이언트로 단방향 스트리밍
- HTTP 기반
- 자동 재연결
- 텍스트 기반 이벤트

### SSE 형식
```http
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache

data: First message

data: Second message
id: 123

event: update
data: {"user": "alice", "message": "hello"}

: heartbeat

retry: 10000
```

## Long Polling

### Long Polling이란?
- HTTP 요청을 열어둔 채로 대기
- 이벤트 발생 시 응답 전송
- 응답 후 즉시 재연결
- WebSocket 대안

### Long Polling vs Regular Polling
```
Regular Polling:
Client: Request? → Server: No data (immediate)
Client: Request? → Server: No data (immediate)
Client: Request? → Server: Data! (immediate)

Long Polling:
Client: Request? → Server: (wait...)
                          (wait...)
                          Data! (when available)
Client: Request? → Server: (wait...)
```

## 실시간 통신 비교

| 기술 | 방향 | 프로토콜 | 장점 | 단점 | 사용 사례 |
|------|------|----------|------|------|-----------|
| **WebRTC** | P2P | UDP/SRTP | 낮은 지연, P2P | 복잡한 설정 | 화상회의 |
| **WebSocket** | 양방향 | WS/WSS | 양방향, 효율적 | 연결 유지 | 채팅, 게임 |
| **SSE** | 단방향 | HTTP | 간단, 자동 재연결 | 단방향만 | 알림, 피드 |
| **Long Polling** | 양방향 | HTTP | 방화벽 친화적 | 리소스 낭비 | 폴백 옵션 |

## NAT Traversal

### NAT 문제
```
Private Network A          Internet          Private Network B
192.168.1.100         ┌─────────────┐         192.168.2.100
      │              │               │              │
      ↓              │               │              ↓
   NAT A             │               │           NAT B
  1.2.3.4:5000       │               │        5.6.7.8:6000
      ║              │               │              ║
      ╚══════════════╧═══════════════╧══════════════╝
                    Direct P2P? ❌
```

### STUN (Session Traversal Utilities for NAT)
- 공인 IP 주소 발견
- NAT 타입 확인
- 포트 매핑 정보 제공

### TURN (Traversal Using Relays around NAT)
- 릴레이 서버 경유
- Symmetric NAT 해결
- 대역폭 비용 발생

### ICE (Interactive Connectivity Establishment)
```
1. 로컬 후보 수집
2. STUN으로 공인 IP 획득
3. TURN 서버 할당 (필요시)
4. 후보 교환
5. 연결 테스트
6. 최적 경로 선택
```

## 실제 구현 예시

### TypeScript로 실시간 통신 구현

```typescript
import { EventEmitter } from 'events';
import * as crypto from 'crypto';

// ===== WebRTC 구현 =====
class SimpleWebRTC extends EventEmitter {
  private localConnection: RTCPeerConnectionMock | null = null;
  private remoteConnection: RTCPeerConnectionMock | null = null;
  private dataChannel: RTCDataChannelMock | null = null;
  private signaling: SignalingServer;
  
  constructor(signaling: SignalingServer) {
    super();
    this.signaling = signaling;
    this.setupSignaling();
  }
  
  // 시그널링 설정
  private setupSignaling(): void {
    this.signaling.on('offer', (offer) => this.handleOffer(offer));
    this.signaling.on('answer', (answer) => this.handleAnswer(answer));
    this.signaling.on('ice-candidate', (candidate) => this.handleIceCandidate(candidate));
  }
  
  // 연결 시작 (Caller)
  async initiateCall(): Promise<void> {
    console.log('Initiating WebRTC call...');
    
    // Peer Connection 생성
    this.localConnection = new RTCPeerConnectionMock({
      iceServers: [
        { urls: 'stun:stun.l.google.com:19302' },
        { urls: 'turn:turn.example.com', username: 'user', credential: 'pass' }
      ]
    });
    
    // Data Channel 생성
    this.dataChannel = this.localConnection.createDataChannel('chat');
    this.setupDataChannel(this.dataChannel);
    
    // ICE 후보 수집
    this.localConnection.onicecandidate = (event) => {
      if (event.candidate) {
        this.signaling.send('ice-candidate', event.candidate);
      }
    };
    
    // Offer 생성
    const offer = await this.localConnection.createOffer();
    await this.localConnection.setLocalDescription(offer);
    
    // Signaling 서버로 Offer 전송
    this.signaling.send('offer', offer);
  }
  
  // Offer 처리 (Callee)
  private async handleOffer(offer: any): Promise<void> {
    console.log('Received offer, creating answer...');
    
    // Peer Connection 생성
    this.remoteConnection = new RTCPeerConnectionMock({
      iceServers: [
        { urls: 'stun:stun.l.google.com:19302' }
      ]
    });
    
    // Data Channel 이벤트
    this.remoteConnection.ondatachannel = (event) => {
      const channel = event.channel;
      this.setupDataChannel(channel);
    };
    
    // ICE 후보 수집
    this.remoteConnection.onicecandidate = (event) => {
      if (event.candidate) {
        this.signaling.send('ice-candidate', event.candidate);
      }
    };
    
    // Offer 설정
    await this.remoteConnection.setRemoteDescription(offer);
    
    // Answer 생성
    const answer = await this.remoteConnection.createAnswer();
    await this.remoteConnection.setLocalDescription(answer);
    
    // Answer 전송
    this.signaling.send('answer', answer);
  }
  
  // Answer 처리
  private async handleAnswer(answer: any): Promise<void> {
    console.log('Received answer');
    if (this.localConnection) {
      await this.localConnection.setRemoteDescription(answer);
    }
  }
  
  // ICE 후보 처리
  private async handleIceCandidate(candidate: any): Promise<void> {
    console.log('Received ICE candidate');
    const connection = this.localConnection || this.remoteConnection;
    if (connection) {
      await connection.addIceCandidate(candidate);
    }
  }
  
  // Data Channel 설정
  private setupDataChannel(channel: RTCDataChannelMock): void {
    channel.onopen = () => {
      console.log('Data channel opened');
      this.emit('connected');
    };
    
    channel.onmessage = (event) => {
      console.log('Received:', event.data);
      this.emit('message', event.data);
    };
    
    channel.onclose = () => {
      console.log('Data channel closed');
      this.emit('disconnected');
    };
    
    this.dataChannel = channel;
  }
  
  // 메시지 전송
  sendMessage(message: string): void {
    if (this.dataChannel && this.dataChannel.readyState === 'open') {
      this.dataChannel.send(message);
    }
  }
  
  // 연결 종료
  close(): void {
    if (this.dataChannel) {
      this.dataChannel.close();
    }
    if (this.localConnection) {
      this.localConnection.close();
    }
    if (this.remoteConnection) {
      this.remoteConnection.close();
    }
  }
}

// WebRTC Mock 클래스들 (실제로는 브라우저 API)
class RTCPeerConnectionMock {
  onicecandidate: ((event: any) => void) | null = null;
  ondatachannel: ((event: any) => void) | null = null;
  
  constructor(config: any) {}
  
  createDataChannel(label: string): RTCDataChannelMock {
    return new RTCDataChannelMock(label);
  }
  
  async createOffer(): Promise<any> {
    return { type: 'offer', sdp: 'mock-sdp-offer' };
  }
  
  async createAnswer(): Promise<any> {
    return { type: 'answer', sdp: 'mock-sdp-answer' };
  }
  
  async setLocalDescription(desc: any): Promise<void> {}
  async setRemoteDescription(desc: any): Promise<void> {}
  async addIceCandidate(candidate: any): Promise<void> {}
  close(): void {}
}

class RTCDataChannelMock {
  label: string;
  readyState: string = 'connecting';
  onopen: (() => void) | null = null;
  onmessage: ((event: any) => void) | null = null;
  onclose: (() => void) | null = null;
  
  constructor(label: string) {
    this.label = label;
    setTimeout(() => {
      this.readyState = 'open';
      if (this.onopen) this.onopen();
    }, 100);
  }
  
  send(data: string): void {
    console.log(`DataChannel sending: ${data}`);
  }
  
  close(): void {
    this.readyState = 'closed';
    if (this.onclose) this.onclose();
  }
}

// ===== WebSocket 서버 구현 =====
class WebSocketServer extends EventEmitter {
  private clients: Map<string, WebSocketClient>;
  private rooms: Map<string, Set<string>>;
  
  constructor() {
    super();
    this.clients = new Map();
    this.rooms = new Map();
  }
  
  // 클라이언트 연결
  handleConnection(socket: WebSocketClient): void {
    const clientId = this.generateClientId();
    socket.id = clientId;
    
    this.clients.set(clientId, socket);
    console.log(`WebSocket client connected: ${clientId}`);
    
    // 이벤트 핸들러
    socket.on('message', (data) => this.handleMessage(clientId, data));
    socket.on('close', () => this.handleDisconnection(clientId));
    socket.on('join', (room) => this.joinRoom(clientId, room));
    socket.on('leave', (room) => this.leaveRoom(clientId, room));
    
    // 연결 확인
    socket.send(JSON.stringify({
      type: 'connected',
      clientId: clientId
    }));
  }
  
  // 메시지 처리
  private handleMessage(clientId: string, data: string): void {
    try {
      const message = JSON.parse(data);
      
      switch (message.type) {
        case 'broadcast':
          this.broadcast(message.data, clientId);
          break;
        
        case 'room':
          this.roomBroadcast(message.room, message.data, clientId);
          break;
        
        case 'private':
          this.sendTo(message.to, message.data);
          break;
        
        default:
          this.emit('message', { clientId, message });
      }
    } catch (error) {
      console.error('Invalid message:', error);
    }
  }
  
  // 연결 해제
  private handleDisconnection(clientId: string): void {
    const client = this.clients.get(clientId);
    if (client) {
      // 모든 룸에서 제거
      this.rooms.forEach((members, room) => {
        if (members.has(clientId)) {
          members.delete(clientId);
          this.roomBroadcast(room, {
            type: 'user-left',
            userId: clientId
          }, clientId);
        }
      });
      
      this.clients.delete(clientId);
      console.log(`WebSocket client disconnected: ${clientId}`);
    }
  }
  
  // 룸 참가
  private joinRoom(clientId: string, room: string): void {
    if (!this.rooms.has(room)) {
      this.rooms.set(room, new Set());
    }
    
    this.rooms.get(room)!.add(clientId);
    
    // 룸 멤버들에게 알림
    this.roomBroadcast(room, {
      type: 'user-joined',
      userId: clientId
    }, clientId);
    
    console.log(`Client ${clientId} joined room ${room}`);
  }
  
  // 룸 나가기
  private leaveRoom(clientId: string, room: string): void {
    const members = this.rooms.get(room);
    if (members) {
      members.delete(clientId);
      
      // 룸 멤버들에게 알림
      this.roomBroadcast(room, {
        type: 'user-left',
        userId: clientId
      }, clientId);
      
      // 빈 룸 제거
      if (members.size === 0) {
        this.rooms.delete(room);
      }
    }
  }
  
  // 브로드캐스트
  private broadcast(data: any, excludeId?: string): void {
    const message = JSON.stringify({
      type: 'broadcast',
      data: data
    });
    
    this.clients.forEach((client, id) => {
      if (id !== excludeId && client.readyState === 'open') {
        client.send(message);
      }
    });
  }
  
  // 룸 브로드캐스트
  private roomBroadcast(room: string, data: any, excludeId?: string): void {
    const members = this.rooms.get(room);
    if (!members) return;
    
    const message = JSON.stringify({
      type: 'room-message',
      room: room,
      data: data
    });
    
    members.forEach(memberId => {
      if (memberId !== excludeId) {
        const client = this.clients.get(memberId);
        if (client && client.readyState === 'open') {
          client.send(message);
        }
      }
    });
  }
  
  // 특정 클라이언트에게 전송
  private sendTo(clientId: string, data: any): void {
    const client = this.clients.get(clientId);
    if (client && client.readyState === 'open') {
      client.send(JSON.stringify({
        type: 'private',
        data: data
      }));
    }
  }
  
  // 클라이언트 ID 생성
  private generateClientId(): string {
    return crypto.randomBytes(16).toString('hex');
  }
  
  // 서버 상태
  getStats(): object {
    return {
      clients: this.clients.size,
      rooms: this.rooms.size,
      roomDetails: Array.from(this.rooms.entries()).map(([room, members]) => ({
        room,
        members: members.size
      }))
    };
  }
}

// WebSocket 클라이언트 Mock
class WebSocketClient extends EventEmitter {
  id: string = '';
  readyState: string = 'open';
  
  send(data: string): void {
    console.log(`Sending to client ${this.id}:`, data);
  }
}

// ===== SSE 서버 구현 =====
class SSEServer {
  private connections: Map<string, SSEConnection>;
  
  constructor() {
    this.connections = new Map();
  }
  
  // SSE 연결 처리
  handleConnection(req: any, res: any): void {
    const connectionId = this.generateConnectionId();
    
    // SSE 헤더 설정
    res.writeHead(200, {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
      'Access-Control-Allow-Origin': '*'
    });
    
    // 연결 저장
    const connection = new SSEConnection(connectionId, res);
    this.connections.set(connectionId, connection);
    
    // 초기 메시지
    connection.send({
      type: 'connected',
      connectionId: connectionId
    });
    
    // Heartbeat
    const heartbeat = setInterval(() => {
      connection.sendHeartbeat();
    }, 30000);
    
    // 연결 종료 처리
    req.on('close', () => {
      clearInterval(heartbeat);
      this.connections.delete(connectionId);
      console.log(`SSE connection closed: ${connectionId}`);
    });
    
    console.log(`SSE connection established: ${connectionId}`);
  }
  
  // 모든 클라이언트에게 이벤트 전송
  broadcast(event: string, data: any): void {
    this.connections.forEach(connection => {
      connection.sendEvent(event, data);
    });
  }
  
  // 특정 클라이언트에게 이벤트 전송
  sendTo(connectionId: string, event: string, data: any): void {
    const connection = this.connections.get(connectionId);
    if (connection) {
      connection.sendEvent(event, data);
    }
  }
  
  // 연결 ID 생성
  private generateConnectionId(): string {
    return crypto.randomBytes(8).toString('hex');
  }
}

// SSE 연결 클래스
class SSEConnection {
  private id: string;
  private response: any;
  private eventId: number = 0;
  
  constructor(id: string, response: any) {
    this.id = id;
    this.response = response;
  }
  
  // 데이터 전송
  send(data: any): void {
    const message = `data: ${JSON.stringify(data)}\n\n`;
    this.response.write(message);
  }
  
  // 이벤트 전송
  sendEvent(event: string, data: any): void {
    const message = [
      `id: ${++this.eventId}`,
      `event: ${event}`,
      `data: ${JSON.stringify(data)}`,
      '',
      ''
    ].join('\n');
    
    this.response.write(message);
  }
  
  // Heartbeat
  sendHeartbeat(): void {
    this.response.write(': heartbeat\n\n');
  }
}

// ===== Long Polling 구현 =====
class LongPollingServer {
  private pendingRequests: Map<string, PendingRequest>;
  private messageQueue: Map<string, any[]>;
  
  constructor() {
    this.pendingRequests = new Map();
    this.messageQueue = new Map();
  }
  
  // Long Polling 요청 처리
  handleRequest(sessionId: string, req: any, res: any): void {
    // 기존 요청이 있으면 종료
    const existing = this.pendingRequests.get(sessionId);
    if (existing) {
      existing.resolve([]);
    }
    
    // 메시지 큐 확인
    const messages = this.messageQueue.get(sessionId) || [];
    
    if (messages.length > 0) {
      // 즉시 응답
      res.json(messages);
      this.messageQueue.set(sessionId, []);
    } else {
      // 대기 상태로 전환
      const pending = new PendingRequest(req, res);
      this.pendingRequests.set(sessionId, pending);
      
      // 타임아웃 설정 (30초)
      setTimeout(() => {
        if (this.pendingRequests.has(sessionId)) {
          pending.resolve([]);
          this.pendingRequests.delete(sessionId);
        }
      }, 30000);
    }
  }
  
  // 메시지 전송
  sendMessage(sessionId: string, message: any): void {
    const pending = this.pendingRequests.get(sessionId);
    
    if (pending) {
      // 대기 중인 요청에 즉시 응답
      pending.resolve([message]);
      this.pendingRequests.delete(sessionId);
    } else {
      // 메시지 큐에 추가
      const queue = this.messageQueue.get(sessionId) || [];
      queue.push(message);
      this.messageQueue.set(sessionId, queue);
    }
  }
}

// 대기 중인 요청
class PendingRequest {
  private req: any;
  private res: any;
  
  constructor(req: any, res: any) {
    this.req = req;
    this.res = res;
  }
  
  resolve(data: any): void {
    this.res.json(data);
  }
}

// ===== Signaling 서버 =====
class SignalingServer extends EventEmitter {
  private peers: Map<string, any>;
  
  constructor() {
    super();
    this.peers = new Map();
  }
  
  // 메시지 전송
  send(type: string, data: any): void {
    this.emit('signal', { type, data });
  }
  
  // 메시지 수신
  receive(message: any): void {
    this.emit(message.type, message.data);
  }
}

// ===== 채팅 애플리케이션 예제 =====
class RealtimeChatApp {
  private wsServer: WebSocketServer;
  private sseServer: SSEServer;
  
  constructor() {
    this.wsServer = new WebSocketServer();
    this.sseServer = new SSEServer();
  }
  
  // WebSocket 채팅
  demonstrateWebSocketChat(): void {
    console.log('\n=== WebSocket Chat Demo ===\n');
    
    // 클라이언트 시뮬레이션
    const alice = new WebSocketClient();
    const bob = new WebSocketClient();
    
    // 연결
    this.wsServer.handleConnection(alice);
    this.wsServer.handleConnection(bob);
    
    // 룸 참가
    alice.emit('join', 'room1');
    bob.emit('join', 'room1');
    
    // 메시지 전송
    alice.emit('message', JSON.stringify({
      type: 'room',
      room: 'room1',
      data: { text: 'Hello, Bob!' }
    }));
    
    // 상태 출력
    console.log('\nServer Stats:', this.wsServer.getStats());
  }
  
  // SSE 알림
  demonstrateSSENotifications(): void {
    console.log('\n=== SSE Notifications Demo ===\n');
    
    // 가상 응답 객체
    const mockRes = {
      writeHead: () => {},
      write: (data: string) => console.log('SSE:', data)
    };
    
    const mockReq = new EventEmitter();
    
    // SSE 연결
    this.sseServer.handleConnection(mockReq, mockRes);
    
    // 알림 전송
    setTimeout(() => {
      this.sseServer.broadcast('notification', {
        title: 'New Message',
        body: 'You have a new message!'
      });
    }, 1000);
    
    setTimeout(() => {
      this.sseServer.broadcast('update', {
        type: 'user-status',
        userId: 'alice',
        status: 'online'
      });
    }, 2000);
  }
}

// 사용 예제
async function demonstrateRealtimeCommunication() {
  console.log('=== Real-time Communication Demonstration ===');
  
  // WebRTC 데모
  console.log('\n=== WebRTC Demo ===');
  const signaling = new SignalingServer();
  const peerA = new SimpleWebRTC(signaling);
  const peerB = new SimpleWebRTC(signaling);
  
  // 시그널링 릴레이 (실제로는 서버가 중개)
  signaling.on('signal', (message) => {
    signaling.receive(message);
  });
  
  // 연결 시작
  await peerA.initiateCall();
  
  // 채팅 앱 데모
  const chatApp = new RealtimeChatApp();
  chatApp.demonstrateWebSocketChat();
  chatApp.demonstrateSSENotifications();
}

// 실행
// demonstrateRealtimeCommunication().catch(console.error);
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. **WebRTC**: P2P 실시간 미디어 통신, NAT 트래버설 필요
> 2. **WebSocket**: 양방향 실시간 통신, 낮은 오버헤드
> 3. **SSE**: 서버 → 클라이언트 단방향, 자동 재연결
> 4. **Long Polling**: HTTP 기반, 방화벽 친화적
> 5. **선택 기준**: 지연시간, 방향성, 브라우저 지원, 복잡도

## 참고 자료

### 📖 추가 학습
- [WebRTC - MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API)
- [WebSocket - RFC 6455](https://tools.ietf.org/html/rfc6455)
- [Server-Sent Events - W3C](https://www.w3.org/TR/eventsource/)
- [WebRTC Samples](https://webrtc.github.io/samples/)

### 🔗 관련 문서
- [[19_websocket|WebSocket]]
- [[14_http2-optimization|HTTP/2와 최적화]]
- [[16_cdn-load-balancing|CDN과 로드 밸런싱]]