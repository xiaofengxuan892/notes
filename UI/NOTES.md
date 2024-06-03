#### slot在除了本身可以传递的参数外，还接受从外部接收新的数值传递：

ParameterizedSlot(self. self.ResetSelected, page)：这里在使用ParameterizedSlot封装方法self.ResetSelected时传递的参数“page”
是只要self.ResetSelected方法执行就会直接使用在封装时传入的数值page，这个page在任何情况下都不会被外界改变。
只有封装self.ResetSelected时可以设定其值，封装完成后就无法再改变。
page只要self.ResetSelected执行就可以使用page的数值，类似于self.ResetSelected的一种static常量变量
但是self.ResetSelected除了可以使用自身的static变量外，还可以接受从外部获取新的数值，并且参数数量类型完全自定义
比如：panel:SetClickHandler(ParameterizedSlot(self, self.ResetSelected, page))后，
在该panel对应的lua脚本中，
function xxx:OnItemclicked()
    if self.handler then
          self.handler(a, b, c,........)     -- 可以在这里传入任何类型数量的参数
    end
end

在self.ResetSelected的方法体定义时：
page是该方法自带的static变量，任何时候都拥有；a,b,c则是从外部获取的参数
参数的顺序按照方法体自带的变量，外部传递进来的变量的顺序来排列
function self:ResetSelected(page, a, b, c......)
     .........
end





##### 创建表中的元素时，如果元素的key可能变化，没有固定的name，此时可以使用这种方式来替代：

self[memberName] = sq.facade:GetUIManager():CreatePanel(panelName)
self[memberName]:Hide()
以上“memberName”是变化的



### 很实用的组件：

UI Button Group



### **做WarGameTipsDialog自适应屏幕边界时，遇到的坑：**

**1.**不要在textDesc上挂content size fitter组件，在计算textDesc理想宽高时直接使用text.preferredHeight来获取，不要使用“LayoutRebuild.ForceRebuildImmediate”

**2.**由于当前UI中设置了textDesc与上层content之间四角Anchors的sizeDelta，所以在设置上层content的宽高时，**这里默认是不改变textDesc的宽度的**。所以只要在代码中使用

```lua
self.content:SetSizeWithCurrentAnchors(RectTransform.Axis.Vertical, finalPreferredHeight);
```

即可。不要调用：

```lua
self.content:SetSizeWithCurrentAnchors(RectTransform.Axis.Horizontal, finalPreferredWidth);
```



#### 父panel关闭时会自动调用subPanel的close方法：

在每一个panel中如果其中某个子对象使用了新的subPanel，那么在关闭该页面时会自动的调用subPanel中的close方法。这个很重要，查看下具体执行逻辑，详情：WarGameWindow中的UIPageDragCommonController。**当WarGameWindow关闭时会自动关闭子panel**





#### 根据字段来打表的机制要仔细研究下：

XlsxFiles.json文件
  {"basename": "BattleGuideConf", "sheet_name": "BattleGuideStep", "indexers": [["GroupId"],["GroupId","StepId"],["StepId"]]},
  {"basename": "WantedConf", "sheet_name": "DayMission", "indexers": [["WantedId"]]},
  {"basename": "WantedConf", "sheet_name": "Cost", "indexers": [["SpeedCostID"]]}



#### 在OnOpen中return false可以直接关闭已经打开的页面：

这个要研究下里面具体的机制：

```lua
function MailWindow:OnOpen(data)
    self.items = {}

    if not self:FetchDataOnOpen() then
        return false
    end
    self:Refresh()
end
```





#### 对于Unity中文件的刷新：

为了方便调试，通常不会开启Unity的自动刷新，只有在当前阶段代码写完后测试实际运行效果时才会需要刷新Unity。所以点击“Ctrl + R”即可。

Lua代码并不需要点击“Ctrl + R”，Unity会自动识别修改后的Lua文件，不需要刷新。

PS: 点击“Ctrl +R”后，Unity会对修改后的文件进行刷新，如C#代码则需要编译，prefab如果是从git上pull后也是需要点击“Ctrl + R”后刷新的





#### 如何获取Unity中内置shader的源码脚本文件：

有些情况下可能需要查看Unity内置shader的源码文件，如在处理粒子和UI遮挡关系时。其实无需去Unity官网下载bultinShader压缩包，直接在项目内部即可查看到shader源文件：

