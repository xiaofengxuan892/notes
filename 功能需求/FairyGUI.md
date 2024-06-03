[TOC]





待解决问题：

1.为何调整外层UI位置Pos会导致内层文本的“底延展”自动拉伸底部？？

2.当修改UI元件的Pivot为中心点后，该UI元件的Position代表的是“左上角”还是“中心点”的坐标？

解答：经过实际测试，在FGUI中修改Pivot并不会改变UI元件的Position的统计原则。在FGUI中，UI元件的坐标始终代表“左上角”的坐标，因此UI元件的中心点的坐标则为“GObject.x  + GObject.width * 0.5f”

3.包A中的元件A1，用到了包B中的元件B1，那么在加载A1前需要先使用“UIPackage.AddPackage”将包B加载到内存中，否则元件A1显示不正常，且需要在FGUI编辑器中设置元件B1为“导出”状态

由于FGUI不处理“包之间的依赖关系”，所以需要手动实现？？

4.Ggraph.SetNativeObject如何设置原生对象？？  类型“FairyGUI.DisplayObject”

6.组件的“自定义属性”感觉可以有非常强大的丰富效果，可以研究下

7.动态创建的GComponent默认开启“点击穿透”，这个具体的用法是怎样的

8.如何获取元件中的控制器，以及如何使用controller实现“切页”效果？

9.TMP字体在代码中的注册有问题，需要研究下！！！！

10.如何在代码中获取动效时间轴中指定“Label”？

11."隐私协议"中无法滑动到底部，但是“仅调整com_text组件”的“边缘”中的“下”数值后即可解决该问题，**<font color=red>这个要研究下原理，应该会很常用的</font>**！！！！！！！！

PS：该特性是调整“组件”本身的特性的：在组件中的内容超出其原本宽高时则会自动开启“溢出处理”的功能

![image-20240131140625985](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240131140625985.png)

12.针对FGUI的异形屏、挖孔屏的适配，可通过获取“屏幕的安全区域”来设置，如“Screen.safeArea”；并且在显示UI时，需要先设置该UI的pivot为“(0.5，0.5)”，这样在调用“SetSize(safeArea.x, safeArea.y)”时就可以以“中心点”开始绘制该UI的显示区域 —— 这个要具体验证下，目前只是理论上大体可以走通，另外还需要确定是否在每个UI打开时都要这样处理，还是说只需要设置“GRoot.inst”的“safeArea”就可以了，这个需要确定下

13.在设置FGUI的发布路径时，可以选择“相对路径”，如：

```
..\..\..\dcbubbleb_UClient\Assets\_Res\AssetBundles\UI
```

如何确定前面的“右划线”的数量？**<font color=red>感觉上这种“确定相对路径”的方式应该在很多地方都可以用到，所以最好研究下，之后会很方便的</font>**





#### 使用细节：

##### 重要特性：

1.FGUI中任何元件或组件的“显示”和“隐藏”是独立的，如果某个元件或组件不会再被使用，则需要调用“Dispose”方法销毁，否则会产生内存泄漏

2.**<font color=red>“关联系统”只对元件的“宽高”有效，对“Scale”没有作用</font>**。因此当直接设置元件的“Scale”后，并不会触发其他元件的“关联系统”

3.定义了遮罩的组件，其内部的元件无法与外部的元件合并Drawcall，因为其采用了不同的材质

4.当需要将某个Texture2D对象赋值给FGUI中某个GLoader或GImage时，则可使用“new NTexture()”

```c#
//必须注意GLoader不管理外部对象的生命周期，不会主动销毁your_Texture2D
aLoader.texture = new NTexture(your_Texture2D);
```

5.当动态改变FGUI界面中的某个元件的位置，如某个图片A原本在图片B的上方，在使用Tween动画移动位置后可能会被图片B或其他元件遮挡，此时需要调用该图片A的“**InvalidateBatchingState()**”方法手动触发深度调整：

```c#
aObject.InvalidateBatchingState();
```









#### FairyGUI编辑器：

##### 基本属性：

1.**倾斜**：针对基础元件，如图片、图形、动画、装载器，可以使用“倾斜”参数实现自定义效果，且不会产生额外消耗。但对于“组件”或其他较复杂的元件则尽量不要使用

##### 组件GComponent：

1.当使用`RemoveChild/RemoveChildAt/RemoveChildren`将某个元件从容器中移除时，该元件仅仅只是不再显示出来，但其仍然在内存中占用。因此如果该元件后续还需要使用，则可以将其缓存起来；如果确定不再使用，则需要调用Dispose方法销毁该对象，否则会产生内存泄漏

##### 高级组：

