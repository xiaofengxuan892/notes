#### 遮罩：Mask

UGUI中常用的遮罩类型包含：Mask、ReceMask2D

基于“原生Mask组件”的性能缺点，这里使用优化后的“SoftMask组件”，源码工程：https://github.com/mob-sakai/SoftMaskForUGUI

该SoftMask组件在性能和功能上均优于“原生的Mask组件”



### 2.“Soft Mask”切割“UI Text Outline”文字：

使用“Soft Mask”可以专门用于切割一些使用"UI Text Outline"的文字，普通的“Mask”或者“Rect Mask”是无法切割“Text Outline”文字的：

![image-20221028162319743](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221028162319743.png)

并且“Soft Mask”无需添加“image”图片，直接设置“Source”为“Graphic”即可

这个特性与“UI Empty Graphic”类似，要研究下具体的实现逻辑

