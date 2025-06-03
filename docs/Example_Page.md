# Verilog 学习心得

本人为初学者，此为Verilog 的入门心得，来自于一边学习一边记录，如有错误，还请指正.

## 1.Verilog介绍

**Verilog** 是一种​​硬件描述语言​​，主要用于​​设计和模拟数字电路和系统​​。它是电子设计自动化领域的基石之一（与 VHDL 并列为两大主流 HDL），在芯片设计（ASIC）、现场可编程门阵列设计（FPGA）、以及复杂的数字系统设计中扮演着核心角色。

## 2.模块化设计

**模块化设计​**​是一种将复杂系统分解为独立、可复用、功能明确的子模块（组件）的设计方法。在硬件设计和硬件描述语言中，这是构建复杂数字系统的​​核心思想和最佳实践​​。

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
