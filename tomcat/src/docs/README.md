## 미션: Tomcat 구현하기

### 목표

- HTTP 요청의 기본 동작을 직접 구현해 본다.
- 정적 리소스 제공, 로그인/회원가입, 세션, 멀티스레드 처리까지 한 번에 다룬다.

### 공통 조건

- 실행 포트: 8080
- 브라우저에서 http://localhost:8080/...로 접근 가능해야 함
- 프레임워크 없이 java.net, io, 컬렉션 위주로 구현
- 각 요청은 최소한 아래 응답 형식을 만족해야 함
  HTTP/1.1 <status> <reason> + Header + Body

## Step 1: 정적 파일과 쿼리 파싱

### 기능 요구사항

1. GET /index.html 요청 시 HTML 파일을 읽어 200 OK로 응답한다.
2. css, js 등 정적 파일을 응답한다.
3. URI의 QueryString(?a=1&b=2)을 파싱할 수 있다.

### 체크포인트

- GET /index.html 가능
- GET /css/..., GET /js/... 가능
- http://localhost:8080/page?foo=bar&name=b 에서 foo, name 값 추출 가능


## Step 2: POST, 리다이렉트, 쿠키/세션

### 기능 요구사항

1. POST 요청 바디를 파싱한다.
   (폼(form) 스타일 application/x-www-form-urlencoded 형태)
2. 회원가입 기능 구현
    - /register POST로 account, password, email 입력 처리
    - 성공 시 302 Found + Location 헤더로 로그인 페이지로 이동
    - 중복 계정 예외 처리
3. 로그인 기능 구현
    - /login POST로 계정 인증
    - 성공 시 302 Found + Location 헤더로 메인 페이지 이동
    - 성공 시 Set-Cookie: JSESSIONID=<id> 포함
4. 세션 저장소 구현
    - SessionManager가 세션 객체(Session)를 JSESSIONID 기준으로 보관
    - 같은 쿠키가 들어온 경우 기존 세션 유지 또는 갱신
5. 실패 처리
    - 로그인 실패 시 401 응답

### 체크포인트

- 로그인 성공 응답에 302 + Location + Set-Cookie
- 요청 바디에서 account, password 추출 가능
- 잘못된 계정/비밀번호는 401


## Step 3: 구조화 및 라우팅 추상화

### 기능 요구사항

1. HttpRequest 클래스로 요청값(메서드, URL, Header, Body, QueryString)을 모델링
2. HttpResponse 클래스로 응답값(상태코드, 헤더, 바디)을 모델링
3. Controller 인터페이스 도입
4. if-else 의존 라우팅 제거 및 RequestMapping으로 경로 분기
    - /, /login, /register, 정적 파일, 그 외(404) 처리

### 체크포인트

- 요청/응답 객체 생성 및 사용이 일관적이어야 함
- 라우팅이 URL 기준으로 컨트롤러 분기
- 미지원 경로는 404 페이지 응답

## Step 4: 동시성(Thread Pool) 적용

### 기능 요구사항

1. ExecutorService를 사용해 클라이언트 소켓 처리를 병렬화한다.
2. 서버가 동시에 여러 요청을 수락·처리할 수 있어야 한다.
3. 동시성 접근 가능한 자료구조를 사용해 세션/저장소 상태를 관리한다.

### 체크포인트

- 수락 스레드와 워커 스레드 분리
- 다중 요청에서 세션/저장 로직 동작 안정성
