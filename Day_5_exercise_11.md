# Exercise 11 — 4-bit Magnitude Comparator
##  Implement a comparator using only XOR, AND, and OR gates (no relational operators).

### Truthtable

| Input A | Input B | A > B (gt) | A < B (lt) | A = B (eq) |
| :---: | :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0 | 1 |
| 0 | 1 | 0 | 1 | 0 |
| 1 | 0 | 1 | 0 | 0 |
| 1 | 1 | 0 | 0 | 1 |


### Code
**Design Code**
```verilog
module comparator_1bit (input A, B, output gt, lt, eq);
    wire x = A ^ B;
    assign gt = A & x;
    assign lt = B & x;
    assign eq = 1'b1 ^ (gt | lt);
endmodule
```
**Testbench Code**
```verilog
`timescale 1ns/1ps
module tb_comparator;
    reg A, B; 
    wire gt, lt, eq;
    comparator_1bit uut (A, B, gt, lt, eq);
    initial begin
        $dumpfile("tb_comparator.vcd");
        $dumpvars(0, tb_comparator);
        $monitor("%b %b | %b %b %b", A, B, gt, lt, eq);
        A=0; B=0; 
        #1; A=0; B=1; 
        #1; A=1; B=0; 
        #1; A=1; B=1;
    end
endmodule
```

### Simulation results

<img width="1142" height="296" alt="image" src="https://github.com/user-attachments/assets/15a7687f-bb9e-47b2-a1fb-4d9fb7b5eebc" />


## Add cascading inputs (gt_in, eq_in, lt_in) for multi-stage comparison.
### Truthtable

| GT<sub>in</sub> | LT<sub>in</sub> | EQ<sub>in</sub> | A | B | GT<sub>out</sub> | LT<sub>out</sub> | EQ<sub>out</sub> |
|:---------------:|:---------------:|:---------------:|:-:|:-:|:----------------:|:----------------:|:----------------:|
| 1 | 0 | 0 | 0 | 1 | 1 | 0 | 0 |
| 0 | 0 | 1 | 0 | 1 | 0 | 1 | 0 |
| 0 | 0 | 1 | 1 | 0 | 1 | 0 | 0 |
| 0 | 0 | 1 | 1 | 1 | 0 | 0 | 1 |

### Code
**Design Code**
```verilog
module comparator_cascade (
    input A, B, gt_in, lt_in, eq_in, 
    output gt_out, lt_out, eq_out
);
    wire x = A ^ B;
    assign gt_out = gt_in | (eq_in & A & x);
    assign lt_out = lt_in | (eq_in & B & x);
    assign eq_out = 1'b1 ^ (gt_out | lt_out);
endmodule
```

**Testbench Code**
```verilog
module tb_cascade;
    reg A, B, gt_i, lt_i, eq_i; wire gt_o, lt_o, eq_o;
    comparator_cascade uut (A, B, gt_i, lt_i, eq_i, gt_o, lt_o, eq_o);
    initial begin
        $dumpfile("comparator_cascade.vcd");
        $dumpvars(0, tb_cascade);
        $monitor("In: gt=%b lt=%b eq=%b | A=%b B=%b | Out: gt=%b lt=%b eq=%b", gt_i, lt_i, eq_i, A, B, gt_o, lt_o, eq_o);
        // Test 1: Previous stage already determined A > B (Overrides current bits)
        gt_i=1; lt_i=0; eq_i=0; A=0; B=1; #1;
        // Test 2: Previous stages are equal, current stage determines A < B
        gt_i=0; lt_i=0; eq_i=1; A=0; B=1; #1;
        // Test 3: Previous stages are equal, current stage determines A > B
        gt_i=0; lt_i=0; eq_i=1; A=1; B=0; #1;
        // Test 4: Previous stages are equal, current stage remains equal
        gt_i=0; lt_i=0; eq_i=1; A=1; B=1;
    end
endmodule
```

### Simulation results

<img width="985" height="353" alt="image" src="https://github.com/user-attachments/assets/ac4931a3-118c-4d13-9954-bb9cd3d27177" />


## Build an 8-bit comparator by cascading two 4-bit comparators.
### Truthtable

| A   | B   | GT | LT | EQ |
|:---:|:---:|:--:|:--:|:--:|
| 50  | 50  | 0  | 0  | 1  |
| 200 | 50  | 1  | 0  | 0  |
| 10  | 80  | 1  | 0  | 0  |
| 105 | 100 | 1  | 0  | 0  |
| 100 | 105 | 0  | 1  | 0  |

### Code
**Design Code**
```verilog
module comp4 (
    input [3:0] A, B, 
    input gi, li, ei, 
    output go, lo, eo
);
    wire [3:0] x = A ^ B, e = 4'b1111 ^ x;
    wire g = (A[3]&x[3]) | (e[3]&A[2]&x[2]) | (e[3]&e[2]&A[1]&x[1]) | (e[3]&e[2]&e[1]&A[0]&x[0]);
    wire l = (B[3]&x[3]) | (e[3]&B[2]&x[2]) | (e[3]&e[2]&B[1]&x[1]) | (e[3]&e[2]&e[1]&B[0]&x[0]);
    assign go = gi | (ei & g), lo = li | (ei & l), eo = 1'b1 ^ (go | lo);
endmodule

module comparator_8bit (
    input [7:0] A, B, 
    output GT, LT, EQ
);
    wire g_mid, l_mid, e_mid;
    comp4 lsb (A[3:0], B[3:0], 1'b0, 1'b0, 1'b1, g_mid, l_mid, e_mid);
    comp4 msb (A[7:4], B[7:4], g_mid, l_mid, e_mid, GT, LT, EQ);
endmodule
```

**Testbench Code**
```verilog
module tb_8bit;
    reg [7:0] A, B; 
    wire GT, LT, EQ;
    comparator_8bit uut (A, B, GT, LT, EQ);
    initial begin
        $dumpfile("comp4_tb.vcd");
        $dumpvars(0, tb_8bit);
        $monitor("A=%d B=%d | GT=%b LT=%b EQ=%b", A, B, GT, LT, EQ);
        A=8'd50;  B=8'd50;  #1; // Case 1: Equal values
        A=8'd200; B=8'd50;  #1; // Case 2: MSB dominates (A > B)
        A=8'd10;  B=8'd80;  #1; // Case 3: MSB dominates (A < B)
        A=8'd105; B=8'd100; #1; // Case 4: MSB equal, LSB breaks tie (A > B)
        A=8'd100; B=8'd105;     // Case 5: MSB equal, LSB breaks tie (A < B)
    end
endmodule
```

### Simulation results

<img width="1186" height="274" alt="image" src="https://github.com/user-attachments/assets/1427a4a1-762e-4bc3-83e8-43e705676ae6" />
