# ✅ Chapter 7: ZeroMQ 메모리 관리 & 메시지 수명 (Memory & Message Lifecycle)

> ZeroMQ는 효율적인 메시지 처리 구조를 가지고 있지만, 잘못 사용하면 메모리 누수 또는 참조 오류가 발생할 수 있습니다.  
> 이 장에서는 메시지의 수명, 메시지 복사 방식, Zero-Copy 전송, 누수 방지 전략 등을 설명합니다.

---

## 📌 다룰 내용

- 메시지 객체의 수명 이해
- 메시지 전송 후 재사용 여부
- 메시지 복사 vs 이동
- Zero-Copy 메시지
- 누수 방지를 위한 코드 습관
- Python에서의 자동 메모리 관리

---

## 🧱 메시지 수명의 기본 원칙

### 💡 개념 요약

- Python에서는 메시지는 일반적으로 `bytes`, `str`, 혹은 `list` 형태로 사용됨
- ZeroMQ는 내부적으로 메시지를 큐에 넣고, 전송 완료 후 **자동으로 메모리 해제**함
- 다만 **전송 후 동일 메시지를 재사용하거나 참조 유지**하면 예기치 않은 오류가 발생할 수 있음

---

## ❌ 메시지 재사용 금지

```python
msg = b"hello"
socket.send(msg)

# 전송한 msg를 다시 수정하거나 재사용하지 않도록 주의
# ZeroMQ 내부적으로 복사될 수도 있지만, 명시적 복사를 추천
```

### 🧠 메시지 복사 전략

```python
original = b"data"
copy = bytes(original)  # 명시적으로 새 객체 생성

socket.send(copy)
```

- 전송할 메시지를 새로 복사하면 side effect를 줄일 수 있음

---

## 📦 Multipart 메시지의 수명 관리

```python
parts = [b"header", b"body", b"footer"]
socket.send_multipart(parts)

# 주의: 전송 후 parts를 다시 전송하면 안 됨
# parts를 재사용할 경우 반드시 복사
```

➡️ 전송에 사용한 리스트나 바이트 객체는 다시 전송하지 말고 새로 생성하세요.

---

## 🚀 Zero-Copy 메시지 전송

> pyzmq는 고급 기능으로 **Zero-Copy 메시지**를 제공합니다.  
> 매우 큰 메시지를 다룰 때 유용합니다.

### 🐍 Python 예시

```python
import zmq
import numpy as np

ctx = zmq.Context()
socket = ctx.socket(zmq.PUSH)
socket.bind("tcp://*:5555")

# numpy 배열을 공유 메모리 기반으로 전송 (zero-copy)
data = np.arange(1000, dtype=np.int32)
socket.send(data.data, copy=False)
```

- `copy=False` 옵션으로 복사 없이 전송됩니다.
- numpy 배열을 직접 전송할 때는 `.data` 또는 `.tobytes()` 사용

---

## ❗ 복사되지 않은 메시지의 주의점

- 전송한 버퍼 객체는 **전송이 완료되기 전까지 수정/해제 금지**
- `copy=False`는 성능은 좋지만 안전하지는 않을 수 있음
- 네트워크 전송이 끝나기 전 수정하면 undefined behavior

---

## 🧪 메모리 누수 방지를 위한 팁

| 상황                         | 안전한 처리 방법                         |
|------------------------------|------------------------------------------|
| 전송 후 데이터 재사용        | ❌ → `bytes()`로 복사 후 전송            |
| multipart 재사용             | ❌ → 리스트 자체를 새로 생성             |
| Zero-copy 사용 시 버퍼 수정 | ❌ → 수정 금지, 전송 완료 후 수정 가능   |
| 소켓/Context 정리            | ✅ `.close()` → `.term()` 순서로 호출    |

---

## ✅ 전체 예제: Zero-Copy와 안전한 종료

```python
import zmq
import numpy as np

ctx = zmq.Context()
socket = ctx.socket(zmq.PUSH)
socket.setsockopt(zmq.LINGER, 0)
socket.bind("tcp://*:5555")

# 대용량 numpy 배열을 zero-copy 전송
arr = np.arange(1000000, dtype=np.float32)

# 전송 완료 전까지 arr 수정 금지
socket.send(arr.data, copy=False)

socket.close()
ctx.term()
```

---

## 🔍 Python에서의 자동 메모리 관리

- Python은 가비지 컬렉션 기반으로 메시지 객체를 자동 관리
- 하지만 `ZeroMQ`는 내부적으로 **C 기반의 버퍼**를 사용하므로,
  - 전송 직후 메시지를 해제하거나 수정하는 행위는 주의해야 함
- Zero-Copy나 multipart를 사용할 때는 더욱 **참조 안정성**을 신경 써야 함

---

## 📎 요약

| 항목                         | 요약 설명                                               |
|------------------------------|----------------------------------------------------------|
| 메시지 재사용 금지            | 전송 후 같은 객체를 다시 사용하지 말 것                 |
| Zero-Copy 전송                | `send(data, copy=False)` 사용 가능                      |
| multipart 재사용 금지        | 리스트 자체를 복사해서 새로 구성                        |
| 누수 방지                    | 전송 전후 메시지 참조에 주의, 소켓/컨텍스트 종료 철저히 |
| numpy + ZeroMQ 전송         | `.data` 또는 `.tobytes()` 사용                           |

---

## 📝 참고

- `copy=False`는 고성능을 원할 때만 사용하세요.
- 일반적인 경우는 `copy=True`가 기본이며, 안전함
- 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)
- pyzmq 문서: [https://pyzmq.readthedocs.io](https://pyzmq.readthedocs.io)