# NCC开发

## 股权维护

## 外系统数据集成

### 	外系统档案注册

### 	映射维护

### 	导入任务模板设置

#### 		返回数据导入和凭证导入的数据的格式

##### 		Action层

​				OutSysTask0701TaskTreeQueryAction.java   --- 左侧显示树的格式  0 全部  1 数据导入  2 凭证集成

​				QueryVouchImportAction.java

​				SaveVouchImportAction.java

​				DeleteVouchImportAction.java

##### 				Service层

​				VouchService.java

​				getVouchImports()---返回的数据类型  VouchImportVO[]

​				getVouchImports(String pk_voucher)  --- 返回的数据类型 DTORowsData<Object, VouchImportDTO>

​				insertVouchImport(VouchImportVO vo)   ---  插入模板

​				deleteVouchImport(String pk_voucher)   ---  删除模板

​				updateVouchImport(VouchImportVO vo)  --- 更新模板

### 	数据导入

##### 		Action层

​			OutSysTask0801TaskTreeToExecListAction   --- 导入模板树选中显示导入任务列表  tb_sysimptask主表           tb_imptask_p导入任务参数表

​			OutSysTask0802ExecTaskToLogListAction     ---导入任务选中显示日志列表   tb_imptasklog                   

### 	凭证集成

##### 		Action层

​			OutSysTask0701TaskTreeQueryAction.java   --- 左侧显示树的格式  0 全部  1 数据导入  2 凭证集成

​			ExecutVouchTaskAction.java

​			QueryVouchImportLogAction.java

​			QueryImportSitAction.java

​			DeleteVouchLogAction.java

##### 		Service层

​			VouchService.java

​		   executTask(Map<String, Object> map)   ---  凭证集成的执行

​		   deleteLastData(String pk_vouchImport)  ---  删除上次导入的数据

​		   queryVouchImportLog(String pk_vouchImport)  --- 查询指定模板下的导入日志

​		   deleteVouchImportLogByPk(String[] pks)   ---  根据选中日志，删除日志

​		   deleteVouchImportLog(String pk)  ---  根据模板删除日志

  		 queryImportData(String pk_vouchLog)  --- 查询导入的数据

##### 		DB层

​			TB_VOUCHIMPORT_LOG 日志表，TB_VOUCHERIMPORT 凭证模板表，

​			TB_VOUCH_SPOON 假数据(POC0)，TB_VOUCH_HEAD 凭证的主表，TB_VOUCH_BODY 凭证的子表

##### 		设计

​			 1 从 凭证模板表 加载 模板数据--需要的是view字段

​			 2 查询 改模板下的日志

​			 3  执行模板，利用kettle将poc0的假数据根据字段匹配，分别导入凭证的主表和字表，并去org_orgs表中查取对应主体下币种，填充到body表中，

从结果集Result获取插入的数据（getRows），将插入的数据的主表的pk和字表的pk以逗号分隔的形式分别存入数据库，同时记录导入数据的情况，插入操作得到日志表的pk，更新凭证的head表，将日志的pk插入到这批数据中---在查看导入数据的时候，将日志的PK传给凭证维护的接口

写日志，写个屁日志，直接读结果集 result----getRows ---> 数据行

​			4  删除日志，根据选中日志，或者模板，删除日志

​			5  删除上次导入的数据，根据模板的PK查取日志表中最新的一条数据，取出主表的pks，字表的pks，执行删除操作

​            6  查询导入的数据，将日志的PK传给凭证维护，跳转到凭证维护页面

## 维度切片的表达式解析

#### 		需求：

##### 		 	   1  维度切片解析

##### 				2  表达式解析和执行

​			A.1001 = A.1002 + A.1003*0.6 -> new Find('A.1001').set(Find('A.1002').mul(0.6).add('A.1003')) 这是

<img src="C:\Users\Administrator\Desktop\葵花宝典\表情包\表达式解析.jpg" alt="表达式解析" style="zoom:33%;" />			<img src="C:\Users\Administrator\Desktop\葵花宝典\表情包\维度切片.jpg" alt="维度切片" style="zoom:33%;" />

####        解决方式

##### 			1 维度切片   

​				给后台传入   A.1002#C2.[C2,N]#E.S1#I.[[ICP NONE]]    后台解析 

```
{
    "Account":[
        "1002"
    ],
    "Entity":[
        "S1"
    ],
    "Icp":[
        "[ICP None]"
    ],
    "C2":[
        "C2",
        "N"
    ]
}
```

​	维度切片代码：Find.java

