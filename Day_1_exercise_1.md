# Exercise 01 — AND Gate
## Modify the design to create a 3-input AND gate

### 3-Input AND Gate Truth Table

| A | B | C | Y = A & B & C |
|:-:|:-:|:-:|:-------------:|
| 0 | 0 | 0 |       0       |
| 0 | 0 | 1 |       0       |
| 0 | 1 | 0 |       0       |
| 0 | 1 | 1 |       0       |
| 1 | 0 | 0 |       0       |
| 1 | 0 | 1 |       0       |
| 1 | 1 | 0 |       0       |
| 1 | 1 | 1 |       1       |

### Code
 
```verilog
// 3 input AND gate
module and_3inp(a,b,c,y);

input a,b,c;
output y;

assign y = a & b & c;

endmodule
```
---

## Implement AND gate using only NAND gates.

### 2-Input AND Gate Using NAND Gates

| A | B | X = NAND(A,B) | Y = NAND(X,X) |
|:-:|:-:|:-------------:|:-------------:|
| 0 | 0 |       1       |       0       |
| 0 | 1 |       1       |       0       |
| 1 | 0 |       1       |       0       |
| 1 | 1 |       0       |       1       |

### Code

```verilog
// AND using NAND
module and_using_nand(a,b,x,y);

input a,b;
output x,y;

assign x = ~(a & b);
assign y = ~(x & x);

endmodule
```
--- 

## Testbench code

### Code

```verilog
// Testbench of 3 input AND gate
`timescale 1ns/1ps
module tb_and_3inp;

reg a, b, c;
wire y;

and_3inp uut (.a(a),.b(b),.c(c),.y(y));

integer i;
initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, tb);

    $monitor("Time = %0t ns | a = %b, b = %b, c = %b | y = %b", $time, a, b, c, y);

    for (i = 0; i < 8; i = i + 1) begin
      {a, b, c} = i;
      #10;
    end

    $finish;
end

endmodule
```
```verilog
// Testbench of AND using NAND
`timescale 1ns/1ps
module tb;

reg a,b; 
wire x,y; 

integer i;

and_using_nand uut(a,b,x,y);

initial begin

  $dumpfile("wave.vcd"); $dumpvars(0,tb);
  $monitor("Time=%0t ns | a=%b b=%b | x=%b y=%b",$time,a,b,x,y);
  
  for(i=0;i<4;i=i+1) begin {a,b}=i; #10; end $finish;

end

endmodule
```
---

## Simulaton results

### 3 input AND gate
<img width="1918" height="542" alt="image" src="https://github.com/user-attachments/assets/f17f22ef-c959-4dcc-93ce-33a30b8554b6" />

### AND using NAND
<img width="1920" height="552" alt="image" src="https://github.com/user-attachments/assets/f3257809-2226-45c6-b05e-f8601fd46261" />
