# QUANTAXIS

![quantaxis 3.0 beta](https://github.com/yutiansut/QUANTAXIS/blob/master/QUANTAXIS.jpg)

[Version]:2.5.1<br>
[Author]:yutiansut<br>
[Website]:www.yutiansut.com | http://quantaxis.yutiansut.com<br>
[Contact]:QQ 279336410<br>


QUANTAXIS是一个量化平台，使用matlab对象化编程写出  主要对接的数据接口是wind数据库<br>
*<big>[Attention]</big>*:QUANTAXIS在使用之前需要安装Wind大奖章免费应用 以及 Mysql 5.6以上版本+JDBC-MYSQL的数据库
JDBC添加完成后需要在Classpath文件中增加
```
$matlabroot/java/jar/toolbox/mysql-connector-java-5.1.7-bin.jar
```

## [QUANTAXIS](https://github.com/yutiansut/QUANTAXIS/blob/master/QUANTAXIS.m)
调用类 classdef [xx] < QUANTAXIS
----
主函数 主要是一个量化平台，负责策略实现和数据更新
```
QA=QUANTAXIS;
QA.Init()   初始化平台（数据库连接设置等）
QA.Fetch()  数据更新平台
QA.Start()  策略回测平台
```

## [QUANTAXIS FREEMARKETS](https://github.com/yutiansut/QUANTAXIS/blob/master/%2BFreeMarkets/%2BMultiDealer/FreeMarkets.m)
调用类 classdef [xx] < FreeMarkets.MultiDealer.FreeMarkets
----
```
FM=FreeMarkets;
FM.Try();  一个随机策略的金融市场
>数据将在FM.Price.History 中呈现
>FM.BidPool中是报价池

```


<big>关于quantaxis-FreeMarkets</big>
动态匹配交易池系统和自衍生随机策略系统
-------

主要考虑的问题在于这个是一个闭环的复合机制

首先 价格会导致市场上所有的交易者的预期改变，交易者的预期改变会改变他们的报价
他们的报价改变会影响成交价和成交量

而成交价和量的改变会进一步形成新的市场价格进行下一个循环


使用id去控制每一步 或者说过程的每一步

-------
>循环

询价阶段
按照round
每一个[Memeber]进行报价,以一个对象的形式按包表达出来 
结构是  Strategy-BID[id]-Price-Amount
并回调给系统

系统会以这种形式接收
[Price-Amount-Date-Varities-StrategyID-SystemTime] 并记录到[FM.BidPool.Board]中

系统在记录了报价单并形成报价池之后，会进行下一步的回调

将报价单中的数据交给一个交易函数  并形成价格
`````
notify(FM,'transaction')
`````

FM.transaction 函数对于报价池中的报价按量进行对冲并形成价格


>>To Do List
需要对于撤单以及相关的功能进一步完善

几个函数的主要功能

>TSBoard 价格形成函数
-----
得到回调的价格并记录相关
改变当前系统价格


>REPLY 客户端策略函数
-------
不同的策略会根据价格的变化改变预期（报价），并将修改后的报价提交给系统
round控制

>ASK 询价函数
------
系统根据用户的不同的报价，记录到报价池并动态匹配报价和量
如果出现可以对冲的报价就进行对冲并回调价格