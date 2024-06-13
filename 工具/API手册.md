[TOC]



## C#：

### 1.枚举：

#### Enum.IsDefined

```c#
enum AnimalType
{
    cat = 1,
    dog = 2,
    pig = 3
}

.... 
void Start(){
     object val = 1;
     object key = "dog";
     Debug.LogFormat("value: {0}, enum.isDefines: {1}", val, Enum.IsDefined(typeof(AnimalType), val));
     Debug.LogFormat("value: {0}, enum.isDefines: {1}", key, Enum.IsDefined(typeof(AnimalType), key));
}
....
```

- 作用：返回一个boolean值，表示给定的整数值“val”，或者名称的string形式，是否存在于指定的枚举中，即val或者key是否已经在枚举中声明过
- 运行结果：

![image-20220728151517089](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220728151517089.png)

#### Enum.GetNames，Enum.GetValues

- 作用：获取Enum中元素的name，和后面的int数值。如：

  ```
  enum AnimalType
  {
      cat = 1,
      dog = 2,
      pig = 3
  }
  ```

  **Enum.GetNames: 得到“cat, dog, pig”的集合**

  **Enum.GetValues: 得到“1， 2， 3”的集合**

  - 使用方式：

    ```c#
     string[] names = Enum.GetNames(typeof(AnimalType));
     for (int i = 0; i < names.Length; ++i) {
    	 Debug.LogFormat("name: {0}", names[i]);
     }
     var values = Enum.GetValues(typeof(AnimalType));
     foreach (int tempVal in values) {
    	 Debug.LogFormat("value: {0}", tempVal);
     }
    ```

    注意：**在输出“Enum.GetValues”中的数值时需要将value强转为int**，如果仍旧使用“foreach (var tempVal in values)”，那么得到的仍然是“cat, dog, pig”

    运行结果如下：

    ![image-20220728201406893](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220728201406893.png)

- **<font color=red>获取单个Enum类型变量的值：</font>**

  ```c#
  AnimalType temp = AnimalType.cat;
  Debug.LogFormat("Enum变量的值： name: {0}, value: {1}", temp.ToString(), (int)temp);
  ```

  运行结果如下：

  ![image-20220805205846355](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220805205846355.png)

- **<font color=red>将某个val数值直接转换成Enum类型的参数：</font>**

  ```c#
  int enumVal = 1;
  Debug.LogFormat("value: {0} 对应Enum中的 {1}", enumVal, (AnimalType)enumVal);
  ```

  运行结果如下：

  <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220807180242822.png" alt="image-20220807180242822" style="zoom:80%;" />

  以上<font color=red>**还可以使用“Enum.ToObject”方法**</font>：

  ```c#
  (AnimalType)Enum.ToObject(typeof(AnimalType), enumVal);
  ```

​        **PS:** `Enum.ToObject`得到的是默认为Object类型，需要使用"AnimalType"转化下

- **<font color=red>将string转换成Enum类型的参数：</font>**

  ```c#
  string animName = "pig";
  Debug.LogFormat("animName: {0} 所对应的枚举是: {1}", animName, (AnimalType)Enum.Parse(typeof(AnimalType), animName));
  ```

  运行结果如下：

  ![image-20220808155041566](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220808155041566.png)

  - **<font color=red>Enum.Parse的扩展</font>**

    ```c#
    --- @brief 定义
    enum AnimalType
    {
    	cat = 1,
    	dog = 2,
    	pig = 3
    }
    
    public static class EnumUtils
    {
        public static T StringToEnum<T>(this string key) {
            if (!Enum.IsDefined(typeof(T), key)) {
                throw new Exception(string.Format("type {0} not has {1}", typeof(T), key));
            }
    
            return (T) Enum.Parse(typeof(T), key);
        }
    }
    
    
    --- @brief 调用
     string str2 = "dog";
     AnimalType result = str2.StringToEnum<AnimalType>();
     Debug.LogFormat("result: {0}", result);
    ```

    **PS: this关键字用于扩展类型中的方法，只有在static方法中可以使用**

    - 运行结果：
      ![image-20220728191814436](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220728191814436.png)



#### Enum的高级用法：与“位运算符”的融合

在使用枚举参数进行条件判定时，通常和其他条件一样直接使用“==”，但如果枚举使用的是“多重赋值”，那么无法通过“==”直接进行比较：

##### 实际案例：

1.在定义Enum类型时，使用“Flags”标记为Enum中的参数赋值，为方便条件判断，**这里使用“位移”赋值**：

```c#
[Flags]
public enum AnimalType
{
    cat = 1,
    dog = 1 << 1,
    pig = 1 << 2,
    horse = 1 << 3,
    sheep = 1 << 4
}
```

<font color=red>**PS: 由于Enum参数最终的数值其实是int格式，而int默认为32位(部分机器上int为64位)，因此使用“位移”方式赋值“1 << 2”时，最多移位32，否则超出Enum的最大取值范围，赋值失败**</font>

2.使用该Enum类型参数：多重赋值和条件判断

```c#
public AnimalType animal;

void Start() {
    //多重赋值：多个数值之间使用“或”运算
	animal = AnimalType.dog | AnimalType.cat;
	string animalStr = animal.ToString();
	Debug.LogFormat("多选枚举的string形式：{0}", animalStr);

    //条件判断:检测参数是否包含目标值，使用“与”运算
	bool isHasDogType = (animal & AnimalType.dog) == AnimalType.dog;
	if (isHasDogType) {
		Debug.LogFormat("has dog type");
	}
	else {
		Debug.LogFormat("not has dog type");
	}
}
```

将“多重赋值”的枚举参数转换成string输出时，<font color=blue>**各个枚举间用逗号隔开**</font>：

![image-20221114173440371](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221114173440371.png)

<font color=red>**当为枚举参数增加赋值时可直接使用“|”，当判断枚举参数是否包含某个数值时可使用“&”进行判断**</font>



##### 扩展：如何在Unity的Inspector面板中支持枚举多选？

**以上代码只是通过C#中的“Flags”标记使得某个枚举类型在代码中支持多选以及条件判断，**

若要在Unity的Inspector中<font color=red>**显示枚举多选**</font>，则必须要<font color=red>**通过Unity自带的“UnityEditor.dll”重写Enum属性在Unity中默认的显示方式**</font>，即重写“OnInspectorGUI” —— Editor 或者“OnGUI” —— <font color=red>**PropertyDrawer(Editor下的子类)**</font>

```c#
public class MultiEnum : PropertyAttribute{}

[CustomPropertyDrawer(typeof(MultiEnum))]
public class MultiEnumEditor : PropertyDrawer
{
    public override void OnGUI(Rect position, SerializedProperty property, GUIContent label) {
        property.intValue = EditorGUI.MaskField(position, label, property.intValue, property.enumNames);
    }
}
```

**PS：**

1.**自定义类型也可以继承自“Editor”，此时是将某个自定义的Class类型在Inspector中的显示方式进行重写**；<font color=red>**由于这里只是简单的对某个属性，如“Enum”，进行重显示，因此直接使用“PropertyAttribute”和“PropertyDrawer”即可**</font>

2.**由于“MultiEnum”默认使用的是“int”类型，因此这里调用“property.intValue”**。**不同的参数类型会调用“property”中不同的value，如bool, float,vector等**

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221114174949792.png" alt="image-20221114174949792" style="zoom:80%;" />

3.在Unity中声明该Enum参数时，只需要在参数前添加自定义标记“MultiEnum”即可在Inspector中显示多选：

```c#
[MultiEnum]
public AnimalType animal;
```

**PS**: 在Odin插件的“SkillEditor”工程中，检测发现只要在“汉化形式的枚举”前添加“Flags”标记即可自动在编辑器窗口中多选枚举，这个可能和Odin内部的机制有关。但两者在实现原理上必然相近，有空可以看下



##### 避坑指南：

当在Unity的Inspector中增加“枚举多选”后，**会在原有Enum类型的基础上增加“Nothing”和“EveryThing”两个选项**：

![image-20221114180012430](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221114180012430.png)

**如果一个数值都没有选中，那么Unity会自动选中“Nothing”；反之，如果Enum中所有的类型都选中，则会额外选中“EveryThing”：**

![image-20221114180155854](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221114180155854.png)

<font color=red>**这两个选项是Unity引擎内部自动选中的，无法避免**</font>。并且此时若在代码中将该Enum参数以ToString输出：

<font color=red>**EveryThing时数值为-1， Nothing时数值为0，当前仅当Enum参数为两者之外时，才会输出“逗号串联的各个选项”**</font>

**测试如下：**

```c#
public AnimalType animal;

string animalStr = animal.ToString();
Debug.LogFormat("多选枚举的string形式：{0}", animalStr);
```

- 当EveryThing被选中时：

  ![image-20221114180747887](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221114180747887.png)

- 当Nothing被选中时：

  ![image-20221114180814517](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221114180814517.png)

- 当以上两者没有选中时，如勾选“cat”, "dog"，此时才会输出：

  ![image-20221114180922055](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221114180922055.png)

  ![image-20221114180957228](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221114180957228.png)

#### 使用“>=”或“<=”比较两个枚举变量：

若使用“>=”或“<=”比较枚举对象，则默认会使用该对象的“基础类型数据，如int, byte等”进行比较。如果没有为“该枚举类型中的元素指定基础类型”，则默认会使用“int”作为其基础类型，如：

```c#
enum MyEnum{
    Value1 = 1,
    Value2 = 2,
    Value3 = 3
}

MyEnum enumValue1 = MyEnum.Value1;
MyEnum enumValue2 = MyEnum.Value2;
if(enumValue1 >= enumValue2){
    Debug.Log("oh, yeah!");
}
```

总之，如果需要使用“>=”或“<=”比较枚举对象，则**需要在“枚举类型的声明”中为其中的各个元素赋“基础数据类型”的数值，如“1，2，3， 1<<2”等**，如此才能利用该数值进行比较。如果完全没有赋值，则**<font color=red>使用“默认赋值：int类型，数值从0开始递增”</font>**



### 2.this关键字在方法体中的使用：

在方法体中使用“this”关键字时，用于对某些参数类型进行扩展。

效果：==使用“this”声明该参数类型TypeA后，“TypeA”即拥有本static方法，可以在任何情况下调用，极大的方便代码调用。因此通常将“this”用于扩展类型，为该类型提供某些常用的方法==

**条件：**使用this用于类型扩展时，<font color=red>**该方法必须使用static修饰，且本类型不能是“抽象类”**</font>

**实际使用：**

需求：在UI中经常会遇到需要将某些Graphic组件如Text，Image等隐藏或者显示的情况，这里通过借助this关键字为“Transform”提供可以直接设置其component的扩展方法

```c#
public static class TransformExtension
{
    public static void SetGraphicEnabledInChildren(this Transform transform, bool targetEnabledState) {
        var graphics = transform.GetComponentsInChildren<Graphic>();
        for (int i = 0; i < graphics.Length; ++i) {
            if (graphics[i].enabled != targetEnabledState) {
                graphics[i].enabled = targetEnabledState;
            }
        }
    }
}
```

**注意**：在调用该扩展方法时使用的是**<font color=red>该类型的“实例对象transform”</font>**，但该方法“static”修饰代表的是该类型Transform本身就具备的方法，并且**<font color=red>在方法体内部可以直接使用该“实例对象transform”</font>**。

在实际调用时是通过**<font color=red>该类型的“实例对象transform”来启动</font>**该方法并将“该实例对象transform”本身作为隐藏默认参数传递到方法体的“第一个this修饰的参数”，并且**<font color=red>只需要将this之后的参数传递进来</font>**即可



### 3.base关键字在类继承中的作用：

#### 调用基类的非构造方法：

