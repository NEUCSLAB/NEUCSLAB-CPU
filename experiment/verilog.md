# Verilog 基础回顾

本章主要帮助你回顾一下verilog基础内容，以及一些实际开发中的思维方法。如果有任何细节问题，请自行参考verilog教材与网络内容。记住：机器永远是对的。

## 前言
- Verilog是硬件描述语言，用语言描述的方式进行电路设计，最终要实现出硬件电路。
- 评价一个verilog代码的好坏是看最终实现的功能和性能。合理的设计方法是首先理解要设计的电路，也就是把需求转化为数字电路，对此电路的结构和连接十分清晰，然后再用verilog表达出这段电路。
- verilog只是简化了电路设计的工作量，本质上就是设计数字电路，永远绕不开电路这点！
- 要理解硬件的并行性，搞清楚时序关系，一定要摒弃顺序执行的思维。
- 描述电路的难点不在于写代码，而在于结构框图（空间维度）、状态机（时间维度）。

## 建模

所谓建模，指的是通过一定的模型去描述你所需要使用的电路。随着电路结构愈发复杂，通过门级电路去描述整个电路结构工作量非常大。通过模型去描述电路，有助于用户聚焦于电路的关键部分，而不是将精力过多浪费在细枝末节上。  

Verilog常用的建模方式有两种：行为级建模和数据流建模。

### 行为级建模

Verilog支持设计者从算法的角度，即从电路外部行为的角度对其进行描述．因此行为级建模是从一个很高的抽象角度来表示电路的。在这个层次上设计数字电路更类似于使用C语言编程，而且Verilog 行为级建模的语法结构与C语言相当类似。Verilog提供了许多行为级建模语法结构.为设计者的使用提供了很大的灵活性。  

行为级建模常常使用always块进行描述，通过if和case等分支语句对电路行为进行控制。

### 数据流建模

Verilog支持用户从数据流的角度对电路建模。数据流建模意味着根据数据在寄存器之间的流动和处理过程对电路进行插述，而不是直接对电路的逻辑门进行实例引用。
数据流建模通常使用assign语句进行描述。  

两种建模方式各有优劣，用户需要在设计电路时选择合适的建模方式去描述电路。

## 基础语法

```verilog
//二选一多路选择器
module mux2i1o_1b_no_dly (
    input       sel_i,
    input       in_0_i,
    input       in_1_i,

    output  reg out_o
);  

    always @(*)begin
        if(sel_i) begin
            out_o = in_1_i;
        end else begin
            out_o = in_0_i;
        end
    end

endmodule
```
模块声明  
```verilog
module mux2i1o_1b_no_dly (  
```
端口声明，注意结尾要闭合括号  
```verilog
input       sel_i,  
input       in_0_i,  
input       in_1_i,  
output  reg out_o
);
```
另一种端口声明，根据个人喜好使用
```verilog
module mux2i1o_1b_no_dly ( 
    sel_i,
    in_0_i,
    in_1_i,
    out_o
);
    input   sel_i;
    input   in_0_i;
    input   in_1_i;
    output  out_o;

    reg     out_o;
```
行为级建模描述多路选择器
```verilog
always @(*)begin
    if(sel_i) begin
        out_o = in_1_i;
    end else begin
        out_o = in_0_i;
    end
end
```
数据流建模描述多路选择器
```verilog
wire    out_o;

assign  out_o = sel_i ? in_1_i : in_0_i;
```
模块结束标识
```verilog
endmodule
```

TIPS  
- “模块名”是模块唯一的标识符，区分大小写。  
- “端口列表”是由模块各个输入、输出和双向端口组成的列表。(input,output,inout)  
- 端口用来与其它模块进行连接，括号中的列表以“,”来区分，列表的顺序没有规定，先后自由。  
- 模块中用到的所有信号都必须进行数据类型的定义。  
- 声明变量的数据类型后，不能再进行更改。  
- 在VerilogHDL中只要在使用前声明即可。  
- 声明后的变量、参数不能再次重新声明。  
- 声明后的数据使用时的配对数据必须和声明的数据类型一致。  

## 时序逻辑

时序逻辑是指对时钟信号敏感的电路，常见于寄存器电路。时序逻辑通常用于将数据或控制信号对齐到时钟边沿。  

时序逻辑描述方法如下，通常只能通过always块或Verilog原语描述。
```verilog
input data_i;

reg data;

always @(posedge clk_i or posedge rst_i) begin
    if(rst_i) begin
        data <= 1'b0;
    end else begin
        data <= data_i;
    end
end
```
上述语句描述了一个异步复位D触发器，在每个时钟上升沿将data_i对齐至data。描述电路与下述Verilog原语一致。
```verilog
FDRE #(
	.INIT(1'b0) 
)inst_D_flipflop (
	.C (clk_i),      
	.R (rst_i),      
	.CE (1'b1), 
	.D (data_i),      
	.Q (data)      
);
```
接下来给这个触发器加上使能信号，更接近数电课中所描述的D触发器。  
```verilog
input data_i;
input ce_i;

reg data;

always @(posedge clk_i or posedge rst_i) begin
    if(rst_i) begin
        data <= 1'b0;
    end else begin
        if(ce_i) begin
            data <= data_i;
        end else begin
            data <= data;
        end
    end
end
```
描述电路与下述Verilog原语一致。
```verilog
FDRE #(
	.INIT(1'b0) 
)inst_D_flipflop (
	.C (clk_i),      
	.R (rst_i),      
	.CE (ce_i), 
	.D (data_i),      
	.Q (data)      
);
```
在使用always块描述时序逻辑时，敏感边沿必须包括时钟信号，赋值语句必须使用非阻塞赋值。

