[TOC]



##### SetParent与“xx.parent”：

当需要设置某个GameObject的parent时通常使用“SetParent(xx, false)”，不要直接通过该“GameObject.parent”属性来设置(部分情况下，使用parent属性会报异常)。并且“SetParent(xx, false)”中第二个参数通常为“false”



##### UGUI中设置各个对象的渲染顺序：

rectTransform.SetAsFirstSibling/SetAsLastSibling：设置该对象为第一个或最后一个渲染的对象

rectTransform.SetSiblingIndex：设置该对象按指定index进行渲染



##### [RuntimeInitializeOnLoadMethod] 标识

当需要游戏启动后自动执行某个static方法时，可在该方法前添加[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]标识，则该方法会自动被调用。



##### 从“包含Assets文件夹前缀的资源路径”获取到该资源的GUID：`AssetDatabase.AssetPathToGUID(assetName)`

从“GUID”获取到该资源的路径：`AssetDatabase.GUIDToAssetPath(string guid)`

###### GUID和InstanceID的区别：

```c#
string path = AssetDatabase.GetAssetPath(Selection.activeObject);
string guid = AssetDatabase.AssetPathToGUID(path);
Debug.Log(guid + "      <color=blue>" + Selection.activeInstanceID + "</color>");
```

当在Project视图中选择任意一个object后，出现类似如下的输出结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107225044228.png" alt="image-20231107225044228" style="zoom:80%;" />

**InstanceID**：在unity引擎内部针对所有的实例分别设置ID，包括Editor模式和Playmode模式下所有能够从Project视图，Hierarchy视图中获取到的所有实例Object。通常都是int类型，并且范围不会太大 —— 因为Unity引擎中的实例对象并不会无限制增长，取决于游戏开发中的项目复杂程度

**GUID**: 针对于目标物体在不同的操作系统中所对应的ID，相当于一个普通的File文件在操作系统中分配的ID，不同的操作系统如windows，mac中的文件管理系统是不同的，ID设置规则也是不同的。

**获取物体InstanceID的方法**：`this.gameObject.GetInstanceID()`

**由InstanceID转换成GameObject**：`EditorUtility.InstanceIDToObject(int id)`



##### transform的rotate和rotation的区别：

###### transform.Rotate: 

旋转多少度——是相较于之前的rotation的变化值，不论里面参数使用哪种重载组合，代表的都是相较上次的变化值。有的重载组合中会有space.self/space.world，如果GameObject本身没有父对象，那其实不论选择space.self/world，效果都是一样的。若该GameObject有父对象，则会存在local/world之别。所谓的space.self/world，具体的实际表现来看，就是是否有父对象之别。transform.Rotate 和 transform.Translate 都表示相较于上次的变化值，而非最终值。transform.Rotate(axis, angle):指绕着axis旋转angle度。若为绕着Y轴旋转，则表现形式就是物体在XOZ平面转动，rotation中只有Y数值会变动

###### transform.rotation: 

旋转到多少度——是最终目标值，跟inspector中显示的数值一样。

如：transform.rotation = Quaternion.Euler(x, y, z); 表示该GameObject的rotation就是（x, y, z）

若要达到插值的渐变效果：

```c#
float smooth;
Quaternion targetRotation;
void Update(){      
    transform.rotation = Quaternion.Euler(transform.rotation, targetRotation, Time.deltaTime * smooth);
}
```



Color.clear —— 完全透明； new Color(1,1,1,1); —— 白色，new Color(0, 0, 0, 0) 则为黑色。

**黑色代表没有任何的r, g, b分量，完全没有颜色，因此(0, 0, 0, 0)是最能表达的**



“\n”代表“换行”，“\r”代表“回车”，“\t”代表下一个制表位置

```c#
Debug.Log("第一行\r\n第二行");
Debug.Log("制表位置1\t制表位置2");
Debug.Log("第三行\n第四行");
Debug.Log("第五行\r第六行");
```

执行结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107173619738.png" alt="image-20231107173619738" style="zoom:80%;" />



while和do......while的区别：前者当条件不满足时一次都不会执行，后者则是先执行后判断条件是否满足，因此即使条件不满足都会执行一次

  for(;;)循环中出现这样的情况时会陷入死循环，因为都没有指定限定的条件。



**中文汉字占用的字节数：**

**在不同的编码标准下，中文字符占有2-4个字节。**

**在GBK编码中，中文汉字通常占2个字节；****一个汉字=一个汉字标点=两个英文字符=两个字节=16位，一个英文标点全角时16位，半角时8位**

**在UTF-8编码中，由于UTF-8为可变长编码，通常占3个字节，扩展B区后则占4个字节**



