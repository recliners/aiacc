# github内SMC代码个人运行流程及测试结果

下载地址：https://github.com/seclabBupt/aiacc.git

## 1 使用流程

### 1.1 文件传输

本人采用Xfpt8进行文件传输，该软件是一个免费软件，可到官网上下载，下载完成后让其与本地虚拟机进行连接，连接成功后即可传输。

### 1.2 代码运行

该代码的运行流程是根据Sunny学长发在群内的名为**Berkeley SoftFloat库**的文件进行，再加上本人对一些运行过程中会出现的问题的解决措施。

首先要进入到berkeley-softfloat-3-master里面的Linux-x86_64-GCC文件内，个人的具体路径为：
```Linux
cd /home/five/Desktop/aiacc-main/SMC/berkeley-softfloat-3-master/build/Linux-x86_64-GCC
```
可根据自己的路径进行修改。

#### 第一个可能会出现的错误：

之后在在makefile中加入fPIC，其代码具体为：
```
COMPILE_C = \
	gcc -c -Werror-implicit-function-declaration -DSOFTFLOAT_FAST_INT64 \
		$(SOFTFLOAT_OPTS) $(C_INCLUDES) -O2 -fPIC -o $@
MAKELIB = ar crs $@
```
之后在终端里输入make即可编译SoftFloat库。

**注**：如果直接复制"**Berkeley SoftFloat库**"这个文件内的代码会产生错误，错误为：
```
[five@eda1 Linux-x86_64-GCC]$ make
  gcc -c -Werror-implicit-function-declaration -DSOFTFLOAT_FAST_INT64     -DSOFTFLOAT_ROUND_ODD -DINLINE_LEVEL=5 -DSOFTFLOAT_FAST_DIV32TO16 -DSOFTFLOAT_FAST_DIV64TO32 -I. -I../../source/8086-SSE -I../../source/include -O2 -fPIC -o s_eq128.o ../../source/s_eq128.c
make:  : Command not found
make: *** [s_eq128.o] Error 127
```
该错误的具体原因Makefile规则使用了​​空格缩进​​而不是​​制表符(Tab)​​。Make工具严格要求命令必须以制表符开头（而不是空格），否则会报Command not found错误。简单来说就是应该用Tab进行缩进，但是这个文件里的代码用了空格导致出现这个问题。

之后运行:
```
g++ -shared -o libruntime.so softfloat_dpi.o -L/home/five/Desktop/aiacc-main/SMC/berkeley-softfloat-3-master/build/Linux-x86_64-GCC/ -lsoftfloat
```
将softfloat_dpi.c编译为共享库libruntime.so，并链接softfloat.a。不过SMC内的文件似乎以及进行了链接，所以不需要进行这步操作，当然也可以运行，但是会报错，具体原因是：
```
[five@eda1 Linux-x86_64-GCC]$ g++ -shared -o libruntime.so softfloat_dpi.o -L/home/five/Desktop/aiacc-main/SMC/berkeley-softfloat-3-master/build/Linux-x86_64-GCC/ -lsoftfloat

bash: g++ -shared -o libruntime.so softfloat_dpi.o -L/home/five/Desktop/aiacc-main/SMC/berkeley-softfloat-3-master/build/Linux-x86_64-GCC/ -lsoftfloat: No such file or directory
```
不过这个其实不是什么大问题，因为以及进行了链接，所以这不是不需要操作的。

如果是下载的github内的SMC文件，之后的包括代码的增加是不需要进行的。一直到第五步使用run_sim.sh脚本编译和运行仿真前都是不需要操作的。之后当你运行：
```
./run_sim.sh
```
会报错，错误原因为：
```
[five@eda1 fp16_to_fp32_multiplier]$ ./run_sim.sh
bash: ./run_sim.sh: Permission denied
```
具体原因为是没有权限执行run_sim.sh脚本。因此在运行该代码前需要先运行

## 16 to 32

问题1：COMPILE_C = \
  gcc -c -Werror-implicit-function-declaration -DSOFTFLOAT_FAST_INT64 \
    $(SOFTFLOAT_OPTS) $(C_INCLUDES) -O2 -fPIC -o $@
MAKELIB = ar crs $@

Make
格式错误


问题2：改文件路径

问题3：结果解释

0符号位 10010指数位 1100000000尾数位

0->+

10010->18-15=3

1100000000->1.75

实际尾数​​ = 1 + 小数部分

小数部分 = 1×2⁻¹ + 1×2⁻² + 0×2⁻³ + ... = 0.5 + 0.25 = 0.75

实际尾数 = 1 + 0.75 = 1.75

## 32 adder tree

问题一：
```
../softfloat_fp32_dpi.c:61:5: note: use option -std=c99 or -std=gnu99 to compile your code
[错误] 目标文件编译失败
错误: DPI-C 文件编译失败
```

解决方法：
将：
```
# 编译 DPI-C 源文件为目标文件
gcc -c -fPIC \
    -I"$SOFTFLOAT_INCLUDE" \
    "$DPI_SOURCE" \
    -o "$OBJ_FILE"
```
改为
```
# 编译 DPI-C 源文件为目标文件 - 添加 C99 标准支持
gcc -c -fPIC \
    -I"$SOFTFLOAT_INCLUDE" \
    -std=c99 \# 添加 C99 标准支持
    "$DPI_SOURCE" \
    -o "$OBJ_FILE"
```





