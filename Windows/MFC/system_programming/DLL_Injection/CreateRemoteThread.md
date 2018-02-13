# DLL Injection by RemoteThread

### DLL 인젝션
* 윈도우즈에서는 다른 프로세스에 자신이 만든 코드를 주입할 수 있습니다.
* 코드를 DLL 파일로 만들어서 해당 프로세스에 주입하는 것입니다. 이것을 DLL Injection이라고 부릅니다.

### 구현 방법
* 여기서는 RemoteThread를 이용하는 방법을 설명합니다.


### 인젝션 알고리즘

1. Kernel32.dll에 있는 LoadLibrary()의 주소를 알아낸다.
2. 자기의 dll 경로를 알아낸다.
3. 타깃 프로세스의 핸들을 통해 가상 메모리의 일부를 할당한다.
4. 할당한 메모리에 본인 dll을 카피한다.
5. CreateRemoteThread()를 호출하여 해당 프로세스(스레드)가 LoadLibrary()를 실행하고, 그 인자로 자신의 dll을 사용하게 한다.
6. 스레드가 완료될 때까지 기다린 후, exitCode를 통해 LoadLibrary()의 핸들을 저장해둔다.

### 이젝션 알고리즘

* 이젝션도 인젝션과 동일한 흐름으로 진행됩니다.
* 다만, exitCode로 받은 LoadLibrary()의 핸들을 통해 FreeLibrary()로 해제해주면 됩니다.

### 코드

```
HMODULE CExamUserMessageDlg::DLLInjection(const char *pDllName, DWORD targetPid) {
	char szFilePath[MAX_PATH + _MAX_FNAME] = { 0 };
	HMODULE(__stdcall *pLoadLibrary)(LPCSTR); pLoadLibrary = NULL;
	HMODULE hKernel32 = NULL;
	HANDLE hTargetProcess = NULL;
	HANDLE hTargetThread = NULL;
	PVOID pDllAddr = NULL;
	DWORD exitCode = NULL;
	DWORD errorCode = 0;
	CString errorText = _T("");
	::GetCurrentDirectoryA(MAX_PATH, szFilePath);
	::lstrcatA(szFilePath, "\\");
	::lstrcatA(szFilePath, pDllName);


	hKernel32 = ::GetModuleHandleA("Kernel32");
	if (hKernel32 == NULL) goto PrintError;
	

	pLoadLibrary = (HMODULE(__stdcall *)(LPCSTR))::GetProcAddress(hKernel32, "LoadLibraryA");
	if (pLoadLibrary == NULL) goto PrintError;
	

	hTargetProcess = ::OpenProcess(PROCESS_ALL_ACCESS, 0, targetPid);
	if (hTargetProcess == NULL) goto PrintError;
	

	pDllAddr = ::VirtualAllocEx(hTargetProcess, NULL, MAX_PATH + _MAX_FNAME, MEM_COMMIT, PAGE_READWRITE);
	if (pDllAddr == NULL) goto PrintError;
	::WriteProcessMemory(hTargetProcess, pDllAddr, szFilePath, lstrlenA(szFilePath), 0);

	hTargetThread = ::CreateRemoteThread(hTargetProcess, NULL, NULL
		, (LPTHREAD_START_ROUTINE)pLoadLibrary, pDllAddr, 0, NULL);
	if (hTargetThread == NULL) goto PrintError;
	
	if (::WaitForSingleObject(hTargetThread, INFINITE) == WAIT_OBJECT_0) {
		::GetExitCodeThread(hTargetThread, &exitCode);
	}
	else AfxMessageBox(_T("Remote Thread가 종료되지 않았습니다..."));

	

	

PrintError:
	errorCode = ::GetLastError();
	errorText.Format(_T("error code : %d"), errorCode);
	AfxMessageBox(errorText);
CleanUP:
	if (hTargetThread) {
		::CloseHandle(hTargetThread);
	}
	if (pDllAddr) {
		::VirtualFreeEx(hTargetProcess, pDllAddr, MAX_PATH + _MAX_FNAME, MEM_RELEASE);
	}
	if (hTargetProcess) {
		::CloseHandle(hTargetProcess);
	}
	if (hKernel32) {
		::FreeLibrary(hKernel32);
	}


	return (HMODULE)exitCode;
}
```

```
void CExamUserMessageDlg::MyDLLFree(HMODULE hDLL, DWORD targetPid) {
	HMODULE hKernel32 = ::GetModuleHandleA("Kernel32");
	BOOL(__stdcall *pFreeLibrary)(HMODULE); pFreeLibrary = NULL;
	HANDLE hTargetProcess = NULL;
	PVOID hTargetThread = NULL;
	DWORD exitCode = 0;
	DWORD errorCode = 0;
	CString errorText = _T("");
	if (hKernel32 == NULL) {
		goto PrintError;
	}

	pFreeLibrary = (BOOL(__stdcall *)(HMODULE))::GetProcAddress(hKernel32, "FreeLibrary");
	if (pFreeLibrary == NULL) {
		goto PrintError;
	}

	hTargetProcess = ::OpenProcess(PROCESS_ALL_ACCESS, FALSE, targetPid);
	if (hTargetProcess == NULL) {
		goto PrintError;
	}

	hTargetThread = ::CreateRemoteThread(hTargetProcess, NULL, NULL
		, (LPTHREAD_START_ROUTINE)pFreeLibrary, hDLL, 0, NULL);
	if (hTargetThread == NULL) {
		goto PrintError;
	}
	

PrintError:
	errorCode = ::GetLastError();
	errorText.Format(_T("error code : %d"), errorCode);
	AfxMessageBox(errorText);
CleanUP:
	if (hTargetThread) ::CloseHandle(hTargetThread);
	if (hTargetProcess) ::CloseHandle(hTargetProcess);
	if (hKernel32) ::FreeLibrary(hKernel32);
}
```