1.当设置为“**<font color=blue>水平布局</font>**”时，则**<font color=red>只有“列距”参数是有效的</font>**，**无论是拉伸整体组的“Width”，还是拉伸单个元件的“Width”，都会自动适配“列距”参数，从而自动伸缩整个“高级组”的“Width”**。而在此“水平布局”下调整“行距”参数是无效的。同理，若当前为“**<font color=blue>垂直布局</font>**”，则**<font color=red>只有“行距”参数是有效的</font>**

##### 关联系统：

1.**<font color=red>关联系统仅对元件的宽高有效，不支持scale的影响</font>**，当设置元件A关联元件B，且“顶->底”时，直接调整元件B的宽高时，元件A可以自适应；但若调整元件B的“缩放scale”，则元件A不会自适应：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227140824925.png" alt="image-20231227140824925" style="zoom: 74%;" />   <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227140910968.png" alt="image-20231227140910968"  />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227141049674.png" alt="image-20231227141049674" style="zoom:80%;" />    <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227141147881.png" alt="image-20231227141147881" style="zoom:80%;" />

2.**关联->左/右延展**：当外层UI宽高改变时，内层的文本框的大小会自动延申。“xx延展”指的是内部UI的某个方向上可以随着外层UI宽高的改变而自动改变大小

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231226103044553.png" alt="image-20231226103044553" style="zoom: 67%;" />

如上图，当外部背景UI高度拉长时，则内部文本UI的高度也会自动延申，因此设置为**“底延展 -> 底”。第一个“底延展”代表内部文本UI的底部，由于需要将其自适应拉伸，因此设置为“底延展”；第二个“底”指代外层UI的底部**。如此的效果就是**<font color=blue>“内部文本UI的底部”与“外部背景UI的底部”距离始终保持一致，且会随着“外层UI底部”的拉长而自动拉长“内部文本UI的底部”</font>**

##### 图形GGraph：

1.**动态创建图形**

```c#
GGraph mGraph2 = new GGraph();
//设置图形的坐标(左上角)
mGraph2.xy = new Vector2(60, 60);
//绘制图形(宽高、线条粗细大小、颜色等)
mGraph2.DrawRect(150, 160, 5, Color.red, Color.blue);
mainCom.AddChild(mGraph2);
```

效果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227145411555.png" alt="image-20231227145411555" style="zoom:80%;" />

##### 自定义数据：

任意元件和组件都包含“自定义数据”，可以在FGUI编辑器中为某些元件和组件设置“自定义数据”。并且这些“自定义数据”可以在代码中获取：

![image-20240122205816220](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240122205816220.png)

```c#
//获取组件的“自定义数据”: aComponent.baseUserData
//获取元件的“自定义数据”：aObject.data
GComponent mainCom = UIPackage.CreateObject("PackageTest01", "Controller&Loader").asCom;
GObject mObject = mainCom.GetChild("n8");
Debug.LogFormat("singleData: {0}, comData: {1}", mObject.data, mainCom.baseUserData);
```

输出结果：

![image-20240122210200544](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240122210200544.png)

##### 组件的“点击穿透”

主要针对组件的“空白区域”：若勾选，则可将“点击事件”传递到下层；否则，“点击事件”被当前元件阻挡，无法传递到下层。该属性也可在代码中设置：

```c#
//“true”表示“不可穿透”，“false”表示“可穿透”
aComponent.opaque = true
```

**注意**：如果在代码中动态创建一个空组件，则该组件默认是“点击可穿透”的，所以如果只是new一个空组件用于接收“点击用途”，则需要使用“`aComponent.opaque = false`”设置其为“不可穿透”





#### Unity编辑器：

1.Sorting Order：在“UIPanel”脚本中，**<font color=blue>“Sorting Order”数值越大越显示在前面，越靠近屏幕</font>**

2.当“UIPanel”中的“Render Mode”为**<font color=blue>“Screen Space XXX”</font>**时，如果需要调整FairyGUI的位置，建议**<font color=red>直接使用“UIPanel”中的“UI Transform”参数</font>**，而不要调整“该UIPanel.gameObject”的“Transform组件”；但如果**<font color=blue>“RenderMode”为“WorldSpace”</font>**，则相反，**<font color=red>直接调整“UIPanel.gameObject”的“Transform组件”</font>**







#### 文本GTextField和字体：

1.**<font color=red>支持直接使用文本模板，不需要通过“string.format()”拼接好完整的string后再赋值给GTextField</font>**。如FGUI中为某个“文本元件”赋值为`{a=oh} today is a good {b=yeah}`，此时FGUI编辑器中显示为：<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227233131846.png" alt="image-20231227233131846" style="zoom:80%;" />

