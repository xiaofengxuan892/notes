[TOC]





### 析构函数、Finalize、Dispose、Close的区别



#### 析构函数与“Finalize”方法的关系：

**<font color=red>析构函数的“内部实现逻辑”</font>**中会生成一个针对该类型对象的“Finalize”方法，该方法用于释放对象占用的内存资源，但其释放过程**<font color=red>很缓慢</font>**，**<font color=red>且会占用额外的时间和内存，影响性能</font>**



#### 析构函数和“Dispose”方法的区别：

**相同点**：两者的效果都是释放对象占用的内存资源，因此只需要使用其中一种即可满足需求

**不同点**：

1.前者只能由GC调用，外部无法手动调用“析构函数”；而后者只能由程序员手动外部调用。

2.虽然两者都可以用来释放对象占用的内存资源，但由于GC执行逻辑复杂，效率较低，因此为了降低GC的工作量，通常使用“Dispose”方法释放该对象占用的资源，之后通过`GC.SuppressFinalize(this)`告知GC：在回收该对象时不需要再调用其“析构函数”



#### 哪些对象需要手动调用“Dispose”方法？

针对**<font color=red>文件流如“FileStream, StreamWriter”，网络连接如“UnityWebRequest, Socket”，数据库连接如“SqlConnection,OleDBDataReader”等“非托管类型”</font>**的对象，**由于其占用大量内存，如果不及时释放，可能会导致“内存泄漏”和“游戏性能”问题**。此时则需要手动调用“Dispose”方法

**虽然“析构函数”中也可以对该类型对象进行释放**，但由于GC过程较为繁琐且效率低，为避免增加GC负担，提升游戏性能，通常不会依赖“析构函数”释放其资源占用。

**注意**：

1.**<font color=red>C#中“80%”的类型均为“托管类型”，而“托管类型”会自动释放内存占用</font>**。**当该类型对象不再使用时会自动释放资源，不会等到“GC”来释放**。因此“托管类型”对象完全无需考虑释放过程

2.所有的值类型均是“托管类型”。“引用类型”中有一部分也是“托管类型”，只有一小部分属于“非托管类型”。**<font color=red>针对“非托管类型”的对象，其释放需要手动调用“Dispose”，以减轻GC负担，提供游戏性能</font>**

3.**若某类型中包含以上FileStream, UnityWebRequest等“非托管资源”**，则**<font color=red>该类型需要实现“IDispose”接口，以向外部提供“Dispose”方法用于释放该类型对象占用的资源</font>**。然后在“Dispose”方法体中专门释放“FileStream, UnityWebRequest等非托管资源”



#### 使用"Dispose"的过程：

若要向外部提供“Dispose”方法，则**<font color=red>该类型需要实现“IDispose”接口，并实现其“public void Dispose()”方法</font>**。但对于“Dispose方法”其实**包含两种形式**：

```c#
//提供给外部的public dispose方法，“IDispose”接口中的方法，并且子类不能重写该方法
public void Dispose(){
    Dispose(true);
    //通知GC“该类型的对象在回收时无需调用析构方法”，因为这里已经手动释放了其资源占用
    GC.SuppressFinalize(this);  
}

//每个对象中都包含的隐藏“Dispose(bool disposing)”方法，其为“protected”修饰，并且为“virtual”方法，因此可以被子类重写
protected virtual void Dispose(bool disposing){
    if(m_Disposed)
        return;
    
    if(disposing){
        //释放“UnityWebRequest”,"FileStream"等变量占用的资源
    }
    
    m_Disposed = true;  //标记该对象已经被释放过了，无需GC通过“析构函数再次调用”
}

```

**第二种隐藏的“protected virtual void Dispose(bool disposing)”方法**可以被两个地方调用：

**1**.**在外部调用的“public void Dispose()”的方法体内调用本方法**，并且传递参数“true”，即“**<font color=red>Dispose(true)</font>**”

**2**.**<font color=red>在“析构函数”的“内部逻辑Finalize”方法体中调用“本隐藏dispose方法”</font>**，并且传递参数“false”，即“**<font color=red>Dispose(false)</font>**”

**注意**：

1).在外部调用的“public void Dispose()”方法体中，使用“Dispose(true)”释放内存资源后，**<font color=red>还需要增加“GC.SuppressFinalize(this)”语句以通知GC</font>**，**否则GC依然会调用该对象的“析构函数”**。虽然程序不会报错，但会增加GC负担，影响游戏性能

2).为防止外部调用“public void Dispose()”方法后，GC依然调用该对象的“析构函数”来释放资源占用，这里**<font color=red>增加“标记位m_Disposed”</font>**：只要外部调用过“public void Dispose()”方法，则**<font color=red>设置“m_Disposed = true”</font>**，以此减轻GC负担



#### 外部使用“Dispose”方法的方式：

1.直接调用该类型提供的“public void Dispose()”方法

2.**<font color=red>使用“using语句”，当该类型对象超出“自身作用域”时，则会自动将该对象设置为null，但在离开该作用域前依然需要手动调用public void Dispose()方法</font>**，如：

```c#
using(FileStream fs = new FileStream(path, ......)){

    fs.Dispose();
}
```



#### “Dispose”与“Close”的区别：

前者“Dispose”用于释放该对象的资源占用，但**<font color=red>后者"Close"仅仅只是一个普通的自定义方法而已，并不具备强制约束力</font>**。如“FileStream.Close”，其==只是将该文件流关闭，但后续依然可以再次被打开，并且其占用的内存资源也没有释放==。

各个不同类型中有没有Close方法都没关系，也**可以在Close中自由定义逻辑，和普通的自定义方法一样，不具强制约束力**

















