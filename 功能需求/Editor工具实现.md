[TOC]





#### 1.查找指定路径下的对象，并将满足条件的对象保存到Json或Excel文件或在prefab中添加目标组件

##### 查找指定路径下的目标对象

```c#
private static string Prefab_Dir = "Assets/Resources/UI";
.........................
var prefabGuids = AssetDatabase.FindAssets("t:Prefab", new[] { Prefab_Dir });
foreach (var prefabGuid in prefabGuids)
{
    var path = AssetDatabase.GUIDToAssetPath(prefabGuid);
    var prefab = AssetDatabase.LoadMainAssetAtPath(path) as GameObject;
    if (null == prefab) continue;

    var textComs = prefab.GetComponentsInChildren<Text>();
    foreach (var v in textComs)
    {
        ........
    }
}
```

##### 将满足条件的对象保存到Json或Excel文件中

###### 序列化需要保存的目标对象的相关信息

JsonUtility序列化只支持public修饰的Class或字段(带有“[Serializable]”或“[SerializeField]”标记)，无法序列化属性，无法直接序列化“`List<T>`” —— 针对“`List<T>`”类型，需要另外封装一个“public class”(带有"[Serializable]"标记)，其内部包含public修饰的“`List<T>`”类型参数，如：

```c#
[Serializable]
public class LocalizationFixedTextInfo
{
    public int id;                                  //对应“通用-》文本配置表 -》界面文本”编号
    public string text;                             //该Text组件上的文本内容
    public string prefabName;                       //所属的prefab名字
    public string prefabPath;                       //prefab的路径
    public string textPath;                         //该Text组件在prefab内部的路径
    public string desc;                             //说明描述
}

//JsonUtility无法直接序列化“List<T>”类型，因此这里增加新的类型对其进行封装
[Serializable]
public class LocalizationTextList
{
    public List<LocalizationFixedTextInfo> textList;
}
```

###### 将数据保存到Json文件或Excel中

- 查找目标信息：

```c#
var textComs = prefab.GetComponentsInChildren<Text>();
foreach (var v in textComs)
{
    ....................................
    //记录下当前需要的文本信息
    var fixedTextInfo = new LocalizationFixedTextInfo() {
        id = UniqueId++,
        text = v.text,
        prefabName = prefab.name,
        prefabPath = path,
        textPath = GetFullPath(v.gameObject),
        desc = "多语言界面文本提取(10800600-10804000)"
    };
    allFixedTextList.Add(fixedTextInfo);
}

.........................
//获取子节点在prefab下的路径
private static string GetFullPath(GameObject obj)
{
    if (null == obj.transform.parent) return "";

    var parentFullPath = GetFullPath(obj.transform.parent.gameObject);
    return parentFullPath == "" ? obj.name : (parentFullPath + "/" + obj.name);
}
```

- **保存到Json文件**：

```c#
private static void SaveToJson()
{
    var infoList = new LocalizationTextList() { textList = allFixedTextList };
    var jsonStr = JsonUtility.ToJson(infoList, true);
    File.WriteAllText(JsonFile_SavePath, jsonStr);
}
```

- **保存到Excel文件**：

该功能需要“EPPlus.dll”，先导入项目中，并放置在"Editor文件夹"下的“Plugins”子文件夹中：
<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20241016101151095.png" alt="image-20241016101151095" style="zoom:80%;" />

```c#
private static void SaveToExcel()
{
    using var package = new ExcelPackage();
    var workSheet = package.Workbook.Worksheets.Add("文本信息");
    //表头信息
    workSheet.Cells[1, 1].Value = "编号";
    workSheet.Cells[1, 2].Value = "文本内容";
    workSheet.Cells[1, 3].Value = "所属prefab";
    workSheet.Cells[1, 4].Value = "prefab的路径";
    workSheet.Cells[1, 5].Value = "text组件在prefab中的路径";
    workSheet.Cells[1, 6].Value = "描述说明";
    //设置表头中“从1开始，一共6列”的Style
    using var range = workSheet.Cells[1, 1, 1, 6];
    range.Style.Font.Bold = true;
    range.Style.Fill.PatternType = ExcelFillStyle.Solid;
    range.Style.Fill.BackgroundColor.SetColor(Color.Gray);

    //向Excel中添加数据
    for (int i = 0; i < allFixedTextList.Count; ++i)
    {
        var data = allFixedTextList[i];
        workSheet.Cells[i + 2, 1].Value = data.id;
        workSheet.Cells[i + 2, 2].Value = data.text;
        workSheet.Cells[i + 2, 3].Value = data.prefabName;
        workSheet.Cells[i + 2, 4].Value = data.prefabPath;
        workSheet.Cells[i + 2, 5].Value = data.textPath;
        workSheet.Cells[i + 2, 6].Value = data.desc;
    }

    //设置列宽
    workSheet.Column(1).AutoFit(10);
    workSheet.Column(2).AutoFit(10, 30);
    workSheet.Column(3).AutoFit(10);
    workSheet.Column(4).AutoFit(20, 70);
    workSheet.Column(5).AutoFit(20, 70);
    workSheet.Column(6).AutoFit(20);

    //保存文件
    var fileInfo = new FileInfo(ExcelFile_SavePath);
    package.SaveAs(fileInfo);
    Debug.Log("Localization Text excel file finished");
}
```

##### 为目标GameObject添加指定组件并保存到prefab中

修改prefab后，若需要保存这些修改，则必须执行"==AssetDatabase.SaveAssets()=="语句。否则这些修改不会生效

```c#
AssetDatabase.SaveAssets();
AssetDatabase.Refresh();
```

实现代码如下：

```c#
public static void AddUITextBinder()
{
    if(!File.Exists(JsonFile_SavePath)) return;

    //反序列化json数据
    var textData = File.ReadAllText(JsonFile_SavePath);
    var jsonData = JsonUtility.FromJson<LocalizationTextList>(textData);
    var fixedTextList = jsonData.textList;

    //依次为各个Text添加“UITextBinder组件”
    foreach (var v in fixedTextList)
    {
        var prefab = AssetDatabase.LoadMainAssetAtPath(v.prefabPath) as GameObject;
        if(null == prefab) continue;
        var textTrans = prefab.transform.Find(v.textPath);
        if(null == textTrans) continue;
        var textCom = textTrans.GetComponent<Text>();
        if(null == textCom) continue;

        if (textCom.text == v.text)
        {
            var textBinderCom = textTrans.GetComponent<UITextIDBinder>();
            if (null == textBinderCom)
            {
                textBinderCom = textTrans.gameObject.AddComponent<UITextIDBinder>();
                textBinderCom.m_textID = v.id;
                Debug.Log($"add textBinder: {v.prefabName}  {v.textPath}");
            }
        }
    }

    //必须包含以下语句，否则“改动”无法保存到prefab上
    AssetDatabase.SaveAssets();
    AssetDatabase.Refresh();
    Debug.Log("add TextBinder finished.");
}
```































