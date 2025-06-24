# vcs+verdi使用介绍与案例讲解

参考链接：https://blog.csdn.net/JasonFuyz/article/details/107508893

https://www.bilibili.com/video/BV1hX4y137Ph

## 1 vcs+verdi使用介绍

### 1.1 vcs+verdi安装

vcs+verdi是常用的FPGA仿真软件，它需要在linux系统中使用，所以一般情况需要在虚拟机上部署。

具体的部署流程十分漫长，本人是跟着b站视频号**BV1x88rebEKC**后面进行安装。

对于安装有一些个人的建议：第一遍安装时，对于用户的名称和各个文件的位置尽量和视频的一致，等到之后安装熟练了就可以按自己的喜好安装了。

安装过程中也出现一些问题，我的问题是在安装到最后出现[ERROR] Could not checkout Verdi license. Use verdi -licdebug for more information问题，最后我看网上的解答说是没有将虚拟机和主机的时间对其，之后我调整了时间再重启就解决问题了。

如果遇到不能解决的问题，可以多去csdn上搜索一下相关的问题，或者去问问AI，不过AI的建议仅供参考，不一定要按照AI的建议修改，AI有时候会让你修改一些很底层的东西，导致你有些其他的功能无法使用。

### 1.2 verdi的使用

如果按照1.1推荐的视频进行安装的话，想要开启verdi，需要先进入终端，输入**lmg_synopsys**，之后输入verdi即可进入verdi界面。
![example picture](/images/verdi1.png)
<div style="text-align: center;">
verdi界面
</div>

接下来对verdi的使用进行讲解。

![example picture](/images/verdi2.png)
根据该图指示先点击左上角的file，之后点击Import Design...，进入选择界面。

![example picture](/images/verdi3.png)
根据这四步将你的文件导入到verdi中。

![example picture](/images/verdi4.png)
根据这三步来开始导入你的波形图。

![example picture](/images/verdi5.png)
根据这两步导入你需要观测的波形图，注意第一步需要双击。
不过双击后仍然没有波形图出现。所以还需要最后一步。

![example picture](/images/verdi6.png)
根据这两步即可成功输出波形。

## 2 vcs+verdi联合使用实例

### 2.1 反相器

反相器是一种基本且至关重要的​​逻辑门​​，它的核心功能是将取值取反，如果取值是0，则经过反相器之后变为1，如果取值不为0，则经过反相器后变为0，其真值表为：
|输入|输出|
|----|----|
| 0 | 1 |
| 1 | 0 |

接下来进行反相器的vcs+verdi实例介绍。

1.先在虚拟机的桌面创建一个文件夹，用来放置接下来要使用的文件。如果对于Linux的操作指令不熟悉的话可以直接右键创建，也可以用以下代码实现：
```verilog
cd Desktop     //进入桌面
mkdir Study  //创建名为Study的文件夹
cd Study    //进入Study
```
如果系统装了gvim就用gvim指令，如果没有就用vim指令。接下来使用统一的vim指令进行。

2.在Study文件夹里新建inv.v，tb_inv.v和timescale.v文件，其步骤与内容如下：
```verilog
vim inv.v
```
```verilog
module inv(
		A,
		Y
		);
input		A;
output		Y;
assign		Y=~A;
endmodule
```
完成inv.v的创建与内容编辑

```verilog
vim tb_inv.v
```
```verilog
//testbench
module inv_tb;
reg		aa;
wire		yy;
inv 		inv(
			.A(aa),
			.Y(yy)
			);

initial begin
		aa<=0;      //reg类型的的变量用<=
	#10	aa<=1;
	#10	aa<=0;
	#10	aa<=1;
	#10	$finish;
end

`ifdef FSDB
initial begin
	$fsdbDumpfile("tb_inv.fsdb");
	$fsdbDumpvars;
end
`endif

endmodule
```
完成tb_inv.v的创建与内容编辑。

**注**：要VCS与Verdi联合仿真，需要在testbench里面必须加入`ifdef FSDB到endif`的代码，这样才能生成fsdb文件提供Verdi读取，不然不会输出波形。

