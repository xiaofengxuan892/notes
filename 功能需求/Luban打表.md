[TOC]





#### Luban配置系统使用环境：

##### “Luban打表”的执行环境：

“Luban打表”系统在Windows、Mac、Linux均可使用，但需要安装dotnet sdk 7.0：[dotnet-sdk-7.0.408-win-x64.exe](..\资源文件\A-KEY\dotnet-sdk-7.0.408-win-x64.exe) 

**PS**：官网下载地址：https://dotnet.microsoft.com/zh-cn/download/dotnet/7.0。**<font color=red>当前“Luban打表”中的“Luban.dll”需配合SDK7.0使用，其他版本不支持</font>**。下载时点击**左侧的“SDK”**(已包含右侧的运行时)即可：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240418105815607.png" alt="image-20240418105815607" style="zoom: 50%;" />

##### 在Unity中使用“Luban打表”生成的脚本和数据文件：

1.在PlayerSetting中开启“Allow 'unsafe' Code”选项：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413190406621.png" alt="image-20240413190406621" style="zoom:80%;" />

2.安装Luban插件：打开“Windows -> Package Manager”窗口，点击右上角选择“Add package from git URL”，并填入“https://github.com/focus-creative-games/luban_unity.git”链接即可安装Luban插件



#### Luban打表的命令行参数解析：

##### bat文件中".."和"."的作用：

"."：**<font color=blue>一个点号，代表当前目录</font>**；".."：**<font color=blue>两个点号，代表父级目录</font>**。通常使用这两个符号来设置路径以执行某些文件或操作，如：

```bat
set WORKSPACE=..
set LUBAN_DLL=%WORKSPACE%\Tools\Luban\Luban.dll
set CONF_ROOT=.

dotnet %LUBAN_DLL% ^
    -t all ^
    -c cs-simple-json ^
    -d json ^
    --conf %CONF_ROOT%\luban.conf ^
    -x outputCodeDir=GenerateDatas/csharp ^
    -x outputDataDir=GenerateDatas/json ^

pause
```

**PS**：如果是“当前目录”，则可以省略，直接填写该目录下其他的文件或文件夹名字即可，如以上“`--conf %CONF_ROOT%\luban.conf`”可直接使用“`--conf luban.conf`”

##### -c, -d 参数的作用：

`-c xxx`即“-code target xxx”，用于指定生成的脚本代码语言，如cs-bin,cs-simple-json,lua-bin,java-bin,java-json等；`-d xxx`即“-data target xxx”，用于指定生成的数据文件的格式，如json,lua,bin,xml,yml,msgpack,bson等

- -c xxx 和 -d xxx 需要配合使用，即**<font color=red>当前者为“-c cs-bin”时，则后者只能为“-d bin”</font>**，否则生成文件失败
- 如果**仅仅只需要校验配置，不需要生成对应的脚本和数据文件**，则**<font color=blue>不要在“gen.bat”文件中配置“-c”和“-d”即可</font>**

`-x outputCodeDir=xxx`，用于指定生成的代码脚本的路径；`-x outputDataDir=xxx`，用于指定生成的数据文件的路径

- **两者主要用于配合“-c xxx”和“-d xxx”来使用**。如果“gen.bat”文件中没有声明“-c”和“-d”，则无需配置“-x outputCodeDir”和“-x outputDataDir”

###### 同时生成多种格式的文件

如果需要同时生成多个不同语言或不同数据格式的文件，则需要**<font color=red>分别设置不同语言的脚本和数据文件的存放路径</font>**

**同时生成cs-bin和java-bin文件**(当数据文件格式相同时，只写一次“`-d xxx`”即可)：

```bat
-c cs-bin ^
-c java-bin ^
-d bin ^
--conf %CONF_ROOT%/luban.conf ^
-x cs-bin.outputCodeDir=GenerateDatas/csharp ^
-x java-bin.outputCodeDir=GenerateDatas/java ^
-x outputDataDir=GenerateDatas/bytes ^
```

