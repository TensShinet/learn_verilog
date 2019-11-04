# 如何用 verilog 写一个 ALU



> 前言：之前也写过一个 toy_vm，里面也涉及到四个标志位的更新，ALU 的难点也是这个四个标志位的更新，我每次都会忘这四个标志位如何更新的，每次都要搜索半天，这篇博客主要给出四个标志位的更新规则，以及相应的 verilog 代码！



## 0X00 哪四个标志位



有这四个标志位，在 ALU 中非常重要：



+ zero 标志位



+ carry 标志位



+ negative 标志位



+ overflow 标志位





## 0X01 四个标志位更新的规则



以下标志位的更新只有关于：



`ADD、ADDU、SUB、SUBU、AND、OR、XOR、NOR、LUI（至高位立即数）、SLT（有符号比较）、SLTU（无符号比较）、SRA（算数右移）、SLL/SLR（逻辑、算数左移）、SRL（算数右移）`



### zero 标志位更新规则



+ Z = 1 表示结果为 0，Z = 0 表示结果不为 0



+ 对于比较操作，若 a - b = 0。Z = 1。表示两个数相等，否则为 0



+ 所有操作都要更新 zero 标志位





### carry 标志位更新规则



+ 对于加减运算来说，只有无符号运算才会更新此标志位

  

+ 对于比较运算来说，也只有无符号的比较才会更新此标志位

  

+ 无论是加还是减法最会都会变成加法，只用看所有位相加以后，最高位会不会产生进位，产生进位 carry = 1，反之 carry = 0



+ 所有左移运算，carry 标志位等于最后一个移出的值



+ 其他的运算不更新此标志位





### negative 标志位更新规则



+ 对于所有的运算来说，negative 就是最高位的值。比较运算就是相减以后的最高位。比如有两个 32 位的变量相加，哪怕产生了 33 位，negative 标志位就是最高位 32 位的值



### overflow 标志位更新规则



+ 只有有符号的运算才会更新 overflow 标志位  

  

+ 只有超出有符号的表示范围的时候 overflow = 1



## 0X02 Carry vs Overflow 



Carry 和 Overflow 很容易搞混，我来说说这两者的区别。



+ 首先最大的区别就是 Carry 标志位是对无符号运算（除所有左移运算）来说的，Overflow 标志位是对有符号运算来说的



+ 两者表示的意思也不一样，Carry 表示运算过后最高位有没有产生进位（除所有左移运算），而 Overflow 是判断有符号数字有没有超过当前能表示的最大范围



+ 所以这两者更新的代码也不一样



## 0X03 相应代码



zero 与 negative 标志位好判断，主要关注 carry 和 overflow



+ 更新 zero，carry，negative 标志位



```verilog
module ADDU(
    input [31:0] a,
    input [31:0] b,
    output reg [31:0] r,

    output reg zero,
    output reg carry,
    output reg negative,
    output reg overflow
);
    parameter WIDTH = 32;
    parameter MSB   = WIDTH - 1;

    reg extra;
    always @(*) begin
        // 计算
        {extra, r} = a + b;
        // 更新 flag
        zero = r ? 0 : 1;
        carry = extra;
        negative = r[MSB];
    end
    
endmodule
```



carry 只需多保留一位。如果进位，这一位为 1，不进位为 0。与 carry 标志位特点一致





+ 更新 zero，overflow，negative 标志位



```verilog
module ADD(
    input [31:0] a,
    input [31:0] b,
    output reg [31:0] r,

    output reg zero,
    output reg carry,
    output reg negative,
    output reg overflow
);
    parameter WIDTH = 32;
    parameter MSB   = WIDTH - 1;

    reg extra;
    always @(*) begin
        // 计算
        {extra, r} = {a[MSB], a} + {b[MSB], b};
        // 更新 flag
        zero = r ? 0 : 1;
        negative = r[MSB];
        overflow  = ({extra, r[MSB]} == 2'b01);
    end
endmodule
```



overflow 判断的方法来自：https://stackoverflow.com/questions/24586842/signed-multiplication-overflow-detection-in-verilog/24587824#24587824





最后的 ALU 代码，我会整理以后，在 GitHub 中放出。



