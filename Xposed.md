# Xposed

> **Xposed**（也被称作**Xposed框架**、**XP框架**、**Xposed framework**），是一个运行于[Android](https://zh.wikipedia.org/wiki/Android)操作系统的[钩子](https://zh.wikipedia.org/wiki/钩子编程)[框架](https://zh.wikipedia.org/wiki/軟體框架)。其通过替换Android系统的关键文件，可以拦截几乎所有[Java](https://zh.wikipedia.org/wiki/Java)[函数](https://zh.wikipedia.org/wiki/子程序#函数)的调用，并允许通过Xposed模块中的自定义代码更改调用这些函数时的行为。[[7\]](https://zh.wikipedia.org/zh-hans/Xposed_(框架)#cite_note-7)因此，Xposed常被用来修改Android系统和应用程序的功能。
>
> 由于Xposed框架的开发已不再活跃，且不支持[Android Pie](https://zh.wikipedia.org/wiki/Android_Pie)，有第三方开发者对其进行了[移植](https://zh.wikipedia.org/wiki/移植_(軟體))。[[12\]](https://zh.wikipedia.org/zh-hans/Xposed_(框架)#cite_note-12)
>
> 目前活跃的项目有：
>
> - LSPosed

Xposed的原作者是**rovo89** github链接：https://github.com/rovo89。Xposed早在2018年就已经停更，现在Xposed的fork项目LSPosed仍活跃更新，github上获得13k star，超过了原项目。

Xposed和LSPosed都需要root，但是有些设备获取root权限比较困难，所以LSPosed团队推出了LSPatch，通过对需要修改的apk进行patch，就可以达到免root使用xposed的目的。

## 安装

### LSPosed

LSPosed需要把设备root，root的方法自己去网上查，root完之后打开LSPosed的github<img src="https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104211438409.png" alt="image-20231104211438409" style="zoom:33%;" />

下载红色圈起来的![image-20231104211721179](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104211721179.png)

在magisk的设置中，勾上zygisk![image-20231104211806785](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104211806785.png)

然后到模块界面，安装刚刚下载好的模块，安装，然后重启。

重启完之后就可以在通知栏打开LSPosed的管理器了![image-20231104211926038](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104211926038.png)

### LSPatch

同样，在github找到LSPatch的仓库，在release中下载最新的manager.apk，然后打开。![image-20231104212206480](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104212206480.png)

选择需要hook的应用，![image-20231104212316225](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104212316225.png)

可以看到有两种修补模式，一般选择本地模式即可。

便捷模式会把模块内嵌进apk内，并且不能再用LSPatch管理器 管理，适合把模块分享给别人的情况下。

这两种方式都会输出一个apk，选择好然后安装就能发挥对应的作用。

## 配置开发环境

Android项目，一般情况下都是用Java或者kotlin写的，所有开发xposed模块肯定要会Java语言。如果你没有学过Java，但是只要学过任何一门编程语言，再来学Java都是很容易的，建议买一本书来看。![image-20231104222257281](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104222257281.png)







打开idea，clone这个项目 https://github.com/114514ns/XposedTemplate，这是我自己做的xposed项目模板。

打开后，应该会看到如下的结构。![image-20231104221537321](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104221537321.png)

在main/assets下有一个xposed_init的文件，这个文件需要写每一个hook入口类的全限定类名，比如我这个类，就需要写**cn.pprocket.xposed.MainHook**

一个Xposed项目可以有多个入口类，如果有多个入口类，就一行写一个类名，比如这样![image-20231104222831651](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104222831651.png)

一般如果我们的模块适用于多个应用，那么每个应用都单独写一个入口类，例如

![image-20231104223351656](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104223351656.png)

![image-20231104223404076](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104223404076.png)

## API

Xposed最常用的api只有一个，就是XposedHelpers.findAndHookMethod

![image-20231104223829987](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231104223829987.png)

这个函数有5个参数，从名字可以看得出来，第一个参数写的是类名，第二个是classloader，第三个是方法名（在Java中，一般把函数称为方法），第四个是要hook的函数的参数的Class对象,最后一个是hook回调。回调 就是要hook的方法被执行后，xposed内部会自动调用你编写的回调方法。

XC_MethodHook是一个抽象类，这里我们直接new一个出来。然后idea会自动补全XC_MethodHook的两个方法：afterHookedMethod和beforeHookedMethod，从名字就可以看出来，在目标方法执行之前xposed会调用beforeHookedMethod，执行之后会调用afterHookedMethod。

下面看一个实例。

```java
        ClassLoader classLoader = param.classLoader;
		findAndHookMethod("com.aiqiyi.youtube.play.bean.response.VideoDetailBean", classLoader, "getIs_vip", new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                param.setResult("n");
            }
        });
```

第一行是获取目标应用的classloader，如果没有特殊情况这样写就行了。再看findAndHookMethod，我们怎么知道要hook的应用的方法名和参数呢？

这时要使用Jadx，这是一款apk反编译工具，github链接：https://github.com/skylot/jadx/releases/tag/v1.4.7

idea和jadx都会占用大量的内存，所以电脑的内存至少需要有16g，最好有32g

![image-20231105111221472](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231105111221472.png)

下载完打开![image-20231105103226300](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231105103226300.png)

打开目标apk，找到要hook的方法，右键点击**复制xposed片段**![image-20231105103432305](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231105103432305.png)

然后粘贴进idea，现在我们的代码应该像这样

```java
public class MainHook implements IXposedHookLoadPackage{

    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam param) throws Throwable {
        ClassLoader classLoader = param.classLoader;
        XposedHelpers.findAndHookMethod("com.spaceseven.qidu.bean.ConfigBean", classLoader, "getDomain_name", new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                super.beforeHookedMethod(param);
            }
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                super.afterHookedMethod(param);
            }
        });
    }
}
```

再来看MethodHookParam，它有这几个方法

![image-20231105104144898](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231105104144898.png)

- param.args 它返回一个数组，里面的内容就是要hook的方法的参数，可以beforeHookedMethod内对其进行修改，但是不要在afterHookedMethod内修改它，因为要hook的方法已经执行完了，修改它没有任何效果。

- param.getResult() 顾名思义，获取方法的返回值。不要在beforeHookedMethod里调用，否则会返回null，原因也很好理解，目标方法还没有开始执行，当然没有返回值。

- param.setResult() 修改方法的返回值。

- param.thisObject获取被hook的方法所在的类的实例，比如要hook ConfigBean.getDomain_name() ,thisObject就会返回一个ConfigBean的实例。

  

  xposed的api大致就是这样，是不是很简单？

  下面我们来实战一下，看看xposed的威力。

## 尝试

我们以bilibili的国际版为例，下载mt管理器，提取安装包，![image-20231105112056558](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231105112056558.png)

bilibili使用了分包，我们选择第一个包，其他的包都是素材资源或者native lib，只有第一个才有代码。发送给电脑，然后用jadx打开。

打开后可以看到这个apk被混淆过了![image-20231105113005739](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231105113005739.png)

但是没有完全混淆，找到bilibili的包名，还是可以看到可读的类名的![image-20231105113040868](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20231105113040868.png)

# 实战

### 爱酱

打开目标应用，可以看到，它检测了root![image-20240113170301327](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113170301327.png)

，这里直接打开jadx，搜索字符串，![image-20240113170340799](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113170340799.png)

![image-20240113170438274](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113170438274.png)

可以看到，如果j函数的返回值为true，就会弹窗root的提示，所以j应该就是判断root的。

所以直接hook它，修改返回值为true即可

![image-20240113170547445](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113170547445.png)

确实可以打开，说明前面的猜测是正确的。![image-20240113170659430](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113170659430.png)

随便打开一个视频，然后开启httpcanary，会发现和服务端之间的通信是加密的，应该是AES加密。![image-20240113170858185](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113170858185.png)

这里可以使用SImpleHook的扩展函数功能![image-20240113170932057](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113170932057.png)

选中爱酱，然后勾上AES加密算法，然后重启爱酱再打开一个视频，再回到simplehook，就可以看到刚刚的解密结果![image-20240113171100990](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113171100990.png)

![image-20240113171316475](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113171316475.png)

这条结果应该是视频的信息，格式化看一下，![image-20240113171339035](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113171339035.png)

视频的信息都在里面了，包括需要的钻石，是否免费，标题，点赞数等等。

咱们当然不能直接修改这些信息，就算修改了，也只是在本地生效，只有自己能看见，服务端并不认为这条视频是免费的。

再看看服务端返回的视频链接，它是m3u8格式。

> **M3U**（[MP3](https://zh.wikipedia.org/wiki/MP3) [URL](https://zh.wikipedia.org/wiki/统一资源定位符)的缩写）是一种播放多媒体列表的[文件格式](https://zh.wikipedia.org/wiki/檔案格式)，它的设计初衷是为了播放音频文件，比如[MP3](https://zh.wikipedia.org/wiki/MP3)，但是越来越多的软件现在用来播放视频文件列表，M3U也可以指定在线流媒体音频源。很多播放器和软件都支持M3U文件格式。
>
> M3U文件是一种[纯文本文件](https://zh.wikipedia.org/wiki/文本文件)，可以指定一个或多个多媒体文件的位置，其[文件扩展名](https://zh.wikipedia.org/wiki/文件扩展名)是“M3U”或者“m3u”。
>
> M3U文件具有多个条目，每个条目的格式可以是以下几种格式之一：
>
> - 一个*绝对路径*；比如：C:\My Music\Heavysets.mp3
> - 一个*相对路径*（相对于M3U文件的路径）；比如：Heavysets.mp3
> - 一个[URL](https://zh.wikipedia.org/wiki/URL)
>
> M3U文件也有[注释](https://zh.wikipedia.org/wiki/注释)，注释行以"#"字符开头，在**扩展M3U**文件中，"#"还引入了扩展M3U指令。
>
> M3U文件的作用通常是创建指向在线[流媒体](https://zh.wikipedia.org/wiki/流媒体)的播放列表，创建的文件可以轻松访问流媒体。M3U文件通常作为网站的下载资源、通过email收发，并可以收听[网络电台](https://zh.wikipedia.org/wiki/网络电台)。
>
> 如果使用编辑器编辑M3U文件，必须将该文件用[Windows-1252](https://zh.wikipedia.org/wiki/Windows-1252)格式保存，这种格式是[ASCII](https://zh.wikipedia.org/wiki/ASCII)编码的超集。M3U文件也可以使用[Latin-1](https://zh.wikipedia.org/wiki/Latin-1)[字符编码](https://zh.wikipedia.org/wiki/字符编码)。
>
> 

简单地说，就是把视频分割成很多片，然后存放在一个目录里。然后m3u8文件记录每一个分片所在的位置。客户端再根据m3u8文件的内容去获取每一个分片然后播放。

![image-20240113171956480](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113171956480.png)

服务端返回的m3u8文件，里面只有一个分片。![image-20240113172039538](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113172039538.png)

咱分析一下这条链接。后面跟了一个sign参数，看起来像签名。

试试把sign参数去掉，发现仍可以正常访问。

![image-20240113172205401](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113172205401.png)

再看前面一串像hash的东西。

![image-20240113172322214](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113172322214.png)

长度是18位，一般md5是16位或者32位。把后面的33去掉，就是16位。

![image-20240113172418289](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113172418289.png)

把33改成32，可以正常访问。由此可以猜测，这一串字符串，是16位的md5再加上视频分片的序号。

![image-20240113172546218](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113172546218.png)



所以咱可以根据一条分片，自己还原出一个m3u8。

至于最大的序号是多少，我们可以根据视频的长度算出来。比如，一个分片的长度是10秒，而一个完整视频的长度是10分钟，所以分片的序号就到60为止



需要自己写一个服务端，根据客户端的数据，生成m3u8，然后返回给客户端。

![image-20240113172708350](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113172708350.png)

客户端再根据服务端的返回，修改视频的链接。

![image-20240113173006430](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//imgimage-20240113173006430.png)

