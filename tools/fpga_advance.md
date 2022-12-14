# FPGA开发进阶  

本章主要讨论一些进阶向的功能. 当你通过仿真验证了自己的电路功能正确后，你需要上板验证电路能否正常工作，这时你又会遇到新问题：物理约束、时序约束是啥，为什么编译器会报这么多奇奇怪怪的错误. 本章将会简单介绍这些知识.  

## 物理约束  

物理约束主要包括两部分内容，一是约束硬件资源放置于板上的位置，称为布局约束；二是约束输入输出端口对应的FPGA管脚，称为IO约束. 剩余的还有布线约束和配置约束. 对于初学者而言，最重要的是IO约束，其余约束等到你的设计能力触及到编译器的优化上限时，你会自然而然地开始学习它们的.  

IO约束主要用于指定端口对应的管脚以及对应的电平. 你可能是第一次接触电平这个概念. 在数字电子技术中，我们使用的74系列芯片输入电平使用的是Transistor-Transistor Logic(TTL)标准，高于2.0V识别为高电平，低于0.8V识别为低电平. 但是，不同的晶体管工艺能输出的电压值是不一样的，输出电平越低意味着所需要消耗的功率也越低. 因此，现行的电子器件有着多种不同的电平标准，而FPGA对电平的配置提供了很高的自由度，对于每个管脚你都可以配置不同的电平标准，从而适应不同的器件.  

下面给出一些常用的电平标准

| 电平标准 | 输入高限 | 输入低限 | 输出高限 | 输出低限 | 
| ------- | -------- | -------- | ------- | -------- | 
| LVTTL3V3 | 2.0V | 0.8V | 2.4V | 0.4V | 
| LVCOMS15 | 0.65VCC | 0.35VCC | VCC | 0V | 
| LVCOMS25 | 2.0V | 0.4V | 1.7V | 0.7V | 

LVCOMS15后面的数字15代表着内部有源器件的电压，也就是VCC=1.5V. 当然这是有一定容忍度的，这个容忍度会影响到它的输入输出情况. 而对于LVCOMS25来说，输入输出与VCC无关，但要求VCC=2.5V. 

如何知道我们所需要的管脚能支持什么电平标准呢？最直白的方法是通过Tcl命令，在Tcl Console中输入
```
get_io_standards -of [get_package_pins <PIN>]
```
比如我想查看A15管脚能支持的电平标准，就需要输入以下内容
```
get_io_standards -of [get_package_pins A15]
```
命令行中会返回如下值
```
LVTTL LVCMOS33 LVCMOS25 LVCMOS18 LVCMOS15 LVCMOS12 HSUL_12 HSTL_I HSTL_II HSTL_I_18 HSTL_II_18 SSTL18_I SSTL18_II SSTL15 SSTL15_R SSTL135 SSTL135_R DIFF_HSTL_I DIFF_HSTL_II DIFF_HSTL_I_18 DIFF_HSTL_II_18 DIFF_SSTL18_I DIFF_SSTL18_II DIFF_SSTL15 DIFF_SSTL15_R DIFF_SSTL135 DIFF_SSTL135_R DIFF_HSUL_12 PCI33_3 MOBILE_DDR DIFF_MOBILE_DDR BLVDS_25 LVDS_25 RSDS_25 TMDS_33 MINI_LVDS_25 PPDS_25
```
关于Tcl命令，我们会在之后的小节中讲解，现在先回到IO约束上来. 大多数时候，我们并不会这么操作IO管脚. Vivado给我们提供了图形化的界面进行控制.   

在完成综合后，点击Flow Navigator->SYNTHESIS->Open Synthesized Design打开综合报告，之后在上方菜单栏点击Layout->I/O Planning打开管脚分配菜单.  

![IO分配](/img/Vivado_IO_Planning.png "IO分配")

可以看到，我们分配管脚时，主要需要关心Package Pin(分配到哪个管脚)和I/O Std(电平标准)即可.  

那么，我们该如何知道我们需要使用的是哪个管脚呢？  

首先我们要了解一个概念：Bank. 由于FPGA管脚很多，不同的管脚需要的电平标准不一样，因此为了便于管理和适应多种电器标准，FPGA的IO被划分为若干个组（Bank），每个Bank的接口标准由其接口电压VCCO决定，一个Bank只能有一种VCCO，但不同Bank的VCCO可以不同，只有相同电气标准的端口才能连接在一起。   

