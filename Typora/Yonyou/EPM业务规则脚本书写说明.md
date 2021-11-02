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
exp = 'E.100#I.[ICP_None]#A.[301,302,313,322]#V.<EO>#C1.[None]'

# 维度也可以用全称表示
exp = 'ENTITY.100#ICP.[ICP_None]#ACCOUNT.[301,302,313,322]#TRAIL.<EO>#C1.[None]'
```



## 维度成员

业务规则中的维度均可以用维度的编码来表示。

```python
# 单个维度成员用字符串表示
member = '301'

# 多个维度成员用字符串数组表示
members = ['301','302','313','322']

# 维度成员在多维表达式中放在"."之后，多个成员包含在"[]"中，用","隔开
exp = 'E.100#I.[ICP_None]#A.[301,302,313,322,[None]]#V.<EO>#C1.[None]'
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
'E.100#I.[ICP_None]#A.[301,302,313,322]#V.<EO>#C1.[None]'
```



## 计算表达式

多维数据切片计算表达式用于描述多个切片之间的运算关系，通过“以右定左”的算法对切片进行运算。

```python
'V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]'
```



## 多维数据集

多维数据集用python中的列表+字典（key-value）来描述，字典中的key值为模型中所有维度的编码、VALUE和TXT（仅FindTxt结果中包含），value为维度成员编码、数值型度量值和文本型数据

示例

```python
# 多维数据集
dataSet = [{ACCOUNT: '301', C1: '101', TRAIL: '[Elim]', VALUE: '100.00'}, 
           {ACCOUNT: '302', C1: '101', TRAIL: '[Elim]', VALUE: '200.00'}]
```



# 计算上下文

计算上下文中包含某次计算中的背景维度值，用python中的字典（key-value）表示，默认的上下文中包含业务规则调试和报表工作台页面中选择的维度值（主体，年，月），用户也可以自定义背景维度。计算上下文在脚本中用预置变量```context```表示，脚本中定义其他变量请勿与之同名。

```python
# 从计算上下文中获取当前规则执行的主体
entity = context['ENTITY']
# entity = 100，表示当前合并执行的主体为100
```



## 默认计算上下文(环境变量)

```python
context
# context的默认内容：{ENTITY: '100', C1: '0101', YEAR: '2020', MONTH: '10'}
```



## 自定义计算上下文

```python
context['C1'] = 'BI_S'
# 自定义context的内容：{ENTITY: '100', YEAR: '2020', MONTH: '10'}
```



## 按照计算上下文做背景维度复用

```python
# 按照背景维度1计算
context1 = {ENTITY: '100', C1: 'V0101', YEAR: '2020', MONTH: '10'}
epm1 = EPM(context1)
epm1.Exp('V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]')

# 按照背景维度2计算
context2 = {ENTITY: '137', C1: '101', YEAR: '2020', MONTH: '10'}
epm2 = EPM(context2)
epm2.Exp('V.[Elim]#A.1580000=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]')
```



# 函数

业务规则脚本只能通过预置的公共函数对多维数据进行查询和修改，公共函数需要通过EPM类的实例epm去调用，例如

```python
epm.Find('A.331#C1.105#V.[Elim]')
```

公共函数定义于公共文件中，可以被所有的脚本所使用，可以通过在公共文件中自定义函数来扩展公共函数。



## 多维数据函数

### Find
函数说明

```
通过指定维度组合查询多维数据
```

```python
Find(tuple):
"""
获取当前选中模型切片对应的数据
Args:
    tuple:多维切片表达式
Returns:
    切片对应的数据集，类型：列表+字典
"""

Find(cube, tuple):
"""
获取指定模型切片对应的数据
Args:
    cube:模型编码
    tuple:多维切片表达式
Returns:
    切片对应的数据集，类型：列表+字典
"""
```
示例
```python
# 不指定模型取当前选中模型
epm.Find('A.331#C1.105#V.[Elim]')
# [{ACCOUNT: '331', C1: '105', c2:'[None]', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '331', C1: '105', c2:'CNY', TRAIL: '[Elim]', VALUE: '200.00'}]

