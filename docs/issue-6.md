# EDA学习总结

# EDA概述

- 了解EDA是什么？用来干什么？
- 我学这门课的目的是什么？要做到什么水平？

## EDA： Electronic Design Automation 电子设计自动化

## 如何理解？

以计算机为工具，基于EDA软件平台，采用原理图或者硬件描述语言完成设计输入，并由计算机自动完成逻辑综合，优化，布局布线，仿真，直至对目标芯片（CPLD，FPGA）的适配和编程下载的工作。

## 数字系统设计（eda是包含在其中的一种方法）

数字系统实现依赖两种器件：

1. PLD：可编程逻辑器件 programmable logic device 包括CPLD 和 FPGA
2. ASIC：专用集成电路 application specific integrated circuit

_**基于FPGA/CPLD的数字系统设计流程**_

设计输入

- 原理图
- HDL文本

* 综合：synthesis  自动将抽象的**行为级**描述转到**RTL**（结构描述），再到**逻辑门级**（包括触发器），再到**版图表示**（ASIC）/**配置网表**（PLD）

* 布局布线：Place&Route / Fitting

* 仿真：simulation 分为 Function simulation and Timing Simulation

## top-down 和 bottom-up

- top-down:从行为级/RTL级开始描述（从想要的结果开始）
- bottom-up:从底层元件（如：加法器）开始 （从手里有什么开始）


# Verilog HDL语言要素

## 空白符

提高可读性

## 注释符

```verilog
//:单行注释
/* */：多行注释
注意注释最好用英文
```

## 标识符（identifier）

可以是任意一组字母、数字、**$** 符号和 **_**(下划线)符号的组合，但标识符的**第一个字符必须是字母或者下划线**，不能以数字或者美元符开始。

另外，标识符是**区分大小写**的。

```verilog
reg [3:0] counter ; //reg 为关键字， counter 为标识符
input clk; //input 为关键字，clk 为标识符
input CLK; //CLK 与 clk是 2 个不同的标识符
```

## 关键字

所有关键字**均小写**

```verilog
ALWAYS 与 always 不同
```

## 数值

### 数值种类

Verilog HDL 有下列**四种**基本的值来表示硬件电路中的电平逻辑：

- 0：逻辑 0 或 "假"
- 1：逻辑 1 或 "真"
- x 或 X：不定
- z 或 Z：高阻

### 整数及其表示

 **错误表示**

4'd-4 数值不能为负，有负号应放最左边 3' b001 '和基数处之间不允许出现空格 (4+4)b11 位宽不能是表达式形式（在位宽和'之间，以及进制和数值之间允许出现空格，但'和进制之间，数值间是不允许出现空格的）

## 数据类型

### 1.线网型（net型）

<aside> 💡 最常用： wire 只有在两种情况下被赋值：

1. 结构描述中一个门元件或模块的输出端
2. 持续赋值语句assign

</aside>

### 2.寄存器型（register型/variable型）

```verilog
1.reg  //可综合
reg[3:0]  =  reg[4:1]   
2.integer   //可综合
//常用于做循环变量
```

### 3.向量

- 标量与向量：

<aside> 💡 线宽大于1位的变量（包括net型和variable型）称为向量（vector）。向量的宽度用下面的形式定义： [msb : lsb] 比如： wire[3:0] bus; //4位的总线

</aside>

```verilog
//注意向量的方向：
reg[7:0]  a; //a[7]为最高有效位
reg[0:7]  b; //b[0]为最高有效位
```

- 位选择与域选择：

标量类向量：scalared **可以**进行位选择和域选择 向量类向量：vectored**不可以**进行位选择和域选择

```verilog
A=mybyte[6];          //位选择
B=mybyte[5:2];       //域选择
```

_默认为标量类_

### 4.参数

在Verilog语言中，用参数parameter来定义符号常量，即用parameter来定义一个标志符代表一个常量。参数常用来定义时延和变量的宽度。

相当于C中的define来定义常量

### 运算符和表达式

- 算术运算符：

加法(+)；减法(-)；乘法(*)；除法(/)：取模(%) (1)算术操作结果的位宽 算术表达式结果的长度由最长的操作数决定。在赋值语句下，算术操作结果的长度由操作**左端目标长度**决定。

```verilog
例：
reg[3:0]A,B,C;
reg[5:0]D;
A=B+C;  //四位
D=B+C;  //六位
```

- 等式运算符： == 相等： 相等两边若有一个含有x，则结果为x 比较的是数值在大小上相等不相等

* 全等： 全等是逐个按位比较，若“x=x”结果为1 位数不相等结果为0

```
例：
a=4'b0xx1;
b=4'b0xx1;
c=4'b0011;
d=2'b11;
$display(a==b)  //以结果为不定值x
$display(c==d)  //结果为真1
$display(a===b)  //结果为真1
$display(c===d)  //结果为假0
```

- 逻辑运算符：

&&   ||   ！
操作数不止一位时，看作整体，有一位是1整体为1


- 按位操作符

```verilog
~ & | ^  ~^
```

- 归约/缩减操作符

```verilog
& ~& | ~| ^~
//和自己本身从低位到高位逐位运算
```


- 位拼接运算符：

拼接操作符用大括号 `**{}**` 来表示，用于将多个操作数（向量）拼接成新的操作数（向量），信号间用逗号隔开。
不代表任何电路


# 数据流建模

_**结构 = assign + 运算符**_

可以描述所有的**组合逻辑电路**

# 连续赋值语句

verilog有两种赋值语句，一种用在这里，continuous assignments；一种用在过程语句中

- 显式赋值： 定义和赋值分开
- 隐式赋值： 定义和赋值在一起

```verilog
//显式赋值
module example1 assignment (a,b,m,n,c,y);
input[3:0]a,b,m,n;
output[3:0]c,y;
wire[3:0]a,b,m,n,c,y;
assign y=m|n;
assign#(3,2,4)c=a&b:
endmodule

