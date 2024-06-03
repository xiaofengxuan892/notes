[TOC]



##### if....else...格式：

```lua
if index <= 0 or index > #self.summaries then
    return
end
```

```lua
if self.playerA and self.playerA.Source then
    ........
else
    ......
end
```



##### if...then...语句的执行次数：

```lua
local num = 6
if num > 4 then
    print(string.formatex("greater than 4"))
elseif num > 5 then
    print(string.formatex("greater than 5"))
end
```

以上语句只会输出一次：

![image-20220802102133449](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220802102133449.png)

if 和 else之间同时只会有一个被执行，因此以上代码中“greater than 5”不会输出



##### for.... do 格式：

```lua
local num = 0
for i = 1, num do
    print(string.format("hello, world...."))
    print(string.formatex("ceshi yixia ...........  i : {0}", i))
end
```

**由于"num < 1”，所以print语句一次都不会执行。**

运行结果完全空白：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220525104551571.png" alt="image-20220525104551571" style="zoom: 80%;" />

**将num数值改为1后，print语句会执行一次**

```lua
local num = 1
for i = 1, num do
    print(string.formatex("ceshi yixia ...........  i : {0}, num: {1}", i, num))
end
```

输出结果如下：

![image-20220525102548531](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220525102548531.png)

**<!--总结：由此可见，for循环内部相当于while的遍历，在执行之前会检测"i <= num"，只有在满足条件时才会执行。而不是“do.....while”的遍历-->**



##### for _, v in pairs的格式：

```lua
local data
for _, v in ipairs(self.bossDrop) do
    if v.monsterId == monsterId then
        data = v
        break
    end
end
return data
```

如上可见，break关键字也可正常使用——跳出当前循环

注意：使用ipairs遍历集合时，**<font color=red>默认key值从1开始，每次递增1</font>**，如果无法找到目标数值的key元素，则遍历结束。所以使用“ipairs”遍历集合有可能无法获取到集合中的所有元素

```lua
local t1 = {"hello", "world", [3] = "yeah", ["haha"] = 4};
for _, v in ipairs(t1) do
    print(v);
end
```

输出结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231107193221898.png" alt="image-20231107193221898" style="zoom:80%;" />

解析：当元素没有key值时默认以元素的下表index为key值，因此可以正常输出”hello“， ”world“

由于表中有“key == 3”的元素，满足递增条件，所以可以正常输出”yeah“

由于表中没有“key == 4”的元素，不满足递增条件，所以遍历结束

**注意**：

1).**”ipairs“并不是从左到右或从右到左，而是直接在所有元素中遍历，当有满足递增条件的key元素时，则直接输出该元素**

2).通常使用“pairs”代替“ipairs”遍历集合。为避免错误，尽量少使用“ipairs”遍历ji'he





##### for _, v in pairs()无法对nil表进行遍历，必然会报错：

```lua
local testTable = nil
for _, v in pairs(testTable) do
    local num = 1
end
```

当pairs中的参数为nil时会直接报错。因此该for... in pairs无法自动对nil表做判断



**注意**：

**1.使用“for k,v in pairs”对table进行遍历时，无法确保按照key值从1开始依次递增的顺序来输出**，**因此当对遍历输出的顺序有严格要求时，建议使用“for i = 1, ... do”**

**2.Lua中没有“continue”语法**，所以在遍历时要与C#的功能写法区分开





##### lua中使用string.split得到的数组序号是从1开始的：

```lua
function ItemIconBuilder:ParsingItemString(value)
    assert(not string.isnullorempty(value), "item string should not be empty")

    local szValue = string.split(value, "_")
    assert(#szValue >= 2,
            "item string format error, expected 'ItemType_ItemId_ItemCount' or 'ItemType_ItemId', got: %s", value)

    local itemType = checknumber(szValue[1])
    assert(itemType > 0, "itemType should be greater than 0, got: %s", szValue[1])

    local itemId = checknumber(szValue[2])
    assert(itemId > 0, "itemId should be greater than 0, got: %s", szValue[2])

    local itemCount = checknumber(szValue[3])

    return itemType, itemId, itemCount
end
```



