- [JNI知识点](#jni---)
  * [动态注册 vs 静态注册](#-----vs-----)
    + [静态注册](#----)
    + [动态注册](#----)
  * [JavaVM vs JNIEnv](#javavm-vs-jnienv)
  * [JNI中多线程回调到Java层如何实现](#jni-------java-----)
  * [局部引用 vs 全局引用](#-----vs-----)
    + [局部引用](#----)
    + [全局引用](#----)
    + [弱全局引用](#-----)
- [MediaCodec知识点](#mediacodec---)
  * [MediaCodec同步 vs 异步](#mediacodec---vs---)
  * [MediaCodec工作原理](#mediacodec----)
  * [MediaCodec解码导致绿边问题](#mediacodec--------)
  * [MediaCodec如何提升清晰度](#mediacodec-------)
- [音视频基础知识点](#--------)
  * [H264编码流程](#h264----)
    + [H264压缩技巧](#h264----)
    + [H264压缩过程](#h264----)
  * [H264 vs H265](#h264-vs-h265)
  * [SPS vs PPS](#sps-vs-pps)
  * [FLV格式介绍](#flv----)
  * [TS格式介绍](#ts----)
  * [MP4格式介绍](#mp4----)
  * [HLS格式介绍](#hls----)
  * [DASH格式介绍](#dash----)
  * [RTMP协议分析](#rtmp----)
    + [RTMP协议介绍](#rtmp----)
    + [RTMP为什么还要建连](#rtmp-------)
    + [RTMP建连流程](#rtmp----)
- [Android基础知识](#android----)
  * [Dalvik vs ART](#dalvik-vs-art)
  * [GC Root有哪些](#gc-root---)
  * [SurfaceView vs TextureView](#surfaceview-vs-textureview)
- [播放器知识点](#------)
  * [播放器架构分析](#-------)
  * [播放器成功率优化](#--------)
  * [播放器首帧速度优化](#---------)
  * [播放器卡顿优化](#-------)
  * [使用MediaExtractor和MediaCodec播放视频](#--mediaextractor-mediacodec----)
  * [ffplay完整执行流程](#ffplay------)
- [FFmpeg知识点](#ffmpeg---)
- [Camera知识点](#camera---)
  * [Camera vs Camera2](#camera-vs-camera2)
- [音视频合成问题](#-------)
  * [如何将两个视频拼接为一个视频](#--------------)
  * [如何将两个音频拼接为一个音频](#--------------)
  * [MediaExtractor如何做精确seek](#mediaextractor-----seek)
- [C++知识点](#c-----)
  * [智能指针](#----)
  * [lamba表达式](#lamba---)
  * [vector底层扩容原理](#vector------)

## JNI知识点
### 动态注册 vs 静态注册
执行一个java的native方法，要想让虚拟机知道调用so库中的哪一个方法，需要知道native方法和so库中对应的函数映射表，如何将native方法和so库中函数映射起来，就用到了注册的概念。
注册分为动态注册和静态注册，默认情况下是静态注册，我们不需要管。
#### 静态注册
通过JNIEXPORT和JNICALL两个宏定义声明，在虚拟机加载so的时候通过这两个宏定义来查找对应的native方法。
通常的命名规则是：Java_包名_类名_方法名

静态注册的优点：
> * 非常简单明了，开发者只要遵循jni的规则就行，不需要额外的费事了

缺点：
> * 必须按照特定的命名规则
> * 函数的名字太长了
> * 运行时效率不高
#### 动态注册
动态注册，顾名思义，就是在运行过程中通过调用RegisterNatives方法手动实现native方法和so中方法的绑定，虚拟机可以通过函数映射表直接找到对应的方法。
实现动态注册的地方是JNI_OnLoad方法，这是so加载的入口。如果动态注册成功返回JNI_OK，失败则返回一个负的值。

### JavaVM vs JNIEnv
JavaVM 代表java的虚拟机，Android中一个进程只有一个JavaVM，我们so加载的入口就是JNI_OnLoad(JavaVM* jvm, void* reverved)

JNIEnv是提供JNI Native函数的基础环境，不同的线程的JNIEnv相互独立的。

在native层中，想要获得当前线程所使用的JNIEnv，可以使用Dalvik虚拟机对象的JavaVM* jvm->GetEnv()返回当前线程的JNIEnv*
### JNI中多线程回调到Java层如何实现
从上面的分析已经得知JavaVM是进程相关的，JNIEnv是线程相关的，可以通过JavaVM->AttachCurrentThread获取子线程的JNIEnv引用，在调用结束之后，通过JavaVM->DetachCurrentThread()解除挂在当前的线程。

### 局部引用 vs 全局引用
#### 局部引用
通过NewLocalRef和各种JNI接口创建，例如可以通过FindClass/NewObject/GetObjectClass等JNI接口创建，不能再本地函数中跨函数调用，也不能跨线程使用，函数返回中局部引用的对象会被JVM自动释放，也可以调用DeleteLocalRef释放。

形成一个良好的开发习惯，在创建了局部引用之后，如果已经使用过了，还是建议手动释放。
#### 全局引用
通过调用NewGlobalRef创建，可以跨方法、跨线程使用，JVM不会自动释放，必须要使用DeleteGlobalRef手动释放。
#### 弱全局引用
通过调用NewWeakGlobalRef创建，一般情况下引用不会自动释放，当内存比较紧张的情况下，JVM也会回收它的，可以通过DeleteWeakGlobalRef手动释放。
## MediaCodec知识点
### MediaCodec同步 vs 异步
### MediaCodec工作原理
### MediaCodec解码导致绿边问题
### MediaCodec如何提升清晰度
## 音视频基础知识点
### H264编码流程
#### H264压缩技巧
H264压缩技术主要采用了以下几种方法对视频数据进行压缩：
> * 帧内预测压缩，解决的是空间数据冗余的问题
> * 帧间预测压缩，就是运动估计与补偿，解决是时域数据冗余的问题
> * 整数离散余弦变换（DCT），将空间上的相关性变为频域上无关的数据进行量化处理
> * CABAC压缩

经过压缩后的帧分为：I帧、P帧、B帧
> * I帧：关键帧，采用的帧内压缩技术
> * P帧：向前参考帧，在压缩时，只参考前面已经处理过的帧
> * B帧：双向参考帧，在压缩时，参考前面的帧，又参考后面的帧，采用帧间压缩技术

一个GOP：Group of pictures，一组图像序列，两个I帧之间是一个图像序列。

#### H264压缩过程
> * 原始数据送到H264编码器的缓冲区，编码器为每一帧图片划分宏块。
> * H264默认使用16 * 16 大小的区域作为一个宏块，划分好宏块，计算宏块的像素值，进而计算每一帧图片中所有宏块的像素值
> * 在宏块的基础上划分更小的子块，子块大小可以自选，但是要求比宏块更小
> * 进行帧分组，视频数据有时间上的数据冗余和空间上的数据冗余，时间上的数据冗余比较大。相邻帧的图片差异不会很大。这样我们就推出了I帧、B帧、P帧的概念
> * 运动估计与补偿
> * 帧内预测
> * 对残差数据做DCT
> * CABAC，属于无损压缩技术

### H264 vs H265
H265仍然采用混合编解码，编解码结构与H264基本一致。
主要提升在于：
> * 编码块划分结构：采用CU/PU/TU的递归结构
> * 并行工具：增加了Tile以及WPP等并行工具集以提高编码速度
> * 滤波器：在区块滤波之后增加了SAO滤波模块
> * H265优化了实现细节，对高清视频的处理更加精细

### SPS vs PPS

### FLV格式介绍
Flv格式的视频由Flv Header和Flv Body组成，Flv Body由一个一个Tag组成，每个Tag都有一个preTagSize字段，标记着前面一个Tag的大小。
### TS格式介绍
### MP4格式介绍
### HLS格式介绍
### DASH格式介绍
### RTMP协议分析
#### RTMP协议介绍
RTMP协议是应用层协议，是要靠底层可靠的传输层（TCP），
协议（通常是TCP）来保证信息传输的可靠性的。在基于传输层协议的链接建立完成后，RTMP协议也要客户端和服务器通过“握手”来建立基于传输层链接之上的RTMP Connection链接。播放一个RTMP协议的流媒体需要经过以下几个步骤：握手，建立网络连接，建立网络流，播放。服务器和客户端之间只能建立一个网络连接，但是基于该连接可以创建很多网络流。
#### RTMP为什么还要建连
这儿埋下一个小疑问？为什么传输层已经建立了TCP连接，RTMP还需要再次建立一个连接，有这个必要吗？

RTMP协议传输时会对数据做自己的格式化，这种格式的消息我们称之为RTMP Message，而实际传输的时候为了更好地实现多路复用、分包和信息的公平性，发送端会把Message划分为带有Message ID的Chunk，每个Chunk可能是一个单独的Message，也可能是Message的一部分，在接受端会根据chunk中包含的data的长度，message id和message的长度把chunk还原成完整的Message，从而实现信息的收发。

因为它们需要商量一些事情，保证以后的传输能正常进行。主要就是两个事情，一个是版本号，如果客户端、服务器的版本号不一致，则不能工作。另一个就是时间戳，视频播放中，时间是很重要的，后面的数据流互通的时候，经常要带上时间戳的差值，因而一开始双方就要知道对方的时间戳。


#### RTMP建连流程
> * RTMP握手
> * 拉流
> * 播放

RTMP握手流程:
> * 客户端发送 C0、C1、 C2，服务器发送 S0、 S1、 S2。
> * 首先，客户端发送 C0 表示自己的版本号，不必等对方的回复，然后发送 C1 表示自己的时间戳。
> * 服务器只有在收到 C0 的时候，才能返回 S0，表明自己的版本号，如果版本不匹配，可以断开连接。
> * 服务器发送完 S0 后，也不用等什么，就直接发送自己的时间戳 S1。客户端收到 S1 的时候，发一个知道了对方时间戳的 ACK C2。同理服务器收到 C1 的时候，发一个知道了对方时间戳的 ACK S2。
> * 握手建立完成。

RTMP拉流流程:
> * 建立网络连接---> Connect
> * 创建网络流---> Create Stream
> * 播放---> Play

## Android基础知识
### Dalvik vs ART
### GC Root有哪些
### SurfaceView vs TextureView
## 播放器知识点
### 播放器架构分析
### 播放器成功率优化
### 播放器首帧速度优化
### 播放器卡顿优化
### 使用MediaExtractor和MediaCodec播放视频
### ffplay完整执行流程
## FFmpeg知识点
## Camera知识点
### Camera vs Camera2
## 音视频合成问题
### 如何将两个视频拼接为一个视频
### 如何将两个音频拼接为一个音频
### MediaExtractor如何做精确seek
## C++知识点
### 智能指针
### lamba表达式
### vector底层扩容原理
