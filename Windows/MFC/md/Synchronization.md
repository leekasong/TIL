# 커널객체와 동기화

* 동기화용 커널 객체에는 뮤텍스, 세마포어, 이벤트가 있고, 동기화용 사용자 객체에는 크리티컬 섹션이 있다.
* Lock(), Unlock()으로 제어. 데드락을 피하기 위해 Lock()인자에 시간을 명시할 수 있다.
* CEvent 객체의 SetEvent()를 사용하여 메인 윈도우 종료시, 작업 스레드가 종료될 수 있도록 이벤트를 설정해줘야 한다

### 뮤텍스 vs 세마포어 vs  크리티컬섹션
* 크리티컬섹션은 스레드 간의 동기화시 사용한다. 다른 기법보다 가볍다.
* 뮤텍스는 프로세스 간의 동기화시 사용한다. 스레드 간도 가능하지만, 오버헤드가 더 많다.
* 세마포어는 여러 개의 스레드 혹은 프로세스가 자원을 공유해야 할 때 사용한다.

### 이벤트

```
CEvent m_event(FALSE, TRUE, _T("UPDATE_TEST_EVENT"))
```
* 첫째 인자는 이벤트가 설정된 상태로 생성될지 여부
* 둘째 인자는 설정된 상태를 유지할 것인지 여부
* 셋째 인자는 이벤트 객체의 이름으로, 일반적인 문자열은 로그온한 사용자 세션에서 유효하고 'Global\\\\'이라고 붙이면 윈도우 시스템 전체에서 유효하다.
* 셋쩨 인자를 동일하게 사용한 다른 프로그램들에 대해 이벤트를 날려 특정한 동작을 할 수 있도록 만들 수 있다.


#### 중복 실행 방지

```
// InitInstance()
mh_event = ::CreateEvent(NULL, FALSE, FALSE, _T("DUPLICATION_TEST"));

if(::GetLastError() == ERROR_ALREADY_EXISTS){
    AfxMessageBox(_T("ERROR : app is running"));
    return FALSE;
}

//ExitInstance()
if(mh_event) ::CloseHandle(mh_event);
```
