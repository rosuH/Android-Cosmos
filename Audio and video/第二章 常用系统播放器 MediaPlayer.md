# 第二章 常用系统播放器 MediaPlayer



## 2.1 状态图和生命周期

### 1. MediaPlayer 的状态图

MediaPlayer 用于控制视频/音频文件及流的播放，由状态机进行控制。下图是 MediaPlayer 的状态周期。

![mediaplayer_state_diagram](../images/l7UhONFQoJSbaHn.gif)

- 椭圆代表 MediaPlayer 的驻留状态，弧代表播放控制并且驱动 MediaPlayer 状态进行过度
	- 单箭头弧表示同步函数调用
	- 双箭头弧表示异步函数调用 

### 2. Idle 状态和 End 状态

- MediaPlayer 创建实例或调用 reset 函数后，播放器就被创建了，此时处于 Idle 状态（就绪状态）
- MediaPlayer 调用 release 函数后，播放器就会变成 End 状态

这两种状态之间就是 MediaPlayer 的生命周期。

### 3. Error 状态

触发条件：

- 构建新的 MediaPlayer 或调用 reset 函数后，上传应用程序调用 `getCurrentPosition`, `getVideoHeight`, `getDuration`, `getVideoWidth`, `setAudioStreamType`, `setLooping`, `setVolume`, `pause`, `start`, `stop`, `seekTo`, `prepare`, `prepareAsync`这些函数会出错。
  - 如果调用 reset 函数后再调用他们，用户提供的回调函数`OnErrorListener.onError`将会触发`MediaPlayer`状态到`Error`状态，所以一旦不再使用，就需要`release`函数，以便 MediaPlayer 资源得到合理释放。
  - 当 MediaPlayer 到达 End 状态时，他将不能再被使用，也无法回到其他状态，本次生命周期就此终结
- 由于支持的音视频格式分辨率过高，输入数据流超时，或者其他各种各样的原因导致播放失败
  - 此种条件下，事先注册的 `OnErrorListener.onError` 将会被 MediaPlayer 回调，并返回错误信息
  - 此时 MediaPlayer 进入到 Error 状态；此时可以调用 reset 函数，将重新恢复到 Idle 状态
  - 故此我们要给 MediaPlayer 设置错误监听，出错之后就可以从播放器内部返回的信息中找到错误原因

### 4. Initialized 状态

- 触发条件：调用 `setDataSource(FileDescriptor)`, `setDataSource(String)`, `setDataSource(COntext, Uri)`, `setDataSource(FileDescriptor, long, long)` 其中一个函数
- 过程：MediaPlayer 状态将会从 Idle 变为 Initialized
- 异常：如果`setDataSource`在非 Idle 时调用，则会抛出`IllegalArgumentException`
- 注意：重载`setDataSource`方法是，需要抛出`IlleagalArgumentException`和`IOException`异常

### 5. Prepared 状态

- 触发条件
  - 同步方式
    - 调用 `prepare()` 方法，适用于本地音视频文件
    - 直接将状态从 Initialized --> prepared
  - 异步方式
    - `prepareAsync` 适用于网络数据，需要缓冲
    - 状态：Initialized --> preparing（时间较短） --> prepared
- 到达 Prepared 状态后，回调 `OnPreparedListener.onPrepared()`监听器










