# 커널 객체로 동기화

### 하나의 스레드가 다른 하나의 스레드 기다리기
* WaitForSingleObject()를 사용한다.

```
#include <Windows.h>
#include <stdio.h>
DWORD WINAPI ThreadProc(PVOID pParam) {
	printf("Thread[%d] enter\n", GetCurrentThreadId());
	Sleep(3000);
	printf("Thread[%d] leave\n", GetCurrentThreadId());
	return 0;
}

void main() {
	printf("Main Thread Creating sub Thread...\n");

	DWORD dwThreadId = 0;
	HANDLE hThread = CreateThread(NULL, 0, ThreadProc, NULL, 0, &dwThreadId);

	WaitForSingleObject(hThread, INFINITE);
	getchar();
	CloseHandle(hThread);
}
```

### 하나의 스레드가 여러 스레드를 기다리기
* WaitForMultipleObjects()를 사용한다.

```
#include <Windows.h>
#include <stdio.h>
DWORD WINAPI ThreadProc(PVOID pParam) {
	printf("Thread[%d] enter\n", GetCurrentThreadId());
	Sleep(3000);
	printf("Thread[%d] leave\n", GetCurrentThreadId());
	return 0;
}

void main() {
	printf("Main Thread Creating sub Thread...\n");


	HANDLE arhThreads[5];
	for (int i = 0; i < 5; i++) {
		DWORD dwThreadId = 0;
		arhThreads[i] = CreateThread(NULL, 0, ThreadProc, (PVOID)(i + 3), 0, &dwThreadId);
	}


	WaitForMultipleObjects(5, arhThreads, TRUE, INFINITE);
	printf("All sub Thread terminated.....\n");
	getchar();
	for (int i = 0; i < 5; i++) {
		CloseHandle(arhThreads[i]);
	}
}
```

#### WaitForMultipleObjects()의 반환값
* WAIT_FAILED 및 WAIT_TIMEOUT : WaitForSingleObject()에서의 의미와 동일
* 성공적 대기
* bWaitAll == TRUE : WAIT_OBJECT_0가 리턴. 모든 커널 객체가 시그널된 상태
* bWaitAll == FALSE : WAIT_OBJECT_0 ~ WAIT_OBJECT_0 + nCount - 1 사이의 값. 즉 WAIT_OBJECT_0에 커널 객체 배열의 인덱스를 더한 값이다.(물론 실제로 반환되는 값은 시그널된 객체의 인덱스다.)
* 포기된 뮤텍스
* WAIT_ABANDONED_0 ~(WAIT_ABANDONED_0 + nCount - 1) 사이의 값
* bWaitAll == TRUE : 모든 객체가 시그널 상태, 그중하나는 포기된 뮤텍스.
* bWaitAll == FALSE : pHandles 배열 원소 중에서 포기된 인덱스를 반환


### SignalObjectAndWait()
* 해당 커널 객체를 시그널 상태로 만들고, WaitForSingleObject()하는 것과 같은 동작.
* 그러나 하나의 함수이기 때문에 좀 더 원자성이 보장되며, 하나의 함수만 호출함으로써 커널 모드와 유저 모드를 두 번 왔다갔다 하는 비효율(성능저하)을 막는다.
* 상당히 유용하게 쓰이는 함수

#### reference
![](../../../images/Windows_System_Technology/6.PNG)