注意：

1.**如果该方法没有在派生类中被重写，那么此时无需使用“base”，直接调用该方法即可**(这种情况下，由于子类继承于父类，并且没有重写父类的同名方法，那么该方法也是子类的一部分，因此可以直接调用，无需使用“base”来区分)

2.“base”的主要作用在于：当派生类中有和父类的同名方法时，使用“base”来区分当前调用的是父类还是子类的同名方法

```c#
class A
{
    public int _num;

    public virtual void CheckValue() {
        if (_num > 0) {
            Debug.LogFormat("Value of _num is greater than zeor");
        }
    }
}

class B : A
{
    public override void CheckValue() {
        base.CheckValue();   //调用父类同名方法
        Debug.LogFormat("num: {0}", _num);
    }
}

//实例化该派生类：
...........
void Start()
{
	B b = new B();
	b._num = 6;
	b.CheckValue();
}
............
```

运行结果如下：

![image-20220809104100077](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220809104100077.png)

#### 调用基类的构造方法：

```c#
class A
{
    protected int _num;
    public A(int num) {
        _num = num;
    }
}

class B : A
{
    public B(int i) : base(i) {
        _num = i + 10;
    }

    public void ShowValue() {
        Debug.LogFormat("num: {0}", _num);
    }
}

//派生类的实例化：
......
void Start()
{
	B b = new B(5);
	b.ShowValue();
}
......
```

运行结果如下：

![image-20220809102429017](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220809102429017.png)

参考链接：https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/base

**<font color=red>总结：</font>**

**1.这里的“基类A”仅仅只是当前类B的直接父类，如果该父类A本身又继承于其他的基类C，那么是无法使用“base”访问另一基类C的**

**2."protected"表示在基类和派生类中可以访问该变量或方法，<font color=red>但在派生类的实例中是无法访问“protected”变量的</font>**

3.“return”关键字代表返回某个方法或类的结果，“return”之后的语句不会再执行。但是如果：**“Method1”调用“Method2”，“Method2”中包含有“return”关键字**。那么在执行“Method1”时，“return”关键字只对“Method2”中的内容起效，并不会对“Method1”有影响，**<font color=red>即“Method2”中的“return”只会结束“Method2”自身，不会结束“Method1”的执行</font>**

**<font color=red>4.特别注意：“override”关键字可以对基类的virtual、abstract、override修饰的方法进行重写。但是当使用“override”对基类某同名方法定义后，会完全覆盖掉基类的同名方法：</font>**

```c#
class A
{
    public int _num;

    public virtual void CheckValue() {
        if (_num > 0) {
            Debug.LogFormat("Value of _num is greater than zeor");
        }
    }
}

class B : A
{
    public override void CheckValue() {
        //由于没有使用“base.CheckValue”，因此会完全覆盖基类的同名方法

    }
}

//实例化派生类并调用派生类中的方法：
void Start()
{
    B b = new B();
    b._num = 6;
    b.CheckValue();
}
```

运行结果完全为空：

![image-20220809120545613](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220809120545613.png)

解析：<font color=red>**“override”会完全覆盖掉基类的同名方法**</font>，又因为派生类中同名方法没有日志输出，因此输出结果为空

**<font color=red>这是与“base”的不同之处</font>**，在构造方法中使用“base”时：

```
public B(int i) : base(i) {
        
}
```

纵然方法体为空，也依然会通过以上方式调用基类的构造方法

**所以如果在派生类中没有对基类同名方法重写，需要将override修饰的方法删除掉。**

**<font color=red>5.构造方法在基类和派生类中的关系：</font>**

**如果基类中没有对构造方法进行重写** —— 即基类使用系统默认的构造方法(该方法没有参数，方法体中没有语句)，那么派生类也不需要构造方法：

```c#
class A{
   public int _num;
                 //没有重写构造方法，即使用默认构造方法
}

class B : A{
                //由于基类使用默认构造方法，因此派生类也不需要写构造方法
}
```

**但是如果基类中对默认构造方法进行了重写**(只要类中有构造方法，就会把默认的构造方法覆盖掉)，那么派生类中就必须书写构造方法：

```c#
class A {
   public int _num;
   
   public A(int num) {       //基类对构造方法进行重写
       _num = num;
   }
}

class B : A {
   public B(int i) : base(i) {     
            //派生类必须要有该语句，即使方法体内容为空
   }
}
```



#### 构造方法的真实作用：

如B类继承自A类，但B类的构造方法中完全没有调用A类的构造方法，也没有为A类的参数赋值：

```c#
public class A {
   public string msg;
}

public class B : A {
    public int _num;
    
    public B(int num){
        _num = num;
    }
}
```

**问**：如果在外部创建B类型的实例，由于B类的构造方法中并没有为“基类的msg参数”赋值，那么该实例中是否包含“msg”参数？

**答**：**<font color=red>必然包含</font>**，只要B继承自A，则必然包含基类的所有参数。这与“B类的构造方法中是否为A类的参数赋值”完全无关。**<font color=red>构造方法的作用仅仅只是为参数赋值而已，这与“参数的声明”完全无关</font>**。

**<font color=red>“该参数存在与否”</font>**只与**<font color=red>“类型本身”</font>**有关，如果是“static”变量，则其与类型本身同在；如果是“非static”变量，则其与该类型的实例同在。**<font color=red>这与“构造方法”中是否为该变量赋值完全无关</font>**



### static关键字的作用：

#### “static修饰的变量”和“没有static修饰的变量”的区别：

##### static修饰的变量：

**要点1**：在脚本中**<font color=red>初次</font>**涉及该类型class时，即会**<font color=red>马上为该class中的static变量赋值，即static变量“=”右边的语句会马上执行</font>**

**要点2**：**<font color=red>static变量的自动赋值只会执行一次，后续不会再次执行“=”右边的语句</font>**

**要点3**：后续如果需要改变static变量的数值则必须手动调用该static变量并同时为其赋值才可

```c#
public class UniqueId{
    private static int id;
    public static int Id => ++id;
}

public class Test03
{
    public static int num = UniqueId.Id;
    public static string str = "hello";
}

//调用如下：
void Start{
    Debug.LogFormat("第一次调用：num数值: {0}", Test03.num);
    Debug.LogFormat("str数值: {0}", Test03.str);
    Debug.LogFormat("第二次调用：num数值: {0}", Test03.num);
}
```

运行结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230926230302205.png" alt="image-20230926230302205" style="zoom:80%;" />

###### 重要结论：第一次调用该类型时，该类型中static变量“=”右边的赋值语句在外部调用该变量之前执行

**验证如下**：

```c#
public class Test2
{
    public Test2(int count) {
        Debug.LogFormat("[Test2] count: {0}", count);
    }
}

public class Test03
{
    public static string str = "hello";
    public static Test2 a = new Test2(1);
}

//调用如下：
void Start(){
    Debug.LogFormat("str数值: {0}", Test03.str);
}
```

运行结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230926230926775.png" alt="image-20230926230926775" style="zoom:80%;" />

**解析**：先执行“=”右边的语句，因此Test2中的日志先输出



##### 没有static修饰的变量：

没有static修饰的变量，则**<font color=red>必须依赖具体的实例才能存在</font>**。因此**<font color=red>该类型的变量在创建本类型实例时会自动执行其赋值语句</font>**，且该赋值语句只会自动执行一次：

```c#
public class Test2
{
    public Test2(int count) {
        Debug.LogFormat("[Test2] count: {0}", count);
    }
}

public class Test
{
    public Test2 b = new Test2(2);
}

//调用如下：
void Start(){
    Test temp1 = new Test();
    Test temp2 = new Test();
}
```

运行结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230926231733890.png" alt="image-20230926231733890" style="zoom:80%;" />

**解析**：这里是创建两个Test类型的实例对象，每次创建实例对象时都会将普通变量进行赋值。两个实例中都包含普通变量b，但两个实例以及其中包含的变量b都完全独立，互不相关



##### 总结：

1.无论是“static修饰的变量”还是“没有static修饰的变量”，**<font color=red>其初始赋值都是自动执行的</font>**，**<font color=red>外部并不需要手动调用该变量才能为其赋值</font>**。但是**“初始赋值语句”都只会自动执行一次**：“static修饰的变量”在“第一次”提及该类型时会马上执行，“没有static修饰的变量”则在“每次实例化新的实例”时自动执行

2.**<font color=red>“static修饰的变量”的赋值在“没有static修饰的变量”的赋值之前</font>**。因为前者只要“第一次提及该类型”时即会马上执行“初始赋值”，而后者需要“实例化该类型”才可得到：T必然在“new T()”之前执行

3.“**初始赋值语句**”只会自动执行一次，**<font color=red>后续调用该变量时不会触发“=”右边的语句</font>**

4.**执行顺序优先级**：如果是初次涉及该类型，则会自动为该类型中所有static变量执行初始化(“=”右边的语句会自动执行) > 如果后续创建该类型的实例，那么在实例化时会自动为该类型所有“非static变量”执行初始化赋值(“=”右边的语句会自动执行) > 手动调用该类型中的任意变量或方法

**简而言之**：**<font color=red>变量的初始化是自动执行的，static变量在初次提及该类型时自动执行，非static变量则在每次创建该类型的实例时自动执行；而方法必须要手动调用才会执行，无论是static方法还是非static方法</font>**



#### “static修饰的方法”和“没有static修饰的方法”的区别：

无论有没有static修饰该方法，**<font color=red>该方法中的语句都不会自动执行</font>**，**这是与上述“变量的初次赋值”不同的地方**。

如果需要执行方法中的语句，必须要外部手动调用才可以：其中“static修饰的方法”可以直接调用，而“没有static修饰的方法”则必须依赖该类型的实例才可以调用





### readonly修饰的变量的特性：

**<font color=red>当使用readonly修饰某个变量时，则只能在“声明该变量”以及“构造方法”中才可以为其赋值，其他地方不能再改变该参数的值</font>**：

```c#
public class BookData
{
    //赋值时机1：声明该变量时可以为其赋值
    public readonly int id = 100;
    public readonly string name = "hello";

    //赋值时机2：构造方法中为该参数赋值
    public BookData(int _id, string _name) {
        id = _id;
        name = _name;
    }

    //这里是错误的，无法在后续修改“readonly”参数的值
    public void ChangeData(int mId, string mName) {
        id = mId;
        name = mName;
    }
}
```



### final关键字：

可以用于修饰属性，方法，类；修饰属性时代表该属性不可被更改，修饰方法时表示方法无法被覆盖——重写或重载，修饰类时代表类无法被继承。



### sealed关键字：

密封类sealed class不能作为基类被继承，但是这个类本身可以继承其他的类或接口。

如成员前有sealed修饰，则该成员在派生类中不可重写对该成员的实现

```c#
public sealed class a:b{
      public sealed override void Method(){                    
              //这样的方法就是密封方法，即使该方法所在的类被继承，该方法也不能被重写了
              base.Method();
              ...........
       }
}
```



### default关键字的作用：

当使用default为某个变量赋值时，如果**该变量为“值类型”**，则默认赋值为该类型的初始值，即int类型，则default值为0；如果**该变量为“引用类型”**，则默认值为null。使用方式如下：

```c#
int num = default(int);
Debug.LogFormat("num的数值为: {0}", num);

string str = default(string);
if(str == null) Debug.LogFormat("str的数值为null");
```

运行结果为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230926221633911.png" alt="image-20230926221633911" style="zoom:80%;" />

**PS**： 如果该类型为**泛型T**，则使用“`default(T)`”即可



### extern关键字：

在方法声明中使用extern表示该方法可以在外部实现，不需要在本脚本中即明确定义其方法体内容

通常与"DLLImport"一起配合使用，**该方法还必须为static**

