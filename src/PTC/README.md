# PTC语法简介

PTC(Pure Text Configuration，纯文本配置)是一种语法与`*.ini`文件高度相似的键值对型配置文件。以下是一个PTC配置文件的示例：

```ini
Value1 with key.
Value2 with key.
IamKey = IamValue ///I am description.
IamKey2 =   ///Will get an empty string as the value.

[UserInfo]
name = Cissia
job = police
address = "new eridu"

[Local]
language = zh_cn
```

首先来看如下内容：

```ini
[UserInfo]
name = Cissia
job = police
address = "new eridu"
```

`[UserInfo]`被称为段头部(Segment head)，`UserInfo`为段名。`name = Cissia`被称为配置对(`Pair`)，其中`name`是键，`Cissia`是值，PTC会自动裁剪掉键和值开头和末尾的空格。

在当前段头部到下一个段头部或者文件末尾之间的配置对都属于当前段，例如在本例中，`[UserInfo]`到`[Local]`之间的配置对都属于`[UserInfo]`。PTC解析器会将一个段中每个配置对都解析为一个`Pair`对象，而最终将这些`Pair`对象保存到一个`PairSegment`对象中。

PTC支持无头配置，即在文件最开始，可以允许在不声明段头的情况下直接编写配置对，这些配置对将会被PTC解析器解析到一个单独的无头`PairSegment`中。除此之外，PTC还支持无键配置，即可以直接编写值而无需给出对应的键，例如本例的`Value1 with key.`，这些值同样会被解析到一个`Pair`中，但它们的键是空的。

PTC允许使用单引号`'`或双引号`"`显式地标记出值的范围，例如本例中的`"new eridu"`。



# 类定义

## Pair

```c#
class Pair
{
	private string _key;
	private string _value;
	
    public Pair(string value)
    {
        _value = value
    }
    
	public Pair(string key, string value)
	{
		_key = key
        _value = value
	}
    
    public string Key
    {
        get{ return _key; }
        set{ _key = value; }
    }
    
    public string Value
    {
        get{ return _value != null ? _value : ""; } //Getter will be sure the value is not null.
        set{ _value =value; }
    }
}
```



## PairSegment

```c#
class PairSegment
{
    private Dictionary<string, int> _fastIndices; //Provides for mapping the key to index.
    private List<Pair> _pairs;  //Storage pairs.
        
	public PairSegment()
    {
        _fastIndices = new Dictionary<string, int>();
        _pairs = new List<Pair>;
    }
    
    //todo
}
```



## PTCFile

```c#
class PTCFile
{
    private PairSegement _headless;
    private Dictionary<string, int> _segments;
    
    public PTCFile()
    {
        _headless = new PairSegment();
        _segments = new Dictionary<string, int>();
    }
    
    //todo
}
```
