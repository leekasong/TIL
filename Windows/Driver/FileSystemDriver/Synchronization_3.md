# 동기화-3

### 리소스 오브젝트
* 파일 시스템 필터 드라이버와 같은 상위 레벨 드라이버에서 사용되는 오브젝트
* Exclusive access와 Shared access를 모두 제공한다.
* Exclusive access는 데이터 수정이 필요한 경우 단일 스레드에게 제공된다.
* Shared access는 읽기 전용으로 데이터를 접근할 경우 여러 개의 스레드에게 동시에 제공된다.
* 데드록에 대한 디버깅 정보를 담고 있어 유용하다.

### Fast Mutex
* 하나의 스레드가 반복해서 뮤텍스를 얻을 수 없다.
* 뮤텍스보다 최적화가 더 되어 있어 빠르다.
* Fast Mutex를 소유한 스레드는 XXXUnsafe를 호출하지 않으면 APC를 수신할 수 없다.
* KeWaitForMultipleObjects()의 인수로 사용할 수 없다.