因此，要找到我们需要使用的管脚，首先需要从开发板原理图上找到我们所需要的外设的引脚，再沿着原理图找到这个引脚连接到FPGA芯片上的哪个Bank上哪个Pin，再根据外设设置IO标准即可. 这里以AX7325开发板寻找LED引脚为例进行说明  

![原理图中LED的位置](/img/Sch_led.png "原理图中LED的位置")

从这里知道连接这条线的网络名叫LED1-LED4，在PDF中搜索LED1，就可以找到对应的管脚了  

![原理图中Bank的位置](/img/Sch_bank.png "原理图中Bank的位置")  

可以看到是连接到了Bank17的A22、C19、B19、E18管脚，在I/O Planning中填入对应的Bank和Pin.  

接下来查看电平，对于输出端口来说，使用的电平标准取决于内部的有源器件. 这里我们注意到左上角的VCCO接到了DIMM_VCCO，查看这条信号，发现它接到了AMS1117上

![原理图中的LDO POWER](/img/Sch_LDO_POWER.png "原理图中的LDO POWER")  

这是一个线性稳压元件，输出电压为1.5V，因此选择的电平标准应是有源电压为1.5V的电平标准. 具体还是需要查看相关外设的文档.   
<del>其实这个可以通过tcl命令查到</del>  

现在回来看输出器件，LED一端接到电源，一端接到三极管上，利用三极管的截止区来控制电路断开. 这里使用的三极管型号是MMBT3904LT1G，基极电压Vbe是0.65V-0.85V，只要我们的高电平低于VCC(3.3V)，低电平低于Vbe即可. 这里我们选择LVCMOS15，理由是低电平能接近0V，保证三极管能工作于截止区. 

>如果上一段你难以理解，你可能需要翻翻你的模拟电子技术课本. 不过不用担心，一般都不会这么麻烦. 

一般如果你不确定所需要使用的电平标准的话，就去查所需要使用的外设的芯片手册，或是查看你所用的开发板的例程中所使用的电平标准，最后实在没办法可以参考上面的操作，确定电平和驱动器件之后选择一个电平标准进行尝试. 只要电压不是太高一般不会有太大问题，但一定要谨慎行事.  

顺带提一句，除了单端输入输出端口以外，FPGA还拥有大量差分端口，即用两条线进行传输. 差分信号拥有着自己特殊的电平标准，一般以DIFF开头. 差分信号广泛应用于高速传输中，拥有很好的抗干扰能力. 很多芯片的时钟源信号就是差分信号，在送入FPGA后需要通过时钟缓冲转换为单端时钟信号.  

## 时序电路

为了讲明白时序违例和时序约束，必须先讲清楚什么是时序电路，以及为什么我们要使用时序电路. 

我们在数电实验中使用的逻辑门电路，当输入改变的时候，输出也会跟着改变. 对于我们的反应速度来说，电路状态的改变确实是一瞬间的事情，但是对于电路本身而言就不一定了. 每个数电元件的电气特性里都有一项上升时间，这意味着元件将电平从0翻转到1所需要的时间. 比如74LS161，这个参数的值在20ns左右. 但是参数中给出的值并不代表着电平翻转时间就一定是这个值. 文档中一般会给出三个数据：最小值、典型值、最大值，这三个数据描述了这个参数的范围和最容易出现的数值.   

这意味着什么？意味着我们无法确定数据从输入端到输出端所需要的时间，这个时间完全取决于数据通路上所有器件的延时，如果输入数据不断变化的话，我们无法确定数据通路上数据何时送到哪个位置了. 如果我们的计算电路有前后的数据依赖的话，这点是非常致命的.   

现在，想想D触发器，你就会明白时序电路最重要的作用了：它会把输入数据对齐到另一个信号的上升沿. 现在，只要我们提供一个可控的信号，我们就能通过D触发器控制数据通路中数据的流动顺序. 这个可控的信号，被简化为一个方波信号，我们称为时钟信号.   

绕了一大圈，算是讲完了时序电路以及时钟信号. 我们将受到时钟信号控制的电路称为时序逻辑，而不受时钟信号控制的电路称为组合逻辑. 所有的FPGA电路设计都是由时序逻辑和组合逻辑组成的，从时序逻辑输出的数据，经过组合逻辑的计算，又送到了另一个时序逻辑的输入端.   