# 指定模型取数
epm.Find('cube_001', 'A.3310000#C1.105#V.[Elim]')
# [{ACCOUNT: '331', C1: '105', c2:'[None]', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '331', C1: '105', c2:'CNY', TRAIL: '[Elim]', VALUE: '200.00'}]
```

### FindTxt

函数说明

```
通过指定维度组合查询多维文本数据
```

```python
FindTxt(tuple):
"""
获取当前选中切片对应的文本数据,同时包含数值
Args:
    tuple:多维切片表达式
Returns:
    切片对应的数据集（只包含Base数据），类型：列表+字典
"""

FindTxt(cube, tuple):
"""
获取指定模型切片对应的文本数据,同时包含数值
Args:
    cube:模型编码
    tuple:多维切片表达式
Returns:
    切片对应的数据集（只包含Base数据），类型：列表+字典
"""
```

示例

```python
# 通过FindTxt获取配置表中的文本信息
epm.FindTxt('A.REMARK#C3.001#V.[None]')
# [{ACCOUNT: 'REMARK', C1: 'TAX_RECLASS_ACCS', C3: '001', TRAIL: '[None]', VALUE: null, 'TXT': '301,302,313,322'}, {ACCOUNT: 'REMARK', C1: 'TAX_RECLASS_JUDGE', C3: '001', TRAIL: '[None]', VALUE: null, 'TXT': '1'}]
```

### FindVal

函数说明

```
查询指定维度组合的值的总和
```

```python
FindVal(tuple):
"""
获取当前切片的值
Args:
	tuple:多维切片表达式
Returns:
    切片对应数据集value的和，类型：double
"""
```

示例

```python
num = epm.FindVal('A.331#C1.0105#V.[Elim]')
# num = 100.00
```

### FindValString

函数说明

```
获取切片的字符串类型的值
```

```python
def FindValString(tupleStr):
        """
        获取切片的字符串类型的值
        Args:
            tupleStr:切片字符串
        Returns:
            切片对应数据集value的和(float)
        """
```

示例

```python
value = epm.FindValString('A.331#C1.0105#V.[Elim]')
# value= "100"
```

### Clear

函数说明

```
清除指定维度组合的多维数据
```

```python
def Clear(tuple):
	"""
	切片数据清空
	Args:
		tuple:多维切片表达式
	"""

def Clear(data):
	"""
	数据数据集清空
	Args:
		data:数据集，类型：列表+字典
	"""
```

示例

```python
# 切片数据清空
epm.Clear('A.331#C1.V0105#V.[Elim]')
# 无返回值

# 数据数据集清空
data = [{ACCOUNT: '331', C1: '105', c2:'[None]', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '331', C1: '105', c2:'CNY', TRAIL: '[Elim]', VALUE: '200.00'}]
# 清除data中的两条数据
epm.Clear(data)
# 无返回值
```

### BatchClear

函数说明

```
批量清除多维数据
```

```python
def BatchClear(self, tuples):
	"""
	切片数据清零
	Args:
    	tuples:切片字符串的列表
	"""
```

示例

```python
list = ['A.331#C1.V0105#V.[Elim]','A.321#C1.V0105#V.[Elim]']
# 批量清除切片数据
epm.BatchClear(list)
```

### Set

函数说明

```
对指定多维组合的多维数据赋值
```

```python
def Set(dataSet):
	"""
	写入切片的数据集,覆盖原数据
	Args:
		dataSet:多维数据集，类型：列表+字典
	"""

def Set(dataSet, isSum):
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
dataSet = [{ACCOUNT: '331', C1: '105', TRAIL: '[Elim]', VALUE: '100.00'}, 
           {ACCOUNT: '593', C1: '105', TRAIL: '[Elim]', VALUE: '200.00'}]

# 写入并覆盖原数据
epm.Set(dataSet)
# 等价于
epm.Set(dataSet, False)

