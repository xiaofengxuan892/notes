[TOC]





### UnityWebRequest的作用：

#### 1.下载服务器文件：

##### 1).开始下载：

**<font color=red>下载“远程指定Url文件”的所有数据</font>**：

```c#
UnityWebRequest m_UnityWebRequest = new UnityWebRequest(downloadUrl);
yield return m_UnityWebRequest.SendWebRequest();
```

**<font color=red>下载“远程指定Url文件”部分区间的数据</font>**：

UnityWebRequest可通过`SetRequestHeader("Range", "bytes={区间起始位置}-{区间结束位置}")`来设置**<font color=red>只下载限定区间内的数据</font>**：

**(1).区间范围：从“起始位置”到“<font color=red>文件末尾</font>”**

```c#
UnityWebRequest m_UnityWebRequest = new UnityWebRequest(downloadUrl);
m_UnityWebRequest.SetRequestHeader("Range", string.format("bytes={0}-", startPos));
yield return m_UnityWebRequest.SendWebRequest();
```

**(2).区间范围：从“起始位置”到“<font color=red>结束位置</font>”**

```c#
UnityWebRequest m_UnityWebRequest = new UnityWebRequest(downloadUrl);
m_UnityWebRequest.SetRequestHeader("Range", string.format("bytes={0}-{1}", startPos, endPos));
yield return m_UnityWebRequest.SendWebRequest();
```

例子：下载指定Url的文件并保存在本地

```c#
IEnumerator DownloadAudioClip() {
	string downloadUri = "http://192.168.0.102/testab";
	UnityWebRequest m_UnityWebRequest = new UnityWebRequest(downloadUri);
    //在下载中该语句必要要有，否则无法获取下载到的数据；
    //对于使用“UnityWebRequest.Get/Post”时则不需要设置
	m_UnityWebRequest.downloadHandler = new DownloadHandlerBuffer();
	yield return m_UnityWebRequest.SendWebRequest();

	byte[] data = m_UnityWebRequest.downloadHandler.data;
	FileStream fs = new FileStream(targetPath, FileMode.Create, FileAccess.ReadWrite);
	fs.Write(data, 0, data.Length);
	fs.Close();
}
```



##### 2).监听下载过程中从服务器返回的bytes数据：

**基础环境与需求**：

1.任何情况下从服务器下载文件时，会将该文件转换成bytes数据在网络上传输。但是根据当前的网络带宽，会将该文件的数据分成多个部分，一次只传输部分bytes数据。并不是将该文件的所有bytes数据一次性放到网络上传输

2.为了提升下载效率，及时存储已经从服务器接收到的数据，避免重复下载相同的数据(支持“断点下载”)，需要在下载过程中监控“每次从服务器传输来的bytes数据”

**解决方案**：

**实现原理**：

**UnityWebRequest支持自主设置其“downloadHandler参数”**(**<font color=red>该参数负责处理“服务器返回的数据”</font>**)。因此可创建自定义脚本，**继承“UnityEngine.Networking”命名空间中的“DownloadHandlerScript.cs”**，并**<font color=red>重写其“ReceiveData”方法</font>**即可

**实现过程**：

1.**<font color=red>创建“新类型CustomDownloadHandler.cs”，继承自“DownloadHandlerScript”</font>**：

```c#
public sealed class CustomDownloadHandler : DownloadHandlerScript
{
    public CustomDownloadHandler(byte[] m_CachedBytes){
        base.DownloadHandlerScript(m_CachedBytes);
    }
    
    protected override bool ReceiveData(byte[] data, int dataLength)
    {
        if (dataLength > 0)
        {
            //将接收到的数据使用事件传递给各个业务逻辑
            //注意：不要在这里直接处理bytes数据，如“fileStream.Write”等。这个交由“各个业务逻辑”自由处理。因此这里使用"观察者事件"将这些数据传递出去即可
            ........
        }

        return base.ReceiveData(data, dataLength);
    }
}
```

**注意**：

1).“ReceiveData”方法**需要返回值**，因此**末尾直接调用“基类同名方法”即可**