```verilog
vim timescale.v
```
```verilog
`timescale 1ns/10ps
```
完成timescale.v的创建与内容编辑。

之后在你文件的路径下运行
```verilog
vcs -R -full64 +v2k -fsdb +define+FSDB -sverilog inv.v tb_inv.v timescale.v -l run.log
```

之后在终端输入verdi，按照第1章的讲解进行操作即可获取波形，该代码的波形结果为：
![example picture](/images/inv1.png)

此外，本文分享一个测试起来更快（可能）的方法。

常规方法你需要复制vcs -R ... run.log一长串的代码，但是有一种方法可以不用打出每一个文件的名称，具体方法如下：
```verilog
cd Desktop/Study
vim file.f
```
file.f的内容为：
```verilog
tb_inv.v inv.v timescale.v
```
这样file.f的文件内就包含了这三个文件的名称，之后在终端输入：
```verilog
vcs -R -full64 +v2k -fsdb +define+FSDB -sverilog -f file.f -l run.log
```
当然，还有更简便的用法，不过我暂时没有研究，有待后续补充。

### 2.2 与非门

在数字逻辑电路中，​​与非门​​（NAND Gate）是最重要、最常用的​​复合逻辑门​​之一。它的名称来源于其功能：​​先执行“与”操作，再执行“非”操作​​。其真值表为：
|输入A|输入B|输出Y|
|----|----|----|
| 0 | 0 | 1 |
| 1 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 1 | 0 |

接下来我将各个部分代码贴出，具体测试流程与2.1一致。
nand.v:
```verilog
module nand_gate(
		A,
		B,
		Y
		);
input		A;
input		B;
output		Y;

assign		Y=~(A&B); //先与后非
endmodule
```

tb_nand.v:
```verilog
//testbench
module nand_gate_tb;
reg		aa,bb;
wire		yy;
nand_gate 	nand_gate(
			.A(aa),
			.B(bb),
			.Y(yy)
			);

initial begin
		aa<=0;bb<=0;
	#10	aa<=0;bb<=1;
	#10	aa<=1;bb<=0;
	#10	aa<=1;bb<=1;
	#10	$finish;
end

`ifdef FSDB
initial begin
	$fsdbDumpfile("tb_nand.fsdb");
	$fsdbDumpvars;
end
`endif

endmodule
```

定时与2.1一致。
最后结果展示：
![example picture](/images/nand1.png)

### 2.3 四位与非门

四位与非门是与非门​的进阶版，具体是什么意思举个例子就可以：
A = 4'b0011
B = 4'b1110
C = ~(A&B) = 4'b1101
四位与非门就是将每一位进行与非计算，最后和到一起就可以。

接下来我将各个部分代码贴出，具体测试流程与2.1一致。
nand_4bits.v:
```verilog
module nand_gate_4bits(
		A,
		B,
		Y
		);
input[3:0]	A;
input[3:0]	B;
output[3:0]	Y;

assign		Y=~(A&B); //先与后非
endmodule
```

tb_nand_4bits.v:
```verilog
//testbench
module nand_gate_4bits_tb;
reg[3:0]	aa,bb;
wire[3:0]	yy;
nand_gate_4bits 	nand_gate_4bits(
			.A(aa),
			.B(bb),
			.Y(yy)
			);

initial begin
		aa<=4'b0000;bb<=4'b1111;
	#10	aa<=4'b0010;bb<=4'b0110;
	#10	aa<=4'b0111;bb<=4'b0100;
	#10	aa<=4'b1111;bb<=4'b1110;
	#10	$finish;
end

`ifdef FSDB
initial begin
	$fsdbDumpfile("tb_nand.fsdb");
	$fsdbDumpvars;
end
`endif

endmodule
```

定时与2.1一致。
最后结果展示：
![example picture](/images/nand2.png)


### 2.4 选择器

选择器是数字逻辑电路中常见的一种器件，其通常的效果是当达成某种条件就进行某些操作，这里放一种常见的选择器。
![example picture](/images/fn.png)

该模块fn_sw的功能为：

当sel为0时y是a与b的与；
当sel为1时y是a与b的异或。

接下来我将各个部分代码贴出，具体测试流程与2.1一致。
fn_sw.v:
```verilog
module fn_sw(
		a,
		b,
		sel,
		y
		);
input		a;
input		b;
input		sel;
output		y;

