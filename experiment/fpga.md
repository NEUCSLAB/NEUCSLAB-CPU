# FPGA开发入门

在本课程的学习中，你可能有这样的疑问：我学了这么多电路究竟要怎么实现呢. 有可能你会想到数电实验中使用过的74系列逻辑电路，但是使用这些基础器件搭建电路显然不太现实. 现在，我们有了更强大的验证工具：现场可编程逻辑门阵列(Field Programmable Gate Array, FPGA). 这是一种很特殊的器件，它通过可编程的查找表来模拟逻辑门的功能，为工程师提供设计实现平台.  

不同于我们常见的微机，FPGA中并不存在处理器，而是通过编程在内部实现电路功能，这要求工程师在开发时必须对自己即将实现的电路有着清晰的认识.

>实际上，在目前一些高端的FPGA开发板上集成了ARM协核，来完成一些FPGA不擅长的工作，如Xilinx的Zynq系列开发板.

FPGA的开发语言是VHDL和Verilog HDL，这两种语言都被称为硬件描述语言(Hardware Describe Language, HDL). 注意到名称中的“描述”二字，HDL与常用的编程语言有一个很大的区别就是工程师在写代码时其实是将他脑海里的电路通过代码进行描述，如果按照平时进行高级语言编程时那样描述行为交给编译器优化，那很有可能写出来的代码无法综合为实际可实现的电路. 

本实验中使用的语言为Verilog HDL. 如果你对你的VHDL编程能力很有自信，也可以使用VHDL.  

## 开发平台
实验室使用的FPGA开发板上搭载的是Xilinx公司出品的xc7a100t，因此我们需要使用Xilinx公司推出的Vivado开发套件进行开发.  
> 世界上主要有两个公司在销售商用FPGA器件，一个是Xilinx，一个是在2015年被Intel收购的Altera. 由于FPGA的开发就是研究如何利用官方提供给你的硬件资源，因此一般使用哪家公司的芯片就使用哪家公司的开发套件.  
>目前国内有着紫光同创等公司在做商用FPGA器件，占有一定的市场.  
><del>Xilinx已在2021年被AMD收购.</del>  

Vivado可以在官网上下载，下载时请确保自己电脑上有着100G以上的空闲空间(安装包40G，安装后占用60G+的空间).  

不同版本的Vivado工程相互打开会有一定的问题，因此我们要求所有同学都使用Vivado2020.1版本.  

