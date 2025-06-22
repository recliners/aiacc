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

1