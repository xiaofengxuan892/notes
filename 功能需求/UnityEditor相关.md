[TOC]



1.获取BuildSetting中要打包的Scene列表：

```c#
using UnityEditor.SceneManagement;

EditorSceneManager.GetSceneAt(0).path   //获取BuildSetting中第一个Scene的路径
```

2.在string类型变量前添加“[TextArea]”或“[TextField]”，可在Inspector面板中有更大的显示区域用于设置该string值

3.若需要某个public变量不显示在“Inspector面板”中，则可在该变量前添加“`[HideInInspector]`”标记

注意：`[HideInInspector]`仅控制该变量在Inspector面板的显示，与该变量是否会被序列化完全没有关系，不可混淆

4.当某个变量或类型前有多个标记时，可将多个标记写在一起，各个标记间使用“,”连接，如：

```c#
[SerializeField, Range(0, 100)]
private int num;
```

##### Editor模式下常用的编辑关键字：

###### 在菜单栏添加方法名：MenuItem、AddComponentMenu、ContextMenu

`MenuItem`：可以在任意菜单中添加自定义方法，使用形式为“`[MenuItem("name 快捷键", 是否显示，优先级)]`”，**<font color=red>该方法需要为static修饰</font>**

**在MeunItem中调用其他已经定义好的MenuItem方法**：通过EditorApplication.ExecuteMenuItem可以调用已使用MenuItem定义过的方法

```c#
[MenuItem("Custom/Trick")]
static void Trick()
{
    Debug.Log("A little trick.");
}

[MenuItem("Custom/ExcuteFunc")]
static void ExcuteFunc(MenuCommand menuCommand)
{
    EditorApplication.ExecuteMenuItem("Custom/Trick");
}
```

`AddComponentMenu`：默认在“菜单栏Component”中添加子菜单，针对于类型，相当于为某个物体添加组件

`ContextMenuItem`：在Inspector面板脚本右侧的菜单列表中添加方法**<font color=red>，该方法只需要“public”修饰即可</font>**，无需static方法，使用形式为“[ContextMeunItem("name"，"method")]”，如：

```c#
public int num = 100;

[ContextMenuItem("hello", "MethodFunc")]
void MethodFunc()
{
    Debug.Log("HELLO.....   " + num);
}
```

效果如下图：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112190347941.png" alt="image-20231112190347941" style="zoom:80%;" />

###### 其他常用标记：

[RequireComponent(typeof("component name"))] —— 本组件依赖于其他组件而运行，如果该物体当前没有添加该组件则会报错

[HelpURL("[https://www.google.com](https://www.google.com/)")] -- 组件“？”点击后自动打开目标网页 

[Header("name")] -- 为下面出现的一系列变量添加描述

[Multiline(int line)]-- 为变量提供多行显示的输入

[Range(min, max)]-- 提供滑动的变量区间

[Space(float height)]-- 相邻变量间设置显示距离

[Tooltip("tip")]-- 当鼠标在变量上时即显示

```c#
[Header("一些参数变量：")]
public Vector3 num;
[Multiline(6)]
public string folderPath = "";
[Range(-6, 6)]
public int value;
[Space(50)]
[Tooltip("这是一条tooltip")]
public char letter;
```

[InitializeOnLoad] -- 用于修饰class，启动Unity时会自动执行。当需要针对方法时，则使用[InitializeOnLoadMethod]

```c#
using UnityEditor;
using UnityEngine;

[InitializeOnLoad]
class MyClass
{
    static MyClass ()   -- 静态构造方法
    {
        EditorApplication.update += Update;
    }
    static void Update ()
    {
        Debug.Log("Updating");
    }
}
```

**ExcuteInEditMode**：在脚本前添加该语句，则脚本中的方法在Edit模式下也会执行。通常用于在Inspector检测数值的变化，调试专用

###### Selection关键字的作用：

```c#
Object[] labels = Selection.GetFiltered(typeof( GameObject), SelectionMode.Deep);
```

解析：选择的模式是deep类型，该类型包括了本物体及其所有子物体的子物体，这就是所谓的“deep”模式