**同时生成cs-bin和cs-simple-json数据文件**：

```bat
-c cs-bin ^
-d bin ^
-c cs-simple-json ^
-d json ^
--conf %CONF_ROOT%/luban.conf ^
-x cs-bin.outputCodeDir=GenerateDatas/csharp_bin ^
-x bin.outputDataDir=GenerateDatas/bytes ^
-x cs-simple-json.outputCodeDir=GenerateDatas/csharp_json ^
-x json.outputDataDir=GenerateDatas/json ^
```





#### 配置细节：

1.对于bool类型的数，在Excel中填写“False”，“false”等数值后会自动转换为“大写的TRUE”(该过程是自动的)

2.**datetime类型**的数据在打表时会**<font color=red>默认按照“当前本地的时区”转换成UTC秒数</font>**。如果游戏需要在不同时区上线，则在对应时区时需要将该数值重新转换后才可使用

3.在“Table.cs”填写的每一行配置中，**<font color=blue>“index”参数代表“主键值”，在其所属的表格中必须唯一：可以“多主键联合索引如key1+key2”，也可以“多主键独立索引如key1,key2”</font>**。**当为空时，则默认以表格中第一个字段为“主键”**

基于该特性，**<font color=red>如果某个表格中的“主键”存在重复情况，则显示打表失败</font>**：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240409100245053.png" alt="image-20240409100245053" style="zoom:80%;" />

4.如果**<font color=red>Excel表格中某个sheet的“A1单元格”不以“##”开头，则在打表时会被“Luban系统”自动忽略</font>**，不会读取该sheet的数据来校验和导出。

- 在功能开发中，**策划有时会在Excel中增加一个sheet用来说明部分配置数据或逻辑**，即可使用本机制实现



##### 可空类型：

**<font color=red>基础类型和自定义类型都支持可空，但容器类型不支持可空，且容器中的key和value也不支持可空</font>**。

- 基础类型如bool,byte,short,int,long,float,double,string，当单元格留空时，则**<font color=red>取该类型的“默认值”进行赋值(非null)</font>**。若需要修改默认值为null，则**必须给该类型添加“?”后缀**

  **注意**：string的默认值是`""`，而`string?`的默认值是null

- datetime类型虽为基础类型，但“Luban配置系统”中没有为其指定“默认值”。故**如果单元格留空，为避免报错则必须添加“?”后缀，此时默认赋值为null**

- **所有自定义枚举类型、自定义子类型**，若需要支持“单元格留空”，则必须添加“?”后缀

##### tag标签：

- **escape=1**，如`string#escape=1`

  在“Luban打表系统”的基础类型中，**<font color=red>string类型默认不处理字符串中的转义字符</font>**，如“\n”等。因此当在表格中配置为“获得狂暴效果，持续5秒\n战斗力增加100”，此时**无论是直接输出日志，还是将该文本复制给UIText，均不会有“换行”效果**。

  <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240412165353620.png" alt="image-20240412165353620" style="zoom: 80%;" /> <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240412165830196.png" alt="image-20240412165830196" style="zoom:80%;" />

  此时则可通过**在string后添加“escape=1”的tag**，即`string#escape=1`，来自动将**<font color=red>字符串中的“\n”替换为“换行符”</font>**

  ![image-20240412165639544](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240412165639544.png) <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240412165700731.png" alt="image-20240412165700731" style="zoom:70%;" />

- 如果需要为Excel中**某行数据添加标签**，则可**<font color=red>在该行第一个单元格中添加自定义标签“customTag”</font>**即可。但需要注意：除非在“执行文件gen.bat”中添加 `--excludeTag customTag`，否则该行数据会被导出。因此“tag标签”的使用并不是很方便，**通常情况一般不建议使用**

##### 注释行或列

**注释行**：在该行的第一列中填写“**<font color=red>##</font>**”，即代表该行数据为“注释行”

