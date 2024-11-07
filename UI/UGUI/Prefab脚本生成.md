[TOC]



在业务逻辑开发中，经常需要调用页面prefab中的某个节点，如text, image, button等，若手动填写节点路径，则显得太过繁琐。因此开发此工具，其会自动提取prefab下“特殊命名前缀”节点的相应组件，在实际业务逻辑中，直接调用该节点name即可

#### 实现原理：

1.该工具的执行需要事先定义提取规则：==只有特殊命名前缀的节点才会被提取对应组==件。规则如下：

- m_text前缀 —— Text组件
- m_img前缀 —— Image组件
- m_btn前缀 —— Button组件
- m_vlay前缀 —— Vertical Layout  Group组件
- m_hlay前缀 —— Horizontal Layout Group组件

2.为了简化代码，使其美观易读，故==将“组件提取”逻辑与“该界面实际的业务执行逻辑”分离出来，创建两个脚本==，分别为：**xxx.cs** —— 负责该界面的业务逻辑， **xxx_Bind.cs** —— 只存储“组件提取”逻辑，==两者使用“partial”关键字，同属于一个Class==

- **partial关键字在定义方法时同样有效**。如在“xxx_Bind.cs”中声明==“partial void MethodA();” —— 不包含方法体==，即可在“xxx_Bind.cs”内部正常使用该MethodA方法，如“按钮监听”或直接执行该方法“MethodA();”。在“xxx.cs”脚本中，则实现该方法==“partial void MethodA(){}” —— 包含方法体==，同样也可以正常调用该方法

#### 实现步骤：

1.编辑脚本生成工具 “UIEditorScriptGenerator.cs”：

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Text;
using Unity.VisualScripting;
using UnityEditor;
using UnityEngine;

//每个"特殊前缀命名"节点的数据
struct NodeInfo
{
    public string paramDeclaration;            //生成的脚本中该节点的“参数声明”字符串
    public string paramAssignment;             //该参数在脚本中的“赋值语句”字符串
}

//提取prefab中各个“特殊命名前缀”节点的目标组件，并将其与该界面的执行逻辑分离，简化脚本
//实现过程：1.提取prefab所有“特殊命名前缀”的节点数据  2.为该prefab生成脚本文件
public class UIEditorScriptGenerator : Editor
{
    //存储所有“特殊前缀命名”节点的信息
    private static List<NodeInfo> allSpecialNodeInfos = new List<NodeInfo>();

    //1.仅针对特殊前缀的节点提取其对应组件  2.若UGUI组件做了定制化，则组件名称可能已修改，直接在这里调整即可
    //3.由于最终生成的脚本文件中也是由“字符串”构成，因此这里可以直接将“目标组件”用“string”定义
    private static Dictionary<string, string> allSpecialNamePrefix = new Dictionary<string, string>() {
        //一般
        {"m_text", "Text"},
        {"m_img", "Image"},
        {"m_tf", "Transform"},
        {"m_rect", "RectTransform"},
        {"m_vlay", "VerticalLayoutGroup"},
        {"m_hlay", "HorizontalLayoutGroup"},

        //特殊
        {"m_go", "GameObject"},


    };

    private const string NAMESPACE_STR = "XGame1";
    //每行代码前的缩进(默认为4个空格)
    private const string CODE_INDENTATION_STR = "    ";

    [MenuItem("Assets/ScriptGenerator/Prefab脚本生成器/Window")]
    public static void GenerateScriptsForWindow() {
        GenerateScripts("UIWindow");
    }

    [MenuItem("Assets/ScriptGenerator/Prefab脚本生成器/Panel")]
    public static void GenerateScriptsForPanel() {
        GenerateScripts("UIPanel");
    }