首先在Unity中任意新建Image文件，选择对应的material，点击“Edit Shader”即可自动打开目标shader源文件

PS: 其实可以使用以上方式查看任意shader文件，只需要在任意Material中选择对应的shader，然后在右上角的菜单中选择“Edit Shader”即可





#### 图片遮挡按钮点击响应：

情况一：

如果图片和按钮没有任何父子关系，是完全相互独立的，那么只要两者在canvas中有重合的区域，那么图片就可以遮挡按钮的点击

这种情况如果要消除图片对按钮的遮挡：

1.关掉该image的“raycast target”即可

2.添加“canvas group”组件：

关掉“Blocks Raycasts”即可。  虽然这种方式也能用，但没有第一种方便



情况二：如果image和button之间有父子关系：
通常情况下，如果父类是button，那么子物体中除了子button外的所有点击都会传递给父类button，如果要阻断父类button的响应，就必须子类中也有button，这样才能阻断父类button响应。

具体情形：父类是button，子类是image

要求效果：当点击子类image时，阻断父类button的响应

**解决办法：在子类image中添加button**

**PS: 当前是否有更好的办法可以阻断父类button的响应，除了在子类中添加button外。**

**使用子button来阻断父button，虽然可行，但是否有更好的办法**





##### 当需要隐藏或者显示某个image或者text时，不要直接的setActive，可以使用：

```lua
GraphicUtils.SetGraphicEnabled(target, value)
```

target可以是transform，理论上应该任何component都可以，不一定必须是transform，

```lua
function GraphicUtils.SetGraphicEnabled(target, value)
    local transform = target
    if type(target) == "table" then
        transform = assert(target.transform)
    end
    transform:SetGraphicEnableInChildren(value)
end
```

这样比setActive，对UGUI的合批优化有好处



#### 重要：在lua的子脚本中声明父类的同名方法真的会完全覆盖父类的同名方法吗？

经过实际的测试对比，当lua中”MailDetailDialog2“继承自”MailDetailDialog“，然后在”MailDetailDialog2“和”MailDetailDialog“中的同名方法”Initialize“以及”Open“, "UpdateView"中都加入print输出语句：

```lua
--- @class MailDetailDialog2 : MailDetailDialog
--- @field data MailDetail
--- @field mailModule MailModule
local MailDetailDialog2 = assert(self)
```

”MailDetailDialog“中：

```lua
function MailDetailDialog:Initialize()
   print(string.formatex("first MailDetailDialog Initialize.......1111111111"))
end

function MailDetailDialog:OnOpen(context)
	assert(context.data, "data cannot be null")
	self.data = context.data
	self:UpdateView()
	
	print(string.formatex("first maildetail open.......1111111111"))
end

function MailDetailDialog:UpdateView()
	print(string.formatex("first maildetail UpdateView.......1111111111"))
end
```

"MailDetailDialog2"中：

```lua
function MailDetailDialog2:Initialize()
    print(string.formatex("second maildetail2 Initialize.......2222222222"))
end

function MailDetailDialog2:OnOpen(context)
    assert(context.data, "data cannot be null")
    self.data = context.data
    self:UpdateView()

    print(string.formatex("second maildetail2 open.......2222222222"))
end

function MailDetailDialog2:UpdateView()
    print(string.formatex("second maildetail2 UpdateView.......2222222222"))
end
```

当在”MailListItem“中打开”MailDetailDialog2“时：

```lua
local panel = sq.facade:GetUIManager():CreatePanel("MailDetailDialog2", {data = self.data})
```

输出结果如下：

<u>**从以上结果可知：”MailDetailDialog“中的同名方法被完全覆盖，没有任何语句输出**</u>

<u>**所以在lua中使用继承时需要非常小心，方法命名需要很注意**</u>



## 如何将panel与GameObject绑定起来：

```lua
function UIManager:ActivatePanelByGameObject(className, go, reuse)
    assert(go, "Invalid game object")

    local uiCls = class.get(className)
    assert(uiCls, "Cannot find class %s", (className or "nil"))
    assert(uiCls.PanelType == UIDef.PanelType.UIRepeatable, "Error panel type: %s, must be 'UIRepeatable'", (uiCls.PanelType or "nil"))

    -- 设置reuse选项
    uiCls.Reuse = reuse or false

    local panel = uiCls:Create()
    panel:Activate(go)

    return panel
end
```