- **<font color=blue>表格中可以同时有多个注释行</font>**。所有的“注释行”中，**<font color=red>有且仅有第一行的数据会被导出作为“生成脚本”中各个字段的注释，其他行的数据不会被导出</font>**
- 在配置表格时，部分情况下某行的数据只填写了一部分，如果直接提交，则打表时会失败。但是如果直接删除，后续又需要重新填写。此时则可以**<font color=red>使用“##”将该行注释，由于该行数据不会被导出，因此也不会影响打表</font>**

**注释列**：excel表格中如果希望注释某列数据，则在该字段前**<font color=blue>添加“#”或将该字段留空</font>**即可

![image-20240412162046530](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240412162046530.png)

如上图所示，第4行、第7行为注释行，其中第4行作为各个字段的注释说明而被导出，第7行则不会被导出

**D列字段为空，E列字段前有“#”**，因此这两列数据均被注释，不会被导出

##### 容器类型的声明方式：

**array类型**：

**格式**：`array,元素类型`，如`array,int`，则代表数组中的元素为“int”类型

**list类型**：

**格式**：`(list#sep=集合中元素间的分隔符),元素类型`，如`(list#sep=;),CommonItemCom`，即list中存放的是“CommonItemCom”类型的元素。在填写该集合数值时，相邻元素间以“;”分隔，如“10008_1_2;10009_2_3”(其中的“_”是CommonItemCom类型元素内部的分隔符，与list容器无关)

**map类型**：

**格式**：`(map#sep=依次填写“key与value之间”以及“相邻元素间”的分隔符),key的类型,value的类型`，如`(map#sep=,;),string,int`，即map中的元素为“string,int”键值对。填写数据时，相邻键值对之间用“;”分隔，键值对内部使用“,”分隔，如“today,100;tomorrow,200”

- 键值对中“key”尽量使用基础类型，不要使用“bean”中自定义的新类型

##### “##group”参数配置：

**##group参数特性**：

1).针对"`__beans__.xlsx`","`__enums__.xlsx`"表格中的“group参数”默认不填。在打表时会自动根据“一般表格”中**<font color=red>是否有用到以上表格定义的“枚举类型或子结构类型”</font>*****<font color=blue>来确定是否需要将这些类型导出</font>**。但**如果指定了某个枚举类型的“group参数”，如设置为“c”**，则**<font color=blue>对于“client”无论该枚举类型是否被用到，都会被导出</font>**

2).对于一般表格中的“group参数”默认不填，其代表该列数据会默认导出到“c,s”等所有端。但**如果为某列数据的“##group”设置为“c”**，则**<font color=red>有且仅有“c”端才能导出该列数据，“s”端不会有该列数据</font>**

**总结**：

针对“一般表格”中的“group参数”默认不填，<font color=red>**当且仅当确定某列数据只有在“c”或“s”端才会用到时才会填写**</font>；

针对"`__beans__.xlsx`","`__enums__.xlsx`"表格中的“group参数”**<font color=red>默认不填</font>**，其中声明的类型在打表时会根据是否被“一般表格”用到来确定是否导出，无需手动设置

**“##group”参数的实际使用**：

Excel中只有少量数据是服务端需要的，**<font color=red>为了避免打表后有大量数据传递到服务端，可以对Excel表格中只有客户端才需要的数据单独标记出来</font>**；若该数据双端都需要，则无需标记

**根据“group”设置打表执行文件“gen.bat”**：

在“打表执行文件gen.bat”中，设置“-t server” —只将Excel表格中服务端需要的数据打出来，“-t client” —只将Excel表格中客户端需要的数据打出来，“-t all” —将Excel表格中所有数据都打出来

```bat
dotnet %GEN_TOOL% ^
     -t server ^
     -t client ^
     -t all ^
```

##### "input文件列表输入源”参数配置

“Table.cs”中支持为每个“valueType - 生成的CS脚本类名”设置多个文件输入源，其支持以下类型：

