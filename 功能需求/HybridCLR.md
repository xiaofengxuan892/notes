[TOC]





#### 安装及初始化：

1.打开“Package Manager”并点击左上角“+”号，选择“Add package form git URL”并输入链接：https://gitee.com/focus-creative-games/hybridclr_unity.git 或 https://github.com/focus-creative-games/hybridclr_unity.git

2.插件安装完成后，点击菜单栏“HybridCLR -> Installer”打开如下界面，并点击下方的“Install”按钮。当显示为“**<font color=red>Installed: True</font>**”时则代表整个安装过程结束，才可以进行后续功能开发

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240325211401866.png" alt="image-20240325211401866" style="zoom:80%;" />



#### 配置HybbridCLR Settings：

将需要热更的dll填写到“Hot Update Assembly Definitions”或“Hot Update Assemblies”中。**<font color=red>两者作用相同，因此填写其中一个即可，重复填写会报错</font>**。且前者选择对应的“程序集定义”即可，**后者直接填写该dll的名称(不带dll后缀**，即若热更dll为“HotUpdate.dll”，则直接填写“HotUpdate”即可)

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240326183241463.png" alt="image-20240326183241463" style="zoom:80%;" />



#### 热更Dll的使用规范：

##### 热更新dll中的代码不支持的特性

绝大多数代码都可以在热更d'l'l中良好运行，只有以下特性不支持，如：

1.**不支持在热更新dll的脚本中定义“extern”函数**，但可以将这些“extern函数”在“AOT”的dll中定义，然后在热更新的脚本中调用这些方法

2.**不建议在脚本内使用“RequireComponent(typeof(xxx))”来限制其必要的组件**。**只有在该xxx之前已被实例化或使用“AddComponent<xxx>()”添加过，否则其不会被识别**，导致“RequireComponent(typeof(xxx))”无法起到限制作用



##### 程序集的划分：

1.项目代码分为“AOT”程序集和“热更新”程序集，并且游戏启动至少需要一个AOT程序集来负责初始化以及触发热更新相关流程，所以该AOT程序集必然随主包发布

2.**<font color=red>不能在AOT程序集的代码逻辑中引用“热更程序集”</font>**。**如果将“Assembly-CSharp.dll”作为AOT程序集**(该dll为最顶层assembly，其会自动引用所有剩余assembly)，则**<font color=red>需要关闭“热更dll”的“auto reference”选项</font>**，否则容易出现打包错误的情况

3.**如果有多个热更dll**，则一定要按照各个热更dll之间的依赖顺序来加载：**<font color=red>先加载被依赖的热更dll</font>**，但针对<font color=red>**多个“补充元数据dll”则没有严格加载顺序**</font>



##### 代码裁剪：

HybridCLR使用IL2CPP的打包方式，其会自动裁剪AOT代码来减小包体大小。为了避免代码裁剪可采用如下方式：

**1)**.点击菜单栏“**HybridCLR -> Generate -> LinkXml**”，其会遍历热更dll代码，将其中用到的各个AOT中的类型和方法列出来并**在“Assets/HybridCLRGenerate”文件夹下生成“link.xml”文件**。

这样**<font color=red>在Unity打包时会参考该“link.xml”文件，避免将AOT dll中用到的这些类型和方法裁剪掉</font>**

**2)**.在“Assets”文件夹根目录下新建“link.xml”文件，并参照如下形式写下需要保留的类型和方法：

```c#
<?xml version="1.0" encoding="utf-8"?>
<linker>
  <assembly fullname="DOTween">
    <type fullname="DG.Tweening.Core.DOGetter`1" preserve="all" />
    <type fullname="DG.Tweening.Core.DOSetter`1" preserve="all" />
  </assembly>
