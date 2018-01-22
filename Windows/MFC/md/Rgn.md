# CRgn

```
void CExamRgnView::OnPaint()
{
	CPaintDC dc(this); // device context for painting

	CRect rectLeft = CRect(50, 50, 250, 150);
	CRect rectRight = CRect(250, 50, 450, 150);
	dc.FillSolidRect(&rectLeft, RGB(192, 0, 0));
	dc.FillSolidRect(&rectRight, RGB(192, 192, 192));

	CRgn rgnLeft, rgnRight;
	rgnLeft.CreateRectRgnIndirect(&rectLeft);
	rgnRight.CreateRectRgnIndirect(&rectRight);
	LOGFONT lf;
	::ZeroMemory(&lf, sizeof(LOGFONT));
	wsprintf(lf.lfFaceName, _T("%s"), _T("Arial Black"));
	lf.lfHeight = 70;
	CFont font;
	font.CreateFontIndirectW(&lf);
	CFont *p_old_font = dc.SelectObject(&font);
	dc.SetBkMode(TRANSPARENT);
	dc.SetTextColor(RGB(192, 192, 192));
	dc.SelectClipRgn(&rgnLeft);
	dc.TextOutW(60, 65, _T("TEST STRING"));

	dc.SetTextColor(RGB(192, 0, 0));
	dc.SelectClipRgn(&rgnRight);
	dc.TextOutW(60, 65, _T("TEST_STRING"));

	dc.SelectObject(p_old_font);
}

```

![](../../images/Rgn/1.png)


```
void CExamRgn2View::OnPaint()
{
	CPaintDC dc(this); // device context for painting
	CImage image;
	image.Load(_T("1.jpg"));
	CRect m_rect;
	GetClientRect(&m_rect);

	image.AlphaBlend(dc.m_hDC, 0, 0, 50);
	CRgn rgn;
	rgn.CreateEllipticRgn(m_rect_pos.left, m_rect_pos.top, m_rect_pos.right, m_rect_pos.bottom);
	dc.SelectObject(&rgn);
	image.BitBlt(dc.m_hDC, 0, 0);
}


void CExamRgn2View::OnLButtonDown(UINT nFlags, CPoint point)
{
	m_click_flag = !m_click_flag;
	m_rect_pos = CRect(point.x - 100, point.y - 100, point.x + 100, point.y + 100);
	Invalidate();
	CView::OnLButtonDown(nFlags, point);
}


void CExamRgn2View::OnLButtonUp(UINT nFlags, CPoint point)
{
	m_click_flag = FALSE;

	CView::OnLButtonUp(nFlags, point);
}


void CExamRgn2View::OnMouseMove(UINT nFlags, CPoint point)
{
	if (m_click_flag) {
		m_rect_pos = CRect(point.x - 100, point.y - 100, point.x + 100, point.y + 100);
		Invalidate();
	}

	CView::OnMouseMove(nFlags, point);
}

```
![](../../images/Rgn/2.png)