##### 监听prefab数据变化 —— `PrefabUtility.prefabInstanceUpdated`：

```c#
[InitializeOnLoadMethod]
static void PrefabFunc()
{
    PrefabUtility.prefabInstanceUpdated += delegate (GameObject instance)
    {
        Debug.Log("  update @@@@@@@@@");
    };
}
```

PrefabUtility.prefabInstanceUpdated是委托类型，当prefab中数据发生变化时即会触发。

**注意**：以上代码中需要添加`[InitializeOnLoadMethod]`标记，使用`[ExecuteInEditMode]`是不管用的

##### 展示进度条对话框 —— EditorUtility.DisplayCancelableProgressBar和EditorApplication.update

EditorUtility.DisplayCancelableProgressBar：展示进度条面板，使用方式为：

```c#
EditorUtility.DisplayCancelableProgressBar("进度条", "匹配中。。。。",  0.5f)
```

展示效果为：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112172150943.png" alt="image-20231112172150943" style="zoom:80%;" />

但是上图中点击“Cancel”按钮后无法取消该面板，因此需要结合“EditorApplication.update”来实现：

```c#
[MenuItem("Custom/OtherTools")]
static void OtherCustomTools()
{
    progressTime = 0f;
    progressCount = 1;
    EditorApplication.update += ShowProgressBar;    //注册事件中的方法
}

static float progressTime = 0f;    //为了得到进度条循序渐进增长的效果，引入计时
static int progressCount = 1;      //随着数值的增长，进度条逐渐增长
static void ShowProgressBar()
{                                                                //进度条最大值是1，为了观察方便，引入“0.1f”降低进度增长过程
    bool isCanceled = EditorUtility.DisplayCancelableProgressBar("进度条", "匹配中。。。。", progressCount * 0.1f);
    progressTime += Time.deltaTime;                   
    if (progressTime >= 20f)
    {
        ++progressCount;
        progressTime = 0;
        Debug.Log("<color=red>" + progressCount + "</color>   <i>" + Time.deltaTime +  "</i>");
    }
    if (isCanceled || progressCount > 10)
    {
        EditorUtility.ClearProgressBar();               //关闭进度条窗口
        EditorApplication.update -= ShowProgressBar;    //注销事件中的方法
    }
}
```

EditorApplication.update类似于“`MonoBehavior.Update`”，专用于处理“Editor模式”中“需要每帧执行”的逻辑。只有这样，当点击“Cancel按钮”时，才能实时监测到该面板的状态并通过“isCanceled”参数调用“`EditorUtility.ClearProgressBar()`”方法关闭该进度条面板，同时注销该`EditorApplication.update`委托监听

**PS：“Editor模式”和“Playmode模式”下，Time.deltaTime数值的区别**

分别输出Editor模式和Playmode模式下Time.deltaTime的数值：

**Editor模式**：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112173053487.png" alt="image-20231112173053487" style="zoom:80%;" />

**Playmode模式**：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112173115284.png" alt="image-20231112173115284" style="zoom:80%;" />

**总结**：EditorApplication.update和MonoBehavior.Update分别用于“Editor模式”和“Playmode模式”下需要每帧执行的逻辑，由于Editor模式下并不需要频繁执行其中的Update逻辑，因此“Time.deltaTime”数值较大，而Playmode模式中Time.deltaTime数值则较小

###### 查找指定文件在项目中的引用记录(基于“进度条”特性实现)