之后可以在代码中重新设置“a”和“b”参数，如：

```c#
GTextField mTextField = mainCom.GetChild("n7").asTextField;
mTextField.SetVar("a", "Hello").SetVar("b", "day").FlushVars();
```

显示结果：<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227150800824.png" alt="image-20231227150800824" style="zoom:80%;" />

**PS**：在代码中动态关闭“文本模板”功能直接使用“GTextField.templateVars = null”；动态开启“文本模板”功能，则直接调用“GTextField.SetVar()”即可

2.描边在文字的所有方向，投影则主要影响文字的某一个方向，且投影和描边通常共用相同的颜色设置

3.**"UBB"常用语法**：

[b]text[/b] —— 粗体，[i]text[/i] —— 斜体，[u]text[/u] —— 下划线，[size=10]text[/size] —— 设置“限定text”文本字体大小，[color=#FFFF00]text[/color] —— 颜色，仅支持“十六进制”颜色代码

**仅支持富文本，不支持普通文本的UBB语法**：

[img]imageUrl[/img]：“imageUrl”代表图片的url

[url=link]click me[/url]：超链接。点击后可以获取link数据，需要自己实现点击监听

**PS**：1.无论是UBB语法还是“文本模板”，当实际需要在文本中显示“[”或“{”时，都需要通过“转义字符\”来实现。如果是在FGUI编辑器中，则使用`\[`或`\{`即可；如果在代码中，则需要使用`\\`才能代表“反斜杠”，因此需要使用`\\[`或`\\{`。只需要转义`[`或`{`，不需要转义`]`或`}`

2.文本模板优先于UBB解析，因此可在UBB语法中嵌套“文本模板”，如：这是[color={colorTemplate=#FF0000}]变色文本[/color]，效果为：<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227234626580.png" alt="image-20231227234626580" style="zoom:80%;" />。如此即可动态修改文本颜色

3.当需要多颜色时，则可使用`[color=#fee9bb,#ffffff]Daily Tasks[/color]`形式

##### FGUI字体在Unity中使用的常用配置：

1.开启字体的加粗和斜体：`FontManager.GetFont("字体名称").customBoldAndItalic = true`

2.当文本内容修改时，FGUI会自动在“LateUpdate”中重新绘制，因此文本赋值操作建议在“Update”中执行，不要放到“LateUpdate”中

###### 注册字体：

FGUI中使用的“TTF”字体，无论是否设置为“导出”都不会被发布。因此如果需要在Unity中使用该字体，则需要**<font color=red>将其放置在“Resources文件夹”或“Resources/Fonts文件夹”下，在这两个文件夹中的字体会被Unity自动识别</font>**。但**如果字体包含在AB中或放置在其他目录，则需要使用“FairyGUI.FontManager.RegisterFont”注册该字体**：

```c#
DynamicFont mFont1 = new DynamicFont();
//该name属性代表“字体的名称”
mFont1.name = "SIMLI";   
//如果字体在AB中则需要使用“AB的加载方法”得到真实的“字体对象”；
//如果在“Resources下指定目录”，则使用如下方式即可
mFont1.nativeFont = Resources.Load<Font>("TestFonts/SIMLI");
FontManager.RegisterFont(mFont1);
```

**PS**：在显示FGUI中某个界面前，通常会先注册该字体，以方便正常显示，否则会使用默认字体显示该文本

###### TextMeshPro支持

TextMeshPro的优势：使用SDF技术渲染文字，文字可以无限放大且保持清晰，并且针对“描边、发光、抗锯齿”等效果都有性能优化，且渲染的文字纹理占用内存很小

**使用方式**：

1.选中TTF字体后鼠标右键打开“属性”面板，设置“渲染方式”为“SDFAA”，即可将普通的TTF字体设置为“支持TextMeshPro显示”的字体

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231228103503184.png" alt="image-20231228103503184" style="zoom:67%;" />

2.当在“文本”文件中选择“TMP字体”后，会出现额外选项：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231228103258924.png" alt="image-20231228103258924" style="zoom:80%;" />

PS：通常建议在增大“粗细”后，“扩张”数值也会同时增大

3.在Unity中增加宏配置：FAIRYGUI_TMPRO，并在代码中注册该字体：

```c#

```

##### 输入文本GTextInput：

输入文本内容改变的监听：GTextInput.onChanged.Add()

输入框焦点获取和焦点丢失的监听：GTextInput.onFocusIn.Add()，GTextInput.onFocusOut.Add()

输入文本内容提交的监听：GTextInput.onSubmit.Add()







#### 列表GList：

1.当列表中某个item被隐藏时，可以勾选“折叠隐藏的项目”选项，此时列表在计算各个item的布局位置时，则会忽略该“隐藏item”；如果不勾选，则该“隐藏item”在列表中显示为“一个空的占位item”。也可直接**<font color=red>在代码中使用“GList.foldInvisibleItems = true”来开启</font>**

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227160744031.png" alt="image-20231227160744031" style="zoom:80%;" />

2.列表的刷新和item的排列在本帧绘制之前，因此**<font color=red>如果需要马上获取“item的正确坐标”，则可以调用“GList.EnsureBoundsCorrect()”方法通知GList立刻重新绘制</font>**。该方法可以重复多次调用，不会有额外性能消耗

3.GLits内部使用了“对象池”，通常在收到服务器数据消息后，先将列表清空，然后重新绘制。添加item时使用GetFromPool、AddItemFromPool，移除item时使用RemoveChildToPool、RemoveChildrenToPool、ReturnToPool。**<font color=blue>注意：放入Pool中的item无需再手动调用“item.Dispose()”销毁，否则会报错</font>**

4.当把所有元素都填充到GList后，可以调用“`GList.ResizeToFit()`”重置GList的大小；

5.监听GList中各个Item的点击：`GList.onClickItem.Add((callback) => {})`，其中**<font color=red>“callback.data”即代表当前点击的item对象</font>**

##### 虚拟列表：

###### 列表刷新：

虚拟列表的数据和渲染分离，**当需要删除或修改某个item时，先修改数据，然后重新绘制列表**。

**触发“虚拟列表”刷新的方式**：1.设置GList.numItems参数    2.调用GList.RefreshVirtualList()方法

**注意**：

- 不要使用“AddChild”或“RemoveChild”增删虚拟列表中的对象。**<font color=red>当需要清空列表时，使用“GList.numItems = 0”</font>**
- **<font color=red>GList.numChildren</font>** 代表列表中当前显示出来的item对象，**<font color=red>GList.numItems</font>** 代表虚拟列表一共需要显示的item总数量

###### item刷新GList.itemRenderer：

用于刷新每个item的处理函数“`GList.itemRenderer”`，其**<font color=red>内部逻辑需要尽量简化，尽量不要包含“协程、异步、IO、高密度计算”等操作，否则会出现卡顿</font>**。且由于在滚动列表的过程中，“itemRenderer”方法调用频次非常高，因此**<font color=red>也不要包含“new实例化”操作，否则会产生大量GC</font>**

**当需要在itemRenderer中添加监听时，需要使用指定形式**：

```c#
void EventCallback()
{
}
EventCallback0 callback = EventCallback;

void OnRenderItem(int index, GObject obj)
{
    GButton btn = obj.asCom.GetChild("btn").asButton;

    //错误！，临时函数会造成添加多次回调。Lua里使用“function() end”类似。
    btn.onClick.Add(()=> { });

    //可以，同一个方法只会添加一次。但直接使用方法名会生成几十B的GC。
    btn.onClick.Add(EventCallback);

    //正确，callback是缓存的代理实例，不会产生GC。
    btn.onClick.Add(callback);

    //正确，使用Set设置可以保证不会重复添加。
    btn.onClick.Set(callback);

    //错误！，不能对ITEM使用onClick.Set，你需要用GList.onClickItem
    obj.onClick.Set(EventCallback);
}
```

###### item的实际索引与“当前展示的索引”之间的转换：

```c#
void OnItemRenderer(int index, GObject item){
    GButton mBtn = item.asButton;
    mBtn.onClick.Set((callback) => {
        //点击item后展示该item“真实的索引”
        int showIndex = loopList.ItemIndexToChildIndex(index);
        Debug.LogFormat(" Click item. itemIndex: {0}, showIndex: {1}", index, showIndex);
    });
}
```

运行效果：当点击如下列表中“第1个显示item”，以及“第3个显示item”时(**绿色**代表自定义的标识，**红色**代表列表中第1个、第3个显示的item)

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227180412327.png" alt="image-20231227180412327" style="zoom:80%;" />

输出结果为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227180538499.png" alt="image-20231227180538499" style="zoom:80%;" />

**注意**：1).索引编号默认都从0开始  2).这里获取的**<font color=blue>“显示索引”对应“Display Object Info”中的“ChildIndex”，而不是Hierarchy视图中的“从上往下的编号顺序”</font>**

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227181346768.png" alt="image-20231227181346768" style="zoom:80%;" />

###### 滑动到指定索引处，并获取该索引的GObject：

```c#
//这里要注意，因为我们要立即访问新滚动位置的对象，所以第二个参数scrollItToView不能为true，即不使用动画效果
aList.ScrollToView(500);

//转换到显示对象索引
int index = aList.ItemIndexToChildIndex(500);

//这就是你要的第500个对象
GObject obj = aList.GetChildAt(index);
```

**PS**：当需要刷新某个item时，需要先检测该索引当前是否有“对应的显示对象”，有则更新，没有则不更新

###### 可变宽高的item处理：

改变item宽高的两种方式：

1.在itemRenderer内部直接修改GObject的“width、height或SetSize”改变item的大小

2.在FGUI编辑器中针对该item建立其对内部某个元件的关联，如垂直列表中“高高关联”，当内部某个文本内容修改时，会自动关联外部item高度发生改变

###### 虚拟列表支持多种item同时使用：

通过GList.itemProvider可设置使用不同的item的URL：

1).在FGUI编辑器中创建三个Component，分别为展示列表的“VirtualListComponent”，以及用到的item：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231228180016006.png" alt="image-20231228180016006" style="zoom:80%;" />  

VirtualListItem1 —— 关联“左-》左，顶-》顶”：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231228180031712.png" alt="image-20231228180031712" style="zoom:67%;" /> 

VirtualListItem2 —— 关联“右-》右，顶-》顶”：

![image-20231228180050104](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231228180050104.png) 

VirtualListItem3 —— 关联“左右居中”：

![image-20231228180105361](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231228180105361.png)

2).在Unity中动态更换列表中使用的item：

```c#
GRoot.inst.SetContentScaleFactor(800, 600);
UIPackage.AddPackage("UI/PackageTest01");
GComponent mainCom = UIPackage.CreateObject("PackageTest01", "VirtualListComponent").asCom;
GRoot.inst.AddChild(mainCom);

GList mList = mainCom.GetChild("n0").asList;
mList.SetVirtual();
mList.itemProvider = index => {
    int num = index % 3;
    switch (num) {
        case 0:
            return UIPackage.GetItemURL("PackageTest01", "VirtualListItem1");
        case 1:
            return UIPackage.GetItemURL("PackageTest01", "VirtualListItem2");
        case 2:
            return UIPackage.GetItemURL("PackageTest01", "VirtualListItem3");
        default:
            return UIPackage.GetItemURL("PackageTest01", "VirtualListItem1");
    }
};
mList.itemRenderer = (index, item) => { };
mList.numItems = 10;
```

运行效果如下：

![VirtualList3](https://gitee.com/kakaix892/image-host/raw/main/Typora/VirtualList3.gif)



###### 循环列表：

“循环列表”是指“item首尾相连”的列表，可通过“`GList.SetVirtualAndLoop()`”开启。

由于“item首尾相连”，因此当需要滑动到指定位置时，通常不使用“item索引” —— GList.ScrollToView(index)，而**<font color=red>通过“GList.scrollPane”的“ScrollLeft/ScrollRight/ScrollUp/ScrollDown”来实现滑动</font>**



##### 虚拟列表在实际使用中遇到的问题：

###### 1.初始打开界面，如果有在宽高上较长的item，则会出现“无法滑动到content真实底端”的情况

**详细描述**：由于虚拟列表支持“宽高可变”的item，因此如果某些item过长，那么在初始打开界面时则会出现“无法滑动到content实际底端”的情况。但是在打开界面后，**<font color=blue>如果需要重新刷新列表并滑动到列表最底端，则可以正常实现</font>**

**解决方案**：虽然在初始打开界面时“无法滑动到列表实际底端”，但之后刷新列表却可以满足需求。因此在“初始打开界面”时增加一道流程：**<font color=red>延时指定帧数后重新刷新列表</font>**，以满足“滑动到列表真实底端”的需求；同时为保证画面效果，**<font color=red>先将该面板隐藏，在滑动完毕后再显示该面板</font>**(通过“isVisible”参数控制)



#### 屏幕坐标：

FGUI的屏幕坐标原点为“左上角”，UGUI则以“左下角”为屏幕坐标原点，因此在计算“屏幕坐标值”时需要考虑，如：

```c#
//Unity的屏幕坐标系，以左下角为原点
Vector2 pos = Input.mousePosition;

//转换为FairyGUI的屏幕坐标
pos.y = Screen.height - pos.y;
```

##### 坐标转换：

FGUI针对“坐标”的设定：1.屏幕坐标以“左上角”为原点   2.任何GObject的坐标都是“局部坐标”

###### UI元件的“局部坐标”与“FGUI的屏幕坐标”的转换：

```c#
//获取该UI元件在“FGUI屏幕”上的坐标：
Vector2 screenPos = GRoot.inst.LocalToGlobal(pos);

//将“FGUI屏幕”上某个坐标转换成“局部坐标”
Vector2 pt = GRoot.inst.GlobalToLocal(screenPos);
```

注意：这里的“screenPos坐标”都是以“左上角”为原点计算的

###### 世界空间的坐标与“FGUI屏幕坐标”的转换：

```c#
//第一步：先将世界坐标转换成“UGUI屏幕”中的坐标
Vector3 screenPos = Camera.main.WorldToScreenPoint(worldPos);

//第二步：将“UGUI屏幕坐标”与“FGUI屏幕坐标”转换
//由于两者仅“坐标原点”不同，因此可直接计算
screenPos.y = Screen.height - screenPos.y; 
```

**PS**：**<font color=blue>以上最终计算得到的“screenPos”是在“FGUI”中的屏幕坐标，如果需要将该坐标转换成“FGUI”中某个UI元件的坐标，则需要再次经过`GRoot.inst.GlobalToLocal`转换才可使用</font>**





#### 图片、动画和纹理集：

##### 图片：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231225183519100.png" alt="image-20231225183519100" style="zoom: 80%;" />

**“重复边缘像素”**：在图片拼接或者平铺时可能产生缝隙，因此勾选该设置即可避免

**“禁用裁剪边缘空白”**：FGUI编辑器在打包图集时会自动裁剪图片的边缘，但如果某个图片被用于“90度填充”，则需要勾选该设置，以避免在打包图集时自动裁剪

**注意**：如果图片在正式使用时使用“填充”的方式，如下图所示，则需要勾选“禁用裁剪边缘空白”，否则图片显示位置可能会不正确

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240122194705479.png" alt="image-20240122194705479" style="zoom:80%;" />



##### 动画：

动画在设置纹理集时，不能被分布到“纹理集的不同页面”，动画中使用的图片必须在同一页纹理集中。此时可设置动画所属的纹理集为“单独的纹理集”

```c#
//设置动画循环播放，如从第几帧播放到第几帧，循环播放多少次，播放结束后停止在第几帧
aMovie.SetPlaySettings(0, -1, 0, -1);

//监听动画播放结束的回调(如果是循环播放，则在所有循环结束后才算播放完成)
aMovie.onPlayEnd.Set();
```





2.在“发布”包时，在“纹理集”部分可以开启“分离Alpha通道”，这样对于“图集”到导出两张图片：一张不包含“Alpha通道”，一张将“原贴图透明通道”以“等价灰度”的形式展示

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231225183927593.png" alt="image-20231225183927593" style="zoom:80%;" />

**这样设置的好处**：可以在Unity中将这两张贴图都设置为“Automatic Compressed”或其他压缩格式，**<font color=red>减少内存占用</font>**

**PS**：在实际使用中，FGUI会自动处理两张贴图，不需要开发者关心

3.游戏中的“背景图片”包含“透明通道”和“不包含透明通道”的应该分开放置







##### Unity中针对“FGUI中导出的图集”的设置：

Texture Type：选择“Default”。由于是作为“图集”使用，所以不要选择“sprite”

Generate Mip maps：主要用于UI，因此关闭mipmap

TextureShape：保持默认“2D”

FilterMode：保持默认“Bilinear”，这样图像在缩放时能产生比较平滑的过渡效果，副作用是会产生一定的模糊。而且单色的图像缩放会产生不必要的渐变边缘。而使用Point，则图像在缩放时会块状化。

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231225184946805.png" alt="image-20231225184946805" style="zoom: 80%;" />

**PS**：一般使用Bilinear即可，你也可以在FGUI编辑器里将图片安排到不同纹理集，然后每个纹理集设置不同的Filter Mode以满足特殊需求





### FGUI在功能逻辑中的使用：

#### 显示面板、按钮点击、动效触发以及监听动效中的事件：

```c#
//显示面板
GRoot.inst.SetContentScaleFactor(800, 600);
UIPackage.AddPackage("UI/PackageTest01");
GComponent messCom = UIPackage.CreateObject("PackageTest01", "Animation01").asCom;
GRoot.inst.AddChild(messCom);
//按钮点击、触发动效
GObject btnClick = messCom.GetChild("n1");
GGroup groupFlag = messCom.GetChild("n4").asGroup;
Transition animTrans = messCom.GetTransition("t1");

//监听“动效”中的事件
animTrans.SetHook("Rotate1", () => { Debug.LogFormat("TriggerEvent.........."); }); 
//按钮点击监听
btnClick.onClick.Add((() => {
    groupFlag.visible = false; //初始时隐藏该group
    btnClick.visible = false;
    animTrans.Play(() => {
        groupFlag.visible = true;
        btnClick.visible = true;
    });
}));
```

#### 循环列表：

FGUI中**<font color=red>支持复用item</font>**，且可以自由设置item中的Image以及文本。同时**<font color=blue>当item滑动到“视角中心区域”时会有“缩放效果”</font>**

```c#
GRoot.inst.SetContentScaleFactor(800, 600);
UIPackage.AddPackage("UI/PackageTest01");
GComponent messCom = UIPackage.CreateObject("PackageTest01", "LoopListComponent").asCom;
GRoot.inst.AddChild(messCom);

//循环列表
GList loopList = messCom.GetChild("n1").asList;
loopList.SetVirtualAndLoop();
loopList.itemRenderer = (index, item) => {
    //刷新item的图片
    GLoader iconLoader = item.asCom.GetChild("n0").asLoader;
    iconLoader.icon = UIPackage.GetItemURL("PackageTest01", "HomeIconBtn");
    //刷新item的文本编号
    GTextField textNum = item.asCom.GetChild("TextNum").asTextField;
    textNum.text = (index + 1).ToString();
};
loopList.numItems = 10;

//设置item位置：当item滑动到“中心区域”时则放大
loopList.scrollPane.onScroll.Add(() => {
    //设置item放大效果：当item靠近中心区域时，则scale会放大
    float listCenterPosX = loopList.scrollPane.posX + loopList.viewWidth * 0.5f;
    for (int i = 0; i < loopList.numChildren; ++i) {
        GObject item = loopList.GetChildAt(i);
        float itemCenterPosX = item.x + item.width * 0.5f;
        float scaleFactor = 1 - Mathf.Abs(listCenterPosX - itemCenterPosX) / item.width;
        scaleFactor = scaleFactor > 0 ? scaleFactor : 0;
        item.SetPivot(0.5f, 0.5f);  //为展现正确的“缩放显示效果”，这里手动设定其pivot
        item.SetScale(1 + scaleFactor * 0.3f, 1 + scaleFactor * 0.3f);
    }
});
```

运行效果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/jelly%20(2).gif" alt="jelly (2)" style="zoom:67%;" />

**注意**：如上代码中“刷新item缩放”的方法仅在“OnScroll”时才会触发，因此在初次展示列表中的item时，并没有“中心区域的item被放大”的效果，所以只需要在“**初次展示item列表**”时**手动调用一次该方法即可**。

#### 复选框和进度条：

```c#
GRoot.inst.SetContentScaleFactor(800, 600);
UIPackage.AddPackage("UI/PackageTest01");
GComponent mainCom = UIPackage.CreateObject("PackageTest01", "ComboBoxShow").asCom;
GRoot.inst.AddChild(mainCom);

GProgressBar mProgressBar = mainCom.GetChild("n1").asProgress;
mProgressBar.value = 0;

GComboBox mComboBox = mainCom.GetChild("n0").asComboBox;
//监听“复选框”输入
mComboBox.onChanged.Add((() => {
    Debug.LogFormat("ComboBox Value: {0}", mComboBox.value);
    mProgressBar.value = 0;   //重置进度条，以方便后续“TweenValue动画”
    mProgressBar.TweenValue(100, float.Parse(mComboBox.value));
}));
```

在FGUI编辑器中设置复选框各个选项的数值：选中复选框，在右侧“检查器”中点击“编辑列表项目”弹出如下窗口，自由设定每个选项对应的数值

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231227123918284.png" alt="image-20231227123918284" style="zoom: 80%;" />

运行效果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/ComboBox.gif" alt="ComboBox" style="zoom:80%;" />

#### 获取资源URL地址：

部分情况下，可能需要获取某个具体GObject的URL、name、所属的包名等信息：

```c#
//对象的URL地址
Debug.Log(aObject.resourceURL);
//对象在资源库中的名称
Debug.Log(aObject.packageItem.name);
//对象所在包的名称
Debug.Log(aObject.packageItem.owner.name);
//根据URL获得资源名称
Debug.Log(UIPackage.GetItemByURL(resourceURL).name);
```

**注意**：1.在获取某个Gobject的name和包名时，需要先将其转换成PackageItem对象再获取

2.URL地址同样支持**<font color=red>“ui://包名/资源名”，该地址不包含“文件夹”，只需要用到包名和资源名</font>**，如“ui://ui://G002_main/tv”(资源名使用FGUI编辑器中的名字，不需要文件后缀)

#### 设置GLoader的图片：

当图片为Unity中某个Sprite或Texture2D，而不是FGUI中导出的“打成图集的某个图片”时，则需要使用“NTexture”来赋值：

```c#
//使用“资源系统”的加载方法得到sprite或Texture对象
Sprite mSprite = ResMgr.Instance.Load("");
//创建NTexture对象
NTexture mNTexture = new NTexture(mSprite);
//为GLoader所使用的texture赋值
mGLoader.texture = mNTexture;
```

当需要为GLoader赋值某个对象时，也可直接使用“url”，该方式对“图片、动画”等所有类型均有效

```c#
mGLoader.url = "ui://pmf3wbjith1l3ls";
//只要该url所属的package已经被加载到内存中即可
```

#### 使用遮罩实现不同长度的进度条效果：

美术提供的“进度条”通常只有某个固定长度，但由于游戏中会展示不同长度的进度条，如：

![image-20240226155921156](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240226155921156.png) ![image-20240226155937229](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240226155937229.png)

当该进度条的数值为“1/100”时，如果长度过短则会出现以下异常效果：<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240226160146438.png" alt="image-20240226160146438" style="zoom:80%;" />

如上图所示，当为“1/100”时，左侧的进度条太细了，显示异常

**解决方案**：

1.让美术提供不同长度的进度条原图

2.使用遮罩特性：由于创建时可以设置“进度条的伸缩部分”，因此其从本质上讲：支持设置任何组件

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240226160541664.png" alt="image-20240226160541664" style="zoom:67%;" />

因此创建专用于伸缩的组件com_progressBar：其内部包含“正常展示的进度条图片”和“专用于拉伸的遮罩”

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240226160928799.png" alt="image-20240226160928799" style="zoom:80%;" />

其中“遮罩n1”的关联设置为“针对容器的宽宽，高高”。这样当进度条伸缩时会直接修改"com_progressBar"的宽度，即会直接影响该组件内部的“遮罩n1”的宽度，如此实现目标需求

**PS**：com_progressBar内部的“n0”的宽度需要根据进度条实际需要的宽度来设置

#### 文字渐变色：

经过实际测试，在文字中使用UBB语法：`[color=#ff0000,#00ff00]today[/color]`可以得到如下渐变的效果：<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240226161721854.png" alt="image-20240226161721854" style="zoom:80%;" />







### 资源的加载和卸载

#### 加载：

当使用代码在游戏中显示FGUI界面时，该界面会自动被分配到“Don'tDestroyOnLoad”下，因此使用这种方式显示的界面在“切换界面”时不会自动销毁，需要手动处理，如：

```c#
GRoot.inst.SetContentScaleFactor(800, 600);
UIPackage.AddPackage("UI/FGUIShowTest01");
GComponent comp1 = UIPackage.CreateObject("FGUIShowTest01", "Component1").asCom;
GRoot.inst.AddChild(comp1);
```

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231225161904050.png" alt="image-20231225161904050" style="zoom:80%;" />

但是若直接在场景中使用“鼠标右键 -> FairyGUI -> UIPanel”创建“UIPanel”对象，并在其Inspector中设置“Package Name”和“Component Name”参数，如：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231225162133978.png" alt="image-20231225162133978" style="zoom:80%;" />

此时则会直接在该物体下显示“Component1”界面，且该界面在“切换界面”时会随着“UIPanel物体”一起被销毁，无需手动管理

##### 界面的异步加载：

当界面较为复杂时，“UIPackage.CreateObject”方法可能会导致卡顿，因此可通过“UIPackage.CreateObjectAsync”来显示界面，由于该方法是“异步”的，因此可有效缓解卡顿情况，但“加载速度”相对于“同步加载UIPackage.CreateObject”较慢：

```c#
UIPackage.CreateObjectAsync("包名","组件名", MyCreateObjectCallback);

void MyCreateObjectCallback(GObject obj)
{
}
```

#### 卸载：

当某个资源或界面不再使用时，可以使用Dispose方法将其销毁，以释放其内存占用。但对于“纹理、声音”等资源，使用Dispose并不会回收这些资源占用的内存，还需要使用“UIPackage.RemovePackage”来回收













### 输入文本的变化：

```c#
GRoot.inst.SetContentScaleFactor(800, 600);
UIPackage.AddPackage("UI/PackageTest01");
GComponent messCom = UIPackage.CreateObject("PackageTest01", "ComboBoxShow").asCom;
GRoot.inst.AddChild(messCom);

Vector2 leftBottom = new Vector2(0, 1);
GList mList = messCom.GetChild("n37").asList;
GComponent objCom = mList.GetChildAt(0).asCom;
GTextInput input = objCom.GetChild("n4").asTextInput;
input.onChanged.Add((() => {
    Debug.LogFormat("height: {0}", input.textHeight);
    mList.scrollPane.SetContentSize(input.textWidth, input.textHeight);
    mList.height = input.textHeight <= 100 ? input.textHeight : 100;
    //input.pivot = input.textHeight <= 100 ? Vector2.zero : leftBottom;
}));
```







