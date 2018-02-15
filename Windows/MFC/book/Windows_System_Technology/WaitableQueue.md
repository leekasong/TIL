# 대기가능 큐.

* 동기화가 적용되어 있는 큐입니다.
* 스레드는 큐잉 작업이 있기 전에는 잠들어있다가 신호가 오면 작업을 수행합니다.
* 클래스 형태로 만든 것이며, 아래는 전체 코드입니다.

```
#include <Windows.h>
#include <list>
#define CMD_NONE	-1
#define CMD_EXIT	0
#define CMD_STR		1
#define CMD_POINT	2
#define CMD_TIME	3

class WAIT_QUE {
	struct  NOTI_ITEM{
		LONG _cmd, _size;
		PBYTE _data;

		NOTI_ITEM() {
			_cmd = _size = 0; _data = NULL;
		}
		NOTI_ITEM(LONG cmd, LONG size, PBYTE data) {
			_cmd = cmd; _size = size; _data = data;
		}
	};
	typedef std::list<NOTI_ITEM> ITEM_QUE;

	HANDLE m_hMutex; //
	HANDLE m_hSema; //엔큐, 디큐의 순차성 보호. 잠자기용..
	ITEM_QUE m_queue;

public:
	WAIT_QUE() {
		m_hMutex = m_hSema = NULL;
	}
	~WAIT_QUE() {
		if (m_hMutex) CloseHandle(m_hMutex);
		if (m_hSema) CloseHandle(m_hSema);
	}

public:
	void Init() {
		m_hSema = CreateSemaphore(NULL, 0, LONG_MAX, NULL);
		m_hMutex = CreateMutex(NULL, FALSE, NULL);
	}

	void Enqueue(LONG cmd, LONG size = 0, PBYTE data = NULL) {
		NOTI_ITEM ni(cmd, size, data);

		WaitForSingleObject(m_hMutex, INFINITE);
		m_queue.push_back(ni);
		ReleaseMutex(m_hMutex);

		ReleaseSemaphore(m_hSema, 1, NULL);
	}

	PBYTE Dequeue(LONG& cmd, LONG& size) {
		DWORD dwExitCode = WaitForSingleObject(m_hSema, INFINITE);
		if (dwExitCode == WAIT_FAILED) {
			cmd = CMD_NONE; size = HRESULT_FROM_WIN32(GetLastError());
			return NULL;
		}

		NOTI_ITEM ni;

		WaitForSingleObject(m_hMutex, INFINITE);
		ni = m_queue.front();
		m_queue.pop_front();
		ReleaseMutex(m_hMutex);

		cmd = ni._cmd;
		size = ni._size;
		return ni._data;
	}
};

DWORD WINAPI ThreadProc(PVOID pParam) {
	WAIT_QUE *pWQ = (WAIT_QUE *)pParam;
	DWORD dwThreadId = GetCurrentThreadId();
	while (true) {
		LONG lSize, lCmd;
		PBYTE pData = pWQ->Dequeue(lCmd, lSize);
		if (lCmd == CMD_EXIT) {
			break;
		}

		printf("[%d] Thread reading...\n");
		if (lCmd == CMD_STR) {
			printf("data : %s\n", pData);
			memset(pData, 0, lSize);
		}
		else if (lCmd == CMD_POINT) {
			PPOINT pPoint = (PPOINT)pData;
			printf("data : (%d, %d)\n", pPoint->x, pPoint->y);
		}
		else if (lCmd == CMD_TIME) {
			printf("time!!\n");
		}
	}

	printf(" --- worker Thread exits... \n");
	return 0;
}

void main() {
	WAIT_QUE wq;
	wq.Init();

	HANDLE hThread = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)ThreadProc, &wq, 0, NULL);

	char szIn[512] = { 0 };
	PBYTE pData = new BYTE[512];
	memset(pData, 0, sizeof(BYTE) * 512);
	while (true) {
		memset(szIn, 0, sizeof(szIn));
		scanf("%s", szIn);
		if (_stricmp(szIn, "quit") == 0) {
			break;
		}
		LONG lCmd = CMD_NONE, lSize = 0;


		if (_stricmp(szIn, "time") == 0) {
			lCmd = CMD_TIME;
		}
		else if (_stricmp(szIn, "point") == 0) {
			POINT point;
			point.x = rand() % 1000; point.y = rand() % 1000;
			*((PPOINT)pData) = point;
			lCmd = CMD_POINT;
			lSize = sizeof(point);
		}
		else {
			lSize = strlen(szIn);
			memcpy(pData, szIn, lSize);
			lCmd = CMD_STR;
		}



		wq.Enqueue(lCmd, lSize, pData);
	}

	wq.Enqueue(CMD_EXIT);
	WaitForSingleObject(hThread, INFINITE);

	CloseHandle(hThread);

	delete[] pData;
	printf("main thread exit\n");
	getchar();
}

```

#### reference
![](../../../images/Windows_System_Technology/6.PNG)
