---
tags:
  - os
  - process
  - memory-layout
  - process-state
created: 2025-01-08
updated: 2025-01-08
---

# 프로세스 관리

> [!info] 프로세스란?
> 프로세스는 실행 중인 프로그램의 인스턴스입니다. 메모리에 적재되어 CPU에 의해 실행되는 프로그램으로, 독립적인 메모리 공간과 시스템 자원을 할당받습니다.

## 📑 목차

- [[#🎯 프로세스 개념]]
- [[#🏗️ 프로세스 메모리 구조]]
- [[#🔄 프로세스 상태 전이]]
- [[#📊 프로세스 제어 블록 (PCB)]]
- [[#💡 TypeScript로 구현하는 프로세스]]
- [[#🔍 실제 사례]]

---

## 🎯 프로세스 개념

### 프로세스 vs 프로그램

```typescript
// 프로그램: 디스크에 저장된 실행 파일
class Program {
  private readonly executable: string;
  private readonly size: number;
  private readonly metadata: ProgramMetadata;

  constructor(path: string) {
    this.executable = path;
    this.size = this.getFileSize(path);
    this.metadata = this.loadMetadata(path);
  }

  private getFileSize(path: string): number {
    // 파일 크기 반환
    return 1024 * 100; // 100KB 예시
  }

  private loadMetadata(path: string): ProgramMetadata {
    return {
      name: path.split('/').pop() || 'unknown',
      version: '1.0.0',
      author: 'system',
      permissions: 0o755
    };
  }
}

// 프로세스: 메모리에서 실행 중인 프로그램
class Process {
  readonly pid: number;
  readonly program: Program;
  private state: ProcessState;
  private memory: ProcessMemory;
  private resources: ProcessResources;
  private context: ProcessContext;

  constructor(pid: number, program: Program) {
    this.pid = pid;
    this.program = program;
    this.state = ProcessState.NEW;
    this.memory = new ProcessMemory();
    this.resources = new ProcessResources();
    this.context = new ProcessContext();
  }

  // 프로세스 실행
  execute(): void {
    this.state = ProcessState.RUNNING;
    console.log(`Process ${this.pid} is running`);
  }

  // 프로세스 종료
  terminate(exitCode: number): void {
    this.state = ProcessState.TERMINATED;
    this.releaseResources();
    console.log(`Process ${this.pid} terminated with code ${exitCode}`);
  }

  private releaseResources(): void {
    this.memory.release();
    this.resources.releaseAll();
  }
}

interface ProgramMetadata {
  name: string;
  version: string;
  author: string;
  permissions: number;
}

enum ProcessState {
  NEW = 'NEW',
  READY = 'READY',
  RUNNING = 'RUNNING',
  WAITING = 'WAITING',
  TERMINATED = 'TERMINATED'
}
```

### 프로세스 특징

```typescript
class ProcessCharacteristics {
  // 1. 독립성
  static demonstrateIndependence(): void {
    class IndependentProcess {
      private memory: Map<string, any> = new Map();
      private pid: number;

      constructor(pid: number) {
        this.pid = pid;
        // 각 프로세스는 독립적인 메모리 공간을 가짐
        this.memory.set('data', `Process ${pid} data`);
      }

      accessMemory(address: string): any {
        // 자신의 메모리 공간만 접근 가능
        return this.memory.get(address);
      }

      // 다른 프로세스의 메모리는 직접 접근 불가
      cannotAccessOtherProcess(other: IndependentProcess): void {
        // 이것은 불가능 (보호됨)
        // const data = other.memory.get('data'); // Error!
        console.log('Cannot directly access other process memory');
      }
    }

    const process1 = new IndependentProcess(1);
    const process2 = new IndependentProcess(2);
    process1.cannotAccessOtherProcess(process2);
  }

  // 2. 동시성
  static demonstrateConcurrency(): void {
    class ConcurrentProcessManager {
      private processes: Process[] = [];
      private currentIndex: number = 0;

      // 시분할을 통한 동시 실행 시뮬레이션
      async runConcurrently(): Promise<void> {
        const timeSlice = 10; // ms

        while (this.hasRunningProcesses()) {
          const process = this.getNextProcess();
          
          // 타임 슬라이스 동안 실행
          await this.executeForTimeSlice(process, timeSlice);
          
          // 다음 프로세스로 전환
          this.switchToNext();
        }
      }

      private async executeForTimeSlice(process: Process, ms: number): Promise<void> {
        console.log(`Running process ${process.pid} for ${ms}ms`);
        await new Promise(resolve => setTimeout(resolve, ms));
      }

      private hasRunningProcesses(): boolean {
        return this.processes.some(p => p.state !== ProcessState.TERMINATED);
      }

      private getNextProcess(): Process {
        return this.processes[this.currentIndex];
      }

      private switchToNext(): void {
        this.currentIndex = (this.currentIndex + 1) % this.processes.length;
      }
    }
  }

  // 3. 자원 소유
  static demonstrateResourceOwnership(): void {
    interface SystemResource {
      type: ResourceType;
      id: number;
      owner?: number; // Process ID
    }

    enum ResourceType {
      FILE = 'FILE',
      SOCKET = 'SOCKET',
      MEMORY = 'MEMORY',
      DEVICE = 'DEVICE'
    }

    class ResourceManager {
      private resources: Map<number, SystemResource> = new Map();
      private processResources: Map<number, Set<number>> = new Map();

      allocateResource(pid: number, type: ResourceType): number {
        const resourceId = this.generateResourceId();
        const resource: SystemResource = {
          type,
          id: resourceId,
          owner: pid
        };

        this.resources.set(resourceId, resource);
        
        if (!this.processResources.has(pid)) {
          this.processResources.set(pid, new Set());
        }
        this.processResources.get(pid)!.add(resourceId);

        return resourceId;
      }

      releaseProcessResources(pid: number): void {
        const resources = this.processResources.get(pid);
        if (resources) {
          resources.forEach(resourceId => {
            this.resources.delete(resourceId);
          });
          this.processResources.delete(pid);
        }
      }

      private generateResourceId(): number {
        return Math.floor(Math.random() * 100000);
      }
    }
  }
}
```

---

## 🏗️ 프로세스 메모리 구조

### 메모리 레이아웃

```typescript
class ProcessMemory {
  private segments: MemorySegments;
  private pageTable: Map<number, number>; // 가상 -> 물리 주소 매핑

  constructor() {
    this.segments = {
      text: new TextSegment(),
      data: new DataSegment(),
      bss: new BSSSegment(),
      heap: new HeapSegment(),
      stack: new StackSegment()
    };
    this.pageTable = new Map();
  }

  // 메모리 할당
  allocate(size: number, segment: SegmentType): number {
    switch (segment) {
      case SegmentType.HEAP:
        return this.segments.heap.allocate(size);
      case SegmentType.STACK:
        return this.segments.stack.allocate(size);
      default:
        throw new Error(`Cannot allocate in ${segment} segment`);
    }
  }

  // 메모리 해제
  free(address: number): void {
    // 힙 영역의 메모리만 명시적으로 해제 가능
    this.segments.heap.free(address);
  }

  // 메모리 읽기
  read(address: number, size: number): Buffer {
    const segment = this.findSegment(address);
    return segment.read(address, size);
  }

  // 메모리 쓰기
  write(address: number, data: Buffer): void {
    const segment = this.findSegment(address);
    segment.write(address, data);
  }

  private findSegment(address: number): MemorySegment {
    // 주소 범위로 세그먼트 찾기
    for (const segment of Object.values(this.segments)) {
      if (segment.contains(address)) {
        return segment;
      }
    }
    throw new Error(`Invalid memory address: ${address}`);
  }

  getMemoryMap(): string {
    return `
Memory Layout:
━━━━━━━━━━━━━━━━━━━━━
┌─────────────────┐ 0xFFFFFFFF
│     Kernel      │
├─────────────────┤
│      Stack      │ ↓ (grows down)
│        ↓        │
├─────────────────┤
│                 │
│   Free Space    │
│                 │
├─────────────────┤
│        ↑        │
│      Heap       │ ↑ (grows up)
├─────────────────┤
│       BSS       │ (uninitialized data)
├─────────────────┤
│      Data       │ (initialized data)
├─────────────────┤
│      Text       │ (code)
└─────────────────┘ 0x00000000
    `;
  }
}

interface MemorySegments {
  text: TextSegment;
  data: DataSegment;
  bss: BSSSegment;
  heap: HeapSegment;
  stack: StackSegment;
}

enum SegmentType {
  TEXT = 'TEXT',
  DATA = 'DATA',
  BSS = 'BSS',
  HEAP = 'HEAP',
  STACK = 'STACK'
}

// 각 세그먼트 구현
abstract class MemorySegment {
  protected baseAddress: number;
  protected size: number;
  protected data: Buffer;
  protected permissions: MemoryPermissions;

  constructor(baseAddress: number, size: number, permissions: MemoryPermissions) {
    this.baseAddress = baseAddress;
    this.size = size;
    this.data = Buffer.alloc(size);
    this.permissions = permissions;
  }

  contains(address: number): boolean {
    return address >= this.baseAddress && 
           address < this.baseAddress + this.size;
  }

  read(address: number, size: number): Buffer {
    if (!this.permissions.read) {
      throw new Error('Read permission denied');
    }
    const offset = address - this.baseAddress;
    return this.data.slice(offset, offset + size);
  }

  write(address: number, data: Buffer): void {
    if (!this.permissions.write) {
      throw new Error('Write permission denied');
    }
    const offset = address - this.baseAddress;
    data.copy(this.data, offset);
  }
}

interface MemoryPermissions {
  read: boolean;
  write: boolean;
  execute: boolean;
}

// Text Segment (코드 영역)
class TextSegment extends MemorySegment {
  constructor() {
    super(0x00400000, 0x100000, { read: true, write: false, execute: true });
  }

  loadCode(machineCode: Buffer): void {
    // 실행 코드 로드
    machineCode.copy(this.data);
  }
}

// Data Segment (초기화된 전역 변수)
class DataSegment extends MemorySegment {
  private variables: Map<string, { offset: number; size: number }> = new Map();

  constructor() {
    super(0x00500000, 0x100000, { read: true, write: true, execute: false });
  }

  declareVariable(name: string, initialValue: any): void {
    const size = Buffer.byteLength(JSON.stringify(initialValue));
    const offset = this.allocateSpace(size);
    this.variables.set(name, { offset, size });
    
    const buffer = Buffer.from(JSON.stringify(initialValue));
    buffer.copy(this.data, offset);
  }

  private allocateSpace(size: number): number {
    // 간단한 할당 로직
    return Math.floor(Math.random() * (this.size - size));
  }
}

// BSS Segment (초기화되지 않은 전역 변수)
class BSSSegment extends MemorySegment {
  constructor() {
    super(0x00600000, 0x100000, { read: true, write: true, execute: false });
    // BSS는 0으로 초기화됨
    this.data.fill(0);
  }
}

// Heap Segment (동적 메모리)
class HeapSegment extends MemorySegment {
  private freeList: FreeBlock[] = [];
  private nextAddress: number;

  constructor() {
    super(0x00700000, 0x1000000, { read: true, write: true, execute: false });
    this.nextAddress = this.baseAddress;
    this.freeList.push({
      address: this.baseAddress,
      size: this.size
    });
  }

  allocate(size: number): number {
    // First Fit 알고리즘
    for (let i = 0; i < this.freeList.length; i++) {
      const block = this.freeList[i];
      if (block.size >= size) {
        const address = block.address;
        
        if (block.size === size) {
          this.freeList.splice(i, 1);
        } else {
          block.address += size;
          block.size -= size;
        }
        
        return address;
      }
    }
    throw new Error('Out of heap memory');
  }

  free(address: number): void {
    // 메모리 해제 및 병합
    console.log(`Freeing memory at ${address.toString(16)}`);
    // 실제 구현은 더 복잡함
  }
}

interface FreeBlock {
  address: number;
  size: number;
}

// Stack Segment (지역 변수, 함수 호출)
class StackSegment extends MemorySegment {
  private stackPointer: number;
  private basePointer: number;

  constructor() {
    super(0x7FFFF000, 0x100000, { read: true, write: true, execute: false });
    this.stackPointer = this.baseAddress + this.size; // 스택은 아래로 성장
    this.basePointer = this.stackPointer;
  }

  push(data: Buffer): void {
    this.stackPointer -= data.length;
    if (this.stackPointer < this.baseAddress) {
      throw new Error('Stack overflow');
    }
    data.copy(this.data, this.stackPointer - this.baseAddress);
  }

  pop(size: number): Buffer {
    if (this.stackPointer + size > this.baseAddress + this.size) {
      throw new Error('Stack underflow');
    }
    const data = this.data.slice(
      this.stackPointer - this.baseAddress,
      this.stackPointer - this.baseAddress + size
    );
    this.stackPointer += size;
    return data;
  }

  // 함수 호출 프레임
  pushFrame(frameSize: number): void {
    const oldBasePointer = this.basePointer;
    this.push(Buffer.from([oldBasePointer])); // 이전 base pointer 저장
    this.basePointer = this.stackPointer;
    this.allocate(frameSize); // 지역 변수 공간 할당
  }

  popFrame(): void {
    this.stackPointer = this.basePointer;
    const oldBP = this.pop(8); // base pointer 크기
    this.basePointer = oldBP.readBigInt64LE();
  }

  allocate(size: number): number {
    this.stackPointer -= size;
    if (this.stackPointer < this.baseAddress) {
      throw new Error('Stack overflow');
    }
    return this.stackPointer;
  }
}
```

---

## 🔄 프로세스 상태 전이

### 5-상태 모델

```typescript
class ProcessStateManager {
  private state: ProcessState;
  private transitions: Map<string, ProcessState>;
  private stateHandlers: Map<ProcessState, StateHandler>;

  constructor() {
    this.state = ProcessState.NEW;
    this.initializeTransitions();
    this.initializeHandlers();
  }

  private initializeTransitions(): void {
    this.transitions = new Map([
      // NEW -> READY
      [`${ProcessState.NEW}:admit`, ProcessState.READY],
      
      // READY -> RUNNING
      [`${ProcessState.READY}:dispatch`, ProcessState.RUNNING],
      
      // RUNNING -> READY (preemption)
      [`${ProcessState.RUNNING}:interrupt`, ProcessState.READY],
      
      // RUNNING -> WAITING
      [`${ProcessState.RUNNING}:wait`, ProcessState.WAITING],
      
      // RUNNING -> TERMINATED
      [`${ProcessState.RUNNING}:exit`, ProcessState.TERMINATED],
      
      // WAITING -> READY
      [`${ProcessState.WAITING}:complete`, ProcessState.READY]
    ]);
  }

  private initializeHandlers(): void {
    this.stateHandlers = new Map([
      [ProcessState.NEW, this.handleNew],
      [ProcessState.READY, this.handleReady],
      [ProcessState.RUNNING, this.handleRunning],
      [ProcessState.WAITING, this.handleWaiting],
      [ProcessState.TERMINATED, this.handleTerminated]
    ]);
  }

  // 상태 전이
  transition(event: StateEvent): boolean {
    const key = `${this.state}:${event}`;
    const nextState = this.transitions.get(key);

    if (!nextState) {
      console.log(`Invalid transition: ${key}`);
      return false;
    }

    console.log(`State transition: ${this.state} -> ${nextState} (${event})`);
    
    // 이전 상태 핸들러 종료
    const exitHandler = this.stateHandlers.get(this.state);
    if (exitHandler) {
      exitHandler.exit();
    }

    // 상태 변경
    this.state = nextState;

    // 새 상태 핸들러 시작
    const enterHandler = this.stateHandlers.get(nextState);
    if (enterHandler) {
      enterHandler.enter();
    }

    return true;
  }

  // 상태별 핸들러
  private handleNew: StateHandler = {
    enter: () => {
      console.log('Process created, initializing resources');
    },
    exit: () => {
      console.log('Process initialization complete');
    }
  };

  private handleReady: StateHandler = {
    enter: () => {
      console.log('Process ready for execution');
      // Ready 큐에 추가
    },
    exit: () => {
      console.log('Process leaving ready queue');
    }
  };

  private handleRunning: StateHandler = {
    enter: () => {
      console.log('Process starts execution');
      // CPU 할당
    },
    exit: () => {
      console.log('Process stops execution');
      // CPU 반환
    }
  };

  private handleWaiting: StateHandler = {
    enter: () => {
      console.log('Process waiting for I/O or event');
      // 대기 큐에 추가
    },
    exit: () => {
      console.log('Wait condition satisfied');
    }
  };

  private handleTerminated: StateHandler = {
    enter: () => {
      console.log('Process terminated');
      // 자원 해제
    },
    exit: () => {
      // 종료 상태에서는 다른 상태로 전이 불가
    }
  };

  getCurrentState(): ProcessState {
    return this.state;
  }
}

type StateEvent = 'admit' | 'dispatch' | 'interrupt' | 'wait' | 'complete' | 'exit';

interface StateHandler {
  enter: () => void;
  exit: () => void;
}

// 상태 전이 시뮬레이션
class ProcessLifecycleSimulator {
  private processes: Map<number, ProcessStateManager> = new Map();

  simulateProcessLifecycle(pid: number): void {
    const process = new ProcessStateManager();
    this.processes.set(pid, process);

    console.log(`\n=== Process ${pid} Lifecycle ===`);

    // NEW -> READY
    process.transition('admit');

    // READY -> RUNNING
    process.transition('dispatch');

    // RUNNING -> WAITING (I/O)
    process.transition('wait');

    // WAITING -> READY
    process.transition('complete');

    // READY -> RUNNING
    process.transition('dispatch');

    // RUNNING -> READY (time slice expired)
    process.transition('interrupt');

    // READY -> RUNNING
    process.transition('dispatch');

    // RUNNING -> TERMINATED
    process.transition('exit');
  }
}
```

---

## 📊 프로세스 제어 블록 (PCB)

### PCB 구조

```typescript
class ProcessControlBlock {
  // 프로세스 식별 정보
  readonly pid: number;
  readonly ppid: number; // Parent PID
  readonly uid: number;  // User ID
  readonly gid: number;  // Group ID

  // 프로세스 상태
  state: ProcessState;
  priority: number;
  
  // CPU 레지스터
  programCounter: number;
  registers: Map<string, number>;
  statusRegister: number;

  // 메모리 관리 정보
  memoryLimits: MemoryLimits;
  pageTable: PageTable;
  
  // 스케줄링 정보
  cpuTime: number;
  arrivalTime: number;
  burstTime: number;
  waitingTime: number;
  turnaroundTime: number;

  // I/O 상태
  openFiles: FileDescriptor[];
  ioRequests: IORequest[];

  // 시그널 정보
  signalHandlers: Map<number, SignalHandler>;
  pendingSignals: Set<number>;

  constructor(pid: number, ppid: number, uid: number, gid: number) {
    this.pid = pid;
    this.ppid = ppid;
    this.uid = uid;
    this.gid = gid;

    this.state = ProcessState.NEW;
    this.priority = 0;
    
    this.programCounter = 0;
    this.registers = new Map();
    this.statusRegister = 0;

    this.memoryLimits = {
      stackSize: 8 * 1024 * 1024,  // 8MB
      heapSize: 64 * 1024 * 1024,  // 64MB
      dataSize: 16 * 1024 * 1024   // 16MB
    };
    this.pageTable = new PageTable();

    this.cpuTime = 0;
    this.arrivalTime = Date.now();
    this.burstTime = 0;
    this.waitingTime = 0;
    this.turnaroundTime = 0;

    this.openFiles = [];
    this.ioRequests = [];

    this.signalHandlers = new Map();
    this.pendingSignals = new Set();
  }

  // 컨텍스트 저장
  saveContext(context: CPUContext): void {
    this.programCounter = context.pc;
    this.registers = new Map(context.registers);
    this.statusRegister = context.status;
  }

  // 컨텍스트 복원
  restoreContext(): CPUContext {
    return {
      pc: this.programCounter,
      registers: new Map(this.registers),
      status: this.statusRegister
    };
  }

  // 통계 업데이트
  updateStatistics(cpuBurst: number, waitTime: number): void {
    this.cpuTime += cpuBurst;
    this.waitingTime += waitTime;
    this.turnaroundTime = Date.now() - this.arrivalTime;
  }

  // 파일 디스크립터 관리
  openFile(path: string, mode: string): number {
    const fd = this.getNextFileDescriptor();
    this.openFiles.push({
      fd,
      path,
      mode,
      position: 0
    });
    return fd;
  }

  closeFile(fd: number): void {
    this.openFiles = this.openFiles.filter(f => f.fd !== fd);
  }

  private getNextFileDescriptor(): number {
    const maxFd = Math.max(...this.openFiles.map(f => f.fd), 2);
    return maxFd + 1;
  }

  // 시그널 처리
  sendSignal(signal: number): void {
    this.pendingSignals.add(signal);
  }

  handleSignals(): void {
    for (const signal of this.pendingSignals) {
      const handler = this.signalHandlers.get(signal);
      if (handler) {
        handler(signal);
      } else {
        this.defaultSignalHandler(signal);
      }
      this.pendingSignals.delete(signal);
    }
  }

  private defaultSignalHandler(signal: number): void {
    console.log(`Default handler for signal ${signal}`);
    if (signal === 9) { // SIGKILL
      this.state = ProcessState.TERMINATED;
    }
  }

  // PCB 정보 출력
  toString(): string {
    return `
PCB Information:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PID: ${this.pid}
PPID: ${this.ppid}
State: ${this.state}
Priority: ${this.priority}
CPU Time: ${this.cpuTime}ms
Waiting Time: ${this.waitingTime}ms
Turnaround Time: ${this.turnaroundTime}ms
Open Files: ${this.openFiles.length}
Pending Signals: ${this.pendingSignals.size}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    `;
  }
}

interface MemoryLimits {
  stackSize: number;
  heapSize: number;
  dataSize: number;
}

interface FileDescriptor {
  fd: number;
  path: string;
  mode: string;
  position: number;
}

interface IORequest {
  device: string;
  operation: 'read' | 'write';
  data?: Buffer;
  callback?: () => void;
}

interface CPUContext {
  pc: number;
  registers: Map<string, number>;
  status: number;
}

type SignalHandler = (signal: number) => void;

// 페이지 테이블
class PageTable {
  private entries: Map<number, PageTableEntry> = new Map();

  addEntry(virtualPage: number, physicalFrame: number): void {
    this.entries.set(virtualPage, {
      frameNumber: physicalFrame,
      valid: true,
      dirty: false,
      referenced: false,
      protection: 0b111 // RWX
    });
  }

  translate(virtualAddress: number): number {
    const pageSize = 4096;
    const virtualPage = Math.floor(virtualAddress / pageSize);
    const offset = virtualAddress % pageSize;

    const entry = this.entries.get(virtualPage);
    if (!entry || !entry.valid) {
      throw new Error('Page fault');
    }

    entry.referenced = true;
    return entry.frameNumber * pageSize + offset;
  }
}

interface PageTableEntry {
  frameNumber: number;
  valid: boolean;
  dirty: boolean;
  referenced: boolean;
  protection: number;
}
```

---

## 💡 TypeScript로 구현하는 프로세스

### 완전한 프로세스 관리 시스템

```typescript
class ProcessManager {
  private processes: Map<number, ProcessInstance> = new Map();
  private readyQueue: ProcessInstance[] = [];
  private waitingQueue: ProcessInstance[] = [];
  private runningProcess: ProcessInstance | null = null;
  private nextPid: number = 1;
  private scheduler: Scheduler;

  constructor() {
    this.scheduler = new RoundRobinScheduler(50); // 50ms time quantum
  }

  // 프로세스 생성 (fork)
  fork(parentPid?: number): number {
    const pid = this.nextPid++;
    const ppid = parentPid || 0;
    
    const pcb = new ProcessControlBlock(pid, ppid, 1000, 1000);
    const process = new ProcessInstance(pcb);
    
    this.processes.set(pid, process);
    
    // 부모 프로세스가 있으면 메모리 복사
    if (parentPid && this.processes.has(parentPid)) {
      const parent = this.processes.get(parentPid)!;
      process.copyMemoryFrom(parent);
    }
    
    // Ready 큐에 추가
    this.readyQueue.push(process);
    process.pcb.state = ProcessState.READY;
    
    console.log(`Process ${pid} created (parent: ${ppid})`);
    return pid;
  }

  // 프로그램 실행 (exec)
  exec(pid: number, program: string): void {
    const process = this.processes.get(pid);
    if (!process) {
      throw new Error(`Process ${pid} not found`);
    }

    // 새 프로그램 로드
    process.loadProgram(program);
    console.log(`Process ${pid} executing ${program}`);
  }

  // 프로세스 종료
  exit(pid: number, exitCode: number): void {
    const process = this.processes.get(pid);
    if (!process) return;

    process.pcb.state = ProcessState.TERMINATED;
    
    // 자식 프로세스들을 init(PID 1)에게 입양
    this.adoptOrphans(pid);
    
    // 자원 해제
    process.releaseResources();
    
    // 프로세스 제거
    this.processes.delete(pid);
    
    console.log(`Process ${pid} exited with code ${exitCode}`);
  }

  // 프로세스 대기 (wait)
  wait(parentPid: number): number | null {
    // 자식 프로세스가 종료될 때까지 대기
    const children = this.getChildren(parentPid);
    
    for (const child of children) {
      if (child.pcb.state === ProcessState.TERMINATED) {
        const pid = child.pcb.pid;
        this.processes.delete(pid);
        return pid;
      }
    }
    
    // 대기 상태로 전환
    const parent = this.processes.get(parentPid);
    if (parent) {
      parent.pcb.state = ProcessState.WAITING;
      this.waitingQueue.push(parent);
    }
    
    return null;
  }

  // 스케줄러 실행
  schedule(): void {
    // 현재 프로세스가 있고 시간 할당량이 남아있으면 계속 실행
    if (this.runningProcess && !this.scheduler.shouldPreempt(this.runningProcess)) {
      return;
    }

    // 현재 프로세스를 Ready 큐로
    if (this.runningProcess && this.runningProcess.pcb.state === ProcessState.RUNNING) {
      this.runningProcess.pcb.state = ProcessState.READY;
      this.readyQueue.push(this.runningProcess);
    }

    // 다음 프로세스 선택
    const nextProcess = this.scheduler.selectNext(this.readyQueue);
    if (nextProcess) {
      this.readyQueue = this.readyQueue.filter(p => p !== nextProcess);
      this.dispatch(nextProcess);
    } else {
      this.runningProcess = null;
    }
  }

  private dispatch(process: ProcessInstance): void {
    // 컨텍스트 스위칭
    if (this.runningProcess) {
      const context = this.saveContext();
      this.runningProcess.pcb.saveContext(context);
    }

    this.runningProcess = process;
    process.pcb.state = ProcessState.RUNNING;
    
    const context = process.pcb.restoreContext();
    this.restoreContext(context);
    
    console.log(`Dispatching process ${process.pcb.pid}`);
  }

  private saveContext(): CPUContext {
    // CPU 상태 저장
    return {
      pc: 0x1000, // 예시
      registers: new Map([
        ['AX', 0],
        ['BX', 0],
        ['CX', 0],
        ['DX', 0]
      ]),
      status: 0
    };
  }

  private restoreContext(context: CPUContext): void {
    // CPU 상태 복원
    console.log('Context restored');
  }

  private adoptOrphans(parentPid: number): void {
    const children = this.getChildren(parentPid);
    children.forEach(child => {
      child.pcb.ppid = 1; // init 프로세스에게 입양
    });
  }

  private getChildren(parentPid: number): ProcessInstance[] {
    return Array.from(this.processes.values())
      .filter(p => p.pcb.ppid === parentPid);
  }

  // 프로세스 목록
  listProcesses(): void {
    console.log('\nProcess List:');
    console.log('PID\tPPID\tState\t\tPriority\tCPU Time');
    console.log('━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
    
    for (const process of this.processes.values()) {
      const pcb = process.pcb;
      console.log(
        `${pcb.pid}\t${pcb.ppid}\t${pcb.state}\t${pcb.priority}\t\t${pcb.cpuTime}ms`
      );
    }
  }
}

class ProcessInstance {
  pcb: ProcessControlBlock;
  private memory: ProcessMemory;
  private openFiles: Map<number, FileHandle> = new Map();

  constructor(pcb: ProcessControlBlock) {
    this.pcb = pcb;
    this.memory = new ProcessMemory();
  }

  loadProgram(path: string): void {
    // 프로그램 로드 시뮬레이션
    console.log(`Loading program: ${path}`);
    // 메모리 초기화
    // 코드 세그먼트 로드
    // 진입점 설정
  }

  copyMemoryFrom(parent: ProcessInstance): void {
    // Copy-on-Write 구현
    console.log('Copying memory from parent (CoW)');
  }

  releaseResources(): void {
    // 메모리 해제
    this.memory.release();
    
    // 파일 닫기
    for (const [fd, handle] of this.openFiles) {
      handle.close();
    }
    this.openFiles.clear();
  }
}

interface FileHandle {
  path: string;
  mode: string;
  close(): void;
}

// 스케줄러 인터페이스
interface Scheduler {
  selectNext(readyQueue: ProcessInstance[]): ProcessInstance | null;
  shouldPreempt(current: ProcessInstance): boolean;
}

// Round Robin 스케줄러
class RoundRobinScheduler implements Scheduler {
  private timeQuantum: number;
  private currentQuantum: number = 0;

  constructor(timeQuantum: number) {
    this.timeQuantum = timeQuantum;
  }

  selectNext(readyQueue: ProcessInstance[]): ProcessInstance | null {
    if (readyQueue.length === 0) return null;
    
    this.currentQuantum = 0;
    return readyQueue[0]; // FIFO
  }

  shouldPreempt(current: ProcessInstance): boolean {
    this.currentQuantum++;
    return this.currentQuantum >= this.timeQuantum;
  }
}
```

---

## 🔍 실제 사례

### Linux 프로세스 관리

```typescript
// Linux task_struct 시뮬레이션
interface LinuxTaskStruct {
  // 프로세스 식별
  pid: number;
  tgid: number; // Thread Group ID
  
  // 상태
  state: number;
  flags: number;
  
  // 스케줄링
  prio: number;
  static_prio: number;
  normal_prio: number;
  rt_priority: number;
  
  // 메모리
  mm: MemoryDescriptor;
  active_mm: MemoryDescriptor;
  
  // 파일 시스템
  fs: FileSystemInfo;
  files: FilesStruct;
  
  // 시그널
  signal: SignalStruct;
  sighand: SignalHandler;
  
  // 통계
  utime: bigint;
  stime: bigint;
  start_time: bigint;
}

// /proc 파일 시스템 시뮬레이션
class ProcFileSystem {
  getProcessInfo(pid: number): string {
    return `
/proc/${pid}/status:
Name:   node
Pid:    ${pid}
PPid:   1
State:  S (sleeping)
Threads: 1
VmSize: 123456 kB
VmRSS:  45678 kB

/proc/${pid}/stat:
${pid} (node) S 1 ${pid} 0 0 -1 0 0 0 0 0 0 0 0 0 20 0 1 0 123456 0 0 0 0 0 0

/proc/${pid}/cmdline:
node app.js

/proc/${pid}/environ:
PATH=/usr/bin:/bin
HOME=/home/user
    `;
  }
}

// Windows 프로세스 관리
interface WindowsProcess {
  processId: number;
  parentProcessId: number;
  sessionId: number;
  
  // 핸들
  processHandle: number;
  threadHandle: number;
  
  // 보안
  token: SecurityToken;
  
  // 우선순위
  priorityClass: PriorityClass;
  basePriority: number;
  
  // Job 객체
  jobObject?: JobObject;
}

enum PriorityClass {
  IDLE = 0x40,
  BELOW_NORMAL = 0x4000,
  NORMAL = 0x20,
  ABOVE_NORMAL = 0x8000,
  HIGH = 0x80,
  REALTIME = 0x100
}

interface SecurityToken {
  user: string;
  groups: string[];
  privileges: string[];
}

interface JobObject {
  name: string;
  processes: number[];
  limits: JobLimits;
}

interface JobLimits {
  maxProcesses: number;
  maxMemory: number;
  cpuRateControl: number;
}

interface MemoryDescriptor {
  startCode: number;
  endCode: number;
  startData: number;
  endData: number;
  startBrk: number;
  brk: number;
  startStack: number;
}

interface FileSystemInfo {
  root: string;
  pwd: string;
  umask: number;
}

interface FilesStruct {
  maxFds: number;
  fdTable: Map<number, FileDescriptor>;
}

interface SignalStruct {
  pending: Set<number>;
  blocked: Set<number>;
}
```

> [!tip] 실무 팁
> 프로세스 관리는 운영체제의 핵심 기능입니다. 멀티프로세싱 애플리케이션을 개발할 때는 프로세스 생성 비용, 메모리 사용량, IPC 오버헤드를 고려해야 합니다. 가벼운 동시성이 필요한 경우 스레드나 비동기 프로그래밍을 고려하세요.

## 📚 참고 자료
- Operating System Concepts (Silberschatz)
- The Linux Programming Interface (Kerrisk)
- Windows Internals (Russinovich)
- Advanced Programming in the UNIX Environment (Stevens)