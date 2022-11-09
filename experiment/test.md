# 测试你的CPU

使用Testbench来完成自动化测试工作。  

## Testbench

下载群里上传的测试工程并打开。测试工程编写的版本是Vivado2019.2，请确认你的Vivado高于或正处于这个版本。

在工程内导入你的cpu文件，在soc_lite_top.v文件的第137行将mycpu的例化块更换为你的cpu。  

你的cpu应该给出以下输入输出端口： 

时钟与复位，外部中断可选
```verilog
    .clk              (cpu_clk   ),
    .rst_n            (cpu_resetn),  //low active
    .ext_int          (6'd0      ),  //interrupt,high active
```
指令ram读写端口
```verilog
    .inst_sram_en     (cpu_inst_en   ),
    .inst_sram_we     (cpu_inst_wen  ),
    .inst_sram_addr   (cpu_inst_addr ),
    .inst_sram_wdata  (cpu_inst_wdata),
    .inst_sram_rdata  (cpu_inst_rdata),
```
数据ram读写端口
```verilog
    .data_sram_en     (cpu_data_en   ),
    .data_sram_we     (cpu_data_wen  ),
    .data_sram_addr   (cpu_data_addr ),
    .data_sram_wdata  (cpu_data_wdata),
    .data_sram_rdata  (cpu_data_rdata),
```
指令ram和数据ram已经在soc_lite_top.v为你例化好了，你只需要将接口接上即可。

## 汇编器

根据RISC-V手册将指令翻译为机器码

一个简单的Python汇编器示例如下(施工中)

```python

```