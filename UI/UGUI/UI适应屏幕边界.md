[TOC]



### 1.Anchor：

#### AnchorPosition

当UI的四个Anchor汇集于一点时，本参数代表“这个汇集点”距离“UI的Pivot”的距离

##### AnchorPosition与LocalPosition的区别

在设置UI的坐标时，以上两者均可以使用。但两者分别是从不同的角度来代表UI的位置，因此其意义不同。

**实际使用场景**：

**1.“根据鼠标点击位置设置UI”**：

该种情况通常会用到“空间坐标转换”，常用

```c#
RectTransformUtility.ScreenPointToLocalPointInRectangle
```

将“屏幕空间”的坐标转换，但该方法得到的是“localPosition”，所以在设置UI的坐标时，只能使用“localPosition”参数

**2.“按钮自适应圆圈边界”**：

该情况下在设置按钮的“Pivot”后，为了方便设置按钮的坐标，故将四个anchor汇集于一点，并且与“Pivot”保持重合，即设置`self.buttonExit.anchoredPosition = Vector2.zero`

##### 补充说明：

![image-20220905110241226](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220905110241226.png)

如上图所示，UI的RectTransform的坐标“PosX, PosY”代表的是该UI的“AnchorPosition”，而非“LocalPosition”或“Position”

但“RectTransform”继承自“Transform”，因此该组件同样包含“LocalPosition，Position”参数。只是在RectTransform的Inspector面板中通常不会显示“LocalPosition”数值



#### AnchorMin与AnchorMax

该数值代表UI的四个锚点的分布，其可以在Prefab上直接设置，也可以在代码中动态设置

当两者数值相等时，则代表该UI的“四个锚点位置重合”，此时的“AnchorPosition”数值为“重合点与该UI的Pivot”之间的距离

**PS**：**AnchorMin与AnchorMax不同情况下的数值**

当本UI完全以父UI自适应时：

![image-20220905104952358](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220905104952358.png)

当以父UI的顶端自适应时：

![image-20220905105126332](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220905105126332.png)

可以在代码中动态调整UI的“Anchor”与父UI的自适应标准



#### Anchor与SizeDelta之间的关系：

SizeDelta体现的是该UI的“Rect区域”与“四个Anchor包围的矩形区域”之间的差值，即：

```c#
sizeDelta.x = rect.x - anchorRectangle.x;
sizeDelta.y = rect.y - anchorRectangle.y;
```

因此SizeDelta的“x, y”值有正负





### 2.Pivot：

代表UI的中心点。在设置UI的位置时，“LocalPosition或Position”实际代表的是该中心点的位置。这也是“AnchorPosition”与“LocalPosition”的区别

**使用场景**：

1.当设置UI的Width/Height时，**以该Pivot为对称中心，向左右或上下展开“Width/2, Height/2”距离**，即为该UI“上下左右”四个顶点的位置。

2.当在2D平面旋转该UI时，则以Pivot为中心进行旋转





### 3.世界空间、屏幕空间、Canvas空间、本地空间

#### 1).三个空间下坐标代表的意义

**世界坐标**：在世界空间下的坐标，指的是渲染该3D物体所使用的camera，通常为“mainCamera”

**屏幕坐标**：处于屏幕空间内，通常是指某些UGUI物体，渲染UI物体时有专门的Camera，与世界空间中使用的Camera不同

**Canvas坐标**：屏幕空间的坐标在UGUI的canvas下的UI坐标

**本地坐标**：强调作为某个物体的子物体所属的坐标



#### 2).“世界空间”和“屏幕空间”下的坐标转换

**核心**：在转换“世界空间”和“屏幕空间”中的坐标时，需要**<font color=red>严格注意“当前待转换的坐标”所使用的“Camera”</font>**。如“屏幕坐标转换成世界坐标”，则需要使用“该屏幕坐标的Camera”.ScreenToWorldPoint；若“世界坐标转换成屏幕坐标”，则使用“该世界坐标的Camera”.WorldToScreenPoint

**使用场景**：

##### 将“屏幕坐标”转换成“世界坐标”

**方法**：`xx.ScreenToWorldPoint(Vector3 screenPos)`

