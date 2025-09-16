---
tags:
  - os
  - memory
  - hierarchy
  - cache
created: 2025-01-08
updated: 2025-01-08
---

# 메모리 계층 구조

> [!info] 메모리 계층 구조란?
> 메모리 계층 구조는 속도와 용량, 비용의 트레이드오프를 최적화하기 위해 여러 레벨의 메모리를 계층적으로 구성한 시스템입니다. 상위 계층은 빠르지만 작고 비싸며, 하위 계층은 느리지만 크고 저렴합니다.

## 📑 목차

- [[#🎯 메모리 계층 구조 개요]]
- [[#⚡ CPU 레지스터]]
- [[#🏗️ 캐시 메모리]]
- [[#💾 주 메모리 (RAM)]]
- [[#💿 보조 저장장치]]
- [[#💡 TypeScript로 구현하는 메모리 계층]]
- [[#🔍 실제 사례]]

---

## 🎯 메모리 계층 구조 개요

### 계층 구조 시뮬레이션

```typescript
class MemoryHierarchy {
  private levels: MemoryLevel[] = [];
  private hitRatios: Map<string, number> = new Map();
  private accessCounts: Map<string, number> = new Map();

  constructor() {
    this.initializeLevels();
  }

  private initializeLevels(): void {
    // 레벨 0: CPU 레지스터
    this.levels.push({
      name: 'Registers',
      size: 1024,  // 1KB
      accessTime: 0.25,  // 0.25ns
      bandwidth: 2000,  // GB/s
      cost: 10000,  // $/KB
      volatile: true
    });

    // 레벨 1: L1 캐시
    this.levels.push({
      name: 'L1 Cache',
      size: 64 * 1024,  // 64KB
      accessTime: 1,  // 1ns
      bandwidth: 1000,  // GB/s
      cost: 100,  // $/KB
      volatile: true
    });

    // 레벨 2: L2 캐시
    this.levels.push({
      name: 'L2 Cache',
      size: 256 * 1024,  // 256KB
      accessTime: 4,  // 4ns
      bandwidth: 500,  // GB/s
      cost: 10,  // $/KB
      volatile: true
    });

    // 레벨 3: L3 캐시
    this.levels.push({
      name: 'L3 Cache',
      size: 8 * 1024 * 1024,  // 8MB
      accessTime: 10,  // 10ns
      bandwidth: 200,  // GB/s
      cost: 1,  // $/KB
      volatile: true
    });

    // 레벨 4: 주 메모리 (RAM)
    this.levels.push({
      name: 'Main Memory (RAM)',
      size: 16 * 1024 * 1024 * 1024,  // 16GB
      accessTime: 100,  // 100ns
      bandwidth: 25,  // GB/s
      cost: 0.01,  // $/KB
      volatile: true
    });

    // 레벨 5: SSD
    this.levels.push({
      name: 'SSD',
      size: 512 * 1024 * 1024 * 1024,  // 512GB
      accessTime: 100000,  // 100μs
      bandwidth: 0.5,  // GB/s
      cost: 0.0001,  // $/KB
      volatile: false
    });

    // 레벨 6: HDD
    this.levels.push({
      name: 'HDD',
      size: 2 * 1024 * 1024 * 1024 * 1024,  // 2TB
      accessTime: 10000000,  // 10ms
      bandwidth: 0.1,  // GB/s
      cost: 0.00001,  // $/KB
      volatile: false
    });
  }

  // 메모리 접근 시뮬레이션
  access(address: number): AccessResult {
    let totalTime = 0;
    let level = 0;

    // 각 레벨에서 데이터 검색
    for (let i = 0; i < this.levels.length; i++) {
      const memLevel = this.levels[i];
      totalTime += memLevel.accessTime;
      
      // 히트 확률 계산 (상위 레벨일수록 히트 확률 낮음)
      const hitProbability = this.calculateHitProbability(i);
      const hit = Math.random() < hitProbability;
      
      this.updateStatistics(memLevel.name, hit);
      
      if (hit) {
        level = i;
        break;
      }
    }

    return {
      level,
      levelName: this.levels[level].name,
      accessTime: totalTime,
      hit: level < this.levels.length - 1
    };
  }

  private calculateHitProbability(level: number): number {
    // 실제 히트 확률 시뮬레이션
    const probabilities = [0.0, 0.95, 0.97, 0.98, 0.99, 0.999, 1.0];
    return probabilities[level];
  }

  private updateStatistics(levelName: string, hit: boolean): void {
    const count = this.accessCounts.get(levelName) || 0;
    const hits = (this.hitRatios.get(levelName) || 0) * count;
    
    this.accessCounts.set(levelName, count + 1);
    this.hitRatios.set(levelName, (hits + (hit ? 1 : 0)) / (count + 1));
  }

  // 평균 접근 시간 계산
  calculateAverageAccessTime(): number {
    let avgTime = 0;
    let cumulativeMissRate = 1;

    for (let i = 0; i < this.levels.length; i++) {
      const level = this.levels[i];
      const hitRate = this.hitRatios.get(level.name) || 0;
      
      avgTime += cumulativeMissRate * level.accessTime;
      cumulativeMissRate *= (1 - hitRate);
    }

    return avgTime;
  }

  // 성능 메트릭 출력
  printStatistics(): void {
    console.log('\n=== Memory Hierarchy Statistics ===');
    console.log('Level\t\t\tSize\t\tAccess Time\tHit Rate');
    console.log('━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
    
    for (const level of this.levels) {
      const hitRate = this.hitRatios.get(level.name) || 0;
      console.log(
        `${level.name.padEnd(20)}\t${this.formatSize(level.size)}\t${level.accessTime}ns\t\t${(hitRate * 100).toFixed(2)}%`
      );
    }
    
    console.log(`\nAverage Access Time: ${this.calculateAverageAccessTime().toFixed(2)}ns`);
  }

  private formatSize(bytes: number): string {
    const units = ['B', 'KB', 'MB', 'GB', 'TB'];
    let size = bytes;
    let unitIndex = 0;
    
    while (size >= 1024 && unitIndex < units.length - 1) {
      size /= 1024;
      unitIndex++;
    }
    
    return `${size.toFixed(0)}${units[unitIndex]}`;
  }
}

interface MemoryLevel {
  name: string;
  size: number;  // bytes
  accessTime: number;  // nanoseconds
  bandwidth: number;  // GB/s
  cost: number;  // $/KB
  volatile: boolean;
}

interface AccessResult {
  level: number;
  levelName: string;
  accessTime: number;
  hit: boolean;
}
```

---

## ⚡ CPU 레지스터

### 레지스터 구현

```typescript
class CPURegisters {
  // 범용 레지스터
  private generalPurpose: Map<string, Register> = new Map();
  
  // 특수 목적 레지스터
  private programCounter: Register;
  private stackPointer: Register;
  private basePointer: Register;
  private instructionRegister: Register;
  private statusRegister: StatusRegister;
  
  // 세그먼트 레지스터
  private segmentRegisters: Map<string, Register> = new Map();

  constructor() {
    this.initializeRegisters();
  }

  private initializeRegisters(): void {
    // x86-64 아키텍처 기준
    // 64비트 범용 레지스터
    const gpRegisters = [
      'RAX', 'RBX', 'RCX', 'RDX',  // 데이터 레지스터
      'RSI', 'RDI',  // 인덱스 레지스터
      'R8', 'R9', 'R10', 'R11', 'R12', 'R13', 'R14', 'R15'  // 추가 레지스터
    ];

    for (const name of gpRegisters) {
      this.generalPurpose.set(name, new Register(name, 64));
    }

    // 특수 목적 레지스터
    this.programCounter = new Register('RIP', 64);  // Instruction Pointer
    this.stackPointer = new Register('RSP', 64);  // Stack Pointer
    this.basePointer = new Register('RBP', 64);  // Base Pointer
    this.instructionRegister = new Register('IR', 64);
    this.statusRegister = new StatusRegister();

    // 세그먼트 레지스터
    const segments = ['CS', 'DS', 'SS', 'ES', 'FS', 'GS'];
    for (const name of segments) {
      this.segmentRegisters.set(name, new Register(name, 16));
    }
  }

  // 레지스터 읽기 (1 사이클)
  read(name: string): bigint {
    const register = this.findRegister(name);
    return register.read();
  }

  // 레지스터 쓰기 (1 사이클)
  write(name: string, value: bigint): void {
    const register = this.findRegister(name);
    register.write(value);
  }

  // 부분 레지스터 접근 (x86 호환성)
  readPartial(name: string): bigint {
    // RAX -> EAX (32비트), AX (16비트), AL (8비트)
    const fullName = this.getFullRegisterName(name);
    const register = this.findRegister(fullName);
    const bits = this.getPartialBits(name);
    
    return register.readPartial(bits);
  }

  private findRegister(name: string): Register {
    return this.generalPurpose.get(name) ||
           this.segmentRegisters.get(name) ||
           (() => {
             switch(name) {
               case 'RIP': return this.programCounter;
               case 'RSP': return this.stackPointer;
               case 'RBP': return this.basePointer;
               case 'IR': return this.instructionRegister;
               default: throw new Error(`Unknown register: ${name}`);
             }
           })();
  }

  private getFullRegisterName(partial: string): string {
    // 부분 레지스터 매핑
    const mapping: Record<string, string> = {
      'EAX': 'RAX', 'AX': 'RAX', 'AL': 'RAX', 'AH': 'RAX',
      'EBX': 'RBX', 'BX': 'RBX', 'BL': 'RBX', 'BH': 'RBX',
      'ECX': 'RCX', 'CX': 'RCX', 'CL': 'RCX', 'CH': 'RCX',
      'EDX': 'RDX', 'DX': 'RDX', 'DL': 'RDX', 'DH': 'RDX'
    };
    return mapping[partial] || partial;
  }

  private getPartialBits(name: string): number {
    if (name.startsWith('E')) return 32;
    if (name.endsWith('X')) return 16;
    if (name.endsWith('L') || name.endsWith('H')) return 8;
    return 64;
  }

  // 레지스터 덤프
  dump(): void {
    console.log('\n=== Register Dump ===');
    console.log('General Purpose:');
    for (const [name, reg] of this.generalPurpose) {
      console.log(`  ${name}: 0x${reg.read().toString(16).padStart(16, '0')}`);
    }
    console.log('\nSpecial Purpose:');
    console.log(`  RIP: 0x${this.programCounter.read().toString(16).padStart(16, '0')}`);
    console.log(`  RSP: 0x${this.stackPointer.read().toString(16).padStart(16, '0')}`);
    console.log(`  RBP: 0x${this.basePointer.read().toString(16).padStart(16, '0')}`);
    console.log(`  FLAGS: ${this.statusRegister.toString()}`);
  }
}

class Register {
  private name: string;
  private bits: number;
  private value: bigint = 0n;

  constructor(name: string, bits: number) {
    this.name = name;
    this.bits = bits;
  }

  read(): bigint {
    return this.value;
  }

  write(value: bigint): void {
    // 비트 마스킹
    const mask = (1n << BigInt(this.bits)) - 1n;
    this.value = value & mask;
  }

  readPartial(bits: number): bigint {
    const mask = (1n << BigInt(bits)) - 1n;
    return this.value & mask;
  }
}

class StatusRegister {
  private flags: Map<string, boolean> = new Map([
    ['CF', false],  // Carry Flag
    ['PF', false],  // Parity Flag
    ['AF', false],  // Auxiliary Flag
    ['ZF', false],  // Zero Flag
    ['SF', false],  // Sign Flag
    ['TF', false],  // Trap Flag
    ['IF', false],  // Interrupt Flag
    ['DF', false],  // Direction Flag
    ['OF', false]   // Overflow Flag
  ]);

  setFlag(flag: string, value: boolean): void {
    this.flags.set(flag, value);
  }

  getFlag(flag: string): boolean {
    return this.flags.get(flag) || false;
  }

  toString(): string {
    const flagStr = Array.from(this.flags.entries())
      .map(([flag, value]) => `${flag}=${value ? 1 : 0}`)
      .join(' ');
    return `[${flagStr}]`;
  }
}
```

---

## 🏗️ 캐시 메모리

### 캐시 구조와 동작

```typescript
class CacheMemory {
  private levels: CacheLevel[] = [];
  private inclusionPolicy: InclusionPolicy;
  private replacementPolicy: ReplacementPolicy;

  constructor() {
    this.inclusionPolicy = InclusionPolicy.INCLUSIVE;
    this.replacementPolicy = ReplacementPolicy.LRU;
    this.initializeCacheLevels();
  }

  private initializeCacheLevels(): void {
    // L1 캐시 (분리형)
    this.levels.push(new CacheLevel({
      name: 'L1-I',  // Instruction Cache
      size: 32 * 1024,  // 32KB
      lineSize: 64,  // 64 bytes
      associativity: 8,  // 8-way
      accessTime: 1
    }));

    this.levels.push(new CacheLevel({
      name: 'L1-D',  // Data Cache
      size: 32 * 1024,  // 32KB
      lineSize: 64,
      associativity: 8,
      accessTime: 1
    }));

    // L2 캐시 (통합형)
    this.levels.push(new CacheLevel({
      name: 'L2',
      size: 256 * 1024,  // 256KB
      lineSize: 64,
      associativity: 8,
      accessTime: 4
    }));

    // L3 캐시 (공유)
    this.levels.push(new CacheLevel({
      name: 'L3',
      size: 8 * 1024 * 1024,  // 8MB
      lineSize: 64,
      associativity: 16,
      accessTime: 10
    }));
  }

  // 캐시 읽기
  read(address: number): CacheAccessResult {
    const startTime = performance.now();
    
    for (const level of this.levels) {
      const result = level.lookup(address);
      
      if (result.hit) {
        const endTime = performance.now();
        return {
          hit: true,
          level: level.name,
          latency: level.accessTime,
          realTime: endTime - startTime
        };
      }
    }

    // 캐시 미스 - 메모리에서 가져오기
    this.handleCacheMiss(address);
    
    const endTime = performance.now();
    return {
      hit: false,
      level: 'Memory',
      latency: 100,  // 메모리 접근 시간
      realTime: endTime - startTime
    };
  }

  private handleCacheMiss(address: number): void {
    // 메모리에서 데이터 가져오기
    const data = this.fetchFromMemory(address);
    
    // 캐시 라인 채우기 (포함 정책에 따라)
    if (this.inclusionPolicy === InclusionPolicy.INCLUSIVE) {
      // 모든 레벨에 캐시 라인 추가
      for (const level of this.levels.reverse()) {
        level.fill(address, data);
      }
    } else if (this.inclusionPolicy === InclusionPolicy.EXCLUSIVE) {
      // L1에만 추가
      this.levels[0].fill(address, data);
    }
  }

  private fetchFromMemory(address: number): any {
    // 메모리 접근 시뮬레이션
    return `Data at ${address}`;
  }

  // 캐시 통계
  getStatistics(): CacheStatistics {
    const stats: CacheStatistics = {
      totalAccesses: 0,
      totalHits: 0,
      totalMisses: 0,
      hitRate: 0,
      missRate: 0,
      averageLatency: 0,
      levelStats: []
    };

    for (const level of this.levels) {
      const levelStat = level.getStatistics();
      stats.levelStats.push(levelStat);
      stats.totalAccesses += levelStat.accesses;
      stats.totalHits += levelStat.hits;
      stats.totalMisses += levelStat.misses;
    }

    stats.hitRate = stats.totalHits / stats.totalAccesses;
    stats.missRate = 1 - stats.hitRate;
    
    return stats;
  }
}

class CacheLevel {
  private config: CacheLevelConfig;
  private sets: CacheSet[];
  private statistics: CacheLevelStatistics;

  constructor(config: CacheLevelConfig) {
    this.config = config;
    this.statistics = {
      accesses: 0,
      hits: 0,
      misses: 0,
      evictions: 0
    };
    
    this.initializeSets();
  }

  private initializeSets(): void {
    const numSets = this.config.size / (this.config.lineSize * this.config.associativity);
    this.sets = [];
    
    for (let i = 0; i < numSets; i++) {
      this.sets.push(new CacheSet(this.config.associativity));
    }
  }

  lookup(address: number): { hit: boolean; data?: any } {
    this.statistics.accesses++;
    
    const { tag, setIndex } = this.parseAddress(address);
    const set = this.sets[setIndex];
    const line = set.find(tag);
    
    if (line) {
      this.statistics.hits++;
      set.updateLRU(tag);
      return { hit: true, data: line.data };
    } else {
      this.statistics.misses++;
      return { hit: false };
    }
  }

  fill(address: number, data: any): void {
    const { tag, setIndex } = this.parseAddress(address);
    const set = this.sets[setIndex];
    
    if (set.isFull()) {
      // 교체 필요
      const evicted = set.evict();
      this.statistics.evictions++;
      
      if (evicted.dirty) {
        // Write-back 필요
        this.writeBack(evicted);
      }
    }
    
    set.insert(tag, data);
  }

  private parseAddress(address: number): { tag: number; setIndex: number; offset: number } {
    const offsetBits = Math.log2(this.config.lineSize);
    const setBits = Math.log2(this.sets.length);
    
    const offset = address & ((1 << offsetBits) - 1);
    const setIndex = (address >> offsetBits) & ((1 << setBits) - 1);
    const tag = address >> (offsetBits + setBits);
    
    return { tag, setIndex, offset };
  }

  private writeBack(line: CacheLine): void {
    console.log(`Writing back dirty line with tag ${line.tag}`);
  }

  getStatistics(): CacheLevelStatistics & { name: string } {
    return {
      name: this.config.name,
      ...this.statistics
    };
  }

  get name(): string {
    return this.config.name;
  }

  get accessTime(): number {
    return this.config.accessTime;
  }
}

class CacheSet {
  private lines: CacheLine[] = [];
  private associativity: number;
  private lruCounter: number = 0;

  constructor(associativity: number) {
    this.associativity = associativity;
  }

  find(tag: number): CacheLine | null {
    return this.lines.find(line => line.tag === tag && line.valid) || null;
  }

  insert(tag: number, data: any): void {
    this.lines.push({
      tag,
      data,
      valid: true,
      dirty: false,
      lruCount: this.lruCounter++
    });
  }

  isFull(): boolean {
    return this.lines.length >= this.associativity;
  }

  evict(): CacheLine {
    // LRU 교체
    let lruIndex = 0;
    let minLRU = this.lines[0].lruCount;
    
    for (let i = 1; i < this.lines.length; i++) {
      if (this.lines[i].lruCount < minLRU) {
        minLRU = this.lines[i].lruCount;
        lruIndex = i;
      }
    }
    
    return this.lines.splice(lruIndex, 1)[0];
  }

  updateLRU(tag: number): void {
    const line = this.lines.find(l => l.tag === tag);
    if (line) {
      line.lruCount = this.lruCounter++;
    }
  }
}

interface CacheLevelConfig {
  name: string;
  size: number;  // bytes
  lineSize: number;  // bytes
  associativity: number;  // n-way
  accessTime: number;  // cycles
}

interface CacheLine {
  tag: number;
  data: any;
  valid: boolean;
  dirty: boolean;
  lruCount: number;
}

interface CacheAccessResult {
  hit: boolean;
  level: string;
  latency: number;
  realTime: number;
}

interface CacheStatistics {
  totalAccesses: number;
  totalHits: number;
  totalMisses: number;
  hitRate: number;
  missRate: number;
  averageLatency: number;
  levelStats: (CacheLevelStatistics & { name: string })[];
}

interface CacheLevelStatistics {
  accesses: number;
  hits: number;
  misses: number;
  evictions: number;
}

enum InclusionPolicy {
  INCLUSIVE = 'INCLUSIVE',
  EXCLUSIVE = 'EXCLUSIVE',
  NINE = 'NINE'  // Non-Inclusive Non-Exclusive
}

enum ReplacementPolicy {
  LRU = 'LRU',
  LFU = 'LFU',
  FIFO = 'FIFO',
  RANDOM = 'RANDOM'
}
```

---

## 💾 주 메모리 (RAM)

### RAM 구현

```typescript
class MainMemory {
  private capacity: number;
  private pageSize: number;
  private pages: Map<number, Page> = new Map();
  private freeFrames: Set<number> = new Set();
  private technology: MemoryTechnology;
  
  constructor(capacity: number, pageSize: number = 4096) {
    this.capacity = capacity;
    this.pageSize = pageSize;
    this.technology = MemoryTechnology.DDR4;
    
    this.initializeFrames();
  }

  private initializeFrames(): void {
    const numFrames = Math.floor(this.capacity / this.pageSize);
    for (let i = 0; i < numFrames; i++) {
      this.freeFrames.add(i);
    }
  }

  // 페이지 할당
  allocatePage(virtualPage: number): number | null {
    if (this.freeFrames.size === 0) {
      return null;  // 메모리 부족
    }

    const frame = this.freeFrames.values().next().value;
    this.freeFrames.delete(frame);
    
    this.pages.set(virtualPage, {
      frameNumber: frame,
      data: Buffer.alloc(this.pageSize),
      dirty: false,
      accessed: false,
      present: true,
      protection: 0b111  // RWX
    });

    return frame;
  }

  // 페이지 해제
  freePage(virtualPage: number): void {
    const page = this.pages.get(virtualPage);
    if (page) {
      this.freeFrames.add(page.frameNumber);
      this.pages.delete(virtualPage);
    }
  }

  // 메모리 읽기
  read(address: number): number {
    const pageNumber = Math.floor(address / this.pageSize);
    const offset = address % this.pageSize;
    
    const page = this.pages.get(pageNumber);
    if (!page || !page.present) {
      throw new Error('Page fault');
    }

    page.accessed = true;
    return page.data[offset];
  }

  // 메모리 쓰기
  write(address: number, value: number): void {
    const pageNumber = Math.floor(address / this.pageSize);
    const offset = address % this.pageSize;
    
    const page = this.pages.get(pageNumber);
    if (!page || !page.present) {
      throw new Error('Page fault');
    }

    if (!(page.protection & 0b010)) {  // Write permission check
      throw new Error('Write protection violation');
    }

    page.data[offset] = value;
    page.dirty = true;
    page.accessed = true;
  }

  // DRAM 특성 시뮬레이션
  simulateDRAMCharacteristics(): DRAMCharacteristics {
    return {
      refreshRate: 64,  // ms
      rowActivationTime: 12.5,  // ns (tRCD)
      columnAccessTime: 12.5,  // ns (tCL)
      prechargeTime: 12.5,  // ns (tRP)
      refreshTime: 7.8,  // μs per row
      powerConsumption: this.calculatePowerConsumption()
    };
  }

  private calculatePowerConsumption(): number {
    const activePower = 2.5;  // Watts
    const idlePower = 0.5;  // Watts
    const utilizationRate = this.getUtilizationRate();
    
    return idlePower + (activePower - idlePower) * utilizationRate;
  }

  private getUtilizationRate(): number {
    const usedFrames = Math.floor(this.capacity / this.pageSize) - this.freeFrames.size;
    const totalFrames = Math.floor(this.capacity / this.pageSize);
    return usedFrames / totalFrames;
  }

  // 메모리 통계
  getStatistics(): MemoryStatistics {
    const totalFrames = Math.floor(this.capacity / this.pageSize);
    const usedFrames = totalFrames - this.freeFrames.size;
    
    return {
      totalCapacity: this.capacity,
      usedMemory: usedFrames * this.pageSize,
      freeMemory: this.freeFrames.size * this.pageSize,
      utilizationRate: usedFrames / totalFrames,
      pageSize: this.pageSize,
      totalPages: this.pages.size,
      dirtyPages: Array.from(this.pages.values()).filter(p => p.dirty).length
    };
  }
}

interface Page {
  frameNumber: number;
  data: Buffer;
  dirty: boolean;
  accessed: boolean;
  present: boolean;
  protection: number;
}

interface DRAMCharacteristics {
  refreshRate: number;
  rowActivationTime: number;
  columnAccessTime: number;
  prechargeTime: number;
  refreshTime: number;
  powerConsumption: number;
}

interface MemoryStatistics {
  totalCapacity: number;
  usedMemory: number;
  freeMemory: number;
  utilizationRate: number;
  pageSize: number;
  totalPages: number;
  dirtyPages: number;
}

enum MemoryTechnology {
  DDR3 = 'DDR3',
  DDR4 = 'DDR4',
  DDR5 = 'DDR5',
  LPDDR4 = 'LPDDR4',
  LPDDR5 = 'LPDDR5'
}

// 가상 메모리 관리
class VirtualMemoryManager {
  private pageTable: Map<number, PageTableEntry> = new Map();
  private tlb: TLB;
  private mainMemory: MainMemory;
  private swapSpace: SwapSpace;

  constructor(memorySize: number, swapSize: number) {
    this.mainMemory = new MainMemory(memorySize);
    this.swapSpace = new SwapSpace(swapSize);
    this.tlb = new TLB(64);  // 64 entries
  }

  // 가상 주소 변환
  translateAddress(virtualAddress: number): number {
    const pageSize = 4096;
    const virtualPage = Math.floor(virtualAddress / pageSize);
    const offset = virtualAddress % pageSize;

    // TLB 확인
    let physicalFrame = this.tlb.lookup(virtualPage);
    
    if (physicalFrame === null) {
      // TLB 미스 - 페이지 테이블 확인
      const pte = this.pageTable.get(virtualPage);
      
      if (!pte || !pte.present) {
        // 페이지 폴트
        this.handlePageFault(virtualPage);
        physicalFrame = this.pageTable.get(virtualPage)!.frameNumber;
      } else {
        physicalFrame = pte.frameNumber;
      }
      
      // TLB 업데이트
      this.tlb.insert(virtualPage, physicalFrame);
    }

    return physicalFrame * pageSize + offset;
  }

  private handlePageFault(virtualPage: number): void {
    console.log(`Page fault for virtual page ${virtualPage}`);
    
    // 스왑 공간에서 페이지 로드
    const pageData = this.swapSpace.readPage(virtualPage);
    
    // 물리 메모리에 페이지 할당
    const frame = this.mainMemory.allocatePage(virtualPage);
    
    if (frame === null) {
      // 메모리 부족 - 페이지 교체 필요
      this.evictPage();
      frame = this.mainMemory.allocatePage(virtualPage);
    }
    
    // 페이지 테이블 업데이트
    this.pageTable.set(virtualPage, {
      frameNumber: frame!,
      present: true,
      dirty: false,
      accessed: false,
      protection: 0b111
    });
  }

  private evictPage(): void {
    // LRU 페이지 교체 알고리즘
    let lruPage: number | null = null;
    let lruTime = Infinity;
    
    for (const [page, entry] of this.pageTable) {
      if (entry.present && entry.accessTime < lruTime) {
        lruTime = entry.accessTime;
        lruPage = page;
      }
    }
    
    if (lruPage !== null) {
      const entry = this.pageTable.get(lruPage)!;
      
      if (entry.dirty) {
        // 더티 페이지는 스왑 공간에 저장
        this.swapSpace.writePage(lruPage, Buffer.alloc(4096));
      }
      
      // 메모리에서 페이지 제거
      this.mainMemory.freePage(lruPage);
      entry.present = false;
    }
  }
}

// TLB (Translation Lookaside Buffer)
class TLB {
  private entries: Map<number, TLBEntry> = new Map();
  private capacity: number;
  private hits: number = 0;
  private misses: number = 0;

  constructor(capacity: number) {
    this.capacity = capacity;
  }

  lookup(virtualPage: number): number | null {
    const entry = this.entries.get(virtualPage);
    
    if (entry) {
      this.hits++;
      entry.lastAccess = Date.now();
      return entry.physicalFrame;
    }
    
    this.misses++;
    return null;
  }

  insert(virtualPage: number, physicalFrame: number): void {
    if (this.entries.size >= this.capacity) {
      this.evict();
    }
    
    this.entries.set(virtualPage, {
      physicalFrame,
      lastAccess: Date.now()
    });
  }

  private evict(): void {
    // LRU 교체
    let lruPage: number | null = null;
    let lruTime = Infinity;
    
    for (const [page, entry] of this.entries) {
      if (entry.lastAccess < lruTime) {
        lruTime = entry.lastAccess;
        lruPage = page;
      }
    }
    
    if (lruPage !== null) {
      this.entries.delete(lruPage);
    }
  }

  getHitRate(): number {
    const total = this.hits + this.misses;
    return total > 0 ? this.hits / total : 0;
  }

  flush(): void {
    this.entries.clear();
  }
}

interface TLBEntry {
  physicalFrame: number;
  lastAccess: number;
}

interface PageTableEntry {
  frameNumber: number;
  present: boolean;
  dirty: boolean;
  accessed: boolean;
  protection: number;
  accessTime?: number;
}

// 스왑 공간
class SwapSpace {
  private capacity: number;
  private pages: Map<number, Buffer> = new Map();

  constructor(capacity: number) {
    this.capacity = capacity;
  }

  writePage(pageNumber: number, data: Buffer): void {
    this.pages.set(pageNumber, data);
  }

  readPage(pageNumber: number): Buffer {
    return this.pages.get(pageNumber) || Buffer.alloc(4096);
  }

  removePage(pageNumber: number): void {
    this.pages.delete(pageNumber);
  }
}
```

---

## 💿 보조 저장장치

### SSD와 HDD 구현

```typescript
// SSD (Solid State Drive)
class SSD {
  private capacity: number;
  private pageSize: number = 4096;  // 4KB
  private blockSize: number = 512 * 1024;  // 512KB
  private blocks: Map<number, SSDBlock> = new Map();
  private wearLeveling: WearLevelingAlgorithm;
  private garbageCollector: GarbageCollector;

  constructor(capacity: number) {
    this.capacity = capacity;
    this.wearLeveling = new WearLevelingAlgorithm();
    this.garbageCollector = new GarbageCollector();
    this.initializeBlocks();
  }

  private initializeBlocks(): void {
    const numBlocks = Math.floor(this.capacity / this.blockSize);
    for (let i = 0; i < numBlocks; i++) {
      this.blocks.set(i, {
        id: i,
        pages: new Map(),
        eraseCount: 0,
        freePages: 128,  // 128 pages per block
        lastModified: Date.now()
      });
    }
  }

  // SSD 읽기 (매우 빠름)
  async read(address: number): Promise<Buffer> {
    const latency = this.calculateReadLatency();
    await this.simulateDelay(latency);
    
    const blockId = Math.floor(address / this.blockSize);
    const pageId = Math.floor((address % this.blockSize) / this.pageSize);
    
    const block = this.blocks.get(blockId);
    if (!block) throw new Error('Invalid address');
    
    const page = block.pages.get(pageId);
    return page || Buffer.alloc(this.pageSize);
  }

  // SSD 쓰기 (복잡한 과정)
  async write(address: number, data: Buffer): Promise<void> {
    const blockId = this.wearLeveling.selectBlock(this.blocks);
    const block = this.blocks.get(blockId)!;
    
    if (block.freePages === 0) {
      // 가비지 컬렉션 필요
      await this.garbageCollector.collect(block);
    }
    
    const pageId = this.findFreePage(block);
    block.pages.set(pageId, data);
    block.freePages--;
    block.lastModified = Date.now();
    
    const latency = this.calculateWriteLatency();
    await this.simulateDelay(latency);
  }

  private findFreePage(block: SSDBlock): number {
    for (let i = 0; i < 128; i++) {
      if (!block.pages.has(i)) {
        return i;
      }
    }
    throw new Error('No free pages in block');
  }

  private calculateReadLatency(): number {
    return 0.1;  // 100μs
  }

  private calculateWriteLatency(): number {
    return 1;  // 1ms (더 느림)
  }

  private async simulateDelay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  // TRIM 명령
  trim(address: number, size: number): void {
    console.log(`TRIM: Marking ${size} bytes at ${address} as deleted`);
    // 삭제된 데이터 마킹
  }

  getStatistics(): SSDStatistics {
    let totalEraseCount = 0;
    let totalFreePages = 0;
    
    for (const block of this.blocks.values()) {
      totalEraseCount += block.eraseCount;
      totalFreePages += block.freePages;
    }
    
    return {
      capacity: this.capacity,
      usedSpace: (this.blocks.size * 128 - totalFreePages) * this.pageSize,
      freeSpace: totalFreePages * this.pageSize,
      averageEraseCount: totalEraseCount / this.blocks.size,
      writeAmplification: this.garbageCollector.getWriteAmplification()
    };
  }
}

interface SSDBlock {
  id: number;
  pages: Map<number, Buffer>;
  eraseCount: number;
  freePages: number;
  lastModified: number;
}

interface SSDStatistics {
  capacity: number;
  usedSpace: number;
  freeSpace: number;
  averageEraseCount: number;
  writeAmplification: number;
}

class WearLevelingAlgorithm {
  selectBlock(blocks: Map<number, SSDBlock>): number {
    // 최소 지우기 횟수를 가진 블록 선택
    let minEraseCount = Infinity;
    let selectedBlock = 0;
    
    for (const [id, block] of blocks) {
      if (block.eraseCount < minEraseCount && block.freePages > 0) {
        minEraseCount = block.eraseCount;
        selectedBlock = id;
      }
    }
    
    return selectedBlock;
  }
}

class GarbageCollector {
  private writeAmplification: number = 1.0;

  async collect(block: SSDBlock): Promise<void> {
    console.log(`Garbage collecting block ${block.id}`);
    
    // 유효한 페이지를 다른 블록으로 이동
    const validPages = Array.from(block.pages.values());
    
    // 블록 지우기
    block.pages.clear();
    block.eraseCount++;
    block.freePages = 128;
    
    // Write Amplification 계산
    this.writeAmplification = (validPages.length + 128) / validPages.length;
  }

  getWriteAmplification(): number {
    return this.writeAmplification;
  }
}

// HDD (Hard Disk Drive)
class HDD {
  private capacity: number;
  private sectorSize: number = 512;  // bytes
  private rpm: number = 7200;
  private platters: Platter[] = [];
  private headPosition: number = 0;

  constructor(capacity: number) {
    this.capacity = capacity;
    this.initializePlatters();
  }

  private initializePlatters(): void {
    const plattersCount = 4;
    const capacityPerPlatter = this.capacity / plattersCount;
    
    for (let i = 0; i < plattersCount; i++) {
      this.platters.push(new Platter(capacityPerPlatter));
    }
  }

  // HDD 읽기 (기계적 지연)
  async read(address: number): Promise<Buffer> {
    const { platter, track, sector } = this.parseAddress(address);
    
    // 탐색 시간
    const seekTime = this.calculateSeekTime(track);
    await this.simulateDelay(seekTime);
    
    // 회전 지연
    const rotationalLatency = this.calculateRotationalLatency();
    await this.simulateDelay(rotationalLatency);
    
    // 데이터 전송
    const transferTime = this.calculateTransferTime(1);
    await this.simulateDelay(transferTime);
    
    return this.platters[platter].read(track, sector);
  }

  // HDD 쓰기
  async write(address: number, data: Buffer): Promise<void> {
    const { platter, track, sector } = this.parseAddress(address);
    
    // 탐색 시간
    const seekTime = this.calculateSeekTime(track);
    await this.simulateDelay(seekTime);
    
    // 회전 지연
    const rotationalLatency = this.calculateRotationalLatency();
    await this.simulateDelay(rotationalLatency);
    
    // 데이터 전송
    const transferTime = this.calculateTransferTime(1);
    await this.simulateDelay(transferTime);
    
    this.platters[platter].write(track, sector, data);
  }

  private parseAddress(address: number): { platter: number; track: number; sector: number } {
    const sectorsPerTrack = 63;
    const tracksPerPlatter = 16383;
    
    const sector = address % sectorsPerTrack;
    const track = Math.floor(address / sectorsPerTrack) % tracksPerPlatter;
    const platter = Math.floor(address / (sectorsPerTrack * tracksPerPlatter));
    
    return { platter, track, sector };
  }

  private calculateSeekTime(targetTrack: number): number {
    const distance = Math.abs(targetTrack - this.headPosition);
    this.headPosition = targetTrack;
    
    // 평균 탐색 시간: 8.5ms
    return Math.min(distance * 0.01, 15);  // 최대 15ms
  }

  private calculateRotationalLatency(): number {
    // 평균 회전 지연: (60 / RPM / 2) * 1000 ms
    return (60 / this.rpm / 2) * 1000;  // 약 4.17ms for 7200 RPM
  }

  private calculateTransferTime(sectors: number): number {
    // 전송 속도: ~150MB/s
    const bytesPerSecond = 150 * 1024 * 1024;
    const bytes = sectors * this.sectorSize;
    return (bytes / bytesPerSecond) * 1000;  // ms
  }

  private async simulateDelay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  // 조각 모음
  async defragment(): Promise<void> {
    console.log('Starting defragmentation...');
    // 조각난 데이터를 연속적으로 재배치
    await this.simulateDelay(60000);  // 1분 시뮬레이션
    console.log('Defragmentation complete');
  }
}

class Platter {
  private capacity: number;
  private tracks: Map<number, Track> = new Map();

  constructor(capacity: number) {
    this.capacity = capacity;
  }

  read(trackNumber: number, sectorNumber: number): Buffer {
    const track = this.tracks.get(trackNumber);
    if (!track) return Buffer.alloc(512);
    return track.readSector(sectorNumber);
  }

  write(trackNumber: number, sectorNumber: number, data: Buffer): void {
    if (!this.tracks.has(trackNumber)) {
      this.tracks.set(trackNumber, new Track());
    }
    this.tracks.get(trackNumber)!.writeSector(sectorNumber, data);
  }
}

class Track {
  private sectors: Map<number, Buffer> = new Map();

  readSector(sectorNumber: number): Buffer {
    return this.sectors.get(sectorNumber) || Buffer.alloc(512);
  }

  writeSector(sectorNumber: number, data: Buffer): void {
    this.sectors.set(sectorNumber, data);
  }
}
```

---

## 💡 TypeScript로 구현하는 메모리 계층

### 통합 메모리 계층 시스템

```typescript
class IntegratedMemorySystem {
  private registers: CPURegisters;
  private cache: CacheMemory;
  private mainMemory: MainMemory;
  private virtualMemory: VirtualMemoryManager;
  private ssd: SSD;
  private hdd: HDD;
  private statistics: MemoryAccessStatistics;

  constructor() {
    this.registers = new CPURegisters();
    this.cache = new CacheMemory();
    this.mainMemory = new MainMemory(16 * 1024 * 1024 * 1024);  // 16GB
    this.virtualMemory = new VirtualMemoryManager(
      16 * 1024 * 1024 * 1024,  // 16GB RAM
      32 * 1024 * 1024 * 1024   // 32GB Swap
    );
    this.ssd = new SSD(512 * 1024 * 1024 * 1024);  // 512GB
    this.hdd = new HDD(2 * 1024 * 1024 * 1024 * 1024);  // 2TB
    this.statistics = {
      totalAccesses: 0,
      registerHits: 0,
      cacheHits: 0,
      memoryHits: 0,
      pageFiles: 0,
      swapOuts: 0
    };
  }

  // 통합 메모리 접근
  async access(virtualAddress: number): Promise<any> {
    this.statistics.totalAccesses++;
    const startTime = performance.now();
    
    // 1. 레지스터 확인 (가장 빠름)
    // 실제로는 컴파일러/CPU가 관리
    
    // 2. 캐시 확인
    const cacheResult = this.cache.read(virtualAddress);
    if (cacheResult.hit) {
      this.statistics.cacheHits++;
      return cacheResult;
    }
    
    // 3. 주소 변환 및 메모리 접근
    try {
      const physicalAddress = this.virtualMemory.translateAddress(virtualAddress);
      const data = this.mainMemory.read(physicalAddress);
      this.statistics.memoryHits++;
      return data;
    } catch (error) {
      // 페이지 폴트 처리
      this.statistics.pageFiles++;
      await this.handlePageFault(virtualAddress);
      return this.access(virtualAddress);  // 재시도
    }
  }

  private async handlePageFault(virtualAddress: number): Promise<void> {
    console.log(`Handling page fault for address ${virtualAddress.toString(16)}`);
    
    // SSD 스왑 공간에서 먼저 확인
    const pageData = await this.ssd.read(virtualAddress);
    
    // 메모리에 로드
    // 실제 구현...
  }

  // 성능 분석
  analyzePerformance(): PerformanceAnalysis {
    const hitRate = (this.statistics.cacheHits + this.statistics.memoryHits) / 
                   this.statistics.totalAccesses;
    
    const avgAccessTime = this.calculateAverageAccessTime();
    
    return {
      hitRate,
      missRate: 1 - hitRate,
      avgAccessTime,
      pageFaultRate: this.statistics.pageFiles / this.statistics.totalAccesses,
      swapRate: this.statistics.swapOuts / this.statistics.totalAccesses
    };
  }

  private calculateAverageAccessTime(): number {
    // 가중 평균 계산
    const weights = {
      register: 0.01,
      cache: 0.7,
      memory: 0.25,
      disk: 0.04
    };
    
    const times = {
      register: 0.25,
      cache: 4,
      memory: 100,
      disk: 10000000
    };
    
    return Object.entries(weights).reduce((sum, [level, weight]) => {
      return sum + weight * times[level as keyof typeof times];
    }, 0);
  }
}

interface MemoryAccessStatistics {
  totalAccesses: number;
  registerHits: number;
  cacheHits: number;
  memoryHits: number;
  pageFiles: number;
  swapOuts: number;
}

interface PerformanceAnalysis {
  hitRate: number;
  missRate: number;
  avgAccessTime: number;
  pageFaultRate: number;
  swapRate: number;
}
```

---

## 🔍 실제 사례

### Intel 메모리 계층 구조

```typescript
// Intel Core i9 메모리 계층 시뮬레이션
class IntelCoreI9Memory {
  getSpecifications(): any {
    return {
      registers: {
        general: 16,  // 16개의 64비트 범용 레지스터
        vector: 32,   // 32개의 512비트 AVX-512 레지스터
        accessTime: '< 1 cycle'
      },
      l1Cache: {
        instruction: '32KB per core',
        data: '32KB per core',
        associativity: '8-way',
        lineSize: '64 bytes',
        accessTime: '4-5 cycles'
      },
      l2Cache: {
        unified: '256KB per core',
        associativity: '4-way',
        lineSize: '64 bytes',
        accessTime: '12 cycles'
      },
      l3Cache: {
        shared: '16-24MB',
        associativity: '16-way',
        lineSize: '64 bytes',
        accessTime: '42 cycles'
      },
      mainMemory: {
        type: 'DDR4-3200',
        channels: 2,
        maxCapacity: '128GB',
        bandwidth: '51.2 GB/s',
        accessTime: '~100ns'
      }
    };
  }
}

// ARM 메모리 계층 구조
class ARMCortexA78Memory {
  getSpecifications(): any {
    return {
      registers: {
        general: 31,  // 31개의 64비트 범용 레지스터
        vector: 32,   // 32개의 128비트 NEON 레지스터
        accessTime: '< 1 cycle'
      },
      l1Cache: {
        instruction: '32-64KB per core',
        data: '32-64KB per core',
        associativity: '4-way',
        accessTime: '3 cycles'
      },
      l2Cache: {
        private: '256-512KB per core',
        associativity: '8-way',
        accessTime: '9 cycles'
      },
      l3Cache: {
        shared: '2-4MB',
        associativity: '16-way',
        accessTime: '26-31 cycles'
      }
    };
  }
}
```

> [!tip] 실무 팁
> 메모리 계층 구조를 이해하면 캐시 친화적인 코드를 작성할 수 있습니다. 데이터 지역성(spatial/temporal locality)을 활용하고, 캐시 라인 크기를 고려한 데이터 구조 설계, false sharing 방지 등이 중요합니다.

## 📚 참고 자료
- Computer Architecture: A Quantitative Approach (Hennessy & Patterson)
- What Every Programmer Should Know About Memory (Ulrich Drepper)
- Intel 64 and IA-32 Architectures Optimization Reference Manual
- ARM Cortex-A Series Programmer's Guide