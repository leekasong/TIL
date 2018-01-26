# IPC

### 공유메모리
```
/* 멤버변수 */
HANDLE m_hMap;
TCHAR *mp_sm;

/* 공유메모리 할당 */
BOOL CExamSharedMemoryApp::InitSharedMemory()
{
	m_hMap = ::CreateFileMapping(INVALID_HANDLE_VALUE, NULL,
		PAGE_READWRITE, 0, sizeof(TCHAR) * 128,
		_T("IPC_TEST_SHARED_MEMORY"));
	if (::GetLastError() == ERROR_ALREADY_EXISTS) {
		m_hMap = ::OpenFileMapping(FILE_MAP_ALL_ACCESS, FALSE, _T("IPC_TEST_SHARED_MEMORY"));
	}

	if (m_hMap == NULL) {
		AfxMessageBox(_T("Error : failed to create file mapping"));
		return FALSE;
	}

	mp_sm = (TCHAR *)::MapViewOfFile(m_hMap, FILE_MAP_ALL_ACCESS,
		0, 0, sizeof(TCHAR) * 128);
	if (!mp_sm) {
		AfxMessageBox(_T("Error : failed to create file mapping"));
		return FALSE;
	}

	return TRUE;
}
```
* CreateFileMapping() : 공유 메모리 HANDLE 반환
* 4번째, 5번째는 64GB까지의 메모리를 사용할 수 있도록, 상위 메모리, 하위 메모리로 나누어서 기입할 수 있게 만들어놓음. 즉 예제에서는 상위 메모리는 사용하지 않고, 하위 메모리만 사용한다.
* MapViewOfFile() : 공유 메모리를 유저 모드에서 접근할 수 있는 메모리로 변환

```
UINT CExamSharedMemoryApp::ThreadSharedMemory(LPVOID pParam)
{
	DWORD dwResult = WAIT_OBJECT_0 + 1;
	HANDLE arhList[2];
	arhList[0] = theApp.m_exitEvent;
	arhList[1] = theApp.m_readEvent;

	while (dwResult == WAIT_OBJECT_0 + 1) {
		dwResult = ::WaitForMultipleObjects(2, arhList, FALSE, INFINITE);
		if (dwResult == WAIT_OBJECT_0) {
			break;
		}
		else if (dwResult == WAIT_OBJECT_0 + 1) {
			theApp.m_pMainWnd->PostMessageW(UM_RECV_EVENT);
			::Sleep(10);
		}
	}

	return 0;
}
```

* MFC는 스레드에 안전하지 않으므로, 직접적인 멤버 함수 호출이나 멤버 접근보다 메시지를 날려주는 것이 좋다. 따라서 이벤트 발생시 미리 등록한 사용자 정의 메시지를 날린다.


```
void CExamSharedMemoryDlg::OnBnClickedButton1()
{
	if (!UpdateData(TRUE) || m_str.GetLength() <= 0) return;


	if (theApp.m_mutex.Lock(1000)) {
		CEvent send(FALSE, TRUE, _T("IPC_READ_SHAREDMEMORY"));
		wsprintf(theApp.mp_sm, _T("%s"), m_str);

		send.SetEvent();
		::Sleep(1);
		send.ResetEvent();
		theApp.m_mutex.Unlock();
	}
	else {
		AfxMessageBox(_T("Error : Lock()"));
		return;
	}
	m_str = _T("");
	UpdateData(FALSE);

}
```
* 같은 이름의 이벤트 객체를 생성하여 이벤트를 설정한다.

### 클립보드

```
void CExamClipDlg::OnBnClickedButton1()
{

	if (m_str.GetLength() <= 0) return;

	HGLOBAL h_global_memory;
	TCHAR *pszBuffer = NULL;

	if (!::OpenClipboard(m_hWnd)) {
		AfxMessageBox(_T("Error : Failed to open clip"));
		return;
	}

	int nLength = (m_str.GetLength() + 1) * sizeof(TCHAR);
	h_global_memory = ::GlobalAlloc(GMEM_MOVEABLE | GMEM_ZEROINIT, nLength);

	if (!h_global_memory) {
		AfxMessageBox(_T("Error : Failed to allocate global memory"));
		::CloseClipboard();
		return;
	}

	pszBuffer = (TCHAR *)::GlobalLock(h_global_memory);
	wsprintf(pszBuffer, _T("%s"), m_str);
	::GlobalUnlock(h_global_memory);

	::EmptyClipboard();
	::SetClipboardData(CF_UNICODETEXT, h_global_memory);
	::CloseClipboard();
}


void CExamClipDlg::OnBnClickedButton2()
{
	if (!::IsClipboardFormatAvailable(CF_UNICODETEXT)) return;
	if (!::OpenClipboard(m_hWnd)) return;
	HGLOBAL h_global_memory = ::GetClipboardData(CF_UNICODETEXT);
	if (h_global_memory) {
		TCHAR *pszBuffer = (TCHAR *)::GlobalLock(h_global_memory);
		if (pszBuffer) {
			m_str.Format(_T("%s"), pszBuffer);
			::GlobalUnlock(h_global_memory);
		}
	}

	::CloseClipboard();
	UpdateData(FALSE);


}
```
* GlobalAlloc() : 클립보드 메모리 할당
* GlobalLock() : 클립보드 메모리를 유저 모드 메모리로 사용할 수 있도록 변환과 동시에 운영체제가 메모리 값을 바꾸지 못하도록 Lock을 건다.
* SetClipboardData()를 통해 값을 넣는다.


#### reference
Visual C++ 2008 MFC 윈도우 프로그래밍
