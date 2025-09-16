---
tags:
  - programming
  - data-structure
  - abstraction
  - sicp
  - scheme
created: 2025-01-06
updated: 2025-01-06
aliases:
  - 데이터 추상화
  - 추상 데이터 타입
  - ADT
description: 데이터 추상화의 기본 개념과 추상 장벽의 중요성
---

# 데이터 추상화 기초

> [!info] 개요
> 데이터 추상화는 복잡한 데이터 구조의 내부 구현을 숨기고, 사용자에게는 필요한 연산만을 제공하는 프로그래밍 기법입니다. 이를 통해 프로그램의 모듈성과 유지보수성이 크게 향상됩니다.

## 📑 목차

- [[#⚡ 빠른 시작]]
- [[#🔧 핵심 개념]]
- [[#💡 실전 예제]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## ⚡ 빠른 시작

> [!example] 유리수를 통한 데이터 추상화
> ```scheme
> ; 생성자 (Constructor)
> (define (make-rat n d) (cons n d))
> 
> ; 선택자 (Selector)
> (define (numer x) (car x))
> (define (denom x) (cdr x))
> 
> ; 사용 예제
> (define one-half (make-rat 1 2))
> (numer one-half)  ; 1
> (denom one-half)  ; 2
> ```

데이터 추상화는 **데이터를 사용하는 방법**과 **데이터가 구현된 방법**을 분리합니다.

---

## 🔧 핵심 개념

### 추상화의 층위

> [!note] 추상화 계층
> 프로그램은 여러 추상화 층으로 구성되며, 각 층은 아래 층의 세부사항을 숨깁니다.

```
프로그램 사용 층
    ↑
유리수 연산 (add-rat, mul-rat)
    ↑
유리수 추상 장벽 ← [make-rat, numer, denom]
    ↑
Pairs 구현 (cons, car, cdr)
    ↑
메모리/하드웨어
```

### 생성자와 선택자

#### 생성자 (Constructor)

> [!tip] 생성자의 역할
> 여러 부분을 조합하여 복합 데이터를 만드는 프로시저

```scheme
; 2D 점 생성자
(define (make-point x y)
  (cons x y))

; 선분 생성자
(define (make-segment start end)
  (cons start end))
```

#### 선택자 (Selector)

> [!tip] 선택자의 역할
> 복합 데이터에서 구성 요소를 추출하는 프로시저

```scheme
; 2D 점 선택자
(define (x-point p) (car p))
(define (y-point p) (cdr p))

; 선분 선택자
(define (start-segment s) (car s))
(define (end-segment s) (cdr s))
```

### 추상 장벽 (Abstraction Barrier)

> [!important] 추상 장벽의 중요성
> 추상 장벽은 시스템의 다른 레벨을 분리하여, 각 레벨이 독립적으로 변경될 수 있게 합니다.

#### 추상 장벽의 장점

1. **유지보수성**: 구현 변경이 다른 부분에 영향을 주지 않음
2. **재사용성**: 추상화된 데이터 타입을 다양한 곳에서 활용
3. **가독성**: 프로그램의 의도가 명확해짐
4. **확장성**: 새로운 기능을 쉽게 추가 가능

### 데이터 지향 프로그래밍

> [!example] 다양한 표현 지원
> 같은 추상 데이터를 여러 방식으로 구현 가능

```scheme
; 직교 좌표 표현
(define (make-point-rect x y)
  (cons x y))

(define (x-point-rect p) (car p))
(define (y-point-rect p) (cdr p))

; 극좌표 표현
(define (make-point-polar r theta)
  (cons r theta))

(define (x-point-polar p)
  (* (car p) (cos (cdr p))))

(define (y-point-polar p)
  (* (car p) (sin (cdr p))))
```

---

## 💡 실전 예제

### 예제 1: 개선된 유리수 구현

> [!example] 기약분수로 자동 변환
> 생성자에서 최대공약수로 나누어 저장

```scheme
(define (make-rat n d)
  (let ((g (gcd n d)))
    (cons (/ n g) (/ d g))))

(define (numer x) (car x))
(define (denom x) (cdr x))

; 유리수 연산
(define (add-rat x y)
  (make-rat (+ (* (numer x) (denom y))
               (* (numer y) (denom x)))
            (* (denom x) (denom y))))

(define (mul-rat x y)
  (make-rat (* (numer x) (numer y))
            (* (denom x) (denom y))))

(define (equal-rat? x y)
  (= (* (numer x) (denom y))
     (* (numer y) (denom x))))

; 출력
(define (print-rat x)
  (display (numer x))
  (display "/")
  (display (denom x))
  (newline))

; 사용 예제
(define one-third (make-rat 1 3))
(define two-thirds (make-rat 2 3))
(print-rat (add-rat one-third two-thirds))  ; 1/1
```

### 예제 2: 복소수 시스템

> [!example] 다중 표현을 지원하는 복소수
> 직교좌표와 극좌표 표현 모두 지원

```scheme
; 직교좌표 표현
(define (make-complex-rect real imag)
  (cons real imag))

(define (real-part-rect z) (car z))
(define (imag-part-rect z) (cdr z))

(define (magnitude-rect z)
  (sqrt (+ (square (real-part-rect z))
           (square (imag-part-rect z)))))

(define (angle-rect z)
  (atan (imag-part-rect z) (real-part-rect z)))

; 극좌표 표현
(define (make-complex-polar mag ang)
  (cons mag ang))

(define (magnitude-polar z) (car z))
(define (angle-polar z) (cdr z))

(define (real-part-polar z)
  (* (magnitude-polar z) (cos (angle-polar z))))

(define (imag-part-polar z)
  (* (magnitude-polar z) (sin (angle-polar z))))

; 복소수 연산 (표현 독립적)
(define (add-complex z1 z2)
  (make-complex-rect (+ (real-part z1) (real-part z2))
                     (+ (imag-part z1) (imag-part z2))))

(define (mul-complex z1 z2)
  (make-complex-polar (* (magnitude z1) (magnitude z2))
                      (+ (angle z1) (angle z2))))
```

### 예제 3: 간단한 은행 계좌

> [!example] 메시지 패싱을 통한 데이터 추상화
> 객체지향 스타일의 데이터 추상화

```scheme
(define (make-account balance)
  (define (withdraw amount)
    (if (>= balance amount)
        (begin (set! balance (- balance amount))
               balance)
        "잔액 부족"))
  
  (define (deposit amount)
    (set! balance (+ balance amount))
    balance)
  
  (define (dispatch message)
    (cond ((eq? message 'withdraw) withdraw)
          ((eq? message 'deposit) deposit)
          ((eq? message 'balance) balance)
          (else (error "Unknown request" message))))
  
  dispatch)

; 사용 예제
(define acc (make-account 100))
((acc 'withdraw) 50)  ; 50
((acc 'deposit) 40)   ; 90
(acc 'balance)        ; 90
```

---

## ⚠️ 주의사항

> [!warning] 추상화 설계 시 주의점

### 1. 추상 장벽 위반

```scheme
; 나쁜 예: 직접 cons 사용
(define (bad-add-rat x y)
  (cons (+ (* (car x) (cdr y))    ; 추상 장벽 위반!
           (* (car y) (cdr x)))
        (* (cdr x) (cdr y))))

; 좋은 예: 선택자 사용
(define (good-add-rat x y)
  (make-rat (+ (* (numer x) (denom y))
               (* (numer y) (denom x)))
            (* (denom x) (denom y))))
```

### 2. 과도한 추상화

> [!danger] 균형 찾기
> 너무 많은 추상화 층은 프로그램을 복잡하게 만듭니다

```scheme
; 과도한 추상화
(define (make-number n) n)
(define (get-number n) n)

; 적절한 추상화
; 기본 타입은 그대로 사용
```

### 3. 불완전한 인터페이스

> [!warning] 완전성 확인
> 추상 데이터 타입은 필요한 모든 연산을 제공해야 합니다

```scheme
; 불완전한 인터페이스
(define (make-interval low high) (cons low high))
(define (lower-bound x) (car x))
; upper-bound가 없음!

; 완전한 인터페이스
(define (make-interval low high) (cons low high))
(define (lower-bound x) (car x))
(define (upper-bound x) (cdr x))
(define (width-interval x)
  (/ (- (upper-bound x) (lower-bound x)) 2))
```

---

## 📚 참고자료

### 관련 문서
- [[08_cons-car-cdr|cons, car, cdr - 기본 데이터 구조]]
- [[04_abstraction-patterns|추상화 패턴]]
- [[06_let-and-lexical-scope|let과 렉시컬 스코프]]

### 심화 학습
- SICP Chapter 2.1 - Introduction to Data Abstraction
- [추상 데이터 타입 이론](https://en.wikipedia.org/wiki/Abstract_data_type)
- [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns)

### 실습 자료
- [유리수 계산기 구현하기](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book-Z-H-14.html)
- [복소수 시스템 확장하기](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book-Z-H-17.html)

---

> [!quote]
> "프로그램은 사람이 읽기 위해 작성되어야 하며, 기계가 실행하는 것은 부차적인 일이다. 데이터 추상화는 이 목표를 달성하는 핵심 도구다." - Hal Abelson