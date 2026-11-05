# Articles

## 技术文章

- [Android面试题](/Android面试题总结.md)
- [Kotlin笔记](/KotlinNote.md)
- [Git笔记](/GitNote.md)
- [多渠道](/多渠道版本发布.md)

## 非技术文章

- [ISTJ人格](/ISTJ.md)
- [如何租个好房子](/RentGuide.md)
- [常见IT单词错误发音](/WrongPronunciation.md)
- [如何反馈Bug？](/HowToSubmitBug.md)
- [隱私權條款](/隱私權條款.md)

# DevelopNote

  ### 双缓冲机制

  问题的由来
  CPU访问内存的速度要远远快于访问屏幕的速度。如果需要绘制大量复杂的图像时，每次都一个个从内存中读取图形然后绘制到屏幕就会造成多次地访问屏幕，从而导致效率很低。这就跟CPU和内存之间还需要有三级缓存一样，需要提高效率。

  第一层缓冲
  在绘制图像时不用上述一个一个绘制的方案，而采用先在内存中将所有的图像都绘制到一个Bitmap对象上，然后一次性将内存中的Bitmap绘制到屏幕，从而提高绘制的效率。Android中View的onDraw()方法已经实现了这一层缓冲。onDraw()方法中不是绘制一点显示一点，而是都绘制完后一次性显示到屏幕。

  第二层缓冲
  onDraw()方法的Canvas对象是和屏幕关联的，而onDraw()方法是运行在UI线程中的，如果要绘制的图像过于复杂，则有可能导致应用程序卡顿，甚至ANR。因此我们可以先创建一个临时的Canvas对象，将图像都绘制到这个临时的Canvas对象中，绘制完成之后再将这个临时Canvas对象中的内容(也就是一个Bitmap)，通过drawBitmap()方法绘制到onDraw()方法中的canvas对象中。这样的话就相当于是一个Bitmap的拷贝过程，比直接绘制效率要高，可以减少对UI线程的阻塞。

  ## ADB

  `adb tcpip 5555` --> `adb connect 192.168.*.*(设置-关于手机里面的ip)`

  `adb logcat -c` 清空日志

  `adb logcat -t 500` > dante.log 输出最近500行日志并保存到项目目录下的 dante.log 文件

  ### SurfaceView 

  - **WMS中单独创建窗口**
  - 可以在子线程刷新画面
  - 双缓冲机制（更新视图时用了两张Canvas）
  - 不可进行旋转缩放等动画
  - 适合2D游戏和视频播放器开发

  ### TextureView

  - **作为View hierachy中的普通View**
  - 单独的渲染线程
  - 必须在硬件加速的窗口中
  - 适合视频播放器或者相机开发
  - 内存占用比SurfaceView高

  ### 事件分发

   - 对于事件分发：（dispatchTouchEvent）
     **如果想事件不向下传递，自己消费掉**  ：将当前的dispatchTouchEvent返回true；
      **如果想事件不向下传递，返回给上层**  ：将当前的dispatchTouchEvent返回false；
   - 对于事件拦截：（onInterceptTouchEvent）
     **如果想拦截事件，给自己的onTouchEvent方法消费** ：将onInterceptTouchEvent返回true
      **如果不拦截事件，默认向下（子view）传递** ：将onInterceptTouchEvent返回false或者返回默认值
   - 对于事件消费：（onTouchEvent）
     **如果不想消费，返回给上层** ：将onTouchEvent返回默认或者返回false；
      **如果想消费，不再返回** ：将onTouchEvent返回true；
   - 当`dispatchTouchEvent（）`事件分发时，只有前一个事件（如ACTION_DOWN）返回true，才会收到后一个事件（ACTION_MOVE和ACTION_UP）
   - onInterceptTouchEvent 并不能消费事件，它相当于是一个分叉口起到分流导流的作用，对后续的ACTION_MOVE和ACTION_UP事件接收起到非常大的作用
   - 接收了ACTION_DOWN事件的函数不一定能收到后续事件

  ###  其他技巧

  - sdk 初始化失败，可能是初始化所在的进程不对
  - 使AlertDialog的文本可点击：

  ```
              AlertDialog dialog = new AlertDialog.Builder(this)
                      .setTitle("关于应用")
                      .setMessage("You message...  <a href="http://your.link">点这里</a> blah blah")
                      .setPositiveButton("检查更新", (dialog1, which) -> AppUtil.goMarket(MainActivity.this))
                      .show();
              ((TextView) dialog.findViewById(android.R.id.message)).setMovementMethod(LinkMovementMethod.getInstance());
  ```

- 如果App内部分界面很卡，可能是因为`hardwareAccelerated`被关闭了（view可以通过`myView.setLayerType(View.LAYER_TYPE_HARDWARE, null)`设置硬件加速，默认为`LAYER_TYPE_NONE`）

- 一个很好的经验之谈是在屏幕内每帧率绘制像素数不要超过屏幕像素的2.5倍（包括bitmap中的透明像素）。不要修改shape、path和circles太频繁；不要修改bitmap太频繁，每次修改都会上传作为GPU texture

