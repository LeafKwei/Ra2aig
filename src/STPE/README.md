# 1.STPE简介

STPE(Single-line Text Processing Engine，单行文本处理引擎)，是一个配置文件解析工具，以单行文本为单位进行处理。以下是其支持的配置文件格式的示例：

```stpe
/// 我是注释
fmt load="14,0; 43,0"
fmt atkBuilding="46,${BUILD}"
fmt over="49,0"

param arch="GATECH 2882 8383 9 63554; GAWEAP 9393 3939 64844"
fmt team="Taskid=${taskid}; AiScript=${aiscript}; Sucide=${sucide:yes}"
```



# 2.STPE语法

## 2.1.基本用法

### 2.1.1.



### 2.1.2.类型标签



#### 2.1.3.



### 2.1.4.注释

注释以连续的三个斜杠开头`///`，可以独占一行，也可以附加在配置行的末尾(即最后一个`"`之后)。以下都是正确的注释：

```stpe
/// 我是注释
fmt desc="Your configuration." /// 我也是注释
```



## 2.2.内置类型标签

#### param

#### fmt