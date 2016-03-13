# Intents 和 Intent Filters

[Intent][Intent] 是一个消息对象，主要用于向另一个应用程序组件 ([app component](http://developer.android.com/guide/components/fundamentals.html#Components))发出请求。总体而言，有一下三种基本用法：

* 启动一个活动 (activity)
	* 每个 [Activity](Activity) 代表着一个应用程序界面。
	* 将 [Intent][Intent] 作为参数传给 [startActivity()] 
* 启动一个服务 (service)
* 传输一个广播 (broadcast)

[Intent]: http://developer.android.com/reference/android/content/Intent.html "Intent"
[Activity]: http://developer.android.com/reference/android/app/Activity.html "Activity"