# Exercise 12 — 4-bit ALU
## Add a Carry flag for signed arithmetic.

### Truthtable

| Ctrl | A  | B  | Sum | Cout | Ov |
|:----:|:--:|:--:|:---:|:----:|:--:|
| 0 |  5 |  4 | -7 | 0 | 1 |
| 0 | -5 | -4 |  7 | 1 | 1 |
| 1 |  5 |  4 |  1 | 1 | 0 |
| 0 | -5 |  4 | -1 | 0 | 0 |

### Code
**Design Code**
```verilog
module signed_adder_sub #(parameter N = 4) (
    input [N-1:0] A, B,
    input ctrl, 
    output [N-1:0] sum,
    output cout, overflow
);
    wire [N-1:0] B_mux = B ^ {N{ctrl}};
    assign {cout, sum} = A + B_mux + ctrl;

    assign overflow = (A[N-1] == B_mux[N-1]) && (sum[N-1] != A[N-1]);
endmodule
```

**Testbench Code**
```verilog
module tb;
    // Declaring these as signed fixes the iverilog $monitor error
    reg signed [3:0] A, B; 
    reg ctrl; 
    wire signed [3:0] sum; 
    wire cout, ov;

    signed_adder_sub #(4) uut (A, B, ctrl, sum, cout, ov);

    initial begin
        $dumpfile("cf_signed_arithmetic.vcd");
        $dumpvars(0, tb);
        $monitor("Ctrl=%b | A=%d B=%d | Sum=%d | Cout=%b Ov=%b", ctrl, A, B, sum, cout, ov);
        
        A = 5;  B = 4;  ctrl = 0; #10; // 5 + 4 = 9  (Overflow!)
        A = -5; B = -4; ctrl = 0; #10; // -5 + (-4) = -9 (Overflow!)
        A = 5;  B = 4;  ctrl = 1; #10; // 5 - 4 = 1  (OK)
        A = -5; B = 4;  ctrl = 0; #10; // -5 + 4 = -1 (OK)
        $finish;
    end
endmodule
```

### Simulation results

<img width="1010" height="325" alt="image" src="https://github.com/user-attachments/assets/52496ec4-3cd8-41a3-83f7-e4d76be44a1d" />

##  Add a zero flag, if A or B is ‘b0.
### Truthtable

| A | B  | Ctrl | Sum | Ov | Z |
|:-:|:--:|:----:|:---:|:--:|:-:|
|  5 |  0 | 0 |  5 | 0 | 1 |
|  0 | -4 | 0 | -4 | 0 | 1 |
|  5 |  4 | 0 | -7 | 1 | 0 |
|  0 |  0 | 1 |  0 | 0 | 1 |

### Code
**Design Code**
```verilog
module signed_adder_sub #(parameter N = 4) (
    input signed [N-1:0] A, B,
    input ctrl, // 0: Add, 1: Sub
    output signed [N-1:0] sum,
    output cout, overflow, zero
);
    wire [N-1:0] B_mux = B ^ {N{ctrl}};
    assign {cout, sum} = A + B_mux + ctrl;
    assign overflow = (A[N-1] == B_mux[N-1]) && (sum[N-1] != A[N-1]);
    
    // Zero flag triggers if A == 0 OR B == 0
    assign zero = (A == 0) || (B == 0);
endmodule

```

**Testbench Code**
```verilog
module tb;
    reg signed [3:0] A, B; reg ctrl;
    wire signed [3:0] sum; wire cout, ov, z;

    signed_adder_sub #(4) uut (A, B, ctrl, sum, cout, ov, z);

    initial begin
        $dumpfile("signed_adder_sub.vcd"); 
        $dumpvars(0, tb);
        $monitor("A=%d B=%d Ctrl=%b | Sum=%d Ov=%b Z=%b", A, B, ctrl, sum, ov, z);
        
        A = 5;  B = 0;  ctrl = 0; #10; // B is 0 -> Z should be 1
        A = 0;  B = -4; ctrl = 0; #10; // A is 0 -> Z should be 1
        A = 5;  B = 4;  ctrl = 0; #10; // Neither is 0 -> Z should be 0
        A = 0;  B = 0;  ctrl = 1; #10; // Both are 0 -> Z should be 1
        $finish;
    end
endmodule
```


### Simulation results

<img width="1010" height="329" alt="image" src="https://github.com/user-attachments/assets/f585568c-29eb-4b76-8c39-ed1d979261c4" />


##  Extend to 8-bit operands.
### Truthtable

| A    | B    | Ctrl | Sum  | Cout | Ov | Z |
|:----:|:----:|:----:|:----:|:----:|:--:|:-:|
| 127  |   1  | 0 | -128 | 0 | 1 | 0 |
| -128 |   1  | 1 | 127  | 1 | 1 | 0 |
|   0  |  50  | 0 | 50   | 0 | 0 | 1 |
| -45  |   0  | 1 | -45  | 1 | 0 | 1 |
|  10  | -20  | 0 | -10  | 0 | 0 | 0 |

### Code
**Design Code**
```verilog
module signed_adder_sub_8bit (
    input signed [7:0] A, B,
    input ctrl, // 0: Add, 1: Sub
    output signed [7:0] sum,
    output cout, overflow, zero
);
    wire [7:0] B_mux = B ^ {8{ctrl}};
    assign {cout, sum} = A + B_mux + ctrl;
    // MSB is index 7 for 8-bit signed math overflow
    assign overflow = (A[7] == B_mux[7]) && (sum[7] != A[7]);
    assign zero = (A == 8'b0) || (B == 8'b0);
endmodule
```

**Testbench Code**
```verilog
module tb;
    reg signed [7:0] A, B; 
    reg ctrl;

    wire signed [7:0] sum; 
    wire cout, ov, z;

    signed_adder_sub_8bit uut (A, B, ctrl, sum, cout, ov, z);

    initial begin
        $dumpfile("signed_adder_sub_8bit.vcd"); 
        $dumpvars(0, tb);
        $monitor("A=%d B=%d Ctrl=%b | Sum=%d Cout=%b Ov=%b Z=%b", A, B, ctrl, sum, cout, ov, z);
        
        A = 127; B = 1;   ctrl = 0; #10; // 127 + 1 = -128 (Overflow!)
        A = -128; B = 1;  ctrl = 1; #10; // -128 - 1 = 127 (Overflow!)
        A = 0;   B = 50;  ctrl = 0; #10; // A is 0 -> Z = 1
        A = -45; B = 0;   ctrl = 1; #10; // B is 0 -> Z = 1
        A = 10;  B = -20; ctrl = 0; #10; // Safe math -> Z = 0, Ov = 0
        $finish;
    end
endmodule
```

### Simulation Results

<img width="982" height="345" alt="image" src="https://github.com/user-attachments/assets/6a990be1-0e6c-4818-b710-73b627540c36" />

