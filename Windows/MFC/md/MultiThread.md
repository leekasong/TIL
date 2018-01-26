# Multi Thread

### 스레드를 사용하지 않는 경우

```
void CExamThread1Dlg::OnBnClickedButton1()
{
	TCHAR szWinPath[MAX_PATH];
	::GetWindowsDirectory(szWinPath, MAX_PATH);
	lstrcat(szWinPath, _T("\\notepad.exe"));
	SHELLEXECUTEINFO sei;
	::ZeroMemory(&sei, sizeof(SHELLEXECUTEINFO));
	sei.cbSize = sizeof(sei);
	sei.hwnd = NULL;
	sei.lpFile = szWinPath;
	sei.nShow = SW_SHOW;
	sei.fMask = SEE_MASK_NOCLOSEPROCESS | SEE_MASK_NO_CONSOLE;
	sei.lpVerb = _T("open");
	sei.lpParameters = NULL;

	if (::ShellExecuteEx(&sei)) {
		::WaitForSingleObject(sei.hProcess, INFINITE);
	}
}
```
#### sei.fMask = SEE_MASK_NOCLOSEPROCESS | SEE_MASK_NO_CONSOLE;
* SEE_MASK_NOCLOSEPROCESS : 프로세스를 성공적으로 만들었을 경우, 그 값이 sei.hProcess에 저장되도록 한다.
* SEE_MASK_NO_CONSOLE : 새로운 윈도우를 만들어 프로세스를 실행한다.  

#### ::WaitForSingleObject(sei.hProcess, INFINITE)
* 첫번째인자는 HANDLE, 두번째인자는 시간.
* 명시한 핸들의 값이 변화될 때까지 명시한 시간만큼 기다린다.  

### 스레드를 사용하는 경우

```
UINT ThreadWaitNotePad(LPVOID pParam) {
	TCHAR szWinPath[MAX_PATH];
	::GetWindowsDirectory(szWinPath, MAX_PATH);
	lstrcat(szWinPath, _T("\\notepad.exe"));

	SHELLEXECUTEINFO sei;
	::ZeroMemory(&sei, sizeof(SHELLEXECUTEINFO));
	sei.cbSize = sizeof(sei);
	sei.hwnd = NULL;
	sei.lpFile = szWinPath;
	sei.nShow = SW_SHOW;
	sei.fMask = SEE_MASK_NOCLOSEPROCESS | SEE_MASK_NO_CONSOLE;
	sei.lpVerb = _T("open");
	sei.lpParameters = NULL;

	if (::ShellExecuteEx(&sei)) {
		::WaitForSingleObject(sei.hProcess, INFINITE);
		AfxMessageBox(_T("메모장 종료!"));
	}

	return 0;
}

void CExamThread1Dlg::OnBnClickedButton1()
{
	CWinThread *p_thread = AfxBeginThread(ThreadWaitNotePad, NULL);
	if (p_thread == NULL) {
		AfxMessageBox(_T("Error : Failed to begin worker-Thread"));
	}
}
```
* 스레드 함수는 항상 예와 같은 프로토타입을 가져야하며, 전역 함수나 스태틱이어야 한다.


#### 중복실행 방지하기

```

//스레드 함수의 일부
if (::ShellExecuteEx(&sei)) {
    ::WaitForSingleObject(sei.hProcess, INFINITE);
    AfxMessageBox(_T("메모장 종료!"));
}
    gp_thread = NULL;
    return 0;
}

void CExamThread1Dlg::OnBnClickedButton1()
{
    if (gp_thread != NULL) {
        AfxMessageBox(_T("이미 실행중"));
        return;
    }
    gp_thread = AfxBeginThread(ThreadWaitNotePad, NULL);
    if (gp_thread == NULL) {
        AfxMessageBox(_T("Error : Failed to begin worker-Thread"));
    }
}
```
* 전역변수를 두어서 처리

#### 스레드 종료

* 스레드로 돌고있는 윈도우보다 원래 윈도우가 먼저 종료될 때, 메모리릭이 발생한다.
* 이유는 이 경우 TerminateThread() API가 내부적으로 발생하여 스레드 관리용 메모리를 제대로 해제하지 못하기 때문
* 이를 해결하기 위해 스레드는 메인 윈도우 종료시점을 파악하여 알아서 종료돼야 한다.

```
//스레드 함수의 일부
if (::ShellExecuteEx(&sei)) {
    HANDLE arhList[2];
    arhList[0] = sei.hProcess;
    arhList[1] = (HANDLE)g_exitEvent;
    DWORD dwResult = ::WaitForMultipleObjects(2, arhList, FALSE, INFINITE);
    if (dwResult == WAIT_OBJECT_0) {
        AfxMessageBox(_T("메모장 종료~"));
    }
    else if (dwResult == WAIT_OBJECT_0 + 1) {
        OutputDebugString(_T("ExamThread1 종료!\n"));
    }
}
gp_thread = NULL;
return 0;

int CExamThread1App::ExitInstance()
{
	g_exitEvent.SetEvent();
	::Sleep(100);

	return CWinApp::ExitInstance();
}
```
* ExitInstance() 이 호출되는 시점은 메인 윈도우가 종료되는 시점 -> 이벤트를 발생시킨다.
* 스레드 함수에서는 WaitForMultipleObjects()를 통해 이벤트와 스레드 윈도우의 종료를 모두 기다리도록 만든다.
* 이 함수의 세번째인자가 FALSE이면 기다리는 여러 객체 중 하나만 통지되도 값을 반환한다.

### 참고

* 스레드 생성하는 함수는 세 가지가 있다.  
* AfxBeginThread() : MFC 클래스를 이용할 때 쓰면 좋다.
* CreateThread() : Windows API를 이용할 때 쓰면 좋다. 물론 AfxBeginThread()를 써도 크게 문제되진 않는다.
* \_beginthread() : C 런타임 라이브러리 함수를 사용할 때 쓰면 좋다.

### 스레드 제어

```
/* 스레드 클래스(CWinThread 상속) */
BOOL CUIThread::InitInstance()
{
	CMainFrame *p_main = new CMainFrame;
	if (!p_main) {
		return FALSE;
	}
	m_pMainWnd = p_main;

	p_main->LoadFrame(IDR_MAINFRAME, WS_OVERLAPPEDWINDOW | FWS_ADDTOTITLE, NULL, NULL);
	p_main->ShowWindow(SW_SHOW);
	p_main->UpdateWindow();
	return TRUE;
}

int CUIThread::ExitInstance()
{
	gp_thread = NULL;
	return CWinThread::ExitInstance();
}

/* 메뉴 버튼으로 스레드 제어 */
void CMainFrame::OnUithreadCreate()
{
	if (gp_thread != NULL) {
		AfxMessageBox(_T("사용자 인터페이스 스레드가 실행중"));
		return;
	}

	gp_thread = AfxBeginThread(RUNTIME_CLASS(CUIThread));
}
void CMainFrame::OnUithreadResume()
{
	if (gp_thread != NULL) {
		gp_thread->ResumeThread();
	}
}


void CMainFrame::OnUithreadSuspend()
{
	if (gp_thread != NULL) {
		gp_thread->SuspendThread();
	}
}


void CMainFrame::OnUithreadExit()
{
	if (gp_thread != NULL) {
		gp_thread->PostThreadMessageW(WM_QUIT, NULL, NULL);
	}
}
```

#### reference
Visual C++ 2008 MFC 윈도우 프로그래밍
