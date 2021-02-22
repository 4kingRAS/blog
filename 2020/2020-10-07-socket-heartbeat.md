socket 心跳机制
---
一般有两种方法，应用层自己实现心跳，另一种利用TCP的`KeepAlive`

`KeepAlive`的定时是7200秒，很蠢，所以一般都是应用层自己实现。

思路很简单：一般是服务端listen， 客户端连上， 因为网络等问题断开。此时服务端的socket是不知道的，所以还在不断重传或阻塞在`recv`。所以需要一个定时任务，超过某个时间服务端收不到这个包就默认客户端已经断网。



```java
public class MyCallable implements Callable {
    private ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(5);
    private long timeDelay = 5;
    public void excute(){
        scheduledExecutorService.submit(this);
    }
    public void stop(){
        scheduledExecutorService.shutdown();
    }
    public void setTimeDelay(long timeDelay){
        this.timeDelay = timeDelay;
    }
    @Override
    public Object call() throws Exception {
        //一些业务逻辑，也就是你要循环执行的代码
        System.out.println("my call Executed!");
        scheduledExecutorService.schedule(this, timeDelay, TimeUnit.SECONDS);
        return "Called!";
    }
}  
```