reg		y;	//这里是因为always里赋值的变量必须是reg型

always@(a or b or sel)
begin
	if(sel==1) begin
		y<=a^b;
	end
	else begin
		y<=a&b;
	end
end
		//上面这段always代码也可以改成 
		//assign  y=sel?(a^b):(a&b);
endmodule
```

tb_fn_sw.v:
```verilog
module	fn_sw_tb;
reg		a,b,sel;
wire		y;
fn_sw fn_sw(
		.a(a),
		.b(b),
		.sel(sel),
		.y(y)
		);

initial begin
		a<=0;b<=0;sel<=0;
	#10	a<=0;b<=0;sel<=1;
	#10	a<=0;b<=1;sel<=0;
	#10	a<=0;b<=1;sel<=1;
	#10	a<=1;b<=0;sel<=0;
	#10	a<=1;b<=0;sel<=1;
	#10	a<=1;b<=1;sel<=0;
	#10	a<=1;b<=1;sel<=1;
	#10	$finish;
end


`ifdef FSDB
initial begin
	$fsdbDumpfile("tb_fn_sw.fsdb");
	$fsdbDumpvars;
end
`endif

endmodule
```

定时与2.1一致。
最后结果展示：
![example picture](/images/fn2.png)

### 2.5 四位选择器

四位选择器选择器是选择器的一种进阶功能，本次设计的四位选择器fn_sw_4功能为：

当sel为00时y是a与b的与；
当sel为01时y是a与b的或；
当sel为10时y是a与b的异或；
当sel为11时y是a与b的同或。

接下来我将各个部分代码贴出，具体测试流程与2.1一致。
fn_sw_4.v:
```verilog
module fn_sw_4(
		a,
		b,
		sel,
		y
		);
input		a;
input		b;
input[1:0]	sel;
output		y;

reg		y;

always@(a or b or sel)
begin
	case(sel)
	2'b00:begin y<=a&b;end
	2'b01:begin y<=a|b;end
	2'b10:begin y<=a^b;end
	2'b11:begin y<=~(a^b);end
	endcase
end

endmodule
```

tb_fn_sw.v:
```verilog
module	fn_sw_4_tb;
reg[3:0]	absel;
wire		y;
fn_sw_4 fn_sw_4(
		.a(absel[0]),
		.b(absel[1]),
		.sel(absel[3:2]),
		.y(y)
		);

initial begin
		absel<=0;
	#200	$stop;
end

always #10 absel<=absel+1;

`ifdef FSDB
initial begin
	$fsdbDumpfile("tb_fn_sw_4.fsdb");
	$fsdbDumpvars;
end
`endif

endmodule
```

注：#200	$stop;这句话也可以改成#200 $finish，如果用的stop，则需要在终端卡住的时候输入finish手动结束，不过我看网上当说使用stop的时候应该输入run或者run-all，但是当我输入run的时候会一直进行，然后怎么都退不掉，所以个人建议还是使用finish为好，至于原因个人不太清楚，感兴趣可以去看看。

定时与2.1一致。
最后结果展示：
![example picture](/images/fn3.png)

这个结果在使用该代码进行测试的时候不会出现absel这一栏的结果，这个结果主要是为了看他的值是多少，有没有将所有的结果遍历。想要出现以上的结果需要按照以下步骤进行：
![example picture](/images/fn4.png)

### 2.6 原码转换补码

正数补码与原码相同；

负数补码转换方法是符号位不变，幅度按位取反加1.

所以对于转换方式有个比较大致的思路。首先先判断这个数是正数还是负数，正数的话结果就是原结果，负数的话结果就先将除符号位的其他几位取反加1，最后再加上符号位。

接下来我将各个部分代码贴出，具体测试流程与2.1一致。
fn_sw_4.v:
```verilog
module comp_conv(
		a,
		a_comp
		);
input[7:0]	a;
output[7:0]	a_comp;

wire[6:0]	b;//按位取反的幅度位
wire[7:0]	y;//负数的补码

assign		b=~a[6:0];
assign		y[6:0]=b+1;//按位取反+1
assign		y[7]=a[7];//符号位不变

assign		a_comp=a[7]?y:a;


