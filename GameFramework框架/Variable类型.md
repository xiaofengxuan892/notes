[TOC]



在开发中经常遇到这样的情况：**需要将不同类型的变量都放到同一个集合中，并且这些类型还同时包含“值类型”和“引用类型”**。**<font color=red>若直接使用“System.Object”作为基类来表示这些类型，则在放入集合中时必然会出现“装箱”和“拆箱”的性能消耗</font>**。

为了规避以上的问题，这里新增加“`Variable`”类型。并且为了兼容各个不同的类型，**<font color=red>以“Variable”为基类</font>**，扩展得到“`Variable<T>`”类型：

### 基础类型“`Variable`”和“`Variable<T>`”：

"`Variable.cs`"脚本如下：

```c#
//为了方便使用“引用池系统”，这里需要实现“IReference”接口。
//PS：如果框架中没有“引用池系统”，则无需实现该接口
public abstract class Variable : IReference
{
	public Variable() {}
    //注意：属性也可以是“abstract”，因此也可以被子类重写
	public abstract Type Type { get; }
    
    public abstract void Clear();
	public abstract object GetValue();
	public abstract void SetValue(object value);
}
```

以“`Variable`”类型为基类，扩展得到的“`Variable<T>`”类型：

```c#
//注意：如果“泛型T”没有限制，则无需使用“where T ：xx”语句
public abstract class Variable<T> : Variable
{
	private T m_Value;

	public Variable()
	{
		m_Value = default(T);
	}

	public T Value
	{
		get
		{
			return m_Value;
		}
		set
		{
            //在为“m_Value”赋值前，外部已经根据本属性的“T”限制将“value”进行了转换，
            //因此这里无需再次转换。
            //“SetValue”方法中，是需要对“外部传递过来的object类型对象”强转为“T”，
            //才能赋值给“m_Value”参数
			m_Value = value;
		}
	}
	
	public override Type Type
	{
		get
		{
			return typeof(T);
		}
	}

	//重写基类的“Clear”方法
	public override void Clear()
	{
		m_Value = default(T);
	}

	//部分情况下可能会用到该类型变量的string形式，因此重写“ToString()”方法
	public override string ToString()
	{
		//无论“m_Value”本身为何种类型，其自身必定有“ToString()”方法，因此直接调用即可
		return (m_Value != null) ? m_Value.ToString() : "<Null>";
	}

    //提供给外部，用于获取该类型对象value的方法
	//PS：外部在赋值时其实可以直接通过“Value”属性来实现，因此这两个方法的作用其实已经没有了
	//    之所以依然“override”这两个方法，是为了避免“Variable<T>”的子类依然为“abstract抽象类”
	//    所以这里重写这两个方法。但从实际作用来看，大多情况直接使用“Value”属性满足了效果，
    //    基本不会用到这两个方法，因此可以直接从“Variable基类”中删除
	public override object GetValue()
	{
		return m_Value;
	}
	public override void SetValue(object value)
	{
		m_Value = (T)value;
	}
}
```



#### 扩展问题：

**<font color=red>问题：“Variable基类”的内容极为简单，完全可以直接写在“Variable<T>”中，那为什么要将“Variable基类”单独分离出来呢？</font>**

**解答**：当外部声明某个类型的变量时，若直接使用“Variable<T>”则显得不大方便，**在显示上也不雅观**。因此单独提供“Variable基类”，外部使用“`List<Variable> m_List`”声明变量，则比使用“`List<Variable<T>> m_List`”声明变量要雅观得多，再者“T”也会限制集合只能存储一种类型的数据



### “`Variable<T>`”类型的扩展：

针对于各个不同的类型，包含“值类型：int, bool, char, byte等”，或“引用类型：GameObject, Material等”，**<font color=red>均可以继承“Variable<T>”类型，创建属于其自身需求的新类型“VarChar, VarInt32, VarGameObject, VarMaterial”等</font>**：

`VarChar.cs`类型如下：

```c#
public sealed class VarChar : Variable<char>
{
	//通过“隐式类型转换VarChar”方法将“char“类型变量转换为“VarChar”类型
	public static implicit operator VarChar(char value)
	{
		VarChar varValue = ReferencePool.Acquire<VarChar>();
		varValue.Value = value;
		return varValue;
	}

	//通过“隐式类型转换char”方法将“VarChar”类型变量转换成外部可以直接接受的“char”类型
	public static implicit operator char(VarChar value)
	{
		return value.Value;
	}
}
```

**<font color=red>“implicit operator”的作用</font>**：用于在“VarChar”类型和“Char”类型之间相互转换的“隐式类型方法”。其使用如下：

