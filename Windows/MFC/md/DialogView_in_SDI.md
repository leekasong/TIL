# SDI에서 다이얼로그뷰로 작업하기

![](../../images/DialogViewInSDI/1.png)

* TestDialog.cpp에서 doc, view 헤더 삭제

* 그러면 에러 뜨는 런타임 클래스 코드 없애고, 아래와 같은 코드 작성  

![](../../images/DialogViewInSDI/2.png)  

코드
```
CMainFrame *pMainFrame = new CMainFrame();
m_pMainWnd = pMainFrame;
pMainFrame->LoadFrame(IDR_MAINFRAME);
```

* 클래스 위저드로, CFormView를 상속받는 MFC 클래스를 만든다.(MainView라고 하겠다)
* MainFrm.h에 아래와 같이 멤버를 선언

![](../../images/DialogViewInSDI/2.png)  

* 그리고 MainFrm.cpp의 OnCreate()에서 아래와 같이 코드를 작성

![](../../images/DialogViewInSDI/3.png)  

코드
```
CCreateContext ccx;
ccx.m_pNewViewClass = RUNTIME_CLASS(MainView);
m_pMainView = DYNAMIC_DOWNCAST(MainView, CreateView(&ccx));
m_pMainView->ShowWindow(SW_SHOW);
m_pMainView->OnInitialUpdate();
SetActiveView(m_pMainView);

m_pMainView->ResizeParentToFit(FALSE);
```

#### reference
http://nowonbun.tistory.com/195
