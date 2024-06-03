[TOC]



##### 1.GUI.Enabled

要做出某个GUI按钮或控件不可点击的情况可以使用：GUI.Enabled,  可以使用该参数来控制控件的显示情况。

 设置GUI控件的enabled状态，enabled属性并不会改变该GUI控件是否会被绘制在Game视图中，即纵然GUI.enabled = false, OnGUI绘制的控件依然会显示出来。

 “GUI.enabled = false”时只会使UI控件不可交互，并且显示为灰色

并且可以**<font color=blue>在相邻的GUI控件间分别使用“GUI.Enabled = true”和“GUI.Enabled = false”来区分每个GUI控件的“不可交互”状态</font>**



##### 2.GUI.Changed

用于检测OnGUI中任何的变化，如TextField、InputField、slider等。当这些控件数值发生变化时，GUI.Changed会自动变成“true”



##### 3.GUI.depth

当在同一位置上绘制多个GUI控件时，可以通过设置GUI控件的“GUI.depth”来确定控件的渲染顺序。

若在多个脚本中都有GUI控件的绘制，**在每个脚本中设置该脚本中GUI元素的GUI.depth，如此当多个脚本在同一位置绘制GUI元素时可以根据depth来决定控件之间的显示效果。**

**depth值越小，越显示在前面。(可以把该depth值当做该GUI控件距离camera的远近)**

只需要该这些GUI元素分开在不同的脚本中即可，两个脚本可以挂在同一个对象上，由GUI.depth来决定控件间的先后顺序



##### 4.GUISkin和GUIStyle

GUISkin是多个GUIStyle的合集，里面包含各个UI控件的style，具体使用时需要使用"skin.FindStyle(styleName)"找到button，label等控件的目标style

如若没有在Assets中创建新的GUISkin，而直接使用unity自带的style，则 "GUI.skin.label".

```c#
//默认使用的是unity自带的Button的style
GUI.Button(new Rect(0, 0, 100, 30), "BUTTON");   
//则是用skin中的"label"的style显示这个Button
GUI.Button(new Rect(0, 0, 100, 30), "BUTTON", skin.FindStyle("label"));  
```

可以反复的调整GUI的skin是否显示，如：

```c#
public GUISkin customSkin;

void OnGUI(){
     GUI.skin = customSkin;  //使用自定以的skin
     GUI.Button(new Rect(0, 0, 100, 30), "BUTTON");
     GUI.skin = null; //使用unity UI自带的label GUIStyle
     GUI.Lable(new Rect(0, 0, 100, 30), "LABEL");
     GUI.skin = customSkin;
     GUI.Box(new Rect(0, 0, 100, 50), "this is a text file.");
}
```



###### 设置GUI控件颜色：

 改变背景颜色：GUI.backgroundColor

 改变内容颜色： GUI.contentColor

 改变内容和背景颜色： GUI.color



##### 5.OnGUI每帧执行次数研究：

为何OnGUI会至少执行两次？

一次为Layout布局，用于记录下各个控件的位置，大小等信息；另一次为Repaint，即将这些需要显示的控件绘制出来。

```c#
void OnGUI(){
    if(Input.GetMouseButtonDown(0))
    //当鼠标点击时，该语句会输出至少两次
        Debug.Log("MouseButton is clicked......");    
    if(Input.GetKeyDown(KeyCode.W))
        //按下W键时，该语句输出四次
        Debug.Log("Keycode.W is pressed......");      
    
    //以上语句如果换成“GetMouseButton”， “GetKey”，则会输出更多的次数

    //当点击GUI BUTTON时，该语句只会输出一次
    if(GUI.Button(new Rect(100, 100, 100, 30), "BUTTON"))
        Debug.Log("GUI Button is clicked....");       
}
```

**在OnGUI中用于检测点击事件时，通常只用GUI控件来作为检测标志，而不能用鼠标或按键等非GUI控件的标志。** Input.GetKeyDown(KeyCode.UpArrow) 是指键盘的方向键的上键，KeyCode.DownArrow是下键

至于为何会至少执行两次，使用Input.GetKeyDown时执行了四次，其原因暂时不知道。

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107073603172.png" alt="image-20231107073603172" style="zoom: 50%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107073645242.png" alt="image-20231107073645242" style="zoom: 50%;" />



##### 6.为何在GUI.Button条件下无法绘制新的GUI控件？

```c#
void OnGUI(){
   if(GUI.Button(new Rect(100, 100, 100, 50), "CLICK"){
       Debug.Log("Button is clicked.");
       GUI.Label(new Rect(100, 200, 200, 50), "SHOW THE LABEL");
   }
}
```