</linker>
```

IL2CPP打包方式下，**Unity会自动扫描项目中所有的“link.xml”文件来避免裁剪掉“需要保留的类型和方法”**，因此**项目中可以包含多个“link.xml”文件**

**注意**：默认情况下在点击“HybridCLR -> Generate -> All”后，无论代码裁剪级别为何，打出的包均不会有问题；但**若后续通过“热更新机制”频繁更新“热更dll”**，而**<font color=red>该“热更dll”可能会用到新的“AOT dll 中的类型和方法”</font>**，此时则有可能会出现异常。因此**<font color=red>代码裁剪级别尽量设置为“最低 low”</font>**：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240328185927744.png" alt="image-20240328185927744" style="zoom:80%;" />



##### 热更dll的调用：

###### 补充元数据dll：

由于HybridCLR默认使用IL2CPP脚本后端，游戏运行时先执行AOT Dll，再加载热更dll。但**<font color=red>若要在热更dll中调用AOT dll中的类型或方法，则需要先为这些AOT dll加载“补充元数据”</font>**

**注意**：**“补充元数据dll”会增大包体大小**，且**<font color=red>在加载该dll后，其内存占用是该dll原大小的3-4倍</font>**。因此对于“补充元数据dll”并非越多越好，只补充必要的dll即可

**确定需要补充的元数据dll列表**：

**来源1**：在点击“HybridCLR -> Generate -> All”或“**<font color=red>Generate -> AOTGenericReference</font>**”后，其会自动查询热更dll的代码，确定需要补充元数据的AOT dll并生成“Assets/HybridCLRGenerate/**<font color=red>AOTGenericReferences.cs</font>**”文件，该文件会列出热更dll中大致会用到的需要元数据的dll：
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240328163905370.png" alt="image-20240328163905370" style="zoom:80%;" />

**来源2**：额外需要添加的补充元数据dll，可自由添加。但**由于补充元数据dll会增大包体且运行时内存占用为原大小的3-4倍**，因此**<font color=red>尽量从“AssembliesPostIL2CppStrip文件夹”下挑选需要补充元数据的dll</font>**



**生成“补充元数据dll”**：

点击菜单栏“HybridCLR -> Generate -> AOTDlls”，其会生成裁剪后的AOT dll，保存在“{projectPath}/HybridCLRData/AssembliesPostIL2CppStrip/{BuildTarget}/”

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240328164140927.png" alt="image-20240328164140927" style="zoom:80%;" />



**补充元数据的加载模式**：

**HomologousImageMode.SuperSet**：该模式下，补充元数据dll可以是“AssembliesPostIL2CppStrip文件夹”中裁剪后的dll，也可以是“未经裁剪的原始dll”。**<font color=red>为避免报错，通常使用该模式来加载“补充元数据dll”</font>**

**HomologousImageMode.Consistent**：该模式下，补充元数据dll必须是“AssembliesPostIL2CppStrip文件夹”中裁剪后的dll，否则会报错



###### 加载热更dll：

**完整流程**：1.执行AOT dll (游戏启动后会默认执行) -》 2.加载AOT程序集的“补充元数据dll”以方便“热更dll”调用其中的类型和方法 -》 3.加载“热更dll” -》4.通过“反射”调用“热更dll”中的类型和方法

**加载“补充元数据dll”**：

根据**“补充元数据dll”列表**依次**<font color=red>从“AssembliesPostIL2CppStrip文件夹”下拷贝对应的dll文件</font>**(需**重命名添加“*.bytes”后缀**)，并放入“StreamingAssets文件夹”或“CDN热更资源服”以方便后续加载

```c#
HybridCLR.RuntimeApi.LoadMetadataForAOTAssembly(dllBytes, HomologousImageMode.SuperSet)
```

**加载热更Dll**：

```c#
// Editor环境下，HotUpdate.dll.bytes已经被自动加载，不需要加载，重复加载反而会出问题。
#if !UNITY_EDITOR
   Assembly hotUpdateAss = Assembly.Load(File.ReadAllBytes($"HotUpdate.dll.bytes"));
#else
   // Editor下无需加载，直接查找获得HotUpdate程序集
   Assembly hotUpdateAss = System.AppDomain.CurrentDomain.GetAssemblies().First(a => a.GetName().Name == "HotUpdate");
#endif
```

**注意**：1.不论是加载AOT或热更dll前，都需要先通过“File.ReadAllBytes()”或其他方法获取到该dll的字节数组bytes

2.不论是AOT或热更dll，都需要将该dll文件本身重命名 —— **添加后缀“.bytes”**，即**<font color=red>热更dll在编译完成后得到的是“HotUpdate.dll”，需要先将其重命名为“HotUpdate.dll.bytes”文件，之后再通过“File.ReadAllBytes()”读取该文件的数据</font>**以方便后续加载该dll

3.Editor模式下禁止使用Assembly.Load再次加载热更dll。由于**<font color=red>Editor下已经默认加载了所有程序集，再次加载则会出现问题</font>**，可使用UNITY_EDITOR宏进行区分

4.**当在热更dll中调用AOT泛型时**，**<font color=red>先在AOT程序集中查找，如果没有则会在“加载的补充元数据dll”中查找</font>**。“补充元数据dll”和“热更dll”由于都是通过“动态加载”的方式，因此属于“**<font color=red>dll的解释器版本</font>**”，在执行效率上肯定不如初始的“AOT” dll。所以如果可以的话，最好的方式是**<font color=red>直接在AOT代码中显式调用该泛型的实例化方法</font>**，**其效率比通过“补充元数据dll”解释执行要快得多**

之所以目前没有采用这种快速的方式，则是**因为手动查找热更dll用到的泛型，然后添加到AOT代码中。这种方式太繁琐，对开发人员有较大的负担**。所以统一采用“补充元数据dll”的方式，虽然效率上差一点，但整体而言可以极大减轻开发负担

**触发热更dll中的方法**：

1.使用“反射”的形式(该方式成本较高，若需要带参数则有额外GC。因此并没有将“入口”脚本挂载在某个场景或prefab上方便)：

```c#
//通过反射调用热更dll中某个脚本的static方法
Type mType = hotUpdateDll.GetType("GameLauncher");
mType.GetMethod("StartGame")?.Invoke(null, null);
```

**PS**：当通过“反射”调用的方法**<font color=red>需要传递参数时，为避免额外GC</font>**，可采用如下优化方式

```c#
//PS：该方式更为高效，且支持传递任意数量的参数。在实际使用中推荐使用本方式
//1).调用不带参数的方法
MethodInfo mMethod1 = type.GetMethod("Run");
Action mAction1 = (Action) Delegate.CreateDelegate(typeof(Action), mMethod1);
mAction1();