    private static void GenerateScripts(string baseClassName) {
        var obj = Selection.activeObject;
        if(null == obj) return;
        if (!(obj is GameObject go)) return;

        //非prefab则不执行
        var objType = PrefabUtility.GetPrefabAssetType(go);
        if(objType == PrefabAssetType.NotAPrefab) return;

        //开始脚本生成逻辑
        allSpecialNodeInfos.Clear();      //清空集合中的节点数据
        //1.先获取特殊节点数据(prefab根节点无需检测，故“ownPath”为空)
        GeneratorDataForAllNodes(go.transform, "");
        //为代码美观，按string长度对节点进行排序
        allSpecialNodeInfos.Sort((x, y) =>
            x.paramDeclaration.Length.CompareTo(y.paramDeclaration.Length));
        //2.生成脚本
        var assetPath = AssetDatabase.GetAssetPath(go);
        var assetDirPath = Path.GetDirectoryName(assetPath);
        if(string.IsNullOrEmpty(assetDirPath)) return;
        var assetDirInfo = new DirectoryInfo(assetDirPath);
        var scriptDirName = assetDirInfo.Name;          //脚本文件所属的文件夹名字
        GenerateBindScript(go.name, scriptDirName);
        GenerateMainScript(go.name, $"{baseClassName}", scriptDirName);

        //脚本生成结束后
        allSpecialNodeInfos.Clear();
        AssetDatabase.Refresh();
        Debug.Log("Generate all scripts finished.");
    }


    // ====================== START: 为Prefab生成脚本文件 =======================
    //生成该prefab的脚本文件A，代码内容主要包含：1.获取各个特殊命名前缀节点的目标组件  2.添加按钮监听以及方法声明
    //PS: 1.该文件每次都会重新生成(若同路径下已有该文件，则旧文件会被删除，重新生成新的)
    //2."className"为生成的脚本名字， “directoryName”为该脚本所在的文件夹的名字
    private static void GenerateBindScript(string className, string directoryName) {
        var scriptCode = new StringBuilder();
        scriptCode.Append("//Auto generate by tools. Don't modify manually.\n");
        scriptCode.Append("\n");
        //需要引用的namespace
        scriptCode.Append("using UnityEngine;\n");
        scriptCode.Append("using UnityEngine.UI;\n");
        scriptCode.Append("\n");

        //本脚本所属的namespace
        scriptCode.Append($"namespace {NAMESPACE_STR}\n");
        scriptCode.Append("{\n");
        //1.由于该脚本非工具类，仅用于表示该prefab的UI界面，因此不会被外部dll调用，故添加“internal”关键字
        //2.由于将prefab的各特殊组件和实际执行逻辑相分离，故添加“partial”关键字
        scriptCode.Append($"{CODE_INDENTATION_STR}internal partial class {className}\n");
        scriptCode.Append($"{CODE_INDENTATION_STR}{{\n");

        //所有“特殊命名前缀”的节点信息
        foreach (var v in allSpecialNodeInfos) {
            //添加节点的“参数声明语句”
            scriptCode.Append($"{CODE_INDENTATION_STR}{CODE_INDENTATION_STR}{v.paramDeclaration}\n");
        }
        scriptCode.Append("\n");
        //基类“UIBase”中包含“AutoFetchComponents”方法，这里进行重写，且该方法外部不会调用，故使用“protected”关键字修饰
        scriptCode.Append($"{CODE_INDENTATION_STR}{CODE_INDENTATION_STR}protected override void AutoBindComponents()\n");
        scriptCode.Append($"{CODE_INDENTATION_STR}{CODE_INDENTATION_STR}{{\n");
        //添加节点的“参数赋值语句”
        foreach (var v in allSpecialNodeInfos) {
            scriptCode.Append($"{CODE_INDENTATION_STR}{CODE_INDENTATION_STR}{CODE_INDENTATION_STR}{v.paramAssignment}\n");
        }

        //TODO： 针对“按钮”组件添加监听


        scriptCode.Append($"{CODE_INDENTATION_STR}{CODE_INDENTATION_STR}}}\n");


        //TODO: 按钮点击监听的方法






        //脚本末尾
        scriptCode.Append($"{CODE_INDENTATION_STR}}}\n");
        scriptCode.Append("}");

        //保存该脚本文件
        var scriptPath = $"{Application.dataPath}/Scripts/Generate/UI/{directoryName}/{className}_Bind.cs";
        //预先检查目标文件夹是否存在(若没有，则创建)
        var dirName = Path.GetDirectoryName(scriptPath);
        if(string.IsNullOrEmpty(dirName)) return;
        if (!Directory.Exists(dirName)) {
            Directory.CreateDirectory(dirName);
        }
        //若该文件已存在，则先删除旧文件
        if (File.Exists(scriptPath)) {
            File.Delete(scriptPath);
        }
        File.WriteAllText(scriptPath, scriptCode.ToString());
        scriptCode.Clear();
        Debug.Log($"Generate {className}_Bind success.");
    }