##### Lua中为多个变量一起赋值：

###### C#：

int num1 = 1;

int num2 = 5;

可以在同一句中直接赋值：int num1 = 1, num2 = 5;

**但是当num1,num2的类型不确定时，如“var num1 =1, num2 = 5”此时则会报错**

如果多个参数赋值相同时，也可以使用如下的方式：

```c#
int numC, numD;
numC = numD = 8;
```

注意：不能使用“int numA = numB = 8”，因为第二个参数“numB”在赋值前没有被声明

###### Lua:

多个变量一起赋值，以“=”号左侧参数顺序为准，依次将右侧的数值赋值给左侧参数

如： a, b, c = 1, 3, 2

此时a, b, c 分别被赋值1，3， 2

如果左侧参数数量 > 右侧赋值数量，则左侧多余的参数没有赋值，保持为nil

如果左侧参数数量 < 右侧赋值数量，则右侧多余的参数直接忽略



##### 当指定table中元素的key值后再获取其中的数值和table的长度：

```lua
local numList = {}
numList[12] = "hello"
numList[15] = "world"
for k, v in pairs(numList) do
    print(string.formatex(" numList: k: {0}, v: {1}", k, v))
end
print(string.formatex("numList[1]: {0}, numList[2]: {1}, len: {2}", numList[1], numList[2], #numList))
```

输出结果如下：

![image-20220527100033361](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220527100033361.png)

由此可见，**当未指定table中元素的key时则使用默认从1开始**，如果指定了顺序，那么则使用指定的key值

变化一：当改为：

```lua
local numList = {}
numList[1] = "hello"
numList[15] = "world"
for k, v in pairs(numList) do
    print(string.formatex(" numList: k: {0}, v: {1}", k, v))
end
print(string.formatex("numList[1]: {0}, numList[2]: {1}, len: {2}", numList[1], numList[2], #numList))
```

输出结果如下：

![image-20220527100315323](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220527100315323.png)



变化二：当改为

```lua
local numList = {}
numList[15] = "world"
for k, v in pairs(numList) do
    print(string.formatex(" numList: k: {0}, v: {1}", k, v))
end
print(string.formatex("numList[1]: {0}, numList[2]: {1}, len: {2}", numList[1], numList[2], #numList))
```

输出结果：
![image-20220527100429868](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220527100429868.png)



##### 针对nil的对象的判断：

###### 例子1：

```lua
local itemConfig = sq.facade:GetConfigManager():GetConfig()
if not itemConfig then
    return nil
end
```

```lua
local config = self:LoadConfig(configName)
if not config then
    assert(false, "Load config failed: " .. configName)
end
```

###### 例子2：

对于nil对象直接获取其长度是会报错的

```lua
local point = nil
print(string.formatex("len: {0}", #point))
```

运行之后报错：

![image-20220525103751775](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220525103751775.png)

tonumber无法转换空字符串，并不会自动转为0：

```lua
local str = ""
print(string.formatex("zhuanhan xia :  {0}", tonumber(str)))
```

![image-20220526173102926](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220526173102926.png)

###### 例子3：

**针对于number或其他类型的对象是可以直接用于判空的操作的**，如：

```lua
local num = 1
if num then
   print(string.format("num is not nil"))
end
```

如上是可以正常执行的。事实上，任何非空的参数都可以直接用来作为判空的条件

```lua
local num = 2
if not num then
   print(string.format("num is nil"))
end
```

以上print语句则永远不会执行

**从实际效果来看，任意非空的参数在用于判空时始终代表的都是true**

所以在使用table.indexof时，是可以直接将table.indexof的结果用于判空操作的

table.indexof方法的详情如下：

```lua
function table.indexof(array, value, begin)
    for i = begin or 1, #array do
        if array[i] == value then return i end
    end
    return false
end
```

所以table.indexof可能返回false —boolean，或者具体的数值number —非nil

