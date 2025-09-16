---
tags:
  - 운영체제
  - OS
  - 데드락
  - 동기화
  - 시스템프로그래밍
  - CTO강의
created: 2025-01-12
updated: 2025-01-12
aliases:
  - 데드락
  - Deadlock
  - 교착상태
description: 데드락의 개념, 발생 조건, 예방/회피/탐지 기법 및 TypeScript 구현
status: published
category: guide
---

# 데드락 (Deadlock) - 운영체제 핵심 개념

> [!info] 개요
> 데드락(교착상태)은 두 개 이상의 프로세스가 서로 상대방의 자원을 기다리며 무한정 대기하는 상황입니다. 시스템 설계에서 매우 중요한 문제로, 예방, 회피, 탐지 및 복구 방법을 다룹니다.

## 📑 목차

- [[#🔒 데드락의 정의와 특징]]
- [[#⚠️ 데드락 발생 조건]]
- [[#🛡️ 데드락 예방 (Prevention)]]
- [[#🚦 데드락 회피 (Avoidance)]]
- [[#🔍 데드락 탐지와 복구]]
- [[#💻 TypeScript 구현]]
- [[#📊 성능 분석]]
- [[#🏢 실제 시스템 사례]]
- [[#📚 참고자료]]

---

## 🔒 데드락의 정의와 특징

### 데드락이란?

> [!note] 데드락 정의
> 두 개 이상의 프로세스가 서로 상대방이 점유한 자원을 요청하며, 어느 쪽도 자원을 양보하지 않아 무한정 대기하게 되는 상황

### 실생활 예시

> [!example] 교차로 데드락
> ```
> 북쪽에서 온 차 A: 동쪽으로 가려고 함 (남쪽 차선 필요)
> 동쪽에서 온 차 B: 남쪽으로 가려고 함 (서쪽 차선 필요)  
> 남쪽에서 온 차 C: 서쪽으로 가려고 함 (북쪽 차선 필요)
> 서쪽에서 온 차 D: 북쪽으로 가려고 함 (동쪽 차선 필요)
> → 모든 차가 서로를 기다리며 교착상태 발생
> ```

### 데드락의 특징

- **순환 대기**: 프로세스들이 원형으로 서로를 대기
- **불가역성**: 외부 개입 없이는 해결 불가능
- **자원 낭비**: 시스템 자원이 비효율적으로 사용
- **시스템 중단**: 심각한 경우 전체 시스템 마비

---

## ⚠️ 데드락 발생 조건

### Coffman의 4가지 필요조건

> [!warning] 데드락 발생의 4가지 조건
> 다음 4가지 조건이 **모두** 성립해야 데드락 발생

#### 1. 상호 배제 (Mutual Exclusion)

```typescript
class MutualExclusionResource {
    private owner: string | null = null;
    
    acquire(processId: string): boolean {
        if (this.owner === null) {
            this.owner = processId;
            return true;
        }
        return false; // 이미 다른 프로세스가 소유
    }
    
    release(processId: string): boolean {
        if (this.owner === processId) {
            this.owner = null;
            return true;
        }
        return false;
    }
}
```

#### 2. 점유와 대기 (Hold and Wait)

```typescript
class HoldAndWaitProcess {
    private heldResources: Set<string> = new Set();
    private waitingFor: string | null = null;
    
    requestResource(resourceId: string, resourceManager: ResourceManager): boolean {
        if (this.heldResources.size > 0) {
            // 이미 자원을 보유하면서 새로운 자원 요청
            this.waitingFor = resourceId;
            return resourceManager.request(resourceId, this.processId);
        }
        return false;
    }
}
```

#### 3. 비선점 (No Preemption)

```typescript
class NonPreemptiveResource {
    private owner: string | null = null;
    
    // 강제로 자원을 빼앗을 수 없음
    forceRelease(): boolean {
        return false; // 비선점 자원은 강제 해제 불가
    }
    
    // 오직 소유자만이 자원을 해제할 수 있음
    voluntaryRelease(processId: string): boolean {
        if (this.owner === processId) {
            this.owner = null;
            return true;
        }
        return false;
    }
}
```

#### 4. 순환 대기 (Circular Wait)

```typescript
class CircularWaitDetector {
    private waitForGraph: Map<string, Set<string>> = new Map();
    
    addWaitRelation(waiter: string, holder: string): void {
        if (!this.waitForGraph.has(waiter)) {
            this.waitForGraph.set(waiter, new Set());
        }
        this.waitForGraph.get(waiter)!.add(holder);
    }
    
    detectCycle(): string[] | null {
        const visited = new Set<string>();
        const recursionStack = new Set<string>();
        
        for (const [node] of this.waitForGraph) {
            if (this.hasCycle(node, visited, recursionStack, [])) {
                return this.findCycle(node);
            }
        }
        return null;
    }
    
    private hasCycle(node: string, visited: Set<string>, 
                    recursionStack: Set<string>, path: string[]): boolean {
        visited.add(node);
        recursionStack.add(node);
        path.push(node);
        
        const neighbors = this.waitForGraph.get(node) || new Set();
        for (const neighbor of neighbors) {
            if (!visited.has(neighbor)) {
                if (this.hasCycle(neighbor, visited, recursionStack, path)) {
                    return true;
                }
            } else if (recursionStack.has(neighbor)) {
                return true;
            }
        }
        
        recursionStack.delete(node);
        path.pop();
        return false;
    }
}
```

---

## 🛡️ 데드락 예방 (Prevention)

### 4가지 조건 중 하나를 제거하여 예방

#### 1. 상호 배제 제거

> [!tip] 공유 가능한 자원 사용
> 일부 자원을 여러 프로세스가 동시에 사용할 수 있도록 설계

```typescript
class ShareableResource {
    private readers: Set<string> = new Set();
    private writer: string | null = null;
    private readonly maxReaders: number = 10;
    
    acquireRead(processId: string): boolean {
        if (this.writer === null && this.readers.size < this.maxReaders) {
            this.readers.add(processId);
            return true;
        }
        return false;
    }
    
    acquireWrite(processId: string): boolean {
        if (this.writer === null && this.readers.size === 0) {
            this.writer = processId;
            return true;
        }
        return false;
    }
    
    releaseRead(processId: string): void {
        this.readers.delete(processId);
    }
    
    releaseWrite(processId: string): void {
        if (this.writer === processId) {
            this.writer = null;
        }
    }
}
```

#### 2. 점유와 대기 제거 - 일괄 할당

```typescript
class AtomicResourceAllocation {
    private resources: Map<string, string | null> = new Map();
    private allocationLock = new Mutex();
    
    async allocateAll(processId: string, resourceIds: string[]): Promise<boolean> {
        await this.allocationLock.acquire();
        
        try {
            // 모든 자원이 사용 가능한지 확인
            for (const resourceId of resourceIds) {
                if (this.resources.get(resourceId) !== null) {
                    return false; // 하나라도 사용 중이면 실패
                }
            }
            
            // 모든 자원을 한 번에 할당
            for (const resourceId of resourceIds) {
                this.resources.set(resourceId, processId);
            }
            
            return true;
        } finally {
            this.allocationLock.release();
        }
    }
    
    releaseAll(processId: string, resourceIds: string[]): void {
        for (const resourceId of resourceIds) {
            if (this.resources.get(resourceId) === processId) {
                this.resources.set(resourceId, null);
            }
        }
    }
}
```

#### 3. 비선점 제거 - 강제 회수

```typescript
class PreemptiveResourceManager {
    private resourceOwners: Map<string, string> = new Map();
    private processPriorities: Map<string, number> = new Map();
    
    requestResource(processId: string, resourceId: string): boolean {
        const currentOwner = this.resourceOwners.get(resourceId);
        
        if (!currentOwner) {
            // 자원이 비어있으면 바로 할당
            this.resourceOwners.set(resourceId, processId);
            return true;
        }
        
        // 우선순위 비교하여 선점 가능한지 확인
        const requestorPriority = this.processPriorities.get(processId) || 0;
        const ownerPriority = this.processPriorities.get(currentOwner) || 0;
        
        if (requestorPriority > ownerPriority) {
            // 높은 우선순위 프로세스가 자원을 선점
            this.preemptResource(currentOwner, resourceId);
            this.resourceOwners.set(resourceId, processId);
            return true;
        }
        
        return false;
    }
    
    private preemptResource(victimProcess: string, resourceId: string): void {
        // 피해자 프로세스에게 자원 회수 알림
        console.log(`Resource ${resourceId} preempted from process ${victimProcess}`);
        // 프로세스 상태를 rollback하거나 재시작 준비
        this.rollbackProcess(victimProcess);
    }
    
    private rollbackProcess(processId: string): void {
        // 프로세스 상태를 이전 체크포인트로 복구
        console.log(`Rolling back process ${processId} due to resource preemption`);
    }
}
```

#### 4. 순환 대기 제거 - 자원 순서화

```typescript
class OrderedResourceAllocation {
    private resourceOrder: Map<string, number> = new Map([
        ['printer', 1],
        ['scanner', 2],
        ['network', 3],
        ['database', 4],
        ['file_system', 5]
    ]);
    
    private processResources: Map<string, Set<string>> = new Map();
    
    requestResource(processId: string, resourceId: string): boolean {
        const requestedOrder = this.resourceOrder.get(resourceId);
        if (requestedOrder === undefined) {
            throw new Error(`Unknown resource: ${resourceId}`);
        }
        
        const processRes = this.processResources.get(processId) || new Set();
        
        // 현재 보유한 자원 중 가장 높은 순서 번호 찾기
        let maxHeldOrder = 0;
        for (const heldResource of processRes) {
            const heldOrder = this.resourceOrder.get(heldResource) || 0;
            maxHeldOrder = Math.max(maxHeldOrder, heldOrder);
        }
        
        // 순서가 증가하는 방향으로만 자원 요청 허용
        if (requestedOrder <= maxHeldOrder) {
            console.log(`Request denied: Resource order violation for ${processId}`);
            return false;
        }
        
        // 자원 할당
        processRes.add(resourceId);
        this.processResources.set(processId, processRes);
        return true;
    }
    
    releaseResource(processId: string, resourceId: string): void {
        const processRes = this.processResources.get(processId);
        if (processRes) {
            processRes.delete(resourceId);
        }
    }
    
    // 자원 요청 순서 검증
    validateRequestOrder(processId: string, resourceSequence: string[]): boolean {
        const orders = resourceSequence.map(r => this.resourceOrder.get(r) || 0);
        
        for (let i = 1; i < orders.length; i++) {
            if (orders[i] <= orders[i-1]) {
                return false; // 순서 위반
            }
        }
        return true;
    }
}
```

---

## 🚦 데드락 회피 (Avoidance)

### 은행원 알고리즘 (Banker's Algorithm)

> [!note] 은행원 알고리즘의 원리
> 모든 프로세스의 최대 자원 요구량을 미리 알고, 안전한 상태를 유지하면서 자원을 할당

```typescript
interface ProcessInfo {
    id: string;
    maxNeed: number[];        // 최대 필요량
    allocation: number[];     // 현재 할당량
    need: number[];          // 추가 필요량 (maxNeed - allocation)
}

class BankersAlgorithm {
    private processes: Map<string, ProcessInfo> = new Map();
    private available: number[];              // 사용 가능한 자원
    private resourceTypes: number;            // 자원 종류 수
    
    constructor(resourceTypes: number, totalResources: number[]) {
        this.resourceTypes = resourceTypes;
        this.available = [...totalResources];
    }
    
    // 프로세스 등록
    addProcess(processId: string, maxNeed: number[]): void {
        const allocation = new Array(this.resourceTypes).fill(0);
        const need = [...maxNeed];
        
        this.processes.set(processId, {
            id: processId,
            maxNeed,
            allocation,
            need
        });
    }
    
    // 자원 요청 처리
    requestResources(processId: string, request: number[]): boolean {
        const process = this.processes.get(processId);
        if (!process) {
            throw new Error(`Process ${processId} not found`);
        }
        
        // 1. 요청이 need보다 작거나 같은지 확인
        for (let i = 0; i < this.resourceTypes; i++) {
            if (request[i] > process.need[i]) {
                console.log(`Request exceeds declared need for process ${processId}`);
                return false;
            }
        }
        
        // 2. 요청이 available보다 작거나 같은지 확인
        for (let i = 0; i < this.resourceTypes; i++) {
            if (request[i] > this.available[i]) {
                console.log(`Not enough resources available for process ${processId}`);
                return false;
            }
        }
        
        // 3. 가상으로 자원을 할당해보고 안전한지 확인
        this.simulateAllocation(process, request);
        
        if (this.isSafeState()) {
            // 안전한 상태이면 실제로 할당
            this.allocateResources(process, request);
            return true;
        } else {
            // 안전하지 않으면 할당을 취소
            this.rollbackAllocation(process, request);
            console.log(`Request from ${processId} would lead to unsafe state`);
            return false;
        }
    }
    
    private simulateAllocation(process: ProcessInfo, request: number[]): void {
        for (let i = 0; i < this.resourceTypes; i++) {
            this.available[i] -= request[i];
            process.allocation[i] += request[i];
            process.need[i] -= request[i];
        }
    }
    
    private rollbackAllocation(process: ProcessInfo, request: number[]): void {
        for (let i = 0; i < this.resourceTypes; i++) {
            this.available[i] += request[i];
            process.allocation[i] -= request[i];
            process.need[i] += request[i];
        }
    }
    
    private allocateResources(process: ProcessInfo, request: number[]): void {
        // 이미 simulateAllocation에서 할당했으므로 추가 작업 없음
        console.log(`Resources allocated to ${process.id}:`, request);
    }
    
    // 안전 상태 검사
    private isSafeState(): boolean {
        const work = [...this.available];
        const finish = new Map<string, boolean>();
        const safeSequence: string[] = [];
        
        // 모든 프로세스를 완료되지 않은 상태로 초기화
        for (const [processId] of this.processes) {
            finish.set(processId, false);
        }
        
        let found = true;
        while (found) {
            found = false;
            
            for (const [processId, process] of this.processes) {
                if (!finish.get(processId)) {
                    // 프로세스가 완료될 수 있는지 확인
                    let canFinish = true;
                    for (let i = 0; i < this.resourceTypes; i++) {
                        if (process.need[i] > work[i]) {
                            canFinish = false;
                            break;
                        }
                    }
                    
                    if (canFinish) {
                        // 프로세스 완료 시뮬레이션
                        for (let i = 0; i < this.resourceTypes; i++) {
                            work[i] += process.allocation[i];
                        }
                        finish.set(processId, true);
                        safeSequence.push(processId);
                        found = true;
                    }
                }
            }
        }
        
        // 모든 프로세스가 완료되었는지 확인
        const allFinished = Array.from(finish.values()).every(f => f);
        if (allFinished) {
            console.log(`Safe sequence found: ${safeSequence.join(' -> ')}`);
        }
        
        return allFinished;
    }
    
    // 자원 해제
    releaseResources(processId: string, release: number[]): void {
        const process = this.processes.get(processId);
        if (!process) {
            throw new Error(`Process ${processId} not found`);
        }
        
        for (let i = 0; i < this.resourceTypes; i++) {
            if (release[i] > process.allocation[i]) {
                throw new Error(`Cannot release more resources than allocated`);
            }
            
            this.available[i] += release[i];
            process.allocation[i] -= release[i];
            process.need[i] += release[i];
        }
        
        console.log(`Resources released by ${processId}:`, release);
    }
    
    // 현재 상태 출력
    printSystemState(): void {
        console.log('\n=== Current System State ===');
        console.log('Available resources:', this.available);
        console.log('\nProcess Information:');
        console.log('PID\tAllocation\tMax\t\tNeed');
        
        for (const [processId, process] of this.processes) {
            console.log(`${processId}\t[${process.allocation.join(',')}]\t\t[${process.maxNeed.join(',')}]\t\t[${process.need.join(',')}]`);
        }
        console.log('============================\n');
    }
}
```

### 은행원 알고리즘 사용 예제

```typescript
// 사용 예제
const banker = new BankersAlgorithm(3, [10, 5, 7]); // 3종류 자원, 총 개수 [10,5,7]

// 프로세스 등록
banker.addProcess('P0', [7, 5, 3]);
banker.addProcess('P1', [3, 2, 2]);
banker.addProcess('P2', [9, 0, 2]);
banker.addProcess('P3', [2, 2, 2]);
banker.addProcess('P4', [4, 3, 3]);

// 초기 할당
banker.requestResources('P0', [0, 1, 0]);
banker.requestResources('P1', [2, 0, 0]);
banker.requestResources('P2', [3, 0, 2]);
banker.requestResources('P3', [2, 1, 1]);
banker.requestResources('P4', [0, 0, 2]);

banker.printSystemState();

// 추가 자원 요청
console.log('P1 requests [1, 0, 2]:', banker.requestResources('P1', [1, 0, 2]));
banker.printSystemState();
```

---

## 🔍 데드락 탐지와 복구

### 자원 할당 그래프를 이용한 탐지

```typescript
interface ResourceNode {
    id: string;
    type: 'process' | 'resource';
    instances?: number;  // 자원인 경우 인스턴스 수
}

interface Edge {
    from: string;
    to: string;
    type: 'request' | 'assignment';
}

class ResourceAllocationGraph {
    private nodes: Map<string, ResourceNode> = new Map();
    private edges: Edge[] = [];
    
    addProcessNode(processId: string): void {
        this.nodes.set(processId, {
            id: processId,
            type: 'process'
        });
    }
    
    addResourceNode(resourceId: string, instances: number = 1): void {
        this.nodes.set(resourceId, {
            id: resourceId,
            type: 'resource',
            instances
        });
    }
    
    addRequestEdge(processId: string, resourceId: string): void {
        this.edges.push({
            from: processId,
            to: resourceId,
            type: 'request'
        });
    }
    
    addAssignmentEdge(resourceId: string, processId: string): void {
        this.edges.push({
            from: resourceId,
            to: processId,
            type: 'assignment'
        });
    }
    
    // Wait-for 그래프 생성 (단일 인스턴스 자원용)
    private buildWaitForGraph(): Map<string, Set<string>> {
        const waitFor = new Map<string, Set<string>>();
        
        // 모든 프로세스 초기화
        for (const [nodeId, node] of this.nodes) {
            if (node.type === 'process') {
                waitFor.set(nodeId, new Set());
            }
        }
        
        // 대기 관계 구성
        for (const requestEdge of this.edges.filter(e => e.type === 'request')) {
            const waitingProcess = requestEdge.from;
            const requestedResource = requestEdge.to;
            
            // 해당 자원을 점유한 프로세스 찾기
            for (const assignmentEdge of this.edges.filter(e => 
                e.type === 'assignment' && e.from === requestedResource)) {
                const holdingProcess = assignmentEdge.to;
                waitFor.get(waitingProcess)?.add(holdingProcess);
            }
        }
        
        return waitFor;
    }
    
    // 데드락 탐지 (단일 인스턴스 자원)
    detectDeadlockSingleInstance(): string[] | null {
        const waitForGraph = this.buildWaitForGraph();
        return this.findCycleInWaitForGraph(waitForGraph);
    }
    
    private findCycleInWaitForGraph(waitForGraph: Map<string, Set<string>>): string[] | null {
        const visited = new Set<string>();
        const recursionStack = new Set<string>();
        
        for (const [processId] of waitForGraph) {
            if (!visited.has(processId)) {
                const cycle = this.dfsForCycle(processId, waitForGraph, visited, recursionStack, []);
                if (cycle) {
                    return cycle;
                }
            }
        }
        return null;
    }
    
    private dfsForCycle(node: string, graph: Map<string, Set<string>>, 
                       visited: Set<string>, recursionStack: Set<string>, 
                       path: string[]): string[] | null {
        visited.add(node);
        recursionStack.add(node);
        path.push(node);
        
        const neighbors = graph.get(node) || new Set();
        for (const neighbor of neighbors) {
            if (!visited.has(neighbor)) {
                const result = this.dfsForCycle(neighbor, graph, visited, recursionStack, path);
                if (result) return result;
            } else if (recursionStack.has(neighbor)) {
                // 순환 발견
                const cycleStart = path.indexOf(neighbor);
                return path.slice(cycleStart);
            }
        }
        
        recursionStack.delete(node);
        path.pop();
        return null;
    }
    
    // 다중 인스턴스 자원용 데드락 탐지
    detectDeadlockMultiInstance(): boolean {
        // 현재 가용 자원 계산
        const available = this.calculateAvailableResources();
        const processes = Array.from(this.nodes.entries())
            .filter(([_, node]) => node.type === 'process')
            .map(([id, _]) => id);
        
        const work = new Map(available);
        const finish = new Map<string, boolean>();
        
        // 모든 프로세스를 미완료 상태로 초기화
        for (const processId of processes) {
            finish.set(processId, false);
        }
        
        let changed = true;
        while (changed) {
            changed = false;
            
            for (const processId of processes) {
                if (!finish.get(processId)) {
                    const canFinish = this.canProcessFinish(processId, work);
                    if (canFinish) {
                        // 프로세스 완료 시뮬레이션
                        const allocation = this.getProcessAllocation(processId);
                        for (const [resourceId, count] of allocation) {
                            const currentWork = work.get(resourceId) || 0;
                            work.set(resourceId, currentWork + count);
                        }
                        finish.set(processId, true);
                        changed = true;
                    }
                }
            }
        }
        
        // 완료되지 않은 프로세스가 있으면 데드락
        return Array.from(finish.values()).some(f => !f);
    }
    
    private calculateAvailableResources(): Map<string, number> {
        const total = new Map<string, number>();
        const allocated = new Map<string, number>();
        
        // 총 자원 계산
        for (const [nodeId, node] of this.nodes) {
            if (node.type === 'resource') {
                total.set(nodeId, node.instances || 1);
            }
        }
        
        // 할당된 자원 계산
        for (const edge of this.edges.filter(e => e.type === 'assignment')) {
            const resourceId = edge.from;
            const current = allocated.get(resourceId) || 0;
            allocated.set(resourceId, current + 1);
        }
        
        // 사용 가능한 자원 = 총 자원 - 할당된 자원
        const available = new Map<string, number>();
        for (const [resourceId, totalCount] of total) {
            const allocatedCount = allocated.get(resourceId) || 0;
            available.set(resourceId, totalCount - allocatedCount);
        }
        
        return available;
    }
    
    private canProcessFinish(processId: string, available: Map<string, number>): boolean {
        const requests = this.getProcessRequests(processId);
        
        for (const [resourceId, requestCount] of requests) {
            const availableCount = available.get(resourceId) || 0;
            if (requestCount > availableCount) {
                return false;
            }
        }
        return true;
    }
    
    private getProcessRequests(processId: string): Map<string, number> {
        const requests = new Map<string, number>();
        
        for (const edge of this.edges.filter(e => 
            e.type === 'request' && e.from === processId)) {
            const resourceId = edge.to;
            const current = requests.get(resourceId) || 0;
            requests.set(resourceId, current + 1);
        }
        
        return requests;
    }
    
    private getProcessAllocation(processId: string): Map<string, number> {
        const allocation = new Map<string, number>();
        
        for (const edge of this.edges.filter(e => 
            e.type === 'assignment' && e.to === processId)) {
            const resourceId = edge.from;
            const current = allocation.get(resourceId) || 0;
            allocation.set(resourceId, current + 1);
        }
        
        return allocation;
    }
    
    // 그래프 시각화용 출력
    printGraph(): void {
        console.log('\n=== Resource Allocation Graph ===');
        console.log('Nodes:');
        for (const [id, node] of this.nodes) {
            const typeInfo = node.type === 'resource' ? ` (${node.instances} instances)` : '';
            console.log(`  ${id} [${node.type}]${typeInfo}`);
        }
        
        console.log('\nEdges:');
        for (const edge of this.edges) {
            const arrow = edge.type === 'request' ? ' --requests--> ' : ' --assigned--> ';
            console.log(`  ${edge.from}${arrow}${edge.to}`);
        }
        console.log('================================\n');
    }
}
```

### 데드락 복구 기법

```typescript
interface DeadlockRecoveryStrategy {
    recoverFromDeadlock(deadlockedProcesses: string[]): Promise<void>;
}

class ProcessTerminationRecovery implements DeadlockRecoveryStrategy {
    private processManager: ProcessManager;
    private resourceManager: ResourceManager;
    
    constructor(processManager: ProcessManager, resourceManager: ResourceManager) {
        this.processManager = processManager;
        this.resourceManager = resourceManager;
    }
    
    async recoverFromDeadlock(deadlockedProcesses: string[]): Promise<void> {
        // 전략 1: 모든 데드락 프로세스 종료
        await this.terminateAllDeadlockedProcesses(deadlockedProcesses);
        
        // 전략 2: 하나씩 종료하여 최소 비용으로 해결
        // await this.terminateMinimalProcesses(deadlockedProcesses);
    }
    
    private async terminateAllDeadlockedProcesses(processes: string[]): Promise<void> {
        console.log(`Terminating all deadlocked processes: ${processes.join(', ')}`);
        
        for (const processId of processes) {
            await this.terminateProcess(processId);
        }
    }
    
    private async terminateMinimalProcesses(processes: string[]): Promise<void> {
        // 프로세스 종료 비용 계산
        const costs = await this.calculateTerminationCosts(processes);
        
        // 비용 순으로 정렬
        const sortedProcesses = processes.sort((a, b) => costs.get(a)! - costs.get(b)!);
        
        for (const processId of sortedProcesses) {
            await this.terminateProcess(processId);
            
            // 데드락이 해결되었는지 확인
            if (!this.isStillDeadlocked()) {
                console.log(`Deadlock resolved by terminating process ${processId}`);
                break;
            }
        }
    }
    
    private async calculateTerminationCosts(processes: string[]): Promise<Map<string, number>> {
        const costs = new Map<string, number>();
        
        for (const processId of processes) {
            let cost = 0;
            
            // 프로세스 우선순위 (낮을수록 종료 비용 높음)
            const priority = await this.processManager.getPriority(processId);
            cost += (10 - priority) * 10;
            
            // 실행 시간 (오래 실행된 프로세스일수록 종료 비용 높음)
            const runtime = await this.processManager.getRuntime(processId);
            cost += runtime / 1000; // milliseconds to cost units
            
            // 보유 자원 수 (많이 보유할수록 종료 시 해제되는 자원 많음 = 비용 낮음)
            const resourceCount = await this.resourceManager.getProcessResourceCount(processId);
            cost -= resourceCount * 5;
            
            costs.set(processId, cost);
        }
        
        return costs;
    }
    
    private async terminateProcess(processId: string): Promise<void> {
        // 1. 프로세스가 보유한 모든 자원 해제
        await this.resourceManager.releaseAllResources(processId);
        
        // 2. 프로세스 종료
        await this.processManager.terminate(processId);
        
        // 3. 대기 중인 다른 프로세스들에게 알림
        await this.notifyWaitingProcesses(processId);
        
        console.log(`Process ${processId} terminated for deadlock recovery`);
    }
    
    private async notifyWaitingProcesses(terminatedProcessId: string): Promise<void> {
        const waitingProcesses = await this.resourceManager.getProcessesWaitingFor(terminatedProcessId);
        
        for (const waitingProcess of waitingProcesses) {
            // 대기 중인 프로세스를 깨워서 다시 시도하도록 함
            await this.processManager.wakeup(waitingProcess);
        }
    }
    
    private isStillDeadlocked(): boolean {
        // 데드락 탐지 알고리즘을 다시 실행하여 확인
        const graph = new ResourceAllocationGraph();
        // ... 현재 상태로 그래프 재구성
        return graph.detectDeadlockSingleInstance() !== null;
    }
}

class ResourcePreemptionRecovery implements DeadlockRecoveryStrategy {
    private resourceManager: ResourceManager;
    
    constructor(resourceManager: ResourceManager) {
        this.resourceManager = resourceManager;
    }
    
    async recoverFromDeadlock(deadlockedProcesses: string[]): Promise<void> {
        const victims = this.selectVictims(deadlockedProcesses);
        
        for (const victim of victims) {
            await this.preemptResources(victim);
            
            // 데드락이 해결되었는지 확인
            if (!this.isStillDeadlocked()) {
                break;
            }
        }
    }
    
    private selectVictims(processes: string[]): string[] {
        // 선점 비용이 낮은 프로세스들 선택
        return processes.sort((a, b) => {
            const costA = this.calculatePreemptionCost(a);
            const costB = this.calculatePreemptionCost(b);
            return costA - costB;
        });
    }
    
    private calculatePreemptionCost(processId: string): number {
        let cost = 0;
        
        // 체크포인트 이후 작업량 (rollback 비용)
        const workSinceCheckpoint = this.getWorkSinceLastCheckpoint(processId);
        cost += workSinceCheckpoint * 2;
        
        // 프로세스 우선순위
        const priority = this.getProcessPriority(processId);
        cost += (10 - priority) * 3;
        
        // 보유 자원의 중요도
        const resourceImportance = this.getResourceImportance(processId);
        cost += resourceImportance;
        
        return cost;
    }
    
    private async preemptResources(victimProcessId: string): Promise<void> {
        console.log(`Preempting resources from process ${victimProcessId}`);
        
        // 1. 체크포인트 생성 (rollback을 위해)
        await this.createCheckpoint(victimProcessId);
        
        // 2. 프로세스의 모든 자원 선점
        const preemptedResources = await this.resourceManager.preemptAllResources(victimProcessId);
        
        // 3. 프로세스를 안전한 상태로 rollback
        await this.rollbackToCheckpoint(victimProcessId);
        
        // 4. 선점된 자원들을 대기 중인 프로세스들에게 재할당
        await this.redistributeResources(preemptedResources);
    }
    
    private async createCheckpoint(processId: string): Promise<void> {
        // 프로세스 상태를 체크포인트로 저장
        console.log(`Creating checkpoint for process ${processId}`);
    }
    
    private async rollbackToCheckpoint(processId: string): Promise<void> {
        // 프로세스를 마지막 체크포인트 상태로 복구
        console.log(`Rolling back process ${processId} to last checkpoint`);
    }
    
    private async redistributeResources(resources: string[]): Promise<void> {
        for (const resourceId of resources) {
            const waitingProcesses = await this.resourceManager.getWaitingProcesses(resourceId);
            if (waitingProcesses.length > 0) {
                // 가장 우선순위가 높은 프로세스에게 할당
                const highestPriority = this.getHighestPriorityProcess(waitingProcesses);
                await this.resourceManager.allocate(resourceId, highestPriority);
            }
        }
    }
}
```

---

## 💻 TypeScript 구현

### 통합 데드락 관리 시스템

```typescript
class DeadlockManager {
    private resourceGraph: ResourceAllocationGraph;
    private banker: BankersAlgorithm | null = null;
    private recoveryStrategy: DeadlockRecoveryStrategy;
    private preventionMode: boolean = false;
    private detectionInterval: number = 5000; // 5초마다 탐지
    private detectionTimer: NodeJS.Timeout | null = null;
    
    constructor(recoveryStrategy: DeadlockRecoveryStrategy) {
        this.resourceGraph = new ResourceAllocationGraph();
        this.recoveryStrategy = recoveryStrategy;
        this.startPeriodicDetection();
    }
    
    // 예방 모드 활성화 (순서화된 자원 할당)
    enablePrevention(): void {
        this.preventionMode = true;
        console.log('Deadlock prevention mode enabled');
    }
    
    // 회피 모드 활성화 (은행원 알고리즘)
    enableAvoidance(resourceTypes: number, totalResources: number[]): void {
        this.banker = new BankersAlgorithm(resourceTypes, totalResources);
        console.log('Deadlock avoidance mode enabled with Banker\'s Algorithm');
    }
    
    // 자원 요청 처리
    async requestResource(processId: string, resourceId: string): Promise<boolean> {
        if (this.banker) {
            // 회피 모드: 은행원 알고리즘 사용
            return this.banker.requestResources(processId, this.parseResourceRequest(resourceId));
        }
        
        if (this.preventionMode) {
            // 예방 모드: 순서화된 할당 검사
            return this.checkOrderedAllocation(processId, resourceId);
        }
        
        // 일반 모드: 단순 할당 후 탐지
        return this.allocateAndDetect(processId, resourceId);
    }
    
    private parseResourceRequest(resourceId: string): number[] {
        // 실제 구현에서는 resourceId를 배열로 파싱
        // 여기서는 단순화
        return [1];
    }
    
    private checkOrderedAllocation(processId: string, resourceId: string): boolean {
        // 순서화된 자원 할당 검사 로직
        const resourceOrder = this.getResourceOrder(resourceId);
        const lastAllocatedOrder = this.getLastAllocatedResourceOrder(processId);
        
        if (resourceOrder <= lastAllocatedOrder) {
            console.log(`Order violation: ${processId} cannot request ${resourceId}`);
            return false;
        }
        
        this.resourceGraph.addRequestEdge(processId, resourceId);
        this.resourceGraph.addAssignmentEdge(resourceId, processId);
        return true;
    }
    
    private async allocateAndDetect(processId: string, resourceId: string): Promise<boolean> {
        // 자원 할당
        this.resourceGraph.addAssignmentEdge(resourceId, processId);
        
        // 즉시 데드락 탐지
        const deadlock = this.detectDeadlock();
        if (deadlock.length > 0) {
            console.log(`Deadlock detected involving: ${deadlock.join(', ')}`);
            
            // 할당 취소
            this.resourceGraph.removeAssignmentEdge(resourceId, processId);
            
            return false;
        }
        
        return true;
    }
    
    // 주기적 데드락 탐지
    private startPeriodicDetection(): void {
        this.detectionTimer = setInterval(() => {
            const deadlock = this.detectDeadlock();
            if (deadlock.length > 0) {
                console.log(`Periodic detection found deadlock: ${deadlock.join(', ')}`);
                this.handleDeadlock(deadlock);
            }
        }, this.detectionInterval);
    }
    
    private detectDeadlock(): string[] {
        const cycle = this.resourceGraph.detectDeadlockSingleInstance();
        return cycle || [];
    }
    
    private async handleDeadlock(deadlockedProcesses: string[]): Promise<void> {
        console.log(`Handling deadlock with ${deadlockedProcesses.length} processes`);
        await this.recoveryStrategy.recoverFromDeadlock(deadlockedProcesses);
    }
    
    // 자원 해제
    releaseResource(processId: string, resourceId: string): void {
        this.resourceGraph.removeAssignmentEdge(resourceId, processId);
        
        if (this.banker) {
            this.banker.releaseResources(processId, this.parseResourceRequest(resourceId));
        }
        
        console.log(`Resource ${resourceId} released by process ${processId}`);
    }
    
    // 시스템 상태 모니터링
    getSystemStatus(): any {
        return {
            preventionMode: this.preventionMode,
            avoidanceMode: this.banker !== null,
            detectionInterval: this.detectionInterval,
            currentDeadlock: this.detectDeadlock()
        };
    }
    
    // 정리
    shutdown(): void {
        if (this.detectionTimer) {
            clearInterval(this.detectionTimer);
        }
        console.log('Deadlock manager shutdown');
    }
}
```

---

## 📊 성능 분석

### 데드락 관리 기법별 성능 비교

```typescript
class DeadlockPerformanceAnalyzer {
    private metrics: Map<string, any[]> = new Map();
    
    async benchmarkDeadlockStrategies(): Promise<void> {
        console.log('🔍 데드락 관리 기법별 성능 분석\n');
        
        const strategies = [
            { name: 'Prevention (Ordered)', strategy: 'prevention' },
            { name: 'Avoidance (Banker)', strategy: 'avoidance' },
            { name: 'Detection & Recovery', strategy: 'detection' }
        ];
        
        const workloads = [
            { name: 'Light Load', processes: 5, resources: 3, requests: 20 },
            { name: 'Medium Load', processes: 10, resources: 5, requests: 50 },
            { name: 'Heavy Load', processes: 20, resources: 8, requests: 100 }
        ];
        
        for (const workload of workloads) {
            console.log(`\n📊 ${workload.name} 워크로드 분석:`);
            console.log(`   프로세스: ${workload.processes}, 자원: ${workload.resources}, 요청: ${workload.requests}`);
            
            for (const strategy of strategies) {
                const result = await this.measureStrategy(strategy, workload);
                this.recordMetrics(strategy.name, workload.name, result);
                
                console.log(`\n   ${strategy.name}:`);
                console.log(`     처리량: ${result.throughput.toFixed(2)} req/sec`);
                console.log(`     평균 응답시간: ${result.avgResponseTime.toFixed(2)} ms`);
                console.log(`     데드락 발생률: ${(result.deadlockRate * 100).toFixed(2)}%`);
                console.log(`     오버헤드: ${(result.overhead * 100).toFixed(2)}%`);
            }
        }
        
        this.printSummary();
    }
    
    private async measureStrategy(strategy: any, workload: any): Promise<any> {
        const startTime = Date.now();
        let completedRequests = 0;
        let deadlockCount = 0;
        let totalResponseTime = 0;
        
        // 워크로드 시뮬레이션
        const results = await this.simulateWorkload(strategy.strategy, workload);
        
        const endTime = Date.now();
        const totalTime = (endTime - startTime) / 1000; // seconds
        
        return {
            throughput: completedRequests / totalTime,
            avgResponseTime: results.avgResponseTime,
            deadlockRate: results.deadlockCount / workload.requests,
            overhead: results.overhead,
            resourceUtilization: results.resourceUtilization
        };
    }
    
    private async simulateWorkload(strategyType: string, workload: any): Promise<any> {
        switch (strategyType) {
            case 'prevention':
                return this.simulatePrevention(workload);
            case 'avoidance':
                return this.simulateAvoidance(workload);
            case 'detection':
                return this.simulateDetection(workload);
            default:
                throw new Error(`Unknown strategy: ${strategyType}`);
        }
    }
    
    private async simulatePrevention(workload: any): Promise<any> {
        // 순서화된 자원 할당 시뮬레이션
        let rejectedRequests = 0;
        let totalResponseTime = 0;
        
        // 자원에 순서 부여
        const resourceOrder = new Map<string, number>();
        for (let i = 0; i < workload.resources; i++) {
            resourceOrder.set(`R${i}`, i + 1);
        }
        
        // 프로세스별 요청 시뮬레이션
        for (let p = 0; p < workload.processes; p++) {
            const processId = `P${p}`;
            let lastOrder = 0;
            
            for (let r = 0; r < Math.floor(workload.requests / workload.processes); r++) {
                const resourceId = `R${Math.floor(Math.random() * workload.resources)}`;
                const resourceOrderNum = resourceOrder.get(resourceId)!;
                
                const startTime = Date.now();
                
                if (resourceOrderNum <= lastOrder) {
                    rejectedRequests++;
                } else {
                    lastOrder = resourceOrderNum;
                }
                
                totalResponseTime += (Date.now() - startTime);
            }
        }
        
        return {
            avgResponseTime: totalResponseTime / workload.requests,
            deadlockCount: 0, // 예방 기법은 데드락 발생 없음
            overhead: rejectedRequests / workload.requests * 0.1, // 거부된 요청으로 인한 오버헤드
            resourceUtilization: 0.7 // 순서 제약으로 인한 낮은 활용도
        };
    }
    
    private async simulateAvoidance(workload: any): Promise<any> {
        // 은행원 알고리즘 시뮬레이션
        let safeRequests = 0;
        let unsafeRequests = 0;
        let totalResponseTime = 0;
        
        // 각 요청에 대해 안전성 검사 시뮬레이션
        for (let i = 0; i < workload.requests; i++) {
            const startTime = Date.now();
            
            // 안전성 검사 시간 (복잡도 O(n²m))
            const safetyCheckTime = Math.pow(workload.processes, 2) * workload.resources * 0.01;
            await this.sleep(safetyCheckTime);
            
            // 80% 확률로 안전한 요청
            if (Math.random() < 0.8) {
                safeRequests++;
            } else {
                unsafeRequests++;
            }
            
            totalResponseTime += (Date.now() - startTime);
        }
        
        return {
            avgResponseTime: totalResponseTime / workload.requests,
            deadlockCount: 0, // 회피 기법은 데드락 발생 없음
            overhead: unsafeRequests / workload.requests * 0.15, // 안전성 검사 오버헤드
            resourceUtilization: 0.85 // 보수적 할당으로 인한 활용도
        };
    }
    
    private async simulateDetection(workload: any): Promise<any> {
        // 탐지 및 복구 시뮬레이션
        let deadlockCount = 0;
        let recoveryTime = 0;
        let totalResponseTime = 0;
        
        // 데드락 탐지 간격 (예: 1초)
        const detectionInterval = 1000;
        const detectionCost = workload.processes * workload.resources * 0.1; // O(nm)
        
        // 전체 실행 시간 추정
        const estimatedRunTime = workload.requests * 10; // 평균 10ms per request
        const detectionRuns = Math.ceil(estimatedRunTime / detectionInterval);
        
        // 데드락 발생 확률 (5%)
        deadlockCount = Math.floor(workload.requests * 0.05);
        
        // 복구 시간 (프로세스 재시작 비용)
        recoveryTime = deadlockCount * 50; // 50ms per recovery
        
        totalResponseTime = workload.requests * 8 + // 일반 요청 처리 시간
                          detectionRuns * detectionCost + // 탐지 비용
                          recoveryTime; // 복구 비용
        
        return {
            avgResponseTime: totalResponseTime / workload.requests,
            deadlockCount: deadlockCount,
            overhead: (detectionRuns * detectionCost + recoveryTime) / totalResponseTime,
            resourceUtilization: 0.95 // 높은 자원 활용도
        };
    }
    
    private sleep(ms: number): Promise<void> {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
    
    private recordMetrics(strategy: string, workload: string, result: any): void {
        if (!this.metrics.has(strategy)) {
            this.metrics.set(strategy, []);
        }
        this.metrics.get(strategy)!.push({ workload, result });
    }
    
    private printSummary(): void {
        console.log('\n📈 성능 분석 요약:');
        console.log('==========================================');
        
        console.log('\n🎯 처리량 (requests/second):');
        console.log('   Prevention: 높음 (거부된 요청 제외)');
        console.log('   Avoidance:  중간 (안전성 검사 오버헤드)');
        console.log('   Detection:  높음 (탐지 간격에 따라 변동)');
        
        console.log('\n⏱️ 응답 시간:');
        console.log('   Prevention: 낮음 (즉시 거부/승인)');
        console.log('   Avoidance:  높음 (복잡한 안전성 검사)');
        console.log('   Detection:  낮음 (탐지는 백그라운드)');
        
        console.log('\n🔒 데드락 방지 효과:');
        console.log('   Prevention: 100% (발생 불가능)');
        console.log('   Avoidance:  100% (안전 상태 유지)');
        console.log('   Detection:  복구 의존 (발생 후 처리)');
        
        console.log('\n💰 자원 활용도:');
        console.log('   Prevention: 낮음 (순서 제약)');
        console.log('   Avoidance:  중간 (보수적 할당)');
        console.log('   Detection:  높음 (적극적 할당)');
        
        console.log('\n🏗️ 구현 복잡도:');
        console.log('   Prevention: 낮음 (간단한 순서 검사)');
        console.log('   Avoidance:  높음 (복잡한 알고리즘)');
        console.log('   Detection:  중간 (그래프 탐지 + 복구)');
    }
}
```

---

## 🏢 실제 시스템 사례

### Linux 커널의 데드락 방지

```typescript
// Linux lockdep (Lock Dependency Tracking) 시스템 시뮬레이션
class LinuxLockdep {
    private lockClasses: Map<string, LockClass> = new Map();
    private lockChains: LockChain[] = [];
    private warningsEnabled: boolean = true;
    
    // 락 클래스 등록
    registerLockClass(name: string, type: 'spinlock' | 'mutex' | 'rwlock'): void {
        this.lockClasses.set(name, {
            name,
            type,
            acquisitionCount: 0,
            contentionCount: 0
        });
    }
    
    // 락 획득 추적
    trackLockAcquisition(lockName: string, context: string): void {
        const lockClass = this.lockClasses.get(lockName);
        if (!lockClass) {
            console.warn(`Unknown lock class: ${lockName}`);
            return;
        }
        
        lockClass.acquisitionCount++;
        
        // 락 순서 추적
        this.updateLockChain(lockName, context);
        
        // 순환 의존성 검사
        if (this.detectLockInversion()) {
            this.reportLockingBug(lockName, context);
        }
    }
    
    private updateLockChain(lockName: string, context: string): void {
        const currentTime = Date.now();
        
        // 현재 컨텍스트의 락 체인 찾기
        let chain = this.lockChains.find(c => c.context === context);
        if (!chain) {
            chain = {
                context,
                locks: [],
                timestamp: currentTime
            };
            this.lockChains.push(chain);
        }
        
        chain.locks.push({
            name: lockName,
            timestamp: currentTime
        });
        
        // 오래된 체인 정리 (메모리 절약)
        this.cleanupOldChains();
    }
    
    private detectLockInversion(): boolean {
        // ABBA 데드락 패턴 검사
        const lockPairs = new Map<string, Set<string>>();
        
        for (const chain of this.lockChains) {
            for (let i = 0; i < chain.locks.length - 1; i++) {
                const lockA = chain.locks[i].name;
                const lockB = chain.locks[i + 1].name;
                
                if (!lockPairs.has(lockA)) {
                    lockPairs.set(lockA, new Set());
                }
                lockPairs.get(lockA)!.add(lockB);
            }
        }
        
        // 순환 검사
        for (const [lockA, followers] of lockPairs) {
            if (this.hasCircularDependency(lockA, followers, lockPairs, new Set())) {
                return true;
            }
        }
        
        return false;
    }
    
    private hasCircularDependency(start: string, followers: Set<string>,
                                 allPairs: Map<string, Set<string>>,
                                 visited: Set<string>): boolean {
        if (visited.has(start)) {
            return true; // 순환 발견
        }
        
        visited.add(start);
        
        for (const follower of followers) {
            const nextFollowers = allPairs.get(follower);
            if (nextFollowers && this.hasCircularDependency(follower, nextFollowers, allPairs, visited)) {
                return true;
            }
        }
        
        visited.delete(start);
        return false;
    }
    
    private reportLockingBug(lockName: string, context: string): void {
        if (!this.warningsEnabled) return;
        
        console.error('🚨 LOCKDEP WARNING: Potential deadlock detected!');
        console.error(`   Lock: ${lockName}`);
        console.error(`   Context: ${context}`);
        console.error('   Possible unsafe locking scenario:');
        
        // 문제가 되는 락 순서 출력
        const problematicChains = this.findProblematicChains(lockName);
        for (const chain of problematicChains) {
            console.error(`     Chain: ${chain.locks.map(l => l.name).join(' -> ')}`);
        }
        
        console.error('   Consider reordering lock acquisitions to prevent deadlock');
    }
    
    private findProblematicChains(lockName: string): LockChain[] {
        return this.lockChains.filter(chain => 
            chain.locks.some(lock => lock.name === lockName)
        ).slice(0, 5); // 최대 5개 체인만 출력
    }
}

interface LockClass {
    name: string;
    type: 'spinlock' | 'mutex' | 'rwlock';
    acquisitionCount: number;
    contentionCount: number;
}

interface LockChain {
    context: string;
    locks: { name: string; timestamp: number }[];
    timestamp: number;
}
```

### 데이터베이스 데드락 처리

```typescript
class DatabaseDeadlockManager {
    private transactionGraph: Map<string, Set<string>> = new Map();
    private lockTable: Map<string, LockInfo> = new Map();
    private deadlockDetectionInterval: number = 1000; // 1초
    
    // 락 요청 처리
    async requestLock(transactionId: string, resourceId: string, 
                     lockType: 'shared' | 'exclusive'): Promise<boolean> {
        const lockKey = `${resourceId}:${lockType}`;
        const currentLock = this.lockTable.get(lockKey);
        
        if (!currentLock) {
            // 락이 비어있으면 즉시 획득
            this.grantLock(transactionId, resourceId, lockType);
            return true;
        }
        
        // 락 호환성 검사
        if (this.isLockCompatible(currentLock, lockType)) {
            this.grantLock(transactionId, resourceId, lockType);
            return true;
        }
        
        // 대기 관계 생성
        this.addWaitRelation(transactionId, currentLock.holders);
        
        // 데드락 검사
        if (this.detectDeadlock()) {
            const victim = this.selectVictim();
            await this.abortTransaction(victim);
            
            // 대기 관계 제거 후 다시 시도
            this.removeWaitRelation(transactionId);
            return this.requestLock(transactionId, resourceId, lockType);
        }
        
        // 대기 큐에 추가
        this.addToWaitQueue(transactionId, resourceId, lockType);
        return false; // 대기 상태
    }
    
    private isLockCompatible(currentLock: LockInfo, requestedType: string): boolean {
        // 락 호환성 매트릭스
        const compatibilityMatrix = {
            'shared-shared': true,
            'shared-exclusive': false,
            'exclusive-shared': false,
            'exclusive-exclusive': false
        };
        
        const key = `${currentLock.type}-${requestedType}`;
        return compatibilityMatrix[key as keyof typeof compatibilityMatrix] || false;
    }
    
    private detectDeadlock(): boolean {
        // Floyd-Warshall 알고리즘을 사용한 데드락 탐지
        const transactions = Array.from(this.transactionGraph.keys());
        const n = transactions.length;
        
        if (n === 0) return false;
        
        // 인접 행렬 생성
        const adj: boolean[][] = Array(n).fill(null).map(() => Array(n).fill(false));
        
        for (let i = 0; i < n; i++) {
            const txnId = transactions[i];
            const waitingFor = this.transactionGraph.get(txnId) || new Set();
            
            for (const waitedTxn of waitingFor) {
                const j = transactions.indexOf(waitedTxn);
                if (j !== -1) {
                    adj[i][j] = true;
                }
            }
        }
        
        // Floyd-Warshall로 경로 존재성 확인
        for (let k = 0; k < n; k++) {
            for (let i = 0; i < n; i++) {
                for (let j = 0; j < n; j++) {
                    adj[i][j] = adj[i][j] || (adj[i][k] && adj[k][j]);
                }
            }
        }
        
        // 자기 자신으로의 경로가 있으면 사이클 존재
        for (let i = 0; i < n; i++) {
            if (adj[i][i]) {
                console.log(`Deadlock detected involving transaction ${transactions[i]}`);
                return true;
            }
        }
        
        return false;
    }
    
    private selectVictim(): string {
        // 희생자 선택 전략: 최소 비용
        let minCost = Infinity;
        let victim = '';
        
        for (const [txnId] of this.transactionGraph) {
            const cost = this.calculateAbortCost(txnId);
            if (cost < minCost) {
                minCost = cost;
                victim = txnId;
            }
        }
        
        return victim;
    }
    
    private calculateAbortCost(transactionId: string): number {
        // 트랜잭션 abort 비용 계산
        let cost = 0;
        
        // 트랜잭션 실행 시간 (오래 실행된 트랜잭션일수록 비용 높음)
        const runtime = this.getTransactionRuntime(transactionId);
        cost += runtime * 0.1;
        
        // 수행한 작업량 (많은 작업을 한 트랜잭션일수록 비용 높음)
        const workDone = this.getTransactionWorkDone(transactionId);
        cost += workDone * 0.5;
        
        // 트랜잭션 우선순위 (높은 우선순위일수록 비용 높음)
        const priority = this.getTransactionPriority(transactionId);
        cost += (10 - priority) * 2;
        
        return cost;
    }
    
    private async abortTransaction(transactionId: string): Promise<void> {
        console.log(`🔄 Aborting transaction ${transactionId} to resolve deadlock`);
        
        // 1. 트랜잭션이 보유한 모든 락 해제
        this.releaseAllLocks(transactionId);
        
        // 2. 트랜잭션 롤백
        await this.rollbackTransaction(transactionId);
        
        // 3. 대기 관계 제거
        this.removeWaitRelation(transactionId);
        
        // 4. 대기 중인 트랜잭션들 깨우기
        this.wakeupWaitingTransactions();
    }
    
    private async rollbackTransaction(transactionId: string): Promise<void> {
        // 트랜잭션 로그를 사용하여 롤백
        console.log(`Rolling back transaction ${transactionId}`);
        
        // 실제 구현에서는 데이터베이스의 변경사항을 되돌림
        // - Undo log entries in reverse order
        // - Release all acquired locks
        // - Clean up transaction state
    }
    
    // 트랜잭션 통계 출력
    printDeadlockStatistics(): void {
        console.log('\n📊 데드락 통계:');
        console.log('===================');
        console.log(`활성 트랜잭션 수: ${this.transactionGraph.size}`);
        console.log(`대기 중인 락 요청 수: ${this.getWaitingRequestCount()}`);
        console.log(`현재 보유 락 수: ${this.lockTable.size}`);
        
        // 대기 그래프 출력
        if (this.transactionGraph.size > 0) {
            console.log('\n대기 그래프:');
            for (const [txn, waitingFor] of this.transactionGraph) {
                if (waitingFor.size > 0) {
                    console.log(`  ${txn} -> [${Array.from(waitingFor).join(', ')}]`);
                }
            }
        }
    }
}

interface LockInfo {
    holders: Set<string>;    // 락을 보유한 트랜잭션들
    waiters: string[];       // 대기 중인 트랜잭션들
    type: 'shared' | 'exclusive';
    resourceId: string;
}
```

---

## 📚 참고자료

### 학술 자료
- Coffman, E. G., Jr., Elphick, M. J., & Shoshani, A. (1971). "System deadlocks"
- Dijkstra, E. W. (1965). "Solution of a problem in concurrent programming control"
- Holt, R. C. (1972). "Some deadlock properties of computer systems"

### 실무 가이드
- [[Linux Kernel Lock Validation]]
- [[Database Concurrency Control]]
- [[Java Concurrent Programming]]

### 관련 문서
- [[03_process-management]] - 프로세스 동기화
- [[04_thread-synchronization]] - 스레드 동기화
- [[10_synchronization-primitives]] - 동기화 프리미티브

---

> [!success] 데드락 관리 핵심 정리
> 1. **예방**: 4가지 조건 중 하나를 제거
> 2. **회피**: 은행원 알고리즘으로 안전 상태 유지  
> 3. **탐지**: 주기적 검사 후 복구
> 4. **복구**: 프로세스 종료 또는 자원 선점
> 5. **실무**: 시스템 특성에 맞는 전략 선택이 중요

> [!quote]
> "데드락은 완전히 피할 수는 없지만, 적절한 설계와 구현으로 그 영향을 최소화할 수 있다. 성능과 안전성 간의 균형을 찾는 것이 핵심이다." - 운영체제 설계 원칙