//隐式赋值
module example2 assignment(a,b,m,n,c,y,w);
input[3:0]a,b,m,n;
output[3:0]c，y,w;
wire[3:0]a,b,m,n;
wire[3:0]y= m|n;
wie[3:0](3,2,4)c=a&b
wire(strongo,weak1)[3:0]#(2,1,3)w=(a b)&(m n):
endmodule
```

<aside> 💡 连续赋值语句需要注意的以下几点：

1. 赋值目标只能是线网类型;
2. 在连续赋值中，只要赋值语句右边表达式任何一个变量有变化，表达式 立即被计算，计算的结果立即赋给左边信号（若没有定义延时量）;
3. 连续赋值语句不能出现在过程块中（过程对应inital always）
4. 多个连续赋值语句之间是并行语句，因此与位置顺序无关
5. 连续赋值语句中的延时具有硬件电路中惯性延时的特性，任何小于其延时的信号变化脉冲都将被滤除掉，不会体现在输出端口上 </aside>

# 行为级语句
包括赋值语句，条件语句，循环语句

## initial语句

_**不可综合，一般用于仿真**_

initial 语句从 0 时刻开始执行，只执行一次，多个 initial 块之间是相互独立的。

如果 initial 块内包含多个语句，需要使用关键字 begin 和 end 组成一个块语句。

```verilog
`timescale 1ns/1ns  // 1ns/1ns ---- 时间单位/时间精度
module demo;
  reg clk,set;
  initial
  begin
    #0 set = 1'b1;clk = 1'b0;
    #100 set = 1'b0;
    #100 set = 1'b1;
    #100 set = 1'b0;
    #200 set = 1'b1;
  end
  always #50 clk = ~clk;
endmodule
```

## always语句

从语法描述角度，相对于initial过程块，always语句块的**触发状态是一直存在的**，只要满足always后面的敏感事件列表，就执行过程块 其语法格式是： always(@(<敏感事件列表>) 多个敏感信号可以用`or`或者`，`隔开 语句块 例如：

```verilog
@(a) //当信号a的值发生改变时
@(a or b) (a,b)//当信号a或信号b的值发生改变时  
@(posedge clock) //当cock的上升沿到来时
@(negedge clock) //当clock的下降沿到来时
@(posedge clk or negedge reset) //当ck的上升沿到来或reset信号的下降沿到来时
```

<aside> 💡 注意：

在信号定义形式方面，无论是对时序逻辑还是组合逻辑描述，Verilog HDL要求在过程语句(initial和always)中，被赋值信号必须定义为"reg"类型 (1)采用过程对组合电路进行描述时，作为全部的输入信号需要列入敏感信号列表。 (2)采用过程对时序电路进行描述时，需要把时间信号和部分输入信号列入敏感信号列表。应当注意的是，不同的敏感事件列表会产生不同的电路形式。

</aside>

```verilog
module count (out,data,load,reset,clk);
input load,clk,reset; 
input[7:0] data;
output reg[7:0] out; 

always @(posedge clk)
   begin
   if(!reset) out=8'h00;
   else if(load) out=data;
   else	 	out=out+1;
   end
endmodule

module count (out,data,load,reset,clk);
input load,clk,reset; 
input[7:0] data;
output reg[7:0] out; 

always @(posedge clk or negedge reset)
   begin
   if(!reset) out=8'h00;
   else if(load) out=data;
   else	 	out=out+1;
   end
endmodule
```

# 语句块

### **顺序块**

顺序块用关键字 begin 和 end 来表示。

顺序块中的语句是一条条执行的。当然，非阻塞赋值除外。

顺序块中每条语句的时延总是与其前面语句执行的时间相关。

### **并行块**

并行块有关键字 fork 和 join 来表示。

并行块中的语句是并行执行的，即便是阻塞形式的赋值。

并行块中每条语句的时延都是与块语句开始执行的时间相关。

# 过程赋值语句

- 阻塞赋值：=

1. 按顺序执行，会堵住
2. 语句结束立即完成赋值，两个连续的赋值之间无延时

- 非阻塞赋值：<=

1. 并发执行，执行很快
2. 整个语句块结束一块儿赋值，两个连续的赋值之间有延时

_对于verilog语句，只有在行为级语句中的串行语句块（begin-end)的阻塞赋值语句才是串行的_