```c#
[DllImport("avifil32.dll")]
public static extern void Foo();
```

以上代表在”avifil32.dll“中已有方法”Foo()“的具体实现，无需在本脚本中具体其方法体内容



### partial关键字的作用：

当确定某些类型不会被外部调用，仅仅只在本class内部使用时，如某些enum类型，如果确定只在本class内部使用，则在class声明时可以使用“partial”关键字

但如果该enum除了本class外，其他地方也会调用，则不能将该enum使用“partial”声明



### LinQ语句的作用：

==不会导致大量GC，可以正常使用==的LinQ方法：**First，FirstOrDefault**

- 遍历数组中的元素，获取第一个满足指定条件的元素“array.First()”：

  ```c#
  string[] strArr = {"ac", "dasfdsa", "dfjasfldas", "hello, world"};
  string target = strArr.First(a => a.Substring(0, 2) == "he");
  Debug.Log($"final result: {target}");
  ```

  运行结果：<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240325223918364.png" alt="image-20240325223918364" style="zoom:80%;" />

==会导致大量GC的LinQ方法==：**ToList、ToArray、Select、SelectMany、Where、OrderBy、GroupBy、Join**

#### 非LinQ语句且常用的方法Sort或Find或IndexOf：

针对List或Array类型，可以==常用“Find”或“Sort”方法(属于List或Array类型自身的方法，与LinQ无关)==，但其作用与LinQ中的“First、FirstOrDefault”，“OrderBy”作用类似。

**案例1**：对整数或字符串列表排序。==默认排序通常按照从小到大排列==

```c#
List<int> numbers = new List<int>(5, 3, 8, 1, 2);
numbers.Sort();      //默认为升序排列
//输出结果：1, 2, 3, 5, 8

List<string> names = new List<string> { "Alice", "Bob", "Charlie", "David" };
names.Sort();     //按字母顺序升序排列
//输出结果：Alice, Bob, Charlie, David
```

**案例2**：使用自定义排序

```c#
List<string> names = new List<string> { "Alice", "Bob", "Charlie", "David" };
names.Sort((x, y) => x.Length.CompareTo(y.Length));   //按字符串长度升序排列
//输出结果：Bob, Alice, David, Charlie
```

==注意：当需要按照元素中的“bool参数”进行排列时==

```c#
public class MyItem{
    public string name {get; set;}
    public bool IsActive {get; set;}
}
..........
    
List<MyItem> items = new List<MyItem>{
    new MyItem { name = "Item1", IsActive = false };
    new MyItem { name = "Item2", IsActive = true };
    new MyItem { name = "Item3", IsActive = true };
}
//将列表中的元素按照bool参数排序，“false”排在前面，“true”排在后面
items.Sort((x, y) => x.IsActive.CompareTo(y.IsActive));
```

==如果需要将元素按照bool值：“true”排在前面，“false”排在后面，可调换“x, y”顺序即可==，如

```c#
items.Sort((x, y) => y.IsActive.CompareTo(x.IsActive));
```

**List的实例化方法IndexOf**：

该方法在List中查找指定元素并返回其索引值，**如果未找到则返回“-1”**。

**注意**：该方法支持在指定范围内查找该元素。如：

- IndexOf(T item)：在整个List集合中查找该元素
- IndexOf(T item, int startIndex)：从“startIndex”开始查找元素item的位置
- IndexOf(T item, int startIndex, int count)：==从“startIndex”开始的“count”个元素范围内==查找元素item

无论是否设定查找的起始范围，**返回的索引值都是以List本身的索引编号为准**，而非“设定范围后的重新编号的索引”







### Exception语句导致的代码执行顺序

在方法体A内调用“throw new exception()”后，该语句后续的代码不会再执行。

若在方法体B内执行方法A，则方法A后续的代码不会再执行：

```c#
void Method1() {
    int num = 100;
    if (num == 100) {
        throw new Exception("num is 100.");
    }

    Debug.Log("yooh..........");
}

void Method2() {
    Method1();
    Debug.Log("method2.........");
}
```

以上代码中，不论调用Method1或Method2，执行到“throw new exception()”后，后续代码都不会再执行。

总结：当在代码中==使用“throw new exception()”主动抛出异常后，则该语句后续的代码不会再执行==(包含“**嵌套方法**”)





### 4.操作符：

#### 计算规则：

##### 位运算：

**与：**当且仅当两者都为true时，结果为true

**或：**a,b 中其中至少有一个为true，结果为true

**异或：**如果a,b两者数值不同，则为1，否则为0。即“1 xor 1 = 0, 1 xor 0 = 1， 0 xor 1 = 1, 0 xor 0 = 0”。可通过该特性用于判断两个数值是否相等：如果相同，则“异或结果”为0，否则为1

##### 数学运算：

加减乘除：+,-,*,/           取余： %

**<font color=red>幂次：^， 如2的5次方，则为“2^5” —— Lua中表示“幂次”时也是同样的表达式</font>**



#### 操作符号：

##### 位运算符：

**C#:** 与 ——  “&”， 或 -—— “|”， 异或 —— “^”，逻辑左移 —— “<<”, 逻辑右移 —— “>>”

- C#中“逻辑左移”使用形式： 1 << 2 = 4, “逻辑右移” —— 16 >> 2 = 4 （结果均为十进制形式）<font color=red>**系统中数值后没有特殊标记的基本都代表十进制**</font>

**Lua:** 与 —— “bit.band”, 或 —— “bit.bor”, 异或 —— “bit.bxor”, 逻辑左移 —— “bit.lshift”, 逻辑右移 —— “bit.rshift”

- Lua 5.2之前的标准库中不提供按位操作函数，即无法对二进制数值按位进行“与”，“或”， “异或”等操作，Lua5.2之后才提供。使用方式：

  ```
  -- 所有参数和执行结果均为十进制形式
  bit.bor                  -- 按位或
  bit.bxor                 -- 按位异或
  bit.band                 -- 按位与
  bit.bnot                 -- 取反
  bit.lshift(x, n)         -- 逻辑左移，如bit.lshift(1, 2) = 100(二进制)，转换为十进制则为4
  bit.rshift(x, n)         -- 逻辑右移
  ```

  如下：![image-20220804163813455](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220804163813455.png)![image-20220804163844690](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220804163844690.png)

- 使用Lua5.3提供的标准库中bit，将数值在二进制和十进制之间转换：

  ```lua
  bit = {}
  bit.data32 = {}
  for i = 1, 32 do
     bit.data32[i] = 2 ^ (32 - i)
  end
  
  -- 将十进制转换成二进制
  function bit.d2b(arg)
     arg = arg >= 0 and arg or (0xFFFFFFFF + arg + 1)
     local result = {}
     for i = 1, 32 do
         if (arg >= bit.data32[i]) then
             result[i] = 1
             arg = arg - bit.data32[i]
         else
             result[i] = 0
         end
      end
      
      return result
  end
      
  -- 将二进制转换成十进制
  function bit.b2d(arg)
     local result = 0
     for i = 1, 32 do
          if(arg[i] == 1) then
              result = result + bit.data32[i]
          end
     end
      
     return result
  end  
  ```

  也可以自定义bit中的“与”， “或”，“异或”等操作：

  ```lua
  -- “与”操作
  function bit._and(a,b)
      local op1 = bit.d2b(a)
      local op2 = bit.d2b(b)
      local r = {}
      for i = 1, 32 do
          if op1[i] == 1 and op2[i] == 1  then
              r[i] = 1
          else
              r[i] = 0
          end
      end
      
      return  bit._b2d(r)    -- 将结果转换为十进制
  end
  
  -- “或”操作
  function bit._or(a,b)
      local op1 = bit.d2b(a)
      local op2 = bit.d2b(b)
      local r = {}
  
      for i = 1, 32 do
          if op1[i] == 1 or op2[i] == 1  then
              r[i] = 1
          else
              r[i] = 0
          end
      end
      
      return bit._b2d(r)
  end
  ```

  同理根据“异或”规则也可以自定义“bit._xor”方法。本质上和bit库中提供的"bit.bor, bit.band, bit.bxor"功能是一样的，但自定义之后，自由度更高。可以根据具体需求决定是否重写

  参考链接：

  https://www.cnblogs.com/lyh916/p/9030358.html，

  https://blog.csdn.net/yangguanghaozi/article/details/52100501

##### 逻辑运算符：

根据bool值计算：

C#： 与 —— “&&”， 或 —— “||”

Lua:  与 —— “and”， 或 —— “or”

- **"&", "&&"的区别：**

  a & b，a && b：两者都是进行“与”操作。但两者在以下两处不同：

  1.前者最终结果为数值类型，后者则为bool类型
  
  2.在“a && b”中，如果a为false，则b不用计算，直接忽略。这是正常现象，并且这种情况称为“断路”
  
  PS:  "|", "||"的区别与此类似





### 5."?"的作用：

#### 1.用于判空：

##### 形式一：

```c#
if (null == arenaEntity || null == arenaEntity.arenaCurrentTurn ) {
	return 0;
}
```

可以直接使用“？”将代码简化：

```c#
if (arenaEntity?.arenaCurrentTurn == null ) {
	return 0;
}
```

##### 形式二：

```c#
if (arenaEntity == null) {
	return;
}
arenaEntity.ReplaceArenaBattleStage(stage);
```

可以直接简化为：

```c#
arenaEntity?.ReplaceArenaBattleStage(stage);
```



#### 2.用于为值类型变化赋值为null

##### 单问号“?”：

用于为值类型变量赋值为null

```c#
int? b;   //b的默认数值为null
```

当需要判定委托在“非null”的情况下才能调用“Invoke”方法：

```c#
Action a = CustomMethod1;
a?.Invoke();
```



##### 双问号“??”：

格式通常为“int a = b ?? 0”，用于判断“当前面参数为null时则直接返回后面的数值”。

```c#
int a = b ?? 0;  //当b为null时，则将“0”赋值给a参数
```

有时候也可以直接用于某些方法体的返回值：

```c#
public int GetNum(){
    return b ?? 0;            //当b为null时，则该方法体返回数值“0”
}
```



### 反斜杠`"\"`和正斜杠`"/"`：

由于在代码中`'\'`有“转义字符”的含义，因此如果需要在代码中替换“正斜杠”和“反斜杠”：
`string.Replace("\\", "/")`或`string.Replace('\\', '/')`(双引号和单引号均可)



### 6.判断两个浮点数是否相等

对于浮点数，如“float，double”，**<font color=red>由于其底层的存储格式限制，如“浮点数1.0”，在其底层可能存储为“0.999999”或“1.00001”</font>**，因此**不能直接使用“==”或“!=”来比较**。

`Mathf.Approximately`专用于判断两个浮点数是否相等：

```c#
public static bool Approximately(float a, float b)
{
    return Mathf.Abs(b - a) < Mathf.Max(1E-06 * Mathf.Max(Mathf.Abs(a), Mathf.Abs(b)), Mathf.Epsilon * 8);
}
```

也可以自定义方法：在某个误差范围内说明两个浮点数相等

```c#
public bool IsEqual(){
    if (Math.Abs(prevScale.Value - newScale.Value) < 0.001f) {
        Debug.Log("这两个数相等");
     }
    else{
        Debug.Log("这两个数不等");
     }
}
```





### 7.for循环(C# && Lua)：

#### 不论是在lua还是C#中，如果集合中没有元素，那么for循环内部的语句一次都不会执行

**在C#中：**

```c#
List<int> numList = new List<int>();
for (int i = 0; i < numList.Count; ++i) {
   Debug.LogFormat("$$$$$ i: {0}", i);
}
```

以上的“Debug.LogFormat”语句一次都不会执行

只有在集合中有元素时：

