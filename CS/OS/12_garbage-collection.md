---
tags:
  - 운영체제
  - OS
  - 가비지컬렉션
  - 메모리관리
  - GC
  - CTO강의
created: 2025-01-12
updated: 2025-01-12
aliases:
  - 가비지컬렉션
  - Garbage Collection
  - 메모리관리
description: 가비지 컬렉션 알고리즘, 메모리 관리 기법, 다양한 GC 전략의 TypeScript 구현
status: published
category: guide
---

# 가비지 컬렉션 (Garbage Collection) - 자동 메모리 관리

> [!info] 개요
> 가비지 컬렉션은 프로그램이 동적으로 할당한 메모리 중 더 이상 사용되지 않는 메모리를 자동으로 해제하는 메모리 관리 기법입니다. Reference Counting, Mark & Sweep, Generational GC 등 다양한 알고리즘을 다룹니다.

## 📑 목차

- [[#🎯 가비지 컬렉션 개요]]
- [[#📊 Reference Counting]]
- [[#🔍 Mark & Sweep]]
- [[#🚀 Generational GC]]
- [[#💡 Concurrent & Incremental GC]]
- [[#⚡ 고급 GC 알고리즘]]
- [[#💻 TypeScript 구현]]
- [[#📈 성능 분석]]
- [[#🏢 실제 시스템 사례]]
- [[#📚 참고자료]]

---

## 🎯 가비지 컬렉션 개요

### 메모리 관리 방식 비교

> [!note] 수동 vs 자동 메모리 관리
> - **수동 관리**: malloc/free, new/delete - 프로그래머가 직접 관리
> - **자동 관리**: 가비지 컬렉터가 자동으로 불필요한 메모리 해제

```typescript
// 메모리 관리 시뮬레이션
class MemoryManager {
    private heap: Map<number, MemoryBlock> = new Map();
    private nextAddress: number = 0x1000;
    private totalAllocated: number = 0;
    private totalFreed: number = 0;
    
    // 수동 메모리 관리
    malloc(size: number): number {
        const address = this.nextAddress;
        const block: MemoryBlock = {
            address,
            size,
            data: new Array(size).fill(0),
            allocated: true,
            timestamp: Date.now()
        };
        
        this.heap.set(address, block);
        this.nextAddress += size;
        this.totalAllocated += size;
        
        console.log(`Allocated ${size} bytes at address 0x${address.toString(16)}`);
        return address;
    }
    
    free(address: number): void {
        const block = this.heap.get(address);
        if (!block) {
            throw new Error(`Invalid address: 0x${address.toString(16)}`);
        }
        
        if (!block.allocated) {
            throw new Error(`Double free detected at address 0x${address.toString(16)}`);
        }
        
        block.allocated = false;
        this.totalFreed += block.size;
        
        console.log(`Freed ${block.size} bytes at address 0x${address.toString(16)}`);
    }
    
    // 메모리 누수 감지
    detectLeaks(): MemoryLeak[] {
        const leaks: MemoryLeak[] = [];
        const now = Date.now();
        
        for (const [address, block] of this.heap) {
            if (block.allocated && (now - block.timestamp) > 60000) { // 1분 이상
                leaks.push({
                    address,
                    size: block.size,
                    age: now - block.timestamp
                });
            }
        }
        
        return leaks;
    }
    
    // 메모리 통계
    getStatistics(): MemoryStatistics {
        let currentlyAllocated = 0;
        let fragmentedSpace = 0;
        
        for (const block of this.heap.values()) {
            if (block.allocated) {
                currentlyAllocated += block.size;
            } else {
                fragmentedSpace += block.size;
            }
        }
        
        return {
            totalAllocated: this.totalAllocated,
            totalFreed: this.totalFreed,
            currentlyAllocated,
            fragmentedSpace,
            leakCount: this.detectLeaks().length
        };
    }
}

interface MemoryBlock {
    address: number;
    size: number;
    data: any[];
    allocated: boolean;
    timestamp: number;
}

interface MemoryLeak {
    address: number;
    size: number;
    age: number;
}

interface MemoryStatistics {
    totalAllocated: number;
    totalFreed: number;
    currentlyAllocated: number;
    fragmentedSpace: number;
    leakCount: number;
}
```

### 가비지 컬렉션의 기본 개념

```typescript
// 객체 그래프와 도달 가능성
class ObjectGraph {
    private objects: Map<string, GCObject> = new Map();
    private roots: Set<string> = new Set();
    
    // 객체 생성
    createObject(id: string, size: number): void {
        const obj: GCObject = {
            id,
            size,
            references: new Set(),
            marked: false,
            generation: 0,
            lastAccessed: Date.now()
        };
        
        this.objects.set(id, obj);
        console.log(`Created object ${id} (${size} bytes)`);
    }
    
    // 루트 설정 (전역 변수, 스택 변수 등)
    addRoot(objectId: string): void {
        if (!this.objects.has(objectId)) {
            throw new Error(`Object ${objectId} not found`);
        }
        
        this.roots.add(objectId);
        console.log(`Added root: ${objectId}`);
    }
    
    removeRoot(objectId: string): void {
        this.roots.delete(objectId);
        console.log(`Removed root: ${objectId}`);
    }
    
    // 참조 추가
    addReference(fromId: string, toId: string): void {
        const fromObj = this.objects.get(fromId);
        const toObj = this.objects.get(toId);
        
        if (!fromObj || !toObj) {
            throw new Error('Invalid object reference');
        }
        
        fromObj.references.add(toId);
        console.log(`Added reference: ${fromId} -> ${toId}`);
    }
    
    // 참조 제거
    removeReference(fromId: string, toId: string): void {
        const fromObj = this.objects.get(fromId);
        if (fromObj) {
            fromObj.references.delete(toId);
            console.log(`Removed reference: ${fromId} -> ${toId}`);
        }
    }
    
    // 도달 가능한 객체 찾기
    findReachableObjects(): Set<string> {
        const reachable = new Set<string>();
        const stack = Array.from(this.roots);
        
        while (stack.length > 0) {
            const objectId = stack.pop()!;
            
            if (reachable.has(objectId)) continue;
            
            reachable.add(objectId);
            
            const obj = this.objects.get(objectId);
            if (obj) {
                for (const refId of obj.references) {
                    if (!reachable.has(refId)) {
                        stack.push(refId);
                    }
                }
            }
        }
        
        return reachable;
    }
    
    // 가비지 식별
    identifyGarbage(): string[] {
        const reachable = this.findReachableObjects();
        const garbage: string[] = [];
        
        for (const [id] of this.objects) {
            if (!reachable.has(id)) {
                garbage.push(id);
            }
        }
        
        return garbage;
    }
    
    // 객체 그래프 시각화 정보
    getGraphInfo(): any {
        const nodes = Array.from(this.objects.entries()).map(([id, obj]) => ({
            id,
            size: obj.size,
            isRoot: this.roots.has(id),
            referenceCount: obj.references.size
        }));
        
        const edges = [];
        for (const [fromId, obj] of this.objects) {
            for (const toId of obj.references) {
                edges.push({ from: fromId, to: toId });
            }
        }
        
        return { nodes, edges };
    }
}

interface GCObject {
    id: string;
    size: number;
    references: Set<string>;
    marked: boolean;
    generation: number;
    lastAccessed: number;
}
```

---

## 📊 Reference Counting

### 기본 Reference Counting

```typescript
class ReferenceCountingGC {
    private objects: Map<string, RCObject> = new Map();
    private totalMemory: number = 0;
    private freedMemory: number = 0;
    
    // 객체 생성
    allocate(id: string, size: number): void {
        const obj: RCObject = {
            id,
            size,
            referenceCount: 1, // 생성 시 참조 카운트 1
            references: new Set(),
            isRoot: false
        };
        
        this.objects.set(id, obj);
        this.totalMemory += size;
        
        console.log(`Allocated object ${id} (${size} bytes), refCount: 1`);
    }
    
    // 참조 추가
    addReference(fromId: string | null, toId: string): void {
        const toObj = this.objects.get(toId);
        if (!toObj) {
            throw new Error(`Object ${toId} not found`);
        }
        
        toObj.referenceCount++;
        
        if (fromId) {
            const fromObj = this.objects.get(fromId);
            if (fromObj) {
                fromObj.references.add(toId);
            }
        } else {
            // 루트에서의 참조
            toObj.isRoot = true;
        }
        
        console.log(`Reference added to ${toId}, refCount: ${toObj.referenceCount}`);
    }
    
    // 참조 제거
    removeReference(fromId: string | null, toId: string): void {
        const toObj = this.objects.get(toId);
        if (!toObj) return;
        
        toObj.referenceCount--;
        
        if (fromId) {
            const fromObj = this.objects.get(fromId);
            if (fromObj) {
                fromObj.references.delete(toId);
            }
        } else {
            toObj.isRoot = false;
        }
        
        console.log(`Reference removed from ${toId}, refCount: ${toObj.referenceCount}`);
        
        // 참조 카운트가 0이 되면 즉시 해제
        if (toObj.referenceCount === 0) {
            this.deallocate(toId);
        }
    }
    
    // 객체 해제
    private deallocate(id: string): void {
        const obj = this.objects.get(id);
        if (!obj) return;
        
        console.log(`Deallocating object ${id} (${obj.size} bytes)`);
        
        // 이 객체가 참조하는 다른 객체들의 참조 카운트 감소
        for (const refId of obj.references) {
            this.removeReference(id, refId);
        }
        
        this.freedMemory += obj.size;
        this.objects.delete(id);
    }
    
    // 순환 참조 감지 (Reference Counting의 한계)
    detectCycles(): string[][] {
        const cycles: string[][] = [];
        const visited = new Set<string>();
        const recursionStack = new Set<string>();
        
        for (const [id] of this.objects) {
            if (!visited.has(id)) {
                const cycle = this.dfsDetectCycle(id, visited, recursionStack, []);
                if (cycle.length > 0) {
                    cycles.push(cycle);
                }
            }
        }
        
        return cycles;
    }
    
    private dfsDetectCycle(
        nodeId: string,
        visited: Set<string>,
        recursionStack: Set<string>,
        path: string[]
    ): string[] {
        visited.add(nodeId);
        recursionStack.add(nodeId);
        path.push(nodeId);
        
        const obj = this.objects.get(nodeId);
        if (obj) {
            for (const neighborId of obj.references) {
                if (!visited.has(neighborId)) {
                    const cycle = this.dfsDetectCycle(neighborId, visited, recursionStack, [...path]);
                    if (cycle.length > 0) return cycle;
                } else if (recursionStack.has(neighborId)) {
                    // 순환 발견
                    const cycleStart = path.indexOf(neighborId);
                    return path.slice(cycleStart);
                }
            }
        }
        
        recursionStack.delete(nodeId);
        return [];
    }
    
    // 통계
    getStatistics(): any {
        return {
            objectCount: this.objects.size,
            totalMemory: this.totalMemory,
            freedMemory: this.freedMemory,
            currentMemory: this.totalMemory - this.freedMemory,
            cycles: this.detectCycles()
        };
    }
}

interface RCObject {
    id: string;
    size: number;
    referenceCount: number;
    references: Set<string>;
    isRoot: boolean;
}
```

### Weighted Reference Counting

```typescript
class WeightedReferenceCountingGC {
    private objects: Map<string, WRCObject> = new Map();
    
    allocate(id: string, size: number): void {
        const totalWeight = 1 << 16; // 초기 가중치 (2^16)
        
        const obj: WRCObject = {
            id,
            size,
            weight: totalWeight,
            referenceWeight: totalWeight,
            references: new Map()
        };
        
        this.objects.set(id, obj);
        console.log(`Allocated ${id} with weight ${totalWeight}`);
    }
    
    addReference(fromId: string, toId: string): void {
        const fromObj = this.objects.get(fromId);
        const toObj = this.objects.get(toId);
        
        if (!fromObj || !toObj) {
            throw new Error('Invalid reference');
        }
        
        // 참조 가중치를 반으로 나눔
        const referenceWeight = Math.floor(toObj.referenceWeight / 2);
        toObj.referenceWeight -= referenceWeight;
        
        fromObj.references.set(toId, referenceWeight);
        
        console.log(`Added reference ${fromId} -> ${toId} with weight ${referenceWeight}`);
    }
    
    removeReference(fromId: string, toId: string): void {
        const fromObj = this.objects.get(fromId);
        const toObj = this.objects.get(toId);
        
        if (!fromObj || !toObj) return;
        
        const referenceWeight = fromObj.references.get(toId);
        if (referenceWeight !== undefined) {
            toObj.referenceWeight += referenceWeight;
            fromObj.references.delete(toId);
            
            console.log(`Removed reference ${fromId} -> ${toId}, weight restored: ${referenceWeight}`);
            
            // 모든 가중치가 복원되면 객체 해제
            if (toObj.referenceWeight === toObj.weight) {
                this.deallocate(toId);
            }
        }
    }
    
    private deallocate(id: string): void {
        const obj = this.objects.get(id);
        if (!obj) return;
        
        console.log(`Deallocating ${id}`);
        
        // 참조하는 객체들의 가중치 복원
        for (const [refId, weight] of obj.references) {
            const refObj = this.objects.get(refId);
            if (refObj) {
                refObj.referenceWeight += weight;
                if (refObj.referenceWeight === refObj.weight) {
                    this.deallocate(refId);
                }
            }
        }
        
        this.objects.delete(id);
    }
}

interface WRCObject {
    id: string;
    size: number;
    weight: number;
    referenceWeight: number;
    references: Map<string, number>;
}
```

---

## 🔍 Mark & Sweep

### 기본 Mark & Sweep

```typescript
class MarkAndSweepGC {
    private heap: Map<string, MSObject> = new Map();
    private roots: Set<string> = new Set();
    private totalMemory: number = 0;
    private lastGCTime: number = 0;
    private gcThreshold: number = 1024 * 1024; // 1MB
    private gcCount: number = 0;
    
    allocate(id: string, size: number): void {
        const obj: MSObject = {
            id,
            size,
            marked: false,
            references: new Set(),
            allocatedAt: Date.now()
        };
        
        this.heap.set(id, obj);
        this.totalMemory += size;
        
        console.log(`Allocated ${id} (${size} bytes)`);
        
        // 메모리 임계값 초과 시 GC 실행
        if (this.totalMemory > this.gcThreshold) {
            this.collect();
        }
    }
    
    addRoot(id: string): void {
        this.roots.add(id);
    }
    
    removeRoot(id: string): void {
        this.roots.delete(id);
    }
    
    addReference(fromId: string, toId: string): void {
        const fromObj = this.heap.get(fromId);
        if (fromObj) {
            fromObj.references.add(toId);
        }
    }
    
    // 가비지 컬렉션 실행
    collect(): void {
        const startTime = Date.now();
        this.gcCount++;
        
        console.log(`\n🗑️ Starting GC #${this.gcCount}...`);
        
        // Phase 1: Mark
        this.markPhase();
        
        // Phase 2: Sweep
        const freedMemory = this.sweepPhase();
        
        const gcTime = Date.now() - startTime;
        this.lastGCTime = gcTime;
        
        console.log(`✅ GC #${this.gcCount} completed in ${gcTime}ms`);
        console.log(`   Freed: ${freedMemory} bytes`);
        console.log(`   Current memory: ${this.totalMemory} bytes\n`);
    }
    
    // Mark 단계: 도달 가능한 객체 표시
    private markPhase(): void {
        console.log('  Mark phase...');
        
        // 모든 객체의 marked 플래그 초기화
        for (const obj of this.heap.values()) {
            obj.marked = false;
        }
        
        // 루트에서 시작하여 도달 가능한 객체 마킹
        const stack = Array.from(this.roots);
        let markedCount = 0;
        
        while (stack.length > 0) {
            const objectId = stack.pop()!;
            const obj = this.heap.get(objectId);
            
            if (!obj || obj.marked) continue;
            
            obj.marked = true;
            markedCount++;
            
            // 참조하는 객체들도 스택에 추가
            for (const refId of obj.references) {
                if (this.heap.has(refId)) {
                    stack.push(refId);
                }
            }
        }
        
        console.log(`    Marked ${markedCount} objects`);
    }
    
    // Sweep 단계: 마킹되지 않은 객체 해제
    private sweepPhase(): number {
        console.log('  Sweep phase...');
        
        let freedMemory = 0;
        let freedCount = 0;
        const toDelete: string[] = [];
        
        for (const [id, obj] of this.heap) {
            if (!obj.marked) {
                toDelete.push(id);
                freedMemory += obj.size;
                freedCount++;
            }
        }
        
        // 실제 삭제
        for (const id of toDelete) {
            this.heap.delete(id);
            console.log(`    Freed object ${id}`);
        }
        
        this.totalMemory -= freedMemory;
        console.log(`    Swept ${freedCount} objects`);
        
        return freedMemory;
    }
    
    // 통계
    getStatistics(): any {
        let liveObjects = 0;
        let liveMemory = 0;
        
        for (const obj of this.heap.values()) {
            liveObjects++;
            liveMemory += obj.size;
        }
        
        return {
            liveObjects,
            liveMemory,
            totalMemory: this.totalMemory,
            gcCount: this.gcCount,
            lastGCTime: this.lastGCTime,
            gcThreshold: this.gcThreshold
        };
    }
    
    // GC 임계값 조정
    setGCThreshold(threshold: number): void {
        this.gcThreshold = threshold;
    }
}

interface MSObject {
    id: string;
    size: number;
    marked: boolean;
    references: Set<string>;
    allocatedAt: number;
}
```

### Tri-Color Marking

```typescript
enum Color {
    WHITE = 'white', // 미방문
    GRAY = 'gray',   // 방문했지만 자식 미처리
    BLACK = 'black'  // 완전히 처리됨
}

class TriColorMarkingGC {
    private heap: Map<string, TCMObject> = new Map();
    private roots: Set<string> = new Set();
    private graySet: Set<string> = new Set();
    
    allocate(id: string, size: number): void {
        const obj: TCMObject = {
            id,
            size,
            color: Color.WHITE,
            references: new Set()
        };
        
        this.heap.set(id, obj);
    }
    
    collect(): void {
        console.log('Starting Tri-Color Marking GC...');
        
        // 초기화: 모든 객체를 WHITE로
        this.initializeColors();
        
        // 루트 객체들을 GRAY로 표시
        for (const rootId of this.roots) {
            const obj = this.heap.get(rootId);
            if (obj) {
                obj.color = Color.GRAY;
                this.graySet.add(rootId);
            }
        }
        
        // GRAY 객체가 없을 때까지 처리
        while (this.graySet.size > 0) {
            const grayId = this.graySet.values().next().value;
            this.processGrayObject(grayId);
        }
        
        // WHITE 객체들을 수거
        this.sweepWhiteObjects();
    }
    
    private initializeColors(): void {
        for (const obj of this.heap.values()) {
            obj.color = Color.WHITE;
        }
        this.graySet.clear();
    }
    
    private processGrayObject(objectId: string): void {
        const obj = this.heap.get(objectId);
        if (!obj || obj.color !== Color.GRAY) return;
        
        // 참조하는 WHITE 객체들을 GRAY로 변경
        for (const refId of obj.references) {
            const refObj = this.heap.get(refId);
            if (refObj && refObj.color === Color.WHITE) {
                refObj.color = Color.GRAY;
                this.graySet.add(refId);
            }
        }
        
        // 현재 객체를 BLACK으로 변경
        obj.color = Color.BLACK;
        this.graySet.delete(objectId);
    }
    
    private sweepWhiteObjects(): void {
        const toDelete: string[] = [];
        
        for (const [id, obj] of this.heap) {
            if (obj.color === Color.WHITE) {
                toDelete.push(id);
            }
        }
        
        for (const id of toDelete) {
            this.heap.delete(id);
            console.log(`Collected object ${id}`);
        }
    }
    
    // Write Barrier (증분 GC를 위한)
    writeBarrier(fromId: string, toId: string): void {
        const fromObj = this.heap.get(fromId);
        const toObj = this.heap.get(toId);
        
        if (!fromObj || !toObj) return;
        
        fromObj.references.add(toId);
        
        // Dijkstra's Write Barrier: BLACK -> WHITE 참조 방지
        if (fromObj.color === Color.BLACK && toObj.color === Color.WHITE) {
            toObj.color = Color.GRAY;
            this.graySet.add(toId);
        }
    }
}

interface TCMObject {
    id: string;
    size: number;
    color: Color;
    references: Set<string>;
}
```

---

## 🚀 Generational GC

### 세대별 가비지 컬렉션

```typescript
class GenerationalGC {
    private youngGeneration: Map<string, GenObject> = new Map();
    private oldGeneration: Map<string, GenObject> = new Map();
    private roots: Set<string> = new Set();
    
    // 세대별 설정
    private readonly youngGenSize: number = 10 * 1024 * 1024; // 10MB
    private readonly oldGenSize: number = 100 * 1024 * 1024;  // 100MB
    private readonly survivalThreshold: number = 3; // 3번 생존 시 Old Gen으로 승격
    
    private youngGenUsed: number = 0;
    private oldGenUsed: number = 0;
    private minorGCCount: number = 0;
    private majorGCCount: number = 0;
    
    allocate(id: string, size: number): void {
        // 대부분의 객체는 Young Generation에 할당
        const obj: GenObject = {
            id,
            size,
            generation: 0,
            survivalCount: 0,
            marked: false,
            references: new Set(),
            createdAt: Date.now(),
            lastAccessed: Date.now()
        };
        
        this.youngGeneration.set(id, obj);
        this.youngGenUsed += size;
        
        console.log(`Allocated ${id} in Young Gen (${size} bytes)`);
        
        // Young Generation이 가득 차면 Minor GC 실행
        if (this.youngGenUsed > this.youngGenSize) {
            this.minorGC();
        }
    }
    
    // Minor GC (Young Generation만 수집)
    minorGC(): void {
        const startTime = Date.now();
        this.minorGCCount++;
        
        console.log(`\n🔄 Starting Minor GC #${this.minorGCCount}...`);
        
        // Young Generation의 살아있는 객체 마킹
        this.markYoungGeneration();
        
        // 살아있는 객체들 처리
        const survivors: GenObject[] = [];
        const promoted: GenObject[] = [];
        let freedMemory = 0;
        
        for (const [id, obj] of this.youngGeneration) {
            if (obj.marked) {
                obj.survivalCount++;
                obj.marked = false;
                
                if (obj.survivalCount >= this.survivalThreshold) {
                    // Old Generation으로 승격
                    promoted.push(obj);
                } else {
                    // 생존자 영역으로 이동
                    survivors.push(obj);
                }
            } else {
                // 가비지 수집
                freedMemory += obj.size;
            }
        }
        
        // Young Generation 정리
        this.youngGeneration.clear();
        this.youngGenUsed = 0;
        
        // 생존자들 다시 추가
        for (const obj of survivors) {
            this.youngGeneration.set(obj.id, obj);
            this.youngGenUsed += obj.size;
        }
        
        // Old Generation으로 승격
        for (const obj of promoted) {
            obj.generation = 1;
            this.oldGeneration.set(obj.id, obj);
            this.oldGenUsed += obj.size;
            console.log(`  Promoted ${obj.id} to Old Gen`);
        }
        
        const gcTime = Date.now() - startTime;
        console.log(`✅ Minor GC completed in ${gcTime}ms`);
        console.log(`   Freed: ${freedMemory} bytes`);
        console.log(`   Survivors: ${survivors.length}`);
        console.log(`   Promoted: ${promoted.length}\n`);
        
        // Old Generation이 가득 차면 Major GC 실행
        if (this.oldGenUsed > this.oldGenSize) {
            this.majorGC();
        }
    }
    
    // Major GC (전체 힙 수집)
    majorGC(): void {
        const startTime = Date.now();
        this.majorGCCount++;
        
        console.log(`\n🗑️ Starting Major GC #${this.majorGCCount}...`);
        
        // 전체 힙 마킹
        this.markAllGenerations();
        
        // Young Generation 정리
        const youngSurvivors: GenObject[] = [];
        let youngFreed = 0;
        
        for (const [id, obj] of this.youngGeneration) {
            if (obj.marked) {
                obj.marked = false;
                youngSurvivors.push(obj);
            } else {
                youngFreed += obj.size;
            }
        }
        
        // Old Generation 정리
        const oldSurvivors: GenObject[] = [];
        let oldFreed = 0;
        
        for (const [id, obj] of this.oldGeneration) {
            if (obj.marked) {
                obj.marked = false;
                oldSurvivors.push(obj);
            } else {
                oldFreed += obj.size;
            }
        }
        
        // 힙 재구성
        this.youngGeneration.clear();
        this.oldGeneration.clear();
        this.youngGenUsed = 0;
        this.oldGenUsed = 0;
        
        for (const obj of youngSurvivors) {
            this.youngGeneration.set(obj.id, obj);
            this.youngGenUsed += obj.size;
        }
        
        for (const obj of oldSurvivors) {
            this.oldGeneration.set(obj.id, obj);
            this.oldGenUsed += obj.size;
        }
        
        const gcTime = Date.now() - startTime;
        console.log(`✅ Major GC completed in ${gcTime}ms`);
        console.log(`   Young Gen freed: ${youngFreed} bytes`);
        console.log(`   Old Gen freed: ${oldFreed} bytes`);
        console.log(`   Total freed: ${youngFreed + oldFreed} bytes\n`);
    }
    
    private markYoungGeneration(): void {
        // Young Generation의 루트에서 도달 가능한 객체 마킹
        const stack: string[] = [];
        
        // 루트에서 시작
        for (const rootId of this.roots) {
            if (this.youngGeneration.has(rootId)) {
                stack.push(rootId);
            }
        }
        
        // Old Generation에서 Young Generation을 참조하는 경우 (Remember Set)
        for (const oldObj of this.oldGeneration.values()) {
            for (const refId of oldObj.references) {
                if (this.youngGeneration.has(refId)) {
                    stack.push(refId);
                }
            }
        }
        
        // 마킹
        while (stack.length > 0) {
            const objectId = stack.pop()!;
            const obj = this.youngGeneration.get(objectId);
            
            if (!obj || obj.marked) continue;
            
            obj.marked = true;
            
            for (const refId of obj.references) {
                if (this.youngGeneration.has(refId)) {
                    stack.push(refId);
                }
            }
        }
    }
    
    private markAllGenerations(): void {
        const stack = Array.from(this.roots);
        
        while (stack.length > 0) {
            const objectId = stack.pop()!;
            
            let obj = this.youngGeneration.get(objectId) || 
                     this.oldGeneration.get(objectId);
            
            if (!obj || obj.marked) continue;
            
            obj.marked = true;
            
            for (const refId of obj.references) {
                stack.push(refId);
            }
        }
    }
    
    // 통계
    getStatistics(): any {
        return {
            youngGeneration: {
                size: this.youngGenSize,
                used: this.youngGenUsed,
                objects: this.youngGeneration.size
            },
            oldGeneration: {
                size: this.oldGenSize,
                used: this.oldGenUsed,
                objects: this.oldGeneration.size
            },
            gcCount: {
                minor: this.minorGCCount,
                major: this.majorGCCount
            }
        };
    }
}

interface GenObject {
    id: string;
    size: number;
    generation: number;
    survivalCount: number;
    marked: boolean;
    references: Set<string>;
    createdAt: number;
    lastAccessed: number;
}
```

---

## 💡 Concurrent & Incremental GC

### Incremental Mark & Sweep

```typescript
class IncrementalGC {
    private heap: Map<string, IncrObject> = new Map();
    private roots: Set<string> = new Set();
    private graySet: Set<string> = new Set();
    private gcState: GCState = GCState.IDLE;
    private incrementalWorkUnit: number = 10; // 한 번에 처리할 객체 수
    
    // GC를 여러 단계로 나누어 실행
    incrementalStep(): void {
        switch (this.gcState) {
            case GCState.IDLE:
                if (this.shouldStartGC()) {
                    this.startMarking();
                }
                break;
                
            case GCState.MARKING:
                this.incrementalMark();
                break;
                
            case GCState.SWEEPING:
                this.incrementalSweep();
                break;
        }
    }
    
    private shouldStartGC(): boolean {
        // GC 시작 조건 검사
        const memoryPressure = this.calculateMemoryPressure();
        return memoryPressure > 0.8; // 80% 이상 사용 시
    }
    
    private startMarking(): void {
        console.log('Starting incremental marking...');
        
        // 모든 객체를 WHITE로 초기화
        for (const obj of this.heap.values()) {
            obj.color = Color.WHITE;
        }
        
        // 루트 객체들을 GRAY로 표시
        for (const rootId of this.roots) {
            const obj = this.heap.get(rootId);
            if (obj) {
                obj.color = Color.GRAY;
                this.graySet.add(rootId);
            }
        }
        
        this.gcState = GCState.MARKING;
    }
    
    private incrementalMark(): void {
        let processed = 0;
        
        // 정해진 작업량만큼만 처리
        while (this.graySet.size > 0 && processed < this.incrementalWorkUnit) {
            const grayId = this.graySet.values().next().value;
            const obj = this.heap.get(grayId);
            
            if (obj && obj.color === Color.GRAY) {
                // 참조하는 객체들을 GRAY로 변경
                for (const refId of obj.references) {
                    const refObj = this.heap.get(refId);
                    if (refObj && refObj.color === Color.WHITE) {
                        refObj.color = Color.GRAY;
                        this.graySet.add(refId);
                    }
                }
                
                // 현재 객체를 BLACK으로 변경
                obj.color = Color.BLACK;
                this.graySet.delete(grayId);
            }
            
            processed++;
        }
        
        // 마킹 완료 확인
        if (this.graySet.size === 0) {
            console.log('Marking phase completed');
            this.gcState = GCState.SWEEPING;
            this.startSweeping();
        }
    }
    
    private startSweeping(): void {
        // Sweep를 위한 준비
        (this as any).sweepIterator = this.heap.keys();
    }
    
    private incrementalSweep(): void {
        const iterator = (this as any).sweepIterator;
        let processed = 0;
        let freedCount = 0;
        
        while (processed < this.incrementalWorkUnit) {
            const result = iterator.next();
            
            if (result.done) {
                console.log(`Sweeping completed, freed ${freedCount} objects`);
                this.gcState = GCState.IDLE;
                break;
            }
            
            const objectId = result.value;
            const obj = this.heap.get(objectId);
            
            if (obj && obj.color === Color.WHITE) {
                this.heap.delete(objectId);
                freedCount++;
            }
            
            processed++;
        }
    }
    
    // Write Barrier for incremental GC
    writeBarrier(mutatorId: string, newRefId: string): void {
        const mutator = this.heap.get(mutatorId);
        const newRef = this.heap.get(newRefId);
        
        if (!mutator || !newRef) return;
        
        mutator.references.add(newRefId);
        
        // Incremental update write barrier
        if (this.gcState === GCState.MARKING) {
            if (mutator.color === Color.BLACK && newRef.color === Color.WHITE) {
                // BLACK -> WHITE 참조가 생성되면 즉시 GRAY로 변경
                newRef.color = Color.GRAY;
                this.graySet.add(newRefId);
            }
        }
    }
    
    private calculateMemoryPressure(): number {
        // 메모리 압력 계산 (0.0 ~ 1.0)
        const totalObjects = this.heap.size;
        const maxObjects = 10000;
        return Math.min(totalObjects / maxObjects, 1.0);
    }
}

enum GCState {
    IDLE = 'idle',
    MARKING = 'marking',
    SWEEPING = 'sweeping'
}

interface IncrObject {
    id: string;
    size: number;
    color: Color;
    references: Set<string>;
}
```

### Concurrent Mark & Sweep (CMS)

```typescript
class ConcurrentMarkSweepGC {
    private heap: Map<string, CMSObject> = new Map();
    private roots: Set<string> = new Set();
    private isGCRunning: boolean = false;
    private mutatorPaused: boolean = false;
    
    // CMS 단계들
    async collectConcurrent(): Promise<void> {
        if (this.isGCRunning) return;
        
        this.isGCRunning = true;
        console.log('Starting Concurrent Mark & Sweep...');
        
        try {
            // Phase 1: Initial Mark (STW - Stop The World)
            await this.initialMark();
            
            // Phase 2: Concurrent Mark (concurrent with mutator)
            await this.concurrentMark();
            
            // Phase 3: Remark (STW)
            await this.remark();
            
            // Phase 4: Concurrent Sweep (concurrent with mutator)
            await this.concurrentSweep();
            
        } finally {
            this.isGCRunning = false;
            console.log('CMS GC completed');
        }
    }
    
    private async initialMark(): Promise<void> {
        console.log('Phase 1: Initial Mark (STW)');
        this.pauseMutator();
        
        // 루트 직접 참조 객체만 마킹
        for (const rootId of this.roots) {
            const obj = this.heap.get(rootId);
            if (obj) {
                obj.marked = true;
                obj.markBit = 1;
            }
        }
        
        this.resumeMutator();
        console.log('Initial Mark completed');
    }
    
    private async concurrentMark(): Promise<void> {
        console.log('Phase 2: Concurrent Mark');
        
        const worklist: string[] = Array.from(this.roots);
        const BATCH_SIZE = 100;
        
        while (worklist.length > 0) {
            const batch = worklist.splice(0, BATCH_SIZE);
            
            for (const objectId of batch) {
                const obj = this.heap.get(objectId);
                if (!obj || obj.markBit === 2) continue;
                
                obj.markBit = 2; // 완전히 마킹됨
                
                // 참조 객체들 추가
                for (const refId of obj.references) {
                    const refObj = this.heap.get(refId);
                    if (refObj && !refObj.marked) {
                        refObj.marked = true;
                        refObj.markBit = 1;
                        worklist.push(refId);
                    }
                }
            }
            
            // Mutator가 실행될 기회를 줌
            await this.yieldToMutator();
        }
        
        console.log('Concurrent Mark completed');
    }
    
    private async remark(): Promise<void> {
        console.log('Phase 3: Remark (STW)');
        this.pauseMutator();
        
        // Concurrent Mark 중 변경된 참조 처리
        const dirtyObjects = this.findDirtyObjects();
        
        for (const obj of dirtyObjects) {
            for (const refId of obj.references) {
                const refObj = this.heap.get(refId);
                if (refObj && !refObj.marked) {
                    refObj.marked = true;
                    refObj.markBit = 2;
                }
            }
        }
        
        this.resumeMutator();
        console.log('Remark completed');
    }
    
    private async concurrentSweep(): Promise<void> {
        console.log('Phase 4: Concurrent Sweep');
        
        const toDelete: string[] = [];
        
        for (const [id, obj] of this.heap) {
            if (!obj.marked) {
                toDelete.push(id);
            } else {
                // 다음 GC를 위해 마킹 초기화
                obj.marked = false;
                obj.markBit = 0;
            }
            
            // 주기적으로 Mutator에게 제어권 양보
            if (toDelete.length % 100 === 0) {
                await this.yieldToMutator();
            }
        }
        
        // 실제 삭제
        for (const id of toDelete) {
            this.heap.delete(id);
        }
        
        console.log(`Concurrent Sweep completed, freed ${toDelete.length} objects`);
    }
    
    private pauseMutator(): void {
        this.mutatorPaused = true;
        console.log('  Mutator paused');
    }
    
    private resumeMutator(): void {
        this.mutatorPaused = false;
        console.log('  Mutator resumed');
    }
    
    private async yieldToMutator(): Promise<void> {
        // Mutator가 실행될 수 있도록 제어권 양보
        await new Promise(resolve => setImmediate(resolve));
    }
    
    private findDirtyObjects(): CMSObject[] {
        // Write barrier로 추적된 변경된 객체들 찾기
        const dirty: CMSObject[] = [];
        
        for (const obj of this.heap.values()) {
            if (obj.dirty) {
                dirty.push(obj);
                obj.dirty = false;
            }
        }
        
        return dirty;
    }
    
    // Write Barrier for CMS
    writeBarrier(fromId: string, toId: string): void {
        if (this.mutatorPaused) {
            // STW 중에는 write barrier 불필요
            return;
        }
        
        const fromObj = this.heap.get(fromId);
        if (fromObj && this.isGCRunning) {
            fromObj.dirty = true;
        }
    }
}

interface CMSObject {
    id: string;
    size: number;
    marked: boolean;
    markBit: number; // 0: 미방문, 1: 부분마킹, 2: 완전마킹
    dirty: boolean;  // Write barrier 추적용
    references: Set<string>;
}
```

---

## ⚡ 고급 GC 알고리즘

### G1 (Garbage-First) GC 시뮬레이션

```typescript
class G1GC {
    private regions: Map<number, Region> = new Map();
    private regionSize: number = 1024 * 1024; // 1MB per region
    private maxRegions: number = 2048; // 2GB heap
    private collectionSet: Set<number> = new Set();
    private rememberSets: Map<number, Set<number>> = new Map();
    
    constructor() {
        // 힙을 리전으로 분할
        for (let i = 0; i < this.maxRegions; i++) {
            this.regions.set(i, {
                id: i,
                type: RegionType.FREE,
                liveBytes: 0,
                capacity: this.regionSize,
                objects: new Map(),
                rememberSet: new Set()
            });
        }
    }
    
    allocate(size: number): number {
        // 적절한 리전 찾기
        const region = this.findAvailableRegion(size);
        if (!region) {
            // Young GC 또는 Mixed GC 트리거
            this.collectGarbage();
            return this.allocate(size); // 재시도
        }
        
        // 객체 할당
        const objectId = this.generateObjectId();
        region.objects.set(objectId, {
            id: objectId,
            size,
            marked: false,
            references: new Set()
        });
        
        region.liveBytes += size;
        
        // 리전 타입 설정
        if (region.type === RegionType.FREE) {
            region.type = RegionType.EDEN;
        }
        
        return objectId;
    }
    
    collectGarbage(): void {
        console.log('Starting G1 Collection...');
        
        // Collection Set 선택
        this.selectCollectionSet();
        
        // Evacuation 수행
        this.evacuate();
        
        // Collection Set 정리
        this.cleanupCollectionSet();
    }
    
    private selectCollectionSet(): void {
        // 가비지 비율이 높은 리전들을 우선 선택
        const regionStats: Array<{id: number; garbageRatio: number}> = [];
        
        for (const [id, region] of this.regions) {
            if (region.type !== RegionType.FREE) {
                const totalBytes = region.objects.size * 100; // 추정
                const garbageRatio = 1 - (region.liveBytes / totalBytes);
                regionStats.push({ id, garbageRatio });
            }
        }
        
        // 가비지 비율 기준 정렬
        regionStats.sort((a, b) => b.garbageRatio - a.garbageRatio);
        
        // 상위 리전들을 Collection Set에 추가
        const targetCount = Math.min(10, regionStats.length);
        for (let i = 0; i < targetCount; i++) {
            this.collectionSet.add(regionStats[i].id);
        }
        
        console.log(`Selected ${this.collectionSet.size} regions for collection`);
    }
    
    private evacuate(): void {
        // Collection Set의 살아있는 객체들을 다른 리전으로 복사
        for (const regionId of this.collectionSet) {
            const region = this.regions.get(regionId)!;
            
            for (const [objId, obj] of region.objects) {
                if (this.isObjectAlive(obj)) {
                    // 새 리전으로 복사
                    const targetRegion = this.findEvacuationTarget();
                    if (targetRegion) {
                        targetRegion.objects.set(objId, { ...obj });
                        targetRegion.liveBytes += obj.size;
                    }
                }
            }
        }
    }
    
    private cleanupCollectionSet(): void {
        // Collection Set 리전들을 FREE로 변경
        for (const regionId of this.collectionSet) {
            const region = this.regions.get(regionId)!;
            region.type = RegionType.FREE;
            region.liveBytes = 0;
            region.objects.clear();
            region.rememberSet.clear();
        }
        
        this.collectionSet.clear();
        console.log('Collection completed');
    }
    
    private findAvailableRegion(size: number): Region | null {
        for (const region of this.regions.values()) {
            if (region.type === RegionType.FREE || 
                (region.capacity - region.liveBytes) >= size) {
                return region;
            }
        }
        return null;
    }
    
    private findEvacuationTarget(): Region | null {
        for (const region of this.regions.values()) {
            if (region.type === RegionType.FREE) {
                region.type = RegionType.SURVIVOR;
                return region;
            }
        }
        return null;
    }
    
    private isObjectAlive(obj: any): boolean {
        // 간단한 도달 가능성 검사
        return Math.random() > 0.3; // 시뮬레이션용
    }
    
    private generateObjectId(): number {
        return Math.floor(Math.random() * Number.MAX_SAFE_INTEGER);
    }
}

enum RegionType {
    FREE = 'free',
    EDEN = 'eden',
    SURVIVOR = 'survivor',
    OLD = 'old',
    HUMONGOUS = 'humongous'
}

interface Region {
    id: number;
    type: RegionType;
    liveBytes: number;
    capacity: number;
    objects: Map<number, any>;
    rememberSet: Set<number>;
}
```

---

## 💻 TypeScript 구현

### 통합 GC 시스템

```typescript
class UnifiedGCSystem {
    private heap: Map<string, UnifiedObject> = new Map();
    private roots: Set<string> = new Set();
    private gcStrategy: GCStrategy;
    private statistics: GCStatistics;
    
    constructor(strategy: GCStrategy = GCStrategy.GENERATIONAL) {
        this.gcStrategy = strategy;
        this.statistics = {
            totalAllocations: 0,
            totalDeallocations: 0,
            totalGCRuns: 0,
            totalGCTime: 0,
            currentHeapSize: 0,
            peakHeapSize: 0
        };
    }
    
    allocate(id: string, size: number): void {
        const obj: UnifiedObject = {
            id,
            size,
            references: new Set(),
            metadata: {
                createdAt: Date.now(),
                lastAccessed: Date.now(),
                accessCount: 0,
                generation: 0,
                marked: false,
                referenceCount: 0
            }
        };
        
        this.heap.set(id, obj);
        this.statistics.totalAllocations++;
        this.statistics.currentHeapSize += size;
        this.statistics.peakHeapSize = Math.max(
            this.statistics.peakHeapSize, 
            this.statistics.currentHeapSize
        );
        
        // GC 트리거 조건 확인
        if (this.shouldTriggerGC()) {
            this.collect();
        }
    }
    
    collect(): void {
        const startTime = Date.now();
        this.statistics.totalGCRuns++;
        
        console.log(`\n🗑️ Starting GC (Strategy: ${this.gcStrategy})...`);
        
        let freedObjects = 0;
        let freedMemory = 0;
        
        switch (this.gcStrategy) {
            case GCStrategy.REFERENCE_COUNTING:
                ({ freedObjects, freedMemory } = this.collectReferenceCouting());
                break;
                
            case GCStrategy.MARK_AND_SWEEP:
                ({ freedObjects, freedMemory } = this.collectMarkAndSweep());
                break;
                
            case GCStrategy.GENERATIONAL:
                ({ freedObjects, freedMemory } = this.collectGenerational());
                break;
                
            case GCStrategy.INCREMENTAL:
                ({ freedObjects, freedMemory } = this.collectIncremental());
                break;
        }
        
        const gcTime = Date.now() - startTime;
        this.statistics.totalGCTime += gcTime;
        this.statistics.totalDeallocations += freedObjects;
        this.statistics.currentHeapSize -= freedMemory;
        
        console.log(`✅ GC completed in ${gcTime}ms`);
        console.log(`   Freed: ${freedObjects} objects (${freedMemory} bytes)`);
        console.log(`   Heap size: ${this.statistics.currentHeapSize} bytes\n`);
    }
    
    private collectReferenceCouting(): {freedObjects: number; freedMemory: number} {
        let freedObjects = 0;
        let freedMemory = 0;
        
        // Reference counting 로직
        const toDelete: string[] = [];
        
        for (const [id, obj] of this.heap) {
            if (obj.metadata.referenceCount === 0 && !this.roots.has(id)) {
                toDelete.push(id);
                freedObjects++;
                freedMemory += obj.size;
            }
        }
        
        for (const id of toDelete) {
            this.heap.delete(id);
        }
        
        return { freedObjects, freedMemory };
    }
    
    private collectMarkAndSweep(): {freedObjects: number; freedMemory: number} {
        // Mark phase
        this.markReachableObjects();
        
        // Sweep phase
        let freedObjects = 0;
        let freedMemory = 0;
        const toDelete: string[] = [];
        
        for (const [id, obj] of this.heap) {
            if (!obj.metadata.marked) {
                toDelete.push(id);
                freedObjects++;
                freedMemory += obj.size;
            } else {
                obj.metadata.marked = false; // Reset for next GC
            }
        }
        
        for (const id of toDelete) {
            this.heap.delete(id);
        }
        
        return { freedObjects, freedMemory };
    }
    
    private collectGenerational(): {freedObjects: number; freedMemory: number} {
        // Generational GC 로직
        let freedObjects = 0;
        let freedMemory = 0;
        
        // Young generation collection
        for (const [id, obj] of this.heap) {
            if (obj.metadata.generation === 0) {
                // Young generation 처리
                if (!this.isReachable(id)) {
                    this.heap.delete(id);
                    freedObjects++;
                    freedMemory += obj.size;
                } else {
                    // Promote to old generation
                    obj.metadata.generation++;
                }
            }
        }
        
        return { freedObjects, freedMemory };
    }
    
    private collectIncremental(): {freedObjects: number; freedMemory: number} {
        // Incremental GC - 부분적으로만 수집
        const WORK_UNIT = 100;
        let processed = 0;
        let freedObjects = 0;
        let freedMemory = 0;
        
        for (const [id, obj] of this.heap) {
            if (processed >= WORK_UNIT) break;
            
            if (!this.isReachable(id)) {
                this.heap.delete(id);
                freedObjects++;
                freedMemory += obj.size;
            }
            
            processed++;
        }
        
        return { freedObjects, freedMemory };
    }
    
    private markReachableObjects(): void {
        const stack = Array.from(this.roots);
        
        while (stack.length > 0) {
            const objectId = stack.pop()!;
            const obj = this.heap.get(objectId);
            
            if (!obj || obj.metadata.marked) continue;
            
            obj.metadata.marked = true;
            
            for (const refId of obj.references) {
                stack.push(refId);
            }
        }
    }
    
    private isReachable(objectId: string): boolean {
        // 간단한 도달 가능성 검사
        if (this.roots.has(objectId)) return true;
        
        // DFS로 루트에서 도달 가능한지 확인
        const visited = new Set<string>();
        const stack = Array.from(this.roots);
        
        while (stack.length > 0) {
            const id = stack.pop()!;
            if (id === objectId) return true;
            
            if (visited.has(id)) continue;
            visited.add(id);
            
            const obj = this.heap.get(id);
            if (obj) {
                for (const refId of obj.references) {
                    stack.push(refId);
                }
            }
        }
        
        return false;
    }
    
    private shouldTriggerGC(): boolean {
        // GC 트리거 조건
        const heapThreshold = 10 * 1024 * 1024; // 10MB
        const allocationRate = this.statistics.totalAllocations % 1000 === 0;
        
        return this.statistics.currentHeapSize > heapThreshold || allocationRate;
    }
    
    // 통계 조회
    getStatistics(): GCStatistics {
        return {
            ...this.statistics,
            averageGCTime: this.statistics.totalGCRuns > 0 
                ? this.statistics.totalGCTime / this.statistics.totalGCRuns 
                : 0
        };
    }
    
    // GC 전략 변경
    setStrategy(strategy: GCStrategy): void {
        this.gcStrategy = strategy;
        console.log(`GC strategy changed to: ${strategy}`);
    }
}

enum GCStrategy {
    REFERENCE_COUNTING = 'reference_counting',
    MARK_AND_SWEEP = 'mark_and_sweep',
    GENERATIONAL = 'generational',
    INCREMENTAL = 'incremental'
}

interface UnifiedObject {
    id: string;
    size: number;
    references: Set<string>;
    metadata: {
        createdAt: number;
        lastAccessed: number;
        accessCount: number;
        generation: number;
        marked: boolean;
        referenceCount: number;
    };
}

interface GCStatistics {
    totalAllocations: number;
    totalDeallocations: number;
    totalGCRuns: number;
    totalGCTime: number;
    currentHeapSize: number;
    peakHeapSize: number;
    averageGCTime?: number;
}
```

---

## 📈 성능 분석

### GC 알고리즘 벤치마크

```typescript
class GCBenchmark {
    async runBenchmarks(): Promise<void> {
        console.log('🚀 가비지 컬렉션 알고리즘 벤치마크\n');
        
        const workloads = [
            { name: 'Small Objects', objectSize: 100, objectCount: 10000 },
            { name: 'Large Objects', objectSize: 10000, objectCount: 1000 },
            { name: 'Mixed Size', objectSize: -1, objectCount: 5000 }
        ];
        
        const strategies = [
            GCStrategy.REFERENCE_COUNTING,
            GCStrategy.MARK_AND_SWEEP,
            GCStrategy.GENERATIONAL,
            GCStrategy.INCREMENTAL
        ];
        
        for (const workload of workloads) {
            console.log(`\n📊 Workload: ${workload.name}`);
            console.log(`   Objects: ${workload.objectCount}, Size: ${workload.objectSize} bytes`);
            
            for (const strategy of strategies) {
                const result = await this.benchmarkStrategy(strategy, workload);
                
                console.log(`\n   ${strategy}:`);
                console.log(`     Total Time: ${result.totalTime}ms`);
                console.log(`     GC Time: ${result.gcTime}ms (${result.gcOverhead.toFixed(1)}%)`);
                console.log(`     Throughput: ${result.throughput.toFixed(0)} ops/sec`);
                console.log(`     Avg Pause: ${result.avgPause.toFixed(2)}ms`);
                console.log(`     Max Pause: ${result.maxPause}ms`);
            }
        }
        
        this.printAnalysis();
    }
    
    private async benchmarkStrategy(
        strategy: GCStrategy, 
        workload: any
    ): Promise<BenchmarkResult> {
        const gc = new UnifiedGCSystem(strategy);
        const startTime = Date.now();
        
        const gcPauses: number[] = [];
        let totalGCTime = 0;
        
        // 워크로드 실행
        for (let i = 0; i < workload.objectCount; i++) {
            const id = `obj_${i}`;
            const size = workload.objectSize > 0 
                ? workload.objectSize 
                : Math.floor(Math.random() * 10000) + 100;
            
            gc.allocate(id, size);
            
            // 일부 객체를 루트로 설정
            if (Math.random() < 0.1) {
                gc.addRoot(id);
            }
            
            // 랜덤하게 참조 추가
            if (i > 0 && Math.random() < 0.3) {
                const targetId = `obj_${Math.floor(Math.random() * i)}`;
                gc.addReference(id, targetId);
            }
            
            // 랜덤하게 루트 제거 (가비지 생성)
            if (Math.random() < 0.05) {
                const targetId = `obj_${Math.floor(Math.random() * i)}`;
                gc.removeRoot(targetId);
            }
        }
        
        // 최종 GC 실행
        const gcStart = Date.now();
        gc.collect();
        const gcEnd = Date.now();
        gcPauses.push(gcEnd - gcStart);
        
        const endTime = Date.now();
        const stats = gc.getStatistics();
        
        return {
            strategy,
            totalTime: endTime - startTime,
            gcTime: stats.totalGCTime,
            gcOverhead: (stats.totalGCTime / (endTime - startTime)) * 100,
            throughput: (workload.objectCount / ((endTime - startTime) / 1000)),
            avgPause: stats.averageGCTime || 0,
            maxPause: Math.max(...gcPauses, 0),
            totalGCRuns: stats.totalGCRuns
        };
    }
    
    private printAnalysis(): void {
        console.log('\n\n📈 분석 요약:');
        console.log('='.repeat(60));
        
        console.log('\n🎯 알고리즘별 특성:');
        console.log('\nReference Counting:');
        console.log('  ✅ 즉시 메모리 해제');
        console.log('  ✅ 예측 가능한 성능');
        console.log('  ❌ 순환 참조 처리 불가');
        console.log('  ❌ 참조 카운트 오버헤드');
        
        console.log('\nMark & Sweep:');
        console.log('  ✅ 순환 참조 처리 가능');
        console.log('  ✅ 구현 단순');
        console.log('  ❌ Stop-the-world 일시정지');
        console.log('  ❌ 메모리 단편화');
        
        console.log('\nGenerational:');
        console.log('  ✅ 대부분 객체가 young에서 죽는 특성 활용');
        console.log('  ✅ Minor GC로 대부분 처리');
        console.log('  ❌ 복잡한 구현');
        console.log('  ❌ Write barrier 오버헤드');
        
        console.log('\nIncremental:');
        console.log('  ✅ 짧은 일시정지 시간');
        console.log('  ✅ 실시간 시스템 적합');
        console.log('  ❌ 전체 처리량 감소');
        console.log('  ❌ Write barrier 필수');
        
        console.log('\n🏆 사용 권장사항:');
        console.log('  • 실시간 시스템: Incremental GC');
        console.log('  • 서버 애플리케이션: Generational GC');
        console.log('  • 임베디드 시스템: Reference Counting');
        console.log('  • 단순한 애플리케이션: Mark & Sweep');
    }
}

interface BenchmarkResult {
    strategy: GCStrategy;
    totalTime: number;
    gcTime: number;
    gcOverhead: number;
    throughput: number;
    avgPause: number;
    maxPause: number;
    totalGCRuns: number;
}
```

---

## 🏢 실제 시스템 사례

### JVM G1GC 시뮬레이션

```typescript
// JVM의 G1GC 동작 시뮬레이션
class JVMG1GC {
    private youngGen: YoungGeneration;
    private oldGen: OldGeneration;
    private heapRegions: Region[];
    private concurrentMark: ConcurrentMark;
    
    constructor() {
        // G1GC 초기화
        this.heapRegions = this.initializeRegions();
        this.youngGen = new YoungGeneration(this.heapRegions);
        this.oldGen = new OldGeneration(this.heapRegions);
        this.concurrentMark = new ConcurrentMark(this.heapRegions);
    }
    
    private initializeRegions(): Region[] {
        const regions: Region[] = [];
        const regionCount = 2048; // 2GB heap with 1MB regions
        
        for (let i = 0; i < regionCount; i++) {
            regions.push({
                id: i,
                type: RegionType.FREE,
                liveBytes: 0,
                capacity: 1024 * 1024,
                objects: new Map(),
                rememberSet: new Set()
            });
        }
        
        return regions;
    }
    
    // Young GC (STW)
    youngGC(): void {
        console.log('Starting G1 Young GC (STW)...');
        
        // Eden과 Survivor regions 수집
        const collectionSet = this.youngGen.selectCollectionSet();
        
        // 살아있는 객체 복사 (evacuation)
        this.evacuate(collectionSet);
        
        // Collection set 정리
        this.cleanup(collectionSet);
        
        console.log('Young GC completed');
    }
    
    // Mixed GC
    mixedGC(): void {
        console.log('Starting G1 Mixed GC...');
        
        // Young regions + 일부 Old regions 수집
        const youngSet = this.youngGen.selectCollectionSet();
        const oldSet = this.oldGen.selectCandidates(5); // 5개 old regions
        
        const collectionSet = [...youngSet, ...oldSet];
        
        this.evacuate(collectionSet);
        this.cleanup(collectionSet);
        
        console.log('Mixed GC completed');
    }
    
    // Concurrent Marking Cycle
    async concurrentMarkingCycle(): Promise<void> {
        console.log('Starting Concurrent Marking Cycle...');
        
        // Initial Mark (STW)
        await this.concurrentMark.initialMark();
        
        // Root Region Scan
        await this.concurrentMark.rootRegionScan();
        
        // Concurrent Mark
        await this.concurrentMark.concurrentMark();
        
        // Remark (STW)
        await this.concurrentMark.remark();
        
        // Cleanup (STW)
        await this.concurrentMark.cleanup();
        
        console.log('Concurrent Marking Cycle completed');
    }
    
    private evacuate(collectionSet: Region[]): void {
        // 살아있는 객체들을 새로운 regions로 복사
        for (const region of collectionSet) {
            for (const [objId, obj] of region.objects) {
                if (this.isAlive(obj)) {
                    const targetRegion = this.findTargetRegion();
                    if (targetRegion) {
                        targetRegion.objects.set(objId, obj);
                        targetRegion.liveBytes += obj.size;
                    }
                }
            }
        }
    }
    
    private cleanup(collectionSet: Region[]): void {
        for (const region of collectionSet) {
            region.type = RegionType.FREE;
            region.liveBytes = 0;
            region.objects.clear();
            region.rememberSet.clear();
        }
    }
    
    private isAlive(obj: any): boolean {
        // 도달 가능성 검사 시뮬레이션
        return Math.random() > 0.4;
    }
    
    private findTargetRegion(): Region | null {
        return this.heapRegions.find(r => r.type === RegionType.FREE) || null;
    }
}

class YoungGeneration {
    constructor(private regions: Region[]) {}
    
    selectCollectionSet(): Region[] {
        return this.regions.filter(r => 
            r.type === RegionType.EDEN || r.type === RegionType.SURVIVOR
        );
    }
}

class OldGeneration {
    constructor(private regions: Region[]) {}
    
    selectCandidates(count: number): Region[] {
        const oldRegions = this.regions.filter(r => r.type === RegionType.OLD);
        
        // 가비지가 많은 region 우선 선택
        oldRegions.sort((a, b) => {
            const aGarbage = a.capacity - a.liveBytes;
            const bGarbage = b.capacity - b.liveBytes;
            return bGarbage - aGarbage;
        });
        
        return oldRegions.slice(0, count);
    }
}

class ConcurrentMark {
    constructor(private regions: Region[]) {}
    
    async initialMark(): Promise<void> {
        console.log('  Initial Mark (STW)');
        // Root set 마킹
    }
    
    async rootRegionScan(): Promise<void> {
        console.log('  Root Region Scan');
        // Survivor regions 스캔
    }
    
    async concurrentMark(): Promise<void> {
        console.log('  Concurrent Mark');
        // 전체 heap 마킹 (concurrent)
    }
    
    async remark(): Promise<void> {
        console.log('  Remark (STW)');
        // SATB (Snapshot-At-The-Beginning) 처리
    }
    
    async cleanup(): Promise<void> {
        console.log('  Cleanup (STW)');
        // 완전히 비어있는 regions 회수
    }
}
```

---

## 📚 참고자료

### 학술 자료
- McCarthy, J. (1960). "Recursive functions of symbolic expressions and their computation by machine"
- Cheney, C. J. (1970). "A nonrecursive list compacting algorithm"
- Ungar, D. (1984). "Generation Scavenging: A non-disruptive high performance storage reclamation algorithm"

### 실무 자료
- [[Java Garbage Collection Tuning]]
- [[.NET GC Internals]]
- [[V8 JavaScript Engine GC]]

### 관련 문서
- [[05_memory-hierarchy]] - 메모리 계층 구조
- [[11_async-programming]] - 비동기 프로그래밍과 메모리 관리

---

> [!success] 가비지 컬렉션 핵심 정리
> 1. **Reference Counting**: 즉시 해제, 순환 참조 문제
> 2. **Mark & Sweep**: 순환 참조 해결, STW 문제
> 3. **Generational GC**: 세대별 관리로 효율성 향상
> 4. **Concurrent GC**: 애플리케이션과 동시 실행
> 5. **선택 기준**: 애플리케이션 특성에 따른 적절한 GC 선택이 중요

> [!quote]
> "가비지 컬렉션은 프로그래머를 메모리 관리의 부담에서 해방시키지만, 그 원리를 이해하는 것은 효율적인 프로그램 작성의 핵심이다." - 현대 프로그래밍 언어 설계