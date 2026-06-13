# Exercise 04 — Full Adder
## Implement Full Adder using two Half Adders and an OR gate

### Truth table
| A | B | C | S1 (HA1 Sum) | C1 (HA1 Carry) | C2 (HA2 Carry) | S (Final Sum) | Carry (Final Out) |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| 0 | 0 | 0 | 0 | 0 | 0 | **0** | **0** |
| 0 | 0 | 1 | 0 | 0 | 0 | **1** | **0** |
| 0 | 1 | 0 | 1 | 0 | 0 | **1** | **0** |
| 0 | 1 | 1 | 1 | 0 | 1 | **0** | **1** |
| 1 | 0 | 0 | 1 | 0 | 0 | **1** | **0** |
| 1 | 0 | 1 | 1 | 0 | 1 | **0** | **1** |
| 1 | 1 | 0 | 0 | 1 | 0 | **0** | **1** |
| 1 | 1 | 1 | 0 | 1 | 0 | **1** | **1** |

### Code
**Design Code**
```verilog
module ha(input a, b, output s, c);
assign s = a ^ b, c = a & b; 
endmodule

module full_adder(input A, B, C, output S, Carry);
    wire s1, c1, c2;
    ha ha1(A, B, s1, c1), ha2(s1, C, S, c2); 
    assign Carry = c1 | c2;                  
endmodule
```

**Testbench Code**
```verilog
module tb;
    reg A, B, C; 
    wire S, Carry; 
    integer i;
    full_adder uut(A, B, C, S, Carry);
    initial begin
        $dumpfile("dump.vcd");
        $dumpvars(0, tb);
        $monitor("%b %b %b | %b %b", A, B, C, S, Carry);
        for(i = 0; i < 8; i = i + 1) begin {A, B, C} = i; #5; end
    end
endmodule
```

## Build Full Adder using only NAND gates (minimum count).

### Truth table
| A | B | C | n1 | n2 | n3 | n4 (S1) | n5 | n6 | n7 | S (Sum) | Carry |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| 0 | 0 | 0 | 1 | 1 | 1 | **0** | 1 | 1 | 1 | **0** | **0** |
| 0 | 0 | 1 | 1 | 1 | 1 | **0** | 1 | 1 | 0 | **1** | **0** |
| 0 | 1 | 0 | 1 | 1 | 0 | **1** | 1 | 0 | 1 | **1** | **0** |
| 0 | 1 | 1 | 1 | 1 | 0 | **1** | 0 | 1 | 1 | **0** | **1** |
| 1 | 0 | 0 | 1 | 0 | 1 | **1** | 1 | 0 | 1 | **1** | **0** |
| 1 | 0 | 1 | 1 | 0 | 1 | **1** | 0 | 1 | 1 | **0** | **1** |
| 1 | 1 | 0 | 0 | 1 | 1 | **0** | 1 | 1 | 1 | **0** | **1** |
| 1 | 1 | 1 | 0 | 1 | 1 | **0** | 1 | 1 | 0 | **1** | **1** |

### Code
**Design Code**
```verilog
module full_adder_nand9 (
    input  wire A, B, C,
    output wire S, Carry
);
    wire n[1:7];

    nand g1(n[1], A, B);
    nand g2(n[2], A, n[1]);
    nand g3(n[3], B, n[1]);
    nand g4(n[4], n[2], n[3]); 

    nand g5(n[5], n[4], C);
    nand g6(n[6], n[4], n[5]);
    nand g7(n[7], C, n[5]);
    nand g8(S,  n[6], n[7]); 
    
    nand g9(Carry, n[1], n[5]); 

endmodule
```

**Testbench Code**
```verilog
module tb_nand9;
    reg A, B, C; 
    wire S, Carry; 
    integer i;
    full_adder_nand9 uut(A, B, C, S, Carry);
    initial begin
        $dumpfile("nand9.vcd"); 
        $dumpvars(0, tb_nand9);
        $monitor("Time=%0t | A=%b B=%b C=%b | Sum=%b Carry=%b", $time, A, B, C, S, Carry);
        for(i = 0; i < 8; i = i + 1) begin {A, B, C} = i; #5; end
    end
endmodule
```

## Simulation Results

**Implement Full Adder using two Half Adders and an OR gate**
<img width="998" height="304" alt="image" src="https://github.com/user-attachments/assets/94e4b814-c1a6-4cc1-b845-1c22b6dffb7e" />


**Build Full Adder using only NAND gates (minimum count).**
<img width="998" height="303" alt="image" src="https://github.com/user-attachments/assets/664d0934-977b-41f4-a456-f6532d3ed26d" />