在读取某些文件路径时，通常需要在路径前加上"@"，作用是告诉编译器：后面string中出现的某些特殊字符如'\', '/','\n'等是作为元字符存在，而不是转义字符



C语言中`*.h`和`*.cpp`文件的区别：

.cpp是源文件，.h是头文件，前者需要编译后使用，后者不需要编译，且后者主要用于存储某些常用变量或函数的声明，不能包含该对象的定义，其作用就相当于简单的文本替换的作用，在.cpp中，预编译阶段会把“#include "math.h"”这部分进行文本替换，即把math.h的所有内容都包含进来，但这个是不需要编译的。



##### UnityEvent在游戏中的使用：

“UnityEvent类型”源自“UnityEngine.Events”命名空间，支持序列化。当在代码中声明“public UnityEvent responses”时，即可在Inspector面板中得到如下效果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231108120148995.png" alt="image-20231108120148995" style="zoom:80%;" />

###### UnityEvent的泛型扩展：

UnityEvent类型本身并不支持传递参数，所以为了方便使用，可基于UnityEvent创建新类型：

```c#
[System.Serializable]
public class CustomEvent : UnityEvent<string, int> {}
```

当在代码中声明“`public CustomEvent myEvents`”时，即可在Inspector面板中显示如下效果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231108120923713.png" alt="image-20231108120923713" style="zoom:80%;" />

**注意**：在新类型“CustomEvent”前必须**<font color=red>添加“System.Serializable”</font>**，否则无法序列化

###### UnityEvent添加监听：

**直接在Inspector面板中添加监听**：使用这种方式添加的监听称为“Persistent Listener”，无法在代码中移除该监听，但可以通过GetPersistentEventCount、GetPersistentMethodName、GetPersistentTarget获取该监听的相关信息，并调用SetPersistentListenerState方法关闭该监听

**在代码中通过“AddListener”添加监听**：该方式添加的监听称为“Non persistent listener”。可以在代码中通过RemoveListener移除监听。

**注意**：

1).当使用“AddListener”方法添加监听时，其不会检测该物体上是否已经添加了“同名监听”，因此很有可能会重复添加“同一个监听”，在触发时该监听也会被触发多次。因此当不需要该监听时需要使用“RemoveListener”及时移除物体上的监听

2).RemoveAllListener和RemoveListener仅针对“使用代码添加的监听”，对于“在Inspector面板中添加的监听”，则无法移除



##### Process.Start的使用：

在C#中使用Process.Start指定应用程序打开某个文件：

```c#
using System.Dialognastics;

Process.Start(@"d:\Downloads\阳光灿烂的日子.pdf");
```

若没有指定应用程序，则操作系统会使用默认的文件打开程序来打开该文件

```c#
Process.Start(@"www.baidu.com");
```

则会使用默认的浏览器打开以上网址

###### Process.Start中经常遇到的问题：

当使用某个指定的应用程序exe文件打开某个文件时，通常会出现：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107173935355.png" alt="image-20231107173935355" style="zoom: 67%;" />

表示找不到目标路径的exe应用程序。此时的原因一般是目标exe程序的路径不对，并不在于左斜线或右斜线的问题，而是在找寻exe程序的绝对路径时出现空格的缺失：

如使用如下方式查找到的notepad++.exe程序的路径：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107174109688.png" alt="image-20231107174109688" style="zoom:58%;" />

将此路径作为代码中使用的路径，前面加“@”，则会出现以上无法找到目标应用程序的问题。

而倘若在文件资源管理器”Explorer.exe“中

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107174154148.png" alt="image-20231107174154148" style="zoom:80%;" />

使用以上路径则可以正常运行。

**两个路径比较之后发现，后者在”****Program Files (x86)“上保留了磁盘中的空格，前者没有。**

**总结：目标exe程序的路径以文件资源管理器中的路径为唯一依据**

例子如下：

```c#
void Update()
{
    if (Input.GetKeyDown(KeyCode.W))
    {
        string path = Application.streamingAssetsPath + "/c.txt";
        Debug.Log(File.Exists(path));

        System.Diagnostics.Process process = new System.Diagnostics.Process();
        System.Diagnostics.ProcessStartInfo startInfo = new  System.Diagnostics.ProcessStartInfo()
        {
            WindowStyle = System.Diagnostics.ProcessWindowStyle.Hidden,
            //使用文件属性得到的exe文件路径中会省略掉部分文件夹的空格，导致无法查找该文件
            //FileName = @"C:\Program Files(x86)\Notepad++\notepad++.exe",
            //直接使用文件资源管理器Explorer.exe定位到该exe应用程序，使用磁盘中的路径
            FileName = "C:/Program Files (x86)/Notepad++/notepad++.exe",
            Arguments = path
        };
        process.StartInfo = startInfo;
        process.Start();
    }
}
```



