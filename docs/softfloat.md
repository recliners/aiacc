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





