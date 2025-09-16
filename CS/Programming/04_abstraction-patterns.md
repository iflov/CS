---
tags:
  - 추상화
  - 데이터추상화
  - 프로세스추상화
  - 프로그래밍패턴
  - SICP
created: 2025-01-04
updated: 2025-01-04
aliases:
  - Abstraction Patterns
  - Data Abstraction
  - 추상 데이터 타입
  - ADT
description: 데이터와 프로세스의 추상화를 통한 효과적인 프로그램 설계 방법을 학습합니다
category: guide
---

# 추상화 패턴 (Abstraction Patterns)

> [!info] 개요
> 추상화는 복잡한 시스템을 관리 가능한 부분으로 나누는 핵심 기법입니다. 데이터 추상화는 구현 세부사항을 숨기고 인터페이스만 노출하며, 프로세스 추상화는 계산 패턴을 일반화합니다. 이를 통해 유지보수성과 확장성이 뛰어난 프로그램을 작성할 수 있습니다.

## 📑 목차

- [[#🎯 추상화란 무엇인가?]]
- [[#📦 데이터 추상화]]
- [[#🔄 프로세스 추상화]]
- [[#🏗️ 추상 데이터 타입 (ADT)]]
- [[#💡 구간 산술 예제]]
- [[#⚡ 리스트 처리 패턴]]
- [[#🎨 추상화 벽과 계층]]
- [[#⚠️ 추상화 설계 원칙]]
- [[#📚 참고자료]]

---

## 🎯 추상화란 무엇인가?

> [!note] 핵심 정의
> **추상화(Abstraction)**: 복잡한 구현 세부사항을 숨기고 본질적인 기능만 드러내는 과정. 사용자는 "무엇을" 할 수 있는지만 알면 되고, "어떻게" 구현되는지는 몰라도 됩니다.

### 추상화의 두 가지 유형

| 유형 | 설명 | 예시 |
|------|------|------|
| **데이터 추상화** | 데이터 구조의 내부 표현을 숨김 | 유리수, 좌표, 날짜 |
| **프로세스 추상화** | 계산 과정의 세부사항을 숨김 | map, filter, reduce |

### 일상 속 추상화

> [!example] 자동차 운전
> 운전자는 핸들, 브레이크, 액셀을 사용하여 자동차를 조작합니다. 엔진 내부의 복잡한 연소 과정이나 변속기의 기어 메커니즘을 알 필요가 없습니다. 이것이 추상화입니다.

---

## 📦 데이터 추상화

### 생성자와 선택자

> [!tip] 데이터 추상화의 핵심 구성요소
> - **생성자(Constructor)**: 데이터를 만드는 함수
> - **선택자(Selector)**: 데이터에서 정보를 추출하는 함수
> - **술어(Predicate)**: 데이터의 속성을 검사하는 함수

```scheme
; 유리수 추상화 예제
; 생성자
(define (make-rat n d)
  (let ((g (gcd n d)))  ; 최대공약수로 약분
    (cons (/ n g) (/ d g))))

; 선택자
(define (numer x) (car x))  ; 분자
(define (denom x) (cdr x))  ; 분모

; 연산
(define (add-rat x y)
  (make-rat (+ (* (numer x) (denom y))
               (* (numer y) (denom x)))
            (* (denom x) (denom y))))

(define (mul-rat x y)
  (make-rat (* (numer x) (numer y))
            (* (denom x) (denom y))))

; 출력
(define (print-rat x)
  (display (numer x))
  (display "/")
  (display (denom x))
  (newline))

; 사용 예
(define one-half (make-rat 1 2))
(define one-third (make-rat 1 3))
(print-rat (add-rat one-half one-third))  ; 5/6
```

### 추상화 벽 (Abstraction Barrier)

> [!abstract] 계층적 설계
> 각 추상화 수준은 그 아래 수준의 구현 세부사항을 알 필요가 없습니다.

```
┌─────────────────────────────────┐
│    프로그램 (유리수 사용)         │
├─────────────────────────────────┤
│  add-rat, sub-rat, mul-rat      │  ← 유리수 연산
├─────────────────────────────────┤
│  make-rat, numer, denom         │  ← 유리수 인터페이스
├─────────────────────────────────┤
│       cons, car, cdr            │  ← Pair 구현
└─────────────────────────────────┘
```

---

## 🔄 프로세스 추상화

### 고차 함수를 통한 패턴 추상화

> [!example] 공통 패턴 추출

```scheme
; 패턴: 리스트의 모든 요소 처리

; 특정 구현들
(define (sum-list items)
  (if (null? items)
      0
      (+ (car items) (sum-list (cdr items)))))

(define (product-list items)
  (if (null? items)
      1
      (* (car items) (product-list (cdr items)))))

; 추상화된 패턴
(define (accumulate op initial items)
  (if (null? items)
      initial
      (op (car items)
          (accumulate op initial (cdr items)))))

; 재구현
(define (sum-list items)
  (accumulate + 0 items))

(define (product-list items)
  (accumulate * 1 items))

(define (length items)
  (accumulate (lambda (x y) (+ 1 y)) 0 items))
```

### 시퀀스 처리 패턴

```scheme
; 시퀀스 연산의 추상화
(define (enumerate-interval low high)
  (if (> low high)
      '()
      (cons low (enumerate-interval (+ low 1) high))))

(define (enumerate-tree tree)
  (cond ((null? tree) '())
        ((not (pair? tree)) (list tree))
        (else (append (enumerate-tree (car tree))
                     (enumerate-tree (cdr tree))))))

; 파이프라인 스타일 프로그래밍
(define (sum-odd-squares tree)
  (accumulate +
              0
              (map square
                   (filter odd?
                          (enumerate-tree tree)))))

; 단계별 분해
; 1. enumerate-tree: 트리 → 리스트
; 2. filter odd?: 홀수만 선택
; 3. map square: 제곱
; 4. accumulate +: 합계
```

---

## 🏗️ 추상 데이터 타입 (ADT)

### 좌표 시스템 ADT

> [!note] 다중 표현 지원
> 동일한 인터페이스로 여러 내부 표현 지원 가능

```scheme
; 직교 좌표계 표현
(define (make-rect-point x y)
  (cons x y))

(define (x-point-rect p) (car p))
(define (y-point-rect p) (cdr p))

; 극좌표계 표현
(define (make-polar-point r theta)
  (cons r theta))

(define (x-point-polar p)
  (* (car p) (cos (cdr p))))

(define (y-point-polar p)
  (* (car p) (sin (cdr p))))

; 통합 인터페이스 (타입 태그 사용)
(define (make-point type x y)
  (cond ((eq? type 'rectangular)
         (cons 'rectangular (make-rect-point x y)))
        ((eq? type 'polar)
         (cons 'polar (make-polar-point x y)))
        (else (error "Unknown type"))))

(define (x-point p)
  (let ((type (car p))
        (contents (cdr p)))
    (cond ((eq? type 'rectangular)
           (x-point-rect contents))
          ((eq? type 'polar)
           (x-point-polar contents))
          (else (error "Unknown type")))))
```

### 메시지 전달 스타일

```scheme
; 객체 지향적 접근
(define (make-account balance)
  (define (withdraw amount)
    (if (>= balance amount)
        (begin (set! balance (- balance amount))
               balance)
        "Insufficient funds"))
  
  (define (deposit amount)
    (set! balance (+ balance amount))
    balance)
  
  (define (dispatch message)
    (cond ((eq? message 'withdraw) withdraw)
          ((eq? message 'deposit) deposit)
          ((eq? message 'balance) balance)
          (else (error "Unknown request"))))
  
  dispatch)

; 사용 예제
(define acc (make-account 100))
((acc 'withdraw) 50)  ; => 50
((acc 'deposit) 40)   ; => 90
(acc 'balance)        ; => 90
```

---

## 💡 구간 산술 예제

> [!example] 불확실성을 다루는 추상화

```scheme
; 구간 생성자와 선택자
(define (make-interval a b) 
  (cons a b))

(define (lower-bound x) (car x))
(define (upper-bound x) (cdr x))

; 구간 연산
(define (add-interval x y)
  (make-interval (+ (lower-bound x) (lower-bound y))
                 (+ (upper-bound x) (upper-bound y))))

(define (mul-interval x y)
  (let ((p1 (* (lower-bound x) (lower-bound y)))
        (p2 (* (lower-bound x) (upper-bound y)))
        (p3 (* (upper-bound x) (lower-bound y)))
        (p4 (* (upper-bound x) (upper-bound y))))
    (make-interval (min p1 p2 p3 p4)
                   (max p1 p2 p3 p4))))

(define (div-interval x y)
  (mul-interval x
                (make-interval (/ 1.0 (upper-bound y))
                               (/ 1.0 (lower-bound y)))))

; 구간의 중심과 너비
(define (center i)
  (/ (+ (lower-bound i) (upper-bound i)) 2))

(define (width i)
  (/ (- (upper-bound i) (lower-bound i)) 2))

; 퍼센트 오차 표현
(define (make-center-percent c p)
  (let ((width (* c (/ p 100.0))))
    (make-interval (- c width) (+ c width))))

(define (percent i)
  (* (/ (width i) (center i)) 100.0))
```

---

## ⚡ 리스트 처리 패턴

### 표준 리스트 연산 추상화

```scheme
; 리스트 생성 패턴
(define (list-ref items n)
  (if (= n 0)
      (car items)
      (list-ref (cdr items) (- n 1))))

(define (length items)
  (if (null? items)
      0
      (+ 1 (length (cdr items)))))

(define (append list1 list2)
  (if (null? list1)
      list2
      (cons (car list1)
            (append (cdr list1) list2))))

; 리스트 변환 패턴
(define (reverse items)
  (define (iter items result)
    (if (null? items)
        result
        (iter (cdr items) 
              (cons (car items) result))))
  (iter items '()))

; 중첩 리스트 처리
(define (deep-reverse items)
  (cond ((null? items) '())
        ((not (pair? items)) items)
        (else (append (deep-reverse (cdr items))
                     (list (deep-reverse (car items)))))))

; 트리 연산
(define (count-leaves tree)
  (cond ((null? tree) 0)
        ((not (pair? tree)) 1)
        (else (+ (count-leaves (car tree))
                (count-leaves (cdr tree))))))

(define (tree-map f tree)
  (cond ((null? tree) '())
        ((not (pair? tree)) (f tree))
        (else (cons (tree-map f (car tree))
                   (tree-map f (cdr tree))))))
```

### 시퀀스 인터페이스

```scheme
; 통일된 시퀀스 인터페이스
(define (filter-accumulate filter? combine null-value term a next b)
  (cond ((> a b) null-value)
        ((filter? a)
         (combine (term a)
                 (filter-accumulate filter? combine 
                                  null-value term 
                                  (next a) next b)))
        (else (filter-accumulate filter? combine 
                               null-value term 
                               (next a) next b))))

; 소수의 제곱합
(define (sum-of-prime-squares a b)
  (filter-accumulate prime? + 0 square a inc b))

; 상대적으로 소인 정수의 곱
(define (product-of-coprimes n)
  (define (coprime? i) (= (gcd i n) 1))
  (filter-accumulate coprime? * 1 identity 1 inc n))
```

---

## 🎨 추상화 벽과 계층

### 계층적 설계의 이점

> [!success] 모듈화의 장점
> 1. **유지보수성**: 각 레벨 독립적 수정 가능
> 2. **재사용성**: 인터페이스를 통한 컴포넌트 재사용
> 3. **테스트 용이성**: 각 레벨 독립적 테스트
> 4. **이해하기 쉬움**: 한 번에 한 레벨만 이해

### 추상화 위반의 위험

> [!danger] 추상화 누수 (Leaky Abstraction)
> 하위 구현 세부사항이 상위 레벨로 노출되는 현상

```scheme
; 나쁜 예 - 추상화 위반
(define (bad-rat-equal? x y)
  (and (= (car x) (car y))    ; cons 구현에 의존
       (= (cdr x) (cdr y))))

; 좋은 예 - 추상화 준수
(define (good-rat-equal? x y)
  (and (= (numer x) (numer y))
       (= (denom x) (denom y))))
```

---

## ⚠️ 추상화 설계 원칙

### 좋은 추상화의 특징

> [!tip] 설계 가이드라인
> 1. **완전성**: 필요한 모든 연산 제공
> 2. **일관성**: 예측 가능한 동작
> 3. **최소성**: 중복 없는 최소 인터페이스
> 4. **독립성**: 구현과 인터페이스 분리

### 추상화 수준 결정

```scheme
; 적절한 추상화 수준 선택

; 너무 낮은 수준
(define (process-data data)
  (cons (car (car data))           ; 구현 세부사항 노출
        (cdr (cdr data))))

; 너무 높은 수준
(define (do-everything data)       ; 너무 많은 책임
  ; ... 100줄의 코드 ...)

; 적절한 수준
(define (transform-record record)  ; 명확한 단일 책임
  (make-record (extract-key record)
               (process-value record)))
```

### 인터페이스 진화

> [!note] 점진적 개선
> 추상화는 한 번에 완벽할 필요 없음. 사용하면서 개선됩니다.

```scheme
; 버전 1: 기본 기능
(define (make-stack) '())
(define (push stack item) (cons item stack))
(define (pop stack) (cdr stack))
(define (top stack) (car stack))

; 버전 2: 안정성 추가
(define (empty-stack? stack) (null? stack))
(define (safe-pop stack)
  (if (empty-stack? stack)
      (error "Empty stack")
      (pop stack)))

; 버전 3: 크기 제한 추가
(define (make-bounded-stack max-size)
  (cons 0 (cons max-size '())))
; ... 추가 구현 ...
```

---

## 📚 참고자료

### 관련 문서
- [[01_functional-programming-basics|함수형 프로그래밍 기초]]
- [[03_lambda-and-higher-order|람다와 고차 함수]]
- [[01_data-structures-overview|자료구조 개요]]

### 핵심 개념
- **추상화(Abstraction)**: 구현 세부사항을 숨기는 기법
- **ADT(Abstract Data Type)**: 추상 데이터 타입
- **생성자(Constructor)**: 데이터 생성 함수
- **선택자(Selector)**: 데이터 접근 함수
- **추상화 벽(Abstraction Barrier)**: 구현과 인터페이스의 경계
- **메시지 전달(Message Passing)**: 객체 지향적 추상화

### 더 읽어보기
- SICP Chapter 2.1-2.2: Building Abstractions with Data
- "Design Patterns" - Gang of Four
- "Clean Architecture" - Robert C. Martin

---

> [!quote]
> "프로그램은 사람이 읽기 위해 작성되어야 하며, 기계가 실행하는 것은 부차적이다." - Harold Abelson