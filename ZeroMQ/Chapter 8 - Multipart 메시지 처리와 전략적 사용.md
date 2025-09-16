
# ✅ Chapter 8: Multipart 메시지 처리와 전략적 사용

> ZeroMQ에서는 하나의 메시지를 여러 파트(part)로 나누어 전송할 수 있습니다.  
> 이 기능은 헤더/바디 분리, 프레임 기반 전송, 메타데이터 분리에 유용하며, ROUTER/DEALER 구조에서는 필수적입니다.

---

## 📌 다룰 내용

- Multipart 메시지란?
- `send_multipart()` / `recv_multipart()` 사용법
- ROUTER/DEALER와 multipart 구조
- 메시지 프레임 설계 전략
- 실전 응용 예제

---

## 📦 Multipart 메시지란?

> 여러 개의 메시지 프레임을 하나의 논리적인 메시지로 구성한 것

### 🧠 특징

- 각 파트는 `bytes` 객체
- 순서가 유지됨
- 전송 중 일부만 수신할 수는 없음 → 전체 수신 필요
- ROUTER 소켓은 multipart 구조로 클라이언트를 식별

---

## 🧪 기본 예제: multipart 송수신

```python
import zmq

ctx = zmq.Context()

# 송신
sender = ctx.socket(zmq.PUSH)
sender.bind("tcp://*:5555")

# 수신
receiver = ctx.socket(zmq.PULL)
receiver.connect("tcp://localhost:5555")

# 전송
sender.send_multipart([b"Header", b"Body", b"Footer"])

# 수신
parts = receiver.recv_multipart()
for i, part in enumerate(parts):
    print(f"Part {i+1}: {part.decode()}")
```

---

## 🧭 ROUTER 소켓에서의 multipart 구조

```text
[Client ID][Empty Frame][Message Body]
```

- ROUTER는 클라이언트를 직접 식별해야 하므로 **ID를 메시지 앞에 붙임**
- 중간에 `b''` (empty frame) 이 있어야 ROUTER가 경계 인식 가능

---

## 📪 ROUTER 메시지 수신 예시

```python
msg = router.recv_multipart()
client_id, empty, body = msg
```

- `client_id`: ROUTER가 자동으로 붙이는 클라이언트 식별자
- `empty`: 반드시 존재해야 함 (`b''`)
- `body`: 실제 메시지 데이터

---

## 📤 ROUTER → 특정 클라이언트로 응답하기

```python
router.send_multipart([client_id, b'', b'Response to you'])
```

- **반드시 동일한 client_id 사용**  
- 순서도 반드시 `[ID, b'', message]`

---

## 🧪 DEALER 예제: multipart 응답

```python
dealer = ctx.socket(zmq.DEALER)
dealer.connect("tcp://localhost:5555")

dealer.send(b"Hello")

reply = dealer.recv_multipart()
print("Reply:", reply)
```

- DEALER는 ID가 자동 삽입되므로 단순 메시지 전송으로 처리 가능
- ROUTER가 이를 수신하면 multipart 형태로 ID + 메시지로 분리됨

---

## 🎯 multipart 메시지 응용 시나리오

| 시나리오                | 예시 구조                                |
|-------------------------|-------------------------------------------|
| 헤더 + 본문 분리         | `[b"header", b"body"]`                   |
| 사용자 ID + 요청 데이터  | `[b"user123", b"get:/profile"]`          |
| 이미지 데이터 + 라벨     | `[b"image_bytes", b"class:cat"]`         |
| 인증 토큰 + 메시지       | `[b"token:abc123", b"payload"]`          |

---

## ❗ 주의할 점

- 모든 프레임은 전부 전송되거나 전부 실패 → 부분 전송 없음
- 수신자는 항상 `.recv_multipart()`를 사용해야 함
- 전송한 리스트를 재사용해서 다시 보내면 안 됨 → 새로 만들 것

---

## 🧪 고급 예제: 헤더/본문 구조

```python
# Sender
socket.send_multipart([
    b"content-type:json",
    b'{"message": "hello"}'
])

# Receiver
header, body = socket.recv_multipart()
print("Header:", header.decode())
print("Body:", body.decode())
```

---

## 📎 요약

| 항목                | 설명                                             |
|---------------------|--------------------------------------------------|
| Multipart 전송       | `send_multipart([part1, part2, ...])` 사용       |
| Multipart 수신       | `recv_multipart()`로 리스트 형태로 수신          |
| ROUTER 메시지 구조   | `[client_id, b'', message_body]` 형식 필수       |
| 순서 유지            | 파트 순서는 변경 불가                           |
| 부분 수신 불가       | 한 번에 전체 메시지 수신 필요                    |

---

## 📝 참고

- multipart는 ZeroMQ의 중요한 고급 기능으로, 라우팅, 브로커, 메타데이터 전송에 필수적입니다.
- 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)
- pyzmq 문서: [https://pyzmq.readthedocs.io](https://pyzmq.readthedocs.io)