endmodule
```

tb_comp_conv.v:
```verilog
module comp_conv_tb;
reg[7:0]	a_in;
wire[7:0]	y_out;
comp_conv	comp_conv(
			.a(a_in),
			.a_comp(y_out)
			);

initial begin
		a_in<=0;
	#3000	$finish;
end

always #10 a_in<=a_in+1;


`ifdef FSDB
initial begin
	$fsdbDumpfile("tb_comp_conv.fsdb");
	$fsdbDumpvars;
end
`endif

endmodule
```

定时与2.1一致。
最后结果展示：
![example picture](/images/comp1.png)

![example picture](/images/comp2.png)

有的人的表示可能是十六进制的，这样不方便看原码和补码到底是什么，想要变成二进制的话有可以按住Alt键，之后依次点击W、R、B能选择二进制。也可以按照下图的方式进行点击：
![example picture](/images/comp3.png)

### 2.7 七段码译码器

七段码译码器就是大家熟悉的数码管。
![example picture](/images/dec1.png)

他的用出有很多，不如说红绿灯、定时器、指示灯等等，其由七条线以及一个点组成，我们将最上面的一横记作a，之后顺时针依次为b,c,d,e,f，最中间的记为g。

接下来我将各个部分代码贴出，具体测试流程与2.1一致。
seg_dec.v:
```verilog
module seg_dec(
		num,
		a_g
		);
input[3:0]	num;
output[6:0]	a_g;

reg[6:0]	a_g;//a_g={a,b,c,d,e,f,g}
always @(num) begin
	case(num)
	4'd0:begin a_g<=7'b1111110;end
	4'd1:begin a_g<=7'b0110000;end
	4'd2:begin a_g<=7'b1101101;end
	4'd3:begin a_g<=7'b1111001;end
	4'd4:begin a_g<=7'b0110011;end
	4'd5:begin a_g<=7'b1011011;end
	4'd6:begin a_g<=7'b1011111;end
	4'd7:begin a_g<=7'b1110000;end
	4'd8:begin a_g<=7'b1111111;end
	4'd9:begin a_g<=7'b1111011;end
	default:begin a_g<=7'b0000001;end
	endcase
end

endmodule
```

tb_seg_dec.v:
```verilog
module seg_dec_tb;
reg[3:0]	num;
wire[7:0]	a_g;
seg_dec		seg_dec(
			.num(num),
			.a_g(a_g)
			);

initial begin
		num<=0;
	#100	$finish;
end

always #10 num<=num+1;


`ifdef FSDB
initial begin
	$fsdbDumpfile("tb_seg_dec.fsdb");
	$fsdbDumpvars;
end
`endif

endmodule
```

定时与2.1一致。
最后结果展示：
![example picture](/images/dec.png)

### 2.8 计数器

计数器，顾名思义就是计数用的，这次的设计是时钟信号没经过一次上升沿计数器就加一。

接下来我将各个部分代码贴出，具体测试流程与2.1一致。
counter.v:
```verilog
module counter(
		clk,
		res,
		y

		);
input		clk;
input		res;
output[7:0]	y;

wire[7:0]	sum;//+1yun suan jie guo
assign		sum=y+1;//zu he luo ji bu fen
reg[7:0]	y;

always@(posedge clk or negedge res)
	if(~res)begin
		y<=0;
	end
	else begin
		y<=sum;
	end

endmodule
```

tb_counter.v:
```verilog
module	counter_tb();
reg		clk,res;
wire[7:0]	y;		
counter counter(
		.clk(clk),
		.res(res),
		.y(y)
		);

initial begin
		clk<=0;	res<=0;
	#17	res<=1;
	#6000	$finish;
end

always	#5 clk<=~clk;

`ifdef FSDB
initial begin
	$fsdbDumpfile("tb_counter.fsdb");
	$fsdbDumpvars;
end
`endif

endmodule
```

定时与2.1一致。
最后结果展示：
![example picture](/images/counter1.png)

### 2.9 4级伪随机码发生器

**伪随机码发生器**是一种​​电子设备或算法​​，它能生成一个​​看似随机​​的比特或符号序列，但实际上这个序列是由​​确定性的算法​​生成的，并且可以​​完全重复​​。

4级伪随机码发生器是由4个伪随机码发生器级联而成，本次所设计的4级伪随机码发生器具体形状如下图所示：
![example picture](/images/gen1.png)

接下来我将各个部分代码贴出，具体测试流程与2.1一致。
gen.v:
```verilog
module m_gen(
		clk,
		res,
		y
		);

