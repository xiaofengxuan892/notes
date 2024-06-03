### **ReactiveSystem**：当group中对象有变化时会执行

触发时机：

原理：通过“GetTrigger”方法和“IMatcher<T>”参数来设定其触发时机

实现过程：

通常在该脚本内声明“IMatcher<T>”类型参数，并重写“GetTrigger”方法，只有在指定`IMatcher<T>`的组件发生变化时才会触发其“OnExecute”方法

“IMatcher.Added”：代表添加“IMatcher<T>”类型的组件或该类型的组件内部数据发生变化时，则会触发“ReactiveSystem.OnExecute”方法，此时传递过来的参数“List<T> entities”只包含发生“添加了新组件或组件数据发生变化”的实体。对于没有改变的实体则不会包含的“OnExecute(List<T> entities)”中的“entities”集合中

“IMatcher.Changed”：只有在该组件数据发生变化时才会触发“ReactiveSystem.OnExecute”方法。添加该类型组件不会触发

“IMatcher.Removed”：移除该类型组件时触发

“IMatcher”：其后没有指定具体的规则，则相当于“IMatcher.Any”，指“添加组件，组件数据发生改变，删除组件”均会触发“ReactiveSystem.OnExecute”方法

注意：

在组件数据发生改变时，其内部会先将之前的组件从实体上移除，之后添加新的组件，因此“组件发生改变的操作”会同时满足“IMatcher.Added”和“IMatcher.Removed”规则

因此通常情况下，只需要在在“GetTrigger”方法体内使用“IMatcher.Added()”即可，因为其已经包含了“添加组件和组件数据发生改变”两种情况，此时不要设定为“IMatcher.Added() 和 IMatcher.Changed”。因为“组件数据改变”时会导致“ReactiveSystem.OnExecute”方法执行两次：一次为“满足IMatcher.Added”规则时触发，一次则是满足“IMatcher.Changed”规则时触发

![image-20230512192654492](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230512192654492.png)



2.ReactiveSystem提供“Filter”方法筛选出满足条件的实体，提供给“OnExecute”方法种的“List<T> entities”参数：

可以自由重写“Filter”方法，只有满足其指定条件的实体才能放入“List<T> entities”集合种

注意：“Filter方法”与“GetTrigger”方法是配套使用的，“List<T> entities”集合中只包含“满足IMatcher规则”，同时满足“Filter”过滤条件的实体才会包含在该集合中



3.核心执行逻辑：

对Filter方法筛选过的“List<T> entities”集合中的所有实体进行遍历



4.构造方法其实没啥用，主要对本脚本中需要用到的一些参数做初始化而已，是个很普通的方法。但由于每个自定义脚本通常都是继承自“ReactiveSystem.cs”而来，因此需要在脚本中保留此构造方法



例子如下：

```c#
//就只是个普通的构造方法而已，主要对本脚本中要用到的一些参数做初始化
public LaunchArenaSystem(LogicContexts contexts) : base(contexts, contexts.logicArena) {
}

//使用“GetTrigger”方法和“IMatcher”参数，控制该ReactiveSystem的触发时机
//注意：此IMatcher变量可同时包含多个组件
private static readonly IMatcher<LogicArenaEntity> ArenaMatcher = LogicArenaMatcher.Arena;
protected override ICollector<LogicArenaEntity> GetTrigger(IContext<LogicArenaEntity> context) {
	return context.CreateCollector(ArenaMatcher.Added());
}

//虽然该ReactiveSystem被触发，但只有满足特定条件的实体才可以包含在“List<T> entities”集合中
protected override bool Filter(LogicArenaEntity entity) {
	return entity.isArena;
}

//核心逻辑：真正负责ReactiveSystem中核心部分的方法，对满足Filter筛选后的“实体集合”中每个实体进行操作
protected override void OnExecute(List<LogicArenaEntity> entities) {
    ........
}
```

#### 对“ReactiveSystem”进行扩展建立“EventSystem”——事件监听系统：

