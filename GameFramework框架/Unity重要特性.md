



#### Unity的内存释放逻辑：

Unity释放内存用到的各个方法如下：

##### 针对单个Object的卸载：

<font color=red>**Destroy**</font>：通常用于销毁某个Clone的具有实际物理载体的GameObject以及其上挂载的Component。但是该GameObject所使用的prefab却不受影响，即虽然销毁了GameObject，但该GameObject所使用的prefab依然存在于内存中

**<font color=red>Resource.UnloadAsset(Object)</font>**：释放某个独立的object。当确定某个object没有任何引用后可调用该方法释放其占用的内存，如“资源系统”中的“AssetPool”，将“从AB中加载到内存的某个Asset”进行卸载

##### 针对AB的卸载：

**<font color=red>AssetBundle.Unload(false)</font>**：释放该AB占用的内存资源

**<font color=red>AssetBundle.Unload(true)</font>**：释放该AB以及所有从该AB中加载得到的Asset占用的内存资源(该方法慎用，否则某些Asset被释放后，如果后续还要使用则需要重新从AB中加载——该过程会导致卡顿)

##### 全局统筹，不针对单个对象的卸载：

**<font color=red>Resource.UnloadUnusedAssets()</font>**：通常作为整个游戏中“内存管理”的辅助补充，在“**资源模块**”的“ResourceComponent.**<font color=red>Update</font>**"逻辑中**<font color=red>“按照指定时间间隔”自动执行</font>**

**<font color=red>GC.Collect</font>**：同样作为游戏中内存管理的辅助补充，但该方法通常不会在代码中手动调用，除非遇到“lowMemory”情况——该情况可使用“**<font color=red>Application.lowMemory</font>** += OnLowMemory”进行监听，**“Application.lowMemory”为事件Event类型，因此可自由设置遇到该情况时的执行逻辑**



##### 为什么UI界面关闭并清空其对象池后再次打开界面时不会卡顿？

**解答**：“UI界面”所使用的对象池是以“实例化出来的UI面板的物理GameObject”构成的，**当某个UI界面关闭时会调用该UI界面的ObjectBase.Unspawn进行回收**，**<font color=red>只有在其对象池内部需要释放某个对象时才会调用该UI界面的ObjectBase.Release释放内存占用</font>**，即Step1——Destroy该GameObject + Step2——释放该GameObject的prefab占用的内存

但是**关键点在“Step2的prefab的内存释放”**：

**在“资源系统”中，针对所有的AB和Asset分别创建有“AB池”和“Asset池”**，以上的**<font color=red>“Step2”中会调用“Asset池”中针对每一个Asset的ObjectBase.Unspawn方法进行回收</font>**。所以**该prefab是否最终会从内存中被释放则由“资源系统的Asset池”内部逻辑决定**：根据该prefab当前的m_SpawnCount, m_LastUseTime, m_Priority以及maxCapacity等参数依次筛选。

所以如果在短时间内再次打开该界面，由于该界面的prefab的“m_LastUseTime”并未超出“过期时间”或其他原因，导致该prefab依然存在于内存中。此时是不会出现卡顿的



##### 第一次打开某个UI界面时通常会卡顿一下，后续打开时则不会，这是为什么？

**解答**：其中一种见解是，第一次打开某个UI界面时，该UI界面的Prefab并没有从其所属的AB加载到内存中，因此需要先执行“AssetBundle.LoadAsset”的过程。在后续再次打开该界面时，如果短时间内该prefab依然存在于内存中，因此该过程可省略，直接执行Instantiate方法。说明链接为：https://gwb.tencent.com/community/detail/109766，有机会要验证一下





### 配置 && 参数：

#### UnityEngine.dll中已经包含的工具class：

1.**<font color=red>AndroidJNI, AndroidJNIHelper</font>**: 用于获取Android相关的方法或参数

2.**<font color=red>JsonUtility</font>**：用于将object转成json字符串或相反的过程





**设置UI对象的Depth**：

只要是**<font color=red>具有Canvas组件的对象</font>**即可以通过设置其“**<font color=red>canvas.sortingOrder</font>**”得到目标显示图像。可参考GameFramework的UI模块中“UIForm.OnDepthChanged”方法



