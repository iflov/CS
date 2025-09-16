---
tags:
  - programming
  - data-structure
  - scheme
  - sicp
  - lisp
  - pair
created: 2025-01-06
updated: 2025-01-06
aliases:
  - cons cell
  - 쌍
  - Pair
  - 리스트 구조
description: cons, car, cdr - Scheme/Lisp의 기본 데이터 구조
---

# cons, car, cdr - 기본 데이터 구조

> [!info] 개요
> cons, car, cdr은 Scheme과 Lisp의 가장 기본적인 데이터 구조인 pair(쌍)를 다루는 연산입니다. 이 단순한 구조로부터 리스트, 트리, 그래프 등 모든 복잡한 데이터 구조를 만들 수 있습니다.

## 📑 목차

- [[#⚡ 빠른 시작]]
- [[#🔧 핵심 개념]]
- [[#💡 실전 예제]]
- [[#⚠️ 주의사항]]
- [[#📚 참고자료]]

---

## ⚡ 빠른 시작

> [!example] cons, car, cdr의 기본
> ```scheme
> ; cons: 두 요소를 묶어 pair 생성
> (define x (cons 1 2))
> 
> ; car: pair의 첫 번째 요소
> (car x)  ; 1
> 
> ; cdr: pair의 두 번째 요소
> (cdr x)  ; 2
> 
> ; 리스트 만들기
> (cons 1 (cons 2 (cons 3 '())))  ; (1 2 3)
> ```

**cons**는 건설(construct), **car**는 첫 번째(Contents of Address Register), **cdr**은 나머지(Contents of Decrement Register)를 의미합니다.

---

## 🔧 핵심 개념

### Pair (쌍) 구조

> [!note] Pair의 본질
> Pair는 두 개의 포인터를 가진 가장 단순한 복합 데이터 구조입니다.

```
cons cell (pair):
┌───┬───┐
│ ● │ ● │
└─┼─┴─┼─┘
  ↓   ↓
  1   2
```

#### Box-and-Pointer 표기법

```scheme
(cons 1 2)
; Box diagram:
; [●|●]→2
;  ↓
;  1

(cons (cons 1 2) 3)
; Box diagram:
; [●|●]→3
;  ↓
; [●|●]→2
;  ↓
;  1
```

### 리스트 구조

> [!tip] 리스트의 재귀적 정의
> 리스트는 빈 리스트이거나, 원소와 리스트의 pair입니다.

#### 리스트 구성

```scheme
; 명시적 cons 체인
(cons 1 (cons 2 (cons 3 (cons 4 '()))))

; list 프로시저 사용
(list 1 2 3 4)

; 두 표현은 동일함
```

#### 리스트 시각화

```
(1 2 3 4) =
[●|●]→[●|●]→[●|●]→[●|/]
 ↓    ↓    ↓    ↓
 1    2    3    4
```

### 고급 선택자

> [!example] 중첩된 car/cdr 조합
> ```scheme
> (define x '((1 2) (3 4)))
> 
> (car x)      ; (1 2)
> (cdr x)      ; ((3 4))
> (caar x)     ; 1 = (car (car x))
> (cadr x)     ; (3 4) = (car (cdr x))
> (cdar x)     ; (2) = (cdr (car x))
> (cddr x)     ; () = (cdr (cdr x))
> (cadar x)    ; 2 = (car (cdr (car x)))
> ```

### 트리 구조

> [!note] 트리는 리스트의 리스트
> cons를 사용해 계층적 구조를 표현할 수 있습니다.

```scheme
(define tree '((1 2) (3 (4 5))))

; 트리 구조:
;     [●|●]
;     /   \
;   [●|●] [●|●]
;   /  \   /  \
;  1    2 3  [●|●]
;            /  \
;           4    5
```

---

## 💡 실전 예제

### 예제 1: 리스트 연산 구현

> [!example] 기본 리스트 함수들
> ```scheme
> ; 리스트 길이
> (define (length lst)
>   (if (null? lst)
>       0
>       (+ 1 (length (cdr lst)))))
> 
> ; 리스트 결합
> (define (append lst1 lst2)
>   (if (null? lst1)
>       lst2
>       (cons (car lst1)
>             (append (cdr lst1) lst2))))
> 
> ; 리스트 역순
> (define (reverse lst)
>   (define (iter lst result)
>     (if (null? lst)
>         result
>         (iter (cdr lst)
>               (cons (car lst) result))))
>   (iter lst '()))
> 
> ; 사용 예제
> (length '(1 2 3 4))        ; 4
> (append '(1 2) '(3 4))     ; (1 2 3 4)
> (reverse '(1 2 3 4))       ; (4 3 2 1)
> ```

### 예제 2: 맵과 필터

> [!example] 고차 리스트 함수
> ```scheme
> ; map: 모든 원소에 함수 적용
> (define (map proc lst)
>   (if (null? lst)
>       '()
>       (cons (proc (car lst))
>             (map proc (cdr lst)))))
> 
> ; filter: 조건을 만족하는 원소만 선택
> (define (filter pred lst)
>   (cond ((null? lst) '())
>         ((pred (car lst))
>          (cons (car lst)
>                (filter pred (cdr lst))))
>         (else
>          (filter pred (cdr lst)))))
> 
> ; 사용 예제
> (map square '(1 2 3 4))           ; (1 4 9 16)
> (filter even? '(1 2 3 4 5 6))     ; (2 4 6)
> (map (lambda (x) (* x 2))
>      (filter odd? '(1 2 3 4 5)))  ; (2 6 10)
> ```

### 예제 3: 트리 연산

> [!example] 트리 순회와 변환
> ```scheme
> ; 트리의 모든 잎 개수 세기
> (define (count-leaves tree)
>   (cond ((null? tree) 0)
>         ((not (pair? tree)) 1)
>         (else (+ (count-leaves (car tree))
>                  (count-leaves (cdr tree))))))
> 
> ; 트리의 모든 값에 함수 적용
> (define (tree-map proc tree)
>   (cond ((null? tree) '())
>         ((not (pair? tree)) (proc tree))
>         (else (cons (tree-map proc (car tree))
>                     (tree-map proc (cdr tree))))))
> 
> ; 트리 평탄화 (flatten)
> (define (flatten tree)
>   (cond ((null? tree) '())
>         ((not (pair? tree)) (list tree))
>         (else (append (flatten (car tree))
>                       (flatten (cdr tree))))))
> 
> ; 사용 예제
> (define x '((1 2) (3 (4 5))))
> (count-leaves x)            ; 5
> (tree-map square x)         ; ((1 4) (9 (16 25)))
> (flatten x)                 ; (1 2 3 4 5)
> ```

### 예제 4: 연관 리스트 (Association List)

> [!example] 키-값 쌍 저장
> ```scheme
> ; 연관 리스트 생성
> (define (make-alist) '())
> 
> ; 키-값 쌍 추가
> (define (assoc-add key value alist)
>   (cons (cons key value) alist))
> 
> ; 키로 값 찾기
> (define (assoc-find key alist)
>   (cond ((null? alist) #f)
>         ((equal? key (car (car alist)))
>          (cdr (car alist)))
>         (else (assoc-find key (cdr alist)))))
> 
> ; 사용 예제
> (define dict '())
> (set! dict (assoc-add 'name "Alice" dict))
> (set! dict (assoc-add 'age 25 dict))
> (set! dict (assoc-add 'city "Seoul" dict))
> 
> (assoc-find 'name dict)  ; "Alice"
> (assoc-find 'age dict)   ; 25
> ```

---

## ⚠️ 주의사항

> [!warning] cons 사용 시 주의점

### 1. Improper List

```scheme
; Proper list (올바른 리스트)
(cons 1 (cons 2 (cons 3 '())))  ; (1 2 3)

; Improper list (부적절한 리스트)
(cons 1 (cons 2 3))  ; (1 2 . 3)

; 점 표기법
'(1 . 2)  ; pair이지만 리스트가 아님
```

### 2. 무한 재귀 위험

> [!danger] 순환 구조 주의
> ```scheme
> ; 순환 참조 생성 (주의!)
> (define x (cons 1 2))
> (set-cdr! x x)  ; x의 cdr이 자기 자신을 가리킴
> 
> ; 이제 x를 출력하면 무한 루프
> ; x = (1 . #0=(1 . #0#))
> ```

### 3. 효율성 고려

> [!warning] 리스트 연산의 시간 복잡도
> ```scheme
> ; O(n) - 선형 시간
> (length lst)
> (append lst1 lst2)
> 
> ; O(1) - 상수 시간
> (cons x lst)
> (car lst)
> (cdr lst)
> 
> ; 비효율적인 패턴
> (define (bad-reverse lst)
>   (if (null? lst)
>       '()
>       (append (bad-reverse (cdr lst))
>               (list (car lst)))))  ; O(n²)
> ```

### 4. 메모리 공유

> [!note] 구조 공유
> ```scheme
> (define x '(3 4))
> (define y (cons 1 x))
> (define z (cons 2 x))
> 
> ; y = (1 3 4)
> ; z = (2 3 4)
> ; x 부분은 y와 z가 공유함
> 
> (set-car! x 'changed)
> ; y = (1 changed 4)
> ; z = (2 changed 4)
> ```

---

## 📚 참고자료

### 관련 문서
- [[07_data-abstraction-basics|데이터 추상화 기초]]
- [[02_recursion-and-tail-recursion|재귀와 꼬리 재귀]]
- [[03_lambda-and-higher-order|람다와 고차 함수]]

### 심화 학습
- SICP Chapter 2.2 - Hierarchical Data and the Closure Property
- [Lisp의 역사와 cons cell](https://en.wikipedia.org/wiki/CAR_and_CDR)
- [함수형 데이터 구조](https://www.cs.cmu.edu/~rwh/theses/okasaki.pdf)

### 연습 문제
1. deep-reverse: 중첩된 리스트까지 역순으로 만들기
2. fringe: 트리의 모든 잎을 왼쪽에서 오른쪽 순서로 나열
3. mobile: 균형 잡힌 모빌 구조 구현

---

> [!quote]
> "cons, car, cdr이라는 단순한 연산으로부터 무한히 복잡한 데이터 구조를 만들 수 있다. 이것이 Lisp의 우아함이다." - John McCarthy