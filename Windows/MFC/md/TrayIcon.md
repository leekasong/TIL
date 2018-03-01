# 트레이 아이콘

* WM_CREATE 핸들러에서 트레이 아이콘을 생성
* WM_CLOSE 핸들러에서 트레이 아이콘 삭제

### 트레이 아이콘 생성 루틴  

```
void CExamTrayDlg::GoToTray() {
	NOTIFYICONDATA nid;
	nid.cbSize = sizeof(nid);
	nid.hWnd = m_hWnd;
	nid.uID = IDR_MAINFRAME;
	nid.uFlags = NIF_MESSAGE | NIF_ICON | NIF_TIP; //어떤 필드가 의미 있는지 명시
	nid.uCallbackMessage = UM_TRAY; //UM_TRAY는 사용자 정의 메시지.
	nid.hIcon = ::LoadIcon(NULL, IDI_APPLICATION); //트레이 아이콘 이미지
	lstrcpy(nid.szTip, L"트레이 윈도우 예제"); //strip
	::Shell_NotifyIcon(NIM_ADD, &nid);

}
```

* uCallbackMessage를 통해 트레이 아이콘에서 받은 명령을 처리한다.
* 이를 위해 사용자 정의 메시지를 이용한다.

### 사용자 정의 메시지 처리
* 오른쪽 마우스 클릭시 메뉴 뜨게 하는 루틴  

```
afx_msg LRESULT CExamTrayDlg::OnUmTray(WPARAM wParam, LPARAM lParam)
{
	if (lParam == WM_RBUTTONDOWN) {
		CMenu menu, *pMenu;
		CPoint point;
		menu.LoadMenuW(IDR_MENU1);
		pMenu = menu.GetSubMenu(0);
		GetCursorPos(&point);
		SetForegroundWindow();
		pMenu->TrackPopupMenu(TPM_LEFTALIGN | TPM_LEFTBUTTON,
			point.x, point.y, this);
		//::PostMessage(m_hWnd, WM_NULL, 0, 0);
	}
	return 0;
}
```

### 종료 루틴

```
void CExamTrayDlg::OnClose()
{
	NOTIFYICONDATA nid;
	nid.cbSize = sizeof(nid);
	nid.hWnd = m_hWnd;
	nid.uID = IDR_MAINFRAME;
	::Shell_NotifyIcon(NIM_DELETE, &nid);

	CDialogEx::OnClose();
}
```