**1)**.**某个excel文件的所有单元薄**，如**<font color=red>xxx.xlsx</font>**。打表时会依次**<font color=red>读取该excel文件的所有单元薄</font>**，按照“valueType记录类名”中的字段依次匹配数据，所以**需要所有单元薄的“字段名、结构”都一致**，否则打表失败。

使用场景：通常**在“道具表item.xlsx”中区分多个单元薄，如“武器weapon，盔甲armor，时装fashion、消耗品consume”等**，所有单元薄的字段结构都相同。划分成多个单元薄则是为了“查阅、修改”，同时也不影响打表

**2)**.**某个excel文件的指定单元薄**，如**<font color=red>sheet@xxx.xlsx</font>**。打表时只读取该指定单元薄中的数据

使用场景：在功能开发中通常会将**某个业务逻辑所有需要的表格放在同一个excel文件中**，并以“各个不同的单元薄”区分，每个单元薄的“字段结构”都不同。主要在于方便整理该业务模块所有用到的表格数据，避免零散。同时Luban打表系统支持读取excel文件的指定单元薄，使用极为方便

**3)**.**<font color=red>某个目录下的所有文件(包含递归子目录)</font>**，如**<font color=blue>使用技能编辑器得到的所有武将的技能配置文件“xx.json”，可以将这些技能配置文件放在一个统一的文件夹下，如“skill_json_dir”</font>**。这些“技能配置文件xx.json”的“字段结构”必须完全相同，否则在匹配“valueType记录类名”中的各个参数时会失败

**注意**：**<font color=blue>该目录下最好放置同一类型的文件</font>**，**不要把“xx.json”文件和“excel文件”放置在同一文件夹目录下**

**4)**.**<font color=blue>其他lua, json, xml, yaml文件</font>**，如xx.lua, xx.json, xx.xml, xx.yml

**5)**.**<font color=red>以上任意文件的组合，如“input文件列表输入源”中可以同时填写 xx.xlsx, sheet@xx.xlsx, xx.json, xx_lua_dir</font>**。当同时设置多个输入源时，**<font color=red>每个输入源的“字段结构”必须完全相同</font>**(该字段下对应的实际数据可以不同)，否则在匹配“valueType记录类名”时会失败





#### bean类型的配置

当需要新的数据类型时，通常在“`__bean__.xlsx`”或“DataTables/Define/builtin.xml”中声明，然后在其他表格中使用该类型配置数据。**如果需要与“外部已有类型”映射，则通常在“builtin.xml”中声明**，否则直接在“`__bean__.xlsx`”中声明即可

##### 普通bean类型

- 包含“sep”分隔符

  在配置bean类型时，填写了“sep分隔符”，则在一般表格中使用该类型时可直接在一列中填写各个字段的数值：<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413115223072.png" alt="image-20240413115223072" style="zoom:80%;" />

  <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413115303423.png" alt="image-20240413115303423" style="zoom:80%;" />

  **PS**：“sep分隔符”可以是一个字符或多个字符。当为多个字符时，则代表其中任意一个字符均可用于拆分字段

- 不包含“sep”分隔符

  部分情况下如果bean类型中包含“string字段”，且**在填写该string字段的数据时，“字符串”本身已包含“sep”分隔符。如果仍旧按上述将各个字段在一个单元格中填写，则以“sep”分隔符拆分数据时可能会出错**

  在这种情况下，通常不配置该bean类型的分隔符，使用另一种方式在“一般表格”中配置该bean类型的数据

  <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413120040343.png" alt="image-20240413120040343" style="zoom:80%;" />

  在使用该“CommonWeaponData”类型时，**<font color=red>增加一个“##var”行并依次展示各个子字段</font>**
  <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413120250642.png" alt="image-20240413120250642" style="zoom:80%;" />

  同时在该种形式下，**<font color=blue>部分单元格可以留空，直接使用默认赋值，无需填写</font>**

  此时导出的数据如下：

  <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413121445401.png" alt="image-20240413121445401" style="zoom:80%;" /> <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413121504414.png" alt="image-20240413121504414" style="zoom:80%;" /> <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413121522711.png" alt="image-20240413121522711" style="zoom:80%;" />

