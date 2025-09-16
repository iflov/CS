---
tags:
  - database
  - nosql
  - rdbms
  - cap-theorem
  - mongodb
  - redis
created: 2025-01-08
updated: 2025-01-08
---

# NoSQL vs RDBMS

> [!info] 개요
> NoSQL과 RDBMS는 각각 다른 데이터 모델과 특성을 가진 데이터베이스 시스템입니다. CAP 이론, BASE 속성, 그리고 각 데이터베이스 유형의 특징과 사용 사례를 학습합니다.

## 📑 목차

- [[#🎯 NoSQL 개념]]
- [[#📊 CAP 이론]]
- [[#🔄 BASE vs ACID]]
- [[#📚 NoSQL 유형]]
- [[#⚖️ RDBMS vs NoSQL]]
- [[#💡 선택 기준]]
- [[#🔧 실전 구현]]
- [[#📚 참고자료]]

---

## 🎯 NoSQL 개념

### NoSQL이란?

> [!note] NoSQL 정의
> NoSQL(Not Only SQL)은 관계형 모델을 사용하지 않는 데이터베이스 시스템의 총칭입니다.

### NoSQL의 특징

| 특징 | 설명 |
|------|------|
| **유연한 스키마** | 스키마리스 또는 동적 스키마 |
| **수평적 확장** | 샤딩을 통한 스케일 아웃 |
| **빠른 쿼리** | 특정 쿼리 패턴에 최적화 |
| **대용량 처리** | 빅데이터 처리에 적합 |
| **다양한 데이터 모델** | Key-Value, Document, Graph, Column-Family |

---

## 📊 CAP 이론

### CAP Theorem

> [!warning] CAP 이론
> 분산 시스템에서 Consistency, Availability, Partition Tolerance 중 최대 2개만 보장 가능

```typescript
// TypeScript: CAP 이론 시뮬레이션
interface CAPSystem {
  consistency: boolean;
  availability: boolean;
  partitionTolerance: boolean;
}

class DistributedDatabase {
  private nodes: Map<string, any> = new Map();
  private networkPartitioned = false;
  
  // CP 시스템 (Consistency + Partition Tolerance)
  // 예: MongoDB, HBase, Redis
  async writeCP(key: string, value: any): Promise<boolean> {
    if (this.networkPartitioned) {
      // 네트워크 파티션 시 일관성 유지를 위해 쓰기 거부
      throw new Error('System unavailable to maintain consistency');
    }
    
    // 모든 노드에 동기적으로 쓰기
    const writePromises = Array.from(this.nodes.keys()).map(node => 
      this.syncWrite(node, key, value)
    );
    
    await Promise.all(writePromises);
    return true;
  }
  
  // AP 시스템 (Availability + Partition Tolerance)
  // 예: Cassandra, DynamoDB, CouchDB
  async writeAP(key: string, value: any): Promise<boolean> {
    // 네트워크 파티션에도 쓰기 허용
    const availableNodes = this.getAvailableNodes();
    
    if (availableNodes.length === 0) {
      throw new Error('No available nodes');
    }
    
    // 사용 가능한 노드에만 쓰기 (일관성 희생)
    await this.asyncWrite(availableNodes[0], key, value);
    
    // 나중에 동기화 (Eventually Consistent)
    this.scheduleSync(key, value);
    
    return true;
  }
  
  // CA 시스템 (Consistency + Availability)
  // 예: 단일 노드 RDBMS
  async writeCA(key: string, value: any): Promise<boolean> {
    // 네트워크 파티션을 허용하지 않음 (단일 노드)
    if (this.nodes.size > 1) {
      throw new Error('CA systems cannot handle network partitions');
    }
    
    // 단일 노드에 쓰기 (항상 일관성과 가용성 보장)
    await this.syncWrite('single-node', key, value);
    return true;
  }
  
  private async syncWrite(node: string, key: string, value: any): Promise<void> {
    // 동기 쓰기 구현
  }
  
  private async asyncWrite(node: string, key: string, value: any): Promise<void> {
    // 비동기 쓰기 구현
  }
  
  private getAvailableNodes(): string[] {
    // 사용 가능한 노드 반환
    return Array.from(this.nodes.keys());
  }
  
  private scheduleSync(key: string, value: any): void {
    // 나중에 동기화 스케줄링
    setTimeout(() => {
      this.syncAllNodes(key, value);
    }, 5000);
  }
  
  private async syncAllNodes(key: string, value: any): Promise<void> {
    // 모든 노드 동기화
  }
}
```

---

## 🔄 BASE vs ACID

### BASE 속성

> [!tip] BASE (Basically Available, Soft state, Eventual consistency)
> NoSQL 데이터베이스의 일관성 모델

```typescript
// TypeScript: BASE vs ACID 비교
interface TransactionModel {
  name: 'ACID' | 'BASE';
  guarantees: string[];
  tradeoffs: string[];
}

class ACIDTransaction {
  private data: Map<string, any> = new Map();
  private transactionLog: any[] = [];
  private locked: Set<string> = new Set();
  
  async beginTransaction(): Promise<void> {
    this.transactionLog = [];
    console.log('Transaction started');
  }
  
  async read(key: string): Promise<any> {
    // Isolation: 다른 트랜잭션의 영향 받지 않음
    if (this.locked.has(key)) {
      throw new Error('Resource locked');
    }
    return this.data.get(key);
  }
  
  async write(key: string, value: any): Promise<void> {
    // Atomicity: 로그에 기록
    this.transactionLog.push({ action: 'write', key, value, oldValue: this.data.get(key) });
    this.locked.add(key);
    this.data.set(key, value);
  }
  
  async commit(): Promise<void> {
    try {
      // Durability: 영구 저장
      await this.persistToStorage();
      
      // Consistency: 제약조건 검증
      if (!this.validateConstraints()) {
        throw new Error('Constraint violation');
      }
      
      console.log('Transaction committed');
      this.cleanup();
    } catch (error) {
      await this.rollback();
      throw error;
    }
  }
  
  async rollback(): Promise<void> {
    // Atomicity: 모든 변경사항 취소
    for (const log of this.transactionLog.reverse()) {
      if (log.action === 'write') {
        this.data.set(log.key, log.oldValue);
      }
    }
    console.log('Transaction rolled back');
    this.cleanup();
  }
  
  private async persistToStorage(): Promise<void> {
    // WAL (Write-Ahead Logging)
    console.log('Persisting to storage...');
  }
  
  private validateConstraints(): boolean {
    // 제약조건 검증
    return true;
  }
  
  private cleanup(): void {
    this.transactionLog = [];
    this.locked.clear();
  }
}

class BASETransaction {
  private nodes: Map<string, Map<string, any>> = new Map();
  
  constructor() {
    // 여러 노드 초기화
    this.nodes.set('node1', new Map());
    this.nodes.set('node2', new Map());
    this.nodes.set('node3', new Map());
  }
  
  // Basically Available: 기본적으로 사용 가능
  async write(key: string, value: any): Promise<void> {
    const availableNodes = this.getAvailableNodes();
    
    if (availableNodes.length === 0) {
      throw new Error('No nodes available');
    }
    
    // 최소 하나의 노드에 쓰기 성공하면 OK
    const writePromises = availableNodes.map(node => 
      this.writeToNode(node, key, value).catch(() => null)
    );
    
    const results = await Promise.all(writePromises);
    const successCount = results.filter(r => r !== null).length;
    
    if (successCount === 0) {
      throw new Error('Write failed on all nodes');
    }
    
    // Eventual Consistency: 백그라운드에서 동기화
    this.scheduleReplication(key, value);
  }
  
  // Soft State: 상태가 시간에 따라 변할 수 있음
  async read(key: string): Promise<any> {
    const availableNodes = this.getAvailableNodes();
    
    // Quorum Read: 과반수 노드에서 읽기
    const readPromises = availableNodes.map(node => 
      this.readFromNode(node, key)
    );
    
    const results = await Promise.all(readPromises);
    
    // 가장 최신 버전 반환 (Vector Clock 사용)
    return this.resolveConflicts(results);
  }
  
  private getAvailableNodes(): string[] {
    // 실제로는 헬스체크를 통해 확인
    return Array.from(this.nodes.keys());
  }
  
  private async writeToNode(node: string, key: string, value: any): Promise<void> {
    const nodeData = this.nodes.get(node);
    if (nodeData) {
      // 타임스탬프와 함께 저장
      nodeData.set(key, {
        value,
        timestamp: Date.now(),
        version: this.generateVersion()
      });
    }
  }
  
  private async readFromNode(node: string, key: string): Promise<any> {
    const nodeData = this.nodes.get(node);
    return nodeData?.get(key);
  }
  
  private resolveConflicts(results: any[]): any {
    // Last-Write-Wins (LWW) 전략
    const validResults = results.filter(r => r !== undefined);
    
    if (validResults.length === 0) return null;
    
    return validResults.reduce((latest, current) => {
      if (!latest || current.timestamp > latest.timestamp) {
        return current;
      }
      return latest;
    }).value;
  }
  
  private scheduleReplication(key: string, value: any): void {
    // Anti-Entropy: 주기적으로 노드 간 동기화
    setTimeout(() => {
      this.replicate(key, value);
    }, 1000);
  }
  
  private async replicate(key: string, value: any): Promise<void> {
    // Gossip Protocol로 전파
    console.log('Replicating to all nodes...');
  }
  
  private generateVersion(): string {
    return `v${Date.now()}-${Math.random()}`;
  }
}
```

---

## 📚 NoSQL 유형

### Document Store

```typescript
// TypeScript: MongoDB 스타일 Document Store
interface Document {
  _id?: string;
  [key: string]: any;
}

class DocumentStore {
  private collections: Map<string, Document[]> = new Map();
  
  // 컬렉션 생성
  createCollection(name: string): void {
    if (!this.collections.has(name)) {
      this.collections.set(name, []);
    }
  }
  
  // 문서 삽입
  async insertOne(collection: string, document: Document): Promise<string> {
    const docs = this.collections.get(collection) || [];
    const doc = {
      ...document,
      _id: document._id || this.generateId()
    };
    docs.push(doc);
    this.collections.set(collection, docs);
    return doc._id!;
  }
  
  // 문서 조회
  async findOne(collection: string, query: any): Promise<Document | null> {
    const docs = this.collections.get(collection) || [];
    return docs.find(doc => this.matchQuery(doc, query)) || null;
  }
  
  // 복잡한 쿼리
  async find(collection: string, query: any, options?: {
    sort?: any;
    limit?: number;
    skip?: number;
  }): Promise<Document[]> {
    let docs = this.collections.get(collection) || [];
    
    // 필터링
    docs = docs.filter(doc => this.matchQuery(doc, query));
    
    // 정렬
    if (options?.sort) {
      docs = this.sortDocuments(docs, options.sort);
    }
    
    // 페이징
    if (options?.skip) {
      docs = docs.slice(options.skip);
    }
    if (options?.limit) {
      docs = docs.slice(0, options.limit);
    }
    
    return docs;
  }
  
  // 집계 파이프라인
  async aggregate(collection: string, pipeline: any[]): Promise<any[]> {
    let docs = this.collections.get(collection) || [];
    
    for (const stage of pipeline) {
      if (stage.$match) {
        docs = docs.filter(doc => this.matchQuery(doc, stage.$match));
      } else if (stage.$group) {
        docs = this.groupDocuments(docs, stage.$group);
      } else if (stage.$sort) {
        docs = this.sortDocuments(docs, stage.$sort);
      }
    }
    
    return docs;
  }
  
  private matchQuery(doc: Document, query: any): boolean {
    for (const [key, value] of Object.entries(query)) {
      if (typeof value === 'object' && value !== null) {
        // 연산자 처리 ($gt, $lt, $in 등)
        if (value.$gt && doc[key] <= value.$gt) return false;
        if (value.$lt && doc[key] >= value.$lt) return false;
        if (value.$in && !value.$in.includes(doc[key])) return false;
      } else {
        if (doc[key] !== value) return false;
      }
    }
    return true;
  }
  
  private sortDocuments(docs: Document[], sort: any): Document[] {
    const entries = Object.entries(sort);
    return [...docs].sort((a, b) => {
      for (const [key, order] of entries) {
        if (a[key] < b[key]) return order === 1 ? -1 : 1;
        if (a[key] > b[key]) return order === 1 ? 1 : -1;
      }
      return 0;
    });
  }
  
  private groupDocuments(docs: Document[], group: any): any[] {
    // 간단한 그룹핑 구현
    const grouped = new Map();
    
    for (const doc of docs) {
      const key = doc[group._id];
      if (!grouped.has(key)) {
        grouped.set(key, []);
      }
      grouped.get(key).push(doc);
    }
    
    return Array.from(grouped.entries()).map(([key, docs]) => ({
      _id: key,
      count: docs.length,
      docs
    }));
  }
  
  private generateId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }
}

// 사용 예제
const db = new DocumentStore();
db.createCollection('users');

await db.insertOne('users', {
  name: '김철수',
  age: 30,
  email: 'kim@example.com',
  address: {
    city: '서울',
    country: '한국'
  }
});

const user = await db.findOne('users', { name: '김철수' });
const adults = await db.find('users', { age: { $gt: 18 } }, { sort: { age: -1 } });
```

### Key-Value Store

```typescript
// TypeScript: Redis 스타일 Key-Value Store
class KeyValueStore {
  private data: Map<string, any> = new Map();
  private expires: Map<string, number> = new Map();
  
  // 기본 연산
  async set(key: string, value: any, ttl?: number): Promise<void> {
    this.data.set(key, value);
    
    if (ttl) {
      const expireAt = Date.now() + ttl * 1000;
      this.expires.set(key, expireAt);
      
      setTimeout(() => {
        this.delete(key);
      }, ttl * 1000);
    }
  }
  
  async get(key: string): Promise<any> {
    if (this.isExpired(key)) {
      this.delete(key);
      return null;
    }
    return this.data.get(key);
  }
  
  async delete(key: string): Promise<boolean> {
    this.expires.delete(key);
    return this.data.delete(key);
  }
  
  // 데이터 구조 - Lists
  async lpush(key: string, ...values: any[]): Promise<number> {
    let list = this.data.get(key) || [];
    list = [...values.reverse(), ...list];
    this.data.set(key, list);
    return list.length;
  }
  
  async rpush(key: string, ...values: any[]): Promise<number> {
    let list = this.data.get(key) || [];
    list = [...list, ...values];
    this.data.set(key, list);
    return list.length;
  }
  
  async lrange(key: string, start: number, stop: number): Promise<any[]> {
    const list = this.data.get(key) || [];
    if (stop === -1) stop = list.length - 1;
    return list.slice(start, stop + 1);
  }
  
  // 데이터 구조 - Sets
  async sadd(key: string, ...members: any[]): Promise<number> {
    let set = this.data.get(key) || new Set();
    const initialSize = set.size;
    members.forEach(m => set.add(m));
    this.data.set(key, set);
    return set.size - initialSize;
  }
  
  async smembers(key: string): Promise<any[]> {
    const set = this.data.get(key);
    return set ? Array.from(set) : [];
  }
  
  // 데이터 구조 - Sorted Sets
  async zadd(key: string, ...scoreMembers: [number, any][]): Promise<number> {
    let zset = this.data.get(key) || new Map();
    let added = 0;
    
    for (const [score, member] of scoreMembers) {
      if (!zset.has(member)) added++;
      zset.set(member, score);
    }
    
    this.data.set(key, zset);
    return added;
  }
  
  async zrange(key: string, start: number, stop: number): Promise<any[]> {
    const zset = this.data.get(key);
    if (!zset) return [];
    
    const sorted = Array.from(zset.entries())
      .sort((a, b) => a[1] - b[1])
      .map(([member]) => member);
    
    if (stop === -1) stop = sorted.length - 1;
    return sorted.slice(start, stop + 1);
  }
  
  // 데이터 구조 - Hashes
  async hset(key: string, field: string, value: any): Promise<void> {
    let hash = this.data.get(key) || new Map();
    hash.set(field, value);
    this.data.set(key, hash);
  }
  
  async hget(key: string, field: string): Promise<any> {
    const hash = this.data.get(key);
    return hash ? hash.get(field) : null;
  }
  
  async hgetall(key: string): Promise<any> {
    const hash = this.data.get(key);
    if (!hash) return {};
    
    const obj: any = {};
    hash.forEach((value, field) => {
      obj[field] = value;
    });
    return obj;
  }
  
  private isExpired(key: string): boolean {
    const expireAt = this.expires.get(key);
    return expireAt ? Date.now() > expireAt : false;
  }
}
```

### Graph Database

```typescript
// TypeScript: Neo4j 스타일 Graph Database
interface Node {
  id: string;
  labels: Set<string>;
  properties: Map<string, any>;
}

interface Relationship {
  id: string;
  type: string;
  startNode: string;
  endNode: string;
  properties: Map<string, any>;
}

class GraphDatabase {
  private nodes: Map<string, Node> = new Map();
  private relationships: Map<string, Relationship> = new Map();
  private nodeRelationships: Map<string, Set<string>> = new Map();
  
  // 노드 생성
  createNode(labels: string[], properties: any = {}): Node {
    const node: Node = {
      id: this.generateId(),
      labels: new Set(labels),
      properties: new Map(Object.entries(properties))
    };
    
    this.nodes.set(node.id, node);
    this.nodeRelationships.set(node.id, new Set());
    
    return node;
  }
  
  // 관계 생성
  createRelationship(
    startNodeId: string,
    endNodeId: string,
    type: string,
    properties: any = {}
  ): Relationship {
    const relationship: Relationship = {
      id: this.generateId(),
      type,
      startNode: startNodeId,
      endNode: endNodeId,
      properties: new Map(Object.entries(properties))
    };
    
    this.relationships.set(relationship.id, relationship);
    
    // 노드에 관계 연결
    this.nodeRelationships.get(startNodeId)?.add(relationship.id);
    this.nodeRelationships.get(endNodeId)?.add(relationship.id);
    
    return relationship;
  }
  
  // Cypher 스타일 쿼리 (간단한 버전)
  async match(pattern: string): Promise<any[]> {
    // 예: MATCH (n:Person)-[:KNOWS]->(m:Person) WHERE n.name = 'Alice'
    // 실제로는 복잡한 파싱 필요
    
    const results: any[] = [];
    
    // 모든 노드 순회
    for (const [nodeId, node] of this.nodes) {
      if (this.matchesNodePattern(node, pattern)) {
        const relatedNodes = this.findRelatedNodes(nodeId, pattern);
        results.push({ node, related: relatedNodes });
      }
    }
    
    return results;
  }
  
  // 최단 경로 찾기 (Dijkstra)
  findShortestPath(startId: string, endId: string): string[] | null {
    const distances = new Map<string, number>();
    const previous = new Map<string, string | null>();
    const unvisited = new Set(this.nodes.keys());
    
    // 초기화
    for (const nodeId of this.nodes.keys()) {
      distances.set(nodeId, nodeId === startId ? 0 : Infinity);
      previous.set(nodeId, null);
    }
    
    while (unvisited.size > 0) {
      // 최소 거리 노드 찾기
      let current: string | null = null;
      let minDistance = Infinity;
      
      for (const nodeId of unvisited) {
        const distance = distances.get(nodeId)!;
        if (distance < minDistance) {
          current = nodeId;
          minDistance = distance;
        }
      }
      
      if (!current || minDistance === Infinity) break;
      if (current === endId) break;
      
      unvisited.delete(current);
      
      // 이웃 노드 업데이트
      const neighbors = this.getNeighbors(current);
      for (const neighbor of neighbors) {
        if (!unvisited.has(neighbor)) continue;
        
        const alt = distances.get(current)! + 1; // 가중치 1로 가정
        if (alt < distances.get(neighbor)!) {
          distances.set(neighbor, alt);
          previous.set(neighbor, current);
        }
      }
    }
    
    // 경로 재구성
    const path: string[] = [];
    let current: string | null = endId;
    
    while (current !== null) {
      path.unshift(current);
      current = previous.get(current) || null;
    }
    
    return path[0] === startId ? path : null;
  }
  
  private matchesNodePattern(node: Node, pattern: string): boolean {
    // 패턴 매칭 로직
    return true; // 간단화
  }
  
  private findRelatedNodes(nodeId: string, pattern: string): Node[] {
    const related: Node[] = [];
    const relationshipIds = this.nodeRelationships.get(nodeId) || new Set();
    
    for (const relId of relationshipIds) {
      const rel = this.relationships.get(relId);
      if (rel) {
        const otherNodeId = rel.startNode === nodeId ? rel.endNode : rel.startNode;
        const otherNode = this.nodes.get(otherNodeId);
        if (otherNode) {
          related.push(otherNode);
        }
      }
    }
    
    return related;
  }
  
  private getNeighbors(nodeId: string): string[] {
    const neighbors: string[] = [];
    const relationshipIds = this.nodeRelationships.get(nodeId) || new Set();
    
    for (const relId of relationshipIds) {
      const rel = this.relationships.get(relId);
      if (rel) {
        const neighbor = rel.startNode === nodeId ? rel.endNode : rel.startNode;
        neighbors.push(neighbor);
      }
    }
    
    return neighbors;
  }
  
  private generateId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }
}
```

---

## ⚖️ RDBMS vs NoSQL

### 비교 표

| 측면 | RDBMS | NoSQL |
|------|-------|-------|
| **데이터 모델** | 테이블/행/열 | 다양함 (Document, KV, Graph, Column) |
| **스키마** | 고정 스키마 | 유연한 스키마 |
| **확장성** | 수직적 확장 | 수평적 확장 |
| **ACID** | 완벽 지원 | 제한적 지원 |
| **JOIN** | 강력한 JOIN | 제한적/없음 |
| **쿼리** | SQL 표준 | 데이터베이스별 상이 |
| **일관성** | 강한 일관성 | 최종 일관성 |

---

## 💡 선택 기준

### 의사결정 트리

```typescript
// TypeScript: 데이터베이스 선택 도우미
interface Requirements {
  dataStructure: 'structured' | 'semi-structured' | 'unstructured';
  scalability: 'vertical' | 'horizontal';
  consistency: 'strong' | 'eventual';
  queryComplexity: 'simple' | 'complex';
  transactionNeeds: 'acid' | 'base' | 'none';
  dataRelationships: 'heavy' | 'moderate' | 'minimal';
}

class DatabaseSelector {
  recommend(requirements: Requirements): string[] {
    const recommendations: string[] = [];
    
    // RDBMS 추천 조건
    if (requirements.dataStructure === 'structured' &&
        requirements.consistency === 'strong' &&
        requirements.transactionNeeds === 'acid' &&
        requirements.dataRelationships !== 'minimal') {
      recommendations.push('RDBMS (MySQL, PostgreSQL, Oracle)');
    }
    
    // Document Store 추천 조건
    if (requirements.dataStructure === 'semi-structured' &&
        requirements.scalability === 'horizontal' &&
        requirements.queryComplexity === 'complex') {
      recommendations.push('Document Store (MongoDB, CouchDB)');
    }
    
    // Key-Value Store 추천 조건
    if (requirements.queryComplexity === 'simple' &&
        requirements.dataRelationships === 'minimal' &&
        requirements.scalability === 'horizontal') {
      recommendations.push('Key-Value Store (Redis, DynamoDB)');
    }
    
    // Graph Database 추천 조건
    if (requirements.dataRelationships === 'heavy' &&
        requirements.queryComplexity === 'complex') {
      recommendations.push('Graph Database (Neo4j, Amazon Neptune)');
    }
    
    // Column-Family Store 추천 조건
    if (requirements.scalability === 'horizontal' &&
        requirements.dataStructure === 'structured' &&
        requirements.consistency === 'eventual') {
      recommendations.push('Column-Family Store (Cassandra, HBase)');
    }
    
    return recommendations.length > 0 ? recommendations : ['Consider Polyglot Persistence'];
  }
}
```

---

## 🔧 실전 구현

### Polyglot Persistence

```typescript
// TypeScript: 다중 데이터베이스 사용
class PolyglotPersistence {
  private mysql: any;  // RDBMS for transactions
  private redis: any;  // Cache & sessions
  private mongodb: any; // Document storage
  private neo4j: any;  // Graph relationships
  
  // 사용자 프로필 - MongoDB
  async saveUserProfile(userId: string, profile: any): Promise<void> {
    await this.mongodb.collection('profiles').updateOne(
      { userId },
      { $set: profile },
      { upsert: true }
    );
  }
  
  // 거래 내역 - MySQL (ACID 필요)
  async recordTransaction(transaction: any): Promise<void> {
    const connection = await this.mysql.getConnection();
    await connection.beginTransaction();
    
    try {
      await connection.query(
        'INSERT INTO transactions SET ?',
        transaction
      );
      
      await connection.query(
        'UPDATE accounts SET balance = balance - ? WHERE id = ?',
        [transaction.amount, transaction.fromAccount]
      );
      
      await connection.query(
        'UPDATE accounts SET balance = balance + ? WHERE id = ?',
        [transaction.amount, transaction.toAccount]
      );
      
      await connection.commit();
    } catch (error) {
      await connection.rollback();
      throw error;
    } finally {
      connection.release();
    }
  }
  
  // 세션 관리 - Redis
  async saveSession(sessionId: string, data: any): Promise<void> {
    await this.redis.setex(
      `session:${sessionId}`,
      3600, // 1시간 TTL
      JSON.stringify(data)
    );
  }
  
  // 소셜 네트워크 - Neo4j
  async findFriendRecommendations(userId: string): Promise<any[]> {
    const query = `
      MATCH (user:User {id: $userId})-[:FRIEND]->(friend)-[:FRIEND]->(fof)
      WHERE NOT (user)-[:FRIEND]->(fof) AND user <> fof
      RETURN fof, COUNT(*) as mutualFriends
      ORDER BY mutualFriends DESC
      LIMIT 10
    `;
    
    const result = await this.neo4j.run(query, { userId });
    return result.records.map(r => r.get('fof'));
  }
}
```

---

## 📚 참고자료

### 내부 문서
- [[01_database-fundamentals|데이터베이스 기초]]
- [[06_transaction-acid|트랜잭션과 ACID]]
- [[12_partitioning-sharding|파티셔닝과 샤딩]]

### 외부 리소스
- [CAP Theorem Explained](https://www.ibm.com/cloud/learn/cap-theorem)
- [NoSQL Databases Comparison](https://www.mongodb.com/nosql-explained)
- [Martin Fowler - NoSQL Distilled](https://martinfowler.com/books/nosql.html)
- [Database Selection Guide](https://aws.amazon.com/nosql/)

### 추가 학습 주제
- NewSQL Databases
- Time-Series Databases
- Vector Databases
- Multi-Model Databases
- Blockchain as Database

---

> [!quote]
> "NoSQL vs RDBMS는 우열의 문제가 아닙니다. 각각의 장단점을 이해하고, 요구사항에 맞는 최적의 도구를 선택하는 것이 중요합니다."