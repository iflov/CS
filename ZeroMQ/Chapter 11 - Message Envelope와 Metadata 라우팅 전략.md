
# ✅ Chapter 11: Message Envelope와 Metadata 라우팅 전략

> 이 장에서는 ZeroMQ의 메시지를 헤더(Envelope)와 바디로 분리하여 전송하고,  
> 이를 기반으로 **라우팅, 인증, 필터링, 트래킹** 같은 기능을 설계하는 방법을 설명합니다.

---

## 📌 다룰 내용

- Message Envelope 개념
- Header + Body 프레임 분리
- 커스텀 메타데이터 설계
- 메시지 경로 추적
- Envelope 기반 라우팅 전략
- ROUTER/DEALER에서의 적용

---

## ✉️ Envelope란?

> 메시지를 전송할 때, **메타 정보(Envelope)**와 **실제 데이터(Body)**를  
> 서로 다른 프레임으로 구분해서 전송하는 구조입니다.

### 🧠 구조 예시

```text
[client_id][empty][header][body]
```

| 프레임 번호 | 내용                          |
|-------------|-------------------------------|
| 0           | client_id (ROUTER가 붙임)     |
| 1           | 빈 프레임 (구분자)             |
| 2           | header (메타데이터, JSON 등)  |
| 3           | body (실제 메시지 내용)        |

---

## 🧪 Envelope 기반 수신 예제

```python
client_id, _, header, body = router.recv_multipart()
print("헤더:", header.decode())
print("본문:", body.decode())
```

### 💡 설명

- header는 JSON, XML, 혹은 key-value string 등으로 설계 가능
- body는 실제 전송 데이터
- header에서 요청 타입, 인증 정보, 사용자 ID 등을 추출하여 분기 처리 가능

---

## 🛠 커스텀 헤더 설계 예시

```python
import json

header_data = {
    "user_id": "u123",
    "auth": "token123",
    "type": "ping"
}
header_bytes = json.dumps(header_data).encode()

router.send_multipart([
    client_id, b'', header_bytes, b''
])
```

➡️ 수신 측에서는 `json.loads()`로 파싱하여 라우팅이나 필터링에 활용 가능

---

## 🧪 ROUTER에서 header 기반 분기

```python
import json

client_id, _, header, body = router.recv_multipart()
info = json.loads(header.decode())

if info["type"] == "ping":
    router.send_multipart([client_id, b'', b'PONG'])
elif info["type"] == "echo":
    router.send_multipart([client_id, b'', body])
else:
    router.send_multipart([client_id, b'', b'Unknown request'])
```

---

## 🔁 Envelope 활용 예시 시나리오

| 시나리오                 | Header에 포함될 항목             |
|--------------------------|----------------------------------|
| 인증                      | `token`, `user_id`               |
| 라우팅 결정               | `dest`, `group`, `topic`         |
| 메시지 유형 구분          | `type`, `command`                |
| 메시지 추적               | `trace_id`, `timestamp`          |
| QoS 수준 지정             | `priority`, `retries`            |

---

## 📤 클라이언트에서 메시지 구성

```python
import json

dealer = ctx.socket(zmq.DEALER)
dealer.connect("tcp://localhost:5555")

header = json.dumps({
    "type": "echo",
    "user_id": "u456"
}).encode()

body = b"Hello"

dealer.send_multipart([header, body])
```

---

## 🛂 Envelope 기반 인증 처리 예시

```python
client_id, _, header, body = router.recv_multipart()
meta = json.loads(header.decode())

if meta["auth"] != "valid_token":
    router.send_multipart([client_id, b'', b'Auth Failed'])
else:
    router.send_multipart([client_id, b'', b'Welcome!'])
```

➡️ 인증 토큰을 헤더에서 읽고, 인증 실패 시 즉시 응답  
➡️ 본문(body)을 확인하지 않고도 메시지를 필터링할 수 있음

---

## 🧪 멀티파트 메시지 라우팅 전체 예제

```python
# 송신 측
header = json.dumps({
    "type": "file",
    "filetype": "png"
}).encode()
body = open("image.png", "rb").read()

dealer.send_multipart([header, body])
```

```python
# 수신 측 (ROUTER)
client_id, _, header, body = router.recv_multipart()
info = json.loads(header.decode())

if info["filetype"] == "png":
    with open("received.png", "wb") as f:
        f.write(body)
```

---

## 🧵 라우팅 전략 요약

| 전략                        | 설명                                          |
|-----------------------------|-----------------------------------------------|
| 사용자 기반 라우팅           | header에 user_id 포함 → 라우팅 테이블 매핑     |
| 유형 기반 핸들링             | type 필드로 메시지 처리 분기                  |
| 필터링/정책 적용             | header 값 기반으로 허용 여부 판단             |
| 응답 ID 관리                | trace_id 등을 이용해 응답과 요청을 매칭        |

---

## 📎 요약

| 항목               | 설명                                              |
|--------------------|---------------------------------------------------|
| Envelope 구조       | `[ID, b'', header, body]`                         |
| header 포맷        | JSON 또는 key-value string                        |
| 라우팅/필터링 활용 | header 분석을 통해 라우팅/정책 적용 가능          |
| 멀티파트 유지       | 순서대로 전송, 전부 수신 또는 전부 실패           |
| client_id 주의      | ROUTER가 자동 추가, 응답 시 반드시 동일하게 포함  |

---

## 📝 참고

- envelope 처리 패턴은 `REST API`, `RPC`, `메시지 브로커`와 유사한 구조를 구현할 수 있게 합니다.
- 공식 문서: [https://zguide.zeromq.org](https://zguide.zeromq.org)