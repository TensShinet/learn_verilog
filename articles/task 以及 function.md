# Task 以及 Function



> 前言：这是一篇翻译文章，希望对你有一些帮助，原文在这里：https://documentation-rp-test.readthedocs.io/en/latest/tutorfpga07.html



function 和 task 是用来减少重复代码的利器！不仅仅如此，如果你想要你的代码变得更具有「可读性」不妨多用用 function 和 task。





## 0X00 Task



Task 是可以在该 Task 被定义的模块中任意时刻调用的子例程。但是可以在其他文件中定义它们，并将文件 include 到一个 module 中。



Task 有以下几种特性：



+ Task 可以包括 delay，但是 function 不能。



+ Task 可以有任意数量的输入和输出，但是 function 只能有一个输出



+ 在 Task 中定义的「变量」不能在 Task 外被使用



+ Task 是用语句调用的，不能在表达式中使用，function 可以在表达式中使用



+ Task 可以使用全局变量



再我们来看看它的语法（可以有任意数量的输入输出）：



```verilog
  // Style 1
  task [name];
    input  [port_list];
    inout  [port_list];
    output [port_list];
    begin
      [statements]
    end
  endtask
 
  // Style 2
  task [name] (input [port_list], inout [port_list], output [port_list]);
    begin
      [statements]
    end
  endtask
```



接着我们来看一个例子：



```verilog
module  task_calling (adc_a, adc_b, adc_a_conv, adc_b_conv);
    input [7:0] adc_a, adc_b;
    output [7:0] adc_a_conv, adc_b_conv;

    reg [7:0] adc_a_conv, adc_b_conv;

    task convert;
    input [7:0] adc_in;
    output [7:0] out;
    begin
        out = (9/5) *( adc_in + 32)
    end
    endtask

    always @ (adc_a) begin
      convert (adc_a, adc_a_conv);
    end

    always @ (adc_b) begin
      convert (adc_b, adc_b_conv);
    end

endmodule
```







## 0X01 Function





`Function 的本质更像是定义在 module 的一套组合逻辑，而 Task 是一个定义在 module 内的 module`  



Function 就像 Task 但还是有很多不同的，接下来我们说 Function 的特性：  



+ 函数不能包含时序延迟，例如 posege，negedge，delay，这意味着 Function 仅仅实现了组合逻辑  



+ 函数可以具有任意数量的输入，但只有一个输出



+ 考虑函数中的参数的声明顺序，并且必须与调用时参数的顺序相同



+ 函数可以调用其他函数，但不能调用任务



+ `最重要的是当对同一个函数多次使用的时候，它的局部变量是共享的，与 c 相反，如果要每次调用一个函数就生成一套新的电路，就必须使用 automatic 关键字`





我们来看 Function 的语法：



```verilog
  function [automatic] [return_type] name ([port_list]);
    [statements]
  endfunction
```



再看一个具体例子：



```verilog
module  function_calling(a, b, c, d, e, f);

input a, b, c, d, e ;
output f;
wire f;

function  myfunction;
input a, b, c, d;
begin
    myfunction = ((a+b) + (c-d));
end
endfunction

assign f =  (myfunction (a,b,c,d)) ? e :0;

endmodule
```











