
# ✅ Chapter 5: ZeroMQ와 멀티스레드 (Working with Threads)

> ZeroMQ는 스레드 간 통신에서도 매우 강력합니다.  
> 이 장에서는 ZeroMQ를 활용해 **멀티스레드 구조**를 어떻게 설계하는지 설명합니다.

---

## 📌 다룰 내용

- ZeroMQ의 thread-safe 개념
- `inproc://` 전송 방식
- 스레드 간 통신 구조 설계
- `PAIR` 소켓 사용법
- ZeroMQ + Thread 조합 예제
- 안전한 종료 처리

---

## 🔒 ZeroMQ는 thread-safe한가요?

### 💡 핵심 정리

- **Context(`zmq.Context`)는 멀티스레드 공유 가능**
- **소켓(`zmq.Socket`)은 단일 스레드 전용**
  - 하나의 소켓은 반드시 한 스레드에서만 사용해야 함
  - 다른 스레드에서 사용 시 예외 발생 또는 데이터 손상 가능

➡️ 스레드 간 통신은 반드시 **각 스레드에서 자체 소켓을 생성**해야 합니다.  
➡️ 이때 사용하는 것이 `inproc://` 전송 방식입니다.

---

## 🔗 `inproc://` 전송 방식

> ZeroMQ에서 가장 빠르고 효율적인 소켓 간 전송 방법입니다.  
> 같은 프로세스 내에서 작동하며, **메모리 복사 없이 직접 버퍼 공유**로 동작합니다.

```text
[Thread A] ↔ inproc://pipe ↔ [Thread B]
```

- 반드시 **동일한 context 객체를 공유**해야 함
- bind/accept 순서 중요: bind는 먼저 실행되어야 안전

---

## 🧪 스레드 간 통신 예제

### 🐍 Python 코드: 메인 → 워커 스레드로 메시지 전송

```python
import zmq
import threading
import time

ctx = zmq.Context()

# 워커 스레드에서 실행될 함수
def worker():
    sock = ctx.socket(zmq.PAIR)
    sock.connect("inproc://pipe")

    while True:
        msg = sock.recv()
        print("[워커] 수신:", msg.decode())
        if msg == b"exit":
            break

    sock.close()

# 메인 스레드
threading.Thread(target=worker).start()

main_sock = ctx.socket(zmq.PAIR)
main_sock.bind("inproc://pipe")

# 메시지 전송
main_sock.send(b"작업 시작")
time.sleep(1)
main_sock.send(b"exit")

main_sock.close()
ctx.term()
```

---

## 🧵 PAIR 소켓의 특징

| 특징                          | 설명 |
|-------------------------------|------|
| 1:1 전용 연결                  | 두 소켓 간 직접 통신만 지원 |
| REQ/REP처럼 순서 제한 없음     | 자유롭게 send/recv 가능 |
| `inproc://`와 함께 사용 시 최적 | 멀티스레드 내부 통신에 적합 |

---

## 🧠 구조 요약

```text
[Main Thread]
   └─ bind("inproc://pipe")
           ↕
   ┌──────────────────────┐
   │ ZeroMQ inproc channel│
   └──────────────────────┘
           ↕
[Worker Thread]
   └─ connect("inproc://pipe")
```

- ZeroMQ 내부적으로 shared queue를 통해 메시지를 전달
- context가 동일해야 `inproc://` 작동함

---

## 🧼 안전한 종료 처리

> ZeroMQ를 멀티스레드에서 사용할 때는 다음을 유의하세요:

1. **항상 소켓을 먼저 종료 → context를 종료**
2. **쓰레드 종료 시 `join()`으로 안전하게 대기**
3. **`recv()`는 기본적으로 blocking → `poll()` 또는 timeout 설정 권장**

---

## 🧪 timeout 또는 비동기 수신

```python
socket.setsockopt(zmq.RCVTIMEO, 1000)  # 1초 timeout

try:
    msg = socket.recv()
except zmq.Again:
    print("수신 타임아웃")
```

---

## 🔁 확장 예시: 다중 워커 구조

```text
[Main Thread] 
   └→ inproc://pipe1 → [Worker 1]
   └→ inproc://pipe2 → [Worker 2]
   └→ ...
```

- 각각의 worker는 독립적인 소켓/스레드로 구성됨
- push/pull 또는 router/dealer 패턴과 조합 가능

---

## 📌 실전 활용 포인트

- 웹 서버에서 요청을 worker 스레드로 분산 처리
- CPU 바운드 연산을 별도 스레드에 오프로드
- I/O 작업 분리 처리

---

## 📎 요약

| 항목               | 설명                                 |
|--------------------|--------------------------------------|
| 소켓 thread-safe?   | ❌ (각 스레드에서 새로 생성해야 함)       |
| context 공유 가능?  | ✅                                  |
| 스레드 통신 전송방식 | `inproc://`                         |
| 추천 소켓 타입      | `PAIR` 또는 `PUSH/PULL`               |
| 속도                | `inproc://` > `ipc://` > `tcp://` 순 |

---

## 📝 참고

- Python ZeroMQ는 `threading` 모듈과 완벽히 호환됩니다.
- `inproc://`은 같은 context 내에서만 작동합니다.
- 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)