```java
public Find(String str) throws BusinessException {
		this(getMap(str,DimServiceGetter.getDimManager()));
	}
	
	public Find(String cubeCode,String str) throws BusinessException {
		this(cubeCode, getMap(str,DimServiceGetter.getDimManager()));
	}
	
	public static Map<String, List<String>> getMap(String findDefStr,IDimManager dimManager) throws BusinessException {
		Map<String, List<String>> map = new HashMap<String, List<String>>();
		String[] t  = findDefStr.split("#");
		for (String temp : t) {
			List<String> list = new ArrayList<>();
			String[] split = temp.split("\\.");
			if (split[1].contains("[") && split[1].contains(",") && !split[1].contains("!")) {
				String a = (String) split[1].subSequence(1, split[1].length() - 1);
				list = Arrays.asList(a.split("\\,"));
			} else if (split[1].contains("!")) {
				String a = (String) split[1].subSequence(2, split[1].length() - 1);
				list = Arrays.asList(a.split("\\,"));
			} else {
				list.add(split[1]);
			}
			String dimlevel = dimManager.getDimLevelByBusiCode(split[0]).getObjCode();
			map.put(dimlevel, list);
		}
		return map;
	}
```

##### 		2. 表达式解析

   		思路：对于一个表达式每一个元素都是一个整体，那么可以用a,b,c...来表示每一个元素

```
要解析的表达式：A.1001=((A.1002#C2.[C2,N]#E.S1+3)+A.1003+A.1003)*(6+1+(5+3)) 
表达式的元素：[A.1001, A.1002#C2.[C2,N]#E.S1, 3, A.1003, A.1003, 6, 1, 5, 3] --- getSubUtil(s)
运算符的顺序：[=((, +, )+, +, )*(, +, +(, +, ))]  --- getOpOrder(s)
解析后的表达式：a=((b+c)+d+e)*(f+g+(h+i))	
```



```java
	private static List<String> getSubUtil(String s) {
		// 消除所有的空格
		String temp = s.replaceAll(" +", "");
		temp = temp.replaceAll("\\(", "");
		temp = temp.replaceAll("\\)", "");
		// 截取四则运算符中间的内容
		String rgex = "(.+?(?=\\=))" + "|" + "((?<=\\=).+?(?=[\\+|)|\\-|\\*|\\/]))" + "|" + "((?<=[\\+|\\-|\\*|\\/]).+?(?=[\\+|\\-|\\*|\\/]))" + "|"
				+ "((?<=[\\+|\\-|\\*|\\/]).+)";
		// + ".*";
		List<String> list = new ArrayList<String>();
		Pattern pattern = Pattern.compile(rgex);// 匹配的模式
		Matcher m = pattern.matcher(temp);
		while (m.find()) {
			list.add(m.group(0));
		}
		System.out.println("表达式的元素：" + list);
		return list;
	}

	private static List<String> getOpOrder(String s) {
		Queue<String> queue = new LinkedList<String>();

		List<String> opList = new ArrayList<String>();
		Map<String, Integer> map = new HashMap<String, Integer>();
		map.put("=", 0);
		map.put(")", 0);
		map.put("(", 0);
		map.put("+", 0);
		map.put("-", 0);
		map.put("*", 0);
		map.put("/", 0);
		StringBuilder sb = new StringBuilder();
		// 循环字符串
		for (Character c : s.toCharArray()) {
			String op = String.valueOf(c);
			// 将运算符put到队列中，遇到非运算符，队列的元素全部出队列
			if (map.containsKey(op)) {
				queue.add(op);
			} else {
				while (!queue.isEmpty())
					sb.append(queue.poll());
				if (!String.valueOf(sb).equals(""))
					opList.add(String.valueOf(sb));
				sb = new StringBuilder();
			}
		}
		// 检查边界
		while (!queue.isEmpty())
			sb.append(queue.poll());
		if (!String.valueOf(sb).equals(""))
			opList.add(String.valueOf(sb));

		System.out.println("运算符的顺序：" + opList);
		return opList;
	}
```

#####        3. 表达式人的执行  ---  Antlr

​	    Antlr (ANother Tool for Language Recognition) 是一个强大的跨语言语法解析器，可以用来读取、处理、执行或翻译结构化文本或二进制文件。它被广泛用来构建语言，工具和框架。Antlr可以从语法上来生成一个可以构建和遍历解析树的解析器。



```
基本概念
1.抽象语法树 (Abstract Syntax Tree,AST) 抽象语法树是源代码结构的一种抽象表示，它以树的形状表示语言的语法结构。抽象语法树一般可以用来进行代码语法的检查，代码风格的检查，代码的格式化，代码的高亮，代码的错误提示以及代码的自动补全等等。
2.语法解析器 (Parser) 语法解析器通常作为编译器或解释器出现。它的作用是进行语法检查，并构建由输入单词(Token)组成的数据结构(即抽象语法树)。语法解析器通常使用词法分析器(Lexer)从输入字符流中分离出一个个的单词(Token)，并将单词(Token)流作为其输入。实际开发中，语法解析器可以手工编写，也可以使用工具自动生成。
3.词法分析器 (Lexer) 词法分析是指在计算机科学中，将字符序列转换为单词(Token)的过程。执行词法分析的程序便称为词法分析器。词法分析器(Lexer)一般是用来供语法解析器(Parser)调用的。
```

   