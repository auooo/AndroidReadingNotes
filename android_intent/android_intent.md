# Intents 和 Intent Filters

[Intent][Intent] 是一个消息对象，主要用于向另一个应用程序组件 ([app component](http://developer.android.com/guide/components/fundamentals.html#Components))发出请求。总体而言，有一下三种基本用法：

* 启动一个活动 (activity)
	* 每个 [Activity](Activity) 代表着一个应用程序界面
	* 将 [Intent][Intent] 作为参数传给 [startActivity()][startActivity] 方法，从而启动一个 [Activity][Activity] 的实例
	* [Intent][Intent] 指定了要启动的 activity ,并且包含了任何必需的数据
	* 若想要在目标 activity 结束后获得结果，则调用 [startActivityForResult()][startActivityForResult] 方法。该方法的回调会使源 activity 接收到另一个[Intent][Intent] 对象，从而获得需要的结果
* 启动一个服务 (service)
	* 服务是在后台运行的，没有用户界面的组件
	* 将 [Intent][Intent] 作为参数传递给 [startService()][startService] 方法， 从而启动一个服务来执行某项一次性操作。
	* [Intent][Intent] 制定了要启动的 Service，并且包含了任何必需的数据
	* 如果服务是设计用于客户端-服务器 (client-server) 的接口，则可以将 [Intent][Intent] 作为参数传递给 [bindService()](http://developer.android.com/reference/android/content/Context.html#bindService(android.content.Intent, android.content.ServiceConnection, int)) 方法，这样就能够从其他组件绑定服务了
* 传输一个广播 (broadcast)
	* 广播是任何应用程序都能接收的信息 (message)
	* 将 [Intent][Intent] 作为参数传递给 [sendBroadcast()](http://developer.android.com/reference/android/content/Context.html#sendBroadcast(android.content.Intent)), [sendOrderedBroadcast()](http://developer.android.com/reference/android/content/Context.html#sendOrderedBroadcast(android.content.Intent, java.lang.String)) 或者  [sendStickyBroadcast()](http://developer.android.com/reference/android/content/Context.html#sendStickyBroadcast(android.content.Intent)) 方法，来发送广播给其他应用程序。

## Intent 类型

Intent 有两种类型：

* 显式 intent：指明了要启动的组件的名称（完全匹配的类名称）。当我们直到我们想要启动的活动或者服务的名称时，一般采用显式 intent。
* 隐式 intent：并没有指明组件的名称，而是声明了要执行的一般性操作，其他应用程序的组件便能响应它。

若创建显式 intent 来启动活动或服务，系统会立即启动 [Intent][Intent] 对象中指定的组件。
若创建隐式 intent，android 系统会查找其他应用程序 [manifest](http://developer.android.com/guide/topics/manifest/manifest-intro.html) 文件中声明的 *intent-filters* 内容来确定恰当的组件。如果 intent 满足某个 intent filter，系统就会启动这个组件，并传输 [Intent][Intent]对象。如果有多个 intent filter 都符合条件，系统会弹出一个选择对话框，由用户决定具体要使用的应用程序。
intent filter 是在应用程序 manifest 文件中的表达式，它指明了所在组件所能响应的 intent。若没有声明 intent filter,则只能通过显式 intent 来启动对应组件。

>注意：为确保应用程序的安全性，启动 [Service][Service] 应当采取显式 intent，并且不要给 service 声明任何 intent filter。否则的话，由于 service 不可见，人们没法确定何种 service 响应了 intent，因而使用隐式 intent 启动服务是危害安全的行为。从 Android 5.0 (API level 21) 开始，如果调用 [bindService](http://developer.android.com/reference/android/content/Context.html#bindService(android.content.Intent, android.content.ServiceConnection, int)) 方法并使用了隐式 intent 作为参数， 系统会抛出异常。

隐式 intent 传输示意图：
![](https://raw.githubusercontent.com/auooo/AndroidReadingNotes/master/android_intent/intent-filters%402x.png)
1. Activity A 创建一个 [Intent][Intent]，描述了要进行的操作，然后将它传递给 [startActivity()][startActivity] 方法
2. Android 系统在所有 app 的 intent filter 中搜索满足条件的 intent，当成功匹配上时
3. 系统通过调用匹配活动 (Activity B) 的 [onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 方法来启动 Activity B，并将 Intent 传递给它

## 构建 Intent

在一个 [Intent][Intent] 中包含的主要信息有以下这些：

**组件名称 (Component name)**
要启动的组件名称。
这是可选的，但却是决定 intent 的显式的重要信息，有了它意味着 intent 只会传递到名称与定义相符的应用程序组件。否则，intent 就是隐式的，系统需要根据 intent 中的其他信息（例如下面所说的 action，data 和 category）来决定接收 intent 的组件。
>注：当启动 [Service][Service] 时，应当总是使用显式 intent。



**行为 (Action)**

**数据 (Data)**

**类别 (Category)**

**附加信息 (Extras)**

**标识信息 (Flags)**



[Intent]: http://developer.android.com/reference/android/content/Intent.html "Intent"
[Activity]: http://developer.android.com/reference/android/app/Activity.html "Activity"
[Service]: http://developer.android.com/reference/android/app/Service.html "Service"
[startActivity]: http://developer.android.com/reference/android/content/Context.html#startActivity(android.content.Intent) "startActivity"
[startService]: http://developer.android.com/reference/android/content/Context.html#startService(android.content.Intent) "startService"