所以如果直接将table.indexof的结果用于判空是完全符合条件的。但某些情况下可能需要将table.indexof直接作为boolean使用，如：

```lua
local isInMission = self.battleSoulCopyModule:CheckGeneralInMission(self.eId)
GraphicUtils.SetGraphicEnabled(self.imageInMission.transform, isInMission)
```

“CheckGeneralInMission”的方法体内部则会直接返回table.indexof的返回结果。因此需要将其进行转换后才能直接作为boolean来使用：

```lua
function BattleSoulCopyModule:CheckGeneralInMission(eId)
    return self.allGeneralEntityIdInMission and table.indexof(self.allGeneralEntityIdInMission, eId) and true or false
end
```

如上根据table.indexof的结果非空或false来转换得到最终的boolean结果

###### 例子4：

对于判空与true/false叠加的情况需要将两者区分开

如对于红点的判断，

- 首先需要检测本地数据时，有两种情况：

  1. 有本地数据，因此可以得到确定的true/false   
  2. 没有本地数据，此时返回结果为nil

- 当没有本地数据时才会检测服务器数据

  此时需要明确的区分“nil” 和 “true/false”两种情况。

  因此在使用if判断时则需要使用如下形式：

  ```lua
  if result ~= nil then
     return result    -- 当本地数据非nil时，则使用本地检测结果
  end
  
  -- result为nil, 开始检测服务器数据
  ```

  而不能直接使用：

  ```lua
  if not result then       -- "not result"无法将nil和false区分开
     ....
  end
  ```






##### 空表的使用：

```lua
local kEmptyTable = {}

if not table.indexof(self.systemOpenIds or kEmptyTable, config.SystemID) then
    table.insert(self.systemUnopenedConfigs, config)
end
```

当self.systemOpenIds为nil时，则直接使用kEmptyTable，这样避免报错

###### Lua中的空表是不能用于“==”或“Equal”比较的：

```lua
local table1 = {}
local table2 = {}
if table1 == table2 then
    print("yes.............")
else
    print("no..............")
end
```

输出结果如下：

![image-20221020115652769](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221020115652769.png)

**解析**：<font color=red>**虽然table1和table2都赋值为空表“{}”，但每创建一个新的table时得到的地址以及编号都是不同的，两者无法等价**</font>

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221020120041587.png" alt="image-20221020120041587"  />![image-20221020120129991](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221020120129991.png)

如上图，<font color=red>**两者的编号等信息都是完全不同的，虽然table中的元素都为空**</font>

<font color=blue>**但是在获取数据时，如果为nil，可以使用“{}”代替，然后对“{}”进行"for..in pairs"遍历，这样可以避免遍历时表格为nil的报错。**</font>

<font color=red>**但不可将“{}”用于等价比较**</font>



##### tonumber将空格string转换时：

```lua
local str = ""
if tonumber(str) > 0 then
    print(string.formatex("greater than 0"))
end
print(string.formatex("str: {0}", tonumber(str)))
```

此时在执行“if tonumber(str) > 0 then”时就会报错，当str为空字符串时，tonumber转换得到的number也是nil，此时无法将nil值与0进行比较。

所以当str为nil或空字符串时需要单独处理。tonumber并没有将空的字符串转换成0的内部操作

![image-20220527150545503](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220527150545503.png)



##### table.clone处理“表的引用连带”：

lua中也存在表的引用连带：

```lua
local numList = {}
local numList2 = numList
numList2[1] = 100
print(string.formatex("count: numList: {0}, numList2: {1}, value: numList: {2}, numList2: {3}", #numList, #numList2, numList[1], numList2[1]))
```

![image-20220525230308330](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220525230308330.png)

从以上结果可知，table之间的连续引用也会把所有的对象都改变

**解决办法**：

**使用table.clone进行拷贝：**

```lua
local numList = {}
local numList2 = table.clone(numList)
numList2[1] = 100
print(string.formatex("count: numList: {0}, numList2: {1}", #numList, #numList2))
```

运行结果：