##### CRC,MD5,SHA1三种不同的验证算法：

CRC,MD5,SHA1是不同的算法，但是这三种主要用于数据校验或安全或加密时使用。算法的复杂度依次提升，安全性越来越高，但同时效率也会降低，因为同时加密和解密时占用的资源也会增多。CRC一般用于通信类数据校验，检测数据在传输途中断掉的情况。MD5和SHA1则用于安全性方面，SHA1安全性最高



##### 计算游戏当前运行帧率

帧率实际代表“每秒内Update执行的次数”，因此可直接在Update中统计数值，当满足“1秒”时，该数值则代表当前“设备运行帧率”(应该就是这样的原理，但也可以查看下“GameFramework工程中的实现过程”来确认)



##### GameObject.Find与transform.Find

GameObject.Find：当该物体在Hierarchy中不可见时(不论是自身还是其父物体的activeSelf = false导致的)，本方法获取到的对象为null；只有该物体可见时，才能查找到该物体

transform.Find：在本物体的子对象中查找目标物体。如果需要查找“多个子层级”下的物体，则在“路径参数”中添加“/”间隔即可，如transform.Find("Cube1/Cube2")。本方法支持查找“隐藏对象”，即使该物体activeSelf为false也能查找到



##### GetComponentsInChildren

该方法会获取本对象以及其子孙对象上所有满足要求的组件。如果本对象自身包含该组件，则会包含在集合中，并且会作为集合中的第一个元素。所以在部分特殊情况下需要额外排除下自身

**注意**：当**孩子对象GameObject为false状态**时，返回的集合中是不会包含该孩子物体的；而倘若孩子对象GameObject为true状态，但是目标组件为false状态，此时依然会包含该孩子物体。

因此若要即使该孩子GameObject本身为false状态，结果中依然要包含该孩子的目标组件MeshFilter，则需使用：`MeshFilter[] mrs = this.GetComponentsInChildren<MeshFilter>(true)`

**验证如下**：

```c#
MeshFilter[] mfs = this.GetComponentsInChildren<MeshFilter>();
foreach(var temp in mfs)
{
     Debug.Log(temp.gameObject.name + "      NAME    ");
}
```

脚本挂载在父物体“Cube”上。当父子物体结构如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107224118093.png" alt="image-20231107224118093" style="zoom:80%;" />

执行结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107224134058.png" alt="image-20231107224134058" style="zoom:80%;" />



##### Unity中的Input输入设定：

当需要自定义按钮的触发按键时，则可在ProjectSetting的InputManager中设定，如要使用Input.GetButtonDown("Fire2")，则需要在InputManager中设定“Fire2”按钮的触发按键，否则在代码中无法被触发

注意：GetKeyDown、GetButtonDown、GetMouseButtonDown 与 GetKeyUp、GetButtonUp、GetMouseButtonUp是对应的，在一次完整的对应中只会执行一次

但GetKey、GetButton、GetMouseButton则会每帧都执行。



##### Destroy和DestroyImmediate的区别：

```c#
public class OthersFunc : MonoBehaviour
{
    GameObject go;
    bool flag = false;
    int count;
    private void Start()
    {
        go = GameObject.CreatePrimitive(PrimitiveType.Capsule);
        count = this.gameObject.transform.childCount;
    }
    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Backspace))
        {
            //destroy不会马上释放内存，只有在该帧末尾时把go置为null，具体的释放内存由系统GC调用
            Destroy(go);
            //执行该语句后，go马上被置为null
            //DestroyImmediate(go);
            flag = true;
        }
        if (flag)
        {
            if (go)
                Debug.Log("go is still real.");
            else
                Debug.Log("go is gone.");
        }
        //利用Destroy和DestroyImmediate特性处理如下问题：
        for(int i = 0; i < count; ++i)
        {
            Destroy(this.transform.GetChild(i));
        }
        //由于Destroy特性只有在本帧末尾才会将目标对象置为null，因此在删除掉“0”号子child后，“1”号
        //子child还是原来队列中的对象，所以可以正常的destroy所有的子对象。
        //此与List中RemoveAt(int index)删除掉index为0号元素后，1号元素会默认自动靠前顶替，
        //因此需要始终删除0号元素才可。
        //鉴于Destroy的特性，以上方法在destroy所有子对象的要求上是满足的，并不会出现删除了错乱的情况
    }
}
```

DestroyImmediate会马上销毁对象，由于unity是单线程，并且调用GC.collect()，虽然释放了内存占用，但占用了cpu资源，影响主线程的执行，其实是得不偿失的。

Destroy会在本update帧末尾才将实例对象置为null，GC会在需要释放内存时自动调用，无法知道具体的时间，其会搜索内存中没有被引用的资源释放。

