# 키 메시지 처리하기

### 이론

1. KeyDown 메시지 발생
2. TranslateMessage()가 Key를 분류
3. Key가 문자나 엔터, Escape, Space라면 WM_CHAR~~ 발생
4. Key가 Alt라면 WM_SYSCHAR 발생.

가상 키의 비트  
http://aesire.tistory.com/19

### 다이얼로그에서 KeyDown 메시지 받기
* 컨트롤이 있으면 keyDown 메시지를 받기 어렵다.
* PretranslateMessage()를 이용한다.  

```
BOOL CTestChar2Dlg::PreTranslateMessage(MSG* pMsg)
{
	UINT msg = pMsg->message;
	if (msg == WM_KEYDOWN) {
		CString str;
		int lo = LOWORD(pMsg->wParam);
		//0 key code : 0x30
		if (lo == 0x30) {
			SetDlgItemText(IDC_STATIC, _T("0이 눌렸닷!"));
		}

	}

	return CDialogEx::PreTranslateMessage(pMsg);
}

```
KeyCode 관련 MSDN 링크  
https://msdn.microsoft.com/en-us/library/windows/desktop/dd375731(v=vs.85).aspx
https://msdn.microsoft.com/ko-kr/library/windows/desktop/ms646280(v=vs.85).aspx

### 시스템 키보드 메시지

Alt나 F10 키는 System Key로 분류. 메세지 이름은 WM_SYSKEYDOWN, WM_SYSKEYUP 이다.

```
void CChildView::OnSysKeyDown(UINT nChar, UINT nRepCnt, UINT nFlags)
{
	CString str;
	WORD keyState = ::GetKeyState(VK_SPACE);
	BYTE byHigh = HIBYTE(keyState);

	if (byHigh & 0x01) {
		str += _T("Alt + Space, ");
		keyState = ::GetKeyState(VK_CAPITAL);
		BYTE byLow = LOBYTE(keyState);
		if (byLow & 0x01) {
			str += _T("CAPS ON");
		}
		else {
			str += _T("CAPS OFF");
		}

		AfxMessageBox(str);
	}

	CWnd::OnSysKeyDown(nChar, nRepCnt, nFlags);
}
```

* GetKeyState 함수에 대한 레퍼런스를 찾아보면 사용법을 알 수 있다.
* CAPS 같은 토글 키 처리하는 방법도 나와 있으니 참고하자.

https://msdn.microsoft.com/query/dev14.query?appId=Dev14IDEF1&l=EN-US&k=k(WINUSER%2FGetKeyState);k(GetKeyState);k(DevLang-C%2B%2B);k(TargetOS-Windows)&rd=true


#### Alt와 같이 눌렀는지 확인하는 다른 방법

* WM_SYSCHAR는 Alt와 동시에 키를 눌렀을 때 발생하는 메시지이다.
* TranslateMessage()가 해당 메시지를 발생시킨다.

```
void CChildView::OnSysChar(UINT nChar, UINT nRepCnt, UINT nFlags)
{
	if (nChar == VK_RETURN) {
		AfxMessageBox(_T("Alt + Enter"));
	}
	else if (nChar == 's' || nChar == 'S') {
		AfxMessageBox(_T("Alt + s"));
	}
	else if (nChar == 'x' || nChar == 'X') {
		AfxMessageBox(_T("Alt + x"));
	}

	CWnd::OnSysChar(nChar, nRepCnt, nFlags);
}
```




### 연습하기

1. 방향키를 눌러서 프레임 윈도우의 위치를 바꾸기

```
void CChildView::OnKeyDown(UINT nChar, UINT nRepCnt, UINT nFlags)
{
	TRACE(_T("OnKeyDown()\n"));
	CRect rect;
	int move = 10;
	CWnd *p_parent = GetParentFrame();
	p_parent->GetWindowRect(&rect);
	int nx = 0, ny = 0;

	switch (nChar) {
		case VK_LEFT:
			nx -= 10;
			break;
		case VK_RIGHT:
			nx += 10;
			break;
		case VK_UP:
			ny -= 10;
			break;
		case VK_DOWN:
			ny += 10;
			break;
	}

	p_parent->SetWindowPos(NULL, rect.left + nx, rect.top + ny, 100, 100, SWP_NOSIZE);
	CWnd::OnKeyDown(nChar, nRepCnt, nFlags);
}
```

위 예제를 통해 프레임 윈도우의 핸들을 얻는 방법을 참고하자.





#### reference
Visual C++ 2008 MFC 윈도우 프로그래밍
