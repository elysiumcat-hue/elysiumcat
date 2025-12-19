# HTTP 프로토콜 완벽 가이드

## 목차
1. [HTTP 기초](#http-기초)
2. [HTTP 특징](#http-특징)
3. [HTTP 메시지 구조](#http-메시지-구조)
4. [HTTP 메서드](#http-메서드)
5. [HTTP 상태 코드](#http-상태-코드)
6. [HTTP 헤더](#http-헤더)
7. [HTTP 버전 비교](#http-버전-비교)
8. [HTTPS와 보안](#https와-보안)
9. [HTTP 캐싱](#http-캐싱)
10. [쿠키와 세션](#쿠키와-세션)
11. [CORS와 보안 정책](#cors와-보안-정책)

---

## HTTP 기초

### HTTP란?

**HTTP(HyperText Transfer Protocol)**는 웹에서 데이터를 주고받기 위해 사용되는 통신 프로토콜입니다. 클라이언트(웹 브라우저)와 서버 간의 요청과 응답을 규정하여 웹 페이지의 전송과 표시를 가능하게 합니다.

- 기본 포트: 80
- 평문(Plain text)으로 통신
- 요청(Request)과 응답(Response)으로 구성

---

## HTTP 특징

### 1. 클라이언트-서버 구조 (Client-Server Architecture)

HTTP 프로토콜 이전에는 클라이언트와 서버가 명확하게 구분되지 않았습니다. HTTP의 등장으로 클라이언트가 요청(Request)을 보내면 서버가 응답(Response)을 반환하는 명확한 구조가 확립되었습니다. 이를 통해 각각의 역할에 맞게 기능을 최적화할 수 있게 되었습니다.

### 2. 무상태(Stateless) 프로토콜

HTTP는 무상태 프로토콜입니다. 즉, 서버가 클라이언트의 상태를 보존하지 않습니다.

**장점:**
- 서버가 이전 요청에 대한 정보를 저장할 필요가 없음
- 서버의 자원을 효율적으로 사용 가능
- 확장성이 뛰어남

**단점:**
- 클라이언트가 필요한 상태 정보를 매 요청마다 함께 보내야 함
- 이를 보완하기 위해 쿠키와 세션 사용

### 3. 비연결성(Connectionless)

HTTP는 비연결 프로토콜입니다. 클라이언트가 서버에 요청을 하면, 서버가 응답을 반환한 후 연결을 끊습니다.

**특징:**
- 요청-응답 후 연결 종료
- HTTP/1.1 이후로 Keep-Alive 옵션으로 연결 재사용 가능
- 리소스 절약

---

## HTTP 메시지 구조

### HTTP 요청(Request) 메시지

```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Content-Type: application/json
Accept: text/html, application/xhtml+xml

{요청 본문 데이터}
```

**구성 요소:**

1. **요청 라인(Request Line)**
   - 메서드: GET, POST, PUT, DELETE 등
   - 요청 URI: 요청할 리소스의 경로
   - HTTP 버전: HTTP/1.1, HTTP/2 등

2. **헤더(Headers)**
   - 요청에 대한 추가 정보
   - "이름: 값" 형식
   - 여러 개의 헤더 포함 가능

3. **본문(Body)**
   - 요청에 포함되는 데이터
   - POST, PUT, PATCH 요청에 주로 사용
   - GET 요청에는 일반적으로 본문 없음

### HTTP 응답(Response) 메시지

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 1024
Cache-Control: max-age=3600

<html>응답 본문 데이터</html>
```

**구성 요소:**

1. **상태 라인(Status Line)**
   - HTTP 버전
   - 상태 코드: 3자리 숫자 (200, 404, 500 등)
   - 상태 메시지: 상태 코드 설명 (OK, Not Found 등)

2. **헤더(Headers)**
   - 응답에 대한 메타 정보
   - Content-Type, Content-Length, Set-Cookie 등

3. **본문(Body)**
   - 응답 데이터 (HTML, JSON, 이미지 등)

---

## HTTP 메서드

HTTP 메서드는 서버에 수행해야 할 동작을 지정합니다.

### 주요 메서드

| 메서드 | 목적 | 멱등성 | 안전성 | 바디 |
|--------|------|--------|--------|------|
| GET | 리소스 조회 | O | O | X |
| POST | 리소스 생성 | X | X | O |
| PUT | 리소스 완전 교체 | O | X | O |
| PATCH | 리소스 부분 수정 | X | X | O |
| DELETE | 리소스 삭제 | O | X | X |
| HEAD | GET과 동일하나 바디 없음 | O | O | X |
| OPTIONS | 통신 옵션 확인 | O | O | X |

### 메서드 상세 설명

#### GET
리소스를 조회합니다. 데이터를 가져올 때만 사용하며, 서버의 상태를 변경하지 않습니다.

```
GET /users/123 HTTP/1.1
Host: api.example.com
```

#### POST
새로운 리소스를 생성합니다. 요청 바디에 데이터를 담아 전송합니다.

```
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "John",
  "email": "john@example.com"
}
```

#### PUT
리소스를 완전히 교체합니다. 기존 데이터 전체를 새로운 데이터로 대체합니다.

```
PUT /users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Jane",
  "email": "jane@example.com"
}
```

#### PATCH
리소스를 부분적으로 수정합니다. 요청에 포함된 필드만 업데이트됩니다.

```
PATCH /users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "email": "newemail@example.com"
}
```

기존 데이터가 `{"name": "John", "email": "john@example.com"}`이었다면, 수정 후에는 `{"name": "John", "email": "newemail@example.com"}`이 됩니다.

#### DELETE
리소스를 삭제합니다.

```
DELETE /users/123 HTTP/1.1
Host: api.example.com
```

### 멱등성(Idempotency)과 안전성(Safety)

**멱등성:**
- 같은 요청을 여러 번 보내도 최종 결과가 동일한 성질
- GET, PUT, DELETE, HEAD, OPTIONS는 멱등성 있음
- POST, PATCH는 멱등성 없음

**안전성:**
- 요청을 보낼 때 서버의 상태를 변경하지 않는 성질
- GET, HEAD, OPTIONS, TRACE는 안전함
- POST, PUT, PATCH, DELETE는 안전하지 않음

---

## HTTP 상태 코드

상태 코드는 클라이언트의 요청에 대한 처리 결과를 나타내는 3자리 숫자입니다.

### 1xx (정보)
요청이 수신되어 처리 중인 상태입니다.

- **100 Continue**: 요청을 받았으며 계속 진행하라는 의미
- **101 Switching Protocols**: 프로토콜 업그레이드

### 2xx (성공)
요청이 성공적으로 처리되었습니다.

- **200 OK**: 요청 성공, 가장 일반적인 응답
- **201 Created**: 리소스 생성 성공 (POST 요청)
- **202 Accepted**: 요청 접수했으나 처리 진행 중
- **204 No Content**: 성공하나 응답 본문 없음
- **206 Partial Content**: 범위 요청의 일부 전송

### 3xx (리다이렉션)
요청을 완료하기 위해 추가 조치가 필요합니다.

- **300 Multiple Choices**: 여러 선택지 제공
- **301 Moved Permanently**: 영구적 이동, 새 URL로 자동 이동
- **302 Found**: 임시 이동
- **304 Not Modified**: 캐시된 버전 사용 가능
- **307 Temporary Redirect**: 임시 리다이렉트 (메서드 유지)
- **308 Permanent Redirect**: 영구 리다이렉트 (메서드 유지)

### 4xx (클라이언트 오류)
클라이언트의 요청에 문제가 있습니다.

- **400 Bad Request**: 잘못된 요청 문법
- **401 Unauthorized**: 인증 필요
- **403 Forbidden**: 접근 권한 없음
- **404 Not Found**: 요청한 리소스 없음
- **405 Method Not Allowed**: 허용되지 않은 메서드
- **409 Conflict**: 요청이 현재 서버 상태와 충돌
- **429 Too Many Requests**: 요청 제한 초과

### 5xx (서버 오류)
서버에서 요청 처리에 실패했습니다.

- **500 Internal Server Error**: 서버 내부 오류
- **501 Not Implemented**: 구현되지 않은 기능
- **502 Bad Gateway**: 게이트웨이 오류
- **503 Service Unavailable**: 서비스 이용 불가 (점검 중)
- **504 Gateway Timeout**: 게이트웨이 타임아웃

---

## HTTP 헤더

HTTP 헤더는 요청 또는 응답에 대한 메타 정보를 담습니다.

### 공통 헤더 (요청/응답 모두 사용)

| 헤더 | 설명 | 예시 |
|------|------|------|
| Content-Type | 메시지 바디의 미디어 타입 | `application/json; charset=utf-8` |
| Content-Length | 메시지 바디의 길이(바이트) | `1024` |
| Content-Encoding | 바디 압축 방식 | `gzip`, `deflate` |
| Cache-Control | 캐시 제어 | `max-age=3600, public` |
| Expires | 리소스 만료 시간 | `Wed, 21 Oct 2025 07:28:00 GMT` |

### 요청 헤더 (Request Headers)

| 헤더 | 설명 | 예시 |
|------|------|------|
| Host | 요청 대상 호스트 | `www.example.com` |
| User-Agent | 요청 클라이언트 정보 | `Mozilla/5.0 (Windows NT 10.0)` |
| Accept | 허용 가능한 콘텐츠 타입 | `text/html, application/json` |
| Accept-Language | 선호하는 언어 | `ko-KR, ko; q=0.9` |
| Accept-Encoding | 허용 가능한 압축 방식 | `gzip, deflate` |
| Authorization | 인증 정보 | `Bearer token123456` |
| Referer | 이전 페이지 URL | `https://example.com/page1` |
| Cookie | 클라이언트 쿠키 | `sessionId=abc123; userId=456` |
| If-Modified-Since | 캐시 검증용 시간 | `Wed, 21 Oct 2025 07:28:00 GMT` |
| If-None-Match | 캐시 검증용 ETag | `"33a64df551425fcc55e4d42a148795d9f25f89d4"` |

### 응답 헤더 (Response Headers)

| 헤더 | 설명 | 예시 |
|------|------|------|
| Server | 서버 정보 | `Apache/2.4.41 (Ubuntu)` |
| Set-Cookie | 클라이언트에 쿠키 설정 | `sessionId=abc123; Path=/; HttpOnly` |
| Location | 리다이렉트 URL | `https://example.com/newpage` |
| Last-Modified | 리소스 마지막 수정 시간 | `Wed, 21 Oct 2025 07:28:00 GMT` |
| ETag | 리소스 버전 식별자 | `"33a64df551425fcc55e4d42a148795d9f25f89d4"` |
| Access-Control-Allow-Origin | CORS 허용 출처 | `https://client.example.com` |
| Access-Control-Allow-Methods | CORS 허용 메서드 | `GET, POST, PUT` |
| Vary | 캐시 구분 기준 | `Content-Encoding, Accept-Language` |

### Content-Type MIME 타입

| 타입 | 설명 | 예시 |
|------|------|------|
| text | 텍스트 파일 | `text/html`, `text/plain`, `text/css` |
| application | 애플리케이션 파일 | `application/json`, `application/xml` |
| image | 이미지 파일 | `image/jpeg`, `image/png`, `image/gif` |
| audio | 오디오 파일 | `audio/mpeg`, `audio/wav` |
| video | 비디오 파일 | `video/mp4`, `video/webm` |

---

## HTTP 버전 비교

### HTTP/1.1 (현재 가장 많이 사용)

**특징:**
- TCP 연결 기반
- Keep-Alive로 연결 재사용 가능
- 요청/응답 순서대로 처리 (직렬 처리)

**문제점:**
- **HOLB(Head-of-Line Blocking)**: 첫 번째 요청이 느리면 후속 요청들이 대기
- 한 번에 하나의 요청만 처리
- 많은 리소스 요청 시 성능 저하

**장점:**
- 광범위한 호환성
- 디버깅 용이
- 단순한 구조

### HTTP/2

**주요 개선사항:**
- **멀티플렉싱 (Multiplexing)**: 단일 TCP 연결에서 여러 요청/응답을 동시 처리
- **서버 푸시 (Server Push)**: 서버가 먼저 리소스 전송 가능
- **헤더 압축 (HPACK)**: 반복되는 헤더 압축으로 대역폭 절감
- **바이너리 프레이밍**: 바이너리 형식으로 더 효율적인 처리

**장점:**
- 성능 향상 (HTTP/1.1보다 빠름)
- HOLB 문제 해결
- 데이터 효율성 증가

**단점:**
- 구현 복잡도 증가
- 디버깅 어려움
- HTTP/1.1보다 호환성 낮음

### HTTP/3

**기술 기반:**
- QUIC 프로토콜 사용 (TCP 대신 UDP 기반)
- 최신 프로토콜 표준

**주요 특징:**
- **0-RTT (Zero Round Trip Time)**: 캐시된 정보로 즉시 연결 수립
- **연결 ID 기반**: IP 주소 변경해도 연결 유지 (Wi-Fi↔5G 전환)
- **독립적 스트림**: 한 스트림의 패킷 손실이 다른 스트림에 영향 없음
- **연결 다중화 지원**: HTTP/2처럼 동시 요청 처리

**장점:**
- 가장 빠른 성능
- 모바일 환경에서 우수한 성능
- 완전한 HOLB 해결

**단점:**
- 가장 적은 호환성
- 지원 브라우저 제한적
- 구현 복잡도 가장 높음

### 버전 비교표

| 항목 | HTTP/1.1 | HTTP/2 | HTTP/3 |
|------|----------|--------|--------|
| 전송 프로토콜 | TCP | TCP | UDP (QUIC) |
| 다중화 | X | O | O |
| 연결 유지 | Keep-Alive | 자동 | 자동 |
| 압축 | X | HPACK | HPACK |
| 속도 | 느림 | 빠름 | 가장 빠름 |
| 호환성 | 매우 높음 | 높음 | 낮음 |
| HOLB 문제 | O | X | X |

---

## HTTPS와 보안

### HTTP의 보안 문제

HTTP는 평문으로 통신하기 때문에 다음과 같은 보안 문제가 있습니다:

- **스니핑(Sniffing)**: 통신 내용을 누구나 볼 수 있음
- **패킷 변조**: 중간에 패킷을 가로채 내용 수정 가능
- **신원 위장**: 서버가 진짜 서버인지 확인 불가능

### HTTPS (HTTP over SSL/TLS)

**HTTPS**는 HTTP에 SSL(Secure Sockets Layer) 또는 TLS(Transport Layer Security) 암호화 프로토콜을 추가한 보안 버전입니다.

- 기본 포트: 443
- 모든 데이터가 암호화됨
- 서버 신원 확인 가능

### SSL/TLS 핸드셰이크 과정

1. **Client Hello**: 클라이언트가 서버에 연결 요청
   - 지원 가능한 암호화 방식 전송
   - 클라이언트 버전 정보 전송

2. **Server Hello**: 서버가 응답
   - 사용할 암호화 방식 선택
   - SSL 인증서 전송

3. **인증서 검증**: 클라이언트가 인증서 검증
   - 인증 기관(CA)의 공개키로 서명 확인
   - 서버 신원 확인

4. **키 교환**: 세션 암호화 키 교환
   - 클라이언트가 대칭키 생성
   - 서버의 공개키로 암호화해 전송
   - 서버가 비밀키로 복호화

5. **통신 시작**: 대칭키로 암호화된 통신 시작

### SSL 인증서

**구성 요소:**
- 서버의 공개키
- 서버 정보 (도메인, 조직)
- 발급 기관(CA) 정보
- 유효 기간
- CA의 디지털 서명

**발급 과정:**
1. 서버가 공개키와 개인키 생성
2. CA에 인증서 발급 요청
3. CA가 서버 정보 검증
4. CA가 자신의 비밀키로 인증서 서명
5. 서명된 인증서를 서버에 전달

### SSL vs TLS

| 항목 | SSL | TLS |
|------|-----|-----|
| 메시지 인증 | MAC (MD5) | HMAC (SHA) |
| 암호화 | 구식 알고리즘 | 고급 알고리즘 |
| 핸드셰이크 | 복잡하고 느림 | 단계 적고 빠름 |
| 보안 취약점 | O | X |
| 현재 사용 | X | O |

현재는 SSL이 더 이상 사용되지 않으며, TLS가 표준입니다. 하지만 관례적으로 "SSL 인증서"라고 부르기도 합니다.

---

## HTTP 캐싱

HTTP 캐싱은 이전에 다운로드한 리소스를 로컬에 저장하여 재사용함으로써 성능을 향상시킵니다.

### 캐시의 이점

- **대역폭 절감**: 서버에서 반복 전송 불필요
- **레이턴시 감소**: 로컬 캐시에서 빠르게 로드
- **서버 부하 감소**: 네트워크 트래픽 감소

### 캐시 유형

#### 1. 사설 캐시 (Private Cache)
- 단일 사용자만 사용
- 브라우저 캐시
- 보안 정보 저장 가능

#### 2. 공유 캐시 (Shared Cache)
- 여러 사용자가 공유
- 프록시 서버 캐시, CDN
- 공개 정보만 저장

### 캐시 제어

#### Cache-Control 헤더

서버는 응답 헤더에 `Cache-Control`을 설정하여 캐시 동작을 제어합니다.

```
Cache-Control: max-age=3600, public, must-revalidate
```

**주요 지시자:**

| 지시자 | 설명 |
|--------|------|
| `max-age=초` | 캐시 유효 시간 (초 단위) |
| `public` | 공유 캐시에 저장 가능 |
| `private` | 사설 캐시에만 저장 |
| `no-cache` | 캐시 사용 전 서버 검증 필수 |
| `no-store` | 캐시 저장 금지 (민감한 데이터) |
| `must-revalidate` | 만료 후 반드시 서버 검증 |
| `immutable` | 변경 불가능한 리소스 |

### 캐시 검증 (Revalidation)

#### ETag 방식

서버가 리소스마다 고유한 버전 식별자(ETag)를 제공합니다.

**요청:**
```
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

**응답:**
- ETag 일치 → `304 Not Modified` (캐시 사용)
- ETag 불일치 → `200 OK` + 새로운 리소스

#### Last-Modified 방식

서버가 리소스의 마지막 수정 시간을 제공합니다.

**요청:**
```
If-Modified-Since: Wed, 21 Oct 2025 07:28:00 GMT
```

**응답:**
- 수정 시간 이후 변경 없음 → `304 Not Modified`
- 변경됨 → `200 OK` + 새로운 리소스

### 캐시 최적화 기법

#### 1. 파일명 해싱 (Revving)

빌드 시 파일 내용의 해시값을 파일명에 포함시킵니다.

```javascript
// webpack 설정
output: {
  filename: "[name][contenthash].[ext]"
}
```

결과:
- `main.a1b2c3d4.js` (변경되면 새 파일명 생성)
- 오래된 캐시와 구분되어 항상 최신 버전 사용

#### 2. Vary 헤더

같은 URL이라도 다른 요청 조건에 따라 다른 리소스를 제공할 때 사용합니다.

```
Vary: Accept-Encoding, Accept-Language
```

캐시는 다음을 기준으로 저장됩니다:
- URL: `/api/content`
- Accept-Encoding: `gzip`
- Accept-Language: `ko-KR`

### 캐시 동작 흐름

```
1. 브라우저가 리소스 요청
   ↓
2. 캐시에 존재 & 유효 → 캐시 반환 (Hit)
   ↓
3. 캐시 없거나 만료 → 서버에 요청 (Miss)
   ↓
4. 서버가 304 또는 200으로 응답
   ↓
5. 브라우저가 캐시 업데이트
```

---

## 쿠키와 세션

HTTP는 무상태 프로토콜이므로 클라이언트 상태를 저장하지 않습니다. 이를 보완하기 위해 쿠키와 세션을 사용합니다.

### 쿠키 (Cookie)

**정의:** 클라이언트(브라우저) 로컬에 저장되는 작은 텍스트 파일

**특징:**
- 클라이언트에 저장
- 자동으로 모든 요청에 포함
- 최대 4KB 크기 제한
- 보안성 낮음

**동작 순서:**

1. 클라이언트가 서버에 페이지 요청
2. 서버가 `Set-Cookie` 헤더로 쿠키 전송
3. 클라이언트가 쿠키를 로컬에 저장
4. 이후 동일 도메인의 모든 요청에 쿠키 자동 포함
5. 서버가 쿠키로 클라이언트 확인

**예시:**

```
// 응답 헤더
Set-Cookie: sessionId=abc123; Path=/; Expires=Wed, 21 Oct 2025 07:28:00 GMT; HttpOnly

// 이후 요청 헤더
Cookie: sessionId=abc123
```

**쿠키 속성:**

| 속성 | 설명 |
|------|------|
| `Path` | 쿠키 적용 경로 |
| `Domain` | 쿠키 적용 도메인 |
| `Expires` | 쿠키 만료 시간 |
| `Max-Age` | 쿠키 유효 시간 (초) |
| `Secure` | HTTPS에서만 전송 |
| `HttpOnly` | JavaScript에서 접근 불가 (보안) |
| `SameSite` | CSRF 공격 방지 (Strict/Lax/None) |

### 세션 (Session)

**정의:** 서버에서 관리하는 사용자 상태 정보

**특징:**
- 서버에 저장
- 세션 ID만 클라이언트에 쿠키로 저장
- 크기 제한 없음
- 보안성 높음
- 서버 자원 사용

**동작 순서:**

1. 클라이언트가 서버에 첫 요청
2. 서버가 세션 ID 생성
3. 서버가 세션 ID를 쿠키에 담아 응답 (JSESSIONID)
4. 클라이언트가 세션 ID 쿠키 저장
5. 이후 요청마다 세션 ID 전송
6. 서버가 세션 ID로 사용자 정보 조회

**예시:**

```
// 응답 헤더
Set-Cookie: JSESSIONID=abc123def456; Path=/; HttpOnly

// 서버의 세션 저장소
{
  "abc123def456": {
    "userId": 1,
    "username": "john",
    "loginTime": "2025-12-19T07:57:00Z",
    "lastAccess": "2025-12-19T08:00:00Z"
  }
}
```

### 쿠키 vs 세션

| 항목 | 쿠키 | 세션 |
|------|------|------|
| 저장 위치 | 클라이언트 | 서버 |
| 저장 정보 | 실제 데이터 | 세션 ID만 |
| 보안 | 낮음 | 높음 |
| 서버 자원 | 사용 안 함 | 사용 |
| 속도 | 빠름 | 느림 |
| 용량 | 4KB 제한 | 무제한 |
| 브라우저 종료 | 설정에 따라 유지 | 일반적으로 삭제 |

### 캐시 (Cache) vs 쿠키 vs 세션

| 항목 | 캐시 | 쿠키 | 세션 |
|------|------|------|------|
| 목적 | 성능 향상 | 사용자 식별 | 상태 저장 |
| 저장 위치 | 클라이언트 | 클라이언트 | 서버 |
| 주체 | 브라우저/서버 | 브라우저 | 서버 |
| 자동 전송 | X | O | O (ID만) |
| 사용 예 | 이미지, CSS, JS | 사용자 선호도 | 로그인 정보 |

---

## CORS와 보안 정책

### 동일 출처 정책 (SOP: Same-Origin Policy)

**정의:** 브라우저의 보안 정책으로, 같은 출처에서만 리소스를 공유할 수 있도록 제한합니다.

**출처 (Origin) 구성:**

```
https://example.com:8080/page
│       │           │  │
│       │           │  └─ 포트 (Port)
│       │           └───── 호스트 (Host)
│       └───────────────── 도메인 (Domain)
└───────────────────────── 프로토콜 (Protocol)
```

**같은 출처 판단:**
- 프로토콜, 호스트, 포트가 모두 동일해야 함

**예시:**

| 비교 대상 | 같은 출처? | 이유 |
|----------|-----------|------|
| `https://example.com` | O | 완전히 동일 |
| `http://example.com` | X | 프로토콜 다름 |
| `https://www.example.com` | X | 호스트 다름 |
| `https://example.com:8080` | X | 포트 다름 |
| `https://example.com/page` | O | 경로는 상관없음 |

### SOP가 제한하는 것

- **읽기 제한 (Read)**: 다른 출처 리소스 내용 읽기 불가
- **쓰기는 가능 (Write)**: 폼 제출, 링크 클릭은 가능
- **임베딩은 가능 (Embedding)**: `<img>`, `<script>`, `<style>` 등

### CORS (Cross-Origin Resource Sharing)

**정의:** SOP 정책의 예외로, 서로 다른 출처 간의 리소스 공유를 안전하게 허용하는 메커니즘

**동작 원리:**

서버가 응답 헤더에 `Access-Control-Allow-Origin`을 설정하여 특정 출처의 접근을 허용합니다.

```
// 서버 응답 헤더
Access-Control-Allow-Origin: https://client.example.com
```

브라우저는 요청의 `Origin` 헤더값과 응답의 `Access-Control-Allow-Origin`값을 비교하여 CORS 정책 위반 여부를 판단합니다.

### CORS 요청 유형

#### 1. Simple Request (단순 요청)

다음 조건을 모두 만족하면 단순 요청입니다:

**메서드:**
- GET, HEAD, POST만 가능

**헤더:**
- Accept, Accept-Language, Content-Language, Content-Type만 가능

**Content-Type:**
- `application/x-www-form-urlencoded`
- `multipart/form-data`
- `text/plain`

**동작:**
```
클라이언트 요청
Origin: https://client.example.com

서버 응답
Access-Control-Allow-Origin: https://client.example.com
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type
```

#### 2. Preflight Request (사전 요청)

단순 요청 조건을 벗어나면 자동으로 OPTIONS 사전 요청을 먼저 보냅니다.

**사전 요청이 필요한 경우:**
- PUT, PATCH, DELETE 메서드
- Authorization, X-Custom-Header 등 커스텀 헤더
- `application/json` Content-Type

**동작 흐름:**

```
1. 클라이언트 프리플라이트 요청 (OPTIONS)
   OPTIONS /api/resource HTTP/1.1
   Origin: https://client.example.com
   Access-Control-Request-Method: POST
   Access-Control-Request-Headers: Content-Type, Authorization

2. 서버 프리플라이트 응답
   HTTP/1.1 200 OK
   Access-Control-Allow-Origin: https://client.example.com
   Access-Control-Allow-Methods: GET, POST, PUT, DELETE
   Access-Control-Allow-Headers: Content-Type, Authorization
   Access-Control-Max-Age: 3600

3. 클라이언트 실제 요청
   POST /api/resource HTTP/1.1
   Origin: https://client.example.com
   Content-Type: application/json
   Authorization: Bearer token123

4. 서버 실제 응답
   HTTP/1.1 200 OK
   Access-Control-Allow-Origin: https://client.example.com
   ...
```

### CORS 관련 응답 헤더

| 헤더 | 설명 | 예시 |
|------|------|------|
| `Access-Control-Allow-Origin` | 접근 허용 출처 | `https://client.example.com` 또는 `*` |
| `Access-Control-Allow-Methods` | 허용 HTTP 메서드 | `GET, POST, PUT, DELETE` |
| `Access-Control-Allow-Headers` | 허용 요청 헤더 | `Content-Type, Authorization` |
| `Access-Control-Allow-Credentials` | 쿠키/인증 정보 포함 | `true` |
| `Access-Control-Max-Age` | 프리플라이트 캐시 시간 (초) | `3600` |
| `Access-Control-Expose-Headers` | 클라이언트에 노출할 헤더 | `X-Total-Count` |

### 인증 정보 포함 (Credentials)

CORS 요청에서 쿠키나 인증 헤더를 포함하려면 특별한 설정이 필요합니다.

**클라이언트 요청:**
```javascript
fetch('https://api.example.com/data', {
  credentials: 'include', // 쿠키 포함
  headers: {
    'Authorization': 'Bearer token123'
  }
})
```

**서버 응답:**
```
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: https://client.example.com (와일드카드 * 사용 불가)
```

### CORS 오류 해결 방법

#### 1. 서버에서 CORS 설정

```javascript
// Node.js/Express
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', 'https://client.example.com');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  next();
});
```

#### 2. 프록시 서버 사용

개발 환경에서 프록시를 통해 같은 출처로 요청을 우회합니다.

```javascript
// 개발 환경 설정
devServer: {
  proxy: {
    '/api': {
      target: 'https://api.example.com',
      changeOrigin: true
    }
  }
}
```

#### 3. JSONP 사용 (레거시)

GET 요청만 가능하며 보안이 낮습니다. 현대적 방법은 CORS 사용을 권장합니다.

---

## HTTP 실전 예제

### 예제 1: REST API 호출

```javascript
// GET 요청 - 사용자 조회
fetch('https://api.example.com/users/123', {
  method: 'GET',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer eyJhbGc...'
  }
})
.then(response => {
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  return response.json();
})
.then(data => console.log(data))
.catch(error => console.error('Error:', error));

// POST 요청 - 새 사용자 생성
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer eyJhbGc...'
  },
  body: JSON.stringify({
    name: 'John Doe',
    email: 'john@example.com'
  })
})
.then(response => response.json())
.then(data => console.log('Created:', data))
.catch(error => console.error('Error:', error));

// PUT 요청 - 사용자 완전 업데이트
fetch('https://api.example.com/users/123', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'Jane Doe',
    email: 'jane@example.com'
  })
})
.then(response => response.json())
.then(data => console.log('Updated:', data));

// PATCH 요청 - 사용자 부분 업데이트
fetch('https://api.example.com/users/123', {
  method: 'PATCH',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    email: 'newemail@example.com'
  })
})
.then(response => response.json())
.then(data => console.log('Partially updated:', data));

// DELETE 요청 - 사용자 삭제
fetch('https://api.example.com/users/123', {
  method: 'DELETE'
})
.then(response => {
  if (response.status === 204) {
    console.log('Deleted successfully');
  }
})
.catch(error => console.error('Error:', error));
```

### 예제 2: 상태 코드 처리

```javascript
async function fetchWithStatusHandling(url) {
  try {
    const response = await fetch(url);
    
    switch (response.status) {
      case 200: // OK
        return await response.json();
      
      case 201: // Created
        console.log('리소스가 생성되었습니다');
        return await response.json();
      
      case 304: // Not Modified
        console.log('캐시된 리소스를 사용합니다');
        return getCachedData(url);
      
      case 400: // Bad Request
        console.error('잘못된 요청입니다');
        throw new Error('요청 형식이 잘못되었습니다');
      
      case 401: // Unauthorized
        console.error('인증이 필요합니다');
        redirectToLogin();
        break;
      
      case 403: // Forbidden
        console.error('접근 권한이 없습니다');
        throw new Error('접근 권한이 없습니다');
      
      case 404: // Not Found
        console.error('리소스를 찾을 수 없습니다');
        throw new Error('요청한 리소스가 없습니다');
      
      case 429: // Too Many Requests
        console.error('너무 많은 요청을 보냈습니다');
        // 재시도 로직
        await sleep(5000);
        return fetchWithStatusHandling(url);
      
      case 500: // Internal Server Error
      case 502: // Bad Gateway
      case 503: // Service Unavailable
        console.error('서버 오류가 발생했습니다');
        throw new Error('서버에 일시적인 문제가 발생했습니다');
      
      default:
        throw new Error(`예상치 못한 상태 코드: ${response.status}`);
    }
  } catch (error) {
    console.error('요청 중 오류 발생:', error);
    throw error;
  }
}
```

---

## 체크리스트

HTTP를 제대로 이해했는지 확인해봅시다.

**기본 개념:**
- [ ] HTTP가 무상태, 비연결 프로토콜인 이유 이해
- [ ] 클라이언트-서버 아키텍처 이해
- [ ] HTTP 메시지 구조(요청/응답) 파악

**메서드와 상태 코드:**
- [ ] GET, POST, PUT, PATCH, DELETE의 차이 이해
- [ ] 멱등성과 안전성 개념 이해
- [ ] 2xx, 3xx, 4xx, 5xx 상태 코드 분류 이해
- [ ] 주요 상태 코드(200, 201, 301, 304, 400, 401, 403, 404, 500) 숙지

**헤더와 본문:**
- [ ] Content-Type, User-Agent, Authorization 헤더 이해
- [ ] 요청 헤더와 응답 헤더의 차이 파악
- [ ] MIME 타입 이해

**버전 비교:**
- [ ] HTTP/1.1, HTTP/2, HTTP/3의 차이 이해
- [ ] 멀티플렉싱, HOLB 개념 이해
- [ ] QUIC 프로토콜 기본 이해

**보안:**
- [ ] HTTPS와 HTTP의 차이 이해
- [ ] SSL/TLS 핸드셰이크 과정 파악
- [ ] 인증서의 역할 이해

**캐싱:**
- [ ] 캐시의 이점 이해
- [ ] Cache-Control 헤더 사용법 숙지
- [ ] ETag와 Last-Modified 방식 이해

**상태 관리:**
- [ ] 쿠키와 세션의 차이 이해
- [ ] 로그인 구현 시 쿠키/세션 활용

**보안 정책:**
- [ ] SOP(Same-Origin Policy) 이해
- [ ] CORS 정책 이해
- [ ] CORS 오류 해결 방법 숙지

---

## 참고 자료

- MDN Web Docs - HTTP 가이드
- RFC 7230-7237 (HTTP/1.1 표준)
- RFC 7540 (HTTP/2 표준)
- RFC 9000 (QUIC 표준)
- OWASP 웹 보안 가이드
