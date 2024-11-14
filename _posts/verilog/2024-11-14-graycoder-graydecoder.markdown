---
layout:     post
title:      "GrayCoder&GrayDecoder"
subtitle:   " \"4bit格雷编码器译码器仿真实现\""
date:       2024-11-14 17:06:00
author:     "Ryan"
header-img: "img/post-bg-dcb.jpg"
catalog: true
tags:
    - verilog
    - gray
---

> 有时候赤心在祸心之下，英雄在狗熊之间

# 格雷编码
格雷编码器（Gray Encoder）是一种将二进制数转换为格雷码的编码器。格雷码的特点是相邻的两个数只有一位不同，这使得在硬件设计中，特别是在传感器读取、旋转编码器等应用中，减少了由于多位变化引起的错误。
## 四位格雷编码器
这个编码器将一个4位的二进制输入转换为4位的格雷码输出。
### 原理
对于4位二进制输入 B[3:0]，它对应的格雷编码输出 G[3:0] 的转换规则如下：  
$G_3=B_3$  
$G_3=B_3⊕B_2$  
$G_3=B_2⊕B_1$  
$G_3=B_1⊕B_0$  
其中，⊕表示异或操作。

![格雷编码原理](https://ryanaqu.github.io/img/inpost_verilog/graycoder.png)
### Verilog代码
模块代码：
```
`timescale 1ns / 1ps
module gray_coder_4bit(
    input  [3:0] binary, //4bit自然二进制
    output [3:0] gray    //4bit格雷码
    );
    //格雷编码
    assign gray[3] = binary[3];    
    assign gray[2] = binary[3] ^ binary[2];  
    assign gray[1] = binary[2] ^ binary[1];  
    assign gray[0] = binary[1] ^ binary[0]; 
endmodule
```

testbench：
```
`timescale 1ns / 1ps
module tb_gray_coder;
    //引脚定义
    reg  [3:0] binary;  //4bit自然二进制
    wire [3:0] gray;   //4bit格雷码
    //模块
    gray_coder_4bit u0(
        .binary(binary),  //4bit自然二进制
        .gray(gray)    //4bit格雷码
    );
    
    //测试代码
    initial 
    begin    
        // 测试不同的输入值
        binary = 4'b0000; #10;  // 等待10时间单位
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b0001; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b0010; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b0011; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b0100; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b0101; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b0110; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b0111; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b1000; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b1001; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b1010; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b1011; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b1100; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b1101; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b1110; #10
        $display("binary: %b, gray: %b", binary, gray);

        binary = 4'b1111; #10
        $display("binary: %b, gray: %b", binary, gray);
        
        $finish;
    end

endmodule
```

### 仿真结果
编码结果：
![](https://ryanaqu.github.io/img/inpost_verilog/result_gray_coder.png)
### Schematic


## 四位格雷译码器
### 原理
### Verilog代码
### 仿真结果
### Schematic

