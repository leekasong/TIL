# Thread.Intterupt()
Android에서 interrupt 메서드 사용법입니다.

## Interrupt와 sleep
interrupt와 sleep을 같이 사용할 때 문제가 발생할 수 있습니다.
sleep할 때의 스레드는 interrupt가 먹히지 않기 때문에 예외를 발생시킵니다.
따라서 아래와 같이 코드를 작성해야 합니다.

``` 
if(mThread == null) {
            mThread = new Thread(){
                @Override
                public void run() {
                    super.run();
                    //interrupt를 받았는지 체크
                    while(!isInterrupted()) {
                        try{
                            Log.i("kslee", ""+mCount);
                            mCount++;
                            Thread.sleep(1000);
                        }catch (Exception e){
                            //Interrupt를 다시 걸어준다.
                            this.interrupt();
                            Log.i("kslee", "interrupt()");
                        }
                    }
                }
            };
            mThread.start();
        }
```

#### reference 
> https://stackoverflow.com/questions/9791361/thread-not-interrupting