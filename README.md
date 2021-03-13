# 如何用计算机组成原理，数字电路的知识仿真一个CPU？



![image-20210314003812402](https://i.loli.net/2021/03/14/MQBJ62la5RkzxIL.png)

为了简单，此CPU采用哈佛架构(非冯诺伊曼)
主要实现 PC,RAM，ROM，Register，等等的8位计算器，最终会开发出此CPU的汇编程序

# why I Do this

好玩！

# 指令参考MIPS

31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0

此时CPU的第二个输入设置为1

|      功能      | 位置  |               具体描述               | 汇编          |
| :------------: | :---: | :----------------------------------: | ------------- |
| 寄存器读地址线 | 25-21 |   选择一个寄存器组输出到ALU或者RAM   | 原寄存器区rs  |
| 寄存器写地址线 | 20-16 | 选择一个寄存器写入来自ALU或RAM的数据 | 目的寄存器rt  |
|  ALU控制信号   |  5-0  | 控制ALU的模式，参考MIPS指令集留出6位 | 功能码区funct |
|  寄存器写允许  |  31   |         允许向寄存器写入数据         |               |
|  RAM、ALU选择  |  30   | 选择RAM或ALU的数据进入寄存器的数据线 | 高6位操作码区 |
|  RAM读写控制   |  29   |           RAM读写控制信号            |               |
|     立即数     | 15-6  |                立即数                | im            |

暂时忽略RAM的地址线

![image-20210313174502475](https://i.loli.net/2021/03/13/kAZ97BNV15rLORH.png)

若要将r0的值+1，然后存入r1。则可以得到以下的指令

100000 00000 00001 0000000001 000110

然后我们只需要将以下数据接入到相应的位置，就能自动运算了！


# 解决蛋鸡问题

在CPU启动时，我们如何将数据加载进寄存器？
我们可能会想，从RAM中读取，那么RAM中的数据从哪里来？RAM中的数据要从寄存器中来（由于我们采用哈佛架构，不考虑RAM的数据从ROM中读入）这样追问下去，就产生了蛋鸡问题。
+ 想一下可能的解决方案
立即数直接与CPU相连，那么是否可以通过CPU的加法将数据保存进寄存器呢？
答案是肯定的，但是此时，寄存器的值没有被处始化，另一个加数从何而来.....

最后为了解决蛋鸡问题，我们决定直接创造蛋，即直接将一个寄存器r7从数据线剥离，去掉读写控制线，并且将clear置为1，创造一个0

然后，向寄存器存数据就变成了AUL的加法操作，零加任何数都没影响

所以这个问题的解决方案是：创造鸡蛋 :>

# 如何读取RAM

从上面的简易CPU到现在，我们并没有用到RAM，并且RAM的地址线和数据线没有连接，也就是说上一代CPU只能操作寄存器，那么思考一下如何读取RAM？

+ 数据由谁提供？

​		首先，按照管理，任何数据的来往，都是ALU通过寄存器转存去存入或者读入RAM里面的数据，因此，寄存器的输出应该与RAM的数据线相连 （在上一代CPU中，RAM的输出已经要通过多路选择器和CPU的输出，一起连接至了寄存器的数据线）

+ 那么地址由谁提供？

​		那么在现有的硬件情况下，如果直接用立即数当作地址，与RAM地址线相连，看起来已经解决了RAM的寻址问题，但是这样好么？

​		回想一下《x86_从实模式到保护模式》,这岂不是就是实模式么，这样的操作，会导致很不利于读写连续的地址空间，并且借鉴该书中8086处理器的解决方式，地址由CPU计算产生，[段地址:偏移地址]的模式去访问RAM，那么就很好的解决了连续内存空间访问的问题。

因此，立即数不应该与RAM的地址线相连，而是CPU的输出线与地址线相连。
首先，类比8086，我们也需要一个段寄存器，用r0代替，然后立即数表示段内偏移。如此一来我们便可以计算出RAM的地址，而且很好的解决了连续内存访问的问题。

+ 寄存器的输出即充当数据线，又充当地址线？（参与LAU运算计算地址）

​		显然，这样不行！

采用最简单的方式，直接增加一对寄存器的输出线和地址线。

将Rout0还与LAU连接，去计算地址，和其他的算数运算，Rout1与RAM的数据线连接，提供数据去写入RAM

+ 那么多出来的要写入RAM的寄存器的地址线由谁控制？

答：rt  😏️ （此处参考网络来源）

rt即控写入地址又控制Rread1地址

![image-20210313223151161](https://i.loli.net/2021/03/13/5VFALjfoMCJgtql.png)

+ 那么思考一下，为什么可以这么干。

根据上述指令的规则，很容易发现，由于读写控制线的存在，rt不可能同时是写入地址，又是读取地址。因此rt可以即当作读取地址又当作写入地址。
