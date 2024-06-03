[TOC]



在开发中经常需要将某些常用的类型变量进行组合，形成新的类型。**<font color=red>该类型会作为某些变量的唯一标识</font>**。如：

### TypeNamePair

在“FSM系统”中，每个owner可以有多个Fsm实例，并且各个Fsm实例的name不同。如果只使用“name”来唯一标识该Fsm实例，则需要“name”有尽可能多的取值范围，否则很容易被用尽。

**为了增加“name”的适用范围，同时方便外部查找该Fsm实例**，**<font color=red>这里将“owner”与“name”组合成“TypeNamePair”对象，以此来唯一标识Fsm实例</font>**

**注意**：

1.对于这样需求组建而成的新类型，**<font color=red>通常直接声明为“struct结构”，而非“class”</font>**

2.由于该类型主要用于“唯一标识某个对象”，因此必须实现“`IEquatable<T>`接口”。**<font color=red>该接口包含用于比较两个对象是否相等的“Equals”方法</font>**。并且**<font color=red>部分情况下会直接使用“关系运算符”来比较该类型对象</font>**，因此也需要在“新类型”中提供“`operator ==`”和“`operator !=`”两个方法

3.**<font color=red>C#中任何类型，无论其为“值类型”或“引用类型”，都会包含"Equals(object obj)", "ToString()", "GetHashCode()"方法</font>**。因此可根据新类型的不同特性，重写这些方法



`TypeNamePairs.cs`模板如下：

```c#
[StructLayout(LayoutKind.Auto)]     
internal struct TypeNamePair : IEquatable<TypeNamePair>
{
    //注意：这些参数均为“readonly”修饰，在创建该类型变量并为各个参数赋值后，则不能再改变其值
	private readonly Type m_Type;
	private readonly string m_Name;

	public TypeNamePair(Type type, string name)
	{
		m_Type = type;
		m_Name = name ?? string.Empty;
	}

    //实现IEquatable<T>"接口中的“Equals”方法
	public bool Equals(TypeNamePair value)
	{
		return m_Type == value.m_Type && m_Name == value.m_Name;
	}

    //部分情况下会直接使用“关系运算符”比较两个“TypeNamePair”类型对象，
    //因此也需要为本类型添加“operator ==”和“operator !=”方法，并且使用“static”修饰
    //至于该方法的内部实现逻辑，则可以直接调用“IEquatable<T>”接口中的“Equals”方法
	public static bool operator ==(TypeNamePair a, TypeNamePair b)
	{
		return a.Equals(b);
	}
	public static bool operator !=(TypeNamePair a, TypeNamePair b)
	{
		return !(a == b);
	}

    //C#中任何object对象，无论其为“值类型”或“引用类型”都包含“Equals”,"ToString","GetHashCode"     
    //因此这里针对新类型的不同特性，将“基类System.Object”中的这些方法重写
    
    //1.重写“object基类”的“Equals”方法
    //注意：该方法的参数是“object”类型，而非“IEquatable<T>”接口中的“T”类型
    public override bool Equals(object obj)
	{
		return obj is TypeNamePair && Equals((TypeNamePair)obj);
	}
    //2.重写“object基类”的“ToString”方法
	public override string ToString()
	{
		string typeName = m_Type.FullName;
		return string.IsNullOrEmpty(m_Name) ? typeName : Utility.Text.Format("{0}.{1}", typeName, m_Name);
	}
	//3.重写“object基类”的“GetHashCode”方法
	public override int GetHashCode()
	{
        //对于本类型对象的“HashCode”，可通过将各个参数变量，“按位”执行“异或”操作
		return m_Type.GetHashCode() ^ m_Name.GetHashCode();
        //如果有三个参数，则将这三个变量“按位异或”即可
        //PS：该方式需要记住，所有这样需求的类型都会使用这种方式计算“新类型的HashCode”
	}

	#region 属性
    public string FullName{
        get{
            return ToString();
        }
    }
    
	public Type Type
	{
		get
		{
			return m_Type;
		}
	}

	public string Name
	{
		get
		{
			return m_Name;
		}
	}
	#endregion
}
```



