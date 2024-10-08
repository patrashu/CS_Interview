## 웹서버 접속 과정

1. 브라우저 주소 창에 www.google.com을 입력했을 때 일어나는 일은?
	- google.com을 입력하게 되면, 우선 DNS 서버에 접근하여 google.com이라는 도메인 네임을 기반으로 실제 IP주소를 탐색하는 과정을 진행함
	- 실제 IP주소를 바탕으로 서버에 HTTP 요청을 보내며 웹서버와 통신
	- 웹 서버는 브라우저의 요청을 처리한 후, 요청에 대한 결과를 HTML 형식으로 브라우저에게 전송
	- 요청한 브라우저는 응답 결과인 HTML을 받아와 사용자에게 웹페이지를 보여줌

2. DNS 조회는 구체적으로 어떻게 이뤄지나요?
	- 브라우저가 DNS 조회를 요청하면 로컬 DNS 캐시를 확인 후 조회 결과가 저장되어 있는지 확인
	- 그렇지 않을 경우 인터넷 상의 DNS 서버들을 차례대로 방문하며, 해당 도메인에 대한 IP주소를 찾을 때까지 반복
	- IP 주소는 사용자의 로컬 DNS 캐시에 저장되어 후속 조회를 빠르게 할 수 있도록 도와줌

## IP

1. IP가 무엇인가요?
	- IP는 Internet Protocol의 약자로, 네트워크에서 데이터를 전송하는 데 사용되는 프로토콜임
	- 송신 측에서 보낸 데이터 패킷을 수신 측으로 안전하게 이동할 수 있도록 도와주는 역할을 함(라우팅)
	- 수신 측에서 받을 수 있는 크거나 작은 경우, 데이터 패킷을 조립/분할하여 수신 측으로 전달시키는 역할을 함

2. IP 라우팅은 어떻게 이뤄지나요?
	- IP 라우팅이란 데이터 패킷이 송신 측에서 수신 측으로 이동하는 과정을 의미함
	- 라우터는 IP 헤더 내 목적지 주소(수신 측 IP)를 읽은 후, 내부 라우팅 테이블을 통해 패킷을 다음 목적지로 전달 (hop by hop)
	- 라우팅 테이블은 다양한 네트워크 경로와 조건을 포함하고 있으며, 해당 테이블 내에서 목적지까지 가기 위한 최적의 경로를 선택함 (다익스트라/벨만포드 등)

3. IP 주소의 구조와 종류에는 어떤 것들이 있나요?
	- IP 주소는 크게 IPv4와 IPv6로 나뉨
	- IPv4는 32bit로 구성되어 있으며, 점으로 구분된 네 부분으로 표시됨 (127.0.0.1)
	- IPv6는 128bit로 구성되어 있으며, 콜론으로 구분된 여덟 부분으로 표시됨 (0000:0000:...)

4. IP 패킷의 구조는 어떻게 되나요?
	- 기본적으로 IP 패킷은 헤더 부분과 페이로드(데이터) 부분으로 구성됨
	- 헤더에는 프로토콜 버전(IPv4/IPv6), 헤더 길이, 전체 길이, 프로토콜, 체크썸(오류 검출), 송/수신측 IP 주소 등이 포함됨
	- 데이터 부분은 전송할 실제 데이터를 포함하며, 헤더 정보에 의해 수신 측으로 올바르게 전달될 수 있도록 라우팅 됨

5. IPv4를 사용하는 장비와 IPv6를 사용하는 장비 간 네트워크 통신이 가능한지? 가능하다면 어떤 방법을 사용하는지?
	- 기본적으로 IPv4와 IPv6 장비 간 직접적인 통신은 불가능함
	- 터널링이나 NAT64와 같은 방법을 활용할 경우 IPv4와 IPv6 장비 간 통신이 가능함

6. IP주소와 MAC주소의 차이점
	- IP 주소는 네트워크 계층에서 사용되며, 논리적인 주소로써 네트워크 내/외부에서 데이터 전송을 위해 사용됨
	- MAC 주소는 링크 계층에서 사용되며, 물리적인 주소로써 통신을 진행하는 단말기(Terminal)의 고유 주소를 의미함

## TCP/UDP
	
