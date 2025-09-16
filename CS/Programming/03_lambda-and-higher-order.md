---
tags:
  - 람다
  - 고차함수
  - 함수형프로그래밍
  - 추상화
  - SICP
created: 2025-01-04
updated: 2025-01-05
aliases:
  - Lambda Functions
  - Higher-Order Functions
  - 익명함수
  - 일급함수
description: 람다 표현식과 고차 함수를 통한 강력한 추상화 기법을 학습합니다
category: guide
---

# 람다와 고차 함수 (Lambda and Higher-Order Functions)

> [!info] 개요
> 람다 표현식은 이름 없는 함수를 만드는 강력한 도구이며, 고차 함수는 함수를 값처럼 다루어 추상화 수준을 높이는 프로그래밍 패러다임입니다. 이 두 개념은 함수형 프로그래밍의 핵심이며, 코드의 표현력과 재사용성을 극대화합니다.

## 📑 목차

- [[#🎯 람다 표현식이란?]]
- [[#🔧 람다의 문법과 의미]]
- [[#⚡ 고차 함수의 개념]]
- [[#💡 함수를 반환하는 함수]]
- [[#🔄 함수를 인자로 받는 함수]]
- [[#📊 고차 함수 패턴들]]
- [[#🚀 실전 응용]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## 🎯 람다 표현식이란?

> [!note] 핵심 정의
> **람다(Lambda)**: 이름이 없는 익명 함수를 만드는 특별한 형식. 함수를 값처럼 직접 생성하고 전달할 수 있게 해주는 일급 객체(first-class object) 생성자입니다.

### 람다의 탄생 배경

> [!abstract] 역사적 배경
> 1930년대 Alonzo Church의 람다 대수(Lambda Calculus)에서 유래. 모든 계산을 함수의 정의와 적용만으로 표현하는 수학적 체계입니다.

```scheme
; 일반 함수 정의
(define (square x) (* x x))

; 람다를 이용한 동일한 함수
(define square (lambda (x) (* x x)))

; 즉시 사용하는 익명 함수
((lambda (x) (* x x)) 5)  ; => 25
```

### 람다의 특징

1. **익명성**: 이름 없이 함수 생성 가능
2. **일급 객체**: 변수에 저장, 인자로 전달, 반환값으로 사용 가능
3. **클로저**: 정의된 환경의 변수를 캡처

---

## 🔧 람다의 문법과 의미

### 기본 문법 구조

```scheme
(lambda (매개변수1 매개변수2 ...) 
  본문표현식)
```

> [!example] 다양한 람다 표현식

```scheme
; 단일 매개변수
(lambda (x) (* x 2))

; 다중 매개변수
(lambda (x y) (+ (* x x) (* y y)))

; 매개변수 없음
(lambda () (display "Hello, World!"))

; 가변 인자 (rest parameter)
(lambda args (apply + args))

; 최소 1개 + 가변 인자
(lambda (first . rest) 
  (cons first rest))
```

### 람다와 환경 바인딩

> [!tip] 클로저(Closure) 개념
> 람다는 정의된 시점의 환경을 "캡처"하여 함께 저장합니다.

```scheme
; 클로저 예제
(define (make-adder n)
  (lambda (x) (+ x n)))  ; n을 캡처

(define add5 (make-adder 5))
(define add10 (make-adder 10))

(add5 3)   ; => 8
(add10 3)  ; => 13
```

---

## ⚡ 고차 함수의 개념

> [!success] 고차 함수란?
> **고차 함수(Higher-Order Function)**: 다음 중 하나 이상을 만족하는 함수
> 1. 하나 이상의 함수를 인자로 받거나
> 2. 함수를 결과로 반환하거나
> 3. 또는 둘 다

### 고차 함수의 장점

| 장점 | 설명 | 예시 |
|------|------|------|
| **추상화** | 공통 패턴을 함수로 추출 | map, filter, reduce |
| **재사용성** | 동작을 매개변수화 | 정렬의 비교 함수 |
| **조합성** | 작은 함수들을 조합 | 함수 합성 |
| **표현력** | 간결하고 명확한 코드 | 선언적 프로그래밍 |

---

## 💡 함수를 반환하는 함수

### 함수 팩토리 패턴

> [!example] 함수를 생성하는 함수들

```scheme
; 곱셈 함수 생성기
(define (make-multiplier factor)
  (lambda (x) (* x factor)))

(define double (make-multiplier 2))
(define triple (make-multiplier 3))

(double 5)  ; => 10
(triple 5)  ; => 15

; 비교 함수 생성기
(define (make-comparator op)
  (lambda (x y) (op x y)))

(define greater? (make-comparator >))
(define equal? (make-comparator =))
```

### 커링(Currying)

> [!note] 커링이란?
> 여러 인자를 받는 함수를 단일 인자를 받는 함수들의 체인으로 변환하는 기법

```scheme
; 일반 함수
(define (add x y) (+ x y))

; 커링된 버전
(define (curry-add x)
  (lambda (y) (+ x y)))

; 사용 예
((curry-add 5) 3)  ; => 8

; 일반화된 커링
(define (curry f)
  (lambda (x)
    (lambda (y)
      (f x y))))

(define curried-+ (curry +))
((curried-+ 5) 3)  ; => 8
```

---

## 🔄 함수를 인자로 받는 함수

### Map: 변환 함수

> [!example] 리스트의 각 요소에 함수 적용

```scheme
(define (map proc items)
  (if (null? items)
      '()
      (cons (proc (car items))
            (map proc (cdr items)))))

; 사용 예제
(map (lambda (x) (* x x)) '(1 2 3 4 5))
; => (1 4 9 16 25)

(map abs '(-10 2.5 -11.6 17))
; => (10 2.5 11.6 17)
```

### Filter: 선택 함수

```scheme
(define (filter predicate items)
  (cond ((null? items) '())
        ((predicate (car items))
         (cons (car items) 
               (filter predicate (cdr items))))
        (else (filter predicate (cdr items)))))

; 사용 예제
(filter (lambda (x) (> x 0)) '(-1 2 -3 4 -5))
; => (2 4)

(filter even? '(1 2 3 4 5 6))
; => (2 4 6)
```

### Reduce/Fold: 집계 함수

```scheme
(define (reduce op initial items)
  (if (null? items)
      initial
      (op (car items)
          (reduce op initial (cdr items)))))

; 사용 예제
(reduce + 0 '(1 2 3 4 5))     ; => 15
(reduce * 1 '(1 2 3 4 5))     ; => 120
(reduce cons '() '(1 2 3 4))  ; => (1 2 3 4)
```

---

## 📊 고차 함수 패턴들

### 함수 합성 (Function Composition)

> [!tip] 작은 함수들을 조합하여 복잡한 기능 구현

```scheme
; 함수 합성 연산자
(define (compose f g)
  (lambda (x) (f (g x))))

; 사용 예제
(define inc (lambda (x) (+ x 1)))
(define double (lambda (x) (* x 2)))

(define double-then-inc (compose inc double))
(double-then-inc 5)  ; => 11 (5*2 + 1)

; 여러 함수 합성
(define (pipe . functions)
  (reduce compose
          (lambda (x) x)
          functions))

(define process (pipe double inc square))
(process 3)  ; => 49 ((3*2 + 1)^2)
```

### 부분 적용 (Partial Application)

```scheme
; 일반적인 부분 적용
(define (partial f . args1)
  (lambda args2
    (apply f (append args1 args2))))

; 사용 예제
(define add10 (partial + 10))
(add10 5)  ; => 15

(define starts-with-hello 
  (partial string-append "Hello, "))
(starts-with-hello "World")  ; => "Hello, World"
```

### 메모이제이션 고차 함수

```scheme
; 함수 결과를 캐싱하는 고차 함수
(define (memoize f)
  (let ((cache '()))
    (lambda args
      (let ((cached (assoc args cache)))
        (if cached
            (cdr cached)
            (let ((result (apply f args)))
              (set! cache (cons (cons args result) cache))
              result))))))

; 사용 예제
(define memo-fib
  (memoize
    (lambda (n)
      (if (<= n 1)
          n
          (+ (memo-fib (- n 1))
             (memo-fib (- n 2)))))))
```

---

## 🚀 실전 응용

### 예제 1: 다형성 정렬

> [!example] 비교 함수를 받는 정렬

```scheme
(define (insert x lst compare)
  (cond ((null? lst) (list x))
        ((compare x (car lst))
         (cons x lst))
        (else (cons (car lst)
                   (insert x (cdr lst) compare)))))

(define (insertion-sort lst compare)
  (if (null? lst)
      '()
      (insert (car lst)
              (insertion-sort (cdr lst) compare)
              compare)))

; 오름차순 정렬
(insertion-sort '(3 1 4 1 5) <)  ; => (1 1 3 4 5)

; 내림차순 정렬
(insertion-sort '(3 1 4 1 5) >)  ; => (5 4 3 1 1)
```

### 예제 2: 이벤트 핸들러 시스템

```scheme
(define (make-event-emitter)
  (let ((handlers '()))
    (lambda (action . args)
      (case action
        ((on) (set! handlers 
                (cons (car args) handlers)))
        ((emit) (for-each 
                  (lambda (h) (apply h args))
                  handlers))
        ((off) (set! handlers 
                 (filter (lambda (h) 
                          (not (eq? h (car args))))
                        handlers)))))))

; 사용 예제
(define emitter (make-event-emitter))

(emitter 'on (lambda (x) (display x)))
(emitter 'on (lambda (x) (* x x)))
(emitter 'emit 5)  ; 5를 출력하고 25를 계산
```

### 예제 3: 함수형 데이터 구조

```scheme
; 함수로 구현한 pair
(define (functional-cons x y)
  (lambda (msg)
    (cond ((eq? msg 'car) x)
          ((eq? msg 'cdr) y)
          (else (error "Unknown message")))))

(define (functional-car pair) (pair 'car))
(define (functional-cdr pair) (pair 'cdr))

; 사용 예제
(define p (functional-cons 1 2))
(functional-car p)  ; => 1
(functional-cdr p)  ; => 2
```

---

## ⚠️ 주의사항

### 성능 고려사항

> [!warning] 고차 함수 사용 시 주의점
> 1. **함수 호출 오버헤드**: 과도한 추상화는 성능 저하 가능
> 2. **클로저 메모리**: 캡처된 환경이 메모리 누수 원인이 될 수 있음
> 3. **디버깅 어려움**: 익명 함수는 스택 트레이스 해석이 어려움

### 가독성과 균형

> [!tip] 최적의 사용 지침
> - 간단한 변환은 람다 사용
> - 복잡한 로직은 명명된 함수로 분리
> - 3단계 이상의 함수 합성은 피하기
> - 의도가 명확한 이름 사용

```scheme
; 좋은 예 - 명확한 의도
(filter positive? numbers)

; 나쁜 예 - 과도한 중첩
((compose (partial filter even?) 
          (partial map (lambda (x) (* x 2)))
          (partial reduce + 0))
 data)
```

---

## 📚 참고자료

### 관련 문서
- [[01_functional-programming-basics|함수형 프로그래밍 기초]]
- [[02_recursion-and-tail-recursion|재귀와 꼬리재귀]]
- [[04_abstraction-patterns|추상화 패턴]]

### 핵심 개념
- **람다(Lambda)**: 익명 함수 생성자
- **고차 함수(Higher-Order Function)**: 함수를 다루는 함수
- **클로저(Closure)**: 환경을 캡처하는 함수
- **커링(Currying)**: 다중 인자 함수를 단일 인자 함수 체인으로 변환
- **부분 적용(Partial Application)**: 일부 인자를 고정한 새 함수 생성
- **함수 합성(Composition)**: 여러 함수를 조합

### 더 읽어보기
- SICP Chapter 1.3: Formulating Abstractions with Higher-Order Procedures
- "JavaScript: The Good Parts" - Douglas Crockford
- "Functional Programming in Scala" - Paul Chiusano

---

> [!quote]
> "함수를 일급 시민으로 다루는 능력은 표현력 있는 프로그래밍 언어의 핵심이다." - John McCarthy