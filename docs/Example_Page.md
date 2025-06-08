# Verilog 学习心得

本人为初学者，此为Verilog 的入门心得，来自于一边学习一边记录，如有错误，还请指正.

## 1 Verilog介绍

**Verilog** 是一种​​硬件描述语言​​，主要用于​​设计和模拟数字电路和系统​​。它是电子设计自动化领域的基石之一（与 VHDL 并列为两大主流 HDL），在芯片设计（ASIC）、现场可编程门阵列设计（FPGA）、以及复杂的数字系统设计中扮演着核心角色。

## 2 数值表示

在verilog中的数值表示方式通常为 **<位宽>'<基数><数值>**，如 **4'b1011**。以下对该表示方式的各个参数进行解释。

**位宽​**是数字电路和计算机系统中的核心概念，指的是​​一个数字信号用多少位（bit）二进制数来表示​​。它直接决定了该信号能表示的​​数值范围、精度和分辨率​​。信号的位宽取决于要该信号要表示的最大值。该信号能表示的无符号数最大值是：2^n-1，其中n表示该信号的位宽。例如，信号a的最大值为100，其二进制为1100110，那么信号a的位宽必须大于或等于7位。

**基数​**是表示数字是多少进制的。常用的进制数为二进制（b或B）、十进制（d或D）、八进制（o或O）和十六进制（h或H）。

**数值**是由基数所决定的表示常量真实值的一串ASCII码。如果基数定义为b或B，数值可以是0，1，x，X，z或Z。如果基数定义为o或O，数值是0-7。如果基数定义为h或H，数值是0-F。对于基数为d或者D的情况，数值是0-9。例如，4'b12是错误的，因为b表示二进制，数值只能是0、1、x或者z，不包含2。32'h12等同于32'h00000012，即数值未写完整时，高位置补0。

**补充**：32'h00000012中h为十六进制，每一位相当于四位二进制，因此位宽为32，数值前面只补六个0。

## 3 数据类型

在Verilog中，数据类型分为三种，分别为寄存器类型，线网类型，参数类型（非线路类型）。其中主要使用的是前两种。

**寄存器类型**通常用reg来对储存单元进行描述。如D型触发器、ROM等。寄存器类型信号的特点是在某种触发机制下分配了一个值，在下一触发机制到来之前保留原值。
通过赋值语句可以改变寄存器中储存的值。
```verilog
reg [31:0] cnt; //位宽32位的寄存器
reg [32:0] cnt1,cnt2,cnt3; //三个位宽为32位的寄存器
```
它可以在always语句和initial语句中使用。
如果该过程语句描述的是时序逻辑，即always语句带有时钟信号，则该寄存器变量对应为寄存器；如果该过程语句描述的是组合逻辑，即always语句不带有时钟信号，则该寄存器变量对应为硬件连线；

**类型**



![example picture](/images/D-Flip-Flop.png)
<div style="text-align: center;">
图3.1 D触发器图片
</div>

图3.1为D触发器的基本图片，
### 2.1模块案例与介绍
模块有五个主要部分：端口定义、参数定义（可选）、 I/O 说明、内部信号声明、功能定义。
以下是一个经典的verilog模块：
```verilog
module module_name(
            clk , //端口 1，时钟
            rst_n , //端口 2，复位
            dout //其他信号,如 dout
                )
    parameter DATA_W = 8;
    input clk ; //输入信号定义
    input rst_n ; //输入信号定义 
    output [DATA_W-1:0] dout; //输出信号定义
    reg [DATA_W-1:0] dout; //信号类型
    reg signal1; //信号类型
    
endmodule
```

#### This is also an example title3
##### This is still an example title4
###### This is an example title again

This is a text.

- This is also a text.

This is **Highlight**

This is  _underscore_.

This is a paragraph.This is a paragraph.

This is a code block.
```java
public class Test {
    // some code
}
```

---

![example picture](/images/img.png)
 *figure 1*

This is a formula:
$a^2 + b^2 = c^2$

This is a form:

| Column | Column |
|--------|--------|
| Column | Column |

- [This is a sub page](A/sub)