# 写入并累加原数据
epm.Set(dataSet, True)
```

### SetVal

函数说明

```
对指定多维组合的多维数据赋值
```

```python
def SetVal(value, tuple):
	"""
	为切片已存在的数据赋值,覆盖原数据,注意:为了保证数据量不以笛卡尔积的方式增长,本函数只能为已存在的数据赋值,不能创造新数据
	Args:
		value:需要写入的值(float)
		tuple:多维切片表达式
	"""

def SetVal(value, tuple, isSum):
	"""
	为切片已存在的数据赋值,覆盖/累加到原数据
	Args:
		value:需要写入的值(float)
		tuple:多维切片表达式
		isSum:是否累加到切片上已有的数据,True:累加原数据,False:覆盖原数据
	"""
```

示例

```python
# 写入并覆盖原数据
epm.SetVal(200.00, 'A.331#C1.105#V.[Elim]')
# 等价于
epm.SetVal(200.00, False)

# 写入并累加原数据
epm.SetVal(200.00, True)
```

### Exp

函数说明

```
多维数据计算，保存结果
```

```python
def Exp(express):
	"""
	执行计算表达式，保存结果
	Args:
		express:计算表达式
	Returns:
		多维数据集，类型：列表+字典
	"""

def Exp(express, isSum):
	"""
	执行计算表达式，保存结果
	Args:
		express:计算表达式
		isSum:是否累加到切片上已有的数据,True:累加原数据,False:覆盖原数据
	Returns:
		多维数据集，类型：列表+字典
	"""
```

示例

```python
# 计算并覆盖原数据
epm.Exp('V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]')
# 返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '200.00'}]

# 计算并累加原数据
epm.Exp('V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]', True)
# 返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '150.00'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '300.00'}]
```

### Calc

函数说明

```
多维数据计算，不保存结果
```

```python
def Calc(express, isSum):
	"""
	执行计算表达式，不保存结果
	Args:
		express:计算表达式
	Returns:
		多维数据集，类型：列表+字典
	"""

def Calc(express):
	"""
	执行计算表达式，不保存结果
	Args:
		express:计算表达式
		isSum:是否累加到切片上已有的数据,True:累加原数据,False:覆盖原数据
	Returns:
		多维数据集，类型：列表+字典
	"""
```

示例

```python
# 计算并覆盖原数据，不保存
epm.Calc('V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]')
# 返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '200.00'}]

# 计算并累加原数据，不保存
epm.Calc('V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]', True)
# 返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '150.00'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '300.00'}]
```



### ABSExp

函数说明

```
多维数据计算，结果取绝对值，保存结果
```

```python
def ABSExp(express):
	"""
	执行计算表达式，结果取绝对值，保存结果
	Args:
		express:计算表达式
	Returns:
		多维数据集，类型：列表+字典
	"""

def ABSExp(express, isSum):
	"""
	执行计算表达式，结果取绝对值，保存结果
	Args:
		express:计算表达式
		isSum:是否累加到切片上已有的数据,True:累加原数据,False:覆盖原数据
	Returns:
		多维数据集，类型：列表+字典
	"""
```

示例

```python
# EXP执行，覆盖
epm.ABSExp('V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]')
# 返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '-100.00'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '-200.00'}]

# ABSExp执行，覆盖
epm.ABSExp('V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]')
# 返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '200.00'}]

# EXP执行，累加
epm.ABSExp('V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]', True)
# 返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '-150.00'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '-300.00'}]

# ABSExp执行，累加
epm.ABSExp('V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]', True)
# 返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '150.00'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '300.00'}]
```

### BatchExp

函数说明

```
批量计算
```

```python
def BatchExp(self, *args):
 	"""
	批量执行计算表达式，
	Args:
        args[0]:计算表达式,必输
    	args[1]:是否串行
    	args[2]:保留小数位数
    	args[3]:是否绝对值
	Returns:
     	多维数据集
	"""
```

示例

```python
expressList = ['V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]+0.12345']

# 并行执行表达式列表
epm.BatchExp(expressList)
#返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '-100.12345'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '-100.12345'}

# 串行执行表达式列表
epm.BatchExp(expressList，True)
#返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '-100.12345'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '-200.12345'}