鉴于引用对象的特性，实例本身会存储在堆中，对该实例的引用存储在栈中，将实例对象置为null后，栈中的空间应该会释放，堆中的空间占用需要等GC来释放。

两者其实都会释放内存占用，只是DestroyImmediate会多些。

在实际使用中，尽量避免使用DestroyImmediate

使用Destroy可以避免频繁对内存的读写操作。**回收器会定时清理一次内存中引用计数为0的对象**，很可能你要销毁的对象在其他地方还有引用而你自己不清楚，直接销毁就可能导致其他地方空引用错误。

```c#
void Start(){
    Destroy(this);
    Debug.Log("<COLOR=YELLOW> START: DESTROY </COLOR>");
    StartCoroutine(EnuFunc());
}

IEnumerator EnuFunc()
{
    Debug.Log("Coroutine start.......");
    yield return null;
    Debug.Log("Coroutine end.........");
}

void Update()
{
    Debug.Log("Update......");
}

void OnDestroy()
{
    Debug.Log("OnDestroy........");
}
```

执行结果为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107225534865.png" alt="image-20231107225534865" style="zoom:80%;" />

由此可看出，Destroy方法被调用后并不会马上销毁该组件，**与Destroy语句同在一个执行隔断的后面代码依然会正常执行，直到本执行隔断完毕为止**。

从以上也可看出，**”Start“与”Update“分属两个不同的执行隔断**

注意：在以上代码中加入“OnDisable”：

```c#
void OnDisable()
{
    Debug.Log("OnDisable..........");
}
```

执行结果为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107225657832.png" alt="image-20231107225657832" style="zoom:80%;" />

**由以上执行结果可知，当调用”Destroy“后，该组件会首先执行”OnDisable“方法，而后继续执行Destroy语句后同一执行隔断中的语句****。**



##### SendMessage，BroadcastMessage，SendMessageUpwards的区别：

发送消息有三种：

给自己发送用SendMessage，该物体上所有的脚本都会受到这个消息，只要是与目标同名的方法，不论是public或private，都可以受到消息后触发该方法

支持发送消息时附带参数：Gameobject.sendMessage(“”，参数）

给子类发送用BroadcastMessage，

给父类发送用SendMessageUpwards。

三者都可以发送消息给自身，不过SendMessageUpwards还可以发送给其所有父对象，BroadcastMessage则可以发送给所有子对象。

**可以是GameObject，也可以是Component来发送消息**



##### C#脚本中using引入库文件的详情：

在脚本开头使用using引入多个namespace，程序编译时并不会把该namespace对应的dll文件编译，该dll是否被编译取决于本script中是否有代码调用了该dll文中的方法或参数。

只有在有调用的情况下该dll才会被编译。

因此并不是使用“using xxxxx”，该“xxxxx”对应的dll就会被编译。这避免了dll被所谓的“过度引用”的问题。



逆向播放动画的方法：1.设置动画的speed = -1         2.调用animation.Rewind()方法



OnGUI相比其他GUI的缺点是？欧拉角相对于四元素的缺点是?

解答：耗费较大性能的同时没有产生相匹配的效率;    欧拉角容易产生万向锁死锁



##### 陀螺仪：

检测当前设备是否支持陀螺仪：`SystemInfo.supportsGyroScope`

开启陀螺仪：`Input.gyro.enabled = true`

获取陀螺仪相关参数：`Input.gyro.attitude`

###### 根据陀螺仪相关数值调整物体的rotation：

```c#
void Update(){
   transform.rotation = Quaternion.Slerp(transform.rotation, Input.gyro.attitude, 0.2f);
}
```

###### 根据陀螺仪相关数值触发设备震动：

```c#
void Update()
{
    new_y = Input.acceleration.y;        //根据在晃动手机时变化情况来决定是否震动
    d_y = new_y - old_y;
    old_y = new_y;
    if (d_y>0.8f)
    {
        Handheld.Vibrate();              //Android系统的震动方法
    }                                    //当为iPhone时则为iPhoneUtils.vibrate()
}
```

**PS**：1).获取输入的加速度：Input.acceleration，其与“陀螺仪”无关，代表输入的变化量

2).鼠标左键：0， 中键：1， 右键：2，在调用Input.GetMouseButtonDown(int index)时需注意区分



##### Application相关方法：

1.截图：`Application.CaptureScreenshot`

2.打开指定网址：`Application.OpenURL`

3.监听游戏当前运行是否低内存：`Application.lowMemory`，其为委托类型变量，可添加方法用于监听，如`Application.lowMemory += OnLowMemory`

4.检测当前平台是否为移动端：`Application.isMobilePlatform`

###### 方法：

OnApplicationFocus：按home键后应用进入后台，或长时间不触碰屏幕导致屏幕熄灭，或按下锁屏键，都会触发该方法