##### 多态bean类型

**声明该类型**：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413164304549.png" alt="image-20240413164304549" style="zoom:80%;" />

基类为“QuizQuestionBase”，子类为“QuizTextQuestion”、“QuizImageQuestion”。**<font color=blue>由于QuizTextQuestion与基类的字段相同，没有需要额外增加的字段，因此无需再次声明</font>**；QuizImageQuestion在基类的基础上还需要图片链接，因此**增加一个参数“imageUrl”**

**使用该类型**：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413164632287.png" alt="image-20240413164632287" style="zoom:80%;" />

如上图所示，**<font color=red>当使用多态基类声明某个字段时，需要专门增加一个字段“$type”用来表示当前数据所属的“多态子类型”，否则无法解析数据</font>**

**打表后导出的数据文件**内容如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413182908717.png" alt="image-20240413182908717" style="zoom:67%;" />

**<font color=blue>但在游戏运行时实际获取该配置数据</font>**：

```c#
QuizQuestionDetail mQuestion1 = mTables.TbQuizQuestionDetail[1001];
QuizQuestionDetail mQuestion2 = mTables.TbQuizQuestionDetail[1002];
Debug.Log($"{mQuestion1}");
Debug.Log($"{mQuestion2}");
```

输出结果如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413165357214.png" alt="image-20240413165357214" style="zoom:80%;" />

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240413165435756.png" alt="image-20240413165435756" style="zoom:80%;" />

如上所示，"QuestionDetailText"字段中完全没有“$type”参数，该参数仅用于识别“多态bean子类型”。且当其为“QuizTextQuestion”子类型时，**<font color=blue>QuestionDetailText字段的数据中完全没有“ImageUrl”子字段，已被自动筛除</font>**

###### 在业务逻辑中解析多态bean类型数据：

通过比较各个子类型的typeId来将配置数据进行转换，具体实现如下：

```c#
QuizQuestionDetail mQuestion = mTables.TbQuizQuestionDetail[1002];
//获取目标多态字段(使用父类进行声明，后续根据typeId来转换)
QuizQuestionBase mQuizQuestionBase = mQuestion.QuestionDetailText;
switch (mQuizQuestionBase.GetTypeId()) {
    case QuizTextQuestion.__ID__:
        QuizTextQuestion mTextQ = (QuizTextQuestion)mQuizQuestionBase;
        Debug.Log($"{mTextQ}");
        break;
    case QuizImageQuestion.__ID__:
        QuizImageQuestion mImageQ = (QuizImageQuestion) mQuizQuestionBase;
        string imageUrl = mImageQ.ImageUrl;  //此时可以直接获取到该子类型特有的字段
        Debug.Log($"{mImageQ}");
        break;
}
```

##### 总结：

1.当使用多行“##var”表示各层级的“子字段”时，**<font color=red>无需增加“##type”对这些子字段的类型进行说明</font>**，**<font color=red>其会自动到“`__bean__.xlsx`”或“builtin.xml”中获取该子字段的type</font>**，“A1单元格”中第一行“##”注释行也仅用于第一行“##var”中的字段，对其他“##var”行中的“子字段”不起注释作用

2.**<font color=red>“##var”之间不必紧密相连</font>**，可以任意穿插在"A1单元格"各列之间

3.**<font color=red>“##var”之间嵌套使用</font>**，即如果“子字段”下还有“子子字段”，则可以再增加一行“##var”来展示这些“子子字段”





#### 数据校验：

##### 非默认值检验

当某个字段不能取“默认值”时，则可在**<font color=red>该字段的类型后添加“!”后缀</font>**。如

`int!`：代表该字段不能使用“int类型的默认值0”进行赋值

`int?!`：则代表不能使用“null”对该字段赋值

`array,int!`：对于“int数组”，该数组中的每个元素都不能使用“0”来赋值

`map,int!,string`：对于“map容器”，容器中每个键值对的key值不能为0

##### ref校验器

