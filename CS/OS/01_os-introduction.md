---
tags:
  - os
  - operating-system
  - computer-science
  - fundamentals
created: 2025-01-08
updated: 2025-01-08
---

# 운영체제 개요

> [!info] 운영체제란?
> 운영체제(Operating System, OS)는 컴퓨터 하드웨어와 사용자 사이의 인터페이스 역할을 하는 시스템 소프트웨어입니다. 하드웨어 자원을 효율적으로 관리하고 응용 프로그램이 실행될 수 있는 환경을 제공합니다.

## 📑 목차

- [[#🎯 운영체제의 목적]]
- [[#🏗️ 운영체제의 구조]]
- [[#⚙️ 운영체제의 주요 기능]]
- [[#📊 운영체제의 종류]]
- [[#💡 TypeScript로 구현하는 OS 개념]]
- [[#🔍 실제 사례]]

---

## 🎯 운영체제의 목적

### 1. 자원 관리 (Resource Management)
```typescript
interface ResourceManager {
  cpu: CPUManager;
  memory: MemoryManager;
  io: IOManager;
  filesystem: FileSystemManager;
}

class OperatingSystem implements ResourceManager {
  cpu: CPUManager;
  memory: MemoryManager;
  io: IOManager;
  filesystem: FileSystemManager;

  constructor() {
    this.cpu = new CPUManager();
    this.memory = new MemoryManager();
    this.io = new IOManager();
    this.filesystem = new FileSystemManager();
  }

  // 자원 할당
  allocateResources(process: Process): ResourceAllocation {
    return {
      cpuTime: this.cpu.allocate(process.priority),
      memorySpace: this.memory.allocate(process.memoryRequirement),
      ioDevices: this.io.allocate(process.ioRequirements)
    };
  }
}
```

### 2. 사용자 인터페이스 제공
- **CLI (Command Line Interface)**: 텍스트 기반 인터페이스
- **GUI (Graphical User Interface)**: 그래픽 기반 인터페이스
- **API (Application Programming Interface)**: 프로그램 인터페이스

### 3. 프로그램 실행 환경
```typescript
class ProcessManager {
  private processes: Map<number, Process> = new Map();
  private nextPid: number = 1;

  // 프로세스 생성
  createProcess(program: Program): Process {
    const pid = this.nextPid++;
    const process = new Process({
      pid,
      program,
      state: ProcessState.NEW,
      priority: program.priority || 0
    });
    
    this.processes.set(pid, process);
    return process;
  }

  // 프로세스 실행
  async executeProcess(pid: number): Promise<void> {
    const process = this.processes.get(pid);
    if (!process) throw new Error(`Process ${pid} not found`);
    
    process.state = ProcessState.RUNNING;
    await process.execute();
    process.state = ProcessState.TERMINATED;
  }
}
```

---

## 🏗️ 운영체제의 구조

### 계층적 구조 (Layered Architecture)

```typescript
enum SystemLayer {
  HARDWARE = 0,
  KERNEL = 1,
  SYSTEM_CALLS = 2,
  UTILITIES = 3,
  APPLICATIONS = 4,
  USER = 5
}

class LayeredOS {
  private layers: Map<SystemLayer, Layer> = new Map();

  constructor() {
    this.initializeLayers();
  }

  private initializeLayers(): void {
    // 하드웨어 계층
    this.layers.set(SystemLayer.HARDWARE, {
      name: 'Hardware',
      components: ['CPU', 'Memory', 'Disk', 'Network'],
      accessLevel: 'privileged'
    });

    // 커널 계층
    this.layers.set(SystemLayer.KERNEL, {
      name: 'Kernel',
      components: ['Process Manager', 'Memory Manager', 'File System', 'Device Drivers'],
      accessLevel: 'kernel'
    });

    // 시스템 콜 계층
    this.layers.set(SystemLayer.SYSTEM_CALLS, {
      name: 'System Calls',
      components: ['fork()', 'exec()', 'open()', 'read()', 'write()'],
      accessLevel: 'system'
    });

    // 유틸리티 계층
    this.layers.set(SystemLayer.UTILITIES, {
      name: 'System Utilities',
      components: ['Shell', 'Compiler', 'Editor', 'Debugger'],
      accessLevel: 'user'
    });
  }

  // 계층 간 통신
  communicate(from: SystemLayer, to: SystemLayer, message: any): any {
    if (Math.abs(from - to) !== 1) {
      throw new Error('Layers can only communicate with adjacent layers');
    }
    // 통신 처리 로직
    return this.processMessage(from, to, message);
  }
}
```

### 커널 모드 vs 사용자 모드

```typescript
enum ExecutionMode {
  KERNEL = 'kernel',
  USER = 'user'
}

class CPUMode {
  private currentMode: ExecutionMode = ExecutionMode.USER;
  private privilegedInstructions = new Set([
    'HALT', 'IO_OPERATION', 'MEMORY_MANAGEMENT', 'INTERRUPT_CONTROL'
  ]);

  // 모드 전환
  switchMode(targetMode: ExecutionMode): void {
    if (targetMode === ExecutionMode.KERNEL) {
      // 인터럽트나 시스템 콜을 통해서만 커널 모드 진입 가능
      this.enterKernelMode();
    } else {
      this.exitKernelMode();
    }
  }

  private enterKernelMode(): void {
    this.currentMode = ExecutionMode.KERNEL;
    console.log('Entered kernel mode - privileged operations allowed');
  }

  private exitKernelMode(): void {
    this.currentMode = ExecutionMode.USER;
    console.log('Exited to user mode - restricted operations only');
  }

  // 명령어 실행
  executeInstruction(instruction: string): void {
    if (this.privilegedInstructions.has(instruction)) {
      if (this.currentMode !== ExecutionMode.KERNEL) {
        throw new Error(`Privileged instruction '${instruction}' requires kernel mode`);
      }
    }
    // 명령어 실행 로직
    console.log(`Executing: ${instruction} in ${this.currentMode} mode`);
  }
}
```

---

## ⚙️ 운영체제의 주요 기능

### 1. 프로세스 관리 (Process Management)

```typescript
enum ProcessState {
  NEW = 'new',
  READY = 'ready',
  RUNNING = 'running',
  WAITING = 'waiting',
  TERMINATED = 'terminated'
}

class Process {
  pid: number;
  state: ProcessState;
  priority: number;
  memoryUsage: number;
  cpuTime: number;
  private context: ProcessContext;

  constructor(config: ProcessConfig) {
    this.pid = config.pid;
    this.state = ProcessState.NEW;
    this.priority = config.priority || 0;
    this.memoryUsage = 0;
    this.cpuTime = 0;
    this.context = new ProcessContext();
  }

  // 컨텍스트 스위칭
  saveContext(): ProcessContext {
    return { ...this.context };
  }

  restoreContext(context: ProcessContext): void {
    this.context = { ...context };
  }
}

interface ProcessContext {
  programCounter: number;
  registers: Map<string, number>;
  stack: any[];
  heap: any[];
}
```

### 2. 메모리 관리 (Memory Management)

```typescript
class MemoryManager {
  private totalMemory: number;
  private freeMemory: number;
  private memoryMap: Map<number, MemoryBlock> = new Map();

  constructor(totalMemory: number) {
    this.totalMemory = totalMemory;
    this.freeMemory = totalMemory;
  }

  // 메모리 할당
  allocate(processId: number, size: number): MemoryBlock | null {
    if (size > this.freeMemory) {
      return null; // 메모리 부족
    }

    const block: MemoryBlock = {
      processId,
      startAddress: this.findFreeBlock(size),
      size,
      allocated: true
    };

    this.memoryMap.set(processId, block);
    this.freeMemory -= size;
    return block;
  }

  // 메모리 해제
  deallocate(processId: number): void {
    const block = this.memoryMap.get(processId);
    if (block) {
      this.freeMemory += block.size;
      this.memoryMap.delete(processId);
    }
  }

  private findFreeBlock(size: number): number {
    // First Fit, Best Fit, Worst Fit 등의 알고리즘 구현
    return 0; // 간단한 예시
  }
}

interface MemoryBlock {
  processId: number;
  startAddress: number;
  size: number;
  allocated: boolean;
}
```

### 3. 파일 시스템 관리

```typescript
class FileSystem {
  private root: Directory;
  private openFiles: Map<number, FileDescriptor> = new Map();
  private nextFd: number = 3; // 0: stdin, 1: stdout, 2: stderr

  constructor() {
    this.root = new Directory('/');
  }

  // 파일 열기
  open(path: string, mode: FileMode): number {
    const file = this.findFile(path);
    if (!file) throw new Error(`File not found: ${path}`);

    const fd = this.nextFd++;
    this.openFiles.set(fd, {
      file,
      mode,
      position: 0
    });
    return fd;
  }

  // 파일 읽기
  read(fd: number, buffer: Buffer, count: number): number {
    const descriptor = this.openFiles.get(fd);
    if (!descriptor) throw new Error(`Invalid file descriptor: ${fd}`);
    
    // 읽기 로직
    const bytesRead = Math.min(count, descriptor.file.size - descriptor.position);
    descriptor.position += bytesRead;
    return bytesRead;
  }

  // 파일 쓰기
  write(fd: number, data: Buffer): number {
    const descriptor = this.openFiles.get(fd);
    if (!descriptor) throw new Error(`Invalid file descriptor: ${fd}`);
    
    if (descriptor.mode === FileMode.READ_ONLY) {
      throw new Error('File is opened in read-only mode');
    }
    
    // 쓰기 로직
    descriptor.file.content = data;
    descriptor.file.size = data.length;
    return data.length;
  }

  // 파일 닫기
  close(fd: number): void {
    this.openFiles.delete(fd);
  }

  private findFile(path: string): File | null {
    // 경로 탐색 로직
    return null; // 간단한 예시
  }
}

enum FileMode {
  READ_ONLY = 'r',
  WRITE_ONLY = 'w',
  READ_WRITE = 'rw'
}

interface FileDescriptor {
  file: File;
  mode: FileMode;
  position: number;
}
```

### 4. 입출력 관리

```typescript
class IOManager {
  private devices: Map<string, IODevice> = new Map();
  private ioQueue: IORequest[] = [];

  // 디바이스 등록
  registerDevice(device: IODevice): void {
    this.devices.set(device.id, device);
  }

  // IO 요청
  async requestIO(request: IORequest): Promise<any> {
    const device = this.devices.get(request.deviceId);
    if (!device) throw new Error(`Device not found: ${request.deviceId}`);

    // IO 스케줄링
    this.ioQueue.push(request);
    return this.processIOQueue();
  }

  private async processIOQueue(): Promise<any> {
    while (this.ioQueue.length > 0) {
      const request = this.ioQueue.shift()!;
      const device = this.devices.get(request.deviceId)!;
      
      // 디바이스가 사용 가능할 때까지 대기
      await this.waitForDevice(device);
      
      // IO 작업 수행
      const result = await device.performIO(request);
      return result;
    }
  }

  private async waitForDevice(device: IODevice): Promise<void> {
    while (device.isBusy()) {
      await new Promise(resolve => setTimeout(resolve, 10));
    }
  }
}

interface IODevice {
  id: string;
  type: 'disk' | 'network' | 'display' | 'keyboard';
  isBusy(): boolean;
  performIO(request: IORequest): Promise<any>;
}

interface IORequest {
  deviceId: string;
  operation: 'read' | 'write';
  data?: any;
  callback?: (result: any) => void;
}
```

---

## 📊 운영체제의 종류

### 1. 단일 작업 운영체제 (Single-tasking OS)
- MS-DOS
- 한 번에 하나의 프로그램만 실행

### 2. 다중 작업 운영체제 (Multi-tasking OS)
```typescript
class MultiTaskingOS {
  private scheduler: Scheduler;
  private processes: Process[] = [];
  private currentProcess: Process | null = null;
  private timeSlice: number = 100; // ms

  constructor() {
    this.scheduler = new RoundRobinScheduler();
  }

  // 타임 슬라이스 기반 스케줄링
  async run(): Promise<void> {
    while (this.processes.length > 0) {
      this.currentProcess = this.scheduler.selectNext(this.processes);
      
      // 프로세스 실행 (타임 슬라이스 동안)
      await this.executeForTimeSlice(this.currentProcess);
      
      // 컨텍스트 스위칭
      this.contextSwitch();
    }
  }

  private async executeForTimeSlice(process: Process): Promise<void> {
    const startTime = Date.now();
    while (Date.now() - startTime < this.timeSlice) {
      // 프로세스 실행
      await process.executeInstruction();
      
      if (process.state === ProcessState.TERMINATED) {
        this.processes = this.processes.filter(p => p.pid !== process.pid);
        break;
      }
    }
  }

  private contextSwitch(): void {
    if (this.currentProcess) {
      // 현재 프로세스의 컨텍스트 저장
      const context = this.currentProcess.saveContext();
      // 다음 프로세스로 전환
      console.log(`Context switch from PID ${this.currentProcess.pid}`);
    }
  }
}
```

### 3. 실시간 운영체제 (Real-time OS)
```typescript
class RealTimeOS {
  private hardDeadlines: Map<number, number> = new Map();
  private softDeadlines: Map<number, number> = new Map();

  // 실시간 태스크 스케줄링
  scheduleRealTimeTask(task: RealTimeTask): void {
    if (task.type === 'hard') {
      // 하드 실시간: 반드시 데드라인 내 완료
      this.scheduleHardRealTime(task);
    } else {
      // 소프트 실시간: 데드라인 초과 허용
      this.scheduleSoftRealTime(task);
    }
  }

  private scheduleHardRealTime(task: RealTimeTask): void {
    // EDF (Earliest Deadline First) 알고리즘
    const deadline = Date.now() + task.deadline;
    this.hardDeadlines.set(task.id, deadline);
    
    // 데드라인이 가장 빠른 태스크 우선 실행
    this.executeByDeadline();
  }

  private scheduleSoftRealTime(task: RealTimeTask): void {
    // Rate Monotonic 스케줄링
    const deadline = Date.now() + task.deadline;
    this.softDeadlines.set(task.id, deadline);
  }

  private executeByDeadline(): void {
    // 데드라인 순으로 정렬하여 실행
    const sorted = Array.from(this.hardDeadlines.entries())
      .sort((a, b) => a[1] - b[1]);
    
    for (const [taskId, deadline] of sorted) {
      const remainingTime = deadline - Date.now();
      if (remainingTime < 0) {
        throw new Error(`Hard deadline missed for task ${taskId}`);
      }
      // 태스크 실행
    }
  }
}

interface RealTimeTask {
  id: number;
  type: 'hard' | 'soft';
  deadline: number; // ms
  priority: number;
  execute(): void;
}
```

### 4. 분산 운영체제 (Distributed OS)
```typescript
class DistributedOS {
  private nodes: Map<string, Node> = new Map();
  private messageQueue: Message[] = [];

  // 노드 추가
  addNode(node: Node): void {
    this.nodes.set(node.id, node);
  }

  // 프로세스 마이그레이션
  async migrateProcess(processId: number, fromNode: string, toNode: string): Promise<void> {
    const source = this.nodes.get(fromNode);
    const target = this.nodes.get(toNode);
    
    if (!source || !target) {
      throw new Error('Invalid node');
    }

    // 프로세스 상태 직렬화
    const processState = await source.serializeProcess(processId);
    
    // 네트워크를 통한 전송
    await this.transferData(fromNode, toNode, processState);
    
    // 대상 노드에서 프로세스 복원
    await target.deserializeProcess(processState);
    
    // 원본 노드에서 프로세스 제거
    source.removeProcess(processId);
  }

  // 분산 동기화
  async distributedLock(resource: string): Promise<DistributedLock> {
    // Lamport 타임스탬프 사용
    const timestamp = this.getLamportTimestamp();
    
    // 모든 노드에 락 요청 전송
    const requests = Array.from(this.nodes.values()).map(node => 
      this.sendLockRequest(node, resource, timestamp)
    );
    
    // 모든 노드로부터 승인 대기
    await Promise.all(requests);
    
    return new DistributedLock(resource, timestamp);
  }

  private getLamportTimestamp(): number {
    // Lamport 논리적 시계 구현
    return Date.now(); // 간단한 예시
  }

  private async transferData(from: string, to: string, data: any): Promise<void> {
    // 네트워크 전송 시뮬레이션
    await new Promise(resolve => setTimeout(resolve, 100));
  }

  private async sendLockRequest(node: Node, resource: string, timestamp: number): Promise<void> {
    // 락 요청 전송
    const message: Message = {
      type: 'LOCK_REQUEST',
      sender: 'coordinator',
      receiver: node.id,
      resource,
      timestamp
    };
    this.messageQueue.push(message);
  }
}

interface Node {
  id: string;
  ipAddress: string;
  resources: Map<string, any>;
  serializeProcess(pid: number): Promise<any>;
  deserializeProcess(state: any): Promise<void>;
  removeProcess(pid: number): void;
}

interface Message {
  type: string;
  sender: string;
  receiver: string;
  [key: string]: any;
}

class DistributedLock {
  constructor(
    public resource: string,
    public timestamp: number
  ) {}

  release(): void {
    // 락 해제 로직
  }
}
```

---

## 💡 TypeScript로 구현하는 OS 개념

### 시스템 콜 인터페이스

```typescript
class SystemCallInterface {
  private kernel: Kernel;

  constructor(kernel: Kernel) {
    this.kernel = kernel;
  }

  // 프로세스 관련 시스템 콜
  fork(): number {
    return this.kernel.createProcess();
  }

  exec(program: string): void {
    this.kernel.executeProgram(program);
  }

  exit(status: number): void {
    this.kernel.terminateProcess(status);
  }

  // 파일 관련 시스템 콜
  open(path: string, flags: number): number {
    return this.kernel.fileSystem.open(path, flags);
  }

  read(fd: number, buffer: Buffer, count: number): number {
    return this.kernel.fileSystem.read(fd, buffer, count);
  }

  write(fd: number, buffer: Buffer, count: number): number {
    return this.kernel.fileSystem.write(fd, buffer, count);
  }

  close(fd: number): void {
    this.kernel.fileSystem.close(fd);
  }

  // 메모리 관련 시스템 콜
  malloc(size: number): number {
    return this.kernel.memoryManager.allocate(size);
  }

  free(address: number): void {
    this.kernel.memoryManager.deallocate(address);
  }
}

// 사용 예시
class Application {
  private syscall: SystemCallInterface;

  constructor(syscall: SystemCallInterface) {
    this.syscall = syscall;
  }

  async run(): Promise<void> {
    // 파일 열기
    const fd = this.syscall.open('/data/input.txt', 0);
    
    // 메모리 할당
    const bufferAddr = this.syscall.malloc(1024);
    const buffer = Buffer.alloc(1024);
    
    // 파일 읽기
    const bytesRead = this.syscall.read(fd, buffer, 1024);
    
    // 처리
    console.log(`Read ${bytesRead} bytes`);
    
    // 정리
    this.syscall.close(fd);
    this.syscall.free(bufferAddr);
    
    // 종료
    this.syscall.exit(0);
  }
}
```

---

## 🔍 실제 사례

### Linux 커널 구조
```typescript
interface LinuxKernel {
  // 프로세스 관리
  scheduler: CFS; // Completely Fair Scheduler
  processManager: TaskStruct;
  
  // 메모리 관리
  virtualMemory: VMM;
  pageCache: PageCache;
  slabAllocator: SLAB;
  
  // 파일 시스템
  vfs: VirtualFileSystem;
  filesystems: Map<string, FileSystem>;
  
  // 네트워킹
  networkStack: TCPIPStack;
  netfilter: Firewall;
  
  // 디바이스 드라이버
  deviceDrivers: Map<string, Driver>;
  udev: DeviceManager;
}
```

### Windows 커널 구조
```typescript
interface WindowsKernel {
  // 실행 계층
  hal: HardwareAbstractionLayer;
  kernel: NTKernel;
  executive: Executive;
  
  // 서브시스템
  win32: Win32Subsystem;
  posix: POSIXSubsystem;
  
  // 관리자
  objectManager: ObjectManager;
  processManager: ProcessManager;
  memoryManager: MemoryManager;
  ioManager: IOManager;
  
  // 보안
  securityManager: SecurityReferenceMonitor;
  lsa: LocalSecurityAuthority;
}
```

> [!tip] 실무 팁
> 운영체제의 개념을 이해하면 애플리케이션 개발 시 성능 최적화와 리소스 관리를 더 효과적으로 할 수 있습니다. 특히 동시성 프로그래밍, 메모리 관리, 파일 시스템 작업 시 OS 레벨의 이해가 중요합니다.

## 📚 참고 자료
- Operating System Concepts (Silberschatz)
- Modern Operating Systems (Tanenbaum)
- Linux Kernel Development (Love)
- Windows Internals (Russinovich)