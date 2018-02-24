# MFC 구조 - 객체 간의 접근방법

* SDI 응용프로그램은 '응용프로그램 자체 객체', '메인 프레임', '클라이언트 뷰', '문서'로 구성된다.
* 이들은 각각 클래스로 만들어져 있다.
* 따라서 이러한 구조에서 각 객체의 주소를 가져오는 방법이 중요하다.

![](../../images/MFC_structure_3/1.png)  

* AfxGetMainWnd() : 프로그램의 최상위 프레임 윈도우의 주소 반환
* AfxGetApp() : 응용 프로그램 자체 객체의 주소 반환
* GetActiveView() : 액티브된 뷰 반환
* GetActiveDocument() : 활성화된 문서 반환


### reference
Visual C++ 2008 MFC 윈도우 프로그래밍