当需要引用其他表中的某个字段时，可通过**ref关键字**，**格式为**：目标字段**类型**#ref=目标字段**name**@目标表格**<font color=red>全名</font>**

如：`int#ref=id@Tb_BattleSoul_Fashion`，即代表引用“Tb_BattleSoul_Fashion”表格中的“id”字段，其类型为“int”

- 如果目标表格仅有一个主键，且为表格中第一个字段时，则可使用`int#ref=Tb_BattleSoul_Fashion`格式，此时会默认引用目标表格中的默认主键字段
- **格式中的“目标字段类型”**，其同时**<font color=red>也代表当前表格中本字段的类型</font>**

##### range校验器

该校验器主要针对int,float等类型的数据(根据取值范围的“`(),[]`”来确定**<font color=red>是否包含“边界数值”</font>**)，如：

int#range=`[1, 10]`：代表本字段取值必须在“1, 10”之间，**包含1和10**

int#range=`(1, 10]`：取值在“1，10”之间，**不包含1，包含10**

int#range=`[1,)`：取值在**[1,无穷大)之间**，左侧包含1

int#range=`(,100)`：取值在**(无穷小，100)之间**，不包含100

##### set校验器

**针对基础类型**，用来限制**其取值必须在指定集合内**(各个取值之间**<font color=red>用";"分隔</font>**)，如：

`int#set=2;3`：该字段只能在2或3中取值其中一个

`string#set=ab;cd;ef`：该字段只能取值ab,cd,ef中的一个

`DemoEnum#set=B;C`：该枚举字段取值只能在B或C

- 使用这种方式可以防止配置出错，**如枚举字段在赋值时必须是指定的几个枚举项，防止拼写错误**
- 若在容器中使用，如`list,int#set=2;3;4`，**<font color=red>该set校验器主要针对于list容器中的int元素</font>**，**<font color=red>不会对list容器本身有限制</font>**

##### size校验器

该校验器**<font color=red>仅用于容器类型</font>**，用来限制容器中元素的总个数，**支持固定个数或范围**

`list#size=4`：该list容器中**元素数量必须为4个**

`list#size=[2,5]`：该list容器中**元素数量范围在[2,5]之间(包含2和5)**

- 当使用范围size时，**<font color=blue>尽量用"[]"，不要使用"()"</font>**。**尤其是在该list有多个“#”标记时，否则容易打表识别失败**
- 当list有多个“#”时，相互之间直接添加，不要空格，如`(list#sep=;#size=[1,5]),int`





#### 外部类型映射

