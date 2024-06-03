[TOC]



### 重要问题1：当把资源打成AB后，是否需要将其从项目中删除，以防止该资源重复打入“最终包”中？

**核心原理**：

在Unity中打包时，以下资源会被打入最终包中

**<font color=red>1.“Resources文件夹”或“StreamingAssets文件夹”中的资源</font>**：

无论其在游戏运行中是否用到，都会被打入最终包中

区别在于：前者中的资源会被压缩，而后者的资源则不做任何处理，直接打入最终包中

**验证如下**：

不包含任何资源的安装包大小为“20.8MB”，音频资源大小为“14.1MB”

**1)**.若将该资源放在“StreamgingAssets文件夹”下，最终包大小为“34.9M”，即“20.8M + 14.1M”

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230520172413593.png" alt="image-20230520172413593" style="zoom:67%;" />

**2)**.若将该音频资源放在“Resources文件夹”下，最终包大小为“33.2M”，由此说明Resource文件夹中的资源会被压缩

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230520172235345.png" alt="image-20230520172235345" style="zoom: 67%;" />

**2.其他非Editor文件夹中满足指定条件的资源**：

**规则**：在打最终包时，Unity会依次检测“BuildSettings”中的各个场景：**<font color=red>如果某个资源与该场景有“直接依赖”关系，那么该资源必然会被打入最终包中</font>**。

**注意**：这里的“直接依赖”指的是该场景**<font color=red>在非运行状态</font>**时，场景中的某个GameObject直接用到了某个资源

如果该场景在运行状态时动态加载某个资源，则**<font color=red>不属于“直接依赖”关系</font>**。此时该资源不会被打入最终包中

**验证如下**：

**1)**.创建自定义文件夹“ResForAB”，里面包含音频资源，大小为“14.1M”：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230520202012233.png" alt="image-20230520202012233" style="zoom: 67%;" />

**2)**.在场景中创建空GameObject，并为其添加“AudioSource”组件，并且场景为“非运行状态”：

当组件中没有使用音频资源时，最终包大小为“20.8M”：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230520202319581.png" alt="image-20230520202319581" style="zoom:67%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230520202246327.png" alt="image-20230520202246327" style="zoom:67%;" />

当组件中直接使用音频资源，最终包大小为“33.2M”：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230520202603969.png" alt="image-20230520202603969" style="zoom:67%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230520202644028.png" alt="image-20230520202644028" style="zoom:67%;" />

**3)**.**<font color=red>核心情况验证：非运行状态，该组件中没有使用音频资源，但在运行状态时，动态加载指定资源文件为该组件赋值</font>**

**<font color=red>此时最终包大小为“20.8M”</font>**

经过此验证说明：对于项目中已经打成AB的资源，其确实可以减小“最终包的大小”。并且**<font color=red>这些资源并不需要删除，只要保证这些资源不在“Resources或StreamgingAssets文件夹”中，并且其不会与场景有“直接依赖”关系，则该资源必然不会重复打入“AB和最终包”中</font>**。

脚本如下：

```c#
using System.Collections;
using System.IO;
using UnityEngine;
using UnityEngine.Networking;

public class Test01 : MonoBehaviour
{
    string targetPath = "";
    private string assetName = "Red right hand.mp3";
    private bool isDone = false;

    void Start() {
        targetPath = Application.persistentDataPath + "/testab";
    }

    void Update()
    {
        if (Input.GetMouseButtonDown(0)) {
            if (!isDone) {
                isDone = true;
                CheckAudioClip();
            }
        }
    }

    void CheckAudioClip() {
        if (!File.Exists(targetPath)) {
            StartCoroutine(DownloadAudioClip());
            return;
        }

        SetAudioClip();
    }

    void SetAudioClip() {
        AssetBundle ab = AssetBundle.LoadFromFile(targetPath);
        AudioSource asCompnent = this.GetComponent<AudioSource>();
        asCompnent.clip = ab.LoadAsset<AudioClip>(assetName);
        asCompnent.Play();
    }

    IEnumerator DownloadAudioClip() {
        string downloadUri = "http://192.168.0.102/testab";
        UnityWebRequest m_UnityWebRequest = new UnityWebRequest(downloadUri);
        m_UnityWebRequest.downloadHandler = new DownloadHandlerBuffer();
        yield return m_UnityWebRequest.SendWebRequest();

        byte[] data = m_UnityWebRequest.downloadHandler.data;
        FileStream fs = new FileStream(targetPath, FileMode.Create, FileAccess.ReadWrite);
        fs.Write(data, 0, data.Length);
        fs.Close();

        SetAudioClip();
    }
}
```

