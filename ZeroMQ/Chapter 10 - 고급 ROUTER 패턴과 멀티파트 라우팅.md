
# ✅ Chapter 10: 고급 ROUTER 패턴과 멀티파트 라우팅

> 이 장에서는 ROUTER 소켓의 고급 사용법을 중심으로, 클라이언트 라우팅, 멀티파트 응답, 메시지 ID 처리 전략 등을 다룹니다.

---

## 📌 다룰 내용

- ROUTER의 메시지 흐름 구조
- 클라이언트 식별자(ID)의 역할
- DEALER ↔ ROUTER 메시지 구조 이해
- 멀티파트 라우팅 응용
- 라우팅 테이블 및 수동 라우팅 전략

---

## 🧭 ROUTER 메시지 구조 복습

ROUTER는 수신 메시지를 다음과 같은 **멀티파트 구조**로 전달합니다:

```text
[client_id][empty][body]
```

| 파트        | 설명                                      |
|-------------|-------------------------------------------|
| `client_id` | ROUTER가 자동 추가한 식별자 (bytes 타입)   |
| `empty`     | 메시지 경계 구분용 (항상 존재)            |
| `body`      | 실제 클라이언트가 보낸 메시지 내용         |

---

## 🧪 ROUTER 서버 기본 예제

```python
import zmq

ctx = zmq.Context()
router = ctx.socket(zmq.ROUTER)
router.bind("tcp://*:5555")

while True:
    msg = router.recv_multipart()
    client_id, _, body = msg
    print(f"📨 From {client_id.hex()}: {body.decode()}")

    # 응답도 반드시 client_id 포함 + 빈 프레임 필요
    router.send_multipart([client_id, b'', b'ACK'])
```

---

## 🧪 DEALER 클라이언트 예제

```python
dealer = ctx.socket(zmq.DEALER)
dealer.connect("tcp://localhost:5555")

dealer.send(b"Hello ROUTER")

reply = dealer.recv()
print("✅ Reply:", reply.decode())
```

---

## 🧠 왜 빈 프레임이 필요한가?

> ROUTER는 클라이언트 ID와 실제 메시지의 경계를 구분하기 위해 **중간에 빈 프레임(b'')**을 요구합니다.

```text
[client_id] ← ZeroMQ 내부 식별 ID  
[b'']       ← 메시지 헤더와 바디 구분  
[body]      ← 실제 메시지
```

➡️ 이 형식을 유지하지 않으면 라우팅 오류 발생  
➡️ 응답 메시지에서도 동일한 포맷을 유지해야 클라이언트가 제대로 수신 가능

---

## 🔁 멀티파트 메시지 응답 예제

```python
router.send_multipart([
    client_id, b'',
    b"Header: OK",
    b"Body: response data here"
])
```

➡️ 응답도 3개 이상의 파트로 나눌 수 있음  
➡️ 수신 측에서는 `.recv_multipart()`로 모두 수신 가능

---

## 🧪 클라이언트에서 멀티파트 수신

```python
reply_parts = dealer.recv_multipart()
for part in reply_parts:
    print("💬 Part:", part.decode())
```

---

## 🗂 라우팅 테이블 만들기

> ROUTER는 클라이언트 ID를 직접 관리할 수 있기 때문에 **라우팅 테이블**을 구현할 수 있습니다.

```python
routing_table = {}

# 수신
client_id, _, body = router.recv_multipart()
routing_table["user123"] = client_id  # 사용자 ID와 ROUTER 식별자 매핑

# 응답 시
router.send_multipart([
    routing_table["user123"], b'', b"Hello user123"
])
```

➡️ 클라이언트를 별명(ID)으로 매핑하여 수동 라우팅 가능  
➡️ 중간에 인증이나 정책 필터 삽입도 가능

---

## 🔐 메시지 인증 구조

```text
[client_id][b''][auth_token][body]
```

```python
client_id, _, token, body = router.recv_multipart()
if token == b"valid_token":
    # 인증 성공
    router.send_multipart([client_id, b'', b'Welcome'])
else:
    router.send_multipart([client_id, b'', b'Access Denied'])
```

➡️ ROUTER 레벨에서 메시지를 분석해 인증/필터링 가능

---

## ⚠️ 주의: 클라이언트 ID는 bytes

- ROUTER가 붙이는 `client_id`는 `bytes` 형식이며 사람이 읽을 수 없는 이진 값일 수 있음
- 로그 출력 시 `.hex()` 혹은 `.decode(errors='ignore')` 사용

```python
print("From client:", client_id.hex())
```

---

## 🧪 ROUTER로 여러 클라이언트 핸들링 예제

```python
clients = {}

while True:
    client_id, _, msg = router.recv_multipart()
    print("From", client_id.hex(), ":", msg.decode())

    if msg == b"register":
        user_id = f"user_{len(clients)}"
        clients[user_id] = client_id
        router.send_multipart([client_id, b'', f"Registered as {user_id}".encode()])
    else:
        router.send_multipart([client_id, b'', b"Echo: " + msg])
```

---

## 📎 요약

| 항목                  | 설명                                          |
|-----------------------|-----------------------------------------------|
| ROUTER 메시지 구조     | `[client_id, b'', message]`                   |
| DEALER 응답 구조       | 단일 메시지 or 멀티파트 응답 수신 가능         |
| 빈 프레임 사용 이유     | 클라이언트 식별자와 실제 메시지를 구분하기 위함 |
| 멀티파트 응답         | 응답도 `[id, b'', part1, part2, ...]`로 가능   |
| 라우팅 테이블 전략     | 사용자 ID와 client_id를 매핑하여 직접 라우팅   |

---

## 📝 참고

- ROUTER 소켓은 ZeroMQ에서 가장 강력한 소켓입니다.  
  인증, 라우팅, 멀티파트 제어 등 고급 기능이 모두 포함됩니다.
- 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)