[TOC]



##### UGUI合批规则：

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









