## event-based HTTP 프록시 서버
### 프로젝트 개요

이 프로젝트는 HTTP/1.0 프록시 서버를 구현하여 클라이언트와 웹 서버 간의 통신을 중계하는 역할을 수행하며, 구현된 주요 기능과 테스트 결과는 다음과 같다.

### 주요 기능

* **클라이언트 요청 처리:** HTTP GET 요청을 받아들이고, 요청 메시지를 파싱하여 요청 URI와 Host 정보를 추출
* **서버 통신:** 추출된 정보를 바탕으로 대상 서버에 연결하여 HTTP 요청을 전송하고, 서버 응답을 수신
* **응답 전달:** 서버로부터 받은 응답 메시지를 클라이언트에게 전달
* **User-Agent 변경:**
    * 클라이언트가 User-Agent 헤더를 제공하지 않으면, 프록시 서버는 자체 User-Agent (`proxy309/1.0`)를 추가하여 서버에 요청을 보낸다.
    * 클라이언트가 User-Agent 헤더를 제공하면, 해당 헤더 값을 서버에 전달
* **에러 처리:** 대상 서버와의 연결 실패 또는 잘못된 요청으로 인한 오류 발생 시, HTTP 500 Internal Server Error 응답을 반환
* **epoll 기반 동시성 처리:** epoll 시스템 콜을 사용하여 클라이언트와 리스너 소켓을 효율적으로 관리

### 테스트 결과

* **User-Agent 변경:**
    * **클라이언트가 User-Agent를 제공하지 않을 때:** 프록시 서버가 `proxy309/1.0` User-Agent를 추가하여 서버에 요청을 보내는 것을 확인
    * **클라이언트가 User-Agent를 제공할 때:** 클라이언트가 제공한 User-Agent가 서버에 전달되는 것을 확인 (예: `abcd/1.0`)
* **오류 처리:**
    * **존재하지 않는 IP 주소/도메인 이름으로 요청 시:** HTTP 500 Internal Server Error 응답을 반환하는 것을 확인 (예: `127.0.0.1`, `www.maliciousnonexisting.com`)
* **정상적인 도메인 요청:** 정상적인 도메인으로 요청 시: 서버로부터 정상적인 HTTP 응답을 받아 클라이언트에게 전달하는 것을 확인 (예: `www.google.com`)

### 추가 설명

* Destination server와의 연결은 요청 처리 시에만 이루어지며, 응답 후 연결을 종료하는 방식으로 구현되었습니다. 따라서 destination server socket에 대한 epoll 관리는 필요하지 않음.
* HTTP/1.0 프로토콜을 지원하며, GET 요청만 처리함.