```c#
List<int> numList = new List<int>();
numList.Add(1);    //向集合中加入元素
for (int i = 0; i < numList.Count; ++i) {
   Debug.LogFormat("$$$$$ i: {0}", i);
}
```

此时输出结果：

![image-20220913195432830](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220913195432830.png)



**在Lua中：**

当表中一个元素都没有时：

```lua
for _, subConditionType in ipairs(subConditions) do
       self:AddSubConditions(subConditionType)
end
```


  当 subConditions = {}时，`self:AddSubConditions`一次都不会执行

  

### 8.引用类型变量在置空时的影响：

```c#
ReferType A = new ReferType();
ReferType B = A;         //将引用类型变量A的值赋值给引用类型变量B，此时两者同时指向同一地址的变量
A = null;   //将A指向一个新的引用类型变量null的地址。但此时B并没有改变，依然指向之前的地址
Debug.LogFormat("B.num: {0}", B.num);   //获取B所指向的地址中的内容“num”
```

实际例子：

```c#
A tempA = new A(6);
A tempB = tempA;
tempA = null;
Debug.LogFormat("tempB: num: {0}", tempB._num);

class A
{
    public int _num;

    public A(int num) {
        _num = num;
    }
}
```

运行效果如下：

![image-20220916164142206](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220916164142206.png)



#### 扩展：

在战斗项目中，当使用“ReplaceXXX”设置参数时，如果该参数为list对象(即该参数可以使用ListPool来获取和回收)，那么此时在使用"ReplaceXXX"时，需要先将该参数原来的数值回收掉，否则原来的参数依然会占有内存

```c#
var isChanged = false;    //是否改变了buff图标列表中的数据
var buffIconList = Contexts.ListPool<BuffIconData>().Get();
buffIconList.AddRange(hudEntity.hUDBuffIcons.BuffIconList);
for (int i = 0; i < buffIconList.Count; ++i) {
	var iconData = buffIconList[i];
	if (iconData.Id == message.LayerId) {
		buffIconList.RemoveAt(i);
		isChanged = true;
		break;
	}
}

//更新buffIcon列表
if (!isChanged) {
	Contexts.ListPool<BuffIconData>().Return(buffIconList);
	return;
}

var lastBuffIconList = hudEntity.hUDBuffIcons.BuffIconList;   //先将原来的值存起来

//在更换了新的值后：由于是引用类型对象，这里实际更换的是指向新的地址，原来地址的对象并没有改变，
//所以在更换了新的地址后，原来的地址中的对象就需要释放掉
hudEntity.ReplaceHUDBuffIcons(buffIconList);  

Contexts.ListPool<BuffIconData>().Return(lastBuffIconList); //将原来的数值所占有的内存施放掉
```



### 9.Switch....Case语句：

当有多个Case语句的执行逻辑都是一样的时，可以直接使用如下的方式简写：

```c#
switch(propertyDef){
        case zh_PropertyDef.当前HP:
        case zh_PropertyDef.最大HP:
        case zh_PropertyDef.攻击力:
            return "当前属性/" + propertyDef;
        case zh_PropertyDef.初始刺盾:
        case zh_PropertyDef.初始普攻连击率:
        case zh_PropertyDef.初始吸血:
            return "初始属性/" + propertyDef;
}
```

**<font color=red>这样表示上述的多个“case xxx”情况都使用一个执行逻辑，简化代码</font>**

Switch结构中有一个特殊的关键字“goto”，专用于直接跳转到指定“标签”的语句来执行：

```c#
int i = 0;
switch(i){
    case 1:
        while(i < 40){
            i++;
            if(i == 12)
                goto Found;
        }
        break;
    case 2:
        .....
        break;
    case Found:
        Debug.Log("hello, world");
        break;
}
```

"goto"关键字专用于“Switch”结构中跳转到“指定标签”执行的作用



### 10.引用类型变量中"new XXX()"的作用逻辑：

```c#
var image = grid.GetComponent<Image>();
var color = image.color;
color = new Color(color.r, color.g, color.b, 0);
image.color = color;
```

如上： 在`color = new Color(color.r, color.g, color.b, 0)`中，先执行右边的逻辑，使用color原内容中的参数生成一个新的引用类型变量，因此此时是可以直接使用color原来的参数r, g, b的数值的。

当使用“new Color”生成新的引用类型变量后再赋值给左边的color

**注意**：只有在此时，“color”才会指向新生成的引用类型变量，此时“color.a”才会真的变成“0”

**<font color=red>"new xxx()"在创建引用类型变量时的执行顺序需要注意，有些情况可以简化代码过程</font>**



### 11.“#pragma  warning disable”的作用：

在脚本如果声明了某个变量，但是没有地方调用到这个变量，此时在编译时则会有“告警提示”，如“xxxxx is declared, but it's is never used.”

如果需要屏蔽此类信息，只需要在该变量前添加 `#pragma warning disable` 即可，结束时使用 `#pragma warning restore` 恢复

**总结**：使用 `#pragma warning disable`  和 `#pragma warning restore` 将某部分不需要告警的代码包围起来，防止编译器警告。如：

![image-20221125202433800](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221125202433800.png)



### 12.Dictionary在初始化时使用自定义比较器：

Dictionary在赋值时默认使用**自带的比较器**，<font color=red>**当向Dictionary中添加元素或者执行ContainKey等方法时，会使用默认的比较器用于检测集合中是否存在该key，如果存在，则再次使用“Add”方法时会报错**</font>：

**使用默认比较器：**

```c#
void TestDic() {
	Dictionary<int, string> strDic = new Dictionary<int, string>();
	strDic.Add(1, "a");
	strDic.Add(1, "b");

	foreach (var VARIABLE in strDic) {
		Debug.LogFormat("key: {0}, value: {1}", VARIABLE.Key, VARIABLE.Value);
	}
}
```

运行后报错：
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221202125353084.png" alt="image-20221202125353084" style="zoom:80%;" />



**使用自定义比较器：**

```c#
void TestDic() {
    //使用默认比较器，只要key值不同即会判定为不同的元素
    Dictionary<string, string> strDic = new Dictionary<string, string> {
        {"abc", "one"},
        {"AbC", "two"}
    };
    foreach (var variable in strDic) {
        Debug.LogFormat("#### strDic ####: key: {0}, value: {1}", variable.Key, variable.Value);
    }

    //自定义比较器，key不区分大小写
    Dictionary<string, string> strDic2 = new Dictionary<string, string>(new CustomDicKeyComparer());
    strDic2.Add("abc", "one");
    strDic2.Add("AbC", "two");
    foreach (var variable in strDic2) {
        Debug.LogFormat("#### strDic2 ####: key: {0}, value: {1}", variable.Key, variable.Value);
    }
}
```

比较器Comparer实现逻辑：

```c#
class CustomDicKeyComparer : IEqualityComparer<object>
{
    //key不区分大小写，故这里全部转换成小写用于比较
    public new bool Equals(object x, object y) {
        var xLower = x != null ? x.ToString().ToLower() : String.Empty;
        var yLower = y != null ? y.ToString().ToLower() : String.Empty;
        var result = xLower.Equals(yLower);
        return result;
    }

    //由于“IEqualityComparer”中只有在“HashCode”一致时才会调用“Equals”方法，否则自动判定为不同的元素
    //这里为了方便直接调用“Equals”方法，直接设定所有的key的HashCode完全一致
    public int GetHashCode(object obj) {
        //var hashCode = RuntimeHelpers.GetHashCode(obj);  //获取HashCode的方法
        return 0;
    }
}
```

执行结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221202173214806.png" alt="image-20221202173214806" style="zoom:80%;" />

**解析：**

1.比较器接口`IEqualityComparer`中，<font color=red>**当且仅当两个对象的`GetHashCode`相同时才会调用`Equals`进一步判定**</font>；如果两个对象的`GetHashCode`不同，则完全不会执行`Equals`方法，直接判定为两个不同的对象

参考链接：https://www.cnblogs.com/wesson2019-blog/articles/12126310.html

2.基于以上“key不区分大小写”的判定逻辑，当执行`strDic2["AbC"] = "two"`后，<font color=red>**由于两者的key被判定为相同**</font>，因此集合中原有的`strDic2["abc"] = "one"`<font color=red>**元素的value值会被替换成“two”**</font>

3.Microsoft官方为Dictionary提供了一些封装好的比较器，如以上“不区分key大小写”的自定义比较器可以直接使用`StringComparer.CurrentCultureIgnoreCase`，即直接使用：

```c#
Dictionary<string, string> strDic2 = new Dictionary<string, string>(StringComparer.CurrentCultureIgnoreCase)
```

其他官方比较器：https://learn.microsoft.com/zh-cn/dotnet/api/system.stringcomparer?view=net-7.0

```
StringComparer.Ordinal   区分大小写的字符串比较
StringComparer.CurrentCultureIgnoreCase  不区分大小写的字符串比较
```



#### 字典排序类型：SortDictionary

如果需要对字典中的元素按照key进行排序，则可以直接使用“SortDictionary<string, xxx>”类型：

`SortedDictionary<string, xx> m_Assets = new SortedDictionary<string, Asset>(StringComparer.Ordinal)`



#### 分别获取字典的key集合和value集合：

`Dictionary<string, List<string>> result = new Dictionary<string, List<string>>()`

使用result.Values得到的是List<string> 集合，result.Keys 是key的集合



### 13.Assembly.GetExecutingAssembly的作用：

`Assembly.GetExecutingAssembly`代表获取“包含当前执行代码”的程序集，通常**<font color=red>结合“GetType()”和“for循环”来筛选该程序集内指定的类型</font>**：

```c#
Assembly assembly = Assembly.GetExecutingAssembly();  //获取“包含当前执行代码”的程序集
Type[] types = assembly.GetTypes();
for (int i = 0; i < types.Length; i++)
{
    if (!types[i].IsClass || types[i].IsAbstract)
    {
        continue;
    }

    if (types[i].BaseType == typeof(SCPacketBase))
    {
        //使用“Activator.CreateInstance”方法创建指定类型实例对象
        PacketBase packetBase = (PacketBase)Activator.CreateInstance(types[i]);
        ................
    }
}   
```



### 14.Type相关的一些重要参数：

```c#
namespace DetailComment
{
    public class Src01
    {
        //嵌套类
        public class A
        {

        }
    }
}

//测试：
public class CustomTest : MonoBehaviour
{
    void Start() {
        Src01 temp = new Src01();
        Debug.LogFormat("temp type: {0}, namespace: {1}", temp.GetType(), temp.GetType().Namespace);

        Src01.A temp02 = new Src01.A();
        Debug.LogFormat("temp02 type: {0}, namespace: {1}", temp02.GetType(), temp02.GetType().Namespace);
        if (temp02.GetType().IsNested) {
            Debug.LogFormat("temp02 is a nested type.");
        }
    }
}
```

输出结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221208170723439.png" alt="image-20221208170723439" style="zoom:80%;" />

**解析**：从以上结果可知，当获取某个对象的type时，会自动带上该type的namespace，并且如果该类型是嵌套在其他class内部，则会使用“+”串联该类型

#### isNested 嵌套类：

指其声明在其他class内部，并不是一个独立的文件来声明该class。通常在解析type类型时会用到，需要将“+”拆分才能得到该type真实的name

#### IsPrimitive 基元类：

指该type是boolean, int, char, byte, double等基础类型

**<font color=red>注意：基元类不包含string，也不包含枚举类型Enum</font>**

#### IsGenericTypeDefinition与IsGenericType的区别：

指该type是否是用来构建其他泛型的基础类，如

```c#
public class Singleton<T> where T: new()
```

此时**<font color=blue>typeof(Singleton).IsGenericTypeDefinition = true</font>**