```c#
char a = 'h';
VarChar b = a;    //此时会“隐式调用”第一个“implicit operator”修饰的“VarChar”方法
```

 

### “`Variable<T>`”扩展类的实际使用：

当需要将各个不同类型的变量放到同一集合中时，如该集合为“List<Variable>”类型：

**在向集合中添加元素时**：如该元素本身为“char”类型，则使用“隐式转换VarChar”将其转换后放入集合中，即：

```c#
List<Variable> m_List = new List<Variable>();
char a = 'h';
VarChar b = a; //编译器自动调用“VarChar.cs”中第一个“隐式类型转换方法VarChar”
m_List.Add(b); //m_List中的元素类型为“Variable”，因此必然可以将“VarChar”类型变量添加入该集合
```

而**在从集合中获取元素时**：此时则必须要预先知晓目标元素的实际类型，之后选择相对应的“VarXXX”类型将其“隐式转换”

```c#
Variable d = m_List[0];   
//这里不会调用“隐式转换”：将Variable类型转成“VarChar”类型是“父类”和“子类”的关系
VarChar e = (VarChar)d; 
char f = e;  //编译器自动调用“VarChar.cs”中第二个“隐式类型转换方法char”
```

**注意**：

**1.如果框架中有“引用池系统”**，则需要在该模块接入。此时“Variable基类”需要实现“IReference”接口。

因此如果需要**更新集合中某个元素的数值**，则**<font color=red>需要将其“旧数值”使用“ReferencePool.Release”回收后，再为其赋新值</font>**；

并且**在清空该集合中的元素时**，**<font color=red>需要使用“ReferencePool.Release”将每个元素逐一回收，之后才能清空该集合</font>**。

**2.如果框架中没有“引用池系统”**，则“Variable基类”无需实现“IReference”接口。自然在更新集合中元素数值或清空该集合时均不需要调用“ReferencePool.Release”进行回收



### “隐式类型转换”和“显式类型转换”的区别：

两者均是本类型与其他类型之间相互转换的public static方法。其中使用“`implicit operator`”声明的为“隐式类型转换”方法，使用“`explicit operator`”声明的为“显式类型转换”的方法

**两者在使用方式上的区别**：

**“隐式类型转换”**：无需手动指定目标类型，编译器会遇到“特定类型转换”的情况时会自动调用该方法

**“显式类型转换”**：外部需要手动指定目标转换类型，否则编译器无法完成类型转换

**示例一**：正确的使用方法

- ```c#
  char a = 'h';
  VarChar b = a; 
  //编译器会默认调用“隐式类型转换方法VarChar”，自动将“char”类型变量转换成VarChar类型
  ```

**示例二**：编译器执行以下语句时会报错

- ```
  char a = 'h';
  VarChar c = (VarChar)a;  //执行该语句时报错
  ```

- **原因**：**编译器在执行“`VarChar c = (VarChar)a`”语句时会调用“显式类型转换方法VarChar”**。**<font color=red>但无论是“VarChar.cs”，还是“char.cs”中都没有将“char”类型转换成“VarChar”类型的“显式类型转换”方法</font>**，因此会报错

- **解决方案**：在“VarChar.cs”中增加“`public static explicit operator VarChar(char value)`”的显式类型转换方法，如此外部即可手动指定目标转换类型来调用该“显式类型转换方法VarChar”

**总结**：**<font color=red>“隐式类型转换”方法是编译器根据当前两个参数的类型自动调用的，不需要外部手动指定</font>**。**并且如果外部在参数前手动指定了目标类型，则编译器触发会“显式类型转换”方法完成类型转换**



#### 扩展问题：

**<font color=red>问题1：在变量前手动指定目标类型，是否一定会触发“显式类型转换”方法？</font>**

**解答**：**不一定**。**<font color=red>如果“变量的类型”与“目标类型”之间存在继承关系，则不会执行“显式类型转换方法”</font>**；**但若两个类型之间完全没有任何关系，则必然会执行对应的“显式类型转换方法”**。

示例一：

```c#
Variable a = 'h'; 
VarChar b = (VarChar)a;
```

由于“Variable”和“VarChar”之间存在间接继承关系，因此可以直接进行类型转换

示例二：

```c#
char a = 'h';
VarChar b = (VarChar)a;
```

由于“char”和“VarChar”类型之间没有任何直接或间接的关系，因此这里必然会调用“显式类型转换方法”。此时如果“char.cs”和“VarChar.cs”均没有相应的“显式类型转换方法”，则编译器会报错



**<font color=red>问题2：为什么本框架中的“VarXXX.cs”脚本都只有“implicit operator”修饰的“隐式类型转换”方法，而没有“explicit operator”显式类型转换方法？</font>**