# 串行执行表达式式，并且保留指定小数位数
epm.BatchExp(expressList，True,3)
#返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '-100.123'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '-200.123'}

# 串行执行表达式，并且取绝对值
epm.BatchExp(expressList，True,True)
#返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '100.12345'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '200.12345'}

# 串行执行表达式，保留指定小数位数，并且取绝对值
epm.BatchExp(expressList，True,3，True)
#返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '100.123'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '200.123'}
```

### RoundExp

函数说明

```
多维数据计算，指定保存小数位数，保存结果
```

```python
def RoundExp(self, *args):
	"""
	执行计算表达式，保存结果
	Args:
    	args[0]:计算表达式,必输
    	args[1]:是否累加原来的数据,可选,缺省为覆盖
    	args[2]:保留小数位数
    	args[3]:是否绝对值
	Returns:
    	多维数据集
	"""
```

示例

```python
express = 'V.[Elim]#A.158=V.[Elim]#A.151-V.[Elim]#A.[301,302,313,322]+0.12345'

# 指定保留小数位数
epm.RoundExp(express,3)
#返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '-100.123'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '-200.123'}

# 累加，且指定保留小数位数
epm.RoundExp(express,True,3)
#返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '-150.123'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '-250.123'}

# 取绝对值，且指定保留小数位数
epm.RoundExp(express,3,True)
#返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '100.123'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '200.123'}

# 累加，且指定保留小数位数，且取绝对值
epm.ROundExp(express,True,3,True)
#返回等号左边的结果集:[{ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '-150.123'}, {ACCOUNT: '158', C1: '105', TRAIL: '[Elim]', VALUE: '-250.123'}
```



## 元数据函数

### GetParent

函数说明

```python
def GetParent(dim, member):
	"""
	获取指定维度成员的父成员
	Args:
		dim:维度编码
    	member:维度成员编码
	Returns:
		父成员编码，类型：字符串
	"""
```

示例

```python
# 获取130主体的上级主体
parent = epm.GetParent('ENTITY', '1301')
# parent = '130'
```

### GetChildren

函数说明

```python
def GetChildren(dim, member):
	"""
	获取指定维度成员的直接下级
	Args:
		dim:维度编码
    	member:维度成员编码
	Returns:
		维度成员编码数组，类型：字符串列表
	"""
```

示例

```python
# 获取100主体的直接下级主体
children = epm.GetChildren('ENTITY', '100')
# children = ['1001', '1002']
```

### GetAllChildren

函数说明

```python
def GetAllChildren(dim, member):
	"""
	获取指定维度成员的所有下级
	Args:
		dim:维度编码
    	member:维度成员编码
	Returns:
		维度成员编码数组，类型：字符串列表
	"""
```

示例

```python
# 获取100主体的所有下级主体
children = epm.GetAllChildren('ENTITY', '100')
# children = ['1001', 1002', '100101', '100102', '100201']
```

### GetLeaves

函数说明

```python
def GetLeaves(dim):
	"""
	获取指定维度下的所有末级成员
	Args:
		dim:维度编码
    	member:维度成员编码
	Returns:
		维度成员编码数组，类型：字符串列表
	"""

def GetLeaves(dim, member):
	"""
	获取指定维度成员下的所有末级成员
	Args:
		dim:维度编码
    	member:维度成员编码
	Returns:
		维度成员编码数组，类型：字符串列表
	"""
```

示例

```python
# 获取主体维度下的所有末级主体
leaves = epm.GetAllChildren('ENTITY')
# leaves = ['100101', '100102', '100201']

# 获取1001主体的所有末级主体
leaves = epm.GetAllChildren('ENTITY', '1001')
# leaves = ['100101', '100102']
```

### IsLeaf

函数说明

```python
def IsLeaf(dim, member):
	"""
	指定维度成员是否是末级成员
	Args:
		dim:维度编码
    	member:维度成员编码
	Returns:
		True/False
	"""
```

示例

```python
# 判断主体10010101是否是末级
IsLeaf('ENTITY', '10010101')
# True

