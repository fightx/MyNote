# 多维概念与业务规则脚本



## 维度

业务规则中的维度均可以用维度的编码和简码来表示。

| 维度名称 | 维度编码 | 维度简码 |
| -------- | -------- | -------- |
| 科目     | ACCOUNT  | A        |
| 主体     | Entity   | E        |
| 客商     | ICP      | I        |
| 年       | YEAR     | Y        |
| 月       | MONTH    | M        |
| 币种     | CURRENCY | CUR      |
| 审计线索 | TRAIL    | V        |
| 合并范围 | SCOPE    | SP       |
| 度量维   | VIEW     | W        |
| C1       | C1       | C1       |
| C2       | C2       | C2       |
| C3       | C3       | C3       |
| C4       | C4       | C4       |
| C5       | C5       | C5       |

示例

```python
# 维度成员在多维表达式中放在"."之前
exp = 'E.1251G#I.[ICP_None]#A.[3010000,3020000,3130000,3220500]#V.<EO>#C1.[None]'
```



## 维度成员

业务规则中的维度均可以用维度的编码来表示。

```python
# 单个维度成员用字符串表示
member = '3010000'

# 多个维度成员用字符串数组表示
members = ['3010000','3020000','3130000','3220500']

# 维度成员在多维表达式中放在"."之后，多个成员包含在"[]"中，用","隔开
exp = 'E.1251G#I.[ICP_None]#A.[3010000,3020000,3130000,3220500]#V.<EO>#C1.[None]'
```



## 多维切片表达式

一个表示多维数据范围的字符串，规则如下：

1. 维度和维度成员均用编码表示，维度建议用简码
2. 多个维度之间用```#```分隔
3. 维度与维度成员之间用```.```分隔
4. 多个维度成员用```[]```将多个成员包含，成员之间用```,```隔开
5. 缺省维度（未在表达式中显式指定，但模型中存在的维度）：若该维度为主体、年、月，则通过页面上的设置自动填充，显式指定则以指定维度的为准；若为其他维度，则按照该维度下所有的末级成员作为数据查询条件

```python
# 通用格式
'维度1.维度成员1#维度2.[维度成员1,维度成员2].维度3.[None]...'

# 示例
'E.1251G#I.[ICP_None]#A.[3010000,3020000,3130000,3220500]#V.<EO>#C1.[None]'
```



## 计算表达式

多维数据切片计算表达式用于描述多个切片之间的运算关系，通过“以右定左”的算法对切片进行运算。--先平转，再查询右边表达式，再替换

```python
'V.[Elim]#A.1580000=V.[Elim]#A.1510101-V.[Elim]#A.[3010000,3020000,3130000,3220500]'
```



## 多维数据集

多维数据集用python中的列表+字典（key-value）来描述，字典中的key值为模型中所有维度的编码、VALUE和TXT（仅FindTxt结果中包含），value为维度成员编码、数值型度量值和文本型数据

示例

```python
# 多维数据集
dataSet = [{ACCOUNT: '3010000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '100.00'}, 
           {ACCOUNT: '3020000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '200.00'}]
```



# 计算上下文

计算上下文中包含某次计算中的背景维度值，用python中的字典（key-value）表示，默认的上下文中包含业务规则调试和报表工作台页面中选择的维度值（主体，年，月），用户也可以自定义背景维度。计算上下文在脚本中用预置变量```context```表示，脚本中定义其他变量请勿与之同名。

```python
# 从计算上下文中获取当前规则执行的主体
entity = context['ENTITY']
# entity = 1251G，表示当前合并执行的主体为1251G
```



## 默认计算上下文

```python
context
# context的默认内容：{ENTITY: '1251G', C1: 'V0101', YEAR: '2020', MONTH: '10'}
```



## 自定义计算上下文

```python
context['C1'] = 'BI_S'
# 自定义context的内容：{ENTITY: '1251G', C1: 'V0101', YEAR: '2020', MONTH: '10', C1: 'BI_S'}
```



# 函数

业务规则脚本只能通过预置的公共函数对多维数据进行查询和修改，公共函数需要通过EPM类的实例epm去调用，例如

```python
epm.Find('A.3310000#C1.V0105#V.[Elim]')
```

公共函数定义于公共文件中，可以被所有的脚本所使用，可以通过在公共文件中自定义函数来扩展公共函数。



## 多维数据函数

### Find
函数说明
```python
Find(tuple):
"""
获取切片对应的数据
Args:
    tuple:多维切片表达式
Returns:
    切片对应的数据集
"""

Find(cube, tuple):
"""
获取指定模型切片对应的数据
Args:
    cube:模型编码
    tuple:多维切片表达式
Returns:
    切片对应的数据集
"""
```
示例
```python
epm.Find('A.3310000#C1.V0105#V.[Elim]')
# [{ACCOUNT: '3010000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '3020000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '200.00'}]
```

### FindTxt

函数说明