用于验证的项目链接：https://download.csdn.net/download/m0_47975736/87803907





### 重要问题2：如何降低最终包的大小？

#### 从资源的角度：

##### 1.清除项目中没有使用的资源：

该步骤可以使用工具来完成：https://github.com/tsubaki/UnityAssetCleaner

该工具可以检测出项目中所有未被使用的资源列表，并提供自主删除指定资源的功能

注意：根据“Unity最终包的打包规则”，如果某个资源不在“Resources或StreamingAssets文件夹”中，并且也没有被任何场景直接使用，则其必然不会包含在“最终包”中。因此本步骤并不能显著的降低最终包的大小(因为这些未被使用的资源从来就没有被打入最终包中)，但执行此步骤后，项目在“import”的时间会大大缩短，有利于提升项目效率

##### 2.检测“BuildSettings”中的场景列表：

该列表的场景直接决定会将其“直接依赖的资源”打入最终包中，因此需要仔细排查其中不需要的场景

注意：针对场景中的各个GameObject，如果该场景功能较为简单，可以将其包含的所有GameObject集中在一个Prefab中。之后在游戏运行中“动态加载”该prefab。这样可以减少“打入最终包的场景数量”，最终包的大小也自然减少了

##### 3.纹理压缩和"音频/视频"质量压缩

注意：在将图片或音频视频导入到Unity时，如果该资源较大，则导入时间会很长；

在设置图片的格式或分辨率时，其并不会改变该图片原始的大小(无论在Unity中如何改变该纹理的分辨率，在“Windows资源管理器”中该图片的大小始终不会改变)

如在导入音频资源时，Unity会自动对该音频做压缩处理：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230520223158488.png" alt="image-20230520223158488" style="zoom:67%;" />

而该音频在“Windows资源管理器”中显示的仍为其初始大小：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230520223336155.png" alt="image-20230520223336155" style="zoom:67%;" />

并且压缩之后的“音频大小12.4M”才是该音频资源在“最终包”的大小：20.8M + 12.4M = 33.2M

**注意**：当选中某个资源后，其**在Inspector面板中显示的大小**通常都是该资源如果打入最终包，则在**最终包中的占用的大小**。如**“纹理图片”，在不同分辨率下该纹理会有不同的大小**，如果打入最终包中，则会使得最终包变大

**PS**：为了开发方便，通常会限定美术人员为各个不同功能的纹理切图时的大小。虽然即使切图很大也仍然可以在Unity中重新设置分辨率，但这样会增加程序人员的工作量，并且纹理导入的时间也会大大增加。

##### 4.打图集：

图集不仅可以降低Drawcall，也可以降低“该图集包含的所有纹理”占用的**<font color=red>总大小</font>**。将多个尺寸相近的纹理放入同一图集中，充分利用图集的空间，可以有效降低纹理占用的总大小(如果该图集包含在“最终包”中，则必然可以降低最终包的大小；如果其包含在AB中，则也可以降低AB的大小)



#### 从代码的角度：

##### 脚本后端：使用“IL2CPP”代替“Mono”

“Mono”方式(JIT编译)出包较快，但游戏运行速度慢，通常在开发过程中使用该方式；

“IL2CPP”方式(AOT编译)出包较慢，但游戏运行速度块，并且由于该种方式会裁剪掉项目中无需使用的代码，因此也可以降低“最终包”大小

