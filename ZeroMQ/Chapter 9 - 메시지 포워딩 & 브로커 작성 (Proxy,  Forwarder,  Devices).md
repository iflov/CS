
# ✅ Chapter 9: 메시지 포워딩 & 브로커 작성 (Proxy / Forwarder / Devices)

> ZeroMQ는 메시지를 **중계(Forwarding)** 하거나 **라우팅(Proxy)** 하는 브로커(중간자)를 매우 간단히 구현할 수 있는 기능을 제공합니다.

---

## 📌 다룰 내용

- 브로커란 무엇인가?
- `zmq.proxy()` 사용법
- XPUB/XSUB 기반 메시지 라우팅
- custom 브로커 구현 전략
- 제어용 control 소켓 사용

---

## 🧠 브로커란?

> 발신자(Sender)와 수신자(Receiver) 사이에 위치하여 메시지를 중계하고, 경우에 따라 라우팅/필터링/기록 등을 수행하는 중간 서버

```text
[Publisher] → [Broker] → [Subscriber]
[Client]    → [Broker] → [Worker]
```

ZeroMQ는 이를 위해 `zmq.proxy()`를 제공합니다.

---

## 🔁 기본 proxy 구조

### 🐍 Python 예시

```python
import zmq

ctx = zmq.Context()

frontend = ctx.socket(zmq.XSUB)  # Publisher들이 연결
frontend.bind("tcp://*:5555")

backend = ctx.socket(zmq.XPUB)   # Subscriber들이 연결
backend.bind("tcp://*:5556")

zmq.proxy(frontend, backend)
```

---

### 💡 설명

- `XSUB`: publisher의 메시지를 수신
- `XPUB`: subscriber의 구독 요청을 수신 및 전송
- `zmq.proxy()`는 메시지를 자동으로 중계함

---

## 🔧 왜 XSUB/XPUB을 써야 하나요?

| 소켓 타입 | 설명                                   |
|-----------|----------------------------------------|
| XSUB      | 여러 publisher 연결을 수용함           |
| XPUB      | subscribe/unsubscribe 메시지를 감지 가능 |
| PUB/SUB   | 단순 송수신만 가능, 제어 불가           |

XPUB/XSUB은 **브로커 입장에서 구독 상태를 알 수 있음**  
→ 메시지를 기록하거나 필터링, 인증 등 추가 동작 가능

---

## 🎛 제어 채널 (Control Socket)

`zmq.proxy()`는 세 번째 인자로 **제어용 control 소켓**을 받을 수 있습니다.

```python
zmq.proxy(frontend, backend, control)
```

- control 소켓은 proxy 이벤트를 외부로 전달하는 채널
- 예: subscribe 이벤트 로깅, shutdown 신호 수신 등

---

## 🧪 control socket 예시 (구독 이벤트 로깅)

```python
import zmq

ctx = zmq.Context()

frontend = ctx.socket(zmq.XSUB)
frontend.bind("tcp://*:5555")

backend = ctx.socket(zmq.XPUB)
backend.bind("tcp://*:5556")

# Control 소켓을 통해 구독 이벤트 수신
control = ctx.socket(zmq.PAIR)
control.bind("inproc://control")

# 브로커 실행 (다른 쓰레드에서)
import threading
def run_proxy():
    zmq.proxy(frontend, backend, control)

threading.Thread(target=run_proxy).start()

# Control 메시지 수신
control_recv = ctx.socket(zmq.PAIR)
control_recv.connect("inproc://control")

while True:
    msg = control_recv.recv()
    print("🔧 Control Event:", msg)
```

---

## 🧪 브로커 없이 직접 연결했을 때 구조

```text
[PUB1] → [SUB1]
[PUB2] → [SUB2]
```

➡️ 각각의 connection을 수동으로 관리해야 하며  
➡️ **확장성, 로깅, 라우팅 불가능**

---

## 🏗 커스텀 브로커 구현 전략

> 직접 메시지를 받아 라우팅하는 구조도 가능

### 예시: PUSH/PULL 기반 라우터

```python
pull = ctx.socket(zmq.PULL)
pull.bind("tcp://*:6000")

push = ctx.socket(zmq.PUSH)
push.bind("tcp://*:6001")

while True:
    msg = pull.recv()
    print("📥 수신:", msg.decode())
    push.send(msg)
```

---

## 📚 proxy()와 device()의 차이점

> 과거 ZeroMQ는 `zmq.device()`를 사용했지만, 현재는 `zmq.proxy()`로 대체됨

| 항목         | 설명                    |
|--------------|-------------------------|
| `zmq.device` | 더 이상 권장되지 않음     |
| `zmq.proxy`  | Python에서도 사용 가능     |
| control 지원 | `proxy()`만 가능          |

---

## ✅ 실제 활용 예

- **Pub-Sub 메시지 브로커**
  - publisher 여러 개 → subscriber 여러 개
- **클라이언트-워커 구조**
  - 클라이언트는 dealer, 워커는 router
- **로그 저장 브로커**
  - 모든 메시지를 저장소에 기록
- **멀티 채널 중계기**
  - 구독자별 채널 라우팅, 권한 필터링

---

## 📎 요약

| 항목                 | 설명                                     |
|----------------------|------------------------------------------|
| 기본 브로커           | `zmq.proxy(frontend, backend)` 사용       |
| 고급 제어             | control 소켓을 통해 이벤트 처리 가능       |
| 브로커 소켓 타입       | XSUB (수신), XPUB (송신)                   |
| 다대다 라우팅         | publisher/subscriber 수에 관계 없이 지원   |
| 확장성               | 제어 로직/필터/로깅 등 추가 구현 가능       |

---

## 📝 참고

- 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)
- pyzmq proxy docs: [https://pyzmq.readthedocs.io](https://pyzmq.readthedocs.io/en/latest/api/zmq.html#zmq.proxy)