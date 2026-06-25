# Exercise 02 — Basic Logic Gates
## Implement all gates using only NAND gates.

### Truth Table / Function Table
| A | B | OR | AND | NOT A | NOT B | NAND | NOR | XOR | XNOR |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0 | 1 | 1 | 1 | 1 | 0 | 1 |
| 0 | 1 | 1 | 0 | 1 | 0 | 1 | 0 | 1 | 0 |
| 1 | 0 | 1 | 0 | 0 | 1 | 1 | 0 | 1 | 0 |
| 1 | 1 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 1 |

### Code

**Design Code**

```verilog 
module all_gate_using_nand ( input wire a,b,
output wire out_not_a, out_not_b, out_or, out_and, out_nor, out_nand, out_xor, out_xnor );

wire w[0:2];

//NOT Gate
assign out_not_a = ~( a & a );
assign out_not_b = ~( b & b );

//OR Gate
assign out_or = ~(out_not_a & out_not_b );

//AND Gate
assign w[0] = ~( a & b );
assign out_and = ~( w[0] & w[0] );

//NOR Gate
assign out_nor = ~( out_or & out_or );

//NAND Gate
assign out_nand = ~( a & b);

//XOR Gate
assign w[1] = ~( w[0] & a );
assign w[2] = ~( w[0] & b );
assign out_xor = ~( w[1] & w[2] );

//XNOR Gate
assign out_xnor = ~( out_xor & out_xor);

endmodule
```
**Testbench Code**
```verilog
module tb_all_gates_using_nand;
reg a, b;
wire [7:0] out; 

all_gate_using_nand uut (a, b, out[7], out[6], out[5], out[4], out[3], out[2], out[1], out[0]);

integer i; 

initial begin
$dumpfile("waveform.vcd");
$dumpvars(0, tb_all_gates_using_nand);

$monitor("t=%0t | a=%b b=%b | NOTa=%b NOTb=%b OR=%b AND=%b NOR=%b NAND=%b XOR=%b XNOR=%b", 
        $time, a, b, out[7], out[6], out[5], out[4], out[3], out[2], out[1], out[0]);
for (i = 0; i < 4; i++) begin
{a, b} = i; #10;
end

end
endmodule
```
---
## Build a 4-input XOR using 2-input XOR primitives
### Truth Table
| A | B | C | D | Out  |
| :-: | :-: | :-: | :-: | :--------------: |
| 0 | 0 | 0 | 0 |        0         |
| 0 | 0 | 0 | 1 |        1         |
| 0 | 0 | 1 | 0 |        1         |
| 0 | 0 | 1 | 1 |        0         |
| 0 | 1 | 0 | 0 |        1         |
| 0 | 1 | 0 | 1 |        0         |
| 0 | 1 | 1 | 0 |        0         |
| 0 | 1 | 1 | 1 |        1         |
| 1 | 0 | 0 | 0 |        1         |
| 1 | 0 | 0 | 1 |        0         |
| 1 | 0 | 1 | 0 |        0         |
| 1 | 0 | 1 | 1 |        1         |
| 1 | 1 | 0 | 0 |        0         |
| 1 | 1 | 0 | 1 |        1         |
| 1 | 1 | 1 | 0 |        1         |
| 1 | 1 | 1 | 1 |        0         |

### Code

**Design Code**
```verilog 
module xor4_using_xor2 (input wire a, b, c, d,output wire out);

    wire w[0:1]; 
    xor u1 (w[0], a, b); 
    xor u2 (w[1], c, d);  

    xor u3 (out, w[0], w[1]); 

endmodule
```

**Testbench Code**
```verilog
module tb_xor4_using_xor2;

reg a, b, c, d;
wire out;

xor4_using_xor2 uut (.a(a),.b(b),.c(c),.d(d),.out(out));

integer i;

initial begin
    $dumpfile("xor4_waveform.vcd");
    $dumpvars(0, tb_xor4_using_xor2);

    $monitor("t=%0t | ABCD=%b%b%b%b | Out=%b", $time, a, b, c, d, out);

    for (i = 0; i < 16; i = i + 1) begin
      {a, b, c, d} = i;
      #10; 
    end

    $finish; 
  end

endmodule
```
---
## Write a parameterized N-input OR gate

### Truth Table
| IN[3] | IN[2] | IN[1] | IN[0] | Out |
| :---: | :---: | :---: | :---: | :---: |
|   0   |   0   |   0   |   0   |   0   |
|   0   |   0   |   0   |   1   |   1   |
|   0   |   0   |   1   |   0   |   1   |
|   0   |   0   |   1   |   1   |   1   |
|   0   |   1   |   0   |   0   |   1   |
|   0   |   1   |   0   |   1   |   1   |
|   0   |   1   |   1   |   0   |   1   |
|   0   |   1   |   1   |   1   |   1   |
|   1   |   0   |   0   |   0   |   1   |
|   1   |   0   |   0   |   1   |   1   |
|   1   |   0   |   1   |   0   |   1   |
|   1   |   0   |   1   |   1   |   1   |
|   1   |   1   |   0   |   0   |   1   |
|   1   |   1   |   0   |   1   |   1   |
|   1   |   1   |   1   |   0   |   1   |
|   1   |   1   |   1   |   1   |   1   |

### Code
**Design Code**
```verilog
module param_or_gate #( parameter N = 4 )( input wire [N-1:0] in, output wire out );

assign out = |in;   

endmodule
```

## Simulation Results
**all gates using only NAND gates**
<img width="1023" height="415" alt="image" src="https://github.com/user-attachments/assets/598b642f-3438-406d-84bc-0cbe8b12322d" />

**4-input XOR using 2-input XOR primitives**
<img width="985" height="323" alt="image" src="https://github.com/user-attachments/assets/8c60caac-32fa-48d0-9d3a-7765c46f6f67" />
