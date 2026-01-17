# 파이썬 웹 크롤링: Cookies & Session 완벽 학습 가이드

## 목차
1. [기초 개념](#기초-개념)
2. [HTTP Cookies 상세 이해](#http-cookies-상세-이해)
3. [Session 원리와 동작](#session-원리와-동작)
4. [Requests 라이브러리 활용](#requests-라이브러리-활용)
5. [실전 로그인 구현](#실전-로그인-구현)
6. [고급 기법](#고급-기법)
7. [보안 고려사항](#보안-고려사항)
8. [FAQ & 트러블슈팅](#faq--트러블슈팅)

---

## 기초 개념

### HTTP의 특성: Stateless & Connectionless

웹 크롤링에서 cookies와 session을 배워야 하는 근본적인 이유를 이해해야 합니다.

**HTTP 프로토콜의 문제점:**
- **Connectionless (비연결성)**: 요청-응답이 완료되면 연결을 끊음
- **Stateless (비상태성)**: 이전 요청의 정보를 저장하지 않음
- **결과**: 매 요청마다 새로운 사용자로 인식됨

예시:
```
요청1: "안녕하세요"
응답1: "안녕하세요"
(연결 종료)

요청2: "안녕하세요" (다시)
응답2: "누구신가요?" (이전 요청을 기억하지 못함)
```

**해결책**: Cookies와 Session을 통해 상태를 유지

### Cookies vs Session 비교표

| 항목 | Cookies | Session |
|------|---------|---------|
| **저장 위치** | 클라이언트(브라우저) | 서버 |
| **보안** | 낮음 (조작 가능) | 높음 (서버 관리) |
| **크기 제한** | ~4KB | 무제한 |
| **유효 기간** | 명시적 설정 가능 | 브라우저 종료 시 삭제 |
| **속도** | 빠름 | 상대적으로 느림 |
| **사용 사례** | 사용자 선호도, 추적 | 로그인, 인증 |

---

## HTTP Cookies 상세 이해

### Cookies의 작동 원리

#### 1단계: 서버가 쿠키 설정
```
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: sessionid=abc123def456
Set-Cookie: username=john
Set-Cookie: theme=dark; Secure; HttpOnly; Path=/;

[페이지 내용]
```

#### 2단계: 클라이언트가 쿠키 저장
브라우저는 `Set-Cookie` 헤더의 데이터를 로컬에 저장합니다.

#### 3단계: 이후 요청에 쿠키 포함
```
GET /dashboard HTTP/1.1
Host: example.com
Cookie: sessionid=abc123def456; username=john; theme=dark
```

### Cookie 속성 상세 설명

```
Set-Cookie: <name>=<value>; <attribute1>; <attribute2>; ...
```

**필수 속성:**
- `Name=Value`: 쿠키 이름과 값 (key-value 형식)

**선택적 속성:**

1. **Domain**: 쿠키를 전송할 도메인
   ```
   Set-Cookie: id=123; Domain=.example.com
   
   # example.com, sub.example.com, api.example.com에 전송
   # other.com에는 전송 안됨
   ```

2. **Path**: 쿠키를 전송할 경로
   ```
   Set-Cookie: user=john; Path=/admin/
   
   # /admin/, /admin/users/, /admin/settings/ → 전송
   # /public/, /api/ → 전송 안됨
   ```

3. **Expires / Max-Age**: 만료 시간
   ```
   # Expires: 특정 날짜
   Set-Cookie: id=123; Expires=Wed, 09 Jun 2025 10:18:14 GMT
   
   # Max-Age: 초 단위 지속 시간 (Expires 보다 우선순위 높음)
   Set-Cookie: id=123; Max-Age=3600  # 1시간
   ```

4. **Secure**: HTTPS에서만 전송
   ```
   Set-Cookie: token=xyz; Secure
   
   # HTTPS에서만 전송, HTTP에서는 전송 안됨
   ```

5. **HttpOnly**: JavaScript에서 접근 불가능 (XSS 방지)
   ```
   Set-Cookie: sessionid=abc; HttpOnly
   
   # document.cookie로 접근 불가능
   ```

6. **SameSite**: CSRF 공격 방지
   ```
   Set-Cookie: id=123; SameSite=Strict
   
   # Strict: 크로스 사이트 요청에서 쿠키 전송 안됨
   # Lax: 특정 상황(링크 클릭 등)에만 전송
   # None: 항상 전송 (Secure와 함께 사용 필수)
   ```

### Python에서 Cookies 다루기

#### 기본: requests 라이브러리

```python
import requests

# 1. 쿠키 가져오기
response = requests.get('https://example.com')
print(response.cookies)  # RequestsCookieJar 객체
print(response.cookies.get_dict())  # 딕셔너리로 변환

# 2. 쿠키 설정해서 요청하기
cookies = {'sessionid': 'abc123', 'username': 'john'}
response = requests.get('https://example.com', cookies=cookies)

# 3. 응답에서 Set-Cookie 헤더 보기
print(response.headers.get('Set-Cookie'))
```

#### 쿠키 저장 및 로드

**JSON으로 저장 (권장):**
```python
import requests
import json

# 쿠키 저장
session = requests.Session()
response = session.get('https://example.com/login')

# 딕셔너리로 변환 후 JSON으로 저장
cookies_dict = session.cookies.get_dict()
with open('cookies.json', 'w') as f:
    json.dump(cookies_dict, f)

# 쿠키 로드
with open('cookies.json', 'r') as f:
    cookies = json.load(f)
    
# 로드한 쿠키로 요청
response = requests.get('https://example.com/dashboard', cookies=cookies)
```

**Pickle로 저장 (복잡한 객체용):**
```python
import requests
import pickle

session = requests.Session()
response = session.get('https://example.com')

# RequestsCookieJar 객체를 pickle로 저장
with open('cookies.pkl', 'wb') as f:
    pickle.dump(session.cookies, f)

# 로드
with open('cookies.pkl', 'rb') as f:
    cookies = pickle.load(f)
    session.cookies.update(cookies)
```

---

## Session 원리와 동작

### Session의 동작 흐름

```
1. 클라이언트 첫 요청 (쿠키 없음)
   GET / HTTP/1.1

2. 서버: 세션ID 생성 후 응답
   HTTP/1.0 200 OK
   Set-Cookie: JSESSIONID=F4A8D3B7C2E9F1A6

3. 클라이언트: 세션ID를 쿠키에 저장
   (브라우저 메모리에 저장)

4. 이후 모든 요청에 세션ID 포함
   GET /dashboard HTTP/1.1
   Cookie: JSESSIONID=F4A8D3B7C2E9F1A6

5. 서버: 세션ID로 사용자 식별
   session_data = session_store['F4A8D3B7C2E9F1A6']
   # → {username: 'john', logged_in: true, ...}
```

### 서버 측 Session 저장소

**메모리 저장 (개발용):**
```python
session_store = {}

def create_session(user_id):
    import uuid
    session_id = str(uuid.uuid4())
    session_store[session_id] = {
        'user_id': user_id,
        'logged_in': True,
        'created_at': time.time()
    }
    return session_id
```

**Redis 저장 (프로덕션):**
```python
import redis

redis_client = redis.Redis(host='localhost', port=6379)

def create_session(user_id):
    import uuid
    session_id = str(uuid.uuid4())
    redis_client.setex(
        f'session:{session_id}',
        3600,  # 1시간 만료
        json.dumps({'user_id': user_id, 'logged_in': True})
    )
    return session_id
```

### Session의 생명주기

```
┌─────────────────────────────────────────────┐
│ 1. Session 생성 (로그인 시)                  │
│ server.session_create(user_id)              │
└─────────────────────┬───────────────────────┘
                      ↓
┌─────────────────────────────────────────────┐
│ 2. Session 활성 (로그인 상태 유지)           │
│ - 매 요청마다 세션ID로 사용자 확인           │
│ - Last Access Time 갱신                      │
└─────────────────────┬───────────────────────┘
                      ↓
┌─────────────────────────────────────────────┐
│ 3. Session 만료                              │
│ - 브라우저 종료                              │
│ - 타임아웃 (보통 30분)                       │
│ - 사용자 로그아웃                            │
└─────────────────────┬───────────────────────┘
                      ↓
┌─────────────────────────────────────────────┐
│ 4. Session 삭제 (서버)                       │
│ server.session_delete(session_id)           │
└─────────────────────────────────────────────┘
```

---

## Requests 라이브러리 활용

### Session 객체의 강력함

`requests.Session()`은 단순한 요청 반복이 아닙니다.

**Session 사용 전:**
```python
import requests

# 매 요청마다 새로운 연결 → TCP 핸드셰이크 반복
response1 = requests.get('https://api.example.com/users')
response2 = requests.get('https://api.example.com/posts')
response3 = requests.get('https://api.example.com/comments')

# 성능: 느림, 쿠키 자동 유지 안됨
```

**Session 사용 후:**
```python
import requests

session = requests.Session()

# HTTP Keep-Alive로 연결 재사용
response1 = session.get('https://api.example.com/users')
response2 = session.get('https://api.example.com/posts')
response3 = session.get('https://api.example.com/comments')

# 성능: 빠름, 쿠키 자동 유지됨, 헤더 재사용됨
```

### Session의 주요 기능

#### 1. 자동 쿠키 관리

```python
session = requests.Session()

# 첫 요청: 서버가 쿠키 설정
response = session.get('https://example.com/login')

# 두 번째 요청: 자동으로 쿠키 포함됨
response = session.get('https://example.com/dashboard')
# 쿠키를 명시적으로 전달할 필요 없음!

# 쿠키 확인
print(session.cookies)
print(session.cookies.get_dict())
```

#### 2. 헤더 재사용

```python
session = requests.Session()

# User-Agent, Authorization 등을 일괄 설정
session.headers.update({
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)',
    'Referer': 'https://example.com/'
})

# 모든 요청에 자동 적용
response1 = session.get('https://example.com/page1')
response2 = session.get('https://example.com/page2')
```

#### 3. 타임아웃과 재시도

```python
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()

# 재시도 전략
retry_strategy = Retry(
    total=3,  # 최대 3회 재시도
    backoff_factor=1,  # 1초, 2초, 4초 대기
    status_forcelist=[429, 500, 502, 503, 504],  # 재시도할 상태코드
    allowed_methods=["HEAD", "GET", "OPTIONS"]  # 재시도할 메소드
)

adapter = HTTPAdapter(max_retries=retry_strategy)
session.mount("http://", adapter)
session.mount("https://", adapter)

# 자동으로 재시도 적용됨
response = session.get('https://unstable-server.com')
```

#### 4. SSL 검증 무시 (개발용)

```python
session = requests.Session()

# SSL 인증서 검증 무시
response = session.get(
    'https://self-signed-cert.example.com',
    verify=False  # ⚠️ 프로덕션에서는 사용 금지!
)

# 특정 인증서 파일 사용
response = session.get(
    'https://example.com',
    verify='/path/to/ca-bundle.crt'
)
```

### Session 컨텍스트 매니저 사용

```python
# with 문으로 자동 정리
with requests.Session() as session:
    response = session.get('https://example.com')
    # 작업 완료 후 자동으로 세션 종료, 리소스 해제
```

---

## 실전 로그인 구현

### 시나리오 1: 기본 로그인 (Cookies만 사용)

```python
import requests
from bs4 import BeautifulSoup

session = requests.Session()

# 1단계: 로그인 페이지 접속 (쿠키 초기화)
response = session.get('https://example.com/login')
print(f"로그인 페이지 상태: {response.status_code}")

# 2단계: 로그인 요청
login_data = {
    'username': 'myemail@example.com',
    'password': 'mypassword123'
}

response = session.post(
    'https://example.com/login',
    data=login_data
)

# 3단계: 로그인 성공 확인
if response.status_code == 200 and 'dashboard' in response.text:
    print("✓ 로그인 성공!")
else:
    print("✗ 로그인 실패")

# 4단계: 로그인 상태로 크롤링
response = session.get('https://example.com/dashboard')
soup = BeautifulSoup(response.text, 'html.parser')
print(soup.find('h1').text)

# 5단계: 세션 쿠키 저장
import json
with open('session_cookies.json', 'w') as f:
    json.dump(session.cookies.get_dict(), f)
```

### 시나리오 2: CSRF 토큰 처리 (보안 강화)

많은 현대 웹사이트는 CSRF(Cross-Site Request Forgery) 공격을 방지하기 위해 토큰을 요구합니다.

```python
import requests
from bs4 import BeautifulSoup
import re

session = requests.Session()

# 1단계: 로그인 페이지 접속
response = session.get('https://example.com/login')

# 2단계: HTML에서 CSRF 토큰 추출
soup = BeautifulSoup(response.text, 'html.parser')

# 방법1: hidden input에서 추출
csrf_token = soup.find('input', {'name': 'csrf_token'})['value']
# 또는
csrf_token = soup.find('input', {'name': '_csrf'})['value']

print(f"CSRF 토큰: {csrf_token}")

# 3단계: 토큰 포함해서 로그인
login_data = {
    'username': 'myemail@example.com',
    'password': 'mypassword123',
    'csrf_token': csrf_token  # ← 중요!
}

response = session.post(
    'https://example.com/login',
    data=login_data
)

# 4단계: 로그인 확인
if response.history and response.history[0].status_code == 302:
    print("✓ 로그인 성공 (리다이렉트 감지)")
elif 'error' not in response.text.lower():
    print("✓ 로그인 성공")
else:
    print("✗ 로그인 실패")
```

### 시나리오 3: API 인증 (Bearer Token)

```python
import requests

# Bearer Token 방식 (OAuth2, JWT 등)
access_token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'

session = requests.Session()

# 방법1: Authorization 헤더에 직접 설정
session.headers.update({
    'Authorization': f'Bearer {access_token}',
    'Content-Type': 'application/json'
})

response = session.get('https://api.example.com/user/profile')
print(response.json())

# 방법2: 개별 요청마다 지정
response = session.get(
    'https://api.example.com/user/posts',
    headers={'Authorization': f'Bearer {access_token}'}
)

# 방법3: 토큰 갱신 (만료 시)
def refresh_token(refresh_token_value):
    response = requests.post(
        'https://api.example.com/token/refresh',
        json={'refresh_token': refresh_token_value}
    )
    if response.status_code == 200:
        new_token = response.json()['access_token']
        session.headers.update({'Authorization': f'Bearer {new_token}'})
        return new_token
    return None
```

### 시나리오 4: 302 리다이렉트 처리

일부 서버는 로그인 후 다른 페이지로 리다이렉트합니다.

```python
import requests

session = requests.Session()

# 로그인 요청 (리다이렉트 자동 따라가지 않음)
response = session.post(
    'https://example.com/login',
    data={'username': 'user', 'password': 'pass'},
    allow_redirects=False  # 중요!
)

print(f"상태 코드: {response.status_code}")

if response.status_code == 302:
    # 리다이렉트 위치 확인
    redirect_url = response.headers.get('Location')
    print(f"리다이렉트 URL: {redirect_url}")
    
    # 리다이렉트된 페이지 접속 (세션의 쿠키 자동 포함)
    response = session.get(redirect_url)
    print(f"최종 상태: {response.status_code}")
```

---

## 고급 기법

### 1. 멀티 세션 관리 (여러 계정 동시 사용)

```python
import requests
from typing import Dict

class SessionManager:
    def __init__(self):
        self.sessions: Dict[str, requests.Session] = {}
    
    def create_session(self, account_id: str, username: str, password: str):
        """새 세션 생성 및 로그인"""
        session = requests.Session()
        
        # 로그인
        response = session.post(
            'https://example.com/login',
            data={'username': username, 'password': password}
        )
        
        if response.status_code == 200:
            self.sessions[account_id] = session
            print(f"✓ {account_id} 세션 생성")
            return True
        return False
    
    def get_session(self, account_id: str) -> requests.Session:
        """저장된 세션 가져오기"""
        return self.sessions.get(account_id)
    
    def close_all(self):
        """모든 세션 종료"""
        for session in self.sessions.values():
            session.close()
        self.sessions.clear()

# 사용 예
manager = SessionManager()
manager.create_session('account1', 'user1@example.com', 'password1')
manager.create_session('account2', 'user2@example.com', 'password2')

# 각 계정으로 크롤링
session1 = manager.get_session('account1')
response1 = session1.get('https://example.com/dashboard')

session2 = manager.get_session('account2')
response2 = session2.get('https://example.com/dashboard')

manager.close_all()
```

### 2. 클라우드 기반 프록시와 쿠키

```python
import requests

session = requests.Session()

# 프록시 설정
proxies = {
    'http': 'http://proxy.example.com:8080',
    'https': 'https://proxy.example.com:8080'
}

# 프록시를 통해 로그인
response = session.post(
    'https://target.com/login',
    data={'username': 'user', 'password': 'pass'},
    proxies=proxies
)

# 이후 요청도 같은 프록시와 세션 쿠키 사용
response = session.get(
    'https://target.com/protected',
    proxies=proxies
)

# 프록시 회전 (여러 IP 사용)
proxy_list = [
    'http://proxy1.example.com:8080',
    'http://proxy2.example.com:8080',
    'http://proxy3.example.com:8080'
]

import random
proxies = {'http': random.choice(proxy_list)}
response = session.get('https://target.com', proxies=proxies)
```

### 3. 자동 쿠키 갱신

```python
import requests
import time
import json
from datetime import datetime, timedelta

class SmartSessionManager:
    def __init__(self, login_url, username, password):
        self.login_url = login_url
        self.username = username
        self.password = password
        self.session = requests.Session()
        self.last_login_time = None
        self.session_timeout = 1800  # 30분
        
    def is_session_expired(self) -> bool:
        """세션 만료 여부 확인"""
        if self.last_login_time is None:
            return True
        
        elapsed = (datetime.now() - self.last_login_time).total_seconds()
        return elapsed > self.session_timeout
    
    def login(self):
        """로그인 실행"""
        try:
            response = self.session.post(
                self.login_url,
                data={'username': self.username, 'password': self.password},
                timeout=10
            )
            
            if response.status_code == 200:
                self.last_login_time = datetime.now()
                print(f"✓ 로그인 성공 ({self.last_login_time.strftime('%H:%M:%S')})")
                return True
            else:
                print(f"✗ 로그인 실패: {response.status_code}")
                return False
        except requests.RequestException as e:
            print(f"✗ 로그인 오류: {e}")
            return False
    
    def get_with_auto_login(self, url: str) -> requests.Response:
        """필요시 자동 로그인하고 요청"""
        if self.is_session_expired():
            print("⚠ 세션 만료, 재로그인 중...")
            self.login()
        
        return self.session.get(url)

# 사용 예
manager = SmartSessionManager(
    'https://example.com/login',
    'user@example.com',
    'password'
)

# 첫 요청: 자동 로그인
response = manager.get_with_auto_login('https://example.com/page1')

# 29분 후...
time.sleep(29 * 60)

# 재요청: 자동으로 재로그인됨
response = manager.get_with_auto_login('https://example.com/page2')
```

### 4. Selenium과 쿠키 연동

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import pickle
import requests

# Selenium으로 로그인 후 쿠키 저장
chrome_options = Options()
# chrome_options.add_argument("--headless")  # 헤드리스 모드

driver = webdriver.Chrome(options=chrome_options)
driver.get('https://example.com/login')

# (수동 또는 자동으로 로그인)
# username = driver.find_element("id", "username")
# username.send_keys("user@example.com")
# password = driver.find_element("id", "password")
# password.send_keys("mypassword")
# driver.find_element("id", "login_button").click()

# 로그인 완료 후 쿠키 저장
import time
time.sleep(3)  # 로그인 완료 대기

cookies = driver.get_cookies()
with open('selenium_cookies.pkl', 'wb') as f:
    pickle.dump(cookies, f)

driver.quit()

# Requests로 쿠키 로드 및 사용
session = requests.Session()

with open('selenium_cookies.pkl', 'rb') as f:
    cookies = pickle.load(f)

# 쿠키를 requests 형식으로 변환
for cookie in cookies:
    session.cookies.set(
        cookie['name'],
        cookie['value'],
        domain=cookie.get('domain'),
        path=cookie.get('path')
    )

# 크롤링
response = session.get('https://example.com/dashboard')
print(response.status_code)
```

---

## 보안 고려사항

### 1. HTTPS 필수

```python
import requests

# 위험: HTTP 사용
session = requests.Session()
response = session.post('http://example.com/login', data={...})
# ⚠️ 쿠키가 평문으로 전송됨!

# 안전: HTTPS 사용
response = session.post('https://example.com/login', data={...})
# ✓ 암호화됨
```

### 2. 쿠키 보안 속성

```python
# 서버에서 설정해야 하는 속성들:

# HttpOnly: JavaScript 접근 방지 (XSS 방어)
Set-Cookie: sessionid=xyz; HttpOnly

# Secure: HTTPS만 전송
Set-Cookie: sessionid=xyz; Secure

# SameSite: CSRF 방어
Set-Cookie: sessionid=xyz; SameSite=Strict

# 완벽한 설정
Set-Cookie: sessionid=xyz; Secure; HttpOnly; SameSite=Strict; Path=/; Max-Age=3600
```

### 3. 민감한 정보 보호

```python
import os
from dotenv import load_dotenv

# 환경변수 사용 (하드코딩 금지)
load_dotenv()

username = os.getenv('CRAWLER_USERNAME')
password = os.getenv('CRAWLER_PASSWORD')

# 쿠키 파일 권한 제한
import stat
import json

with open('cookies.json', 'w') as f:
    json.dump(session.cookies.get_dict(), f)

# 파일 권한 600 (소유자만 읽기/쓰기)
os.chmod('cookies.json', stat.S_IRUSR | stat.S_IWUSR)
```

### 4. Rate Limiting & 정중한 크롤링

```python
import requests
import time

session = requests.Session()

urls = ['https://example.com/page1', 'https://example.com/page2']

for url in urls:
    response = session.get(url)
    
    # 서버 부하 고려
    time.sleep(2)  # 2초 대기
    
    # Robots.txt 확인
    robots = session.get('https://example.com/robots.txt')
    # 크롤링 규칙 파싱 및 준수

# User-Agent 설정 (자신을 명시)
session.headers.update({
    'User-Agent': 'MyBot/1.0 (+http://mybot.com/bot)'
})
```

### 5. 예외 처리

```python
import requests
from requests.exceptions import (
    ConnectionError,
    Timeout,
    HTTPError,
    RequestException
)

session = requests.Session()

try:
    response = session.get(
        'https://example.com/api',
        timeout=10  # 타임아웃 설정
    )
    response.raise_for_status()  # HTTP 에러 발생
    
except HTTPError as e:
    print(f"HTTP 에러: {e.response.status_code}")
except Timeout:
    print("요청 타임아웃")
except ConnectionError:
    print("연결 오류")
except RequestException as e:
    print(f"요청 실패: {e}")
finally:
    session.close()
```

---

## FAQ & 트러블슈팅

### Q1: "403 Forbidden" 에러가 발생합니다

**원인:** 서버가 봇 요청을 차단함

**해결책:**
```python
session = requests.Session()

# User-Agent 설정
session.headers.update({
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
})

# Referer 설정
session.headers.update({
    'Referer': 'https://example.com/'
})

# 쿠키 포함
response = session.get('https://example.com/api')
```

### Q2: 로그인은 성공하는데 대시보드 접근이 안 됩니다

**원인:** 세션 쿠키가 올바르게 유지되지 않음

**디버깅:**
```python
import requests

session = requests.Session()

# 로그인
response = session.post('https://example.com/login', data={...})
print(f"로그인 응답: {response.status_code}")
print(f"세션 쿠키: {session.cookies.get_dict()}")

# 대시보드 접근
response = session.get('https://example.com/dashboard')
print(f"대시보드 응답: {response.status_code}")
print(f"대시보드 내용 길이: {len(response.text)}")

# 리다이렉트 확인
print(f"리다이렉트 히스토리: {response.history}")

# HTML에서 "로그인" 텍스트 있는지 확인 (로그인 실패 여부)
if 'login' in response.text.lower():
    print("⚠️ 로그인 상태 유지 실패!")
```

### Q3: CSRF 토큰이 계속 변경됩니다

**원인:** 매 요청마다 새로운 토큰 발급 (보안 강화)

**해결책:**
```python
import requests
from bs4 import BeautifulSoup

session = requests.Session()

# 토큰 추출 함수
def get_csrf_token():
    response = session.get('https://example.com/form')
    soup = BeautifulSoup(response.text, 'html.parser')
    return soup.find('input', {'name': 'csrf_token'})['value']

# 매 요청마다 토큰 갱신
csrf_token = get_csrf_token()
response = session.post(
    'https://example.com/submit',
    data={'csrf_token': csrf_token, 'data': 'value'}
)

# 또는 로그인 폼에서 미리 추출
response = session.get('https://example.com/login')
soup = BeautifulSoup(response.text, 'html.parser')
csrf_token = soup.find('input', {'name': '_csrf'})['value']

# 동일 세션 내에서 사용
response = session.post(
    'https://example.com/login',
    data={'username': 'user', 'password': 'pass', '_csrf': csrf_token}
)
```

### Q4: 저장한 쿠키가 만료되었습니다

**해결책:**
```python
import requests
import json
from datetime import datetime

session = requests.Session()

# 저장된 쿠키 로드 및 만료 확인
with open('cookies.json', 'r') as f:
    saved_cookies = json.load(f)

# 만료된 쿠키 필터링
valid_cookies = {}
current_time = datetime.now().timestamp()

for name, value in saved_cookies.items():
    # Expires 정보가 있으면 확인
    # (JSON에는 보통 저장 안됨, pickle 사용 권장)
    valid_cookies[name] = value

session.cookies.update(valid_cookies)

# 테스트
response = session.get('https://example.com/protected')

if response.status_code == 401:
    print("쿠키 만료, 재로그인 필요")
    # 재로그인 로직
```

### Q5: 동적 콘텐츠(JavaScript 렌더링)를 크롤링할 수 없습니다

**해결책:** Selenium 또는 Playwright 사용

```python
# Selenium 예시
from selenium import webdriver
import requests

driver = webdriver.Chrome()
driver.get('https://example.com/login')

# JavaScript 렌더링된 페이지에서 로그인
# (수동 또는 자동 로그인)

# 렌더링 완료 후 쿠키 추출
cookies = {cookie['name']: cookie['value'] for cookie in driver.get_cookies()}

# Requests로 계속 사용
session = requests.Session()
session.cookies.update(cookies)

response = session.get('https://example.com/api')
print(response.json())
```

---

## 정리: 선택 가이드

| 상황 | 추천 기술 |
|-----|--------|
| 간단한 정적 페이지 | `requests.Session()` + cookies |
| API 인증 필요 | `Bearer Token` + headers |
| 세션 유지 중요 | `requests.Session()` |
| JavaScript 렌더링 필요 | Selenium / Playwright |
| 대규모 크롤링 | Redis + Session Manager |
| 높은 보안 필요 | HTTPS + CSRF + HttpOnly |

---

## 연습 문제

1. 특정 웹사이트에서 로그인 후 개인 정보를 크롤링하는 코드 작성
2. CSRF 토큰을 자동으로 추출하고 처리하는 함수 개발
3. 여러 계정으로 동시에 크롤링하는 SessionManager 작성
4. 만료된 토큰을 감지하고 자동으로 갱신하는 로직 구현
5. Selenium과 Requests를 연동한 하이브리드 크롤러 개발

---

**마지막 팁:** 항상 대상 웹사이트의 로봇 정책(robots.txt)을 확인하고, 서버 부하를 고려하여 정중하게 크롤링하세요!