- 谨慎使用setAlpha，alphaAnimation或ObjectAnimator来修改透明度，它会在off-screen buffer中渲染，导致所需fill-rate加倍。对于很大的view应用alpha时，考虑设置view的layerType为`LAYER_TYPE_HARDWARE`

- Iterable和Sequence(Flow相当于Sequence+Coroutine)区别：

  ```
  words.filter { println("filter: $it"); it.length > 3 }
      .map { println("length: ${it.length}"); it.length }
      .take(4)
  ```

  Iterable会立刻执行，先把words走完filter再走map；Sequence会懒执行（用到的时候才会走），每个元素分别走完filter+map再轮到下一个元素。Sequence可以避免中间步骤的results创建因此提高集合处理操作链的性能，但是这种懒执行的特性对于处理小集合或者简单计算来说会增加较大开销。

  Flow是利用协程建造起来的，通过suspend和resume来同步生产者(flow)和消费者(collect或者其他terminal operator如toList)的执行。

- recyclerView动画是根据canReuseUpdatedViewHolder来决定是复用同一个ViewHolder来跑动画 or 复制一份ViewHolder并且itemAnimator用这俩item来跑动画。然后根据remove/add/move/change动作，存储对应的动作信息（如ChangeInfo），执行pending动画，顺序是移动、更新和添加。如果有删除动作，则每一步的最后都会执行删除动画。

- 网络请求完成了，但是没走到response（liveData没触发回调），可能是线程问题。

- 查看签名信息：keytool -list -v -keystore

- 遇到无论如何keyboard也隐藏不了的情况，请先尝试注释掉第三方的方法比如：`KeyboardUtils.showSoftInput`，这些方法可能会导致键盘强制显示

- includeFontPadding 设为 false 会导致underline不能显示。例如`<u>用户协议</u>`

- SAM：Single Abstract Method，单一抽象方法接口或者functional interface，只具有一个无默认方法和默认属性的接口，比如`Runnable`类（只有一个run方法）和`OnClickListener`等。在kotlin中可以用 fun interface 定义

- 如果没有使用 Immutable 标记组件，当其他组件发生变化时，可能会触发其重建和重绘从而影响性能。为什么不给只含有val变量的类自动标记Immutable？因为即使都是val变量，其内容可能是可变的。

- MQTT 低带宽、低功耗，格式简单，适合物联网（IoT），HTTP 消息结构复杂，header 占用较多，支持分块传输，适合传输大量数据或者动态数据（如网页、下载等）

- Socket 是底层网络通讯机制，适合开发服务器和游戏，功能强大，较为复杂；MQTT 是基于发布-订阅的消息传输协议，适合物联网，开发较简单。两者都是基于 TCP（Socket 支持UDP）

- Compose 目的是为了减少 xml 和 ViewModel 的耦合，增加内聚。方法是使用 Kotlin 语言来替代 xml ，这样的话一些以前隐含的依赖关系可能会变得更加明确。我们还可以重构代码，把一些东西移到可以减少耦合度和增加内聚力的地方。还可以处理复杂的 UI 逻辑

- 对于调试构建类型的清单文件或资源文件中的属性，请始终使用静态值。使用动态版本代码、版本名称、资源或任何其他会更改清单文件的构建逻辑，都需要在每次运行更改时进行完整的应用程序构建，即使实际更改可能只需要 Hot Swap。如果您的构建配置需要此类动态属性，请加在release版本：

  ```
  onVariants(selector().withBuildType("release")) { variant ->
        // Because an app module can have multiple outputs when using multi-APK, versionCode
        // is only available on the variant output.
        // Gather the output when we are in single mode and there is no multi-APK.
        val mainOutput = variant.outputs.single { it.outputType == OutputType.SINGLE }
  
        // Create the version code generating task.
        val versionCodeTask = project.tasks.register("computeVersionCodeFor${variant.name}", VersionCodeTask::class.java) {
            it.outputFile.set(project.layout.buildDirectory.file("versionCode${variant.name}.txt"))
        }
  
        // Wire the version code from the task output.
        // map will create a lazy Provider that:
        // 1. Runs just before the consumer(s), ensuring that the producer (VersionCodeTask) has run
        //    and therefore the file is created.
        // 2. Contains task dependency information so that the consumer(s) run after the producer.
        mainOutput.versionCode.set(versionCodeTask.flatMap { it.outputFile.map { it.asFile.readText().toInt() } })
    }
  ```

- 自定义View绘制的过大，可能是把长度(width/height/length)当成了半径(radius)使用

- lambda会导致代码体积增加，和额外的lambda对象创建，Android平台lambda的开销几乎只产生在编译时

- 闭包就是method版本的lambda，不过持有外部的引用，所以会带来额外开销（值得注意的是和 inline 标识符一起用可以消除这种开销）。延迟计算的时候，可能会需要返回一个闭包

- 动态计算布局高度时，遇到了布局边界对不齐（偏移）的现象，经过onGlobalPostionChange打印布局的positionInWindow，确认了这个问题是单位转换导致的。view系统应该也有同样的问题。解决方案，尺寸使用Float，并且在小数点后面加0.5（整数部分最好是2或者5的倍数）

- remember的key选择要非常慎重，选少了，可能会导致数据不更新。