PS：“IL2CPP”会默认设置“代码裁剪级别”为“Low”，并且无法关闭，其会自动将部分没有使用的代码裁剪掉。随着其“裁剪级别”的提升(最高为“High”)，有可能会将代码中“有用的代码”也裁剪掉。因此该设置保持默认即可，不要轻易改变

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230520225659217.png" alt="image-20230520225659217" style="zoom:67%;" />



#### 王牌方法：打AB

该方法是缩小“最终包”的万能方法，即将用到的资源打成AB，在运行时从“资源服务器”下载。

但这些资源不能放在“Resources或StreamingAssets文件夹”中，或被某个场景“直接依赖”，否则依然会被打入“最终包”中

**PS**：经过问题1的验证，在将这些资源打成AB后，并不需要将其从项目中删除，只要不放在“Resources/StreamingAssets文件夹”或被场景“直接依赖”，则其必定不会重复被打入“最终包”中



#### 基于以上规则，并为了提高开发效率，做如下限制：

1.对于游戏中需要动态加载的资源，统一使用“加载AB”的方式来获取，而不再使用“Resource.Load”的方式。

2.如果某个场景在“非运行状态”**<font color=red>必须要使用某些资源</font>**，则将这些资源放在“Resources文件夹”中，否则“Resource文件夹”并没有存在的必要

3.“StreamingAssets文件夹”中可以存放“已经打包完成的AB文件”。

PS：部分情况下不适宜从“资源服务器”上下载AB文件，则可以将这些AB文件放在“StreamingAssets文件夹”中。由于在打AB时已经进行了“压缩处理”，因此其占用的空间已经被大量缩小。所以如果影响不大，可以将这些AB直接放在“StreamingAssets文件夹”中

4.在“编辑器”下运行时，可以直接使用“AssetDatabase.LoadAsset”加载指定资源，不用打AB；

而在“指定目标平台”运行时，则统一使用“AssetBundle.Load”的方式加载指定资源



参考链接：https://www.cnblogs.com/jeferwang/p/14038849.html





### 重要问题3：如何获取每个Asset自身依赖的所有其他资源？

`AssetDatabase.GetDependencies(assetName, isRecursiveSearch)`：返回值为**<font color=red>string[]类型</font>**，专用于查找指定assetName所依赖的所有资源，返回这些资源的路径(**<font color=red>路径中包含“Assets文件夹”前缀</font>**)。

