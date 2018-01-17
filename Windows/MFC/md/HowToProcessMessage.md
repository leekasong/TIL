# How to process Windows Message in MFC

#### MFC에서 메시지 처리 방법
* 메시지 핸들러
* WindowProc() : 메시지 핸들러로 처리하지 못하는 메시지를 처리할 수 있게 마련. 물론 메시지 핸들러를 통해 처리할 수 있는 메시지도 처리 가능.
* PreTranslateMessage() : 메시지큐에서 나온 메시지를 핸들러나 WindowProc이 처리하기 전에 필터링한다.

##### 메시지는 항상 메시지큐에 들어가 처리되는 건 아님.
PostMessage() : 메시지큐에 넣는다.
SendMessage() : 메시지 핸들러를 직접 호출한다.
