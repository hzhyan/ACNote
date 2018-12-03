# Activity

Activity 通常大家接触 Android 的第一个概念。四大组件之首。承载了用户和手机之前的交互。

## UI概念

<center>
![Activity UI 层次结构](../res/activity_ui.png)
</center>

Activity的UI结构如图所示，我们通常操作的是ContentView的内容。

## 生命周期

Activity由ActivityManager负责管理，同时给出声明周期的回调。关于Activity声明周期的回调，[官网](https://developer.android.com/guide/components/activities) 给出的图片阐述的很清楚。
<center>
![ActivityLifecycle](../res/activity_lifecycle.png)
</center>

`onResume` 在 Activity 即将开始与用户进行交互之前调用。 此时，Activity 处于 Activity 堆栈的顶层，并具有用户输入焦点。

`onPause` 在 当系统即将开始继续另一个 Activity 时调用。所以弹出对话框和Toast等行为并不会触发这个。

`onDestroy` 在 Activity 被销毁前调用。这是 Activity 将收到的最后调用。 当 Activity 结束（有人对 Activity 调用了 finish()），或系统为节省空间而暂时销毁该 Activity 实例时，**可能** 会调用它。这里用了**可能**，只有当进程存活时才会执行，如果用户直接杀死了App（杀死了进程）是不会执行的。

通过这个图片可以看出来，启动另一个页面时，会先执行新页面的onCreate后执行当前页面的onDestroy。

如果启动一个透明的Activity(主题透明，和ContentView背景无关)，当前页面只有onPause会被触发，onStop则不会。这个说明，onResume和onPause更多是对Activity栈的反馈，表示当前Activity是否处于栈顶，而onStart 和 onStop更多侧重于是否对用户可见。这也契合了官方的解释，onStart表示即将对用户可见，onResume表示处于堆栈顶层。

## Activity状态保存于恢复

` onSaveInstanceState()` 和 `onRestoreInstanceState()`这两个方法负责Activity状态的保存和恢复。

<center>
![Activity状态保存于恢复](../res/restore_instance.png)
</center>

**注意：** 这个图片仅仅是解释了APP被压栈（另外一个页面盖到）的情况，如果是触发了`Activity`的`finish()`方法，则不会执行`onSaveInstanceState()`。

`onSaveInstanceState()`的调用时机分可以理解为Activity依然存活但是被压栈或者所在的Activity栈进入后台的情况下。

## 启动模式与任务管理

`Task`官方翻译为任务，指在执行特定作业时与用户交互的一系列 Activity。我们大多数时候说的 task 比较侧重于它的堆栈（返回栈）。

Android中，同一个App会有多个Activity栈，一个栈中可以存在多个Activity。

一个Activity相对用户可见必须满足两个条件，第一：其所在的栈是当前栈（当前任务）；第二：该Activity处于栈顶。

### 启动模式

Activity的启动模式有四种。

1. Standard
2. SingleTop
3. SingleTask
4. SingleInstance

##### Standard 

默认，Activity可以多次实例化，同一个栈可以有多个实例。

##### SingleTop

如果当前栈栈顶已经是该页面，则不会再次实例化该页面，而是触发其`onNewIntent`方法。但是，不同栈中可以存在多个实例，同一个栈中也可以存在不相连的多个实例。

##### SingleTask

在官网的介绍中写的是：***系统创建新任务并实例化位于新任务底部的 Activity。***但是我发现不是这样子的，并不一定会打开新的任务，这个是由`taskAffinity`属性决定的，SingleTask本身好像不会打开新的任务。

如果该Activity在栈内已经存在，再次启动时会触发`onNewIntent`，同时会清除栈中处于其上的Activity，被清除的Activity的`onDestroy`会被触发。

结论：**SingleTask并不会导致启动新的栈，而是看 taskAffinity属性是否有变化。也就是说，SingleTask的时候，系统会尝试将这个Activity放在其同一个taskAffinity的task里面，如果没有，则会创建一个新的。**

##### SingleInstance

一定会创建一个新栈来存放Activity，并且该栈中只存放这个Activity。和SingleTask不同，如果该页面已经存在，再次启动同样会触发`onNewIntent`，但是这时候页面切换实际上做的是改变了任务（栈）的顺序，将该页面所在的栈移到了前台，这时并不会触发任何Activity的销毁。

#### 返回操作的响应

通常情况下，如果点击返回按钮，在View和Fragment不消费的情况下，执行的出栈操作，也就是首先操作的是当前栈的栈顶Activity出栈，次栈顶页面获得焦点。如果当前栈已经空了，就展示下一个栈。距离说明。

依次启动 A B C三个页面，其中，A C在同一个栈，B在另一个栈，点击返回按钮展示的顺序是  C A B。

#### 启动模式小结

这部分是参照官网的同时自己写了测试代码。也许是Android版本更迭导致的兼容问题，官网对于SingleTop的描述和实验结果并不符合，我测试用的手机是Android 8.1.0  API 27  compile 和 target SDK版本都是27。
