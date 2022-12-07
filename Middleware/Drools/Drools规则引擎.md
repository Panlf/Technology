# Drools规则引擎

用户信息合法性检查

- 1、硬编码实现业务规则难以维护
- 2、硬编码实现业务规则难以应对变化
- 3、业务规则发生变化需要修改代码，重启服务后才能生效
	

可以使用规则引擎，解决上述问题
	
## 概述
规则引擎，全称业务规则管理系统。规则引擎的主要思想是将应用程序中的业务决策部分分离出来，并使用预定义的语义模板编写业务决策(业务规则)，由用户或开发这在需要时进行配置、管理。

目前市面上具体的规则引擎产品有:drools、VisualRules、iLog等

规则引擎实现了将业务决策从应用程序代码中分离出来，接收数据输入，解释业务规则，并根据业务规则做出业务决策。规则引擎其实就是一个输入输出平台。
	
## 优势
- 1、业务规则与系统代码分离，实现业务规则的集中管理
- 2、在不重启服务的情况下可随时对业务规则进行扩展和维护
- 3、可以动态修改业务规则，从而快速响应需求变更
- 4、规则引擎是相对独立的，只关心业务规则，使得业务分析人员也可以参与编辑、维护系统的业务规则
- 5、减少了硬编码业务规则的成本和风险
- 6、使用规则引擎提供的规则偏辑工具，使复杂的业务规则实现变得的简单
	
## 规则引擎应用场景
对于一些存在比较复杂的业务规则并且业务规则会频繁变动的系统比较适合使用规则引擎

- 1、风险控制系统 - 风险贷款、风险评估
- 2、反欺诈项目 - 银行贷款、征信验证
- 3、决策平台系统 - 财务计算
- 4、促销平台系统 -  满减、打折、加价购
	
## drools Api开发步骤
```		
获取kieServices - 获取kieContainer - kieSession - Insert fact - 触发规则 - 关闭kieSession
```

## 规则引擎构成
drools规则引擎由以下三部分组成
-  Working Memory 工作内存
-  Rule Base 规则库
-  Inference Engine 推理引擎

其中Inference Engine 推理引擎又包括
-  Pattern Matcher 匹配器
-  Agenda 议程
-  Execution Engine 执行引擎

### Working Memory 工作内存
drools规则引擎会从Working Memory中获取数据并和规则文件中定义的规则进行模式匹配，所以我们开发的应用程序只需要将我们的数据插入到Working Memory中即可。

### Fact 事实 
是指在drools规则应用中，将一个普通的JavaBean插入到Working Memory后的对象就是Fact对象。

### Rule Base 规则库	
我们在规则文件中定义的规则都会被加载到规则库中。

### Pattern Matcher 匹配器 
将Rule Base 中的所有规则与Working Memory中的Fact对象进行模式匹配，匹配成功的规则将被激活并放入Agenda中。

### Agenda 议程 
用于存放通过匹配器进行模式匹配后被激活的规则。

### Execution Engine 执行引擎
执行Agenda中被激活的规则。
	
## 规则引擎执行过程
```
a	将初始数据输入至工作内存中
b	使用Pattern Matcher将规则库中的规则和数据比较
c	如果执行的规则存在冲突，即激活多个规则，将冲突的规则放入冲突集合
d	解决冲突，将激活的规则按顺序放入Agenda
e	执行Agenda中的规则，重复步骤b至e，直达执行完毕Agenda中所有规则
```
## Drools基础语法

### 规则文件构成
通常规则文件以drl为后缀名，即drools rool language。一般还支持Excel文件类型。
	
- package		包名 只限于逻辑上的管理 同一个包名下的查询或者函数可以直接调用
- import		用于导入类或者静态方法
- global		全局变量
- function	自定义函数
- query		查询
- rule end  规则体
	