**说明**：

对于“UI界面的坐标”，则需要根据当前"UGUI的Canvas渲染模式"来确定：

若Canvas的渲染模式为“ScreenSpace - Overlay”，则使用“渲染3D物体的Camera”，如Camera.main等；

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230528184913004.png" alt="image-20230528184913004" style="zoom:67%;" />

若Canvas的渲染模式为“ScreenSpace - Camera”或“WorldSpace”，则需要使用当前配置的Camera用于坐标转换：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230528191254258.png" alt="image-20230528191254258" style="zoom:67%;" /><img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230528191332219.png" alt="image-20230528191332219" style="zoom:67%;" />

###### 易错点：

该方法的参数为“Vector3”类型，而“屏幕空间”的“z坐标”默认为0。在将“屏幕坐标”转换成“世界坐标”时，需要手动为该Z分量赋值，否则转换之后均为“所使用的Camera的世界坐标”(Camera的世界坐标Z分量始终为0)。

**<font color=red>默认设置“z > 0”，且大于该Camera的“Near Clipping Planes”的距离，代表在Camera正前方，否则不会被Camera渲染到</font>**：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220905162528202.png" alt="image-20220905162528202" style="zoom: 80%;" />

**原因**：世界坐标的Z分量代表该位置距离Camera的远近，当将其投影到屏幕空间时，Z分量会重置为0。因此若要将该“屏幕空间”的坐标还原成“世界坐标”，则必须要知道该坐标在Z轴上的分量。对于该分量的数值可自由设置，只需要满足“z > 0”，其大于“该Camera的Near Clipping Planes”数值即可；

参考链接：https://gamedevbeginner.com/how-to-convert-the-mouse-position-to-world-space-in-unity-2d-3d/#screen_to_world_3d

##### 将“世界坐标”转换成“屏幕坐标”

**方法**：`xx.WorldToScreenPoint(Vector3 worldPos)`

**说明**：同样需要确定“该世界坐标所属的Camera”。通常情况可以直接使用Camera.main，但如果项目中为某个3D物体使用了“指定的Camera”用于渲染，则必须要使用该“指定的Camera”进行坐标转换

**PS**：由于“世界坐标”本身具备“z分量”，因此无需手动设定Z分量



#### 3).“屏幕空间”与“Canvas空间”下的坐标转换

**方法**：`RectTransformUtility.ScreenPointToLocalPointInRectangle(RectTransform parentRect, Vector2 screenPos, Camera screenPosCamera, out Vector2 localPos)`

**说明**：该方法用于将某个屏幕坐标转换成“指定UI物体下的直接子节点的坐标”，由于设置了该位置的“父UI”，因此该方法返回的为“LocalPosition”

**参数解析**：

***<font color=red>parentRect</font>***：该节点在UGUI中的直接父UI的RectTransform
<font color=red>***screenPos***</font>：屏幕空间中的坐标
<font color=red>***screenPosCamera***</font>：该screenPos所属的Camera。通常使用当前UGUI的“Canvas渲染模式”所使用的Camera
<font color=red>***localPos***</font>：该screenPos在当前UGUI的Canvas渲染模式下，作为"父UI的parentRect"的直接子UI的“localPos”。由于screenPos与父UI的position不同，因此转换后得到的“localPos”必然不为0

**使用案例**：

**1)**.根据鼠标点击位置设置UI

```lua
-- 由于当前主要用于计算UGUI中某个UI界面的位置，因此针对“鼠标坐标”则使用“UGUI的Canvas渲染模式”中指定的Camera
local uiCamera2D = sq.facade:GetCameraGroup().UICamera2D
local ok, pos = RectTransformUtility.ScreenPointToLocalPointInRectangle(self.safeArea, Input.mousePosition, uiCamera2D, nil)
if ok then
	self.content.localPosition = pos
end
```

**解析**：由于RectTransformUtility.ScreenPointToLocalPointInRectangle方法获取到的是“localPosition”，因此在设置UI的位置时也需要使用“localPosition”，而非“anchorPosition”