```c#
#region 查找目标文件在所有资源中的引用关系
[MenuItem("Custom/FindReferences")]
static void FindReference()
{
    string path = AssetDatabase.GetAssetPath(Selection.activeObject);
    string guid = AssetDatabase.AssetPathToGUID(path);  //根据GUID来查找引用情况

    //获取项目中所有的文件。“Application.dataPath”对应游戏开发目录
    string[] paths = Directory.GetFiles(Application.dataPath, "*.*",  SearchOption.AllDirectories);
    if (paths.Length == 0) return;
    //在Editor模式下加入不断执行的“Update”更新机制

    int index = 0;
    string fileText = "";
    EditorApplication.update += delegate  //匿名委托的好处在于可以直接使用原方法体中的参数，而不用把目标参数设置成全局的
    {
        bool isCancel = EditorUtility.DisplayCancelableProgressBar("查找引用", paths[index],  index / (float) paths.Length);

        fileText = File.ReadAllText(paths[index]);
        if (Regex.IsMatch(fileText, guid))
            Debug.Log("<color=green>" + paths[index] + "</color>");

        ++index;  //无论内容是否匹配，都会递增，以进入下一次检测

        //当点击进度条窗口中的取消按钮或者递增后的值超出界限后，则关闭匹配窗口
        if (isCancel || index >= paths.Length)
        {
            EditorUtility.ClearProgressBar();
            EditorApplication.update = null; //直接置空该委托——并不是可取的办法，但针对当前的情形，这是唯一可行的。
            //匿名委托其好处在于可以直接写在方法体内部，这样可以直接使用方法体中的变量，而不需要声明该变量为全局变量，直接局部变量也可以使用
            //坏处在于，使用了匿名委托后，是无法直接的移除该匿名委托的。因为该匿名委托没有方法名，也无法查找到该委托。
            //直接置空“EditorApplication.update = null”并不是可取的办法
            return;
        }
    };
}
#endregion
```

执行结果：当任意选中一个物体执行本方法后，会出现类似如下的结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112174400962.png" alt="image-20231112174400962" style="zoom:80%;" />

**注意**：当表示进度条递增效果时，使用以下两种形式，其结果是完全不同的：

```c#
//形式一：进度结果始终显示为“0”
EditorUtility.DisplayCancelableProgressBar("查找引用", paths[index], (float)(index / paths.Length))
//形式二：进度结果可以正常显示
EditorUtility.DisplayCancelableProgressBar("查找引用", paths[index], index / (float) paths.Length)
```

**原因**：

**(float)(index / paths.Length)**： 代表两个int数值的参数相除，其结果必定为int，而当index < paths.Length时，其相除的结果必定为“0”，纵然在商的外部强行转换为float，也只是将int型的“0”转换成float型的“0”

**index / (float) paths.Length**： 在相除之前已经完成了转换，其结果自然的保留为float型 —— 不论是转换“index”或“paths.Length”都可以

##### 打开外部操作系统的“文件”和“文件夹”窗口，或打开“对话框”窗口

```c#
void Update()
{
    if (Input.GetKeyDown(KeyCode.W))
    {
        string filePath = EditorUtility.OpenFilePanel("打开文件窗口", "", "");
        Debug.Log(filePath + "@@@@@@@@@@@@");
    }
    if (Input.GetKeyDown(KeyCode.Q))
    {
        string folderPath = EditorUtility.OpenFolderPanel("打开文件夹窗口", "", "");
        Debug.Log("<COLOR=YELLOW> " + folderPath + "</color>");
    }
    if (Input.GetKeyDown(KeyCode.E))
    {
        bool flag = EditorUtility.DisplayDialog("对话框窗口", "看看对话框窗口的样子",  "请点击");
        Debug.Log("<COLOR=BLUE> " + flag + "</COLOR>");
    }
}
```

**解析**：通过“OpenFilePanel”，“OpenFolderPanel”打开文件和文件夹窗口时，可以直接得到被打开文件的绝对路径

“DisplayDialog”可以根据是否点击“OK”按钮得到返回值。部分需求可能需要以上方法的返回值

“对话框”示意图如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112175328560.png" alt="image-20231112175328560" style="zoom:80%;" />

当点击上图中的按钮后，console中输出如下日志：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112175354318.png" alt="image-20231112175354318" style="zoom:80%;" />

###### 打开窗口自由设定文件的保存路径：

该功能可通过`EditorUtility.SaveFilePanel`来实现

当需要利用Unity来创建某个文件时，可以在程序中指定其存储路径：

```c#
string path = EditorUtility.SaveFilePanel("保存Excel文件", "Assets/Resources/", "",  ".xls");
```