1. TCP와 무엇인지 말씀해주세요. 어떤 특징을 가지고 있으며, 어떤 상황에서 사용하기에 적합한가요?
	- TCP는 신뢰성있는 연결 지향 프로토콜로써, 데이터 전송을 관리하고 데이터의 순서와 무결성을 보장함
	- 연결 지향형 특성을 가지고 있으며, 송/수신측 간 3way handshake를 거친 후 신뢰성을 기반으로 데이터를 주고 받음
	- 수신 측이 처리할 수 있는 속도에 맞게 흐름을 제어하며, 네트워크 상태에 맞게 전송 속도를 제어함
	- 중간에 데이터 패킷이 손실되었을 경우, 해당 부분에 대한 패킷을 수신측으로 재전송
	- FTP, SMTP, DB연결 등과 같은 곳에서 TCP가 사용됨

2. 혼잡 제어에 대해서 설명해주세요.
	- 혼잡 제어란 네트워크 내에서 데이터 트래픽이 과도하게 몰려 네트워크 성능이 저하되는 현상을 방지하기 위한 매커니즘임
	- 송신 측에서 혼잡 제어가 수행되며, 네트워크 상태를 모니터링하며 전송 속도를 동적으로 조절함

3. 흐름 제어에 대해서 설명해주세요. 흐름 제어를 처리할 수 있는 방법도 한 가지 말씀해주세요
	- 흐름 제어란 
	- 수신 측에서 흐름 제어가 수행되며, 수신 버퍼의 상태를 모니터링하며 송신 측에 이를 알림으로써 데이터 속도를 조절함
	- 대표적인 흐름 제어 방법으로 슬라이딩 윈도우 방식이 존재함
		- 송신자는 수신자로부터 수신 윈도우 크기를 받고, 해당 크기 내에서 데이터를 전송함
		- 송신자가 1000 바이트의 데이터를 보내려 하고, 수신 윈도우 크기가 500 바이트라면, 송신자는 처음에 500 바이트를 보내고, 수신자로부터 ACK를 받은 후 나머지 500 바이트를 전송함

4. UDP가 무엇인지 말씀해주세요. 어떤 특징을 가지고 있으며, 어떤 상황에서 사용하기에 적합한가요?
	- UDP는 신뢰성이 낮은 비연결 지향 프로토콜로써, 빠르게 데이터를 전송하기 위하여 설계됨
	- 데이터 전송 전에 송/수신측 간 연결 없이 각 데이터 패킷이 독립적으로 전송됨
	- 패킷이 도착하는 순서가 보장되지 않으며, TCP에 비하여 헤더 크기가 작음 
	- 실시간 스트리밍 서비스나 DNS 조회와 같은 곳에서 UDP가 사용됨

## TCP/IP

1. TCP/IP 4계층에 대해 설명해주세요
	- TCP/IP는 데이터 전송의 단계를 4단계 계층으로 나누어 정의함
	- 각 계층은 독립적으로 동작하며, 상위 계층에서 요청한 서비스를 하위 계층에서 제공함
	- 네트워크 인터페이스 / 인터넷(Internet=Network) / 전송(Transport) / 응용(Application)으로 나뉨

2. TCP/IP 4계층 중 인터넷 계층의 역할과 주요 프로토콜에 대해서 말씀해주세요
	- 인터넷 계층은 IP를 기반으로 하는 계층으로, 주로 데이터 패킷의 전송을 담당함
	- 인터넷 계층에서는 논리적 주소 지정(송/수신 IP), 패킷 라우팅, 패킷 조립 및 분할, 오류 제어 등을 진행함
	- 패킷 라우팅의 경우 송신지에서 수신지로 전달하는 경로를 결정하는 단계이며, 내부 라우팅 테이블을 통해 패킷을 다음 목적지로 전달함
	- 패킷 분할 및 재조립의 경우 수신 측에서 받을 수 있는 버퍼 크기에 따라 송신 측의 데이터 패킷을 작은 크기로 분할하는 작업을 진행함
	- 인터넷 계층의 대표적인 프로토콜은 핵심인 IP, 네트워크 오류를 판단해주는 ICMP, IP주소를 MAC주소로 매핑하는 ARP 등이 존재함