### ResourceName

部分情况下，对于这些新声明的类型，**除了需要比较该类型的对象是否相等外，还需要对该类型的对象进行排序**。**<font color=red>前者只关注两者是否相等</font>**，而**<font color=red>后者还需要知道两者之间具体的大小关系</font>**

对于该需求，则需要实现“`IComparable`”接口和“`IComparable<T>`”接口

**示例**：在GameFramework框架的“Resource模块”中，由于需要对多个Resource对象进行排序，因此实现以上两个接口

`ResourceName.cs`模板如下：

```c#
[StructLayout(LayoutKind.Auto)]
private struct ResourceName : IComparable, IComparable<ResourceName>, IEquatable<ResourceName>
{
    //均为“readonly”修饰
    private readonly string m_Name;
    private readonly string m_Variant;
    private readonly string m_Extension;

    public ResourceName(string name, string variant, string extension)
    {
        m_Name = name;
        m_Variant = variant;
        m_Extension = extension;
    }

    //实现“IEquatable<T>”接口中的“bool Equals(T other)”方法
    public bool Equals(ResourceName value)
    {
        return string.Equals(m_Name, value.m_Name, StringComparison.Ordinal)
               && string.Equals(m_Variant, value.m_Variant, StringComparison.Ordinal)
               && string.Equals(m_Extension, value.m_Extension, StringComparison.Ordinal);
    }

    //部分情况下会直接使用“关系运算符”来比较两个“ResourceName”对象，
    //因此专门提供“operator ==”和“operator !=”方法
    public static bool operator ==(ResourceName a, ResourceName b)
    {
        return a.Equals(b);
    }
    public static bool operator !=(ResourceName a, ResourceName b)
    {
        return !(a == b);
    }

    //实现“IComparable<T>”接口中的“int CompareTo(T other)”方法
    public int CompareTo(ResourceName resourceName)
    {
        //“string.CompareOrdinal”方法会直接返回两个string对象的比较结果：
        //若不为0，则说明两个对象不等; 若小于0，则说明第一个参数在第二个参数之前；反之，则之后
        int result = string.CompareOrdinal(m_Name, resourceName.m_Name);
        if (result != 0)
        {
            return result;
        }

        result = string.CompareOrdinal(m_Variant, resourceName.m_Variant);
        if (result != 0)
        {
            return result;
        }

        return string.CompareOrdinal(m_Extension, resourceName.m_Extension);
    }
    
    //实现“IComparable”接口中的“int CompareTo(object obj)”方法
    public int CompareTo(object value)
    {
        if (value == null)
        {
            return 1;
        }

        if (!(value is ResourceName))
        {
            throw new GameFrameworkException("Type of value is invalid.");
        }

        return CompareTo((ResourceName)value);
    }

    //重写“System.Object基类”的“Equals”，“ToString”，“GetHashCode”方法：

    //1.重写“object基类的Equals”方法，其内部可通过“IEquatable<T>”中的“bool Equals(T other)”方法来实现
    public override bool Equals(object obj)
    {
        return (obj is ResourceName) && Equals((ResourceName)obj);
    }
    //2.重写“object基类的ToString”方法
    public override string ToString()
    {
        return FullName;
    }
    //3.重写“object基类的GetHashCode”方法
    public override int GetHashCode()
    {
        if (m_Variant == null)
        {
            return m_Name.GetHashCode() ^ m_Extension.GetHashCode();
        }

        return m_Name.GetHashCode() ^ m_Variant.GetHashCode() ^ m_Extension.GetHashCode();
    }

    #region 属性
    public string FullName
    {
        get
        {
            if(null != m_Variant)
                return Utility.Text.Format("{0}.{1}.{2}", m_Name, m_Variant, m_Extension);
            else
                return Utility.Text.Format("{0}.{1}", m_Name, m_Extension);
        }
    }

    public string Name
    {
        get
        {
            return m_Name;
        }
    }

    public string Variant
    {
        get
        {
            return m_Variant;
        }
    }

    public string Extension
    {
        get
        {
            return m_Extension;
        }
    }
    #endregion
}
```

