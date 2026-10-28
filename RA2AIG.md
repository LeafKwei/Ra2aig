# 任务配置与脚本定义 DSL 设计草稿

> 时间：10.28 13:35

## 1. 任务定义语法

```
taskName
  Do scriptName
  [ Self allied / soviet / yuri ]
  [ Enemy allied / soviet / yuri / all ]
  [ SelfHas / EnemyHas weapName ]
  [ Level easy normal hard all ]
  [ Cover fields1 fields2 ... ]
```



### 字段说明

| 字段       | 说明                                                         |
| :--------- | :----------------------------------------------------------- |
| `taskName` | 任务名称。任务定义需要指出使用哪些兵种和国家。               |
| `Do`       | 执行哪个脚本。                                               |
| `SelfHas`  | 我方拥有指定单位时，触发此AI。                               |
| `EnemyHas` | 敌方拥有指定单位时，触发此AI。                               |
| `Self`     | 自身是哪个国家，决定 script 中的 `${self:weap}` 建筑值。     |
| `Enemy`    | 敌对国家，决定 script 中的 `${enemy:weap}` 建筑值。选 `all` 会保持其他数据不变，然后分别为每个国家生成一次。 |
| `Level`    | 此AI在哪些难度下可触发，可选多个，选all在所有难度都可触发。  |
| `Cover`    | 覆盖模板（Team 和触发器）中的字段，可以指定多个。            |

## 2. 系统模板（生成器自动读取）

这些定义都在 `./meta/ra2.tmp` 内。

```
[teamTemp]
Suicide=no
...

[aiTemp]
value=xxxxx,xxxx,xxx,${techlevel}...
```



## 3. 用户预定义（通过 `--predef` 参数给出）

### 3.1 声明与定义

```
[countries]
allied
soviet
yuri
all
```



### 3.2 任务

```
///任务
[taskName]
E1,10 ///没有`=`时，直接存为值
```



### 3.3 脚本

解析器需要提供可遍历所有只有值的配置项的功能，方便处理脚本。

```
[scriptName]
13,0
49,0
53,${self:weap} ///self: enemy:
```



### 3.4 Team 字段替换

```
[fields]
team.suicide=yes
ai.operator=eq  ///等于(直接写lt/le/eq/ge/gt)
ai.number=1     ///单位数量，直接写10进制
```



### 3.5 建筑

```
[weapName]
type=weap ///供script的${weap}使用
allied.name=GAWEAP ///国家.Name
soviet.name=NAWEAP
yuri.name=YAWEAP
allied.id=14
soviet.id=15
yuri.id=16
```
