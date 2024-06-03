[TOC]



### 1.TextMeshPro创建中文字体

TextMeshPro**<font color=blue>自带的字体不支持中文显示</font>**，因此需要**<font color=blue>单独制作“中文字体”</font>**

当使用UGUI中的Text组件时会自动选择“TextMeshPro”版本的Text组件(若需要使用普通的Text组件则需要在“Legacy”中选择)，此时会自动在项目中导入“TextMeshPro”相关的资源(也可以在“PackageManager”中导入该插件)

**PS**：TextMeshPro具有2D和3D的UI，如果只是从UGUI中创建则默认为2D，3D则需要从“Hierarchy”中鼠标右键“3D对象->文本对象(TextMeshPro)”中创建

#### 1.打开字体生成窗口：

Window -> TextMeshPro -> FontAssetCreator

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230316155123427.png" alt="image-20230316155123427" style="zoom:67%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230316155320078.png" alt="image-20230316155320078" style="zoom:67%;" />

**参数解析**：

1).**Source Font File**：制作字体使用的源文件，该“字体文件”需要自身支持中文

TextMeshPro默认的字体不支持中文显示，因此需要从外部获取到可以显示中文的字体。使用“Windows”操作系统的中文字体，路径“C:\Windows\Fonts”

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230316155636309.png" alt="image-20230316155636309" style="zoom:67%;" />

从这些字体中选中一个，鼠标右键“复制”，然后在Unity项目中“粘贴”即可(或者直接选中目标字体，将其拖到Unity即可正常导入)

**PS**：字体文件的<font color=red>**后缀需要为“*.ttf”**</font>，并且为“**<font color=red>TrueType</font>**”类型

2).**Atlas Resolution**：最后生成的“新字体”的图集分辨率。

该分辨率的选择需要根据实际“**<font color=red>目标字符”的数量</font>**来设置。如为“5000个中文常用字符”生成新字体，则设置分辨率为**“2048 x 2048**”；如为“常用的中文标点符号”生成字体，由于“目标字符数量”较少，因此直接选择“**128 x 128**”即可

**注意**：该分辨率的选择会直接影响生成“新字体”的速度以及最后“新字体”文件的大小。当选择“4096 x 4096”分辨率时，生成“新字体”的时间会明显延长，并且最终得到的“新字体”文件很大。这样不利于日常开发

**3).Character Set/File**：目标字符的文件

新建“*.txt”文件，粘贴“5000个中文常用汉字”，并设置“编码格式”为“UTF-8”(避免乱码)

后续“中文常用标点符号”，“中文简体、繁体”均使用相同步骤生成“新字体”



#### 2.生成新字体：

以上参数设置完毕后，点击“Generate Font Atlas”即可生成新字体

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230316162253737.png" alt="image-20230316162253737" style="zoom:67%;" />

**结果解析**：

1 —— 代表本次新字体生成的结果：“4611”代表该新字体中包含“目标字符文件Character File”中的“4611”个中文字符，“4846”代表“目标字符文件Character File”一共包含的中文字符数量。

2 —— 代表上述“SourceFontFile”中缺失的字符，而这些字符在“目标字符文件Character File”中包含。所以基于此原因，在选择“SourceFontFile”时尽量选择通用的中文字体，如“Windows”操作系统中的中文字体即可。

3 —— 代表“新字体”图集。也可根据该图集空余区域的大小设置“Atlas Resolution”参数(右上角黑色区域即为图集中剩余的区域)

**PS**：从“Windows”中选择一款常用字体设置为“SourceFontFile”后生成结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230316163916002.png" alt="image-20230316163916002" style="zoom:67%;" />

如上成功为所有“目标字符”生成“新字体”，并且该字体的图集中“剩余区域”很小，说明“Atlas Resolution”数值设置合理。



#### 3.保存新字体：

“新字体”生成结束后，点击“Save”即可将其保存下来



使用上述步骤分别为“5000常用中文**<font color=blue>简体/繁体</font>**汉字”，“常用**<font color=blue>中文标点</font>**符号”分别生成字体：
![image-20230316164821919](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230316164821919.png)

**注意**：只有上图中“F”标的字体才可被“TextMeshPro”使用，“Aa”图标的字体无法直接使用(**<font color=red>上述用到的“5000个常用中文简体/繁体汉字”的文件在“资源文件夹”中</font>**)



#### 4.为字体添加“Fallback”备用字体：

部分情况下，已经生成的字体中依然会缺少部分字符导致TextMeshPro上无法显示。但是如果将“缺少的字符”加入“目标字符文件”中，然后重新生成“新字体”。这过程又有些麻烦，并且那些“缺少的字符”大多时候不会用到，如果“重新生成新字体”其实不大划算。

因此**增加“Fallback”备用字体**。当在原字体上找不到该字符时，则去“Fallback备用字体”中查找。

