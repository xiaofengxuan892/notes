[TOC]



#### 常用API：

`await UniTask.Delay(10000)`：默认以**<font color=red>毫秒</font>**为单位(**如果需要以秒为单位**，则可使用`TimeSpan.FromSeconds(10)`得到以秒为单位的TimeSpan数值)

`await UniTask.NextFrame()`：等到下一帧，**<font color=red>与协程中“yield return null”作用相同</font>**

`await UniTask.Yield()`：等到**下一次调用**，**与“yield return null/UniTask.NextFrame()”作用不同**。其仅代表“下一次调用”，因此如果当前帧后续还有调用，则会直接在当前帧执行，不会等到下一帧

`await UniTask.WaitForEndOfFrame(this)`：等到帧末尾(**<font color=red>this参数非常重要，不可省略，代表this.MonoBehavior</font>**)，与协程中“yield return new WaitForEndOfFrame()”作用相同

`await UniTask.WaitForFixedUpdate()`：等到“物理每帧更新之后”

`await UniTask.WaitUntil(() => isActive == false)`：等到参数isActive满足目标值



#### await UniTask.WhenAll/WhenAny：

**作用**：当所有任务**<font color=red>都执行完毕</font>**/**<font color=red>任意一个任务</font>**执行完毕后。

**注意**：**<font color=red>WhenAll/WhenAny方法中的各个Task任务是“并行执行”的，相互之间没有严格的先后顺序</font>**

```c#
async void CallMethod() {
    List<UniTask> taskList = new List<UniTask>();
    taskList.Add(Task01());
    taskList.Add(Task02());
    taskList.Add(Task03());
    await UniTask.WhenAll(taskList);
    Debug.Log("finish..............");
    
    //也可直接使用如下形式，避免“List<UniTask>”对象消耗
    await UniTask.WhenAll(Task01(), Task02(), Task03());
}

async UniTask Task01() {
    await UniTask.Delay(1000);
    Debug.Log("11111111");
}

async UniTask Task02() {
    await UniTask.Delay(500);
    Debug.Log("2222222222222");
}

async UniTask Task03() {
    await UniTask.Delay(100);
    Debug.Log("3333333333333333");
}
```

执行结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230801234141764.png" alt="image-20230801234141764" style="zoom:80%;" />



#### UniTask任务按先后顺序依次执行：

**主要思想**：将每个UniTask封装成新的UniTaskObject类型，并放入UniTaskQueue中并依次执行

##### 实现过程：

1.针对每个任务的新封装类型UniTaskObject：

```c#
using System;
using Cysharp.Threading.Tasks;

public partial class UniTaskQueue
{
    public enum UniTaskObjectState
    {
        Waiting,
        Running,
        Done
    }

    public class UniTaskObject
    {
        public string mTaskName;
        public UniTaskObjectState mState;
        private Func<UniTask> mUniTaskFunc;   //该任务的实际执行逻辑
    
        public UniTaskObject(string taskName, Func<UniTask> uniTask) {
            mTaskName = taskName;
            mUniTaskFunc = uniTask;
            mState = UniTaskObjectState.Waiting;
        }

        //执行该UniTask
        public async UniTask Run() {
            if(mUniTaskFunc == null) return;
            
            await mUniTaskFunc.Invoke();
        }
    
        //重置数据以放入“引用池”中
        public void Clear() {
            mTaskName = "";
            mUniTaskFunc = null;
            mState = UniTaskObjectState.Done;
        }
    }
}
```

2.将所有UniTaskObject放入任务队列UniTaskQueue中：

```c#
using System;
using System.Collections.Generic;
using Cysharp.Threading.Tasks;

public partial class UniTaskQueue
{
    private string mTaskQueueName;    
    //由于队列中任务大多按照顺序依次执行，且“查找指定任务”的需求较少，因此或许可以使用“GameFrameworkListNode”结构来存储
    //PS: 不建议使用“字典结构”增加内存占用
    private List<UniTaskObject> mTaskQueue;
    
    public UniTaskQueue(string taskQueueName) {
        mTaskQueueName = taskQueueName;
        mTaskQueue = new List<UniTaskObject>();
    }
    
    //向队列中添加任务(默认在列表末尾添加新的任务)
    public void AddTask(string taskName, Func<UniTask> taskFunc) {
        UniTaskObject taskObject = new UniTaskObject(taskName, taskFunc);
        mTaskQueue.Add(taskObject);
    }
    
    //按顺序依次执行该队列中的任务
    public async UniTask RunTaskQueue() {
        foreach (var taskObject in mTaskQueue) {
            if (taskObject.mState == UniTaskObjectState.Waiting) {
                taskObject.mState = UniTaskObjectState.Running;
                await taskObject.Run();
                taskObject.mState = UniTaskObjectState.Done;
            }
        }
    }

    //专用于“引用池系统”回收任务队列
    public void Clear() {
        mTaskQueueName = "";
        foreach (var taskObject in mTaskQueue) {
            //TODO: 使用引用池系统回收每个UniTaskObject
            
        }
        mTaskQueue.Clear();
        mTaskQueue = null;
    }
    
    //在指定位置插入任务
    public void InsertTaskByIndex(int index, string taskName, Func<UniTask> taskFunc) {
        index = index < 0 ? 0 : index;
        index = index >= mTaskQueue.Count ? mTaskQueue.Count : index;
        UniTaskObject taskObject = new UniTaskObject(taskName, taskFunc);
        mTaskQueue.Insert(index, taskObject);
    }
    
    //删除指定name的任务（默认添加任务时不允许重名）
    public void DeleteTaskByName(string taskName) {
        int targetIndex = FindTaskByName(taskName);
        if(targetIndex == -1) return;
        mTaskQueue.RemoveAt(targetIndex);
    }
    
    //查找指定name的任务并返回其编号index，默认返回“-1”
    private int FindTaskByName(string taskName) {
        int index = mTaskQueue.FindIndex(m => m.mTaskName == taskName);
        return index;
    }
}
```

3.调用和验证：

各个UniTask任务：

```c#
private async UniTask HandleMethod1() {
    Debug.LogFormat("method1 start............");
    await UniTask.Delay(TimeSpan.FromSeconds(2));
    Debug.LogFormat("11111111111 ");
}

public async UniTask HandleMethod2() {
    Debug.LogFormat("method2 start........");
    await UniTask.Delay(TimeSpan.FromSeconds(10));
    Debug.LogFormat("2222222222");
}

public async UniTask HandleMethod3() {
    Debug.LogFormat("method3 start.......");
    await UniTask.Delay(TimeSpan.FromSeconds(2));
    Debug.LogFormat("333333333333");
}
```

使用任务队列UniTaskQueue依次执行每个UniTask：

```c#
public async UniTaskVoid HandleTasks() {
    UniTaskQueue taskQueue = new UniTaskQueue("Queue1");
    taskQueue.AddTask("task1", HandleMethod1);
    taskQueue.AddTask("task2", HandleMethod2);
    taskQueue.AddTask("task3", HandleMethod3);
    await taskQueue.RunTaskQueue();

    //队列中所有任务执行完毕后，可使用“引用池系统”回收该任务队列

}
```

调用该方法：`HandleTasks().Forget()`。执行结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230925200703460.png" alt="image-20230925200703460" style="zoom:80%;" />



#### 自定义UniTask任务监测：

当需要自定义某个UniTask的监测时，通常使用UniTaskCompletionSource来实现

```c#
var utcs = new UniTaskCompletionSource();
.........................
await utcs.Task;     //一直监测该utcs任务的完成
```

**注意**：UniTaskCompletionSource类型的**实例中包含pubic参数Task**(**<font color=red>其为UniTask类型</font>**)，`await utcs.Task`代表需要等待该UniTask执行完毕为止。控制UniTask任务完成的时机则由"utcs.TrySetResult()"来决定，因此具备充分的自由度，如：

```c#
var utcs = new UniTaskCompletionSource();
GuideManager.Instance.Completed += () => { utcs.TrySetResult(); };
GuideManager.Instance.StartGuide(GuideType.Arms);
await utcs.Task;
```

##### 当需要提前取消或完成某个UniTask任务时：

**注意**：任何需要提前取消某个UniTask的监测，**<font color=red>不要使用utcs.TrySetCanceled() 或 WithCancellation(xx.Token)</font>**，**因为其会导致一直用于监听utcs的“await utcs.Task”报错**。因此**<font color=red>当需要提前结束某个UniTask时，一律使用utcs.TrySetResult()，以避免Exception报错</font>**

**例如**：在UI点击时可能导致之前的某个utcs的监听结束，此时可直接使用utcs.TrySetResult()

##### 将UniTaskCompletionSource变量作为参数进行传递，由外部控制该UniTask的结束：

```c#
var utcs = new UniTaskCompletionSource();
m_EventComponent.Fire(this, DownloadSuccessEventArgs.Create(xx, utcs));
await utcs.Task;
```

将utcs作为参数传递给外部，同时在这里使用`await utcs.Task`的结束：只有在外部“事件处理函数”中执行了“`utcs.TrySetResults()`”时，才能反馈给这里的`await utcs.Task`



#### 获取UniTask任务执行的进度：

针对部分Unity中的异步操作，如UnityWebRequest、Resource.LoadAsync、SceneManager.LoadSceneAsync等，既可使用“await”进行监测，也可同时使用“ToUniTask”方法将其转换成UniTask并监测“异步操作的进度”：

```c#
var progressFunc = Progress.Create<float>(x => Debug.LogFormat("x: {0}", x));
await UnityWebRequest.Get("https://www.baidu.com").SendWebRequest().ToUniTask(progress: progressFunc);
```

输出结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230918200148305.png" alt="image-20230918200148305" style="zoom:80%;" />

**注意**：1.这里使用的“Progress”**<font color=red>必须为“Cysharp.Threading.Tasks.Progress”，而非“System.Progress”</font>**，否则每次进度日志输出时都会导致GC

2.Unity中大多数asyncOperation均可通过“ToUniTask”将其转换成UniTask，同时增加“进度Progress监测”。但针对**“SceneManager.LoadSceneAsync”通常不要使用“ToUniTask”方法**(据说会导致执行顺序问题)



#### async void, async UniTaskVoid, async UniTask的区别：

`async void`是C#任务系统中的原生方法，**不能在UniTask中直接使用**；

`async UniTaskVoid`与`async void`作用相同，但**其为UniTask中专用的方法**。

`async UniTask`与`async UniTaskVoid`同属于UniTask中的方法，后者是根据某些特殊情况简化出来的特殊版本，**相当于async UniTask下的一个特殊分支**

**注意**：任何UniTask(**包括async UniTaskVoid**)，当**需要等待**该UniTask时则使用**await**关键字；如果**不需要等待**，则必须在**<font color=red>该UniTask末尾添加Forget()</font>**，否则会报异常；

##### 使用上的区别：

###### 1.当其作为普通的方法被调用时

```c#
public void Method1(){}
public async void Method2(){ await UniTask.Delay(TimeSpan.FromSeconds(2));}
public async UniTaskVoid Method3(){ await UniTask.Delay(TimeSpan.FromSeconds(2));}
public async UniTask Method4(){ await UniTask.Delay(TimeSpan.FromSeconds(2));}

//调用以上方法
Method1();
Method2();
Method3().Forget();
Method4().Forget() 或 await Method4();
```

如上所示，当直接调用该方法时，其与普通方法的调用一样，没有区别(UniTask类型的方法需要增加await或Forget)

###### 2.作为参数传递到其他方法中

由于UniTask(包括UniTaskVoid)平常使用上和普通方法一致，因此若作为参数传递，则同样可借助“委托”来实现：

**情况一**：Func委托

该情况较为简单，直接使用`Func<UniTask>`和`Func<UniTaskVoid>`，然后和普通方法一样赋值即可。如：

```c#
public async UniTask Method1(){
    Debug.LogFormat("start.......");
    await UniTask.Delay(TimeSpan.FromSeconds(2f));
    Debug.LogFormat("over.......");
}
public void Method2(Func<UniTask> taskFunc){}  

Method2(Method1);  //调用
```

如上直接执行Method2(Method1)即可

**PS**：当需要执行该UniTask时，则直接调用`taskFunc.Invoke()`即可。同时由于该Func委托的返回类型为UniTask，因此也可使用`await taskFunc.Invoke()`

**易错点**：**<font color=red>绝对不能直接在方法中声明UniTask或UniTaskVoid类型的参数</font>**，如

```c#
public void Method2(UniTask param){}    //错误的声明形式
Method2(Method1());  //错误，UniTask会马上执行
Method2(Method1);  //编译直接报错
```

该方式会导致“传入Method1()”参数的同时马上执行该UniTask，即“Debug.LogFormat("start.......")”语句会马上输出



**情况二**：Action委托

当参数类型为Action时，则只接受async UniTaskVoid类型的UniTask

```c#
public async UniTaskVoid Method3() {
    Debug.LogFormat("开始执行");
    await UniTask.Delay(TimeSpan.FromSeconds(2));
    Debug.LogFormat("执行完毕");
}

public void Method4(Action ac) {
    Debug.LogFormat("赋值完成");
    ac?.Invoke();
}

Method4(Method3);  //编译报错，无法直接使用UniTaskVoid类型参数为Action委托赋值
Method4(UniTask.Action(Method3));  //正确
```

如上所示，**<font color=red>Action类型参数只接受"void"方法，不能直接将UniTaskVoid方法为其赋值</font>**。必须**使用UniTask.Action将UniTaskVoid转换后才能赋值**，如`Method4(UniTask.Action(Method3))`。输出结果：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230925192720774.png" alt="image-20230925192720774" style="zoom: 80%;" />

**注意**：在UGUI中由于通常使用**UnityAction类型**(**从Action封装而来**)，因此需要使用**<font color=red>UniTask.UnityAction来转换UniTaskVoid</font>**，如`this.GetComponent<Button>.onClick.AddListener(UniTask.UnityAction(Method1))`



#### 语法规范上的混淆点：

1.针对UniTask方法，其是可以直接使用return进行返回的，如：

```c#
public UniTask Method(Func<UniTask> taskFunc){
    if(taskFunc == null) return;     //这样写是没问题的，已验证过
    await taskFunc.Invoke();
}
```

如果为`UniTask<T>`，则使用`return T`即可。和普通方法的使用一样