**StopAllCoroutines**：

UI模块中有时会短时间内**快速点击导致“频繁的显示/隐藏界面”，并且界面的“显示/隐藏”通常都会加入“Fade”渐变效果**(Sound模块切换声音时也是如此)。由于没有“动画系统”中的"CrossFade"等方法，因此通常都会使用“协程”自定义“Fade渐变”效果。

为了防止短时间内频繁“StartCoroutine”导致协程执行错乱，因此每次需要“StartCoroutine”前都需要**<font color=red>先执行"MonoBehavior"中的“StopAllCoroutines”方法</font>**，以停止**所有在本脚本MonoBehavior中开启的协程**(**<font color=red>其他MonoBehavior扩展子类中开启的协程不受影响</font>**)



#### 各个不同Version的区别：

**<font color=red>applicableGameVersion/GameVersion</font>**：指该游戏在“PlayerSetting”中设置的版本号，可以直接使用代码“**<font color=red>Application.version</font>**”获取：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221227170206736.png" alt="image-20221227170206736" style="zoom: 67%;" />

**<font color=red>InternalGameVersion</font>**：项目内部使用的版本号。不具有实质意义，仅仅在项目内部开发过程中使用的

**<font color=red>InternalResourceVersion</font>**：内部Resource资源版本号，只要重新打了AssetBundle，该数值就会递增。主要用于在“热更新”时区分是否需要下载最新的“Resource资源”

**<font color=red>Application.unityversion</font>**：获取该程序使用的Unity版本





#### Unity内置脚本参数：

**<font color=red>注意：Unity内部不区分大小写</font>**。即**<font color=red>同一路径下的“Test01.cs”和“test01.cs”</font>**两个文件**<font color=red>会被认为是同一个文件</font>**，**此时会报错的**。但**<font color=red>若两个文件“后缀名不同”，即“Test01.cs”和“Test01.txt”</font>**，**这两个文件是支持在同一路径下存在的**

##### Application.persistentDataPath: 

 `C:/Users/Frank/AppData/LocalLow/Frank/UnityGameFramework`（项目名称）

##### Application.dataPath: 

`C:/Work/Unity/GitHub/GameFrameworkUtility/StarForce/Assets`

注意：以上参数获取到的路径必然是“**<font color=red>绝对路径</font>**”

##### 根据Unity的版本设定宏指令：

如`UNITY_2017_3_OR_NEWER`

```
#if UNITY_2017_3_OR_NEWER
     "UnityGameFramework.Runtime",
#endif
```

##### Application.systemLanguage：

获取应用的系统语言

##### [DisallowMultipleComponent]：

该标记代表同一GameObject只能同时挂载一个该类型组件

##### Time.timeScale:

暂停游戏时，直接设置“Time.timeScale = 0”即可

**LayerMask.NameToLayer**：

根据string类型的“layer name”获取到Unity中默认的“layer编号”。使用方式：

```c#
m_InstanceRoot.gameObject.layer = LayerMask.NameToLayer("UI")
```

**GameObject.SetLayerRecursively**：

设置包含目标gameObject以及其所有子gameObject的Layer层



设置屏幕不息屏：Screen.sleepTimeout = Screen.NeverSleep

设置鼠标显示或隐藏：Screen.showCursor = false。设置鼠标显示样式：Cursor.SetCursor

图片导入Unity后可能变得模糊：部分情况下是由于“mipmap”造成的，建议关闭(具体原因暂不清楚)





### Unity的特殊文件夹：

#### **Resources**：

放置项目中需要用到的一些资源文件，如纹理、音频等

**使用方式**：`Resource.Load(string path)`

1.**加载资源时使用的路径“path”是“Resources”文件夹的相对路径**，如某个文件Editor内的绝对地址是“Assets/xxx/xx/Resources/File/a.txt”，则此时参数“path”直接使用“File/a”即可

2.**<font color=red>加载的文件无需后缀名</font>**，即无论该文件名是“a.txt”, "a.png"，直接使用“a”即可

##### 注意细节：