使用到panel中的”Activate“:

```lua
function UIBasePanel:Activate(gameObject)
    self.gameObject = gameObject
    self.transform = gameObject.transform
    self.originParent = self.transform.parent

    self:BindingModules()
    self:Binding()

    Logger.Debug(AppDef.LogTag.UI, "Panel activate, name: {0}, self: {1}", self:classname(), self:unique_id())

    self:SafeCall(self.OnActivate)
    self.panelState = UIDef.PanelState.ACTIVATED

    -- 在所有元素绑定完成后才能重布局, 否则会查找不到对应的节点
    self:ReorderChildren()

    Logger.Debug(AppDef.LogTag.UI, "Panel activate finished, name: {0}, self: {1}", self:classname(), self:unique_id())
end
```

**核心：在lua中使用lua脚本panel来统一的管理所有的对象，gameObject只是其中很小的一部分，**

**重点全部在lua脚本panel上。这才是伟大的思想所在。**





#### page中元素在任何分辨率下始终居中显示：

针对于“战争游戏”、“战魂副本”page中的元素在不同分辨率下会出现上下或左右的偏移，为了保证页面中的元素始终显示在居中的位置：

**1.垂直方向：由于当前使用的是“UIPoolableHorizontalLayoutGroup”，所以对于垂直方向并不会因为content下多个item的出现而变化，**因此可以设置content的anchor在垂直方向上始终与屏幕分辨率保持一致

同时设置content下item的对齐方式为“Middle Left”：

如此纵然在page页面打开后切换不同分辨率，依然会在垂直方向上居中显示

**2.水平方向：由于content下会创建多个item，导致content的水平width出现变化，当前暂无法只通过改变UI来打到自适应的效果。**

方法：根据当前分辨率width来设置item的width，当显示page时，该page的chapterItem可以正好占满viewport，由于chapterItem内部的UI元素anchor设置——内部UI本身就是居中的

因此这样得到的结果就是：在显示某个page元素时正好可以占满viewport，page内部的UI也正好居中显示

代码：两个地方需要设置： 1.设置UIPoolableHorizontalLayoutGroup中protoTypePrefab的宽度，在"ForceRebuildItems"时会根据prototypePrefab的宽度来计算content的总宽度以及设置各个item的localposition

2.由于当前方法缺陷，在“OnItemEnabled”后依然需要重新设置item的宽度

PS：理论上来讲在设置UIPoolableHorizontalLayoutGroup中protoTypePrefab的宽度后，应该在UIPoolableHorizontalLayoutGroup中实例化item时会使用改变后的item的宽度。但这里没有







#### 直接设置Image的alpha值：

```lua
if self:IsFadeEnabled() then
    self.compImage:DOFade(1, kFadeDuration)
else
    GraphicUtils.SetAlpha(self.compImage.transform, 1)
end
```

```lua
--- @brief 设置目标以及孩子节点Alpha值
--- @param transform UnityEngine.RectTransform
--- @param value number
--- @public
function GraphicUtils.SetAlpha(transform, value)
    local graphic = transform:GetComponent(typeof(Graphic))
    if graphic then
        __SetAlpha(graphic, value)
    end
end
```

```lua
--- @brief 设置graphic透明度
--- @param graphic UnityEngine.UI.Graphic
--- @param value number
--- @private
local function __SetAlpha(graphic, value)
    local color = graphic.color
    color.a = value
    graphic.color = color
end
```

GraphicUtils这个脚本挺好用的





#### 当scrollRect中的元素较少时，希望元素的排列可以居中显示

但实际上是无法设置item的localPosition居中排列的，因为scrollRect的layout会自动将item排列。

**解决办法：**根据item的content总大小来设置scrollView的size。当item的总content较小，并且小于scrollView原本的width时，由于item默认是从左往右依次排列，因此item是无法居中的

**此时可以根据content的总大小来直接设置scrollView原本的width。**当设置scrollview的width时，**由于scrollview的pivot默认设置为(0.5，0.5)**，那么此时会默认居中显示所有的item





### 设置InputField的自动换行：

对于InputField的显示区域，设置“Line Type”即可让text超出单行界限时自动换行：