parameter 1：代表该windows panel的标识

parameter 2：可以为空（**该EditorUtility.SaveFilePanel方法默认打开本项目所在的目录**），若需要自主设定目录，可以再设定(仍旧是相对于该项目目录)

parameter 3: 可以为空

parameter 4：文件的后缀

**注意**：==该EditorUtility.SaveFilePanel只是返回path路径==，==并不是直接在文件夹下创建实体资源==，对于unity资源对象可借助”AssetDatabase.CreateAsset“来创建实体资源。

如果是File对象，则直接使用File.Create；FileInfo则需要借助实例化对象来调用Create方法，两种方式都可以创建实体文件



**打开指定文件夹命令**：`EditorUtility.RevealInFinder(Application.persistentDataPath)`



##### Editor模式下实现的自定义效果：

###### 1.自定义任意组件在Inspector面板中的显示效果：

自定义组件“Frank”的效果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112193353827.png" alt="image-20231112193353827" style="zoom: 80%;" />

**实现过程**：

1).首先编写继承MonoBehavior的组件脚本Frank，内部包含所有可以被其他组件调用的变量和方法

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public enum Gender
{
    male = 2, mid,  female = 5
}
public enum Company
{
    NINTENDO, GOOGLE, NETFLIX, PINTEREST
}
public class Frank : MonoBehaviour {
    public string name;     //这里的参数name并不是最终显示在inspector面板中的name，但是该name是唯一可以被其他class获取到的参数name
    public int age;
    public Gender gender;
    public int stage;
    public string target;
    public string method;
    public string key;
    public string tasks;
    public string prohibiton;
    public float year = 20f;
    public Company[] favoredCompany = new Company[4];
    public string level1, level2, level3;
    public string tips1, tips2, tips3;
  
