---
tags:
  - algorithm
  - divide-and-conquer
  - recursion
  - problem-solving
  - cs-fundamentals
created: 2025-01-04
updated: "updated: 2025-09-05"
aliases:
  - 분할 정복
  - Divide and Conquer
  - D&C
description: 분할 정복 알고리즘 패러다임의 개념과 응용
status: published
category: tutorial
---

# 분할 정복 (Divide and Conquer)

> [!info] 개요
> 분할 정복은 복잡한 문제를 작은 부분 문제로 나누어 해결한 후, 그 결과를 합쳐서 원래 문제를 해결하는 알고리즘 설계 패러다임입니다.

## 📑 목차

- [[#🎯 분할 정복이란?]]
- [[#⚙️ 3단계 프로세스]]
- [[#📊 시간 복잡도 분석]]
- [[#💡 대표적인 알고리즘]]
- [[#🔍 병합 정렬 심화]]
- [[#⚡ 퀵 정렬 심화]]
- [[#🎯 이진 탐색 응용]]
- [[#💪 고급 응용]]
- [[#📚 참고자료]]

---

## 🎯 분할 정복이란?

> [!note] 핵심 원리
> "큰 문제를 작은 문제로 나누고, 각각을 해결한 후 합쳐라"

### 분할 정복의 철학

```
    [큰 문제]
       ↓ 분할(Divide)
   [작은 문제들]
       ↓ 정복(Conquer)
   [부분 해결책]
       ↓ 결합(Combine)
    [전체 해답]
```

### 적용 조건

> [!tip] 분할 정복 적용 가능 조건
> 1. **분할 가능성**: 문제를 더 작은 하위 문제로 분할 가능
> 2. **독립성**: 하위 문제들이 서로 독립적
> 3. **동일 구조**: 하위 문제가 원래 문제와 같은 형태
> 4. **결합 가능**: 하위 문제 해답을 결합하여 원래 문제 해결 가능

---

## ⚙️ 3단계 프로세스

### 1️⃣ 분할 (Divide)
문제를 더 작은 하위 문제로 분할

### 2️⃣ 정복 (Conquer)
하위 문제를 재귀적으로 해결

### 3️⃣ 결합 (Combine)
하위 문제의 해를 결합하여 원래 문제 해결

> [!example] 일반적인 구조
> ```python
> def divide_and_conquer(problem):
>     # 기저 사례: 문제가 충분히 작으면 직접 해결
>     if is_small_enough(problem):
>         return solve_directly(problem)
>     
>     # 분할: 문제를 하위 문제로 나눔
>     subproblems = divide(problem)
>     
>     # 정복: 각 하위 문제를 재귀적으로 해결
>     subsolutions = []
>     for subproblem in subproblems:
>         subsolutions.append(divide_and_conquer(subproblem))
>     
>     # 결합: 하위 해답을 합쳐서 전체 해답 생성
>     return combine(subsolutions)
> ```

---

## 📊 시간 복잡도 분석

### 마스터 정리 (Master Theorem)

재귀 관계식: **T(n) = aT(n/b) + f(n)**

- `a`: 하위 문제의 개수
- `b`: 각 하위 문제의 크기 (n/b)
- `f(n)`: 분할과 결합 비용

> [!info] 마스터 정리 케이스
> 
> **Case 1**: f(n) = O(n^c) where c < log_b(a)
> → T(n) = O(n^(log_b(a)))
> 
> **Case 2**: f(n) = O(n^(log_b(a)))
> → T(n) = O(n^(log_b(a)) * log n)
> 
> **Case 3**: f(n) = O(n^c) where c > log_b(a)
> → T(n) = O(f(n))

### 예제 분석

| 알고리즘 | 재귀 관계식 | 시간 복잡도 |
|---------|------------|------------|
| 이진 탐색 | T(n) = T(n/2) + O(1) | O(log n) |
| 병합 정렬 | T(n) = 2T(n/2) + O(n) | O(n log n) |
| 퀵 정렬(평균) | T(n) = 2T(n/2) + O(n) | O(n log n) |
| 퀵 정렬(최악) | T(n) = T(n-1) + O(n) | O(n²) |

---

## 💡 대표적인 알고리즘

### 분할 정복 알고리즘 목록

1. **병합 정렬 (Merge Sort)**
2. **퀵 정렬 (Quick Sort)**
3. **이진 탐색 (Binary Search)**
4. **거듭제곱 계산 (Power)**
5. **행렬 곱셈 (Strassen)**
6. **최근접 점쌍 (Closest Pair)**
7. **고속 푸리에 변환 (FFT)**

---

## 🔍 병합 정렬 심화

> [!info] 병합 정렬
> 배열을 반으로 나누고, 각각을 정렬한 후 병합하는 안정적인 정렬 알고리즘

### 구현

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    # 분할
    mid = len(arr) // 2
    left = arr[:mid]
    right = arr[mid:]
    
    # 정복 (재귀 호출)
    left = merge_sort(left)
    right = merge_sort(right)
    
    # 결합 (병합)
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    
    # 두 배열을 비교하며 병합
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    # 남은 요소 추가
    result.extend(left[i:])
    result.extend(right[j:])
    
    return result
```

### 시각화

```
        [38, 27, 43, 3, 9, 82, 10]
              /            \
      [38, 27, 43, 3]    [9, 82, 10]
         /      \           /     \
    [38, 27]  [43, 3]   [9, 82]  [10]
     /   \     /   \     /   \     |
   [38] [27] [43] [3]  [9] [82]  [10]
     \   /     \   /     \   /     |
    [27, 38]  [3, 43]   [9, 82]  [10]
         \      /           \     /
      [3, 27, 38, 43]    [9, 10, 82]
              \            /
        [3, 9, 10, 27, 38, 43, 82]
```

### 특징

> [!success] 장점
> - **안정 정렬**: 동일한 값의 순서 유지
> - **예측 가능**: 항상 O(n log n)
> - **외부 정렬 가능**: 대용량 데이터 처리

> [!failure] 단점
> - **추가 메모리**: O(n) 공간 필요
> - **작은 데이터 비효율**: 오버헤드 존재

---

## ⚡ 퀵 정렬 심화

> [!info] 퀵 정렬
> 피벗을 기준으로 분할하여 정렬하는 효율적인 제자리 정렬 알고리즘

### 구현

```python
def quick_sort(arr, low=0, high=None):
    if high is None:
        high = len(arr) - 1
    
    if low < high:
        # 분할: 피벗 위치 찾기
        pivot_index = partition(arr, low, high)
        
        # 정복: 피벗 기준 양쪽 정렬
        quick_sort(arr, low, pivot_index - 1)
        quick_sort(arr, pivot_index + 1, high)
    
    return arr

def partition(arr, low, high):
    # 피벗 선택 (마지막 요소)
    pivot = arr[high]
    i = low - 1  # 작은 요소의 끝 인덱스
    
    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    
    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1
```

### 피벗 선택 전략

> [!tip] 피벗 선택 방법
> 1. **첫 번째/마지막 요소**: 간단하지만 정렬된 데이터에 취약
> 2. **중간값**: 정렬된 데이터에 효과적
> 3. **랜덤**: 평균적으로 좋은 성능
> 4. **Median of Three**: 첫/중간/끝 요소의 중간값

```python
def median_of_three(arr, low, high):
    mid = (low + high) // 2
    
    # 세 값을 정렬
    if arr[low] > arr[mid]:
        arr[low], arr[mid] = arr[mid], arr[low]
    if arr[mid] > arr[high]:
        arr[mid], arr[high] = arr[high], arr[mid]
    if arr[low] > arr[mid]:
        arr[low], arr[mid] = arr[mid], arr[low]
    
    # 중간값을 피벗 위치로
    return mid
```

### 성능 비교

| 경우 | 시간 복잡도 | 발생 조건 |
|------|------------|-----------|
| 최선 | O(n log n) | 균등 분할 |
| 평균 | O(n log n) | 랜덤 데이터 |
| 최악 | O(n²) | 불균등 분할 (정렬된 데이터) |

---

## 🎯 이진 탐색 응용

### 기본 이진 탐색

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1
```

### 변형 문제들

> [!example] Lower Bound (첫 번째 위치)
> ```python
> def lower_bound(arr, target):
>     left, right = 0, len(arr)
>     
>     while left < right:
>         mid = (left + right) // 2
>         if arr[mid] < target:
>             left = mid + 1
>         else:
>             right = mid
>     
>     return left
> ```

> [!example] Upper Bound (마지막 다음 위치)
> ```python
> def upper_bound(arr, target):
>     left, right = 0, len(arr)
>     
>     while left < right:
>         mid = (left + right) // 2
>         if arr[mid] <= target:
>             left = mid + 1
>         else:
>             right = mid
>     
>     return left
> ```

### 실수 범위 이진 탐색

```python
def binary_search_float(f, target, epsilon=1e-9):
    """
    f: 단조 증가 함수
    target: 찾고자 하는 값
    epsilon: 오차 허용 범위
    """
    left, right = 0.0, 1000.0  # 범위 설정
    
    while right - left > epsilon:
        mid = (left + right) / 2
        
        if f(mid) < target:
            left = mid
        else:
            right = mid
    
    return (left + right) / 2
```

---

## 💪 고급 응용

### 거듭제곱 계산

> [!example] 빠른 거듭제곱
> ```python
> def power(base, exp):
>     if exp == 0:
>         return 1
>     if exp == 1:
>         return base
>     
>     # 분할
>     half = power(base, exp // 2)
>     
>     # 결합
>     if exp % 2 == 0:
>         return half * half
>     else:
>         return half * half * base
> 
> # 시간 복잡도: O(log n)
> ```

### 최근접 점쌍 문제

```python
import math

def closest_pair(points):
    """2D 평면에서 가장 가까운 두 점 찾기"""
    
    def distance(p1, p2):
        return math.sqrt((p1[0] - p2[0])**2 + (p1[1] - p2[1])**2)
    
    def brute_force(points):
        min_dist = float('inf')
        n = len(points)
        for i in range(n):
            for j in range(i+1, n):
                min_dist = min(min_dist, distance(points[i], points[j]))
        return min_dist
    
    def closest_pair_recursive(px, py):
        n = len(px)
        
        # 작은 경우 브루트 포스
        if n <= 3:
            return brute_force(px)
        
        # 중간점으로 분할
        mid = n // 2
        midpoint = px[mid]
        
        # y 좌표로 정렬된 좌/우 부분
        pyl = [p for p in py if p[0] <= midpoint[0]]
        pyr = [p for p in py if p[0] > midpoint[0]]
        
        # 재귀 호출
        dl = closest_pair_recursive(px[:mid], pyl)
        dr = closest_pair_recursive(px[mid:], pyr)
        
        # 최소 거리
        d = min(dl, dr)
        
        # 중간 영역 확인
        strip = [p for p in py if abs(p[0] - midpoint[0]) < d]
        
        # strip 내 점들 확인
        for i in range(len(strip)):
            j = i + 1
            while j < len(strip) and (strip[j][1] - strip[i][1]) < d:
                d = min(d, distance(strip[i], strip[j]))
                j += 1
        
        return d
    
    # x, y 좌표로 정렬
    px = sorted(points, key=lambda p: p[0])
    py = sorted(points, key=lambda p: p[1])
    
    return closest_pair_recursive(px, py)
```

### 행렬 곱셈 (Strassen)

```python
def strassen(A, B):
    """
    Strassen 알고리즘: O(n^2.807) 시간 복잡도
    일반 행렬 곱셈: O(n^3)
    """
    n = len(A)
    
    # 기저 사례
    if n == 1:
        return [[A[0][0] * B[0][0]]]
    
    # 행렬 분할
    mid = n // 2
    A11, A12, A21, A22 = split_matrix(A)
    B11, B12, B21, B22 = split_matrix(B)
    
    # Strassen 공식
    M1 = strassen(add(A11, A22), add(B11, B22))
    M2 = strassen(add(A21, A22), B11)
    M3 = strassen(A11, subtract(B12, B22))
    M4 = strassen(A22, subtract(B21, B11))
    M5 = strassen(add(A11, A12), B22)
    M6 = strassen(subtract(A21, A11), add(B11, B12))
    M7 = strassen(subtract(A12, A22), add(B21, B22))
    
    # 결과 조합
    C11 = add(subtract(add(M1, M4), M5), M7)
    C12 = add(M3, M5)
    C21 = add(M2, M4)
    C22 = add(subtract(add(M1, M3), M2), M6)
    
    return combine_matrix(C11, C12, C21, C22)
```

---

## 📚 참고자료

### 추가 학습 자료
- [[01_algorithm-performance|알고리즘 성능 분석]]
- [[03_sorting-algorithms|정렬 알고리즘 비교]]
- [[01_data-structures-overview|자료구조와 분할 정복]]

### 연습 문제

> [!question] 도전 과제
> 1. 배열의 역순 쌍 개수 세기 (병합 정렬 활용)
> 2. k번째 작은 원소 찾기 (퀵 선택)
> 3. 최대 부분 배열 합 (분할 정복)
> 4. 정렬된 행렬에서 원소 찾기

### 분할 정복 vs 동적 프로그래밍

| 특성 | 분할 정복 | 동적 프로그래밍 |
|------|----------|---------------|
| 하위 문제 | 독립적 | 중복 가능 |
| 메모이제이션 | 불필요 | 필수 |
| 접근 방식 | Top-down | Bottom-up 가능 |
| 예제 | 병합 정렬 | 피보나치 |

---

> [!quote]
> "분할하여 정복하라 (Divide et impera)" - 필립 2세
---

## 🔄 재귀함수의 구조와 복잡도 분석

### 재귀의 두 가지 필수 조건

> [!important] 재귀 설계의 핵심
> 재귀함수가 올바르게 동작하려면 반드시 두 가지 조건이 필요합니다.

#### 1️⃣ 초기 조건 (Base Case)

> [!tip] 역할
> 재귀 호출을 멈추고 값을 직접 반환할 수 있도록 해주는 조건입니다.

```python
def factorial(n):
    if n == 0:  # ← 이게 base case!
        return 1
    return n * factorial(n - 1)
```

초기 조건이 없으면 **무한 재귀**로 스택 오버플로우 발생!

#### 2️⃣ 반복 조건 (Recursive Case)  

> [!tip] 역할
> base case가 아닌 경우, 문제를 더 작은 문제로 쪼개서 자기 자신을 호출하는 규칙입니다.

```python
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)  # ← 이게 반복 조건!
```

### 재귀와 점화식의 관계

> [!abstract] 수학적 사고
> 재귀를 수학적으로 이해하고 싶다면 **점화식**으로 생각하세요.

- 반복 조건 = 점화식의 재귀항
- 초기 조건 = 점화식의 시작값
- 점화식을 풀면 시간복잡도 계산 가능

예시:
```
T(n) = T(n-1) + 1, T(1) = 1 → T(n) = O(n)
T(n) = 2T(n-1) + 1 → T(n) = O(2ⁿ)
T(n) = T(n/2) + 1 → T(n) = O(log n)
T(n) = 2T(n/2) + n → T(n) = O(n log n)
```

### 시간복잡도 영향 요인

> [!example] 계산 공식
> ```
> 총 시간복잡도 = 호출 횟수 × 각 호출의 비용
> ```

**대표적인 재귀 패턴과 복잡도:**

| 재귀 패턴 | 점화식 | 시간복잡도 | 예시 |
|---------|--------|-----------|------|
| 선형 재귀 | T(n) = T(n-1) + O(1) | O(n) | 팩토리얼 |
| 이진 재귀 | T(n) = 2T(n-1) + O(1) | O(2ⁿ) | 피보나치 (naive) |
| 분할 정복 (반분) | T(n) = T(n/2) + O(1) | O(log n) | 이진 탐색 |
| 분할 정복 (양분) | T(n) = 2T(n/2) + O(n) | O(n log n) | 병합 정렬 |
| 분할 정복 (불균등) | T(n) = T(n-1) + O(n) | O(n²) | 퀵 정렬 (최악) |

### 재귀 최적화 기법

> [!success] 성능 개선 방법
> 1. **메모이제이션**: 중복 계산 결과 캐싱
> 2. **꼬리 재귀**: 스택 사용 최소화
> 3. **반복문 변환**: 단순 재귀는 반복문으로
> 4. **동적 프로그래밍**: Bottom-up 접근

#### 피보나치 예시: 재귀 vs 동적 프로그래밍

```python
# 비효율적인 재귀 - O(2ⁿ)
def fib_recursive(n):
    if n <= 1:
        return n
    return fib_recursive(n-1) + fib_recursive(n-2)

# 메모이제이션 - O(n)
def fib_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo)
    return memo[n]

# 동적 프로그래밍 - O(n)
def fib_dp(n):
    if n <= 1:
        return n
    dp = [0, 1]
    for i in range(2, n + 1):
        dp.append(dp[-1] + dp[-2])
    return dp[n]
```

> [!quote]
> "재귀는 점화식을 코드로 옮긴 것이다. 수학적 사고로 재귀를 이해할 때 진짜 성장이 시작된다."