1.**<font color=red>放在该目录下的资源无论是否在项目中被使用，都会被打进最终apk或ipa的安装包中</font>**。**所以为了控制最终安装包的大小对于Resources下的文件需要额外小心**。通常**结合热更新只会把项目中确定不会被频繁修改的文件放在“Resources”文件夹下**

2.**<font color=red>对于“Resources”并不严格要求在“Assets”根目录下，只要该文件夹名字为“Resources”即可</font>**，**<font color=red>如“Assets/Resources”和“Assets/xxxx/xxx/Resources”同样有效</font>**，**但要求“Resources”不能放置在某些特殊文件夹下如“Editor”**。

3.Resources.Load由于不需要要文件后缀名，因此**只支持读取某些特定后缀的文件，如“xxx.txt”, "xxx.png"等**，不支持读取“xxx.lua”文件，即**<font color=red>不可以直接使用“Resources.Load”读取Lua文件</font>**

**PS:当加载的是“xxx.txt”文件时**，可以直接使用"UnityEngine.**<font color=red>TextAsset</font>**"来转换：

```c#
TextAsset file = (TextAsset)Resources.Load(filename);
Debug.LogFormat("content: {0}", file.text);
```

如上直接通过**<font color=red>TextAsset.text获取文本的内容</font>**



"Resources"可以是一级目录，也可以是”xx/xx/Resources“，只要名字是”Resources“，都会被打进最终发布包中。并且程序安装完成后，该目录是不存在的



##### 扩展：加载方式的区别

**Resources.Load**: **运行时和“EditorMode”都可以使用**

Resources.LoadAssetAtPath：当前已废弃，只支持“EditorMode”下加载资源，建议使用“AssetDatabase.LoadAssetAtPath”

**AssetDatabase.LoadAssetAtPath**: **只能在“EditorMode”下使用**，**<font color=red>只能读取“Assets”目录下的文件</font>**，如“Assets/xxx/xxx/xx/a.txt”，**<font color=red>并且文件名需要带后缀</font>**



#### "Standard Assets"：

该文件夹中的脚本会比别的脚本先被编译，编译后的文件会集中于”Assembly-CSharp-firstpass.csproj“项目文件中。由于此特性，若需要在C#脚本中引用其他语言的脚本，则需要将其放在”Standard Assets“中。

在Standard Assets中的脚本会被预先编译，这样可以方便在C#脚本中调用js脚本或其他语言的脚本。



#### **Editor**: 

不一定要在根目录下，任何地方都可以，这和Resources一样

"Editor”文件夹可以是"Assets"下的一级目录，也可以是”xx/xx/Editor“。只要是”Editor“下的文件都不会被打包进最终发布包中。

当脚本中要用到UnityEditor的API时，需要将该脚本置于Editor文件夹下。Editor文件夹可置于普通文件夹下任意层级目录，也可以是根目录

Editor，在build时不会被打包进去

UnityEditor是在Editor文件夹下的脚本使用的，注意该文件夹下的所有资源都是不会被build到最终包的。



#### **Editor Default Resources:** 

必须要在根目录下，可以使用EditorGUIUtility.Load()方法加载资源，但要求该资源也必须在Editor Default Resources下

"Editor Default Resources"只能是"Assets"下的一级目录，和"Editor"一样都不会被打进最终发布包中。

Editor Default Resources只能是根目录下，并可以通过**EditorGUIUtility****.Load()**来加载其中的资源



#### **Unity项目的某些编译文件的作用：**

把项目基本框架或者某些平常不会轻易改动的代码放在Plugins或者StandardAssets文件夹下，这些文件夹下的代码会提前编译，且在没有改动时不会重新编译的，坏处是此处的代码无法引用外部其他的脚本，因为是提前编译的。

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230525161544216.png" alt="image-20230525161544216" style="zoom:67%;" />

当自主创建的脚本代码有改动时会重新编译csharp.dll文件，只有当plugin下的C#脚本改动时才会编译firstpass.dll

这里的核心就是适量减少需要反复编译的脚本



#### Unity中各个方法的执行顺序：

Awake、Start、OnEnable、FixedUpdate、Update、LateUpdate、OnDisable、OnDestroy等，官方解析如下：https://docs.unity3d.com/2018.2/Documentation/Manual/ExecutionOrder.html

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F.png" alt="执行顺序" style="zoom:50%;" />





