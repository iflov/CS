# ✅ Chapter 6: Signals & Clean Shutdown (신호 처리와 안전한 종료)

> ZeroMQ 애플리케이션을 안전하게 종료하고, Ctrl+C 같은 인터럽트를 처리하는 방법을 다룹니다.

---

## 📌 다룰 내용

- Ctrl+C (SIGINT) 처리
- `zmq.Context.term()`의 의미
- `zmq.Socket.close()` 처리 시점
- recv/send 블로킹 시 안전하게 빠져나오기
- 폴링 기반 안전 종료 구조
- `RCVTIMEO`, `Linger`, `poll()` 활용

---

## 🚦 Ctrl+C (SIGINT) 처리

> Python에서는 `signal` 모듈을 이용해 Ctrl+C 인터럽트를 잡아낼 수 있습니다.

### 🐍 Python 예시

```python
import zmq
import signal
import sys

ctx = zmq.Context()
socket = ctx.socket(zmq.REP)
socket.bind("tcp://*:5555")

def shutdown_handler(sig, frame):
    print("\n🛑 인터럽트 감지됨, 종료 중...")
    socket.close()
    ctx.term()
    sys.exit(0)

# SIGINT (Ctrl+C) 수신 시 shutdown_handler 호출
signal.signal(signal.SIGINT, shutdown_handler)

while True:
    msg = socket.recv()
    print("수신:", msg)
    socket.send(b"ACK")
```

---

### 💡 설명

- `signal.signal(signal.SIGINT, handler)`로 인터럽트를 잡습니다.
- 종료 시 반드시 `socket.close()` → `ctx.term()` 순서로 자원을 정리해야 합니다.
- `sys.exit(0)` 호출 시 정상 종료 처리됩니다.

---

## ⏱ 블로킹 수신에서 안전하게 빠져나오기

> 기본 `recv()`는 **무한 블로킹**입니다. Ctrl+C가 수신되어도 종료되지 않을 수 있습니다.

### 해결 방법 1: timeout 설정

```python
socket.setsockopt(zmq.RCVTIMEO, 1000)  # 1000ms (1초)

try:
    msg = socket.recv()
except zmq.Again:
    print("수신 타임아웃 발생")
```

- 일정 시간 동안 응답이 없으면 `zmq.Again` 예외 발생 → 루프를 빠져나올 수 있음
- 서버 루프 내에서 `try-except` 블록으로 수신 타임아웃을 처리하세요.

---

### 해결 방법 2: `zmq.Poller`로 폴링 처리

```python
poller = zmq.Poller()
poller.register(socket, zmq.POLLIN)

while True:
    socks = dict(poller.poll(1000))  # 1초 타임아웃

    if socket in socks:
        msg = socket.recv()
        print("수신:", msg.decode())
```

- `poll()`은 timeout을 설정할 수 있어 `recv()` 블로킹을 피할 수 있습니다.
- 폴링은 다중 소켓에도 적합한 구조입니다.

---

## 🔌 안전한 종료 순서

| 단계 | 설명                                      |
|------|-------------------------------------------|
| 1    | `recv()`에서 빠져나오기 (`timeout`, `poll()`) |
| 2    | `socket.close()`                          |
| 3    | `ctx.term()`                              |
| 4    | `exit()` 또는 `return`                    |

---

## ⏳ linger 설정

> 소켓을 닫을 때 전송되지 않은 메시지를 처리할지 즉시 버릴지를 설정합니다.

```python
socket.setsockopt(zmq.LINGER, 0)  # 닫을 때 메시지 즉시 삭제
```

- 기본 linger는 **-1 (무한 대기)**  
- `0`으로 설정하면 빠른 종료가 가능 (메시지 유실 허용)

---

## ✅ 전체 예제: 안전 종료 가능한 REP 서버

```python
import zmq
import signal
import sys

ctx = zmq.Context()
socket = ctx.socket(zmq.REP)
socket.setsockopt(zmq.RCVTIMEO, 1000)
socket.setsockopt(zmq.LINGER, 0)
socket.bind("tcp://*:5555")

running = True

def shutdown(sig, frame):
    global running
    print("🛑 종료 시그널 수신")
    running = False

signal.signal(signal.SIGINT, shutdown)

while running:
    try:
        msg = socket.recv()
        print("수신:", msg.decode())
        socket.send(b"OK")
    except zmq.Again:
        pass  # timeout → 다음 루프로 진행

socket.close()
ctx.term()
print("🎉 종료 완료")
```

---

## 📎 요약 정리

| 항목                      | 설명                                                         |
|---------------------------|--------------------------------------------------------------|
| Ctrl+C 처리                | `signal.signal(signal.SIGINT, handler)`                     |
| 블로킹 recv 방지           | `RCVTIMEO`, `poll()` 사용                                     |
| 종료 순서                  | `recv()` 탈출 → `close()` → `term()`                         |
| linger 옵션                | `LINGER = 0` → 즉시 종료                                     |
| 멀티 소켓 처리             | `zmq.Poller`를 사용한 이벤트 기반 구조 추천                   |

---

## 📝 참고

- 공식 문서: https://zguide.zeromq.org
- `zmq.Context.term()`은 모든 소켓이 닫힐 때까지 내부에서 대기합니다.
- `RCVTIMEO`, `LINGER`, `Poller`는 모두 `.setsockopt()`로 설정 가능