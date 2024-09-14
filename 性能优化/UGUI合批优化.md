[TOC]



#### UGUI合批规则：

==以canvas为基本单位，根据depth、sorting order等依次进行排序==，对于**同一个canvas下的UI**，按照==从上到下==的顺序依次解析各个UI组件的“合批状态”：1.UI之间是否使用相同的材质和图集   2.与之前的某个UI是否有“重叠的绘制区域”，若有，

**解析要点**：

1.**判断UI是否可以合批**：只要两个UI使用的材质和图集相同，则具备合批的必要条件

2.**该UI是否会打断合批**：检测本UI与“从上到下顺序”的某个UI之间是否有“重叠的绘制区域”，若有，则检测两者使用的材质和图集是否相同：若不同，则会打断合批。只要==本UI与之前任意一个UI==出现“绘制区域重叠”且“材质或图集不同”的情况，则==本UI无法与之前的任意一个UI进行合批，且之前的任意一个UI也无法与后续的任何UI之间进行合批==，即**“合批”在此被打断**
**注意**：==若某个UI打断合批，则后续的解析从本UI开始==，该UI之前的组件已计入“合批累计”里，无需再考虑

##### 案例解析：

如下图所示：从上到下依次绘制“white, text, pink, green, blue”五个UI，其中“white, pink, green, blue”为“Image组件”，“text”为“Text组件”。
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240912094447596.png" alt="image-20240912094447596" style="zoom:80%;" />

**解析**：打开“Profiler”窗口并查看“UI”栏目
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240912095103709.png" alt="image-20240912095103709" style="zoom:80%;" />

打开“FrameDebugger”窗口，可查看对应“Profiler -》 UI”中每一个Batch所使用的“图集”以及Shader
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240912095412688.png" alt="image-20240912095412688" style="zoom:80%;" />

从“FrameDebugger”可知，“white、pink、green、blue”所使用的图集为“UnityWhite”(**当没有为“image组件”设置sprite时，则会默认使用该图集**。且==“Image”的“Color”属性不会有影响合批==)，“text”的图集为“Font Texture”

**详情**：从“white”开始，由于“text”与“white”使用的图集不同，因此无法合批，且两者有重叠的绘制区域，因此两者必然分属两个不同的“Batch”；“pink”与“white、text”具有重叠区域，且“pink”与“text”两者使用的“图集”不同，因此会打断合批。
故截止到“pink”，该UI界面至少有“3个Batch”，分别为：“white”、“text”、“pink”，三个组件分别绘制，无法合批。

之后从“pink”开始，“green”、“blue”三者虽有重叠区域，但使用的图集相同，因此可以合批，故“green、blue”在同一Batch被渲染

因此该UI界面共消耗“3”个Batch

##### 案例扩展：

1.若将“pink”下拉，不与“text”重叠，则“white、pink、green、blue”在同一Batch渲染，“text”单独渲染，故总消耗“2个Batch”
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240912101553465.png" alt="image-20240912101553465" style="zoom:80%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240912101748168.png" alt="image-20240912101748168" style="zoom:80%;" />

2.如下图所示：将“green”上拉，与“text”重叠
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240912102043197.png" alt="image-20240912102043197" style="zoom:80%;" />

按照“从上到下”的顺序，“white、text、pink”分别独立渲染，各占一个Batch。且==“pink”打断合批，因此之前的“white、text”已渲染完毕，不再考虑，后续从pink重新开始==。由于“green、blue”与“pink”使用相同的图集和材质，因此可以与“pink”合批，一起渲染。故总共消耗也是“3”个Batch
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240912102600526.png" alt="image-20240912102600526" style="zoom:80%;" />



##### 注意事项：

1.查看UI之间的重叠区域：Scene视图 -》Wireframe
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240912104615968.png" alt="image-20240912104615968" style="zoom:80%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240912104745190.png" alt="image-20240912104745190" style="zoom:80%;" />

