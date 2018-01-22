# Notify Message

* 자식 윈도우가 부모 윈도우에게 보내는 메시지
* WM_NOTIFY 메시지의 파라미터로 정보를 전달
* SendMessage() 이용 -> 메시지큐 거치지 않는다

```
//wParam : 컨트롤 윈도우의 리소스 아이디
//lParam : NMHDR 구조체 정보, 즉 전달할 정보
lResult = SendMessage(hwnd,WM_NOTIFY,wParam, lParam);

typedef struct tagNMHDR{
    HWND hwndFrom;
    UINT_PTR idFrom;
    UINT code;
}NMHDR;

```
#### reference
Visual C++ 2008 MFC 윈도우 프로그래밍
