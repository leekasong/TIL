# 부팅시 자동실행하기

* 사용자 권한의 프로그램  

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
```

* 등록 루틴  

```
HKEY hKey;
#define SUBKEY	_T("SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run")
CString key_name, file_name;
key_name = _T("leekasong");
DWORD dwRes = RegOpenKeyEx(
    HKEY_CURRENT_USER,
    SUBKEY,
    0,
    KEY_WRITE,
    &hKey
);

if (ERROR_SUCCESS == dwRes) {
    char curPath[MAX_PATH + _MAX_FNAME] = { 0 };
    GetModuleFileNameA(NULL, curPath, MAX_PATH + _MAX_FNAME);
    DWORD dwLength = strlen(curPath) + 1;

    dwRes = RegSetValueExA(hKey, "leekasong", 0, REG_SZ, (PBYTE)curPath, dwLength);
    if (dwRes == ERROR_SUCCESS) AfxMessageBox(_T("등록성공"));
    else AfxMessageBox(_T("등록실패"));

    RegCloseKey(hKey);

}
```
* 안시와 유니코드는 신경써서 작성할것. 안시면 안시, 유니코드면 유니코드로 통일해야 불필요한 디버깅을 하지 않게 된다.
