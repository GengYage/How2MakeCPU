# 如何用计算机组成原理，数字电路的知识仿真一个CPU？
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