```c#
internal abstract class LogicEventSystem<T> : LogicReactiveSystem<LogicEventEntity> where T: IEventContext
{
    //普通的构造方法
    protected LogicEventSystem(LogicContexts contexts)
		: base(contexts, contexts.logicEvent) {
	}
    
    //设置“触发时机”
	private static readonly IMatcher<LogicEventEntity> EventMatcher = LogicEventMatcher.EventContext;
	protected override ICollector<LogicEventEntity> GetTrigger(IContext<LogicEventEntity> context) {
		return context.CreateCollector(EventMatcher.Added());
	}

    //设置“集合entities”中的所有实体必须满足的筛选条件
	protected override bool Filter(LogicEventEntity entity) {
		return entity.hasEventContext;
	}

	protected override void OnExecute(List<LogicEventEntity> entities) {
		foreach (var eventEntity in entities) {
			if (eventEntity.eventContext.Value is T context) {
				OnEvent(context);
			}
		}
	}

    //提供给所有派生类自由重写
	protected abstract void OnEvent(T context);
}
```



### IExecuteSystem：每帧都会执行

IExecuteSystem接口中包含“void Execute()”方法。如果某个脚本需要包含“每帧执行”的方法，则实现该接口即可

```c#
public abstract class LogicExecuteSystem : IExecuteSystem
{
	protected LogicExecuteSystem(LogicContexts contexts){
		
	}

    //核心方法：每帧都会执行
	public void Execute() {
		OnExecute();
	}

    //提供给派生类自由重写
	protected abstract void OnExecute();
}
```



### PS：获取指定类型组件的实体集合的方法：

注意：该方法与“ReactiveSystem.cs”中“通过GetTrigger, IMatcher设置ReactiveSystem.OnExecute的触发时机，并通过“Filter()”筛选实体是不同的。

“GetTrigger，IMatcher”控制的是“ReactiveSystem.OnExecute”被触发的时机：只要满足IMatcher的条件，在“添加目标类型的组件”或该组件数据发生修改时，则触发“ReactiveSystem.OnExecute”方法

但是“OnExecute”方法中的“List<T> entities”集合中的元素则由“Filter”方法设定。

部分情况下除了目标组件外，还需要该实体中同时具有其他一些组件或满足特定的需求。此时则可以充分利用“Filter”来实现

“GetTrigger，IMatcher”决定的是该实体中“目标组件发生变化”可以触发，但其却不一定通过“Filter”的筛选

而本方法可以在任何情况下使用，仅仅只是用于查找满足目标“IMatcher规则”的所有实体，其不具备触发任何方法的能力，仅用于查找使用。相当于一个普通的方法而已



实现逻辑：这里声明了两个变量“_buffGroup”和“BuffMatcher规则”，最后通过“_buffGroup.GetEntities()”方法获取到所有满足“BuffMatcher规则”的实体，并将其放入“_buffer”集合中

例子如下：

```c#
public class TriggerCountHurtTimesSystem : LogicEventSystem<SkillExpHurtEvent>
{
    //获取满足“BuffMatcher规则”的所有实体
	private readonly IGroup<LogicBuffEntity> _buffGroup;
	private readonly List<LogicBuffEntity> _buffer = new List<LogicBuffEntity>();
	private static readonly IMatcher<LogicBuffEntity> BuffMatcher = LogicBuffMatcher.AllOf(
		LogicBuffMatcher.OwnerId,
		LogicBuffMatcher.TriggerByHurt)
	.NoneOf(
		LogicBuffMatcher.Destroyed,
		LogicBuffMatcher.Terminated);

    //由于其为构造方法，在最初即会被调用，此时“_buffGroup”中并没有实际的实体对象
	public TriggerCountHurtTimesSystem(LogicContexts contexts) : base(contexts) {
		_buffGroup = contexts.logicBuff.GetGroup(BuffMatcher);
	}

    //使用“_buffGroup.GetEntities()”获取到所有满足“BuffMatcher规则”的实体，并将其放入“_buffer”集合中
	protected override void OnEvent(SkillExpHurtEvent context) {
		foreach (var buffEntity in _buffGroup.GetEntities(_buffer)) {
			........
		}
	}
}
```











