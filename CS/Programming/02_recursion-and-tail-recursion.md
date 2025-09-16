---
tags:
  - 재귀
  - 꼬리재귀
  - 알고리즘
  - 프로그래밍
  - SICP
created: 2025-01-04
updated: "updated: 2025-09-05"
aliases:
  - Recursion
  - Tail Recursion
  - 재귀함수
description: 재귀와 꼬리재귀의 개념, 원리, 최적화 기법을 학습합니다
category: guide
---

# 재귀와 꼬리재귀 (Recursion and Tail Recursion)

> [!info] 개요
> 재귀는 함수가 자기 자신을 호출하는 프로그래밍 기법으로, 복잡한 문제를 단순한 부분 문제로 분해하여 해결하는 강력한 도구입니다. 꼬리재귀는 재귀의 특별한 형태로, 메모리 효율성을 크게 개선할 수 있는 최적화 기법입니다.

## 📑 목차

- [[#🎯 재귀란 무엇인가?]]
- [[#🔄 재귀의 기본 원리]]
- [[#💡 선형 재귀 vs 트리 재귀]]
- [[#⚡ 꼬리재귀 최적화]]
- [[#📊 재귀 프로세스 분석]]
- [[#🔧 실전 예제]]
- [[#⚠️ 주의사항과 최적화]]
- [[#📚 참고자료]]

---

## 🎯 재귀란 무엇인가?

> [!note] 핵심 정의
> **재귀(Recursion)**: 문제를 해결하기 위해 함수가 자기 자신을 호출하는 프로그래밍 기법. 큰 문제를 작은 부분 문제로 나누어 해결하는 분할 정복의 핵심 도구입니다.

### 재귀의 두 가지 필수 요소

1. **기저 사례(Base Case)**: 재귀 호출을 멈추는 조건
2. **재귀 단계(Recursive Step)**: 문제를 더 작은 부분으로 분해

```scheme
; 재귀의 기본 구조
(define (recursive-function x)
  (if (base-case? x)          ; 기저 사례 검사
      base-value               ; 기저 값 반환
      (combine x               ; 현재 값과
        (recursive-function    ; 재귀 호출 결과를
          (simplify x)))))     ; 단순화된 문제에 대해 결합
```

### 일상 속 재귀의 예

> [!example] 러시아 인형 (마트료시카)
> 러시아 인형을 여는 과정은 완벽한 재귀입니다:
> 1. 인형을 연다
> 2. 안에 인형이 있으면, 그 인형에 대해 1번 과정 반복
> 3. 더 이상 안에 인형이 없으면 종료

---

## 🔄 재귀의 기본 원리

### 선형 재귀적 프로세스 (Linear Recursive Process)

> [!info] 특징
> - 지연된 연산들이 체인처럼 쌓임
> - 메모리 사용량이 입력 크기에 비례하여 증가
> - 계산 과정이 확장 후 수축하는 형태

```scheme
; 팩토리얼 - 선형 재귀적 프로세스
(define (factorial n)
  (if (= n 1)
      1
      (* n (factorial (- n 1)))))

; 실행 과정 (n=5)
(factorial 5)
(* 5 (factorial 4))
(* 5 (* 4 (factorial 3)))
(* 5 (* 4 (* 3 (factorial 2))))
(* 5 (* 4 (* 3 (* 2 (factorial 1)))))
(* 5 (* 4 (* 3 (* 2 1))))
(* 5 (* 4 (* 3 2)))
(* 5 (* 4 6))
(* 5 24)
120
```

### 선형 반복적 프로세스 (Linear Iterative Process)

> [!tip] 특징
> - 상태 변수를 통해 현재 상태를 완전히 표현
> - 일정한 메모리 사용량 (O(1) 공간 복잡도)
> - 계산이 일정한 패턴으로 진행

```scheme
; 팩토리얼 - 선형 반복적 프로세스 (꼬리재귀)
(define (factorial n)
  (fact-iter 1 1 n))

(define (fact-iter product counter max-count)
  (if (> counter max-count)
      product
      (fact-iter (* counter product)
                 (+ counter 1)
                 max-count)))

; 실행 과정 (n=5)
(factorial 5)
(fact-iter 1 1 5)
(fact-iter 1 2 5)
(fact-iter 2 3 5)
(fact-iter 6 4 5)
(fact-iter 24 5 5)
(fact-iter 120 6 5)
120
```

---

## 💡 선형 재귀 vs 트리 재귀

### 트리 재귀 (Tree Recursion)

> [!warning] 트리 재귀의 특성
> 함수가 자신을 두 번 이상 호출하는 재귀 패턴으로, 계산 트리가 지수적으로 성장합니다.

```scheme
; 피보나치 - 트리 재귀 (비효율적)
(define (fib n)
  (cond ((= n 0) 0)
        ((= n 1) 1)
        (else (+ (fib (- n 1))
                 (fib (- n 2))))))

; 계산 트리 (n=5)
;                    fib(5)
;                   /      \
;              fib(4)        fib(3)
;             /     \        /     \
;        fib(3)   fib(2)  fib(2)  fib(1)
;        /   \     /   \    /   \
;    fib(2) fib(1) ...  ... ...  ...
```

### 트리 재귀를 선형 재귀로 변환

```scheme
; 피보나치 - 선형 반복적 프로세스
(define (fib n)
  (fib-iter 1 0 n))

(define (fib-iter a b count)
  (if (= count 0)
      b
      (fib-iter (+ a b) a (- count 1))))

; 실행 과정 (n=5)
(fib 5)
(fib-iter 1 0 5)
(fib-iter 1 1 4)
(fib-iter 2 1 3)
(fib-iter 3 2 2)
(fib-iter 5 3 1)
(fib-iter 8 5 0)
5
```

---

## ⚡ 꼬리재귀 최적화

> [!success] 꼬리재귀란?
> **꼬리재귀(Tail Recursion)**: 재귀 호출이 함수의 마지막 연산인 재귀. 컴파일러가 이를 반복문으로 최적화할 수 있어 스택 오버플로우를 방지합니다.

### 꼬리재귀 조건

1. 재귀 호출이 함수의 **마지막 표현식**이어야 함
2. 재귀 호출 결과에 **추가 연산이 없어야 함**
3. 모든 상태가 **매개변수로 전달**되어야 함

### 일반 재귀 vs 꼬리재귀

```scheme
; 일반 재귀 - 꼬리재귀 아님 (재귀 후 곱셈 연산)
(define (factorial n)
  (if (= n 1)
      1
      (* n (factorial (- n 1)))))  ; 재귀 후 곱셈 필요

; 꼬리재귀 - 최적화 가능
(define (factorial n)
  (define (iter product counter)
    (if (> counter n)
        product
        (iter (* counter product)   ; 모든 계산이 매개변수로
              (+ counter 1))))       ; 재귀가 마지막 연산
  (iter 1 1))
```

### 꼬리재귀 최적화의 효과

> [!tip] 성능 비교
> | 구분 | 일반 재귀 | 꼬리재귀 |
> |------|-----------|----------|
> | 공간 복잡도 | O(n) | O(1) |
> | 스택 사용 | n개 프레임 | 1개 프레임 |
> | 최대 깊이 | 제한적 | 무제한 |

---

## 📊 재귀 프로세스 분석

### 복잡도 분석

> [!abstract] 시간 및 공간 복잡도
> 재귀 알고리즘의 효율성은 호출 횟수와 각 호출의 작업량으로 결정됩니다.

| 알고리즘 | 시간 복잡도 | 공간 복잡도 | 재귀 타입 |
|---------|------------|------------|-----------|
| 팩토리얼 (일반) | O(n) | O(n) | 선형 재귀 |
| 팩토리얼 (꼬리) | O(n) | O(1) | 꼬리 재귀 |
| 피보나치 (트리) | O(2^n) | O(n) | 트리 재귀 |
| 피보나치 (선형) | O(n) | O(1) | 꼬리 재귀 |
| 이진 탐색 | O(log n) | O(log n) | 분할 재귀 |

---

## 🔧 실전 예제

### 예제 1: 파스칼의 삼각형

> [!example] 파스칼 삼각형 계산

```scheme
; 파스칼 삼각형의 (row, col) 위치 값 계산
(define (pascal row col)
  (cond ((= col 0) 1)           ; 첫 번째 열은 항상 1
        ((= col row) 1)         ; 마지막 열은 항상 1
        (else (+ (pascal (- row 1) (- col 1))
                 (pascal (- row 1) col)))))

; 파스칼 삼각형 출력
;         1
;       1   1
;     1   2   1
;   1   3   3   1
; 1   4   6   4   1
```

### 예제 2: 동전 나누기 문제

> [!example] 재귀를 이용한 경우의 수 계산

```scheme
; amount원을 n종류의 동전으로 나누는 경우의 수
(define (count-change amount)
  (cc amount 5))

(define (cc amount kinds-of-coins)
  (cond ((= amount 0) 1)                    ; 정확히 나누어떨어짐
        ((or (< amount 0)                   ; 음수가 됨
             (= kinds-of-coins 0)) 0)       ; 동전 종류 소진
        (else (+ (cc amount
                    (- kinds-of-coins 1))    ; 현재 동전 사용 안함
                 (cc (- amount
                       (first-denomination 
                         kinds-of-coins))    ; 현재 동전 사용
                     kinds-of-coins)))))

(define (first-denomination kinds-of-coins)
  (cond ((= kinds-of-coins 1) 1)     ; 1센트
        ((= kinds-of-coins 2) 5)     ; 5센트
        ((= kinds-of-coins 3) 10)    ; 10센트
        ((= kinds-of-coins 4) 25)    ; 25센트
        ((= kinds-of-coins 5) 50)))  ; 50센트
```

### 예제 3: 거듭제곱 계산

> [!example] 효율적인 거듭제곱 알고리즘

```scheme
; O(n) 선형 재귀
(define (expt b n)
  (if (= n 0)
      1
      (* b (expt b (- n 1)))))

; O(log n) 빠른 거듭제곱
(define (fast-expt b n)
  (cond ((= n 0) 1)
        ((even? n) (square (fast-expt b (/ n 2))))
        (else (* b (fast-expt b (- n 1))))))

(define (square x) (* x x))
(define (even? n) (= (remainder n 2) 0))
```

---

## ⚠️ 주의사항과 최적화

### 재귀 사용 시 주의점

> [!warning] 일반적인 함정들
> 1. **스택 오버플로우**: 깊은 재귀는 스택 한계 초과 가능
> 2. **중복 계산**: 트리 재귀에서 동일한 계산 반복
> 3. **기저 사례 누락**: 무한 재귀 발생
> 4. **비효율적 분할**: 문제 크기가 충분히 감소하지 않음

### 최적화 기법

#### 1. 메모이제이션 (Memoization)

```scheme
; 메모이제이션을 이용한 피보나치
(define memo-fib
  (let ((memo '()))
    (lambda (n)
      (let ((cached (assoc n memo)))
        (if cached
            (cdr cached)
            (let ((result
                   (cond ((= n 0) 0)
                         ((= n 1) 1)
                         (else (+ (memo-fib (- n 1))
                                 (memo-fib (- n 2)))))))
              (set! memo (cons (cons n result) memo))
              result))))))
```

#### 2. 꼬리재귀 변환

> [!tip] 변환 전략
> 1. 누적 매개변수(accumulator) 추가
> 2. 중간 결과를 매개변수로 전달
> 3. 재귀 호출을 마지막 연산으로 이동

#### 3. 반복문으로 변환

```scheme
; 재귀를 반복문으로 (의사 코드)
; while (not base-case):
;   update state
; return result
```

---

## 📚 참고자료

### 관련 문서
- [[01_functional-programming-basics|함수형 프로그래밍 기초]]
- [[03_lambda-and-higher-order|람다와 고차 함수]]
- [[01_algorithm-performance|알고리즘 성능 분석]]
- [[02_divide-and-conquer|분할 정복 알고리즘]]

### 핵심 개념
- **재귀(Recursion)**: 자기 자신을 호출하는 함수
- **꼬리재귀(Tail Recursion)**: 최적화 가능한 특별한 재귀 형태
- **기저 사례(Base Case)**: 재귀 종료 조건
- **선형 재귀**: 한 번의 재귀 호출
- **트리 재귀**: 여러 번의 재귀 호출
- **메모이제이션**: 결과 캐싱을 통한 최적화

### 더 읽어보기
- SICP Chapter 1.2: Procedures and the Processes They Generate
- "The Little Schemer" - 재귀적 사고 훈련
- "Introduction to Algorithms" - 재귀 알고리즘 분석

---

> [!quote]
> "재귀를 이해하려면, 먼저 재귀를 이해해야 한다." - 컴퓨터 과학 격언
### 예제: 최대공약수 (유클리드 호제법)

> [!example] 효율적인 GCD 알고리즘

```scheme
; 유클리드 호제법: gcd(a, b) = gcd(b, a mod b)
(define (gcd a b)
  (if (= b 0)
      a
      (gcd b (remainder a b))))

; 사용 예제
(gcd 48 18)   ; => 6
(gcd 60 48)   ; => 12

; 연산 과정 추적
(gcd 206 40)
(gcd 40 6)    ; 206 = 40 × 5 + 6
(gcd 6 4)     ; 40 = 6 × 6 + 4  
(gcd 4 2)     ; 6 = 4 × 1 + 2
(gcd 2 0)     ; 4 = 2 × 2 + 0
; => 2
```

### 예제: 빠른 거듭제곱 (Fast Exponentiation)

> [!example] O(log n) 거듭제곱 알고리즘

```scheme
; 일반적인 거듭제곱 O(n)
(define (slow-expt b n)
  (if (= n 0)
      1
      (* b (slow-expt b (- n 1)))))

; 빠른 거듭제곱 O(log n)
; 아이디어: b^n = (b^(n/2))^2 (n이 짝수일 때)
;          b^n = b × b^(n-1) (n이 홀수일 때)
(define (fast-expt b n)
  (cond ((= n 0) 1)
        ((even? n) (square (fast-expt b (/ n 2))))
        (else (* b (fast-expt b (- n 1))))))

(define (square x) (* x x))
(define (even? n) (= (remainder n 2) 0))

; 연산 과정 비교
; slow-expt: 2^8 = 2×2×2×2×2×2×2×2 (8번 곱셈)
; fast-expt: 2^8 = (2^4)^2 = ((2^2)^2)^2 = ((4)^2)^2 = (16)^2 = 256 (3번 곱셈)
```

### 예제: 반복적 버전 구현

> [!example] 반복적 거듭제곱

```scheme
; 반복적 빠른 거듭제곱 (꼬리 재귀)
(define (fast-expt-iter b n)
  (define (iter a b n)
    (cond ((= n 0) a)
          ((even? n) (iter a (square b) (/ n 2)))
          (else (iter (* a b) b (- n 1)))))
  (iter 1 b n))

; 불변식(invariant): a × b^n = 상수
; 초기값: a=1, b=b, n=n → 1 × b^n = b^n
; 최종: a=결과, n=0 → a × b^0 = a
```