---
tags:
  - os
  - cache
  - algorithms
  - memory-management
created: 2025-01-08
updated: 2025-01-08
---

# 캐시 알고리즘

> [!info] 캐시 알고리즘이란?
> 캐시 알고리즘은 제한된 캐시 공간에서 어떤 데이터를 유지하고 어떤 데이터를 제거할지 결정하는 전략입니다. 효율적인 캐시 알고리즘은 캐시 적중률을 높여 시스템 성능을 크게 향상시킵니다.

## 📑 목차

- [[#🎯 캐시 기본 개념]]
- [[#📊 캐시 교체 알고리즘]]
- [[#🔄 FIFO (First In First Out)]]
- [[#⚡ LRU (Least Recently Used)]]
- [[#📈 LFU (Least Frequently Used)]]
- [[#🎲 Random & OPT]]
- [[#💡 TypeScript 구현]]
- [[#🔍 실제 사례]]

---

## 🎯 캐시 기본 개념

### 캐시 성능 지표

```typescript
class CacheMetrics {
  private hits: number = 0;
  private misses: number = 0;
  private evictions: number = 0;
  private totalAccessTime: number = 0;

  // 캐시 적중
  recordHit(accessTime: number): void {
    this.hits++;
    this.totalAccessTime += accessTime;
  }

  // 캐시 미스
  recordMiss(accessTime: number): void {
    this.misses++;
    this.totalAccessTime += accessTime;
  }

  // 캐시 제거
  recordEviction(): void {
    this.evictions++;
  }

  // 적중률 계산
  getHitRate(): number {
    const total = this.hits + this.misses;
    return total > 0 ? this.hits / total : 0;
  }

  // 미스율 계산
  getMissRate(): number {
    return 1 - this.getHitRate();
  }

  // 평균 접근 시간
  getAverageAccessTime(): number {
    const total = this.hits + this.misses;
    return total > 0 ? this.totalAccessTime / total : 0;
  }

  // 통계 출력
  printStatistics(): void {
    console.log('\n=== Cache Performance Metrics ===');
    console.log(`Total Accesses: ${this.hits + this.misses}`);
    console.log(`Hits: ${this.hits} (${(this.getHitRate() * 100).toFixed(2)}%)`);
    console.log(`Misses: ${this.misses} (${(this.getMissRate() * 100).toFixed(2)}%)`);
    console.log(`Evictions: ${this.evictions}`);
    console.log(`Average Access Time: ${this.getAverageAccessTime().toFixed(2)}ms`);
  }
}

// 지역성 원리
class LocalityPrinciple {
  // 시간적 지역성 (Temporal Locality)
  static demonstrateTemporalLocality(): void {
    console.log('Temporal Locality: Recently accessed data is likely to be accessed again');
    
    class TemporalCache {
      private recentAccess: Map<string, number> = new Map();
      
      access(key: string): void {
        const now = Date.now();
        const lastAccess = this.recentAccess.get(key);
        
        if (lastAccess) {
          const timeDiff = now - lastAccess;
          console.log(`Key ${key} accessed again after ${timeDiff}ms`);
        }
        
        this.recentAccess.set(key, now);
      }
      
      predictNextAccess(key: string): number {
        // 최근 접근 패턴 기반 예측
        const lastAccess = this.recentAccess.get(key);
        if (!lastAccess) return Infinity;
        
        const timeSinceLastAccess = Date.now() - lastAccess;
        // 접근 간격이 짧을수록 다시 접근할 확률이 높음
        return 1 / (timeSinceLastAccess + 1);
      }
    }
    
    const cache = new TemporalCache();
    cache.access('data1');
    setTimeout(() => cache.access('data1'), 10);  // 곧 다시 접근
    setTimeout(() => cache.access('data1'), 20);  // 또 다시 접근
  }

  // 공간적 지역성 (Spatial Locality)
  static demonstrateSpatialLocality(): void {
    console.log('Spatial Locality: Nearby data is likely to be accessed together');
    
    class SpatialCache {
      private cache: Map<number, any[]> = new Map();
      private lineSize: number = 64;  // 캐시 라인 크기
      
      // 프리페칭: 인접 데이터도 함께 로드
      prefetch(address: number): void {
        const lineStart = Math.floor(address / this.lineSize) * this.lineSize;
        const lineEnd = lineStart + this.lineSize;
        
        console.log(`Prefetching addresses ${lineStart} to ${lineEnd - 1}`);
        
        const data = [];
        for (let addr = lineStart; addr < lineEnd; addr++) {
          data.push(`Data at ${addr}`);
        }
        
        this.cache.set(lineStart, data);
      }
      
      access(address: number): any {
        const lineStart = Math.floor(address / this.lineSize) * this.lineSize;
        const offset = address - lineStart;
        
        const line = this.cache.get(lineStart);
        if (line) {
          console.log(`Cache hit for address ${address} (line ${lineStart})`);
          return line[offset];
        }
        
        console.log(`Cache miss for address ${address}`);
        this.prefetch(address);
        return this.cache.get(lineStart)![offset];
      }
    }
    
    const cache = new SpatialCache();
    cache.access(100);  // 미스, 64-127 프리페치
    cache.access(101);  // 히트 (같은 캐시 라인)
    cache.access(102);  // 히트 (같은 캐시 라인)
  }
}
```

---

## 📊 캐시 교체 알고리즘

### 기본 캐시 인터페이스

```typescript
interface CacheAlgorithm<K, V> {
  get(key: K): V | undefined;
  put(key: K, value: V): void;
  remove(key: K): boolean;
  clear(): void;
  size(): number;
  getHitRate(): number;
}

abstract class BaseCache<K, V> implements CacheAlgorithm<K, V> {
  protected capacity: number;
  protected metrics: CacheMetrics;
  
  constructor(capacity: number) {
    this.capacity = capacity;
    this.metrics = new CacheMetrics();
  }
  
  abstract get(key: K): V | undefined;
  abstract put(key: K, value: V): void;
  abstract remove(key: K): boolean;
  abstract clear(): void;
  abstract size(): number;
  
  getHitRate(): number {
    return this.metrics.getHitRate();
  }
  
  printMetrics(): void {
    this.metrics.printStatistics();
  }
}
```

---

## 🔄 FIFO (First In First Out)

### FIFO 구현

```typescript
class FIFOCache<K, V> extends BaseCache<K, V> {
  private queue: K[] = [];
  private cache: Map<K, V> = new Map();

  get(key: K): V | undefined {
    const startTime = performance.now();
    const value = this.cache.get(key);
    
    if (value !== undefined) {
      this.metrics.recordHit(performance.now() - startTime);
    } else {
      this.metrics.recordMiss(performance.now() - startTime);
    }
    
    return value;
  }

  put(key: K, value: V): void {
    // 이미 존재하는 키인 경우 값만 업데이트
    if (this.cache.has(key)) {
      this.cache.set(key, value);
      return;
    }

    // 캐시가 가득 찬 경우
    if (this.cache.size >= this.capacity) {
      this.evict();
    }

    // 새 항목 추가
    this.cache.set(key, value);
    this.queue.push(key);
  }

  private evict(): void {
    // 가장 오래된 항목 제거 (큐의 첫 번째 요소)
    const oldestKey = this.queue.shift();
    if (oldestKey !== undefined) {
      this.cache.delete(oldestKey);
      this.metrics.recordEviction();
      console.log(`FIFO: Evicted ${oldestKey}`);
    }
  }

  remove(key: K): boolean {
    if (this.cache.delete(key)) {
      const index = this.queue.indexOf(key);
      if (index > -1) {
        this.queue.splice(index, 1);
      }
      return true;
    }
    return false;
  }

  clear(): void {
    this.cache.clear();
    this.queue = [];
  }

  size(): number {
    return this.cache.size;
  }

  // FIFO 특성 분석
  analyzePerformance(accessPattern: K[]): void {
    console.log('\n=== FIFO Cache Analysis ===');
    console.log('Characteristics:');
    console.log('- Simple to implement');
    console.log('- O(1) time complexity for all operations');
    console.log('- Poor performance with temporal locality');
    console.log('- Can suffer from Belady\'s anomaly');
    
    // Belady's Anomaly 시뮬레이션
    this.demonstrateBeladysAnomaly();
  }

  private demonstrateBeladysAnomaly(): void {
    console.log('\nBelady\'s Anomaly Demonstration:');
    
    const pattern = [1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5];
    
    // 3개 프레임
    const cache3 = new FIFOCache<number, number>(3);
    let misses3 = 0;
    for (const page of pattern) {
      if (cache3.get(page) === undefined) {
        misses3++;
        cache3.put(page, page);
      }
    }
    
    // 4개 프레임 (더 많은 프레임인데 미스가 더 많을 수 있음)
    const cache4 = new FIFOCache<number, number>(4);
    let misses4 = 0;
    for (const page of pattern) {
      if (cache4.get(page) === undefined) {
        misses4++;
        cache4.put(page, page);
      }
    }
    
    console.log(`3 frames: ${misses3} misses`);
    console.log(`4 frames: ${misses4} misses`);
    if (misses4 > misses3) {
      console.log('Belady\'s Anomaly detected!');
    }
  }
}
```

---

## ⚡ LRU (Least Recently Used)

### LRU 구현

```typescript
class LRUCache<K, V> extends BaseCache<K, V> {
  private cache: Map<K, LRUNode<K, V>> = new Map();
  private head: LRUNode<K, V> | null = null;
  private tail: LRUNode<K, V> | null = null;

  get(key: K): V | undefined {
    const startTime = performance.now();
    const node = this.cache.get(key);
    
    if (node) {
      // 노드를 리스트 앞으로 이동 (최근 사용)
      this.moveToFront(node);
      this.metrics.recordHit(performance.now() - startTime);
      return node.value;
    }
    
    this.metrics.recordMiss(performance.now() - startTime);
    return undefined;
  }

  put(key: K, value: V): void {
    const existingNode = this.cache.get(key);
    
    if (existingNode) {
      // 기존 노드 업데이트
      existingNode.value = value;
      this.moveToFront(existingNode);
      return;
    }

    // 캐시가 가득 찬 경우
    if (this.cache.size >= this.capacity) {
      this.evictLRU();
    }

    // 새 노드 추가
    const newNode = new LRUNode(key, value);
    this.cache.set(key, newNode);
    this.addToFront(newNode);
  }

  private moveToFront(node: LRUNode<K, V>): void {
    if (node === this.head) return;
    
    // 노드를 현재 위치에서 제거
    this.removeNode(node);
    
    // 노드를 앞에 추가
    this.addToFront(node);
  }

  private addToFront(node: LRUNode<K, V>): void {
    node.next = this.head;
    node.prev = null;
    
    if (this.head) {
      this.head.prev = node;
    }
    
    this.head = node;
    
    if (!this.tail) {
      this.tail = node;
    }
  }

  private removeNode(node: LRUNode<K, V>): void {
    if (node.prev) {
      node.prev.next = node.next;
    } else {
      this.head = node.next;
    }
    
    if (node.next) {
      node.next.prev = node.prev;
    } else {
      this.tail = node.prev;
    }
  }

  private evictLRU(): void {
    if (!this.tail) return;
    
    const lruNode = this.tail;
    this.cache.delete(lruNode.key);
    this.removeNode(lruNode);
    this.metrics.recordEviction();
    console.log(`LRU: Evicted ${lruNode.key}`);
  }

  remove(key: K): boolean {
    const node = this.cache.get(key);
    if (node) {
      this.cache.delete(key);
      this.removeNode(node);
      return true;
    }
    return false;
  }

  clear(): void {
    this.cache.clear();
    this.head = null;
    this.tail = null;
  }

  size(): number {
    return this.cache.size;
  }

  // LRU 최적화: 2Q 알고리즘
  static create2QCache<K, V>(capacity: number): TwoQueueCache<K, V> {
    return new TwoQueueCache(capacity);
  }
}

class LRUNode<K, V> {
  key: K;
  value: V;
  prev: LRUNode<K, V> | null = null;
  next: LRUNode<K, V> | null = null;

  constructor(key: K, value: V) {
    this.key = key;
    this.value = value;
  }
}

// 2Q 알고리즘 (LRU 개선)
class TwoQueueCache<K, V> extends BaseCache<K, V> {
  private a1in: LRUCache<K, V>;  // 최근 접근 큐
  private a1out: Map<K, null>;   // 최근 제거된 키 추적
  private am: LRUCache<K, V>;    // 주 캐시 (자주 접근되는 데이터)
  
  constructor(capacity: number) {
    super(capacity);
    const a1inSize = Math.floor(capacity * 0.25);
    const amSize = capacity - a1inSize;
    
    this.a1in = new LRUCache(a1inSize);
    this.a1out = new Map();
    this.am = new LRUCache(amSize);
  }

  get(key: K): V | undefined {
    // Am에서 먼저 확인 (자주 사용되는 데이터)
    let value = this.am.get(key);
    if (value !== undefined) {
      return value;
    }
    
    // A1in에서 확인
    value = this.a1in.get(key);
    if (value !== undefined) {
      return value;
    }
    
    return undefined;
  }

  put(key: K, value: V): void {
    // Am에 있으면 업데이트
    if (this.am.get(key) !== undefined) {
      this.am.put(key, value);
      return;
    }
    
    // A1out에 있으면 Am으로 승격
    if (this.a1out.has(key)) {
      this.a1out.delete(key);
      this.am.put(key, value);
      return;
    }
    
    // 새 항목은 A1in에 추가
    if (this.a1in.size() >= this.a1in.capacity) {
      // A1in에서 제거된 항목을 A1out에 추가
      // 실제 구현에서는 제거된 키를 추적해야 함
    }
    
    this.a1in.put(key, value);
  }

  remove(key: K): boolean {
    return this.a1in.remove(key) || this.am.remove(key);
  }

  clear(): void {
    this.a1in.clear();
    this.a1out.clear();
    this.am.clear();
  }

  size(): number {
    return this.a1in.size() + this.am.size();
  }
}
```

---

## 📈 LFU (Least Frequently Used)

### LFU 구현

```typescript
class LFUCache<K, V> extends BaseCache<K, V> {
  private cache: Map<K, LFUNode<K, V>> = new Map();
  private frequencyMap: Map<number, Set<K>> = new Map();
  private minFrequency: number = 0;

  get(key: K): V | undefined {
    const startTime = performance.now();
    const node = this.cache.get(key);
    
    if (node) {
      this.updateFrequency(node);
      this.metrics.recordHit(performance.now() - startTime);
      return node.value;
    }
    
    this.metrics.recordMiss(performance.now() - startTime);
    return undefined;
  }

  put(key: K, value: V): void {
    if (this.capacity === 0) return;

    const existingNode = this.cache.get(key);
    
    if (existingNode) {
      // 기존 노드 업데이트
      existingNode.value = value;
      this.updateFrequency(existingNode);
      return;
    }

    // 캐시가 가득 찬 경우
    if (this.cache.size >= this.capacity) {
      this.evictLFU();
    }

    // 새 노드 추가
    const newNode = new LFUNode(key, value);
    this.cache.set(key, newNode);
    
    // 빈도 1로 추가
    if (!this.frequencyMap.has(1)) {
      this.frequencyMap.set(1, new Set());
    }
    this.frequencyMap.get(1)!.add(key);
    this.minFrequency = 1;
  }

  private updateFrequency(node: LFUNode<K, V>): void {
    const oldFreq = node.frequency;
    const newFreq = oldFreq + 1;
    
    // 이전 빈도에서 제거
    const oldFreqSet = this.frequencyMap.get(oldFreq);
    if (oldFreqSet) {
      oldFreqSet.delete(node.key);
      if (oldFreqSet.size === 0) {
        this.frequencyMap.delete(oldFreq);
        if (this.minFrequency === oldFreq) {
          this.minFrequency++;
        }
      }
    }
    
    // 새 빈도에 추가
    if (!this.frequencyMap.has(newFreq)) {
      this.frequencyMap.set(newFreq, new Set());
    }
    this.frequencyMap.get(newFreq)!.add(node.key);
    
    node.frequency = newFreq;
  }

  private evictLFU(): void {
    const minFreqSet = this.frequencyMap.get(this.minFrequency);
    if (!minFreqSet || minFreqSet.size === 0) return;
    
    // 같은 빈도에서는 LRU 정책 사용 (첫 번째 요소가 가장 오래됨)
    const keyToEvict = minFreqSet.values().next().value;
    
    minFreqSet.delete(keyToEvict);
    if (minFreqSet.size === 0) {
      this.frequencyMap.delete(this.minFrequency);
    }
    
    this.cache.delete(keyToEvict);
    this.metrics.recordEviction();
    console.log(`LFU: Evicted ${keyToEvict} (frequency: ${this.minFrequency})`);
  }

  remove(key: K): boolean {
    const node = this.cache.get(key);
    if (!node) return false;
    
    const freqSet = this.frequencyMap.get(node.frequency);
    if (freqSet) {
      freqSet.delete(key);
      if (freqSet.size === 0) {
        this.frequencyMap.delete(node.frequency);
      }
    }
    
    this.cache.delete(key);
    return true;
  }

  clear(): void {
    this.cache.clear();
    this.frequencyMap.clear();
    this.minFrequency = 0;
  }

  size(): number {
    return this.cache.size;
  }

  // LFU with Aging (시간 기반 감쇠)
  static createLFUWithAging<K, V>(capacity: number, agingFactor: number = 0.9): LFUWithAging<K, V> {
    return new LFUWithAging(capacity, agingFactor);
  }
}

class LFUNode<K, V> {
  key: K;
  value: V;
  frequency: number = 1;
  lastAccess: number = Date.now();

  constructor(key: K, value: V) {
    this.key = key;
    this.value = value;
  }
}

// LFU with Aging
class LFUWithAging<K, V> extends LFUCache<K, V> {
  private agingFactor: number;
  private lastAgingTime: number = Date.now();
  private agingInterval: number = 60000; // 1분

  constructor(capacity: number, agingFactor: number = 0.9) {
    super(capacity);
    this.agingFactor = agingFactor;
    this.startAgingProcess();
  }

  private startAgingProcess(): void {
    setInterval(() => {
      this.ageFrequencies();
    }, this.agingInterval);
  }

  private ageFrequencies(): void {
    console.log('Aging frequencies...');
    
    for (const [key, node] of this.cache) {
      const oldFreq = node.frequency;
      const newFreq = Math.max(1, Math.floor(oldFreq * this.agingFactor));
      
      if (newFreq !== oldFreq) {
        // 빈도 맵 업데이트
        const oldFreqSet = this.frequencyMap.get(oldFreq);
        if (oldFreqSet) {
          oldFreqSet.delete(key);
          if (oldFreqSet.size === 0) {
            this.frequencyMap.delete(oldFreq);
          }
        }
        
        if (!this.frequencyMap.has(newFreq)) {
          this.frequencyMap.set(newFreq, new Set());
        }
        this.frequencyMap.get(newFreq)!.add(key);
        
        node.frequency = newFreq;
      }
    }
    
    // 최소 빈도 재계산
    this.minFrequency = Math.min(...this.frequencyMap.keys());
  }
}
```

---

## 🎲 Random & OPT

### Random 교체 알고리즘

```typescript
class RandomCache<K, V> extends BaseCache<K, V> {
  private cache: Map<K, V> = new Map();

  get(key: K): V | undefined {
    const startTime = performance.now();
    const value = this.cache.get(key);
    
    if (value !== undefined) {
      this.metrics.recordHit(performance.now() - startTime);
    } else {
      this.metrics.recordMiss(performance.now() - startTime);
    }
    
    return value;
  }

  put(key: K, value: V): void {
    if (this.cache.has(key)) {
      this.cache.set(key, value);
      return;
    }

    if (this.cache.size >= this.capacity) {
      this.evictRandom();
    }

    this.cache.set(key, value);
  }

  private evictRandom(): void {
    const keys = Array.from(this.cache.keys());
    const randomIndex = Math.floor(Math.random() * keys.length);
    const keyToEvict = keys[randomIndex];
    
    this.cache.delete(keyToEvict);
    this.metrics.recordEviction();
    console.log(`Random: Evicted ${keyToEvict}`);
  }

  remove(key: K): boolean {
    return this.cache.delete(key);
  }

  clear(): void {
    this.cache.clear();
  }

  size(): number {
    return this.cache.size;
  }
}

// OPT (Optimal) - 이론적 최적 알고리즘
class OptimalCache<K, V> extends BaseCache<K, V> {
  private cache: Map<K, V> = new Map();
  private futureAccesses: K[] = [];
  private currentIndex: number = 0;

  // OPT는 미래의 접근 패턴을 알아야 함
  setFutureAccesses(accesses: K[]): void {
    this.futureAccesses = accesses;
    this.currentIndex = 0;
  }

  get(key: K): V | undefined {
    const startTime = performance.now();
    const value = this.cache.get(key);
    
    this.currentIndex++;
    
    if (value !== undefined) {
      this.metrics.recordHit(performance.now() - startTime);
    } else {
      this.metrics.recordMiss(performance.now() - startTime);
    }
    
    return value;
  }

  put(key: K, value: V): void {
    if (this.cache.has(key)) {
      this.cache.set(key, value);
      return;
    }

    if (this.cache.size >= this.capacity) {
      this.evictOptimal();
    }

    this.cache.set(key, value);
  }

  private evictOptimal(): void {
    let farthestKey: K | null = null;
    let farthestDistance = -1;
    
    // 각 캐시 항목에 대해 다음 접근까지의 거리 계산
    for (const key of this.cache.keys()) {
      const nextAccess = this.findNextAccess(key);
      
      if (nextAccess === -1) {
        // 다시 접근하지 않는 항목은 즉시 제거
        farthestKey = key;
        break;
      }
      
      if (nextAccess > farthestDistance) {
        farthestDistance = nextAccess;
        farthestKey = key;
      }
    }
    
    if (farthestKey !== null) {
      this.cache.delete(farthestKey);
      this.metrics.recordEviction();
      console.log(`OPT: Evicted ${farthestKey} (next access: ${farthestDistance})`);
    }
  }

  private findNextAccess(key: K): number {
    for (let i = this.currentIndex; i < this.futureAccesses.length; i++) {
      if (this.futureAccesses[i] === key) {
        return i - this.currentIndex;
      }
    }
    return -1; // 다시 접근하지 않음
  }

  remove(key: K): boolean {
    return this.cache.delete(key);
  }

  clear(): void {
    this.cache.clear();
  }

  size(): number {
    return this.cache.size;
  }

  // OPT 알고리즘 성능 분석
  static analyzeOptimalPerformance(capacity: number, accessPattern: number[]): void {
    console.log('\n=== Optimal Algorithm Analysis ===');
    
    const opt = new OptimalCache<number, number>(capacity);
    opt.setFutureAccesses(accessPattern);
    
    let misses = 0;
    for (const page of accessPattern) {
      if (opt.get(page) === undefined) {
        misses++;
        opt.put(page, page);
      }
    }
    
    console.log(`Optimal Algorithm Misses: ${misses}`);
    console.log('This is the theoretical minimum number of misses');
    opt.printMetrics();
  }
}
```

---

## 💡 TypeScript 구현

### 고급 캐시 구현

```typescript
// Adaptive Replacement Cache (ARC)
class ARCCache<K, V> extends BaseCache<K, V> {
  private t1: Map<K, V> = new Map();  // 최근 캐시 미스
  private t2: Map<K, V> = new Map();  // 최근 캐시 히트
  private b1: Set<K> = new Set();     // t1 고스트 리스트
  private b2: Set<K> = new Set();     // t2 고스트 리스트
  private p: number = 0;               // 적응 매개변수

  get(key: K): V | undefined {
    // T1에서 확인
    if (this.t1.has(key)) {
      // T1 -> T2로 이동 (빈도 증가)
      const value = this.t1.get(key)!;
      this.t1.delete(key);
      this.t2.set(key, value);
      return value;
    }
    
    // T2에서 확인
    if (this.t2.has(key)) {
      // T2에서 MRU로 이동
      const value = this.t2.get(key)!;
      this.t2.delete(key);
      this.t2.set(key, value);
      return value;
    }
    
    return undefined;
  }

  put(key: K, value: V): void {
    // Case 1: 캐시에 이미 존재
    if (this.t1.has(key) || this.t2.has(key)) {
      this.get(key); // 위치 업데이트
      if (this.t2.has(key)) {
        this.t2.set(key, value);
      }
      return;
    }

    // Case 2: B1에 존재 (최근 t1에서 제거됨)
    if (this.b1.has(key)) {
      // p 증가 (t1 크기 증가)
      const delta = this.b2.size > this.b1.size ? 
                    this.b1.size / this.b2.size : 1;
      this.p = Math.min(this.p + delta, this.capacity);
      
      this.replace(key, false);
      this.b1.delete(key);
      this.t2.set(key, value);
      return;
    }

    // Case 3: B2에 존재 (최근 t2에서 제거됨)
    if (this.b2.has(key)) {
      // p 감소 (t2 크기 증가)
      const delta = this.b1.size > this.b2.size ? 
                    this.b2.size / this.b1.size : 1;
      this.p = Math.max(this.p - delta, 0);
      
      this.replace(key, true);
      this.b2.delete(key);
      this.t2.set(key, value);
      return;
    }

    // Case 4: 완전히 새로운 항목
    if (this.t1.size + this.t2.size >= this.capacity) {
      if (this.t1.size === this.capacity) {
        // T1이 꽉 참
        this.evictFromT1();
      } else {
        this.replace(key, false);
      }
    }
    
    this.t1.set(key, value);
  }

  private replace(key: K, inB2: boolean): void {
    const t1Size = this.t1.size;
    
    if (t1Size >= 1 && (inB2 || t1Size > this.p)) {
      // T1에서 제거
      this.evictFromT1();
    } else {
      // T2에서 제거
      this.evictFromT2();
    }
  }

  private evictFromT1(): void {
    const key = this.t1.keys().next().value;
    if (key !== undefined) {
      this.t1.delete(key);
      this.b1.add(key);
      
      // B1 크기 제한
      if (this.b1.size > this.capacity) {
        this.b1.delete(this.b1.values().next().value);
      }
    }
  }

  private evictFromT2(): void {
    const key = this.t2.keys().next().value;
    if (key !== undefined) {
      this.t2.delete(key);
      this.b2.add(key);
      
      // B2 크기 제한
      if (this.b2.size > this.capacity) {
        this.b2.delete(this.b2.values().next().value);
      }
    }
  }

  remove(key: K): boolean {
    return this.t1.delete(key) || this.t2.delete(key);
  }

  clear(): void {
    this.t1.clear();
    this.t2.clear();
    this.b1.clear();
    this.b2.clear();
    this.p = 0;
  }

  size(): number {
    return this.t1.size + this.t2.size;
  }

  getStatistics(): any {
    return {
      t1Size: this.t1.size,
      t2Size: this.t2.size,
      b1Size: this.b1.size,
      b2Size: this.b2.size,
      p: this.p,
      hitRate: this.getHitRate()
    };
  }
}

// 캐시 알고리즘 비교 테스트
class CacheAlgorithmComparison {
  static compare(capacity: number, accessPattern: number[]): void {
    console.log('\n=== Cache Algorithm Comparison ===');
    console.log(`Capacity: ${capacity}, Access Pattern Length: ${accessPattern.length}`);
    console.log('━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');

    const algorithms = [
      { name: 'FIFO', cache: new FIFOCache<number, number>(capacity) },
      { name: 'LRU', cache: new LRUCache<number, number>(capacity) },
      { name: 'LFU', cache: new LFUCache<number, number>(capacity) },
      { name: 'Random', cache: new RandomCache<number, number>(capacity) },
      { name: 'ARC', cache: new ARCCache<number, number>(capacity) }
    ];

    // OPT는 미래를 알아야 하므로 별도 처리
    const opt = new OptimalCache<number, number>(capacity);
    opt.setFutureAccesses(accessPattern);

    const results: any[] = [];

    // 각 알고리즘 테스트
    for (const { name, cache } of algorithms) {
      const startTime = performance.now();
      let hits = 0;
      let misses = 0;

      for (const page of accessPattern) {
        if (cache.get(page) !== undefined) {
          hits++;
        } else {
          misses++;
          cache.put(page, page);
        }
      }

      const endTime = performance.now();
      results.push({
        algorithm: name,
        hits,
        misses,
        hitRate: hits / (hits + misses),
        time: endTime - startTime
      });
    }

    // OPT 테스트
    let optHits = 0;
    let optMisses = 0;
    for (const page of accessPattern) {
      if (opt.get(page) !== undefined) {
        optHits++;
      } else {
        optMisses++;
        opt.put(page, page);
      }
    }

    results.push({
      algorithm: 'OPT (Theoretical)',
      hits: optHits,
      misses: optMisses,
      hitRate: optHits / (optHits + optMisses),
      time: 0
    });

    // 결과 출력
    console.log('\nResults:');
    console.log('Algorithm\tHits\tMisses\tHit Rate\tTime(ms)');
    console.log('─────────────────────────────────────────────────');
    
    results.sort((a, b) => b.hitRate - a.hitRate);
    for (const result of results) {
      console.log(
        `${result.algorithm.padEnd(16)}\t${result.hits}\t${result.misses}\t` +
        `${(result.hitRate * 100).toFixed(2)}%\t\t${result.time.toFixed(2)}`
      );
    }
  }

  // 실제 워크로드 시뮬레이션
  static simulateWorkload(): number[] {
    const pattern: number[] = [];
    
    // 시간적 지역성 시뮬레이션
    const hotPages = [1, 2, 3, 4, 5];
    for (let i = 0; i < 100; i++) {
      if (Math.random() < 0.8) {
        // 80% 확률로 hot pages 접근
        pattern.push(hotPages[Math.floor(Math.random() * hotPages.length)]);
      } else {
        // 20% 확률로 랜덤 페이지 접근
        pattern.push(Math.floor(Math.random() * 20));
      }
    }
    
    // 공간적 지역성 시뮬레이션
    for (let i = 0; i < 50; i++) {
      const start = Math.floor(Math.random() * 10);
      for (let j = 0; j < 5; j++) {
        pattern.push(start + j);
      }
    }
    
    return pattern;
  }
}
```

---

## 🔍 실제 사례

### 실제 시스템의 캐시 알고리즘

```typescript
// Linux 페이지 캐시
class LinuxPageCache {
  // Linux는 LRU 변형 사용
  static getImplementation(): string {
    return `
Linux Page Cache Implementation:
- Two-list LRU (Active and Inactive lists)
- Page reference counting
- Second chance algorithm for eviction
- Adaptive sizing based on memory pressure
    `;
  }
}

// Redis 캐시 정책
class RedisCachePolicies {
  static getPolicies(): any {
    return {
      'noeviction': 'Return error when memory limit reached',
      'allkeys-lru': 'LRU eviction from all keys',
      'volatile-lru': 'LRU eviction from keys with TTL',
      'allkeys-lfu': 'LFU eviction from all keys',
      'volatile-lfu': 'LFU eviction from keys with TTL',
      'allkeys-random': 'Random eviction from all keys',
      'volatile-random': 'Random eviction from keys with TTL',
      'volatile-ttl': 'Evict keys with shortest TTL'
    };
  }
}

// CPU 캐시 교체 정책
class CPUCachePolicies {
  static getIntelPolicy(): string {
    return `
Intel CPU Cache Replacement:
- L1/L2: Pseudo-LRU (Tree-based approximation)
- L3: Adaptive replacement with QoS
- Hardware-based implementation for speed
    `;
  }

  static getARMPolicy(): string {
    return `
ARM CPU Cache Replacement:
- Random or Round-Robin for simplicity
- Some implementations use Pseudo-LRU
- Power efficiency focused
    `;
  }
}

// 웹 브라우저 캐시
class BrowserCache {
  static getChromeImplementation(): string {
    return `
Chrome Browser Cache:
- Multi-tier cache (Memory + Disk)
- LRU for memory cache
- Complex scoring for disk cache
- Considers: Frequency, Recency, Size, Type
    `;
  }
}
```

> [!tip] 실무 팁
> 캐시 알고리즘 선택 시 워크로드 특성을 고려하세요. 시간적 지역성이 강하면 LRU, 특정 데이터가 자주 접근되면 LFU, 예측 불가능한 패턴이면 ARC나 2Q 같은 적응형 알고리즘이 좋습니다. 또한 구현 복잡도와 메모리 오버헤드도 고려해야 합니다.

## 📚 참고 자료
- Operating System Concepts (Silberschatz)
- Computer Architecture: A Quantitative Approach (Hennessy & Patterson)
- The ARC Paper (Megiddo & Modha)
- Linux Kernel Source Code