![image-20220525231438549](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220525231438549.png)





**将tableA中所有元素都放入tableB中：table.insertto**

```lua
local ret = {}
for _, filterType in ipairs(kBagGroupIndexMap[selectedPage]) do
    local temp = self.packageModule:GetGoodsByPackageGroup(filterType)
    table.insertto(ret, temp)
end
```



##### table.pick:

对table中的元素进行pick过滤：

```lua
--- @brief 根据战斗类型获取技能列表
--- @param battleType BattleType
--- @return table[]
--- @public
function SkillComponent:GetSkillsByBattleType(battleType)
    local filter = function(v)
        local skillConf = sq.facade:GetConfigManager():GetConfig("SkillConf_SkillLevel",
                "SkillId", v.id, "SkillLevel", v.level)
        if skillConf.BattleType == BattleDef.BattleSkillType.Both then
            return true
        end
        if battleType == BattleDef.BattleType.TowerDefense 
                and skillConf.BattleType == BattleDef.BattleSkillType.TowerDefense then
            return true
        end
        if battleType == BattleDef.BattleType.BattleArena 
                and skillConf.BattleType == BattleDef.BattleSkillType.BattleArena then
            return true
        end
        return false
    end

    local skills = self:GetAllSkills()
    local ret = table.pick(skills, filter)
    return ret
end
```



##### Table.Sort：

###### 描述：

对列表进行排序

- **使用范围：**该table的key需要从1开始递增，最好是使用“table.insert”最终得到的列表。如果key不统一，则无法得到正确的排序结果。比如：

  ```lua
  local test = {[3] = 2, ["hello"] = 5, [1] = 6, ["world"] = 0, ["today"] = 8, ["is"] = 1}
  ```

   这样的列表是无法使用table.sort来排序的

- **比较参数：**

  **这里参与比较的“a,b"代表的是table中的具体元素，但是这个元素不包含”keyValuePair“的键值对，只包含value，不包含key。**

  因此如果value中不包含特别的参数，那么可以直接使用“return a > b”来比较；如果value中包含其他的具体参数，则需要使用参数来比较，比如“return a.xxx > b.xxx”

  例子如下：

  ```lua
  local test = {3, 1, 5, 3, 5, 8, 2, 0}
  local test1 = table.clone(test)
  local function _sortFunc(a, b)
      return a > b
  end
  table.sort(test1, _sortFunc)
  for i = 1, #test1 do
      print(string.formatex("test1: i: {0}, value: {1}", i, test1[i]))
  end
  for i = 1, #test do
      print(string.formatex("test: i: {0}, value: {1}", i, test[i]))
  end
  ```

  运行结果如下：

  ![image-20220724172258115](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220724172258115.png)



###### 使用方式：

**单个条件排序**：

1.可以直接使用“>”, "<"的参数类型，如number：

```lua
table.sort(list, function(a, b)
    return a.count > b.count
end)
```

2.无法直接使用“>”, "<"的参数类型，如boolean：

```lua
table.sort(list, function(a, b)
    return a.state and not b.state           -- 顺序：true -》false
end)
```

**多条件排序**：

**<font color=red>在多个参数之间依次排序时需要借助“~=”来切换</font>**

**<font color=red>“~=”的主要作用在于将使用当前比较条件无法得出结果的情况区分出来，留给下一个条件去进一步判断</font>**

```lua
local function _sortRulesFunc(a, b)
	if a.state ~= b.state then
		return a.state and not b.state   -- 顺序：true -> false
	elseif a.count ~= b.count then
		return a.count > b.count
	else                        -- 最后一个比较参数，无需再使用"~="
		return a.id < b.id
	end
end

-- 注意：每一个参数都要有明确的true/false的比较结果，如以上最后一个参数，如果使用:
....
    elseif a.id ~= b.id then
        return a.id < b.id
    else
        return false      -- 专门对“a.id == b.id”的情况说明比较结果
    end
....
```

**完整例子如下：**

