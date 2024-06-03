#### 问题：

1.从PreparingState到下一个state：
ExitPreparingStateSystem，该system继承自“LogicReactiveSystem”，其中的“OnExecute”会一直执行，并不需要手动的驱动
这里没看懂，要再仔细看下
从FightingState完毕后到“LauchArenaSystem”,也是通过继承自“LogicReactiveSystem”的LaunchArenaSystem来监听，
并且在“OnExcute”中也是通过“entities.SingleEntity()”来判断是否进入下一个阶段
这个“entities.SingleEntity()”到底是如何发挥作用的啊？？？？
***继承“LogicReactiveSystem”的system会按照顺序依次执行，并非一直在检测执行



2.**AreaComponent**中并没有“isArea”，这个生成的逻辑是怎样的？
例子一：
    [LogicArena]
    public class ArenaComponent : LogicComponent
    {
    }

例子二：EnableAICountDownFinishedComponent
    [LogicArena]
    public class EnableAICountDownFinishedComponent : LogicComponent
    {

​    }        

在生成时会自动的生成bool类型的
    static readonly Battle.Logic.Arena.Component.ArenaComponent arenaComponent = new Battle.Logic.Arena.Component.ArenaComponent();

    public bool isArena {
        get { return HasComponent(LogicArenaComponentsLookup.Arena); }
        set {
            if (value != isArena) {
                var index = LogicArenaComponentsLookup.Arena;
                if (value) {
                    var componentPool = GetComponentPool(index);
                    var component = componentPool.Count > 0
                            ? componentPool.Pop()
                            : arenaComponent;
    
                    AddComponent(index, component);
                } else {
                    RemoveComponent(index);
                }
            }
        }
    }
对于内部没有参数，但继承自“LogicComponent”的component，在生成时自动按照bool类型来生成，并且去掉“component”?
这个生成的逻辑要再测试一下



3.“**GetSingleEntity**”: 获取集合中唯一的Entity，当集合该目标matcher的entity超出1时则报错
那么在使用之前其实已经很明确的知道该component是否是唯一的。即该component对应的属性从一开始是否是唯一的
MatchOne的例子中对于这个“GetSingleEntity”的component添加了前缀“Unique”。但战斗项目中没有
Entitas.Matcher匹配系统详细是怎样的？这个很重要
实际调用在：FightingState.SendBattleStartMessage -> BattleStateUtils.GetMonsterProgress
“var progressEntity = contexts.logicBattleState.GetGroup(MonsterProgressMatcher).GetSingleEntity()”
这里用到了Matcher以及“GetSingleEntity”



4.Component中某个参数前加“[**PrimaryEntityIndex**]”标记的作用是什么？
例子一：
public class ArenaTeamComponent : LogicComponent
{
	[PrimaryEntityIndex]
	public CampFlag Camp; // 队伍阵营

	public List<ulong> Members = new List<ulong>(); // 队伍成员ID列表
}
“[PrimaryEntityIndex]”应该是该实体所有component中首要的，可以作为Entity的index的参数



5.BattleEventDef.OnArenaEnterTurn = class.get("Battle.Com.Context.Message.Arena.ArenaEnterTurnMessage").MessageId;
这里通过“class.get("Battle.Com.Context.Message.Arena.ArenaEnterTurnMessage")”得到的“MessageId”应该是恒定不变的吗？
如果是变化的，那么“BattleModule”中的监听是如何起效的呢？
BattleModule:OnBattleStart{
.....
dispatcher:AddEventListener(Slot(self, self.OnArenaEnterTurn), BattleEventDef.OnArenaEnterTurn)
.....
}

lua中的“Broadcast”机制是如何起效的，这个还要再看啊

从战斗层发送message，ArenaEnterTurnMessage，这样的消息到lua层，lua层监听，这个是怎么起效的？？









注意：

1.BattleContext等context类型的赋值：

![image-20220706210938944](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220706210938944.png)