input		clk;
input		res;
output		y;

reg[3:0]	d;
assign		y=d[0];


always@(posedge clk or negedge res)
	if(~res)begin
		d<=4'b1111;
	end
	else begin
		d[2:0]<=d[3:1];//右移一位
		d[3]<=d[3]+d[0];//模2加
	end

endmodule

```

tb_gen.v:
```verilog
module	m_gen_tb();
reg		clk,res;
wire		y;		
m_gen m_gen(
		.clk(clk),
		.res(res),
		.y(y)
		);

initial begin
		clk<=0;	res<=0;
	#17	res<=1;
	#600	$finish;
end

always	#5 clk<=~clk;

`ifdef FSDB
initial begin
	$fsdbDumpfile("tb_gen.fsdb");
	$fsdbDumpvars;
end
`endif

endmodule
```

定时与2.1一致。
最后结果展示：
![example picture](/images/gen2.png)

### 2.10 秒计数器

秒计数器的计数范围是0-9，假设clk是24MHz系统时钟，秒分频产生秒脉冲s_pulse；秒技术模块对秒脉冲计数，计数范围是0-9，秒计数结果是s_num（位宽4）。

接下来我将各个部分代码贴出，具体测试流程与2.1一致。
gen.v:
```verilog
module s_counter(
    clk,
    res,
    s_num
);
input    clk;
input    res;
output[3:0]    s_num;

parameter    frequency_clk=24;  // 24MHz
reg[24:0]    con_t;
reg        s_pulse;  // 秒脉冲信号
reg[3:0]    s_num;

always@(posedge clk or negedge res) begin  // 添加begin包裹整个always块
    if(~res) begin
        con_t <= 0;
        s_pulse <= 0;
        s_num <= 0;
    end
    else begin
        // 计数器逻辑
        if(con_t == frequency_clk*1000000 - 1) begin
            con_t <= 0;
        end
        else begin
            con_t <= con_t + 1;
        end

        // 秒脉冲生成
        if(con_t == 0) begin
            s_pulse <= 1;
        end
        else begin
            s_pulse <= 0;
        end

        // 秒计数器逻辑 - 修改为单独处理
        if(s_pulse) begin  // 检测到秒脉冲时清零
            s_num <= 0;
        end
        else if(s_num == 9) begin  // 普通进位逻辑
            s_num <= 0;
        end
        else begin
            s_num <= s_num + 1;
        end
    end
end  // 包裹always块的end

endmodule

```

tb_gen.v:
```verilog
module	counter_tb();
reg		clk,res;
wire[3:0]	s_num;		
s_counter s_counter(
		.clk(clk),
		.res(res),
		.s_num(s_num)
		);

initial begin
		clk<=0;	res<=0;
	#17	res<=1;
	#1000	$finish;
end

always	#5 clk<=~clk;

`ifdef FSDB
initial begin
	$fsdbDumpfile("tb_s_counter.fsdb");
	$fsdbDumpvars;
end
`endif