3. TCP/IP 4계층에서 각 계층 간의 데이터 단위와 캡슐화 과정에 대해 설명해주세요.
	- 캡슐화 과정은 데이터가 송신측에서 각 계층을 거치며 필요한 헤더 정보를 추가하여 최종적으로 링크 계층을 통해 전송되는 과정을 의미함
	- 응용 계층에서는 데이터의 단위가 Data/Payload라고 불리며, HTTP/FTP/SMTP와 같은 프로토콜이 포함됨
	- 전송 계층에서는 데이터의 단위가 Segment/Datagram이라 불리며, 응용 계층에서 받은 정보 + 헤더(송/수신측 포트번호, SYN, ACK, FIN 등)가 포함됨
	- 인터넷 계층에서는 데이터의 단위가 Packet이라 불리며, 전송 계층에서 받은 정보 + 헤더(송/수신측 IP 주소, TTL, IPv4인지 v6인지 등)가 포함됨
	- 네트워크 인터페이스 계층에서는 데이터의 단위가 Frame이라 불리며, 인터넷 계층에서 받은 정보 + 헤더(송/수신측 MAC주소, Frame 타입 등) + 트레일러(오류 검출용)가 포함됨
	- 이와 같은 캡슐화 과정을 통해 송/수신측 간 데이터 통신이 가능해지며, 수신 측에서는 반대로 디캡슐화를 통해 Payload를 전달받게 됨

## 3way handshake // 4way handshake

1. 3way handshake가 무엇이며, 어떻게 작동하는지 작동 방식을 설명해보세요
	- TCP 연결을 설정하기 위한 절차로, 신뢰성있는 통신을 보장하기 위해 사용함
	- 작동 방식
		- SYN: 클라이언트가 서버에게 연결 요청을 보내며, 초기 순서 번호(SYN)를 포함한 패킷을 전달
		- SYN-ACK: 서버는 클라이언트의 요청을 수락하고, 서버측 초기 순서 번호(ACK)와 클라이언트에 대한 응답(SYN+1)을 포함한 패킷을 전달
		- ACK: 클라이언트가 서버의 응답을 확인하며, 서버에 대한 응답(ACK+1)을 포함한 패킷을 전달
		=> 총 3단계로 진행함으로써 클라이언트-서버 간 연결을 진행함
		=> 2단계로 진행할 경우 서버는 연결되었다고 생각하지만, 클라이언트에선 그렇지 않다고 생각할 수 있기 때문에 3way로 진행해야 함

2. 4way handshake가 무엇이며, 어떻게 작동하는지 작동 방식을 설명해보세요
	-  TCP 연결을 해제하기 위한 절차로, 양쪽에서 연결을 종료하기 위해 사용됨.
	- 작동 방식
		- FIN: 클라이언트가 서버에게 연결 종료를 요청하는 패킷(FIN)을 서버에 전송하고, 클라이언트는 FIN-WAIT 상태가 됨
	 	- ACK: 서버는 클라이언트의 FIN 패킷을 확인하며, ACK 패킷을 클라이언트에게 전송함
		- FIN: 서버가 연결을 종료할 준비가 되었을 경우, 다시 클라이언트에게 연결 종료가 준비되었다는 패킷(FIN)을 보냄
		- ACK: 클라이언트가 서버의 FIN 패킷을 확인하며, ACK 패킷을 서버에 전송함으로서 연결을 종료함

## HTTP 
	
1. HTTP가 뭔가요?
	- HTTP는 HyperText Transfer Protocol의 약자로써, 웹에서 정보를 교환하기 위한 프로토콜(통신 규약)임
	- 웹 페이지를 요청하고 받기 위해 클라이언트와 서버 간 통신(요청 및 응답)을 관리
	- 기본적으로 HTTP는 Stateless기반 프로토콜이기에 각각의 요청과 응답이 독립적으로 처리됨

2. HTTP의 주요 기능은?
	- 사용자가 웹 페이지를 요청했을 때 HTTP는 해당 요청을 웹 서버에 전달하고, 요청에 대한 응답(결과)을 다시 사용자에게 보내주는 역할을 함
	- 웹 서버의 응답에는 상태 코드가 포함되며, 상태 코드를 통해 성공 및 실패 여부를 확인할 수 있음
	- HTTP는 다양한 Method를 지원하며, 이를 활용하여 다양한 작업을 수행할 수 있음

