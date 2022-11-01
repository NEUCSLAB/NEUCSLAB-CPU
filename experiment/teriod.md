# 单周期指令的执行时间

现在，我们已经有了能够组成一个CPU的所有组件，剩下的只剩下一件事：让它运行起来！现在，我们先进行一些简单的设计，设计一个单周期的处理器，让CPU一个周期只执行一条指令。  

每条指令的执行过程一般需要以下几个步骤：

1. 取指令 ：使用本周期新的PC从指令存储器中取出指令，并将其放入指令寄存器（IR）中
2. 指令译码 ：对取出的指令进行分析，生成本周期执行指令所需的控制信号，并计算下一条指令的地址
3. 读取操作数 ：从寄存器堆中读取寄存器操作数，并完成立即数的生成
4. 运算 ：利用ALU对操作数进行必要的运算
5. 访问内存 ：包括读取或写入内存对应地址的内容
6. 寄存器写回 ：将最终结果写回到目的寄存器中

以R型指令为例，一条指令的操作定时应该如下图所示

![R型指令操作定时](/img/R_op_timing.png "R型指令操作定时")

现在你可能发现一个问题，就是我们之前设计的寄存器和储存器都是受时钟信号控制的，需要等待下一个时钟周期才能得到数据。现在我们来对之前的器件进行简单的改造，使其满足单周期的要求。  

## 寄存器组

```verilog
module regfile(
	input           clk_i,
	input           we_i,
	input   [4:0]   ra1_i,
    input   [4:0]   ra2_i,
    input   [4:0]   wa3_i,
	input   [31:0]  wd3_i,
	output  [31:0]  rd1_o,
    output  [31:0]  rd2_o
);

	reg [31:0] rf[31:0];

	always @(posedge clk_i) begin
		if(we3_i) begin
		    rf[wa3] <= wd3_i;
		end
	end

	assign rd1_o = (~|ra1_i) ? rf[ra1_i] : 32'h0;
	assign rd2_o = (~|ra2_i) ? rf[ra2_i] : 32'h0;

endmodule
```

稍微解释一下这一段代码  

首先依然使用时序逻辑对输入数据进行保存  
```verilog
always @(posedge clk_i) begin
	if(we3_i) begin
		rf[wa3] <= wd3_i;
	end
end
```

读取端使用组合逻辑输出，只要输入的地址ra1、ra2不为0，即可将rd1、rd2选通为对应的寄存器。
```verilog
assign rd1_o = (~|ra1_i) ? rf[ra1_i] : 32'h0;
assign rd2_o = (~|ra2_i) ? rf[ra2_i] : 32'h0;
```
这一段的电路相当于在寄存器的输出端增加了一组复用器。`rf[ra1_i]`与C语言中的数组访问类似，`|ra1_i`是一元约简运算符，效果是将ra1_i中每一位进行或运算，`~|ra1_i`的结果就是如果`ra1_i`全为0则输出高电平，与`ra1_i == 4'h0`效果一致。  

这一段代码依然有不够规范的地方，同时也可以加上读使能，交给同学们自行更正了。  

## 储存器

与寄存器类似的思路，在读取端使用组合逻辑来加速读取。当然不需要我们自己写，Xilinx依然提供了对应的IP核，这就是DRAM。  

在IP Catalog中搜索memory，并选中Block Memory Generator，开始生成DRAM，内容与生成RAM类似，只需要在端口处选择所需要的输入输出类型即可，端口依然选择简单双端口。  

![DRAM IP核配置](/img/Vivado_dram_inst_1.png "DRAM IP核配置")

具体参考官方文档[Distributed Memory Generator v8.0 Product Guide (PG063)](https://docs.xilinx.com/v/u/en-US/pg063-dist-mem-gen)  

a与d用于写入数据，dpra用来给输出数据的地址，当dpra给定时，dpo也会跟着输出数据。  

## 工作时钟

现在，整个CPU中所有的时序逻辑都确认下来了，该选择一个合适的时钟频率了。那么应该怎么选择呢？一个好办法就是使用Clock Wizard IP核输出一个稳定的时钟，然后通过Implemented Design中的时序报告了解当前的时序裕量，从而得到一个大致的时钟值。

首先先例化时钟IP核，在IP Catalog中搜索clock，选中Clock Wizard开始生成，在Output Clock中选择时钟频率为10Mhz，然后将时钟输出端口接到你的CPU上，点击Flow Navigator->IMPLEMENTATION->Run Implementation启动布局布线，等待时序报告。  

