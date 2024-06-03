## Lua的继承和再扩展：

### 继承：

在“ImplementationContainer.lua”中使用“__DefineType(className, srcPath, baseName)”来代表继承关系：

```lua
__DefineType("UIBasePanel", "Game/UI/Base/UIBasePanel", "Container")
```

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221024105552742.png" alt="image-20221024105552742"  />



### 再扩展(方法的复用)：

在“UIBasePanel.lua”的“Initialize”方法中再使用<font color=red>**“AddComponent”方法将其他lua脚本中的方法包含进来**</font>

**实现方法：**

<font color=red>**通过“ExportMethod”的形式将其他lua脚本中部分方法导出，然后使用“AddComponent”将这些导出的方法添加到自身的lua脚本中，作为“UIBasePanel.lua”的一部分而存在**</font>，如此即实现了方法的复用。—— 这更像是继承的另外一种扩展方式

![image-20221024105311651](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221024105311651.png)![image-20221024105329572](https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20221024105329572.png)