当需要将某个自定义的“枚举类型”或“子类型”与外部已知类型相关联时，则可以在“DataTables/**Defines/builtin.xml**”中定义

- 自定义枚举类型和子类型的声明，既可以在“`__enum__.xlsx`”和“`__beans__.xlsx`”文件中，也可以在“DataTables/Defines/builtin.xml”文件中。**<font color=red>两者作用相同，不要重复声明</font>**。但**<font color=blue>如果需要关联“外部已知类型”，则通常使用xml文件</font>**。且xml文件中声明类型时**<font color=red>使用的参数名称与excel文件有部分不同</font>**，具体可参考“官方文档->配置定义”中的xml部分：https://luban.doc.code-philosophy.com/docs/manual/defaultschemacollector
- 不论在excel还是xml中声明枚举类型或子类型，**<font color=red>其group参数通常不填</font>**。因此**<font color=red>只有在一般表格的“数据配置”中实际使用到该类型时，才会将该类型导出</font>**，即**<font color=red>生成对应语言的脚本文件</font>**
- 不论是否与外部类型映射，自定义的枚举类型以及子类型中各个参数的名称均可自由定义

##### 枚举映射

自定义的枚举类型与“外部已知类型”进行映射，是**<font color=red>通过两者的“枚举项数值”进行关联的</font>**。在进行转换时，**<font color=red>直接将“枚举项数值”强转为“外部已知类型”的某个枚举项</font>**。因此**<font color=red>两者的各个枚举项数值分布必须完全相同</font>**，否则转换失败

```xml
<enum name="EnCommonAudioType" flags="False" unique="True" comment="音频资源格式">
  <var name="mUnknown" value="0"/>
  <var name="mAcc" value="1" comment="ACC格式"/>
  <var name="mAiff" value="2"/>
  <var name="mIt" value="10"/>
  <var name="mMpeg" value="13"/>
  <mapper target="client" codeTarget="cs-bin,cs-simple-json">
       <option name="type" value="UnityEngine.AudioType"/>
  </mapper>
  <mapper target="server" codeTarget="cs-bin,cs-dotnet-json">
       <option name="type" value="CustomAudioType"/>
  </mapper>
</enum>
```

**解析**：

**1)**.在xml中可按照如上格式声明枚举类型。**其中flags,unique,comment参数如果没有，则可以不填**

**2)**.当需要与“外部已知类型”映射时，则添加“`<mapper></mapper>`”代码块，针对不同的“target,codeTarget”分别填写

- 映射的“外部已知类型”可以是某个命名空间下的类型，如“UnityEngine.AudioType”，也可以是业务逻辑中新定义的某个类型，如“CustomAudioType”。

  打表时**<font color=red>并不会先校验该“外部已知类型”是否存在，其只会根据xml中填写的“外部已知类型”生成对应的脚本文件</font>**。但**<font color=blue>若将该脚本文件放入Unity项目中进行编译，则其会自动查找该“外部已知类型”</font>**。如果项目中并没有声明“CustomAudioType”类型或查找不到“UnityEngine.AudioType”，则编译会报错。

- **如果不同target或codeTarget映射的外部类型不一致，则需要分开填写**，如上代码所示，client和server端分别映射不同的“外部类型”。但**如果映射的“外部类型”相同，则可合并填写(用","隔开)**，如

  ```xml
  <mapper target="client,server" codeTarget="cs-bin,cs-simple-json">
      <option name="type" value="CustomAudioType"/>
  </mapper>
  ```

##### 自定义类型映射

打表时会**<font color=blue>根据“数据配置”将该自定义子类型实例化</font>**。因此如果需要与“外部类型”映射，则除了类型本身外，**<font color=red>还需要提供将“自定义类型”数据转换为“外部类型”数据的方法</font>**。如：

```xml
<bean name="vector3" valueType="1" sep=",">
    <var name="x" type="float"/>
    <var name="y" type="float"/>
    <var name="z" type="float"/>
    <mapper target="client" codeTarget="cs-bin">
         <option name="type" value="UnityEngine.Vector3"/>
         <option name="constructor" value="ExternalTypeMapUtil.Vector3Map"/>
    </mapper>
    <mapper target="server" codeTarget="cs-dotnet-json">
         <option name="type" value="System.Numerics.Vector3"/>
         <option name="constructor" value="ExternalTypeMapUtil.Vector3Map"/>
    </mapper>
</bean>
```

以上代码中，“ExternalTypeMapUtil.Vector3Map”代表将“自定义的vector3类型数据”转换成“外部已知类型UnityEngine.Vector3数据”的方法，并使用`<option name="constructor" value="xxx"/>`格式进行配置

- **<font color=red>打表时不会校验该“外部已知类型”以及用于转换的“constructor”方法</font>**，其会自动根据“Luban系统的生成模板”导出对应的脚本文件。**<font color=red>只有将该脚本文件放入对应的项目时才会编译</font>**以检测该类型和方法

- 用于映射的外部类型可以是某个现有类型，**<font color=blue>也可以是业务逻辑自定义的新类型</font>**。

- **<font color=red>用于转换的“constructor”方法必须自主实现，否则项目编译时必然失败</font>font>**。

  ```c#
  public class ExternalTypeMapUtil
  {
      public static UnityEngine.Vector3 Vector3Map(cfg.vector3 v) {
          return new UnityEngine.Vector3(v.X, v.Y, v.Z);
      }
  }
  ```



































