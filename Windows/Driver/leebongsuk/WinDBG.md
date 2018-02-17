# 커널 디버거

### 덤프파일 만들기
* PS2 키보드 : HKEY_LOCAL_MACHINE\system\CurrentControlSet\Services\\i8042prt\Parameters
* USB 키보드 : HKEY_LOCAL_MACHINE\system\CurrentControlSet\Services\\kbdhid\Parameters
* CrashOnCtrlScroll = REG_DWORD 0x01
* 덤프파일 만들기 : 오른쪽 Ctrl 누른채로, SCROLL LOCK을 따닥 누르면 된다.

#### reference
하제소프트 https://www.youtube.com/channel/UC7Ek4hbKRdWT1idaZLz-F_Q