但如果某个类继承于“Singleton”，如：

```c#
public class BulletManager : Singleton<BulletManager>{}
```

此时**<font color=blue>typeof(BulletManager).IsGenericTypeDefinition = false</font>**

但若使用typeof(BulletManager).**<font color=red>IsGenericType</font>** = true



#### typeof(Type type) 获取Type对象的Type: 

```c#
int a = 100;
Debug.LogFormat("第一层type: {0}", a.GetType());
var aType = a.GetType();
Debug.LogFormat("第二层type: {0}", aType.GetType());
Debug.LogFormat("是否是valueType: {0}, {1}", a.GetType().IsValueType, a.GetType().GetType().IsValueType);
```

输出结果：
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221208200809620.png" alt="image-20221208200809620" style="zoom:80%;" />

**注意**：以上的“isPrimitive”, "isNested"，“isValueType”都是针对第一层GetType有效，而**<font color=blue>第二层GetType</font>**，**<font color=red>如果在运行时获取，则基本所有类型得到的都是“System.RuntimeType”</font>**

#### BindingFlag.DeclaredOnly:

用于反射时，只获取该type中的元素，对于type继承的父类中的元素则不需要

#### MethodInfo.IsSpecialName:

使用BindingFlag获取type中的Methods时，部分方法是所有type自带的，并且方法名称也固定，如"get_item", "set_item",  "add_xxx", "remove_xxx"，以及操作符相关方法"op_xxx"。

**PS:** 这些方法并不需要wrap，所以需要排除掉

#### MethodInfo.GetParameters：

获取方法体的参数列表，返回值是`ParameterInfo[]`(数组形式)

#### 获取Delegate类型的参数中的Method：

```c#
public delegate void TestMyOwnDelegate(int a);
void Start(){
    TestMyOwnDelegate temp = (a) => { a = a + 1;};
    MethodInfo method = temp.Method;
}
```

以上代码经过测试后可以正常运行，不会报错。

可以直接获取委托类型变量中的方法MethodInfo，如此即可判断该方法的static、paramaters等信息(通过methodInfo获取)

#### IsAssignableFrom与instanceof的作用：

`IsAssignableFrom`：常用例子`superClass.IsAssignableFrom(childClass)`，判定给定参数是否是某个类的子类，<font color=red>**前者为父类，后者为子类，但两者都是类型，而非该类型的实例。并且父子类参数的顺序与通常习惯相反**</font>

`instanceof`：常用例子`obj.instanceof classType`，<font color=red>**前者为实例对象，后者为类型，用于判定是否是某个类的实例**</font>

案例：

```c#
superClass.IsAssignableFrom(childClass);           //true
superClass.IsAssignableFrom(superClass);           //true
childClass.IsAssignableFrom(superClass);           //false，顺序反了，该方法与通常习惯的顺序不一样

childObj.instanceof(superClass);                   //true
childObj.instanceof(childClass);                   //true
```



#### “Is” 与“IsAssignableFrom”的区别：

**当需要知道两个类型之间是否为“继承关系”时**，可以使用“**<font color=red>IsAssignableFrom</font>**”；

**当需要知道某个实例对象是否为某类型时**，则使用“**<font color=red>Is</font>**”，如

```c#
var a = 100;
bool isIntType = a is int;
```

并且还可以将该对象强转成其他类型的对象实例：

```c#
var a = 100f;
a is float b;
```

此时“b”为float类型变量，并且数值为“100f”



#### "IsSubClassOf"与“IsAssignableFrom”的区别：

```c#
//判断A是否是B的派生类，如果是，则返回true
//当使用“IsAssignableFrom”时
typeof(B).IsAssignableFrom(typof(A))
//当使用"IsSubClassOf"时
typeof(A).IsSubClassOf(typeof(B));   
```

**总结**：IsSubClassOf主要用于检测一个类是否是另一个类的子类，但是==无法用于检测一个类是否实现了某个接口==等，其作用有限。通常==在判断接口、基类间的继承或实现关系==时，使用“**IsAssignableFrom**”更为安全





### 15.Lambda表达式的灵活使用：

#### 简化方法体：

```c#
public static void SetVersionHelper(Version.IVersionHelper versionHelper) => Version.s_VersionHelper = versionHelper;
```

当方法体没有参数时：

```c#
static Assembly() => Utility.Assembly.s_Assemblies = AppDomain.CurrentDomain.GetAssemblies();
```

这样一行即可实现该方法，代码更加简洁

#### 隐形传递参数：

```c#
transform.DOMoveX(4, 1).OnComplete(()=>MyCallback(someParam, someOtherParam));
```

如上所示，“OnComplete”中只接受没有“参数”的方法，使用“Lambda表达式”可以在既满足需求的情况下又将参数传递过去，非常便捷





### 16.Path.GetFileName的使用：

当需要获取某个路径的文件名时可以直接使用“`Path.GeiFileName`”来获取，如：

```c#
string path = "Assets/Resources/a.txt";
Debug.LogFormat("filename: {0}", Path.GetFileName(path))
```

输出结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221215101154077.png" alt="image-20221215101154077" style="zoom:80%;" />

PS: 如果只需要获取文件名字，而不需要文件的后缀，则可以使用“`Path.GetFileNameWithoutExtension`”

**注意**：**<font color=red>`Path.GetFileName`对于任意路径都可以直接获取到最终文件夹下的文件名字</font>**，**如果需要单独获取该文件的目录文件夹**则可以使用“`Path.GetDirectoryName`”



#### Path.Combine：

该方法会自动在两个参数之间添加“/”或“\”等连接号，不需要在参数内手动添加



#### Path.GetDirectoryName

获取指定路径的目录。如

```c#
string filePath = @"c:/project/file.txt";
Debug.Log(Path.GetDirectoryName(filePath));
//输出结果为“c:/project”
```



#### Path.GetFileNameWithoutExtension

获取文件name，且不包含扩展名



#### File.Move：

```c#
string path1 = Application.dataPath + "/Resources/hello1.txt";
string path2 = Application.dataPath + "/Resources/hello2.txt";
File.Move(path1, path2);
```

"File.Move"用于将文件移动位置，但：

如果path1, path2在同级目录，如以上的“Resources”文件夹下，此时“hello1.txt”文件会被重命名为“hello2.txt”(该方法只有在“Resources”目录下之前并没有“hello2.txt”时才能正常执行，否则会报错)

**<font color=red>基于此特性，可用于批量为文件改名</font>**

如果path1、path2在不同的目录下，则此时直接将文件移动到新的路径下即可，**<font color=red>此时文件的名字按照新的path2配置生成</font>**



#### File.AppendAllText:

```c#
string path3 = Application.dataPath + "/Resources/hello3.txt";
string msg = "good";
File.AppendAllText(path3, msg, Encoding.UTF8);
```

File.AppendAllText会自动在指定路径下生成新的文件(如果有则覆盖之前的同名文件)，并且两次调用“File.AppendAllText”之间得到的文本会自动间隔行，不会连续写入，如：
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221227161100067.png" alt="image-20221227161100067" style="zoom:80%;" />

即为自动预留的新行



### Directory相关：

#### Directory.GetFiles()

获取目标路径下所有的文件：”*.*“ —— 没有指定文件名和文件后缀

**SearchOption选项的区别：**

SearchOption.AllDirectories 代表遍历子文件夹。

SearchOption.TopDirectoryOnly 或 不加参数 则只访问第一层文件，无法向下递归查找

```c#
string[] files = Directory.GetFiles(path, "*.*", SearchOption.AllDirectories);
```

若需要只获取“指定后缀”的文件，则使用如下形式：

```c#
Directory.GetFiles(path, "*.unity", SearchOption.AllDirectories)
```

得到指定的多个类型后缀的文件：

```c#
var paths = Directory.GetFiles("Assets/Scenes/", "*.*", SearchOption.AllDirectories)
            .Where(s => s.EndsWith(".unity") || s.EndsWith(".meta")); 

//得到以".unity" 和 ".meta" 为后缀名的所有文件。注意此处不能写“s => s.EndsWith("*.unity")” —— 不能带“*”
```

排除掉某些后缀的文件：

```c#
var paths = Directory.GetFiles("Assets/Scenes/", "*.*", SearchOption.AllDirectories)
            .Where(s => !s.EndsWith(".unity") && !s.EndsWith(".meta"));

//排除掉以“.unity” 和 “.meta” 为后缀的文件
```

#### Directory.CreateDirectory

创建文件夹

```c#
void Start()
{
    string sPath = Application.dataPath + "/Hello/good/world/aaa/bbb";
    if (!Directory .Exists(sPath))
    {
        Directory.CreateDirectory(sPath);  //当该文件夹目录不存在时会自动创建          
    }
}
```



### 17.Stream相关：

1.当需要将byte数组封装成stream以便读取操作时通常使用“MemoryStream”

2.使用Stream时通常使用“using”的形式，并在使用完后stream.close，来释放内存占用：

```c#
using(MemoryStream stream = new MemoryStream()){
    .......
    stream.close();
}
```

#### stream.position:

stream.position代表的是当前读写的起始点，如果没有设置则默认为stream.current，即当前索引位置。但如果需要在特定位置Read或写入Byte数据，则需要首先设置stream.position，如：

```
stream.position = 4    //由于后续需要写入数据，因此这里设置写入的起始位置
stream.WriteByte(32)   //十进制“32”代表“空格键”
```

**注意**：**<font color=red>任何stream实例在创建之后，其stream.position都默认为0</font>**，因此不需要额外手动设置



#### stream.Length 与 stream.Capacity的区别：

**stream.Length**：代表该stream**<font color=red>实际包含数据的长度</font>**，以byte字节为统计单位。默认在创建stream实例后，该stream.Length为0

**stream.Capacity**：在使用`stream s = new stream(int capacity)`创建stream实例时，capacity参数代表该stream的容量

但是**<font color=red>“stream.Length”与“stream.Capacity”不同</font>**，**对于新创建的stream实例**，**<font color=red>纵然其capacity>0，但其stream.Lenght依然保持为0</font>**

```c#
MemoryStream ms = new MemoryStream(1024);
Debug.LogFormat("该memoryStream对象的起始位置：{0}， 长度为：{1}, 容量为：{2}", ms.Position, ms.Length, ms.Capacity);
```

**运行结果**：<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230928170915423.png" alt="image-20230928170915423" style="zoom:80%;" />



#### stream.Length在不同情况下输出结果的变化：

```c#
MemoryStream ms = new MemoryStream(1024);
Debug.LogFormat("ms.Length的初始值为：{0}", ms.Length);
//情况一：当”SetLength(int capacity)“中的capacity数值"2" < 将要写入ms的总字节数“10”时， 
ms.SetLength(2);
//情况二：当”SetLength(int capacity)“中的capacity数值“11” > 将要写入ms的总字节数“10”时
ms.SetLength(11);
byte[] dataToWrite = new byte[10];
ms.Write(dataToWrite, 0, dataToWrite.Length);
Debug.LogFormat("ms.Length: {0}", ms.Length); 
```

如上代码所示，当注释“情况二”中的“ms.SetLength(11)”，**<font color=blue>只执行“情况一”中的“ms.SetLength(2)”</font>**，执行结果为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231002193446099.png" alt="image-20231002193446099" style="zoom:80%;" />

当注释“情况一”中的“ms.SetLength(2)”，**<font color=blue>只执行“情况二”中的“ms.SetLength(11)”时</font>**，执行结果为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231002193538052.png" alt="image-20231002193538052" style="zoom:80%;" />

**当把情况一中的“ms.SetLength(2)”和情况二中的“ms.SetLength(11)”都注释掉**时，此时输出结果为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231002193653519.png" alt="image-20231002193653519" style="zoom:80%;" />