由于GUI.Button的内在机制，当点击button后只会响应一次，类似于Update中的”Input.GetMouseButtonDown"。因此以上的“GUI.Label”实际上绘制了一次，但很快会被下一个轮回刷新掉，而在下一个轮回中，GUI.Button并没有被点击——两个轮回间隔极短，且鼠标在按下后并没有松开。**从实际测试中可知：GUI.Button条件下的语句只有在鼠标按下并****松开后****才会执行，****按下的过程中语句并不会执行**

类似于

```c#
bool flag = GUI.Button(new Rect(100, 100, 100, 50), "CLICK");
if(flag){
     GUI.Label(new Rect(100, 200, 200, 50), "SHOW THE LABEL");
}
```

由于每一次的flag都是新声明的局部变量，而默认没有点击button时，flag始终为false，因此”GUI.Label“不会被绘制。

**注意点：**

```c#
bool flag = false;
void OnGUI(){
   flag = GUI.Button(new Rect(100, 100, 100, 50), "CLICK");
   if(flag){
     GUI.Label(new Rect(100, 200, 200, 50), "SHOW THE LABEL");
}
```

使用此语句依然会得到相同的效果——无法绘制出GUI.Label。

原因：当鼠标按下Button并松开后，flag被赋值为true，GUI.Label被绘制。但马上到下一OnGUI轮回，此时GUI.Button并没有被按下，flag又马上被重置为false，因此GUI.Label在本轮回中又无法被绘制。两个轮回间隔极短，呈现在Game视图中则GUI.Label没有显示出来

但是如若是以下的情况：

```c#
bool flag = false;
void OnGUI(){
    if (GUI.Button(new Rect(100, 100, 100, 50), "CLICK"))
    {
        flag = true;
        Debug.Log("CLICK......");
    }
    if (flag)
    {
        GUI.Label(new Rect(100, 200, 200, 50), "SHOW THE LABEL");
    }
}
```

以上情况下，当点击GUI.Button后，flag赋值为true并始终保留该状态，因此GUI.Label可以显示出来



###### 使用GUI.RepeatButton实现“点击后显示某些GUI控件”的效果：

```c#
void OnGUI(){
     if(GUI.RepeatButton(new Rect(100, 100, 100, 50), "CLICK"))
   {
        Debug.Log("CLICK.......");
        GUI.Label(new Rect(100, 150, 200, 50), "SHOW THE LABEL");
   }
}
```

**效果：当点击Button并保持按下状态，则在该条件下的GUI.Label可以显示出来。当松开鼠标后，GUI.Label消失**



###### GUIUtility.RotateAroundPivot/ScaleAroundPivot

**GUIUtility.RotateAroundPivot**：可以将接下来显示的GUI控件中心点进行旋转

  注意：1.该语句执行过后只会对之后的GUI控件起作用，对该语句之前已经绘制完成的GUI控件没有影响

​    2.如果需要之后绘制的GUI控件又正常的pivot，则需要再次调用该语句重新旋转。

​    并且旋转的基准是在紧邻着的上一次角度的基础上进行旋转，而不是最开始的基准。

​    即：GUIUtility.RotateAroundPivot(90, pivot);后再调用GUIUtility.RotateAroundPivot(90, pivot);则会得到相对于初始点180度的旋转朝向

GUI.color = new Color()：和以上一样会对接下来所有绘制的GUI控件有影响，但对之前的GUI控件则不会。

无论是以上哪个方法都只对之后的GUI控件起作用。因此若在绘制完成GUI控件后再调用以上方法，是不会有效果的

```c#
public Texture imagePressed;
    void OnGUI()
    {
        Rect rectPressed = new Rect(200, 100, 200, 200);
        Vector2 pivot = new Vector2(rectPressed.xMin + rectPressed.width * 0.5f,  rectPressed.yMin + rectPressed.height * 0.5f);
        //为了使GUI.RepeatButton绘制的button为透明状态
        GUI.color = new Color(255, 255, 255, 0);  
        if (GUI.RepeatButton(rectPressed, "CLICK")) 
        {
            Debug.Log("PRESS THE IMAGE");
            //旋转接下来GUI控件的pivot
            GUIUtility.RotateAroundPivot(90, pivot); 
            //重置color，使得之后GUI控件能够正常显示出来
            GUI.color = new Color(255, 255, 255, 255); 
            //拉伸之后GUI控件
            GUIUtility.ScaleAroundPivot(Vector2.one * 2, pivot); 
            GUI.DrawTexture(rectPressed, imagePressed);
        }
    }
```