“Multi Line Submit”：当点击回车键结束input时，则会将当前输入的所有内容提交，此时input显示区域会自动定位到文本最上方，并且光标不再闪烁(代表结束edit)。下图中，当结束编辑后会自动跳转到文本最上方“qwqe.....”

“Multi Line Newline”：结束编辑后，光标依然闪烁，并且提交的仅仅只是最后一行的文本内容

以上点击回车结束编辑后，变成：

综上：通常都是选择“Multi Line Submit”的“Line Type”模式





#### 两个scrollRect重合在一起，如何禁止掉内部的scrollRect，但是拖动内部的item时会直接导致外部scrollRect出现拖动：

**“Scroll Rect”是控制列表的滑动，如果关闭“Scroll Rect”，仅仅只是关闭该列表的滑动功能，但是列表下其他item的排列等等是完全不受影响的。也就是说只保留scrollRect的item排列功能。**

当关闭内部的scrollRect时，那么内部列表下的items对于外部的scrollRect就只是一个普通的对象，当拖动内部scrollRect下的items时会导致外部的scrollRect被拖动。

也就是说当关闭了内部的scrollRect后，内部的item则可以直接拖动外部的scrollRect





#### 等待面板关闭后执行的操作：

**sq.facade:GetUIManager():WaitForPanel(panel)**

```lua
local panel = sq.facade:GetUIManager():CreatePanel("ArchiveGeneralDetailsDialog", {eId = entityId})
sq.facade:GetUIManager():WaitForPanel(panel)
archiveDrawingModule:DestroyArchiveDrawingGeneralEntity(generalId)
```





#### 针对UGUI中的Button可以重写，增加“Selected”状态：

可以自动对“Selected”状态做处理，这样不用在代码中再写这部分的逻辑，直接设置“Selected”参数的状态即可



#### 在代码执行中使用协程有效的避免一时的卡顿问题：

```
问题：当初始打开界面或者切换上下方的tab页签时，会出现卡顿的现象。并且当点击购买时，如果成功会弹出领奖界面，此时如果马上更新礼包列表，同样会有卡顿的问题
原因：当列表中item过多或者item内部细节较为复杂时，如果将”VerticalLayoutGroup“与其他UI元素同步绘制，会有卡顿的情况；同样，购买成功后的领奖界面如果和item细节较多的layoutGroup同步刷新，也会有卡顿的情况发生
方案：
优化一：初始时以为是使用”ButtonGroup“自动切换button时有一定的”OnButtonTabChanged“方法调用延时，加上每次切换tab后都需要重新向服务器发起协议获得最新数据。所以初始方案是不使用”ButtonGroup“，直接使用按钮监听。结果：还是一样的卡顿
优化二：使用协程异步执行：WaitForSeconds、WaitForFrames等，同时隐藏item中非必要UI
```

```
-- 刷新下方礼包列表
self:WaitForSeconds(kDelayCallForSecond)
self:SetVerticalLayout(itemCount)
```

```
--- @brief 购买成功后事件回调，刷新当前页面
--- @param event EventContext
--- @private
function LimitedTimeActivityWindow:OnShopBuySuccess(event)
    self:WaitForSeconds(kDelayCallForSecond2)
    self:UpdateCurrentGiftList()
end
```

从实际效果来看，使用协程确实有效减少了卡顿问题



#### "限时活动"面板卡顿问题的总结：

通过实际测试发现，当刷新item列表时会有明显的卡顿。说明卡顿的源头在于item的刷新。

**卡顿的实际表现有两个地方：**

**1.从主页点击按钮打开面板时会明显的卡一下，然后界面才会弹出**

原因：当从主页点击按钮打开“限时活动”面板时，由于UI底层设置会默认执行“OnOpen”方法，必须要把“限时活动”panel中的“OnOpen”方法执行完毕后，默认会返回“true” —— 如果在“OnOpen”中加入“协议获取数据”，当协议失败时会返回“false”关闭界面

从实际逻辑来看，只有在“OnOpen”方法完全执行完毕才会真的打开panel界面

所以如果在panel的“OnOpen”中执行过多的逻辑，尤其是“UpdateView”等，那么就会一直到“UpdateView”执行完毕才会弹出界面。这里就会出现明显的“点击主页按钮后卡顿一下，然后弹出界面”的情况发生

**方案：**

