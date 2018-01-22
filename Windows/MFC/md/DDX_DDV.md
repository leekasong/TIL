# DDX와 DDV

### DDX

* Dialog Data eXchange
* 변수와 컨트롤의 데이터 동기화 매커니즘

```
void CExamDialogDlg::DoDataExchange(CDataExchange* pDX)
{
	CDialogEx::DoDataExchange(pDX);
	DDX_Text(pDX, IDC_EDIT2, m_str);
	DDV_MaxChars(pDX, m_str, 12);
}

void CExamDialogDlg::OnBnClickedOk()
{
//	m_str = _T("Hi~");
//	UpdateData(FALSE);
	UpdateData();
	AfxMessageBox(m_str);
	CDialogEx::OnOK();
}

```

* UpdateData() 함수를 호출하면 데이터 동기화를 발생시킨다.
* 만약 인자가 기본(TRUE)이면 컨트롤에서 변수로 값을 덮어씌운다.
* 반대로 인자가 FALSE이면 변수에서 컨트롤로 값을 업데이트한다.
* 이러한 일을 가능하게 하는 것은 DoDataExchange() 함수에서 DDX, DDV를 설정했기 때문이다.

### DDV

* Dialog Data Validation
* 위 예제에서 보듯, 데이터에 대한 제한을 걸어주는 역할 (예에서는 문자열의 길이를 최대 12개의 문자들로 제한)

#### reference
Visual C++ 2008 MFC 윈도우 프로그래밍
