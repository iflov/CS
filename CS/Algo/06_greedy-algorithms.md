---
tags:
  - algorithm
  - greedy
  - optimization
  - problem-solving
created: 2025-01-06
updated: 2025-01-06
aliases:
  - 탐욕 알고리즘
  - Greedy Algorithm
  - 그리디
description: 탐욕 알고리즘의 원리와 적용 조건
---

# 탐욕 알고리즘 (Greedy Algorithm)

> [!info] 개요
> 탐욕 알고리즘은 매 순간 최적이라고 생각되는 선택을 하는 알고리즘 설계 기법입니다. 지역 최적해(local optimum)를 선택해 나가면서 전역 최적해(global optimum)를 구합니다. 모든 문제에 적용할 수는 없지만, 적용 가능한 경우 매우 효율적입니다.

## 📑 목차

- [[#⚡ 빠른 시작]]
- [[#🔧 핵심 개념]]
- [[#💡 실전 예제]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## ⚡ 빠른 시작

> [!example] 동전 거스름돈 문제
> ```python
> def coin_change_greedy(amount, coins=[500, 100, 50, 10]):
>     """
>     큰 동전부터 사용하는 탐욕 알고리즘
>     주의: 모든 경우에 최적해를 보장하지 않음
>     """
>     coins.sort(reverse=True)  # 큰 동전부터
>     result = []
>     
>     for coin in coins:
>         count = amount // coin
>         if count > 0:
>             result.extend([coin] * count)
>             amount %= coin
>     
>     return result if amount == 0 else None
> 
> # 620원 거스름돈
> print(coin_change_greedy(620))  # [500, 100, 10, 10]
> ```

탐욕 알고리즘은 **"지금 당장 최선의 선택"**을 반복합니다.

---

## 🔧 핵심 개념

### 탐욕 알고리즘의 특징

> [!note] 핵심 특성
> 1. **탐욕적 선택**: 현재 상황에서 최선의 선택
> 2. **최적 부분 구조**: 부분 문제의 최적해가 전체 최적해 구성
> 3. **선택 후 고려 안함**: 한번 선택한 것은 번복하지 않음
> 4. **빠른 실행**: 일반적으로 O(n) 또는 O(n log n)

### 탐욕 알고리즘 적용 조건

#### 탐욕 선택 속성 (Greedy Choice Property)

> [!important] 지역 최적이 전역 최적을 보장
> 각 단계에서의 탐욕적 선택이 최종적으로 최적해를 만들어야 함

```python
# 탐욕 선택이 성공하는 예: 활동 선택
def activity_selection(activities):
    """활동 종료 시간이 빠른 것부터 선택"""
    # activities = [(start, end), ...]
    activities.sort(key=lambda x: x[1])  # 종료 시간 기준 정렬
    
    selected = [activities[0]]
    last_end = activities[0][1]
    
    for start, end in activities[1:]:
        if start >= last_end:  # 겹치지 않으면 선택
            selected.append((start, end))
            last_end = end
    
    return selected
```

#### 최적 부분 구조 (Optimal Substructure)

> [!tip] 부분 문제의 최적해로 전체 최적해 구성
> 문제를 부분 문제로 나눴을 때, 각 부분의 최적해가 전체 최적해가 됨

### 탐욕 vs 동적 계획법

| 특징 | 탐욕 알고리즘 | 동적 계획법 |
|------|--------------|------------|
| 선택 방식 | 현재 최선만 선택 | 모든 경우 고려 |
| 복잡도 | O(n) ~ O(n log n) | O(n²) ~ O(2ⁿ) |
| 정확성 | 특정 조건에서만 최적 | 항상 최적 |
| 구현 난이도 | 상대적으로 쉬움 | 상대적으로 어려움 |
| 메모리 | 적음 | 많음 |

### 대표적인 탐욕 알고리즘

1. **Kruskal's Algorithm** - 최소 신장 트리
2. **Prim's Algorithm** - 최소 신장 트리
3. **Dijkstra's Algorithm** - 최단 경로
4. **Huffman Coding** - 데이터 압축
5. **Activity Selection** - 스케줄링

---

## 💡 실전 예제

### 예제 1: 활동 선택 문제 (Activity Selection)

> [!example] 최대한 많은 활동 선택하기
> ```python
> def activity_selection_detailed(activities):
>     """
>     activities = [(name, start, end), ...]
>     겹치지 않는 최대 개수의 활동 선택
>     """
>     # 종료 시간 기준 정렬
>     activities.sort(key=lambda x: x[2])
>     
>     selected = []
>     selected_names = []
>     last_end = 0
>     
>     for name, start, end in activities:
>         if start >= last_end:
>             selected.append((name, start, end))
>             selected_names.append(name)
>             last_end = end
>     
>     return selected_names, len(selected)
> 
> # 예제 데이터
> activities = [
>     ("A", 0, 6),
>     ("B", 1, 4),
>     ("C", 3, 5),
>     ("D", 5, 7),
>     ("E", 3, 9),
>     ("F", 5, 9),
>     ("G", 6, 10),
>     ("H", 8, 11),
>     ("I", 8, 12),
>     ("J", 2, 14),
>     ("K", 12, 16)
> ]
> 
> names, count = activity_selection_detailed(activities)
> print(f"선택된 활동: {names}")  # ['B', 'D', 'H', 'K']
> print(f"최대 활동 수: {count}")  # 4
> ```

### 예제 2: 분할 가능 배낭 문제 (Fractional Knapsack)

> [!example] 물건을 쪼갤 수 있는 배낭 문제
> ```python
> def fractional_knapsack(items, capacity):
>     """
>     items = [(value, weight), ...]
>     물건을 쪼갤 수 있을 때 최대 가치
>     """
>     # 가치/무게 비율로 정렬 (내림차순)
>     items_with_ratio = [
>         (value, weight, value/weight, i) 
>         for i, (value, weight) in enumerate(items)
>     ]
>     items_with_ratio.sort(key=lambda x: x[2], reverse=True)
>     
>     total_value = 0
>     remaining = capacity
>     selected = []
>     
>     for value, weight, ratio, idx in items_with_ratio:
>         if remaining >= weight:
>             # 물건 전체를 담음
>             total_value += value
>             remaining -= weight
>             selected.append((idx, weight, value))
>         else:
>             # 물건을 쪼개서 담음
>             fraction = remaining / weight
>             total_value += value * fraction
>             selected.append((idx, remaining, value * fraction))
>             remaining = 0
>             break
>     
>     return total_value, selected
> 
> # 예제
> items = [(60, 10), (100, 20), (120, 30)]  # (가치, 무게)
> capacity = 50
> 
> max_value, selection = fractional_knapsack(items, capacity)
> print(f"최대 가치: {max_value}")  # 240.0
> print(f"선택 내역: {selection}")
> ```

### 예제 3: 허프만 코딩 (Huffman Coding)

> [!example] 최적 압축을 위한 코드 생성
> ```python
> import heapq
> from collections import Counter
> 
> class HuffmanNode:
>     def __init__(self, char, freq, left=None, right=None):
>         self.char = char
>         self.freq = freq
>         self.left = left
>         self.right = right
>     
>     def __lt__(self, other):
>         return self.freq < other.freq
> 
> def huffman_coding(text):
>     """텍스트를 위한 허프만 코드 생성"""
>     # 빈도 계산
>     freq = Counter(text)
>     
>     # 최소 힙 초기화
>     heap = [HuffmanNode(char, f) for char, f in freq.items()]
>     heapq.heapify(heap)
>     
>     # 허프만 트리 구성
>     while len(heap) > 1:
>         left = heapq.heappop(heap)
>         right = heapq.heappop(heap)
>         
>         merged = HuffmanNode(
>             None, 
>             left.freq + right.freq,
>             left,
>             right
>         )
>         heapq.heappush(heap, merged)
>     
>     # 코드 생성
>     def generate_codes(node, code="", codes={}):
>         if node:
>             if node.char:  # 리프 노드
>                 codes[node.char] = code or "0"
>             else:
>                 generate_codes(node.left, code + "0", codes)
>                 generate_codes(node.right, code + "1", codes)
>         return codes
>     
>     root = heap[0] if heap else None
>     codes = generate_codes(root)
>     
>     # 인코딩
>     encoded = ''.join(codes[char] for char in text)
>     
>     return codes, encoded
> 
> # 예제
> text = "aabbccddddeeeeee"
> codes, encoded = huffman_coding(text)
> 
> print("허프만 코드:")
> for char, code in sorted(codes.items()):
>     print(f"  '{char}': {code}")
> 
> print(f"\n원본 크기: {len(text) * 8} bits")
> print(f"압축 크기: {len(encoded)} bits")
> print(f"압축률: {len(encoded) / (len(text) * 8) * 100:.1f}%")
> ```

### 예제 4: 작업 스케줄링

> [!example] 마감 기한이 있는 작업 스케줄링
> ```python
> def job_scheduling_with_deadline(jobs):
>     """
>     jobs = [(job_id, deadline, profit), ...]
>     마감 기한 내에 최대 이익을 얻는 스케줄링
>     """
>     # 이익 기준 내림차순 정렬
>     jobs.sort(key=lambda x: x[2], reverse=True)
>     
>     # 최대 마감 시간 찾기
>     max_deadline = max(job[1] for job in jobs)
>     
>     # 스케줄 초기화 (0: 비어있음)
>     schedule = [0] * (max_deadline + 1)
>     total_profit = 0
>     scheduled_jobs = []
>     
>     for job_id, deadline, profit in jobs:
>         # 마감 시간부터 거꾸로 빈 슬롯 찾기
>         for slot in range(min(deadline, max_deadline), 0, -1):
>             if schedule[slot] == 0:
>                 schedule[slot] = job_id
>                 total_profit += profit
>                 scheduled_jobs.append(job_id)
>                 break
>     
>     return scheduled_jobs, total_profit
> 
> # 예제
> jobs = [
>     ("J1", 2, 100),
>     ("J2", 1, 19),
>     ("J3", 2, 27),
>     ("J4", 1, 25),
>     ("J5", 3, 15)
> ]
> 
> scheduled, profit = job_scheduling_with_deadline(jobs)
> print(f"스케줄된 작업: {scheduled}")  # ['J1', 'J3', 'J5']
> print(f"최대 이익: {profit}")  # 142
> ```

---

## ⚠️ 주의사항

> [!warning] 탐욕 알고리즘의 한계

### 1. 최적해 보장 불가

```python
# 반례: 동전 거스름돈
coins = [1, 3, 4]
amount = 6

# 탐욕: [4, 1, 1] = 3개
# 최적: [3, 3] = 2개

def is_greedy_optimal(coins, amount):
    """탐욕 알고리즘이 최적인지 확인"""
    # 구현: DP와 Greedy 결과 비교
    pass
```

### 2. 탐욕 선택 검증 필요

> [!danger] 증명 없이 사용 금지
> ```python
> # 잘못된 탐욕 선택
> def wrong_greedy_knapsack(items, capacity):
>     # 가치가 높은 것부터? → 틀림
>     # 가벼운 것부터? → 틀림
>     # 가치/무게 비율? → 0-1 배낭에서는 틀림!
>     pass
> ```

### 3. 문제 변형 시 재검토

> [!note] 조건 변경 주의
> ```python
> # 활동 선택: 종료 시간 정렬 → 정답
> # 변형: 가중치가 있는 활동 선택 → 탐욕 실패
> 
> def weighted_activity_selection(activities_with_weights):
>     # 탐욕으로는 최적해 보장 못함
>     # DP 필요!
>     pass
> ```

### 4. 근사 알고리즘으로 활용

```python
def approximation_greedy(problem):
    """
    최적해는 아니지만 충분히 좋은 해
    예: TSP의 최근접 이웃 알고리즘
    """
    # 2-근사, 1.5-근사 등
    pass
```

---

## 📚 참고자료

### 관련 문서
- [[04_problem-solving-strategies|문제 해결 전략]]
- [[05_dynamic-programming|동적 계획법]]
- [[01_algorithm-performance|알고리즘 성능 분석]]

### 추천 자료
- "Introduction to Algorithms" - CLRS, Chapter 16
- [Greedy Algorithms - GeeksforGeeks](https://www.geeksforgeeks.org/greedy-algorithms/)
- "Algorithm Design" - Kleinberg & Tardos

### 연습 문제
1. **기초**: 동전 거스름돈, 회의실 배정
2. **중급**: 최소 신장 트리 (Kruskal/Prim), Dijkstra 최단 경로
3. **심화**: Job Scheduling, Huffman Coding 구현

### 탐욕이 최적인 유명 문제들
- Minimum Spanning Tree (Kruskal, Prim)
- Single Source Shortest Path (Dijkstra)
- Huffman Coding
- Activity Selection
- Fractional Knapsack
- Egyptian Fraction

---

> [!quote]
> "탐욕 알고리즘은 미래를 내다보지 않고 현재의 최선을 선택한다. 때로는 이것이 최고의 전략이 된다." - Donald Knuth