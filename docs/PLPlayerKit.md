<a id="overview"></a>
# 1 概述

PLPlayerKit 是一个适用于 iOS 的 HLS 及 RTMP 播放 SDK，可高度定制化和二次开发。特色是支持 RTMP 协议下 H.264 编码 FLV 封装的多媒体流的播放，针对与用户体验密切相关的首开缓冲时间进行了优化，另外还根据移动网络的多变性以及直播场景对播放实时性的需求提供了跳帧机制。

<a id="function-version"></a>
## 1.1 功能以及版本

| 功能 | 描述 |版本|
|---|---|---|
| 支持 RTMP 协议直播播放 | 保证秒级实时性 | v1.1.0 |
| 提供 H.264 视频解码 | 多种 profile level 支持 | v1.1.0 |
| 提供 AAC、MP3 等多种音频解码 | 适配更多音频编码 | v1.1.0 |
| 支持双声道音频 | 更好的听觉体验 | v1.1.0 |
| 支持首屏秒开 | 更好的播放体验 | v1.1.0 |
| 支持弱网情况下跳帧机制 | 更好地满足直播场景的实时性需求  |v1.1.0 |
| 提供 HeaderDoc 文档 | 开发中使用 Quick Help 及时阅读文档 | v1.1.0 |
| 支持 ARM7, ARM64 指令集 | 为最新设备优化 | v1.1.0 |
| 支持模拟器运行 | 不影响模拟器快速调试 | v1.1.2 |
| 支持 HTTP-FLV 直播播放 | 更多的直播选择 | v2.2.0 |
| 支持 H.264 硬解 | 更低的 CPU 占用 | v2.2.0 |
| 提供基于 FFMPEG 点播 | 更多的播放选择 | v2.3.0 |
| 支持 HTTPS | 更多的协议支持 | v2.4.0 |
| 支持 H265 协议 | 更多的协议支持 | v3.0.0 |
| 支持 HLS 七牛私有 DRM 播放 | 更安全的播放 | v3.0.0 |

<a id="feature"></a>
## 1.2 特性

- [x] 高可定制
- [x] 直播累积延迟消除技术
- [x] 支持首屏秒开
- [x] 支持 RTMP 直播流播放
- [x] 支持 HTTP-FLV 直播流播放
- [x] 支持 HLS 播放
- [x] 支持 HTTPS 播放
- [x] 支持多种画面预览模式
- [x] 支持画面旋转与镜像
- [x] 支持播放器音量设置
- [x] 支持纯音频播放
- [x] 支持后台播放
- [x] 支持使用 IP 地址的 URL 
- [x] 支持软硬解自动切换
- [x] 支持 H265 协议
- [x] 支持 HLS 七牛私有 DRM

<a id="reading-user"></a>
# 2 阅读对象

本文档为技术文档，需要阅读者：
  - 具有基本的 iOS 开发能力
  - 准备接入七牛云直播

<a id="build-preparation"></a>
# 3 开发准备

<a id="system-preparation"></a>
## 3.1 设备以及系统

- iPhone 4s 及以上设备
- iOS 7

<a id="precondition"></a>
## 3.2 前置条件

