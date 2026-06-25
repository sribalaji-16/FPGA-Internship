# Exercise 03 — Half Adder
## Build a half adder using only NAND gates
### Truth Table

| Input A | Input B | Sum (S) | Carry (C) |
| :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

### Code 
**Design Code**

```verilog
module half_adder_nand(input A, B, output S, C);

    wire w[0:2];

    nand g1(w[0], A, B);
    nand g2(w[1], A, w1);
    nand g3(w[2], B, w1);
    nand g4(S, w2, w3);
    nand g5(C, w1, w1);

endmodule
```

**Testbench Code**
```verilog
module tb_half_adder_nand;
    reg A, B; 
    wire S, C;

    half_adder_nand uut (A, B, S, C);

    initial begin
        $dumpfile("half_adder.vcd");
        $dumpvars(0, tb_half_adder_nand);

        $monitor("Time=%0dt | A=%b B=%b | Sum=%b Carry=%b", $time, A, B, S, C);
        
        A=0; B=0; #10;
        A=0; B=1; #10;
        A=1; B=0; #10;
        A=1; B=1; #10;
        $finish;
    end
    
endmodule
```
---

## Implement using a 2:1 MUX for the sum output.

### Truth table 
| Input A (Select) | Input B | Sum (S) | Carry (C) |
| :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

### Code
**Design Code**
```verilog
module half_adder_mux(input A, B, output S, C);

    assign S = A ? ~B : B; 
    assign C = A & B;

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns/1ps
module tb_half_adder_mux;
    reg A, B; 
    wire S, C;

    half_adder_mux uut (A, B, S, C);
    
    integer i;
    
    initial begin
        $dumpfile("half_adder_mux.vcd"); $dumpvars(0, tb_half_adder_mux);
        $monitor("Time=%0dt | A=%b B=%b | Sum=%b Carry=%b", $time, A, B, S, C);
        for (i = 0; i < 4; i = i + 1) begin
        {A, B} = i; 
        #10;
        end
        $finish;
    end
endmodule
```

---

## Simulation results
**Build a half adder using only NAND gates**
<img width="994" height="263" alt="image" src="https://github.com/user-attachments/assets/860fa67c-4a71-49dc-a786-d88f610ea98b" />

**Implement using a 2:1 MUX for the sum output**
<img width="1897" height="297" alt="image" src="https://github.com/user-attachments/assets/d3e846df-5c30-4bb5-abbb-933d9e4903bf" />