# 判断主体100101是否是末级
IsLeaf('ENTITY', '100101')
# False
```

### IsFamily

函数说明

```python
def IsFamily(dim, parent, child):
	"""
	指定维度成员是否是父子关系
	Args:
		dim:维度编码
    	parent:父成员编码
    	child:子成员编码
	Returns:
		True/False
	"""
```

示例

```python
# 判断主体100101和主体10010101是否是父子关系
IsFamily('ENTITY', '100101', '10010101')
# True

# 判断主体100102和主体10010101是否是父子关系
IsFamily('ENTITY', '100102', '10010101')
# False
```

### GetProp

函数说明

```python
def GetProp(dim, member, prop):
	"""
	获取指定维度成员的维度属性
	Args:
		dim:维度编码
   		member:维度成员编码
    	prop:维度属性编码
	Returns:
		维度属性值，类型：字符串
	"""
```

示例

```python
# 获取合并主体对应的实体组织
holdingEnt = epm.GetProp('ENTITY', '100_virtual', 'HoldingCompany')
# holdingEnt = '100'

# 获取100101科目的PlugAccount属性的值
prop = epm.GetProp('ACCOUNT', '100101', 'PlugAccount')
# prop = 'Dividend'
```

### GetPropMember

函数说明

```python
def GetPropMember(dim, member, propValue):
	"""
	获取指定维度属性的维度成员编码集合
	Args:
		dim:维度编码
    	member:维度属性编码
    	prop:维度属性值
	Returns:
		维度成员集合，类型：字符串列表
	"""
```

示例

```python
# 获取所有COA属性为PlugAccount的科目
plugAccounts = epm.GetPropMember('ACCOUNT', 'COA', 'PlugAccount')
# plugAccounts = ['10',101', '102','103']
```

### GetPropLeaves

```python
def GetPropLeaves(dim,prop, propValue,*args):
    """
    获取指定维度属性的维度成员编码集合的所有末级成员
    Args:
         dim:维度编码
         member:维度成员编码
         member:维度属性编码
         prop:维度属性值
    Returns:
         末级成员编码数组
    """
```

示例

```python
# 获取所有COA属性为PlugAccount的末级科目
plugAccounts = epm.GetPropLeaves('ACCOUNT', 'COA', 'PlugAccount')
# plugAccounts = ['101', '102','103']
```

### GetEntityParent

函数说明

```python
def GetEntityParent(self, dim):
	"""
	获取指定主体维度成员的父成员--处理共享成员问题
	Args:
    	dim:维度编码
	Returns:
    	父成员编码
	"""
```

示例

```python
# 获取当前执行主体的父级，处理共享成员问题
epm.GetEntityParent("ENTITY")
# 0000G
```

### GetShareParentByActive

函数说明

```python
def GetShareParentByActive(self, dim, member):
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
# 获取共享主体，的active属性为Y的父级
epm.GetShareParentByActive("ENTITY","0911")
# 0091G
```

### RuleMember

函数说明

```python
def RuleMember(self, dim, *args):
	"""
	获取指定维度成员的父成员
	Args:
    	dim:维度编码
    	ruleMemberCode:规则成员编码
    	member:维度成员编码
	Returns:
    	符合规则的维度成员
	"""
```

示例

```python
# 通过规则成员的编码调用 规则成员，获取返回结果
epm.RuleMember('ENTITY',"LEAVES")
# 获取主体维度的末级成员 0541;0551;0561;0071;0081;0021;0022;0911;0912;1021;1022;1023;[None];hdx1;yhl001
epm.RuleMember('ENTITY','PARENT','0561')
# 获取 0561的直接上级 0041G 
```

### IsRelative

函数说明

```python
def IsRelative(dim, dimMember, otherDimMember):
	"""
	指定维度成员是否有血缘关系
	Args:
    	dim:维度编码
    	parent:祖父成员编码
    	child:子成员编码
	Returns:
    	False/False
	"""