#### Time.timeScale修改后的影响：

Time.timeScale = 0 后 **Time.deltaTime会变成0，但update和LateUpdate仍然会执行，但FixedUpdate不会执行；**

因为渲染在继续（如果渲染停掉，那就没有图像了），但物理活动完全停止



**更改"Time.timeScale"后，****Time.time, Time.deltaTime, Time.realtimeSinceStartup的区别：**

1."Time.time","Time.deltaTime","FixedDeltatime"都会受到影响，"realtimeSinceStartup"不会。

2."Update", "LateUpdate"的执行不会受到影响，依然会按照设备本身的性能执行，即60fps/30fps，这是设备硬件的物理性能，与timeScale 无关。但"deltaTime"数值本身会受影响，若代码中用到了"deltaTime"则会明显的看到区别。

 "FixedUpdate"由于直接受到project setting中timeScale的限制，因此"FixedUpdate"的执行会明显受到影响。

 当timeScale = 0时，"FixedUpdate"不会再执行。

3.由于"Update","LateUpdate"依然会照常执行，但deltaTime数值出现了变化，因此在代码逻辑中添加deltaTime，则同样会受到影响

5.协程中yield使用的是非时间相关的值时不受影响

```c#
IEnumerator EnuFunc(){
    Debug.Log(Time.realtimeSinceStartup + "  !!!!!!!!!  " + Time.time);
    //与具体时间相关，因此当timescale = 0时，后面语句不会再执行
    //yield return new WaitForSeconds(2f); 

    //过一帧再执行，与具体时间无关，是设备的执行帧率，因此不会受timescale影响
    yield return 0;  
    Debug.Log(Time.realtimeSinceStartup + "  @@@@@@@@@@" + Time.time);
}
```

PS:"Update","LateUpdate","OnGUI"是机器设备的渲染帧率，以保证图像的正常显示，这和timescale没有关系，与设备的硬件条件有关

 "FixedUpdate"用于处理刚体的物理逻辑，因此会受到timescale影响。



#### 非常重要：Shader是否支持热更新？

结论：Shader也是可以“热更新”的，虽然其为代码，但和C#不同，不需要编译后才能运行。因此可以直接将Shader文件打入AB中。

但在代码中为某个材质设置Shader时，不要使用`Shader.Find`，而要用`AssetBundle.LoadAsset<Shader>()`

参考链接：https://www.cnblogs.com/cpxnet/p/6439706.html



#### 自定义协程：

通过实现IEnumerator接口，自定义其“MoveNext()”方法，可以自由决定“协程结束的标志”：

```c#
public class CustomTimeIEnumerator : IEnumerator
{
    float _time = 0f;
    public CustomTimeIEnumerator(float time)
    {
        _time = Time.realtimeSinceStartup + time;
    }

    public object Current {
        get
        {
            return null;
        }
    }
    
    //表示协程内部执行时是否到下一步，当返回true时则继续协程中下一次；
    public bool MoveNext()          
    {     //当返回false时则协程结束。因此这里的判断条件需要注意是“<=”，而不是“>=”
        return Time.realtimeSinceStartup <= _time;
    }

    public void Reset()
    {
        
    }
}
```

自定义协程的好处在于可以自定义协程执行完毕的条件，而不依赖于具体的时间，尤其是在“Time.timeScale”被修改后，如果需要不受timeScale影响依然正常执行协程，则可以使用本方式

并且其使用途径远非如此，任何需要自定义协程结束标志的情况都可以使用本方式，如以上监测“动画播放”完毕的情况，也可以使用本“自定义协程”的方式

其调用方式如下：

```c#
IEnumerator CustomTimeTest()
{
	Debug.Log("时间：  " + Time.realtimeSinceStartup);
	yield return StartCoroutine(new CustomTimeIEnumerator(6f)); //自定义协程改变的是yield return的实现形式
	Debug.Log("此时的时间是：   " + Time.realtimeSinceStartup);
}   //依然需要在“IEnumerator”的框架内实现

void Start(){
	......
	StartCoroutine(CustomTimeTest());  //和平常协程一样调用
}
```















































