# 커널 객체로 동기화하기 2

### 동기화 전용 커널 객체들
* 뮤텍스
* 이벤트
* 세마포어
* 대기가능 타이머(WaitableTimer)

### 객체 생성

```
HANDLE CreateXXX(LPSECURITY_ATTRIBUTES, pszName)
```
### 반환값
* 실패시 NULL
* 이미 존재시, 해당 객체를 반환하며 GetLastError()는 ERROR_ALREADY_EXISTS를 반환한다. 따라서 이미 존재하는지, 처음 생성한 건지를 판단할 수 있다.


### 뮤텍스
* 소유권이 중요
* CreateMutex시, 두 번째 인자로 bInitialOwner 값을 TRUE로 주면 호출한 스레드가 소유권을 갖게 된다. 즉 뮤텍스는 넌시그널 상태가 된다.

```
void AcquireLock(HANDLE hMutex){
    if(WaitForSingleObject(hMutext, INFINITE) != WAIT_OBJECT_0){
        //에러처리...
    }
}
```
* WaitForSingleObject()의 첫 번째 인자에 뮤텍스를 넘기면, 뮤텍스에 대한 소유권을 WaitForSingleObject()를 호출한 스레드에게 넘긴다. 즉 Acquire 혹은 Enter에 해당하는 동작을 수행한다.

```
void ReleaseLock(HANDLE hMutex){
    ReleaseMutex(hMutex);
}

```

#### Abandoned Mutex
* ReleaseMutex()를 처리하지 않고 스레드가 종료되는 경우이다.
* 시스템은 Thread 종료시 뮤텍스를 갖고 있는지 확인하고, 갖고 있다면 해당 뮤텍스를 ABANDONED로 설정하고 뮤텍스를 해제시킨다.
* 해당 뮤텍스를 다른 스레드가 사용할 수 있으며, ReleaseMutex()를 호출하면 정상적으로 돌아오게 된다.

### 뮤텍스를 종료 통지로 사용하기

```
#include <Windows.h>
#include <stdio.h>
DWORD WINAPI ThreadProc(PVOID pParam) {

	PHANDLE parmWaits = (PHANDLE)pParam;
	while (true) {
		DWORD dwWaitCode = WaitForMultipleObjects(2, parmWaits, FALSE, INFINITE);
		if (dwWaitCode == WAIT_FAILED) {
			printf("=== WaitForMultipleObjects Failed : %d\n", GetLastError());
			break;
		}
		if (dwWaitCode == WAIT_OBJECT_0) {
			ReleaseMutex(parmWaits[0]); //빼먹으면안됨!
			break;
		}
		printf("[%d] : running\n", GetCurrentThreadId());
		ReleaseMutex(parmWaits[1]);
		Sleep(1000);
	}

	printf("[%d] exit...\n", GetCurrentThreadId());
	return 0;
}

void main() {
	printf("Main Thread Creating sub Thread...\n");
	HANDLE hMutexs[2];
	hMutexs[0] = CreateMutex(NULL, TRUE, NULL); //종료 통지
	hMutexs[1] = CreateMutex(NULL, FALSE, NULL); //스레드 동기화
	HANDLE arhThreads[5];
	for (int i = 0; i < 5; i++) {
		DWORD dwThreadId = 0;
		arhThreads[i] = CreateThread(NULL, 0, ThreadProc, hMutexs, 0, &dwThreadId);
	}


	getchar();
	ReleaseMutex(hMutexs[0]);
	WaitForMultipleObjects(5, arhThreads, TRUE, INFINITE);
	printf("All sub Thread terminated.....\n");
	for (int i = 0; i < 5; i++) {
		CloseHandle(arhThreads[i]);
	}
	for (int i = 0; i < 2; i++) {
		CloseHandle(hMutexs[i]);
	}
	getchar();
}
```

### 중복 대기
* 뮤텍스를 소유한 스레드가 중복적으로 WaitForXXX를 호출할 경우, 블록되지 않고 바로 넘어간다.
* 다만 WaitForXXX를 하는 만큼 ReleaseMutex()를 호출해야 한다.