对于verilog，我们通过以下语句描述时序逻辑
```verilog
always @(posedge clk)
```
很显然，时序逻辑对时钟上升沿敏感，只有时钟上升沿到来时才会执行操作.  

我们通过以下语句描述组合逻辑
```verilog
always @(*)
assign
```
也很显然，当输入发生改变时，输出也会发生改变.  

当然，除非你非常确定你的组合逻辑内所有信号都是敏感信号(比如状态机)，否则请尽量避免使用`always @(*)`  

## 时序违例

有了时序逻辑的概念后，我们再来看上一节的一段话  

>所有的FPGA电路设计都是由时序逻辑和组合逻辑组成的，从时序逻辑输出的数据，经过组合逻辑的计算，又送到了另一个时序逻辑的输入端.  

如果你已经理解了时序电路的来源，你很快就会意识到一个很严重的问题：组合逻辑也是有延时的，数据输出时间与输入时间依然有一段距离. 但是，他又会被下一个触发器在时钟上升沿采样，因此这个不会造成问题.   

感觉乱了吗？问题就隐藏在这里，如果组合逻辑的延时过长，导致数据到达触发器时已经超过了下一个时钟上升沿，怎么办.   

这就是我们所说的时序违例的来源. 如果组合逻辑层级过多，就会导致组合逻辑延时超过一个时钟周期，触发器可能需要等到下一个时钟周期才能采样到正确的数据. 对于FPGA而言，内部的器件延时以及线路延时都是可以计算出来的，因此在布局布线后便可以计算出每条组合逻辑路径所需的延时，以及这个延时是否小于一个时钟周期. 如果延时超过时钟周期，便会报告Fail Timing，也就是时序违例.  

时序违例是FPGA设计中最重要，也是最麻烦的错误，一旦时序违例过大，整个电路都不可能正确运行.  

## 如何解决时序违例

有三种办法：降低时钟频率、设置寄存器打断流水线、设置时序约束. 本节介绍前两种方法，时序约束需要单独一节进行讲解.  

### 降低时钟频率

时序违例的来源是组合逻辑的延时长于时钟周期，因此只需要降低时钟频率，增加时钟周期长度就可以了.  

降频是最有效的办法，也是最没有办法的办法. 因为我们进行FPGA设计时，为了达到更快的运行速度，肯定是要想办法提高时钟频率的. 如果选择了降频，就代表着“我已经达到了我设计的上限，没办法再快下去了”，是一种妥协，是宣告设计优化空间结束的标志. 因此，在手段没有用完之前，都不会考虑降频运行.  

### 打断流水线

这里提的流水线指的是组合逻辑的数据通路，既然整个组合逻辑延时长，我可以在中间哪里增加一个寄存器对数据进行采样，降低组合逻辑的层级，将整个延时划分到两个时钟周期中平摊. 

打断流水线是最常用的一种做法，前提是你必须清楚你设计的数据通路，能够确认在哪里设置寄存器能够在不破坏组合逻辑的情况下减少延时. 代价是增加完成计算所需要的时钟周期.  

## 时序约束

时序约束其实严格意义上来说并不是解决你设计问题的，而是解决工具问题的.  

首先我们需要了解，如果碰到时序违例，Vivado会怎么做.  

在FPGA上，决定组合逻辑延时的有两部分，一部分是器件本身延时，一部分是线网延时. 而这两部分都是有优化空间的. 就算是相同的器件，延时也有高有低，可以更换低延时的器件，将器件距离缩短来减小组合逻辑延时. 当Vivado遇到时序违例时，就会进行这样的操作来试图解决时序违例. 因此，有些时序违例并没有被报告，不是因为设计没问题，而是因为Vivado帮助你优化了这个问题.  

但是，板上的高速资源数量是有限的，如果资源被用完了，剩下的时序违例就无法解决了，只能交给用户重新设计电路. 在布局布线后用户拿到的时序报告便报告了这一部分的内容.  

不过，对于我们的设计而言，有一些路径是不需要一定在一个周期内传输完成的，它可以等待多几个周期，或者这条路径根本不需要考虑时序问题但Vivado却将它视作严重时序违例进行优化消耗了大量资源. 这是，我们就需要通过正确的时序约束来指示Vivado如何处理部分路径的延时问题.  

