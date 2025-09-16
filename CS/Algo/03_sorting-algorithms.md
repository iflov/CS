---
tags:
  - algorithm
  - sorting
  - complexity
  - comparison
  - cs-fundamentals
created: 2025-01-04
updated: "updated: 2025-09-05"
aliases:
  - 정렬 알고리즘
  - Sorting Algorithms
  - 정렬 비교
description: 다양한 정렬 알고리즘의 원리와 구현, 성능 비교
status: published
category: tutorial
---

# 정렬 알고리즘 완벽 가이드

> [!info] 개요
> 정렬은 데이터를 특정 순서로 배열하는 기본적이면서도 중요한 알고리즘입니다. 각 정렬 알고리즘의 특징과 상황별 최적 선택을 학습합니다.

## 📑 목차

- [[#⚡ 정렬 알고리즘 개요]]
- [[#🔄 버블 정렬 (Bubble Sort)]]
- [[#🎯 선택 정렬 (Selection Sort)]]
- [[#➡️ 삽입 정렬 (Insertion Sort)]]
- [[#🚀 병합 정렬 (Merge Sort)]]
- [[#⚡ 퀵 정렬 (Quick Sort)]]
- [[#🏔️ 힙 정렬 (Heap Sort)]]
- [[#📊 정렬 알고리즘 비교]]
- [[#💡 실전 가이드]]
- [[#📚 참고자료]]

---

## ⚡ 정렬 알고리즘 개요

### 정렬의 분류

> [!note] 분류 기준
> 
> **비교 기반 vs 비교 없음**
> - 비교 기반: 버블, 선택, 삽입, 병합, 퀵, 힙
> - 비교 없음: 계수, 기수, 버킷
> 
> **안정성 (Stability)**
> - 안정 정렬: 동일 값의 상대적 순서 유지
> - 불안정 정렬: 순서 보장 안됨
> 
> **제자리 정렬 (In-place)**
> - 제자리: O(1) 추가 공간
> - 비제자리: O(n) 이상 추가 공간

### 성능 개요

| 알고리즘 | 최선 | 평균 | 최악 | 공간 | 안정성 | 제자리 |
|---------|------|------|------|------|--------|--------|
| 버블 | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ |
| 선택 | O(n²) | O(n²) | O(n²) | O(1) | ❌ | ✅ |
| 삽입 | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ |
| 병합 | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ | ❌ |
| 퀵 | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ | ✅ |
| 힙 | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | ✅ |

---

## 🔄 버블 정렬 (Bubble Sort)

> [!info] 원리
> 인접한 두 원소를 비교하며 정렬. 큰 값이 거품처럼 뒤로 이동

### 구현

```python
def bubble_sort(arr):
    n = len(arr)
    
    for i in range(n):
        # 최적화: 교환이 없으면 정렬 완료
        swapped = False
        
        # 마지막 i개는 이미 정렬됨
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        
        if not swapped:
            break
    
    return arr
```

### 시각화

```
초기: [5, 3, 8, 4, 2]

Pass 1: [3, 5, 4, 2, 8]  # 8이 끝으로
Pass 2: [3, 4, 2, 5, 8]  # 5가 자리 찾음
Pass 3: [3, 2, 4, 5, 8]  # 4가 자리 찾음
Pass 4: [2, 3, 4, 5, 8]  # 완료
```

### 특징

> [!success] 장점
> - 구현이 매우 간단
> - 안정 정렬
> - 제자리 정렬

> [!failure] 단점
> - O(n²) 시간 복잡도로 느림
> - 많은 교환 연산

---

## 🎯 선택 정렬 (Selection Sort)

> [!info] 원리
> 최솟값을 찾아 맨 앞으로 이동시키는 과정을 반복

### 구현

```python
def selection_sort(arr):
    n = len(arr)
    
    for i in range(n):
        # 최솟값 찾기
        min_idx = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        
        # 최솟값을 맨 앞으로
        arr[i], arr[min_idx] = arr[min_idx], arr[i]
    
    return arr
```

### 시각화

```
초기: [64, 25, 12, 22, 11]

Pass 1: [11, 25, 12, 22, 64]  # 11 선택
Pass 2: [11, 12, 25, 22, 64]  # 12 선택
Pass 3: [11, 12, 22, 25, 64]  # 22 선택
Pass 4: [11, 12, 22, 25, 64]  # 25 선택
```

### 특징

> [!success] 장점
> - 교환 횟수가 최소 (최대 n-1번)
> - 예측 가능한 성능

> [!failure] 단점
> - 항상 O(n²) 시간 복잡도
> - 불안정 정렬

---

## ➡️ 삽입 정렬 (Insertion Sort)

> [!info] 원리
> 정렬된 부분에 새로운 원소를 올바른 위치에 삽입

### 구현

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1
        
        # key보다 큰 원소들을 오른쪽으로 이동
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1
        
        # key를 올바른 위치에 삽입
        arr[j + 1] = key
    
    return arr
```

### 시각화

```
초기: [12, 11, 13, 5, 6]

Pass 1: [11, 12, 13, 5, 6]  # 11 삽입
Pass 2: [11, 12, 13, 5, 6]  # 13 제자리
Pass 3: [5, 11, 12, 13, 6]  # 5 삽입
Pass 4: [5, 6, 11, 12, 13]  # 6 삽입
```

### 특징

> [!success] 장점
> - 거의 정렬된 데이터에 매우 효율적 O(n)
> - 온라인 알고리즘 (실시간 정렬)
> - 안정 정렬

> [!failure] 단점
> - 역순 정렬 시 최악 O(n²)
> - 많은 데이터 이동

### 이진 삽입 정렬

> [!tip] 개선 버전
> ```python
> def binary_insertion_sort(arr):
>     for i in range(1, len(arr)):
>         key = arr[i]
>         left, right = 0, i
>         
>         # 이진 탐색으로 삽입 위치 찾기
>         while left < right:
>             mid = (left + right) // 2
>             if arr[mid] > key:
>                 right = mid
>             else:
>                 left = mid + 1
>         
>         # 원소들 이동 후 삽입
>         for j in range(i - 1, left - 1, -1):
>             arr[j + 1] = arr[j]
>         arr[left] = key
>     
>     return arr
> ```

---

## 🚀 병합 정렬 (Merge Sort)

> [!info] 원리
> 분할 정복을 사용하여 배열을 반으로 나누고 정렬 후 병합

### 구현

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    
    return result
```

### 반복적 구현

```python
def merge_sort_iterative(arr):
    n = len(arr)
    size = 1
    
    while size < n:
        for start in range(0, n, size * 2):
            mid = min(start + size, n)
            end = min(start + size * 2, n)
            
            # 병합
            left = arr[start:mid]
            right = arr[mid:end]
            
            i = j = 0
            k = start
            
            while i < len(left) and j < len(right):
                if left[i] <= right[j]:
                    arr[k] = left[i]
                    i += 1
                else:
                    arr[k] = right[j]
                    j += 1
                k += 1
            
            while i < len(left):
                arr[k] = left[i]
                i += 1
                k += 1
            
            while j < len(right):
                arr[k] = right[j]
                j += 1
                k += 1
        
        size *= 2
    
    return arr
```

### 특징

> [!success] 장점
> - 안정적인 O(n log n) 성능
> - 안정 정렬
> - 대용량 데이터 적합 (외부 정렬)

> [!failure] 단점
> - O(n) 추가 메모리 필요
> - 작은 데이터에 오버헤드

---

## ⚡ 퀵 정렬 (Quick Sort)

> [!info] 원리
> 피벗을 기준으로 분할하여 정렬하는 분할 정복 알고리즘

### 구현

```python
def quick_sort(arr, low=0, high=None):
    if high is None:
        high = len(arr) - 1
    
    if low < high:
        # 파티션
        pi = partition(arr, low, high)
        
        # 재귀 정렬
        quick_sort(arr, low, pi - 1)
        quick_sort(arr, pi + 1, high)
    
    return arr

def partition(arr, low, high):
    # 피벗: 마지막 원소
    pivot = arr[high]
    i = low - 1
    
    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    
    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1
```

### 3-way 퀵 정렬 (중복 처리)

```python
def quick_sort_3way(arr, low=0, high=None):
    if high is None:
        high = len(arr) - 1
    
    if low < high:
        lt, gt = partition_3way(arr, low, high)
        quick_sort_3way(arr, low, lt - 1)
        quick_sort_3way(arr, gt + 1, high)
    
    return arr

def partition_3way(arr, low, high):
    pivot = arr[low]
    i = low
    lt = low
    gt = high
    
    while i <= gt:
        if arr[i] < pivot:
            arr[lt], arr[i] = arr[i], arr[lt]
            lt += 1
            i += 1
        elif arr[i] > pivot:
            arr[i], arr[gt] = arr[gt], arr[i]
            gt -= 1
        else:
            i += 1
    
    return lt, gt
```

### 특징

> [!success] 장점
> - 평균적으로 가장 빠름
> - 제자리 정렬
> - 캐시 효율성 좋음

> [!failure] 단점
> - 최악 시 O(n²)
> - 불안정 정렬
> - 피벗 선택이 중요

---

## 🏔️ 힙 정렬 (Heap Sort)

> [!info] 원리
> 최대 힙을 구성하고 루트를 추출하며 정렬

### 구현

```python
def heap_sort(arr):
    n = len(arr)
    
    # 최대 힙 구성
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)
    
    # 원소 추출
    for i in range(n - 1, 0, -1):
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr, i, 0)
    
    return arr

def heapify(arr, n, i):
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2
    
    if left < n and arr[left] > arr[largest]:
        largest = left
    
    if right < n and arr[right] > arr[largest]:
        largest = right
    
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)
```

### 시각화

```
     초기 배열: [12, 11, 13, 5, 6, 7]
     
     최대 힙 구성:
            13
          /    \
        11      12
       /  \    /
      5    6  7
     
     정렬 과정:
     [13, 11, 12, 5, 6, 7] → [7, 11, 12, 5, 6, |13]
     [12, 11, 7, 5, 6, |13] → [6, 11, 7, 5, |12, 13]
     ...
```

### 특징

> [!success] 장점
> - 항상 O(n log n)
> - 제자리 정렬
> - 최악의 경우도 보장

> [!failure] 단점
> - 불안정 정렬
> - 캐시 지역성 나쁨
> - 실제로는 퀵 정렬보다 느림

---

## 📊 정렬 알고리즘 비교

### 시간 복잡도 그래프

```
n² 정렬 (버블, 선택, 삽입)
시간 ↑     
     |    /
     |   /
     |  /
     | /
     |/___________→ n
     
n log n 정렬 (병합, 퀵, 힙)
시간 ↑
     |     __/
     |   _/
     |  /
     | /
     |/___________→ n
```

### 상황별 최적 선택

| 상황 | 추천 알고리즘 | 이유 |
|------|-------------|------|
| 작은 데이터 (n < 10) | 삽입 정렬 | 오버헤드 최소 |
| 거의 정렬된 데이터 | 삽입 정렬 | O(n) 성능 |
| 일반적인 경우 | 퀵 정렬 | 평균 최고 성능 |
| 최악 성능 보장 필요 | 힙/병합 정렬 | O(n log n) 보장 |
| 안정성 필요 | 병합 정렬 | 안정 정렬 |
| 메모리 제약 | 힙 정렬 | O(1) 공간 |
| 외부 정렬 | 병합 정렬 | 순차 접근 |
| 중복 많은 데이터 | 3-way 퀵 정렬 | 중복 효율적 처리 |

### 하이브리드 정렬

> [!tip] Timsort (Python의 기본 정렬)
> ```python
> def timsort_concept(arr):
>     MIN_MERGE = 32
>     
>     # 작은 부분은 삽입 정렬
>     for start in range(0, len(arr), MIN_MERGE):
>         end = min(start + MIN_MERGE, len(arr))
>         insertion_sort_range(arr, start, end)
>     
>     # 병합 정렬로 결합
>     size = MIN_MERGE
>     while size < len(arr):
>         for start in range(0, len(arr), size * 2):
>             merge_ranges(arr, start, size)
>         size *= 2
>     
>     return arr
> ```

### Introsort (C++ STL)

퀵 정렬 + 힙 정렬 + 삽입 정렬의 하이브리드
- 시작: 퀵 정렬
- 깊이 제한 도달: 힙 정렬로 전환
- 작은 부분: 삽입 정렬

---

## 💡 실전 가이드

### 언어별 기본 정렬

| 언어 | 함수/메서드 | 알고리즘 | 안정성 |
|------|------------|----------|--------|
| Python | `sorted()`, `.sort()` | Timsort | ✅ |
| Java | `Arrays.sort()` | Dual-Pivot Quicksort (primitive), Timsort (object) | 객체만 ✅ |
| C++ | `std::sort()` | Introsort | ❌ |
| JavaScript | `Array.sort()` | 구현 의존적 (보통 Timsort) | 구현 의존 |

### 정렬 알고리즘 선택 플로우차트

```
데이터 크기?
    │
    ├─ 작음 (< 50) → 삽입 정렬
    │
    └─ 큼
        │
        ├─ 거의 정렬? → 삽입 정렬
        │
        └─ 무작위
            │
            ├─ 안정성 필요? → 병합 정렬
            │
            └─ 안정성 불필요
                │
                ├─ 메모리 제약? → 힙 정렬
                │
                └─ 일반적 → 퀵 정렬
```

### 최적화 팁

> [!success] 성능 개선 체크리스트
> - [ ] 작은 부분 배열에 삽입 정렬 사용
> - [ ] 피벗 선택 전략 최적화
> - [ ] 이미 정렬된 부분 건너뛰기
> - [ ] 캐시 친화적 구현
> - [ ] 병렬화 가능한 부분 확인

---

## 📚 참고자료

### 추가 학습 자료
- [[01_algorithm-performance|알고리즘 성능 분석]]
- [[02_divide-and-conquer|분할 정복과 정렬]]
- [[01_data-structures-overview|자료구조와 정렬]]

### 연습 문제
1. K번째 작은 원소 찾기 (퀵 선택)
2. 두 정렬된 배열 병합
3. 정렬 알고리즘 시각화 구현
4. 특수 조건 정렬 (조건부 정렬)

### 고급 주제
- 외부 정렬 (External Sort)
- 병렬 정렬 알고리즘
- 기수 정렬 (Radix Sort)
- 계수 정렬 (Counting Sort)

---

> [!quote]
> "정렬은 컴퓨터 과학의 기초이며, 효율적인 정렬은 많은 문제 해결의 첫걸음이다."

---

## 🔬 병합 정렬 심화 분석

### 점화식 전개와 수학적 증명

> [!info] 상세한 시간복잡도 유도
> 병합 정렬의 점화식을 단계별로 전개하여 O(n log n)을 증명합니다.

점화식: `T(n) = 2T(n/2) + cn, T(1) = c`

**단계별 전개:**
```
T(n) = 2T(n/2) + cn
     = 2[2T(n/4) + c(n/2)] + cn
     = 4T(n/4) + cn + cn
     = 4T(n/4) + 2cn
     = 4[2T(n/8) + c(n/4)] + 2cn
     = 8T(n/8) + 3cn
     ...
     = 2^k T(n/2^k) + k·cn
```

k = log₂n일 때 T(1)에 도달:
```
T(n) = n·T(1) + cn·log₂n
     = cn + cn·log₂n
     = O(n log n)
```

### 재귀 트리 시각화

```
레벨 0:                cn                  = cn
         /                      \
레벨 1:  cn/2                    cn/2       = cn
      /     \                  /     \
레벨 2: cn/4  cn/4            cn/4  cn/4     = cn
...
레벨 log n: c  c  c  ...  c  c  c  c       = cn

총 시간: cn × log n = O(n log n)
```

### Merge Sort vs Quick Sort 실무 비교

| 특성 | Merge Sort | Quick Sort |
|------|-----------|------------|
| 시간복잡도(평균) | O(n log n) | O(n log n) |
| 시간복잡도(최악) | O(n log n) | O(n²) |
| 공간복잡도 | O(n) | O(log n) |
| 안정성 | 안정 정렬 | 불안정 정렬 |
| 캐시 효율 | 낮음 | 높음 |
| 예측 가능성 | 항상 동일 | 데이터 의존적 |

**실무 선택 기준:**
- **Merge Sort 선택**: 안정 정렬 필요, 최악의 경우도 보장되는 성능 필요, 외부 정렬
- **Quick Sort 선택**: 평균적으로 빠른 성능 필요, 메모리 제한적, 캐시 친화적 필요

---

## 🧮 삽입 정렬 수학적 분석

### 평균 시간복잡도가 절반이 되는 이유

> [!question] 왜 평균이 절반인가?
> 삽입 정렬에서 각 원소가 삽입될 위치는 균등하게 분포한다고 가정합니다.

#### 예시 분석
정렬된 배열 `[1, 3, 7, 9]`에 `5`를 삽입하는 경우:

가능한 삽입 위치와 비교 횟수:
```
위치 0: [5, 1, 3, 7, 9] → 4번 비교
위치 1: [1, 5, 3, 7, 9] → 3번 비교  
위치 2: [1, 3, 5, 7, 9] → 2번 비교
위치 3: [1, 3, 7, 5, 9] → 1번 비교
위치 4: [1, 3, 7, 9, 5] → 0번 비교
```

평균 비교 횟수 = (0+1+2+3+4)/5 = 2 = 최대(4)의 절반

#### 수식 유도
```
평균 비교 횟수 = Σ(i/2) for i=1 to n-1
               = (1/2) × Σi 
               = (1/2) × n(n-1)/2
               = n(n-1)/4
               = Θ(n²)
```

### 이진 삽입 정렬의 개선

> [!tip] 비교 횟수는 줄지만 이동은 그대로
> 이진 탐색으로 삽입 위치를 O(log n)에 찾지만, 원소 이동은 여전히 O(n)입니다.

---

## 📊 선택 정렬 vs 삽입 정렬 상세 비교

### 동작 특성 비교

| 항목 | 선택 정렬 | 삽입 정렬 |
|------|----------|----------|
| **비교 횟수** | 항상 n(n-1)/2 | 평균 n(n-1)/4, 최악 n(n-1)/2 |
| **교환 횟수** | 최대 n-1 | 최대 n(n-1)/2 |
| **정렬된 데이터** | 효과 없음 (여전히 O(n²)) | 매우 빠름 (O(n)) |
| **역정렬 데이터** | 영향 없음 (여전히 O(n²)) | 매우 느림 (O(n²)) |
| **부분 정렬** | 효과 없음 | 효율적 |
| **온라인 알고리즘** | 아니오 | 예 (실시간 정렬 가능) |

### 언제 어떤 것을 사용할까?

**선택 정렬 사용:**
- 교환 비용이 매우 큰 경우 (최소 교환 횟수)
- 모든 데이터를 확인해야 하는 경우

**삽입 정렬 사용:**
- 거의 정렬된 데이터
- 실시간으로 데이터가 추가되는 경우
- 작은 데이터셋 (n < 50)

---

## 💻 언어별 정렬 구현 모음

### 삽입 정렬 언어별 비교

**C++:**
```cpp
void insertionSort(std::vector<int>& arr) {
    for (int i = 1; i < arr.size(); ++i) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

**JavaScript:**
```javascript
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    let key = arr[i];
    let j = i - 1;
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = key;
  }
  return arr;
}
```

### 선택 정렬 언어별 비교

**JavaScript:**
```javascript
function selectionSort(arr) {
  const n = arr.length;
  for (let i = 0; i < n - 1; i++) {
    let minIndex = i;
    for (let j = i + 1; j < n; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }
    [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
  }
  return arr;
}
```

**C++:**
```cpp
void selectionSort(std::vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; ++i) {
        int minIndex = i;
        for (int j = i + 1; j < n; ++j) {
            if (arr[j] < arr[minIndex]) {
                minIndex = j;
            }
        }
        std::swap(arr[i], arr[minIndex]);
    }
}
```

---

## 🔬 병합 정렬 심화 분석

### 점화식 전개와 수학적 증명

> [!info] 상세한 시간복잡도 유도
> 병합 정렬의 점화식을 단계별로 전개하여 O(n log n)을 증명합니다.

점화식: `T(n) = 2T(n/2) + cn, T(1) = c`

**단계별 전개:**
```
T(n) = 2T(n/2) + cn
     = 2[2T(n/4) + c(n/2)] + cn
     = 4T(n/4) + cn + cn
     = 4T(n/4) + 2cn
     = 4[2T(n/8) + c(n/4)] + 2cn
     = 8T(n/8) + 3cn
     ...
     = 2^k T(n/2^k) + k·cn
```

k = log₂n일 때 T(1)에 도달:
```
T(n) = n·T(1) + cn·log₂n
     = cn + cn·log₂n
     = O(n log n)
```

### 재귀 트리 시각화

```
레벨 0:                cn                  = cn
         /                      \
레벨 1:  cn/2                    cn/2       = cn
      /     \                  /     \
레벨 2: cn/4  cn/4            cn/4  cn/4     = cn
...
레벨 log n: c  c  c  ...  c  c  c  c       = cn

총 시간: cn × log n = O(n log n)
```

### Merge Sort vs Quick Sort 실무 비교

| 특성 | Merge Sort | Quick Sort |
|------|-----------|------------|
| 시간복잡도(평균) | O(n log n) | O(n log n) |
| 시간복잡도(최악) | O(n log n) | O(n²) |
| 공간복잡도 | O(n) | O(log n) |
| 안정성 | 안정 정렬 | 불안정 정렬 |
| 캐시 효율 | 낮음 | 높음 |
| 예측 가능성 | 항상 동일 | 데이터 의존적 |

**실무 선택 기준:**
- **Merge Sort 선택**: 안정 정렬 필요, 최악의 경우도 보장되는 성능 필요, 외부 정렬
- **Quick Sort 선택**: 평균적으로 빠른 성능 필요, 메모리 제한적, 캐시 친화적 필요

---

## 🧮 삽입 정렬 수학적 분석

### 평균 시간복잡도가 절반이 되는 이유

> [!question] 왜 평균이 절반인가?
> 삽입 정렬에서 각 원소가 삽입될 위치는 균등하게 분포한다고 가정합니다.

#### 예시 분석
정렬된 배열 `[1, 3, 7, 9]`에 `5`를 삽입하는 경우:

가능한 삽입 위치와 비교 횟수:
```
위치 0: [5, 1, 3, 7, 9] → 4번 비교
위치 1: [1, 5, 3, 7, 9] → 3번 비교  
위치 2: [1, 3, 5, 7, 9] → 2번 비교
위치 3: [1, 3, 7, 5, 9] → 1번 비교
위치 4: [1, 3, 7, 9, 5] → 0번 비교
```

평균 비교 횟수 = (0+1+2+3+4)/5 = 2 = 최대(4)의 절반

#### 수식 유도
```
평균 비교 횟수 = Σ(i/2) for i=1 to n-1
               = (1/2) × Σi 
               = (1/2) × n(n-1)/2
               = n(n-1)/4
               = Θ(n²)
```

### 이진 삽입 정렬의 개선

> [!tip] 비교 횟수는 줄지만 이동은 그대로
> 이진 탐색으로 삽입 위치를 O(log n)에 찾지만, 원소 이동은 여전히 O(n)입니다.

---

## 📊 선택 정렬 vs 삽입 정렬 상세 비교

### 동작 특성 비교

| 항목 | 선택 정렬 | 삽입 정렬 |
|------|----------|----------|
| **비교 횟수** | 항상 n(n-1)/2 | 평균 n(n-1)/4, 최악 n(n-1)/2 |
| **교환 횟수** | 최대 n-1 | 최대 n(n-1)/2 |
| **정렬된 데이터** | 효과 없음 (여전히 O(n²)) | 매우 빠름 (O(n)) |
| **역정렬 데이터** | 영향 없음 (여전히 O(n²)) | 매우 느림 (O(n²)) |
| **부분 정렬** | 효과 없음 | 효율적 |
| **온라인 알고리즘** | 아니오 | 예 (실시간 정렬 가능) |

### 언제 어떤 것을 사용할까?

**선택 정렬 사용:**
- 교환 비용이 매우 큰 경우 (최소 교환 횟수)
- 모든 데이터를 확인해야 하는 경우

**삽입 정렬 사용:**
- 거의 정렬된 데이터
- 실시간으로 데이터가 추가되는 경우
- 작은 데이터셋 (n < 50)

---

## 💻 언어별 정렬 구현 모음

### 삽입 정렬 언어별 비교

**C++:**
```cpp
void insertionSort(std::vector<int>& arr) {
    for (int i = 1; i < arr.size(); ++i) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

**JavaScript:**
```javascript
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    let key = arr[i];
    let j = i - 1;
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = key;
  }
  return arr;
}
```

### 선택 정렬 언어별 비교

**JavaScript:**
```javascript
function selectionSort(arr) {
  const n = arr.length;
  for (let i = 0; i < n - 1; i++) {
    let minIndex = i;
    for (let j = i + 1; j < n; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }
    [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
  }
  return arr;
}
```

**C++:**
```cpp
void selectionSort(std::vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; ++i) {
        int minIndex = i;
        for (int j = i + 1; j < n; ++j) {
            if (arr[j] < arr[minIndex]) {
                minIndex = j;
            }
        }
        std::swap(arr[i], arr[minIndex]);
    }
}
```