    //生成该prefab的脚本文件B，用于该UI界面的实际操作逻辑
    //PS：为避免影响“界面功能逻辑”，若同路径下已有该文件，则该方法不会执行
    private static void GenerateMainScript(string className, string baseClassName, string directoryName) {
        //若已有该文件，则返回
        var scriptPath = $"{Application.dataPath}/Scripts/UI/{directoryName}/{className}.cs";
        if(File.Exists(scriptPath)) return;

        var scriptCode = new StringBuilder();
        //添加常用的namespace
        //scriptCode.Append("using XFrameworkBase;\n");
        scriptCode.Append("using System.Collections;\n");
        scriptCode.Append("using System.Collections.Generic;\n");
        scriptCode.Append("\n");

        //代码主体
        scriptCode.Append($"namespace {NAMESPACE_STR}\n");
        scriptCode.Append("{\n");
        //"baseClassName"放在mainScript的好处：当功能逻辑中不需要某个UI界面时，其mainScript会被删除，
        //此时由于bindScript会用到baseClassName中的某些方法，则其会自动报错，如此即可马上找到该bindScript并删除，减少冗余脚本残留
        scriptCode.Append($"{CODE_INDENTATION_STR}internal partial class {className} : {baseClassName}\n");
        scriptCode.Append($"{CODE_INDENTATION_STR}{{\n");

        //TODO：按钮监听方法实现







        //脚本末尾
        scriptCode.Append($"{CODE_INDENTATION_STR}}}\n");
        scriptCode.Append("}");

        //保存该文件
        var dirName = Path.GetDirectoryName(scriptPath);
        if(string.IsNullOrEmpty(dirName)) return;
        if (!Directory.Exists(dirName)) {
            Directory.CreateDirectory(dirName);
        }
        File.WriteAllText(scriptPath, scriptCode.ToString());
        scriptCode.Clear();
        Debug.Log($"Generate {className} success");
    }
    // ====================== END: 为Prefab生成脚本文件 =======================


    // =================== START: 提取prefab下所有“特殊命名前缀”节点的数据 =================
    //遍历prefab下所有节点，统计“特殊命名前缀”的节点的数据信息，以方便后续“脚本文件”生成
    //注意：“path”为包含该transform的“所有父节点和自身name”的完整路径
    public static void GeneratorDataForAllNodes(Transform transform, string ownPath)
    {
        //若“ownPath”为空或空字符串，则不处理本节点，如该prefab根节点
        if (!string.IsNullOrEmpty(ownPath)) {
            GenerateDataForSingleNode(ownPath);
        }

        //检测子节点
        var childCnt = transform.childCount;
        if (childCnt <= 0) return;

        for (int i = 0; i < childCnt; i++)
        {
            var childTrans = transform.GetChild(i);
            var childPath = string.IsNullOrEmpty(ownPath) ? $"{childTrans.name}" : $"{ownPath}/{childTrans.name}";
            GeneratorDataForAllNodes(childTrans, childPath);
        }
    }

