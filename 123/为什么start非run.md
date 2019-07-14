在启动一个线程时，我们是通过调用start方法来启动线程，然后执行run方法中的业务逻辑。

如果直接调用run()方法，相当于还是在当前线程去执行业务逻辑，并没有起新的线程来执行。

start()方法源码如下：

```java
public synchronized void start() {
    if (threadStatus != 0)
            throw new IllegalThreadStateException();
     group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
     }
```

​	可知，start()方法本质上会调用本地方法start0()，会调用JVM启动一个新的本地线程，本地线程的创建会调用当前系统创建线程的方法进行创建，并且线程被执行的时候会回调run方法执行业务逻辑。