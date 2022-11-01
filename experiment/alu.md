# 加法器与ALU

加法是数字系统中最常执行的运算，加法器是ALU（算术逻辑部件 Arithmetic-Logic Unit ）的核心部件。 减法可以看作是被减数与取负后的减数进行加法。即用加法器同时实现加法和减法两种运算。乘法也可以利用移位相加的算法来实现。因此，加法器可以说是计算机中最“繁忙”的部件了。

## 加法器

先从一位加法器开始实现，真值表如下

| $a_i$ | $b_i$ | $c_i$ | $s_i$ | $c_{i+1}$ | 
| :--: | :--: | :--: | :--: | :--: | 
| 0 | 0 | 0 | 0 | 0 | 
| 0 | 0 | 1 | 1 | 0 | 
| 0 | 1 | 0 | 1 | 0 | 
| 0 | 1 | 1 | 0 | 1 | 
| 1 | 0 | 0 | 1 | 0 | 
| 1 | 0 | 1 | 0 | 1 | 
| 1 | 1 | 0 | 0 | 1 | 
| 1 | 1 | 1 | 1 | 1 | 

由上述真值表可以得到$s_i$与$c_{i+1}$的表达式

$$
s_i = a_i\ XOR\ b_i\ XOR\ c_i
$$

$$
c_{i+1} = b_i\ \&\ c_i\ |\ a_i\ \&\ c_i\ |\ a_i\ \&\ b_i
$$

由此可以很轻松写出加法器的verilog表达式  

```verilog
module full_add_1b (
    input           a,
    input           b,
    input           cin,       
	output  reg     sum,
    output  reg     cout               
);  	           
	
	always @(*)begin
		sum = a ^ b ^ cin;
		cout = a & b | (cin & (a ^ b));
	end
	
endmodule
```

另一种写法也是可行的  

```verilog
module full_add_1b (
    input           a,
    input           b,
    input           cin,       
	output          sum,
    output          cout               
);  	           	
	
    assign  sum = a ^ b ^ cin;
	assign  cout = a & b | (cin & (a ^ b));
	
endmodule
```

我们需要的是多位加法器，只需要将一位加法器串联起来即可  

```verilog
module full_add_8b (
    input   [7:0]   a,
    input   [7:0]   b,
    input           cin,       
	output  [7:0]   sum,
    output          cout               
);  	  

    wire    carryout_l0;
    wire    carryout_l1;
    wire    carryout_l2;
    wire    carryout_l3;
    wire    carryout_l4;
    wire    carryout_l5;
    wire    carryout_l6;
	
	full_add_1b  inst_0_full_add_1b (
		.a		(a[0]),
		.b		(b[0]),
		.cin	(cin),
		.sum	(sum[0]),
		.cout	(carryout_l0)
	);

	full_add_1b  inst_1_full_add_1b (
		.a		(a[1]),
		.b		(b[1]),
		.cin	(carryout_l0),
		.sum	(sum[1]),
		.cout	(carryout_l1)
	);

	full_add_1b  inst_2_full_add_1b (
		.a		(a[2]),
		.b		(b[2]),
		.cin	(carryout_l1),
		.sum	(sum[2]),
		.cout	(carryout_l2)
	);

	full_add_1b  inst_3_full_add_1b (
		.a		(a[3]),
		.b		(b[3]),
		.cin	(carryout_l2),
		.sum	(sum[3]),
		.cout	(carryout_l3)
	);

	full_add_1b  inst_4_full_add_1b (
		.a		(a[4]),
		.b		(b[4]),
		.cin	(carryout_l3),
		.sum	(sum[4]),
		.cout	(carryout_l4)
	);

	full_add_1b  inst_5_full_add_1b (
		.a		(a[5]),
		.b		(b[5]),
		.cin	(carryout_l4),
		.sum	(sum[5]),
		.cout	(carryout_l5)
	);

	full_add_1b  inst_6_full_add_1b (
		.a		(a[6]),
		.b		(b[6]),
		.cin	(carryout_l5),
		.sum	(sum[6]),
		.cout	(carryout_l6)
	);

	full_add_1b  inst_7_full_add_1b (
		.a		(a[7]),
		.b		(b[7]),
		.cin	(carryout_l6),
		.sum	(sum[7]),
		.cout	(cout)
	);
	
endmodule
```

