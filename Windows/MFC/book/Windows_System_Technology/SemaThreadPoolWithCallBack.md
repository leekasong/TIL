# 콜백함수를 다루는 스레드 풀 구현하기

### 전체 개요
* 스레드 풀 클래스 구현(SemaThreadPool)
* 사용자가 내용을 채울 수 있는 콜백함수를 등록함으로써 사용자의 코드가 스레드 풀을 통해 실행

### 콜백함수
* 클래스의 멤버로 구현
* 함수포인터와 void * 파라미터를 구조체로 묶어서 관리
* 이 구조체 타입으로 큐잉된다.

### 큐잉

#### 엔큐
* 뮤텍스를 통해 동기화(예시 코드에선 없어도 되긴 함.)
* 엔큐 후 세마포어 시그널링
* 사용자가 본인의 함수를 인자로 넘기면서 엔큐한다.

#### 디큐
* 뮤텍스를 통해 동기화
* 클래스의 static 멤버인 처리함수에서 사용

### 처리함수
* 세마포어의 시그널 기다림(사실 이 두 과정은 디큐에서 해도 됨)
* 시그널을 받으면 디큐
* 데이터는 함수포인터와 파라미터. 따라서 함수포인터를 통해 파라미터를 넘기며 함수 호출
* 즉 사용자가 정의한 함수를 호출하는 것이다.

### 종료
* quit 명령을 받으면 루프에서 빠져나가고, UnInit()을 호출
* 이 함수에서 엔큐 파라미터에 NULL을 넣어서 스레드 개수만큼 호출
* 모든 스레드가 종료하면 메인 스레드 종료

```
#include <Windows.h>
#include <iostream>
#include <cstdio>
#include <list>
#include <string>
using namespace std;
typedef void (WINAPI *FP_WORK)(PVOID);
class SemaThreadPool {
protected:
	typedef struct _WORK_ITEM {
		FP_WORK _pfn_work;
		PVOID _pParam;

		_WORK_ITEM() {
			_pfn_work = NULL;
			_pParam = NULL;
		}

		_WORK_ITEM(FP_WORK pfn_work, PVOID pParam) {
			_pfn_work = pfn_work;
			pParam = pParam;
		}
	} WORK_ITEM;
	typedef WORK_ITEM* PWORK_ITEM;

	static DWORD WINAPI SemaWorkerProc(PVOID pParam);
#define MAX_SEMA_COUNT	10
	HANDLE m_hSema;
	HANDLE m_hMutex;
	list<WORK_ITEM> m_queue;
	int m_ThrCnt;
	PHANDLE m_ThrList;
public:
	SemaThreadPool() {
		m_hMutex = m_hSema = NULL;
	}
	~SemaThreadPool() {
		if (m_hMutex) CloseHandle(m_hMutex);
		if (m_hSema) CloseHandle(m_hSema);
	}
public:
	void Init(int thrCnt = 10) {
		m_hMutex = CreateMutex(NULL, FALSE, NULL);
		m_hSema = CreateSemaphore(NULL, 0, MAX_SEMA_COUNT, NULL);

		m_ThrCnt = thrCnt;

		m_ThrList = new HANDLE[m_ThrCnt];
		for (int i = 0; i < m_ThrCnt; i++) {
			m_ThrList[i] = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)SemaWorkerProc, this, 0, NULL);
		}

	}

	void Uninit() {
		for (int i = 0; i < m_ThrCnt; i++) {
			this->Enqueue(NULL, NULL);
		}
		WaitForMultipleObjects(m_ThrCnt, m_ThrList, TRUE, INFINITE);
		for (int i = 0; i < m_ThrCnt; i++) {
			CloseHandle(m_ThrList[i]);
		}
		delete[] m_ThrList;

	}
	void Enqueue(FP_WORK pfn_work, PVOID pParam) {
		WORK_ITEM wi;
		WaitForSingleObject(m_hMutex, INFINITE);
		wi._pfn_work = pfn_work;
		wi._pParam = pParam;
		m_queue.push_back(wi);
		ReleaseMutex(m_hMutex);

		ReleaseSemaphore(m_hSema, 1, NULL);
	}

	WORK_ITEM Dequeue() {
		WORK_ITEM wi;
		WaitForSingleObject(m_hMutex, INFINITE);
		wi = m_queue.front();
		m_queue.pop_front();
		ReleaseMutex(m_hMutex);

		return wi;
	}



};

DWORD WINAPI SemaThreadPool::SemaWorkerProc(PVOID pParam) {
	//static 멤버라 this로 멤버 접근이 안되기 때문에 파라미터로 넘긴다.
	SemaThreadPool *pSemaTP = (SemaThreadPool *)pParam;

	while (true) {
		DWORD dwWaitCode = WaitForSingleObject(pSemaTP->m_hSema, INFINITE);
		if (dwWaitCode == WAIT_FAILED) {
			printf("WaitForSingleObject() failed in SemaWorkerProc : %d\n", GetLastError());
			break;
		}
		WORK_ITEM wi = pSemaTP->Dequeue();
		if (wi._pfn_work == NULL) {
			break;
		}
		__try {
			wi._pfn_work(wi._pParam);
		}
		__except (EXCEPTION_EXECUTE_HANDLER) {
			printf("Unknown exception occurred : %d\n", GetExceptionCode());
		}

	}

	return 0;
}

void WINAPI MyWorkerCallBack(PVOID pParam) {
	PCHAR pszStr = (PCHAR)pParam;
	DWORD dwThrId = GetCurrentThreadId();
	printf(" == > Thread %d working ... %s\n", dwThrId, pszStr);
	delete pszStr;
}

void main() {
	SemaThreadPool tp;
	tp.Init();

	string str;
	while (true) {
		cin >> str;
		if (str == "quit") {
			break;
		}

		char *pData = new char[str.size() + 1];
		const char *pTemp = str.c_str();
		memcpy(pData, pTemp, str.size() + 1);

		tp.Enqueue(MyWorkerCallBack, pData);
	}

	tp.Uninit();

	printf("Main Thread exit...\n");
	getchar(); getchar();
}
```

#### reference
![](../../../images/Windows_System_Technology/6.PNG)