##### Mesh中各个UV坐标的用途：

Mesh中包含uv, uv2, uv3, uv4。通常uv指代该物体本身的mesh的uv信息，uv2是在光照贴图中使用的，uv3、uv4等则较少使用，一般在法线贴图等情况下使用



##### SneneManager中的相关方法：

获取当前场景：`SceneManager.GetActiveScene()`



##### Camera中的“ClearFlag”的作用：

camera渲染得到的图像都存储在显存中，但是可以通过设置clearflags来清除当前camera渲染时显存中已经存在的颜色或depth信息。例如：使用两个不同的camera，分别用于渲染场景和UI信息，前者用skybox，后者则需要选择depth

PS：同一位置中，depth数值越高，其渲染得到的图像越显示在上面，覆盖“depth数值低”的图像



##### Overdraw视图的作用：

该视图展示某个位置上该像素被绘制的次数，如果有多个物体在某位置有重叠，则重叠区域的Overdraw视图颜色会很深：通过颜色的深浅来展示各个像素被重复绘制的次数。也可通过该视图确认场景中各个物体重叠的区域



##### Transform相关的方法：

transform.childCount：该transform的子节点数量

transform.GetChild(int index)：获取具体的某个子节点

transform.root：当前物体的根节点



##### GetAxisRaw和GetAxis的区别：

前者只会返回“-1，0，1”三个值，其功效与GetButtonDown有点类似，

GetAxis则是一系列的值，随着按压的强度返回“【-1，1】”之间的值



##### activeSelf何activeInHierarchy的区别：

activeSelf仅仅指代inspector中checkbox是否勾选，只能表示该物体本身的状态。SetActive改变的是单个物体的状态

activeInHierachy代表该物体在scene中是否可以被看见，而不是Game视图中是否被看见(取决于当前camera渲染的区域)。

当 activeInHierachy为true时则代表该物体及其所有祖先父物体都是true状态，因为只有如此才能使其在scene中被看见

当物体“activeInHierarchy”为false时，如下图所示：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231108140511892.png" alt="image-20231108140511892" style="zoom:80%;" />

以上条件是"GameObject“为false，而子物体”Sphere“为true。该灰色图像对应为”activeInHierarchy = false”时的状态

如果某物体的父GameObject为false状态，则该物体的activeInHierarchy必然为false。由于SetActive(true)仅额能修改该物体自身的显示状态，所以即使调用该方法也无法将其在Scene场景中显示出来



部分情况下如果仅需要将某个Button不支持点击，可直接设置其“interactable”即可，如“button.interactable = false”



使用射线Raycast获取被碰撞者的信息：hit.collider.gameObject

在代码中打开键盘面板：TouchScreenKeyboard.Open()

Camera.allCameras：获取场景中所有可用的camera，如果该camera.enabled = false，则其不会被获取到

修改天空盒：Window -> Lighting ->Scene，第一个选项更改Skybox的材质即可

当在“Web Player”平台打最终包时，有个参数“offline deployment”，当勾选时表示该游戏发布后可以在没有网络的情况下也可以运行，如果不勾选，那么发布后没有网络就不可以运行



##### MonoBehavior自带的各个方法：

###### Awake、OnEnable、Start、Update等方法的执行顺序：

Awake：只要该脚本组件挂载的GameObject的.activeInHierarchy为true状态，就会触发本方法，与组件本身的enabled状态无关：即使该组件的enabled = false，只要该GameObject.activeInHierarchy = true，则本Awake方法依然会被触发。Awake方法在GameObject的整个生命周期中只会触发一次

OnEnable：当组件本身由false状态转变为true状态或者该组件第一次运行时会触发本方法

Start：Start方法会在Update第一帧之前执行，因此为了方便理解，将Start方法与Update方法绑定，在所有其他方法执行完毕准备执行Update方法前，先执行一次Start方法即可。并且在该组件的整个生命周期中只会执行一次

Update：该GameObject.activeInHierarchy = true，并且该组件的enabled = true，只有同时满足该条件，Update方法才会触发

OnDisable：

触发时机：

1).当调用Destroy方法后会马上触发该物体的“OnDisable”方法，但该物体只有在“下一帧”才会被设置为null

2).当该组件或其挂载的GameObject物体从“true”变成“false”时则会触发本方法

OnDestroy：无论该组件本身enabled为true或false，只要该组件所挂载的GameObject的activeInHierarchy为true状态，那么当本组件或本组件所挂载的GameObject被销毁时则会触发OnDestroy方法

如以下例子：

