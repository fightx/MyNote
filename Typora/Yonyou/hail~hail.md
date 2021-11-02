# #Mark

```
1. daily环境的用户和NC环境的用户是互相访问不到的
本地环境如果要看到所有用户需要：EpmCubeDefCheckUtil.java的isCheckCubeSafeClass() 改为false

2. daily远程调试端口
调试IP：172.20.58.72
调试端口：50112

3. test远程调试端口
调试IP：10.3.15.35
调试端口：50320



C:\Users\Administrator\Desktop\脚本\内部往来往来抵消.py

		String rgex = "(.+?(?=\\=))" + "|" + "((?<=\\=).+?(?=[\\+|)|\\-|\\*|\\/]))" + "|" + "((?<=[\\+|\\-|\\*|\\/]).+?(?=[\\+|\\-|\\*|\\/]))" + "|"
				+ "((?<=[\\+|\\-|\\*|\\/]).+)";
				
                # REVENUE 收入  EXPENSE 成本
                epm.Exp("TRAIL.<PC> = A.PropMember(ConsolidationAccountT,REVENUE)#TRAIL.<EO> * "+str(averRate)+"+A.PropMember(ConsolidationAccountT,REVENUE)#TRAIL.<EO>#M."+str(int(month)-1))
                epm.Exp("TRAIL.<PC> = A.PropMember(ConsolidationAccountT,EXPENSE)#TRAIL.<EO> * "+str(averRate)+"+A.PropMember(ConsolidationAccountT,EXPENSE)#TRAIL.<EO>#M."+str(int(month)-1))
                
1.数据集成流程不通-daily环境
a.导入执行时，报错
b.映射维护查询报错
c.周二演示数据集成的流程

2.数据集成需求：
a.映射关系需要预制
b导入模板需要预制-数据导入时提供模型选择的交互-需求不清楚可以询问路朝霞老师
```