**解析**：由以上输出结果可知，**<font color=red>“ms.Length”在不同情况下输出结果不一致</font>**

在初始情况下“`MemoryStream ms = new MemoryStream(int capacity)`”，`ms.Length`为0；后续该数值则根据“**memoryStream中实际写入的byte大小**”与“**SetLength方法设置的数值**”，两者中最大值来确定`ms.Length`数值

而**<font color=red>“ms.Capacity”则任何情况下都保持三者之间的最大值</font>**：初始分配值、SetLength设置的数值、实际写入数据的容量数值。当**实际写入的数据容量或者“SetLength设置的数值”超出初始分配capacity数值**时，则**<font color=red>以三者中最大值为准</font>**

#### SetLength还有另一作用：

```c#
m_Stream.Position = 0;
m_Stream.SetLength(0);
```

当设置该stream的起始读写位置后——`m_Stream.Position = 0`，再使用`SetLength(0)`则代表将该起始位置后的所有数据全部舍弃



#### stream.Seek

设定stream的起始位置，作用和stream.position一样，但方式更为简洁

```c#
stream.Seek(1, SeekOrigin.Current);  //以当前stream.position向后偏移1个位置
stream.Seek(1, SeekOrigin.Begin);  //以stream的起始位置向后偏移1个位置
stream.seek(1, SeekOrigin.End);   //以stream的结束位置为起点偏移1个位置
```

#### stream.Flush:

将stream保存到存储介质中

```c#
m_FlushSize：
从“资源服务器”下载文件时，由于部分文件容量过大，为了支持“断点下载”和“分段下载”，应该及时将“UnityWebRequest.Get”传送过来的“bytes数据”在使用“FileStream.Write”写入“指定文件流”后，再通过“FileStream.Flush()”确保其存入“本地文件介质”中。

问题1：已经使用“FileStream.Write”写入“指定文件流”后，为什么还要使用“FileStream.Flush()”方法？
解答：
1.“System.IO.FileStream”类型中有一个默认的缓存区，其会将所有使用“FileStream.Write”写入的数据先放入该缓存区中。只有当超出该缓存区大小时，才会将“缓存区中的所有数据”都写入“本地文件介质”中，同时清空该缓存区以方便后续继续写入bytes数据
2.如果某文件数据较少，其使用“FileStream.Write”后数据会暂存在“缓存区”。但由于数据量未达到“缓存区大小限制”，因此如果不使用“FileStream.Flush()”，则这些数据无法保存到“本地文件介质”中。即出现“虽然使用FileStream.Write写入数据，但本地文件中却没有内容”的情况出现。

问题2：“m_FlushSize”的作用是什么，为什么需要该参数？
解答：
“FileStream”中有默认的缓存区，只有当缓存区满时，才会将所有数据写入“本地文件介质”中。为了自主控制“将缓存区数据写入本地文件介质”的时机，这里增加“m_FlushSize”参数，当缓存区数据量超出“m_FlushSize”时，则主动调用“FileStream.Flush()”方法将“当前缓存区的所有数据写入本地文件介质”中。
这样做的好处在于：可以自主控制“写入本地文件介质”的时机，同时方便“断点下载”，及时保存下载的所有数据，提高下载效率

问题3：如果最后的一小部分数据不满足“m_WaitFlushSize”的要求，最后是否会导致“数据丢失”？
解答：
"FileStream.Close"有一个隐藏功能：其在关闭文件流之前，会自动将“当前缓存区中的bytes数据”写入“本地文件介质中”。因此在最后结束时，直接调用“FileStream.Close”方法即可，不会导致“缓存区的数据丢失”。
而之前在“下载过程中”主动调用“FileStream.Flush()”是为了方便“断点下载”和“防止数据丢失导致重新下载相同的数据”
```





#### stream.Close 和 stream.Dispose:

stream.Close是关闭该文件流，但该stream所占用的内存空间并没有马上释放

stream.Dispose则是马上销毁该文件流

之后将该stream对象置为null

```c#
fileStream.Close();
fileStream = null;
```

使用Using语句仅仅只是支持将using中创建的对象置为null，并没有将其“Close”的功能，所以还需要在代码中调用“stream.Close”方法才可以：

```c#
using(FileStream fileStream = new FileStream(path, FileMode.Open, FileAccess.Write)){
    .......
    fileStream.Close();    
    //关闭该filestream需要手动调用，using语句在用完该fileStream对象后将其置为null而已，并不包含“Close”操作
}
```



#### FileStream：

1.在创建FileStream对象使用“**<font color=red>FileMode.Create</font>**”时，**<font color=red>无论本地是否有该文件，都会创建新的文件，并同时删除原有的同名文件</font>**，并不是在原有文件的后面继续写入数据

```c#
private string[] strArray = {"hello", "today", "is"};
void FileStreamMethod(int count) {
	if(count > 2) return;
	string path = Application.dataPath + "/a.txt";
	using (FileStream fs = new FileStream(path, FileMode.Create, FileAccess.Write)) {
		byte[] msg = Encoding.UTF8.GetBytes(strArray[count]);
		fs.Write(msg);
	}
}

//调用该方法：
private int i = 0;
void Update()
{
	if (Input.GetMouseButtonDown(0)) {
		FileStreamMethod(i);
		++i;
	}
}
```

通过查看本地路径下的“a.txt”文件，发现每次写入数据后，原来的数据都会被清空。

2.对于新创建的FileStream对象在使用完后必须要执行“fileStream.Close/Dispose”方法，但也可以直接使用“using”语句，这样就不用再手写代码处理该对象的释放了，如：

```c#
using(FileStream fs = new FileStream("", FileMode.Create, FileAccess.Read)){
    ............
    fs.Close();
}
```



### 18.BinaryFormatter和MemoryStream的使用：

```c#
[System.Serializable]           //表示该对象可以被序列化
class TempObj{                  //需要存储下来的游戏数据
	   public int num = 100;
}

//存档游戏数据
TempObj obj = new TempObj();
BinaryFormatter bf = new BinaryFormatter();
FileStream fs = File.Create("Assets/FrankOwn/objFromBinaryFormatter.save");
bf.Serialize(fs, obj);
fs.Close();
Debug.Log("saved....");

//解析存档数据
BinaryFormatter bf2 = new BinaryFormatter();
FileStream fs = File.Open("Assets/FrankOwn/objFromBinaryFormatter.save",  FileMode.Open);
TempObj obj2 =  bf2.Deserialize(fs) as TempObj;
Debug.Log(obj2.num + "   NUM   ");
```

序列化某些数据，并解析这些数据



**对于任意一个byte数组，如果需要将其反序列化成实际的对象，则一定要事先清楚该byte数组对应的内容的类型:**

可以借助MemoryStream 和 BinaryFormatter来实现反序列化：

```c#
byte[] allBytes =  File.ReadAllBytes("Assets/FrankOwn/objFromBinaryFormatter.save");  //得到byte数组
MemoryStream m_stream = new MemoryStream(allBytes);  //借助MemoryStream将byte数组封装成缓存流
BinaryFormatter bf3 = new BinaryFormatter();
TempObj obj3 = bf3.Deserialize(m_stream) as TempObj;   //反序列化
Debug.Log(obj3.num + "   memoryStream   ");
```



### 18.1 byte、Byte、bit的关系

#### 在C#中“byte”和“Byte”都代表字节吗？两者有什么关系？

**解析**：在C#中，byte和“Byte”都代表字节，是相同的数据类型，**只是"Byte"是“byte”的别名，用于提高可读性**，

但在表示数据大小时，并不会用“1Byte”，而使用“1byte”

#### 在C#中是否可以使用new Byte[1024 * 64]创建数组吗？

**解析**：在C#中**<font color=red>字节的基础数据类型是“byte”</font>**，**而不是“Byte”**，因此在创建数组时应该**<font color=red>使用“new byte[1024 * 64]”</font>**

#### 在C#中，1byte与1Byte相等吗？

**解析**：在C#中，当表示数据大小时通常使用“**<font color=red>小写的byte</font>**”，而不用Byte。并且“1Byte”被解释成以“Byte”为标识，“1”为数量的统计单位(并且Byte并不代表“数据类型byte”，因此**<font color=red>这里的“1Byte”是没有意义的</font>**)。

