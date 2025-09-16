---
tags:
  - algorithm
  - performance
  - big-o
  - complexity
  - cs-fundamentals
created: 2025-01-04
updated: 2025-09-05
aliases:
  - 알고리즘 성능
  - Big O 표기법
  - 시간 복잡도
description: 알고리즘의 성능 분석과 Big O 표기법에 대한 기초 개념
status: published
category: tutorial
---

# 알고리즘 성능과 Big O 표기법

> [!info] 개요
> 알고리즘의 성능을 분석하고 평가하는 방법을 학습합니다. 시간 복잡도와 공간 복잡도의 개념을 이해하고, Big O 표기법을 통해 알고리즘의 효율성을 표현하는 방법을 익힙니다.

## 📑 목차

- [[#⚡ 핵심 개념]]
- [[#🎯 알고리즘 성능 평가]]
- [[#📊 시간 복잡도]]
- [[#💾 공간 복잡도]]
- [[#📈 Big O 표기법]]
- [[#🔍 복잡도 분석 예제]]
- [[#💡 실전 팁]]
- [[#📚 참고자료]]

---

## ⚡ 핵심 개념

> [!note] 알고리즘 성능이란?
> 알고리즘의 성능은 주어진 문제를 해결하는 데 필요한 **시간**과 **공간(메모리)**으로 측정됩니다.

### 왜 알고리즘 성능이 중요한가?

1. **확장성(Scalability)**: 데이터가 커질 때 프로그램이 여전히 효율적으로 동작하는가?
2. **자원 최적화**: 한정된 컴퓨팅 자원을 효율적으로 사용하는가?
3. **사용자 경험**: 빠른 응답 시간으로 더 나은 사용자 경험 제공

---

## 🎯 알고리즘 성능 평가

### 성능 측정 기준

> [!tip] 두 가지 주요 지표
> - **시간 복잡도(Time Complexity)**: 얼마나 빨리 실행되는가?
> - **공간 복잡도(Space Complexity)**: 얼마나 많은 메모리를 사용하는가?

### 성능 분석 방법

1. **실험적 분석(Experimental Analysis)**
   - 실제 구현 후 실행 시간 측정
   - 하드웨어와 환경에 의존적

2. **이론적 분석(Theoretical Analysis)**
   - 수학적 모델을 통한 분석
   - 하드웨어 독립적
   - Big O 표기법 사용

---

## 📊 시간 복잡도

> [!info] 정의
> 시간 복잡도는 입력 크기(n)에 따른 알고리즘 실행 시간의 증가율을 나타냅니다.

### 기본 연산 계산

```typescript
// 예제 1: 상수 시간 - O(1)
function getFirstElement<T>(arr: T[]): T {
   return arr[0];  // 1번 실행
}

// 예제 2: 선형 시간 - O(n)
function sumArray(arr: number[]): number {
   let total = 0;      // 1번
   for (const num of arr) {  // n번
       total += num;  // n번
   }
   return total;   // 1번
   // 총: 2n + 2 → O(n)
}

// 예제 3: 이차 시간 - O(n²)
function bubbleSort(arr: number[]): void {
   const n = arr.length;
   for (let i = 0; i < n; i++) {      // n번
       for (let j = 0; j < n - 1; j++) {  // n-1번
           if (arr[j] > arr[j + 1]) {
               // TypeScript에서 스왑
               [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
           }
       }
   }
   // 총: n * (n-1) → O(n²)
}
```

---

## 💾 공간 복잡도

> [!note] 정의
> 공간 복잡도는 알고리즘이 실행되는 동안 사용하는 메모리의 양을 나타냅니다.

### 공간 복잡도 구성 요소

1. **고정 공간**: 입력 크기와 무관한 공간
   - 코드 저장 공간
   - 단순 변수
   - 상수

2. **가변 공간**: 입력 크기에 따라 변하는 공간
   - 동적 할당 메모리
   - 재귀 호출 스택
   - 임시 변수

> [!example] 공간 복잡도 예제
> ```typescript
> // O(1) 공간 복잡도
> function findMax(arr: number[]): number {
> 	let maxVal = arr[0];
> 	for (const num of arr){
> 		if(num > maxVal) {
> 			maxVal = num;
> 		}
> 	}
> 	return maxVal;
> }
> 
> // O(n) 공간 복잡도
> function createCopy(arr: number[]): number[] {
> 	const copy: number[] = [];
> 	for (const num of arr) {
> 		copy.push(num);
> 	}
> 	return copy;
> }
> ```

---

## 📈 Big O 표기법

> [!info] Big O란?
> Big O 표기법은 알고리즘의 최악의 경우(Worst Case) 성능을 나타내는 수학적 표기법입니다.

### 주요 Big O 복잡도

| Big O | 명칭 | 예제 | 성능 |
|-------|------|------|------|
| O(1) | 상수 시간 | 배열 인덱스 접근 | 최고 ⚡ |
| O(log n) | 로그 시간 | 이진 탐색 | 우수 |
| O(n) | 선형 시간 | 단순 탐색 | 보통 |
| O(n log n) | 선형로그 시간 | 효율적인 정렬(머지, 퀵) | 양호 |
| O(n²) | 이차 시간 | 중첩 반복문 | 나쁨 |
| O(2ⁿ) | 지수 시간 | 재귀 피보나치 | 매우 나쁨 |
| O(n!) | 팩토리얼 시간 | 외판원 문제 | 최악 ❌ |

### Big O 그래프

```
성능 ↑
    |     O(1) ____________
    |     
    |     O(log n) _____/
    |     
    |     O(n) ______/
    |    
    |     O(n log n) ___/
    |    
    |     O(n²) ___/
    |    
    |     O(2ⁿ) _/
    |____________→ 입력 크기(n)
```

### Big O 계산 규칙

> [!tip] 단순화 규칙
> 1. **상수 제거**: O(2n) → O(n)
> 2. **낮은 차수 제거**: O(n² + n) → O(n²)
> 3. **상수 곱셈 제거**: O(3n²) → O(n²)
> 4. **지배적 항만 유지**: O(n³ + n² + n) → O(n³)

---

## 🔍 복잡도 분석 예제

### 예제 1: 배열의 최댓값 찾기

```typescript
function findMax(arr: number[]): number | null {
	if(!arr || arr.length === 0) {
		return null;
	}
	
	let maxVal = arr[0]; // O(1)
	
	for(let i = 1; i < arr.length; i++) { // O(n)
		if(arr[i] > maxVal) { // O(1)
			maxVal = arr[i]; // O(1)
		}
	}
	
	return maxVal; // O(1)
}

// 시간 복잡도: O(n)
// 공간 복잡도: O(1)
```

### 예제 2: 이진 탐색

```typescript
function binarySearch(arr: number[], target: number): number {
	let left = 0;
	let right = arr.length - 1; // O(1)
	
	while (left <= right) { // O(log n)
		const mid = Math.floor((left + right) / 2); // O(1)
		
		if(arr[mid] === target) {
			return mid;
		} else if (arr[mid] < target) {
			left = mid + 1;
		} else {
			right = mid - 1;
		}
	}
	
	return -1;
}

// 시간 복잡도 O(log n)
// 공간 복잡도 O(1)
```

### 예제 3: 피보나치 수열 (재귀 vs 동적 프로그래밍)

```typescript
// 비효율적인 재귀 - O(2ⁿ)
function fibRecursive(n: number): number {
	if (n <= 1) {
		return n;
	}
	return fibRecursive(n - 1) + fibRecursive(n - 2);
}

// 효율적인 동적 프로그래밍 - O(n)
function fibDP(n: number): number {
	if (n <= 1) {
		return n;
	}
	
	const dp: number[] = new Array(n + 1).fill(0);
	dp[0] = 0;
	dp[1] = 1;
	
	for (let i = 2; i <= n; i++) {
		dp[i] = dp[i - 1] + dp[i - 2];
	}
	
	return dp[n];
}

```

> [!warning] 재귀의 함정
> 단순 재귀는 중복 계산으로 인해 지수 시간 복잡도를 가질 수 있습니다.

---

## 💡 실전 팁

> [!success] 성능 최적화 체크리스트
> - [ ] 불필요한 중첩 반복문 제거
> - [ ] 적절한 자료구조 선택
> - [ ] 캐싱/메모이제이션 활용
> - [ ] 조기 종료 조건 추가
> - [ ] 분할 정복 기법 적용

### 자료구조별 시간 복잡도

| 자료구조 | 접근 | 검색 | 삽입 | 삭제 |
|---------|------|------|------|------|
| 배열 | O(1) | O(n) | O(n) | O(n) |
| 연결 리스트 | O(n) | O(n) | O(1) | O(1) |
| 스택/큐 | O(n) | O(n) | O(1) | O(1) |
| 해시 테이블 | - | O(1)* | O(1)* | O(1)* |
| 이진 탐색 트리 | O(log n)* | O(log n)* | O(log n)* | O(log n)* |

*평균적인 경우

### 언제 어떤 복잡도가 허용되는가?

| 입력 크기(n) | 허용 가능한 복잡도 | 예상 연산 횟수 |
|-------------|------------------|---------------|
| n ≤ 10 | O(n!) | ~3,628,800 |
| n ≤ 20 | O(2ⁿ) | ~1,048,576 |
| n ≤ 100 | O(n³) | ~1,000,000 |
| n ≤ 1,000 | O(n²) | ~1,000,000 |
| n ≤ 100,000 | O(n log n) | ~1,660,964 |
| n ≤ 1,000,000 | O(n) | ~1,000,000 |
| n ≤ 10⁹ | O(log n) | ~30 |

---

## 📚 참고자료

### 추가 학습 자료
- [[02_divide-and-conquer|분할 정복 알고리즘]]
- [[03_sorting-algorithms|정렬 알고리즘 비교]]
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)
- [Visualizing Algorithms](https://visualgo.net/)

### 연습 문제
1. 주어진 코드의 시간 복잡도 분석하기
2. O(n²) 알고리즘을 O(n log n)으로 개선하기
3. 공간과 시간의 트레이드오프 상황 찾기

---

> [!quote]
> "Premature optimization is the root of all evil, but that doesn't mean we should write inefficient code." - Donald Knuth (수정됨)

---

## 🔄 재귀 알고리즘의 복잡도 분석

> [!info] 재귀 복잡도의 특징
> 재귀 알고리즘의 복잡도는 **재귀 깊이**와 **각 레벨의 작업량**을 곱하여 계산합니다. 재귀 관계식(Recurrence Relation)을 통해 정확한 분석이 가능합니다.

### 재귀 복잡도 계산 방법

#### 1. 재귀 관계식 수립

```python
# T(n) = 재귀 호출 수 × T(부분 문제 크기) + 현재 레벨 작업량

# 예: 팩토리얼
def factorial(n):
    if n <= 1:  # O(1)
        return 1
    return n * factorial(n-1)  # T(n-1) + O(1)

# T(n) = T(n-1) + O(1) = O(n)
```

#### 2. 마스터 정리 (Master Theorem)

> [!note] 마스터 정리 공식
> T(n) = a·T(n/b) + f(n) 형태의 재귀식에서:
> - a: 재귀 호출 횟수
> - b: 부분 문제 크기 비율
> - f(n): 병합/분할 비용
>
> 1. f(n) = O(n^c), c < log_b(a) → T(n) = O(n^log_b(a))
> 2. f(n) = O(n^c), c = log_b(a) → T(n) = O(n^c log n)
> 3. f(n) = O(n^c), c > log_b(a) → T(n) = O(f(n))

### 재귀 패턴별 복잡도

#### 선형 재귀 (Linear Recursion)

```python
# 한 번의 재귀 호출
def linear_sum(arr, n):
    if n == 0:
        return 0
    return arr[n-1] + linear_sum(arr, n-1)

# T(n) = T(n-1) + O(1) = O(n)
# 공간: O(n) - 재귀 스택
```

#### 이진 재귀 (Binary Recursion)

```python
# 두 번의 재귀 호출 - 비효율적
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# T(n) = T(n-1) + T(n-2) + O(1) ≈ O(2^n)
# 정확히는 O(φ^n) where φ = (1+√5)/2 ≈ 1.618
```

#### 분할 정복 재귀

```python
# 병합 정렬 - 효율적인 분할
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])   # T(n/2)
    right = merge_sort(arr[mid:])  # T(n/2)
    return merge(left, right)       # O(n)

# T(n) = 2·T(n/2) + O(n) = O(n log n)
```

### 재귀 → 꼬리재귀 최적화

> [!tip] 꼬리재귀 최적화
> 꼬리재귀는 컴파일러 최적화를 통해 O(1) 공간 복잡도를 달성할 수 있습니다.

```python
# 일반 재귀 - O(n) 공간
def factorial_recursive(n):
    if n <= 1:
        return 1
    return n * factorial_recursive(n-1)

# 꼬리재귀 - O(1) 공간 (최적화 시)
def factorial_tail(n, accumulator=1):
    if n <= 1:
        return accumulator
    return factorial_tail(n-1, n * accumulator)
```

### 재귀 트리 분석

```
피보나치 F(5) 호출 트리:
                    F(5)
                   /    \
              F(4)        F(3)
             /    \      /    \
        F(3)      F(2)  F(2)  F(1)
       /    \     /  \  /  \
    F(2)   F(1) F(1) F(0) ...

총 호출 횟수: ~O(2^n)
중복 계산: F(3) 2번, F(2) 3번
```

### 메모이제이션을 통한 최적화

```python
# 메모이제이션으로 O(n) 시간, O(n) 공간
def fib_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo)
    return memo[n]

# 각 부분 문제를 한 번만 계산
```

### 재귀 복잡도 요약표

| 재귀 유형 | 예제 | 시간 복잡도 | 공간 복잡도 |
|---------|------|------------|------------|
| 선형 재귀 | 팩토리얼, 선형 탐색 | O(n) | O(n) |
| 꼬리 재귀 | 최적화된 팩토리얼 | O(n) | O(1)* |
| 이진 재귀 | 피보나치(순진) | O(2^n) | O(n) |
| 분할 정복 | 병합 정렬 | O(n log n) | O(log n) |
| 트리 재귀 | 하노이 탑 | O(2^n) | O(n) |
| 동적 프로그래밍 | 피보나치(메모) | O(n) | O(n) |

*컴파일러 최적화 적용 시

### 재귀 최적화 전략

> [!success] 최적화 체크리스트
> - [ ] 꼬리재귀로 변환 가능한가?
> - [ ] 메모이제이션이 적용 가능한가?
> - [ ] 반복문으로 변환이 더 효율적인가?
> - [ ] 부분 문제 중복이 있는가?
> - [ ] 스택 오버플로우 위험이 있는가?

```python
# 재귀 깊이 제한 확인 및 설정
import sys
print(sys.getrecursionlimit())  # 기본: 1000
sys.setrecursionlimit(10000)    # 증가 (주의 필요)
```

> [!warning] 재귀 사용 시 주의사항
> - Python은 꼬리재귀 최적화를 지원하지 않음
> - 깊은 재귀는 스택 오버플로우 위험
> - 중복 계산이 있다면 반드시 메모이제이션 고려

### 관련 문서
- [[02_recursion-and-tail-recursion|재귀와 꼬리재귀]]
- [[02_divide-and-conquer|분할 정복 알고리즘]]

---

## 📊 Elementary Operations와 시간 복잡도 계산

> [!note] Elementary Operations (기본 연산)
> 시간 복잡도를 계산할 때 사용되는 기본 연산 단위입니다:
> 1. **대입 연산**: 변수에 값을 저장
> 2. **사칙 연산**: 덧셈, 뺄셈, 곱셈, 나눗셈
> 3. **비교 구문**: if, while 등의 조건 비교
> 4. **함수 호출**: 함수 실행

### Statement Analysis Table (문장별 분석)

```python
def sum_array(arr):
    total = 0      # 1번 실행 (대입)
    for num in arr:  # n+1번 실행 (비교)
        total += num  # n번 실행 (연산+대입)
    return total   # 1번 실행
    # 총 시간 복잡도: 2n + 3 → O(n)
```

> [!example] Statement Step 분석표
> | Statement | S/E | Frequency | Total Steps |
> |-----------|-----|-----------|-------------|
> | `total = 0` | 1 | 1 | 1 |
> | `for num in arr` | 1 | n+1 | n+1 |
> | `total += num` | 1 | n | n |
> | `return total` | 1 | 1 | 1 |
> | **Total** | | | **2n+3** |

### n이 충분히 클 때의 성능

> [!tip] 왜 큰 입력값이 중요한가?
> - 작은 입력: 모든 알고리즘이 빠름 (차이 미미)
> - 큰 입력: 알고리즘 간 성능 차이가 극명하게 드러남
> - 예: n=10 → O(n²)과 O(n log n) 차이 작음
> - 예: n=1,000,000 → O(n²)은 1조번, O(n log n)은 약 2천만번 (50,000배 차이!)

---

## 📈 Asymptotic Notations (점근 표기법)

> [!important] 세 가지 점근 표기법
> - **Big O (O)**: 상한선(Upper Bound) - 최악의 경우
> - **Omega (Ω)**: 하한선(Lower Bound) - 최선의 경우  
> - **Theta (Θ)**: 상한선과 하한선을 동시에 규정(Tight Bound) - 평균의 경우

### 표기법 비교

```
f(n) = O(g(n))  → f(n) ≤ c·g(n) (상한선)
f(n) = Ω(g(n))  → f(n) ≥ c·g(n) (하한선)
f(n) = Θ(g(n))  → c₁·g(n) ≤ f(n) ≤ c₂·g(n) (상한선과 하한선)
```

> [!example] 예시: 선형 탐색
> - **Best Case (Ω)**: Ω(1) - 첫 번째 원소가 타겟
> - **Average Case (Θ)**: Θ(n/2) → Θ(n) - 중간쯤에서 발견
> - **Worst Case (O)**: O(n) - 마지막 원소가 타겟 또는 없음

---

## 🔀 정렬 알고리즘 복잡도 비교

> [!info] 정렬 알고리즘 성능 비교표
> 다양한 정렬 알고리즘의 시간 복잡도와 공간 복잡도를 비교합니다.

### 주요 정렬 알고리즘 복잡도

| 알고리즘 | Best Case | Average Case | Worst Case | 공간 복잡도 | 안정성 |
|---------|-----------|--------------|------------|------------|--------|
| **Bubble Sort** | O(n) | O(n²) | O(n²) | O(1) | 안정 |
| **Selection Sort** | O(n²) | O(n²) | O(n²) | O(1) | 불안정 |
| **Insertion Sort** | O(n) | O(n²) | O(n²) | O(1) | 안정 |
| **Merge Sort** | O(n log n) | O(n log n) | O(n log n) | O(n) | 안정 |
| **Quick Sort** | O(n log n) | O(n log n) | O(n²) | O(log n) | 불안정 |
| **Heap Sort** | O(n log n) | O(n log n) | O(n log n) | O(1) | 불안정 |
| **Counting Sort** | O(n+k) | O(n+k) | O(n+k) | O(k) | 안정 |
| **Radix Sort** | O(nk) | O(nk) | O(nk) | O(n+k) | 안정 |
| **Bucket Sort** | O(n+k) | O(n+k) | O(n²) | O(n) | 안정 |

> [!tip] 정렬 알고리즘 선택 가이드
> - **작은 데이터(n < 50)**: Insertion Sort (구현 간단, 작은 n에서 빠름)
> - **일반적인 경우**: Quick Sort (평균적으로 가장 빠름)
> - **최악의 경우도 보장**: Merge Sort, Heap Sort (항상 O(n log n))
> - **정수 데이터**: Counting Sort, Radix Sort (선형 시간)
> - **거의 정렬된 데이터**: Insertion Sort, Bubble Sort (O(n)에 가까움)
> - **메모리 제약**: Heap Sort, Quick Sort (in-place 정렬)

### 정렬 알고리즘 특성

> [!note] 안정성(Stability)이란?
> 동일한 값을 가진 원소들의 상대적 순서가 정렬 후에도 유지되는 성질
> - 안정 정렬: Merge Sort, Bubble Sort, Insertion Sort
> - 불안정 정렬: Quick Sort, Heap Sort, Selection Sort

---

## 🔢 재귀 관계식과 점화식

### 재귀 작성의 2가지 핵심 요소

1. **초기 조건(Base Case)**: 재귀를 멈추는 조건
2. **반복 조건(Recursive Case)**: 문제를 더 작은 부분으로 나누는 방법

> [!info] 점화식과 재귀 관계
> 재귀 알고리즘의 시간 복잡도는 점화식(Recurrence Relation)으로 표현됩니다.
> 
> **일반 형태**: T(n) = a·T(n/b) + f(n)
> - a: 재귀 호출 횟수
> - n/b: 부분 문제의 크기
> - f(n): 분할/병합에 드는 비용

### 재귀 패턴별 복잡도 예제

> [!example] 팩토리얼 재귀 분석
> ```python
> def factorial(n):
>     if n <= 1:        # 초기 조건
>         return 1
>     return n * factorial(n-1)  # 반복 조건
> 
> # T(n) = T(n-1) + O(1)
> # = T(n-2) + O(1) + O(1)
> # = T(n-3) + O(1) + O(1) + O(1)
> # ...
> # = T(1) + (n-1)·O(1)
> # = O(n)
> ```

### 피보나치 수열 복잡도 분석

> [!warning] 단순 재귀의 비효율성
> ```python
> def fibonacci(n):
>     if n <= 1:
>         return n
>     return fibonacci(n-1) + fibonacci(n-2)
> 
> # T(n) = T(n-1) + T(n-2) + O(1)
> # 각 레벨에서 호출 수가 2배씩 증가
> # → O(2ⁿ) 시간 복잡도
> ```

**재귀 트리 분석**:
```
        F(5)
       /    \
     F(4)    F(3)     → 2개 호출
    /   \    /   \
  F(3) F(2) F(2) F(1) → 4개 호출
  ...                 → 8개 호출
```

> [!tip] 점화식 해결 방법
> - **반복 조건 내에서 실행되는 함수의 시간 복잡도**가 재귀 함수 성능에 가장 큰 영향
> - 반복되는 횟수 × 반복조건 내 함수 시간복잡도
> - 수학에서 점화식을 생각하면 이해가 쉽습니다