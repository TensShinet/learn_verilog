# 如何写 Test Bench





> 前言：Test Bench 就是用来模拟真实电路的，我们通过看波形图分析自己的电路是不是写对了！也可以通过模拟需要实例化的 module 的逻辑来判断自己的电路是不是写对了！





首先我们写出我们需要测试的 module：



```verilog
// counter.v
module counter (clk, reset, enable, count);
    input clk, reset, enable;
    output [3:0] count;
    reg [3:0] count;                                   

    always @ (posedge clk) begin
        if (reset == 1'b1) begin
            count <= 0;
        end else if ( enable == 1'b1) begin
            count <= count + 1;
        end
    end

endmodule  
```





接下来我们要给这个 counter module 写 Test Bench



## 0X00 实例化



实例化的方法如下：



```verilog
module counter_tb;
  reg clk, reset, enable; 
  wire [3:0] count; 
    
  counter U0 ( 
  .clk    (clk), 
  .reset  (reset), 
  .enable (enable), 
  .count  (count) 
  ); 
    
endmodule 
```



但是如果只写这个的话会报错：



```shell
*** These modules were missing:
        counter referenced 1 times.
***
```



解决方法有两个（如果是用 iverilog 的话）：



+ include

```verilog
`include "conter.v"
```





+ 或者是在 iverilog 命令中加上 counter.v



```shell
iverilog -o counter conter_tb.v counter.v
```



这两者不能重复！



## 0X01 模拟时钟



接下来我们要模拟始终脉冲：



```verilog
module counter_tb; 
    reg clk, reset, enable; 
    wire [3:0] count; 

    counter U0 ( 
        .clk    (clk), 
        .reset  (reset), 
        .enable (enable), 
        .count  (count) 
    ); 

    initial  begin 
        clk = 0; 
        reset = 0; 
        enable = 0; 
		
        // 通常使用 forever 模拟始终脉冲
		forever begin
        	clk = ~clk;
        end
    end 
endmodule 
```





## 0X02 添加比较逻辑



```verilog
module counter_tb; 
    reg clk, reset, enable; 
    wire [3:0] count; 

    counter U0 ( 
        .clk    (clk), 
        .reset  (reset), 
        .enable (enable), 
        .count  (count) 
    ); 
    // 自定义事件用 -> 触发
	event terminate_sim; 
    
    // 监听自定义事件
    initial begin  
        @ (terminate_sim); 
        	#5 $finish; 
    end
    
    
    initial  begin 
        clk = 0; 
        reset = 0; 
        enable = 1; 
		
        // 通常使用 forever 模拟始终脉冲
		forever begin
        	#10 clk = ~clk;
        end
    end
    
    reg [3:0] count_compare; 

	always @ (posedge clk) 
        if (reset  ==  1'b1) begin
  			count_compare <= 0; 
        end else if ( enable  ==  1'b1) begin
  			count_compare <= count_compare + 1; 
		end
    
    always @ (posedge clk) 
  		if (count_compare != count) begin 
   	 		$display ("DUT Error at time %d", $time); 
    		$display (" Expected value %d, Got Value %d", count_compare, count); 
            // 触发自定义事件
   			 #5 -> terminate_sim; 
  end 
    
endmodule 
```



上面这个例子，是在自己的 module 中一边调用 counter 这个 module 一边自己模拟 counter 的逻辑。从而去比较两者的是不是相等的，从而避免去看波形图！





未完待续...