## 组合逻辑

组合逻辑是输出随着输入立即改变的电路。常用于数据计算电路与控制电路，在电路完成计算后再通过时序逻辑将数据对齐至时钟沿。  

组合逻辑可以通过always块和assign描述。

```verilog
reg     out_o;

always @(*)begin
    if(sel_i) begin
        out_o = in_1_i;
    end else begin
        out_o = in_0_i;
    end
end
```

```verilog
wire    out_o;

assign  out_o = sel_i ? in_1_i : in_0_i;
```

组合逻辑的always块中必须使用阻塞赋值。  

always块中赋值的变量需要声明为reg类型，assign赋值的变量需要声明为wire类型。  

在使用组合逻辑时，一定要清楚代码所描述的电路可能的形态。所有的组合逻辑在综合后都是查找表、复用器以及可能的部分计算单元组成，当输入发生变化时输出会直接跟随变化，因此如果需要对其输出，需要使用时序逻辑。  

## 阻塞赋值与非阻塞赋值

之前的课本可能会有讲这两种的赋值区别。但我们比较推荐这么记忆：  

- 在时序逻辑中使用非阻塞赋值。
- 在组合逻辑中使用阻塞赋值。  

不需要搞清楚这两种语句具体的赋值顺序，你的脑海里最应该搞清楚的是这里应该用时序逻辑还是组合逻辑，这样有助于你把握整体电路设计。

## 综合设计

依然以二选一复用器为例
```verilog
module mux2i1o_1b_no_dly (
    input       sel_i,
    input       in_0_i,
    input       in_1_i,

    output  reg out_o
);  

    always @(*)begin
        if(sel_i) begin
            out_o = in_1_i;
        end else begin
            out_o = in_0_i;
        end
    end

endmodule
```
如果需要将`out_o`对齐至时钟沿，可以写出如下代码
```verilog
module mux2i1o_1b (
    input       clk_i,
    input       rst_i,

    input       sel_i,
    input       in_0_i,
    input       in_1_i,

    output  reg out_o
);  
    reg     out;

    always @(*)begin
        if(sel_i) begin
            out = in_1_i;
        end else begin
            out = in_0_i;
        end
    end

    always @(posedge clk_i or posedge rst_i) begin
        if(rst_i) begin
            out_o <= 1'b0;
        end else begin
            out_o <= out;
        end
    end

endmodule
```
当然，这段代码可以简化如下。
```verilog
module mux2i1o_1b (
    input       clk_i,
    input       rst_i,

    input       sel_i,
    input       in_0_i,
    input       in_1_i,

    output  reg out_o
);  

    always @(posedge clk_i or posedge rst_i)begin
        if(rst_i) begin
            out_o       <= 1'b0;
        end else begin
            if(sel_i) begin
                out_o   <= in_1_i;
            end else begin
                out_o   <= in_0_i;
            end
        end
    end

endmodule
```
看起来好像简化了很多，但是我们非常不推荐你这么书写代码，因为这是一段非常“软件”的代码。我们希望你在书写代码时，是从硬件的角度去思考的，逻辑判断和运算就交给组合逻辑，再通过时序逻辑去对齐时钟。  

将时序逻辑与组合逻辑分开描述有助于工程师区分电路结构，定位问题。

## 模块的例化

所有的电路都是从一个个小模块搭建起来的，当你写好一个小模块后，就需要在大模块里调用它，这个过程成为例化。

以上述复用器为例进行例化
```verilog
    wire    sel;
    wire    data_0;
    wire    data_1;
    wire    sel_data;

    mux_2i1o_1b_no_dly inst_mux_2i1o_1b_no_dly_0(
    	.sel_i  (sel),
        .in_0_i (data_0),
        .in_1_i (data_1),
        .out_o  (sel_data)
    );
```
例化的语法如下
```verilog
    module_name inst_name (
        .port_name  (wire_name),
        .port_name  (wire_name),
        .port_name  (wire_name)
    );
```
`inst_name`可以自己定义，port_name与模块内端口保持一致。  

在例化时，不是所有端口都需要连接，部分端口可以悬空，此时在模块内部这个接口处于高阻态，可能会影响电路行为，需要自行抉择。  

## 测试

测试文件称为testbench，简称tb，可以自行定义一些电路行为对电路进行测试。

上述选择器的tb文件描述如下
```verilog
module mux_tb();

    reg     sel;
    reg     data_0;
    reg     data_1;
    wire    sel_data;

    initial begin
        sel     = 1'b0;
        data_0  = 1'b0;
        data_1  = 1'b0;
        
        #10
        data_1  = 1'b1;

        #10
        data_0  = 1'b1;

        #10
        data_1  = 1'b0;

        #10
        sel     = 1'b1;
        data_0  = 1'b0;

        #10
        data_1  = 1'b1;

        #10
        data_0  = 1'b1;

        #10
        data_1  = 1'b0;
    end

    mux_2i1o_1b_no_dly inst_mux_2i1o_1b_no_dly_0(
    	.sel_i  (sel),
        .in_0_i (data_0),
        .in_1_i (data_1),
        .out_o  (sel_data)
    );
```
initial块只能在testbench中使用，如果写在普通的verilog文件中，仿真时可以正常行为，但无法在实际电路中完成操作。同样的，所有的verilog系统任务都只能在仿真中作用，描述电路时还请聚焦在组合逻辑和时序逻辑上。  

使用testbench时，应尽量覆盖所有输入样例。

## 练习

实现一个4选1多路复用器，位宽为16位。