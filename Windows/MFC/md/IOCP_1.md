# iocp - 1

* 커널객체
* 비동기 입출력시, 현재 입출력하지 않는 스레드는 재우고, 입출력 완료 통보가 왔을 때 깨워서 처리하게끔 하는 기능
* 스레드풀로 관리됨. 즉 미리 생성할 스레드가 몇 개인지가 IOCP를 통한 비동기 입출력의 성능을 좌우한다.
* 미리 생성하는 스레드가 많을수록 스레드 컨텍스트 스위칭은 빈번하게 발생하지만, 입출력이 많을 때는 효율적이다.
* 그러나 입출력이 적다면 스레드를 적게 만드는 것이 스위칭을 줄이기 때문에 더 성능이 좋을 수 있다.

### 예제

* 예제는 에코서버이다.
* 서버 스레드가 돌면, 입출력 완료 처리할 스레드를 생성.
* 비동기 소켓을 열고 바인드 후 CreateCompletePort()로 소켓과 CompletionPort 객체를 연결시킨다.
* 입출력 완료 처리 스레드에서는 GetQueuedCompletionState()로 입출력 완료된 클라이언트 정보를 얻은 후, 필요한 동작을 수행한다.

```

// stdafx.cpp : source file that includes just the standard includes
// ExamIOCP_Serv.pch will be the pre-compiled header
// stdafx.obj will contain the pre-compiled type information

#include "stdafx.h"
#include "CClientData.h"

SOCKET g_hSockServer = NULL;
CPtrList g_ptrListClient;
UINT CompletionThread(LPVOID pComPort);

UINT StartServer(LPVOID pParam) {
	SOCKET hClientSocket = NULL;
	SOCKADDR_IN ServerAddr = { 0 };
	SOCKADDR_IN ClientAddr = { 0 };

	//특정 파일에 연결된 것이 아님 -> 두번재인자 반드시 NULL, 세번째인자는 무시. 네 번째 인자 : IOCP에 사용될 스레드의 최대 갯수 -> 0이면 시스템이 허용하는 최대 갯수
	//CompletionPort 객체 생성
	HANDLE hCompletionPort = ::CreateIoCompletionPort(INVALID_HANDLE_VALUE, NULL, 0, 0);


	if (hCompletionPort == NULL) {
		CString err;
		err.Format(_T("IOCP 생성 실패 : %d\n"), GetLastError());
		AfxMessageBox(err);
		return 10;
	}

	//스레드 풀에 들어갈 스레드들 생성
	for (int i = 0; i < 3; i++) {
		AfxBeginThread(CompletionThread, (LPVOID)hCompletionPort);
	}

	//비동기 서버 소켓 생성
	g_hSockServer = ::WSASocket(PF_INET, SOCK_STREAM, 0, NULL, 0, WSA_FLAG_OVERLAPPED); //3~5인자는 잘 사용되지 않음.
	if (g_hSockServer == INVALID_SOCKET) {
		AfxMessageBox(_T("서버 소켓을 생성하는 데 실패했습니다."));
		::CloseHandle(hCompletionPort);
		ExitProcess(0);
		return 20;
	}

	//서버 소켓 설정 및 접속 대기
	ServerAddr.sin_family = AF_INET;
	ServerAddr.sin_addr.S_un.S_addr = ::htonl(INADDR_ANY);
	ServerAddr.sin_port = ::htons(21000);

	if (::bind(g_hSockServer, (SOCKADDR *)&ServerAddr, sizeof(SOCKADDR_IN)) == SOCKET_ERROR) {
		AfxMessageBox(_T("bind error : 포트 사용중"));
		::CloseHandle(hCompletionPort);
		::ExitProcess(0);
		return 30;
	}

	if (::listen(g_hSockServer, 5) == SOCKET_ERROR) {
		AfxMessageBox(_T("클라이언트의 접속을 받을 수 없음.."));
		::ExitProcess(0);
		::CloseHandle(hCompletionPort);
		return 40;
	}

	//클라이언트 접속 처리
	DWORD dwFlag = 0;
	int nAddrSize = sizeof(SOCKADDR_IN);
	DWORD dwReceiveSize = 0;

	while (true) {
		hClientSocket = ::accept(g_hSockServer, (SOCKADDR*)&ClientAddr, &nAddrSize);

		if (hClientSocket == INVALID_SOCKET) break;

		CClientData* pNewClientData = new CClientData;
		pNewClientData->m_hSocket = hClientSocket;
		memcpy(&(pNewClientData->m_Address), &ClientAddr, nAddrSize);
		//객체의 주소를 클라이언트의 식별값으로 사용하기 위해 저장.
		pNewClientData->m_IOData.pClientData = (LPVOID)pNewClientData;

		//OVERLAPPED socket과 Completion Port 연결
		::CreateIoCompletionPort((HANDLE)pNewClientData->m_hSocket, hCompletionPort, (DWORD)&pNewClientData->m_IOData,
			0);

		g_ptrListClient.AddTail(pNewClientData);

		dwReceiveSize = 0;
		//비동기 수신
		::WSARecv(
			pNewClientData->m_hSocket,
			&pNewClientData->m_IOData.WsaBuffer,
			1,
			&dwReceiveSize,
			&dwFlag,
			&pNewClientData->m_IOData.Overlapped,
			NULL
		);

		if (WSAGetLastError() != WSA_IO_PENDING) {
			AfxMessageBox(_T("통신 에러"));
			::CloseHandle(hCompletionPort);
			::ExitProcess(0);
			return 50;
		}
	}

	::CloseHandle(hCompletionPort);
	return 0;
}

//입출력이 있을 때 활성화됨.
UINT CompletionThread(LPVOID pComPort) {
	CString tmp;
	HANDLE hCompletionPort = (HANDLE)pComPort;
	DWORD dwTransferredSize = 0;
	DWORD dwFlag = 0;

	IO_DATA *pIOData = NULL;
	CClientData* pClientData = NULL;
	LPOVERLAPPED pOverlapped = NULL;
	while (true) {
		pIOData = NULL;
		pClientData = NULL;
		pOverlapped = NULL;

		//입출력이 있는 클라이언트 정보를 수신
		::GetQueuedCompletionStatus(
			hCompletionPort,
			&dwTransferredSize,
			(DWORD *)&pIOData,
			(LPOVERLAPPED *)&pOverlapped,
			INFINITE
		);

		if (pIOData != NULL) pClientData = (CClientData *)pIOData->pClientData;
		else continue;

		if (dwTransferredSize == 0) {
			if (pClientData != NULL) {
				::shutdown(pClientData->m_hSocket, SD_BOTH);
				::closesocket(pClientData->m_hSocket);

				POSITION pos = g_ptrListClient.Find(pClientData);
				if (pos != NULL) g_ptrListClient.RemoveAt(pos);
				delete pClientData;
				continue;
			}
		}

		if (pClientData != NULL) {
			pClientData->m_IOData.WsaBuffer.len = dwTransferredSize;
			//에코서버
			::WSASend(pClientData->m_hSocket, &(pClientData->m_IOData.WsaBuffer), 1, NULL, 0, NULL, NULL);

			//3개 초과의 클라이언트부터는 데이터를 처리 못받는 다는 것을 확인하기 위해 메시지박스로 앞의 3개의 클라이언트 처리를 중단한다.
			AfxMessageBox(_T("스레드 처리 중지 상태"));

			//처리후 다시 recv를 호출하여 다음 입력을 기다린다.
			::ZeroMemory(pClientData->m_IOData.byBuffer, 15000);
			pClientData->m_IOData.WsaBuffer.len = 15000;
			::WSARecv(
				pClientData->m_hSocket,
				&(pClientData->m_IOData.WsaBuffer),
				1,
				NULL,
				&dwFlag,
				&pClientData->m_IOData.Overlapped,
				NULL
			);

		}
	}
	return 0;
}

void StopServer()
{
	/////////////////////////////////////////////////////////////////////////
	//클라이언트의 연결을 받지 않도록 서버 소켓을 닫음.
	if (g_hSockServer != NULL)
	{
		::shutdown(g_hSockServer, SD_BOTH);
		::closesocket(g_hSockServer);

		::Sleep(100);
		g_hSockServer = NULL;
	}

	/////////////////////////////////////////////////////////////////////////
	//연결된 클라이언트 소켓을 닫음.
	POSITION pos = g_ptrListClient.GetHeadPosition();
	CClientData* pClient = NULL;
	while (pos)
	{
		pClient = (CClientData*)g_ptrListClient.GetNext(pos);
		if (pClient != NULL)
		{
			::shutdown(pClient->m_hSocket, SD_BOTH);
			::closesocket(pClient->m_hSocket);
		}
	}
}
```

#### reference
윈도우 프로그래밍 - 최호성