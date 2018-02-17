# 루트킷

* 운영체제는 커널을 조작할 수 있는 함수를 제공

### 프로세스 실행 및 종료

#### PsSetCreateProcessNotifyRoutine()
* 새로운 프로세스 실행 및 종료시 운영체제가 커널 드라이버 루틴을 호출한다.


* 첫번째인자 : 콜백함수
```
VOID (*PCREATE_PROCESS_NOTIFY_ROUTINE)(
    IN HANDLE ParentId,
    IN HANDLE ProcessId,
    IN BOOLEAN Create
    )
```
1. 부모 프로세스아이디
2. 자신 프로세스 아이디
3. 생성되는지 종료되는지.

* 두번째 인자 : FALSE : 루틴 등록, TRUE : 루틴 해제

#### PsSetLoadImageNotifyRoutine()

* 다른 드라이버 또는 동적라이브러리가 메모리에 상주할 때마다 운영체제가 커널 드라이버의 루틴을 호출

#### PsRemoveLoadImageNotifyRoutine()
* 위 함수와 반대되는 개념.
