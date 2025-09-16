---
tags:
  - algorithm
  - dynamic-programming
  - optimization
  - memoization
  - dp
created: 2025-01-06
updated: 2025-01-06
aliases:
  - 동적 계획법
  - DP
  - Dynamic Programming
description: 동적 계획법의 원리와 구현 패턴
---

# 동적 계획법 (Dynamic Programming)

> [!info] 개요
> 동적 계획법(DP)은 복잡한 문제를 작은 부분 문제로 나누어 해결하는 알고리즘 설계 기법입니다. 중복되는 부분 문제의 답을 저장하여 재활용함으로써 지수 시간 복잡도를 다항 시간으로 개선할 수 있습니다.

## 📑 목차

- [[#⚡ 빠른 시작]]
- [[#🔧 핵심 개념]]
- [[#💡 실전 예제]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## ⚡ 빠른 시작

> [!example] 피보나치: DP의 시작점
> ```python
> # Top-Down (메모이제이션)
> def fib_td(n, memo={}):
>     if n in memo:
>         return memo[n]
>     if n <= 1:
>         return n
>     memo[n] = fib_td(n-1, memo) + fib_td(n-2, memo)
>     return memo[n]
> 
> # Bottom-Up (타뷸레이션)
> def fib_bu(n):
>     if n <= 1:
>         return n
>     dp = [0] * (n + 1)
>     dp[1] = 1
>     for i in range(2, n + 1):
>         dp[i] = dp[i-1] + dp[i-2]
>     return dp[n]
> 
> # 공간 최적화
> def fib_optimized(n):
>     if n <= 1:
>         return n
>     prev, curr = 0, 1
>     for _ in range(2, n + 1):
>         prev, curr = curr, prev + curr
>     return curr
> ```

DP는 **"기억하며 계산하기"** - 한 번 계산한 것은 다시 계산하지 않습니다.

---

## 🔧 핵심 개념

### DP의 두 가지 조건

> [!important] DP 적용 가능 조건
> 1. **최적 부분 구조 (Optimal Substructure)**: 큰 문제의 최적해가 부분 문제의 최적해로 구성
> 2. **중복 부분 문제 (Overlapping Subproblems)**: 같은 부분 문제가 여러 번 계산됨

### DP 접근 방법

#### 1. Top-Down (메모이제이션)

> [!tip] 재귀 + 캐싱
> - 자연스러운 재귀 구조 유지
> - 필요한 부분만 계산
> - 스택 오버플로우 위험

```python
def solve_td(n):
    memo = {}
    
    def dp(state):
        if state in memo:
            return memo[state]
        
        if base_case(state):
            return base_value
        
        # 점화식
        result = recurrence_relation(state)
        memo[state] = result
        return result
    
    return dp(n)
```

#### 2. Bottom-Up (타뷸레이션)

> [!tip] 반복문 + 테이블
> - 작은 문제부터 순차적 해결
> - 모든 부분 문제 계산
> - 공간 최적화 가능

```python
def solve_bu(n):
    dp = [0] * (n + 1)
    
    # 초기값 설정
    dp[0] = base_value
    
    # 점화식 적용
    for i in range(1, n + 1):
        dp[i] = recurrence_relation(dp, i)
    
    return dp[n]
```

### DP 설계 단계

> [!note] 체계적 접근법
> 1. **상태 정의**: 부분 문제를 나타낼 변수 결정
> 2. **점화식 도출**: 상태 간의 관계식 찾기
> 3. **초기값 설정**: 기본 케이스 정의
> 4. **계산 순서 결정**: 의존 관계에 따른 순서
> 5. **구현 및 최적화**: 시간/공간 복잡도 개선

### 대표적인 DP 패턴

#### 1차원 DP

```python
# 계단 오르기
def climb_stairs(n):
    if n <= 2:
        return n
    dp = [0] * (n + 1)
    dp[1], dp[2] = 1, 2
    for i in range(3, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

#### 2차원 DP

```python
# 최소 경로 합
def min_path_sum(grid):
    m, n = len(grid), len(grid[0])
    dp = [[0] * n for _ in range(m)]
    
    # 초기값
    dp[0][0] = grid[0][0]
    for i in range(1, m):
        dp[i][0] = dp[i-1][0] + grid[i][0]
    for j in range(1, n):
        dp[0][j] = dp[0][j-1] + grid[0][j]
    
    # 점화식
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j]
    
    return dp[m-1][n-1]
```

---

## 💡 실전 예제

### 예제 1: 막대 자르기 (Rod Cutting)

> [!example] 최대 수익 구하기
> ```python
> def rod_cutting(prices, n):
>     """
>     prices[i]: 길이 i+1인 막대의 가격
>     n: 전체 막대 길이
>     """
>     # Top-Down
>     def cut_rod_td(n, memo={}):
>         if n in memo:
>             return memo[n]
>         if n == 0:
>             return 0
>         
>         max_val = 0
>         for i in range(1, n + 1):
>             if i <= len(prices):
>                 max_val = max(max_val, 
>                              prices[i-1] + cut_rod_td(n-i, memo))
>         
>         memo[n] = max_val
>         return max_val
>     
>     # Bottom-Up
>     def cut_rod_bu(n):
>         dp = [0] * (n + 1)
>         
>         for i in range(1, n + 1):
>             max_val = 0
>             for j in range(1, min(i, len(prices)) + 1):
>                 max_val = max(max_val, prices[j-1] + dp[i-j])
>             dp[i] = max_val
>         
>         return dp[n]
>     
>     # 최적 자르기 방법 추적
>     def cut_rod_solution(n):
>         dp = [0] * (n + 1)
>         cuts = [0] * (n + 1)
>         
>         for i in range(1, n + 1):
>             max_val = 0
>             for j in range(1, min(i, len(prices)) + 1):
>                 if prices[j-1] + dp[i-j] > max_val:
>                     max_val = prices[j-1] + dp[i-j]
>                     cuts[i] = j
>             dp[i] = max_val
>         
>         # 자르기 방법 복원
>         solution = []
>         while n > 0:
>             solution.append(cuts[n])
>             n -= cuts[n]
>         
>         return dp[-1], solution
>     
>     return cut_rod_bu(n)
> 
> # 사용 예제
> prices = [1, 5, 8, 9, 10, 17, 17, 20]
> print(rod_cutting(prices, 8))  # 22
> ```

### 예제 2: 0-1 배낭 문제

> [!example] 제한된 용량에서 최대 가치
> ```python
> def knapsack_01(weights, values, W):
>     """
>     weights: 각 아이템의 무게
>     values: 각 아이템의 가치
>     W: 배낭 용량
>     """
>     n = len(weights)
>     
>     # 2D DP
>     def knapsack_2d():
>         dp = [[0] * (W + 1) for _ in range(n + 1)]
>         
>         for i in range(1, n + 1):
>             for w in range(1, W + 1):
>                 # 현재 아이템을 넣지 않는 경우
>                 dp[i][w] = dp[i-1][w]
>                 
>                 # 현재 아이템을 넣는 경우
>                 if weights[i-1] <= w:
>                     dp[i][w] = max(dp[i][w],
>                                   dp[i-1][w-weights[i-1]] + values[i-1])
>         
>         return dp[n][W]
>     
>     # 공간 최적화 (1D DP)
>     def knapsack_1d():
>         dp = [0] * (W + 1)
>         
>         for i in range(n):
>             # 역순으로 순회 (중요!)
>             for w in range(W, weights[i] - 1, -1):
>                 dp[w] = max(dp[w], dp[w-weights[i]] + values[i])
>         
>         return dp[W]
>     
>     # 선택된 아이템 추적
>     def knapsack_with_items():
>         dp = [[0] * (W + 1) for _ in range(n + 1)]
>         
>         for i in range(1, n + 1):
>             for w in range(1, W + 1):
>                 dp[i][w] = dp[i-1][w]
>                 if weights[i-1] <= w:
>                     dp[i][w] = max(dp[i][w],
>                                   dp[i-1][w-weights[i-1]] + values[i-1])
>         
>         # 역추적
>         items = []
>         w = W
>         for i in range(n, 0, -1):
>             if dp[i][w] != dp[i-1][w]:
>                 items.append(i-1)
>                 w -= weights[i-1]
>         
>         return dp[n][W], items[::-1]
>     
>     return knapsack_1d()
> 
> # 사용 예제
> weights = [1, 3, 4, 5]
> values = [1, 4, 5, 7]
> W = 7
> print(knapsack_01(weights, values, W))  # 9
> ```

### 예제 3: 최장 증가 부분 수열 (LIS)

> [!example] 여러 DP 접근법
> ```python
> def longest_increasing_subsequence(nums):
>     """다양한 방법으로 LIS 구하기"""
>     n = len(nums)
>     if n == 0:
>         return 0
>     
>     # 방법 1: 기본 DP (O(n²))
>     def lis_basic():
>         dp = [1] * n
>         
>         for i in range(1, n):
>             for j in range(i):
>                 if nums[j] < nums[i]:
>                     dp[i] = max(dp[i], dp[j] + 1)
>         
>         return max(dp)
>     
>     # 방법 2: 이진 탐색 활용 (O(n log n))
>     def lis_binary_search():
>         from bisect import bisect_left
>         
>         dp = []
>         for num in nums:
>             pos = bisect_left(dp, num)
>             if pos == len(dp):
>                 dp.append(num)
>             else:
>                 dp[pos] = num
>         
>         return len(dp)
>     
>     # 방법 3: 실제 수열 복원
>     def lis_with_sequence():
>         dp = [1] * n
>         parent = [-1] * n
>         
>         for i in range(1, n):
>             for j in range(i):
>                 if nums[j] < nums[i] and dp[j] + 1 > dp[i]:
>                     dp[i] = dp[j] + 1
>                     parent[i] = j
>         
>         # 최장 길이와 끝 인덱스 찾기
>         max_len = max(dp)
>         max_idx = dp.index(max_len)
>         
>         # 수열 복원
>         lis = []
>         idx = max_idx
>         while idx != -1:
>             lis.append(nums[idx])
>             idx = parent[idx]
>         
>         return max_len, lis[::-1]
>     
>     return lis_basic()
> 
> # 사용 예제
> nums = [10, 9, 2, 5, 3, 7, 101, 18]
> print(longest_increasing_subsequence(nums))  # 4
> ```

---

## ⚠️ 주의사항

> [!warning] DP 구현 시 흔한 실수

### 1. 상태 정의 오류

```python
# 잘못된 상태 정의
def wrong_dp(n):
    dp = [0] * n  # 인덱스 오류 가능성
    # ...

# 올바른 상태 정의
def correct_dp(n):
    dp = [0] * (n + 1)  # n까지 포함
    # ...
```

### 2. 초기값 실수

> [!danger] 초기값 확인
> ```python
> # 잘못된 초기화
> dp = [[0] * n] * m  # 얕은 복사!
> 
> # 올바른 초기화
> dp = [[0] * n for _ in range(m)]
> ```

### 3. 계산 순서 오류

```python
# 의존성을 고려한 순서
def dependency_aware_dp(grid):
    # dp[i][j]가 dp[i-1][j]와 dp[i][j-1]에 의존
    for i in range(m):
        for j in range(n):
            # i, j 순서가 중요!
            dp[i][j] = compute(dp[i-1][j], dp[i][j-1])
```

### 4. 공간 복잡도 간과

> [!tip] 공간 최적화 기법
> ```python
> # O(n²) 공간
> dp = [[0] * n for _ in range(n)]
> 
> # O(n) 공간으로 최적화 가능한 경우
> prev = [0] * n
> curr = [0] * n
> # 또는 rolling array
> ```

---

## 📚 참고자료

### 관련 문서
- [[04_problem-solving-strategies|문제 해결 전략]]
- [[06_greedy-algorithms|탐욕 알고리즘]]
- [[02_recursion-and-tail-recursion|재귀와 꼬리 재귀]]

### 추천 자료
- [Dynamic Programming - GeeksforGeeks](https://www.geeksforgeeks.org/dynamic-programming/)
- [DP Patterns - LeetCode](https://leetcode.com/discuss/general-discussion/458695/dynamic-programming-patterns)
- "알고리즘 문제 해결 전략" - 구종만

### 연습 문제
1. **입문**: 계단 오르기, 집 도둑, 동전 거스름돈
2. **중급**: 편집 거리, LCS, 팰린드롬 분할
3. **고급**: 행렬 연쇄 곱셈, 외판원 문제(TSP), 트리 DP

---

> [!quote]
> "동적 계획법은 한 번의 계산으로 여러 번의 사용을 가능하게 하는, 프로그래밍의 재활용 예술이다." - Richard Bellman