3. HTTP 1.0부터 HTTP/3까지 대표적인 특징을 설명해주세요
	- HTTP 1.0 :
		- 클라이언트에서 서버로 한 번 요청할 때마다 연결(3Way handshake)을 반복적으로 수행해야 함
		- 각 요청마다 새로운 연결을 생성하여 성능이 떨어짐.
	- HTTP 1.1 :	
		- 클라이언트의 여러 요청을 한 번의 연결로 처리할 수 있도록 변경 (지속 연결, Persistant Connection)
		- 먼저 요청한 작업이 오래 걸릴 경우 (대용량 파일 다운로드 등), 나중에 요청한 작업의 길이가 짧음에도 불구하고 기다려야 하는 문제점이 발생 (HOL Bloking)
	- HTTP/2   :
		- 단일 TCP 연결에서 멀티 플렉싱(여러 요청을 병렬적으로 처리)을 지원함으로써 성능이 향상됨
		- 헤더 압축을 통해 데이터 전송 효율성을 높임
		- 서버 푸시를 통해 클라이언트가 요청하지 않은 리소스도 전송이 가능함
	- HTTP/3   :
		- QUIC 프로토콜을 기반으로 하며, UDP를 사용함
		- 연결 설정 시간이 단축되고 데이터 전송 지연이 감소함

4. HTTP 메서드란 무엇인가요?
	- HTTP 메서드란 클라이언트가 서버로 요청을 보낼 때 사용하는 명령어로써 요청이나 목적에 따라 형태를 지정함
	- 데이터 전송, 조회, 수정 등과 같은 작업을 진행할 때 활용 가능함

5. 주요 HTTP 메서드와 그 용도를 설명해주세요
	- GET     : 리소스를 조회하기 위한 요청으로써, 서버에 데이터를 조회하고 가져옴
	- POST   : 리소스를 생성하기 위한 요청으로써, 서버에 데이터를 보내고 처리함
	- PUT     : 리소스를 생성하거나 대체하기 위한 요청으로써, 지정된 리소스가 없으면 생성하고 존재한다면 수정함
	- PATCH : 리소스의 일부를 수정하기 위한 요청
	- DELETE : 리소스의 일부를 삭제하기 위한 요청

6. GET/POST의 차이점을 설명해주세요
	- GET
		- 데이터를 조회할 때 사용하며, 데이터 조회에 필요한 값을 URL에 쿼리스트링 형식으로 전달함
		- 데이터가 브라우저 히스토리나 서버 로그에 기록될 수 있음 (보안에 취약)
		- 길이 제한이 존재함
	- POST
		- 데이터를 생성하거나 서버에 제출할 때 사용하며, 쿼리스트링 형식이 아닌 body 태그 안에 데이터를 포함하여 전달함
		- 데이터가 브라우저 히스토리나 서버 로그에 기록되지 않음 (보안에 우수)
		- 데이터 크기에 제한이 없음

7. PUT/PATCH의 차이점을 설명해주세요
	- PUT
		- 리소스를 전체적으로 수정하거나 생성하며, 요청 본문에 리소스 전체를 포함함
		- 사용자의 프로필 정보 전체를 수정하고 싶을 때 활용 가능함
	- PATCH
		- 리소스의 일부만을 수정하고 싶을 때 사용하며, 요청 본문에 수정하고 싶은 리소스만 포함함
		- 사용자의 프로필 내 이름/이메일 등과 같이 일부만 수정하고 싶을 때 활용 가능함

8. Stateless와 Stateful의 차이점 및 예시를 설명해주세요
	- Stateless
		- Stateless는 서버가 이전 요청의 상태를 기억하지 않는 것을 의미함
		- 확장성이 뛰어나며, 쉽게 수평적 확장이 가능함
		- HTTP나 DNS가 대표적인 Stateless임
	- Stateful
		- Stateful은 서버가 클라이언트의 상태를 기억하며, 각 요청이 이전 요청의 상태의 영향을 받음
		- 클라이언트의 상태를 저장해야하기 때문에 확장성이 Stateless에 비해 떨어짐
		- FTP나 DB 연결과 같은 곳에서 Stateful을 사용함
