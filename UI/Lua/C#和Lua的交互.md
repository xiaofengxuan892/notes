[TOC]



## Lua和C#语言的数据交互：

### C#层：

1.**<font color=red>为每一个需要传递到Lua中的csobj生成唯一id</font>**

2.**<font color=blue>在C#层建立数据集合</font>**，用于记录csobj与唯一id之间的关系，可以直接通过csobj查找到其id

3.<font color=red>**将唯一id作为参数传递到Lua层**</font>





### Lua层：

压入csobj过程：

1.创建userdata空间：

`lua_newuserdata`：

在内存中分配指定size的内存块，并将该内存块的地址作为“<font color=blue>LUA_TUSERDATA</font>”入栈(<font color=blue>栈中数据支持的格式</font>)，并返回该地址的<font color=blue>指针形式</font>(由于是C语言，故使用指针形式)，并将csobj的唯一id放入该内存块中

```c
int* pointer = (int*)lua_newuserdata(L, sizeof(int))
*pointer = csobjId
```

注意：这里**<font color=red>入栈的是内存块的地址</font>**，**<font color=blue>虽然是封装成“userdata”的形式，但依然只是地址，而并非内存块或其他内容</font>**

2.在注册表中创建缓存表cache_ref， 用于统一管理所有csojb：

```c
//将注册表中缓存表的值入栈
lua_rawgeti(L, LUA_REGISTRYINDEX, cache_ref);
//将索引“-2”的“userdata”拷贝副本并入栈
lua_pushvalue(L, -2);
//在缓存表中添加“csobjId”键值对
lua_rawseti(L, -2, csobjId);
//设置完毕后，将缓存表出栈，恢复栈初始配置
lua_pop(L, 1);
```

3.为该userdata设置元表：

PS：需要预先获取注册表中元表key： meta_ref

```c
//将注册表中元表的值入栈
lua_rawgeti(L, LUA_REGISTRYINDEX, meta_ref);
//为该userdata设置元表
lua_setmetatable(L, -2);
```

#### 问题：

1.如果userdata中实际存放的是csobjId，那为什么不直接使用LUA_TNUMBER存放即可？

答：如果直接使用LUA_TNUMBER，那么如何将普通的number与代表csobjId的number区分开，所以不能直接使用TNUMBER类型

2.如果需要单独定义一种数据类型，为什么需要在内存中单独分配空间，直接使用int的寄存器存储id即可？

答：使用userdata可以将该id的引用置空，用来检测该obj当前是否压入栈，同时根据这个来处理C#层obj本身的释放







<font color=red>**个人体会**</font>：Lua从实际作用来看，只是一种注释性质的说明，其本质全部会调用C/C++编写的“lua/xlua.dll”中的方法，因此在实际执行时依然会使用C语言中的数据类型。

而Lua语言实际是从“dll”中提取出来一些约定俗成的东西，以方便使用，本质上并不具有脚本语言的一些特性。所以这也就导致其能够完全的跨平台，且无需编译——因此Lua只是注释性质的语言，因此自然无需编译



###### Lua直接使用C#传递过来的数据：

lua有时会直接使用从某些C#方法返回中的值，该值有可能是Array, List等。由于“Array, List”是“userdata”类型，是无法在lua中直接调用的

如：

```lua
for _, v in pairs(imageList) do
	v.grayed = true
end
```

由于“pairs/ipairs”只能对table类型的数据集合进行操作，对于C#返回的”userdata“，**<font color=red>如果使用如上方式则会直接报错</font>**

**<font color=red>解决办法：使用“for i = xx,  do”遍历“userdata”数据集合</font>**

注意：从C#中返回的“userdata”集合<font color=red><b>索引值默认从0开始</b></font>，并且在lua中“for i = xx1, xx2 do”在遍历时，**<font color=red>“i”的数值是可以取到“xx2”最大值的</font>** —— 这是与C#语法不同之处

所以Lua中“for i = xx, do”遍历的索引值通常从“1”开始

```lua
local function GrayAllImage(imageList)
	for i = 1, imageList.Count do
		local imageElement = imageList[i - 1]
		if imageElement.enabled then
			imageElement.grayed = not isUnlock and not isStagePassed
		end
	end
end
```

