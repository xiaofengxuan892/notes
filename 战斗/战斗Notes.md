### Lua端发送消息到战斗层，战斗层检测并执行后返回结果给Lua层

#### 1.lua端发送request：

1.调用BattleRequest中“SendRequest”发送request -》通过“self:GetContainer()”调用“BattleManager”中的“SendRequest”

![image-20220704193408394](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704193408394.png)

![image-20220704193434560](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704193434560.png)

PS: 在“BattleManager”中通过“self:AddComponent("BattleRequest")”建立“BattleRequest”与“BattleManager”之间的联系

![image-20220704193527224](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704193527224.png)

“AddComponent”的作用在于：如果是在脚本A中使用“AddComponent”添加脚本"B"，那么在脚本A中可以直接调用脚本B中的方法，即相当于A是B的载体，A包含B中的所有"ExportMethod"导出的方法

但在脚本B中若要使用A中的方法则需要首先获取容器载体“self:GetContainer”



2.在“BattleManager”中调用“BattleService”中的“SendRequest”方法

![image-20220704194051681](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704194051681.png)

此时只是将该request方法logicController的序列中：

![image-20220704194134664](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704194134664.png)



#### PS: 补充说明

#### 1.放入队列后开始等待：

1.BattleRequest的“GetRespond”等待

![image-20220704195700360](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704195700360.png)

而在“BattleRespond”中加入重写协程接口“IEnumerator”—— 包含“MoveNext”方法：

![image-20220704195831432](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704195831432.png)

由于BattleRespond的构造方法中有协程一直在执行，因此构造方法始终没有执行完毕，直到“MoveNext”为false，即协程执行完毕，因此BattleService中“SendRequest”才执行完毕。

构造方法中协程执行完毕的触发条件“CommandProcessor —— 》 Succeed”：
![image-20220704200300367](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704200300367.png)

使用“Send”打断协程的执行：

![image-20220704200335319](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704200335319.png)



同时会把respond中的结果返回：

![image-20220704200659912](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704200659912.png)

如果BattleRespond中需要有多个返回参数，只需要在“Succeed”前添加respond的对应返回参数即可

在创建每个对应的“Processor”前，基类的“CommandProcessor”已经为request和respond赋值了：

![image-20220704200925958](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704200925958.png)

同时提供给子类重写的“OnProcess”方法



#### 2. 把消息压入队列的过程

![image-20220704203409967](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704203409967.png)

参数“battleLogic”本身就是战斗dll中定义的一个队列类型参数

battleLogic的声明和创建：

![image-20220704203644487](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704203644487.png)

![image-20220704203705007](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704203705007.png)

在“CreateBattleLogic”中先创建“BattleLogicContextClass"，![image-20220704203948391](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704203948391.png)

然后创建“BattleLogicControllerClass”，并将上面创建的“battleLogicContextClass”赋值给LogicController：

![image-20220704204127975](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704204127975.png)

所以“BattleService -> SendRequest”：

![image-20220704204625855](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704204625855.png)

实际上是战斗dll中的“LogicController -> EnqueRequest”：

![image-20220704204709231](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704204709231.png)



#### 3.战斗层检测Request并执行：

CommandSystem中的Execute会循环检测“ProcessCommandQueue”：

![image-20220704205803952](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704205803952.png)

“Contexts.TryDequeueRequest”会去“LogicContext”中调用“TryDequeueRequest”:
![image-20220704210042536](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704210042536.png)

其中“GetController”即为“LogicController”：

![image-20220704210113710](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704210113710.png)

如此也就直接回应到最初将request压入到“LogicController的“_requestQueue”：

![image-20220704210216301](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220704210216301.png)

如此也就实现了lua端发送消息，然后战斗层执行并返回的整体流程





### 开始战斗的流程：

1).调用“BattleManager”中的“EnterBattle”：

![image-20220705115802943](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705115802943.png)

2).通过changeState将“BattleState”以及“battleContext”都传递过来：
![image-20220705115958232](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705115958232.png)

3).将该“BattleState”放入存储所有state的队列中：

![image-20220705120059751](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705120059751.png)

4).从列表中取出当前状态并执行：

![image-20220705120143671](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705120143671.png)

![image-20220705120228905](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705120228905.png)

由此进入“SimpleStateMachine”中的“ChangeState”：

![image-20220705120336615](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705120336615.png)

![image-20220705120411368](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705120411368.png)

通过该“stateName”获取到对应的state详情脚本，如“BattleState”, "HomeState", "LoginState"等多种状态，因此这里其实已经进入了对应的战斗状态，对于战斗状态中所有相关处理都在“BattleState”中具体设置

然后执行到“BatttleState”中“OnEnter”：

![image-20220705141810509](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705141810509.png)

