---
tags:
  - functional-programming
  - scheme
  - lisp
  - programming-paradigm
  - cs-fundamentals
created: 2025-01-05
updated: "updated: 2025-09-05"
aliases:
  - 함수형 프로그래밍
  - Functional Programming
  - FP 기초
  - Scheme 기초
description: 함수형 프로그래밍의 기본 개념과 Scheme/Lisp을 통한 실습
status: published
category: tutorial
---

# 함수형 프로그래밍 기초

> [!info] 개요
> 함수형 프로그래밍은 계산을 수학적 함수의 평가로 취급하는 프로그래밍 패러다임입니다. Scheme과 Lisp를 통해 함수형 프로그래밍의 핵심 개념을 학습합니다.

## 📑 목차

- [[#🎯 함수형 프로그래밍이란?]]
- [[#🔄 Prefix 표기법]]
- [[#📦 기본 구성 요소]]
- [[#🔧 맞바꿈 계산법]]
- [[#🌍 Environment와 Binding]]
- [[#💡 특수 형식 (Special Forms)]]
- [[#🔍 실습 예제]]
- [[#📚 참고자료]]

---

## 🎯 함수형 프로그래밍이란?

> [!note] 핵심 철학
> "프로그램을 작성하는 것은 수학 함수를 정의하는 것과 같다"

### 함수형 프로그래밍의 특징

1. **순수 함수 (Pure Functions)**
   - 동일한 입력에 항상 동일한 출력
   - 부수 효과(Side Effect) 없음

2. **불변성 (Immutability)**
   - 데이터를 변경하지 않고 새로운 데이터 생성
   - 상태 변경 최소화

3. **일급 함수 (First-class Functions)**
   - 함수를 값으로 취급
   - 함수를 인자로 전달, 반환 가능

4. **함수 합성 (Function Composition)**
   - 작은 함수들을 조합해 복잡한 기능 구현

### 명령형 vs 함수형 비교

> [!example] 같은 문제, 다른 접근
> 
> **명령형 (Imperative):**
> ```python
> # 1부터 n까지의 합
> def sum_imperative(n):
>     total = 0
>     for i in range(1, n+1):
>         total += i  # 상태 변경
>     return total
> ```
> 
> **함수형 (Functional):**
> ```scheme
> ; Scheme으로 작성
> (define (sum-functional n)
>   (if (= n 0)
>       0
>       (+ n (sum-functional (- n 1)))))
> ```

---

## 🔄 Prefix 표기법

### 표기법 비교

| 표기법 | 예시 | 설명 |
|--------|------|------|
| **Infix (중위)** | `3 + 4` | 일반적인 수학 표기 |
| **Prefix (전위)** | `+ 3 4` | Lisp/Scheme 방식 |
| **Postfix (후위)** | `3 4 +` | 스택 기반 계산 |

### Prefix 표기법의 장점

> [!success] 왜 Prefix를 사용하나?
> 1. **일관성**: 모든 연산이 동일한 형태
> 2. **가변 인자**: `(+ 1 2 3 4 5)` 가능
> 3. **모호함 없음**: 연산 우선순위 불필요
> 4. **파싱 용이**: 컴파일러 구현 간단

### 복잡한 식 변환

**수학식**: `(3 + 4) × (5 - 2)`

**Prefix 변환**:
```scheme
(* (+ 3 4) (- 5 2))
```

**계산 과정**:
```
(* (+ 3 4) (- 5 2))
= (* 7 (- 5 2))
= (* 7 3)
= 21
```

> [!tip] 읽는 방법
> Prefix 표기법은 "안에서 밖으로" 계산합니다.
> 가장 안쪽 괄호부터 평가하여 바깥으로 진행!

---

## 📦 기본 구성 요소

### 1. 원시 데이터 (Atoms)

```scheme
; 숫자
42
3.14
-17

; 문자열
"Hello, World!"

; 불리언
#t  ; true
#f  ; false

; 심볼
'apple
'x
```

### 2. 조합 (Combinations)

```scheme
; 기본 형태: (연산자 피연산자1 피연산자2 ...)
(+ 2 3)           ; 5
(* 4 5)           ; 20
(- 10 3 2)        ; 5
(/ 12 4)          ; 3
```

### 3. Define - 이름 붙이기

```scheme
; 값에 이름 붙이기
(define pi 3.14159)
(define radius 5)

; 프로시저(함수) 정의
(define (square x)
  (* x x))

; 사용
(square radius)  ; 25

; 원의 넓이 계산
(define (area-of-circle r)
  (* pi (square r)))

(area-of-circle 5)  ; 78.53975
```

### 4. 조건문

```scheme
; if 문
(define (abs x)
  (if (< x 0)
      (- x)    ; then
      x))      ; else

; cond 문 (다중 조건)
(define (grade score)
  (cond ((>= score 90) 'A)
        ((>= score 80) 'B)
        ((>= score 70) 'C)
        ((>= score 60) 'D)
        (else 'F)))
```

---

## 🔧 맞바꿈 계산법

### Applicative Order (적용 순서)

> [!info] 정의
> 인자를 먼저 평가한 후 함수에 적용 (대부분의 언어가 사용)

**예시**:
```scheme
(define (double x) (+ x x))

(double (* 3 4))
; 1단계: (* 3 4) → 12
; 2단계: (double 12) → (+ 12 12) → 24
```

### Normal Order (정규 순서)

> [!info] 정의
> 필요할 때까지 인자 평가를 지연 (Lazy Evaluation)

**예시**:
```scheme
(double (* 3 4))
; 1단계: (+ (* 3 4) (* 3 4))
; 2단계: (+ 12 12)
; 3단계: 24
```

### 차이점 분석

> [!warning] 중요한 차이
> ```scheme
> (define (test x y)
>   (if (= x 0)
>       0
>       y))
> 
> (test 0 (/ 1 0))
> ```
> - **Normal Order**: 0 반환 (y를 평가하지 않음)
> - **Applicative Order**: 에러! (0으로 나누기)

### 실제 사용 예

```scheme
; Scheme은 기본적으로 Applicative Order 사용
; 하지만 특수 형식(if, and, or)은 Normal Order처럼 동작

(and (> x 0) (/ 1 x))  ; x가 0이면 두 번째 평가 안 함
(or #t (/ 1 0))        ; 첫 번째가 참이면 두 번째 평가 안 함
```

---

## 🌍 Environment와 Binding

### Environment란?

> [!note] 정의
> Environment는 이름과 값의 연결(binding)을 저장하는 테이블

### Global vs Local Environment

```scheme
; Global Environment
(define x 10)
(define y 20)

(define (example a)
  ; Local Environment (a는 지역 변수)
  (define b 5)  ; b도 지역 변수
  (+ a b x))    ; x는 global에서 참조

(example 3)  ; 18 (3 + 5 + 10)
```

### Environment 체인

```scheme
(define x 100)  ; Global

(define (outer y)
  (define x 50)  ; outer의 local
  
  (define (inner z)
    (+ x y z))   ; x는 50, y는 outer의 인자
  
  (inner 10))

(outer 20)  ; 80 (50 + 20 + 10)
```

> [!tip] 스코프 규칙
> 변수를 찾을 때 가장 가까운 환경부터 찾아 올라갑니다.
> Local → Enclosing → Global

### Let 바인딩

```scheme
; let으로 임시 바인딩 생성
(let ((x 3)
      (y 4))
  (+ x y))  ; 7

; let은 순차적 바인딩
(let ((x 3)
      (y (* x 2)))  ; 에러! x를 아직 모름
  (+ x y))

; let*는 순차적 바인딩 허용
(let* ((x 3)
       (y (* x 2)))  ; OK, x = 3
  (+ x y))  ; 9
```

---

## 💡 특수 형식 (Special Forms)

### Special Form이란?

> [!important] 핵심 개념
> Special Form은 일반적인 평가 규칙을 따르지 않는 내장 구문입니다.

### 주요 Special Forms

#### 1. define
```scheme
; 평가하지 않고 바인딩 생성
(define x 10)
(define (f x) (* x x))
```

#### 2. if
```scheme
; 조건에 따라 선택적 평가
(if (> x 0)
    (/ 1 x)      ; x > 0일 때만 평가
    "negative")  ; 아니면 이것만 평가
```

#### 3. lambda
```scheme
; 익명 함수 생성
((lambda (x) (* x x)) 5)  ; 25

; define과 동일
(define square (lambda (x) (* x x)))
```

#### 4. quote
```scheme
; 평가 막기
(quote (+ 1 2))  ; (+ 1 2) 그대로
'(+ 1 2)         ; 위와 동일
```

#### 5. and, or
```scheme
; Short-circuit evaluation
(and (> x 0) (< x 10) (even? x))
(or (= x 0) (/ 1 x))
```

---

## 🔍 실습 예제

### 예제 1: 팩토리얼

```scheme
; 재귀 버전
(define (factorial n)
  (if (= n 0)
      1
      (* n (factorial (- n 1)))))

(factorial 5)  ; 120
```

### 예제 2: 피보나치

```scheme
; 단순 재귀 (비효율적)
(define (fib n)
  (cond ((= n 0) 0)
        ((= n 1) 1)
        (else (+ (fib (- n 1))
                 (fib (- n 2))))))
```

### 예제 3: 리스트 처리

```scheme
; 리스트의 합
(define (sum-list lst)
  (if (null? lst)
      0
      (+ (car lst)           ; 첫 원소
         (sum-list (cdr lst)))))  ; 나머지

(sum-list '(1 2 3 4 5))  ; 15

; 리스트 길이
(define (length lst)
  (if (null? lst)
      0
      (+ 1 (length (cdr lst)))))
```

### 예제 4: 고차 함수

```scheme
; map 구현
(define (my-map f lst)
  (if (null? lst)
      '()
      (cons (f (car lst))
            (my-map f (cdr lst)))))

(my-map square '(1 2 3 4))  ; (1 4 9 16)

; filter 구현
(define (filter pred lst)
  (cond ((null? lst) '())
        ((pred (car lst))
         (cons (car lst) (filter pred (cdr lst))))
        (else (filter pred (cdr lst)))))

(filter even? '(1 2 3 4 5 6))  ; (2 4 6)
```

---

## 📚 참고자료

### 추가 학습 자료
- [[02_recursion-and-tail-recursion|재귀와 꼬리재귀]]
- [[03_lambda-and-higher-order|람다와 고차 함수]]
- [[04_abstraction-patterns|추상화 패턴]]

### 책과 문서
- SICP (Structure and Interpretation of Computer Programs)
- The Little Schemer
- [Racket Documentation](https://docs.racket-lang.org/)

### 연습 문제
1. 리스트의 모든 원소를 제곱하는 함수 작성
2. 최대공약수(GCD) 구하기
3. 이진수를 십진수로 변환
4. 간단한 계산기 인터프리터 만들기

---

> [!quote]
> "프로그램은 사람이 읽을 수 있도록 작성되어야 하며, 기계가 실행하는 것은 부차적이다." - Harold Abelson
### 내부 정의 (Internal Definition)

> [!tip] 프로시저 내부의 정의
> 프로시저 내부에서 define을 사용하여 지역 보조 함수를 정의할 수 있습니다.

```scheme
; 제곱근 계산에서 내부 정의 활용
(define (sqrt x)
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001))
  
  (define (improve guess)
    (average guess (/ x guess)))
  
  (define (sqrt-iter guess)
    (if (good-enough? guess)
        guess
        (sqrt-iter (improve guess))))
  
  (sqrt-iter 1.0))

; 장점: 
; 1. 보조 함수들이 외부에 노출되지 않음
; 2. 주 함수의 매개변수에 자유롭게 접근 가능
; 3. 네임스페이스 오염 방지
```

### 블록 구조 (Block Structure)

> [!note] 렉시컬 스코핑
> 내부 정의들은 외부 함수의 매개변수와 지역 변수에 자유롭게 접근할 수 있습니다.

```scheme
; 외부 매개변수 x를 내부 함수들이 공유
(define (sqrt x)
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001))  ; x 자유롭게 사용
  
  (define (improve guess)
    (average guess (/ x guess)))           ; x 자유롭게 사용
  
  (define (sqrt-iter guess)
    (if (good-enough? guess)
        guess
        (sqrt-iter (improve guess))))
  
  (sqrt-iter 1.0))
```