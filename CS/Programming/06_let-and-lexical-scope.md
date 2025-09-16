---
tags:
  - programming
  - functional-programming
  - scheme
  - sicp
  - scope
  - binding
created: 2025-01-06
updated: 2025-01-06
aliases:
  - let 표현식
  - 렉시컬 스코프
  - 지역 변수
description: let 표현식과 렉시컬 스코프의 개념과 활용법
---

# let 표현식과 렉시컬 스코프

> [!info] 개요
> let 표현식은 지역 변수를 생성하는 특별한 형태로, 코드를 더 읽기 쉽고 효율적으로 만들어줍니다. 렉시컬 스코프는 변수가 정의된 위치에 따라 접근 가능한 범위가 결정되는 규칙입니다.

## 📑 목차

- [[#⚡ 빠른 시작]]
- [[#🔧 핵심 개념]]
- [[#💡 실전 예제]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## ⚡ 빠른 시작

> [!example] let의 기본 사용법
> ```scheme
> ; let을 사용한 지역 변수 생성
> (let ((x 3)
>       (y 4))
>   (+ (* x x) (* y y)))  ; 결과: 25
> 
> ; let 없이 작성하면
> ((lambda (x y) (+ (* x x) (* y y))) 3 4)
> ```

let 표현식은 **지역 변수를 만들어 계산을 단순화**하는 도구입니다. 마치 수학에서 "x = 3, y = 4일 때..."라고 설정하는 것과 같습니다.

---

## 🔧 핵심 개념

### let 표현식의 구조

> [!note] 기본 문법
> let 표현식은 변수 바인딩과 본문으로 구성됩니다.

```scheme
(let ((<var1> <exp1>)
      (<var2> <exp2>)
      ...
      (<varn> <expn>))
  <body>)
```

#### let의 내부 동작

let은 실제로 **lambda 표현식의 문법적 설탕(syntactic sugar)**입니다:

```scheme
; let 표현식
(let ((x 3)
      (y 4))
  (+ x y))

; 동등한 lambda 표현식
((lambda (x y) (+ x y)) 3 4)
```

### 렉시컬 스코프 (Lexical Scope)

> [!tip] 렉시컬 스코프의 원칙
> 변수의 유효 범위는 **코드가 작성된 위치**에 의해 결정되며, 실행 시점이 아닌 **정의 시점**의 환경을 참조합니다.

#### 스코프 규칙

1. **내부 우선 원칙**: 같은 이름의 변수가 중첩되면 가장 안쪽 것이 사용됨
2. **정적 바인딩**: 함수가 정의된 시점의 환경을 기억
3. **블록 스코프**: let 블록 내에서만 변수가 유효

```scheme
(define x 10)  ; 전역 x

(let ((x 20))  ; 지역 x
  (let ((x 30))  ; 더 안쪽 지역 x
    x))  ; 결과: 30

; 전역 x는 여전히 10
```

### let의 변형들

#### 1. let* - 순차적 바인딩

> [!example] let*의 특징
> 이전 바인딩을 다음 바인딩에서 참조 가능

```scheme
(let* ((x 3)
       (y (+ x 2))    ; x를 참조할 수 있음
       (z (+ x y)))   ; x와 y 모두 참조 가능
  (* x y z))  ; 결과: 3 * 5 * 8 = 120
```

#### 2. letrec - 재귀적 바인딩

> [!example] letrec의 활용
> 상호 재귀 함수 정의에 사용

```scheme
(letrec ((even? (lambda (n)
                  (if (= n 0)
                      #t
                      (odd? (- n 1)))))
         (odd? (lambda (n)
                 (if (= n 0)
                     #f
                     (even? (- n 1))))))
  (even? 10))  ; 결과: #t
```

---

## 💡 실전 예제

### 예제 1: 계산 최적화

> [!example] 중복 계산 제거
> let을 사용해 복잡한 식을 한 번만 계산

```scheme
; 비효율적인 코드
(define (f x)
  (+ (sqrt (+ x 1))
     (* 2 (sqrt (+ x 1)))
     (/ 1 (sqrt (+ x 1)))))

; let을 사용한 최적화
(define (f-optimized x)
  (let ((root (sqrt (+ x 1))))
    (+ root
       (* 2 root)
       (/ 1 root))))
```

### 예제 2: 클로저와 렉시컬 스코프

> [!example] 카운터 생성기
> 렉시컬 스코프를 활용한 상태 유지

```scheme
(define (make-counter)
  (let ((count 0))  ; 클로저가 기억하는 변수
    (lambda ()
      (set! count (+ count 1))
      count)))

(define c1 (make-counter))
(define c2 (make-counter))

(c1)  ; 1
(c1)  ; 2
(c2)  ; 1 (독립적인 카운터)
(c1)  ; 3
```

### 예제 3: 복잡한 수식 단순화

> [!example] 이차 방정식의 해
> let을 사용한 가독성 향상

```scheme
(define (quadratic a b c)
  (let ((discriminant (- (* b b) (* 4 a c)))
        (2a (* 2 a)))
    (if (< discriminant 0)
        '복소수-해
        (let ((sqrt-d (sqrt discriminant)))
          (list (/ (+ (- b) sqrt-d) 2a)
                (/ (- (- b) sqrt-d) 2a))))))

(quadratic 1 -3 2)  ; (2 1)
```

---

## ⚠️ 주의사항

> [!warning] 일반적인 실수들

### 1. let vs let* 혼동

```scheme
; 오류: let에서는 같은 레벨의 바인딩 참조 불가
(let ((x 3)
      (y (* x 2)))  ; 오류! x를 참조할 수 없음
  y)

; 해결: let* 사용
(let* ((x 3)
       (y (* x 2)))  ; 정상 작동
  y)  ; 6
```

### 2. 스코프 범위 착각

```scheme
(define x 10)

(define (confusing)
  (let ((x 20))
    (define (inner)
      x)  ; 어떤 x를 참조할까?
    (inner)))

(confusing)  ; 20 (렉시컬 스코프)
```

### 3. 바인딩 순서 주의

> [!danger] let의 병렬 평가
> let의 모든 초기값은 **동시에** 평가됩니다

```scheme
(let ((x 10)
      (y 20))
  (let ((x y)     ; 새로운 x는 20
        (y x))    ; 새로운 y는 10 (외부 x)
    (list x y)))  ; (20 10)
```

---

## 📚 참고자료

### 관련 문서
- [[03_lambda-and-higher-order|람다와 고차 함수]]
- [[04_abstraction-patterns|추상화 패턴]]
- [[01_functional-programming-basics|함수형 프로그래밍 기초]]

### 심화 학습
- SICP Chapter 1.3.3 - General Methods as Procedures
- [Scheme 공식 문서 - let 표현식](https://www.scheme.com/tspl4/)
- [렉시컬 스코프 vs 다이나믹 스코프](https://en.wikipedia.org/wiki/Scope_(computer_science))

### 실습 환경
- [MIT Scheme](https://www.gnu.org/software/mit-scheme/)
- [DrRacket](https://racket-lang.org/)
- [Online Scheme Interpreter](https://replit.com/languages/scheme)

---

> [!quote]
> "let 표현식은 복잡한 계산을 단순하게 만드는 강력한 도구입니다. 지역 변수를 통해 코드의 가독성과 효율성을 동시에 향상시킬 수 있습니다." - SICP