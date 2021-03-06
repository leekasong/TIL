# CString Usage


### intro
<br/>

* += operator

```
 CString str = _T("Hello");
 str += _T(", CString");
 AfxMessageBox(str);
```

![](../../images/CString/intro.PNG)


* MBCS

```
  char szBuffer[32] = "hello";
  strcat(szBuffer, ", CString");  
  ::MessageBoxA(NULL, szBuffer, "MBCS", MB_OK);

```
![](../../images/CString/MBCS.PNG)

* Unicode

```
  wchar_t wcsBuffer[32] = L"hello";
  wcscat(wcsBuffer, L", CString");

  wsprintf(wcsBuffer, L"ABV");
  ::MessageBox(NULL, wcsBuffer, L"MBCS", MB_OK);
```
![](../../images/CString/Unicode.PNG)


### Member function
<br />

* GetLength() , GetBuffer()

GetLength() returns the number of charters in a CString object
<br/><br/>
GetBuffer() returns a pointer to the characters in the CString.
```
  CString str = _T("Test");
  int nlength = str.GetLength();
  //LPTSTR == TCHAR * : Long pointer

  LPTSTR pBuffer = str.GetBuffer();
```
![](../../images/CString/GetLength.PNG)


* CStringToWCHAR

use wsprintf()

```
  TCHAR wcsBuffer[32] = { 0 };
  CString strData = _T("Hello");

  //Win32 API
  ::wsprintf(wcsBuffer, _T("%s"), strData);
```

before : ![](../../images/CString/CStringToWCHAR_1.PNG)
<br/>after : ![](../../images/CString/CStringToWCHAR_2.PNG)

* Format

```
  CString strTmp;
  strTmp.Format(_T("Data : %d"), 10);
  AfxMessageBox(strTmp);

```

![](../../images/CString/Format.PNG)


* Array[]


```
  CString strTmp = _T("Hello, World!");
  TCHAR ch = strTmp[2];
  strTmp.SetAt(2, _T('L'));
  AfxMessageBox(strTmp);
```
before operator[] : ![](../../images/CString/Array.PNG)
<br/>after operator[] : ![](../../images/CString/Array2.PNG)
<br/>after setAt() :
![](../../images/CString/Array3.PNG)


* Trim

Trim() == trimLeft() + trimRight()
```
  CString strData = _T(" DATA");
  TCHAR wcsBuffer[32];
  strData.TrimLeft();

  wsprintf(wcsBuffer, _T("%s"), strData);
```
![](../../images/CString/Trim.PNG)

* Find

Left() extracts the left part of a string. <br/>
Right() extracts the right part of a string.

```
  CString strPath = _T("C:\\Windows\\System32\\a.exe");
  int nIndex = strPath.Find(_T("a.exe"));
  if (nIndex >= 0) {
    AfxMessageBox(strPath.Left(nIndex));
    AfxMessageBox(strPath.Right(strPath.GetLength() - nIndex ));
  }
```
![](../../images/CString/Find.PNG)
![](../../images/CString/Find2.PNG)


* ReverseFind

```
 CString strPath = _T("C:\\Windows\\System32\\a.exe");
 int nIndex = strPath.ReverseFind(_T('\\'));
 AfxMessageBox(strPath.Left(nIndex));
 AfxMessageBox(strPath.Right(strPath.GetLength() - nIndex - 1));
```
![](../../images/CString/ReverseFind.PNG)
![](../../images/CString/ReverseFind2.PNG)

#### reference
https://www.youtube.com/channel/UCdGTtaI-ERLjzZNLuBj3X6A