##### 7.GUI.DragWindow

当需要创建可拖动窗口时可使用本方法

```c#
Rect windowRect = new Rect(100, 100, 200, 200);
void OnGUI()
{
   windowRect = GUI.Window(1, windowRect, DragWindowCallback, "WINDOW AREA");
}
void DragWindowCallback(int id)
{
    GUI.DragWindow();
}
```

解析：

**1.windowRect需要是固定的区域，不能在OnGUI中每次刷新时都得到新的不同的引用类型变量——虽然名字依然相同但却是不同的变量**

**因此在OnGUI外定义全局变量**

2.GUI.Window方法的返回值为Rect类型。需要在GUI.Window左侧使用相同的"windowRect"进行承接，**否则该Rect区域是无法拖动的**

3.**GUI.DragWindow方法只有在”GUI.Window“的callback中才可执行，无法在OnGUI中直接调用。**

GUI.DragWindow方法中可以自主设定可拖动区域，经过实测发现：

**当可拖动区域大小>=windowRect的大小时**，通常可以自由拖动窗口到屏幕上任何位置；

**当可拖动区域<windowRect的大小时**，逐渐出现拖动不灵便的情况，随着可拖动区域逐渐变得非常小时，窗口最终完全无法拖动

这里提及的”可拖动区域“的大小

```c#
GUI.DragWindow(new Rect(0, 0, width, height));
```

主要指的是由width, height规划的矩形区域，与前两个偏移参数基本无关。

以上结论是在当时的实测结果得来，有可能会有遗漏。但仍具有一定指导意义。



##### 8.GUI.FocusControl

```c#
public class GUIControl : MonoBehaviour
{
   private string username="Username" , pwd="Pwd";
   void OnGUI()
    {
       GUI.SetNextControlName("focus" );
       //在使用GUI.FocusControl()前需要设置该control的name，要注意这里与GUI或者GUILayout无关，所以下面两种形式都是可以的
        username = GUILayout.TextField(username);
       //username = GUI.TextField(new Rect(100,50,100,30),username);

        GUI.SetNextControlName("focus1" );
        pwd = GUI.TextField(new Rect(100,100,100,30),pwd);

        if (GUI .Button(new Rect(100,200,100,30), "move the focus"))
        {
           //当点击按钮后就把当前的GUI 页面的焦点转移到了目标focus的control处
           GUI.FocusControl("focus" );    
        }
       
        if(GUI .Button(new Rect(100,250,200,30), "move the foucs to focus1" ))
        {
           GUI.FocusControl("focus1" );
        }
    }
}
```

如果是focus到某个窗口则可以使用GUI.FocusWindow(0);

此时不需要先调用**GUI****.SetNextControlName(****"focus"** **);，因为GUI.Window方法中已经**为每个window设置了id

PS：

1.GUI.FocusControl只针对“具有Interactable特性”的控件有效，因此“Label控件”无法使用本方法聚焦

2.当需要在“显示效果”上明确的区分某个控件在“聚焦和点击”等状态时，可使用“GUISkin”，其内部可自由设置该控件在“Normal、Hover、Active”等状态的颜色

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107075651225.png" alt="image-20231107075651225" style="zoom: 67%;" />



##### 9.GUI.tooltip

仅是一个“string类型”的全部变量，可以自由设置每个GUI控件的“tooltip”。当需要显示该tooltip时，则使用“GUI.Label”等控件将其显示出来即可：如果GUI.tooltip为空，则自然该GUI.Label不会显示出来

**"GUI.tooltip"的赋值实际上在控件绘制时就已经注定：“new GUIContent(buttonStr, tooltipStr)”，当鼠标hover在控件上方时，OnGUI会自动的获取本控件的tooltip，将其赋值给全局变量“GUI.tooltip”。因此不需要额外的为该“GUI.tooltip”变量赋值**

**当鼠标在按钮上，系统会自动的给GUI.tooltip赋值；当离开按钮时则GUI.tooltip会被自动 GUI.tooltip = ""**

  **整个OnGUI中，“GUI.tooltip”是全局变量，会随着鼠标当时的位置而实时改变**



##### 10.GUI.HorizontalSlider

使用GUI绘制slider时，需要在左边放置一个变量进行承接：

volumeValue = GUI.HorizontalSlider( new Rect (100,100,200,30),volumeValue,0,1f);  这样Game视图中的slider才可以滑动

