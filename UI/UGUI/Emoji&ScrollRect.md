[TOC]



**<font color=red>Emoji表情Unicode编码汇总</font>**：https://apps.timwhitlock.info/emoji/tables/unicode#block-6b-additional-transport-and-map-symbols



#### C#字符编码概述：

在C#中**<font color=red>默认使用“UTF-16”编码，即使用“两个字节”来表示一个字符Char</font>**，如“英文字母a-z、A-Z，标点符号”。当字符较为复杂，如部分汉字或表情等，由于两个字节已经无法表示该字符，因此可能需要“3或4个字节”才能表示该汉字或表情，此时该汉字或表情则会占用“两个字符”

而当使用“string.Length”或“string.ToCharArray()”，默认都是将其按照“字符Char”来统计，所以：

当string = "abcde"，则string.Length = 5；

当string = <img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231111134158136.png" alt="image-20231111134158136" style="zoom:80%;" />，则string.Length = 8。因为表情占用了两个字符，所以该字符串共占用8个字符

**注意**：**<font color=blue>仅有部分汉字需要使用“3到4个字节”，其他汉字“两个字节”是可以表示的，所以这些汉字仅占用1个字符Char</font>**

##### 如何准确获取字符串string中“玩家看到的字符数量”，而不是编码标准下的字符数量？

在C#中可以使用“System.Globalization.StringInfo”来获取“实际显示的字符数量”：

stringInfo.LengthInTextElements：获取实际显示出来的字符数量

stringInfo.SubstringByTextElements(int startIndex, int count)：截取该字符串中指定字符数

```c#
string str = "发b🐷的🎈😍📌的👀发生";
void Start()
{
    StringInfo strInfo = new System.Globalization.StringInfo(str);
    //获取玩家看到的字符个数
    Debug.Log(strInfo.LengthInTextElements);
    //截取指定的字符
    Debug.Log(strInfo.SubstringByTextElements(2, 3));

    //获取默认的UTF-16编码下str的char个数
    Debug.Log(str.Length);
}
```

##### 将包含“表情符号”的string按字符逐个输出，实现“打字机”的效果：

通过“StringInfo”来实现：

```c#
IEnumerator TyperFunc(string words)
{
	StringInfo wordsInfo = new StringInfo(words);
	int length = wordsInfo.LengthInTextElements;
	if (length == 0)
		yield return 0;
	else
	{
		for(int i = 0; i < length; ++i)
		{
			Debug.Log(wordsInfo.SubstringByTextElements(i, 1) + "   " + i);
			yield return new WaitForSeconds(1f);
		}
	}
}
```



#### 滑动列表ScrollRect：

当需要监听滑动事件时可使用ScrollRect.onValueChanged.AddListener(==UnityAction<Vector2> action==)，其中==Vector2代表滑动条的数值(x, y)，范围均在“[0, 1]”之间==，相当于**normalizedPosition**



















