#### 1.从pb文件中读取数据：

![image-20220705210742870](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705210742870.png)

![image-20220705210801184](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220705210801184.png)



**2.json文件中读取的技能数据在使用时防止篡改原数据，可以深拷贝一份：DeepCopy**

![image-20220708161144103](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220708161144103.png)

#### 

#### 3.获取随机数值： 

```
Contexts.GetRandom().Next(0, nodeData.MonsterIds.Count);
```

![image-20220713210012703](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220713210012703.png)



#### 4.枚举类型与int数值之间的实际使用方式：

**将枚举转换成int数值的形式：**

![image-20220714114223346](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220714114223346.png)

![image-20220714114236668](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220714114236668.png)

以上“conf.MonsterType”为int数值，这里将枚举使用“int”强制转换



**将普通的int数值转换成枚举的形式：**

![image-20220714114607459](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220714114607459.png)

以上“AddMonsterType”的方法体内参数为“枚举类型MonsterType”，因此需要将“conf.MonsterType ” —— int数值，<u>**直接使用“MonsterType”转换成枚举类型即可**</u>

![image-20220714114810612](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220714114810612.png)