<b><font color=blue>PS: 由于使用的是C#中的数据集合，因此可以直接获取该“userdata”集合中的属性值，如“list.Count”, </font></b><font color=red>并且完全按照C#中的规则获取该userdata中的所有属性值</font>



##### 反射在“C#和Lua”中的使用：

###### 静态反射(俗称“去反射”)：

把所有的c#类的public成员变量、成员函数，都导出到一个相对应的Wrap类中，而这些成员函数通过特殊的标记，映射到lua的虚拟机中，当在lua中调用相对应的函数时候，直接调用映射进去的c# wrap函数，然后再调用到实际的c#类，完成调用过程。

因为反射在效率上存在不足，所以通过wrap来提升性能。但是因为wrap需要自己去wrap，所以在大版本更新是可以用到的，小版本更新还是使用“动态反射”。

###### 动态反射的使用方式：

**反射在Lua中的使用**：

```c#
private string script = @"
    luanet.load_assembly('UnityEngine')
    luanet.load_assembly('Assembly-CSharp')
    GameObject = luanet.import_type('UnityEngine.GameObject')        
    ParticleSystem = luanet.import_type('UnityEngine.ParticleSystem')         

    local newGameObj = GameObject('NewObj')
    newGameObj:AddComponent(luanet.ctype(ParticleSystem))
";

//反射调用
void Start () {
    LuaState lua = new LuaState();
    lua.DoString(script);
}
```

可看到通过反射（System.Reflection.Assembly）把UnityEngine程序集加入到lua代码中，通过反射（System.Type）把Unity.GameObject和Unity.ParticleSystem类型加入到lua代码中，这样我们便可以在lua中像在C#里一样调用Unity中的类型了

**C#调用Lua文件中的具体某个方法**：

使用“DoFile”读取Lua文件，然后用“GetFunction”获取具体的“Lua方法”，并调用“Call”执行该Lua方法。如果该方法需要外部传入参数，则可将参数通过“Call(.....)”传递到Lua方法内

```c#
public class OtherFunc : MonoBehaviour
{
     void Start()
     {
         LuaState m_state = new LuaState();
         string path = Application.dataPath + "/Lua/Custom/OtherFuncLua.lua";  //重点在于该路径下确实能找到该Lua文件
         m_state.DoFile(path);  //只有先执行后才能调用Lua脚本中的方法或变量
         m_state.GetFunction("HelloMethod").Call();   
         //其实这里存在一定同名函数的问题：当使用DoFile加载了多个lua文件，而这些lua文件中有同名方法，默认执行最后一个DoFile文件中的同名方法
     }
}
```



**反射在C#中的使用**：

```c#
using UnityEngine;
using System.Reflection;
using System.Collections;

public class OthersFunc : MonoBehaviour
{
    public string GetUserID(int id)
    {
        return "{" + id + "}";
    }

    void Start()
    {
        Debug.Log(this.GetType() + "    @@@@@@@@  ");
        MethodInfo vMethodInfo = this.GetType().GetMethod("GetUserID");
        string returnStr = (string)vMethodInfo.Invoke(this, new object[] { 28 });
        Debug.Log(returnStr);
    }
}
```

输出结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107174601628.png" alt="image-20231107174601628" style="zoom:80%;" />

解析：借助 this.GetType() 来获取脚本中的方法，属性，参数等信息：`this.GetType().GetMethod("methodName")`

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107174652364.png" alt="image-20231107174652364" style="zoom:80%;" />

###### BindingFlags在反射中的作用：

在反射中加入”BindingFlags“用于限定反射中所得到的方法或属性等的范围：

表示访问权限：BindingFlags.Public —— 目标类型中public修饰的；

​            BindingFlags.NonPublic —— 非public修饰的，指所有private/protected/internal修饰的

表示基于实例对象或类型本身的static：BindingFlags.Instance —— 基于实例；BingdingFlags.Static —— 基于类型本身的static

**注意：当方法前面既有public，又有static时，优先以BindingFlags.static来决定,其他标记无法得到该static方法**

```c#
......
public static void Method(){....}
......

MethodInfo[] methodArray = typeof(xxx).GetMethods(BindingFlags.Public | BindingFlags.Instance);
```

以上”BindingFlags.Public | BindingFlags.Instance“的组合形式则无法得到目标static方法

解决方法：

组合方式改为：”BindingFlags.Public | BindingFlags.Static“