```verilog
module  block(c,b,a,clk);
output reg c,b;
input  clk,a;
always @(posedge clk)
  	begin
	  b=a; 
  	c=b;    	 	
  	end
endmodule
```

```verilog
module  non_block(c,b,a,clk);
output reg c,b; 
input  clk,a;
always @(posedge clk)
 	begin 
    	b<=a;
    	c<=b;
 	end
endmodule
```

# 条件分支语句

<aside> 💡 条件列举穷尽： 要注意所有的if else / switch case 条件都要列举穷尽，否则会出现输出默认为输入值，隐含锁存器。

</aside>

### **关键词：if，选择器**

### **条件语句**

条件（if）语句用于控制执行语句要根据条件判断来确定是否执行。

条件语句用关键字 if 和 else 来声明，条件表达式必须在圆括号中。

条件语句使用结构说明如下：

```verilog
if (condition1)       true_statement1 ;
else if (condition2)        true_statement2 ;
else if (condition3)        true_statement3 ;
else                      default_statement ;
```

- if 语句执行时，如果 condition1 为真，则执行 true_statement1 ；如果 condition1 为假，condition2 为真，则执行 true_statement2；依次类推。
- else if 与 else 结构可以省略，即可以只有一个 if 条件判断和一组执行语句 ture_statement1 就可以构成一个执行过程。
- else if 可以叠加多个，不仅限于 1 或 2 个。
- ture_statement1 等执行语句可以是一条语句，也可以是多条。如果是多条执行语句，则需要用 begin 与 end 关键字进行说明。

### case语句

```verilog
module vote3(input a,b,c,
			output reg pass);
always @(a,b,c)
begin
	case({a,b,c})
	3'b000,3'b001,3'b010,3'b100:pass=1'b0;
	3'b011,3'b101,3'b110,3'b111:pass=1'b1;
	default:pass=1'b0;
	endcase
end
endmodule
```

# 循环语句

### **关键词：while, for, repeat, forever**

Verilog 循环语句有 4 种类型，分别是 while，for，repeat，和 forever 循环。循环语句只能在 always 或 initial 块中使用，但可以包含延迟表达式。

# 模块例化

```verilog
full_adder1  u_adder0(
    .Ai     (a[0]),
    .Bi     (b[0]),
    .Ci     (c==1'b1 ? 1'b0 : 1'b1),
    .So     (so_bit0),
    .Co     (co_temp[0]));
              //括号内的相当于要传递的参数
```


# 结构级建模

## 门级结构描述

门元件名 <实例名> （端口列表）

```verilog
module mux4_1a(out,in1,in2,in3,in4,s0,s1);
input in1,in2,in3,in4,s0,s1; 
output out;
wire s0_n,s1_n,w,x,y,z;
not (s0_n,s0),(s1_n,s1);
and (w,in1,s0_n,s1_n),(x,in2,s0_n,s1),
 	(y,in3,s0,s1_n),(z,in4,s0,s1);
or (out,w,x,y,z);
endmodule
```

## 对比三种建模方式

数据流

```verilog
module mux4_1c(out,in1,in2,in3,in4,s0,s1);
input in1,in2,in3,in4,s0,s1; 
output out;
assign out=(in1 & ~s0 & ~s1)|(in2 & ~s0 & s1)|
           (in3& s0 & ~s1)|(in4 & s0 & s1);
endmodule
```

行为级

```verilog
module mux4_1b(out,in1,in2,in3,in4,s0,s1);
input in1,in2,in3,in4,s0,s1; 
output reg out;
  always@(*)
    case({s0,s1})
    2'b00:out=in1;
    2'b01:out=in2;
    2'b10:out=in3;
    2'b11:out=in4;
    default:out=2'bx;
    endcase
endmodule
```