```

示例

```python
# 判断主体中的 0091是否在 0000G的所有下级中
epm.IsRelative("ENTITY","0000G","0091")
# True 是, False 否
```



## 嵌入式元数据函数

元数据函数支持嵌入到Find、FindTxt、FindVal、Exp、Calc等函数中的切片中进行调用，需要去掉原元数据函数中的“Get”，放在维度之后，省略维度参数时，则取本维度，指定时则取指定维度。

示例

```python
## 维度为ENTITY，取ENTITY的直接下级，维度为ICP，指定取ENTITY的层级，取末级
epm.Exp('V.[Elim]#A.PlugAccount#I.[ICP_None]=A.101000#V.[Prop]#E.Children(1251G)#I.Leaves(ENTITY, 1251G)')
```



## 业务函数

### CreateVoucher

函数说明

```python
def CreateVoucher(debitResult, creditResult):
 	"""
 	执行表达式，保存结果，并生成凭证，调用一次生产一张凭证，包含len(debitResult)+len(creditResult)条分录
 	Args:
 		debitFinds:借方计算结果集，类型：列表+字典
    	createFinds:贷方计算结果集，类型：列表+字典
 	"""
def CreateVoucher(debitResult, creditResult,describe):
 	"""
 	执行表达式，保存结果，并生成凭证，调用一次生产一张凭证，包含len(debitResult)+len(creditResult)条分录
 	Args:
 		debitFinds:借方计算结果集，类型：列表+字典
    	createFinds:贷方计算结果集，类型：列表+字典
    	describe:对该凭证的描述
 	"""
```

示例

```python
# 借方数据集
debitRs = [{ACCOUNT: '301', C1: '101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '302', C1: '101', TRAIL: '[Elim]', VALUE: '200.00'}]

# 贷方数据集
creditRs = [{ACCOUNT: '151', C1: '101', TRAIL: '[Elim]', VALUE: '500.00'}, {ACCOUNT: '158', C1: '101', TRAIL: '[Elim]', VALUE: '600.00'}]

# 生成凭证，凭证状态为已过帐
epm.CreateVoucher(debitRs, creditRs)
# 生成一张凭证，借方200，贷方600
```



## 工具函数

### Log

函数说明

```python
def Log(var):
	"""
	打印变量到日志
	Args:
		var:需要打印变量
	"""

def Log(info, var):
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
rs = [{ACCOUNT: '3011', C1: '101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '302', C1: '101', TRAIL: '[Elim]', VALUE: '200.00'}]

# 直接打印
Log(rs)
# [{ACCOUNT: '301', C1: '101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '302', C1: '101', TRAIL: '[Elim]', VALUE: '200.00'}]

# 带说明的打印
Log('Result:', rs)
# Result:
# [{ACCOUNT: '301', C1: '101', TRAIL: '[Elim]', VALUE: '100.00'}, {ACCOUNT: '302', C1: '101', TRAIL: '[Elim]', VALUE: '200.00'}]
```

### OpenLog

函数说明

```
打开自动日志开关
```

```python
def OpenLog(self):
"""
  打开自动日志开关
"""
```

### CloseLog

函数说明

```python
关闭日志开关
```

```python
def CloseLog(self):
"""
   关闭自动日志开关
"""
```

### Double2Str

函数说明

```
double类型的数据转为String类型，解决科学计算法的问题
```

```python
def DoubleToStr(self, value):
     """
     获取切片的值
     Args:
         value:doule类型的数据
     Returns:
         字符类型的value
     """
```

示例

```python
value = epm.DoubleToStr(0.0002)
# value = "0.0002"
```

### List2Str

函数说明

```python
def List2Str(array):
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
accounts = ['301']

# 一个元素以上的数组
entities = ['137', '047', '600']

# 一个元素的数组转字符串
strAccounts = List2Str(accounts)
# strAccounts = '301'

# 一个元素以上的数组转字符串
strEntities = List2Str(entities)
# strEntities = '[137,047,600]'

# 执行计算表达式时可以使用
epm.Exp('V.[Elim]#A.158=V.[Elim]#A.' + strAccounts + '#E.' + strEntities)
```