**针对于static方法的获取，必须组合中包括”BindingFlags.Static“**

表示基于派生类和子类的关系：BindingFlags.DeclaredOnly, 表示只包含本类型自身定义的方法属性等，父类的方法属性不包含。当没有该标记时，则会默认包含所有父类的方法属性 —— 针对于类的继承特性，默认所有对象的基类是Object，所有Object中所有的方法都会包含在内。

**以上三种分类起到相互补充的作用：**

```c#
BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly 
//只局限本派生类，基于实例对象的public方法
```

之间的操作符”|“并不是简单的”或“的关系，三者是逐步补充限制范围的作用

```c#
BindingFlags.Public | BindingFlags.NonPublic| BindingFlags.Instance | BindingFlags.DeclaredOnly
//只局限于此类本身，不包含父类，基于实例对象的所有public/private/protected/internal方法
```

注意：组合的方式中”BindingFlags.Public “或”BindingFlags.NonPublic“一定要配合”BindingFlags.Instance“ 或”BindingFlags.Static“使用

**两种不同的范围划定方式缺一不可**

```c#
BindingFlags.Public | BindingFlags.NonPublic
BindingFlags.Static| BindingFlags.Instance   
//以上两种组合方式无法得到任何结果
```

具体案例：

```c#
using UnityEngine;
using System.Collections;
public class ReflectionFunc : MonoBehaviour {
        public int num = 100;
        private float time = Time.realtimeSinceStartup;
        public static void RefectionMethod()
        {
               Debug.Log("STATIC METHOD .......");
        }
        void Start () {
        
        }
        void Update () {
        
        }
        public void ReflectionMethod1()
        {
               Debug.Log("PUBLIC METHOD .....");
        }
        private void ReflectioMethod2()
        {
               Debug.Log("PRIVATE METHOD .......");
        }
}
```

调用方式：

```c#
 ...........
//调用方式一：
ReflectionFunc refFunc = new ReflectionFunc();
GetReflectionMethodInfo(refFunc);
..........

void GetReflectionMethodInfo(ReflectionFunc refFunc)
{
    MethodInfo[] methodArray =  refFunc.GetType().GetMethods(BindingFlags.Public | BindingFlags.Instance |  BindingFlags.DeclaredOnly);
    for(int i = 0; i < methodArray.Length; ++i)
    {
                  Debug.Log(methodArray[i].Name);
    }
}

//调用方式二：
MethodInfo[] methodArray =  typeof(ReflectionFunc).GetMethods(BindingFlags.Public | BindingFlags.Instance |  BindingFlags.DeclaredOnly);
```

以上两种调用方式执行结果都是一样的：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107175131834.png" alt="image-20231107175131834" style="zoom:80%;" />

由此也可见”BindingFlags.Instance“的作用并不局限于是否有实际的实例对象，而在于类本身的方法或变量的定义方式





## 在项目中导入XLua：

1.下载最新的XLua版本：https://github.com/Tencent/xLua

注意：直接下载源码工程，在Unity中会用到源码中的“Assets”, "Tools"文件夹，不是下载右侧的“Release”版本

2.将源码工程“Assets”下的“Plugins”, “XLua”以及相应的meta文件共4个，都拷贝新建Unity工程的Assets目录下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221126145446089.png" alt="image-20221126145446089" style="zoom: 67%;" />

将源码工程中“Tools”文件夹拷贝到新建Unity工程的Assets同级目录下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221126150014450.png" alt="image-20221126150014450" style="zoom: 67%;" />

<font color=red>**注意："Plugins", "XLua"文件夹是Assets的子文件夹，"Tools"是Assets的同级文件夹，和源码工程中存放的目录结构一致**</font>

3.新建的Unity工程在导入xlua文件后，在“Build Settings/Player Setting”添加宏定义`HOTFIX_ENABLE`：
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221126150726948.png" alt="image-20221126150726948" style="zoom: 67%;" />

4.菜单栏“XLua”中先点击“GenerateCode”，在右下角编译完成后，再点击“Hotfix Inject In Editor”：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221126151124949.png" alt="image-20221126151124949" style="zoom:80%;" />

当Console中出现如下日志则代表导入成功，可以使用XLua正常开发项目了：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221126151312126.png" alt="image-20221126151312126" style="zoom:80%;" />

