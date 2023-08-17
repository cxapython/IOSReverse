0.mac电脑工具安装

```shell
  brew install libimobiledevice
  brew install ideviceinstaller
```

1.开始ssh

打开爱思助手的工具箱有个ssh通道打开。

之后根据提示登录ios手机即可

```
ssh -p 2222 root@127.0.0.1
```

之后输入默认密码alpine,即可进入ios系统。

2.安装scp

为了方便手机和电脑的文件进行传输，这里推荐安装scp。

将电脑的文件推送到ios可以使用下面的命令

```shell
scp -P2222 frida_14.2.12_iphoneos-arm.deb root@localhost:/tmp/
```



3.frida安装

电脑端安装

 ["frida==14.2.12", "frida-tools==9.1.0", "objection==1.9.6"]

ios端需要的

```shell
https://github.com/frida/frida/releases/download/14.2.12/frida_14.2.12_iphoneos-arm.deb
```

然后通过上面的scp里面的命令推送到手机。下面是在手机上执行的命令

```
cd /tmp/

dpkg -i frida_14.2.12_iphoneos-arm.deb
```

输出内容

```shell
iPhone:/tmp root# dpkg -i frida_14.2.12_iphoneos-arm.deb
Selecting previously unselected package re.frida.server.
(Reading database ... 3847 files and directories currently installed.)
Preparing to unpack frida_14.2.12_iphoneos-arm.deb ...
Unpacking re.frida.server (14.2.12) ...
Setting up re.frida.server (14.2.12) ...
```

 之后查看服务是否成功

```shell
iPhone:/tmp root# ps -ef |grep frida
    0  3597     1   0  9:29PM ??         0:00.05 /usr/sbin/frida-server
    0  3601  3533   0  9:30PM ttys000    0:00.01 grep frida
```

或者在电脑执行

```shell
frida-ps -U -ai
```

看到下面类似结果即表示成功

```shell
 PID  Name          Identifier
----  ------------  ---------------------------
2499  Cydia         com.saurik.Cydia
2469  NewTerm       ws.hbang.Terminal
2866  checkra1n     kjc.loader
 359  邮件            com.apple.mobilemail
   -  App Store     com.apple.AppStore
   -  FaceTime 通话   com.apple.facetime
   -  Filza         com.tigisoftware.Filza
```

4.ios砸壳

工具很多推荐使用frida-ios-dump。

```
git clone https://github.com/AloneMonkey/frida-ios-dump
cd frida-ios-dump/
#安装依赖
python -m pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
#低版本的使用包名,frida15以及以上使用app名。
python dump.py net.whatsapp.WhatsApp
```

输出下面内容即为砸壳成功

```shell
 chennan@womendeMacBook-Pro  ~/IOSReverse/frida-ios-dump   master  python dump.py net.whatsapp.WhatsApp
Start the target app net.whatsapp.WhatsApp
Dumping WhatsApp to /var/folders/k7/mjcqygh14y7bfs2l0yywrnbm0000gn/T
[frida-ios-dump]: libswift_Concurrency.dylib has been dlopen.
[frida-ios-dump]: SharedModules.framework has been loaded.
[frida-ios-dump]: Load WABloksKit.framework success.
start dump /var/containers/Bundle/Application/793EEA72-107B-4890-A4F0-FE7993210395/WhatsApp.app/WhatsApp
WhatsApp.fid: 100%|████████████████████████| 53.9M/53.9M [00:01<00:00, 33.5MB/s]
start dump /private/var/containers/Bundle/Application/793EEA72-107B-4890-A4F0-FE7993210395/WhatsApp.app/Frameworks/SharedModules.framework/SharedModules
SharedModules.fid: 100%|███████████████████| 26.7M/26.7M [00:00<00:00, 32.6MB/s]
start dump /private/var/containers/Bundle/Application/793EEA72-107B-4890-A4F0-FE7993210395/WhatsApp.app/Frameworks/libswift_Concurrency.dylib
libswift_Concurrency.dylib.fid: 100%|████████| 407k/407k [00:00<00:00, 4.96MB/s]
start dump /var/containers/Bundle/Application/793EEA72-107B-4890-A4F0-FE7993210395/WhatsApp.app/Frameworks/WABloksKit.framework/WABloksKit
WABloksKit.fid: 100%|██████████████████████| 19.4M/19.4M [00:00<00:00, 31.2MB/s]
Info.plist: 149MB [00:12, 12.6MB/s]
0.00B [00:00, ?B/s]Generating "WhatsApp.ipa"
```

之后解压WhatsApp.ipa，得到一个playload文件夹里面有个WhatsApp.app，查看WhatsApp.app的类型

```shell
file WhatsApp.app
```

可以看到这是个目录，所以可以进入该目录，之后我们搜索mach-o

```
file * |grep -i mach
WhatsApp:                                           Mach-O 64-bit executable arm64
```

可以知道该目录里面的WhatsApp文件就是Mach-O文件类型，类似ELF。

5.Mach-O文件查看

我们可以使用

这里有网友给编译好了直接用就行了,下载地址MachOView查看。

```shell
https://github.com/fangshufeng/MachOView/releases
```

后续补充

6.动态分析app

手机安装 cycript

```shell
iPhone:~ root# cycript -p WhatsApp
```

下面的命令打印任意视图层次结构

```
cy# [[UIApp keyWindow]recursiveDescription ].toString()
```

输出内容