    void Start () {
        Debug.Log((int)gender + "   GENDER   ");
        Debug.Log(stage + "     STAGE    ");
    }
}
```

2).自由设定该组件“Frank”中的参数的显示样式

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

[CustomEditor(typeof(Frank))]   //对目标组件“Frank”在Inspector中的视图作自定义显示
public class FrankModification : Editor
{
    Frank frank;
    bool flag;

    void OnEnable()
    {
        frank = (Frank)target;
    }
    
    //重写目标组件Frank的inspector UI显示，因此需要重写“OnInspectorGUI”方法
    public override void OnInspectorGUI()
    {
        //base.OnInspectorGUI();  //倘若依旧调用基类的“OnInspectorGUI”，则之前的参数变量会重新绘制一遍，如此则没有override的必要了
        EditorGUILayout.BeginVertical();
        EditorGUILayout.LabelField("IDENTITY INFO");
        frank.name = EditorGUILayout.TextField("NAME", frank.name); //这里的“NAME”才是最终显示在Inspector面板中的
        frank.age = EditorGUILayout.IntField("AGE", frank.age);
        frank.gender = (Gender)EditorGUILayout.EnumPopup("GENDER", frank.gender);
        frank.stage = EditorGUILayout.Popup("STAGE", frank.stage, new string[] { "CHILD =  1", "YOUNG = 2", "ADULT = 3", "OLD = 4" });
        EditorGUILayout.Space();
        EditorGUILayout.Space();
        EditorGUILayout.LabelField("DETAILED INFO");
        frank.target = EditorGUILayout.TextField("TARGET", frank.target);
        frank.method = EditorGUILayout.TextField("METHOD", frank.method);
        frank.key = EditorGUILayout.PasswordField("KEY", frank.key);
        EditorGUILayout.LabelField("TASKS");
        frank.tasks = EditorGUILayout.TextArea(frank.tasks, GUILayout.Height(50));
        EditorGUILayout.LabelField("ATTENTION");
        EditorGUILayout.HelpBox("NO LIVE BROADCAST IS ALLOWED", MessageType.Error);
        EditorGUILayout.Space();
        EditorGUILayout.Space();
        EditorGUILayout.Space();
        EditorGUILayout.LabelField("FORMER HALF LIFE");
        frank.year = EditorGUILayout.Slider("YEAR", frank.year, 0, 50);

        //使用不同的颜色显示progressBar
        if (frank.year < 13)
            GUI.color = Color.Lerp(Color.white, Color.gray, 0.5f);
        else if (frank.year >= 13 && frank.year < 15)
            GUI.color = Color.red;
        else if (frank.year >= 15 && frank.year < 19)
            GUI.color = Color.Lerp(Color.red, Color.black, 0.6f);
        else if (frank.year >= 19 && frank.year < 23)
            GUI.color = Color.Lerp(Color.yellow, Color.green, 0.7f);
        else if (frank.year >= 23 && frank.year < 29)
            GUI.color = Color.Lerp(Color.green, Color.red, 0.5f);
        else if (frank.year >= 29)
            GUI.color = Color.green;
        //下面的语句很重要，难点在于在“Year”滑动条后获取到相对的位置区间。
        //因此借助“GUILayoutUtility.GetRect”和“EditorGUI.ProgressBar”来实现
        Rect progressRect = GUILayoutUtility.GetRect(200, 30); //这里并没有设置progressBar的offset，而只是设置其长宽区域
        EditorGUI.ProgressBar(progressRect, frank.year /50 , "");

        GUI.color = Color.white; //为了防止上面的GUI.color影响到下面控件的绘制，因此重置GUI.color
        //借助EditorGUILayout.HelpBox来显示不同的描述
        if (frank.year < 13)
            EditorGUILayout.HelpBox("OBSCURE,       VAGUE,      DULL", MessageType.Info,  true);
        else if (frank.year >= 13 && frank.year < 15)
            EditorGUILayout.HelpBox("BURNING,       TIRED,      BLOOD,      LITTLE HAPPY  BUT PROUD", MessageType.Info);
        else if (frank.year >= 15 && frank.year < 19)
            EditorGUILayout.HelpBox("CRYING,        EXTREMELY DARK,    DEEP DEPRESSION,       SUICIDE", MessageType.Error);
        else if (frank.year >= 19 && frank.year < 23)
            EditorGUILayout.HelpBox("DUMMY,         WILT,       POWERLESS,       WRONG    ", MessageType.Warning);
        else if (frank.year >= 23 && frank.year < 29)
            EditorGUILayout.HelpBox("LIGHT BURST,    INDEPENDENT,     REAL WORKING,      LOAN,      WORTHY", MessageType.Warning);
        else if (frank.year >= 29)
            EditorGUILayout.HelpBox("THIS IS ME,              MY OWN KINGDOM ",  MessageType.Info);
        EditorGUILayout.Space();
        EditorGUILayout.Space();

        //在Inspector中显示自定义类型
        SerializedProperty favoredCompany =  serializedObject.FindProperty("favoredCompany");
        //将组件Frank中的favoredCompany变量绘制在面板中
        EditorGUILayout.PropertyField(favoredCompany, new GUIContent("FAVORED COMPANY"),  true);
        SerializedProperty softFave = favoredCompany.GetArrayElementAtIndex(3);
        EditorGUILayout.PropertyField(softFave, new GUIContent("SOFT FAVE")); //另外添加显示
        serializedObject.ApplyModifiedProperties();  //绘制自定义的参数显示样式

        EditorGUILayout.Space();
        EditorGUILayout.Space();

        //下面的内容是否需要显示出来，“Foldout”代表折叠的小箭头，根据不同的折叠情况——当前是折叠或者不折叠，可以得到不同的返回值flag
        flag = EditorGUILayout.Foldout(flag, "TIPS");
        if (flag)
        {
         //as for "EditorGUILayout.TextField(label, string, GUILayout.Width)", the  third parameter is used in the whole which contains two parts
            //"label and field. The area of "label" and area of "field" is allocated by  system. The former one usually don't need so much space.
            //such that we depart these two and allocate designed space for the first  one.
            //the space of the second "field" accounts for is distritubed by system
            EditorGUILayout.BeginHorizontal();
            EditorGUILayout.LabelField("ONE", GUILayout.MaxWidth(30));
            frank.level1 = EditorGUILayout.TextField(frank.level1);
            EditorGUILayout.LabelField("TWO", GUILayout.MaxWidth(32));
            frank.level2 = EditorGUILayout.TextField(frank.level2);
            EditorGUILayout.LabelField("THREE", GUILayout.MaxWidth(40));
            frank.level3 = EditorGUILayout.TextField(frank.level3);
            EditorGUILayout.EndHorizontal();
            frank.tips1 = EditorGUILayout.TextField("TIPS1", frank.tips1);
            frank.tips2 = EditorGUILayout.TextField("TIPS2", frank.tips2);
            frank.tips3 = EditorGUILayout.TextField("TIPS3", frank.tips3);
        }
        EditorGUILayout.EndVertical();
    }
}
```