//2).调用带参数的方法
MethodInfo mMethod2 = type.GetMethod("Walk");
Action<float> mAction2 = (Action<float>) Delegate.CreateDelegate(typeof(Action<float>), null, mMethod2);
mAction2(5f);

//3).调用包含多个参数的方法
MethodInfo mMethod3 = type.GetMethod("Fly");
Action<float, string> mAction3 =
    (Action<float, string>) Delegate.CreateDelegate(typeof(Action<float, string>), null, mMethod3);
mAction3(10f, "hello, world");
```

如上通过“**<font color=red>System.Delegate.CreateDelegate”将该方法封装成“委托类型的对象”，如此可自由传递任意数量的参数</font>**

具体方式可参考https://github.com/xiaofengxuan892/DemoCollection 中的HybridCLR案例

`Delegate.CreateDelegate`方法中，第一个参数是“委托的类型”，第二个参数是要绑定到的对象(**对于static方法都为null**)，第三个参数为“目标方法methodInfo”

2.将包含“入口”代码的脚本挂载在某个prefab上(该prefab必须打成AssetBundle)，在加载热更dll完毕后，通过Instantiate实例化该prefab来触发：

```c#
//将该脚本挂载在某个prefab上，并将该prefab打成AssetBundle，之后通过Instantiate实例化来触发
public class HotUpdateEntry : MonoBehavior{
    void Start(){
        Debug.Log("Launch Game......");
    }
}
```

**完整代码如下**：

```c#
void Start() {
    LoadMetadataForAOTAssemblies();

    // Editor环境下，HotUpdate.dll.bytes已经被自动加载，不需要加载，重复加载反而会出问题。
#if !UNITY_EDITOR
    Assembly hotUpdateAss = Assembly.Load(File.ReadAllBytes($"{Application.streamingAssetsPath}/HotUpdate.dll.bytes"));
#else
    // Editor下无需加载，直接查找获得HotUpdate程序集
    Assembly hotUpdateAss = System.AppDomain.CurrentDomain.GetAssemblies().First(a => a.GetName().Name == "HotUpdate");
#endif

    Type type = hotUpdateAss.GetType("Hello");
    type.GetMethod("Run")?.Invoke(null, null);
}

void LoadMetadataForAOTAssemblies() {
    List<string> aotDllList = new List<string>{"UnityEngine.CoreModule.dll.bytes"};
    foreach (var dllName in aotDllList) {
        byte[] dllBytes = File.ReadAllBytes(string.Format("{0}/{1}", Application.streamingAssetsPath, dllName));
        int resultCode = (int) HybridCLR.RuntimeApi.LoadMetadataForAOTAssembly(dllBytes, HomologousImageMode.SuperSet);
        Debug.LogFormat("resultCode: {0}, dllName: {1}", resultCode, dllName);
    }
}
```



##### 调用热更dll中的脚本时需要注意的问题：

2.在“热更dll”内部调用脚本，通常来讲有两种形式：

方式一：通过“AddComponent<xxx>()”动态为某个物体添加“热更dll”内部的脚本

方式二：将该脚本挂载在某个物体或场景上，通过“资源加载”的方式来使用这个脚本

**将热更脚本挂载在物体上可能出现的问题**：

1).直接在代码中动态加载该挂载热更脚本的物体时，出现“scripting missing”错误

**解决办法**：**将挂载热更脚本的资源(prefab, scene, ScriptableObject)**，无论该资源是放在“Resources文件夹”下随主包发布，还是后续通过“CDN资源热更”，**<font color=red>都需要将其打包成AssetBundle</font>**，**<font color=red>后续通过AB的方式来加载该物体，即不会出现“scripting missing”的问题</font>**

2).当在主线程中通过AddComponent<T>()添加热更dll中的某个脚本，**<font color=red>同时在另一个异步线程中</font>**通过资源加载系统异步加载某个“挂载该脚本的prefab”，此时则有可能会出现崩溃

**解决方法**：**避免两者同时进行**，或不要将该脚本挂载在prefab上，**<font color=red>两者都通过AddComponent<T>()动态添加该组件</font>**

3).实例化某个“**挂载热更脚本的prefab**”后，无法使用GetComponent(string name)获取该组件

**解决办法**：使用GetComponent<T>()或GetComponent(typeof(T))获取该物体上的组件





