### 规则体语法结构
```
rule "ruleName"
attributes
when
	LHS
then
	RHS
end			
```
- rule：关键字，表示规则开始，参数为规则的唯一名称。
- attributes：规则属性，是rule与when之间的参数，为可选项。
- when：关键字，后面跟规则的条件部分。
- LHS(Left Hand Side)：是规则的条件部分的通用名称。它由零个或多个条件元素组成。如果LHS为空，则它将被视为始终为true的条件元素。
- then：关键字，后面跟规则的结果部分。
- RHS(Right Hand Side)：(是规则的后果或行动部分的通用名称。
- end：关键字，表示一个规则结束。
	
### 注释
单行用//  多行用/*...*/
	
### Pattern模式匹配
pattern的语法结构为: 绑定变量名:Object(Field约束)
其中绑定变量名可以省略，通常绑定变量名的命名一般建议以$开始
	
绑定变量既可以用在对象上，也可以用在对象的属性上
	
LHS部分还可以定于多个pattern，多个pattern之间可以使用and或者or连接，也可以不写，默认为and
	
### 比较操作符
```
			>
			<
			>=
			<=
			==
			!=
			contains		检查一个Fact对象的某个属性值是否包含一个指定的对象值
			not contains	检查一个Fact对象的某个属性值是否不包含一个指定的对象值
			memberOf		判断一个Fact对象的某个属性值是否在一个或多个集合中
			not memberOf    判断一个Fact对象的某个属性值是否不在一个或多个集合中
			matches			判断一个Fact对象的属性是否与提供的标准的Java正则表达式进行匹配
			not matches     判断一个Fact对象的属性是否与提供的标准的Java正则表达式进行匹配
```

### 语法结构
```
Object(Field[Collection/Arrray] contains value)
Object(Field[Collection/Arrray] not contains value)

Object(field memberOf value[Collection/Arrray])
Object(field not memberOf value[Collection/Arrray])

Object(field matches "正则表达式")
Object(field not matches "正则表达式")
```

## 关键字
硬关键字和软关键字
true false null 硬关键字 不能使用
	
## Drools内置方法
- update 	更新工作内存中的数据，并让相关的规则重新匹配
- insert  新增工作内存中的数据，并让相关的规则重新匹配
- retract 删除工作内存中的数据，并让相关的规则重新匹配
	
## 规则属性
### 语法
```
		rule "ruleName"
				attributes
				when
					LHS
				then
					RHS
			end
```
### 属性解释

- salience 指定规则执行优先级
- dialect 指定规则使用的语言类型，取值为java和mvel
- enabled 指定规则是否启用
- date-effective 指定规则生效时间
- date-expires 指定规则失效时间
- activation-group 激活分组，具有相同分组名称的规则只能有一个规则触发
- agenda-group 议程分组，只有获取焦点的组中的规则才有可能触发
- timer 定时器，指定规则触发的时间
- auto-focus 自动获取焦点，一般结合agenda-group一起使用
- no-loop 防止死循环
- enabled false	用于指定当前规则是否启用，取值为true或者false 默认值是true

### dialect
指定规则使用的语言类型，取值为java和mvel，默认为Java
mvel 是一种基于Java语法的表达式语言

mvel像正则表达式一样，由直接支持集合、数组和字符串匹配的操作符
mvel还提供了用来配置和构造字符串的模板语言
mvel表达式内容包括属性表达式，布尔表达式，方法调用，变量赋值，函数定义等
	
### salience 指定规则执行优先级
取值类型Integer。数值越大越优先执行，每个规则都有一个默认的执行顺序，如果不设置salience属性，规则体的执行顺序由上到下
	
### no-loop 防止死循环
当规则通过update之类函数修改Fact对象时，可能使当前规则再次被激活从而导致死循环。取值为Boolean，默认值为false
	
### activation-group 激活分组
取值类型为String，具有相同分组名称的规则只能有一个规则触发
	
### agenda-group 议程分组
属于另一种可控的规则执行方式，用户可以通过设置agenda-group来控制规则执行，只有获取焦点的组中的规则才有可能触发在Java代码中指定分组的焦点
	
### auto-focus 自动获取焦点
取值类型为Boolean，默认值false，一般结合agenda-group一起使用，当一个议程分组未获得焦点时，可以设置auto-focus属性来控制当前所属组自动获取焦点 特别注意所属组
	
### timer	
可以通过定时器的方式指定规则执行的时间，使用方式两种
```
			方法一 timer(int:<initial delay><repeat interval>?)
				遵循java.util.Timer对象的使用方式	第一个参数表示几秒后执行，第二个参数表示每隔几秒执行一次
				第二个参数可选
			方法二	time(cron:<cron expression>)
```

### date-effective 指定规则生效时间
即只有当前系统时间大于等于设置的时间或者日期规则才有可能触发。默认日期格式为dd-MMM-yyyy 用户也可以自定义日期格式
```
需要优先设置格式
// 设置日期格式
 System.setProperty("drools.dateformat","yyyy-MM-dd HH:mm");
KieServices kieServices = KieServices.Factory.get();
```

### date-expires 指定规则失效时间
与上述类似


## Drools高级语法
- package 包名 只限于逻辑上的管理 同一个包名下的查询h或者函数可以直接调用
- import 用于导入类或者静态方法
- global 全局变量
- function 自定义函数
- query 查询
- rule end 规则体

### global 全局变量
global关键字用于在规则文件中定义全局变量，它可以让应用程序的对象在规则文件中能够被访问。可以用来为规则文件提供数据或服务。

```
语法结构为：global 对象类型 对象名称
```

在使用global定义的全局变量时有两点需要注意：

- 1、如果对象类型为包装类型时，在一个规则中改变了global的值，那么只针对当前规则有效，对其他规则中的global不会有影响。可以理解为它是当前规则代码中的global副本，规则内部修改不会影响全局的使用。
- 2、如果对象类型为集合类型或JavaBean时，在一个规则中改变了global的值，对java代码和所有规则都有效。

### query查询

query提供一种查询working memory中符合约束条件的Fact对象的简单方法。它仅包含规则文件中的LHS部分，不用指定when和then部分并且以end结束。

```
query 查询的名称
	LHS
end
```

### function函数

可以在规则体中调用定义的函数

### 函数结构体
```
function 返回值类型 函数名(可选函数){
	//逻辑代码
}
```

## LHS增强
LHS是介于when和then之间的部分，主要用于模式匹配，只有匹配结果为true时，才会触发RHS部分的执行

### 复合值限制 in/not in
```
Object(field in (比较值1,比较值2...))
$s:Student(name in ("J","Y"))
$s:Student(name not in ("J","Y"))
```

### 条件元素eval
```
eval用于规则体的LHS部分，并返回一个Boolean类型的值。

eval(表达式)

eval(true)
eval(false)
eval(1 == 1)
```

### 条件元素not
```
not用于判断Working Memory中是否存在某个Fact对象，如果不存在则返回true，如果存在则返回false

not Object(可选属性约束)

not Student()
not Student(age < 10)
```

### 条件元素exists
```
exists的作用与not相反，用于判断Working Memory中是否存在某个Fact对象，如果存在则返回true，如果不存在则返回false

exists Student()

exists Student(age < 10 && name != null)
```

### 规则的继承
规则之间可以使用extends 关键字进行规则部分的继承

## RHS加强
RHS部分是规则体的重要组成部分，当LHS部分的条件匹配成功后，对应的RHS部分就会触发执行。一般在RHS部分中需要业务处理

在RHS部分Drools为我们提供了一个内置对象，名称就是drools

1、halt
立即终止后面所有的规则

2、getWorkingMemory
返回工作内存对象

3、getRule
返回当前规则

## 规则文件编码规范
- 1、所有规则文件应统一放在一个规定的文件夹中
- 2、书写的每个规则应尽量加上注释
- 3、同一个类型的对象尽量放在一个规则文件中
- 4、规则结果部分(RHS)尽量不要有条件语句
- 5、每个规则最好加上salience，明确执行顺序
- 6、Drools默认dialect为"Java",尽量避免使用dialect "mvel"

## WorkBatch

WorkBatch是KIE组件中的元素，也称为KIE-WB，是Drools与JBPM-WB的结合体。他是可视化规则编辑器

## 决策表