###### 2.自定义窗口Window，并创建一系列菜单：

效果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112193709269.png" alt="image-20231112193709269" style="zoom:80%;" />

实现代码：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

public class WindowCustom : EditorWindow {
    int KEYPOINT = 0;
    
    //在“OnGUI”外设定window的区域才能保证“GUI.DragWindow”正常运行
    //Rect rect = new Rect(100, 150, 150, 150);
    //当在scrollview下绘制subwindow时，“Rect”区域的x, y偏移需要以scrollview左上角为基准
    //GUI控件的绘制都是以左上角为(0, 0)来计算偏移量的
    Rect windowRectUnderScrollview = new Rect(0, 0, 250, 250);
    Vector2 scrollPos = Vector2.zero;  //用于scrollview中的slider起始位置
    Vector2 scrollPos2 = Vector2.zero; //子窗口中的scrollview的slider起始位置
    Texture smoothIcon; //used for second scrollview

    [MenuItem("Window/Create MyWindow")]
    static void Init()
    {
        EditorWindow.GetWindow<WindowCustom>();
    }

    void OnEnable()
    {
        //在自定义窗口的左侧显示图标
        Texture myIcon = (Texture)EditorGUIUtility.Load("Icons/myIcon.png");
        titleContent = new GUIContent("  MyWindow  ", myIcon);
        smoothIcon = (Texture)EditorGUIUtility.Load("Icons/smooth.jpg");
    }

    void OnGUI()
    {
        if (GUILayout.Button("ONEPLUS"))
        {
            //“KEYPOINT.Equals()”可以得到一个在多个子menu中选中的效果：当返回true时，该menu前被勾选；否则不勾选
            GenericMenu menu = new GenericMenu();
            menu.AddItem(new GUIContent("FLUID SCREEN"), KEYPOINT.Equals(1), ShowDetails,  1);
            menu.AddItem(new GUIContent("OXYGENON OS"), KEYPOINT.Equals(2), ShowDetails,  2);
            menu.AddSeparator("");//在多个菜单选项间添加间隔，利于显示美观
            menu.AddItem(new GUIContent("PERFORMANCE/CHIPSET"), KEYPOINT == 31,  ShowDetails, 31);
            menu.AddItem(new GUIContent("PERFORMANCE/UFS"), KEYPOINT == 32, ShowDetails,  32);
            menu.AddSeparator("PERFORMANCE/");//在子菜单间再添加间隔
            menu.AddItem(new GUIContent("PERFORMANCE/HAPTIC ENGINE"), KEYPOINT == 33,  ShowDetails, 33);
            menu.AddItem(new GUIContent("PERFORMANCE/DOLBY ATMOS"), KEYPOINT == 34,  ShowDetails, 34);
            menu.AddSeparator("");
            menu.AddItem(new GUIContent("WARP CHARGE"), KEYPOINT == 4, ShowDetails, 4);
            menu.ShowAsContext();
        }

        switch (KEYPOINT)
        {
            case 1:
                EditorGUILayout.TextArea("EXTREMELY SMOOTH LIKE NEVER BEFORE",  GUILayout.MaxHeight(100));
                scrollPos = GUI.BeginScrollView(new Rect(50, 150, 300, 300), scrollPos,  new Rect(0, 0, 1000, 1000));
                //在editorWindow中绘制subwindow必须要“BeginWindow”和“EndWindow”配合使用
                //GUI 和 GUILayout间的区别：前者需要指定Rect区域，后者则会根据当前GUI控件的绘制情况设置Rect区域
                BeginWindows();
                windowRectUnderScrollview = GUI.Window(1, windowRectUnderScrollview,  DrawWindow, "SMOOTH");
                EndWindows();
                GUI.EndScrollView();
                break;
            case 2:
                EditorGUILayout.BeginVertical();
                EditorGUILayout.LabelField("FEATURES");
                EditorGUILayout.TextArea("1.BASED ON ANDROID 10\r\n\r\n2.INTEGRATED  GOOGLE PACKAGE\r\n\r\n3.CLEAN AND CONCISE",
                                          GUILayout.MaxHeight(100));
                EditorGUILayout.EndVertical();
                break;
            case 31:
                EditorGUILayout.TextArea("SNAPDRAGON 855 PLUS, SUPER FASE CPU AND GPU",  GUILayout.MaxHeight(100));
                break;
            case 32:
                EditorGUILayout.TextArea("SPEED OF INPUT AND OUTPUT OF FILES TO A NEW  LEVEL", GUILayout.MaxHeight(100));
                break;
            case 33:
                EditorGUILayout.TextArea("GOOD FEEDBACK OF SHAKE, MORE LIVELY",  GUILayout.MaxHeight(100));
                break;
            case 34:
                EditorGUILayout.TextArea("IMMERSIVE SOUND EXPERIENCE",  GUILayout.MaxHeight(100));
                break;
            case 4:
                EditorGUILayout.HelpBox("AMAZING SUPER FAST CHARGING EXPERIENCE",  MessageType.Error);
                break;
        }
    }