```lua
local temp = {}
table.insert(temp, { id = 6, state = true, count = 100})
table.insert(temp, { id = 1, state = false, count = 98})
table.insert(temp, { id = 3, state = true, count = 25})
table.insert(temp, { id = 2, state = true, count = 25})
table.insert(temp, { id = 5, state = true, count = 25})

local function _sortRulesFunc(a, b)
	if a.state ~= b.state then
		return a.state and not b.state
	elseif a.count ~= b.count then
		return a.count > b.count
	elseif a.id ~= b.id then
		return a.id < b.id
	else
		return false
	end
end
table.sort(temp, _sortRulesFunc)
for i = 1, #temp do
	print(string.formatex("i: {0}, val: { state: {1}, id: {2} }", i, temp[i].state, temp[i].id))
end
```

运行结果如下：

![image-20220727163450226](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220727163450226.png)

**注意**：

1.**<font color=red>在table.sort排序时不要使用“>=”, "<="  ，使用“>”,"<"</font>**

2.如果用于比较的参数A在table中某些元素内是不存在的，那么比较便捷的方法是：**添加额外的参数B专用于代表参数A是否存在的状态，在比较参数A之前先比较参数B**

```lua
for _, v in pairs(list) do
    if v.state == nil then
        v.hasBeen = false    --"hasBeen"为另外添加的，用于后续比较的参数
    else                     -- 这里也可以使用更方便的number参数
        v.hasBeen = true
    end
end
```

3.为了不影响原本的table中的元素顺序，这里使用table.clone深拷贝(即建立一个新table)

4.为简洁代码，可以使用local function的格式： table.sort(list, _sortFunc)



##### 获取table表格的长度：

<font color=red>**针对数组类型的table，如“local table = {1, 2, 3}”，可以直接使用“#”来获取table的长度**</font>

<font color=red>**但对于“键值对”类型的table，则不能使用“#”获取table长度**</font>

```lua
local tempTable1 = {4, 1, 5, 3}
print(string.formatex("tempTable1.len: {0}", #tempTable1))

local tempTable2 = { a = 1, b = 2, c = 3}
print(string.formatex("tempTable2.len: {0}", #tempTable2))

local tempTable3 = { [1] = "a", [2] = "b", [3] = "c"}
print(string.formatex("tempTable3.len: {0}", #tempTable3))
```

输出结果如下：

![image-20221014152619024](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221014152619024.png)

**注意：**

<font color=blue>**以上代码中“tempTable3”实际上等价于“tempTable3 = {"a", "b", "c"}”，所以本质上“tempTable3”也是数组类型的table**</font>

**对于键值对类型的table可以创建“TableEx.lua”文件，专用于table相关的各种扩展方法**

```lua
function table.nums(t)
    local count = 0
    for k, v in pairs(t) do
        count = count + 1
    end
    return count
end
```



##### 局部方法“local function”的使用：

局部的function，当作单独的参数来使用，不需要声明成”self.....方法“：

```lua
local function __ActiveObject(object, value)
    if object then
        object.gameObject:SetActive(value)
    end
end
```

这个local方法和普通的参数本质上是一样的，

1.如果写在lua脚本之外，就和平常的”local kAlphaNormal = 1“那样写在脚本之外，那么就是当成这个脚本自身的变量而存在

PS: 经过实际测试发现，如果"local kAlphaNormal = 1" 写在该panel之外，那么一旦改变该数值，以后每次使用到该panel时都会使用改变后的数值，除非再次重置该数值。针对这种情况有两种解决方式：1.将该参数设置为self参数，那么每次打开该panel时都会被重置，因为panel被创建时对应的变量也是重新创建的  2.每次在初始化该panel时都重置"local kAlphaNormal = 1"。但这样就和”self“没啥区分了。所以还不如用”self“

2.如果该local方法写在其他地方，则当作普通的参数使用即可



##### Lua中用到的math方法：

![image-20220719152829996](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220719152829996.png)





##### `os.date()`在Lua中的使用：

###### 只有参数“format”, 没有“time”参数 —— 此时默认将os本地的时间进行转换