利用如上生成的“中文繁体”，“中文标点符号”，将其作为“中文简体”的“Fallback备用字体”：
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230316165611605.png" alt="image-20230316165611605" style="zoom:67%;" />

在实际使用中，只需要将“中文简体”赋值给“TextMeshPro”组件即可正常使用。



### 2.TextMeshPro支持的多种富文本标记

#### 常用Tag：

```c#
粗体：<b>Bold</b>
斜体：<i>Italics</i>
下划线：<u>Underline</u>
删除线：<s>Strikethrough</s>
大写字母：<uppercase>hello</uppercase>    -- 输出结果：HELLO
小写字母：<lowercase>Today</lowercase>    -- 输出结果：today

设定字体大小：<size=48>Point size 48</size>
设置字体相对大小：<size=+18>Point size increased by 18</size>
设置字体相对大小：<size=-18>Point size decreased by 18</size>

设置颜色：
方式一：<color=yellow>Yellow text</color>
方式二：<#00ff00>Green text</color>

渐变色：<gradient="渐变资源的名称">hello</gradient>
自定义材质：<material="材质资源的名称">hello</material>
注意：TextMeshPro中各个类型资源存放位置都在“TMP Settings”文件中

超链接：<link="www.baidu.com">Click this</link>    -- 需要配合“点击的监听”才能有效

笔记标注(突出显示)：<mark=#ffff0040>Highlighting</mark>   -- 颜色值需要包含Alpha
```

**所有Tag详情**：https://docs.unity3d.com/Packages/com.unity.textmeshpro@4.0/manual/RichTextMark.html



#### 渐变色：

**基础规则**：当需要自定义创建某个“渐变色gradient”、material、font等资源文件时，其需要放置在指定的文件夹中。在文本中使用相应的tag时则会去相对应的文件夹中查找对应的gradient等文件

各类型资源的路径：

方式一：ProjectSetting -> TextMeshPro -> Setting

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230809160302639.png" alt="image-20230809160302639" style="zoom:80%;" />

方式二：在导入TextMeshPro插件到项目中后，在其Resources下有“TMP Settings”文件

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230809160010016.png" alt="image-20230809160010016" style="zoom:80%;" />

该文件中标注了各个资源存放的位置，各个参数的意义：https://docs.unity3d.com/Packages/com.unity.textmeshpro@4.0/manual/Settings.html

**实现过程**：

1.根据“TMP Settings”中对Gradient资源的设置创建“Color Gradient Presets”文件夹，鼠标右键“Create -> TextMeshPro -> Color gradient”创建“渐变色文件 CustomGradient”:

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230809160406396.png" alt="image-20230809160406396" style="zoom:80%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230809160655478.png" alt="image-20230809160655478" style="zoom:80%;" />

2.在文本中使用标签“gradient”即可

```c#
<gradient="CustomGradient">hello</gradient>
```

运行效果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230809160830414.png" alt="image-20230809160830414" style="zoom:80%;" />



#### 超链接：

**实现过程**：

1.在文本中添加“link”标签：

```c#
<link="https://www.baidu.com">Click this</link>
```

2.创建脚本挂载在本TMP_Text组件的同一GameObject上：

```c#
using TMPro;
using UnityEngine;
using UnityEngine.EventSystems;

[RequireComponent(typeof(TMP_Text))]
public class TextMeshProResearch : MonoBehaviour, IPointerClickHandler
{
    //当点击文本上的超链接时
    public void OnPointerClick(PointerEventData eventData)
    {
        TMP_Text tmpText = GetComponent<TMP_Text>();
        //第三个参数camera需要使用监听当前“点击事件”的camera
        //通常为“Canvas渲染模式”中配置的负责UI模块渲染的Camera
        int linkIndex = TMP_TextUtilities.FindIntersectingLink(tmpText, eventData.position, eventData.enterEventCamera);
        if (linkIndex != -1)    //说明点击了超链接
        {
            //linkIndex编号代表当前点击的是TMP_Text文本中的第几个超链接，
            //数值为“-1”代表“没有点击链接”
            TMP_LinkInfo linkInfo = tmpText.textInfo.linkInfo[linkIndex];
            Application.OpenURL(linkInfo.GetLinkID());
        }
    }
}
```

**两者配合使用，才能在点击文本中超链接时自动跳转**

**PS**：“Application.OpenURL”方法传入的**<font color=red>链接参数需要包含“http://, https://, ftp://”等前缀</font>**。当直接使用“www.baidu.com”时，其在UnityEditor中可以正常打开，但在安卓真机上无法响应





**参考链接**：

大神关于“TMP图文混排以及优化”相关的内容：https://www.lfzxb.top/unity-textmeshpro-something/#%E5%89%8D%E8%A8%80





