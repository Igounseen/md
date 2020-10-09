##  com.google.common.eventbus

#### 主要的类和注解

class:

- ```java
  /**
  * 通过register方法注册监听器，通过post方法将事件分配给监听器
  * eventbus 会自行根据事件的类型将其路由到对应的监听器上
  */
  EventBus
  ```


- ```java
  /**
  * AsyncEventBus是继承EventBus异步消息，可以指定消息的处理在哪里执行
*/
  AsyncEventBus
  ```
  
- ```java
  /**
  * 包装了已发布但没有找到相应的订阅服务器因而无法传递的事件。
  */
  DeadEvent
  ```

 

annotation:

- ```java
   /**
  * 将事件处理方法标记为线程安全的。表示EventBus可以从多个线程同时调用事件处理程序。
  * 该注解需要和 @Subscribe 组合使用
  */
   AllowConcurrentEvents
  ```
  
- ```java
  /**
  * 将方法标记为事件处理程序
  */
  Subscribe
  ```



#### 一个简单样例

- step1: 创建一个事件类

  ```java
  @Data
  @Builder
  @AllArgsConstructor
  class Msg {
      String content;
  }
  ```

- step2: 创建监听类，使用@Subscribe注解Msg事件的处理方法

  ```java
  @Slf4j
  class MsgListener {
      @Subscribe
      public void listener(Msg msg) {
          log.info("msg:{}", msg);
      }
  }
  ```

- step3: 注册监听者，并发布事件

  **eventBus 会将事件投递到所有注册过的listener上，然后listener根据事件的类型，传递到相应的方法上进行处理**

  ```java
  public static void main(String[] args) {
      EventBus eventBus = new EventBus();
      // 注册监听者
      eventBus.register(new MsgListener());
      // 发布一条消息。
      eventBus.post(new Msg("nice"));
  }
  ```

  输出：

  ```
  11:16:08.827 [main] INFO MsgListener - msg:Msg(content=nice)
  ```

  

#### DeadEvent

该类包装了已发布但没有找到相应的订阅服务器因而无法传递的事件

**例**: 在简单样例中增加一条消息发布：

```java
public static void main(String[] args) {
    EventBus eventBus = new EventBus();
    // 注册监听者
    eventBus.register(new MsgListener());
    // 发布一条Msg类型的消息。
    eventBus.post(new Msg("nice"));
     // 再发布一条String类型的消息
    eventBus.post("hello");
}
```

由于新发布的是一条String类型的消息，而相应的MsgListener中没有找到对应的类型（只有Msg）,故该消息被遗弃。

依旧输出，"hello"消息被遗弃：

```
11:30:04.945 [main] INFO MsgListener - msg:Msg(content=nice)
```

**修改**：

可在MsgListener中增加DeadEvent对没到找到类型的消息的统一处理

```java
@Slf4j
class MsgListener {
    @Subscribe
    public void listener(Msg msg) {
        log.info("msg:{}", msg);
    }
    @Subscribe
    public void dead(DeadEvent event) {
        log.info("dead event:{}", event);
    }
}
```

输出：

```
11:30:04.945 [main] INFO MsgListener - msg:Msg(content=nice)
11:30:04.948 [main] INFO MsgListener - dead event:DeadEvent{source=EventBus{default}, event=hello}
```



#### AsyncEventBus

EventBus都是在当前线程中依次执行事件处理，而AsyncEventBus可以指定线程处理相应事件。因此AsyncEventBus在初始化时，需要提供[Executor](http://download.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/Executor.html?is-external=true)对象：

**例**：将简单样例的中EventBus换成AsyncEventBus

```java
public static void main(String[] args) {
    // 指定处理事件的线程池
    ThreadPoolExecutor pool = new ThreadPoolExecutor(2,
                                                    5,
                                                    1,
                                                    TimeUnit.MINUTES,
                                                    new ArrayBlockingQueue(10),
                                                    r -> new Thread((r)));
    AsyncEventBus asyncEventBus = new AsyncEventBus(pool);
	// 注册监听器，并发布两条事件
    asyncEventBus.register(new MsgListener());
    asyncEventBus.post(new Msg("nice"));
    asyncEventBus.post("hello");

    pool.shutdown();
}
```

输出：

```
11:47:28.158 [Thread-2] INFO MsgListener - dead event:DeadEvent{source=AsyncEventBus{default}, event=hello}
11:47:28.158 [Thread-1] INFO MsgListener - msg:Msg(content=nice)
```

事件已在**不同的自定义线程**（Thread-1 和Thread-2）中进行处理, 而EventBus中的事件都在当前线程（样例中为main）中进行处理。



#### AllowConcurrentEvents

如果listener中，被标记了@Subscribe方法存在多线程访问安全的问题，那么需要在添加注解@allowConcurrentEvents:

```java
  @Subscribe
  @AllowConcurrentEvents
  public void methodsafe(){};
```