**解答**：设计“Variable.cs”，“Variable<T>.cs”的目的在于方便将各种不同类型的变量放入同一集合中，同时为了减少外部使用“VarChar, VarInt32, VarGameObject”等新类型的成本，其并不需要手动与“char, int, GameObject”类型进行转换。如：

```c#
List<Variable> m_List = new List<Variable>();
//将新元素放入集合：
char a = 'h';
VarChar b = a; 
m_List.Add(b);

//从集合中获取元素值：
VarChar c = (VarChar)m_List[0];
char d = c;
```

**<font color=red>并且可通过“泛型T”将以上过程整理成独立的方法提供给外部</font>**：

**向集合中放入新元素**：

```c#
public void SetData<T>(T m_Value) where T : Variable
{
    m_List.Add(m_Value);
}

//调用该方法
char a = 'h';
SetData<VarChar>(a);  
//内部会调用“VarChar.cs”中“隐式类型转换VarChar方法”自动将“char”类型变量“a”转换成“VarChar”类型，
//这一步是编译器自动执行的，
//这也是在“VarChar.cs”中使用“implicit operator”，而非“explicit operator”的原因所在
```

**PS**：如果是使用Dictionary存储Variable类型的数据，那么在更新原有key的数据时，需要先使用“ReferencePool.Release”将原有的数据回收，然后才能设置新的数据，如“ReferencePool.Release(dict[name])”，然后设置“dict[name] = 新值”

**从集合中获取元素值**：

```c#
public T GetData<T>(int index) where T : Variable {
    return (T)m_List[index];
}

//调用该方法
char b = GetData<VarChar>(0);  
//虽然“GetData”方法返回的值为“VarChar”类型，但其会自动调用“VarChar.cs”中的“隐式类型转换方法char”
//这一步是编译器自动执行的，非常方便。
//这也是在“VarChar.cs”中使用“implicit operator”，而非“explicit operator”的原因所在
```



### 补充说明：

**<font color=red>任何常用类型均可通过“`Variable<T>`”创建可共存于同一集合的“Variable”新类型</font>**

现列出部分常用类型：

“`VarInt32.cs`”，即**针对常用的“int”类型**：

```c#
public sealed class VarInt32 : Variable<int>
{
	//“隐式转换VarInt32”：将“int”类型变量转换成“VarInt32”类型
	public static implicit operator VarInt32(int value)
	{
		VarInt32 varValue = ReferencePool.Acquire<VarInt32>();
		varValue.Value = value;
		return varValue;
	}

	//“隐式转换int”：将“VarInt32”类型变量转换成“int”类型
	public static implicit operator int(VarInt32 value)
	{
		return value.Value;
	}
}
```

"`VarInt16.cs`"，即**<font color=blue>针对常用的“short”类型</font>**：

```c#
public sealed class VarInt16 : Variable<short>
{
	//"隐式转换VarInt16"：将“short”类型变量转换成“VarInt16”类型
	public static implicit operator VarInt16(short value)
	{
		VarInt16 varValue = ReferencePool.Acquire<VarInt16>();
		varValue.Value = value;
		return varValue;
	}

	//“隐式转换short”：将“VarInt16”类型变量转换成“short”类型
	public static implicit operator short(VarInt16 value)
	{
		return value.Value;
	}
}
```

同理“`VarInt64.cs`”，即**<font color=blue>针对常用的“long”类型</font>**。实现逻辑与上述一致，这里不再赘述



"`VarGameObject.cs`"，即**<font color=blue>针对常用的“GameObject”类型</font>**：

```c#
public sealed class VarGameObject : Variable<GameObject>
{
	//“隐式转换VarGameObject”：将“GameObject”类型变量转换成“VarGameObject”类型
	public static implicit operator VarGameObject(GameObject value)
	{
		VarGameObject varValue = ReferencePool.Acquire<VarGameObject>();
		varValue.Value = value;
		return varValue;
	}

	//“隐式转换GameObject”：将“VarGameObject”类型变量转换成“GameObject”类型
	public static implicit operator GameObject(VarGameObject value)
	{
		return value.Value;
	}
}
```

"`VarMaterial.cs`"，即**<font color=blue>针对常用的“Material”类型</font>**：

```c#
public sealed class VarMaterial : Variable<Material>
{
	//“隐式转换VarMaterial”：将“Material”类型变量转换成“VarMaterial“类型
	public static implicit operator VarMaterial(Material value)
	{
		VarMaterial varValue = ReferencePool.Acquire<VarMaterial>();
		varValue.Value = value;
		return varValue;
	}

	//”隐式转换Material“：将”VarMaterial“类型变量转换成”Material“类型
	public static implicit operator Material(VarMaterial value)
	{
		return value.Value;
	}
}
```







