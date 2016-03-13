>* 2016.3.11
>* android-sdk-23
>* 文中的部分中文术语是自己看文档理解着翻译的，可能有不准确之处，欢迎指出


# Activity 概述

## 定义

[官方 API 文档](http://developer.android.com/guide/components/activities.html)中对Activity的定义如下：
>An [Activity](http://developer.android.com/reference/android/app/Activity.html) is an application component that provides a screen with which users can interact in order to do something, such as dial the phone, take a photo, send an email, or view a map. Each activity is given a window in which to draw its user interface. The window typically fills the screen, but may be smaller than the screen and float on top of other windows.

Activity 作为应用程序的组件之一，在屏幕上显示应用程序的界面，用于和用户的交互。

通常而言，activity 提供的界面窗口会占满整个屏幕，但是也可能小于屏幕尺寸，从而浮现在其他应用程序窗口之上（例如带有 [windowIsFloating](http://developer.android.com/reference/android/R.attr.html#windowIsFloating) set 的主题），或者嵌入在其他 Activity 中(例如使用 [ActivityGroup](http://developer.android.com/reference/android/app/ActivityGroup.html))。

### 代码定义

Activity 类的类定义如下：

```java
public class Activity extends ContextThemeWrapper 
			implements LayoutInflater.Factory2, Window.Callback, KeyEvent.Callback,
			 OnCreateContextMenuListener, ComponentCallbacks2, Window.OnWindowDismissedCallback {
	...
}
```

## 简介

* 一个应用程序 (application)至少要有一个 main activity，在用户启动 application 时显示
* 一般而言， application 都有多个 activity，activity 之间可以相互调
* 每当一个新的 activity 启动，之前的 activity 就会停止 (stopped) ，但是系统会将这个停止的 activity 保存在返回栈 (back stack) 中
* 新的 activity 启动，它会被压入返回栈中，并获得用户焦点
* back stack 遵循基本的先进后出 (last-in, first-out) 的栈规则
* 当前栈顶的 activity 完成后(用户按下*返回* (*Back*) 按键)，该 activity 出栈(并被摧毁 (destroyed))，之前的 activity 恢复
* activity 状态的改变会触发不同的 activity 生命周期 (activity lifecycle) 的回调方法

## 创建 Activity

* 创建 Activity (或者其子类)的子类
* 在创建的子类中实现生命周期的各种回调函数
* 其中最重要的两个回调函数
	* onCreate(Bundle): activity初始化。1.使用 [setContentView(int)](http://developer.android.com/reference/android/app/Activity.html#setContentView(int)) 定义布局。2.使用 [findViewById(int)](http://developer.android.com/reference/android/app/Activity.html#findViewById(int)) 获取需要的控件
	* onPause(): 在这里保存并提交用户的修改信息(通常使用 [ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html) 来保存数据)

### 实现用户界面

* 用户界面由各种 views —— 来自 [View](http://developer.android.com/reference/android/view/View.html) 类的对象提供
* android 提供了一系列的预定义 views 以供设计和组织布局
	* Widgets: 可视化的交互元素
	* Layouts: 来自 [ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html) 的 views，提供独特的布局模型
	* 自己定义: [View](http://developer.android.com/reference/android/view/View.html) 和 [ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html) 的子类
* 通常，布局定义在 XML 布局文件中，实现界面设计与行为代码的分离 ([setContentView(int)](http://developer.android.com/reference/android/app/Activity.html#setContentView(int))) 方法
* 也可以在 activity 中通过代码实现界面设计(参考 UI 部分)

### 在 manifest 中声明 activity

```xml
<manifest ... >
	<application ... >
  		<activity android:name=".ExampleActivity" />
  		...
		</application ... >
		...
</manifest >
```

* 要使用 activity 则必须在 manifest 文件中声明
* 在 [&lt;application>](http://developer.android.com/guide/topics/manifest/application-element.html) 节点中增加 [&lt;activity>](http://developer.android.com/guide/topics/manifest/activity-element.html) 节点
* activity 节点可以增加其他可选属性，如 label，icon，theme 等，但是 [android:name](http://developer.android.com/guide/topics/manifest/activity-element.html#nm) 属性是必须的(参考文章 [Things That Cannot Changed](http://android-developers.blogspot.com/2011/06/things-that-cannot-change.html))

#### 使用 intent filters

```xml
<activity android:name=".ExampleActivity" android:icon="@drawable/app_icon">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

* [&lt;activity>](http://developer.android.com/guide/topics/manifest/activity-element.html) 节点使用各种 [&lt;intent-filter>](http://developer.android.com/guide/topics/manifest/intent-filter-element.html) 元素来声明其他应用程序元素如何调用该 activity
* 不被其他 application 调用的 activity 则不需要任何的 intent filter，只有自己的 application 可以通过显式 intent 来调用
* 应当只有一个 activity 有 "main" action 和 "launchy" category
* 想要使用隐式 intent 调用 activity，必须添加 [&lt;intent-filter>](http://developer.android.com/guide/topics/manifest/intent-filter-element.html) 节点，在该节点中要包含一个 [&lt;action>](http://developer.android.com/guide/topics/manifest/action-element.html) 节点，此外，还可选添加一个 [&lt;category>](http://developer.android.com/guide/topics/manifest/category-element.html) 和/或 [&lt;data>](http://developer.android.com/guide/topics/manifest/data-element.html) 节点。这三种元素指明了 activity 响应的 intent 类型

## 启动 activity

* 通过 [startActivity()](http://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent)) 方法启动活动，该方法接收一个用于描述目标 activity 的 [Intent](http://developer.android.com/reference/android/content/Intent.html)
	* intent 可以显式地指定想要启动的 activity
	* intent 可以隐式地描述你想要进行的 action 类型(系统根据 intent-filter 中的描述来匹配符合描述类型的 activity)
	* intent 中可以携带少量的数据，供启动的 activity 使用

### 显式 intent

eg.启动名为 *SignInActivity* 的 activity
```java
Intent intent = new Intent(this, SignInActivity.class);
startActivity(intent);
```
### 隐式 intent

intent 指明要操作的行为，然后通过 intent 调用能够响应该行为的 activity。若有多个响应 activity，则会提示用户进行选择。

eg.发送 email

```java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_EMAIL, recipientArray);
startActivity(intent);
```

[EXTRA_EMAIL](http://developer.android.com/reference/android/content/Intent.html#EXTRA_EMAIL) 这个 extra 是电子邮件地址的字符串序列，指明了邮件的接收地址。

### 启动 activity 并获取结果
* 使用 [startActivityForResult](http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)) 来接收启动的 activity 返回的结果。
* 为了接收结果，还要实现 [onActivityResult()](http://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent)) 方法。
* 当启动的 activity 结束后，会返回一个 [Intent](http://developer.android.com/reference/android/content/Intent.html) 到 [onActivityResult()](http://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent)) 方法中。

eg.使用 Intent 让用户选取一个联系人，然后在源 activity 中执行某些行为

```java
private void pickContact() {
	// Create an intent to "pick" a contact, as defined by the content Prover URI
	Intent intent = new Intent(Intent.Action_PICK, Contacts.CONTENT_URI);
	startActivity(intent, PICK_CONTACT_REQUEST);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	// If the request went well (OK) and the request was PICK_CONTACT_REQUEST
	if (requestCode == Activity.RESULT_OK && requestCode == PICK_CONTACT_REQUEST) {
		// Perform a query to the contact's content provider for the contact's name
		Cursor cursor = getContentResolver().query(data.getData(), new String[] {Contacts.DISPLAY_NAME},
													null, null, null);
		if (cursor.moveToFirst()) {	// True if the cursor is not empty
			int ColumnIndex = cursor.getColumnIndex(Contacts.DISPLAY_NAME);
			String name = cursor.getString(columnIndex);
			// Do something with the selected contact's name ...
		}
	} 
}
```

在上面的例子中，先检查 request 是否成功，若成功，则 *resultCode* 的值是 [RESULT_OK](http://developer.android.com/reference/android/app/Activity.html#RESULT_OK)。然后检查 request 的响应结果是不是已知的，本例中，*resultCode* 应当和 [startActivityForResult](http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)) 方法传递的第二个参数 (PICK_CONTACT_REQUEST) 相匹配。检测满足条件后，代码开始处理 activity 返回的结果，这个结果包含在一个 [Intent](http://developer.android.com/reference/android/content/Intent.html) 对象中（data 参数）。

然后，[contentResolver](http://developer.android.com/reference/android/content/ContentResolver.html) 向 content provider 进行查询，返回一个包含所需数据的 [Cursor](http://developer.android.com/reference/android/database/Cursor.html) 对象。

## 结束 Activity

* 调用 activity 的 [finish()](http://developer.android.com/reference/android/app/Activity.html#finish()) 方法结束 activity
* 调用 [finishActivity](http://developer.android.com/reference/android/app/Activity.html#finishActivity(int)) 方法单独关闭在之前启动的 activity

>注：大多数情况下，不应当通过上述方法显式关闭 activity。Android 系统会管理为我们管理 activity 的生命周期，自己调用这些方法会显著地影响用户体验。因此，只有当我们确定不希望用户再回到这个 activity 的实例时才会使用这些方法。

## 管理 Activity 的生命周期 (Activity Lifecycle)

提到生命周期首先想到的果断应当就是这个生命周期图了（显示在方框中的就是能实现的各种回调方法）：

![](http://www.auooo.com/wp-content/uploads/2016/03/activity_lifecycle.png)

一个 activity 主要有三种基本状态：
* Resumed: activity 在屏幕前台运行并且有用户焦点(该状态有时候也被成为 "running")
* Paused: 另一个 activity 在屏幕前台且有焦点，但是这个 activity 仍然是可见的。也就是说，另一个 activity 在这个 activity 的上层，
* Stopped

### 实现生命周期的回调

当 activity 在不同状态改变时，会触发各种回调方法。这些回调方法都能被重写。重写回调方法时，必须调用父类的方法实现。如下例所示：

```java
public class ExampleActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // The activity is being created.
    }
    @Override
    protected void onStart() {
        super.onStart();
        // The activity is about to become visible.
    }
    @Override
    protected void onResume() {
        super.onResume();
        // The activity has become visible (it is now "resumed").
    }
    @Override
    protected void onPause() {
        super.onPause();
        // Another activity is taking focus (this activity is about to be "paused").
    }
    @Override
    protected void onStop() {
        super.onStop();
        // The activity is no longer visible (it is now "stopped")
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // The activity is about to be destroyed.
    }
}
```

上例中代码的一系列方法定义了 activity 的整个生命周期。这些方法能够监控 activity 的三个内部生命周期循环：

* 完整生命周期(entire lifetime)： 从调用 [onCreate()][onCreate] 方法开始，到调用 [onDestroy()][onDestroy] 方法结束 
	* 在 [onCreate()][onCreate] 方法中，activity 进行一些全局操作。例如定义布局
	* 在 [onDestroy()][onDestroy] 方法中，释放所有的剩余资源
	* 例如，某个 activity 具有一个后台运行的线程 (Thread) 用于下载数据，可以在 [onCreate()][onCreate] 中创建线程，在 [onDestroy()][onDestroy] 中终止 (stop) 线程 
* 可见生命周期(visible lifetime)： 从调用 [onStart()][onStart] 方法开始，到调用 [onStop()][onStop] 方法结束
	* 在这个生命周期中，用户能在屏幕上看见这个 activity 并且与之交互。
	* 在这两个方法之间，管理展现 activity 所需要的资源 (resources)
	* 例如，在 [onStart()][onStart] 方法中注册一个 [BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)，用于监控那些影响 UI 的改变。当这个 activity 对用户不可见时，在 [onStop()][onStop] 方法中注销该广播接收器 (BroadcastReceiver)
	* 由于 activity 经常会在可见与不可见状态间切换，因此在 activity 的生命周期中，[onStart()][onStart] 和 [onStop()][onStop] 可能会被多次调用
* 前台生命周期(foregroud lifetime)： 从调用 [onResume()][onResume] 方法开始，到调用 [onPause()][onPause] 方法结束
	* 在这个生命周期时，activity 在所有其他 activity 之上并具有用户输入焦点(user input focus) 
	* activity 会经常进入和退出前台状态（例如设备休眠，或者某个对话 (dialog) 弹出），也就是说 [onResume()][onResume] 和 [onPause()][onPause] 方法会经常被调用。
	* 因此，这两个方法中的代码应当是轻量级的，减少用户在状态转换时的等待时间。

生命周期回调方法总结表：

| 方法 | 描述 | Killable after? | Next |
| :--: | :---------- | :-------------: | :---: |
| [onCreate()][onCreate] | 在 activity 第一次创建时被调用。<br>在这里进行所有的正常静态初始化 —— 创建视图 (views), 绑定数据到列表 (list) 等等。<br>该方法会传入一个 Bundle 对象，如果 activity 之前的状态被保存过，这个 Bundle 对象中就会保存这个状态信息，在这之后总是跟着 onStart() 方法 | 否 | onStart() |
| [onRestart()][onRestart] | 在 activity 结束 (stopped) 之后被调用，在 activity 开始(start) 前。<br>紧接着执行的总是 onStart() | 否 | onStart() |
| [onStart()][onStart] | 在 activity 变成可见状态时被调用。<br>如果 activity 接着变成前台运行 (comes to foreground)，onResume() 方法会跟着执行。<br>如果 activity 接着变成隐藏(hidden)状态，onStop() 方法会跟着执行 | 否 | onResume() 或 onStop() |
| [onResume()][onResume] | 在 activity 开始与用户交互时被调用。<br>这时的 activity 在 activity 栈 (activity stack) 的栈顶，用户可以进行操作（可以理解为获得焦点）。<br>下一个执行的总是 onPause() | 否 | onPause() |
| [onPause()][onPause] | 当系统将要启动另外一个 activity 并将之展现到前台时，该方法被调用。<br>该方法常用于将未保存的变化提交到持续性存储中，停止动画以及其他的消耗CPU的行为，诸如此类。<br>它的代码块中代码应当被很快执行，因为直到它返回后，下一个 activity 才能被 resume（求大神，这里的 resume 应该怎么翻译比较好~） 到前台；<br>后面要么是跟着 onResume()（activity 重新回到前台），要么是跟着 onStop() （activity 变成对用户不可见） | 是 | onResume() 或 onStop() |
| [onStop()][onStop] | 当 activity 不再对用户可见时被调用。<br>变为不可见可能是由于该 activity 被销毁 (destroyed) 了，或者是其他的 activity 恢复 (resume) 到前台并将该 activity 覆盖。<br>后面跟着的要么是 onRestart() （activity 重新与用户交互），要么是 onDestroy() （activity 结束） | 是 | onRestart() 或 onDestrooy() |
| [onDestroy()][onDestroy] | 在 activity 被销毁前调用。<br>这是 activity 的最后一个回调。它被调用，有可能是 activity 结束了(finish() 方法被调用)，或者系统为了节约并释放资源暂时销毁了这个 activity 实例。针对这两种情况，可以使用 [isFinishing()](http://developer.android.com/reference/android/app/Activity.html#isFinishing()) 方法进行分辨 | 是 | nothing |

表中的 "Killable after" 用于指示系统能否在方法返回后的任意时间，在不执行其他 activity 代码的情况下，将 activity 的进程终止。
>注： Killable 为否的并不意味着不会被系统终止 (killed)，但是只有在没有其他可用资源的极端情况下才会被终止。

### 保存 activity 的状态

有时候，系统为了节约资源，会销毁一个 activity 对象，这是如果用户想要返回到该 activity，则不会直接 resume 到 activity 的原来状态，而是重新创建 (recreate) 一个 activity 对象。为了保证原先状态下的各种信息也能够体现在这个新对象中，就需要使用一个额外的回调方法来存储之前的 activity 状态信息，这就是 [onSaveInstanceState()][onSaveInstanceState] 方法。

系统在使得 activity 变得易于摧毁 (destruction) 前调用 [onSaveInstanceState()][onSaveInstanceState] 方法。该方法有一个 Bundle 类型的参数，该参数有一系列的方法，例如 [putString()](http://developer.android.com/reference/android/os/BaseBundle.html#putString(java.lang.String, java.lang.String)), [putInt()](http://developer.android.com/reference/android/os/BaseBundle.html#putInt(java.lang.String, int))，通过键值对 (name-value pairs) 的形式储存 activity 的状态信息。在系统摧毁并重建 activity 时，这个 Bundle 类型的参数会传递到 [onCreate()][onCreate] 和 [onRestoreInstanceState()][onRestoreInstanceState] 方法中。然后在这些方法中便可以取出 Bundle 中存储的状态信息并恢复 activity 状态。当没有储存状态信息时，Bundle 参数会传递一个 null 值。

下图展现了 activity 恢复到前台运行状态的两种方式：

![](http://www.auooo.com/wp-content/uploads/2016/03/restore_instance.png)

>注:并不能保证 [onSaveInstanceState()][onSaveInstanceState] 在 activity 被销毁 (destroyed) 前一定被调用，因为有时候并没有保存状态信息（例如用户按下后退按键结束一个 activity，这时用户就是准备结束这个 activity 的）。如果系统调用了 [onSaveInstanceSteate()][onSaveInstanceState]，会在 [onStop()][onStop] 之前调用，也有可能在 [onPause()][onPause] 之前调用。

即使我们并没有在自己的代码中实现 [onSaveInstanceState()][onSaveInstanceSate] 方法，有些 activity 的状态也会通过 [Activity](http://developer.android.com/reference/android/app/Activity.html) 类 [onSaveInstanceSate()][onSaveInstanceState] 的默认实现来存储。特别地，layout 中的每个 [View](http://developer.android.com/reference/android/view/View.html) 都会实现对应的 [onSaveInstanceState()][onSaveInstanceState] 方法，来储存它们应当要被保存的信息。Android framework 中的绝大多数控件 (widget) 都会恰如其分地实现该方法，因此在重新创建 activity 时，UI 的任何可视化变化都会被自动存储和恢复。例如，可编辑文本 ([EditText](http://developer.android.com/reference/android/widget/EditText.html)) 控件会储存用户输入的任何文字信息，复选框 ([Checkbox](http://developer.android.com/reference/android/widget/CheckBox.html)) 控件则储存着其勾选状态。我们唯一需要做的，就是给每个控件指定一个唯一的 ID (通过 [android:id](http://developer.android.com/guide/topics/resources/layout-resource.html#idvalue) 属性)，这样就能够储存它们的状态了，否则系统是不能储存这些控件的状态的。

>注：可以将 [android:saveEnabled](http://developer.android.com/reference/android/R.attr.html#saveEnabled) 属性设置为 "false"，或者调用 [setSaveEnabled](http://developer.android.com/reference/android/view/View.html#setSaveEnabled(boolean)) 显式地禁止布局中的某个 view 储存它自己的状态。

由于 [onSaveInstanceState()][onSaveInstanceState] 的默认实现帮助保存 UI 的状态，因此在我们重写该方法时，要首先调用它的超类实现。同样的，在重写 [onRestoreInstanceState()][onRestoreInstanceState] 方法时，也要同样地调用其超类实现来恢复 view 的状态。

>注：由于 [onSaveInstanceState()][onSaveInstanceState] 并不能保证一定被调用，因此我们应当只用它来储存 activity 的一些暂时状态（UI 的状态），而不应该在这个方法中来储存持久性数据，储存持久性数据改用 [onPause()][onPause] 方法来实现。

一个用于测试应用程序状态存储的好方法是在打开应用程序时旋转屏幕，这时候系统为了应用新的屏幕参数信息，会摧毁并重建 activity。因此，在 activity 重新创建时，有之前 activity 状态的完整保存很重要。

### Activity 的相互协调

* 一个 activity 启动另一个 activity 时，并不是在第二个 activity 启动前就完全停止了，第二个 activity 和第一个 activity 的启动过程是有重叠的。
* activity 的生命周期回调顺序是很清晰的，特别是两个运行于同一线程的 activity 间的相互调用。Activity A 启动 Activity B 的顺序如下：
	1. 执行 Activity A 的 [onPause()][onPause] 方法
	2. 按顺序执行 Activity B 的 [onCreate()][onCreate], [onStart()][onStart], [onResume()][onResume] 方法（此时 Activity B 具有用户焦点）。
	3. 接着，如果 Activity A 不再可见，则执行其 [onStop()][onStop] 方法。
* 上面的生命周期顺序可以帮助我们更好地管理各类信息的变化和传递。例如，在第一个 activity 中储存数据库信息，并使这些信息能被后一个 activity 读取，我们应该在第一个 activity 的 [onPause()][onPause] 方法而不是 [onStop()][onStop] 方法中进行数据的储存。

### Activity 的运行过程

1. 每个应用程序 (app) 都是一系列活动 (activities)，布局文件 (layouts) 和其他资源文件的集合
	* 其中的一个activity 是应用程序的 main activity
	* 每个 activity 都有一个 main activity，由 AndroidManifest.xml 文件指定 
2. 默认情况下每个应用程序都运行在自己的线程中
3. 通过 startActivity() 传递一个 intent 来启动另一个程序的 activity
4. 当一个 activity 需要启动时， android 会检查是否已经有一个目标 activity 所在应用程序的线程，如果线程已经存在，则直接在那个线程中启动 activity，否则的话，android 会创建一个线程
5. Android 启动 activity 时，调用 activity 的 [onCreate()][onCreate] 方法

### Activity 的生命周期实例

个人感觉 [Head First Android Development](http://shop.oreilly.com/product/0636920029045.do) 中的 StopWatch 例子讲得很好。

这个小项目要做的是一个用来计时的秒表。首先是前置工作，不用多说，界面设计之类的。直接贴代码了。

activity_stopwatch.xml:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:paddingBottom="16dp"
	android:paddingTop="16dp"
	android:paddingLeft="16dp"
	android:paddingRight="16dp"
	tools:context=".StopWatchActivity" >
	
	<TextView
		android:id="@+id/time_view"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:layout_alignParentTop="true"
		android:layout_centerHorizontal="true"
		android:layout_marginTop="0dp"
		android:text=""
		android:textAppearance="?android:attr/textAppearanceLarge"
		android:textSize="92sp" />

	<Button
		android:id="@+id/start_button"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:layout_below="@+id/time_view"
		android:layout_centerHorizontal="true"
		android:layout_marginTop="20dp"
		android:onClick="onClickStart"
		android:text="@string/start" />

	<Button
		android:id="@+id/stop_button"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:layout_below="@+id/start_button"
		android:layout_centerHorizontal="true"
		android:layout_marginTop="10dp"
		android:onClick="onClickStop"
		android:text="@string/stop" />
	<Button
		android:id="@+id/reset_button"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:layout_below="@+id/stop_button"
		android:layout_centerHorizontal="true"
		android:layout_marginTop="10dp"
		android:onClick="onClickReset"
		android:text="@string/reset" />

</RelativeLayout>
```

string.xml:
```xml
...
<string name="start">Start</string>
<string name="stop">Stop</string>
<string name="reset">Reset</string>
...
```

以上是界面部分，可以看到，第一个 TextView 用于显示时间，后面跟着三个按钮，分别用于开始，暂停和结束计时。然后开始功能部分的实现。

#### Stopwatch version 1.0 （计时）

```java
package com.hfad.stopwatch;

import android.os.Bundle;
import android.os.Handler;
import android.app.Activity;
import android.view.View;
import android.widget.TextView;

public class StopwatchActivity extends Activity {
	
	//Number of seconds displayed on the stopwatch.
	private int seconds = 0;
	//Is the stopwatch running?
	private boolean running;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_stopwatch);
		runTimer();
	}

	//Start the stopwatch running when the Start button is clicked.
	public void onClickStart(View view) {
		running = true;
	}

	//Stop the stopwatch running when the Stop button is clicked.
	public void onClickStop(View view) {
		running = false;
	}

	//Reset the stopwatch when the Reset button is clicked.
	public void onClickReset(View view) {
		running = false;
		seconds = 0;
	}

	//Sets the number of seconds on the timer.
	private void runTimer() {
		final TextView timeView = (TextView)findViewById(R.id.time_view);
		final Handler handler = new Handler();
		handler.post(new Runnable() {
			@Override
			public void run() {
				int hours = seconds/3600;
				int minutes = (seconds%3600)/60;
				int secs = seconds%60;
				String time = String.format("%d:%02d:%02d",
				hours, minutes, secs);
				timeView.setText(time);
				if (running) {
					seconds++;
				}
				handler.postDelayed(this, 1000);
			}
		});
	}

}
```

运行程序，发现可以运行成功。但是，有一个问题，就是当我们点击 START 按钮开始计时后，如果翻转屏幕，则计时器的值重新变为 00：00：00 了。原因就是系统检测到了屏幕参数的变化，因而摧毁了当前的 activity，并根据当前的屏幕参数重新创建一个 activity。重新调用 onCreate() 方法，因此就变成初始状态了。

#### Stopwatch version 2.0 （屏幕旋转）

这时我们有两个解决方案：
* 忽略 activity 的重建(re-create)
可以修改 AndroidManifest.xml 文件中 activity 节点的属性配置：
	
```xml
	android:configChanges="configuration_change"
```	
在本例中，在 AndroidManifest.xml 文件中作以下修改：
```xml
<activity 
	android:name="com.hfad.stopwatch.StopwatchActivity"
	android:label="@string/app_name"
	android:configChanges="orientation|screenSize" >
```
"|" 符号代表或，也就是说忽略这两种情况。如果 android 系统遇到了上面的这种配置改变，就会调用 onConfigurationChanged(Configuration) 方法，而不是去重建 activity。

* 保存当前的状态值
在 onSaveInstanceState() 方法中，利用 Bundle 的一系列 put 方法，以键值对的形式将 activity 状态数据放入 Bundle 中。
```xml
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
	savedInstanceState.putInt("seconds", seconds);
	savedInstanceState.putBoolean("running", running);
}
```
然后在 onCreate() 方法中恢复这些数据 
```java
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_stopwatch);
	if (savedInstanceState != null) {
		seconds = savedInstanceState.getInt("seconds");
		running = savedInstanceState.getBoolean("running");
	}
	runTimer();
}
```

#### Stopwatch version 3.0 （不可见时暂停计时）

这时候需求又变更了，我们希望来电话时，我们的 stopwatch 能够自动停止计时。换句话说，只有它可见的时候，我们才希望触发计时。但是经过测试发现，即使是来电时也还是继续在后台进行计时的，不满足需求。这时候就要考虑使用生命周期回调函数中的 onStop() 和 onStart() 方法了。修改代码如下：
```java
public class StopwatchActivity extends Activity {
	private int seconds = 0;
	private boolean running;
	private boolean wasRunning;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_stopwatch);
		if (savedInstanceState != null) {
			seconds = savedInstanceState.getInt("seconds");
			running = savedInstanceState.getBoolean("running");
			wasRunning = savedInstanceState.getBoolean("wasRunning");
		}
		runTimer();
	}

	@Override
	public void onSaveInstanceState(Bundle savedInstanceState) {
		savedInstanceState.putInt("seconds", seconds);
		savedInstanceState.putBoolean("running", running);
		savedInstanceState.putBoolean("wasRunning", wasRunning);
	}

	@Override
	protected void onStop() {
		super.onStop();
		wasRunning = running;
		running = false;
	}

	@Override
	protected void onStart() {
		super.onStart();
		if (wasRunning) {
			running = true;
		}
	}
	
	...
```

#### Stopwatch versin 4.0 (如果可见但是失去焦点，怎么办呢？)

有时候可能会有并不是全屏形式的弹窗，这时候 stopwatch 还是可见的，只不过在它的上层还有其他的 activity。我们希望也能够自动停止计时。这时就需要使用到 onResume() 和 onPause() 两个方法了。代码如下：
```java
package com.hfad.stopwatch;

import android.os.Bundle;
import android.os.Handler;
import android.app.Activity;
import android.view.View;
import android.widget.TextView;

public class StopwatchActivity extends Activity {
	//Number of seconds displayed on the stopwatch.
	private int seconds = 0;
	//Is the stopwatch running?
	private boolean running;
	private boolean wasRunning;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_stopwatch);
		if (savedInstanceState != null) {
			seconds = savedInstanceState.getInt("seconds");
			running = savedInstanceState.getBoolean("running");
			wasRunning = savedInstanceState.getBoolean("wasRunning");
		}
		runTimer();
	}

	@Override
	protected void onPause() {
		super.onPause();
		wasRunning = running;
		running = false;
	}

	@Override
	protected void onResume() {
		super.onResume();
		if (wasRunning) {
			running = true;
		}
	}

	@Override
	public void onSaveInstanceState(Bundle savedInstanceState) {
		savedInstanceState.putInt("seconds", seconds);
		savedInstanceState.putBoolean("running", running);
		savedInstanceState.putBoolean("wasRunning", wasRunning);
	}

	//Start the stopwatch running when the Start button is clicked.
	public void onClickStart(View view) {
		running = true;
	}

	//Stop the stopwatch running when the Stop button is clicked.
	public void onClickStop(View view) {
		running = false;
	}

	//Reset the stopwatch when the Reset button is clicked.
	public void onClickReset(View view) {
		running = false;
		seconds = 0;
	}

	//Sets the number of seconds on the timer.
	private void runTimer() {
		final TextView timeView = (TextView)findViewById(R.id.time_view);
		final Handler handler = new Handler();
		handler.post(new Runnable() {
			@Override
			public void run() {
				int hours = seconds/3600;
				int minutes = (seconds%3600)/60;
				int secs = seconds%60;
				String time = String.format("%d:%02d:%02d",
				hours, minutes, secs);
				timeView.setText(time);
				if (running) {
					seconds++;
				}
				handler.postDelayed(this, 1000);
			}
		});
	}

}
```

因此，最终我们的 stopwatch，既可以满足不受屏幕旋转的影响，又可以根据是否有其他更高层次的 activity 运行从而智能暂停和继续计时。而在这过程中，也用到了所有的生命周期回调函数方法。

[onCreate]: http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle) "onCreate"
[onDestroy]: http://developer.android.com/reference/android/app/Activity.html#onDestroy() "onDestroy"
[onStart]: http://developer.android.com/reference/android/app/Activity.html#onStart() "onStart"
[onStop]: http://developer.android.com/reference/android/app/Activity.html#onStop() "onStop"
[onResume]: http://developer.android.com/reference/android/app/Activity.html#onResume() "onResume"
[onPause]: http://developer.android.com/reference/android/app/Activity.html#onPause() "onPause"
[onRestart]: http://developer.android.com/reference/android/app/Activity.html#onRestart() "onRestart"
[onSaveInstanceState]: http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle) "onSaveInstanceState"
[onRestoreInstanceState]: http://developer.android.com/reference/android/app/Activity.html#onRestoreInstanceState(android.os.Bundle) "onRestoreInstanceState"