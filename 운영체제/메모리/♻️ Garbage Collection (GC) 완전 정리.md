

---

## **✅ Garbage Collection (GC)란?**

  

> 사용되지 않는 메모리를 자동으로 회수하는 시스템

  

- 사람이 free(), delete 등으로 메모리 해제하지 않아도 됨
    
- **힙(Heap) 메모리**에 있는 객체 중 더 이상 사용되지 않는 것들을 정리
    
- 대표 언어: JavaScript, Python, Java, Go 등
    

---

## **📚 왜 필요한가?**

|**이유**|**설명**|
|---|---|
|메모리 누수 방지|사용이 끝났지만 참조가 남은 객체 방치 시 메모리 누수 발생|
|코드 단순화|개발자가 직접 메모리 해제하지 않아도 됨|
|안전성 향상|이중 해제, dangling pointer 방지|

---

## **🔍 GC 동작 기본 원리**

  
### **✅ 참조 카운팅 (Reference Counting)**

- 객체가 **몇 군데에서 참조되고 있는지**를 추적
    
- 참조 수가 0이 되면 즉시 메모리 해제
    

```
obj = {}      # 참조 카운트 1
ref = obj     # 참조 카운트 2
ref = None    # 참조 카운트 1
obj = None    # 참조 카운트 0 → GC 대상
```

### **✅ 도달 가능성 분석 (Reachability)**

- window, global, main() 등의 **루트 객체에서 도달 가능한지 검사**
    
- 도달할 수 없는 객체는 회수 대상
    

```
let obj = { name: "hyuntae" };
obj = null; // 어디서도 도달 불가 → GC 수거
```

---

## **🐍 Python의 GC (CPython 기준)**

- **참조 카운트 기반**
    
- 순환 참조 해결을 위해 gc 모듈이 추가 탐색
    

```
import gc

a = []
b = []
a.append(b)
b.append(a)

del a
    del b

# 여전히 서로 참조 중이지만...
gc.collect()  # 순환 참조 탐지 후 수거
```

- Python에서는 gc.get_count(), gc.collect() 등으로 수동 관리도 가능
    

---

## **🌐 JavaScript의 GC (V8 기준)**

- **도달 가능성(reachability) 분석**이 핵심
    
- 참조 카운트는 사용되지 않음 (순환 참조에도 강함)
    

```
function create() {
  let obj = { name: "data" };
  return obj;
}

let x = create(); // 루트에서 도달 가능
x = null;         // 도달 불가능 → GC 대상
```

- 수동 GC 호출은 불가능 (DevTools에서 추적은 가능)
    

---

## **🧠 JS vs Python GC 비교 요약**

|**항목**|**JavaScript (V8)**|**Python (CPython)**|
|---|---|---|
|GC 전략|도달 가능성 중심|참조 카운트 + 순환 참조 탐지|
|순환 참조 처리|자동 해결 (기본 동작)|gc 모듈 필요|
|수동 트리거|없음|gc.collect() 사용 가능|
|해제 시점|예측 불가|참조 수 0이면 즉시 해제 (단, 순환은 예외)|

---

## **✅ GC 관련 실무 팁**

- Python에서는 객체가 많거나 순환 참조 의심되면 gc.collect() 고려
    
- JS에서는 DevTools로 메모리 그래프 추적
    
- 이미지, 영상, socket, file 같은 **외부 자원은 명시적으로 close() 필요**
    

```
with open("file.txt") as f:
    data = f.read()  # 자동 close 처리됨
```

---

## **🔚 요약 정리**

|**항목**|**설명**|
|---|---|
|GC란?|사용되지 않는 메모리를 자동 회수하는 시스템|
|주요 대상|Heap에 생성된 객체들|
|JS 방식|도달 가능성 분석 기반 (Reachability)|
|Python 방식|참조 카운트 기반 + 순환 참조 탐지 (gc 모듈)|
|실무 팁|외부 자원은 반드시 명시적 해제 필요 (close, with)|

---

## **🧠 함께 보면 좋은 키워드**

- 메모리 누수 (Memory Leak)
    
- 순환 참조 (Circular Reference)
    
- WeakMap / WeakRef
    
- GC 튜닝 및 프로파일링 도구
    
- __del__ 메서드의 위험성