当然，verilog并不需要我们做这么多工作，因此我们可以通过最简单的方式去写一个加法器  

```verilog
module add_8b (
    input   [7:0]   a,
    input   [7:0]   b,     
	output  [7:0]   sum             
);  	           	
	
    assign  sum = a + b;
	
endmodule
```

是不是感觉少了什么？进位信号消失了，因为超出了sum的位宽。我们可以通过拓展输出位宽来实现进位信号  

```verilog
module add_8b (
    input   [7:0]   a,
    input   [7:0]   b,     
	output  [7:0]   sum，
    output          cout             
);  	           	
	
    assign  {cout, sum} = a + b;
	
endmodule
```

也可以通过IP核去实现加法器，这是工程上比较常用的方法，时序和面积往往更为优秀，也拥有更好的控制能力。

## 并行进位加法器

回过头来想一下，上一章实现的8位加法器，位于最末尾的器件经历了多少级逻辑门才能计算出来结果？  

尝试根据第三章PPT中的并行进位加法器计算式实现一下吧，参考RTL图分析最末尾的器件延时减少了多少。  

## 加减法器

现在我们想通过一个加法器完成加减法运算，应该如何实现呢？答案是使用补码。  

于是现在思路就很清晰了，首先我们需要一个运算标志位标志进行加法还是减法，还需要对操作数进行求补运算，最后输出结果。关键点就在于求补。  

我们之前学的求补运算是当原码为负数时，将原码取反加1，取反比较简单，关键是加1，这时想想上面的加法器推导过程，是不是意识到了一点，这里不涉及进位运算，因此可以直接使用异或运算代替加1运算。因此我们的加减法器结构图就呼之欲出了。  

## 移位器

移位在计算机中是非常重要的运算，它可以替代大量的常数乘法，减少运算时间。  

先举个例子，十进制下的114，乘10的结果是1140，乘100的结果就是11400，相当于分别将114向左移动1、2位，同理，忽略小数时，除法也可以通过向右移位来代替。这在二进制运算中也是一致的，有兴趣可以自己举例试试。  

移位运算的实现方式通常有两种：移位寄存器与桶形移位器。  

### 移位寄存器

移位寄存器通过D触发器序列组成，根据输入端的数据不同分为循环移位、逻辑移位、算术移位。  

- 循环移位：将移出去的那一位补充到空出的最高/低位  
- 逻辑移位：空缺处补0
- 算术移位：保证符号位不改变，算术左移同逻辑左移一样，算术右移最左面的空位补符号位

### 桶型移位器

在CPU中，我们往往需要对数据进行移位操作。但是传统的移位寄存器一个周期只能移动一位，当要进行多位移位时需要多个时钟周期，效率较低。桶形移位器采用组合逻辑的方式来实现同时移动多位，在效率上优势极大。因此，桶形移位器常常被用在ALU中来实现移位。  

桶型移位器的实质是通过列举所有的移位可能，通过复用器来选通对应的移位结果。  

## ALU  

RISC-V基础指令集RV32I只支持32位整型数值的操作。操作数可以是带符号补码整数或无符号数。ALU不需要完成乘除法，不需要进行溢出判断，相关操作由软件来完成。RV32I的ALU需要完成以下操作：

- 加减法操作 ：完成带符号补码数和无符号数的加减法操作，无需判断溢出和进位。此条件下，可以统一处理带符号数和无符号数。
- 逻辑运算 ：完成XOR、AND及OR操作，其他操作采用软件实现。例如，NOT可以使用与全一操作数进行XOR实现。
- 移位运算 ：完成逻辑左移，逻辑右移、算术右移等功能。移位运算将在后面的移位寄存器实验中介绍。
- 比较运算 ：完成带符号数与无符号数的全等和大小比较。此类运算均可利用减法实现。

## 练习

设计一个能实现如下功能的4位带符号位的<b>补码</b>ALU：

|操作码|功能|操作|  
|:--:|:--:|:--:|
|000|加法|A+B|
|001|减法|A-B|
|010|取反|Not A|
|011|与|A and B|
|100|或|A or B|
|101|异或|A xor B|
|110|比较大小|if A<B then output = 1; else output = 0;|
|111|判断相等|if A==B then output = 1; else output = 0;|

请注意操作数溢出的情况