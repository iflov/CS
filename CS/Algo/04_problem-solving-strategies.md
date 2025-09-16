---
tags:
  - algorithm
  - problem-solving
  - optimization
  - memoization
  - strategy
created: 2025-01-06
updated: 2025-01-06
aliases:
  - 문제 해결 전략
  - 알고리즘 설계
  - Problem Solving
description: 효율적인 문제 해결을 위한 전략과 최적화 기법
---

# 문제 해결 전략

> [!info] 개요
> 문제 해결 전략은 복잡한 계산 문제를 효율적으로 해결하기 위한 체계적인 접근 방법입니다. 메모이제이션, 동적 계획법, 탐욕 알고리즘 등 다양한 전략을 통해 시간과 공간 복잡도를 획기적으로 개선할 수 있습니다.

## 📑 목차

- [[#⚡ 빠른 시작]]
- [[#🔧 핵심 개념]]
- [[#💡 실전 예제]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## ⚡ 빠른 시작

> [!example] 피보나치 수열: 나쁜 방법 vs 좋은 방법
> ```python
> # 나쁜 방법: 단순 재귀 (O(2^n))
> def fib_bad(n):
>     if n <= 1:
>         return n
>     return fib_bad(n-1) + fib_bad(n-2)
> 
> # 좋은 방법: 메모이제이션 (O(n))
> def fib_good(n, memo={}):
>     if n in memo:
>         return memo[n]
>     if n <= 1:
>         return n
>     memo[n] = fib_good(n-1, memo) + fib_good(n-2, memo)
>     return memo[n]
> 
> # fib_bad(40) → 수십 초
> # fib_good(40) → 즉시 반환
> ```

적절한 전략을 선택하면 **불가능해 보이는 문제도 해결 가능**합니다.

---

## 🔧 핵심 개념

### 문제 해결 단계

> [!note] 체계적 접근법
> 1. **문제 이해**: 입력, 출력, 제약사항 파악
> 2. **패턴 인식**: 유사한 문제나 부분 문제 찾기
> 3. **전략 선택**: 적절한 알고리즘 패러다임 선택
> 4. **구현**: 선택한 전략으로 코드 작성
> 5. **최적화**: 병목 지점 개선

### 주요 전략 분류

#### 1. 완전 탐색 (Brute Force)

> [!tip] 모든 경우의 수를 확인
> - **장점**: 정확한 답 보장, 구현 간단
> - **단점**: 시간 복잡도 높음
> - **사용 시기**: 입력 크기가 작거나 다른 방법이 없을 때

```python
# 부분집합 생성 (2^n 가지)
def generate_subsets(arr):
    n = len(arr)
    subsets = []
    for i in range(2**n):
        subset = []
        for j in range(n):
            if i & (1 << j):
                subset.append(arr[j])
        subsets.append(subset)
    return subsets
```

#### 2. 분할 정복 (Divide and Conquer)

> [!tip] 문제를 작은 부분으로 나누어 해결
> - **핵심**: 분할 → 정복 → 결합
> - **예시**: 병합 정렬, 퀵 정렬, 이진 탐색

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])    # 분할
    right = merge_sort(arr[mid:])   # 분할
    
    # 정복 및 결합
    return merge(left, right)
```

#### 3. 메모이제이션 (Memoization)

> [!important] 계산 결과를 저장하여 재사용
> 중복 계산을 제거하는 최적화 기법

```python
def memoize(func):
    cache = {}
    def wrapper(*args):
        if args in cache:
            return cache[args]
        result = func(*args)
        cache[args] = result
        return result
    return wrapper

@memoize
def expensive_function(n):
    # 복잡한 계산
    return result
```

### 최적화 원칙

#### 중복 제거

> [!example] 중복 계산 식별
> ```python
> # 중복이 많은 재귀
> def count_paths(m, n):
>     if m == 1 or n == 1:
>         return 1
>     return count_paths(m-1, n) + count_paths(m, n-1)
> 
> # 같은 (m,n)이 여러 번 계산됨
> # 해결: 메모이제이션 또는 동적 계획법
> ```

#### 불필요한 계산 제거

> [!tip] 가지치기 (Pruning)
> 답이 될 수 없는 경우를 조기에 제외

```python
def n_queens(n):
    def is_safe(board, row, col):
        # 가지치기: 놓을 수 없는 위치 빠르게 판단
        for i in range(row):
            if board[i] == col or \
               board[i] - i == col - row or \
               board[i] + i == col + row:
                return False
        return True
    
    def solve(board, row):
        if row == n:
            return 1
        count = 0
        for col in range(n):
            if is_safe(board, row, col):
                board[row] = col
                count += solve(board, row + 1)
        return count
    
    return solve([-1] * n, 0)
```

---

## 💡 실전 예제

### 예제 1: 최장 공통 부분 수열 (LCS)

> [!example] 여러 전략 비교
> ```python
> # 전략 1: 완전 탐색 (O(2^n))
> def lcs_brute(X, Y, m, n):
>     if m == 0 or n == 0:
>         return 0
>     if X[m-1] == Y[n-1]:
>         return 1 + lcs_brute(X, Y, m-1, n-1)
>     return max(lcs_brute(X, Y, m, n-1),
>                lcs_brute(X, Y, m-1, n))
> 
> # 전략 2: 메모이제이션 (O(mn))
> def lcs_memo(X, Y):
>     memo = {}
>     def helper(m, n):
>         if (m, n) in memo:
>             return memo[(m, n)]
>         if m == 0 or n == 0:
>             return 0
>         if X[m-1] == Y[n-1]:
>             result = 1 + helper(m-1, n-1)
>         else:
>             result = max(helper(m, n-1), helper(m-1, n))
>         memo[(m, n)] = result
>         return result
>     return helper(len(X), len(Y))
> 
> # 전략 3: 동적 계획법 (O(mn))
> def lcs_dp(X, Y):
>     m, n = len(X), len(Y)
>     dp = [[0] * (n+1) for _ in range(m+1)]
>     
>     for i in range(1, m+1):
>         for j in range(1, n+1):
>             if X[i-1] == Y[j-1]:
>                 dp[i][j] = dp[i-1][j-1] + 1
>             else:
>                 dp[i][j] = max(dp[i-1][j], dp[i][j-1])
>     
>     return dp[m][n]
> ```

### 예제 2: 부분집합 합 문제

> [!example] 백트래킹과 메모이제이션
> ```python
> # 백트래킹으로 해결
> def subset_sum_backtrack(nums, target):
>     def backtrack(start, current_sum, path):
>         if current_sum == target:
>             return path[:]
>         if current_sum > target or start == len(nums):
>             return None
>         
>         # 현재 원소 포함
>         path.append(nums[start])
>         result = backtrack(start + 1, current_sum + nums[start], path)
>         if result:
>             return result
>         path.pop()
>         
>         # 현재 원소 제외
>         return backtrack(start + 1, current_sum, path)
>     
>     return backtrack(0, 0, [])
> 
> # 메모이제이션으로 최적화
> def can_partition(nums):
>     total = sum(nums)
>     if total % 2 != 0:
>         return False
>     
>     target = total // 2
>     memo = {}
>     
>     def dp(i, current_sum):
>         if current_sum == target:
>             return True
>         if i >= len(nums) or current_sum > target:
>             return False
>         if (i, current_sum) in memo:
>             return memo[(i, current_sum)]
>         
>         result = dp(i + 1, current_sum + nums[i]) or \
>                  dp(i + 1, current_sum)
>         memo[(i, current_sum)] = result
>         return result
>     
>     return dp(0, 0)
> ```

### 예제 3: 최단 경로 문제

> [!example] 다양한 접근법
> ```python
> # 그리디: 다익스트라 알고리즘
> import heapq
> 
> def dijkstra(graph, start):
>     distances = {node: float('inf') for node in graph}
>     distances[start] = 0
>     pq = [(0, start)]
>     
>     while pq:
>         current_dist, current_node = heapq.heappop(pq)
>         
>         if current_dist > distances[current_node]:
>             continue
>         
>         for neighbor, weight in graph[current_node].items():
>             distance = current_dist + weight
>             if distance < distances[neighbor]:
>                 distances[neighbor] = distance
>                 heapq.heappush(pq, (distance, neighbor))
>     
>     return distances
> 
> # DP: 벨만-포드 알고리즘
> def bellman_ford(graph, start, n):
>     dist = [float('inf')] * n
>     dist[start] = 0
>     
>     for _ in range(n - 1):
>         for u in range(n):
>             for v, weight in graph[u]:
>                 if dist[u] != float('inf') and \
>                    dist[u] + weight < dist[v]:
>                     dist[v] = dist[u] + weight
>     
>     return dist
> ```

---

## ⚠️ 주의사항

> [!warning] 전략 선택 시 고려사항

### 1. 문제 특성 파악

```python
# 최적 부분 구조가 있는가? → DP 고려
# 탐욕 선택이 가능한가? → Greedy 고려
# 중복 계산이 많은가? → Memoization 고려
# 해공간이 작은가? → 완전 탐색 고려
```

### 2. 시간/공간 트레이드오프

> [!danger] 메모리 제한 확인
> ```python
> # 메모리 초과 위험
> def dp_2d_large(n=10000, m=10000):
>     dp = [[0] * m for _ in range(n)]  # 400MB+
>     # ...
> 
> # 공간 최적화
> def dp_1d_optimized(n, m):
>     prev = [0] * m
>     curr = [0] * m
>     # 이전 행만 저장
> ```

### 3. 구현 복잡도

> [!note] 단순한 해법이 더 나을 때
> ```python
> # 과도한 최적화
> def over_optimized_solution():
>     # 200줄의 복잡한 코드
>     pass
> 
> # 충분히 빠른 단순 해법
> def simple_solution():
>     # 20줄의 명확한 코드
>     pass
> 
> # n < 1000이면 simple이 더 나음
> ```

### 4. 엣지 케이스

```python
# 항상 확인해야 할 것들
- 빈 입력
- 단일 원소
- 모두 같은 값
- 음수 값
- 오버플로우 가능성
- 최악의 경우
```

---

## 📚 참고자료

### 관련 문서
- [[05_dynamic-programming|동적 계획법]]
- [[06_greedy-algorithms|탐욕 알고리즘]]
- [[01_algorithm-performance|알고리즘 성능 분석]]

### 추천 도서
- "알고리즘 문제 해결 전략" - 구종만
- "Introduction to Algorithms" - CLRS
- "The Algorithm Design Manual" - Steven Skiena

### 온라인 리소스
- [LeetCode Patterns](https://leetcode.com/discuss/general-discussion/458695/dynamic-programming-patterns)
- [CP-Algorithms](https://cp-algorithms.com/)
- [Competitive Programming Handbook](https://cses.fi/book/book.pdf)

### 연습 플랫폼
- [LeetCode](https://leetcode.com)
- [HackerRank](https://www.hackerrank.com)
- [백준 온라인 저지](https://www.acmicpc.net)

---

> [!quote]
> "문제를 해결할 수 없다면, 더 쉬운 문제를 먼저 해결하라. 그 다음 원래 문제로 돌아가라." - George Pólya