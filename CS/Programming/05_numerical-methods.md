---
tags:
  - 수치계산
  - 알고리즘
  - 근사
  - 수학
  - SICP
created: 2025-01-04
updated: "updated: 2025-09-05"
aliases:
  - Numerical Methods
  - 수치해석
  - Computational Methods
description: 함수형 프로그래밍을 활용한 수치 계산 방법과 근사 알고리즘을 학습합니다
category: guide
---

# 수치 계산법 (Numerical Methods)

> [!info] 개요
> 수치 계산법은 수학적 문제를 컴퓨터로 근사적으로 해결하는 기법입니다. 함수형 프로그래밍의 고차 함수와 재귀를 활용하면 복잡한 수치 알고리즘을 우아하고 명확하게 표현할 수 있습니다. 뉴턴 방법, 이분법, 고정점 찾기 등의 핵심 알고리즘을 학습합니다.

## 📑 목차

- [[#🎯 수치 계산의 기초]]
- [[#🔍 근 찾기 알고리즘]]
- [[#⚡ 뉴턴의 방법]]
- [[#📊 이분법 (Bisection Method)]]
- [[#🔄 고정점 찾기]]
- [[#💡 수치 적분과 미분]]
- [[#🎨 응용 예제]]
- [[#⚠️ 수치 계산의 한계]]
- [[#📚 참고자료]]

---

## 🎯 수치 계산의 기초

> [!note] 핵심 개념
> **수치 계산(Numerical Computation)**: 해석적 해를 구하기 어려운 문제를 반복적 계산을 통해 근사해를 찾는 과정. 정확한 답 대신 "충분히 좋은" 답을 효율적으로 구합니다.

### 근사와 오차

```scheme
; 허용 오차를 정의
(define tolerance 0.00001)

; 두 값이 충분히 가까운지 검사
(define (close-enough? v1 v2)
  (< (abs (- v1 v2)) tolerance))

; 상대 오차 계산
(define (relative-error actual approx)
  (/ (abs (- actual approx)) 
     (abs actual)))

; 퍼센트 오차
(define (percent-error actual approx)
  (* 100 (relative-error actual approx)))
```

### 수렴 판정

> [!tip] 수렴 조건
> 1. 절대 오차: |xₙ - xₙ₋₁| < ε
> 2. 상대 오차: |xₙ - xₙ₋₁|/|xₙ| < ε
> 3. 함수값: |f(xₙ)| < ε

---

## 🔍 근 찾기 알고리즘

### 기본 구조

> [!abstract] 반복적 개선 패턴
> 모든 수치 알고리즘의 공통 구조: 추측 → 개선 → 수렴 검사 → 반복

```scheme
; 일반적인 반복 개선 구조
(define (iterative-improve good-enough? improve)
  (lambda (guess)
    (if (good-enough? guess)
        guess
        ((iterative-improve good-enough? improve)
         (improve guess)))))

; 제곱근 구하기 (바빌로니아 방법)
(define (sqrt x)
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001))
  
  (define (improve guess)
    (average guess (/ x guess)))
  
  ((iterative-improve good-enough? improve) 1.0))

(define (average x y) (/ (+ x y) 2))
(define (square x) (* x x))
```

---

## ⚡ 뉴턴의 방법

> [!success] 뉴턴-랩슨 방법
> **원리**: 함수의 접선을 이용하여 근을 빠르게 찾는 방법
> 
> 공식: xₙ₊₁ = xₙ - f(xₙ)/f'(xₙ)

### 미분 계산

```scheme
; 수치 미분
(define dx 0.00001)

(define (deriv g)
  (lambda (x)
    (/ (- (g (+ x dx)) (g x))
       dx)))

; 사용 예
(define (cube x) (* x x x))
((deriv cube) 5)  ; ≈ 75 (실제: 3*5² = 75)
```

### 뉴턴 방법 구현

```scheme
; 뉴턴 변환
(define (newton-transform g)
  (lambda (x)
    (- x (/ (g x) ((deriv g) x)))))

; 뉴턴 방법
(define (newtons-method g guess)
  (fixed-point (newton-transform g) guess))

; 제곱근 구하기 (뉴턴 방법 사용)
(define (sqrt x)
  (newtons-method (lambda (y) (- (square y) x))
                   1.0))

; 세제곱근 구하기
(define (cube-root x)
  (newtons-method (lambda (y) (- (* y y y) x))
                   1.0))

; 방정식 해 구하기: x³ + 2x² - 5 = 0
(define (solve-equation)
  (newtons-method (lambda (x) (+ (* x x x) 
                                 (* 2 x x) 
                                 -5))
                   1.0))
```

### 수렴 속도

> [!example] 뉴턴 방법의 수렴 과정
> sqrt(2)를 구하는 과정:

```scheme
(define (sqrt-iter x)
  (define (f y) (- (* y y) x))
  (define (improve guess)
    (- guess (/ (f guess) (* 2 guess))))
  
  (define (iter guess count)
    (display count) (display ": ") 
    (display guess) (newline)
    (if (< (abs (f guess)) 0.00001)
        guess
        (iter (improve guess) (+ count 1))))
  
  (iter 1.0 0))

; 출력:
; 0: 1.0
; 1: 1.5
; 2: 1.4166666666666665
; 3: 1.4142156862745097
; 4: 1.4142135623746899
```

---

## 📊 이분법 (Bisection Method)

> [!note] 이분법 원리
> 연속 함수에서 f(a) < 0이고 f(b) > 0이면, a와 b 사이에 근이 존재합니다. 구간을 반으로 나누며 근을 찾습니다.

### 이분법 구현

```scheme
; 반개구간 탐색
(define (search f neg-point pos-point)
  (let ((midpoint (average neg-point pos-point)))
    (if (close-enough? neg-point pos-point)
        midpoint
        (let ((test-value (f midpoint)))
          (cond ((positive? test-value)
                 (search f neg-point midpoint))
                ((negative? test-value)
                 (search f midpoint pos-point))
                (else midpoint))))))

; 안전한 이분법 (부호 검사 포함)
(define (half-interval-method f a b)
  (let ((a-value (f a))
        (b-value (f b)))
    (cond ((and (negative? a-value) (positive? b-value))
           (search f a b))
          ((and (negative? b-value) (positive? a-value))
           (search f b a))
          (else
           (error "Values are not of opposite sign" a b)))))

; 사용 예: sin(x) = 0의 해 찾기 (π 근처)
(half-interval-method sin 2.0 4.0)
; => 3.14111328125 (π의 근사값)

; x³ - 2x - 5 = 0의 해 찾기
(half-interval-method (lambda (x) (- (* x x x) (* 2 x) 5))
                      1.0
                      3.0)
; => 2.094482421875
```

### 수렴 분석

> [!tip] 이분법 vs 뉴턴 방법
> | 특성 | 이분법 | 뉴턴 방법 |
> |------|--------|-----------|
> | 수렴 속도 | 선형 O(n) | 이차 O(n²) |
> | 초기값 요구 | 부호가 다른 두 점 | 하나의 추측값 |
> | 안정성 | 항상 수렴 | 발산 가능 |
> | 미분 필요 | 불필요 | 필요 |

---

## 🔄 고정점 찾기

> [!abstract] 고정점 정의
> **고정점(Fixed Point)**: f(x) = x를 만족하는 x값. 많은 수치 문제가 고정점 문제로 변환 가능합니다.

### 고정점 알고리즘

```scheme
; 기본 고정점 찾기
(define (fixed-point f first-guess)
  (define (close-enough? v1 v2)
    (< (abs (- v1 v2)) tolerance))
  
  (define (try guess)
    (let ((next (f guess)))
      (if (close-enough? guess next)
          next
          (try next))))
  
  (try first-guess))

; cos의 고정점 (cos(x) = x)
(fixed-point cos 1.0)
; => 0.7390822985224024

; 황금비 구하기: x = 1 + 1/x
(fixed-point (lambda (x) (+ 1 (/ 1 x))) 1.0)
; => 1.6180327868852458
```

### 평균 감쇠 기법

> [!warning] 진동 문제 해결
> 단순 반복이 진동할 때 평균 감쇠(average damping)를 사용합니다.

```scheme
; 평균 감쇠
(define (average-damp f)
  (lambda (x) (average x (f x))))

; 개선된 제곱근 (평균 감쇠 적용)
(define (sqrt x)
  (fixed-point (average-damp (lambda (y) (/ x y)))
               1.0))

; 세제곱근 (이중 평균 감쇠)
(define (cube-root x)
  (fixed-point (average-damp (lambda (y) (/ x (square y))))
               1.0))

; n제곱근 일반화
(define (nth-root x n)
  (fixed-point 
   ((repeated average-damp (floor (log2 n)))
    (lambda (y) (/ x (expt y (- n 1)))))
   1.0))

(define (repeated f n)
  (if (= n 1)
      f
      (compose f (repeated f (- n 1)))))

(define (compose f g)
  (lambda (x) (f (g x))))
```

---

## 💡 수치 적분과 미분

### 심슨 적분법

> [!example] 심슨의 1/3 규칙

```scheme
; 심슨 적분: ∫[a,b] f(x)dx
(define (simpson f a b n)
  (define h (/ (- b a) n))
  
  (define (y k) (f (+ a (* k h))))
  
  (define (simpson-term k)
    (cond ((or (= k 0) (= k n)) (y k))
          ((even? k) (* 2 (y k)))
          (else (* 4 (y k)))))
  
  (* (/ h 3)
     (sum simpson-term 0 inc n)))

(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a)
         (sum term (next a) next b))))

(define (inc x) (+ x 1))

; 사용 예: ∫[0,1] x³ dx = 0.25
(simpson cube 0 1 100)  ; => 0.24999999999999992
```

### 수치 미분 개선

```scheme
; 중심 차분법 (더 정확)
(define (deriv-center g)
  (let ((dx 0.00001))
    (lambda (x)
      (/ (- (g (+ x dx)) (g (- x dx)))
         (* 2 dx)))))

; 2차 미분
(define (second-deriv g)
  (deriv (deriv g)))

; n차 미분
(define (nth-derivative g n)
  (if (= n 0)
      g
      (deriv (nth-derivative g (- n 1)))))
```

---

## 🎨 응용 예제

### 예제 1: 연속 분수

> [!example] 황금비의 연속 분수 표현

```scheme
; 연속 분수 계산
(define (cont-frac n d k)
  (define (iter i result)
    (if (= i 0)
        result
        (iter (- i 1)
              (/ (n i) (+ (d i) result)))))
  (iter k 0))

; 황금비: 1/(1 + 1/(1 + 1/(1 + ...)))
(define (golden-ratio k)
  (+ 1 (cont-frac (lambda (i) 1.0)
                  (lambda (i) 1.0)
                  k)))

(golden-ratio 10)   ; => 1.6179775280898876
(golden-ratio 100)  ; => 1.618033988749895

; e - 2의 연속 분수
(define (e-minus-2 k)
  (cont-frac (lambda (i) 1.0)
             (lambda (i) 
               (if (= (remainder i 3) 2)
                   (* 2 (/ (+ i 1) 3))
                   1))
             k))

(+ 2 (e-minus-2 10))  ; => 2.7182817182817183
```

### 예제 2: 최적화 문제

```scheme
; 경사 하강법
(define (gradient-descent f initial-point learning-rate iterations)
  (define grad (gradient f))
  
  (define (update point)
    (vector-map (lambda (x g) (- x (* learning-rate g)))
                point
                (grad point)))
  
  (define (iter point n)
    (if (= n 0)
        point
        (iter (update point) (- n 1))))
  
  (iter initial-point iterations))

; 함수의 최소값 찾기: f(x) = x² - 4x + 4
(define (minimize-quadratic)
  (fixed-point 
   (lambda (x) (- x (* 0.1 (- (* 2 x) 4))))  ; x - α*f'(x)
   0.0))
; => 2.0 (최소값은 x=2에서)
```

### 예제 3: 파이(π) 계산

```scheme
; 라이프니츠 급수: π/4 = 1 - 1/3 + 1/5 - 1/7 + ...
(define (pi-leibniz terms)
  (* 4 (sum (lambda (n)
              (/ (if (even? n) 1 -1)
                 (+ (* 2 n) 1)))
            0
            inc
            terms)))

; 몬테카를로 방법
(define (monte-carlo-pi trials)
  (define (in-circle? x y)
    (<= (+ (square x) (square y)) 1))
  
  (define (trial)
    (let ((x (random-range -1 1))
          (y (random-range -1 1)))
      (if (in-circle? x y) 1 0)))
  
  (* 4.0 (/ (sum-trials trials) trials)))

; 마친 공식 (매우 빠른 수렴)
(define (pi-machin)
  (* 4 (- (* 4 (arctan 1/5))
          (arctan 1/239))))
```

---

## ⚠️ 수치 계산의 한계

### 부동소수점 오류

> [!danger] 주의사항
> 1. **반올림 오류**: 유한 정밀도로 인한 오차
> 2. **취소 오류**: 비슷한 크기의 수를 빼면서 발생
> 3. **오버플로/언더플로**: 표현 범위 초과

```scheme
; 부동소수점 오류 예시
(= 0.1 (- 1.0 0.9))  ; => #f (false!)

; 안전한 비교
(define (float-equal? a b epsilon)
  (< (abs (- a b)) epsilon))

; 수치 안정성 개선
(define (stable-quadratic a b c)
  (let ((discriminant (- (* b b) (* 4 a c))))
    (if (< discriminant 0)
        'no-real-roots
        (let ((sqrt-disc (sqrt discriminant)))
          (if (> b 0)
              (list (/ (- (+ b sqrt-disc)) (* 2 a))
                    (/ (* 2 c) (- (+ b sqrt-disc))))
              (list (/ (* 2 c) (+ (- b) sqrt-disc))
                    (/ (+ (- b) sqrt-disc) (* 2 a))))))))
```

### 수렴 실패 처리

```scheme
; 최대 반복 횟수 제한
(define (fixed-point-limited f first-guess max-iter)
  (define (try guess count)
    (if (> count max-iter)
        (error "Failed to converge" guess)
        (let ((next (f guess)))
          (if (close-enough? guess next)
              next
              (try next (+ count 1))))))
  (try first-guess 0))

; 수렴 히스토리 추적
(define (fixed-point-traced f first-guess)
  (define (try guess)
    (display guess) (newline)
    (let ((next (f guess)))
      (if (close-enough? guess next)
          next
          (try next))))
  (try first-guess))
```

---

## 📚 참고자료

### 관련 문서
- [[01_functional-programming-basics|함수형 프로그래밍 기초]]
- [[02_recursion-and-tail-recursion|재귀와 꼬리재귀]]
- [[03_lambda-and-higher-order|람다와 고차 함수]]
- [[01_algorithm-performance|알고리즘 성능 분석]]

### 핵심 개념
- **수치 해법**: 근사적 문제 해결 방법
- **뉴턴 방법**: 미분을 이용한 빠른 근 찾기
- **이분법**: 안정적인 구간 탐색
- **고정점**: f(x) = x를 만족하는 점
- **평균 감쇠**: 진동 방지 기법
- **심슨 적분**: 정확한 수치 적분

### 더 읽어보기
- SICP Chapter 1.3: Formulating Abstractions with Higher-Order Procedures
- "Numerical Recipes" - William H. Press et al.
- "Introduction to Numerical Analysis" - Stoer & Bulirsch

---

> [!quote]
> "신은 정수를 만들었다. 나머지는 모두 인간의 작품이다." - Leopold Kronecker
### 뉴턴 방법을 이용한 제곱근 구하기

> [!example] 제곱근을 구하는 상세 과정
> √x를 구하는 것은 y² - x = 0의 해를 구하는 것과 같습니다.

```scheme
; y² = x의 해를 구하기 위해 f(y) = y² - x로 설정
; f'(y) = 2y이므로 뉴턴 공식: y_{n+1} = y_n - (y_n² - x)/(2y_n)
; 정리하면: y_{n+1} = (y_n + x/y_n)/2

(define (sqrt-newton x)
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001))
  
  (define (improve guess)
    (average guess (/ x guess)))  ; 뉴턴 공식 적용
  
  (define (sqrt-iter guess)
    (if (good-enough? guess)
        guess
        (sqrt-iter (improve guess))))
  
  (sqrt-iter 1.0))

(define (average x y) (/ (+ x y) 2))
(define (square x) (* x x))

; √2 계산 과정 시각화
; 초기 추측: 1.0
; 1단계: (1.0 + 2/1.0)/2 = 1.5
; 2단계: (1.5 + 2/1.5)/2 = 1.416666...
; 3단계: (1.416666... + 2/1.416666...)/2 = 1.414215...
; 4단계: 1.414213... (실제 √2 = 1.414213562...)
```

### 반복적 개선 추상화

> [!note] 일반화된 반복 개선 패턴
> 뉴턴 방법과 고정점 찾기의 공통 패턴을 추상화할 수 있습니다.

```scheme
; 일반화된 반복적 개선
(define (iterative-improve good-enough? improve)
  (lambda (guess)
    (if (good-enough? guess)
        guess
        ((iterative-improve good-enough? improve)
         (improve guess)))))

; 제곱근을 반복적 개선으로 구현
(define (sqrt-general x)
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001))
  
  (define (improve guess)
    (average guess (/ x guess)))
  
  ((iterative-improve good-enough? improve) 1.0))

; 고정점을 반복적 개선으로 구현
(define (fixed-point-general f first-guess)
  (define (good-enough? guess)
    (< (abs (- guess (f guess))) tolerance))
  
  ((iterative-improve good-enough? f) first-guess))
```