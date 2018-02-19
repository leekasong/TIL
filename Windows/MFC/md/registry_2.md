# 레지스트리로 설치된 소프트웨어 목록 가져오기

* 프로그램에서 현재 컴퓨터의 정보를 가져올 때 레지스트리를 참조합니다.
* 예를 들어, 제어판의 프로그램 추가/삭제에서 나오는 설치된 프로그램 목록 같은 프로그램을 만들 때 그렇죠.

### 코드
* 원하는 정보가 레지스트리에 어디에 위치해 있는지 알고 있다면 알고리즘은 간단합니다.
* 그 위치의 키를 열어서, 데이터를 읽어오면 됩니다.
* 아래는 설치된 프로그램 목록을 가져오는 코드입니다.  

![](../../images/Registry_2/1.PNG)  

![](../../images/Registry_2/2.PNG)  

![](../../images/Registry_2/3.PNG)  

```
void CExamInsalledSWDlg::OnBnClickedButtonShow()
{
	if (m_list.GetCount() > 0) m_list.ResetContent();

    //설치된 목록이 있는 레지스트리
    //사실 이 경로는 32비트 운영체제에 경로이나, API 내부에서 64비트도 호환해주는 것같습니다.
	CString keyPath = _T("Software\\Microsoft\\Windows\\CurrentVersion\\Uninstall");
	HKEY hKey = NULL, hSubKey = NULL;
    //레지스트리의 키값에 대한 핸들을 가져온다.
	DWORD dwResult = ::RegOpenKeyEx(HKEY_LOCAL_MACHINE, keyPath, 0, KEY_READ, &hKey);
	if (dwResult != ERROR_SUCCESS) {
		return;
	}
	TCHAR szKeyName[MAX_PATH] = { 0 };
	BYTE szSWName[_MAX_FNAME] = { 0 };
	DWORD dwIndex = 0;
	DWORD dwBufferSize = MAX_PATH;
	DWORD dwLength = _MAX_FNAME;
	CString subKeyPath = _T("");
    //가져온 핸들로 모든 항목들을 열거한다.
    //이 때 반드시 dwBufferSize가 szKeyName가 담을 수 있는 최대 크기를 명시해야 한다.
	while (ERROR_NO_MORE_ITEMS != RegEnumKeyEx(hKey, dwIndex, szKeyName, &dwBufferSize, NULL, NULL, NULL, NULL)) {
		subKeyPath = keyPath;
		subKeyPath += _T("\\");
		subKeyPath += szKeyName;

		dwResult = RegOpenKeyEx(HKEY_LOCAL_MACHINE, subKeyPath, 0, KEY_READ, &hSubKey);
		if (dwResult != ERROR_SUCCESS) {
			CString err = _T("");
			err.Format(_T("RegOpenKeyEx() err : %d"), GetLastError());
			AfxMessageBox(err);
			return;
		}

        //DisplayName이 설치된 프로그램 이름
        //dwLength는 szSWName이 담을 수 있는 최대 크기로 명시해야 한다.
		if (ERROR_SUCCESS == ::RegQueryValueEx(hSubKey, _T("DisplayName"), NULL, NULL, szSWName, &dwLength)) {
			m_list.InsertString(-1, (TCHAR *)szSWName);
		}

		dwIndex++;
		::ZeroMemory(szSWName, _MAX_FNAME);
		::ZeroMemory(szKeyName, MAX_PATH);
		dwBufferSize = MAX_PATH;
		dwLength = _MAX_FNAME;
	}

	::RegCloseKey(hKey);

}

```

* 여기서 주의할 점은 RegQueryValueEx()인데요. 주석에서 명시한대로, dwLength는 최대크기를 주어야 합니다.
* 만약 0을 주면, 정보가 담기지 않아서 함수가 실패하게 됩니다.


#### reference
윈도우 프로그래밍 - 최호성