所以表示数量时使用“1byte”，而不使用“1Byte”(**在C#中**，**“1Byte”是无效的语法**，应该弃用)

#### 在C#中，1bit与1byte相等吗？

**解析**：在C#中，**1bit代表最小的数据单位，可以用于存储0或1**；1byte则由8个连续的bit组成，因此1byte可表示“0 - 255”个数值，因此**<font color=red>1bit与1byte不相等，且1byte = 8bit</font>**

#### 在C#中，“1b”与“1B”相等吗？

**解析**：不等。如果使用简写，则“1b”代表“1个比特”，即1bit，“1B”代表“1个字节”，即1byte。所以两者不等

#### 总结：

1.在C#中“Byte”是“byte”的别名，仅用于提高可读性，在实际使用中，**<font color=red>无论是用于统计数量，如“1byte”，还是用于创建字节数组，如“new byte[1024]”，都使用“byte”，而不使用“Byte”</font>**

2.**当使用简写的标识时**，**<font color=red>“1b”代表的是“1个比特”</font>**，即最小的计量单位，**<font color=red>相当于“1bit”</font>**，而**<font color=red>“1B”则代表“1个字节”</font>**，其包含“8个比特”，**<font color=red>相当于“1byte”</font>**





### 19.string类型：

#### string.LastIndexOf:

string.LastIndexOf(char a, int startIndex)：从指定索引startIndex**<font color=red>反向</font>**遍历string，返回**<font color=red>最后一次</font>**出现目标“char”的索引位置：

PS: 

1.如果没有“startIndex”则默认为string末尾，从string反向遍历。但**<font color=red>返回的index是以string自身的index为基准</font>**，不是以startIndex的offset来返回

2.**如果string中没有目标要查找的char，则返回“-1”**

```c#
string str = "abcd.xy.mnz";
Debug.LogFormat("index: {0}", str.LastIndexOf('.'));
Debug.LogFormat("index#########: {0}", str.LastIndexOf('.', 5));
```

执行结果：
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221229105141128.png" alt="image-20221229105141128" style="zoom:80%;" />



#### string.CompareOrdinal：

专用于比较两个string对象，对该string中的每个char字符(区分大小写)依次按照“ASCII编码”对应的数值进行比较，并直接返回比较结果result(int类型)：`string.CompareOrdinal(a, b)`

当“result == 0”时，则代表两个string对象相同；当“result < 0”时则代表前者小于后者；反之若“result > 0”则代表前者大于后者



#### string.CompareTo：

将两个string对象逐一按照每个Char的ASCIII码数值进行比较并按照从小到大的顺序对两者排序：`a.CompareTo(b)`

若result < 0，则代表在最终排序中a位于b之前；反之，则说明a与b位置一致或在b之后



#### string.StartWith:

判断某个string对象是否以“指定内容”作为前缀：

```c#
string b = "Knowledge is Power !!!";
b.StartWith("k");      //返回值是bool类型，判断该string的前缀
```



#### string.Split:

该方法支持对分隔符数组中所有元素进行分割：

```c#
string str = "aa|bb@cc^dd*ee(ff!海纳百川"; //字符串中有多个类型的分割符
char[] separator = { '|', '@', '^', '*', '(', '!' }; //对于多个分割符可以设置成一个专门的分割符数组。然后根据分割数组来分割原字符串
string[] strArray = str.Split(separator);
foreach(var temp in strArray)
{
	Debug.Log(temp + "  TEMP   ");
}
```

运行结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230524160412064.png" alt="image-20230524160412064" style="zoom:67%;" />



#### string.Trim:

去掉string左右两端的空格，字符串中间的空格无法去掉：



#### StringBuilder, StringBuffer, String三者的关系：

StringBuffer —— HashTable, 可以并发操作，多线程是安全的

StringBuilder —— HashMap，不能并发操作，多线程不安全，但是在单线程上其性能比StringBuffer高。

所以在某些可以单线程运行或者无需考虑线程同步的情况下，使用StringBuilder

三者在执行速度上：Stringbuilder > StringBuffer > String （针对于单线程运行时）





### 20.List:

#### list.FindIndex：

在list中查找满足指定条件的元素，并返回其索引值

```c#
void FindDesignedDataOfList()
{
    List<BookData> bookList = new List<BookData>();
    bookList.Add(new BookData(){ id = 1, name = "book1"});
    bookList.Add(new BookData(){ id = 2, name = "book2"});
    int index = bookList.FindIndex(t => t.id == 2);
    Debug.LogFormat("index in bookList: {0}", index);
}

struct BookData
{
    public int id;
    public string name;
}
```

运行结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230809164829268.png" alt="image-20230809164829268" style="zoom:80%;" />

**注意**：在使用Struct定义数据结构时，并不需要在其内部添加“构造方法”。而使用“Class”若需要用`new BookData(1, "book1")`格式，则需要在其内部主动添加自定义构造方法





### 21.构造方法的重写：

当Class中默认的构造方法被覆盖后，此时再使用"New"创建class实例时则需要按照重写后的构造方法来赋值

```c#
class A {
    //覆盖默认的构造函数
    public A(int num){
        num = 100;
    }
}

class B {
    void TestMethod(){
        //由于“A”的构造方法被覆盖，因此必须按照新的构造方法来赋值，而不能使用 “A a = new A()”
        A a = new A(100);
    }
}
```



### 22.TryParse方法的特点：

使用int.TryParse, bool.TryParse, float.TryParse等方法时，如果数值无法转换，则保留参数的默认值，并不会被置为null

```c#
string num = "100";
bool.TryParse(num, out bool result);
Debug.LogFormat("result: {0}", result);

string str = "hello";
int.TryParse(str, out int result2);
Debug.LogFormat("result2: {0}", result2);
```

执行结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230208123428308.png" alt="image-20230208123428308" style="zoom:80%;" />



### 23.类型转换的特性：

在代码中经常需要将某个变量转变成其他类型，如:

```c#
object resource;
AssetBundle ab = resource as AssetBundle
```

如上这里将“resource”变量转换成“AssetBundle”。

但是在“类型转换”中有一个“隐藏的特性”：即如果该resource本身并不是“AssetBundle”类型，那么将其强转为“AssetBundle”后得到的“ab”必然为null

所以也可以直接通过**本方法来判断某个变量是否为指定类型**

**PS**：可以参考“`GameFramework`”框架中“`DefaultLoadResourceAgentHelper.cs`”脚本内的“`LoadAsset`”方法(该方法用于从`AssetBundle`中加载目标asset)



### 24.匿名委托的使用：

好处 —— 某些情况下可以直接写在方法体内部，直接使用方法体中的变量，而不需要另外声明该变量

坏处 —— 使用了匿名委托后，是无法直接的移除该匿名委托的。因为该匿名委托没有方法名，也无法查找到该委托。

​        直接置空“EditorApplication.update = null”并不是可取的办法，该方法会直接置空该委托

例子：借助Editor扩展中的“进度条”窗口方法实现的案例来解析

```c#
#region 查找目标文件在所有资源中的引用关系
[MenuItem("Custom/FindReferences")]
static void FindReference()
{
	string path = AssetDatabase.GetAssetPath(Selection.activeObject);
	string guid = AssetDatabase.AssetPathToGUID(path);

	//获取项目中所有的文件。“Application.dataPath”对应游戏开发目录，
	string[] paths = Directory.GetFiles(Application.dataPath, "*.*",  SearchOption.AllDirectories);
	if (paths.Length == 0) return;

	//在Editor模式下加入不断执行的“Update”更新机制
	int index = 0;
	string fileText = "";
	EditorApplication.update += delegate
	{
		bool isCancel = EditorUtility.DisplayCancelableProgressBar("查找引用",  paths[index], (float)(index / paths.Length));
		fileText = File.ReadAllText(paths[index]);
		if (Regex.IsMatch(fileText, guid))
			Debug.Log("<color=green>" + paths[index] + "</color>");
		++index;  //无论内容是否匹配，都会递增，以进入下一次检测
		//当点击进度条窗口中的取消按钮或者递增后的值超出界限后，则关闭匹配窗口
		if (isCancel || index >= paths.Length)
		{
			EditorUtility.ClearProgressBar();
			EditorApplication.update = null; //直接置空该委托——并不是可取的办法，但针对当前的情形，这是唯一可行的。
			return;
		}
	};
}
#endregion
```

有一种可能的解释：

**.net 中 委托类似于函数指针（不过 个人觉得 这个指针应该是可以指向 函数 或者是函数集合的指针） 其中匿名委托的指针的地址是随机分配的  重新移除 只不过清除一个相同函数体的匿名函数，并不能清除原来指向随机地址的 指针**

或者还有另一种解释：

**其实匿名委托 并没有开通新的指针  而可能是在定义的方法中  利用一块区域 构成局部 的小函数 这有点类似于 C语言中复合语句{}中定义的变量外部不可以使用  一个道理**



### 24.数学方法：

Mathf.Atan(float f) —— tan值为f的对应角度（弧度形式）

Mathf.Atan2(x, y); —— 两者是等价的，都是一样的结果，且都是弧度形式

Mathf.Rad2Deg   —— 用于转换，从弧度形式转换成角度形式

float angle = Mathf.Atan2(x, y) * Mathf.Rad2Deg ；

将角度转换成弧度： angle * Mathf.Deg2Rad

#### “System.Math”和“UnityEngine.Mathf”的区别：

在unity中有System.Math, 和 UnityEngine.Mathf。两者在实际使用中仅仅只是参数类型有不同要求。Math要求为double，Unity由于内部使用的是float，所以对System.Math中常用的一些数学方法进行了重新封装，以便开发者可以直接使用。

如：Mathf.Cos(float angle) 其实质为

```c#
float Mathf.Cos(float angle){

   return (float)Math.Cos((double)angle);

}
```

**因此Math比Mathf的运行速度会稍快一些，但非常有限**。游戏代码中通常使用Mathf



#### single代表单精度浮点型（float 为单精度浮点型，double为双精度浮点型）

```c#
int a = 20;
float b = 19.23f;
Debug.Log(a.GetType() + "     "+  b.GetType());  
//输出：System.Int32      System.Single
```

#### Mathf.Round

返回与参数最近的int数值：当参数小数点后第一位为5时，参数的整数部分为奇数则往前进一位，偶数则保持原样：

```c#
Debug.Log(Mathf.Round(2.5f) + "    @@@@      " + Mathf.Round(-5.5f));
//运行结果：   2    @@@@     -6
```

**注意**：与参数的正负无关

```c#
int a = Mathf.FloorToInt(12.68f); //小于等于，返回不大于目标参数的最大整数   12
int b = Mathf.CeilToInt(12.68f);  //大于等于，返回不小于目标参数的最小整数   13
int c = Mathf.RoundToInt(12.68f); //返回与目标参数最近的整数值   13
int d = Mathf.RoundToInt(12.48f); // 12
Debug.Log(a + "  " + b + "  " + c + "   " + d);
```



#### 向量叉乘：

```c#
a=(x,y,z), b=(x',y',z')
a x b=(yz'-zy',zx'-xz',xy'-yx')
```

 得到的法向量方向：从a到b，如果为顺时针，则法向量向着我们，否则逆向



**"Random.Range(float min, float max)" and "Random.Range(int min, int max)"的区别：**

前者参数为float时，最终结果中是可以包含max的，而后者参数为int时，则最终结果不包含max

**当需要比较大小时，如果暂时没有可用的值则可以使用：Mathf.infinity,代表无穷大**



### 数组：

#### 创建数组：

int[] a = new int[4]{1, 2, 3, 4}或int[] a = {1, 2, 3, 4}

如果使用int[] a = new int[4]，则数组中每个元素默认数值为0

#### 多维数组：

```c#
int[][] arr = new int[2][];
arr[0] = new int[]{1, 2};
int[,] arr2 = {{1, 2}, {4, 5}};
```

两种不同方式声明的arr，某些属性值也不同

如：arr.Length = 2, 但是arr2.Length = 4



#### Array.Sort(Array array): 

可以对数组进行排序，默认从小到大排序

#### Array.Reverse(Array array, int index, int length): 

交换数组中某部分元素的顺序 —— 倒序，本身并不具备排序的作用。但结合“已经排序完成”的数组，则可得到“相反的排列顺序”

```c#
int[] arr = {1, 2, 3};
Array.Reverse(arr);   
```

#### Array.Clear()：

该方法并不能删除数组中的元素，仅是将数组中的元素重置为默认数值

```c#
 int[] arr = { 1, 2, 3 };
 Array.Clear(arr, 0, 2);
 foreach(var temp in arr)
 {
     Debug.Log(temp + "    TEMP   ");
 }
```

执行结果：

![image-20231107092844995](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107092844995.png)

从以上输出结果可知，数组“arr”中前两个元素已被重置为int默认值“0”，而不是删除前两个元素





### 25.时间计算：

`TimeSpan.FromSeconds`：将指定秒数的时间转换成TimeSpan形式

当需要判断用户离线时长时，可以计算两次登录的时间差值，如果“timeSpan.Hour > 24”则说明离线时间超出一天。这在某些功能中可以用于计算奖励发放

```c#
//赋值可以是这样的ticks，也可以是上面那样全int数值的
TimeSpan t1 = new TimeSpan(date1.Ticks);   
TimeSpan t2 = new TimeSpan(date2.Ticks);
//Duration是表示绝对值的
TimeSpan temp = t2.Subtract(t1).Duration();   
string timeDiff = temp.Days + " " + temp.Hours + " " + temp.Minutes + " " + temp.Seconds;      
//temp.Days只是取出temp中“天”的部分
```

TimeSpan特有的实例方法：Subtract 减法； Add 加法； Duration 绝对值



`DateTime.Now.Ticks`：获取当前时间，以秒为单位

PS：部分情况在为新创建的文件命名时可加入该数值，以区分不同时间生成的文件，如“日志文件”等





### 26.函数参数默认赋值和指定参数传参

#### 函数参数默认赋值：

当声明某个函数时，可以直接为用到的某个参数设定默认值，如

```c#
void Method(string a, int b, int c = 100){}
```

但需要注意：**<font color=red>拥有“默认赋值的参数”必须靠后放置，并且从该参数起，后续所有参数都是“拥有默认赋值的参数”</font>**，即不允许`void Method(string a, int c = 100, int b)`这样的形式

#### 指定参数传参：

当需要为其中某个指定参数传递数值时，如

```c#
void Method(string a, int b, int c = 100, string d = "hello"){}
```

当需要为“参数d”传参，而“参数c”保持默认赋值时，则使用`Method("ok", 2, d:"today")`，即使用“:”指定目标参数d需要传递的数值，而“参数c”保持默认赋值即可





### 27.异步委托：

针对于需要监听委托执行是否结束的标志时，可以使用该方式：

```c#
using System.Threading;                            //因为会用到Thread，所以引用该命名空间

namespace myThread
{
    class Program
    {
        static string Method()
        {
            Thread.Sleep(200);                     //在使用a.BeginInvoke(null ,null);开启该线程时，该线程会先等待200ms后再执行。
            return "This is myStyle" ;             //线程的单位是毫秒
        }

        static void Main(string[] args)
        {
            Func<string > a = Method;
            IAsyncResult ar= a.BeginInvoke(null ,null);       //使用a.BeginInvoke(null ,null)开启线程，用IAsyncResult对象承接
            Console.WriteLine("main" );            //main线程仍然在执行，不影响，至于这两个线程的顺序是系统自动安排的
            while(ar.IsCompleted == false ){       //使用IAsyncResult对象，当该线程没有执行完时循环的输出
                Console.Write("." );
                Thread.Sleep(5);                   //同样可以使用sleep来使得其输出的频率降低，实际上这也是线程的灵活使用
            }
            string temp = a.EndInvoke(ar);         //当线程执行完毕时使用a.EndInvoke(ar)语句，取得线程执行结束后的结果
            Console.WriteLine(temp);
            Console.ReadKey();
        }
    }
}
```

检测当前的异步委托是否执行完毕，可以使用等待句柄 **ar.AsyncWaitHandle.WaitOne(int waitTime**);

```c#
void Start(){
        Func<string> a = MethodFunc;
        IAsyncResult ar = a.BeginInvoke(null, null);
        bool isEnd = ar.AsyncWaitHandle.WaitOne(10);  
        //括号的10代表等待10ms后该异步委托是否执行完毕，如果执行完毕则返回true，否则false。
        //注意该语句只会执行一次，并且只有在该语句执行完毕有了返回结果后，后面的语句才会执行。
        if (isEnd)
        {
            string temp = a.EndInvoke(ar);     //当异步委托执行完毕后可以得到该方法的返回值
            Debug.Log("TRUE   " + temp + "    @@@@@@@@@@@");
        }
        else
        {
            Debug.Log("<color=red>FALSE </color>");
        }
    }

    string MethodFunc()
    {
        int num = 0;
        for(int i = 0; i < 100; ++i)
        {
            num += i;
            Debug.Log("<color=yellow>迭代中</color>");
        }
        return "hello, the world. " + num;
    }     
```



### try...catch...finally：

**只要try执行，则finally一定会执行**

```c#
void TestFinallyMethod(){
      int[] a = {1, 2, 3};
      int b = a[3];         —— 已经异常 ，导致后面的try语句根本不会执行


      try{
            Debug.Log("try 语句执行");
            Debug.Log(b);
            return;
        }
       catch{
             Debug.Log("catch 语句执行");
         }
        finally{
              Debug.Log("finally 语句执行");
         }
}
```

finally语句执行的条件：try语句被执行，且进程没有被突然killed(可能是程序调用stop进程或硬件设备停电)，则finally语句才会被执行



#### try...catch...finally的执行顺序：

catch语句会被执行的条件——try语句在执行过程中出现异常，才会直接切换到catch语句，try模块里面的内容不会被继续执行

重点：在try模块的return执行前会先执行finally语句，并且如若try中的return是作为方法最终的返回，则在执行finally前，这里的值会被暂时保存起来。

但仅仅限于最终执行的return属于try模块而已。==finally模块中return语句没有任何意义==，==catch模块中的return语句只有在try出现异常时才会有效==。

对于return：整个方法一定只会执行一次return(或者是try模块，或者是catch模块)，但是finally模块一定在try或者catch模块中的return之前被执行

相当无敌的解析：https://blog.csdn.net/qq_39135287/article/details/78455525

```c#
int a = 2;
int b = 1;
int TryFunc()
{
    try
    {
        a = 3;
        Debug.Log(a + "  try     " + b);
        return a;
    }
    catch
    {
        Debug.Log("CATCH.........");
        return a;
    }
    finally
    {
        a = 4;   
        b = 10;
        Debug.Log("FINALLY......");
    }
}

void Start(){
    int num = TryFunc();
    Debug.Log(a + "     result    " + b + "    " + num);
}
```

finally的作用仅仅只是在实际执行中，对try或者catch其中之一起到辅助的作用（具体是哪个要看执行过程中是否有异常），

**并且finally中不需要return语句，不会对方法最终的return返回值造成任何影响，但是会对参数本身有影响**

运行结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107094026378.png" alt="image-20231107094026378" style="zoom:80%;" />



### delegate和event的区别：

event只能在本类型内部被触发，无法在外部触发该event，而delegate变量是可以在外部触发的。并且event反编译之后会发现其只有一个private修饰的委托类型的变量，以及add——“+=”， remove——“-=”方法

1).事件可以在外部被订阅，但是事件无法在外部被执行，即event的执行时机只能在内部被执行。

