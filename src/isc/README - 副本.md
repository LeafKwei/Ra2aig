# 1.简介

ISC(Initialization-Style Command，初始化风格命令)是一种使用`.ini`文件语法风格来组织命令的一种命令风格。本文档用于说明此命令风格的语法和基本实现。



# 2.基本语法

## 2.1.语法概览





### 传统INI语法

下方的代码展示了传统的`.ini`文件的基本语法元素：

```ini
[User]  ;我是注释
username = Alice
userpassword = abc123
```

在传统`.ini`语法中，上面的这段代码整体上被称为section，`[User]`中括号内的内容被解析为section名称`User`，`username = Alice`被解析为一个键值对，键为`username`，值为`Alice`，`userpassword = abc123`也是同理。无论是section名称、键或值，它们都被视为字符串。

在当前section名称到下一个section名称之间的键值对都会被归为当前section所有。在分号`;`后的内容被视为注释，会被解析器忽略。

(注意：此项目提供的ini解析器不支持section名称和键值对跨行，需要遵循一行一个section名称和键值对的规则；此外，也不支持只存在键的键值对存在，例如`foo=`)



### ISC语法

ISC完全兼容传统`.ini`语法，但它可以对按传统`.ini`语法解析得到的内容进行二次解析(遵循惰性求值规则，在首次使用时才进行解析)。以下是一个例子：

```ini
[Add-User] ;我是注释
username = "Alice"
userpassword = abc123
userage = 18
switched = true
options = [poweroff, poweron, "reboot"]
nameless_value1
nameless_value2
```

上述的内容在一开始仍然会以传统`.ini`语法进行解析，这意味着`"Alice"`会被整体视为一个字符串(包括引号)，而`18`、`true`等内容也不例外，同样在一开始也会被作为字符串解析，甚至连形似数组的`[poweroff, poweron, "reboot"]`也会被当做一个字符串。但是，这些内容会在二次解析中被ISC分别解析为字符串、数字、布尔值、数组等内容。最后，我们可以在末尾看到一些不属于传统`.ini`语法的内容，即`nameless_value1`、`nameless_value2`，它们是ISC单独支持的**匿名值**，不需要键的存在，在第一阶段的ini解析中，匿名值会以单独的形式保存，不会体现在原始的ini内容中。

(注：ISC的标识符和关键字是大小写不敏感的，因此`username`等同于`UserName`、`userName`、`USERNAME`。关键字在`2.3`节中列出，标识符包括section名称和键名)



## 2.2.ISC语法元素详解

在二次解析时，ISC解析器会按照`2.2.2`节之后的语法将值解析为不同的语法元素。



### 2.2.1.Section名称

与传统`.ini`一样，在中括号内`[]`的内容被解析为section名称，例如：

```ini
[Add-User] ;我是注释
username = "Alice"
userpassword = abc123
```

其中，`[Add-User]`将会解析出section名称`Add-User`。



### 2.2.2.字符串

支持裸字符串和引号界定的字符串(后称标准字符串)，具体定义为：

* 裸字符串：未使用引号界定的字符串，例如`Alice`。如果裸字符串符合关键字、数字、数组等类型的判定规则，则会按对应类型进行解析而非按字符串类型解析，如果你希望将`true`按字符串解析，那么应该使用标准字符串而非裸字符串
* 标准字符串：使用引号界定的字符串，例如`"Alice"`，其中的字符不会被解析为关键字

例如：

```ini
[Add-User]
username = "Alice"
userpassword = abc123
```

其中，标准字符串`"Alice"`会被解析为字符串`Alice`，裸字符串`abc123`会被解析为字符串`abc123`。对于标准字符串，允许在字符串内使用转义字符`\`对特殊字符串进行转义，支持转义`\n`、`\t`、`\b`等以及引号等特殊字符。裸字符串不允许出现转义字符和ISC语法中涉及的特殊符号。



### 2.2.3.整数和浮点数

支持整数和浮点数，例如：

```ini
[Factor]
inumber = 100
fnumber = 200.0
```

其中，`100`被解析为整数，`200.0`被解析为浮点数。支持如下标记：

* 支持在数字前方添加`+`、`-`符号表示正负数
* 支持`.5`、`5.`等不完整写法
* 支持使用`0x`开头的16进制写法



### 2.2.4.布尔值

支持布尔值`true`和`false`，也支持使用`yes`和`no`分别作为前两者的别名。例如：

```ini
[Flag]
openwindow = true
openbox = false
opendoor = yes
opendesk = no
```



### 2.2.5.数组

支持数组，使用中括号`[]`标识，每个元素使用逗号`,`分隔，所有元素的必须是同一类型，整数和浮点数被视为不同类型。例如：

```ini
[Yep]
names = ["Jonh", Alice, Tom]
flags = [true, false, yes, no]
inums = [10, 20, 30]
fnums = [10.1, 20.2, 30.3]
```



### 2.2.6.匿名值

支持匿名值，即直接给出值而无需键的存在，例如：

```ini
[Nameless]
"Alice"
"Rem"
Bob
yes
true
102.3
```

这些匿名值会解析并归属到section `Nameless`，出现在首个section前的匿名值将会归属于一个名称是空字符串的section。需要注意的是，匿名值不支持数组(与section语法冲突)。



### 2.2.7.空值

使用`null`代表空值，例如：

```ini
[Empty]
data = null
```

在二次解析时，`null`被视为空值标志，表示`data`没有任何值，在API接口中，获取一个空值键值对的值将会得到对应类型的零值，即空字符串`""`、`0`、`0.0`、`false`，如果尝试以数组形式获取，则会得到一个与空数组等效的结果，即`[]`。



## 2.3.ISC关键字

下表列出了ISC当前支持的关键字，用户应该避免将这些关键字作为裸字符串使用：

| 关键字 | 用途        |
| ------ | ----------- |
| null   | 空值标志    |
| true   | 表示真值    |
| false  | 表示假值    |
| yes    | 等同于true  |
| no     | 等同于false |

关键字的优先级高于裸字符串，例如`opendoor = true`中的`true`会被解析为布尔值而非字符串。



## 2.4.ISC特殊符号

下表列出了ISC已使用的特殊符号，应该避免在非转义的情况下这些符号：

| 符号 | 用途            |
| ---- | --------------- |
| "    | 标准字符串定界  |
| []   | section名称定界 |
| =    | 键值对标识      |
| +    | 正数标志        |
| -    | 负数标志        |
| .    | 浮点数标志      |



## 2.5.规则补充

### 重复定义

当出现section名称或键、键值对重复定义的情况时，将停止解析并通过接口返回错误提示信息。



### 空section名称

如果出现形似`[]`的空section名称的情况，则其中的键值对会和首个section前出现的匿名值一样，归属于一个空名称的section。



# 3.ISC命令

ISC命令基于ISC语法实现，将ISC语法的不同语法元素作为ISC命令元素。



## 3.1.命令名称

命令名称使用ISC的section名称，例如：

```ini
[Add-User]
username = Alice
```

其中，`Add-User`被视为命令名称。



## 3.2.命令参数

每一个键值对都被视为命令参数和参数值，数组类型的值可以用于表示具有多个参数值的参数。匿名值用于表示匿名参数值。例如，对于下方的传统命令行命令：

```shell
Add-User --use-tool --name alice --age 18 --option a b c n1 n2 n3
```

将其转化为ISC命令后的形式如下：

```ini
[Add-User]
use-tool = null
name = "alice"
age = 18
option = ["a", "b", "c"]
n1
n2
n3
```



# 4.实现





# 5.API使用