**治标方案：**在“UpdateView”中加入协程“WaitForFrames”，作用在于当调用“OnOpen”时可以“先交出CPU控制权”假定“OnOpen”执行完毕 —— 此时会弹出界面，然后再执行协程后的语句。这样就可以解决“当点击主页按钮后的卡顿时间”。

**缺点：**由于使用协程先弹出界面，后控制权回到协程再执行“UpdateView”，则panel会有明显的“空白间隔”

**治本方案：**根据伟大的UI底层逻辑，“OnOpen”只负责“页面的开启或关闭”， “OnRefresh”只负责“页面数据刷新”。“OnOpen”中的逻辑不要太复杂，绝对不要把“页面刷新”的逻辑加入到“OnOpen”中导致页面打开时间延长即“点击后卡顿”问题。

将“页面刷新”的逻辑只放入“OnRefresh”中。从UI底层设计来看，“OnOpen”执行完毕时页面已经打开，但是UI底层设计的“Open”逻辑中“OnOpen”只是很小的一部分。

由于在panel的“OnOpen”末尾通常会加入“self:Refresh”，查看“UIBasePanel”可知：

```
--- @brief 刷新界面
--- @public
function UIBasePanel:Refresh(...)
    return self:SafeCall(self.OnRefresh, ...)
end
```

“Refresh”本身已经加入了协程的“SafeCall”，与“OnOpen”已经“异步执行”，**因此“点击按钮后卡顿”已经解决。**

并且在“UIBasePanel”的“Open”中，在使用：

```
local ok, ret = self:SafeCall(self.OnOpen, data)
```

**后依然有大量决定panel初始显示效果的逻辑，这些逻辑与“OnRefresh”是异步执行的，即在设置panel初始显示动画的同时，“OnRefresh”也在同步绘制panel的页面数据。**

**由于两者异步执行，“卡顿”的问题直接就没有了，并且当“Open”中页面初始动画效果完毕后，“OnRefresh”中页面数据刷新通常也完成了。**所以看到的效果就是“页面在弹出来时数据也同步绘制成功”，**即“打开界面后的空白间隔”问题也解决了**。

UI底层设计很好的使用了协程，很精妙



**2.当切换顶部或底部的tab按钮时，会重新刷新item列表，此时会有明显的卡顿**

本质上来讲，如果item的prefab中默认有较多UI都处于true状态，那么在切换tab时会有卡顿；

另外，当在tab之间快速切换时，如果使用“layoutGroup:ClearItems()”，那么当item内部逻辑较为复杂时，在“ClearItems”和“ForceRebuildItems”之间会有明显的“空白间隔”，即清除所有item后和重新绘制item之间可以看到明显的时间间隔。

**针对这种情况，在快速切换tab时，可以省却“ClearItems”的操作：**

```
--- @brief 设置VerticalLayoutGroup
--- @private
function LimitedTimeActivityWindow:SetVerticalLayout(itemCount)
    local layoutGroup = self.content:GetComponent(typeof(UIPoolableVerticalLayoutGroup))
    --layoutGroup:ClearItems()
    self.content.anchoredPosition = Vector2.zero
    layoutGroup.ItemCount = itemCount
    layoutGroup:ForceRebuildItems()
end
```



**从实际效果看，即使使用以上两种方式，也不一定可以完全的解决“卡顿”问题。**

1.卡顿问题发生的根本在于创建item列表时，如果item内部逻辑或者prefab内容过多，是一定会造成额外的卡顿的。所以在平常设计item时需要注意这方面。

如何证明一定是因为item列表刷新导致的？

方式一：关闭layoutGroup设置：

```
-- 刷新下方礼包列表
--self:WaitForFrames(kDelayFrames)
--self:SetVerticalLayout(itemCount)
```

结果：当注释掉“self:SetVerticalLayout”后，发现页面可以很快的打开，没有任何卡顿

方式二：当把itemCount设置成1，或者关闭“OnItemEnabled”中“item:UpdateView”时，卡顿情况基本没有了

```
function LimitedTimeActivityWindow:SetVerticalLayout(itemCount)
    local layoutGroup = self.content:GetComponent(typeof(UIPoolableVerticalLayoutGroup))
    --layoutGroup:ClearItems()
    self.content.anchoredPosition = Vector2.zero
    layoutGroup.ItemCount = 1
    layoutGroup:ForceRebuildItems()
end
```