```c#
float num = 0;
void OnGUI()
{
    GUI.HorizontalSlider(new Rect(100, 100, 300, 50), num, 0, 100);  //若只有这种写法，则只会在Game视图中显示slider滑动条，但滑动条不可拖动
    /************* 为何不可拖动 **************/
    /* “GUI.HorizontalSlider”是一种UI表现形式，其参数结构决定了该控件的作用是用UI的形式将“num”的值表现出来，并不会直接的改变目标字段的数值。
       因此当“num = 50f”时，“GUI.HorizontalSlider”则会相应处在slider 50刻度的位置。
       假如有另一外部方法——逐步改变num的值或来回pingpong的改变num，“OnGUI”中的“GUI.HorizontalSlider”也会相应的显示当时num值对应的slider刻度
       不能拖动的原因：如果只是自由拖动，作为slider本身是完全支持这个功能的，但问题在于“OnGUI”刷新帧率太快，每次手动拖动后又因为
       GUI.HorizontalSlider自身的功能原因，slider刻度被重置为num对应的数值。因此最终显示出来的效果就是slider无法拖动。
       假设“OnGUI”刷新帧率每10s一次，则可以在“Game”视图中看到“可以自由拖动slider，然而每10s之后slider又被重新归置为num数值对应的刻度”
     */


    num = GUI.HorizontalSlider(new Rect(100, 100, 300, 50), num, 0, 100); 
    /* 惊喜的是“GUI.HorizontalSlider”本身提供了返回值，当拖动slider后立即将拖动后改变的数值重新赋值给num，此时num本身已经发生了改变，因
       此“OnGUI”刷新时是按照改变后的num数值进行显示的。所以会得到可以自由拖动slider的效果
     */
}
```



##### 支持Selectable的控件：

支持Selectable的控件具有“交互功能”，其衍生类有：Button, Toggle, InputField, Scrollbar, Slide,Dropdown



##### UI上的点击检测：

UI上的Raycast Target属性的作用：当鼠标或Input.touch 时，从画布canvas上的Graphic Raycaster发一条ray进行检测，若UI上该属性为true，则点击时会有响应（若有多个UI在同一位置，则点击会被第一个截获），为false则点击UI无效

该属性主要在UI和3D场景中的collider时，用于区分：**Event.current.IsPointerOverGameObject()  —— 也可以添加参数**

当结果为true时代表点击到了UI上

RaycastResult是针对UI上的射线检测结果，发射点是screenPosition，三维平面则直接是Ray

UGUI中自带的Button预制体包含有image和子对象text，但子对象上的点击并不会被block，只有在独立的两个UI对象被在重叠区域点击时会block。

**对于会被block的UI可为其添加Canvas group，并设置Block Raycasts为false**

**这样后果是该对象自身不会响应raycast点击，会把点击渗透到下一层**

UI中的**Event Trigger组件**可以响应UI上的多种自定义事件

```c#
using UnityEngine.EventSystems;
using UnityEngine.UI;

//方法一 使用该方法的另一个重载方法，使用时给该方法传递一个整形参数
// 该参数即使触摸手势的 id
// int id = Input.GetTouch(0).fingerId;
public bool IsPointerOverUIObject(int fingerID) {
return EventSystem.current.IsPointerOverGameObject(fingerID);
}

//方法二 通过UI事件发射射线
//是 2D UI 的位置，非 3D 位置
public bool IsPointerOverUIObject(Vector2 screenPosition) {
//实例化点击事件
PointerEventData eventDataCurrentPosition = new PointerEventData(EventSystem.current); //将点击位置的屏幕坐标赋值给点击事件
eventDataCurrentPosition.position = new Vector2(screenPosition.x, screenPosition.y);
List<RaycastResult> results = new List<RaycastResult>();
//向点击处发射射线
EventSystem.current.RaycastAll(eventDataCurrentPosition, results);
return results.Count > 0;
}

//方法三 通过画布上的 GraphicRaycaster 组件发射射线
public bool IsPointerOverUIObject(Canvas canvas, Vector2 screenPosition) {
//实例化点击事件
PointerEventData eventDataCurrentPosition = new PointerEventData(EventSystem.current); //将点击位置的屏幕坐标赋值给点击事件
eventDataCurrentPosition.position = screenPosition;
//获取画布上的 GraphicRaycaster 组件 GraphicRaycaster uiRaycaster = canvas.gameObject.GetComponent<GraphicRaycaster>();
List<RaycastResult> results = new List<RaycastResult>();
// GraphicRaycaster 发射射线
uiRaycaster.Raycast(eventDataCurrentPosition, results);
return results.Count > 0;
}
```