**注意**：返回的结果中会包含所有依赖的资源，可以根据需求做部分筛选(如依赖资源为场景scene或C#脚本文件，则忽略)

**PS**：

1.如果需要对字典中的元素按照key进行排序，则可以直接使用“SortDictionary<string, xxx>”类型：

`SortedDictionary<string, xx> m_Assets = new SortedDictionary<string, Asset>(StringComparer.Ordinal)`

2.从“包含Assets文件夹前缀的资源路径”获取到该资源的GUID：`AssetDatabase.AssetPathToGUID(assetName)`

3.从“GUID”获取到该资源的路径：`AssetDatabase.GUIDToAssetPath(string guid)`





### 重要问题4：在为一组图片资源创建图集spriteAtlas后，在设置AB时，是否需要手动将该图集文件添加到AB中？

**解答**：

经过实际验证，在为“目标图片资源”打图集spriteAtlas后，即使只为“目标图片资源”设置“AB标签”，完全没有为“新创建的spriteAtlas”设置相同的AB标签，**<font color=red>在打AB完成后，该AB中依然已经包含了“新创建的图集spriteAtlas”</font>**。由此说明，在打AB时，其实完全不需要手动设置图集文件，该文件会自动关联到AB中

**并且经过深层验证**：

**基础条件**：图片资源组中包含图片A,B,C，和图片D,E,F，并且已经为所有的图片A,B,C,D,E,F打图集spriteAtlasCommon

**执行情况**：分别将图片A,B,C打AB1， 图片D,E,F打AB2

**执行结果**：**<font color=red>查看AB1和AB2，发现无论哪个AB，其中都包含图集spriteAtlasCommon</font>**。

由此可知，在为图片资源打AB时，为了避免图集重复打包，应该正确设置该AB中包含的所有图片。从验证结果看，==若为某个图片打AB，则该AB中必然包含该图片所属的图集，以及该图片在本图集中的“尺寸、位置”等信息(保存在该图片的Sprite文件中)==

所以为了避免图集的重复打包，AB中最好包含该图集中的所有图片，不要有遗漏



#### 验证过程：

为目标图片资源打图集，但在设置中没有为该图集spriteAtlas设置“AB标签”或在“ResourceEditor界面”中没有把spriteAtlas放入AB中，此时AB的设置中只有“目标图片资源本身”

1.创建spriteAtlas图集：在“Project视图”中鼠标右键“Create -> 2D -> Sprite图集”(使用的Unity版本为“2021.3.14f1”)，在该图集spriteAtlas中添加“目标图片资源”

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230526163112787.png" alt="image-20230526163112787" style="zoom:67%;" />

2.设置“目标图片资源”的“AB标签”，但不为上述新创建的图集spriteAtlas添加“AB标签”。

PS：如果使用的是“ResourceEditor界面”，则不要把图集spriteAtlas放入AB中即可。

本步骤的目的在于：设置“AB中具体包含的内容”时，不要添加这些图片资源的图集，只包含“图片资源本身”

图片资源的“AB标签”：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230526163647181.png" alt="image-20230526163647181" style="zoom:67%;" />

这些图片资源打成的图集不设置“AB标签”：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230526163735981.png" alt="image-20230526163735981" style="zoom:67%;" />

3.在打AB完成后，使用UnityStudio工具查看ab包中的内容：

PS：AssetStudio工具：https://github.com/Perfare/AssetStudio，选择相应“.NetFramework”版本的文件即可。该工具支持AB中多种文件的查阅以及导出，非常方便

导入上述的AB文件：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230526163956159.png" alt="image-20230526163956159" style="zoom:67%;" />

得到该AB中包含的所有内容：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230526164508678.png" alt="image-20230526164508678" style="zoom:67%;" />

从上图可以看出，该AB中除了“目标图片资源本身”外，还自动包含了“打包的图集spriteAtlas”

为了更好的体现区别，针对“不打图集，直接对图片资源打AB”的情况继续验证，该种情况下得到的AB中包含的内容如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230526171730869.png" alt="image-20230526171730869" style="zoom:67%;" />



补充说明如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221122200220136.png" alt="image-20221122200220136"  />



### 资源的加载和卸载：

从我理解来看Resources是一个缺省自动打包的特殊AssetBundle。无论从WWW还是AssetBundle.CreateFromFile创建AssetBundle其实是创建了一个文件内存镜像。这时候是没有Asset的。AssetBundle.LoadAsset 和Resource.Load才真正创建出了Asset，而Instaniate复制了这个Asset。注意这个复制有两种，学C++的都知道浅拷贝和深拷贝，这里的复制有的是正真的复制，有的是引用。为什么要这样呢？因为有些游戏资源是只读的，像贴图Texture，这么大而且只读，当然不需要再去完全复制一份。但像GameObject这种资源它的属性是可以通过脚本改变的，必须要复制一份。所以一个资源从AssetBundle到场景中被实例化，其实有3块内存被创建，这3快内存的释放是有不同方法的。

文件内存镜像是通过AssetBundle.Unload(false)来释放的 。 Instaniate出来的Object内存通过Object.Destory来释放 。 AssetBundle.Unload(true)不单会释放文件内存镜像，还会释放AssetBundle.Load创建的Assets 。这个方法是不安全的，除非你能保证这些Assets没有Object在引用，否则就出问题了。 Resources.UnloadAsset和Resources.UnloadUnusedAssets可以用来释放Asse t。

下面这个图很直观：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231108203418734.png" alt="image-20231108203418734" style="zoom:80%;" />















