---
tags:
  - os
  - computer-architecture
  - hardware
  - interrupt
created: 2025-01-08
updated: 2025-01-08
---

# 컴퓨터 구조와 인터럽트

> [!info] 컴퓨터 시스템 구조
> 컴퓨터 시스템은 하드웨어, 운영체제, 응용 프로그램, 사용자의 4계층 구조로 이루어져 있습니다. 각 계층은 상호작용하며 전체 시스템을 구성합니다.

## 📑 목차

- [[#🏗️ 컴퓨터 시스템 구조]]
- [[#⚡ 인터럽트 시스템]]
- [[#🔄 인터럽트 처리 과정]]
- [[#📊 인터럽트 종류]]
- [[#💡 TypeScript로 구현하는 인터럽트]]
- [[#🔍 실제 사례]]

---

## 🏗️ 컴퓨터 시스템 구조

### 계층적 구조

```typescript
class ComputerSystem {
  private layers: SystemLayer[] = [];

  constructor() {
    this.initializeLayers();
  }

  private initializeLayers(): void {
    // 1. 하드웨어 계층
    this.layers.push({
      level: 0,
      name: 'Hardware',
      components: {
        cpu: new CPU(),
        memory: new Memory(),
        storage: new Storage(),
        io: new IODevices()
      }
    });

    // 2. 운영체제 계층
    this.layers.push({
      level: 1,
      name: 'Operating System',
      components: {
        kernel: new Kernel(),
        drivers: new DeviceDrivers(),
        systemServices: new SystemServices()
      }
    });

    // 3. 응용 프로그램 계층
    this.layers.push({
      level: 2,
      name: 'Applications',
      components: {
        systemApps: new SystemApplications(),
        userApps: new UserApplications()
      }
    });

    // 4. 사용자 계층
    this.layers.push({
      level: 3,
      name: 'Users',
      components: {
        interface: new UserInterface(),
        commands: new UserCommands()
      }
    });
  }

  // 계층 간 통신
  communicate(fromLevel: number, toLevel: number, data: any): any {
    if (Math.abs(fromLevel - toLevel) !== 1) {
      throw new Error('Can only communicate with adjacent layers');
    }

    const fromLayer = this.layers[fromLevel];
    const toLayer = this.layers[toLevel];
    
    console.log(`Communication: ${fromLayer.name} -> ${toLayer.name}`);
    return this.processLayerCommunication(fromLayer, toLayer, data);
  }

  private processLayerCommunication(from: SystemLayer, to: SystemLayer, data: any): any {
    // 계층 간 프로토콜 처리
    if (from.level > to.level) {
      // 하위 계층으로의 요청 (시스템 콜)
      return this.handleSystemCall(data);
    } else {
      // 상위 계층으로의 응답 (인터럽트)
      return this.handleInterrupt(data);
    }
  }
}

interface SystemLayer {
  level: number;
  name: string;
  components: Record<string, any>;
}
```

### CPU 구조

```typescript
class CPU {
  private registers: CPURegisters;
  private alu: ArithmeticLogicUnit;
  private controlUnit: ControlUnit;
  private cache: CPUCache;
  private pipeline: InstructionPipeline;

  constructor() {
    this.registers = new CPURegisters();
    this.alu = new ArithmeticLogicUnit();
    this.controlUnit = new ControlUnit();
    this.cache = new CPUCache();
    this.pipeline = new InstructionPipeline();
  }

  // 명령어 실행 사이클
  async executeInstruction(): Promise<void> {
    // 1. Fetch (인출)
    const instruction = await this.fetch();
    
    // 2. Decode (해독)
    const decoded = this.decode(instruction);
    
    // 3. Execute (실행)
    const result = await this.execute(decoded);
    
    // 4. Memory Access (메모리 접근)
    if (decoded.requiresMemoryAccess) {
      await this.memoryAccess(decoded, result);
    }
    
    // 5. Write Back (결과 저장)
    this.writeBack(result);
  }

  private async fetch(): Promise<number> {
    const pc = this.registers.get('PC');
    const instruction = await this.cache.read(pc);
    this.registers.increment('PC');
    return instruction;
  }

  private decode(instruction: number): DecodedInstruction {
    return this.controlUnit.decode(instruction);
  }

  private async execute(decoded: DecodedInstruction): Promise<any> {
    return this.alu.execute(decoded.operation, decoded.operands);
  }

  private async memoryAccess(decoded: DecodedInstruction, data: any): Promise<void> {
    if (decoded.isLoad) {
      const value = await this.cache.read(decoded.address);
      this.registers.set(decoded.targetRegister, value);
    } else if (decoded.isStore) {
      await this.cache.write(decoded.address, data);
    }
  }

  private writeBack(result: any): void {
    if (result.register) {
      this.registers.set(result.register, result.value);
    }
  }
}

class CPURegisters {
  private registers: Map<string, number> = new Map();

  constructor() {
    // 범용 레지스터
    ['AX', 'BX', 'CX', 'DX'].forEach(reg => this.registers.set(reg, 0));
    
    // 특수 목적 레지스터
    this.registers.set('PC', 0);  // Program Counter
    this.registers.set('SP', 0);  // Stack Pointer
    this.registers.set('BP', 0);  // Base Pointer
    this.registers.set('SI', 0);  // Source Index
    this.registers.set('DI', 0);  // Destination Index
    
    // 상태 레지스터
    this.registers.set('FLAGS', 0);  // Status Flags
  }

  get(name: string): number {
    return this.registers.get(name) || 0;
  }

  set(name: string, value: number): void {
    this.registers.set(name, value);
  }

  increment(name: string): void {
    const current = this.get(name);
    this.set(name, current + 1);
  }
}

interface DecodedInstruction {
  operation: string;
  operands: number[];
  requiresMemoryAccess: boolean;
  isLoad: boolean;
  isStore: boolean;
  address?: number;
  targetRegister?: string;
}
```

---

## ⚡ 인터럽트 시스템

### 인터럽트 정의와 목적

```typescript
abstract class Interrupt {
  protected priority: number;
  protected vector: number;
  protected source: string;
  protected timestamp: number;

  constructor(priority: number, vector: number, source: string) {
    this.priority = priority;
    this.vector = vector;
    this.source = source;
    this.timestamp = Date.now();
  }

  abstract handle(): void;
  
  // 인터럽트 우선순위 비교
  comparePriority(other: Interrupt): number {
    return this.priority - other.priority;
  }
}

class InterruptController {
  private interruptQueue: Interrupt[] = [];
  private interruptVectorTable: Map<number, InterruptHandler> = new Map();
  private interruptMask: Set<number> = new Set();
  private servicing: boolean = false;

  constructor() {
    this.initializeVectorTable();
  }

  private initializeVectorTable(): void {
    // 인터럽트 벡터 테이블 초기화
    this.interruptVectorTable.set(0x00, this.handleDivideByZero);
    this.interruptVectorTable.set(0x01, this.handleDebug);
    this.interruptVectorTable.set(0x02, this.handleNMI);
    this.interruptVectorTable.set(0x03, this.handleBreakpoint);
    this.interruptVectorTable.set(0x08, this.handleTimer);
    this.interruptVectorTable.set(0x09, this.handleKeyboard);
    this.interruptVectorTable.set(0x0E, this.handlePageFault);
    this.interruptVectorTable.set(0x20, this.handleSystemCall);
  }

  // 인터럽트 발생
  raiseInterrupt(interrupt: Interrupt): void {
    // 마스크 확인
    if (this.interruptMask.has(interrupt.vector)) {
      console.log(`Interrupt ${interrupt.vector} is masked`);
      return;
    }

    // 우선순위 큐에 추가
    this.insertByPriority(interrupt);
    
    // 인터럽트 처리 시작
    if (!this.servicing) {
      this.serviceInterrupts();
    }
  }

  private insertByPriority(interrupt: Interrupt): void {
    let inserted = false;
    for (let i = 0; i < this.interruptQueue.length; i++) {
      if (interrupt.comparePriority(this.interruptQueue[i]) < 0) {
        this.interruptQueue.splice(i, 0, interrupt);
        inserted = true;
        break;
      }
    }
    if (!inserted) {
      this.interruptQueue.push(interrupt);
    }
  }

  private async serviceInterrupts(): Promise<void> {
    this.servicing = true;

    while (this.interruptQueue.length > 0) {
      const interrupt = this.interruptQueue.shift()!;
      const handler = this.interruptVectorTable.get(interrupt.vector);
      
      if (handler) {
        // 인터럽트 서비스 루틴 실행
        await this.executeISR(interrupt, handler);
      }
    }

    this.servicing = false;
  }

  private async executeISR(interrupt: Interrupt, handler: InterruptHandler): Promise<void> {
    console.log(`Executing ISR for interrupt vector ${interrupt.vector}`);
    
    // 컨텍스트 저장
    const context = this.saveContext();
    
    // 인터럽트 핸들러 실행
    await handler(interrupt);
    
    // 컨텍스트 복원
    this.restoreContext(context);
  }

  private saveContext(): ProcessContext {
    // CPU 상태 저장
    return {
      registers: new Map(),
      programCounter: 0,
      stackPointer: 0,
      flags: 0
    };
  }

  private restoreContext(context: ProcessContext): void {
    // CPU 상태 복원
    console.log('Context restored');
  }

  // 인터럽트 핸들러들
  private handleDivideByZero = async (interrupt: Interrupt): Promise<void> => {
    console.log('Division by zero exception');
    throw new Error('Division by zero');
  };

  private handleDebug = async (interrupt: Interrupt): Promise<void> => {
    console.log('Debug interrupt');
  };

  private handleNMI = async (interrupt: Interrupt): Promise<void> => {
    console.log('Non-Maskable Interrupt');
  };

  private handleBreakpoint = async (interrupt: Interrupt): Promise<void> => {
    console.log('Breakpoint reached');
  };

  private handleTimer = async (interrupt: Interrupt): Promise<void> => {
    console.log('Timer interrupt');
  };

  private handleKeyboard = async (interrupt: Interrupt): Promise<void> => {
    console.log('Keyboard interrupt');
  };

  private handlePageFault = async (interrupt: Interrupt): Promise<void> => {
    console.log('Page fault');
  };

  private handleSystemCall = async (interrupt: Interrupt): Promise<void> => {
    console.log('System call');
  };
}

type InterruptHandler = (interrupt: Interrupt) => Promise<void>;

interface ProcessContext {
  registers: Map<string, number>;
  programCounter: number;
  stackPointer: number;
  flags: number;
}
```

---

## 🔄 인터럽트 처리 과정

### 인터럽트 처리 단계

```typescript
class InterruptProcessor {
  private cpu: CPU;
  private memory: Memory;
  private interruptController: InterruptController;
  private currentProcess: Process | null = null;

  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.interruptController = new InterruptController();
  }

  // 인터럽트 처리 전체 과정
  async processInterrupt(interrupt: Interrupt): Promise<void> {
    // 1. 인터럽트 요청 확인
    if (!this.checkInterruptRequest(interrupt)) {
      return;
    }

    // 2. 현재 명령어 완료
    await this.completeCurrentInstruction();

    // 3. 인터럽트 승인
    if (!this.acknowledgeInterrupt(interrupt)) {
      return;
    }

    // 4. 현재 상태 저장
    const savedState = this.saveProcessState();

    // 5. 인터럽트 벡터 획득
    const vector = this.getInterruptVector(interrupt);

    // 6. ISR 주소 로드
    const isrAddress = this.loadISRAddress(vector);

    // 7. ISR 실행
    await this.executeISR(isrAddress, interrupt);

    // 8. 상태 복원
    this.restoreProcessState(savedState);

    // 9. 인터럽트 종료
    this.endInterrupt();
  }

  private checkInterruptRequest(interrupt: Interrupt): boolean {
    // 인터럽트 가능 상태 확인
    const interruptsEnabled = this.cpu.getInterruptFlag();
    const priority = interrupt.priority;
    const currentPriority = this.getCurrentPriority();

    return interruptsEnabled && priority < currentPriority;
  }

  private async completeCurrentInstruction(): Promise<void> {
    // 현재 실행 중인 명령어 완료
    await this.cpu.completeInstruction();
  }

  private acknowledgeInterrupt(interrupt: Interrupt): boolean {
    // 인터럽트 컨트롤러에 승인 신호
    return this.interruptController.acknowledge(interrupt.vector);
  }

  private saveProcessState(): SavedState {
    return {
      registers: this.cpu.getAllRegisters(),
      programCounter: this.cpu.getProgramCounter(),
      stackPointer: this.cpu.getStackPointer(),
      processStatus: this.cpu.getStatusFlags(),
      processId: this.currentProcess?.pid || 0
    };
  }

  private getInterruptVector(interrupt: Interrupt): number {
    return interrupt.vector;
  }

  private loadISRAddress(vector: number): number {
    // 인터럽트 벡터 테이블에서 ISR 주소 로드
    const ivtBase = 0x0000;
    const isrAddress = this.memory.read(ivtBase + vector * 4);
    return isrAddress;
  }

  private async executeISR(address: number, interrupt: Interrupt): Promise<void> {
    // Program Counter를 ISR 주소로 설정
    this.cpu.setProgramCounter(address);

    // ISR 실행
    console.log(`Executing ISR at address 0x${address.toString(16)}`);
    
    // ISR 코드 실행 시뮬레이션
    await this.simulateISRExecution(interrupt);
  }

  private async simulateISRExecution(interrupt: Interrupt): Promise<void> {
    // ISR 실행 시뮬레이션
    switch (interrupt.vector) {
      case 0x08: // Timer
        await this.handleTimerISR();
        break;
      case 0x09: // Keyboard
        await this.handleKeyboardISR();
        break;
      case 0x0E: // Page Fault
        await this.handlePageFaultISR();
        break;
      default:
        console.log(`Generic ISR for vector ${interrupt.vector}`);
    }
  }

  private async handleTimerISR(): Promise<void> {
    // 타이머 인터럽트 처리
    console.log('Timer ISR: Updating system time and scheduling');
    // 스케줄러 호출
    await this.scheduleNextProcess();
  }

  private async handleKeyboardISR(): Promise<void> {
    // 키보드 인터럽트 처리
    const scanCode = this.readKeyboardBuffer();
    console.log(`Keyboard ISR: Scan code ${scanCode}`);
    // 키 버퍼에 저장
    this.storeInKeyBuffer(scanCode);
  }

  private async handlePageFaultISR(): Promise<void> {
    // 페이지 폴트 처리
    const faultAddress = this.cpu.getCR2(); // 폴트 주소
    console.log(`Page Fault ISR: Address 0x${faultAddress.toString(16)}`);
    // 페이지 로드
    await this.loadPage(faultAddress);
  }

  private restoreProcessState(state: SavedState): void {
    this.cpu.setAllRegisters(state.registers);
    this.cpu.setProgramCounter(state.programCounter);
    this.cpu.setStackPointer(state.stackPointer);
    this.cpu.setStatusFlags(state.processStatus);
  }

  private endInterrupt(): void {
    // EOI (End of Interrupt) 신호 전송
    this.interruptController.sendEOI();
  }

  // Helper methods
  private getCurrentPriority(): number {
    return this.currentProcess?.priority || 999;
  }

  private async scheduleNextProcess(): Promise<void> {
    // 프로세스 스케줄링 로직
  }

  private readKeyboardBuffer(): number {
    // 키보드 버퍼 읽기
    return 0x41; // 'A' key
  }

  private storeInKeyBuffer(scanCode: number): void {
    // 키 버퍼 저장
  }

  private async loadPage(address: number): Promise<void> {
    // 페이지 로드 로직
  }
}

interface SavedState {
  registers: Map<string, number>;
  programCounter: number;
  stackPointer: number;
  processStatus: number;
  processId: number;
}
```

---

## 📊 인터럽트 종류

### 1. 하드웨어 인터럽트

```typescript
class HardwareInterrupt extends Interrupt {
  private device: string;
  private irqLine: number;

  constructor(device: string, irqLine: number, priority: number) {
    super(priority, irqLine, `Hardware:${device}`);
    this.device = device;
    this.irqLine = irqLine;
  }

  handle(): void {
    console.log(`Hardware interrupt from ${this.device} on IRQ ${this.irqLine}`);
  }
}

// 하드웨어 인터럽트 컨트롤러 (8259A PIC 시뮬레이션)
class ProgrammableInterruptController {
  private irqMask: number = 0xFF; // 모든 IRQ 비활성화
  private irqPending: number = 0x00;
  private irqInService: number = 0x00;
  private priority: number[] = [0, 1, 2, 3, 4, 5, 6, 7];

  // IRQ 활성화
  enableIRQ(irq: number): void {
    if (irq >= 0 && irq <= 7) {
      this.irqMask &= ~(1 << irq);
      console.log(`IRQ ${irq} enabled`);
    }
  }

  // IRQ 비활성화
  disableIRQ(irq: number): void {
    if (irq >= 0 && irq <= 7) {
      this.irqMask |= (1 << irq);
      console.log(`IRQ ${irq} disabled`);
    }
  }

  // 인터럽트 요청
  requestInterrupt(irq: number): boolean {
    if (this.irqMask & (1 << irq)) {
      return false; // 마스크됨
    }

    this.irqPending |= (1 << irq);
    return true;
  }

  // 최고 우선순위 인터럽트 획득
  getHighestPriorityInterrupt(): number | null {
    for (const irq of this.priority) {
      if (this.irqPending & (1 << irq)) {
        if (!(this.irqInService & (1 << irq))) {
          return irq;
        }
      }
    }
    return null;
  }

  // 인터럽트 서비스 시작
  beginService(irq: number): void {
    this.irqPending &= ~(1 << irq);
    this.irqInService |= (1 << irq);
  }

  // EOI (End of Interrupt)
  endOfInterrupt(irq: number): void {
    this.irqInService &= ~(1 << irq);
  }
}

// 타이머 인터럽트
class TimerInterrupt extends HardwareInterrupt {
  private interval: number;
  private callback: () => void;

  constructor(interval: number, callback: () => void) {
    super('Timer', 0, 0); // IRQ 0, 최고 우선순위
    this.interval = interval;
    this.callback = callback;
  }

  handle(): void {
    super.handle();
    this.callback();
  }

  start(): NodeJS.Timeout {
    return setInterval(() => {
      this.handle();
    }, this.interval);
  }
}
```

### 2. 소프트웨어 인터럽트

```typescript
class SoftwareInterrupt extends Interrupt {
  private syscallNumber: number;
  private parameters: any[];

  constructor(syscallNumber: number, parameters: any[]) {
    super(5, 0x80, 'Software'); // INT 0x80 (Linux system call)
    this.syscallNumber = syscallNumber;
    this.parameters = parameters;
  }

  handle(): void {
    console.log(`Software interrupt: System call ${this.syscallNumber}`);
    this.executeSystemCall();
  }

  private executeSystemCall(): any {
    // 시스템 콜 디스패처
    switch (this.syscallNumber) {
      case 1: // exit
        return this.sys_exit(this.parameters[0]);
      case 2: // fork
        return this.sys_fork();
      case 3: // read
        return this.sys_read(this.parameters[0], this.parameters[1], this.parameters[2]);
      case 4: // write
        return this.sys_write(this.parameters[0], this.parameters[1], this.parameters[2]);
      case 5: // open
        return this.sys_open(this.parameters[0], this.parameters[1]);
      default:
        throw new Error(`Unknown system call: ${this.syscallNumber}`);
    }
  }

  private sys_exit(status: number): void {
    console.log(`Process exit with status ${status}`);
  }

  private sys_fork(): number {
    console.log('Creating child process');
    return Math.floor(Math.random() * 10000); // PID
  }

  private sys_read(fd: number, buffer: Buffer, count: number): number {
    console.log(`Reading ${count} bytes from fd ${fd}`);
    return count;
  }

  private sys_write(fd: number, buffer: Buffer, count: number): number {
    console.log(`Writing ${count} bytes to fd ${fd}`);
    return count;
  }

  private sys_open(path: string, flags: number): number {
    console.log(`Opening file ${path} with flags ${flags}`);
    return Math.floor(Math.random() * 100); // File descriptor
  }
}

// 예외 (Exception) - 특별한 형태의 소프트웨어 인터럽트
class Exception extends SoftwareInterrupt {
  private type: ExceptionType;
  private errorCode?: number;

  constructor(type: ExceptionType, errorCode?: number) {
    super(0, []);
    this.type = type;
    this.errorCode = errorCode;
    this.vector = this.getExceptionVector(type);
  }

  private getExceptionVector(type: ExceptionType): number {
    const vectorMap: Record<ExceptionType, number> = {
      [ExceptionType.DIVIDE_BY_ZERO]: 0x00,
      [ExceptionType.DEBUG]: 0x01,
      [ExceptionType.BREAKPOINT]: 0x03,
      [ExceptionType.OVERFLOW]: 0x04,
      [ExceptionType.INVALID_OPCODE]: 0x06,
      [ExceptionType.PAGE_FAULT]: 0x0E,
      [ExceptionType.GENERAL_PROTECTION]: 0x0D
    };
    return vectorMap[type];
  }

  handle(): void {
    console.log(`Exception: ${ExceptionType[this.type]}`);
    if (this.errorCode !== undefined) {
      console.log(`Error code: ${this.errorCode}`);
    }
    
    // 예외 처리
    switch (this.type) {
      case ExceptionType.PAGE_FAULT:
        this.handlePageFault();
        break;
      case ExceptionType.DIVIDE_BY_ZERO:
        this.handleDivideByZero();
        break;
      default:
        this.handleGenericException();
    }
  }

  private handlePageFault(): void {
    console.log('Loading page from disk...');
  }

  private handleDivideByZero(): void {
    console.log('Terminating process due to division by zero');
  }

  private handleGenericException(): void {
    console.log('Generic exception handler');
  }
}

enum ExceptionType {
  DIVIDE_BY_ZERO,
  DEBUG,
  BREAKPOINT,
  OVERFLOW,
  INVALID_OPCODE,
  PAGE_FAULT,
  GENERAL_PROTECTION
}
```

---

## 💡 TypeScript로 구현하는 인터럽트

### 완전한 인터럽트 시스템 구현

```typescript
class CompleteInterruptSystem {
  private pic: ProgrammableInterruptController;
  private cpu: CPU;
  private interruptQueue: PriorityQueue<Interrupt>;
  private interruptHandlers: Map<number, InterruptServiceRoutine>;
  private nestingLevel: number = 0;
  private maxNestingLevel: number = 3;

  constructor() {
    this.pic = new ProgrammableInterruptController();
    this.cpu = new CPU();
    this.interruptQueue = new PriorityQueue();
    this.interruptHandlers = new Map();
    this.registerDefaultHandlers();
  }

  private registerDefaultHandlers(): void {
    // 하드웨어 인터럽트 핸들러
    this.registerHandler(0x08, this.timerHandler);
    this.registerHandler(0x09, this.keyboardHandler);
    this.registerHandler(0x0A, this.serialPortHandler);
    this.registerHandler(0x0E, this.diskHandler);

    // 소프트웨어 인터럽트 핸들러
    this.registerHandler(0x80, this.systemCallHandler);

    // 예외 핸들러
    this.registerHandler(0x00, this.divideByZeroHandler);
    this.registerHandler(0x0E, this.pageFaultHandler);
  }

  registerHandler(vector: number, handler: InterruptServiceRoutine): void {
    this.interruptHandlers.set(vector, handler);
  }

  // 인터럽트 발생
  async raiseInterrupt(interrupt: Interrupt): Promise<void> {
    // 중첩 레벨 확인
    if (this.nestingLevel >= this.maxNestingLevel) {
      console.log('Maximum interrupt nesting level reached');
      return;
    }

    // 인터럽트 큐에 추가
    this.interruptQueue.enqueue(interrupt);

    // 인터럽트 처리
    await this.processInterrupts();
  }

  private async processInterrupts(): Promise<void> {
    while (!this.interruptQueue.isEmpty()) {
      const interrupt = this.interruptQueue.dequeue()!;
      
      // 중첩 레벨 증가
      this.nestingLevel++;

      try {
        // 인터럽트 처리
        await this.handleInterrupt(interrupt);
      } finally {
        // 중첩 레벨 감소
        this.nestingLevel--;
      }
    }
  }

  private async handleInterrupt(interrupt: Interrupt): Promise<void> {
    const handler = this.interruptHandlers.get(interrupt.vector);
    
    if (!handler) {
      console.log(`No handler for interrupt vector ${interrupt.vector}`);
      return;
    }

    // 컨텍스트 저장
    const context = this.saveContext();

    // 인터럽트 모드 진입
    this.cpu.enterInterruptMode();

    try {
      // ISR 실행
      await handler(interrupt, context);
    } finally {
      // 인터럽트 모드 종료
      this.cpu.exitInterruptMode();

      // 컨텍스트 복원
      this.restoreContext(context);
    }
  }

  private saveContext(): InterruptContext {
    return {
      registers: this.cpu.getAllRegisters(),
      flags: this.cpu.getFlags(),
      programCounter: this.cpu.getProgramCounter(),
      stackPointer: this.cpu.getStackPointer(),
      timestamp: Date.now()
    };
  }

  private restoreContext(context: InterruptContext): void {
    this.cpu.setAllRegisters(context.registers);
    this.cpu.setFlags(context.flags);
    this.cpu.setProgramCounter(context.programCounter);
    this.cpu.setStackPointer(context.stackPointer);
  }

  // 인터럽트 핸들러들
  private timerHandler: InterruptServiceRoutine = async (interrupt, context) => {
    console.log('Timer interrupt: Updating system time');
    // 시스템 시간 업데이트
    // 프로세스 스케줄링
  };

  private keyboardHandler: InterruptServiceRoutine = async (interrupt, context) => {
    console.log('Keyboard interrupt: Reading key input');
    // 키보드 입력 처리
  };

  private serialPortHandler: InterruptServiceRoutine = async (interrupt, context) => {
    console.log('Serial port interrupt: Data received');
    // 시리얼 데이터 처리
  };

  private diskHandler: InterruptServiceRoutine = async (interrupt, context) => {
    console.log('Disk interrupt: I/O operation complete');
    // 디스크 I/O 완료 처리
  };

  private systemCallHandler: InterruptServiceRoutine = async (interrupt, context) => {
    console.log('System call interrupt');
    // 시스템 콜 처리
  };

  private divideByZeroHandler: InterruptServiceRoutine = async (interrupt, context) => {
    console.log('Divide by zero exception');
    // 프로세스 종료 또는 에러 처리
  };

  private pageFaultHandler: InterruptServiceRoutine = async (interrupt, context) => {
    console.log('Page fault exception');
    // 페이지 로드
  };
}

type InterruptServiceRoutine = (interrupt: Interrupt, context: InterruptContext) => Promise<void>;

interface InterruptContext {
  registers: Map<string, number>;
  flags: number;
  programCounter: number;
  stackPointer: number;
  timestamp: number;
}

// 우선순위 큐 구현
class PriorityQueue<T extends { priority: number }> {
  private items: T[] = [];

  enqueue(item: T): void {
    let added = false;
    for (let i = 0; i < this.items.length; i++) {
      if (item.priority < this.items[i].priority) {
        this.items.splice(i, 0, item);
        added = true;
        break;
      }
    }
    if (!added) {
      this.items.push(item);
    }
  }

  dequeue(): T | undefined {
    return this.items.shift();
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }
}
```

---

## 🔍 실제 사례

### Linux 인터럽트 처리

```typescript
// Linux 스타일 인터럽트 처리
class LinuxInterruptHandler {
  // /proc/interrupts 시뮬레이션
  getInterruptStatistics(): InterruptStatistics {
    return {
      timer: { count: 1234567, cpu: [300000, 300000, 300000, 334567] },
      keyboard: { count: 45678, cpu: [11000, 12000, 11678, 11000] },
      network: { count: 890123, cpu: [220000, 225000, 223123, 222000] },
      disk: { count: 567890, cpu: [140000, 142000, 141890, 144000] }
    };
  }

  // Top half / Bottom half 처리
  async handleInterruptLinuxStyle(irq: number): Promise<void> {
    // Top half: 긴급한 작업만 처리
    await this.topHalf(irq);

    // Bottom half: 지연 가능한 작업은 나중에
    this.scheduleBottomHalf(irq);
  }

  private async topHalf(irq: number): Promise<void> {
    // 최소한의 작업만 수행
    console.log(`Top half: Quick processing for IRQ ${irq}`);
    // 하드웨어 응답
    // 데이터 복사
  }

  private scheduleBottomHalf(irq: number): void {
    // Softirq, Tasklet, Work queue 등으로 처리
    setTimeout(() => {
      this.bottomHalf(irq);
    }, 0);
  }

  private bottomHalf(irq: number): void {
    // 시간이 걸리는 작업 수행
    console.log(`Bottom half: Deferred processing for IRQ ${irq}`);
    // 데이터 처리
    // 프로토콜 처리
  }
}

interface InterruptStatistics {
  [device: string]: {
    count: number;
    cpu: number[];
  };
}

// Windows 인터럽트 처리
class WindowsInterruptHandler {
  // IRQL (Interrupt Request Level)
  private irqlLevels = {
    PASSIVE_LEVEL: 0,
    APC_LEVEL: 1,
    DISPATCH_LEVEL: 2,
    DEVICE_LEVEL: 3,
    HIGH_LEVEL: 31
  };

  // DPC (Deferred Procedure Call)
  private dpcQueue: DeferredProcedureCall[] = [];

  async handleInterruptWindowsStyle(irql: number): Promise<void> {
    // ISR 실행
    await this.executeISR(irql);

    // DPC 큐잉
    if (irql >= this.irqlLevels.DEVICE_LEVEL) {
      this.queueDPC(new DeferredProcedureCall(irql));
    }
  }

  private async executeISR(irql: number): Promise<void> {
    console.log(`ISR executing at IRQL ${irql}`);
    // 최소한의 작업
  }

  private queueDPC(dpc: DeferredProcedureCall): void {
    this.dpcQueue.push(dpc);
    // DISPATCH_LEVEL에서 실행
    this.scheduleDPCExecution();
  }

  private scheduleDPCExecution(): void {
    // DPC 실행
    while (this.dpcQueue.length > 0) {
      const dpc = this.dpcQueue.shift()!;
      dpc.execute();
    }
  }
}

class DeferredProcedureCall {
  constructor(private irql: number) {}

  execute(): void {
    console.log(`DPC executing for IRQL ${this.irql}`);
  }
}
```

> [!tip] 실무 팁
> 인터럽트 처리는 시스템 성능에 큰 영향을 미칩니다. 인터럽트 핸들러는 가능한 한 짧게 유지하고, 시간이 오래 걸리는 작업은 지연 처리(deferred processing)를 사용하세요.

## 📚 참고 자료
- Computer Organization and Design (Patterson & Hennessy)
- Linux Kernel Development (Love)
- Windows Internals (Russinovich)
- Intel 64 and IA-32 Architectures Software Developer's Manual