**因此事件通常用来在class内部某些属性变化或特定情况触发时来执行该事件下外部或者内部订阅的一系列方法**

2).对于委托和事件都可以使用“+=”“-=”来add或remove方法，但事件不可以使用“=”直接赋值，只能使用“+=”来

   并且委托当使用“=”赋值时，直接该委托添加的所有方法都会被重置，只有现在使用“=”赋值的方法了。所以注意”=“和”+=“的使用区别

​    委托类型有“Invoke”，“BeginInvoke”等方法，也可以直接的“delegateFuncInstanceTwo("delegateFuncInstanceTwo");”执行。

​    事件只能用“eventFunc("eventFunc");”，没有其他触发方法

**总结**：事件的好处在于当内部某些条件发生时会引发其他一系列连锁反应；而委托则相当于将一系列方法放在一起，接连一块的执行

**注意**：

1).**当委托使用”+“串联多个方法时，各个方法之间是独立，并不会因为属于同一个委托而传递数值。**

2).**一般情况下，多播委托通常返回值是void类型。但如果是有返回值的，那****么被添加上的众多方法，委托会依次执行，但是返回的结果只会是最后一个被执行的方法的返回值****，之前方法执行后返回值不会被记录下，该委托只会返回最后一个方法的返回值**



### Excel文件“*`.xls`”和“`*.xlsx`”的读取：

"xls" 和"xlsx"不同后缀的Excel文件需要使用不同的读取方法CreateBinaryReader、CreateOpenXmlReader

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231109185322071.png" alt="image-20231109185322071" style="zoom:80%;" />



### Json文件编写规则：

用中括号括起来的代表这里面存放的数组，就是说多个对象，当只有一个对象时就用大括号。如果里面有多个对象就用中括号。然后里面的键值采用冒号连接，各个键值对间用逗号相连

左边的键一定要用引号，表示这个键的名字叫什么，右边的代表这个键的值，看值的类型决定要不要加引号

例子如下：

```json
[
    {                                                                      
        "丐帮": [
            {
                "name": "洪七公",
                "menpai": "丐帮",
                "level": 10,
                "kongfu": "降龙十八掌",
                "age": 100
            },
            {
                "name": "黄蓉",
                "menpai": "丐帮",
                "level": 8,
                "kongfu": "打狗棒法",
                "age": 40
            },
            {
                "name": "苏乞儿",
                "menpai": "丐帮",
                "level": 9,
                "kongfu": "醉拳",
                "age": 70
            }
        ]
    },
    {
        "日月神教": [
            {
                "name": "任我行",
                "menpai": "日月神教",
                "level": 9,
                "kongfu": "吸星大法",
                "age": 90
            },
            {
                "name": "令狐冲",
                "menpai": "日月神教",
                "level": 9,
                "kongfu": "独孤九剑",
                "age": 40
            }
        ]
    }
]
```



### 序列化和反序列化：

**概述**：序列化是将对象以及与其相关的一些状态转换成二进制数据以方便后续的传输或存储。而反序列化则与其相反，专门代表：根据“二进制数据”还原成具体的对象实例。“序列化”的目的在于：方便情况需要，可以直接使用“byte数据”反序列化得到具体的实例对象

**使用方式**：

序列化类型：当需要将某个类型序列化时，需要在该类型前添加“`[System.Serializable]`”。如：

```c#
[System.Serializable]
public class Tree{
      public int height;
}

public class Forest{
      public Tree a;
}
```

序列化变量：当需要将某个变量序列化时，需要在该变量前添加`[SerializeField]`，如：

```c#
public class Forest{
     [SerializeField, Range(0, 100)]
     private int num;
}
```

**注意**：如果某类型支持序列化，并且该类型的变量使用public修饰，则其可以在Inspector面板中显示具体内容；若该变量使用private修饰，则需添加[SerializeField]修饰才能在Inspector面板中显示出来



###### 问题：为什么”readonly“，”const“，”static“修饰的变量不会被序列化？

**解答**：序列化是基于实例对象，而非某个类型的，其作用在于需要某个实例对象时可以直接根据数据文件将该实例在任何需要的地方马上恢复出来。因此在实例化时，需要将该实例相关的所有数据信息以数据流的形式保存下来或用于传输。

由于序列化是基于实例而存在，而实例在游戏运行中通常都是经常改变的，因此对这些实例对象而言，序列化也就显得很重要。

而static修饰的变量，其属于类型本身，而非寄托于实例而存在，因此也就不需要序列化；“readonly”修饰的变量除了声明以及构造方法中，其他情况无法修改该变量的数值，因此也无需序列化；const变量在声明时即需要赋值，且通常后续不会改变，因此也无需序列化



###### 问题：Dictionary字典类型是否支持序列化？

**解答**：C#中Dictionary字典类型是不支持序列化的。如果需要序列化，可以分别提取每个元素的key和value，放入“支持序列化”的List中存储；当需要反序列化时，则将Keys和Values列表中的元素分别提取出来。

实现过程如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111193511101.png" alt="image-20231111193511101" style="zoom:80%;" />



### 代码简化：

#### Debug.LogFormat

通常使用`Debug.LogFormat`时可以自定义输出内容，但同时也可以调整各参数的实际使用顺序：

```c#
Debug.LogFormat("hello {1}, world {0}, today {3}, is {2}", 1, 2, 3, 4);
```

运行结果：

![image-20220811141542763](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220811141542763.png)



#### “out”和“ref”关键字：

在方法体参数中使用“out”关键字时，如果需要调用该方法，那么对于"out"参数不需要在此之前声明：

```c#
RaycastHit hit;
Physics.Raycast(ray, out hit);
if(hit){
    ......
}
```

可以简化成：

```c#
Physics.Raycast(ray, out RaycastHit hit);
if(hit){
    .......
}
```

**实际例子使用：**

```c#
void Start(){
    GetNum(out int num);
    if (num > 0) {
        .........
    }
}

void GetNum(out int num) {
    num = 100;    //该方法结束前必须对num赋值，否则会报错
}
```

但是对于“ref”修饰的参数则必须<font color=red>**在调用该方法前先“声明并赋值”(只声明但不赋初值，同样会报错)**</font>

```c#
void Start(){
    int num = 100;
    GetNum(ref 100)
}

void GetNum(ref int num) { }  //该方法结束前无需对ref赋值等其他操作
```



#### "&"的使用：

当需要使用`if(m_EditorResourceMode && Application.isEditor){}`可以直接使用“&”：

```c#
m_EditorResourceMode &= Application.isEditor;
```

注意：“&=”是算术操作，而非“&&”的逻辑操作，但两者在这里的效果是一样的



#### 声明对象并赋值时的简化写法：

```c#
BarcodeWriter qrCodeWriter = new BarcodeWriter();
qrCodeWriter.Format = BarcodeFormat.QR_CODE;
qrCodeWriter.Options = new EncodingOptions { 
    Height = height, 
    Width = width 
};
```

可以简化成：

```c#
BarcodeWriter qrCodeWriter = new BarcodeWriter {
    Format = BarcodeFormat.QR_CODE,
    Options = new EncodingOptions {
        Height = height,
        Width = width
    }
};
```

如上两种方式都可以正常运行的，但相较而言，第二种更为简洁有力

但注意：这与构造方法完全无关，不要混淆

