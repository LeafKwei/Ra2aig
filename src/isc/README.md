# 1.简介

ISC(Initialization-Style Command，初始化风格命令)是一种使用`.ini`文件语法风格来组织命令的一种命令风格。本文档用于说明此命令风格的语法和基本实现。



# 2.ISC的生效时机

## 2.1.ISC功能概述

ISC解析器的功能可大致分为两个部分：ini解析器+isc转换器。其中，ini解析器负责按照传统ini语法解析文件内容，isc转换器则负责将这些内容在适当的时机转换为isc内容。我们以下方的例子为例：

```ini
[User]
name = alice
password = "123456"
```

在一开始，ini解析器会负责将这段文本按传统ini语法解析为ini配置内容，包括一个section名称`User`和两个键值对`name`、 `alice`和`password` 、`"123456"`，用户可通过调用`GetString`方法来获取指定的键对应的字符串值，例如当用户调用`GetString("name")`时，可以得到一个字符串`alice`。接着我们再来看isc转换器的生效时机，isc转换器只有在用户显式调用包括`GetType`、`GetLong`，`GetDouble`，`GetBool`，`GetStringArray`，`GetLongArray`，`GetDoubleArray`，`GetBoolArray`的其中之一的方法时才会生效，此时它会按照ISC语法将ini格式的字符串值转换为ISC格式的具有类型的值并进行缓存(字符串值仍会保留)。