os.date()：以默认格式输出当前系统时间，格式为：Mon May 16 14:26:32

os.date("%x"): 输出格式为: mm/dd/yy，如 07/26/22

os.date("%Y/%m/%d %H:%M:%S")：将当前系统时间按照“2020/02/03 15:20:30”的形式输出，如果不需要某些参数，可以改成“%m/%d %H:%M”，或者“%Y/%m/%d”，则对应只会获取相应参数，即“年/月/日”，“时/分/秒”的形式。

**PS: 如上使用的是“/”来间隔各个参数，当然也可以使用“-”或空格来间隔，如“%Y-%m-%d”等，此时则输出为“年-月-日”**

###### os.date("*t")： 

**<font color=red>将该时间以lua中特有的table形式输出，并且以这种形式可以得到的日期数据最全</font>**

除了包含“year - 年份，month - 月份， day - 日号，hour - 小时，min - 分钟，sec - 秒”参数外，还包含以下重要参数：

**wday：**该day是本星期中的第几天，(特殊点在于：这个方法是按照欧美的风格设计的，因此Sunday对应数值1，欧美星期表：[Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday])，因此当wday为1时代表Sunday，wday为7时代表Saturday

**yday：**该day是今年的第几天，从今年的第一天开始计算，如果yday为1，则代表今天是1月1号

**isdst：** 世界不同地区根据日照情况会有不同的计算方式。—— 该参数一般不会用到，可忽略

![image-20220726165628704](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220726165628704.png)

![image-20220726165656429](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220726165656429.png)

各参数具体取值范围：

![image-20220726165825098](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220726165825098.png)

Rider中的断点截图如下：

![image-20220726173237194](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220726173237194.png)

参考链接：https://developpaper.com/lua-os-date-notes/



2.有“time”参数，那么则是将该“os.date”方法作为普通方法使用，将传进来的time参数以指定的format格式进行转换。("time"参数默认以秒为单位)

如： local dateTable = os.date("*t", dateTime)，则得到“dateTime”的table表格形式，可以获取table中的所有参数数值



##### `string.format`在Lua中的使用：

###### 常用转义字符：

%s：接受一个字符串，以指定格式输出该字符串

%d，%i：接受一个数字，转换成有符号的整数格式

%o：接受一个数字，转换成八进制格式

%u：接受一个数字，转换成无符号整数格式

%x：接受一个数字，转换成十六进制，使用小写字母

%X：接受一个数字，转换成十六进制，使用大写字母

%e：接受一个数字，转换成科学计数法格式，使用小写字母

%E：接受一个数字，转换成科学计数法格式，使用大写字母

%f：接受一个数字，转换成浮点数格式

###### 将整数以“指定位数”输出

比如将“100”固定转换成“00100”输出，可以使用“**<font color=red>占位符 - 0</font>**”来实现：

```lua
local a = string.format("%5d", 100)
local b = string.format("%05d", 100)
print(string.formatex("a: {0}, b: {1}", a, b))
```

输出结果如下：

![image-20221110165811667](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221110165811667.png)

由上可知：当参数为整数时，必须要有**<font color=red>占位符“0”</font>**才能以指定位数输出

###### 保留小数点后“指定位数”输出：

```lua
local c = string.format("%.3f", 100)
local d = string.format("%.3f", 0.1)
print(string.formatex("c: {0}, d: {1}", c, d))
```

输出结果如下：

![image-20221110170549237](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221110170549237.png)

解析：%.3f：如果小数点后是“f”，则代表参数为浮点数，并且输出结果只保留浮点数小数点后的n位(第n位可以四舍五入)。如`string.format("%.2f", 0.266)`，输出结果为“0.27”

###### 将字符串以“指定字符个数输出：

%.3s：如果小数点后是“s”，则代表参数为字符串，并且输出结果只保留字符串前n位

```lua
local c = string.format("%.3s", "abcdefgh")
print(string.formatex("c: {0}", c))
```

输出结果：

![image-20221110171435265](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221110171435265.png)