当然，进行时序约束的前提是，你能够确定这条路径的时序要求并不是1个时钟周期. 如果你不确定，或者你是个初学者，还是尽量不要修改时序约束为好.  

时序约束有三种：伪路径、多周期路径、最大延时路径。伪路径会指示Vivado不分析本路径，多周期路径会指示Vivado在时序分析时延长本路径的时钟周期，最大延时路径会指示Vivado在时序分析时将最大延时修改为指定的延时.   

一般用的比较多的是伪路径和多周期路径，通常在跨时钟域路径上都需要设置多周期路径，否则Vivado的时序限制会非常严格，很容易造成时序违例.  

考虑到时序约束的难度，我们将时序约束的内容放于参考资料中，有需要进行约束的同学可以自行参考.  

当你开始进行时序约束时，恭喜你，你已经在FPGA开发上走出了重要的一步，从现在开始，你需要考虑的问题要加上工具本身的制约了.  

## 时钟约束

时钟约束其实被算在时序约束里，但它的主要作用是指定时钟信号的频率，用于Vivado的时序分析，因此单开了一节来讲.  

如果你的设计需要引入一个外部时钟，你需要在时钟约束中指定它的时钟频率. 如果你自己通过逻辑对时钟进行分频，你也需要在时钟约束中指定时钟频率. 如果你使用的时钟信号来自于IP核，那么IP核内已经自带了对应的时钟约束，不需要你手动约束了.  

创建时钟约束的方法是点击Flow Navigator->SYNTHESIS->Open Synthesized Design->Constraints Wizard，Vivado会通过图形化的向导指示你约束时钟信号，你只需要选中输入时钟并填入对应的时钟频率即可. 怎么知道输入时钟频率？查开发板文档，查原理图，看看时钟输入管脚接到哪个晶振上了，晶振的时钟频率是多少就是多少. 

编辑时钟约束的方法是点击Flow Navigator->SYNTHESIS->Open Synthesized Design->Edit Timing Constraints，可以看到所有的时钟约束和时序约束，并直接进行修改.  

## 约束文件

上面说的物理约束与时序约束最后都会被写入到一个.xdc文件中，这就是整个工程的约束文件，记录着和物理资源有关的信息. 这个文件可以直接被编辑器打开和编辑，对于熟练的FPGA工程师而言，直接编辑xdc文件比起使用图形化界面更加直观，但对于初学者而言，还是将xdc文件交给Vivado进行编辑吧.  

如果你使用Set up debug来抓信号，xdc文件中会添加dbg相关的约束，当你碰到dbg相关的报错难以解决时，可以考虑在xdc文件中删除所有debug信号重新综合.  

## Tcl命令

Tcl是整个Vivado中最有工程师味的部分. 你在图形界面中所有的操作，最后都会变成一条tcl命令执行，你可以在Tcl Console中查看执行的所有指令. 甚至有一些操作是无法通过图形界面完成的，必须从Tcl Console执行才能完成功能. 当然对于初学者而言，其实可以忽略Tcl命令，完全通过GUI进行操作. 但tcl命令的存在使得自动化操作成为了可能，在Tools->Run Tcl Scripts中可以运行编写好的tcl脚本对Vivado进行控制.  

## 写在最后

Vivado是一个非常庞大的工具，它在提供硬件资源操作接口的同时，尽可能开放整块FPGA的可操控性，让开发者能够完全掌握这块芯片. 其实它在很多地方都留有用户操作接口，比如优化策略，Vivado提供了多种优化方向给用户选择，可以尽可能尝试出一个满足时序要求的版本.   

> 从我们的角度来说，我们要求所有同学使用Vivado默认的优化策略，因为这会暴露很多设计上就存在的问题. 对于本次实验，还不需要通过这种方法来优化时序.  

如果以后你有志向在FPGA方向上工作，相关的开发套件是一定要熟练掌握的，特别是Xilinx，大多数公司都在使用Xilinx的产品，因此Vivado几乎是唯一的选择. 当然，工具永远只是辅助，最关键的还是工程师本身的电路设计水平，这点在FPGA上体现得淋漓尽致.  

## 参考资料
1. [FPGA电平标准](https://www.cnblogs.com/doincli/p/15234990.html)  
2. 时序报告分析  
3. 时序约束  
4. XDC文件  
5. TCL指令  
