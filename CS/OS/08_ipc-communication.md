---
tags:
  - os
  - ipc
  - communication
  - message-passing
created: 2025-01-08
updated: 2025-01-08
---

# IPC (Inter-Process Communication)

> [!info] IPC란?
> IPC(Inter-Process Communication)는 프로세스 간 데이터를 주고받는 메커니즘입니다. 독립적인 프로세스들이 협력하여 작업을 수행하거나 데이터를 공유하기 위해 필요합니다.

## 📑 목차

- [[#🎯 IPC 기본 개념]]
- [[#📝 메시지 패싱]]
- [[#🔄 공유 메모리]]
- [[#📞 파이프와 FIFO]]
- [[#🌐 소켓 통신]]
- [[#💡 TypeScript 구현]]
- [[#🔍 실제 사례]]

---

## 🎯 IPC 기본 개념

### IPC 메커니즘 분류

```typescript
enum IPCType {
  MESSAGE_PASSING = 'MESSAGE_PASSING',
  SHARED_MEMORY = 'SHARED_MEMORY',
  PIPE = 'PIPE',
  FIFO = 'FIFO',
  SOCKET = 'SOCKET',
  SIGNAL = 'SIGNAL',
  SEMAPHORE = 'SEMAPHORE',
  MESSAGE_QUEUE = 'MESSAGE_QUEUE'
}

interface IPCMechanism {
  type: IPCType;
  synchronous: boolean;
  capacity: number;
  overhead: 'low' | 'medium' | 'high';
  flexibility: 'low' | 'medium' | 'high';
  networkCapable: boolean;
}

class IPCComparison {
  private mechanisms: Map<IPCType, IPCMechanism> = new Map();

  constructor() {
    this.initializeMechanisms();
  }

  private initializeMechanisms(): void {
    this.mechanisms.set(IPCType.MESSAGE_PASSING, {
      type: IPCType.MESSAGE_PASSING,
      synchronous: true,
      capacity: 1000,
      overhead: 'high',
      flexibility: 'high',
      networkCapable: true
    });

    this.mechanisms.set(IPCType.SHARED_MEMORY, {
      type: IPCType.SHARED_MEMORY,
      synchronous: false,
      capacity: 1000000,
      overhead: 'low',
      flexibility: 'medium',
      networkCapable: false
    });

    this.mechanisms.set(IPCType.PIPE, {
      type: IPCType.PIPE,
      synchronous: true,
      capacity: 65536,
      overhead: 'medium',
      flexibility: 'low',
      networkCapable: false
    });

    this.mechanisms.set(IPCType.SOCKET, {
      type: IPCType.SOCKET,
      synchronous: false,
      capacity: 65536,
      overhead: 'high',
      flexibility: 'high',
      networkCapable: true
    });
  }

  comparePerformance(): void {
    console.log('\n=== IPC Performance Comparison ===');
    console.log('Mechanism\t\tCapacity\tOverhead\tNetwork');
    console.log('━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
    
    for (const [type, mechanism] of this.mechanisms) {
      console.log(`${type.padEnd(20)}\t${mechanism.capacity}\t\t${mechanism.overhead}\t\t${mechanism.networkCapable ? 'Yes' : 'No'}`);
    }
  }
}
```

---

## 📝 메시지 패싱

### Message Passing 구현

```typescript
interface Message {
  id: string;
  sender: string;
  receiver: string;
  data: any;
  timestamp: number;
  priority: number;
}

class MessageQueue {
  private messages: Message[] = [];
  private capacity: number;
  private waitingReceivers: Map<string, (message: Message) => void> = new Map();

  constructor(capacity: number = 100) {
    this.capacity = capacity;
  }

  // 메시지 전송
  async send(message: Omit<Message, 'id' | 'timestamp'>): Promise<boolean> {
    if (this.messages.length >= this.capacity) {
      console.log(`Queue full, message from ${message.sender} dropped`);
      return false;
    }

    const fullMessage: Message = {
      ...message,
      id: this.generateMessageId(),
      timestamp: Date.now()
    };

    // 우선순위에 따라 삽입
    this.insertByPriority(fullMessage);
    console.log(`Message sent from ${message.sender} to ${message.receiver}`);

    // 대기 중인 수신자에게 즉시 전달
    const waitingReceiver = this.waitingReceivers.get(message.receiver);
    if (waitingReceiver) {
      const msg = this.messages.shift()!;
      waitingReceiver(msg);
      this.waitingReceivers.delete(message.receiver);
    }

    return true;
  }

  // 메시지 수신
  async receive(receiverId: string): Promise<Message> {
    // 해당 수신자를 위한 메시지 찾기
    const messageIndex = this.messages.findIndex(msg => msg.receiver === receiverId);
    
    if (messageIndex !== -1) {
      const message = this.messages.splice(messageIndex, 1)[0];
      console.log(`Message received by ${receiverId} from ${message.sender}`);
      return message;
    }

    // 메시지가 없으면 대기
    return new Promise<Message>((resolve) => {
      this.waitingReceivers.set(receiverId, resolve);
      console.log(`${receiverId} waiting for message...`);
    });
  }

  private insertByPriority(message: Message): void {
    let inserted = false;
    for (let i = 0; i < this.messages.length; i++) {
      if (message.priority > this.messages[i].priority) {
        this.messages.splice(i, 0, message);
        inserted = true;
        break;
      }
    }
    if (!inserted) {
      this.messages.push(message);
    }
  }

  private generateMessageId(): string {
    return Math.random().toString(36).substr(2, 9);
  }

  getQueueStatus(): any {
    return {
      messageCount: this.messages.length,
      capacity: this.capacity,
      waitingReceivers: this.waitingReceivers.size,
      messages: this.messages.map(m => ({
        id: m.id,
        sender: m.sender,
        receiver: m.receiver,
        priority: m.priority
      }))
    };
  }
}

// Producer-Consumer with Message Passing
class MessagePassingExample {
  private messageQueue: MessageQueue;

  constructor() {
    this.messageQueue = new MessageQueue(10);
  }

  async runProducerConsumer(): Promise<void> {
    console.log('\n=== Producer-Consumer with Message Passing ===');

    // Producer
    const producer = async (id: string) => {
      for (let i = 0; i < 5; i++) {
        await this.messageQueue.send({
          sender: `Producer-${id}`,
          receiver: 'Consumer-1',
          data: `Data-${i}`,
          priority: Math.floor(Math.random() * 10)
        });
        await new Promise(resolve => setTimeout(resolve, 100));
      }
    };

    // Consumer
    const consumer = async (id: string) => {
      for (let i = 0; i < 5; i++) {
        const message = await this.messageQueue.receive(`Consumer-${id}`);
        console.log(`${id} processed: ${message.data}`);
        await new Promise(resolve => setTimeout(resolve, 150));
      }
    };

    // 동시 실행
    await Promise.all([
      producer('A'),
      producer('B'),
      consumer('1'),
      consumer('2')
    ]);
  }
}
```

---

## 🔄 공유 메모리

### Shared Memory 구현

```typescript
class SharedMemorySegment {
  private memory: ArrayBuffer;
  private view: DataView;
  private size: number;
  private readers: Set<string> = new Set();
  private writers: Set<string> = new Set();
  private lock: boolean = false;

  constructor(size: number) {
    this.size = size;
    this.memory = new ArrayBuffer(size);
    this.view = new DataView(this.memory);
  }

  // 공유 메모리 읽기
  read(offset: number, length: number, readerId: string): Uint8Array {
    this.readers.add(readerId);
    
    if (offset + length > this.size) {
      throw new Error('Read beyond memory bounds');
    }

    const buffer = new Uint8Array(this.memory, offset, length);
    console.log(`${readerId} read ${length} bytes from offset ${offset}`);
    return buffer;
  }

  // 공유 메모리 쓰기
  write(offset: number, data: Uint8Array, writerId: string): boolean {
    if (this.lock) {
      console.log(`${writerId} blocked - memory locked`);
      return false;
    }

    this.writers.add(writerId);

    if (offset + data.length > this.size) {
      throw new Error('Write beyond memory bounds');
    }

    const targetBuffer = new Uint8Array(this.memory, offset, data.length);
    targetBuffer.set(data);
    
    console.log(`${writerId} wrote ${data.length} bytes to offset ${offset}`);
    return true;
  }

  // 메모리 잠금
  lock(processId: string): boolean {
    if (this.lock) {
      return false;
    }
    this.lock = true;
    console.log(`Memory locked by ${processId}`);
    return true;
  }

  // 메모리 잠금 해제
  unlock(processId: string): void {
    this.lock = false;
    console.log(`Memory unlocked by ${processId}`);
  }

  // 통계 정보
  getStatistics(): any {
    return {
      size: this.size,
      readers: Array.from(this.readers),
      writers: Array.from(this.writers),
      locked: this.lock
    };
  }
}

// 공유 메모리 관리자
class SharedMemoryManager {
  private segments: Map<string, SharedMemorySegment> = new Map();

  // 공유 메모리 세그먼트 생성
  createSegment(key: string, size: number): SharedMemorySegment {
    if (this.segments.has(key)) {
      throw new Error(`Segment ${key} already exists`);
    }

    const segment = new SharedMemorySegment(size);
    this.segments.set(key, segment);
    console.log(`Created shared memory segment: ${key} (${size} bytes)`);
    return segment;
  }

  // 공유 메모리 세그먼트 연결
  attachSegment(key: string, processId: string): SharedMemorySegment | null {
    const segment = this.segments.get(key);
    if (segment) {
      console.log(`Process ${processId} attached to segment ${key}`);
      return segment;
    }
    return null;
  }

  // 공유 메모리 세그먼트 해제
  detachSegment(key: string, processId: string): void {
    console.log(`Process ${processId} detached from segment ${key}`);
  }

  // 세그먼트 삭제
  removeSegment(key: string): boolean {
    if (this.segments.delete(key)) {
      console.log(`Removed shared memory segment: ${key}`);
      return true;
    }
    return false;
  }
}

// 공유 메모리 예제
class SharedMemoryExample {
  private manager: SharedMemoryManager;

  constructor() {
    this.manager = new SharedMemoryManager();
  }

  async runExample(): Promise<void> {
    console.log('\n=== Shared Memory Communication ===');

    // 공유 메모리 세그먼트 생성
    const segment = this.manager.createSegment('data_buffer', 1024);

    // Writer 프로세스
    const writer = async () => {
      const data = new TextEncoder().encode('Hello from writer!');
      segment.write(0, data, 'Writer-1');
      
      // 추가 데이터
      const moreData = new TextEncoder().encode(' More data...');
      segment.write(20, moreData, 'Writer-1');
    };

    // Reader 프로세스
    const reader = async () => {
      await new Promise(resolve => setTimeout(resolve, 100));
      
      const data = segment.read(0, 50, 'Reader-1');
      const text = new TextDecoder().decode(data);
      console.log(`Reader received: ${text.trim()}`);
    };

    await Promise.all([writer(), reader()]);
    
    console.log('\nShared Memory Statistics:');
    console.log(segment.getStatistics());
  }
}
```

---

## 📞 파이프와 FIFO

### Pipe 구현

```typescript
// Anonymous Pipe
class Pipe {
  private buffer: Uint8Array;
  private readPos: number = 0;
  private writePos: number = 0;
  private capacity: number;
  private closed: boolean = false;

  constructor(capacity: number = 4096) {
    this.capacity = capacity;
    this.buffer = new Uint8Array(capacity);
  }

  // 파이프에 쓰기
  write(data: Uint8Array): number {
    if (this.closed) {
      throw new Error('Pipe is closed');
    }

    const availableSpace = this.capacity - (this.writePos - this.readPos);
    const bytesToWrite = Math.min(data.length, availableSpace);

    for (let i = 0; i < bytesToWrite; i++) {
      this.buffer[this.writePos % this.capacity] = data[i];
      this.writePos++;
    }

    console.log(`Wrote ${bytesToWrite} bytes to pipe`);
    return bytesToWrite;
  }

  // 파이프에서 읽기
  read(size: number): Uint8Array {
    if (this.readPos >= this.writePos && this.closed) {
      return new Uint8Array(0); // EOF
    }

    const availableData = this.writePos - this.readPos;
    const bytesToRead = Math.min(size, availableData);
    const result = new Uint8Array(bytesToRead);

    for (let i = 0; i < bytesToRead; i++) {
      result[i] = this.buffer[this.readPos % this.capacity];
      this.readPos++;
    }

    console.log(`Read ${bytesToRead} bytes from pipe`);
    return result;
  }

  // 파이프 닫기
  close(): void {
    this.closed = true;
    console.log('Pipe closed');
  }

  // 파이프 상태
  getStatus(): any {
    return {
      capacity: this.capacity,
      used: this.writePos - this.readPos,
      readPos: this.readPos,
      writePos: this.writePos,
      closed: this.closed
    };
  }
}

// Named Pipe (FIFO)
class NamedPipe {
  private pipes: Map<string, Pipe> = new Map();
  private permissions: Map<string, { read: Set<string>, write: Set<string> }> = new Map();

  // FIFO 생성
  createFIFO(name: string, capacity: number = 4096): boolean {
    if (this.pipes.has(name)) {
      return false; // 이미 존재
    }

    this.pipes.set(name, new Pipe(capacity));
    this.permissions.set(name, { read: new Set(), write: new Set() });
    console.log(`Created FIFO: ${name}`);
    return true;
  }

  // FIFO 열기
  openFIFO(name: string, mode: 'read' | 'write', processId: string): boolean {
    const perms = this.permissions.get(name);
    if (!perms) {
      return false; // FIFO 없음
    }

    if (mode === 'read') {
      perms.read.add(processId);
    } else {
      perms.write.add(processId);
    }

    console.log(`Process ${processId} opened FIFO ${name} for ${mode}`);
    return true;
  }

  // FIFO에 쓰기
  writeFIFO(name: string, data: Uint8Array, processId: string): number {
    const pipe = this.pipes.get(name);
    const perms = this.permissions.get(name);
    
    if (!pipe || !perms?.write.has(processId)) {
      throw new Error('Permission denied or FIFO not found');
    }

    return pipe.write(data);
  }

  // FIFO에서 읽기
  readFIFO(name: string, size: number, processId: string): Uint8Array {
    const pipe = this.pipes.get(name);
    const perms = this.permissions.get(name);
    
    if (!pipe || !perms?.read.has(processId)) {
      throw new Error('Permission denied or FIFO not found');
    }

    return pipe.read(size);
  }

  // FIFO 삭제
  removeFIFO(name: string): boolean {
    const removed = this.pipes.delete(name) && this.permissions.delete(name);
    if (removed) {
      console.log(`Removed FIFO: ${name}`);
    }
    return removed;
  }
}

// Pipe 사용 예제
class PipeExample {
  async runPipeExample(): Promise<void> {
    console.log('\n=== Pipe Communication Example ===');

    const pipe = new Pipe(1024);
    
    // Parent process (writer)
    const parent = async () => {
      const data = new TextEncoder().encode('Hello from parent process!');
      pipe.write(data);
      
      // 파이프 닫기
      setTimeout(() => pipe.close(), 200);
    };

    // Child process (reader)  
    const child = async () => {
      await new Promise(resolve => setTimeout(resolve, 50));
      
      let totalRead = 0;
      while (true) {
        const data = pipe.read(100);
        if (data.length === 0) break; // EOF
        
        const text = new TextDecoder().decode(data);
        console.log(`Child read: ${text}`);
        totalRead += data.length;
      }
      
      console.log(`Child total read: ${totalRead} bytes`);
    };

    await Promise.all([parent(), child()]);
  }

  async runFIFOExample(): Promise<void> {
    console.log('\n=== Named Pipe (FIFO) Example ===');

    const fifoManager = new NamedPipe();
    
    // FIFO 생성
    fifoManager.createFIFO('my_fifo', 512);
    
    // Writer 프로세스
    const writer = async () => {
      fifoManager.openFIFO('my_fifo', 'write', 'writer-1');
      
      const messages = ['Message 1', 'Message 2', 'Message 3'];
      for (const msg of messages) {
        const data = new TextEncoder().encode(msg);
        fifoManager.writeFIFO('my_fifo', data, 'writer-1');
        await new Promise(resolve => setTimeout(resolve, 100));
      }
    };

    // Reader 프로세스
    const reader = async () => {
      fifoManager.openFIFO('my_fifo', 'read', 'reader-1');
      
      await new Promise(resolve => setTimeout(resolve, 50));
      
      for (let i = 0; i < 3; i++) {
        const data = fifoManager.readFIFO('my_fifo', 100, 'reader-1');
        const text = new TextDecoder().decode(data);
        console.log(`Reader received: ${text}`);
        await new Promise(resolve => setTimeout(resolve, 120));
      }
    };

    await Promise.all([writer(), reader()]);
    
    // FIFO 정리
    fifoManager.removeFIFO('my_fifo');
  }
}
```

---

## 🌐 소켓 통신

### Socket 구현

```typescript
interface SocketAddress {
  family: 'AF_INET' | 'AF_UNIX';
  address: string;
  port?: number;
}

class Socket {
  private id: string;
  private type: 'TCP' | 'UDP' | 'UNIX';
  private state: 'CLOSED' | 'LISTENING' | 'CONNECTED' | 'CONNECTING';
  private localAddress: SocketAddress | null = null;
  private remoteAddress: SocketAddress | null = null;
  private buffer: Uint8Array[] = [];

  constructor(type: 'TCP' | 'UDP' | 'UNIX') {
    this.id = Math.random().toString(36).substr(2, 9);
    this.type = type;
    this.state = 'CLOSED';
  }

  // 소켓 바인딩
  bind(address: SocketAddress): boolean {
    if (this.state !== 'CLOSED') {
      return false;
    }

    this.localAddress = address;
    console.log(`Socket ${this.id} bound to ${address.address}:${address.port || 'N/A'}`);
    return true;
  }

  // 연결 대기 (서버)
  listen(backlog: number = 5): boolean {
    if (this.state !== 'CLOSED' || !this.localAddress) {
      return false;
    }

    this.state = 'LISTENING';
    console.log(`Socket ${this.id} listening (backlog: ${backlog})`);
    return true;
  }

  // 연결 시도 (클라이언트)
  connect(remoteAddress: SocketAddress): Promise<boolean> {
    return new Promise((resolve) => {
      if (this.state !== 'CLOSED') {
        resolve(false);
        return;
      }

      this.state = 'CONNECTING';
      this.remoteAddress = remoteAddress;
      
      // 연결 시뮬레이션
      setTimeout(() => {
        this.state = 'CONNECTED';
        console.log(`Socket ${this.id} connected to ${remoteAddress.address}:${remoteAddress.port || 'N/A'}`);
        resolve(true);
      }, 10);
    });
  }

  // 연결 수락 (서버)
  accept(): Promise<Socket> {
    return new Promise((resolve) => {
      if (this.state !== 'LISTENING') {
        throw new Error('Socket not listening');
      }

      // 클라이언트 연결 시뮬레이션
      setTimeout(() => {
        const clientSocket = new Socket(this.type);
        clientSocket.state = 'CONNECTED';
        clientSocket.localAddress = this.localAddress;
        clientSocket.remoteAddress = { family: 'AF_INET', address: '127.0.0.1', port: 12345 };
        
        console.log(`Accepted connection on socket ${clientSocket.id}`);
        resolve(clientSocket);
      }, 10);
    });
  }

  // 데이터 전송
  send(data: Uint8Array): number {
    if (this.state !== 'CONNECTED') {
      return 0;
    }

    console.log(`Socket ${this.id} sent ${data.length} bytes`);
    return data.length;
  }

  // 데이터 수신
  receive(): Uint8Array {
    if (this.buffer.length === 0) {
      return new Uint8Array(0);
    }

    const data = this.buffer.shift()!;
    console.log(`Socket ${this.id} received ${data.length} bytes`);
    return data;
  }

  // 소켓 닫기
  close(): void {
    this.state = 'CLOSED';
    console.log(`Socket ${this.id} closed`);
  }

  // 상태 정보
  getStatus(): any {
    return {
      id: this.id,
      type: this.type,
      state: this.state,
      localAddress: this.localAddress,
      remoteAddress: this.remoteAddress,
      bufferCount: this.buffer.length
    };
  }

  // 내부 메서드: 데이터 수신 시뮬레이션
  _simulateReceive(data: Uint8Array): void {
    this.buffer.push(data);
  }
}

// Socket Manager
class SocketManager {
  private sockets: Map<string, Socket> = new Map();
  private bindingTable: Map<string, Socket> = new Map();

  createSocket(type: 'TCP' | 'UDP' | 'UNIX', processId: string): Socket {
    const socket = new Socket(type);
    this.sockets.set(socket.id, socket);
    console.log(`Process ${processId} created ${type} socket ${socket.id}`);
    return socket;
  }

  bindSocket(socket: Socket, address: SocketAddress): boolean {
    const key = `${address.address}:${address.port || 0}`;
    
    if (this.bindingTable.has(key)) {
      console.log(`Address ${key} already in use`);
      return false;
    }

    if (socket.bind(address)) {
      this.bindingTable.set(key, socket);
      return true;
    }
    return false;
  }

  removeSocket(socket: Socket): void {
    this.sockets.delete(socket.id);
    
    // 바인딩 테이블에서도 제거
    for (const [key, boundSocket] of this.bindingTable) {
      if (boundSocket === socket) {
        this.bindingTable.delete(key);
        break;
      }
    }
    
    socket.close();
  }

  getSocketStatistics(): any {
    return {
      totalSockets: this.sockets.size,
      boundAddresses: this.bindingTable.size,
      sockets: Array.from(this.sockets.values()).map(s => s.getStatus())
    };
  }
}

// Socket 사용 예제
class SocketExample {
  async runClientServerExample(): Promise<void> {
    console.log('\n=== Socket Client-Server Example ===');

    const manager = new SocketManager();

    // 서버
    const server = async () => {
      const serverSocket = manager.createSocket('TCP', 'server');
      
      manager.bindSocket(serverSocket, {
        family: 'AF_INET',
        address: '127.0.0.1',
        port: 8080
      });
      
      serverSocket.listen(5);
      
      // 클라이언트 연결 수락
      const clientSocket = await serverSocket.accept();
      
      // 메시지 수신
      setTimeout(() => {
        const data = clientSocket.receive();
        if (data.length > 0) {
          const message = new TextDecoder().decode(data);
          console.log(`Server received: ${message}`);
          
          // 응답 전송
          const response = new TextEncoder().encode('Hello from server!');
          clientSocket.send(response);
        }
      }, 100);
    };

    // 클라이언트
    const client = async () => {
      await new Promise(resolve => setTimeout(resolve, 50));
      
      const clientSocket = manager.createSocket('TCP', 'client');
      
      await clientSocket.connect({
        family: 'AF_INET',
        address: '127.0.0.1',
        port: 8080
      });
      
      // 메시지 전송
      const message = new TextEncoder().encode('Hello from client!');
      clientSocket.send(message);
      
      // 시뮬레이션을 위해 서버 소켓의 receive 버퍼에 직접 추가
      // 실제 환경에서는 네트워크 스택이 처리
    };

    await Promise.all([server(), client()]);
    
    console.log('\nSocket Statistics:');
    console.log(manager.getSocketStatistics());
  }
}
```

---

## 💡 TypeScript 구현

### 통합 IPC 시스템

```typescript
class IPCSystem {
  private messageQueues: Map<string, MessageQueue> = new Map();
  private sharedMemoryManager: SharedMemoryManager;
  private namedPipeManager: NamedPipe;
  private socketManager: SocketManager;

  constructor() {
    this.sharedMemoryManager = new SharedMemoryManager();
    this.namedPipeManager = new NamedPipe();
    this.socketManager = new SocketManager();
  }

  // 통합 통신 인터페이스
  async communicate(
    mechanism: IPCType,
    operation: 'send' | 'receive',
    config: any
  ): Promise<any> {
    switch (mechanism) {
      case IPCType.MESSAGE_QUEUE:
        return this.handleMessageQueue(operation, config);
      
      case IPCType.SHARED_MEMORY:
        return this.handleSharedMemory(operation, config);
      
      case IPCType.FIFO:
        return this.handleNamedPipe(operation, config);
      
      case IPCType.SOCKET:
        return this.handleSocket(operation, config);
      
      default:
        throw new Error(`Unsupported IPC mechanism: ${mechanism}`);
    }
  }

  private async handleMessageQueue(operation: string, config: any): Promise<any> {
    const { queueName, processId, message } = config;
    
    if (!this.messageQueues.has(queueName)) {
      this.messageQueues.set(queueName, new MessageQueue());
    }
    
    const queue = this.messageQueues.get(queueName)!;
    
    if (operation === 'send') {
      return queue.send({
        sender: processId,
        receiver: message.receiver,
        data: message.data,
        priority: message.priority || 0
      });
    } else {
      return queue.receive(processId);
    }
  }

  private handleSharedMemory(operation: string, config: any): any {
    const { segmentKey, size, offset, data, processId } = config;
    
    if (operation === 'create') {
      return this.sharedMemoryManager.createSegment(segmentKey, size);
    } else if (operation === 'write') {
      const segment = this.sharedMemoryManager.attachSegment(segmentKey, processId);
      return segment?.write(offset, data, processId);
    } else if (operation === 'read') {
      const segment = this.sharedMemoryManager.attachSegment(segmentKey, processId);
      return segment?.read(offset, data.length, processId);
    }
  }

  private handleNamedPipe(operation: string, config: any): any {
    const { pipeName, mode, processId, data, size } = config;
    
    if (operation === 'create') {
      return this.namedPipeManager.createFIFO(pipeName);
    } else if (operation === 'write') {
      this.namedPipeManager.openFIFO(pipeName, 'write', processId);
      return this.namedPipeManager.writeFIFO(pipeName, data, processId);
    } else if (operation === 'read') {
      this.namedPipeManager.openFIFO(pipeName, 'read', processId);
      return this.namedPipeManager.readFIFO(pipeName, size, processId);
    }
  }

  private handleSocket(operation: string, config: any): any {
    const { type, address, processId } = config;
    
    if (operation === 'create') {
      return this.socketManager.createSocket(type, processId);
    }
    // 추가 소켓 작업들...
  }

  // IPC 성능 벤치마크
  async benchmarkIPC(): Promise<void> {
    console.log('\n=== IPC Performance Benchmark ===');
    
    const dataSize = 1024; // 1KB
    const iterations = 100;
    const testData = new TextEncoder().encode('x'.repeat(dataSize));
    
    const mechanisms: { name: string, type: IPCType, test: () => Promise<number> }[] = [
      {
        name: 'Message Queue',
        type: IPCType.MESSAGE_QUEUE,
        test: () => this.benchmarkMessageQueue(testData, iterations)
      },
      {
        name: 'Shared Memory',
        type: IPCType.SHARED_MEMORY,
        test: () => this.benchmarkSharedMemory(testData, iterations)
      }
    ];

    for (const mechanism of mechanisms) {
      const time = await mechanism.test();
      console.log(`${mechanism.name}: ${time.toFixed(2)}ms for ${iterations} operations`);
    }
  }

  private async benchmarkMessageQueue(data: Uint8Array, iterations: number): Promise<number> {
    const queue = new MessageQueue();
    const start = performance.now();
    
    for (let i = 0; i < iterations; i++) {
      await queue.send({
        sender: 'benchmark',
        receiver: 'benchmark',
        data: data,
        priority: 0
      });
      await queue.receive('benchmark');
    }
    
    return performance.now() - start;
  }

  private async benchmarkSharedMemory(data: Uint8Array, iterations: number): Promise<number> {
    const segment = this.sharedMemoryManager.createSegment('benchmark', data.length * 2);
    const start = performance.now();
    
    for (let i = 0; i < iterations; i++) {
      segment.write(0, data, 'benchmark');
      segment.read(0, data.length, 'benchmark');
    }
    
    return performance.now() - start;
  }
}

// IPC 사용 예제
class IPCDemo {
  async runCompleteDemo(): Promise<void> {
    const ipcSystem = new IPCSystem();
    
    console.log('=== Complete IPC System Demo ===');
    
    // 다양한 IPC 메커니즘 시연
    await this.demoMessagePassing(ipcSystem);
    await this.demoSharedMemory(ipcSystem);
    await this.demoPipes();
    await this.demoSockets();
    
    // 성능 비교
    await ipcSystem.benchmarkIPC();
  }

  private async demoMessagePassing(ipcSystem: IPCSystem): Promise<void> {
    console.log('\n--- Message Passing Demo ---');
    const example = new MessagePassingExample();
    await example.runProducerConsumer();
  }

  private async demoSharedMemory(ipcSystem: IPCSystem): Promise<void> {
    console.log('\n--- Shared Memory Demo ---');
    const example = new SharedMemoryExample();
    await example.runExample();
  }

  private async demoPipes(): Promise<void> {
    console.log('\n--- Pipe Demo ---');
    const example = new PipeExample();
    await example.runPipeExample();
    await example.runFIFOExample();
  }

  private async demoSockets(): Promise<void> {
    console.log('\n--- Socket Demo ---');
    const example = new SocketExample();
    await example.runClientServerExample();
  }
}
```

---

## 🔍 실제 사례

### 운영체제별 IPC 구현

```typescript
// POSIX IPC
class POSIXIPCExample {
  static getOverview(): string {
    return `
POSIX IPC Mechanisms:
1. Message Queues (mq_open, mq_send, mq_receive)
2. Semaphores (sem_open, sem_wait, sem_post)
3. Shared Memory (shm_open, mmap, munmap)
4. Pipes (pipe, mkfifo)
5. Signals (kill, signal, sigaction)
    `;
  }

  // System V IPC 예제
  static getSystemVExample(): string {
    return `
System V IPC:
- Message Queues: msgget, msgsnd, msgrcv
- Semaphores: semget, semop, semctl
- Shared Memory: shmget, shmat, shmdt
- IPC Keys: ftok() for unique identifiers
    `;
  }
}

// Windows IPC
class WindowsIPCExample {
  static getOverview(): string {
    return `
Windows IPC Mechanisms:
1. Named Pipes (CreateNamedPipe, ConnectNamedPipe)
2. Anonymous Pipes (CreatePipe)
3. Mailslots (CreateMailslot)
4. Memory-Mapped Files (CreateFileMapping, MapViewOfFile)
5. Windows Sockets (Winsock API)
6. COM (Component Object Model)
7. RPC (Remote Procedure Call)
    `;
  }
}

// Modern IPC patterns
class ModernIPCPatterns {
  static getPatterns(): string[] {
    return [
      'Message Brokers (RabbitMQ, Apache Kafka)',
      'gRPC (Protocol Buffers + HTTP/2)',
      'REST APIs (HTTP-based communication)',
      'WebSockets (Real-time bidirectional)',
      'Docker Container Communication',
      'Kubernetes Service Mesh',
      'Microservices Communication Patterns'
    ];
  }
}
```

> [!tip] 실무 팁
> IPC 메커니즘 선택 시 고려사항: 1) 성능 요구사항 (지연시간 vs 처리량), 2) 데이터 크기, 3) 프로세스 간 관계 (부모-자식 vs 독립), 4) 네트워크 투명성 필요 여부, 5) 동기화 요구사항. 높은 성능이 필요하면 공유 메모리, 네트워크 투명성이 필요하면 소켓, 단순한 통신은 파이프를 고려하세요.

## 📚 참고 자료
- Advanced Programming in the UNIX Environment (Stevens)
- Windows System Programming (Hart)
- The Linux Programming Interface (Kerrisk)
- POSIX.1 Standard Documentation