```
function LimitedTimeActivityWindow:OnTaskItemEnabled(panel, index)
    local currentGoodsType = self:GetCurrentGoodsType()
    panel:Open({ goodsType = currentGoodsType, idx = index + 1 })
    --panel:UpdateView()

    self.verticalScrollAnimController:PlayItemAnim(panel, index)
    return panel
end
```

由此可见，造成卡顿的主要问题在于item内部的“UpdateView”逻辑较多，但如果itemCount = 1时，又完全没有卡顿的情况。



2.同时对于item的初始创建，对于prefab中暂时非必要的UI可以选择隐藏

3.另外，虽然UI Button Group内部支持对button的切换选中，但如果切换后还需要重新发送协议获取最新数据再刷新等对时效性要求很高的情况，则为了减少切换间的等待时间，不适合使用“UI Button Group”来处理，建议单独使用button来改变选中状态

4.通过实际测试发现，纵然将“OnOpen”和“OnRefresh”功能单独分开，如果item内部逻辑复杂，也依然无法起到缓解卡顿的效果：

所以为了解决“点击主页按钮后要停一会再打开界面”的问题，在设置刷新列表“self:SetVerticalLayout(itemCount)”前加入协程，这样界面会先被打开，之后才会开始item列表的刷新

```
-- 刷新下方礼包列表
self:WaitForFrames(kDelayFrames)
self:SetVerticalLayout(itemCount)
```

这样的实际效果就是：打开界面，过一会才会出现item列表；与点击后卡顿相比，较为容易接受一点



#### **“self:GetContainer()”的使用方式：**

通过在lua脚本A中使用“self:AddComponent("B")”即可建立Lua脚本A和B的联系，然后在脚本B中即可以通过“self:GetContainer()”来调用脚本A中的方法



#### 







**2.直接使用“and”来判断连接即可：**

当前面“self.allSkills[sType]”为空时，直接赋新值

**3.设置默认值，根据条件返回默认值或者变化后的值：**

```
public override bool Validate(LogicBattleStateEntity stateEntity) {
    var ret = true;
    ret &= ValidateTeamMember();
    return ret;
}
```

首先设置默认值“var ret = true”，然后将该ret与具体情况返回的值做“与”操作，即可得到最后返回值

PS：如果仅仅只是“bool”返回值，则完全无需该操作。但如果是“int”返回值，则需要将“ret”与目标值进行“与”运算，方法的最终返回值就会不一样

















## 要查看的地方：

2.UIText中有个封装好的“SetLocalizeId”方法，这个挺好用的

3.获取自身以及子对象中所有的graphic组件：

```
public static void SetGraphicEnableInChildren(this Transform transform, bool value) {
    var results = ListPool<Graphic>.Shared.Get();
    transform.GetComponentsInChildren(results);

    foreach (var graphic in results) {
        if (graphic.enabled == value) {
            continue;
        }
        graphic.enabled = value;
    }
    ListPool<Graphic>.Shared.Return(results);
}
```

将text，image等任何继承于graphic的组件都false/true，该方法很适用

在“TransformExtension.cs”中可以看到

***<u>4.在lua某个脚本开始的地方使用“@alias”真的可以直接就一个脚本使用多个其他脚本的内容吗？</u>***有这么神奇？这个后面要仔细研究下：

这里连“AddComponent”都不用了，是不是太方便了，里面具体的逻辑要仔细看下



6.C#中的”vFrame.Core.Events“中实现的”EventDispatcher.cs“很有用，在很多地方都适用，可以仔细看下。这个和lua中的”EventDispatcher.lua“是不一样的，但实现的原理上基本一致

7.将多个变量拼凑成一个json字符串的形式：

8.支持stencil buffer的shader: "UI/Particles/Additive"，查看一下具体内容，为什么和博客上原来自带的shader效果不一致

另外对于常用的遮罩“soft mask”，后来支持了对“UI Text Outline”的裁切，这个要看下使用的shader

然后：很重要的是： 某些特效使用支持stencil buffer的“UI/Particles/Additive”依然会出现与其他UI穿透的情况，**但是把粒子上改成“UI Particle System”后就没问题了**，这个要仔细查看下“UI Particle System”的实现逻辑，为什么可以避免UI和特效之间的穿透问题，以及将粒子上的Render改成“UI Particle System”后依然可以正常显示粒子特效，并且还避免了和UI的穿透问题，这个要看下实现细节























