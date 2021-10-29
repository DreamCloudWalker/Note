# Runnable和Callable的区别:

* Runnable是自从java1.1就有了，而Callable是1.5之后才加上去的
* Callable规定的方法是call(),Runnable规定的方法是run()
* Callable的任务执行后可返回值，而Runnable的任务是不能返回值(是void)
* call方法可以抛出异常，run方法不可以
* 运行Callable任务可以拿到一个Future对象，表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。通过Future对象可以了解任务执行情况，可取消任务的执行，还可获取执行结果。
* 加入线程池运行，Runnable使用ExecutorService的execute方法，Callable使用submit方法。
  Callable接口也是位于java.util.concurrent包中。Callable接口的定义为：

```java
public interface Callable<V>     
{     
    V call() throws Exception;     
}
```

 Callable中的call()方法类似Runnable的run()方法，就是前者有返回值，后者没有。



 当将一个Callable的对象传递给ExecutorService的submit方法，则该call方法自动在一个线程上执行，并且会返回执行结果Future对象。

 同样，将Runnable的对象传递给ExecutorService的submit方法，则该run方法自动在一个线程上执行，并且会返回执行结果Future对象，但是在该Future对象上调用get方法，将返回null。