## 赋值语句的使用

阻塞 or 非阻塞

<aside> 💡 阻塞： 组合逻辑电路中使用 因为阻塞为顺序执行，符合组合逻辑电路规则

非阻塞： 时序逻辑电路中的边沿触发使用 因为时序逻辑电路的赋值就是在时钟沿来临的时候才变化

</aside>

# 基本组合电路设计

组合电路的特点是：电路中任意时刻的稳态输出仅仅取决于该时刻的输入，而与电路原来的状态无关

## 设计方法一：真值表

对应

（例：三人表决器）

```verilog
module vote(OUT,A,B,C);
output OUT;
input A,B,C;
reg OUT;
always@(A or B or C)
case ({A,B,C})
3'b000:OUT=0;
3'b001:OUT=0;
3'b010:OUT=0;
3'b100:OUT=0;
3'b011:OUT=1;
3'b101:OUT=1;
3'b110:OUT=1;
3'b111:OUT=1;
endcase
endmodule
```

## 设计方法二：逻辑代数

对应数据流建模

```verilog
module vote(OUT,A,B,C);
output OUT;
input A,B,C;
assign OUT=(A&B)|(B&C)|(A&C);
endmodule
```

## 设计方法三：结构描述

```verilog
module vote(OUT,A,B,C);
output OUT;
input A,B,C;
and U1 (w1,A,B);
and U2 (w2,B,C);
and U3 (w3,A,C);
or U4 (OUT,w1,w2,w3);
endmodule
```

## 实例

### 加法器

### 数据选择器

- 二选一的数据选择器

```verilog
//数据流
module logic(sel,a,b,out);
input a,b,sel;
output out;
assign out = (sel)? a:b;
endmodule

//行为级
module logic(sel,a,b,out);
input a,b,sel;
output reg out;
always@(sel,a,b)//always@ *
begin
if(sel) out = a;
else out =b;
end
endmodule
```

- 四选一的数据选择器

```verilog
//数据流
module selct_4(s0,s1,in0,in1,in2,in3,out);
input s0,s1;
input in0,in1,in2,in3;
output out;
assign out  = (in0 & ~s0 & ~s1)|(in1 & s0 & ~s1)|
              (in2 & ~s0 & s1) |(in3 & s0 & s1);
				  
endmodule

//行为级--case
module selct_4(sel,in0,in1,in2,in3,out);
input [1:0] sel;
input in0,in1,in2,in3;
output reg out;
always@ *
begin
case (sel)
2'b00:out<=in0;
2'b01:out<=in1;
2'b10:out<=in2;
2'b11:out<=in3;
default:out<=1'bx;
endcase
end
endmodule

//行为级--if
module logic(sel,in0,in1,in2,in3,out);
input [1:0] sel;
input in0,in1,in2,in3;
output reg out;
always@ *
begin
if(sel == 2'b00) out=in0;
else if (sel == 2'b01) out=in1;
else if (sel == 2'b10) out=in2;
else if (sel == 2'b11) out=in3;
else out<=1'bx;
end
endmodule
```

# 有限状态机设计

有限状态机：组合逻辑和寄存器逻辑的特殊组合

## 分类

### 摩尔型

输出仅由当前状态决定

### 米里型

输出由当前状态和输入信号共同决定

# 描述方式

## 三段式：

用三个过程描述：即现态（CS）、次态（NS）、输出逻辑（OL）各用一个always过程描述。

- 例：“101”序列检测器

```verilog
module fsm1_seq101(clk,clr,x,z);
input clk,clr,x; 
output reg z; 
reg[1:0] state,next_state;
parameter 	S0=2'b00,S1=2'b01,S2=2'b11,S3=2'b10;

always @(posedge clk or posedgesa clr) //CS
  begin	if(clr) state<=S0;       
		else state<=next_state;
  end
always @(state or x) //NS
  begin
    case (state)
	  S0:
	      begin
	      if(x) next_state<=S1;
	      else  next_state<=S0; 
			  end
	  S1:
	      begin	
			if(x) next_state<=S1;
	      else  next_state<=S2;
			end
     S2:
	      begin
	      if(x) next_state<=S3;
	      else next_state<=S0; 
			end
     S3: 
	      begin	
			if(x) next_state<=S1;
	      else 	next_state<=S2;
			end
     default:	next_state<=S0;
	 endcase
  end
    
always @(state) //OL
  begin  
    case(state)
		S3: z=1'b1;
		default:z=1'b0;
    endcase
  end
endmodule
```

