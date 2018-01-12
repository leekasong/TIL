# MemoryDC 사용하기

1. 호환할 DC를 생성 </br>
* 메모리 DC가 호환 연동 </br>
* 호환할 비트맵 생성 </br>
* 메모리 DC가 비트맵 선택 </br>
* 바탕색 칠하기 </br>

#### OnInitDialog()

```

	CClientDC dc(this);
	mp_mem_dc = new CDC();
	mp_mem_dc->CreateCompatibleDC(&dc);
    
    //좌표 변환 : Window 좌표 구하고, Client 영역의 좌표로 변환.
	CRect r;
	GetWindowRect(r);
	ScreenToClient(r);
	m_w = r.Width();
	m_h = r.Height();

	CBitmap bmp;
	bmp.CreateCompatibleBitmap(&dc, m_w, m_h);
	mp_mem_dc->SelectObject(bmp);
	mp_mem_dc->FillSolidRect(r, RGB(200, 200, 200));

	mp_mem_dc->FillSolidRect(10, 10, 20, 20, RGB(255, 0, 0));
    
    ```
    
    
#### OnPaint()
```
    dc.BitBlt(0, 0, m_w, m_h, mp_mem_dc, 0, 0, SRCCOPY);

```
    
    