# 프로세스

### 구성요소
* Process ID
* Security Context
* Virual Address Space
* Program Instance
* Handle table
* Threads


#### Process ID
* 프로세스 식별 수단

#### Security Context
* 스레드는 보안 객체(보안속성을 갖는 커널 객체)에 접근할 때 권한을 갖고 있어야 함
* 프로세스는 로그인된 유저 ID와 소속그룹ID, 여러 특권(Privilege)을 지닌 액세스 토큰(Access Token, 역시 커널 객체) 갖음.
* 스레드는 기본적으로 자신이 속한 프로세스의 액세스 토큰을 통해 권한이 부여됨.

#### Virual Address Space
* VMM에 의해 관리되는 가상 메모리

#### Program Instnace
* EXE 파일이 가상 주소 공간에 매핑되어 메모리에 로드된 상태
* WinMain의 hInstance는 매핑된 영역의 시작 주소(가상 주소값)
* 흔히 코드 영역, 데이터 영역이라고 부르는 PE에서의 '섹션'이 여기에 해당

#### Handle Table
* 프로세스는 커널 객체의 포인터를 테이블 형태로 갖고 있다.

#### Threads
* 코드 실행 주체
* 적어도 하나 이상의 스레드가 실행된다.

#### reference
윈도우 시스템 프로그램을 구현하는 기술
