# SDI's Execution Flow

```
function : CSdiSeqApp::CSdiSeqApp(void), Thread : Main Thread
function : CSdiSeqApp::InitInstance(void), Thread : Main Thread
function : CSdiSeqDoc::CSdiSeqDoc(void), Thread : Main Thread
function : CMainFrame::CMainFrame(void), Thread : Main Thread
function : CMainFrame::LoadFrame(unsigned int, unsigned long, CWnd *, CCreateContext *), Thread : Main Thread
function : CMainFrame::PreCreateWindow(tagCREATESTRUCTW &), Thread : Main Thread
function : CMainFrame::PreCreateWindow(tagCREATESTRUCTW &), Thread : Main Thread
function : CMainFrame::OnCreate(tagCREATESTRUCTW *), Thread : Main Thread
	function : CMainFrame::OnCreateClient(tagCREATESTRUCTW *, CCreateContext *), Thread : Main Thread
		function : CSdiSeqView::CSdiSeqView(void), Thread : Main Thread
		function : CSdiSeqView::Create(const wchar_t *, const wchar_t *, unsigned long, const tagRECT &, CWnd *, unsigned int, CCreateContext *), Thread : Main Thread
		function : CSdiSeqView::PreCreateWindow(tagCREATESTRUCTW &), Thread : Main Thread
		function : CSdiSeqView::OnCreate(tagCREATESTRUCTW *), Thread : Main Thread
		function : CSdiSeqView::OnShowWindow(int, unsigned int), Thread : Main Thread
	CMainFrame::OnCreateClient() - return
	CMainFrame::OnCreate() - return
function : CSdiSeqDoc::OnNewDocument(void), Thread : Main Thread
function : CSdiSeqView::OnInitialUpdate(void), Thread : Main Thread
function : CMainFrame::OnActivateApp(int, unsigned long), Thread : Main Thread
function : CMainFrame::OnActivate(unsigned int, CWnd *, int), Thread : Main Thread
function : CMainFrame::OnShowWindow(int, unsigned int), Thread : Main Thread
function : CSdiSeqView::GetDocument(void), Thread : Main Thread
function : CSdiSeqApp::Run(void), Thread : Main Thread
	function : CMainFrame::OnClose(void), Thread : Main Thread
		function : CMainFrame::OnShowWindow(int, unsigned int), Thread : Main Thread
		function : CMainFrame::OnActivate(unsigned int, CWnd *, int), Thread : Main Thread
		function : CMainFrame::OnActivateApp(int, unsigned long), Thread : Main Thread
		function : CMainFrame::DestroyWindow(void), Thread : Main Thread
		function : CMainFrame::OnDestroy(void), Thread : Main Thread
		function : CSdiSeqView::OnDestroy(void), Thread : Main Thread
		function : CSdiSeqView::PostNcDestroy(void), Thread : Main Thread
		function : CSdiSeqView::~CSdiSeqView(void), Thread : Main Thread
		function : CMainFrame::OnNcDestroy(void), Thread : Main Thread
			function : CMainFrame::PostNcDestroy(void), Thread : Main Thread
			function : CMainFrame::~CMainFrame(void), Thread : Main Thread
		CMainFrame::OnNcDestroy() - return
		function : CSdiSeqDoc::~CSdiSeqDoc(void), Thread : Main Thread
	CMainFrame::OnClose() - return
	function : CSdiSeqApp::ExitInstance(void), Thread : Main Thread
CSdiSeqApp::Run() - return

```

#### reference
Visual C++ 2008 MFC 윈도우 프로그래밍