```c#
using UnityEngine;
using System.Collections;
public class test02 : MonoBehaviour {
    void Awake(){
        Debug.Log("awake..........");
    }
    void Start () {
        Debug.Log("start..........");
    }

    void OnEnable(){
        Debug.Log("onEnale...........");
    }

    public void Say(){
        Debug.Log("say..........");
    }
}

//调用该脚本：
public class test03 : MonoBehaviour {
      void Awake () {
         //形式一：
         this.transform.gameObject.AddComponent<test02>().Say();

        //形式二：
        //test02 temp = this.transform.gameObject.AddComponent<test02>();
        //temp.Say();
      }
}
```

以上两种形式输出结果相同，均为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231108142003622.png" alt="image-20231108142003622" style="zoom:80%;" />

注意：1).组件上的Awake，OnEnable会随着组件的添加自动被执行，并不会受制于当前的调用空间。类似于委托，当条件满足时会跳脱当前环境自动被调用

2).unity 自带的Awake， Start， Update等方法是有严格的执行顺序的：只有在场景中所有的Start方法都执行完毕后，Update方法才会被执行

###### OnBecameVisible和OnBecameInvisible：

当物体进入或离开Camera视野时触发

###### OnRenderImage：

在所有的渲染、代码逻辑执行完毕后，将得到的图像使用“Graphics.Blit”作进一步的处理

触发条件：包含该方法的脚本必须与“Camera组件”挂载在同一个GameObject上，否则无法针对图像作后期处理

```c#
public class CameraRenderImageFunc : MonoBehaviour
{
    public Material greyMat;
    RenderTexture destView;

    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        //从以下方法可以直接知道“destination”才是最终Game视图中的图像
        //Graphics.Blit(source, destination, new Vector2(2f, 2f), new Vector2(0, 0));
        Graphics.Blit(source, greyMat, 0);
        
        /** 为何无法把参数“destination”，“source”转移到外部参数“destView”？
         * “OnRenderImage”是在Unity中所有的渲染执行完毕后才由“MonoBehaviour”自动调用
         * 因此若想要把destination通过destView等外部变量临时存储，然后依然继续渲染其他图像
         * 这是无法实现的。
         * destView = destination;
         */
    }
}
```

注意：**“OnRenderImage”是在所有的渲染，逻辑结束之后才会执行，讲得实际一点就是该方法在"LateUpdate"之后**

**才执行。因此若要在该方法内再执行一些新逻辑并呈现于“Game”视图中是无法实现的**

OnMouseEnter()：当鼠标接触到某物体表面时会马上触发。

```c#
void OnMouseEnter(){
     render.material.color =Color.red;
}
```

OnMouseDown()：当鼠标在物体表面按下时触发

OnMouseDrag()：当拖动某个物体时触发

**注意**：无论OnMouseEnter或OnMouseDown、OnMouseDrag，包含该方法的脚本必须挂载在物体上才会起效



##### 进程、线程、协程在Unity中的使用：

######  进程，线程，协程的区别：

进程在内存中有自己独立的堆和栈，调度由操作系统决定；

线程在内存中有独立的栈和共享的堆，和多个线程一起共享其父进程的堆，调度由操作系统决定；

协程在内存中有独立的栈和共享的堆，其调度由程序员在代码中主动调用——yield return 语句

PS: 一个应用程序一般对应一个进程，一个进程中有一个主线程和若干个辅助线程，线程之间是平行运行的。线程内部可以开启协程，在特定的时间后将协程由挂起--》运行。

协程与线程、进程相比其优势在于：1.避免了很多无意义的调度，提高效率  2.调度由程序员自主决定，但同时也失去了对于线程而言可以使用多核CPU的优势。

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231108194323554.png" alt="image-20231108194323554" style="zoom:80%;" />

###### 是否可以在Unity中使用线程？

可以。当仅执行某些数学运算、遍历集合等操作，完全不会用到Unity的任何API或组件、对象，则可以使用线程来完成这些逻辑



###### 在Unity中使用线程需要注意什么？

默认Unity中所有代码都在主线程执行，其本身并不提倡“多线程运行”。

主线程和协程可以正常使用Unity的所有API或组件、对象。但线程则只能执行“与Unity交互极少”的逻辑，如算术运算、集合遍历等，可以在线程中使用Vector3值类型，但对于Unity中的Texture2D类型则无法使用



###### Unix、Linux、Window系统的进程调度规则：

**Unix**：Unix系统使用的时间片算法，即所有进程按照要求CPU资源的先后顺序依次使用CPU，但每人使用固定的时间，然后切换到下一个进程。如此循环往复。

缺点在于会因为固定时间到后要切换而浪费时间，因为并不是每次切换都是有意义的。

优点在于每个进程都是平等的，不会有进程长时间无法使用CPU资源

苹果的Mac OS是基于Unix内核开发的操作系统

**Linux**：Linux是一种免费开源的操作系统，不同于Unix，Windows，该操作系统上的软件大多为开源免费，由全球的Linux开发者共同开发和维护。但也基于免费的特点，软件的质量和维护也不会太好