2.==DC并非越低越好==
根据UI合批的规则：“合并mesh”和“打图集”，可以有效的降低DC。但在“打图集”时，若将大量图片打入同一图集中，如“背包界面的道具icon”，此时该图集过大，会导致加载缓慢且占用较大内存
在“合并mesh”时，若将大量物体的mesh进行合并，得到的总mesh中顶点信息数据过多，除占用内存外，**向GPU传递数据时也较为缓慢**，此时则有可能会降低游戏帧率

3.==UI逻辑中暂不需要的组件要隐藏，避免打断合批==
由于UI之间的重叠绘制区域有可能会因为材质或贴图不同而打断合批，因此对于暂时不需要显示的UI最好隐藏，不要直接使用UI覆盖来实现效果(直接使用UI覆盖虽然可以实现功能，但有可能会打断“UI合批”)

参考链接：https://blog.csdn.net/hzhnzmyz/article/details/121101645



##### 细节要点：

1.对于UGUI中原生的“Image、Text”组件，==直接修改其“Color”数值不会影响合批，即默认为相同的材质和图集==。但是如果==给Text组件添加“Outline”，并修改“Outline”的“Color”数值或者“Width”，则会打断合批==：虽然使用的图集依然为“Font Texture”，但材质的颜色不同，因此无法合批





















以canvas为基本单位，根据depth，sorting order等依次进行排序，对于同一个canvas下的UI，按照从上到下的顺序依次进行绘制，

对于UI中使用同一图集的则会进行合批 —— 使用不同图集的UI会分开进行渲染，无法合批；对于UI物体使用图集可以很明显的降低drawcall

##### UGUI优化的方向：

###### 1.尽量减少Mesh重建发生的次数：

对于UI界面的切换 —— 显示、隐藏等，不要直接的使用实例化或者销毁，也不要使用active、deactive

对于一些常用的界面可以设置RectTransform的坐标，将其移出canvas达到隐藏的效果；

或者设置scale为0，也可以达到效果

或者设置该UI界面的layer

注意：当UI的transform发生改变时，如果不影响UI元素直接的遮挡关系，此时是不会导致mesh重建的

###### 2.降低Mesh重建影响的范围：

对于在游戏运行当中需要频繁变化的UI元素可以将其提取出来放到独立的canvas下，避免影响到其他不需要变化的UI

例如游戏中频繁变化的Text动态文本——在运行时修改text内容，如动态表情等

或者血条变化UI

将这些UI元素提取出来放在独立的canvas下

注意：

1.UGUI以canvas为基本单位进行Batch计算，因此**每个canvas至少会占用一个drawcall，因此canvas数量也不宜过多**

2.各个canvas是独立计算drawcall的，所以**如果有某两个canvas下的UI元素使用的是同一个图集，依然不会被合批，而会分开计算Batch**

###### 3.UGUI中其他可以优化的细节：

1.对于不需要点击交互的UI，不要勾选“Raycast Target”

2.尽量减少Mask的使用：当给Image添加Mask遮罩后，drawcall至少增加2

3.减少给Text组件添加Outline效果，给Text添加Outline后，顶点数大概是之前的7倍

  outline会对每个字的上下左右都绘制一遍

4.Image的图片类型尽量选择simple，不要选择Tiled；

 选择Tiled后，该图片的顶点数也会增长很多倍

参考链接：https://www.cnblogs.com/zhaoqingqing/p/9658403.html

5.PixelPerfect：对齐像素的效果，可以让UI元素显示更好的效果，但是消耗也会增加，

  尤其是背包界面中item cell过多，当滑动时多个cell都会重新渲染，因此PixelPerfect增加的消耗会多很多

6.对于频繁出现的战斗提示信息等内容可以设置text = “”来代替UI的active、deactive

  额外知识点：动画组件的Animator的active, deactive也会有很大的消耗

  https://www.daimajiaoliu.com/daima/4ed7ac5041003fc