endmodule
```

定时与2.1一致。
最后结果展示：
![example picture](/images/s_counter1.png)

还有一些实例暂时还未复刻，等复刻之后再进行补充。

## 3 具体目标狱程序设计

### 3.1 FIFO

#### 3.1.1 目标任务

Verilog 编程练习：设计一个参数化的同步FIFO
目标：
掌握同步时序逻辑设计。
理解FIFO（First-In, First-Out，先入先出队列）的工作原理。
学习使用参数（parameter）进行模块化和可重用设计。
掌握基本状态信号（空、满）的产生逻辑。
练习编写简单的Verilog测试平台（Testbench）。

题目描述：
设计一个同步FIFO模块。该FIFO应具有可配置的数据宽度（DATA_WIDTH）和深度（DEPTH）。FIFO的操作应严格遵循时钟信号，并提供“满”（full）和“空”（empty）状态指示信号。

设计要求：
1.参数化：
使用 parameter 定义数据宽度 DATA_WIDTH (默认值 8)。
使用 parameter 定义FIFO深度 DEPTH (默认值 16)。假设 DEPTH 总是2的幂次方，以便简化地址指针逻辑。
2.接口（Ports）：
|  端口名  |   方向	 |   位宽	|  描述  |
|---|---|----|----|
|clk|input	|1	   |时钟信号|
|rst_n	|input	|1	   |异步复位信号，低电平有效|
|wr_en	|input	|1	   |写使能信号|
|w_data	|input	|DATA_WIDTH	|写入FIFO的数据|
|rd_en	|input	|1	|读使能信号|
|r_data	|output	|DATA_WIDTH	|从FIFO读出的数据|
|full	|output	|1	|FIFO满状态标志 (1 = 满, 0 = 未满)|
|empty	|output	|1	|FIFO空状态标志 (1 = 空, 0 = 非空)|

3.编程语言：基于verilog-2001和verilog-95，有适当的注释。
4.Testbench Verilog代码 (fifo_tb.v)：编写一个测试平台来验证你的FIFO设计。要求交付测试计划，测试计划中包含要测试哪些功能和收集哪些覆盖率；按照测试计划完成测试；输出测试结果文档（说明针对计划的结果）。
5.基于Xilinx的Vivado进行设计和验证，任选FPGA型号。

交付结果：
1）设计文件
2）验证文件及测试向量
3）测试计划文档和测试结果文档。

其他要求：
1）不要求统一的完成时间，每个人按照自己可投入的时间规划完成实践目标，第一周汇报的时候报一下目标完成时间
2）每周汇报进展和收获


#### 3.1.2 计数器

计数器是设计FIFO的基本格式，FIFO的设计建立在计数器的设计之上，本计数器不是第二章给出的历程中的计数器，第二章的历程只能实现累加与清零，本计数器要实现累加、清零以及累减等功能。

下面给出设计后的代码，并对代码做出详细注释。

```verilog
// 定义模块名称为"tst"
module tst
(
    // 输入输出端口定义
    input        clk,     // 系统时钟输入
    input        rst_n,   // 低电平有效的异步复位信号
    input        en,      // 计数使能信号，高电平有效
    input [31:0] cfg_max, // 32位配置的最大计数值输入
    output reg [31:0] cnt // 32位计数器输出（寄存器类型）
);

// 定义状态机的状态编码（使用参数）
parameter IDLE = 2'd0; // 空闲状态（不计数）
parameter INC  = 2'd1; // 递增计数状态
parameter DEC  = 2'd2; // 递减计数状态

// 状态寄存器声明
reg [1:0] stat;       // 当前状态寄存器
reg [1:0] stat_next;  // 下一状态逻辑（组合逻辑输出）

// 组合逻辑连线声明
wire max_pre_vld;     // 指示即将达到最大值的标志（cnt == cfg_max_limit - 1）
wire one_vld;         // 指示计数值为1的标志（cnt == 1）
wire [31:0] cfg_max_limit; // 处理后的有效最大值配置（确保至少为1）

