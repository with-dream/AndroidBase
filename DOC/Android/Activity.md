# Activity
一、屏幕旋转Activity生命周期
  参考文章链接：Activity横竖屏切换生命周期变化 - 简书
  1、不设置Activity的android:configChanges时，切屏会重新调用各个生命周期，切横屏时会执行1次，切竖屏时会执行1次
  2、设置Activity的android:configChanges="orientation"时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次
  3、设置Activity的android:configChanges="orientation|keyboardHidden|screenSize"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法
  4、设置Activity的android:configChanges="orientation|keyboardHidden|screenSize"时，切屏切记要加上screenSize,否则4.0版本以上生命周期不生效

二、Activity 四大启动模式
  1、standard
     Activity 默认的启动模式，每次 startActivity 都会在栈顶创建一个新的实例，在同一个任务中可以存在多个Activity 的实例。
  2、singleTop
     栈顶复用，如果它恰好在当前栈顶，那么直接复用，执行其 onNewIntent 方法。否则，就重新创建一个实例入栈。
  3、singleTask
    在系统中只有一个实例，当再次启动该 Activity 时，会重用已存在的任务和实例，并且会调用这个实例的 onNewIntent()方法，
    将 Intent 实例传递到该实例中。刷新Activity:onNewIntent()
  4、singleInstance
    总是在新的任务中开启，并且这个新的任务中有且只有这一个实例，也就是说被该实例启动的其他 Activity 会自动运行于另一个任务中。
    当再次启动该 Activity 的实例时，会重用已存在的任务和实例。并且会调用这个实例的onNewIntent()方法，将 Intent 实例传递到该实例中。

  【总结】：①standard每一次都会创建新的实例；②singleTop栈顶复用。和standard相似，但是如果栈顶已有实例，复用该实例，回调onNewIntent()方法；③singleTask栈内复用。
  查找栈内有没有该实例，有则复用回调onNewIntent()方法，如果没有，新建Activity，并入栈；④singleInstance单例模式，全局唯一。具备singleTask所有特性，独占一个任务栈。

  singleTop和singleTask的区别
  singleTop：同个Activity实例在栈中可以有多个，即可能重复创建；该模式的Activity会默认进入启动它所属的任务栈，即不会引起任务栈的变更；为防止快速点击时多次startActivity，可以将目标Activity设置为singleTop
  singleTask：同个Activity实例在栈中只有一个，即不存在重复创建；可通过android：taskAffinity设定该Activity需要的任务栈，即可能会引起任务栈的变更；常用于主页和登陆页.

三、Context数量 = Activity数量 + Service数量 + 1（Application）

四、补充解析activity生命周期:【https://www.jianshu.com/p/fb44584daee3】
   1、onCreate和onStart之间有什么区别？
     （1）可见与不可见的区别。前者不可见，后者可见。
     （2）执行次数的区别。onCreate方法只在Activity创建时执行一次，而onStart方法在Activity的切换以及按Home键返回桌面再切回应用的过程中被多次调用。因此Bundle数据的恢复在onStart中进行比onCreate中执行更合适。
     （3）onCreate能做的事onStart其实都能做，但是onstart能做的事onCreate却未必适合做。如前文所说的，setContentView和资源初始化在两者都能做，然而想动画的初始化在onStart中做比较好。
   2、onStart方法和onResume方法有什么区别？
     （1）是否在前台。onStart方法中Activity可见但不在前台，不可交互，而在onResume中在前台。
     （2）职责不同，onStart方法中主要还是进行初始化工作，而onResume方法，根据官方的建议，可以做开启动画和独占设备的操作。
   4.onPause方法和onStop方法有什么区别？
     （1）是否可见。onPause时Activity可见，onStop时Activity不可见，但Activity对象还在内存中。
     （2）在系统内存不足的时候可能不会执行onStop方法，因此程序状态的保存、独占设备和动画的关闭、以及一些数据的保存最好在onPause中进行，但要注意不能太耗时。
   5.onStop方法和onDestroy方法有什么区别？
     onStop阶段Activity还没有被销毁，对象还在内存中，此时可以通过切换Activity再次回到该Activity，而onDestroy阶段Acivity被销毁
   6.为什么切换Activity时各方法的执行次序是(A)onPause→(B)onCreate→(B)onStart→(B)onResume→(A)onStop而不是(A)onPause→(A)onStop→(B)onCreate→(B)onStart→(B)onResume
     （1）一个Activity或多或少会占有系统资源，而在官方的建议中，onPause方法将会释放掉很多系统资源，为切换Activity提供流畅性的保障，而不需要再等多两个阶段，这样做切换更快。
     （2）按照生命周期图的表示，如果用户在切换Activity的过程中再次切回原Activity，是在onPause方法后直接调用onResume方法的，这样比onPause→onStop→onRestart→onStart→onResume要快得多。
   7.与生命周期密切相关的onSaveInstanceState方法和onRestoreInstanceState方法在什么时候执行？
     通过阅读源码会发现，当targetSdkVersion小于3时onSaveInstanceState是在onPause方法中调用的，而大于3时是在onStop方法中调用的。
     而onRestoreInstanceState是在onStart之后、onResume之前调用的。
   8.生命周期对应的代码有：onStart()、onStop();onCreate()、onDestory();onResume()、onPause();onCreateView()、onDestoryView()