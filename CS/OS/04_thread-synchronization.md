---
tags:
  - os
  - thread
  - synchronization
  - concurrency
created: 2025-01-08
updated: 2025-01-08
---

# 스레드와 동기화

> [!info] 스레드란?
> 스레드는 프로세스 내에서 실행되는 경량 실행 단위입니다. 같은 프로세스의 스레드들은 코드, 데이터, 힙 영역을 공유하지만 각자의 스택과 레지스터를 가집니다.

## 📑 목차

- [[#🎯 스레드 개념]]
- [[#🔄 스레드 vs 프로세스]]
- [[#🏗️ 스레드 구현 방식]]
- [[#⚡ 동기화 문제]]
- [[#🔒 동기화 기법]]
- [[#💡 TypeScript로 구현하는 스레드]]
- [[#🔍 실제 사례]]

---

## 🎯 스레드 개념

### 스레드 기본 구조

```typescript
class Thread {
  private tid: number;  // Thread ID
  private process: Process;  // 소속 프로세스
  private state: ThreadState;
  private priority: number;
  
  // 스레드별 독립 자원
  private stack: ThreadStack;
  private registers: ThreadRegisters;
  private programCounter: number;
  
  // 프로세스와 공유하는 자원
  private sharedMemory: SharedMemory;
  private sharedFiles: FileTable;
  private sharedSignalHandlers: SignalHandlers;

  constructor(process: Process, tid: number) {
    this.tid = tid;
    this.process = process;
    this.state = ThreadState.CREATED;
    this.priority = 0;
    
    // 독립 자원 할당
    this.stack = new ThreadStack();
    this.registers = new ThreadRegisters();
    this.programCounter = 0;
    
    // 공유 자원 참조
    this.sharedMemory = process.getSharedMemory();
    this.sharedFiles = process.getFileTable();
    this.sharedSignalHandlers = process.getSignalHandlers();
  }

  // 스레드 실행
  async run(threadFunction: ThreadFunction): Promise<void> {
    this.state = ThreadState.RUNNING;
    
    try {
      // 스레드 함수 실행
      await threadFunction(this.tid);
      this.state = ThreadState.TERMINATED;
    } catch (error) {
      this.state = ThreadState.ERROR;
      throw error;
    }
  }

  // 스레드 일시 정지
  suspend(): void {
    if (this.state === ThreadState.RUNNING) {
      this.state = ThreadState.SUSPENDED;
      this.saveContext();
    }
  }

  // 스레드 재개
  resume(): void {
    if (this.state === ThreadState.SUSPENDED) {
      this.restoreContext();
      this.state = ThreadState.RUNNING;
    }
  }

  private saveContext(): void {
    // 레지스터와 프로그램 카운터 저장
    console.log(`Saving context for thread ${this.tid}`);
  }

  private restoreContext(): void {
    // 레지스터와 프로그램 카운터 복원
    console.log(`Restoring context for thread ${this.tid}`);
  }
}

enum ThreadState {
  CREATED = 'CREATED',
  RUNNING = 'RUNNING',
  SUSPENDED = 'SUSPENDED',
  BLOCKED = 'BLOCKED',
  TERMINATED = 'TERMINATED',
  ERROR = 'ERROR'
}

type ThreadFunction = (tid: number) => Promise<void>;

// 스레드 스택
class ThreadStack {
  private stack: any[] = [];
  private maxSize: number = 1024 * 1024; // 1MB
  private currentSize: number = 0;

  push(data: any): void {
    if (this.currentSize + 1 > this.maxSize) {
      throw new Error('Stack overflow');
    }
    this.stack.push(data);
    this.currentSize++;
  }

  pop(): any {
    if (this.stack.length === 0) {
      throw new Error('Stack underflow');
    }
    this.currentSize--;
    return this.stack.pop();
  }

  getSize(): number {
    return this.currentSize;
  }
}

// 스레드 레지스터
class ThreadRegisters {
  private registers: Map<string, number> = new Map([
    ['EAX', 0],
    ['EBX', 0],
    ['ECX', 0],
    ['EDX', 0],
    ['ESP', 0],  // Stack Pointer
    ['EBP', 0],  // Base Pointer
    ['ESI', 0],
    ['EDI', 0]
  ]);

  get(name: string): number {
    return this.registers.get(name) || 0;
  }

  set(name: string, value: number): void {
    this.registers.set(name, value);
  }

  getAll(): Map<string, number> {
    return new Map(this.registers);
  }

  setAll(registers: Map<string, number>): void {
    this.registers = new Map(registers);
  }
}

// 공유 메모리
class SharedMemory {
  private memory: Map<string, any> = new Map();
  private locks: Map<string, Lock> = new Map();

  read(key: string): any {
    return this.memory.get(key);
  }

  write(key: string, value: any): void {
    this.memory.set(key, value);
  }

  // 동기화된 읽기
  async safeRead(key: string): Promise<any> {
    const lock = this.getLock(key);
    await lock.acquire();
    try {
      return this.memory.get(key);
    } finally {
      lock.release();
    }
  }

  // 동기화된 쓰기
  async safeWrite(key: string, value: any): Promise<void> {
    const lock = this.getLock(key);
    await lock.acquire();
    try {
      this.memory.set(key, value);
    } finally {
      lock.release();
    }
  }

  private getLock(key: string): Lock {
    if (!this.locks.has(key)) {
      this.locks.set(key, new Lock());
    }
    return this.locks.get(key)!;
  }
}
```

---

## 🔄 스레드 vs 프로세스

### 비교 구현

```typescript
class ThreadVsProcessComparison {
  // 프로세스 생성 비용
  static measureProcessCreation(): PerformanceMetrics {
    const startTime = performance.now();
    const startMemory = process.memoryUsage().heapUsed;

    // 프로세스 생성 시뮬레이션
    const processResources = {
      addressSpace: new ArrayBuffer(10 * 1024 * 1024), // 10MB
      pageTable: new Map(),
      fileDescriptors: new Map(),
      signalHandlers: new Map(),
      pcb: {} // Process Control Block
    };

    const endTime = performance.now();
    const endMemory = process.memoryUsage().heapUsed;

    return {
      time: endTime - startTime,
      memory: endMemory - startMemory,
      type: 'Process'
    };
  }

  // 스레드 생성 비용
  static measureThreadCreation(): PerformanceMetrics {
    const startTime = performance.now();
    const startMemory = process.memoryUsage().heapUsed;

    // 스레드 생성 시뮬레이션
    const threadResources = {
      stack: new ArrayBuffer(1 * 1024 * 1024), // 1MB
      registers: new Map(),
      tcb: {} // Thread Control Block
      // 나머지는 프로세스와 공유
    };

    const endTime = performance.now();
    const endMemory = process.memoryUsage().heapUsed;

    return {
      time: endTime - startTime,
      memory: endMemory - startMemory,
      type: 'Thread'
    };
  }

  // 컨텍스트 스위칭 비용
  static compareContextSwitching(): void {
    console.log('\n=== Context Switching Comparison ===');
    
    // 프로세스 컨텍스트 스위칭
    class ProcessContextSwitch {
      static switch(from: Process, to: Process): number {
        const operations = [
          'Save CPU registers',
          'Save memory mappings',
          'Save I/O state',
          'Flush TLB',
          'Load new memory mappings',
          'Load new CPU registers',
          'Load new I/O state'
        ];
        
        const cost = operations.length * 10; // 각 작업당 10 단위 시간
        console.log(`Process switch cost: ${cost} units`);
        return cost;
      }
    }

    // 스레드 컨텍스트 스위칭
    class ThreadContextSwitch {
      static switch(from: Thread, to: Thread): number {
        const operations = [
          'Save CPU registers',
          'Save stack pointer',
          'Load new CPU registers',
          'Load new stack pointer'
        ];
        
        const cost = operations.length * 10; // 각 작업당 10 단위 시간
        console.log(`Thread switch cost: ${cost} units`);
        return cost;
      }
    }

    ProcessContextSwitch.switch({} as Process, {} as Process);
    ThreadContextSwitch.switch(new Thread({} as Process, 1), new Thread({} as Process, 2));
  }

  // 통신 비용 비교
  static compareCommunication(): void {
    console.log('\n=== Communication Comparison ===');

    // 프로세스 간 통신 (IPC)
    class InterProcessCommunication {
      private sharedMemory: SharedMemorySegment;
      private messageQueue: MessageQueue;
      private pipe: Pipe;

      sendMessage(data: any): number {
        // 메시지 직렬화
        const serialized = JSON.stringify(data);
        // 커널 호출
        // 메시지 복사
        // 컨텍스트 스위칭
        const cost = serialized.length * 2 + 100; // 복사 + 오버헤드
        console.log(`IPC cost: ${cost} units`);
        return cost;
      }
    }

    // 스레드 간 통신
    class InterThreadCommunication {
      private sharedData: any;

      sendMessage(data: any): number {
        // 직접 메모리 접근
        this.sharedData = data;
        const cost = 1; // 최소 비용
        console.log(`Thread communication cost: ${cost} units`);
        return cost;
      }
    }

    new InterProcessCommunication().sendMessage({ data: 'test' });
    new InterThreadCommunication().sendMessage({ data: 'test' });
  }
}

interface PerformanceMetrics {
  time: number;
  memory: number;
  type: string;
}

interface SharedMemorySegment {
  shmid: number;
  size: number;
  data: ArrayBuffer;
}

interface MessageQueue {
  mqid: number;
  messages: any[];
}

interface Pipe {
  readEnd: number;
  writeEnd: number;
}
```

---

## 🏗️ 스레드 구현 방식

### 사용자 수준 스레드 vs 커널 수준 스레드

```typescript
// 사용자 수준 스레드 (User-Level Thread)
class UserLevelThread {
  private scheduler: UserThreadScheduler;
  private context: UserThreadContext;
  
  constructor() {
    this.scheduler = new UserThreadScheduler();
    this.context = {
      registers: new Map(),
      stack: [],
      programCounter: 0
    };
  }

  // 장점: 빠른 컨텍스트 스위칭
  switchContext(to: UserLevelThread): void {
    // 사용자 공간에서 처리 (커널 호출 없음)
    this.saveContext();
    to.restoreContext();
    // 매우 빠름 (~100 나노초)
  }

  // 단점: 블로킹 시스템 콜 시 전체 프로세스 블록
  performBlockingIO(): void {
    console.log('Warning: Blocking I/O will block all user threads');
    // 모든 사용자 스레드가 블록됨
  }

  private saveContext(): void {
    // 사용자 공간에서 컨텍스트 저장
  }

  private restoreContext(): void {
    // 사용자 공간에서 컨텍스트 복원
  }
}

// 커널 수준 스레드 (Kernel-Level Thread)
class KernelLevelThread {
  private ktid: number; // Kernel Thread ID
  private kernelScheduler: KernelScheduler;
  
  constructor(ktid: number) {
    this.ktid = ktid;
    this.kernelScheduler = new KernelScheduler();
  }

  // 단점: 느린 컨텍스트 스위칭
  switchContext(to: KernelLevelThread): void {
    // 커널 모드 진입 필요
    this.enterKernelMode();
    // 커널 스케줄러 호출
    this.kernelScheduler.schedule(this, to);
    this.exitKernelMode();
    // 상대적으로 느림 (~1-10 마이크로초)
  }

  // 장점: 진정한 병렬 실행
  performBlockingIO(): void {
    console.log('Only this thread blocks, others continue');
    // 다른 스레드는 계속 실행 가능
  }

  private enterKernelMode(): void {
    // 시스템 콜을 통한 커널 진입
  }

  private exitKernelMode(): void {
    // 사용자 모드로 복귀
  }
}

interface UserThreadContext {
  registers: Map<string, number>;
  stack: any[];
  programCounter: number;
}

// 하이브리드 모델 (M:N 모델)
class HybridThreadModel {
  private userThreads: UserLevelThread[] = [];
  private kernelThreads: KernelLevelThread[] = [];
  private mapping: Map<UserLevelThread, KernelLevelThread> = new Map();

  constructor(userCount: number, kernelCount: number) {
    // M개의 사용자 스레드
    for (let i = 0; i < userCount; i++) {
      this.userThreads.push(new UserLevelThread());
    }

    // N개의 커널 스레드
    for (let i = 0; i < kernelCount; i++) {
      this.kernelThreads.push(new KernelLevelThread(i));
    }

    // 초기 매핑
    this.distributeThreads();
  }

  private distributeThreads(): void {
    // 사용자 스레드를 커널 스레드에 분배
    let kernelIndex = 0;
    for (const userThread of this.userThreads) {
      const kernelThread = this.kernelThreads[kernelIndex];
      this.mapping.set(userThread, kernelThread);
      kernelIndex = (kernelIndex + 1) % this.kernelThreads.length;
    }
  }

  // 동적 스레드 매핑 조정
  rebalance(): void {
    console.log('Rebalancing user threads across kernel threads');
    // 부하 분산을 위한 재매핑
    this.distributeThreads();
  }
}

class UserThreadScheduler {
  private readyQueue: UserLevelThread[] = [];
  private currentThread: UserLevelThread | null = null;

  schedule(): void {
    // 사용자 공간에서 스케줄링
    if (this.readyQueue.length > 0) {
      this.currentThread = this.readyQueue.shift()!;
    }
  }
}

class KernelScheduler {
  schedule(from: KernelLevelThread, to: KernelLevelThread): void {
    // 커널 레벨 스케줄링
    console.log(`Kernel scheduling: Thread ${from.ktid} -> Thread ${to.ktid}`);
  }
}
```

---

## ⚡ 동기화 문제

### 경쟁 조건 (Race Condition)

```typescript
class RaceConditionExample {
  private counter: number = 0;
  
  // 문제: 동기화되지 않은 접근
  async unsafeIncrement(): Promise<void> {
    const threads: Promise<void>[] = [];
    
    for (let i = 0; i < 10; i++) {
      threads.push(this.threadIncrement(i));
    }
    
    await Promise.all(threads);
    console.log(`Final counter (unsafe): ${this.counter}`);
    // 예상: 10000, 실제: 예측 불가능
  }

  private async threadIncrement(tid: number): Promise<void> {
    for (let i = 0; i < 1000; i++) {
      // 경쟁 조건 발생
      const temp = this.counter;  // Read
      const newValue = temp + 1;   // Modify
      await this.simulateDelay();  // 컨텍스트 스위칭 시뮬레이션
      this.counter = newValue;      // Write
    }
  }

  private simulateDelay(): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, 0));
  }

  // 해결: 동기화 적용
  async safeIncrement(): Promise<void> {
    const lock = new Mutex();
    const threads: Promise<void>[] = [];
    
    for (let i = 0; i < 10; i++) {
      threads.push(this.threadSafeIncrement(i, lock));
    }
    
    await Promise.all(threads);
    console.log(`Final counter (safe): ${this.counter}`);
    // 예상: 10000, 실제: 10000
  }

  private async threadSafeIncrement(tid: number, lock: Mutex): Promise<void> {
    for (let i = 0; i < 1000; i++) {
      await lock.acquire();
      try {
        // 임계 구역
        const temp = this.counter;
        const newValue = temp + 1;
        await this.simulateDelay();
        this.counter = newValue;
      } finally {
        lock.release();
      }
    }
  }
}

// 임계 구역 (Critical Section)
class CriticalSection {
  private resource: any;
  private lock: Lock;

  constructor() {
    this.resource = { data: 0 };
    this.lock = new Lock();
  }

  // Peterson의 알고리즘
  async petersonAlgorithm(threadId: 0 | 1): Promise<void> {
    const other = 1 - threadId;
    const flag = [false, false];
    let turn = 0;

    // Entry section
    flag[threadId] = true;
    turn = other;
    
    // Busy waiting
    while (flag[other] && turn === other) {
      await this.simulateDelay();
    }

    // Critical section
    console.log(`Thread ${threadId} in critical section`);
    this.resource.data++;

    // Exit section
    flag[threadId] = false;
  }

  private simulateDelay(): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, 1));
  }
}

// 생산자-소비자 문제
class ProducerConsumerProblem {
  private buffer: BoundedBuffer<any>;
  private mutex: Mutex;
  private empty: Semaphore;
  private full: Semaphore;

  constructor(bufferSize: number) {
    this.buffer = new BoundedBuffer(bufferSize);
    this.mutex = new Mutex();
    this.empty = new Semaphore(bufferSize);
    this.full = new Semaphore(0);
  }

  async producer(item: any): Promise<void> {
    await this.empty.wait();  // 빈 슬롯 대기
    await this.mutex.acquire();
    
    try {
      // 임계 구역
      this.buffer.put(item);
      console.log(`Produced: ${item}`);
    } finally {
      this.mutex.release();
    }
    
    this.full.signal();  // 아이템 생산 알림
  }

  async consumer(): Promise<any> {
    await this.full.wait();  // 아이템 대기
    await this.mutex.acquire();
    
    let item;
    try {
      // 임계 구역
      item = this.buffer.get();
      console.log(`Consumed: ${item}`);
    } finally {
      this.mutex.release();
    }
    
    this.empty.signal();  // 빈 슬롯 알림
    return item;
  }
}

class BoundedBuffer<T> {
  private buffer: T[] = [];
  private capacity: number;

  constructor(capacity: number) {
    this.capacity = capacity;
  }

  put(item: T): void {
    if (this.buffer.length >= this.capacity) {
      throw new Error('Buffer full');
    }
    this.buffer.push(item);
  }

  get(): T {
    if (this.buffer.length === 0) {
      throw new Error('Buffer empty');
    }
    return this.buffer.shift()!;
  }

  isFull(): boolean {
    return this.buffer.length >= this.capacity;
  }

  isEmpty(): boolean {
    return this.buffer.length === 0;
  }
}
```

---

## 🔒 동기화 기법

### 뮤텍스 (Mutex)

```typescript
class Mutex {
  private locked: boolean = false;
  private owner: number | null = null;
  private waitQueue: ((value: void) => void)[] = [];

  async acquire(tid?: number): Promise<void> {
    while (this.locked) {
      await new Promise<void>(resolve => {
        this.waitQueue.push(resolve);
      });
    }
    
    this.locked = true;
    this.owner = tid || null;
  }

  release(): void {
    if (!this.locked) {
      throw new Error('Mutex not locked');
    }
    
    this.locked = false;
    this.owner = null;
    
    // 대기 중인 스레드 깨우기
    const waiter = this.waitQueue.shift();
    if (waiter) {
      waiter();
    }
  }

  tryAcquire(tid?: number): boolean {
    if (!this.locked) {
      this.locked = true;
      this.owner = tid || null;
      return true;
    }
    return false;
  }

  isLocked(): boolean {
    return this.locked;
  }

  getOwner(): number | null {
    return this.owner;
  }
}

// 세마포어 (Semaphore)
class Semaphore {
  private count: number;
  private waitQueue: ((value: void) => void)[] = [];

  constructor(initialCount: number) {
    this.count = initialCount;
  }

  async wait(): Promise<void> {
    this.count--;
    
    if (this.count < 0) {
      await new Promise<void>(resolve => {
        this.waitQueue.push(resolve);
      });
    }
  }

  signal(): void {
    this.count++;
    
    if (this.count <= 0) {
      const waiter = this.waitQueue.shift();
      if (waiter) {
        waiter();
      }
    }
  }

  getCount(): number {
    return this.count;
  }
}

// 조건 변수 (Condition Variable)
class ConditionVariable {
  private waitQueue: ((value: void) => void)[] = [];

  async wait(mutex: Mutex): Promise<void> {
    mutex.release();
    
    await new Promise<void>(resolve => {
      this.waitQueue.push(resolve);
    });
    
    await mutex.acquire();
  }

  signal(): void {
    const waiter = this.waitQueue.shift();
    if (waiter) {
      waiter();
    }
  }

  broadcast(): void {
    while (this.waitQueue.length > 0) {
      const waiter = this.waitQueue.shift()!;
      waiter();
    }
  }
}

// 읽기-쓰기 락 (Read-Write Lock)
class ReadWriteLock {
  private readers: number = 0;
  private writers: number = 0;
  private writeWaiters: number = 0;
  private mutex: Mutex = new Mutex();
  private canRead: ConditionVariable = new ConditionVariable();
  private canWrite: ConditionVariable = new ConditionVariable();

  async acquireRead(): Promise<void> {
    await this.mutex.acquire();
    
    while (this.writers > 0 || this.writeWaiters > 0) {
      await this.canRead.wait(this.mutex);
    }
    
    this.readers++;
    this.mutex.release();
  }

  async releaseRead(): Promise<void> {
    await this.mutex.acquire();
    
    this.readers--;
    
    if (this.readers === 0) {
      this.canWrite.signal();
    }
    
    this.mutex.release();
  }

  async acquireWrite(): Promise<void> {
    await this.mutex.acquire();
    
    this.writeWaiters++;
    
    while (this.readers > 0 || this.writers > 0) {
      await this.canWrite.wait(this.mutex);
    }
    
    this.writeWaiters--;
    this.writers++;
    
    this.mutex.release();
  }

  async releaseWrite(): Promise<void> {
    await this.mutex.acquire();
    
    this.writers--;
    
    this.canWrite.signal();
    this.canRead.broadcast();
    
    this.mutex.release();
  }
}

// 스핀락 (Spinlock)
class Spinlock {
  private locked: boolean = false;

  async acquire(): Promise<void> {
    while (true) {
      if (!this.locked) {
        // Test-and-Set 원자적 연산
        const wasLocked = this.testAndSet();
        if (!wasLocked) {
          break;
        }
      }
      // Busy waiting
      await this.yield();
    }
  }

  release(): void {
    this.locked = false;
  }

  private testAndSet(): boolean {
    const wasLocked = this.locked;
    this.locked = true;
    return wasLocked;
  }

  private yield(): Promise<void> {
    return new Promise(resolve => setImmediate(resolve));
  }
}

// 모니터 (Monitor)
class Monitor {
  private mutex: Mutex = new Mutex();
  private conditions: Map<string, ConditionVariable> = new Map();

  async enter(): Promise<void> {
    await this.mutex.acquire();
  }

  exit(): void {
    this.mutex.release();
  }

  async wait(conditionName: string): Promise<void> {
    if (!this.conditions.has(conditionName)) {
      this.conditions.set(conditionName, new ConditionVariable());
    }
    
    const condition = this.conditions.get(conditionName)!;
    await condition.wait(this.mutex);
  }

  signal(conditionName: string): void {
    const condition = this.conditions.get(conditionName);
    if (condition) {
      condition.signal();
    }
  }

  broadcast(conditionName: string): void {
    const condition = this.conditions.get(conditionName);
    if (condition) {
      condition.broadcast();
    }
  }
}
```

---

## 💡 TypeScript로 구현하는 스레드

### 완전한 스레드 풀 구현

```typescript
class ThreadPool {
  private workers: Worker[] = [];
  private taskQueue: Task[] = [];
  private activeWorkers: number = 0;
  private maxWorkers: number;
  private mutex: Mutex = new Mutex();
  private taskAvailable: ConditionVariable = new ConditionVariable();
  private allTasksComplete: ConditionVariable = new ConditionVariable();
  private shutdown: boolean = false;

  constructor(maxWorkers: number) {
    this.maxWorkers = maxWorkers;
    this.initializeWorkers();
  }

  private initializeWorkers(): void {
    for (let i = 0; i < this.maxWorkers; i++) {
      const worker = new Worker(i, this);
      this.workers.push(worker);
      worker.start();
    }
  }

  async submit(task: Task): Promise<void> {
    await this.mutex.acquire();
    
    try {
      if (this.shutdown) {
        throw new Error('ThreadPool is shutting down');
      }
      
      this.taskQueue.push(task);
      this.taskAvailable.signal();
    } finally {
      this.mutex.release();
    }
  }

  async getTask(): Promise<Task | null> {
    await this.mutex.acquire();
    
    try {
      while (this.taskQueue.length === 0 && !this.shutdown) {
        await this.taskAvailable.wait(this.mutex);
      }
      
      if (this.shutdown && this.taskQueue.length === 0) {
        return null;
      }
      
      return this.taskQueue.shift() || null;
    } finally {
      this.mutex.release();
    }
  }

  async shutdownAndWait(): Promise<void> {
    await this.mutex.acquire();
    
    try {
      this.shutdown = true;
      this.taskAvailable.broadcast();
      
      while (this.activeWorkers > 0 || this.taskQueue.length > 0) {
        await this.allTasksComplete.wait(this.mutex);
      }
    } finally {
      this.mutex.release();
    }
  }

  workerStarted(): void {
    this.activeWorkers++;
  }

  workerFinished(): void {
    this.activeWorkers--;
    if (this.activeWorkers === 0 && this.taskQueue.length === 0) {
      this.allTasksComplete.signal();
    }
  }
}

class Worker {
  private id: number;
  private pool: ThreadPool;
  private running: boolean = false;

  constructor(id: number, pool: ThreadPool) {
    this.id = id;
    this.pool = pool;
  }

  async start(): Promise<void> {
    this.running = true;
    this.pool.workerStarted();
    
    while (this.running) {
      const task = await this.pool.getTask();
      
      if (!task) {
        break;
      }
      
      await this.executeTask(task);
    }
    
    this.pool.workerFinished();
  }

  private async executeTask(task: Task): Promise<void> {
    console.log(`Worker ${this.id} executing task ${task.id}`);
    
    try {
      await task.execute();
      console.log(`Worker ${this.id} completed task ${task.id}`);
    } catch (error) {
      console.error(`Worker ${this.id} failed task ${task.id}:`, error);
    }
  }

  stop(): void {
    this.running = false;
  }
}

interface Task {
  id: string;
  execute(): Promise<void>;
}

// 락-프리 데이터 구조
class LockFreeQueue<T> {
  private head: AtomicNode<T> | null = null;
  private tail: AtomicNode<T> | null = null;

  enqueue(value: T): void {
    const newNode = new AtomicNode(value);
    
    while (true) {
      const last = this.tail;
      const next = last?.next || null;
      
      if (last === this.tail) {
        if (next === null) {
          // CAS (Compare-And-Swap) 연산
          if (this.compareAndSwapNext(last, null, newNode)) {
            this.compareAndSwapTail(last, newNode);
            break;
          }
        } else {
          this.compareAndSwapTail(last, next);
        }
      }
    }
  }

  dequeue(): T | null {
    while (true) {
      const first = this.head;
      const last = this.tail;
      const next = first?.next || null;
      
      if (first === this.head) {
        if (first === last) {
          if (next === null) {
            return null;
          }
          this.compareAndSwapTail(last, next);
        } else {
          const value = next?.value;
          if (this.compareAndSwapHead(first, next)) {
            return value || null;
          }
        }
      }
    }
  }

  private compareAndSwapNext(node: AtomicNode<T> | null, expected: AtomicNode<T> | null, newValue: AtomicNode<T>): boolean {
    // 원자적 CAS 연산 시뮬레이션
    if (node && node.next === expected) {
      node.next = newValue;
      return true;
    }
    return false;
  }

  private compareAndSwapHead(expected: AtomicNode<T> | null, newValue: AtomicNode<T> | null): boolean {
    if (this.head === expected) {
      this.head = newValue;
      return true;
    }
    return false;
  }

  private compareAndSwapTail(expected: AtomicNode<T> | null, newValue: AtomicNode<T> | null): boolean {
    if (this.tail === expected) {
      this.tail = newValue;
      return true;
    }
    return false;
  }
}

class AtomicNode<T> {
  value: T;
  next: AtomicNode<T> | null = null;

  constructor(value: T) {
    this.value = value;
  }
}

// 베리어 (Barrier)
class Barrier {
  private count: number;
  private waiting: number = 0;
  private mutex: Mutex = new Mutex();
  private condition: ConditionVariable = new ConditionVariable();
  private generation: number = 0;

  constructor(count: number) {
    this.count = count;
  }

  async wait(): Promise<void> {
    await this.mutex.acquire();
    
    try {
      const myGeneration = this.generation;
      this.waiting++;
      
      if (this.waiting === this.count) {
        // 모든 스레드가 도착
        this.waiting = 0;
        this.generation++;
        this.condition.broadcast();
      } else {
        // 다른 스레드 대기
        while (myGeneration === this.generation) {
          await this.condition.wait(this.mutex);
        }
      }
    } finally {
      this.mutex.release();
    }
  }
}
```

---

## 🔍 실제 사례

### POSIX 스레드 (pthread)

```typescript
// pthread 시뮬레이션
class PThread {
  private tid: number;
  private attr: PThreadAttr;
  private startRoutine: (arg: any) => any;
  private arg: any;
  private result: any;
  private joinable: boolean = true;

  static create(
    startRoutine: (arg: any) => any,
    arg: any,
    attr?: PThreadAttr
  ): PThread {
    const thread = new PThread();
    thread.startRoutine = startRoutine;
    thread.arg = arg;
    thread.attr = attr || new PThreadAttr();
    thread.tid = Math.floor(Math.random() * 100000);
    
    // 스레드 시작
    thread.run();
    
    return thread;
  }

  private async run(): Promise<void> {
    // 스레드 실행
    this.result = await this.startRoutine(this.arg);
  }

  async join(): Promise<any> {
    if (!this.joinable) {
      throw new Error('Thread is not joinable');
    }
    
    // 스레드 종료 대기
    while (this.result === undefined) {
      await new Promise(resolve => setTimeout(resolve, 10));
    }
    
    return this.result;
  }

  detach(): void {
    this.joinable = false;
  }

  cancel(): void {
    // 스레드 취소
    console.log(`Thread ${this.tid} cancelled`);
  }

  static self(): number {
    // 현재 스레드 ID 반환
    return 0; // 시뮬레이션
  }

  static exit(value: any): never {
    throw new Error(`Thread exit with value: ${value}`);
  }
}

class PThreadAttr {
  detachState: DetachState = DetachState.JOINABLE;
  schedPolicy: SchedPolicy = SchedPolicy.OTHER;
  schedPriority: number = 0;
  stackSize: number = 1024 * 1024; // 1MB
}

enum DetachState {
  JOINABLE = 'JOINABLE',
  DETACHED = 'DETACHED'
}

enum SchedPolicy {
  OTHER = 'OTHER',
  FIFO = 'FIFO',
  RR = 'RR'
}

// pthread 뮤텍스
class PThreadMutex {
  private locked: boolean = false;
  private owner: number | null = null;
  private attr: PThreadMutexAttr;

  constructor(attr?: PThreadMutexAttr) {
    this.attr = attr || new PThreadMutexAttr();
  }

  lock(): void {
    const tid = PThread.self();
    
    if (this.attr.type === MutexType.RECURSIVE && this.owner === tid) {
      // 재귀적 뮤텍스
      return;
    }
    
    while (this.locked) {
      // Busy wait
    }
    
    this.locked = true;
    this.owner = tid;
  }

  trylock(): boolean {
    if (!this.locked) {
      this.locked = true;
      this.owner = PThread.self();
      return true;
    }
    return false;
  }

  unlock(): void {
    if (this.owner !== PThread.self() && this.attr.type !== MutexType.ERRORCHECK) {
      // 에러 체크 뮤텍스가 아니면 무시
    }
    
    this.locked = false;
    this.owner = null;
  }
}

class PThreadMutexAttr {
  type: MutexType = MutexType.NORMAL;
  protocol: MutexProtocol = MutexProtocol.NONE;
}

enum MutexType {
  NORMAL = 'NORMAL',
  ERRORCHECK = 'ERRORCHECK',
  RECURSIVE = 'RECURSIVE'
}

enum MutexProtocol {
  NONE = 'NONE',
  INHERIT = 'INHERIT',
  PROTECT = 'PROTECT'
}

// Java 스타일 동기화
class JavaStyleSync {
  // synchronized 메서드
  private lock = new Lock();
  private counter = 0;

  async synchronizedMethod(): Promise<void> {
    await this.lock.acquire();
    try {
      // 임계 구역
      this.counter++;
      console.log(`Counter: ${this.counter}`);
    } finally {
      this.lock.release();
    }
  }

  // synchronized 블록
  async synchronizedBlock(monitor: any): Promise<void> {
    const monitorLock = this.getMonitorLock(monitor);
    await monitorLock.acquire();
    try {
      // 임계 구역
      console.log('In synchronized block');
    } finally {
      monitorLock.release();
    }
  }

  private monitorLocks = new WeakMap<any, Lock>();
  
  private getMonitorLock(obj: any): Lock {
    if (!this.monitorLocks.has(obj)) {
      this.monitorLocks.set(obj, new Lock());
    }
    return this.monitorLocks.get(obj)!;
  }

  // wait/notify 메커니즘
  async wait(monitor: any): Promise<void> {
    const monitorLock = this.getMonitorLock(monitor);
    // wait는 락을 해제하고 대기
    monitorLock.release();
    // 알림 대기
    await new Promise(resolve => {
      // notify/notifyAll 대기
    });
    // 락 재획득
    await monitorLock.acquire();
  }

  notify(monitor: any): void {
    // 대기 중인 스레드 하나 깨우기
    console.log('Notify one waiting thread');
  }

  notifyAll(monitor: any): void {
    // 대기 중인 모든 스레드 깨우기
    console.log('Notify all waiting threads');
  }
}

// 기본 Lock 클래스
class Lock {
  private locked: boolean = false;
  private queue: (() => void)[] = [];

  async acquire(): Promise<void> {
    if (!this.locked) {
      this.locked = true;
      return;
    }

    return new Promise(resolve => {
      this.queue.push(resolve);
    });
  }

  release(): void {
    if (!this.locked) {
      throw new Error('Lock is not acquired');
    }

    const next = this.queue.shift();
    if (next) {
      next();
    } else {
      this.locked = false;
    }
  }
}
```

> [!tip] 실무 팁
> 스레드 동기화는 매우 중요하지만 복잡합니다. 가능한 한 높은 수준의 동기화 기법(모니터, 세마포어)을 사용하고, 락의 순서를 일관되게 유지하여 데드락을 방지하세요. 또한 락을 잡은 시간을 최소화하여 성능을 향상시킬 수 있습니다.

## 📚 참고 자료
- Operating System Concepts (Silberschatz)
- The Art of Multiprocessor Programming (Herlihy & Shavit)
- Java Concurrency in Practice (Goetz)
- POSIX Threads Programming (Nichols, Buttlar, Farrell)