// 处理配置最大值：如果cfg_max为0，则使用最小值1，否则使用cfg_max的值
assign cfg_max_limit = (cfg_max == 32'd0) ? 32'd1 : cfg_max;

// 状态寄存器更新（时序逻辑）
always @(posedge clk or negedge rst_n)
begin
    if (!rst_n)       // 异步复位（低电平有效）
        stat <= IDLE; // 复位到IDLE状态
    else
        stat <= stat_next; // 正常工作时更新为下一状态
end

// 下一状态逻辑（组合逻辑）
always @(*)
begin
    case(stat) // 根据当前状态决定状态转移
        IDLE: begin  // 空闲状态
            if (en)          // 当使能有效
                stat_next = INC;  // 转移到递增状态
            else
                stat_next = IDLE; // 保持空闲状态
        end
        
        INC: begin   // 递增计数状态
            if (~en)             // 当使能无效
                stat_next = IDLE;  // 返回空闲状态
            else if (max_pre_vld) // 检测到即将达到最大值
                stat_next = DEC;   // 转移到递减状态
            else
                stat_next = INC;   // 保持递增状态
        end

        DEC: begin   // 递减计数状态
            if (~en)         // 当使能无效
                stat_next = IDLE;  // 返回空闲状态
            else if (one_vld)      // 检测到计数值为1
                stat_next = INC;   // 转移到递增状态
            else
                stat_next = DEC;   // 保持递减状态
        end

        default: stat_next = IDLE; // 默认情况返回空闲状态
    endcase
end

// 计数器逻辑（时序逻辑）
always @(posedge clk or negedge rst_n)
begin
    if (!rst_n)           // 异步复位
        cnt <= 32'd0;     // 计数器清零
    else if (stat == IDLE) // 处于空闲状态
        cnt <= 32'd0;     // 计数器清零
    else if (stat == INC)  // 处于递增状态
        cnt <= cnt + 32'd1; // 计数器加1
    else if (stat == DEC)  // 处于递减状态
        cnt <= cnt - 32'd1; // 计数器减1
end

// 组合逻辑：检测是否即将达到最大值（下个时钟周期将到达cfg_max_limit）
assign max_pre_vld = (cnt == cfg_max_limit - 32'd1);

// 组合逻辑：检测计数值是否恰好为1
assign one_vld = (cnt == 32'd1);

// 模块定义结束
endmodule

```

tb代码：
```verilog
// 定义测试模块名为tb
module tb;

// 定义变量用于波形dump控制
int	fsdbDump;    
// 定义随机数种子变量
integer	seed;       

// 定义测试信号
logic	clk;       // 时钟信号
logic	rstn;      // 复位信号（低有效）
logic	[31:0] cnt;// 来自DUT的计数器输出
reg	en;          // 使能信号

// 设置时间显示格式（-9表示纳秒，3位小数）
initial	$timeformat(-9, 3, " ns", 0);

// 初始化块：处理仿真参数
initial
begin
    // 尝试从命令行获取随机种子，默认设为100
    if(!$value$plusargs("seed=%d", seed))
        seed = 100;
    $srandom(seed);    // 设置随机数种子
    $display("seed=%d\n", seed); // 显示当前种子值

    // 尝试从命令行获取波形dump标志，默认设为1（开启）
    if(!$value$plusargs("fsdbDump=%d", fsdbDump))
        fsdbDump = 1;
    if(fsdbDump)  // 如果开启波形dump
    begin
        $fsdbDumpfile("tb.fsdb");  // 创建波形文件
        $fsdbDumpvars(0);         // dump所有变量
    end
end

// 时钟生成：40MHz方波（周期25ns）
initial
begin
    clk = 1'b0;  // 初始时钟为0
    forever       // 永久循环
    begin
        #(1e9/(2.0 * 40e6)) clk = ~clk; // 12.5ns后翻转时钟（半个周期）
    end
end

// 复位信号控制
initial
begin
    rstn = 0;    // 初始复位有效
    #30 rstn = 1; // 30ns后释放复位
end

// 主测试逻辑
initial
begin
    en = 0;      // 初始使能无效
    
    // 等待复位释放
    @(posedge rstn);
    #100;        // 再等待100ns
    
    // 在下一个时钟上升沿后使能有效
    @(posedge clk);
    #1;          // 避免竞争（在时钟沿后1ns）
    en = 1;      // 使能计数

    // 运行1000ns（约40个时钟周期）
    #1000;
    
    // 在下一个时钟上升沿后禁用计数
    @(posedge clk);
    #1;
    en = 0;

    // 等待100ns后结束仿真
    #100;
    $finish;     // 结束仿真
end

// 实例化被测设计(DUT)
tst	tst(
    .clk(clk),     // 连接时钟
    .rst_n(rstn),  // 连接复位
    .en(en),       // 连接使能
    .cfg_max(5),   // 配置最大值为5（固定）
    .cnt(cnt)      // 连接计数器输出
);

endmodule
```
另外不要忘了时间的设置，我这里是和之前一样单独放在一个文件里，代码为:
```verilog
`timescale 1ns/10ps
```

最后分析一下显示结果：
![example picture](/images/fifo1.png)

根据该结果可以发现，当复位信号为0时，计数器结果不进行变动，当复位信号变为1时，由于stat为0，所以计数器继续不进行操作。当stat为1时，系统开始进行累加，cnt的数值增加，一直增加到给定的最大值为5，后面stat为2，cnt开始减小，之后一直重复直到复位信号归于0结束。