**2)**.在设置UI血条跟随3D物体时，先将该3D物体的世界坐标转换成“屏幕坐标”，然后使用本方法获取到“UI血条”的localPosition即可

参考链接：https://blog.csdn.net/xiexian1204/article/details/80919221



#### 4).“世界空间”与“本地空间”的坐标转换

**“世界坐标”与“本地坐标”的区别**：

“世界空间”与“局部空间”的坐标，从本质上讲，二者其实属于同一维度，即“三维空间”的坐标。只是“本地空间”强调本物体是作为某个物体的“子对象”而存在的坐标，因此其实际代表：“子GameObject”相对于“父GameObject”的坐标偏移

***<font color=red>Transform.TransformPoint</font>***：从本地空间转换为世界空间

***<font color=red>Transform.InverseTransformPoint</font>***：从世界空间转换为本地空间

```c#
transformA.InverseTransformPoint(transformB.Position)
//获取transformB相对于transformA的局部坐标，如果transformA有自定义的缩放比例，那么该方法内部会自动进行转换，得到的是在该参照物position以及scale的标准下的局部坐标
```

**参考链接**：https://blog.csdn.net/weixin_41772285/article/details/107725372

**PS**：针对于“本地坐标”，由于其代表的是“相对于父GameObject”的“坐标偏移”，因此实际可以在任何维度的空间中使用，如“UGUI中的某个UI的localPosition”等



但是，如果UGUI中**<font color=blue>canvas设置的渲染模式为“screen space - camera”</font>**，那么使用“GetWorldCorners”得到的是在**<font color=red>指定camera</font>**下的世界空间中的世界坐标(**<font color=red>虽然canvas显示为“screen space”，但实质获得的坐标为世界空间下的世界坐标</font>**)

此时需要将**世界坐标转换为屏幕空间中的坐标**，那么<font color=blue>**与此世界坐标相绑定的camera是canvas中设置的camera。**</font><font color=red>**在转换成屏幕坐标时：camera.canvasCamera.WorldToScreenPoint，也必须使用与此世界坐标相绑定的camera来转换才可以。**</font>





### 4.UI自适应屏幕边界

在“UI自适应屏幕边界”中，通常都会经过以下两步：

#### 第一步：判断UI是否超出屏幕边界 —— GetWorldCorners

##### `RectTransform.GetWorldCorners`

在判断“UI界面是否超出屏幕边界”时，通常使用该方法获取UI界面四个角的坐标，根据坐标数值确定该界面是否超出屏幕。但该方法的返回值会受到当前“Canvas的渲染模式”的影响：**<font color=red>在不同的渲染模式下，该方法获取的坐标代表的意义不同</font>**：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220905172204008.png" alt="image-20220905172204008" style="zoom: 80%;" />

###### 1).当渲染模式为“ScreenSpace - Overlay”时：

该模式下`GetWorldCorners`方法获取到的为“屏幕坐标”，可直接将其与“屏幕边界值”进行比较，无需进行任何转换

###### 2).当渲染模式为“ScreenSpace - Camera”时：

**注意**：在指定本渲染模式下的Camera时，**<font color=red>通常使用“Orthographic正交模式”的Camera</font>**

该模式下GetWorldCorners方法获取到的为“世界空间下的世界坐标”，需要将其使用“xxCamera.WorldToScreenPoint”方法转换成“屏幕坐标”后才能与“屏幕边界数值”比较，以确定是否超出边界。进行坐标转换的Camera必须为“本渲染模式中指定的Camera”

**PS**：为了直接有效的展示该特点，可以将本方法返回的“四个坐标”直接赋值给四个GameObject的Position参数。并且基于此特性，如果需要在某个UI上展示3D模型，则直接设置该3D模式的“Z坐标分量”大于本方法返回的“z分量”即可。

**问题**：部分情况下，如果直接将本渲染模式下`GetWorldCorners`方法返回的坐标赋值给3D物体，可能会出现GameObject不展示的问题。

**方案**：为该GameObject创建新的Layer层，并在“本渲染模式中指定的Camera”的“CullingMask属性”中添加该Layer层，同时去掉MainCamera的“CullingMask”中对该Layer层的渲染。参照demo中的“ScreenSpaceUI_Camera.unity”

