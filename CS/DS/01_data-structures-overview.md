---
tags:
  - data-structure
  - cs-fundamentals
  - array
  - linked-list
  - stack
  - queue
created: 2025-01-04
updated: 2025-01-05
aliases:
  - 자료구조 개요
  - 기본 자료구조
  - Data Structures
description: 기본 자료구조의 개념과 특징, 사용 사례에 대한 종합 가이드
status: published
category: tutorial
---

# 자료구조 개요

> [!info] 개요
> 자료구조는 데이터를 효율적으로 저장하고 관리하는 방법입니다. 각 자료구조는 고유한 특징과 장단점을 가지며, 문제의 성격에 따라 적절한 자료구조를 선택하는 것이 중요합니다.

## 📑 목차

- [[#🎯 자료구조란?]]
- [[#📊 자료구조 분류]]
- [[#📦 배열 (Array)]]
- [[#🔗 연결 리스트 (Linked List)]]
- [[#📚 스택 (Stack)]]
- [[#🚶 큐 (Queue)]]
- [[#🌳 트리와 그래프 미리보기]]
- [[#💡 자료구조 선택 가이드]]
- [[#📚 참고자료]]

---

## 🎯 자료구조란?

> [!note] 정의
> 자료구조(Data Structure)는 데이터를 조직하고, 관리하고, 저장하는 방식입니다. 효율적인 접근과 수정을 가능하게 합니다.

### 자료구조가 중요한 이유

1. **효율성**: 적절한 자료구조는 프로그램 성능을 극적으로 향상
2. **가독성**: 데이터 관계를 명확하게 표현
3. **유지보수**: 코드 구조화와 확장성 개선
4. **문제 해결**: 복잡한 문제를 단순하게 해결

---

## 📊 자료구조 분류

### 1. 선형 자료구조 (Linear)
데이터가 순차적으로 연결된 구조

```
[A] → [B] → [C] → [D]
```

- **배열 (Array)**
- **연결 리스트 (Linked List)**
- **스택 (Stack)**
- **큐 (Queue)**

### 2. 비선형 자료구조 (Non-linear)
계층적이거나 그물망 형태의 구조

```
      [A]
     /   \
   [B]    [C]
   / \    /
 [D] [E] [F]
```

- **트리 (Tree)**
- **그래프 (Graph)**
- **해시 테이블 (Hash Table)**

---

## 📦 배열 (Array)

> [!info] 배열이란?
> 동일한 자료형의 데이터를 연속된 메모리 공간에 저장하는 선형 자료구조

### 특징

- **인덱스 기반 접근**: O(1) 시간 복잡도로 직접 접근
- **고정 크기**: 생성 시 크기 결정 (정적 배열)
- **연속 메모리**: 캐시 친화적

### 구조 시각화

```
Index:  [0]  [1]  [2]  [3]  [4]
Value:  [10] [20] [30] [40] [50]
Memory: 100  104  108  112  116  (주소)
```

### 장단점

> [!success] 장점
> - 빠른 인덱스 접근 O(1)
> - 캐시 지역성 우수
> - 구조가 단순하고 직관적

> [!failure] 단점
> - 크기 변경 불가 (정적 배열)
> - 삽입/삭제 시 O(n) 시간 복잡도
> - 메모리 낭비 가능성

### 코드 예제

**TypeScript:**
```typescript
// 배열 생성과 기본 연산
let arr: number[] = [10, 20, 30, 40, 50];

// 접근: O(1)
console.log(arr[2]);  // 30

// 수정: O(1)
arr[2] = 35;

// 삽입: O(n) - 중간에 삽입 시 요소 이동 필요
arr.splice(2, 0, 25);  // [10, 20, 25, 35, 40, 50]

// 삭제: O(n)
arr.splice(2, 1);  // [10, 20, 35, 40, 50]
```

**C++:**
```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    // 배열 생성과 기본 연산
    vector<int> arr = {10, 20, 30, 40, 50};
    
    // 접근: O(1)
    cout << arr[2] << endl;  // 30
    
    // 수정: O(1)
    arr[2] = 35;
    
    // 삽입: O(n) - 중간에 삽입 시 요소 이동 필요
    arr.insert(arr.begin() + 2, 25);  // [10, 20, 25, 35, 40, 50]
    
    // 삭제: O(n)
    arr.erase(arr.begin() + 2);  // [10, 20, 35, 40, 50]
    
    return 0;
}
```

### 사용 사례
- 데이터 크기가 고정적일 때
- 빈번한 인덱스 접근이 필요할 때
- 행렬, 이미지 픽셀 데이터

---

## 🔗 연결 리스트 (Linked List)

> [!info] 연결 리스트란?
> 노드들이 포인터로 연결된 선형 자료구조. 각 노드는 데이터와 다음 노드의 주소를 저장

### 구조 시각화

```
[데이터|다음] → [데이터|다음] → [데이터|NULL]
    노드1          노드2          노드3
```

### 종류

#### 1. 단일 연결 리스트

**TypeScript:**
```typescript
class Node<T> {
    data: T;
    next: Node<T> | null = null;
    
    constructor(data: T) {
        this.data = data;
    }
}

class LinkedList<T> {
    head: Node<T> | null = null;
    
    append(data: T): void {
        const newNode = new Node(data);
        if (!this.head) {
            this.head = newNode;
            return;
        }
        let current = this.head;
        while (current.next) {
            current = current.next;
        }
        current.next = newNode;
    }
}
```

**C++:**
```cpp
#include <iostream>
using namespace std;

template<typename T>
struct Node {
    T data;
    Node* next;
    
    Node(T val) : data(val), next(nullptr) {}
};

template<typename T>
class LinkedList {
private:
    Node<T>* head;
    
public:
    LinkedList() : head(nullptr) {}
    
    void append(T data) {
        Node<T>* newNode = new Node<T>(data);
        if (!head) {
            head = newNode;
            return;
        }
        Node<T>* current = head;
        while (current->next) {
            current = current->next;
        }
        current->next = newNode;
    }
};
```

#### 2. 이중 연결 리스트
```
NULL ← [이전|데이터|다음] ↔ [이전|데이터|다음] ↔ [이전|데이터|다음] → NULL
```

#### 3. 원형 연결 리스트
```
[데이터|다음] → [데이터|다음] → [데이터|다음]
      ↑                               ↓
      └───────────────────────────────┘
```

### 장단점

> [!success] 장점
> - 동적 크기 조절
> - 삽입/삭제 O(1) (위치를 알 때)
> - 메모리 효율적 사용

> [!failure] 단점
> - 인덱스 접근 O(n)
> - 포인터 저장 공간 추가
> - 캐시 지역성 낮음

### 배열 vs 연결 리스트

| 연산 | 배열 | 연결 리스트 |
|------|------|------------|
| 접근 | O(1) | O(n) |
| 검색 | O(n) | O(n) |
| 삽입(처음) | O(n) | O(1) |
| 삽입(끝) | O(1) | O(n)* |
| 삭제(처음) | O(n) | O(1) |
| 메모리 | 연속 | 분산 |

*tail 포인터 유지 시 O(1)

---

## 📚 스택 (Stack)

> [!info] 스택이란?
> LIFO(Last In First Out) 원칙을 따르는 선형 자료구조. 마지막에 들어간 데이터가 먼저 나옴

### 구조 시각화

```
    Push ↓   ↑ Pop
        [D] ← TOP
        [C]
        [B]
        [A]
        ───
```

### 주요 연산

**TypeScript:**
```typescript
class Stack<T> {
    private items: T[] = [];
    
    push(item: T): void {  // O(1)
        this.items.push(item);
    }
    
    pop(): T | undefined {  // O(1)
        return this.items.pop();
    }
    
    peek(): T | undefined {  // O(1)
        return this.items[this.items.length - 1];
    }
    
    isEmpty(): boolean {  // O(1)
        return this.items.length === 0;
    }
    
    size(): number {  // O(1)
        return this.items.length;
    }
}
```

**C++:**
```cpp
#include <iostream>
#include <vector>
using namespace std;

template<typename T>
class Stack {
private:
    vector<T> items;
    
public:
    void push(T item) {  // O(1)
        items.push_back(item);
    }
    
    T pop() {  // O(1)
        if (!isEmpty()) {
            T top = items.back();
            items.pop_back();
            return top;
        }
        throw runtime_error("Stack is empty");
    }
    
    T peek() {  // O(1)
        if (!isEmpty()) {
            return items.back();
        }
        throw runtime_error("Stack is empty");
    }
    
    bool isEmpty() {  // O(1)
        return items.empty();
    }
    
    int size() {  // O(1)
        return items.size();
    }
};
```

### 활용 예제

> [!example] 괄호 검증

**TypeScript:**
```typescript
function isValidParentheses(s: string): boolean {
    const stack: string[] = [];
    const pairs: { [key: string]: string } = {
        '(': ')',
        '[': ']',
        '{': '}'
    };
    
    for (const char of s) {
        if (char in pairs) {
            stack.push(char);
        } else if (Object.values(pairs).includes(char)) {
            if (stack.length === 0 || pairs[stack.pop()!] !== char) {
                return false;
            }
        }
    }
    
    return stack.length === 0;
}

console.log(isValidParentheses("()[]{}"));  // true
console.log(isValidParentheses("([)]"));    // false
```

**C++:**
```cpp
#include <iostream>
#include <stack>
#include <unordered_map>
#include <string>
using namespace std;

bool isValidParentheses(string s) {
    stack<char> st;
    unordered_map<char, char> pairs = {
        {'(', ')'},
        {'[', ']'},
        {'{', '}'}
    };
    
    for (char c : s) {
        if (pairs.find(c) != pairs.end()) {
            st.push(c);
        } else if (c == ')' || c == ']' || c == '}') {
            if (st.empty()) return false;
            char top = st.top();
            st.pop();
            if (pairs[top] != c) return false;
        }
    }
    
    return st.empty();
}
```

### 사용 사례
- 함수 호출 스택
- 실행 취소 (Undo) 기능
- 괄호 매칭
- 후위 표기법 계산
- DFS (깊이 우선 탐색)

---

## 🚶 큐 (Queue)

> [!info] 큐란?
> FIFO(First In First Out) 원칙을 따르는 선형 자료구조. 먼저 들어간 데이터가 먼저 나옴

### 구조 시각화

```
Enqueue →  [A][B][C][D]  → Dequeue
          Rear      Front
```

### 주요 연산

**TypeScript:**
```typescript
class Queue<T> {
    private items: T[] = [];
    
    enqueue(item: T): void {  // O(1)
        this.items.push(item);
    }
    
    dequeue(): T | undefined {  // O(1)
        return this.items.shift();
    }
    
    front(): T | undefined {  // O(1)
        return this.items[0];
    }
    
    isEmpty(): boolean {  // O(1)
        return this.items.length === 0;
    }
    
    size(): number {  // O(1)
        return this.items.length;
    }
}
```

**C++:**
```cpp
#include <iostream>
#include <queue>
using namespace std;

template<typename T>
class Queue {
private:
    queue<T> items;
    
public:
    void enqueue(T item) {  // O(1)
        items.push(item);
    }
    
    T dequeue() {  // O(1)
        if (!isEmpty()) {
            T front = items.front();
            items.pop();
            return front;
        }
        throw runtime_error("Queue is empty");
    }
    
    T front() {  // O(1)
        if (!isEmpty()) {
            return items.front();
        }
        throw runtime_error("Queue is empty");
    }
    
    bool isEmpty() {  // O(1)
        return items.empty();
    }
    
    int size() {  // O(1)
        return items.size();
    }
};
```

### 큐의 변형

#### 1. 원형 큐 (Circular Queue)
```
     [3]
   [2] [4]
  [1]   [5]
   [0] [6]
     [7]
```

#### 2. 우선순위 큐 (Priority Queue)

**TypeScript:**
```typescript
class PriorityQueue<T> {
    private heap: [number, T][] = [];
    
    push(item: T, priority: number): void {
        this.heap.push([priority, item]);
        this.heap.sort((a, b) => a[0] - b[0]);
    }
    
    pop(): T | undefined {
        const item = this.heap.shift();
        return item ? item[1] : undefined;
    }
}
```

**C++:**
```cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

template<typename T>
class PriorityQueue {
private:
    priority_queue<pair<int, T>, vector<pair<int, T>>, greater<pair<int, T>>> heap;
    
public:
    void push(T item, int priority) {
        heap.push(make_pair(priority, item));
    }
    
    T pop() {
        if (!heap.empty()) {
            T item = heap.top().second;
            heap.pop();
            return item;
        }
        throw runtime_error("Priority queue is empty");
    }
};
```

#### 3. 덱 (Deque - Double-ended Queue)
```
← Dequeue [Front] [A][B][C][D] [Rear] Enqueue →
← Enqueue                            Dequeue →
```

### 사용 사례
- 작업 대기열
- BFS (너비 우선 탐색)
- 프린터 스풀
- 프로세스 스케줄링
- 메시지 큐

---

## 🌳 트리와 그래프 미리보기

### 트리 (Tree)
계층적 구조를 가진 비선형 자료구조

```
        [Root]
       /      \
    [Node]   [Node]
    /    \      \
 [Leaf] [Leaf] [Leaf]
```

**특징:**
- 루트 노드에서 시작
- 부모-자식 관계
- 사이클 없음
- N개 노드, N-1개 간선

### 그래프 (Graph)
노드와 간선으로 이루어진 비선형 자료구조

```
  [A]───[B]
   │ \  / │
   │  ×   │
   │ / \  │
  [C]───[D]
```

**특징:**
- 방향성/무방향성
- 순환/비순환
- 가중치 있음/없음

---

## 💡 자료구조 선택 가이드

> [!tip] 자료구조 선택 체크리스트
> 
> **데이터 특성 파악**
> - [ ] 데이터 크기가 고정적인가?
> - [ ] 데이터 접근 패턴은? (순차적/랜덤)
> - [ ] 삽입/삭제가 빈번한가?
> 
> **성능 요구사항**
> - [ ] 어떤 연산이 가장 빈번한가?
> - [ ] 시간 복잡도 제약이 있는가?
> - [ ] 메모리 제약이 있는가?
> 
> **구현 복잡도**
> - [ ] 구현 난이도는 어떤가?
> - [ ] 유지보수가 쉬운가?

### 상황별 추천 자료구조

| 상황 | 추천 자료구조 | 이유 |
|------|-------------|------|
| 빈번한 인덱스 접근 | 배열 | O(1) 접근 시간 |
| 빈번한 삽입/삭제 | 연결 리스트 | O(1) 삽입/삭제 |
| 후입선출 필요 | 스택 | LIFO 구조 |
| 선입선출 필요 | 큐 | FIFO 구조 |
| 빠른 검색 | 해시 테이블 | O(1) 평균 검색 |
| 정렬된 데이터 | 이진 탐색 트리 | O(log n) 연산 |
| 우선순위 처리 | 힙 | 최소/최대값 빠른 접근 |

---

## 📚 참고자료

### 추가 학습 자료
- [[02_divide-and-conquer|분할 정복과 자료구조]]
- [[03_sorting-algorithms|정렬과 자료구조의 관계]]
- [[01_algorithm-performance|자료구조별 성능 분석]]

### 연습 문제
1. 스택을 사용하여 문자열 뒤집기
2. 큐를 사용하여 이진 트리 레벨 순회
3. 연결 리스트 뒤집기
4. 배열과 연결 리스트 성능 비교

### 외부 리소스
- [Visualgo - 자료구조 시각화](https://visualgo.net/)
- [GeeksforGeeks Data Structures](https://www.geeksforgeeks.org/data-structures/)

---

> [!quote]
> "프로그램 = 자료구조 + 알고리즘" - Niklaus Wirth

---

## 🔑 해시 테이블 (Hash Table)

> [!info] 해시 테이블이란?
> 키-값 쌍을 저장하며 평균적으로 O(1) 시간에 검색, 삽입, 삭제가 가능한 자료구조

### 구조와 원리

```
해시 함수: key → index

[Key] → Hash Function → [Index] → [Bucket]
"John" → hash("John") → 2 → ["John": 25]
```

### 주요 연산 시간복잡도

| 연산 | 평균 | 최악 |
|------|------|------|
| 삽입 | O(1) | O(n) |
| 삭제 | O(1) | O(n) |
| 검색 | O(1) | O(n) |
| 접근 | N/A | N/A |

### 충돌 해결 방법

#### 1. 체이닝 (Chaining)

**TypeScript:**
```typescript
class HashTable<K, V> {
    private buckets: Array<Array<[K, V]>>;
    private size: number;
    
    constructor(size: number = 10) {
        this.size = size;
        this.buckets = new Array(size).fill(null).map(() => []);
    }
    
    private hash(key: K): number {
        const str = String(key);
        let hash = 0;
        for (let i = 0; i < str.length; i++) {
            hash = (hash * 31 + str.charCodeAt(i)) % this.size;
        }
        return hash;
    }
    
    set(key: K, value: V): void {
        const index = this.hash(key);
        const bucket = this.buckets[index];
        
        for (let i = 0; i < bucket.length; i++) {
            if (bucket[i][0] === key) {
                bucket[i][1] = value;
                return;
            }
        }
        bucket.push([key, value]);
    }
    
    get(key: K): V | undefined {
        const index = this.hash(key);
        const bucket = this.buckets[index];
        
        for (const [k, v] of bucket) {
            if (k === key) return v;
        }
        return undefined;
    }
}
```

**C++:**
```cpp
#include <iostream>
#include <vector>
#include <list>
#include <utility>
using namespace std;

template<typename K, typename V>
class HashTable {
private:
    vector<list<pair<K, V>>> buckets;
    int size;
    
    int hash(K key) {
        return std::hash<K>{}(key) % size;
    }
    
public:
    HashTable(int s = 10) : size(s), buckets(s) {}
    
    void set(K key, V value) {
        int index = hash(key);
        auto& bucket = buckets[index];
        
        for (auto& p : bucket) {
            if (p.first == key) {
                p.second = value;
                return;
            }
        }
        bucket.push_back(make_pair(key, value));
    }
    
    V get(K key) {
        int index = hash(key);
        auto& bucket = buckets[index];
        
        for (const auto& p : bucket) {
            if (p.first == key) {
                return p.second;
            }
        }
        throw runtime_error("Key not found");
    }
};
```

#### 2. 개방 주소법 (Open Addressing)
- Linear Probing: 순차적으로 다음 위치 탐색
- Quadratic Probing: 제곱수만큼 이동
- Double Hashing: 두 번째 해시 함수 사용

### 사용 사례
- 데이터베이스 인덱싱
- 캐시 구현
- 집합(Set) 자료구조
- 언어의 Dictionary/Map 구현

---

## 📊 자료구조 연산별 시간복잡도 종합

| 자료구조 | 접근 | 검색 | 삽입 | 삭제 | 비고 |
|---------|------|------|------|------|------|
| 배열 | O(1) | O(n) | O(n) | O(n) | 인덱스 접근 빠름 |
| 연결 리스트 | O(n) | O(n) | O(1) | O(1) | 위치 알 때 |
| 스택 | O(n) | O(n) | O(1) | O(1) | LIFO |
| 큐 | O(n) | O(n) | O(1) | O(1) | FIFO |
| 해시 테이블 | N/A | O(1)* | O(1)* | O(1)* | *평균 시간 |
| 이진 탐색 트리 | O(log n)* | O(log n)* | O(log n)* | O(log n)* | *균형 시 |

---

## 🎯 ADT(Abstract Data Type) 개념

> [!important] ADT의 정의
> ADT는 데이터와 그 데이터에 대한 연산을 추상화한 수학적 모델입니다.
> 구현 방법이 아닌 **무엇을 할 수 있는가**에 초점을 맞춥니다.

### ADT의 중요성

1. **인터페이스와 구현의 분리**
   - 사용자는 내부 구현을 몰라도 사용 가능
   - 구현을 변경해도 인터페이스가 동일하면 영향 없음

2. **다양한 구현 방법 선택 가능**
   - 상황에 따라 최적의 구현 선택
   - 성능 요구사항에 맞는 구현 변경 가능

3. **코드 재사용성과 유지보수성 향상**
   - 표준화된 인터페이스로 호환성 보장
   - 테스트와 디버깅 용이

### ADT 예시

**Stack ADT**:
- Operations: push(), pop(), peek(), isEmpty()
- 구현: 배열 기반 or 연결 리스트 기반

**Queue ADT**:
- Operations: enqueue(), dequeue(), front(), isEmpty()
- 구현: 원형 배열 or 연결 리스트

**List ADT**:
- Operations: add(), remove(), get(), size()
- 구현: ArrayList or LinkedList