

# ✅ Chapter 3: Advanced Request-Reply Patterns (고급 요청-응답 패턴)

> 단순한 `REQ-REP` 패턴을 넘어서 다양한 라우팅/비동기 패턴을 어떻게 구현할 수 있는지를 다룹니다.

---

## 📌 다룰 내용

- REQ의 한계와 문제점
- DEALER-ROUTER 패턴
- 비동기 요청/응답 구현
- 메시지 ID 및 라우팅 관리
- 커스텀 라우터 구현
- Multipart 메시지의 응용

---

## 🚫 REQ-REP의 한계

### 💡 설명

기본 `REQ-REP` 구조는 다음과 같은 **제약**이 있습니다:

1. **동기 구조**  
   - `REQ`는 반드시 `send()` → `recv()` 순서를 지켜야 하며, 반대로 `REP`은 `recv()` → `send()` 순서만 허용됩니다.
   - 요청을 보낸 뒤 응답을 기다리기 전에는 **다른 요청을 보낼 수 없습니다.**

2. **병렬 처리가 불가능**  
   - 하나의 `REP` 소켓은 한 번에 하나의 요청만 처리합니다.  
   - 동시에 들어오는 요청은 순차적으로 처리되어야 하며, 대기열이 쌓이게 됩니다.

3. **라우팅 제어 불가**  
   - 메시지가 어떤 클라이언트로부터 왔는지 직접 알 수 없습니다.
   - 특정 클라이언트로 선택적으로 응답을 보내는 것도 불가능합니다.

➡️ 이 문제를 해결하기 위한 고급 패턴이 필요합니다.

---

## 🧩 DEALER-ROUTER 패턴

> ✅ `REQ-REP`을 완전히 비동기 구조로 확장한 패턴입니다.  
> ✅ 라우팅, 응답 분기, 멀티클라이언트 대응에 강력한 구조를 제공합니다.

---

### 🐍 ROUTER 서버 예제 (멀티 클라이언트 라우팅)

```python
import zmq

ctx = zmq.Context()
socket = ctx.socket(zmq.ROUTER)
socket.bind("tcp://*:5555")

while True:
    # multipart 메시지 수신: [client_id, empty, content]
    msg = socket.recv_multipart()
    client_id, _, content = msg
    print(f"From {client_id}: {content.decode()}")

    # 클라이언트 ID를 포함하여 응답
    socket.send_multipart([client_id, b'', b'ACK'])
```

---

### 🐍 DEALER 클라이언트 예제 (비동기 클라이언트)

```python
import zmq

ctx = zmq.Context()
socket = ctx.socket(zmq.DEALER)
socket.connect("tcp://localhost:5555")

# 메시지 전송 (빈 프레임 생략 가능)
socket.send(b"Hello ROUTER")

# 메시지 수신
reply = socket.recv()
print("Reply:", reply.decode())
```

---

### 💡 설명

- `ROUTER`는 클라이언트 ID를 기반으로 라우팅을 수행할 수 있습니다.
- `DEALER`는 순서 제한이 없어, `send()`, `send()`, `recv()`도 가능합니다.
- 메시지는 항상 **Multipart**로 전송됩니다:
  - `ROUTER`: `[client_id, "", message]`
  - `DEALER`: `[message]` or `[empty, message]` 형식

---

## 🧠 메시지 흐름 이해하기

### ROUTER 수신 메시지 구조

```text
[client_id, empty_frame, message_content]
```

- `client_id`: ROUTER가 클라이언트를 구분하기 위한 주소값
- `empty_frame`: 메시지와 주소의 구분자 (필수!)
- `message_content`: 실제 메시지 데이터

➡️ ROUTER는 반드시 `client_id`를 기억하고, 응답 시 **같은 ID로 응답**해야 합니다.

---

## 🔁 비동기 응답 처리

### 예: 여러 요청 처리 후 순서 없이 응답

```python
# ROUTER server 예시
import zmq, time

ctx = zmq.Context()
router = ctx.socket(zmq.ROUTER)
router.bind("tcp://*:5555")

while True:
    identity, _, request = router.recv_multipart()
    print(f"Received from {identity}: {request.decode()}")

    time.sleep(2)  # 처리 지연 시뮬레이션
    router.send_multipart([identity, b'', b'Response after delay'])
```

➡️ 순차 응답이 아닌, **비동기 응답 처리 가능**  
➡️ 여러 클라이언트 요청을 동시에 받아 순서 없이 응답할 수 있습니다.

---

## 📦 Multipart 메시지 활용

- ROUTER/DEALER 패턴은 기본적으로 **multipart 메시지** 기반입니다.
- 메시지에 `메타 정보 + 본문`을 함께 실을 수 있음

```python
# 예시: 메타 + 본문
socket.send_multipart([b'META:userid=1234', b'Hello World'])
```

```python
meta, body = socket.recv_multipart()
print("Meta:", meta.decode())
print("Body:", body.decode())
```

---

## 🛠 커스텀 라우터 응용

> 다음과 같은 응용이 가능해집니다:

- 특정 사용자에게만 응답 보내기
- 요청을 큐에 넣고, 워커에게 분산 처리
- 응답을 라우팅 테이블에 따라 다른 클라이언트에게 보내기
- REQ-REP 구조를 사용하지 않고도 다양한 비동기 프로토콜 설계 가능

---

## 📌 요약 비교

| 패턴           | 구조        | 특징                         | 유스케이스                              |
|----------------|-------------|------------------------------|------------------------------------------|
| REQ-REP        | 1:1         | 동기, 순서 제한               | 간단한 RPC, 기본 요청/응답                |
| DEALER-ROUTER  | N:N         | 완전 비동기, 라우팅 가능       | 라우팅 서버, 메시지 브로커, Load Balancer |

---

## 📝 참고

- `DEALER`와 `ROUTER`는 기본적으로 **비동기** 통신을 염두에 둔 구조입니다.
- `ROUTER`는 클라이언트를 명시적으로 식별하고 직접 응답을 보낼 수 있습니다.
- 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)