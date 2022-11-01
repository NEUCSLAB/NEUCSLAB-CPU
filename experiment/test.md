# 测试你的CPU

使用Testbench来完成自动化测试工作。  

## Testbench

Testbench文件示例如下(施工中)
```verilog
module ddr_tb();

    localparam      period = 5;
    reg             clk;
    reg             rst_i;
    reg             ctrl_en;
    
    reg             ddr3_cmd_finish;

    reg     [511:0] ddr_data[34815:0];
    reg     [511:0] ddr_din;

    reg     [511:0] ddr_fifo_din[127:0];
    reg     [6:0]   ddr_fifo_wr_pos;
    reg     [6:0]   ddr_fifo_rd_pos;

    always @(posedge clk) begin
        casex({rd_cdf_en,ddr_app_rdf_rden_o})
            2'b10:begin //write
                ddr_fifo_din[ddr_fifo_wr_pos - 1'b1]   <= ddr_din;
                ddr_fifo_wr_pos                 <= ddr_fifo_wr_pos + 7'b1;
            end
            2'b01:begin //read
                ddr_app_rdf_data_i              <= ddr_fifo_din[ddr_fifo_rd_pos];
                ddr_fifo_rd_pos                 <= ddr_fifo_rd_pos + 7'b1;
            end
            2'b11:begin //r&w
                ddr_fifo_din[ddr_fifo_wr_pos - 1'b1]   <= ddr_din;
                ddr_fifo_wr_pos                 <= ddr_fifo_wr_pos + 7'b1;
                ddr_app_rdf_data_i              <= ddr_fifo_din[ddr_fifo_rd_pos];
                ddr_fifo_rd_pos                 <= ddr_fifo_rd_pos + 7'b1;
            end
        endcase
    end

    initial begin
        $readmemb("D:\\ProjectSource\\IOdelay\\testdata\\data.txt",ddr_data);
    end

    //时钟
    always #(period) clk = ~clk;

    //instantiation of your cpu
```

在data.txt中书写你的汇编代码，启动仿真即可对指令进行测试。  

## 汇编器

根据RISC-V手册将指令翻译为机器码

一个简单的Python汇编器示例如下(施工中)

```python

```