**Window**：Windows系统使用竞争优先级的算法分配CPU资源，只有优先级最高的进程才可以获得当前的CPU资源。计算优先级的方式有：当前进程的重要性，饥饿时间——有多久没有使用CPU，等因素。



##### 屏幕后处理效果：

Unity有官方的插件支持“屏幕后处理”效果——Post processing，在AssetStore中导入即可

1.“Image Effect”主要是添加在camera上，用于显示出好的屏幕特效。

“Antialiasing”是用于当camera显示某个多边形或者其他形状的物体时，其边缘会有锯齿的效果，很不平滑，突兀，在camera上添加该脚本则显示出来的图像就是稍微缓和的。

2.在camera上设置特效和场景中添加是不同的，如在“render setting”里开启雾效和直接在camera中绑定“Global fog”脚本效果是不同的

在camera上添加脚本会使draw call增加。

“Glow”代表发光特效

老电视的噪点效果：“noise and grain”脚本。

模仿老旧照片的效果：Sepia Tone

模仿一个很强的光源被物体遮挡产生的光线散射的效果：Sun shaft

模仿漩涡特效：Twirl或者Vortex

3.给camera添加“bloom”泛光特效，看起来梦幻一点。通常“bloom”和“Tonemapping”配合使用，注意：“Tonemapping”特效只有在camera开启了HDR时才能正常使用。

“Tonemapping”模拟人眼看不同角度时的明暗变化的情况，一同添加在camera上使用

之后添加“sun shaft”阳光射线效果，将参数中的“shafter caster”设定为scene中的模拟日光照明的light即可，之后调节参数产生更逼真的效果

###### 屏幕后处理的性能优化：

“屏幕后处理”的具体效果通常都是通过“OnRenderImage”方法实现，但这会导致比较严重的性能问题，其帧率会严重降低，因此需要优化，其优化方案原理则是通过“OnPrerender”和“OnPostRender”分别对图像进行处理，详细过程：https://www.jianshu.com/p/7e08f845f99b，该链接有大佬的研究细节



##### 可编程脚本ScriptableObject：

通过ScriptableObject将“可序列化的数据”通过“真实的Asset资源”存储起来，在游戏运行中读取该ScriptableObject文件获取其中的数据即可(创建ScriptableObject对象的过程需要在“Editor模式”下执行，“Playmode模式”无法创建ScriptableObject对象)

###### 用到的相关API：

创建ScriptableObject对象：`T obj = ScriptableObject.CreateInstance<T>()`

将ScriptableObject对象存储到本地磁盘：`AssetDatabase.CreateAsset(obj, "Assets/xxx/xxx.asset")`

**注意**：ScriptableObject文件的后缀通常使用“`*.asset`”

###### 在脚本中使用ScriptableObject的过程：

1.创建可序列化类型脚本类型存储数据：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
//要让该类型的数据能够在Inspector中显示，必须要加[Serializable],只定义其为public是不够的
public struct Product
{
    //需要在inspector中看到该数据，则需要为public
    public string name;
    public string type;
    public int price;
    
    public Product(string _name, string _type, int _price)
    {
        name = _name;
        type = _type;
        price = _price;
    }
}

public class ProductsInfo : ScriptableObject
{
    public List<Product> productsInfoList = new List<Product>();
}
```

2.创建ScriptableObject对象，并向其中添加数据，然后作为文件存储到本地磁盘

```c#
//创建ScriptableObject对象
ProductsInfo m_productsInfo = ScriptableObject.CreateInstance<ProductsInfo>();

//向集合中添加数据(根据不同的业务逻辑实现即可)
m_productsInfo.productsInfoList.Add(new Product(name, type, price));

//将ScriptableObject对象存储到本地磁盘
AssetDatabase.CreateAsset(m_productsInfo, "Assets/xxx/xxx.asset");
AssetDatabase.SaveAssets();
AssetDatabase.Refresh();  //刷新视图，即可马上展现“新生成的文件”
```



##### Unity和Android之间的调用：

Unity端脚本：

```c#
using UnityEngine;
using System.Collections;

public class testSDKfunction : MonoBehaviour {
    /** 这是第一种方法
    private AndroidJavaClass jc;
    private AndroidJavaObject jo;
     
    void Start () {
        Debug.Log("start test sdk function...........");
        jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
        jo = jc.GetStatic<AndroidJavaObject>("currentActivity");
        Debug.Log("get the unityActivity...........");
    }
      
    void OnGUI()
    {
        if (GUI.Button(new Rect(100, 100, 200, 50), "call function"))
        {
            Debug.Log("start click the button............");
            if (jo != null)
            {
                jo.Call("test");
                Debug.Log("after a click,function is called...........");
            }
        }
    }
     * */