```shell
`<WAWindow: 0x104b04f30; baseClass = UIWindow; frame = (0 0; 375 667); tintColor = UIExtendedSRGBColorSpace 0 0.478431 1 1; gestureRecognizers = <NSArray: 0x282694420>; layer = <UIWindowLayer: 0x282859d40>>
   | <UITransitionView: 0x104bf1c20; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x2828acdc0>>
   |    | <UILayoutContainerView: 0x1069665b0; frame = (0 0; 375 667); clipsToBounds = YES; autoresize = W+H; gestureRecognizers = <NSArray: 0x2826fd620>; layer = <CALayer: 0x2828ab8c0>>
   |    |    | <UINavigationTransitionView: 0x104bf0260; frame = (0 0; 375 667); clipsToBounds = YES; autoresize = W+H; layer = <CALayer: 0x2828ac880>>
   |    |    |    | <UIViewControllerWrapperView: 0x104b158a0; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x282877d00>>
   |    |    |    |    | <UIView: 0x104bc2870; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x2828ac600>>
   |    |    |    |    |    | <UIImageView: 0x104b2af00; frame = (56.5 80.5; 262.5 262.5); opaque = NO; userInteractionEnabled = NO; animations = { UIImageAnimation=<CAKeyframeAnimation: 0x282862880>; }; layer = <CALayer: 0x2828aca80>>
   |    |    |    |    |    | <UILabel: 0x106954e10; frame = (24 403.5; 327 33.5); text = '\u200e\u6b22\u8fce\u4f7f\u7528 WhatsApp'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x280b74230>>
   |    |    |    |    |    |    | <_UILabelContentLayer: 0x282865200> (layer)
   |    |    |    |    |    | <WALinkLabel: 0x106924530; baseClass = UILabel; frame = (16 457; 343 36); text = '\u200e\u8bf7\u9605\u8bfb\u6211\u4eec\u7684 \u9690\u79c1\u653f\u7b56\u3002\u70b9\u51fb \u201c\u200e\u540c\u610f\u5e76\u7ee7\u7eed\u201d ...'; layer = <_UILabelLayer: 0x280b75770>>
   |    |    |    |    |    |    | <_UILabelContentLayer: 0x282857e00> (layer)
   |    |    |    |    |    | <UIButton: 0x106924a70; frame = (16 529.5; 343 39); opaque = NO; layer = <CALayer: 0x2828a1420>>
   |    |    |    |    |    |    | <UIButtonLabel: 0x104b19cd0; frame = (116 6.5; 111.5 26.5); text = '\u200e\u540c\u610f\u5e76\u7ee7\u7eed'; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x280b76a80>>
   |    |    |    |    |    |    |    | <_UILabelContentLayer: 0x2828a5520> (layer)
   |    |    |    |    |    | <UIView: 0x106923290; frame = (67.5 20; 240 60.5); layer = <CALayer: 0x2828a1c00>>
   |    |    |    |    |    | <UIView: 0x106926ee0; frame = (67.5 343; 240 60.5); layer = <CALayer: 0x2828a1d80>>
   |    |    |    |    |    | <UIView: 0x106904f20; frame = (67.5 493; 240 36.5); layer = <CALayer: 0x2828a1ec0>>
   |    |    |    |    |    | <UIView: 0x1069299f0; frame = (67.5 618.5; 240 48.5); layer = <CALayer: 0x2828a1fc0>>`
```

上面的内容有点乱看着不舒服,UIWindow有一个_autolayoutTrace私有函数，可以打印整个视图的层次结构

简化一下

```
cy# [[UIApp keyWindow] _autolayoutTrace ].toString()
```

输出内容

```shell
`
WAWindow:0x104b04f30
|   UITransitionView:0x104bf1c20
|   |   UILayoutContainerView:0x1069665b0
|   |   |   UINavigationTransitionView:0x104bf0260
|   |   |   |   UIViewControllerWrapperView:0x104b158a0
|   |   |   |   |   \u2022UIView:0x104bc2870
|   |   |   |   |   |   *<UILayoutGuide: 0x281125b20 - "UIViewLayoutMarginsGuide", layoutFrame = {{16, 20}, {343, 647}}, owningView = <UIView: 0x104bc2870; frame = (0 0; 375 667); autoresize = W+H; layer = <CALayer: 0x2828ac600>>>
|   |   |   |   |   |   *UIImageView:0x104b2af00
|   |   |   |   |   |   *UILabel:0x106954e10'\u200e\u6b22\u8fce\u4f7f\u7528 WhatsApp'
|   |   |   |   |   |   *WALinkLabel:0x106924530'\u200e\u8bf7\u9605\u8bfb\u6211\u4eec\u7684 \u9690\u79c1\u653f\u7b56\u3002\u70b9\u51fb \u201c\u200e\u540c\u610f\u5e76\u7ee7\u7eed\u201d ...'
|   |   |   |   |   |   *UIButton:0x106924a70'\u200e\u540c\u610f\u5e76\u7ee7\u7eed'
|   |   |   |   |   |   |   UIButtonLabel:0x104b19cd0'\u200e\u540c\u610f\u5e76\u7ee7\u7eed'
|   |   |   |   |   |   *UIView:0x106923290
|   |   |   |   |   |   *UIView:0x106926ee0
|   |   |   |   |   |   *UIView:0x106904f20
|   |   |   |   |   |   *UIView:0x1069299f0

Legend:
\t* - is laid out with auto layout
\t+ - is laid out manually, but is represented in the layout engine because translatesAutoresizingMaskIntoConstraints = YES
\t\u2022 - layout engine host`
```

更多用法后续用到再说





