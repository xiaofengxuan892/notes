# 1.添加AI及stateMachine：

## 步骤一：添加状态机 — ThingFactory.CreateThing

![image-20220727192149429](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727192149429.png)

![image-20220727192457223](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727192457223.png)

## 步骤二：选择不同的Factory创建Thing

添加完状态机后，根据Thing的不同类型选择“GeneralFactory”，“MonsterFactory”, "BulletFactory"等。如果为“GeneralFactory”：

### 添加AI: 

GeneralFactory.CreateGeneral, 如果配置了AI策略，

![image-20220727191357181](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727191357181.png)

![image-20220727193529156](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727193529156.png)

![image-20220727193752996](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727193752996.png)

“BehaviorTree.cs”只是AI插件中的一个脚本



## 步骤三：根据GeneralConf/MonsterConf配置表设置初始状态

ThingFactory.SwitchInitialState:

![image-20220727200559886](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727200559886.png)









# 2.AI唤醒：

















# 3.AI执行：

## 1.获取包含“IntelligentComponent”的实体：

"AIAgentThinkSystem.cs"会始终检测当前存在的所有LogicThingEntity中包含“LogicThingMatcher.Intelligent”的：
![image-20220727202020191](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727202020191.png)

由于“IntelligentComponent”并不包含任何参数：

![image-20220727202052781](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727202052781.png)

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727202215037.png" alt="image-20220727202215037"  />

![image-20220727202256302](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727202256302.png)

所以当“actor.isIntelligent = false”时，“AIAgentThinkSystem.cs”是不会检测到包含“IntelligentComponent”的LogicThingEntity的

## 2.调用该实体中的AIAgent.Think:

![image-20220727202734786](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727202734786.png)

使用“tree.load(factory, cfg)”创建而成的BehaviorTree**依次执行该cfg文件中的所有Node节点的Tick方法**：

在查阅AIStrategy文件“AttackFirstButWillChangeTarget.json”时，发现里面有Node节点“TryAutoCast”:

![image-20220727203033496](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727203033496.png)

PS: AIStrategy的json文件中各个node节点的名字应该也是根据创建的各个C#脚本的名字来定义的

## 3.执行“TryAutoCast.cs”文件中的"Tick"方法：

![image-20220727203450879](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220727203450879.png)

## 4.关联StateMachine切换状态：

根据各个node.Tick的执行情况切换stateMachine，如“entity.Idle”，“entity.Cast”等。

如此AI和StateMachine即关联起来了。StateMachine只是作为一种工具，用来说明entity进入各个状态的一些操作，但是需要AI中的各个Node节点来驱动



