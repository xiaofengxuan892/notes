[TOC]



#### Unity常用插件的安装：

安装方式有很多种，如通过“PackageManager窗口”或在“manifest.json”中填写依赖或直接在Unity中导入“*.unitypackage”插件。但**<font color=blue>考虑到后期的管理——移除或升级该插件</font>**，因此这里推荐使用**官方的“PackageManager窗口”**来安装各个插件(主要针对**<font color=red>开源插件</font>**)

**安装方式**：Window -》Package Manager -》点击左上角选择“Add package from git URL”

##### 开源插件列表：

==Luban打表==：https://github.com/focus-creative-games/luban_unity.git

==HybridCLR热更==：

==NuGetForUnity==：https://github.com/GlitchEnzo/NuGetForUnity.git?path=/src/NuGetForUnity

- **该插件使用时需要注意的细节**：

  **1)**.安装该插件后会自动在“Assets文件夹”下创建“NuGet.config”和“package.config”文件，以及“Packages文件夹”用于存放从“NuGet窗口”中安装的各个dll库。这样会干扰项目原本的文件夹结构，不利于管理。为了避免这种情况，可在安装完成后点击菜单栏“NuGet -> Preferences”打开如下界面：

  <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240416194202082.png" alt="image-20240416194202082" style="zoom:80%;" />

  **<font color=red>将“Placement”设置为“In Packages Folder”</font>**，此时所有NuGet相关的内容都会自动转移到“Assets/Packages”目录下(包含后续通过NuGet安装的各个dll库)，不影响原有的项目开发结构，整体比较美观简洁

  **2)**.**在“In Packages Folder”模式下**，会**<font color=red>自动在“Windows/PackageManager”中安装一个新的插件“NuGetPackages”，该插件不可删除</font>**(显示为**<font color=blue>“Custom”插件</font>**，用于存储后续下载的各个Dll库)

  <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240416203241410.png" alt="image-20240416203241410" style="zoom:80%;" />

  且该模式下，“NuGetForUnity”插件被隐藏，需要打开“Project视图”右侧的自动隐藏图标才可看到<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240417114334634.png" alt="image-20240417114334634" style="zoom:80%;" />

  3).由于安装的**<font color=red>各个Dll库会根据项目当前使用的“.NET版本”来安装对应的Dll版本</font>**，所以**在更换了PlayerSetting中的“Api Compatibility Level”，如“.NET Framework”和“.NET Standard 2.1”后**，**<font color=red>还需要卸载之前的Dll库后重新安装</font>**

  <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240417115125408.png" alt="image-20240417115125408" style="zoom:80%;" />

  **4)**.如果两个NuGet package都引用某个package，但引用的版本不同，则可能出现如下报错：

  <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240418121344562.png" alt="image-20240418121344562" style="zoom:80%;" />

  此时关闭Player Setting中的对应设置即可：

  <img src="../../../AppData/Roaming/Typora/typora-user-images/image-20240418121535259.png" alt="image-20240418121535259" style="zoom:80%;" />







部分特殊开源插件

以下开源插件无法通过“Git URL”添加，也没有“*.unitypackage”文件。

SimpleJson：https://github.com/Bunny83/SimpleJSON







#### 实用工具：

##### PDF阅读器

多功能的PDF工具箱，完全满足平常查看PDF文件的需求。https://github.com/wmjordan/PDFPatcher

##### ShareX

Windows截屏录制工具，支持“**滚动截图**、Gif录制”等，并且截图后还支持在图片上增加文字、水印、马赛克等

https://github.com/ShareX/ShareX

##### Translumo

免费开源的屏幕实时翻译工具，可以对屏幕上选定区域的文本进行实时翻译，包含英语、俄语、中文等语言，并且用户可自由选择识别的OCR引擎。https://github.com/Danily07/Translumo

##### DevToys