**原理**：当把本渲染模式下GetWorldCorners返回的坐标赋值给3D物体时，则代表该物体实际是使用“本渲染模式指定的Camera”来渲染的。默认情况下，3D物体均由MainCamera负责渲染的，该Camera使用Perspective透视模式，而本渲染模式下指定的Camera通常使用“Orthographic正交模式”，两者得到的坐标在视觉上会呈现很大的差异。

###### 3).当渲染模式为“WorldSpace”时：

**注意**：在指定本渲染模式下的Camera时，**<font color=red>通常使用“Perspective透视模式”的Camera</font>**

该模式下GetWorldCorners返回的坐标即为“世界空间下世界坐标”，同样需要使用本模式指定的Camera用于“世界坐标与屏幕坐标的转换”。之后才能将转换后得到的“屏幕坐标”与“屏幕边界数值”进行比较

**PS**：该模式即把UI当成普通的3D物体来用，因此常用于制作类似“刺客信条”中的"3D UI效果"



##### 易错点：

1).无论是`Camera.xxCamera.ScreenToWorldPoint`，还是`RectTransformUtility.ScreenPointToLocalPointInRectangle`，在涉及坐标转换时，所使用的camera都必须是与坐标参数相绑定的camera

2).在将鼠标坐标(屏幕坐标)进行转换时，如果“Hierarchy”中包含有多个不同RenderMode的Canvas，那么需要明确**<font color=red>目标UI所要放置的Canvas</font>**。在进行坐标转换时，使用目标Canvas中的Camera进行转换，这样得到的坐标才是在目标Canvas下的正确坐标。

但如果RenderMode为“ScreenSpace-Overlay”，那么此时鼠标坐标就是正确的屏幕坐标，可以直接使用该坐标为UI赋值，不需要再经过任何坐标转换

**PS**：**完整的项目Demo**：https://download.csdn.net/download/m0_47975736/86510176

参考链接：https://blog.csdn.net/fdyshlk/article/details/78509909



#### 第二步：调整UI的Pivot以适应边界

在调整Pivot时，默认只需要设置Pivot为“上下左右”四角即可。但如果UI“展示在屏幕上的Width/Height”较大，已经超出“屏幕Screen.Width/Height”数值的一半，那么若在屏幕中心区域点击时调整Pivot，则依然有可能“超出屏幕边界”。**<font color=red>需要在“上下左右”四个顶点的基础上，增加Pivot.x、Pivot.y为“0.5”的情况</font>**

**重要**：在获取UI“展示在屏幕上的Width/Height”时，**<font color=red>不要使用“rect.Width”，而只能使用GetWorldCorners方法返回的四个顶点坐标值。在将这些坐标转换成屏幕坐标后，通过计算四个顶点的“屏幕坐标”来获取到该UI实际“展示在屏幕上的Width/Height”</font>**

**原因**：在不同的Canvas渲染模式以及“UI Scale Mode/Resolution”下，UI界面会自动调整其展示在屏幕上的大小。

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221114153808995.png" alt="image-20221114153808995" style="zoom:80%;" />

而UI界面的“rect.Width/Height/Size”等参数是固定不变的，如在不同的渲染模式和ScaleMode/Resolution下，获取“技能详情面板的UI宽高”时：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230529114544399.png" alt="image-20230529114544399" style="zoom: 80%;" />

得到的数值是完全一样的，均为“424.3999”：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221114154519649.png" alt="image-20221114154519649" style="zoom: 80%;" />

因此若要确定UI界面是否超出屏幕边界，则不能使用该数值。

但**<font color=red>若只是单纯的设置某个UI界面的大小，不涉及“屏幕边界检测”，则可以直接使用“rect.Width/Height/Size”参数</font>**

**实现代码**：

**PS**：以下代码只有“上下左右”四个顶点Pivot的设置，尚未增加“Pivot.x/y为0.5”的情况，可根据上述原理添加即可：

