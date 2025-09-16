---
tags:
  - 운영체제
  - OS
  - 비동기프로그래밍
  - 동시성
  - 병행성
  - CTO강의
created: 2025-01-12
updated: 2025-01-12
aliases:
  - 비동기프로그래밍
  - Asynchronous Programming
  - 동시성프로그래밍
description: 비동기 프로그래밍 패턴, 이벤트 루프, Promise/Future, async/await의 TypeScript 구현
status: published
category: guide
---

# 비동기 프로그래밍 (Asynchronous Programming) - 현대적 동시성 패턴

> [!info] 개요
> 비동기 프로그래밍은 프로그램 실행 흐름을 블로킹하지 않고 작업을 수행하는 프로그래밍 패러다임입니다. 콜백, Promise, async/await 등의 패턴을 통해 효율적인 동시성을 구현합니다.

## 📑 목차

- [[#🔄 비동기 프로그래밍 기초]]
- [[#⚡ 이벤트 루프와 실행 모델]]
- [[#🎯 콜백 패턴과 문제점]]
- [[#📦 Promise와 Future]]
- [[#✨ async/await 패턴]]
- [[#🚀 고급 비동기 패턴]]
- [[#💻 TypeScript 구현]]
- [[#📊 성능 최적화]]
- [[#🏢 실제 시스템 사례]]
- [[#📚 참고자료]]

---

## 🔄 비동기 프로그래밍 기초

### 동기 vs 비동기

> [!note] 핵심 차이점
> - **동기(Synchronous)**: 작업이 완료될 때까지 대기
> - **비동기(Asynchronous)**: 작업 시작 후 즉시 반환, 완료 시 콜백/Promise로 결과 처리

```typescript
// 동기적 실행
class SynchronousExecutor {
    executeSync(task: () => number): number {
        console.log('Starting synchronous task');
        const result = task(); // 블로킹 - 작업 완료까지 대기
        console.log('Task completed with result:', result);
        return result;
    }
    
    runMultipleTasks(): void {
        const start = Date.now();
        
        // 각 작업이 순차적으로 실행됨
        const result1 = this.executeSync(() => {
            this.heavyComputation(1000);
            return 1;
        });
        
        const result2 = this.executeSync(() => {
            this.heavyComputation(1000);
            return 2;
        });
        
        const result3 = this.executeSync(() => {
            this.heavyComputation(1000);
            return 3;
        });
        
        const totalTime = Date.now() - start;
        console.log(`Total time: ${totalTime}ms`); // ~3000ms
        console.log(`Results: ${result1}, ${result2}, ${result3}`);
    }
    
    private heavyComputation(duration: number): void {
        const start = Date.now();
        while (Date.now() - start < duration) {
            // CPU 집약적 작업 시뮬레이션
            Math.sqrt(Math.random());
        }
    }
}

// 비동기적 실행
class AsynchronousExecutor {
    async executeAsync(task: () => Promise<number>): Promise<number> {
        console.log('Starting asynchronous task');
        const result = await task(); // 논-블로킹 - 다른 작업 가능
        console.log('Task completed with result:', result);
        return result;
    }
    
    async runMultipleTasks(): Promise<void> {
        const start = Date.now();
        
        // 모든 작업을 동시에 시작
        const promise1 = this.asyncTask(1000, 1);
        const promise2 = this.asyncTask(1000, 2);
        const promise3 = this.asyncTask(1000, 3);
        
        // 모든 작업이 완료될 때까지 대기
        const results = await Promise.all([promise1, promise2, promise3]);
        
        const totalTime = Date.now() - start;
        console.log(`Total time: ${totalTime}ms`); // ~1000ms (병렬 실행)
        console.log(`Results: ${results}`);
    }
    
    private async asyncTask(duration: number, value: number): Promise<number> {
        return new Promise((resolve) => {
            setTimeout(() => {
                resolve(value);
            }, duration);
        });
    }
}
```

### 동시성 vs 병렬성

```typescript
// 동시성 (Concurrency) - 논리적으로 동시 실행
class ConcurrencyExample {
    private tasks: Array<() => Promise<void>> = [];
    private currentTaskIndex: number = 0;
    
    async executeConcurrently(): Promise<void> {
        console.log('Starting concurrent execution (single thread)');
        
        // 이벤트 루프를 통한 동시성 - 실제로는 한 번에 하나씩 실행
        while (this.currentTaskIndex < this.tasks.length) {
            const task = this.tasks[this.currentTaskIndex++];
            
            // 비동기 작업 시작 (논-블로킹)
            task().then(() => {
                console.log(`Task ${this.currentTaskIndex} completed`);
            });
            
            // 다른 작업들이 진행될 수 있도록 제어 양보
            await this.yieldControl();
        }
    }
    
    private async yieldControl(): Promise<void> {
        return new Promise(resolve => setImmediate(resolve));
    }
}

// 병렬성 (Parallelism) - 물리적으로 동시 실행
class ParallelismExample {
    async executeInParallel(workerCount: number): Promise<void> {
        console.log(`Starting parallel execution with ${workerCount} workers`);
        
        // Worker 스레드 시뮬레이션 (실제로는 Worker Threads API 사용)
        const workers = Array(workerCount).fill(null).map((_, index) => 
            this.createWorker(index)
        );
        
        // 모든 워커가 동시에 실행
        const results = await Promise.all(workers);
        console.log('All workers completed:', results);
    }
    
    private async createWorker(id: number): Promise<any> {
        return new Promise((resolve) => {
            // 실제 Worker Thread 대신 시뮬레이션
            setTimeout(() => {
                const result = this.performWork(id);
                resolve(result);
            }, Math.random() * 1000);
        });
    }
    
    private performWork(workerId: number): any {
        return {
            workerId,
            result: Math.random() * 100,
            timestamp: Date.now()
        };
    }
}
```

---

## ⚡ 이벤트 루프와 실행 모델

### 이벤트 루프 구현

```typescript
class EventLoop {
    private microtaskQueue: Array<() => void> = [];
    private macrotaskQueue: Array<() => void> = [];
    private animationFrameQueue: Array<() => void> = [];
    private isRunning: boolean = false;
    private callStack: Array<string> = [];
    
    // 마이크로태스크 추가 (Promise, queueMicrotask)
    queueMicrotask(task: () => void): void {
        this.microtaskQueue.push(task);
    }
    
    // 매크로태스크 추가 (setTimeout, setInterval)
    queueMacrotask(task: () => void): void {
        this.macrotaskQueue.push(task);
    }
    
    // 애니메이션 프레임 콜백 추가
    requestAnimationFrame(callback: () => void): void {
        this.animationFrameQueue.push(callback);
    }
    
    // 이벤트 루프 실행
    async run(): Promise<void> {
        if (this.isRunning) return;
        
        this.isRunning = true;
        console.log('Event loop started');
        
        while (this.isRunning) {
            // 1단계: 매크로태스크 하나 실행
            if (this.macrotaskQueue.length > 0) {
                const macrotask = this.macrotaskQueue.shift()!;
                this.executeTask('Macrotask', macrotask);
            }
            
            // 2단계: 모든 마이크로태스크 실행
            while (this.microtaskQueue.length > 0) {
                const microtask = this.microtaskQueue.shift()!;
                this.executeTask('Microtask', microtask);
            }
            
            // 3단계: 렌더링 (브라우저 환경)
            if (this.shouldRender()) {
                this.render();
            }
            
            // 4단계: requestAnimationFrame 콜백 실행
            while (this.animationFrameQueue.length > 0) {
                const callback = this.animationFrameQueue.shift()!;
                this.executeTask('AnimationFrame', callback);
            }
            
            // 5단계: Idle 콜백 (requestIdleCallback)
            if (this.isIdle()) {
                await this.runIdleCallbacks();
            }
            
            // 큐가 모두 비어있으면 대기
            if (this.isEmpty()) {
                await this.waitForNextTask();
            }
        }
        
        console.log('Event loop stopped');
    }
    
    private executeTask(type: string, task: () => void): void {
        this.callStack.push(`${type}: ${task.name || 'anonymous'}`);
        console.log(`Executing ${type}`, this.getCallStackTrace());
        
        try {
            task();
        } catch (error) {
            console.error(`Error in ${type}:`, error);
        } finally {
            this.callStack.pop();
        }
    }
    
    private shouldRender(): boolean {
        // 60 FPS = 16.67ms per frame
        const now = Date.now();
        const lastRenderTime = (this as any).lastRenderTime || 0;
        
        if (now - lastRenderTime >= 16) {
            (this as any).lastRenderTime = now;
            return true;
        }
        return false;
    }
    
    private render(): void {
        console.log('🎨 Rendering frame');
        // 실제 렌더링 로직
    }
    
    private isIdle(): boolean {
        return this.microtaskQueue.length === 0 && 
               this.macrotaskQueue.length === 0;
    }
    
    private async runIdleCallbacks(): Promise<void> {
        // requestIdleCallback 시뮬레이션
        console.log('Running idle callbacks');
    }
    
    private isEmpty(): boolean {
        return this.microtaskQueue.length === 0 &&
               this.macrotaskQueue.length === 0 &&
               this.animationFrameQueue.length === 0;
    }
    
    private async waitForNextTask(): Promise<void> {
        // 다음 태스크까지 대기 (실제로는 OS 이벤트 대기)
        await new Promise(resolve => setTimeout(resolve, 0));
    }
    
    private getCallStackTrace(): string {
        return this.callStack.join(' -> ');
    }
    
    stop(): void {
        this.isRunning = false;
    }
    
    // 이벤트 루프 상태 정보
    getStatus(): any {
        return {
            isRunning: this.isRunning,
            microtaskQueue: this.microtaskQueue.length,
            macrotaskQueue: this.macrotaskQueue.length,
            animationFrameQueue: this.animationFrameQueue.length,
            callStack: [...this.callStack]
        };
    }
}

// 이벤트 루프 사용 예제
class EventLoopDemo {
    static demonstrateTaskPriority(): void {
        const loop = new EventLoop();
        
        // 매크로태스크
        loop.queueMacrotask(() => {
            console.log('1. Macrotask 1');
            
            // 마이크로태스크는 현재 매크로태스크 후 즉시 실행
            loop.queueMicrotask(() => {
                console.log('2. Microtask 1 (from Macrotask 1)');
            });
            
            loop.queueMicrotask(() => {
                console.log('3. Microtask 2 (from Macrotask 1)');
            });
        });
        
        loop.queueMacrotask(() => {
            console.log('4. Macrotask 2');
        });
        
        // 실행 순서:
        // 1. Macrotask 1
        // 2. Microtask 1
        // 3. Microtask 2
        // 4. Macrotask 2
        
        loop.run();
    }
}
```

### 태스크 큐 우선순위

```typescript
class PriorityTaskQueue {
    private queues: Map<number, Array<Task>> = new Map();
    private priorities: number[] = [];
    
    constructor() {
        // 우선순위 레벨 정의 (낮은 숫자 = 높은 우선순위)
        this.priorities = [
            0,  // 즉시 실행 (마이크로태스크)
            1,  // 애니메이션 프레임
            2,  // 사용자 입력
            3,  // 네트워크 응답
            4,  // 타이머
            5   // 백그라운드 작업
        ];
        
        // 각 우선순위별 큐 초기화
        for (const priority of this.priorities) {
            this.queues.set(priority, []);
        }
    }
    
    enqueue(task: Task, priority: number): void {
        const queue = this.queues.get(priority);
        if (!queue) {
            throw new Error(`Invalid priority: ${priority}`);
        }
        
        queue.push(task);
        console.log(`Task "${task.name}" queued with priority ${priority}`);
    }
    
    dequeue(): Task | null {
        // 우선순위 순서대로 확인
        for (const priority of this.priorities) {
            const queue = this.queues.get(priority)!;
            if (queue.length > 0) {
                const task = queue.shift()!;
                console.log(`Dequeuing task "${task.name}" with priority ${priority}`);
                return task;
            }
        }
        
        return null;
    }
    
    isEmpty(): boolean {
        for (const queue of this.queues.values()) {
            if (queue.length > 0) return false;
        }
        return true;
    }
    
    getQueueSizes(): Map<number, number> {
        const sizes = new Map<number, number>();
        for (const [priority, queue] of this.queues) {
            sizes.set(priority, queue.length);
        }
        return sizes;
    }
}

interface Task {
    id: string;
    name: string;
    callback: () => void | Promise<void>;
    timestamp: number;
    timeout?: number;
}
```

---

## 🎯 콜백 패턴과 문제점

### 콜백 지옥 (Callback Hell)

```typescript
// 콜백 지옥의 예
class CallbackHell {
    // 나쁜 예: 중첩된 콜백
    loadUserDataNested(userId: string, callback: (error: Error | null, data?: any) => void): void {
        this.getUser(userId, (err, user) => {
            if (err) {
                callback(err);
                return;
            }
            
            this.getUserPosts(user!.id, (err, posts) => {
                if (err) {
                    callback(err);
                    return;
                }
                
                this.getPostComments(posts![0].id, (err, comments) => {
                    if (err) {
                        callback(err);
                        return;
                    }
                    
                    this.getCommentAuthors(comments!, (err, authors) => {
                        if (err) {
                            callback(err);
                            return;
                        }
                        
                        callback(null, {
                            user,
                            posts,
                            comments,
                            authors
                        });
                    });
                });
            });
        });
    }
    
    // 개선된 버전: 콜백 체이닝
    loadUserDataChained(userId: string): void {
        const pipeline = new CallbackPipeline();
        
        pipeline
            .add((next) => this.getUser(userId, next))
            .add((user, next) => this.getUserPosts(user.id, next))
            .add((posts, next) => this.getPostComments(posts[0].id, next))
            .add((comments, next) => this.getCommentAuthors(comments, next))
            .execute((err, result) => {
                if (err) {
                    console.error('Error:', err);
                } else {
                    console.log('Result:', result);
                }
            });
    }
    
    private getUser(userId: string, callback: Function): void {
        setTimeout(() => {
            callback(null, { id: userId, name: 'User' });
        }, 100);
    }
    
    private getUserPosts(userId: string, callback: Function): void {
        setTimeout(() => {
            callback(null, [{ id: '1', userId, title: 'Post 1' }]);
        }, 100);
    }
    
    private getPostComments(postId: string, callback: Function): void {
        setTimeout(() => {
            callback(null, [{ id: '1', postId, text: 'Comment' }]);
        }, 100);
    }
    
    private getCommentAuthors(comments: any[], callback: Function): void {
        setTimeout(() => {
            callback(null, [{ id: '1', name: 'Author' }]);
        }, 100);
    }
}

// 콜백 파이프라인 헬퍼
class CallbackPipeline {
    private steps: Array<(input: any, next: Function) => void> = [];
    
    add(step: (input: any, next: Function) => void): this {
        this.steps.push(step);
        return this;
    }
    
    execute(finalCallback: (error: Error | null, result?: any) => void): void {
        let currentStep = 0;
        
        const next = (error: Error | null, result?: any) => {
            if (error) {
                finalCallback(error);
                return;
            }
            
            if (currentStep >= this.steps.length) {
                finalCallback(null, result);
                return;
            }
            
            const step = this.steps[currentStep++];
            try {
                step(result, next);
            } catch (err) {
                finalCallback(err as Error);
            }
        };
        
        next(null, null);
    }
}
```

### 에러 처리 패턴

```typescript
class CallbackErrorHandling {
    // 에러 우선 콜백 패턴 (Error-First Callback)
    asyncOperation(input: any, callback: (error: Error | null, result?: any) => void): void {
        try {
            // 비동기 작업 시뮬레이션
            setTimeout(() => {
                if (input === null || input === undefined) {
                    callback(new Error('Invalid input'));
                    return;
                }
                
                const result = this.processInput(input);
                callback(null, result);
            }, 100);
        } catch (error) {
            // 동기적 에러도 콜백으로 전달
            callback(error as Error);
        }
    }
    
    private processInput(input: any): any {
        if (typeof input === 'number' && input < 0) {
            throw new Error('Negative numbers not allowed');
        }
        return input * 2;
    }
    
    // 재시도 로직이 있는 콜백
    retryableOperation(
        operation: (callback: Function) => void,
        maxRetries: number,
        finalCallback: (error: Error | null, result?: any) => void
    ): void {
        let attempts = 0;
        
        const attemptOperation = () => {
            attempts++;
            
            operation((error: Error | null, result?: any) => {
                if (error && attempts < maxRetries) {
                    console.log(`Attempt ${attempts} failed, retrying...`);
                    setTimeout(attemptOperation, Math.pow(2, attempts) * 100); // 지수적 백오프
                } else if (error) {
                    finalCallback(new Error(`Failed after ${attempts} attempts: ${error.message}`));
                } else {
                    finalCallback(null, result);
                }
            });
        };
        
        attemptOperation();
    }
    
    // 타임아웃이 있는 콜백
    timeoutOperation<T>(
        operation: (callback: (error: Error | null, result?: T) => void) => void,
        timeout: number,
        callback: (error: Error | null, result?: T) => void
    ): void {
        let completed = false;
        
        const timeoutId = setTimeout(() => {
            if (!completed) {
                completed = true;
                callback(new Error(`Operation timed out after ${timeout}ms`));
            }
        }, timeout);
        
        operation((error, result) => {
            if (!completed) {
                completed = true;
                clearTimeout(timeoutId);
                callback(error, result);
            }
        });
    }
}
```

---

## 📦 Promise와 Future

### Promise 구현

```typescript
enum PromiseState {
    PENDING = 'pending',
    FULFILLED = 'fulfilled',
    REJECTED = 'rejected'
}

class MyPromise<T> {
    private state: PromiseState = PromiseState.PENDING;
    private value?: T;
    private reason?: any;
    private onFulfilledCallbacks: Array<(value: T) => void> = [];
    private onRejectedCallbacks: Array<(reason: any) => void> = [];
    
    constructor(executor: (resolve: (value: T) => void, reject: (reason?: any) => void) => void) {
        try {
            executor(this.resolve.bind(this), this.reject.bind(this));
        } catch (error) {
            this.reject(error);
        }
    }
    
    private resolve(value: T): void {
        if (this.state !== PromiseState.PENDING) return;
        
        // Promise Resolution Procedure
        if (value === this) {
            this.reject(new TypeError('Chaining cycle detected'));
            return;
        }
        
        if (value instanceof MyPromise) {
            value.then(
                (v) => this.resolve(v as any),
                (r) => this.reject(r)
            );
            return;
        }
        
        this.state = PromiseState.FULFILLED;
        this.value = value;
        
        // 마이크로태스크로 콜백 실행
        queueMicrotask(() => {
            this.onFulfilledCallbacks.forEach(callback => callback(value));
            this.onFulfilledCallbacks = [];
        });
    }
    
    private reject(reason?: any): void {
        if (this.state !== PromiseState.PENDING) return;
        
        this.state = PromiseState.REJECTED;
        this.reason = reason;
        
        queueMicrotask(() => {
            this.onRejectedCallbacks.forEach(callback => callback(reason));
            this.onRejectedCallbacks = [];
        });
    }
    
    then<U>(
        onFulfilled?: (value: T) => U | MyPromise<U>,
        onRejected?: (reason: any) => U | MyPromise<U>
    ): MyPromise<U> {
        return new MyPromise<U>((resolve, reject) => {
            const handleFulfilled = () => {
                if (!onFulfilled) {
                    resolve(this.value as any);
                    return;
                }
                
                try {
                    const result = onFulfilled(this.value!);
                    if (result instanceof MyPromise) {
                        result.then(resolve, reject);
                    } else {
                        resolve(result);
                    }
                } catch (error) {
                    reject(error);
                }
            };
            
            const handleRejected = () => {
                if (!onRejected) {
                    reject(this.reason);
                    return;
                }
                
                try {
                    const result = onRejected(this.reason);
                    if (result instanceof MyPromise) {
                        result.then(resolve, reject);
                    } else {
                        resolve(result);
                    }
                } catch (error) {
                    reject(error);
                }
            };
            
            if (this.state === PromiseState.FULFILLED) {
                queueMicrotask(handleFulfilled);
            } else if (this.state === PromiseState.REJECTED) {
                queueMicrotask(handleRejected);
            } else {
                this.onFulfilledCallbacks.push(handleFulfilled);
                this.onRejectedCallbacks.push(handleRejected);
            }
        });
    }
    
    catch<U>(onRejected: (reason: any) => U | MyPromise<U>): MyPromise<U> {
        return this.then(undefined, onRejected);
    }
    
    finally(onFinally: () => void): MyPromise<T> {
        return this.then(
            (value) => {
                onFinally();
                return value;
            },
            (reason) => {
                onFinally();
                throw reason;
            }
        );
    }
    
    // 정적 메서드들
    static resolve<T>(value: T): MyPromise<T> {
        return new MyPromise((resolve) => resolve(value));
    }
    
    static reject<T>(reason?: any): MyPromise<T> {
        return new MyPromise((_, reject) => reject(reason));
    }
    
    static all<T>(promises: MyPromise<T>[]): MyPromise<T[]> {
        return new MyPromise((resolve, reject) => {
            const results: T[] = [];
            let completed = 0;
            
            if (promises.length === 0) {
                resolve(results);
                return;
            }
            
            promises.forEach((promise, index) => {
                promise.then(
                    (value) => {
                        results[index] = value;
                        completed++;
                        
                        if (completed === promises.length) {
                            resolve(results);
                        }
                    },
                    reject
                );
            });
        });
    }
    
    static race<T>(promises: MyPromise<T>[]): MyPromise<T> {
        return new MyPromise((resolve, reject) => {
            promises.forEach(promise => {
                promise.then(resolve, reject);
            });
        });
    }
    
    static allSettled<T>(promises: MyPromise<T>[]): MyPromise<Array<{status: string; value?: T; reason?: any}>> {
        return new MyPromise((resolve) => {
            const results: Array<{status: string; value?: T; reason?: any}> = [];
            let settled = 0;
            
            if (promises.length === 0) {
                resolve(results);
                return;
            }
            
            promises.forEach((promise, index) => {
                promise.then(
                    (value) => {
                        results[index] = { status: 'fulfilled', value };
                        settled++;
                        if (settled === promises.length) {
                            resolve(results);
                        }
                    },
                    (reason) => {
                        results[index] = { status: 'rejected', reason };
                        settled++;
                        if (settled === promises.length) {
                            resolve(results);
                        }
                    }
                );
            });
        });
    }
}
```

### Future 패턴

```typescript
class Future<T> {
    private promise: Promise<T>;
    private resolveFunc?: (value: T) => void;
    private rejectFunc?: (reason?: any) => void;
    private completed: boolean = false;
    
    constructor() {
        this.promise = new Promise<T>((resolve, reject) => {
            this.resolveFunc = resolve;
            this.rejectFunc = reject;
        });
    }
    
    // Future를 완료시킴
    complete(value: T): boolean {
        if (this.completed) return false;
        
        this.completed = true;
        this.resolveFunc!(value);
        return true;
    }
    
    // Future를 에러로 완료
    completeError(error: any): boolean {
        if (this.completed) return false;
        
        this.completed = true;
        this.rejectFunc!(error);
        return true;
    }
    
    // Promise 반환
    get(): Promise<T> {
        return this.promise;
    }
    
    // 완료 여부 확인
    isCompleted(): boolean {
        return this.completed;
    }
    
    // 타임아웃 설정
    timeout(ms: number): Future<T> {
        setTimeout(() => {
            this.completeError(new Error(`Future timed out after ${ms}ms`));
        }, ms);
        
        return this;
    }
    
    // Future 변환
    map<U>(fn: (value: T) => U): Future<U> {
        const newFuture = new Future<U>();
        
        this.promise.then(
            (value) => newFuture.complete(fn(value)),
            (error) => newFuture.completeError(error)
        );
        
        return newFuture;
    }
    
    // Future 체이닝
    flatMap<U>(fn: (value: T) => Future<U>): Future<U> {
        const newFuture = new Future<U>();
        
        this.promise.then(
            (value) => {
                const resultFuture = fn(value);
                resultFuture.get().then(
                    (result) => newFuture.complete(result),
                    (error) => newFuture.completeError(error)
                );
            },
            (error) => newFuture.completeError(error)
        );
        
        return newFuture;
    }
    
    // 여러 Future 조합
    static all<T>(futures: Future<T>[]): Future<T[]> {
        const resultFuture = new Future<T[]>();
        const promises = futures.map(f => f.get());
        
        Promise.all(promises).then(
            (results) => resultFuture.complete(results),
            (error) => resultFuture.completeError(error)
        );
        
        return resultFuture;
    }
    
    static race<T>(futures: Future<T>[]): Future<T> {
        const resultFuture = new Future<T>();
        const promises = futures.map(f => f.get());
        
        Promise.race(promises).then(
            (result) => resultFuture.complete(result),
            (error) => resultFuture.completeError(error)
        );
        
        return resultFuture;
    }
}
```

---

## ✨ async/await 패턴

### async/await 구현 원리

```typescript
// async/await는 Generator와 Promise의 조합으로 구현
class AsyncAwaitSimulator {
    // Generator 기반 async 함수 시뮬레이션
    static async(generatorFunction: () => Generator): () => Promise<any> {
        return function(): Promise<any> {
            const generator = generatorFunction();
            
            function handle(result: IteratorResult<any>): Promise<any> {
                if (result.done) {
                    return Promise.resolve(result.value);
                }
                
                return Promise.resolve(result.value)
                    .then(
                        (res) => handle(generator.next(res)),
                        (err) => handle(generator.throw!(err))
                    );
            }
            
            try {
                return handle(generator.next());
            } catch (error) {
                return Promise.reject(error);
            }
        };
    }
    
    // 사용 예제
    static example(): void {
        const asyncFunction = AsyncAwaitSimulator.async(function* () {
            try {
                const result1 = yield Promise.resolve(1);
                console.log('Result 1:', result1);
                
                const result2 = yield Promise.resolve(2);
                console.log('Result 2:', result2);
                
                const result3 = yield new Promise(resolve => 
                    setTimeout(() => resolve(3), 1000)
                );
                console.log('Result 3:', result3);
                
                return result1 + result2 + result3;
            } catch (error) {
                console.error('Error:', error);
                throw error;
            }
        });
        
        asyncFunction().then(result => {
            console.log('Final result:', result);
        });
    }
}

// 실제 async/await 사용 패턴
class AsyncAwaitPatterns {
    // 순차 실행
    async sequential(): Promise<void> {
        console.log('Sequential execution:');
        const start = Date.now();
        
        const result1 = await this.asyncTask(1000, 1);
        const result2 = await this.asyncTask(1000, 2);
        const result3 = await this.asyncTask(1000, 3);
        
        console.log(`Results: ${result1}, ${result2}, ${result3}`);
        console.log(`Time: ${Date.now() - start}ms`); // ~3000ms
    }
    
    // 병렬 실행
    async parallel(): Promise<void> {
        console.log('Parallel execution:');
        const start = Date.now();
        
        const [result1, result2, result3] = await Promise.all([
            this.asyncTask(1000, 1),
            this.asyncTask(1000, 2),
            this.asyncTask(1000, 3)
        ]);
        
        console.log(`Results: ${result1}, ${result2}, ${result3}`);
        console.log(`Time: ${Date.now() - start}ms`); // ~1000ms
    }
    
    // 조건부 병렬 실행
    async conditionalParallel(condition: boolean): Promise<void> {
        const task1 = this.asyncTask(1000, 1);
        
        if (condition) {
            const task2 = this.asyncTask(1000, 2);
            const [result1, result2] = await Promise.all([task1, task2]);
            console.log(`Results: ${result1}, ${result2}`);
        } else {
            const result1 = await task1;
            console.log(`Result: ${result1}`);
        }
    }
    
    // 에러 처리
    async errorHandling(): Promise<void> {
        try {
            const result = await this.riskyTask();
            console.log('Success:', result);
        } catch (error) {
            console.error('Error caught:', error);
            
            // 재시도 로직
            try {
                const retryResult = await this.retryTask(3);
                console.log('Retry success:', retryResult);
            } catch (retryError) {
                console.error('All retries failed:', retryError);
            }
        } finally {
            console.log('Cleanup operations');
        }
    }
    
    // for-await-of 패턴
    async *asyncGenerator(): AsyncGenerator<number> {
        for (let i = 0; i < 5; i++) {
            await this.delay(100);
            yield i;
        }
    }
    
    async consumeAsyncIterator(): Promise<void> {
        for await (const value of this.asyncGenerator()) {
            console.log('Async value:', value);
        }
    }
    
    private async asyncTask(delay: number, value: number): Promise<number> {
        await this.delay(delay);
        return value;
    }
    
    private async riskyTask(): Promise<any> {
        if (Math.random() < 0.5) {
            throw new Error('Random failure');
        }
        return 'Success';
    }
    
    private async retryTask(maxRetries: number): Promise<any> {
        for (let i = 0; i < maxRetries; i++) {
            try {
                return await this.riskyTask();
            } catch (error) {
                if (i === maxRetries - 1) throw error;
                await this.delay(Math.pow(2, i) * 100); // 지수적 백오프
            }
        }
    }
    
    private delay(ms: number): Promise<void> {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}
```

---

## 🚀 고급 비동기 패턴

### Observable 패턴

```typescript
class Observable<T> {
    private observers: Array<Observer<T>> = [];
    
    constructor(private producer: (observer: Observer<T>) => (() => void) | void) {}
    
    subscribe(
        onNext?: (value: T) => void,
        onError?: (error: any) => void,
        onComplete?: () => void
    ): Subscription {
        const observer: Observer<T> = {
            next: onNext || (() => {}),
            error: onError || (() => {}),
            complete: onComplete || (() => {})
        };
        
        this.observers.push(observer);
        
        // Producer 실행
        const teardown = this.producer(observer);
        
        return new Subscription(() => {
            // Observer 제거
            const index = this.observers.indexOf(observer);
            if (index !== -1) {
                this.observers.splice(index, 1);
            }
            
            // Teardown 실행
            if (teardown) teardown();
        });
    }
    
    // 연산자들
    map<U>(fn: (value: T) => U): Observable<U> {
        return new Observable<U>((observer) => {
            const subscription = this.subscribe(
                (value) => observer.next(fn(value)),
                (error) => observer.error(error),
                () => observer.complete()
            );
            
            return () => subscription.unsubscribe();
        });
    }
    
    filter(predicate: (value: T) => boolean): Observable<T> {
        return new Observable<T>((observer) => {
            const subscription = this.subscribe(
                (value) => {
                    if (predicate(value)) {
                        observer.next(value);
                    }
                },
                (error) => observer.error(error),
                () => observer.complete()
            );
            
            return () => subscription.unsubscribe();
        });
    }
    
    debounceTime(ms: number): Observable<T> {
        return new Observable<T>((observer) => {
            let timeoutId: NodeJS.Timeout;
            
            const subscription = this.subscribe(
                (value) => {
                    clearTimeout(timeoutId);
                    timeoutId = setTimeout(() => {
                        observer.next(value);
                    }, ms);
                },
                (error) => observer.error(error),
                () => {
                    clearTimeout(timeoutId);
                    observer.complete();
                }
            );
            
            return () => {
                clearTimeout(timeoutId);
                subscription.unsubscribe();
            };
        });
    }
    
    // 정적 생성자들
    static interval(ms: number): Observable<number> {
        return new Observable<number>((observer) => {
            let count = 0;
            const intervalId = setInterval(() => {
                observer.next(count++);
            }, ms);
            
            return () => clearInterval(intervalId);
        });
    }
    
    static fromEvent<T>(target: EventTarget, eventName: string): Observable<T> {
        return new Observable<T>((observer) => {
            const handler = (event: any) => observer.next(event);
            target.addEventListener(eventName, handler);
            
            return () => target.removeEventListener(eventName, handler);
        });
    }
    
    static merge<T>(...observables: Observable<T>[]): Observable<T> {
        return new Observable<T>((observer) => {
            const subscriptions: Subscription[] = [];
            let completed = 0;
            
            observables.forEach(observable => {
                const subscription = observable.subscribe(
                    (value) => observer.next(value),
                    (error) => observer.error(error),
                    () => {
                        completed++;
                        if (completed === observables.length) {
                            observer.complete();
                        }
                    }
                );
                
                subscriptions.push(subscription);
            });
            
            return () => {
                subscriptions.forEach(sub => sub.unsubscribe());
            };
        });
    }
}

interface Observer<T> {
    next: (value: T) => void;
    error: (error: any) => void;
    complete: () => void;
}

class Subscription {
    constructor(private teardown: () => void) {}
    
    unsubscribe(): void {
        this.teardown();
    }
}
```

### Actor 모델

```typescript
class Actor<T> {
    private mailbox: Array<Message<T>> = [];
    private processing: boolean = false;
    private state: any = {};
    
    constructor(
        private behavior: (state: any, message: Message<T>) => Promise<any>
    ) {}
    
    // 메시지 전송
    send(message: Message<T>): void {
        this.mailbox.push(message);
        this.processMessages();
    }
    
    // 메시지 처리
    private async processMessages(): Promise<void> {
        if (this.processing) return;
        
        this.processing = true;
        
        while (this.mailbox.length > 0) {
            const message = this.mailbox.shift()!;
            
            try {
                this.state = await this.behavior(this.state, message);
            } catch (error) {
                console.error('Actor error:', error);
                this.handleError(error, message);
            }
        }
        
        this.processing = false;
    }
    
    private handleError(error: any, message: Message<T>): void {
        // 에러 처리 전략
        if (message.replyTo) {
            message.replyTo.send({
                type: 'error',
                payload: error,
                from: this
            });
        }
    }
    
    // 상태 조회
    getState(): any {
        return { ...this.state };
    }
}

interface Message<T> {
    type: string;
    payload: T;
    from?: Actor<any>;
    replyTo?: Actor<any>;
}

// Actor 시스템
class ActorSystem {
    private actors: Map<string, Actor<any>> = new Map();
    
    createActor<T>(
        name: string,
        behavior: (state: any, message: Message<T>) => Promise<any>
    ): Actor<T> {
        const actor = new Actor<T>(behavior);
        this.actors.set(name, actor);
        return actor;
    }
    
    getActor(name: string): Actor<any> | undefined {
        return this.actors.get(name);
    }
    
    broadcast<T>(message: Message<T>): void {
        for (const actor of this.actors.values()) {
            actor.send(message);
        }
    }
    
    // Supervisor 패턴
    createSupervisor(): Actor<any> {
        return this.createActor('supervisor', async (state, message) => {
            switch (message.type) {
                case 'child_error':
                    console.log('Child actor error, restarting...');
                    this.restartActor(message.payload.actorName);
                    break;
                    
                case 'monitor':
                    state.monitoring = message.payload;
                    break;
                    
                default:
                    console.log('Supervisor received:', message);
            }
            
            return state;
        });
    }
    
    private restartActor(name: string): void {
        const actor = this.actors.get(name);
        if (actor) {
            // Actor 재시작 로직
            console.log(`Restarting actor: ${name}`);
        }
    }
}
```

---

## 💻 TypeScript 구현

### 통합 비동기 유틸리티

```typescript
class AsyncUtilities {
    // 비동기 큐
    static createAsyncQueue<T>(): AsyncQueue<T> {
        return new AsyncQueue<T>();
    }
    
    // 비동기 세마포어
    static createAsyncSemaphore(permits: number): AsyncSemaphore {
        return new AsyncSemaphore(permits);
    }
    
    // 비동기 뮤텍스
    static createAsyncMutex(): AsyncMutex {
        return new AsyncMutex();
    }
    
    // 디바운스
    static debounce<T extends (...args: any[]) => any>(
        func: T,
        wait: number
    ): (...args: Parameters<T>) => void {
        let timeoutId: NodeJS.Timeout;
        
        return function(...args: Parameters<T>): void {
            clearTimeout(timeoutId);
            timeoutId = setTimeout(() => func(...args), wait);
        };
    }
    
    // 쓰로틀
    static throttle<T extends (...args: any[]) => any>(
        func: T,
        limit: number
    ): (...args: Parameters<T>) => void {
        let inThrottle = false;
        
        return function(...args: Parameters<T>): void {
            if (!inThrottle) {
                func(...args);
                inThrottle = true;
                setTimeout(() => inThrottle = false, limit);
            }
        };
    }
    
    // 재시도
    static async retry<T>(
        operation: () => Promise<T>,
        options: {
            maxAttempts?: number;
            delay?: number;
            backoff?: 'linear' | 'exponential';
            onRetry?: (attempt: number, error: any) => void;
        } = {}
    ): Promise<T> {
        const {
            maxAttempts = 3,
            delay = 1000,
            backoff = 'exponential',
            onRetry
        } = options;
        
        let lastError: any;
        
        for (let attempt = 1; attempt <= maxAttempts; attempt++) {
            try {
                return await operation();
            } catch (error) {
                lastError = error;
                
                if (attempt < maxAttempts) {
                    if (onRetry) onRetry(attempt, error);
                    
                    const waitTime = backoff === 'exponential'
                        ? delay * Math.pow(2, attempt - 1)
                        : delay * attempt;
                    
                    await this.delay(waitTime);
                }
            }
        }
        
        throw lastError;
    }
    
    // 타임아웃
    static async timeout<T>(
        promise: Promise<T>,
        ms: number,
        errorMessage?: string
    ): Promise<T> {
        const timeoutPromise = new Promise<T>((_, reject) => {
            setTimeout(() => {
                reject(new Error(errorMessage || `Operation timed out after ${ms}ms`));
            }, ms);
        });
        
        return Promise.race([promise, timeoutPromise]);
    }
    
    // 지연
    static delay(ms: number): Promise<void> {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
    
    // 병렬 실행 with 제한
    static async parallelLimit<T, R>(
        items: T[],
        limit: number,
        operation: (item: T) => Promise<R>
    ): Promise<R[]> {
        const results: R[] = [];
        const executing: Promise<void>[] = [];
        
        for (const item of items) {
            const promise = operation(item).then(result => {
                results.push(result);
            });
            
            executing.push(promise);
            
            if (executing.length >= limit) {
                await Promise.race(executing);
                executing.splice(executing.findIndex(p => p === promise), 1);
            }
        }
        
        await Promise.all(executing);
        return results;
    }
}

// 비동기 큐
class AsyncQueue<T> {
    private items: T[] = [];
    private resolvers: Array<(value: T) => void> = [];
    
    enqueue(item: T): void {
        if (this.resolvers.length > 0) {
            const resolver = this.resolvers.shift()!;
            resolver(item);
        } else {
            this.items.push(item);
        }
    }
    
    async dequeue(): Promise<T> {
        if (this.items.length > 0) {
            return this.items.shift()!;
        }
        
        return new Promise<T>((resolve) => {
            this.resolvers.push(resolve);
        });
    }
    
    size(): number {
        return this.items.length;
    }
    
    isEmpty(): boolean {
        return this.items.length === 0;
    }
}

// 비동기 세마포어
class AsyncSemaphore {
    private permits: number;
    private waiters: Array<() => void> = [];
    
    constructor(permits: number) {
        this.permits = permits;
    }
    
    async acquire(): Promise<void> {
        if (this.permits > 0) {
            this.permits--;
            return;
        }
        
        return new Promise<void>((resolve) => {
            this.waiters.push(resolve);
        });
    }
    
    release(): void {
        if (this.waiters.length > 0) {
            const waiter = this.waiters.shift()!;
            waiter();
        } else {
            this.permits++;
        }
    }
    
    async withPermit<T>(operation: () => Promise<T>): Promise<T> {
        await this.acquire();
        try {
            return await operation();
        } finally {
            this.release();
        }
    }
}

// 비동기 뮤텍스
class AsyncMutex {
    private locked: boolean = false;
    private waiters: Array<() => void> = [];
    
    async acquire(): Promise<void> {
        if (!this.locked) {
            this.locked = true;
            return;
        }
        
        return new Promise<void>((resolve) => {
            this.waiters.push(resolve);
        });
    }
    
    release(): void {
        if (this.waiters.length > 0) {
            const waiter = this.waiters.shift()!;
            waiter();
        } else {
            this.locked = false;
        }
    }
    
    async withLock<T>(operation: () => Promise<T>): Promise<T> {
        await this.acquire();
        try {
            return await operation();
        } finally {
            this.release();
        }
    }
    
    isLocked(): boolean {
        return this.locked;
    }
}
```

---

## 📊 성능 최적화

### 비동기 성능 벤치마크

```typescript
class AsyncPerformanceBenchmark {
    async runBenchmarks(): Promise<void> {
        console.log('🚀 비동기 프로그래밍 성능 벤치마크\n');
        
        await this.benchmarkCallbackVsPromise();
        await this.benchmarkSequentialVsParallel();
        await this.benchmarkConcurrencyLimits();
        await this.benchmarkEventLoopBlocking();
    }
    
    private async benchmarkCallbackVsPromise(): Promise<void> {
        console.log('📊 Callback vs Promise vs Async/Await:');
        
        const iterations = 1000;
        
        // Callback 벤치마크
        const callbackStart = Date.now();
        await new Promise<void>((resolve) => {
            let completed = 0;
            
            for (let i = 0; i < iterations; i++) {
                this.callbackOperation((err, result) => {
                    completed++;
                    if (completed === iterations) {
                        resolve();
                    }
                });
            }
        });
        const callbackTime = Date.now() - callbackStart;
        
        // Promise 벤치마크
        const promiseStart = Date.now();
        const promises = [];
        for (let i = 0; i < iterations; i++) {
            promises.push(this.promiseOperation());
        }
        await Promise.all(promises);
        const promiseTime = Date.now() - promiseStart;
        
        // Async/Await 벤치마크
        const asyncStart = Date.now();
        const asyncPromises = [];
        for (let i = 0; i < iterations; i++) {
            asyncPromises.push(this.asyncOperation());
        }
        await Promise.all(asyncPromises);
        const asyncTime = Date.now() - asyncStart;
        
        console.log(`  Callback: ${callbackTime}ms`);
        console.log(`  Promise: ${promiseTime}ms`);
        console.log(`  Async/Await: ${asyncTime}ms\n`);
    }
    
    private async benchmarkSequentialVsParallel(): Promise<void> {
        console.log('📊 Sequential vs Parallel Execution:');
        
        const tasks = 10;
        const taskDuration = 100;
        
        // 순차 실행
        const sequentialStart = Date.now();
        for (let i = 0; i < tasks; i++) {
            await this.delay(taskDuration);
        }
        const sequentialTime = Date.now() - sequentialStart;
        
        // 병렬 실행
        const parallelStart = Date.now();
        const parallelPromises = [];
        for (let i = 0; i < tasks; i++) {
            parallelPromises.push(this.delay(taskDuration));
        }
        await Promise.all(parallelPromises);
        const parallelTime = Date.now() - parallelStart;
        
        console.log(`  Sequential: ${sequentialTime}ms`);
        console.log(`  Parallel: ${parallelTime}ms`);
        console.log(`  Speedup: ${(sequentialTime / parallelTime).toFixed(2)}x\n`);
    }
    
    private async benchmarkConcurrencyLimits(): Promise<void> {
        console.log('📊 Concurrency Limits Performance:');
        
        const tasks = 100;
        const results: { limit: number; time: number }[] = [];
        
        for (const limit of [1, 5, 10, 20, 50, 100]) {
            const start = Date.now();
            
            await AsyncUtilities.parallelLimit(
                Array(tasks).fill(null),
                limit,
                async () => {
                    await this.delay(10);
                    return Math.random();
                }
            );
            
            const time = Date.now() - start;
            results.push({ limit, time });
            
            console.log(`  Limit ${limit}: ${time}ms`);
        }
        
        console.log('');
    }
    
    private async benchmarkEventLoopBlocking(): Promise<void> {
        console.log('📊 Event Loop Blocking Impact:');
        
        // Non-blocking
        const nonBlockingStart = Date.now();
        let nonBlockingCount = 0;
        
        const interval = setInterval(() => {
            nonBlockingCount++;
        }, 10);
        
        await this.delay(1000);
        clearInterval(interval);
        
        const nonBlockingTime = Date.now() - nonBlockingStart;
        
        // Blocking
        const blockingStart = Date.now();
        let blockingCount = 0;
        
        const interval2 = setInterval(() => {
            blockingCount++;
        }, 10);
        
        // 블로킹 작업 시뮬레이션
        this.blockEventLoop(500);
        
        await this.delay(500);
        clearInterval(interval2);
        
        const blockingTime = Date.now() - blockingStart;
        
        console.log(`  Non-blocking: ${nonBlockingCount} ticks in ${nonBlockingTime}ms`);
        console.log(`  Blocking: ${blockingCount} ticks in ${blockingTime}ms`);
        console.log(`  Impact: ${((1 - blockingCount/nonBlockingCount) * 100).toFixed(1)}% reduction\n`);
    }
    
    private callbackOperation(callback: (err: any, result?: any) => void): void {
        setImmediate(() => {
            callback(null, Math.random());
        });
    }
    
    private promiseOperation(): Promise<number> {
        return new Promise((resolve) => {
            setImmediate(() => {
                resolve(Math.random());
            });
        });
    }
    
    private async asyncOperation(): Promise<number> {
        await this.delay(0);
        return Math.random();
    }
    
    private delay(ms: number): Promise<void> {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
    
    private blockEventLoop(duration: number): void {
        const start = Date.now();
        while (Date.now() - start < duration) {
            // Blocking operation
            Math.sqrt(Math.random());
        }
    }
}
```

---

## 🏢 실제 시스템 사례

### Node.js libuv 이벤트 루프

```typescript
// Node.js의 libuv 이벤트 루프 시뮬레이션
class LibuvEventLoop {
    private phases: Map<string, Array<() => void>> = new Map([
        ['timers', []],        // setTimeout, setInterval 콜백
        ['pending', []],       // I/O 콜백 (에러, TCP 등)
        ['idle', []],          // 내부용
        ['prepare', []],       // 내부용
        ['poll', []],          // 새로운 I/O 이벤트 가져오기
        ['check', []],         // setImmediate 콜백
        ['close', []]          // close 이벤트 콜백
    ]);
    
    private nextTickQueue: Array<() => void> = [];
    private microTaskQueue: Array<() => void> = [];
    private running: boolean = false;
    
    // process.nextTick
    nextTick(callback: () => void): void {
        this.nextTickQueue.push(callback);
    }
    
    // Promise 콜백
    queueMicroTask(callback: () => void): void {
        this.microTaskQueue.push(callback);
    }
    
    // setTimeout
    setTimeout(callback: () => void, delay: number): NodeJS.Timeout {
        const timerId = setTimeout(() => {
            this.phases.get('timers')!.push(callback);
        }, delay);
        
        return timerId;
    }
    
    // setImmediate
    setImmediate(callback: () => void): void {
        this.phases.get('check')!.push(callback);
    }
    
    // I/O 작업
    scheduleIO(callback: () => void): void {
        // 실제로는 비동기 I/O 완료 시 poll 단계에 추가
        this.phases.get('poll')!.push(callback);
    }
    
    // 이벤트 루프 실행
    async run(): Promise<void> {
        this.running = true;
        
        while (this.running) {
            // 각 페이즈 실행
            for (const [phase, callbacks] of this.phases) {
                console.log(`📍 Phase: ${phase}`);
                
                // 페이즈 콜백 실행
                while (callbacks.length > 0) {
                    const callback = callbacks.shift()!;
                    callback();
                    
                    // 각 콜백 후 nextTick과 microTask 처리
                    await this.processTicksAndMicroTasks();
                }
                
                // Poll 페이즈에서 I/O 대기
                if (phase === 'poll') {
                    await this.pollForIO();
                }
            }
            
            // 모든 큐가 비었으면 종료
            if (this.isAllQueuesEmpty()) {
                console.log('All queues empty, stopping event loop');
                this.running = false;
            }
        }
    }
    
    private async processTicksAndMicroTasks(): Promise<void> {
        // nextTick 큐 처리 (최우선)
        while (this.nextTickQueue.length > 0) {
            const callback = this.nextTickQueue.shift()!;
            callback();
        }
        
        // microTask 큐 처리
        while (this.microTaskQueue.length > 0) {
            const callback = this.microTaskQueue.shift()!;
            callback();
        }
    }
    
    private async pollForIO(): Promise<void> {
        // I/O 이벤트 대기 시뮬레이션
        console.log('⏳ Polling for I/O events...');
        
        // 타이머가 있으면 타이머까지만 대기
        const hasTimers = this.phases.get('timers')!.length > 0;
        const hasImmediate = this.phases.get('check')!.length > 0;
        
        if (!hasTimers && !hasImmediate) {
            // 블로킹 I/O 대기
            await this.delay(100);
        }
    }
    
    private isAllQueuesEmpty(): boolean {
        for (const callbacks of this.phases.values()) {
            if (callbacks.length > 0) return false;
        }
        
        return this.nextTickQueue.length === 0 && 
               this.microTaskQueue.length === 0;
    }
    
    private delay(ms: number): Promise<void> {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
    
    stop(): void {
        this.running = false;
    }
}
```

---

## 📚 참고자료

### 학술 자료
- Liskov, B., & Shrira, L. (1988). "Promises: Linguistic support for efficient asynchronous procedure calls"
- Haller, P., & Odersky, M. (2009). "Scala actors: Unifying thread-based and event-based programming"
- Meijer, E. (2010). "Reactive Extensions: The Dual of the Iterator Pattern"

### 실무 자료
- [[JavaScript Event Loop]]
- [[Node.js Async Programming]]
- [[RxJS Observable Patterns]]

### 관련 문서
- [[04_thread-synchronization]] - 스레드 동기화
- [[10_synchronization-primitives]] - 동기화 프리미티브
- [[12_garbage-collection]] - 메모리 관리

---

> [!success] 비동기 프로그래밍 핵심 정리
> 1. **이벤트 루프**: 태스크 큐와 마이크로태스크 큐의 실행 순서 이해
> 2. **Promise/Future**: 비동기 작업의 결과를 나타내는 객체
> 3. **async/await**: 동기적 코드처럼 보이는 비동기 코드 작성
> 4. **Observable**: 시간에 따른 값의 스트림 처리
> 5. **성능 고려사항**: 블로킹 방지, 동시성 제한, 적절한 패턴 선택

> [!quote]
> "비동기 프로그래밍은 현대 소프트웨어의 핵심이다. 효율적인 자원 활용과 반응성 있는 시스템을 위해서는 비동기 패턴의 올바른 이해와 적용이 필수적이다." - 현대 프로그래밍 패러다임