包含程序员常用的一些小工具，如时间戳转换、各种解码等。是个独立的软件，可在Windows中使用。https://github.com/DevToys-app/DevToys

##### ToastFish

利用Windows通知栏背单词的软件，可以自由选择单词集合，并在背完后进行检测。https://github.com/Uahh/ToastFish

##### AssetStudio

用于查看AB中包含的资源，以及将其中的资源导出的工具，可以很方便的获取任何AB中的资源文件

地址：https://github.com/Perfare/AssetStudio，选择相应“.NetFramework”版本的文件即可

##### OFGB

关闭Win11系统广告的工具：https://github.com/xM4ddy/OFGB





#### 编程助手：

##### Masuit.Tools

C#开发的工具箱，包含日常C#开发中用到的各个操作类：字符串处理、进制转换、日期处理、加密/解密、文件压缩、图像裁剪、**断点续传**、分布式ID等，整体代码量不足2M，非常的方便，强烈建议收藏并使用：https://github.com/ldqk/Masuit.Tools

##### ToolGood.Words

一款**高性能的非法词、敏感词检测库**。还支持简繁体呼唤、获取拼音字母、拼音模糊搜索等功能。https://github.com/toolgood/ToolGood.Words

**PS**：在C#项目中使用ToolGood.Words插件的方法：https://www.duidaima.com/Group/Topic/ASP.NET/10295

##### Common.Utility

C#项目中用到的各式各样的"xxHelper"，并且各个“xxHelper”之间没有任何关系，如果需要某个“xxHelper”则直接将其**<font color=red>对应的脚本复制粘贴到自己的项目</font>**中即可。https://github.com/laochiangx/Common.Utility

**PS**：详细使用指南：https://cloud.tencent.com/developer/article/1028316

##### Newtonsoft.Json

在Unity中使用Json进行数据处理，通常需要使用“ListJson”或“Newtonsoft.Json”，后者在使用中更为方便。https://github.com/JamesNK/Newtonsoft.Json





#### Unity项目：

##### ET

基于C#的双端开发框架，前端也可以无缝开发服务器功能，功能非常强大。目前已经有多个基于ET框架且仅一人开发的MMO游戏上线。强烈建议早点研究：https://github.com/egametang/ET

##### HybridCLR

划时代的C#热更方案，focus creative games出品。之前已经研究过，后续需要整理下接入的demo(之前的demo项目丢失了)，并详细的记录笔记

##### FancyScrollView

Unity滑动列表插件，该项目采用Unity动画系统来定制列表滑动效果，灵活性极高。项目代码规范，接入成本低，易于使用和定制。https://github.com/setchi/FancyScrollView

##### NKGMobaBase：

moba游戏技能系统，包含非常详尽的战斗框架，如“Buff系统”、“技能系统”、“状态系统”、“数值系统”等，开源项目地址：https://github.com/wqaetly/NKGMobaBasedOnET

PS：该项目作者为“烟雨大佬”，其有对应的教程博客：https://www.lfzxb.top/nkgmoba-totaltabs/

##### EGamePlay

基于Unity的战斗/技能框架，配置可选择ScriptableObject或Excel表格。https://github.com/m969/EGamePlay

##### CrazyCar

使用Unity制作的联机赛车游戏，包含客户端、服务器端、网络传输、管理后台。https://github.com/TastSong/CrazyCar

##### QFramework

“凉鞋”老师的框架，简单易用，地址：https://github.com/liangxiegame/QFramework

##### 资源管理系统

###### YooAsset

完善的资源管理方案，可单独导入项目作为“资源管理工具”，包含完善的“资源打包、加载、卸载、下载，资源依赖分析、冗余资源检测”等功能。https://github.com/tuyoogame/YooAsset

###### XAsset：

有免费版和收费版，资源管理工具套件

##### 其他项目：

###### ET Framework