##### “or”和“and”在Lua中的使用：

###### 针对“两个非nil数值”使用“or、and”进行操作：

```lua
local tempNum1 = 1 or 2
local tempNum2 = 1 and 2
print(string.formatex("tempNum1: {0}, tempNum2: {1}", tempNum1, tempNum2))
```

输出结果如下：

![image-20221013174449318](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221013174449318.png)

###### 针对“可能为nil的数值”使用“or、and”进行操作：

```lua
local forwardState = sq.facade:GetBattleManager():GetFastForwardToEndState()
self.fastForwarding = forwardState or false

local num = 100
```

使用断点方式检查，设置断点处为“local num = 100”，因此当代码运行到“local num = 100”时，“self.fastForwarding = forwardState or false”已执行完

![image-20221024115632324](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221024115632324.png)

![image-20221024115649378](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221024115649378.png)

从以上可知：<font color=red>**当forwardState为nil时，“self.fastForwarding”最后会被赋值为“false”**</font>

<font color=blue>**有些情况下参数有指定类型，但在实际赋值时有可能会出现“nil”的情况，因此为了避免“nil”和“false”的混淆，同时不改变参数原有的类型，可以使用这样的方式来达到效果**</font>



##### coroutine相关的方法：

###### coroutine.wait：

等待指定秒数。参数为“int”类型，代表秒数

###### coroutine.step：

等待指定帧数。参数为“int”类型，代表帧数

###### coroutine.running： 

获取当前正在执行的协程id和bool值。针对bool值，如果该协程为“主线程”，则返回true，否则返回false

**PS**：主线程的判定，如果该语句是在某个coroutine中调用，则不是主线程；如果跳出“协程方法体”，则为主线程

```lua
local co = coroutine.create(function()
      .....
      coroutine.running()  -- 此时返回false，因此这不是主线程
)

........
coroutine.running()  -- 此时返回true，因为已经跳出协程方法体，是主线程的执行语句
```

###### coroutine.status：

返回指定协程的状态，如`coroutine.status(co)`：若该协程正在运行，则返回running；若协程被挂起，则返回suspended; 若协程已经执行完毕，则返回dead

###### coroutine.create(f)：

创建一个主体函数为f的新协程("f"代表“lua语言”编写的函数)。该方法仅创建协程，并不会启动该协程

###### coroutine.yield: 

将当前协程挂起，是否有返回值则取决于“yield”后是否带有参数

###### coroutine.resume: 

启动或恢复协程。该方法会有两个返回值：第一个返回值代表该协程当前执行结果 —— 如果遇到yield语句协程被正常挂起或整个协程执行完毕，则返回true；如果是其他情况，则返回false，第二个参数如果该协程有返回值，则返回该结果；如果协程没有返回值，则coroutine.resume只有第一个bool返回值

常用于以下两方面：

1.**<font color=blue>启动或恢复协程</font>**：

- 如果在本次“resume”后并没有遇到“coroutine.yield”语句，则该协程会一直执行到完全结束为止，返回结果：true

- 如果遇到“coroutine.yield”，则本次“resume”结束：

  - 如果“coroutine.yield”有返回值A，则可以得到协程当前阶段执行的结果： true   A

    **此时的“A”作为协程当前的返回值，可以正常给其他参数赋值。但协程当前已被挂起，“A”并不是协程最终的返回值**

  - 如果“coroutine.yield”没有返回值，则只能得到： true

  **如果要继续执行协程则需要再次调用<font color=red>“coroutine.resume(co)”</font>**

2.**<font color=blue>向协程中传递参数</font>**：`coroutine.resume(co, parameter)`

- 如果协程“co”尚未执行过，那么“parameter”代表该方法体中的参数

  ```lua
  co = coroutine.create(function(a)
          ....
          local r = coroutine.yield(2 * a)
          ........
          
          return 3 + r
  )
      
  coroutine.resume(co, 1)     -- 第一次执行“resume”，结果为 true 2
  coroutine.resume(co, 2)     -- 第二次执行“resume”, 结果为 true 5
  ```

  如上，当第一次执行“coroutine.resume(co, 1)”时，则代表将数值“1”赋值给方法体中的参数“a”