```python
FindTxt(tuple):
"""
获取切片对应的文本数据,同时包含数值
Args:
    tuple:多维切片表达式
Returns:
    切片对应的数据集（只包含Base数据）
"""

FindTxt(cube, tuple):
"""
获取切片对应的文本数据,同时包含数值
Args:
    cube:模型编码
    tuple:多维切片表达式
Returns:
    切片对应的数据集（只包含Base数据）
"""
```

示例

```python
epm.FindTxt('A.3310000#C1.V0105#V.[Elim]')
# [{ACCOUNT: 'REMARK', C1: 'TAX_RECLASS_ACCS', C3: 'FT001', VALUE: '100.00', 'TXT': '3010000,3020000,3130000,3220500'}, {ACCOUNT: 'REMARK', C1: 'TAX_RECLASS_JUDGE', C3: 'FT001', VALUE: '200.00', 'TXT': '1'}]
```

### FindVal

函数说明

```python
FindVal(tuple):
"""
获取切片的值
Args:
	tuple:多维切片表达式
Returns:
    切片对应数据集value的和(float)
"""
```

示例

```python
num = epm.FindTxt('A.3310000#C1.V0105#V.[Elim]')
# num = 100.00
```

### Clear

函数说明

```python
Clear(tuple):
"""
切片数据清零
Args:
	tuple:多维切片表达式
"""

```

示例

```python
epm.Clear('A.3310000#C1.V0105#V.[Elim]')
# 无返回值
```

### Set

函数说明

```python
Set(dataSet):
"""
写入切片的数据集,覆盖原数据
Args:
	dataSet:多维数据集
"""

Set(dataSet, isSum):
"""
写入切片的数据集
Args:
	dataSet:多维数据集
    isSum:是否累加到切片上已有的数据,True:累加原数据,False:覆盖原数据
"""
```

示例

```python
# 多维数据集
dataSet = [{ACCOUNT: '3310000', C1: 'V0105', TRAIL: '[Elim]', VALUE: '100.00'}, 
           {ACCOUNT: '5930000', C1: 'V0105', TRAIL: '[Elim]', VALUE: '200.00'}]
# 写入并覆盖原数据
epm.Set(dataSet)
# 等价于
epm.Set(dataSet, False)

# 写入并累加原数据
epm.Set(dataSet, True)
```

### SetVal

函数说明

```python
SetVal(value, tuple):
"""
写入切片的数据集,覆盖原数据
Args:
	value:需要写入的值(float)
	tuple:多维切片表达式
"""

SetVal(value, tuple, isSum):
"""
写入切片的数据集,覆盖原数据
Args:
	value:需要写入的值(float)
	tuple:多维切片表达式
	isSum:是否累加到切片上已有的数据,True:累加原数据,False:覆盖原数据
"""
```

示例

```python
# 写入并覆盖原数据
epm.SetVal(200.00, 'A.3310000#C1.V0105#V.[Elim]')
# 等价于
epm.SetVal(200.00, False)

# 写入并累加原数据
epm.SetVal(200.00, True)
```

### Exp

函数说明

```python
Exp(express):
"""
执行计算表达式，保存结果
Args:
	express:计算表达式
Returns:
	多维数据集
"""

Exp(express, isSum):
"""
执行计算表达式，保存结果
Args:
	express:计算表达式
	isSum:是否累加到切片上已有的数据,True:累加原数据,False:覆盖原数据
Returns:
	多维数据集
"""
```

示例

```python
# 计算并覆盖原数据
epm.Exp('V.[Elim]#A.1580000=V.[Elim]#A.1510101-V.[Elim]#A.[3010000,3020000,3130000,3220500]')
# 返回等号左边的结果集:[{ACCOUNT: '3010000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '3020000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '200.00'}]

# 计算并累加原数据
epm.Exp('V.[Elim]#A.1580000=V.[Elim]#A.1510101-V.[Elim]#A.[3010000,3020000,3130000,3220500]', True)
# 返回等号左边的结果集:[{ACCOUNT: '3010000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '3020000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '200.00'}]
```

### Calc

函数说明

```python
Exp(express):
"""
执行计算表达式，不保存结果
Args:
	express:计算表达式
Returns:
	多维数据集
"""
```

示例

```python
# 计算不保存
epm.Exp('V.[Elim]#A.1580000=V.[Elim]#A.1510101-V.[Elim]#A.[3010000,3020000,3130000,3220500]')
# 返回等号左边的结果集:[{ACCOUNT: '3010000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '3020000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '200.00'}]
```



## 维度层级函数

### GetParent

函数说明

```python
GetParent(dim, member):
"""
获取指定维度成员的父成员
Args:
	dim:维度编码
    member:维度成员编码
Returns:
	父成员编码
"""
```

示例

```python
# 获取1371主体的上级主体
parent = epm.GetParent('ENTITY', '1371')
# parent = '1251G'
```

### GetChildren

函数说明

```python
GetChildren(dim, member):
"""
获取指定维度成员的直接下级
Args:
	dim:维度编码
    member:维度成员编码
Returns:
	维度成员编码数组
"""
```

示例

