# Mouse Handler

```
void CExamUserMessageDlg::OnMouseMove(UINT nFlags, CPoint point)
void CExamUserMessageDlg::OnLButtonUp(UINT nFlags, CPoint point)
void CExamUserMessageDlg::OnLButtonDown(UINT nFlags, CPoint point)
```
* 위와 같은 마우스 핸들러에서 두번째 인자로 넘어오는 point는 클라이언트 좌표이다.
* 따라서 WindowFromPoint 같은 스크린 좌표를 사용하는 함수를 호출할 때에는 ClientToScreen으로 변환해야 한다.

```
void CExamUserMessageDlg::OnMouseMove(UINT nFlags, CPoint point)
{
    ClientToScreen(&point);
    m_hTargetWnd = ::WindowFromPoint(point);
    if(m_hTargetWnd){
        
    }
}
```