```lua
--- @brief 自适应屏幕边界，确保不超出屏幕
--- @desc 当前pivot只考虑Vector2(0, 0)，Vector2(0, 1)， Vector2(1, 1)， Vector2(1, 0)四种情况
--- @private
function BattleSoulCopyMissionRewardTipsDialog:AdjustByScreenEdge()
    local preferredPivot = self.content.pivot    -- 获取原始pivot
    -- 获取当前canvas下的四角屏幕坐标
    local cornerWorldPos = self.content:GetWorldCorners()
    local cornerScreenPos = {}
    for i = 1, 4 do
        -- 以左下角为起点，顺时针方向
        cornerScreenPos[i] = sq.facade:GetCameraGroup().UICamera2D:WorldToScreenPoint(cornerWorldPos[i - 1])
    end

    -- 超出左边边界
    if cornerScreenPos[1].x < 0 then
        if cornerScreenPos[2].y > Screen.height then
            -- 同时超出上边界
            preferredPivot = kPivotsList.upperleft
        elseif cornerScreenPos[1].y < 0 then
            -- 同时超出下边界
            preferredPivot = kPivotsList.bottomLeft
        else
            preferredPivot = Vector2(0, preferredPivot.y)
        end
        -- 超出右边边界
    elseif cornerScreenPos[3].x > Screen.width then
        if cornerScreenPos[3].y > Screen.height then
            -- 同时超出上边界
            preferredPivot = kPivotsList.upperRight
        elseif cornerScreenPos[4].y < 0 then
            -- 同时超出下边界
            preferredPivot = kPivotsList.bottomRight
        else
            preferredPivot = Vector2(1, preferredPivot.y)
        end
        -- 在X轴边界内
    else
        if cornerScreenPos[2].y > Screen.height then
            -- 超出上边界
            preferredPivot = Vector2(preferredPivot.x, 1)
        elseif cornerScreenPos[1].y < 0 then
            -- 超出下边界
            preferredPivot = Vector2(preferredPivot.x, 0)
        end
    end

    return preferredPivot
end
```



#### 扩展案例：当UI超出屏幕边界时，不调整该UI自身，而调整该UI上子对象的分布

如下图：当“绿色环形圆圈”超出屏幕边界时，调整其上“各个子按钮”的分布

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220803145137900.png" alt="image-20220803145137900" style="zoom: 67%;" />

**实现原理**：

1.当“环形圆圈”超出屏幕边界时，**<font color=blue>设置各个子按钮的“AnchorMin、AnchorMax”</font>**

2.在设置UI的“AnchorMin/AnchorMax”后，其“AnchorPosition”会随之改变，**<font color=red>为了忽略“AnchorPosition”的影响，完全满足Pivot的理想情况，需要设置“Pivot = Anchor”，此时“AnchorPosition”则保持为Vector2.Zero即可</font>**

3.子按钮是分布在“白色环形”上，且环形的四个角落是空的，因此“上下左右”四角的“AnchorMin/AnchorMax”设置默认是靠里面的，这样可以保证“按钮正好显示在白色圆形之上”。

而**<font color=red>对于“TopMiddle/LeftMiddle/Right/Middle/BottomMiddle”，为了保证“按钮恰好显示在白色环形之上”，需要设置按钮的Pivot为“0.5, 0.5”，同时设置其AnchorPosition = Vector2.zero，才能保证满足需求</font>**

**<font color=blue>注意：这里的设置顺序必须要是“先设置Anchor，之后设置Pivot，最后设置AnchorPosittion”</font>**

**具体代码**：

