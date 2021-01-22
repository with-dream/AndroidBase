# Handler
1、机制
   主线程不能进行耗时操作，子线程不能更新UI，Handler实现线程间通信，将要发送的消息保存到Message中，
   Handler调用sendMessage()方法将message发送到MessageQueue，Looper对象不断调用loop()方法，
   不断从MessageQueue中取出message交给handler处理，从而实现线程间的通信。
2、主线程handler不需要调用Looper.prepare()，Looper.loop()，通过sendMessage将message添加到messagequeue。
3、子线程可以new Handler，但是必须在new之前，必须调用Looper.prepare()方法，否则报错：java.lang.RuntimeException:Can’t create handler inside thread that has not called Looper.prepare()。
4、当创建Handler时将通过ThreadLocal在当前线程绑定一个Looper对象，而Looper持有MessageQueue对象。
   执行Handler.sendMessage(Message)方法将一个待处理的Message插入到MessageQueue中，这时候通过Looper.loop()
   方法获取到队列中Message，然后再交由Handler.handleMessage(Message)来处理。
5、Handler如何实现线程切换？
   Handler创建的时候会采用当前线程的Looper来构造消息循环系统，Looper在哪个线程创建，就跟哪个线程绑定，并且Handler是在他关联的Looper对应的线程中处理消息的。
6、Handler内部如何获取到当前线程的Looper呢？
   ThreadLocal。ThreadLocal可以在不同的线程中互不干扰的存储并提供数据，通过ThreadLocal可以轻松获取每个线程的Looper。
   当然需要注意的是:
   ①线程是默认没有Looper的，如果需要使用Handler，就必须为线程创建Looper。我们经常提到的主线程，也叫UI线程，它就是ActivityThread，
   ②ActivityThread被创建时就会初始化Looper，这也是在主线程中默认可以使用Handler的原因。
7、子线程有哪些更新UI的方法？
  主线程中定义Handler，子线程通过mHandler发送消息，主线程Handler的handleMessage更新UI。
  用Activity对象的runOnUiThread方法。 创建Handler，传入getMainLooper。 View.post(Runnable)。
8、为什么系统不建议在子线程访问UI？
  不建议在子线程访问UI的原因是，UI控件非线程安全，在多线程中并发访问可能会导致UI控件处于不可预期的状态。而不对UI控件的访问加上锁机制的原因有：
  上锁会让UI控件变得复杂和低效，上锁后会阻塞某些进程的执行。
9、一个Thread可以有几个Looper？几个Handler？
  一个Thread只能有一个Looper，可以有多个Handler
  对应关系Thread(1):Looper(1):MessageQueen(1):Handler(n).
  引申：更多数量关系：Looper有一个MessageQueue，可以处理来自多个Handler的Message；MessageQueue有一组待处理的Message，这些Message可来自不同的Handler；Message中记录了负责发送和处理消息的Handler；Handler中有Looper和MessageQueue。

10、如何将一个Thread线程变成Looper线程？Looper线程有哪些特点？
  通过Looper.prepare()可将一个Thread线程转换成Looper线程。Looper线程和普通Thread不同，它通过MessageQueue来存放消息和事件、Looper.loop()进行消息轮询。

11、Message可以如何创建？哪种效果更好，为什么？
  Message msg = new Message();
  Message msg = Message.obtain();
  Message msg = handler1.obtainMessage();
  后两种方法都是从整个Messge池中返回一个新的Message实例，能有效避免重复Message创建对象，因此更鼓励这种方式创建Message。

12、ThreadLocal有什么作用？
  hreadLocal类可实现线程本地存储的功能，把共享数据的可见范围限制在同一个线程之内，无须同步就能保证线程之间不出现数据争用的问题，这里可理解为ThreadLocal帮助Handler找到本线程的Looper。
  底层数据结构：每个线程的Thread对象中都有一个ThreadLocalMap对象，它存储了一组以ThreadLocal.threadLocalHashCode为key、以本地线程变量为value的键值对，而ThreadLocal对象就是当前线程的ThreadLocalMap的访问入口，也就包含了一个独一无二的threadLocalHashCode值，通过这个值就可以在线程键值值对中找回对应的本地线程变量。

13、主线程中Looper的轮询死循环为何没有阻塞主线程？
  Android是依靠事件驱动的，通过Loop.loop()不断进行消息循环，可以说Activity的生命周期都是运行在 Looper.loop()的控制之下，一旦退出消息循环，应用也就退出了。而所谓的导致ANR多是因为某个事件在主线程中处理时间太耗时，因此只能说是对某个消息的处理阻塞了Looper.loop()，反之则不然。

