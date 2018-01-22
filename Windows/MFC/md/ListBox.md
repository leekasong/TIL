# ListBox

### 입력

```
void CExamListComboDlg::OnBnClickedInsertBtn()
{
	UpdateData();
	m_list.InsertString(0, m_strInput);
}


void CExamListComboDlg::OnBnClickedAddBtn()
{
	UpdateData();
	m_list.AddString(m_strInput);
}

```
* InsertString() : 명시한 인덱스에 문자열을 넣는다.
* AddString() : 리스트박스의 기본설정인 내림차순 정렬에 따라 문자열을 넣는다.

InsertString()  
![](../../images/ListBox/1.png)  

AddString()  
![](../../images/ListBox/2.png)  


### 찾기

```
 int nIndex = m_list.FindString(-1, m_strInput);
 if(nIndex != LB_ERR){
	 CString strFind;
	 m_list.GetText(nIndex, strFind);
 }

```
* FindString() : 찾으려는 문자열을 포함하거나 일치하는 문자열을 검색
* 첫 번째 인자는 검색 시작 인덱스 - 1로, -1로 주면 모든 범위를 검색 대상으로 삼는다

```
 int nIndex = m_list.FindStringExact(-1, m_strInput);
 if(nIndex != LB_ERR){
	 CString strFind;
	 m_list.GetText(nIndex, strFind);
 }

```
* FindStringExact() : 찾으려는 문자열과 정확히 일치하는 문자열을 검색

#### reference
Visual C++ 2008 MFC 윈도우 프로그래밍
