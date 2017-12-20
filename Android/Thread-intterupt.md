# Thread.Intterupt()
How to use interrupt() in Android.<br />

### Interrupt & sleep
don't work intterupt() while thread is sleeping. it thorws InterruptedExeception. <br />
so, re-call interrupt() in 'catch' block as shown below

```
if(mThread == null) {
            mThread = new Thread(){
                @Override
                public void run() {
                    super.run();
                    //check if target thread has received the interrupt.
                    while(!isInterrupted()) {
                        try{
                            Log.i("kslee", ""+mCount);
                            mCount++;
                            Thread.sleep(1000);
                        }catch (Exception e){
                            //re-interrupt for thread to be killed
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