安装过程需要自己参考[Vivado Design Suite User Guide: Release Notes, Installation, and Licensing (UG973)](https://docs.xilinx.com/r/en-US/ug973-vivado-release-notes-install-license/Download-the-Installation-File)进行安装. 不要诧异，学会读官方文档是你入门FPGA开发的第一步，你使用FPGA上所有的资源时，最权威的参考书就是官方的文档.  

此处，我们给出安装Vivado时部分需要自己选定的细节部分，剩下的需要同学们自己参考文档进行操作.  

1. 下载2020.1版本需要前往[Vivado版本归档页面](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive.html)，点击2020.1寻找安装包.
2. 你需要下载的是Vivado Design Suite - HLx Editions - 2020.1  Full Product Installation中的 Vivado HLx 2020.1: All OS installer Single-File Download (TAR/GZIP - 35.51 GB)，注意流量消耗. 建议有一个人下载好后相互拷贝一下.
3. 安装过程中需要选择安装Vivado HL Webpack，这个是免费版本.
4. 器件中只选择7 serial以减少占用空间.

安装完毕后启动Vivado即可.

## 工程环境配置
关于如何使用Vivado，我们还是会丢给你一个文档[Vivado Design Suite User Guide: Using the Vivado IDE (UG893)](https://docs.xilinx.com/r/en-US/ug893-vivado-ide/Using-the-Vivado-IDE)，但是这个文档比较庞大，仅供你需要查找某些功能时查看.   

首先需要学会新建工程，启动Vivado后，点击Create Project，选择工程目录后，开始进入硬件开发特有的一些选项  

![Vivado工程类型](/img/Vivado_new_project_type.png "Vivado工程类型")  

选择RTL Project即可，剩下的选项在你熟悉整个开发流程后就会自然明白.  
之后可以添加已有的文件，我们新建的是空白工程，跳过即可，之后便会选择目标芯片  

![Vivado芯片类型](/img/Vivado_new_project_part.png "Vivado芯片类型")  

这里需要选择我们的开发板xc7a100t，否则程序烧进去是可能不工作的. 之后确认配置无误即可生成工程.  

## 工程文件 
我们推荐初学者通过Vivado来管理工程文件，方法是在Source窗口点击加号新建文件.  

![Vivado新建文件](/img/Vivado_project_new_file.png "Vivado新建文件")  

简单介绍一下这里提供的四个文件夹
* Design Source存放工程源码  
* Constraints存放约束文件
* Simulation Sources存放仿真文件
* Utility Sources存放其他的工具代码，一般用不上

根据自己的需要将文件存放至对应的文件夹中，一般大多数文件都会存放至Design Source，你也可以在里面继续添加文件夹组织源码.  

约束文件会在之后介绍，仿真文件就是测试模块所使用的Testbench.  

和高级语言中的main函数一样，Vivado也需要一个入口模块，在这里被称为顶层模块(Top Module)，通过在需要设置为顶层模块的文件上右键->Set as top实现，顶层模块会作为之后所有操作的入口.  

如果你的工程需要用到Xilinx提供的硬件资源，那么你需要使用Xilinx提供的接口来调用.  

Xilinx提供了两种方式来使用硬件资源，一是Verilog原语，二是IP核，这两种方式在代码中都以例化模块的方式出现.

### Verilog原语
当你需要直接使用D触发器、查找表、时钟缓冲等基础资源时，可以直接使用原语例化. 通过点击Flow->Languange Templates查询官方所提供的例化模板，将其复制进源文件中即可完成例化.  

![Vivado原语](/img/Vivado_language_templates_clock_buffer.png "Vivado原语")  

这个页面也提供了Verilog与VHDL的语法样例.

### IP
这里的IP指的是知识产权，Xilinx提供了大量现成的电路结构，仅需要进行一定的配置即可达到我们所需要的功能. 在Vivado点击Flow Navigator->PROJECT MANAGER->IP Catalog即可打开IP核页面. 下面以例化加法器IP核为例说明.  

![查找IP](/img/Vivado_IP_add.png "查找IP")   

![加法器配置1](/img/Vivado_IP_add_conf_p1.png "加法器配置1")  

![加法器配置2](/img/Vivado_IP_add_conf_p2.png "加法器配置2")  

选择好所需要的配置后，点击OK生成IP核，需要注意，配置好的IP核除了名字以外都可以再次更改，因此给IP核起名时要注意一些.  

之后将IP核例化模板拷贝至源文件即可.

![加法器例化](/img/Vivado_IP_instantiation.png "加法器例化")  

## 生成RTL图
生成RTL图是最快的代码检查方法，同时也可以检查你设计的电路与你描述的电路是否一致. 当然，RTL图的电路与实际的电路可能存在不一致的地方.  

点击Flow Navigator->PROJECT MANAGER->RTL ANALYSIS->Open Elaborated Design->Schematic，Vivado会开始从顶层模块分析HDL，生成对应的电路图.  

## 仿真
点击Flow Navigator->SIMULATION->Run Simulation->Run Behavioral Simulation，Vivado会从Simulation Sources中的顶层模块启动仿真，默认运行1000ns.  

默认的波形窗口只会添加TestBench中的信号，如果你需要观察模块内的信号，可以在Object窗口中拖到Wave窗口内，之后重启仿真即可观察波形.  

## 综合
点击Flow Navigator->SYNTHESIS->Run Synthesis启动综合，Vivado会从顶层模块开始分析RTL代码，使用实际的FPGA资源对电路行为进行描述. 如果代码可以被实际电路描述，也就是代码可被综合，Synthesis过程就可以顺利结束，之后可以点击Flow Navigator->SYNTHESIS->Open Synthesized Design查看综合后报告.  

一般综合后最主要的工作是使用Set up debug设置波形采集器，这个会在之后上板验证时介绍.  

## 布局布线
点击Flow Navigator->IMPLEMENTATION->Run Implementation启动布局布线，这个过程会消耗比较长的时间，可以重新检查一下你的代码.  

在这个操作中，Vivado会根据代码，将综合后得出需要使用的电路资源放置在芯片上，并保证从一个触发器的输出到另一个触发器的输入延时不会大于一个时钟周期(这就是时序约束的一个比较直白的表述). 为了保证所有路径都能满足这一要求，Vivado会花费大量时间寻优，因此良好的时序电路设计和良好的时序约束对于FPGA设计至关重要.  

完成布局布线后，点击Flow Navigator->IMPLEMENTATION->Open Implemented Design查看报告，其中最重要的是观察时序报告，检查是否有路径发生了时序违例. 时序报告相关内容作为拓展自行查看学习.   

一般而言，我们在进行设计时，数据通路的延时应该了然于心中，如果数据通路上出现了时序违例，就需要我们减少组合逻辑层次，或是增加寄存器采样，而这些都是可以在设计阶段避免的问题.  

## 烧录
点击Flow Navigator->PROGRAM AND DEBUG->Generate Bitstream生成比特流，一般到这时候基本都不会出问题了，之后将开发板连接上电脑，点击Flow Navigator->PROGRAM AND DEBUG->Open Hardware Manager->Open Target->Auto Connet，等待Vivado找到开发板后点击Program device，将程序烧录进板子即可.  

程序一但烧录完成，电路结构就会形成，FPGA就会开始全速工作，因此无法像之前debug时那样随意观察，这时就需要集成逻辑分析器(Integrated Logic Analyzer, ILA)抓取信号了. ILA可以将待测信号的数据存入板上ram中，通过线缆传输给Vivado显示. ILA也是电路的一部分，因此在布局布线前必须添加到电路中. 添加ILA可以通过Synthesized Design中的Set up debug实现，也可以通过ILA的IP核实现.  