2).“dataLength”代表服务器返回的bytes数据的大小(似乎有些多余，但可能有便于校验的目的)

3).在下载“大容量”文件，通常无法一次性将文件的所有数据传输过来，因此可通过本方法逐步接收“指定Url的部分数据”，然后将这些数据使用FileStream写入本地文件中。在多次重复该过程后，则可将“整个大容量文件”全部下载到本地。此即为Unity中下载“远程AB文件或资源包”的实际过程



2.**在创建“UnityWebRequest”变量时**，**<font color=red>设置该变量用到的“DownloadHandler”</font>**：

```c#
UnityWebRequest m_UnityWebRequest = new UnityWebRequest(downloadUrl);
m_UnityWebRequest.downloadHandler = new CustomDownloadHandler();
//如果需要下载“指定区间”的数据，则添加“SetRequestHeader”语句即可
.......
m_UnityWebRequest.SendWebRequest();
```

如此即可实现“监听服务器返回的bytes数据”的需求



#### 2.与服务器数据交互：

##### 1).发送协议数据：

**<font color=red>仅发送协议，无需向服务器传递数据</font>**：

```c#
UnityWebRequest m_UnityWebRequest = UnityWebRequest.Get(string serverUrl);
m_UnityWebRequest.SendWebRequest();
```

**<font color=red>向服务器传递的数据为“string”类型</font>**：

```c#
UnityWebRequest m_UnityWebRequest = UnityWebRequest.Post(string serverUrl, string data);
m_UnityWebRequest.SendWebRequest();
```

**<font color=red>向服务器传递的数据为“WWWForm”类型</font>**：

```c#
UnityWebRequest m_UnityWebRequest = UnityWebRequest.Post(string serverUrl, WWWForm data);
m_UnityWebRequest.SendWebRequest();
```



##### 2).监测服务器数据返回：

**如果使用“协程”**：

直接在`m_UnityWebRequest.SendWebRequest()`语句前添加“yield”关键字即可

**如果不使用“协程”**：

需要在“Update”中监测“**<font color=red>UnityWebRequest.isDone</font>**”参数。**当其数值为“true”时，则代表“数据交互的过程结束”**



##### 3).获取“数据交互”结果：

`UnityWebRequest.result`：代表“该过程是否正常”

`UnityWebRequest.error`：代表“数据交互中出现错误，此参数包含错误的详细信息”

`UnityWebRequest.downloadHandler.data`：代表“服务器返回的bytes数据”，仅在m_UnityWebRequest.result == UnityWebRequest.Result.Success时有效。

`UnityWebRequest.downloadHandler.text`：服务器返回的bytes数据的“ UTF8 string”形式





#### 3.加载本地文件

```c#
IEnumerator LoadLocalFile(string fileUrl){
    UnityWebRequest m_UnityWebRequest UnityWebRequest.Get(fileUrl);
    yield return m_UnityWebRequest.SendWebRequest();
    
    //获取服务器返回的结果：
    if(m_UnityWebRequest.result == UnityWebRequest.Result.Success){
        var bytes = m_UnityWebRequest.downloadHandler.data;
        //通知外部，并将bytes数据传递过去
    }
    else{
        var errorMsg = m_UnityWebRequest.error;
        //通知外部，并将错误信息errorMsg传递过去
    }
    
    //释放内存
    m_UnityWebRequest.Dispose();
    m_UnityWebRequest = null;
}
```



### “UnityWebRequest.downloadHandler”变量的设置

如果使用的是“new UnityWebRequest()”，并没有使用“UnityWebRequest.Get/Post”方法，则此时需要设置该UnityWebRequest变量的“downloadHandler”参数，否则不需要设置





### UnityWebRequest变量的销毁：

对于**<font color=red>“UnityWebRequest”对象，在使用完后一定要将其销毁</font>**，否则会报错。销毁方式如下：

```c#
if (m_UnityWebRequest != null)
{
    m_UnityWebRequest.Abort();
    m_UnityWebRequest.Dispose();
    m_UnityWebRequest = null;
}
```