    /**用第一种方法时，核心在于在unity自带的activity中执行，这种方法的关键在于——方法体一定要位于currentActivity中
       如果是需要在游戏运行的mainActivity中执行，那么当改变了mainActivity时就一定要在mainfest中配置
     * 如果是在其他的activity时unity调用android，那么就需要在当前的activity中编写该方法体 **/
    //下面是第二种方法
    private AndroidJavaClass jc;
    void Start()
    {
        jc = new AndroidJavaClass("com.secondway.secondClass");
        //在src目录下新建package：com.secondway,在该package下新建class：secondClass
        jc.CallStatic("test","it's snowing outside!",22);
        //在该secondClass中编写被调用的方法test
        /**unity调用android的方法可以向其传递多个参数，但是android返回给unity的消息使用的是sendmessage，故只能传递一个string参数，因此会用到json**/
        Debug.Log("hello,this is the second way ......");
    }
    void getMsgFromAndroid(string str)
    {
        Debug.Log("从安卓得到的返回消息是："+str);
    }
}
```

Java端：

```java
package com.secondway;
import org.json.JSONException;
import org.json.JSONObject;
import com.unity3d.player.UnityPlayer;
import android.R.integer;
import android.util.Log;
public class secondClass {     
    private static void test(String str,int num){       
        Log.i("unity", "hello,world");       
        Log.i("unity", "...."+str+" .... "+num);             
        
        //向unity回复消息，多个信息时使用json传递       
        JSONObject jsonStr=new JSONObject();       
        try {              
            jsonStr.put("1", "yes");              
            jsonStr.put("2", "I'm");              
            jsonStr.put("three", "OK");            
        } catch (JSONException e) {                  
            // TODO Auto-generated catch block                  
            e.printStackTrace();            
        }       
        UnityPlayer.UnitySendMessage("Main Camera",  "getMsgFromAndroid", jsonStr.toString());     
    }
}
```



##### 自制Dll库并导入到Unity中使用

###### 编写C#语言的Dll库：

创建C#的库并编写C#脚本，之后编译生成Dll即可

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111110248333.png" alt="image-20231111110248333" style="zoom:80%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111110326822.png" alt="image-20231111110326822" style="zoom:80%;" />

注意：如果需要在Unity脚本中调用该Dll库中的类型和方法，则该类型和方法必须使用“public”关键字修饰

###### 编写C++语言的Dll库

1).创建C++的库并编写C++脚本：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111110840407.png" alt="image-20231111110840407" style="zoom:80%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111110903240.png" alt="image-20231111110903240" style="zoom:80%;" />

2).选择<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111110958227.png" alt="image-20231111110958227" style="zoom:80%;" />生成Dll文件

**注意**：部分情况下在将C++ DLL导入到unity中后，出现如下的报错：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111111048149.png" alt="image-20231111111048149" style="zoom:80%;" />

此时则是由于编译C++项目时选择的平台有误，更换为“x64架构”即可

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111111139706.png" alt="image-20231111111139706" style="zoom:80%;" />

###### 在Unity中使用Dll库文件：

1).将Dll库文件导入到Unity的“Plugins文件夹”下

2).在代码中调用该Dll库中的方法

针对C#和C++语言编写的Dll库，其在Unity中的调用方式不同，细节如下：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System.Runtime.InteropServices;  //[DLLImport("xxx")]特性的命名空间
using CsharpDLL;    //自定义的C# DLL

public class DLLImportFunc : MonoBehaviour
{
    /******* START: 使用C++ 编写的DLL  *********/
    [DllImport("CppDll")]
    static extern int GetMaxNumber(int x, int y);
    [DllImport("CppDll")]
    static extern int ChangeSize(int increment);
    /******* END: 使用C++ 编写的DLL ********/

    void Start()
    {
        /******* START: 使用C++ 编写的DLL  *********/
        Debug.Log("Max number :  " + GetMaxNumber(3, 100));
        Debug.Log("New size is :  " + ChangeSize(2));
        /******* END: 使用C++ 编写的DLL  *********/
        
        /****** START: 使用C# 编写的DLL ******/
        DLLFuncKit temp = new DLLFuncKit();
        Debug.Log("TYPE:  " + DLLFuncKit.type + "   PRICE: " + temp.price);
        //“type”参数由于是static修饰，故可直接调用；“price”则需要先实例化才行
        /****** END: 使用C# 编写的DLL ******/
    }
}
```

**PS**：当有多个平台的dll或者在不同的条件下需要调用到不同的dll时，可以将[DLLImport("xxx")]中的"xxx"做成一个参数，借助#if UNITY_ANDROID ......#endif来改变其值。这样可以简化代码

















