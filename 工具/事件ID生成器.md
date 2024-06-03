[TOC]



### 事件ID生成器

项目中有的时候需要为某个特定的参数设定唯一id，便于根据id来直接查找。如消息或者事件系统需要根据id来查找到该事件的所有监听并执行

#### Lua：

`EventDef.lua`脚本：

```lua
local __eventId = 0
local __NewEventId = function()
    __eventId = __eventId + 1
    return __eventId
end

EventDef.GameState.OnEnterState = __NewEventId() 
-- 该语句在执行本脚本时会自动执行，因此“OnEnterState”会被赋值。
-- 但后续如果只是调用本脚本中的“OnEnterState”参数则不会导致“ __NewEventId()”再次执行
```

**解析**：在游戏初始化时，该EventDef.lua文件会被执行一次，因此`EventDef.GameState.OnEnterState`会被赋值。之后在需要发送“OnEnterState”事件时会调用“`EventDef.lua`”脚本中的“`GameState.OnEnterState`”参数，

```lua
sq.facade:GetEventDispatcher():DispatchEvent(EventDef.GameState.OnEnterState, state)
```

但此时只是使用左边的参数，并不会导致右边的`__NewEventId()`被执行，因此该事件的ID值并不会改变

所以针对每个事件来讲，在游戏初始化时，其ID都已固定，后续不会再修改



#### C#：

ID生成工具：

```c#
public class UniqueIdGenerator
{
	private static int _id;
	public static int Id => ++_id;
}
```

**解析**：在使用`UniqueIdGenerator.Id`时，**<font color=red>为了避免同一事件类型时该方法重复执行</font>**，**<font color=red>需要预先使用static参数存储该id数值</font>**，这里其实是利用static修饰的变量的特性：==其在第一次提及该类型时会自动为static变量执行初始化赋值，并且该“初始化赋值”只会执行一次==，因此保证了“事件ID”的唯一性。

**PS**：基于此特性，**直接将“mEventId”声明为public也可以**。但是**<font color=red>为了保证外部绝对没有权限修改该数值</font>**，因此声明为private，同时在“EventId”属性中只提供“get”方法

##### 实现过程：

1.事件类型封装：

```c#
//事件基类
public abstract class BaseEventArgs{
    public abstract int EventId{
        get;
    }
}

//下载成功事件
public class DownloadSuccessEventArgs : BaseEventArgs {
    private static int mEventId = UniqueIdGenerator.Id;
    public override int EventId{
        get{ return mEventId;}
    }
    ........
}
```

2.调用该事件：

```c#
private int i = 0;
void Update(){
    if(Input.GetMouseButtonDown(0)){
        ++i;
        DownloadSuccessEventArgs e = new DownloadSuccessEventArgs();
        Debug.Log("第{0}次发送事件，该事件的Id为{1}", i, e.EventId);
    }
}
```

运行结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230926132731647.png" alt="image-20230926132731647" style="zoom:80%;" />

##### 易错点：

**<font color=red>绝对不能使用如下形式</font>**：

```c
public class DownloadSuccessEventArgs : BaseEventArgs {
    public override int EventId{
        get{ return UniqueIdGenerator.Id;}
    }
    ........
}
```

该方式每次获取“EventId”时都会再次调用`UniqueIdGenerator.Id`，导致数值不断递增

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230926134245814.png" alt="image-20230926134245814" style="zoom:80%;" />



#### 生成事件ID的新方法：

使用该事件的HashCode作为事件ID：

```c#
private static int mEventId = typeof(DownloadSuccessEventArgs).GetHashCode();
```

哈希码是根据对象的内容由系统内部计算出来，当两个对象Equals时，则其哈希码也相同。但哈希算法存在一定的限制和随机性，因此部分情况下两个对象不Equals，其HashCode也有可能相同，这种情况称为“哈希冲突”

除了类型外，单独的变量也可以使用HashCode：

```c#
int num1 = 100;
int num2 = 20;
Debug.LogFormat("num1的HashCode： {0}， num2的HashCode： {1}", num1.GetHashCode(), num2.GetHashCode());
string str1 = "hello";
string str2 = "world";
Debug.LogFormat("str1的HashCode： {0}， str2的HashCode： {1}", str1.GetHashCode(), str2.GetHashCode());
```

运行结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230926165352303.png" alt="image-20230926165352303" style="zoom:80%;" />

**总结**：大多情况下使用对象的HashCode作为标识该对象的方法是正确的，但存在极少数“哈希冲突”的情况，可根据当时需求决定







