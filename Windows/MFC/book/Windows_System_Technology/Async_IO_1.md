# 비동기 입출력 1 - 동기화 객체를 통한 통보

### OVERLAPPED 구조체
* Internal : 입출력 상태 코드
* InternalHigh : 완료후 바이트
* offset, offsetHigh : 32비트로 나누어진 64비트 오프셋.
* hEvent : 완료 통보 이벤트
### 기본 코드
* 이 예제는 소스 파일을 읽고 목적 파일에 쓰기를 무한히 반복합니다.
* 비동기 호출을 사용하기 위하여 OVERLAPPED 구조체를 사용했습니다.
* 파일 핸들을 동기화 객체로 사용하기 위하여 hEvent를 NULL로 설정합니다.
* 따라서 입출력완료통보는 파일 핸들의 시그널 상태를 통해 체크할 수 있습니다.

```
#include <Windows.h>
#include <stdio.h>
#ifndef STATUS_END_OF_FILE
#define STATUS_END_OF_FILE 0xc000001L
#endif

#define BUFF_SIZE 66536
void main(int argc, TCHAR *argv[]) {
	if (argc < 3) {
		printf("Usage!\n");
		return;
	}

	HANDLE hSrc = CreateFile(argv[1], GENERIC_READ, 0, NULL, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, NULL);

	if (hSrc == INVALID_HANDLE_VALUE) {
		printf("%s open failed\n", argv[1]);
		return;
	}

	HANDLE hDst = CreateFile(argv[2], GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_FLAG_OVERLAPPED, NULL);
	if (hDst == INVALID_HANDLE_VALUE) {
		printf("%s open failed\n", argv[2]);
		return;
	}

	OVERLAPPED ro = { 0 };
	OVERLAPPED wo = { 0 };
	ro.Offset = ro.OffsetHigh = wo.Offset = wo.OffsetHigh = 0;
	ro.hEvent = wo.hEvent = NULL;

	DWORD dwErrCode = 0;
	BYTE btBuff[BUFF_SIZE] = { 0 };
	while (true) {
		BOOL bIsOk = ReadFile(hSrc, btBuff, sizeof(btBuff), NULL, &ro);

		if (!bIsOk) {
			dwErrCode = GetLastError();
			if (dwErrCode != ERROR_IO_PENDING) {
				break;
			}
		}

		DWORD dwWaitRet = WaitForSingleObject(hSrc, INFINITE);
		if (dwWaitRet != WAIT_OBJECT_0) {
			dwErrCode = GetLastError();
			break;
		}

		if (ro.Internal != 0) {
			if (ro.Internal == STATUS_END_OF_FILE) {
				dwErrCode = 0;
			}
			else {
				dwErrCode = ro.Internal;
			}
		}

		ro.Offset += ro.InternalHigh;
		printf(" => read bytes : %d\n", ro.Offset);

		bIsOk = WriteFile(hDst, btBuff, ro.InternalHigh, NULL, &wo);
		if (!bIsOk) {
			dwErrCode = GetLastError();
			if (dwErrCode != ERROR_IO_PENDING) {
				break;
			}
		}

		dwWaitRet = WaitForSingleObject(hDst, INFINITE);
		if (dwWaitRet != WAIT_OBJECT_0) {
			dwErrCode = GetLastError();
			break;
		}
		wo.Offset += wo.InternalHigh;

		printf(" <= wrote bytes : %d\n", wo.Offset);
	}

	CloseHandle(hSrc);
	CloseHandle(hDst);
	if (dwErrCode != ERROR_SUCCESS) {
		printf("Error occurred in file copying, code : %d\n", dwErrCode);
	}
	else {
		printf("File Copy successfully completed...\n");
	}

}
```

* 파일 핸들이 아니라 OVERLAPPED 구조체의 필드인 hEvent를 통해 통보를 받고 싶다면, 아래와 같이 변경하면 됩니다.

```
ro.hEvent = CreateEvent(NULL, FALSE, FALSE, NULL);
wo.hEvent = CreateEvent(NULL, FALSE, FALSE, NULL);
...
while(true){
    BOOL bIsOk = ReadFile(hSrc, btBuff, sizeof(btBuff), NULL, &ro);
    ...
    DWORD dwWaitRet = WaitForSingleObject(ro.hEvent, INFINITE);    
}

```

* 또한 GetOverlappedResult() 함수를 사용하면 결과값을 얻는 코드가 간단해진다.  
* 그러나 WaitForMultipleObjects() 처럼 여러 개의 동기화 객체의 통보를 받을 수 없다.

```
DWORD dwReadBytes = 0;
bIsOk = GetOverlappedResult(hSrc, &ro, &dwReadBytes, TRUE);
if(!bIsOk){
    dwErrCode = GetLastError();
    if(dwErrCode == ERROR_HANDLE_ROF){
        dwErrCode = 0;
    }
    break;
}
ro.Offset += dwReadBytes;
```

#### reference
![](../../../images/Windows_System_Technology/6.PNG)