```python
# 获取1251G主体的直接下级主体
children = epm.GetChildren('ENTITY', '1251G')
# children = ['1371', '0471']
```

### GetAllChildren

函数说明

```python
GetAllChildren(dim, member):
"""
获取指定维度成员的所有下级
Args:
	dim:维度编码
    member:维度成员编码
Returns:
	维度成员编码数组
"""
```

示例

```python
# 获取0311G主体的所有下级主体
children = epm.GetAllChildren('ENTITY', '0311G')
# children = ['1251G', 1371', '0471']
```

### GetLeaves

函数说明

```python
GetLeaves(dim):
"""
获取指定维度下的所有末级成员
Args:
	dim:维度编码
    member:维度成员编码
Returns:
	维度成员编码数组
"""

GetLeaves(dim, member):
"""
获取指定维度成员下的所有末级成员
Args:
	dim:维度编码
    member:维度成员编码
Returns:
	维度成员编码数组
"""
```

示例

```python
# 获取主体维度下的所有末级主体
leaves = epm.GetAllChildren('ENTITY', '0311G')
# leaves = ['1371', '0471', '6001']

# 获取0311G主体的所有末级主体
leaves = epm.GetAllChildren('ENTITY', '0311G')
# leaves = ['1371', '0471']
```

### IsLeaf

函数说明

```python
IsLeaf(dim, member):
"""
指定维度成员是否是末级成员
Args:
	dim:维度编码
    member:维度成员编码
Returns:
	False/False
"""
```

示例

```python
# 判断主体1371是否是末级
IsLeaf('ENTITY', '1371')
# True

# 判断主体1251G是否是末级
IsLeaf('ENTITY', '1251G')
# False
```

### IsFamily

函数说明

```python
IsFamily(dim, parent, child):
"""
指定维度成员是否是父子关系
Args:
	dim:维度编码
    parent:父成员编码
    child:子成员编码
Returns:
	False/False
"""
```

示例

```python
# 判断主体1371是否是末级
IsFamily('ENTITY', '1251G', '1371')
# True

# 判断主体1251G是否是末级
IsFamily('ENTITY', '1251G', '6001')
# False
```

### GetProp

函数说明

```python
GetProp(dim, member, prop):
"""
获取指定维度成员的维度属性
Args:
	dim:维度编码
    member:维度成员编码
    prop:维度属性编码
Returns:
	维度属性值（字符串）
"""
```

示例

```python
# 获取合并主体对应的实体组织
holdingEnt = epm.GetProp('ENTITY', '1251G', 'HoldingCompany')
# holdingEnt = '1251'

# 获取科目的PlugAccount属性的值
prop = epm.GetProp('ACCOUNT', account, 'PlugAccount')
# prop = 'PlugBS_Dividend'
```



## 业务函数

### CreateVoucher

函数说明

```python
CreateVoucher(debitResult, createResult):
 """
 执行表达式，保存结果，并生成凭证
 Args:
 	debitFinds:借方计算结果集
    createFinds:贷方计算结果集
 """
```

示例

```python
# 借方数据集
debitRs = [{ACCOUNT: '3010000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '3020000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '200.00'}]

# 贷方数据集
creditRs = [{ACCOUNT: '1510101', C1: 'V0101', TRAIL: '[Elim]', VALUE: '500.00'}, {ACCOUNT: '1580000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '600.00'}]

# 生成凭证，凭证状态为已过帐
epm.CreateVoucher(debitRs, creditRs)
```



## 工具函数

### Log

函数说明

```python
Log(var):
"""
打印变量到日志
Args:
	var:需要打印变量
"""

Log(info, var):
"""
打印变量到日志，并为此次输出添加说明
Args:
	info:日志的说明
   	var:需要打印变量
"""
```

示例

```python
# 打印结果集
rs = [{ACCOUNT: '3010000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '3020000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '200.00'}]

# 直接打印
Log(rs)
# [{ACCOUNT: '3010000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '3020000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '200.00'}]

# 带说明的打印
Log('Result:', rs)
# Result:
# [{ACCOUNT: '3010000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '3020000', C1: 'V0101', TRAIL: '[Elim]', VALUE: '200.00'}]
```

### List2Str

函数说明

```python
List2Str(array):
"""
字符串数组转字符串,若数组只有一个元素,返回该元素;若数组有多个元素,元素时间用逗号分隔,结果两边加上[],常用于计算表达式的构建
Args:
	array:字符串数组
Returns:
	字符串,如1001或[1001,1002,1003]
"""
```

示例

```python
# 一个元素的数组
accounts = ['3010000']

# 一个元素以上的数组
entities = ['1371', '0471', '6001']

# 一个元素的数组转字符串
strAccounts = List2Str(accounts)
# strAccounts = '3010000'

# 一个元素以上的数组转字符串
strEntities = List2Str(entities)
# strEntities = '[1371,0471,6001]'

# 执行计算表达式
epm.Exp('V.[Elim]#A.1580000=V.[Elim]#A.' + strAccounts + '#E.' + strEntities)
```