    void ShowDetails(object obj)
    {
        KEYPOINT = (int)obj;    //非常重要，当选择某菜单选项时，将“KEYPOINT”设置为此菜单对应的数值
    }

    //子窗口的绘制细节
    void DrawWindow(int id)
    {
        scrollPos2 = GUI.BeginScrollView(new Rect(50, 50, 150, 150), scrollPos2, new  Rect(0, 0, 500, 500));
        GUI.DrawTexture(new Rect(20, 20, 100, 100), smoothIcon);
        GUI.EndScrollView();
        //the subwindow area word"Rect rect = new Rect(0, 0, 150, 150)" must be out of  "OnGUI" or "Update"
        //"rect" become a global var, which makes it draggable
        GUI.DragWindow();
    }
}
```

文件资源的结构：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231112193821581.png" alt="image-20231112193821581" style="zoom:80%;" />

解析：

1).当在自定义的editorWindow中绘制subwindow时，需要使用”BeginWindows","EndWindow"

2).“GUI.BeginScrollView”所占据的区域不需要很大，其代表的是直接显示在外部的view大小；真正决定可以scroll的区域的是slide的区域

**即可以****GUI.BeginScrollView的区域很小，但是该区域内可以滑动的区域却很大，体现出来的则是很小的一个区域内，借助slider可以滑动的面积很大**

在scrollview内部绘制时，所有的GUI控件都是相对于该scrollview的相对位置

```c#
scrollPos = GUI.BeginScrollView(new Rect(50, 150, 300, 300), scrollPos,  new Rect(0, 0, 1000, 1000));
```

第一个参数：代表该scrollview所占据的空间大小

第二个参数：代表该slider初始位置

第三个参数：代表该slider可以滑动的区域

3).除了“GUI.window”，“GUILayout.window”用于绘制window的功能外，其他所有的控件绘制都需要在“OnGUI”中

4).当使用GUI.DragWindow时，该window的绘制区域必须是固定 —— 只有这样才能拖着这个固定的区域在不固定的空间中拖动

因此，**当首先使用GUI.Window绘制窗口时，第一个参数Rect必须在"OnGUI"外先定义出来**——由于OnGUI刷新的特性，Rect无法被固定下来，每一帧都要重新定义

dragWindow的功能是**只能在GUI.Window中的方法体中执行的**







































