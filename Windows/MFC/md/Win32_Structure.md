# Win32 Structure


## WinMain(){
### 1. window class register
### 2. window create => CApp::InitInstance()
### 3. Main Event Loop => CApp::Run()
### *-> dispatchWindow in CApp::Run() call the callback function ( WinProc() )*
### 4. WinProc() Process Message
### 5. Exit => CApp::ExitInstance()
## }

#### reference
https://www.youtube.com/channel/UCdGTtaI-ERLjzZNLuBj3X6A