- 如果协程“co”已经执行过现在再次执行“coroutine.resume”，**<font color=red>则需要找到上次“coroutine.yield”的位置</font>。此时的“parameter”代表的则是“coroutine.yield”的真实数值**，**<font color=red>即再次向协程中传递参数</font>**。**<font color=blue>如果没有“parameter”，那么“local r = nil”</font>**

  ```lua
  co = coroutine.create(function(a)
         ............
         local r = coroutine.yield(2 * a)
          ........
         return 3 * a
  end)
  
  print(coroutine.resume(co, 1))   -- 执行结果：true  2
  print(coroutine.resume(co, 2))   -- 执行结果：true  3
  ```

**所以：**

**<font color=red>coroutine.yield：将结果传递到协程外部(如果有返回值)</font>**

**<font color=red>coroutine.resume(co, parameter)：从外部向协程中传递参数</font>**



###### coroutine在“UI界面流程”中的使用：

打开页面后先挂起当前协程，当页面关闭时重新恢复本协程

**例子1**：打开“二次确认框”后挂起当前协程，当“确认框”关闭后再恢复该协程

![image-20220817190633599](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220817190633599.png)

![image-20220817190713222](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220817190713222.png)

**例子2**：打开任意界面后挂起自身并等待“新界面”关闭 —— 图鉴详情面板 

![image-20220826141630603](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220826141630603.png)

![image-20220826141644372](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220826141644372.png)

![image-20220826141653514](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220826141653514.png)

![image-20220826141702506](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20220826141702506.png)

###### 在lua中开启的协程，通常需要统一管理，在界面关闭时要销毁：

```lua
table.insert(self.lazyThreads, self:DelayCallForFrames(1, Slot(self, self.PlayOpeningAnimaThenStartBattle)))
table.insert(self.lazyThreads, self:DelayCallForFrames(1, Slot(self, self.LazyLoadAssets)))
```

```lua
--- @brief 停止所有Lazy线程
--- @private
function BattleManager:StopAllLazyThreads()
    for _, thread in pairs(self.lazyThreads) do
        self:StopCoroutine(thread)
    end
    self.lazyThreads = {}
end
```

在开启协程时，将该协程放入统一的table容器内，关闭界面时要”self:StopCoroutine“，这样比较安全



##### Lua中require关键字的作用：

Lua代码：

```lua
require "System.class"
require "System.Math"
```

与#include作用类型，加载了该模块后就可以使用该lua脚本中的全局变量和全局函数了

并且在Lua中使用require加载一个"xxx.lua“文件时，会先在已加载的chunk中寻找，如果找到则直接返回，不会重复加载；如果没有找到，则去lua文件的默认搜索路径中查找该lua文件，**所以需要先设置lua文件的默认搜索路径：**

```lua
package.path = package.path..";C:/Users/xiaof/Desktop/SimpleFramework_UGUI-0.4.1/Assets/Lua/?.lua;"

require "Custom/OtherFuncLua"
```

在lua文件中添加”package.path = .......“语句则是在lua中设置了查找lua文件的默认搜索路径，否则会报找不到lua文件的错误

注意：在C#中调用lua文件时，需要得到该文件的完整路径



###### require关键字的返回值的使用：

当左边有变量用于承接require模块后的返回值，如`xxx = require "System.class"`

1).如果该模块有返回值，且模块加载成功，则得到该模块的返回值

2).如果该模块没有返回值，但该模块加载成功，则返回true

"require"在lua内部的实现过程：

```lua
function require(name)    
    if not package.loaded[name] then        
        local loader = findloader(name)       
        if loader == nil then            
            error("unable to load module" .. name)        
        end        
        
        package.loaded[name] = true        
        local res = loader(name)        
        if res ~= nil then            
            package.loaded[name] = res        
        end    
    end    
    
    return package.loaded[name]
end
```