“BattleManager” -> "StartBattle"：

![image-20220705142003372](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705142003372.png)

![image-20220705142013868](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705142013868.png)

最终在“BattleService” -> "Start"中开启循环检测：

![image-20220705142120701](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705142120701.png)

![image-20220705142141112](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705142141112.png)

“_battleLogic”实际是“LogicController”：

![image-20220705142250501](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705142250501.png)

最终执行到“CommandSystem.Execute”:

![image-20220705142323732](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705142323732.png)

即从队列中取出request并执行



**对于ViewController 和 LogicController中都有“_messageQueue”**，LogicController中的“messageQueue”消息的读取使用“BattleService.LoopUpdate”中“DequeueAndRouteMessages”

ViewController中的“messageQueue”则使用“MessageSystem”，该system继承自“ViewExecuteSystem”





**关键：在创建“LogicController”的时候就把战斗所有相关的system都创建出来了**

![image-20220705151835565](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705151835565.png)

LogicSystem会把所有相关的system创建：

![image-20220705152312152](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705152312152.png)



### 为什么很多地方可以直接使用“Context“参数？

凡是可以直接使用”Context“参数的脚本必定继承于”LogicBaseSystem“

![image-20220705161238268](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705161238268.png)

而LogicBaseSystem的构造方法：

![image-20220705161331277](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705161331277.png)

在LogicContextBridge中包含有”Contexts“变量，并在”OnCreate“时进行了赋值：

![image-20220705161642436](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705161642436.png)

由于在对战斗初始化时是按照**”先创建LogicController“** - 》”在LogicController的OnCreate“方法中**创建“LogicContexts”变量**，并赋值给后续的“LogicSystems”：

![image-20220705161928658](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705161928658.png)

**即： 创建LogicController -> 创建LogicContexts -> 创建完LogicContexts后赋值给LogicSystems或者CommandSystem**



注意：当使用的是ViewController时，则“Contexts”指代的是“ViewContexts”， 而非“LogicContexts”



**在BattleServices创建之初就已创建了“CreateBattleLogic”， “CreateBattleView”**

![image-20220706180613599](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220706180613599.png)

在“LogicController”中就创建了对应的“LogicContext”，以及“LogicSystem”

同样“CreateBattleView”也是一样的道理：

![image-20220706180756274](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220706180756274.png)

![image-20220706180809066](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220706180809066.png)







**要查找某个对象下有哪些具体的参数可以直接全局查找“[LogicBattleState]”即可知道**

标记了“[LogicBattleState]”的可以用于最后生成

“LogicBattleStateContext”

“LogicBattleStateComponentsLookup”

"LogicBattleStateEntity"

1.在创建某个具体的component时——component的具体命名最好后面加上“component”后缀，或者一律不加后缀也可以，要统一

2.在component前添加以上标记，如“[LogicBattleState]”等，可以直接用于生成对应的三个重要文件



通过添加标记可以使得同一个component出现在很多个具体的context中，这样不用重复创建了：

![image-20220705215755505](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705215755505.png)





#### Lua中创建的BattleContext还需要再转换一次变成战斗dll中使用的BattleContext：

![image-20220705194147312](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705194147312.png)

![image-20220705194232880](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705194232880.png)





#### 战斗dll和lua中使用的EventDispatcher是不一样的：

战斗中使用的EventDispatcher.cs是对UGUI中的”Component，IEventDispatcher“的进一步实现和封装，UGUI本身并没有”EventDispacher“，但提供了”IEventDispatcher“。所以这里对其实现即可

lua中使用的EventDispatcher.lua，则是不使用UGUI中的东西，单独使用table将所有的事件都存储起来，然后进行监听。—— 本质上和战斗层的”EventDispatcherlua“原理一样。

**仔细查看C#中实现的”EventDispacher.cs“后发现，牛逼啊，这个脚本应该很有用**











#### 没有看懂的地方：

**1、还可以自己定义一些在战斗dll中没有的数据集合类型：**

如CardList， SkillDataList：

![image-20220705205339159](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705205339159.png)

![image-20220705205348735](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705205348735.png)









#### 后面要再总结下具体内容：

1.使用this关键字为某些类型添加扩展方法：

![image-20220705175538911](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705175538911.png)

这个还是很实用的：https://blog.csdn.net/niaxiapia/article/details/103104304

2.“is”运算符的格式：

![image-20220705191519208](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705191519208.png)

与通常的“a is B”不同，后面还加了一个具体的B类型变量，

**“is”运算符格式：**

如果是“a is B”，那么只是判断a是否与B类型兼容，结果为bool

如果是“a is B b”，那么当a与B类型兼容时，结果返回true，并且"b = a"；如果不兼容，则直接返回false

![image-20220705192012568](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705192012568.png)