- 已经注册七牛帐号
- 通过[官网](https://portal.qiniu.com)申请并已开通直播权限

<a id="updata"></a>
## 3.3 版本升级需知

### 3.3.1 符号重复引起的崩溃
- PLPlayerKit 从 v2.1.3 开始使用 ffmpeg 。
- PLPlayerKit 从 v2.4.0 开始使用 OpenSSL 与 Speex 。

- 会与其它同样使用使用 ffmpeg , OpenSSL 与 Speex 的 SDK 产生符号重复。某些情况下 Xcode Build 时，不会提示符号重复。但程序运行时，会出现崩溃。

- 在 Build Settings => Other Linker Flags 中添加 -all_load 选项，将使 Xcode Build 时，提示符号重复。 根据提示可以发现与哪个 SDK 有符号冲突。

- 相同的依赖库只可保留一份，避免冲突。

### 3.3.2 打印级别
- PLPLayerKit 从 v2.2.1 开始提供 SDK 日志级别设置。

- 通过 PLPlayerOption 中 PLPlayerOptionKeyLogLevel 选项设置。默认为 kPLLogNone 。

- 日志级别在 kPLLogInfo 及以上，则会缓存日志文件，日志文件存放的位置为 APP Container/Library/Caches/Pili/PlayerLogs 。

- APP 发布时使用 kPLLogNone 关闭打印。

- 提交 SDK 问题时，请使用 kPLLogDebug ，记录日志，提供给 SDK 开发人员分析定位问题。

### 3.3.3 DNS 解析
- PLPLayerKit 从 v2.2.1 开始使用 HappyDNS 做 DNS 解析。

- 如果你期望自己配置 dns 解析的规则，可以通过传递自己定义的 dns manager 来做 dns 查询。 通过 PLPlayerOption 中 PLPlayerOptionKeyDNSManager 选项设置。

- 如果你对 dns 解析部分不清楚，建议使用默认规则。

### 3.3.4 音频部分的特别说明

因为 iOS 的音频资源被设计为单例资源，所以如果在 player 中做的任何修改，对外都可能造成影响，并且带来不能预估的各种问题。

为了应对这一情况，PLPlayerKit 采取的方式是检查是否可以播放及是否可以进入后台，而在内部不做任何设置。具体是通过扩展 `AVAudioSession` 来做到的，提供了两个方法，如下：

```Objective-C
/*!
 * @description 检查当前 AVAudioSession 的 category 配置是否可以播放音频. 当为 AVAudioSessionCategoryAmbient,
 * AVAudioSessionCategorySoloAmbient, AVAudioSessionCategoryPlayback, AVAudioSessionCategoryPlayAndRecord
 * 中的一种时为 YES, 否则为 NO.
 */
+ (BOOL)isPlayable;

/*!
 * @description 检查当前 AVAudioSession 的 category 配置是否可以后台播放. 当为 AVAudioSessionCategoryPlayback,
 * AVAudioSessionCategoryPlayAndRecord 中的一种时为 YES, 否则为 NO.
 */
+ (BOOL)canPlayInBackground;
```

分别可以检查是否可以播放以及当前 category 的设置是否可以后台播放。

# 4 快速开始

<a id="preparation"></a>
## 4.1 开发环境配置

- Xcode 开发工具。App Store [下载地址](https://itunes.apple.com/us/app/xcode/id497799835?ls=1&mt=12)
- 安装 CocoaPods。了解 CocoaPods 使用方法。[官方网站](https://cocoapods.org)

<a id="sdk-import"></a>
## 4.2 导入 SDK

**PLPlayerKit 支持手动导入的方式添加。**

- 将 Pod 目录下的所有文件加入到工程中；
- 添加 HappyDNS 库，把 [链接](https://github.com/qiniu/happy-dns-objc) 中的 HappyDNS 目录下的所有文件加入到工程中；
- Build Setting 下 Other Linker Flags 中添加 -ObjC
- Build Phases 下 Link Binary With Libraries 中添加如图所示
![](http://7xng1t.com1.z0.glb.clouddn.com/PLPlayerKit%20Link%20Binary%20With%20Libraries.png)


<a id="initial-logical"></a>
## 4.3 初始化播放逻辑

<a id="viewcontroller"></a>
### 4.3.1 创建播放器用的 ViewController

* 创建 View Controller 如图所示，
![](http://7xuil4.com1.z0.glb.clouddn.com/CreatePlayer-01.png)

* 选择subclass为 `UIViewController`，如图所示
![](http://7xuil4.com1.z0.glb.clouddn.com/CreatePlayer-02.png)

<a id="reference-add"></a>
### 4.3.2 添加引用

* 在 `PLPlayerViewController.m` 中添加引用
```
#import <PLPlayerKit/PLPlayerKit.h>
```

<a id="declaration-add"></a>
### 4.3.3 添加声明属性

在 `PLPlayerViewController.h` 头文件中声明支持 `PLPlayerDelegate`，并添加一个 `PLPlayer` 的属性，添加完后如下

```Objective-C
#import <UIKit/UIKit.h>
#import <PLPlayerKit/PLPlayerKit.h>
@interface ViewController : UIViewController <PLPlayerDelegate>
@property (nonatomic, strong) PLPlayer  *player;
@end
```

<a id="security-add"></a>
### 4.3.4 添加 App Transport Security Setting

* 如图所示
![](http://7xuil4.com1.z0.glb.clouddn.com/permession.jpg)

<a id="object-create"></a>
### 4.3.5 创建对象

* 在 `PLPlayerViewController.m` 文件中，在 `- (void)viewDidLoad` 方法内实例化 PLPlayerOption 对象并改变所需设置的属性值用于初始化 PLPlayer 的相关配置参数并且实用实例化 PLPlayer，并且指定 delegate，特别说明的是，在创建 PLPlayer 时，我们要用到播放的 url，我假设你已经知道了直播对应的 rtmp 播放地址。
```Objective-C
PLPlayerOption *option = [PLPlayerOption defaultOption];
[option setOptionValue:@15 forKey:PLPlayerOptionKeyTimeoutIntervalForMediaPackets];
NSURL *url = [NSURL URLWithString:@"直播的 rtmp 地址"];
self.player = [PLPlayer playerWithURL:self.URL option:option];
self.player.delegate = self;
```

* 在创建完 player 后，我们将播放界面添加到当前视图
```Objective-C
[self.view addSubview:self.player.playerView];
```

<a id="play-start"></a>
### 4.3.6 开始播放

取一个简单的场景，就是当 PLPlayerViewController 加载完成即开始播放，那么我们在 `- (void)viewDidLoad` 方法内添加
```Objective-C
[self.player play]
```
这样在直播推流开始后，运行代码就可以看到直播流了。

<a id="demo"></a>
## 4.4 DEMO下载
- [点击下载](https://github.com/pili-engineering/PLPlayerKit)

<a id="function-usage"></a>
# 5 功能使用

当你要深入理解 SDK 的一些参数及有定制化需求时，可以从高级功能部分中查询阅读，以下小节无前后依赖。

<a id="PLPlayerOption"></a>
## 5.1 播放器可选配置项

`PLPlayerKit` 中通过初始化时传入不同配置的 PLPlayerOption 对象来设置不同的播放器可选配置项信息，对应的有

| KEY | value 类型 | 描述 | SDK版本 |
|---|---|---|---|
| `PLPlayerOptionKeyTimeoutIntervalForMediaPackets` | NSNumber | 播放器所用 RTMP 连接的超时断开时间长度，单位为秒。小于等于 0 表示无超时限制 | v1.0.0 |
| `PLPlayerOptionKeyMaxL1BufferDuration` | NSNumber | 一级缓存大小，单位为 ms，默认为 1000ms，增大该值可以减小播放过程中的卡顿率，但会增大弱网环境的最大累积延迟。 | v2.1.3 |
| `PLPlayerOptionKeyMaxL2BufferDuration` | NSNumber | 二级缓存大小，单位为 ms，默认为 1000ms，增大该值可以减小播放过程中的卡顿率，但会增大弱网环境的最大累积延迟。 | v2.1.3 |
| `PLPlayerOptionKeyVideoToolbox` | BOOL | 是否使用 video toolbox 硬解码。| v2.1.4 |
| `PLPlayerOptionKeyLogLevel` | PLLogLevel | 配置 log 级别 | v2.2.1 |
| `PLPlayerOptionKeyDNSManager` | QNDnsManager | 自定义 dns dnsmanager 查询，使用 HappyDNS | v2.2.1 |
| `PLPlayerOptionKeyHappyDNSEnable` | BOOL | 开启/关闭 HappyDNS 的 DNS 解析 | v2.3.0 |

<a id="background-configuration"></a>
## 5.2 后台播放配置

`PLPlayerKit` 提供后台播放支持，开发者可以实现后台与前台播放的无缝切换。

<a id="background-modes"></a>
### 5.2.1 配置 Required background modes

* 配置如图所示
![](http://7xuil4.com1.z0.glb.clouddn.com/AllowBackground-4.png)

<a id="player-attribute"></a>
### 5.2.2 设置 player 相关属性支持后台播放

```Objective-C
self.player.backgroundPlayEnable = YES;
[[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayback error:nil];
```

done！

这样， `PLPlayer` 便支持后台播放了。
需要注意的是在后台播放时仅有音频，视频会在回到前台时继续播放。

<a id="play-state"></a>
## 5.3 播放状态获取

在 `PLPlayerKit` 中，通过反馈 `PLPlayer` 的状态来反馈流的状态。我们定义了几种状态，确保 `PLPlayer` 对象在有限的几个状态间切换，并可以较好的反应流的状态。

| 状态名 | 含义 |
|---|---|
| PLPlayerStatusUnknow | 初始化时指定的状态，不会有任何状态会跳转到这一状态 |
| PLPlayerStatusPreparing | 播放器正在准备当中 |
| PLPlayerStatusReady | 播放器准备完成的状态 |
| PLPlayerStatusCaching | 播放器正在缓存的状态 |
| PLPlayerStatusPlaying | 播放器正在播放的状态 |
| PLPlayerStatusPaused | 播放器暂停的状态 |
| PLPlayerStatusStopped | 播放器播放结束或手动停止的状态 |
| PLPlayerStatusError | 播放器出现错误的状态 |
| PLPlayerStateAutoReconnecting | 播放器开始自动重连 |
| PLPlayerStatusCompleted | 点播播放完成 |

<a id="state-delegate"></a>
### 5.3.1 state 状态回调

state 状态对应的 Delegate 回调方法是

```Objective-C
// 实现 <PLPlayerDelegate> 来控制流状态的变更
- (void)player:(nonnull PLPlayer *)player statusDidChange:(PLPlayerStatus)state {
  // 这里会返回流的各种状态，你可以根据状态做 UI 定制及各类其他业务操作
  // 除了 Error 状态，其他状态都会回调这个方法
  // 开始播放，当连接成功后，将收到第一个 PLPlayerStatusCaching 状态
  // 第一帧渲染后，将收到第一个 PLPlayerStatusPlaying 状态
  // 播放过程中出现卡顿时，将收到 PLPlayerStatusCaching 状态
  // 卡顿结束后，将收到 PLPlayerStatusPlaying 状态
  // 点播结束后，将收到 PLPlayerStatusCompleted 状态
}
```

只有在正常连接，正常断开的情况下跳转的状态才会触发这一回调。所谓正常连接是指通过调用 `-play` 方法使得流连接的各种状态，而所谓正常断开是指调用 `-stop` 方法使得流断开的各种状态。所以只有以下状态会触发这一回调方法。

- PLPlayerStatusPreparing
- PLPlayerStatusReady
- PLPlayerStatusCaching
- PLPlayerStatusPlaying
- PLPlayerStatusPaused
- PLPlayerStatusStopped
- PLPlayerStateAutoReconnecting
- PLPlayerStatusCompleted


<a id="error-delegate"></a>
### 5.3.2 error 状态回调

error 状态对应的 Delegate 回调方法是

```Objective-C
- (void)player:(nonnull PLPlayer *)player stoppedWithError:(nullable NSError *)error {
    // 当发生错误，停止播放时，会回调这个方法
}

当除了调用 `-stop` 之外的所有能导致流断开的情况，都被归属于非正常断开的情况，此时就会触发该回调。对于错误的处理，我们不建议触发了一次 error 后就断掉，最好可以在此时尝试调用 `-play` 方法进行有限次数的重连。

<a id="exception-handling"></a>
## 5.4 网络异常处理

直播中，网络异常的情况比我们能意料到的可能会多不少，常见的情况一般有

- 网络环境切换，比如 3G/4G 与 Wi-Fi 环境切换
- 网络不可达，网络断开属于这一类
- 带宽不足，可能触发缓存速度跟不上播放速度

作为开发者我们不能乐观的认为只要是 Wi-Fi 网就是好的，因为即便是 Wi-Fi 也有可能因为运营商下行限制，共享网络带宽等因素导致以上网络异常情况的出现。

为何在直播中要面对这么多的网络异常情况，而在其他上传/下载中很少遇到的，这是因为直播对实时性的要求使得它不得面对这一情况，即无论网络是否抖动，是否能一直良好，直播都要尽可能是可持续，可观看的状态。

对于网络环境的切换，通常需要 App 整体做出调整，不单单是针对直播，所以 `PLPlayerKit` 并未对这一情况做额外的监听，而是需要开发者自己对这些状态做出处理。

<a id="reconnection"></a>
### 5.4.1 重连

`PLPlayerKit` 内部不包含重连逻辑。之所以不包含，主要因素是考虑到 App 的业务逻辑场景多样而复杂，对于直播重连的次数，时机，间隔都会有不同的需求，而此时应该让开发者自己来决定是否重连，以及尝试重连的次数。

当因为网络异常而触发了播放断开时，会通过 error Delegate 回调触发

```Objective-C
- (void)player:(nonnull PLPlayer *)player stoppedWithError:(nullable NSError *)error;
```

你可以在这个方法内通过重新调用 `-play` 方法来尝试重连。此处建议不要立即重连，而是采用重连间隔加倍的方式，比如共尝试 3 次重连，第一次等待 0.5s, 第二次等待 1s, 第三次等待 2s，这样的方式主要考虑到弱网时网络带宽的缓解需要时间，而加倍重连可以更容易在网络恢复的时候连接，而非在网络已经拥塞时还不断做无用功的重连。

<a id="ip-play"></a>
## 5.5 使用 IP 播放

使用 IP 播放的地址格式为 : http://ip:port/path?domain=xxx  

通过 domain 传入对应的播放域名，播放将直接使用地址中的 IP ，不再做 DNS 解析。

<a id="tips"></a>
# 6 知识补充与建议

<a id="frame-strategy"></a>
## 6.1 跳帧策略

下面我们来看看跳帧策略

### 6.1.2 什么是跳帧策略

跳帧策略是指在播放累积延迟逐渐增大时采取的 " 丢弃非实时数据 " 的策略，播放累积延迟是由于缓存不足或其他原因导致的播放暂停而后继续播放会优先播放暂停之前未播放的非实时数据，而实时数据进入缓存队列造成的播放端播放的音视频数据与实时数据之间的时间差，而跳帧策略则是指当播放器发现播放累积延迟超过一定阈值时触发的 " 丢弃非实时数据 " 的策略。

### 6.1.3 为什么要采取跳帧策略

原因非常简单，就是为了保证播放的实时性需求。

直播作为有别于录播的富媒体传播手段，它的第一要素就是实时，没有了实时，直播的价值就会荡然无存。除了实时性以外，非实时数据堆积造成的内存增加也是不得不考虑的问题，在移动设备有限的内存条件之下，我们应该尽可能保证比较小的内存占用率。跳帧策略能够在保证实时性需求的同时减少播放的内存占用率，因此我们有必要采取跳帧策略。


### 6.1.4 利弊

丢帧策略固然保证了直播的实时性，但是它的弊端也是显然的，就是会导致信息的丢失，观众在观看过程中如果发生暂停，那么继续播放的时候暂停期间的音视频信息观众将无法观看到。

因此，我们建议开发者从产品层面考虑直播的实时性与观众获取到的直播的完整性的取舍问题。

<a id="api-reference"></a>
# 7 API参考

- [API 参考地址](http://cocoadocs.org/docsets/PLPlayerKit)

<a id="document-history"></a>
# 8 历史记录
- 2.4.3 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.4.3.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.4.3.md))
- 功能
  - 新增流分辨率变化的通知
  - 新增提供更多音视频信息的回调接口
  - 新增首开耗时接口
  - 增强 FFmpeg 点播硬解兼容性
- 缺陷
  - 修复 AVPlayer 点播 pause 状态切换时播放器状态异常的问题
  - 修复 FFmpeg 点播纯音频流时 seek 失败的问题
  - 修复硬解在某些场景下出现绿屏的问题
- 2.4.2 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.4.2.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.4.2.md))
- 缺陷
  - 修复 AVPlayer 播放时调用 pause 和设置 frame 无效的问题
  - 修复解码器释放时线程并发导致的偶发 crash
- 2.4.1 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.4.1.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.4.1.md))
- 功能
  - 新增 probesize 参数配置
  - 新增播放器初始化后更新 URL 的接口
  - 新增 AVPlayer 点播的缓冲进度接口
  - 增加 http header 中 referer 自定义接口
- 缺陷
  - 修复锁屏且屏幕黑后，播放没有声音的问题
  - 修复播放器释放时偶发的 crash
- 2.4.0 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.4.0.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.4.0.md))
- 功能
  - 新增 https 支持
  - 新增文件播放
  - 新增 speex, ogg 等音视频格式， avi, m4a 等封装格式支持。
  - 新增 display aspect ratio 信息
  - 新增 DNS 预解析接口
  - 新增开播前封面图
- 缺陷
  - 修复一些偶发的 crash
- 2.3.0 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.3.0.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.3.0.md))
- 功能
  - 新增直播流画面旋转模式
  - 新增直播流分辨率信息
  - 新增停止渲染的选项
  - 新增基于 FFMPEG 的点播
- 缺陷
  - 修复一些偶现的 crash
- 优化
  - 优化开始播放的快进时间
- 2.2.4 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.2.4.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.2.4.md))
- 缺陷
  - 修复与 CocoaLumberjack 符号冲突的问题
- 2.2.3 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.2.3.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.2.3.md))
- 功能
  - 新增 QoS 功能
  - 新增渲染数据回调
  - 新增截图功能
  - 新增 MP3 后台播放
- 缺陷
  - 修复后台播放时，触发超时重连，丢失 sps/pps，回到前台画面停住，声音正常的问题
  - 修复 RTMP 扩展时间戳的问题
  - 修复播放器释放阻塞主线程的问题
  - 优化音视频同步机制
  - 优化 caching 状态检查
- 2.2.2 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.2.2.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.2.2.md))
- 功能
  - 新增 AAC HEV2 音频支持
  - 新增 SDK 自动重连功能，默认不开启
- 缺陷
  - 修复长时间播放偶发解码 crash
  - 修复 pause/resmue 快速调用导致 crash
  - 修复重连未更换服务器 IP
  - 修复 rtmp 硬解播放视频抖动
  - 修复 flv 开始播放偶发黑屏
  - 修复 flv 超时机制失效
- 2.2.1 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.2.0.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.2.1.md))
- 功能
  - 支持 SDK 日志级别设置
  - 新增 HappyDNS 支持
- 缺陷
  - 修复回看状态不准确问题
  - 修复跳转第三方应用，出现内存增加。
  - 修复播放卡住 caching 状态。
- 2.2.0 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.2.0.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.2.0.md))
- 功能
    - 新增硬解功能
    - 新增 http-flv 支持
    - 新增 iOS9 下的纯 IPV6 环境支持
- 缺陷
    - 修复快速进入退出黑屏
- 优化
    - 追帧策略优化
    - 退出后台停止视频解码
- 2.1.3 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.1.3.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.1.3.md))
    - 增加设置一级缓存和二级缓存的选项，便于控制卡顿率
    - 修复播放 OBS 及 FFmpeg 推的流黑屏的问题
    - 修复播放结束后无法重播的问题
    - 修复播放过程中内存暴增的问题
    - 拆分 pili-librtmp 为公共依赖，解决模拟器环境下与 PLStreamingKit 冲突的问题
- 2.1.2 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.1.2.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.1.2.md))
    - 增加确切的错误枚举，方便定位错误类型
    - 增加 mute, currentTime, totalDuration, seekTo 等接口
    - 修复首屏开启以及播放过程中出现缓存后网络恢复是可能出现的 UI 卡顿问题
    - 修复 contentMode 偶尔设置无效的问题
    - 修复重新设置播放 url 播放的问题
    - 修复快速 -stop 以及 -play 出现的内存泄露问题
- 2.1.1 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.1.1.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.1.1.md))
    - 首屏开启速度优化，在网络状况良好的情况下能实现秒开效果
    - 弱网情况下的累积延迟问题优化，较好控制累积延迟在数秒以内
    - 解决了上一版遇到的无法设置 playerView.contentMode 以及 playerOption 的问题
    - 解决了不标准流可能出现的音频断续，播放器内存异常增长问题
    - 后台播放体验优化，修复了后台播放被其他音频打断后出现的一系列问题
    - 解决了应用切换时出现的 UI 卡死问题
- 2.1.0 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.1.0.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.1.0.md))
    - 此次更新为重大版本升级，更改了大量 API 并重构了包括解码渲染在内的多项内容，建议所有用户进行升级，并且根据[快速开始](#快速开始)使用新版 API 对工程重新进行配置。
    - 更改了播放器的音频解码和渲染方式
    - 更改了播放器的时钟同步机制
    - 重构了内部逻辑，使播放器更稳定
    - 重构了播放器 API ，使播放器的使用更加简单明了，去除了使用起来不方便的部分 API
    - 解决了播放过程中可能出现声音消失的问题
    - 解决了退后台返回后音视频无法正常同步的问题
    - 修改播放器音视频同步机制
    - 解决持续播放过程中出现部分内存没有正确释放的问题
    - 解决了 iOS 版本小于 8.0 时 Demo 出现的crash问题
- 2.0.4 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.0.4.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.0.4.md))
  - 解决 RTMP 播放时可能黑屏的问题
- 2.0.3 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.0.3.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.0.3.md))
  - 解决 RTMP 播放没有声音
  - 解决 RTMP 无法播放导致内存急增最终 App crash
  - 解决 RTMP 无法播放画面只有声音
  - 解决播放 RTMP 时相关的 crash 问题
- 2.0.2 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.0.2.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.0.2.md))
  - 添加 RTMP Cache 机制
  - 添加数据超时属性
  - 修复 RTMP 播放内存 leak
  - 修复 RTMP 播放音频错误问题
  - 修复 RTMP 播放主线程卡死问题
  - 优化架构，减少内存和 cpu 占用
- 2.0.1 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.0.1.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.0.1.md))
  - 修复 `contentMode` 设置无效的问题
  - 修复 rtmp 无法播放或播放超时时无 error 抛出的问题
  - 修复 rtmp 播放失败时触发的 cpu 飙升问题
  - 修复 stop 可能触发的 crash 问题
  - 更新 demo 确保在 iOS 9.1 下运行正常
- 2.0.0 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-2.0.0.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-2.0.0.md))
  - 添加全新的 `PLPlayer`，弃用 `PLVideoPlayerController` 和 `PLAudioPlayerController`
  - 播放 RTMP 音视频流时，进入后台后声音继续播放，不会断开，返回前台追帧显示最新视频帧
  - 针对 RTMP 直播彻底优化，首屏秒开，最小化缓存
  - 完全无 ffmpeg 依赖，包体积再次缩小
  - 优化资源占用，比 1.x 版本内存占用减少 50% 以上
- 1.2.22 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.22.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.22.md))
  - 修复因收到内存警告而引起的崩溃问题
  - 修复停止播放时，可能进入错误 play state 的问题
- 1.2.21 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.21.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.21.md))
  - 修复 `PLVideoParameterFrameViewContentMode` 与 `PLVideoParameterDisableDeinterlacing` 设置无效的问题
- 1.2.20 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.20.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.20.md))
  - 修复 `seekTo:` 不准确的问题
  - 添加 `PLPlayerStateSeeking` 类型
- 1.2.19 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.19.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.19.md))
  - 修复播放无返回状态的问题（针对无直播的流、hls 回放）
  - 修复 hls 回放结束时无 stopped 回调的问题
  - 修复 hls 回放开始的 duration 不为 0 的问题
- 1.2.18 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.18.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.18.md))
  - 修复在 prepare 状态前释放 player 导致的音频仍然会播放的问题
  - 修复 player 状态返回的类型不正确的问题
  - 优化推出时资源释放
- 1.2.17 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.17.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.17.md))
  - 修复超时时导致的崩溃的问题
- 1.2.16 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.17.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.16.md))
  - 添加了音频播放器后台播放的支持
  - 添加了音频播放器后台播放任务开始和结束的回调
  - 添加了音视频播放器超时时长的设定
  - 添加了音视频播放器准备的方法
  - 添加了音视频完全停止播放器的方法
  - 修复播放器不可释放的问题
- 1.2.15 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.15.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.15.md))
  - 修复 AudioPlayer 无法播放带有视频流的 RTMP 流的问题
- 1.2.14 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.14.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.14.md))
  - 添加 AudioManager
- 1.2.13 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.13.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.13.md))
  - 添加纯音频播放控件
  - 更新参数字段及类型，确保通用类型可以在音频及视频播放器使用
  - 更新类型名称，增加易读性，减少歧义
- 1.2.12 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.12.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.12.md))
    - 更改 repo 地址
- 1.2.11 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.11.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.11.md))
    - 添加对应用状态的判断，减少因进入后台通知延时未能及时暂停播放导致的 crash
- 1.2.10 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.10.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.10.md))
    - 添加音频外设更改时的通知
    - 添加音量变更时的通知
    - 添加打进电话等其他事件导致音频中断的通知
- 1.2.9 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.9.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.9.md))
    - 修复进入后台后崩溃的问题
    - 更新 example 中 player 代码，支持横竖屏旋转操作
- 1.2.8 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.8.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.8.md))
    - 添加播放进度回调方法
    - 修复 seekTo 后流状态不正确的问题
- 1.2.7 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.7.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.7.md))
    - 添加播放器状态属性
    - 添加解码器初始化完成后回调
    - 添加播放器状态回调
    - 添加初始化后自动播放参数
- 1.2.6 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.6.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.6.md))
    - 添加设置播放位置的操作
    - 添加了快进、快退的操作
    - 添加总播放时长的属性
    - 添加获取音量的属性
    - 添加获取当前播放位置的属性
    - 添加静音操作
- 1.2.5 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.5.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.5.md))
    - 修复与部分其他库头文件冲突的问题
- 1.2.4 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.4.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.4.md))
    - 添加了 ```PLMovieParameterFrameViewContentMode``` 参数
    - 修复与部分其他库头文件冲突的问题
    - 修复 Player contentMode 无法更改的问题
- 1.2.3 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.3.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.3.md))
    - 修复初始化占用主线程导致卡顿的问题
    - 修复错误回调无效的问题
- 1.2.2 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.2.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.2.md))
    - 修复 lib 未更新导致的 crash
- 1.2.1 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.1.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.1.md))
    - 添加 failue 情况下的回调，返回 NSError 对象
    - 移除 PLVideoPlayerViewController，请直接使用 PLVideoPlayerController 进行定制
- 1.2.0 ([Release Notes](https://github.com/pili-engineering/PLPlayerKit/blob/master/ReleaseNotes/release-notes-1.2.0.md) && [API Diffs](https://github.com/pili-engineering/PLPlayerKit/blob/master/APIDiffs/api-diffs-1.2.0.md))
    - 极大缩小 lib 大小
    - 增加可定制的播放控件 PLVideoPlayerController
- 1.1.2
    - 拆分 Flat lib
    - 添加了 x86_64 支持，便于在 iPhone 6 Plus 模拟器下调试使用
- 1.1.1
    - 对库引用做了些修改
- 1.1.0
    - 发布 CocoaPods 版本