```lua
local AnchorsList = {
    upperleft = Vector2(0, 1), upperMiddle = Vector2(0.5, 1), upperRight = Vector2(1, 1),
    middleLeft = Vector2(0, 0.5), middleRight = Vector2(1, 0.5),
    bottomLeft = Vector2(0, 0), bottomMiddle = Vector2(0.5, 0), bottomRight = Vector2(1, 0)
}

--- @brief 自适应按钮位置
--- @private
function BattleGeneralInfo:AdjustBtnPos()
    -- 由于当前prefab中UI结构，“ImageBG”是实际影响“限制区域”变化的UI因素
    local cornersScreenPos = {}
    local cornersWorldPos = self.imageBG:GetWorldCorners()   -- 由于当前canvas设置，这里得到的是UI在world space中的世界坐标
    for i = 1, 4 do                                           -- 以左下角为起点，顺时针方向
        cornersScreenPos[i] = sq.facade:GetCameraGroup().MainCamera:WorldToScreenPoint(cornersWorldPos[i - 1])
    end

    -- 自适应按钮位置
    -- 由于当前UICamera3D以及UIRoot3D的设置，根据四个corner的数值可知：每个corner的屏幕坐标x都是不一样的, 但top和bottom的两个corner的Y坐标分别都是一致的
    -- 超出左边边界
    if cornersScreenPos[1].x < 0 or cornersScreenPos[2].x < 0 then
        if cornersScreenPos[1].y < 0 then                      -- 同时超出下边界
            self:SetBtnAnchorByPos(AnchorsList.middleRight, AnchorsList.upperMiddle)
        elseif cornersScreenPos[2].y > Screen.height then      -- 同时超出上边界
            self:SetBtnAnchorByPos(AnchorsList.bottomMiddle, AnchorsList.middleRight)
        else                                                   -- 在Y轴边界内
            self:SetBtnAnchorByPos(AnchorsList.bottomRight, AnchorsList.upperRight)
        end
        -- 超出右边边界
    elseif cornersScreenPos[3].x > Screen.width or cornersScreenPos[4].x > Screen.width then
        if cornersScreenPos[4].y < 0 then                      -- 同时超出下边界
            self:SetBtnAnchorByPos(AnchorsList.middleLeft, AnchorsList.upperMiddle)
        elseif cornersScreenPos[3].y > Screen.height then      -- 同时超出上边界
            self:SetBtnAnchorByPos(AnchorsList.bottomMiddle, AnchorsList.middleLeft)
        else                                                   -- 在Y轴边界内
            self:SetBtnAnchorByPos(AnchorsList.bottomLeft, AnchorsList.upperleft)
        end
        -- 在X轴边界内
    else
        if cornersScreenPos[1].y < 0 then                      -- 超出下边界
            self:SetBtnAnchorByPos(AnchorsList.upperRight, AnchorsList.upperleft)
        elseif cornersScreenPos[2].y > Screen.height then      -- 超出上边界
            self:SetBtnAnchorByPos(AnchorsList.bottomRight, AnchorsList.bottomLeft)
        else                                                   -- 未超出任何边界，使用默认anchor设置
            self:SetBtnAnchorByPos(AnchorsList.bottomMiddle, AnchorsList.upperMiddle)
        end
    end
end

--- @brief 设置按钮Anchor位置
--- @param btnExitAnchor UnityEngine.Vector2 "关闭"按钮Anchor位置
--- @param btnLevelUpAnchor UnityEngine.Vector2 "升级"按钮Anchor位置
--- @private
function BattleGeneralInfo:SetBtnAnchorByPos(btnExitAnchor, btnLevelUpAnchor)
    -- 四个anchor汇集于一点
    self.buttonExit.anchorMin = btnExitAnchor
    self.buttonExit.anchorMax = btnExitAnchor
    self.buttonLevelUp.anchorMin = btnLevelUpAnchor
    self.buttonLevelUp.anchorMax = btnLevelUpAnchor

    -- 设置Pivot与Anchor重合
    if btnExitAnchor.x == 0.5 or btnExitAnchor.y == 0.5 then 
        -- 说明处于topMiddle, middleLeft, middleRight, bottomMiddle其中一种情况
        -- 保证按钮正好显示于“白色圆环之上”
        self.buttonExit.pivot = Vector2(0.5, 0.5)             
    else
        self.buttonExit.pivot = btnExitAnchor                 
    end
    -- 确保显示没有偏移
    self.buttonExit.anchoredPosition = Vector2.zero    
    
    if btnLevelUpAnchor.x == 0.5 or btnLevelUpAnchor.y == 0.5 then
        self.buttonLevelUp.pivot = Vector2(0.5, 0.5)
    else
        self.buttonLevelUp.pivot = btnLevelUpAnchor
    end
    self.buttonLevelUp.anchoredPosition = Vector2.zero
end
```