    //检测单个节点并生成对应的代码串
    private static void GenerateDataForSingleNode(string path) {
        //截取路径最后一段，即该节点的名字(默认以“/”分隔路径)
        var lastIndex = path.LastIndexOf("/", StringComparison.Ordinal);
        var objName = path.Substring(lastIndex + 1);

        //检测是否有特殊前缀
        var specialPrefix = "";
        foreach (var kv in allSpecialNamePrefix) {
            if (objName.StartsWith(kv.Key)) {
                specialPrefix = kv.Key;
                break;
            }
        }
        if(string.IsNullOrEmpty(specialPrefix)) return;
        if(string.IsNullOrEmpty(allSpecialNamePrefix[specialPrefix])) return;

        //为该节点生成对应的“参数声明”和“赋值语句”字符串
        var paramName = objName.FirstCharacterToLower();                //参数名称(首字母小写)
        //该节点的“参数声明”字符串
        var paramHead = new StringBuilder();
        paramHead.Append("private ");            //由于该组件参数大多情况仅在UI内部使用，因此使用“private”修饰
        paramHead.Append($"{allSpecialNamePrefix[specialPrefix]} ");
        paramHead.Append(paramName);
        paramHead.Append(";");
        //该参数的“赋值语句”字符串
        var paramContent = new StringBuilder();
        paramContent.Append(paramName);
        paramContent.Append(" = ");
        var tempContent = "";
        switch (specialPrefix) {
            //一般
            case "m_text":
            case "m_img":
            case "m_tf":
            case "m_rect":
            case "m_vlay":
            case "m_hlay":
                tempContent = $"FindChildComponent<{allSpecialNamePrefix[specialPrefix]}>(\"{path}\")";
                break;

            //特殊


        }
        //若没有赋值语句，则清空StringBuilder对象
        if (string.IsNullOrEmpty(tempContent)) {
            paramHead.Clear();
            paramContent.Clear();
            return;
        }
        paramContent.Append(tempContent);
        paramContent.Append(";");

        //生成该节点的数据，并添加到列表中
        NodeInfo node;
        node.paramDeclaration = paramHead.ToString();
        node.paramAssignment = paramContent.ToString();
        allSpecialNodeInfos.Add(node);
        //清空StringBuilder对象
        paramHead.Clear();
        paramContent.Clear();
    }
    // ================== END: 提取prefab下所有“特殊命名前缀”节点的数据 =================
}
```

2.项目框架中，针对UI需要==预先提供“UIBase、UIPanel、UIWindow”等类型==，同时提供**工具类“UIUtility”**

**UIBase.cs**：

```c#
using System.Collections;
using System.Collections.Generic;
using Unity.VisualScripting;
using UnityEngine;

namespace XGame1
{
    internal class UIBase
    {
        private Transform m_transform;

        public Transform Transform {
            get {
                if (null == m_transform || m_transform.IsDestroyed()) return null;

                return m_transform;
            }
        }

        //仅在各个UI界面的“工具生成脚本xx_Bind.cs”中重写
        //作用：绑定prefab下各个“特殊命名前缀”节点的组件
        protected virtual void AutoBindComponents(){}


        // ======================= START: 工具方法 =======================
        protected T FindChildComponent<T>(string path) where T : Component {
            if (null == m_transform || m_transform.IsDestroyed()) return null;

            var result = m_transform.FindChildComponent<T>(path);
            return result;
        }
        // ======================= END: 工具方法 =======================
    }
}
```

**UIWindow.cs** 和 **UIPanel.cs**：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace XGame1
{
    internal class UIWindow : UIBase
    {

    }
}
```

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace XGame1
{
    internal class UIPanel : UIBase
    {

    }
}
```

UI工具类 **UIUtility.cs**：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public static class UIUtility
{
    //获取指定路径path下的目标组件T
    public static T FindChildComponent<T>(this Transform transform, string path) where T : Component {
        var trans = transform.Find(path);
        if (null == trans) return null;
        var comp = trans.GetComponent<T>();
        return comp;
    }
    
}
```



#### 使用方式：

1.在Project中选中某个prefab资源，其内部结构如下：
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20241107114048345.png" alt="image-20241107114048345" style="zoom:80%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20241107114119586.png" alt="image-20241107114119586" style="zoom:80%;" />

2.选中该prefab后鼠标右键选择该“Window或Panel”等
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20241107114227255.png" alt="image-20241107114227255" style="zoom:80%;" />

生成效果如下：
UIWindowWarGame_Bind.cs：

```c#
//Auto generate by tools. Don't modify manually.

using UnityEngine;
using UnityEngine.UI;

namespace XGame1
{
    internal partial class UIWindowWarGame
    {
        private Image m_imgBg;
        private Text m_textNum;
        private HorizontalLayoutGroup m_hlay;

        protected override void AutoBindComponents()
        {
            m_imgBg = FindChildComponent<Image>("m_hlay/m_imgBg");
            m_textNum = FindChildComponent<Text>("m_hlay/m_textNum");
            m_hlay = FindChildComponent<HorizontalLayoutGroup>("m_hlay");
        }
    }
}
```

UIWindowWarGame.cs：

```c#
using System.Collections;
using System.Collections.Generic;

namespace XGame1
{
    internal partial class UIWindowWarGame : UIWindow
    {
    }
}
```

















