---
tags:
  - 운영체제
  - OS
  - 동기화
  - 프리미티브
  - 병행성
  - CTO강의
created: 2025-01-12
updated: 2025-01-12
aliases:
  - 동기화프리미티브
  - Synchronization Primitives
  - 병행성제어
description: 고급 동기화 프리미티브들과 현대적 병행성 제어 메커니즘의 TypeScript 구현
status: published
category: guide
---

# 동기화 프리미티브 (Synchronization Primitives) - 고급 병행성 제어

> [!info] 개요
> 동기화 프리미티브는 병행 프로그래밍에서 공유 자원에 대한 접근을 제어하고 프로세스/스레드 간의 협력을 가능하게 하는 핵심 메커니즘입니다. 기본적인 뮤텍스와 세마포어부터 현대적인 고성능 동기화 기법까지 다룹니다.

## 📑 목차

- [[#🔐 기본 동기화 프리미티브]]
- [[#⚡ 고성능 동기화 기법]]
- [[#🎯 조건 변수와 모니터]]
- [[#🚀 Lock-Free 프로그래밍]]
- [[#🔄 읽기-쓰기 락]]
- [[#📊 성능 최적화 기법]]
- [[#💻 TypeScript 구현]]
- [[#🏢 실제 시스템 사례]]
- [[#📈 벤치마크 분석]]
- [[#📚 참고자료]]

---

## 🔐 기본 동기화 프리미티브

### 스핀락 (Spinlock)

> [!note] 스핀락의 특징
> CPU가 락을 획득할 때까지 반복문(busy-wait)을 돌면서 대기하는 동기화 기법. 짧은 임계 구역에 적합.

```typescript
class Spinlock {
    private locked: boolean = false;
    private owner: string | null = null;
    
    // Test-and-Set 원자적 연산 시뮬레이션
    private testAndSet(): boolean {
        const oldValue = this.locked;
        this.locked = true;
        return oldValue;
    }
    
    acquire(threadId: string): void {
        const startTime = Date.now();
        let spinCount = 0;
        
        while (this.testAndSet()) {
            spinCount++;
            
            // 적응형 백오프 (Adaptive Backoff)
            if (spinCount > 1000) {
                // 너무 많이 스핀하면 잠시 양보
                this.yield();
                spinCount = 0;
            }
            
            // 스핀 타임아웃 (무한 대기 방지)
            if (Date.now() - startTime > 1000) {
                throw new Error(`Spinlock timeout for thread ${threadId}`);
            }
        }
        
        this.owner = threadId;
    }
    
    release(threadId: string): void {
        if (this.owner !== threadId) {
            throw new Error(`Thread ${threadId} cannot release lock owned by ${this.owner}`);
        }
        
        this.owner = null;
        this.locked = false; // 원자적으로 해제
    }
    
    tryAcquire(threadId: string): boolean {
        if (!this.testAndSet()) {
            this.owner = threadId;
            return true;
        }
        return false;
    }
    
    isLocked(): boolean {
        return this.locked;
    }
    
    getOwner(): string | null {
        return this.owner;
    }
    
    private yield(): void {
        // 스레드 양보 시뮬레이션 (실제로는 운영체제 호출)
        // 실제 구현: sched_yield() 또는 Thread.yield()
    }
}
```

### 적응형 뮤텍스 (Adaptive Mutex)

```typescript
class AdaptiveMutex {
    private locked: boolean = false;
    private owner: string | null = null;
    private waitQueue: string[] = [];
    private spinThreshold: number = 100;
    private contentionCounter: number = 0;
    
    async acquire(threadId: string): Promise<void> {
        // 첫 번째 시도: 스핀락 방식
        if (this.trySpinAcquire(threadId)) {
            return;
        }
        
        // 두 번째 시도: 대기열에 추가 후 블로킹
        await this.blockingAcquire(threadId);
    }
    
    private trySpinAcquire(threadId: string): boolean {
        let spinCount = 0;
        const maxSpin = this.calculateSpinLimit();
        
        while (spinCount < maxSpin) {
            if (!this.locked) {
                // Compare-and-Swap 원자적 연산 시뮬레이션
                if (this.compareAndSwap(false, true)) {
                    this.owner = threadId;
                    return true;
                }
            }
            
            spinCount++;
            this.adaptiveDelay(spinCount);
        }
        
        return false;
    }
    
    private async blockingAcquire(threadId: string): Promise<void> {
        this.waitQueue.push(threadId);
        this.contentionCounter++;
        
        // 실제 구현에서는 조건 변수나 세마포어 사용
        while (this.locked || this.waitQueue[0] !== threadId) {
            await this.sleep(1); // 1ms 대기
        }
        
        this.waitQueue.shift();
        this.locked = true;
        this.owner = threadId;
    }
    
    private compareAndSwap(expected: boolean, newValue: boolean): boolean {
        if (this.locked === expected) {
            this.locked = newValue;
            return true;
        }
        return false;
    }
    
    private calculateSpinLimit(): number {
        // 경합 수준에 따라 스핀 한계 조정
        const baseLimit = this.spinThreshold;
        const contentionFactor = Math.min(this.contentionCounter / 10, 5);
        
        return Math.max(baseLimit - (contentionFactor * 20), 10);
    }
    
    private adaptiveDelay(spinCount: number): void {
        // 지수적 백오프
        if (spinCount > 50) {
            const delay = Math.min(Math.pow(2, (spinCount - 50) / 10), 100);
            // 실제로는 CPU pause 명령어 사용
        }
    }
    
    release(threadId: string): void {
        if (this.owner !== threadId) {
            throw new Error(`Thread ${threadId} cannot release mutex owned by ${this.owner}`);
        }
        
        this.owner = null;
        this.locked = false;
        
        // 대기 중인 스레드가 있으면 깨우기
        if (this.waitQueue.length > 0) {
            this.wakeupNextWaiter();
        }
        
        this.adjustSpinThreshold();
    }
    
    private wakeupNextWaiter(): void {
        // 실제로는 조건 변수 signal 또는 세마포어 post
        console.log(`Waking up next waiter: ${this.waitQueue[0]}`);
    }
    
    private adjustSpinThreshold(): void {
        // 성능 통계에 따라 스핀 임계값 조정
        if (this.contentionCounter < 5) {
            this.spinThreshold = Math.min(this.spinThreshold + 10, 200);
        } else {
            this.spinThreshold = Math.max(this.spinThreshold - 5, 10);
        }
        
        // 경합 카운터 감소
        this.contentionCounter = Math.max(this.contentionCounter - 1, 0);
    }
    
    private async sleep(ms: number): Promise<void> {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}
```

### 계수 세마포어 (Counting Semaphore)

```typescript
class CountingSemaphore {
    private count: number;
    private readonly maxCount: number;
    private waitQueue: Array<{
        threadId: string;
        resolve: () => void;
        timestamp: number;
    }> = [];
    
    constructor(initialCount: number, maxCount?: number) {
        this.count = initialCount;
        this.maxCount = maxCount || Number.MAX_SAFE_INTEGER;
    }
    
    async acquire(threadId: string, timeout?: number): Promise<boolean> {
        if (this.count > 0) {
            this.count--;
            console.log(`Thread ${threadId} acquired semaphore (count: ${this.count})`);
            return true;
        }
        
        // 대기열에 추가
        return new Promise<boolean>((resolve) => {
            const waitItem = {
                threadId,
                resolve: () => resolve(true),
                timestamp: Date.now()
            };
            
            this.waitQueue.push(waitItem);
            
            // 타임아웃 설정
            if (timeout) {
                setTimeout(() => {
                    const index = this.waitQueue.indexOf(waitItem);
                    if (index !== -1) {
                        this.waitQueue.splice(index, 1);
                        resolve(false);
                    }
                }, timeout);
            }
        });
    }
    
    release(threadId: string): void {
        if (this.count >= this.maxCount) {
            throw new Error('Semaphore count cannot exceed maximum');
        }
        
        this.count++;
        console.log(`Thread ${threadId} released semaphore (count: ${this.count})`);
        
        // 대기 중인 스레드 깨우기 (FIFO 순서)
        if (this.waitQueue.length > 0) {
            const waiter = this.waitQueue.shift()!;
            this.count--; // 바로 할당
            waiter.resolve();
        }
    }
    
    tryAcquire(threadId: string): boolean {
        if (this.count > 0) {
            this.count--;
            console.log(`Thread ${threadId} acquired semaphore (count: ${this.count})`);
            return true;
        }
        return false;
    }
    
    getCount(): number {
        return this.count;
    }
    
    getWaitingCount(): number {
        return this.waitQueue.length;
    }
    
    // 우선순위 기반 세마포어
    async acquireWithPriority(threadId: string, priority: number): Promise<void> {
        if (this.count > 0) {
            this.count--;
            return;
        }
        
        return new Promise<void>((resolve) => {
            const waitItem = {
                threadId,
                priority,
                resolve,
                timestamp: Date.now()
            };
            
            // 우선순위에 따라 정렬하여 삽입
            let insertIndex = 0;
            for (let i = 0; i < this.waitQueue.length; i++) {
                const item = this.waitQueue[i] as any;
                if (item.priority && item.priority < priority) {
                    insertIndex = i + 1;
                } else {
                    break;
                }
            }
            
            this.waitQueue.splice(insertIndex, 0, waitItem as any);
        });
    }
}
```

---

## ⚡ 고성능 동기화 기법

### MCS (Mellor-Crummey Scott) 락

> [!tip] MCS 락의 장점
> NUMA 아키텍처에서 캐시 미스를 최소화하는 큐 기반 락. 각 스레드가 자신만의 노드에서 스핀하여 캐시 라인 경합을 줄임.

```typescript
interface MCSNode {
    next: MCSNode | null;
    waiting: boolean;
    threadId: string;
}

class MCSLock {
    private tail: MCSNode | null = null;
    
    acquire(threadId: string): MCSNode {
        const node: MCSNode = {
            next: null,
            waiting: true,
            threadId
        };
        
        // 원자적으로 tail을 교체하고 이전 값 반환
        const predecessor = this.atomicExchange(node);
        
        if (predecessor !== null) {
            // 이전 노드의 next를 현재 노드로 설정
            predecessor.next = node;
            
            // 자신의 노드에서만 스핀 (캐시 지역성 향상)
            while (node.waiting) {
                // CPU pause 명령어 (실제 구현에서)
                this.cpuPause();
            }
        }
        
        return node;
    }
    
    release(node: MCSNode): void {
        if (node.next === null) {
            // Compare-and-Swap: tail이 현재 노드면 null로 설정
            if (this.compareAndSwapTail(node, null)) {
                // 성공적으로 큐가 비었음
                return;
            }
            
            // 다른 스레드가 큐에 들어오는 중이므로 대기
            while (node.next === null) {
                this.cpuPause();
            }
        }
        
        // 다음 노드를 깨움
        node.next.waiting = false;
    }
    
    private atomicExchange(newNode: MCSNode): MCSNode | null {
        const oldTail = this.tail;
        this.tail = newNode;
        return oldTail;
    }
    
    private compareAndSwapTail(expected: MCSNode, newValue: MCSNode | null): boolean {
        if (this.tail === expected) {
            this.tail = newValue;
            return true;
        }
        return false;
    }
    
    private cpuPause(): void {
        // x86 PAUSE 명령어 시뮬레이션
        // 실제로는 하이퍼스레딩 환경에서 CPU 효율성 증대
    }
}
```

### CLH (Craig, Landin, Hagersten) 락

```typescript
interface CLHNode {
    waiting: boolean;
    threadId: string;
}

class CLHLock {
    private tail: CLHNode;
    
    constructor() {
        // 더미 노드로 초기화
        this.tail = { waiting: false, threadId: 'dummy' };
    }
    
    acquire(threadId: string): CLHNode {
        const myNode: CLHNode = {
            waiting: true,
            threadId
        };
        
        // 원자적으로 tail을 교체하고 이전 노드 반환
        const predecessor = this.atomicExchange(myNode);
        
        // 이전 노드가 해제될 때까지 대기
        while (predecessor.waiting) {
            this.cpuPause();
        }
        
        return predecessor; // 재사용을 위해 이전 노드 반환
    }
    
    release(node: CLHNode): void {
        // 현재 스레드의 대기 상태를 false로 설정
        // 실제로는 tail에서 가리키는 노드의 waiting을 false로 설정
        const currentNode = this.tail;
        currentNode.waiting = false;
    }
    
    private atomicExchange(newNode: CLHNode): CLHNode {
        const oldTail = this.tail;
        this.tail = newNode;
        return oldTail;
    }
    
    private cpuPause(): void {
        // CPU pause 명령어
    }
}
```

### Ticket Lock (공정한 락)

```typescript
class TicketLock {
    private nextTicket: number = 0;    // 다음 발급할 티켓 번호
    private nowServing: number = 0;    // 현재 서비스 중인 티켓 번호
    
    acquire(threadId: string): number {
        // 원자적으로 티켓 번호 할당
        const myTicket = this.atomicIncrement();
        
        console.log(`Thread ${threadId} got ticket ${myTicket}`);
        
        // 자신의 번호가 될 때까지 대기
        while (this.nowServing !== myTicket) {
            this.cpuPause();
        }
        
        console.log(`Thread ${threadId} acquired lock with ticket ${myTicket}`);
        return myTicket;
    }
    
    release(threadId: string, ticket: number): void {
        if (this.nowServing !== ticket) {
            throw new Error(`Invalid ticket ${ticket} for thread ${threadId}`);
        }
        
        // 다음 티켓 서비스 시작
        this.nowServing++;
        console.log(`Thread ${threadId} released lock, now serving ${this.nowServing}`);
    }
    
    private atomicIncrement(): number {
        const current = this.nextTicket;
        this.nextTicket++;
        return current;
    }
    
    private cpuPause(): void {
        // CPU pause
    }
    
    // 락 상태 정보
    getStatus(): { nextTicket: number; nowServing: number; waitingCount: number } {
        return {
            nextTicket: this.nextTicket,
            nowServing: this.nowServing,
            waitingCount: this.nextTicket - this.nowServing - 1
        };
    }
}
```

---

## 🎯 조건 변수와 모니터

### 조건 변수 (Condition Variable)

```typescript
class ConditionVariable {
    private waitQueue: Array<{
        threadId: string;
        resolve: () => void;
        predicate?: () => boolean;
    }> = [];
    
    // 조건 변수에서 대기
    async wait(mutex: AdaptiveMutex, threadId: string, predicate?: () => boolean): Promise<void> {
        // 뮤텍스 해제
        mutex.release(threadId);
        
        return new Promise<void>((resolve) => {
            this.waitQueue.push({
                threadId,
                resolve,
                predicate
            });
        }).then(async () => {
            // 깨어난 후 뮤텍스 다시 획득
            await mutex.acquire(threadId);
        });
    }
    
    // 조건을 만족하는 하나의 대기자 깨우기
    signal(): void {
        if (this.waitQueue.length > 0) {
            const waiter = this.waitQueue.shift()!;
            waiter.resolve();
            console.log(`Signaled thread ${waiter.threadId}`);
        }
    }
    
    // 모든 대기자 깨우기
    broadcast(): void {
        const waiters = [...this.waitQueue];
        this.waitQueue = [];
        
        for (const waiter of waiters) {
            waiter.resolve();
            console.log(`Broadcasted to thread ${waiter.threadId}`);
        }
    }
    
    // 조건자 기반 신호 (spurious wakeup 방지)
    signalIf(condition: () => boolean): void {
        for (let i = 0; i < this.waitQueue.length; i++) {
            const waiter = this.waitQueue[i];
            
            if (!waiter.predicate || waiter.predicate()) {
                this.waitQueue.splice(i, 1);
                waiter.resolve();
                console.log(`Conditionally signaled thread ${waiter.threadId}`);
                break;
            }
        }
    }
    
    // 대기 중인 스레드 수
    getWaitingCount(): number {
        return this.waitQueue.length;
    }
}
```

### 모니터 (Monitor)

```typescript
abstract class Monitor {
    private mutex: AdaptiveMutex = new AdaptiveMutex();
    private conditions: Map<string, ConditionVariable> = new Map();
    
    // 동기화된 메서드 실행
    protected async synchronized<T>(threadId: string, operation: () => Promise<T> | T): Promise<T> {
        await this.mutex.acquire(threadId);
        
        try {
            const result = await Promise.resolve(operation());
            return result;
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    // 조건 변수 생성/획득
    protected getCondition(name: string): ConditionVariable {
        if (!this.conditions.has(name)) {
            this.conditions.set(name, new ConditionVariable());
        }
        return this.conditions.get(name)!;
    }
    
    // 조건 대기
    protected async waitFor(threadId: string, conditionName: string, predicate?: () => boolean): Promise<void> {
        const condition = this.getCondition(conditionName);
        await condition.wait(this.mutex, threadId, predicate);
    }
    
    // 조건 신호
    protected signal(conditionName: string): void {
        const condition = this.conditions.get(conditionName);
        if (condition) {
            condition.signal();
        }
    }
    
    // 조건 브로드캐스트
    protected broadcast(conditionName: string): void {
        const condition = this.conditions.get(conditionName);
        if (condition) {
            condition.broadcast();
        }
    }
}

// 생산자-소비자 모니터 구현
class ProducerConsumerMonitor extends Monitor {
    private buffer: any[] = [];
    private readonly capacity: number;
    
    constructor(capacity: number) {
        super();
        this.capacity = capacity;
    }
    
    async produce(threadId: string, item: any): Promise<void> {
        await this.synchronized(threadId, async () => {
            // 버퍼가 가득 찰 때까지 대기
            while (this.buffer.length >= this.capacity) {
                console.log(`Producer ${threadId} waiting - buffer full`);
                await this.waitFor(threadId, 'notFull', () => this.buffer.length < this.capacity);
            }
            
            this.buffer.push(item);
            console.log(`Producer ${threadId} produced item ${item} (buffer: ${this.buffer.length}/${this.capacity})`);
            
            // 소비자에게 신호
            this.signal('notEmpty');
        });
    }
    
    async consume(threadId: string): Promise<any> {
        return await this.synchronized(threadId, async () => {
            // 버퍼가 비어있을 때까지 대기
            while (this.buffer.length === 0) {
                console.log(`Consumer ${threadId} waiting - buffer empty`);
                await this.waitFor(threadId, 'notEmpty', () => this.buffer.length > 0);
            }
            
            const item = this.buffer.shift();
            console.log(`Consumer ${threadId} consumed item ${item} (buffer: ${this.buffer.length}/${this.capacity})`);
            
            // 생산자에게 신호
            this.signal('notFull');
            
            return item;
        });
    }
    
    getBufferStatus(): { size: number; capacity: number; items: any[] } {
        return {
            size: this.buffer.length,
            capacity: this.capacity,
            items: [...this.buffer]
        };
    }
}
```

---

## 🚀 Lock-Free 프로그래밍

### Compare-and-Swap (CAS) 기반 카운터

```typescript
class LockFreeCounter {
    private value: number = 0;
    private readonly maxRetries: number = 1000;
    
    increment(): number {
        let retries = 0;
        
        while (retries < this.maxRetries) {
            const currentValue = this.value;
            const newValue = currentValue + 1;
            
            // Compare-and-Swap 원자적 연산
            if (this.compareAndSwap(currentValue, newValue)) {
                return newValue;
            }
            
            retries++;
            
            // 지수적 백오프
            if (retries > 10) {
                this.exponentialBackoff(retries);
            }
        }
        
        throw new Error('Maximum retries exceeded for atomic increment');
    }
    
    decrement(): number {
        let retries = 0;
        
        while (retries < this.maxRetries) {
            const currentValue = this.value;
            const newValue = currentValue - 1;
            
            if (this.compareAndSwap(currentValue, newValue)) {
                return newValue;
            }
            
            retries++;
            
            if (retries > 10) {
                this.exponentialBackoff(retries);
            }
        }
        
        throw new Error('Maximum retries exceeded for atomic decrement');
    }
    
    addAndGet(delta: number): number {
        let retries = 0;
        
        while (retries < this.maxRetries) {
            const currentValue = this.value;
            const newValue = currentValue + delta;
            
            if (this.compareAndSwap(currentValue, newValue)) {
                return newValue;
            }
            
            retries++;
        }
        
        throw new Error('Maximum retries exceeded for atomic add');
    }
    
    compareAndSwap(expected: number, newValue: number): boolean {
        if (this.value === expected) {
            this.value = newValue;
            return true;
        }
        return false;
    }
    
    get(): number {
        return this.value; // 읽기는 원자적이라고 가정
    }
    
    private exponentialBackoff(retryCount: number): void {
        const maxDelay = Math.min(Math.pow(2, Math.floor(retryCount / 10)), 100);
        const delay = Math.random() * maxDelay;
        
        // 실제로는 더 정교한 백오프 구현
        // CPU pause나 short sleep 사용
    }
}
```

### Lock-Free 스택

```typescript
interface StackNode<T> {
    data: T;
    next: StackNode<T> | null;
}

class LockFreeStack<T> {
    private head: StackNode<T> | null = null;
    private readonly maxRetries: number = 1000;
    
    push(data: T): void {
        const newNode: StackNode<T> = {
            data,
            next: null
        };
        
        let retries = 0;
        
        while (retries < this.maxRetries) {
            const currentHead = this.head;
            newNode.next = currentHead;
            
            // CAS로 head 업데이트
            if (this.compareAndSwapHead(currentHead, newNode)) {
                return;
            }
            
            retries++;
        }
        
        throw new Error('Maximum retries exceeded for push operation');
    }
    
    pop(): T | null {
        let retries = 0;
        
        while (retries < this.maxRetries) {
            const currentHead = this.head;
            
            if (currentHead === null) {
                return null; // 스택이 비어있음
            }
            
            const nextHead = currentHead.next;
            
            // CAS로 head 업데이트
            if (this.compareAndSwapHead(currentHead, nextHead)) {
                return currentHead.data;
            }
            
            retries++;
        }
        
        throw new Error('Maximum retries exceeded for pop operation');
    }
    
    peek(): T | null {
        const currentHead = this.head;
        return currentHead ? currentHead.data : null;
    }
    
    isEmpty(): boolean {
        return this.head === null;
    }
    
    private compareAndSwapHead(expected: StackNode<T> | null, newValue: StackNode<T> | null): boolean {
        if (this.head === expected) {
            this.head = newValue;
            return true;
        }
        return false;
    }
    
    // ABA 문제 해결을 위한 버전 기반 스택
    pushWithVersion(data: T): void {
        const newNode: StackNode<T> & { version: number } = {
            data,
            next: null,
            version: this.generateVersion()
        };
        
        // 실제 구현에서는 더 복잡한 ABA 방지 로직 필요
    }
    
    private generateVersion(): number {
        return Date.now() + Math.random();
    }
}
```

### Lock-Free 큐 (Michael & Scott Algorithm)

```typescript
interface QueueNode<T> {
    data: T | null;
    next: QueueNode<T> | null;
}

class LockFreeQueue<T> {
    private head: QueueNode<T>;
    private tail: QueueNode<T>;
    
    constructor() {
        // 더미 노드로 초기화
        const dummyNode: QueueNode<T> = {
            data: null,
            next: null
        };
        
        this.head = dummyNode;
        this.tail = dummyNode;
    }
    
    enqueue(data: T): void {
        const newNode: QueueNode<T> = {
            data,
            next: null
        };
        
        while (true) {
            const tail = this.tail;
            const next = tail.next;
            
            if (tail === this.tail) { // tail이 여전히 유효한지 확인
                if (next === null) {
                    // tail이 실제 마지막 노드인 경우
                    if (this.compareAndSwapNext(tail, null, newNode)) {
                        break; // 성공적으로 새 노드 연결
                    }
                } else {
                    // tail이 뒤쳐진 경우, tail 업데이트 시도
                    this.compareAndSwapTail(tail, next);
                }
            }
        }
        
        // tail을 새 노드로 이동
        this.compareAndSwapTail(this.tail, newNode);
    }
    
    dequeue(): T | null {
        while (true) {
            const head = this.head;
            const tail = this.tail;
            const next = head.next;
            
            if (head === this.head) { // head가 여전히 유효한지 확인
                if (head === tail) {
                    if (next === null) {
                        return null; // 큐가 비어있음
                    }
                    
                    // tail이 뒤쳐진 경우, tail 업데이트 시도
                    this.compareAndSwapTail(tail, next);
                } else {
                    if (next === null) {
                        continue; // 다시 시도
                    }
                    
                    const data = next.data;
                    
                    // head를 다음 노드로 이동
                    if (this.compareAndSwapHead(head, next)) {
                        return data;
                    }
                }
            }
        }
    }
    
    isEmpty(): boolean {
        const head = this.head;
        const tail = this.tail;
        return (head === tail) && (head.next === null);
    }
    
    private compareAndSwapHead(expected: QueueNode<T>, newValue: QueueNode<T>): boolean {
        if (this.head === expected) {
            this.head = newValue;
            return true;
        }
        return false;
    }
    
    private compareAndSwapTail(expected: QueueNode<T>, newValue: QueueNode<T>): boolean {
        if (this.tail === expected) {
            this.tail = newValue;
            return true;
        }
        return false;
    }
    
    private compareAndSwapNext(node: QueueNode<T>, expected: QueueNode<T> | null, 
                             newValue: QueueNode<T>): boolean {
        if (node.next === expected) {
            node.next = newValue;
            return true;
        }
        return false;
    }
}
```

---

## 🔄 읽기-쓰기 락

### 공정한 읽기-쓰기 락

```typescript
class FairReadWriteLock {
    private readers: number = 0;
    private writers: number = 0;
    private waitingWriters: number = 0;
    
    private readCondition: ConditionVariable = new ConditionVariable();
    private writeCondition: ConditionVariable = new ConditionVariable();
    private mutex: AdaptiveMutex = new AdaptiveMutex();
    
    async acquireReadLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            // 쓰기 대기자가 있으면 읽기도 대기 (공정성 보장)
            while (this.writers > 0 || this.waitingWriters > 0) {
                console.log(`Reader ${threadId} waiting due to writer priority`);
                await this.readCondition.wait(this.mutex, threadId);
            }
            
            this.readers++;
            console.log(`Reader ${threadId} acquired read lock (${this.readers} readers)`);
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    async releaseReadLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            this.readers--;
            console.log(`Reader ${threadId} released read lock (${this.readers} readers)`);
            
            // 마지막 읽기자가 해제되면 쓰기자 깨우기
            if (this.readers === 0 && this.waitingWriters > 0) {
                this.writeCondition.signal();
            }
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    async acquireWriteLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            this.waitingWriters++;
            
            // 읽기자나 다른 쓰기자가 있으면 대기
            while (this.readers > 0 || this.writers > 0) {
                console.log(`Writer ${threadId} waiting (${this.readers} readers, ${this.writers} writers)`);
                await this.writeCondition.wait(this.mutex, threadId);
            }
            
            this.waitingWriters--;
            this.writers++;
            console.log(`Writer ${threadId} acquired write lock`);
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    async releaseWriteLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            this.writers--;
            console.log(`Writer ${threadId} released write lock`);
            
            if (this.waitingWriters > 0) {
                // 쓰기 우선 정책
                this.writeCondition.signal();
            } else {
                // 모든 읽기자 깨우기
                this.readCondition.broadcast();
            }
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    // 읽기 우선 버전
    async acquireReadLockReaderPreferred(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            // 쓰기자만 확인 (읽기 우선)
            while (this.writers > 0) {
                await this.readCondition.wait(this.mutex, threadId);
            }
            
            this.readers++;
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    // 상태 정보 조회
    getStatus(): { readers: number; writers: number; waitingWriters: number } {
        return {
            readers: this.readers,
            writers: this.writers,
            waitingWriters: this.waitingWriters
        };
    }
    
    // 타임아웃 기반 락 획득
    async tryAcquireReadLock(threadId: string, timeoutMs: number): Promise<boolean> {
        const startTime = Date.now();
        
        while (Date.now() - startTime < timeoutMs) {
            await this.mutex.acquire(threadId);
            
            try {
                if (this.writers === 0 && this.waitingWriters === 0) {
                    this.readers++;
                    return true;
                }
            } finally {
                this.mutex.release(threadId);
            }
            
            // 짧은 대기 후 재시도
            await this.sleep(10);
        }
        
        return false; // 타임아웃
    }
    
    private async sleep(ms: number): Promise<void> {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}
```

### 업그레이드 가능한 읽기-쓰기 락

```typescript
class UpgradeableReadWriteLock {
    private readers: number = 0;
    private writers: number = 0;
    private upgraders: number = 0;
    private mutex: AdaptiveMutex = new AdaptiveMutex();
    
    private readCondition: ConditionVariable = new ConditionVariable();
    private writeCondition: ConditionVariable = new ConditionVariable();
    private upgradeCondition: ConditionVariable = new ConditionVariable();
    
    async acquireReadLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            while (this.writers > 0 || this.upgraders > 0) {
                await this.readCondition.wait(this.mutex, threadId);
            }
            
            this.readers++;
            console.log(`Reader ${threadId} acquired read lock`);
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    async acquireUpgradeableLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            while (this.writers > 0 || this.upgraders > 0) {
                await this.upgradeCondition.wait(this.mutex, threadId);
            }
            
            this.upgraders++;
            console.log(`Thread ${threadId} acquired upgradeable lock`);
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    async upgradeToWriteLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            if (this.upgraders === 0) {
                throw new Error(`Thread ${threadId} does not hold upgradeable lock`);
            }
            
            // 다른 읽기자들이 모두 해제될 때까지 대기
            while (this.readers > 0) {
                console.log(`Thread ${threadId} waiting to upgrade (${this.readers} readers)`);
                await this.writeCondition.wait(this.mutex, threadId);
            }
            
            this.upgraders--;
            this.writers++;
            console.log(`Thread ${threadId} upgraded to write lock`);
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    async downgradeToReadLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            if (this.writers === 0) {
                throw new Error(`Thread ${threadId} does not hold write lock`);
            }
            
            this.writers--;
            this.readers++;
            
            console.log(`Thread ${threadId} downgraded to read lock`);
            
            // 대기 중인 다른 읽기자들 깨우기
            this.readCondition.broadcast();
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    async releaseReadLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            this.readers--;
            console.log(`Reader ${threadId} released read lock`);
            
            if (this.readers === 0 && this.upgraders > 0) {
                this.writeCondition.signal();
            }
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    async releaseWriteLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            this.writers--;
            console.log(`Writer ${threadId} released write lock`);
            
            // 모든 대기자 깨우기
            this.readCondition.broadcast();
            this.upgradeCondition.signal();
        } finally {
            this.mutex.release(threadId);
        }
    }
    
    async releaseUpgradeableLock(threadId: string): Promise<void> {
        await this.mutex.acquire(threadId);
        
        try {
            this.upgraders--;
            console.log(`Thread ${threadId} released upgradeable lock`);
            
            this.upgradeCondition.signal();
        } finally {
            this.mutex.release(threadId);
        }
    }
}
```

---

## 📊 성능 최적화 기법

### 적응형 백오프 전략

```typescript
class AdaptiveBackoffStrategy {
    private readonly minDelay: number = 1;
    private readonly maxDelay: number = 1000;
    private readonly backoffFactor: number = 2;
    private currentDelay: number = 1;
    private successCount: number = 0;
    private failureCount: number = 0;
    
    async backoff(): Promise<void> {
        // 지수적 백오프 with jitter
        const jitter = Math.random() * this.currentDelay * 0.1;
        const delay = Math.min(this.currentDelay + jitter, this.maxDelay);
        
        await this.sleep(delay);
        
        // 다음 백오프 지연 시간 조정
        this.currentDelay = Math.min(
            this.currentDelay * this.backoffFactor,
            this.maxDelay
        );
        
        this.failureCount++;
    }
    
    onSuccess(): void {
        this.successCount++;
        
        // 성공률이 높으면 백오프 감소
        if (this.successCount > this.failureCount * 2) {
            this.currentDelay = Math.max(
                this.currentDelay / this.backoffFactor,
                this.minDelay
            );
        }
        
        // 주기적으로 통계 리셋
        if (this.successCount + this.failureCount > 1000) {
            this.resetStats();
        }
    }
    
    private resetStats(): void {
        this.successCount = 0;
        this.failureCount = 0;
        this.currentDelay = this.minDelay;
    }
    
    private sleep(ms: number): Promise<void> {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
    
    getStats(): { currentDelay: number; successRate: number } {
        const total = this.successCount + this.failureCount;
        const successRate = total > 0 ? this.successCount / total : 0;
        
        return {
            currentDelay: this.currentDelay,
            successRate
        };
    }
}
```

### 계층적 락 관리자

```typescript
class HierarchicalLockManager {
    private locks: Map<string, any> = new Map();
    private lockHierarchy: Map<string, number> = new Map();
    private threadLocks: Map<string, Set<string>> = new Map();
    
    constructor() {
        // 락 계층 순서 정의 (데드락 방지)
        this.lockHierarchy.set('global', 1);
        this.lockHierarchy.set('database', 2);
        this.lockHierarchy.set('table', 3);
        this.lockHierarchy.set('row', 4);
    }
    
    async acquireLock(threadId: string, lockId: string, lockType: string): Promise<boolean> {
        const lockLevel = this.lockHierarchy.get(this.getLockCategory(lockId));
        if (!lockLevel) {
            throw new Error(`Unknown lock category for ${lockId}`);
        }
        
        // 계층 순서 검증
        if (!this.validateLockOrder(threadId, lockLevel)) {
            throw new Error(`Lock order violation: cannot acquire ${lockId} after higher level locks`);
        }
        
        // 실제 락 획득
        const lock = this.getOrCreateLock(lockId, lockType);
        const acquired = await this.tryAcquireLock(lock, threadId);
        
        if (acquired) {
            this.recordLockAcquisition(threadId, lockId);
        }
        
        return acquired;
    }
    
    private validateLockOrder(threadId: string, newLockLevel: number): boolean {
        const threadLocks = this.threadLocks.get(threadId) || new Set();
        
        for (const heldLock of threadLocks) {
            const heldLevel = this.lockHierarchy.get(this.getLockCategory(heldLock));
            if (heldLevel && heldLevel >= newLockLevel) {
                return false; // 순서 위반
            }
        }
        
        return true;
    }
    
    private getLockCategory(lockId: string): string {
        // 락 ID에서 카테고리 추출
        const parts = lockId.split(':');
        return parts[0];
    }
    
    private getOrCreateLock(lockId: string, lockType: string): any {
        if (!this.locks.has(lockId)) {
            switch (lockType) {
                case 'mutex':
                    this.locks.set(lockId, new AdaptiveMutex());
                    break;
                case 'rwlock':
                    this.locks.set(lockId, new FairReadWriteLock());
                    break;
                case 'spinlock':
                    this.locks.set(lockId, new Spinlock());
                    break;
                default:
                    throw new Error(`Unknown lock type: ${lockType}`);
            }
        }
        
        return this.locks.get(lockId);
    }
    
    private async tryAcquireLock(lock: any, threadId: string): Promise<boolean> {
        try {
            if (lock instanceof AdaptiveMutex) {
                await lock.acquire(threadId);
            } else if (lock instanceof FairReadWriteLock) {
                await lock.acquireReadLock(threadId);
            } else if (lock instanceof Spinlock) {
                lock.acquire(threadId);
            }
            return true;
        } catch (error) {
            console.error(`Failed to acquire lock: ${error}`);
            return false;
        }
    }
    
    private recordLockAcquisition(threadId: string, lockId: string): void {
        if (!this.threadLocks.has(threadId)) {
            this.threadLocks.set(threadId, new Set());
        }
        this.threadLocks.get(threadId)!.add(lockId);
    }
    
    async releaseLock(threadId: string, lockId: string): Promise<void> {
        const lock = this.locks.get(lockId);
        if (!lock) {
            throw new Error(`Lock ${lockId} not found`);
        }
        
        // 락 해제
        if (lock instanceof AdaptiveMutex) {
            lock.release(threadId);
        } else if (lock instanceof FairReadWriteLock) {
            await lock.releaseReadLock(threadId);
        } else if (lock instanceof Spinlock) {
            lock.release(threadId);
        }
        
        // 기록 제거
        const threadLocks = this.threadLocks.get(threadId);
        if (threadLocks) {
            threadLocks.delete(lockId);
        }
    }
    
    // 스레드가 보유한 모든 락 해제
    async releaseAllLocks(threadId: string): Promise<void> {
        const threadLocks = this.threadLocks.get(threadId);
        if (!threadLocks) return;
        
        // 역순으로 해제 (계층 순서의 반대)
        const lockIds = Array.from(threadLocks).sort((a, b) => {
            const levelA = this.lockHierarchy.get(this.getLockCategory(a)) || 0;
            const levelB = this.lockHierarchy.get(this.getLockCategory(b)) || 0;
            return levelB - levelA; // 내림차순
        });
        
        for (const lockId of lockIds) {
            await this.releaseLock(threadId, lockId);
        }
    }
    
    // 락 통계 조회
    getLockStatistics(): any {
        const stats = {
            totalLocks: this.locks.size,
            activeLocks: 0,
            threadLockCounts: new Map<string, number>()
        };
        
        for (const [threadId, locks] of this.threadLocks) {
            stats.threadLockCounts.set(threadId, locks.size);
            stats.activeLocks += locks.size;
        }
        
        return stats;
    }
}
```

---

## 💻 TypeScript 구현

### 통합 동기화 라이브러리

```typescript
// 모든 동기화 프리미티브를 포함하는 통합 라이브러리
class SynchronizationLibrary {
    private primitives: Map<string, any> = new Map();
    private performanceMonitor: PerformanceMonitor;
    
    constructor() {
        this.performanceMonitor = new PerformanceMonitor();
    }
    
    // 다양한 동기화 프리미티브 팩토리
    createMutex(name: string, type: 'adaptive' | 'spinlock' | 'ticket' = 'adaptive'): any {
        let mutex;
        
        switch (type) {
            case 'adaptive':
                mutex = new AdaptiveMutex();
                break;
            case 'spinlock':
                mutex = new Spinlock();
                break;
            case 'ticket':
                mutex = new TicketLock();
                break;
            default:
                throw new Error(`Unknown mutex type: ${type}`);
        }
        
        this.primitives.set(name, {
            type: 'mutex',
            subtype: type,
            instance: mutex,
            stats: this.performanceMonitor.createStats(name)
        });
        
        return this.createInstrumentedWrapper(name, mutex);
    }
    
    createReadWriteLock(name: string, fair: boolean = true): FairReadWriteLock {
        const rwlock = new FairReadWriteLock();
        
        this.primitives.set(name, {
            type: 'rwlock',
            subtype: fair ? 'fair' : 'unfair',
            instance: rwlock,
            stats: this.performanceMonitor.createStats(name)
        });
        
        return this.createInstrumentedWrapper(name, rwlock);
    }
    
    createSemaphore(name: string, permits: number): CountingSemaphore {
        const semaphore = new CountingSemaphore(permits);
        
        this.primitives.set(name, {
            type: 'semaphore',
            instance: semaphore,
            stats: this.performanceMonitor.createStats(name)
        });
        
        return this.createInstrumentedWrapper(name, semaphore);
    }
    
    createConditionVariable(name: string): ConditionVariable {
        const condition = new ConditionVariable();
        
        this.primitives.set(name, {
            type: 'condition',
            instance: condition,
            stats: this.performanceMonitor.createStats(name)
        });
        
        return this.createInstrumentedWrapper(name, condition);
    }
    
    createLockFreeCounter(name: string): LockFreeCounter {
        const counter = new LockFreeCounter();
        
        this.primitives.set(name, {
            type: 'lockfree',
            subtype: 'counter',
            instance: counter,
            stats: this.performanceMonitor.createStats(name)
        });
        
        return this.createInstrumentedWrapper(name, counter);
    }
    
    // 성능 모니터링 래퍼 생성
    private createInstrumentedWrapper(name: string, instance: any): any {
        const stats = this.performanceMonitor.getStats(name);
        const wrapper = Object.create(instance);
        
        // 모든 메서드를 래핑하여 성능 측정
        const methodNames = Object.getOwnPropertyNames(Object.getPrototypeOf(instance));
        
        for (const methodName of methodNames) {
            if (typeof instance[methodName] === 'function' && methodName !== 'constructor') {
                wrapper[methodName] = this.instrumentMethod(instance, methodName, stats);
            }
        }
        
        return wrapper;
    }
    
    private instrumentMethod(instance: any, methodName: string, stats: any): Function {
        const originalMethod = instance[methodName].bind(instance);
        
        return async function(...args: any[]) {
            const startTime = Date.now();
            stats.callCount++;
            
            try {
                const result = await originalMethod(...args);
                const endTime = Date.now();
                
                stats.totalTime += (endTime - startTime);
                stats.successCount++;
                
                return result;
            } catch (error) {
                stats.errorCount++;
                throw error;
            }
        };
    }
    
    // 성능 통계 조회
    getPerformanceReport(): any {
        const report: any = {
            timestamp: new Date().toISOString(),
            primitives: {}
        };
        
        for (const [name, primitive] of this.primitives) {
            const stats = this.performanceMonitor.getStats(name);
            
            report.primitives[name] = {
                type: primitive.type,
                subtype: primitive.subtype || 'default',
                callCount: stats.callCount,
                successCount: stats.successCount,
                errorCount: stats.errorCount,
                averageTime: stats.callCount > 0 ? stats.totalTime / stats.callCount : 0,
                successRate: stats.callCount > 0 ? stats.successCount / stats.callCount : 0
            };
        }
        
        return report;
    }
    
    // 데드락 탐지
    detectPotentialDeadlocks(): string[] {
        const warnings: string[] = [];
        
        // 간단한 휴리스틱 기반 탐지
        for (const [name, primitive] of this.primitives) {
            const stats = this.performanceMonitor.getStats(name);
            
            // 에러율이 높은 경우
            if (stats.errorCount > stats.successCount * 0.1) {
                warnings.push(`High error rate detected for ${name}: ${stats.errorCount}/${stats.callCount}`);
            }
            
            // 평균 대기 시간이 비정상적으로 긴 경우
            const avgTime = stats.callCount > 0 ? stats.totalTime / stats.callCount : 0;
            if (avgTime > 1000) { // 1초 이상
                warnings.push(`Long average wait time for ${name}: ${avgTime.toFixed(2)}ms`);
            }
        }
        
        return warnings;
    }
    
    // 리소스 정리
    cleanup(): void {
        this.primitives.clear();
        this.performanceMonitor.reset();
    }
}

class PerformanceMonitor {
    private stats: Map<string, any> = new Map();
    
    createStats(name: string): any {
        const stats = {
            callCount: 0,
            successCount: 0,
            errorCount: 0,
            totalTime: 0
        };
        
        this.stats.set(name, stats);
        return stats;
    }
    
    getStats(name: string): any {
        return this.stats.get(name);
    }
    
    reset(): void {
        this.stats.clear();
    }
}
```

---

## 🏢 실제 시스템 사례

### Java의 java.util.concurrent 패키지 시뮬레이션

```typescript
// Java의 StampedLock 구현 시뮬레이션
class StampedLock {
    private static readonly WBIT: number = 1 << 7;    // 쓰기 비트
    private static readonly RBITS: number = this.WBIT - 1;  // 읽기 비트들
    private static readonly RFULL: number = this.RBITS - 1; // 최대 읽기자 수
    
    private state: number = 0;
    private readerOverflow: number = 0;
    
    // 낙관적 읽기 시도
    tryOptimisticRead(): number {
        const stamp = this.state;
        return (stamp & StampedLock.WBIT) === 0 ? stamp : 0;
    }
    
    // 낙관적 읽기 검증
    validate(stamp: number): boolean {
        return (stamp & StampedLock.WBIT) === 0 && stamp === this.state;
    }
    
    // 읽기 락 획득
    readLock(): number {
        let s: number;
        let next: number;
        
        // CAS로 읽기자 수 증가
        while (((s = this.state) & StampedLock.WBIT) === 0) {
            if ((s & StampedLock.RFULL) === StampedLock.RFULL) {
                // 오버플로 처리
                return this.acquireRead(false, 0);
            }
            
            next = s + 1;
            if (this.compareAndSwapState(s, next)) {
                return next;
            }
        }
        
        return this.acquireRead(false, 0);
    }
    
    // 쓰기 락 획득
    writeLock(): number {
        let s: number;
        
        while (((s = this.state) & StampedLock.WBIT) === 0) {
            if (this.compareAndSwapState(s, s | StampedLock.WBIT)) {
                return s | StampedLock.WBIT;
            }
        }
        
        return this.acquireWrite(false, 0);
    }
    
    // 읽기 락 해제
    unlockRead(stamp: number): void {
        let s: number;
        
        if (((s = this.state) & StampedLock.WBIT) !== 0 || 
            (stamp & StampedLock.WBIT) !== 0) {
            throw new Error('Not read locked');
        }
        
        if ((s & StampedLock.RFULL) === 0) {
            if (this.compareAndSwapState(s, s - 1)) {
                return;
            }
        }
        
        this.releaseRead(stamp);
    }
    
    // 쓰기 락 해제
    unlockWrite(stamp: number): void {
        if ((stamp & StampedLock.WBIT) === 0 || stamp !== this.state) {
            throw new Error('Not write locked');
        }
        
        this.unlockWriteInternal(stamp);
    }
    
    private acquireRead(interruptible: boolean, deadline: number): number {
        // 복잡한 읽기 락 획득 로직
        // 실제 구현에서는 큐 관리와 스핀/블로킹 전략 포함
        return this.state + 1;
    }
    
    private acquireWrite(interruptible: boolean, deadline: number): number {
        // 복잡한 쓰기 락 획득 로직
        return this.state | StampedLock.WBIT;
    }
    
    private releaseRead(stamp: number): void {
        // 읽기 락 해제 로직
        this.state--;
    }
    
    private unlockWriteInternal(stamp: number): void {
        // 쓰기 락 해제 로직
        this.state = stamp - StampedLock.WBIT;
    }
    
    private compareAndSwapState(expected: number, newValue: number): boolean {
        if (this.state === expected) {
            this.state = newValue;
            return true;
        }
        return false;
    }
}
```

### Linux Futex 시뮬레이션

```typescript
// Linux의 Fast Userspace Mutex (Futex) 시뮬레이션
class Futex {
    private static readonly FUTEX_WAIT: number = 0;
    private static readonly FUTEX_WAKE: number = 1;
    private static readonly FUTEX_PRIVATE_FLAG: number = 128;
    
    private value: number = 0;
    private waiters: Map<number, Array<{ resolve: () => void; threadId: string }>> = new Map();
    
    // 원자적 값 비교 및 대기
    async wait(expectedValue: number, threadId: string, timeoutMs?: number): Promise<boolean> {
        // 값이 예상과 다르면 즉시 반환
        if (this.value !== expectedValue) {
            return false;
        }
        
        // 대기 큐에 추가
        if (!this.waiters.has(expectedValue)) {
            this.waiters.set(expectedValue, []);
        }
        
        return new Promise<boolean>((resolve) => {
            const waiter = {
                resolve: () => resolve(true),
                threadId
            };
            
            this.waiters.get(expectedValue)!.push(waiter);
            
            // 타임아웃 설정
            if (timeoutMs) {
                setTimeout(() => {
                    const waitList = this.waiters.get(expectedValue);
                    if (waitList) {
                        const index = waitList.indexOf(waiter);
                        if (index !== -1) {
                            waitList.splice(index, 1);
                            resolve(false); // 타임아웃
                        }
                    }
                }, timeoutMs);
            }
        });
    }
    
    // 대기자 깨우기
    wake(maxWakeups: number = 1): number {
        let wokenUp = 0;
        
        for (const [value, waitList] of this.waiters) {
            while (waitList.length > 0 && wokenUp < maxWakeups) {
                const waiter = waitList.shift()!;
                waiter.resolve();
                wokenUp++;
            }
            
            if (waitList.length === 0) {
                this.waiters.delete(value);
            }
            
            if (wokenUp >= maxWakeups) {
                break;
            }
        }
        
        return wokenUp;
    }
    
    // 원자적 값 설정 및 깨우기
    wakeAll(): number {
        return this.wake(Number.MAX_SAFE_INTEGER);
    }
    
    // 원자적 Compare-and-Swap
    compareAndSwap(expected: number, newValue: number): boolean {
        if (this.value === expected) {
            this.value = newValue;
            return true;
        }
        return false;
    }
    
    // 원자적 값 증가
    incrementAndGet(): number {
        return ++this.value;
    }
    
    // 원자적 값 감소
    decrementAndGet(): number {
        return --this.value;
    }
    
    getValue(): number {
        return this.value;
    }
    
    setValue(newValue: number): void {
        this.value = newValue;
    }
    
    // Futex 기반 뮤텍스 구현
    static createMutex(): FutexMutex {
        return new FutexMutex();
    }
}

class FutexMutex {
    private futex: Futex = new Futex();
    private static readonly UNLOCKED: number = 0;
    private static readonly LOCKED_NO_WAITERS: number = 1;
    private static readonly LOCKED_WITH_WAITERS: number = 2;
    
    async lock(threadId: string): Promise<void> {
        // Fast path: 락이 비어있으면 바로 획득
        if (this.futex.compareAndSwap(FutexMutex.UNLOCKED, FutexMutex.LOCKED_NO_WAITERS)) {
            return;
        }
        
        // Slow path: 경합이 있는 경우
        let currentState = this.futex.getValue();
        
        while (true) {
            if (currentState === FutexMutex.LOCKED_WITH_WAITERS ||
                this.futex.compareAndSwap(FutexMutex.LOCKED_NO_WAITERS, FutexMutex.LOCKED_WITH_WAITERS)) {
                
                // 대기자가 있음을 표시하고 대기
                await this.futex.wait(FutexMutex.LOCKED_WITH_WAITERS, threadId);
            }
            
            // 락 획득 재시도
            currentState = this.futex.getValue();
            if (currentState === FutexMutex.UNLOCKED &&
                this.futex.compareAndSwap(FutexMutex.UNLOCKED, FutexMutex.LOCKED_WITH_WAITERS)) {
                break;
            }
        }
    }
    
    unlock(): void {
        const prevState = this.futex.decrementAndGet();
        
        if (prevState === FutexMutex.LOCKED_NO_WAITERS) {
            // 대기자가 없으면 그냥 해제
            return;
        }
        
        if (prevState === FutexMutex.LOCKED_WITH_WAITERS - 1) {
            // 대기자가 있을 수 있으므로 깨우기
            this.futex.setValue(FutexMutex.UNLOCKED);
            this.futex.wake(1);
        }
    }
    
    tryLock(): boolean {
        return this.futex.compareAndSwap(FutexMutex.UNLOCKED, FutexMutex.LOCKED_NO_WAITERS);
    }
    
    isLocked(): boolean {
        return this.futex.getValue() !== FutexMutex.UNLOCKED;
    }
}
```

---

## 📈 벤치마크 분석

### 동기화 프리미티브 성능 벤치마크

```typescript
class SynchronizationBenchmark {
    private results: Map<string, BenchmarkResult[]> = new Map();
    
    async runComprehensiveBenchmark(): Promise<void> {
        console.log('🚀 동기화 프리미티브 성능 벤치마크 시작\n');
        
        const workloads = [
            { name: 'Low Contention', threads: 2, iterations: 1000 },
            { name: 'Medium Contention', threads: 8, iterations: 1000 },
            { name: 'High Contention', threads: 16, iterations: 1000 }
        ];
        
        const primitives = [
            'Spinlock',
            'AdaptiveMutex',
            'TicketLock',
            'MCSLock',
            'FairReadWriteLock',
            'StampedLock'
        ];
        
        for (const workload of workloads) {
            console.log(`📊 ${workload.name} 벤치마크 (${workload.threads} 스레드, ${workload.iterations} 반복):`);
            
            for (const primitive of primitives) {
                const result = await this.benchmarkPrimitive(primitive, workload);
                this.recordResult(primitive, workload.name, result);
                
                console.log(`   ${primitive.padEnd(20)} - ${result.throughput.toFixed(0)} ops/sec, ` +
                          `${result.avgLatency.toFixed(2)} ms avg, ` +
                          `${result.p99Latency.toFixed(2)} ms p99`);
            }
            console.log('');
        }
        
        this.printSummaryAnalysis();
    }
    
    private async benchmarkPrimitive(primitive: string, workload: any): Promise<BenchmarkResult> {
        const startTime = Date.now();
        const latencies: number[] = [];
        let completedOps = 0;
        
        // 동기화 프리미티브 생성
        const syncPrimitive = this.createPrimitive(primitive);
        
        // 워커 스레드들 시뮬레이션
        const promises = [];
        
        for (let t = 0; t < workload.threads; t++) {
            const threadId = `thread-${t}`;
            
            promises.push(this.runWorkerThread(
                threadId,
                syncPrimitive,
                Math.floor(workload.iterations / workload.threads),
                latencies
            ));
        }
        
        await Promise.all(promises);
        
        const endTime = Date.now();
        const totalTime = (endTime - startTime) / 1000; // seconds
        
        // 통계 계산
        latencies.sort((a, b) => a - b);
        const avgLatency = latencies.reduce((sum, lat) => sum + lat, 0) / latencies.length;
        const p99Index = Math.floor(latencies.length * 0.99);
        const p99Latency = latencies[p99Index] || 0;
        
        return {
            primitive,
            throughput: workload.iterations / totalTime,
            avgLatency,
            p99Latency,
            totalTime,
            completedOps: workload.iterations
        };
    }
    
    private createPrimitive(type: string): any {
        switch (type) {
            case 'Spinlock':
                return new Spinlock();
            case 'AdaptiveMutex':
                return new AdaptiveMutex();
            case 'TicketLock':
                return new TicketLock();
            case 'MCSLock':
                return new MCSLock();
            case 'FairReadWriteLock':
                return new FairReadWriteLock();
            case 'StampedLock':
                return new StampedLock();
            default:
                throw new Error(`Unknown primitive: ${type}`);
        }
    }
    
    private async runWorkerThread(
        threadId: string,
        syncPrimitive: any,
        iterations: number,
        latencies: number[]
    ): Promise<void> {
        
        for (let i = 0; i < iterations; i++) {
            const startTime = Date.now();
            
            try {
                // 크리티컬 섹션 시뮬레이션
                await this.performCriticalSection(threadId, syncPrimitive);
            } catch (error) {
                console.error(`Error in ${threadId}: ${error}`);
            }
            
            const endTime = Date.now();
            latencies.push(endTime - startTime);
        }
    }
    
    private async performCriticalSection(threadId: string, syncPrimitive: any): Promise<void> {
        if (syncPrimitive instanceof Spinlock) {
            syncPrimitive.acquire(threadId);
            await this.simulateWork(1); // 1ms 작업 시뮬레이션
            syncPrimitive.release(threadId);
            
        } else if (syncPrimitive instanceof AdaptiveMutex) {
            await syncPrimitive.acquire(threadId);
            await this.simulateWork(1);
            syncPrimitive.release(threadId);
            
        } else if (syncPrimitive instanceof TicketLock) {
            const ticket = syncPrimitive.acquire(threadId);
            await this.simulateWork(1);
            syncPrimitive.release(threadId, ticket);
            
        } else if (syncPrimitive instanceof MCSLock) {
            const node = syncPrimitive.acquire(threadId);
            await this.simulateWork(1);
            syncPrimitive.release(node);
            
        } else if (syncPrimitive instanceof FairReadWriteLock) {
            if (Math.random() < 0.7) { // 70% 읽기 작업
                await syncPrimitive.acquireReadLock(threadId);
                await this.simulateWork(1);
                await syncPrimitive.releaseReadLock(threadId);
            } else { // 30% 쓰기 작업
                await syncPrimitive.acquireWriteLock(threadId);
                await this.simulateWork(2); // 쓰기는 더 오래
                await syncPrimitive.releaseWriteLock(threadId);
            }
            
        } else if (syncPrimitive instanceof StampedLock) {
            const stamp = syncPrimitive.tryOptimisticRead();
            if (stamp !== 0) {
                await this.simulateWork(1);
                if (syncPrimitive.validate(stamp)) {
                    return; // 낙관적 읽기 성공
                }
            }
            
            // 낙관적 읽기 실패시 일반 읽기 락
            const readStamp = syncPrimitive.readLock();
            await this.simulateWork(1);
            syncPrimitive.unlockRead(readStamp);
        }
    }
    
    private async simulateWork(durationMs: number): Promise<void> {
        // CPU 집약적 작업 시뮬레이션
        const start = Date.now();
        while (Date.now() - start < durationMs) {
            // Busy waiting to simulate CPU work
            Math.random();
        }
    }
    
    private recordResult(primitive: string, workload: string, result: BenchmarkResult): void {
        if (!this.results.has(primitive)) {
            this.results.set(primitive, []);
        }
        this.results.get(primitive)!.push({ ...result, workload });
    }
    
    private printSummaryAnalysis(): void {
        console.log('📈 벤치마크 분석 요약:');
        console.log('='.repeat(60));
        
        // 처리량 기준 순위
        console.log('\n🏆 처리량 순위 (High Contention):');
        const highContentionResults = [];
        
        for (const [primitive, results] of this.results) {
            const highContention = results.find(r => r.workload === 'High Contention');
            if (highContention) {
                highContentionResults.push({
                    primitive,
                    throughput: highContention.throughput
                });
            }
        }
        
        highContentionResults
            .sort((a, b) => b.throughput - a.throughput)
            .forEach((result, index) => {
                console.log(`   ${index + 1}. ${result.primitive.padEnd(20)} - ${result.throughput.toFixed(0)} ops/sec`);
            });
        
        console.log('\n📊 특성별 분석:');
        console.log('   • Spinlock: 낮은 지연시간, CPU 사용량 높음');
        console.log('   • AdaptiveMutex: 균형잡힌 성능, 다양한 워크로드에 적합');
        console.log('   • TicketLock: 공정성 보장, 중간 정도 성능');
        console.log('   • MCSLock: NUMA 환경에서 우수, 캐시 효율성 높음');
        console.log('   • ReadWriteLock: 읽기 중심 워크로드에서 우수');
        console.log('   • StampedLock: 낙관적 읽기로 최고 성능 (읽기 중심)');
        
        console.log('\n🎯 사용 권장사항:');
        console.log('   • 짧은 임계구역: Spinlock');
        console.log('   • 긴 임계구역: AdaptiveMutex');
        console.log('   • 공정성 중요: TicketLock');
        console.log('   • 읽기 중심: StampedLock');
        console.log('   • NUMA 시스템: MCSLock');
    }
}

interface BenchmarkResult {
    primitive: string;
    workload?: string;
    throughput: number;
    avgLatency: number;
    p99Latency: number;
    totalTime: number;
    completedOps: number;
}
```

---

## 📚 참고자료

### 학술 자료
- Mellor-Crummey, J. M., & Scott, M. L. (1991). "Algorithms for scalable synchronization on shared-memory multiprocessors"
- Michael, M. M., & Scott, M. L. (1996). "Simple, fast, and practical non-blocking and blocking concurrent queue algorithms"
- Herlihy, M. (1991). "Wait-free synchronization"

### 실무 자료
- [[Linux Kernel Synchronization]]
- [[Java Concurrent Programming Guide]]
- [[High Performance Computing Synchronization]]

### 관련 문서
- [[04_thread-synchronization]] - 스레드 동기화 기초
- [[09_deadlock]] - 데드락 관리
- [[11_async-programming]] - 비동기 프로그래밍

---

> [!success] 동기화 프리미티브 핵심 정리
> 1. **기본 프리미티브**: 뮤텍스, 세마포어, 조건변수의 올바른 사용
> 2. **고성능 기법**: 스핀락, MCS/CLH 락, 적응형 전략
> 3. **Lock-Free**: CAS 기반 무락 프로그래밍 기법
> 4. **읽기-쓰기 락**: 읽기 중심 워크로드 최적화
> 5. **성능 고려사항**: 경합 수준, 임계구역 길이, 하드웨어 특성

> [!quote]
> "완벽한 동기화 프리미티브는 없다. 각각은 특정 상황에서 최적이며, 시스템의 특성과 워크로드를 고려한 선택이 핵심이다." - 고성능 병행 프로그래밍 원칙