14、使用Hanlder的postDealy()后消息队列会发生什么变化？
  postDelay的Message并不是先等待一定时间再放入到MessageQueue中，而是直接进入并阻塞当前线程，然后将其delay的时间和队头的进行比较，按照触发时间进行排序，如果触发时间更近则放入队头，保证队头的时间最小、队尾的时间最大。此时，如果队头的Message正是被delay的，则将当前线程堵塞一段时间，直到等待足够时间再唤醒执行该Message，否则唤醒后直接执行。

15、Android中还了解哪些方便线程切换的类？
  AsyncTask：底层封装了线程池和Handler，便于执行后台任务以及在子线程中进行UI操作。
  HandlerThread：一种具有消息循环的线程，其内部可使用Handler。
  IntentService：是一种异步、会自动停止的服务，内部采用HandlerThread。

  Handler机制存在的问题：多任务同时执行时不易精确控制线程。
  引入AsyncTask的好处：创建异步任务更简单，直接继承它可方便实现后台异步任务的执行和进度的回调更新UI，而无需编写任务线程和Handler实例就能完成相同的任务。

  HandlerThread是一个线程类，它继承自Thread。与普通Thread不同，HandlerThread具有消息循环的效果，这是因为它内部HandlerThread.run()方法中有Looper，能通过Looper.prepare()来创建消息队列，并通过Looper.loop()来开启消息循环。
  HandlerThread实现方法
  实例化一个HandlerThread对象，参数是该线程的名称；
  通过 HandlerThread.start()开启线程；
  实例化一个Handler并传入HandlerThread中的looper对象，使得与HandlerThread绑定；
  利用Handler即可执行异步任务；
  当不需要HandlerThread时，通过HandlerThread.quit()/quitSafely()方法来终止线程的执行。

16、HandlerThread机制原理
   在线程中创建一个Looper循环器循环消息队列，当有耗时任务进入对列时，不需要再开启新线程，避免线程阻塞；
   继承自Thread,在Thread开始执行时跟主线程在ActivityThread.main()方法内执行代码逻辑类似，初始化Looper--Looper.prepare(),轮询消息--Looper.loop();
   初始化Handler时，使用HandlerThread线程的Looper对象初始化---- new Handler(Looper)构造方法。
   此Handler使用的Looper是子线程创建的，执行message.target.dispatchMessage()也在子线程内，所以最终执行的Runnable或者handleMessage()也会在子线程内。

   示例代码
     // 步骤1：创建HandlerThread实例对象
     // 传入参数 = 线程名字，作用 = 标记该线程
     HandlerThread mHandlerThread = new HandlerThread("handlerThread");
     // 步骤2：启动线程
     mHandlerThread.start();
     // 步骤3：创建工作线程Handler & 复写handleMessage（）
     // 作用：关联HandlerThread的Looper对象、实现消息处理操作 & 与 其他线程进行通信
     // 注：消息处理操作（HandlerMessage（））的执行线程 = mHandlerThread所创建的工作线程中执行
     Handler workHandler = new Handler( handlerThread.getLooper() ) {
               @Override
               public boolean handleMessage(Message msg) {
                   ...//消息处理
                   return true;
               }
           });

     // 步骤4：使用工作线程Handler向工作线程的消息队列发送消息
     // 在工作线程中，当消息循环时取出对应消息 & 在工作线程执行相关操作
     // a. 定义要发送的消息
     Message msg = Message.obtain();
     //消息的标识
     msg.what = 1;
     // b. 通过Handler发送消息到其绑定的消息队列
     workHandler.sendMessage(msg);

   // 步骤5：结束线程，即停止线程的消息循环
     mHandlerThread.quit();

17、如何判断当前线程是主线程？
    ① 通过Thread.currentThread()得到当前线程，通过Looper.getMainLooper().getThread()得到主线程，进行比较即可。
    public boolean isMainThread() {
        方法A: return Looper.getMainLooper() == Looper.myLooper(); //通过Looper.getMainLooper()比较
        方法B: return Thread.currentThread() == Looper.getMainLooper().getThread(); //通过Looper.getMainLooper().getThread()比较
        方法C: return Looper.getMainLooper().getThread().getId() == Thread.currentThread().getId(); //通过Looper.getMainLooper().getThread()比较
    }

    ② 另外，在Java中没有Looper对象，所以这种方法没用，可以通过Thread.getName()，来判断是否是主线程
    public boolean isMainThread() {
        return Thread.currentThread().getName().equals("main");
    }