整合“ET 7.0 + **<font color=blue>FGUI + Luban + huatuo + YooAsset + NKGMoba + UniTask</font>**”，包含**<font color=red>完整的“NKGMoba技能系统”</font>**，并提供常用的编辑器工具。https://github.com/wqaetly/ET/tree/et7_fgui_yooasset_luban_huatuo







#### Unity插件：

**<font color=red>OpenUPM</font>**：

专用于Unity插件安装的平台，相比于Unity官方的“Package Manager”，其免费开源，有很多Github上的高分项目在这里都可以找到  https://openupm.cn/packages/

##### NugetForUnity

在Unity中直接安装某些dll，而不用通过“Rider或VS”安装后重新提取dll到Unity中。**<font color=red>对开发有极大的助力，强烈建议安装</font>**，在OpenUPM上可以直接找到，GitHub上也可下载：https://github.com/GlitchEnzo/NuGetForUnity

##### ZString

针对string、StringBuilder的内存优化而封装的新类型“ZString”，效果非常明显 https://github.com/Cysharp/ZString，可通过OpenUPM或Github安装

##### NaughtyAttributes

扩展UnityEditor，可以更方便的实现一些功能。相比Odin，可以更易接入  https://openupm.cn/packages/com.dbrizov.naughtyattributes/

##### MessagePack-CSharp

MessagePack是一种新的序列化格式，其相对于Json、Protobuf-net性能和效率都更高。https://github.com/MessagePack-CSharp/MessagePack-CSharp

**PS**：**<font color=red>MessagePack序列化格式已适配java/c/c++/Go等50多种语言</font>**，使用非常广泛，官网地址：https://msgpack.org/，强烈建议研究下并引入项目中



##### Asset Usage Detector

检测项目中任意资源的引用(选中该资源后鼠标右键选择"Search for references"即可)，可以用于清理未使用的资源，非常方便。可以使用AssetStore导入，也可以直接下载"xx.unitypackage"。https://github.com/yasirkula/UnityAssetUsageDetector

##### UnityAssetCleaner：

用于删除项目中无用资源的插件：https://github.com/tsubaki/UnityAssetCleaner

##### Notch Solution

用于IOS及安卓设备异形屏的适配，效果很好。https://github.com/5argon/NotchSolution。也可使用AssetStore导入

##### AVProVideo

该插件专用于Unity中所有的视频播放，功能丰富，比Unity自带的视频播放插件好非常多。据说视频类游戏，如“被美女包围了”等游戏都是使用该插件来管理游戏中的视频播放。AssetStore有免费版和付费版，github上也可下载 https://github.com/RenderHeads/UnityPlugin-AVProVideo

##### Particle Effect for UGUI

专用于处理UI与粒子特效之间遮挡关系的插件，让粒子特效可以如同普通UI一样完美使用，功能非常强大，并且使用非常简单  https://github.com/mob-sakai/ParticleEffectForUGUI

##### UIEffect

为UI制作不同效果的插件，可以使得UI展示更加好看：https://github.com/mob-sakai/UIEffect

##### SoftMask

专门用来处理UGUI中“Mask”遮罩消耗过高的问题，可以完全替代其作用且性能更好  https://github.com/mob-sakai/SoftMaskForUGUI

##### SafeAreaHelper

用来适配Android或IOS中各种异形屏的免费插件，功能强大效果好。AssetStore可下载：https://assetstore.unity.com/packages/tools/gui/safe-area-helper-130488

##### NotchSolution

用于Android或IOS异形屏适配的插件，免费且开源，GitHub地址：https://github.com/5argon/NotchSolution

该插件有完整的运营官网，使用广泛且有维护：https://exceed7.com/notch-solution/

PS：可在“manifest.json”文件中添加"com.e7.notch-solution": "git://github.com/5argon/NotchSolution.git"来导入该插件

##### Flux

制作技能序列效果时使用的插件，具体没用过。该插件有官网地址：http://www.fluxeditor.com/， AssetStore也可以下载

























