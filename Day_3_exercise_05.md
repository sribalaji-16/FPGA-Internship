# Exercise 05 — 4-bit Ripple Carry Adder

## Extend to an 8-bit RCA by chaining two 4-bit RCAs

### Truth Table
| A | B | Cin | Sum | Cout |
|---|---|-----|-----|------|
| 0 | 0 | 0   | 0   | 0    |
| 0 | 0 | 1   | 1   | 0    |
| 0 | 1 | 0   | 1   | 0    |
| 0 | 1 | 1   | 0   | 1    |
| 1 | 0 | 0   | 1   | 0    |
| 1 | 0 | 1   | 0   | 1    |
| 1 | 1 | 0   | 0   | 1    |
| 1 | 1 | 1   | 1   | 1    |

### Code
**Design Code**
```verilog
module full_adder (
    input  a,
    input  b,
    input  cin,
    output sum,
    output cout
);
    assign sum  = a ^ b ^ cin;
    assign cout = (a & b) | (b & cin) | (cin & a);
endmodule

// Supporting 4-bit RCA Module
module rca_4bit (
    input  [3:0] A,
    input  [3:0] B,
    input        Cin,
    output [3:0] Sum,
    output       Cout
);
    wire [2:0] c; // Internal carries

    full_adder fa0 (.a(A[0]), .b(B[0]), .cin(Cin),  .sum(Sum[0]), .cout(c[0]));
    full_adder fa1 (.a(A[1]), .b(B[1]), .cin(c[0]), .sum(Sum[1]), .cout(c[1]));
    full_adder fa2 (.a(A[2]), .b(B[2]), .cin(c[1]), .sum(Sum[2]), .cout(c[2]));
    full_adder fa3 (.a(A[3]), .b(B[3]), .cin(c[2]), .sum(Sum[3]), .cout(Cout));
endmodule

// Top-Level 8-bit Ripple Carry Adder
module rca_8bit (
    input  [7:0] A,
    input  [7:0] B,
    input        Cin,
    output [7:0] Sum,
    output       Cout
);
    wire c_inter; // Intermediate carry between lower and upper 4-bit RCAs

    // Lower 4 bits [3:0]
    rca_4bit rca_lower (
        .A    (A[3:0]),
        .B    (B[3:0]),
        .Cin  (Cin),
        .Sum  (Sum[3:0]),
        .Cout (c_inter)
    );

    // Upper 4 bits [7:4]
    rca_4bit rca_upper (
        .A    (A[7:4]),
        .B    (B[7:4]),
        .Cin  (c_inter),
        .Sum  (Sum[7:4]),
        .Cout (Cout)
    );
endmodule
```


**Testbench Code**
```verilog
`timescale 1ns / 1ps

module tb_rca_8bit;

    reg [7:0] A;
    reg [7:0] B;
    reg       Cin;

    wire [7:0] Sum;
    wire       Cout;

    rca_8bit uut (
        .A(A),
        .B(B),
        .Cin(Cin),
        .Sum(Sum),
        .Cout(Cout)
    );

    initial begin
        $dumpfile("rca_8bit_dump.vcd");
        $dumpvars(0, tb_rca_8bit);

        $monitor("Time=%0td | A=%d B=%d Cin=%b | Sum=%d Cout=%b", $time, A, B, Cin, Sum, Cout);

        // --- Test Case 1: Simple Addition without Carry ---
        A = 8'd10;  B = 8'd20;  Cin = 1'b0;
        #10;

        // --- Test Case 2: Max value addition causing Outward Carry ---
        A = 8'd255; B = 8'd1;   Cin = 1'b0;
        #10;

        // --- Test Case 3: Verify the internal ripple transition (Lower to Upper) ---
        // 15 + 1 will toggle the lower Cout (c_inter) to 1, pushing into the upper stage
        A = 8'd15;  B = 8'd1;   Cin = 1'b0;
        #10;

        // --- Test Case 4: Random Mix with external Cin ---
        A = 8'd128; B = 8'd127; Cin = 1'b1;
        #10;

        // --- Test Case 5: Zero check ---
        A = 8'd0;   B = 8'd0;   Cin = 1'b0;
        #10;

        $finish;
    end

endmodule
```
### Simulation results
<img width="1897" height="275" alt="image" src="https://github.com/user-attachments/assets/a328f41c-9517-4acd-91d9-a0eec5b6fe60" />


## Measure worst-case propagation delay using timing simulation.
### Code
**Design Code**
```verilog
`timescale 1ns / 1ps

module full_adder (
    input  a,
    input  b,
    input  cin,
    output sum,
    output cout
);

    assign #2 sum  = a ^ b ^ cin;
    assign #2 cout = (a & b) | (b & cin) | (cin & a);
endmodule

module rca_4bit (
    input  [3:0] A,
    input  [3:0] B,
    input        Cin,
    output [3:0] Sum,
    output       Cout
);
    wire [2:0] c;

    full_adder fa0 (.a(A[0]), .b(B[0]), .cin(Cin),  .sum(Sum[0]), .cout(c[0]));
    full_adder fa1 (.a(A[1]), .b(B[1]), .cin(c[0]), .sum(Sum[1]), .cout(c[1]));
    full_adder fa2 (.a(A[2]), .b(B[2]), .cin(c[1]), .sum(Sum[2]), .cout(c[2]));
    full_adder fa3 (.a(A[3]), .b(B[3]), .cin(c[2]), .sum(Sum[3]), .cout(Cout));
endmodule

module rca_8bit (
    input  [7:0] A,
    input  [7:0] B,
    input        Cin,
    output [7:0] Sum,
    output       Cout
);
    wire c_inter;

    rca_4bit rca_lower (.A(A[3:0]), .B(B[3:0]), .Cin(Cin),     .Sum(Sum[3:0]), .Cout(c_inter));
    rca_4bit rca_upper (.A(A[7:4]), .B(B[7:4]), .Cin(c_inter), .Sum(Sum[7:4]), .Cout(Cout));
endmodule
```

**Testbench Code**
```verilog
module tb_rca_timing;

    reg [7:0] A;
    reg [7:0] B;
    reg       Cin;
    
    wire [7:0] Sum;
    wire       Cout;

    real start_time = 0;
    real end_time = 0;
    real total_delay = 0;

    rca_8bit uut (
        .A(A), .B(B), .Cin(Cin), .Sum(Sum), .Cout(Cout)
    );

    initial begin
        $dumpfile("rca_timing_dump.vcd");
        $dumpvars(0, tb_rca_timing);

        A = 8'b1111_1111; 
        B = 8'b0000_0000; 
        Cin = 1'b0;
        #50; 

        $display("[STIMULUS] Toggling Cin at time: %0f ns", $realtime);
        start_time = $realtime;
        
        Cin = 1'b1; // This change triggers the 8-bit ripple avalanche
        
        #50; // Wait for the ripple to complete and display results
        $finish;
    end

    always @(Sum or Cout) begin
        if (start_time > 0 && Sum == 8'b0000_0000 && Cout == 1'b1) begin
            end_time = $realtime;
            total_delay = end_time - start_time;
            
            $display("[STABILITY] Outputs stabilized at time: %0f ns", end_time);
            $display("====================================================");
            $display(" WORST-CASE PROPAGATION DELAY: %0f ns", total_delay);
            $display("====================================================");
        end
    end

endmodule
```
### iverilog output
```sh
…/nielit/day_3_exercise_05 master ✗ ./a.out
VCD info: dumpfile rca_timing_dump.vcd opened for output.
[STIMULUS] Toggling Cin at time: 50.000000 ns
[STABILITY] Outputs stabilized at time: 66.000000 ns
====================================================
 WORST-CASE PROPAGATION DELAY: 16.000000 ns
====================================================
tb_rca_fa_delay.v:33: $finish called at 100000 (1ps)
```

### Simulation results
<img width="1601" height="352" alt="image" src="https://github.com/user-attachments/assets/85a8fe23-ce90-46fb-9c2f-ddb859baaada" />


## Compare with a 4-bit Carry-Lookahead Adder implementation.
### Truth table
| A | B | Cin | Sum | Cout | 
|---|---|-----|-----|------| 
| 0 | 0 | 0 | 0 | 0 | 
| 0 | 0 | 1 | 1 | 0 | 
| 0 | 1 | 0 | 1 | 0 | 
| 0 | 1 | 1 | 0 | 1 | 
| 1 | 0 | 0 | 1 | 0 | 
| 1 | 0 | 1 | 0 | 1 | 
| 1 | 1 | 0 | 0 | 1 | 
| 1 | 1 | 1 | 1 | 1 |

### Code
**Design Code**
```verilog
`timescale 1ns / 1ps
module cla_4bit (
    input  [3:0] A,
    input  [3:0] B,
    input        Cin,
    output [3:0] Sum,
    output       Cout
);
    wire [3:0] G, P;
    wire [4:0] C;

    assign C[0] = Cin;

    assign G = A & B;
    assign P = A ^ B;

    assign C[1] = G[0] | (P[0] & C[0]);
    assign C[2] = G[1] | (P[1] & G[0]) | (P[1] & P[0] & C[0]);
    assign C[3] = G[2] | (P[2] & G[1]) | (P[2] & P[1] & G[0]) | (P[2] & P[1] & P[0] & C[0]);
    assign C[4] = G[3] | (P[3] & G[2]) | (P[3] & P[2] & G[1]) | (P[3] & P[2] & P[1] & G[0]) | (P[3] & P[2] & P[1] & P[0] & C[0]);

    assign Sum  = P ^ C[3:0];
    assign Cout = C[4];

endmodule
```

**Testbench Code**
```verilog
module tb_cla_4bit;

    reg [3:0] A;
    reg [3:0] B;
    reg       Cin;
    
    wire [3:0] Sum;
    wire       Cout;

    cla_4bit uut (
        .A(A),
        .B(B),
        .Cin(Cin),
        .Sum(Sum),
        .Cout(Cout)
    );

    initial begin
        
        $dumpfile("cla_4bit_dump.vcd");
        $dumpvars(0, tb_cla_4bit);

        $monitor("Time=%0td ps | A=%b (%d) B=%b (%d) Cin=%b | Sum=%b (%d) Cout=%b", 
                 $time, A, A, B, B, Cin, Sum, Sum, Cout);

        // Test Case 1: Simple addition without generating any carries
        A = 4'd4;   B = 4'd2;   Cin = 1'b0;
        #10;

        // Test Case 2: Standard internal carry generation (3 + 1 = 4)
        A = 4'd3;   B = 4'd1;   Cin = 1'b0;
        #10;

        // Test Case 3: Absolute maximum values pushing out an intermediate Cout
        A = 4'd15;  B = 4'd15;  Cin = 1'b0;
        #10;

        // Test Case 4: Testing the full propagate path across all bits via Cin
        // 4'b1111 + 4'b0000 with Cin=1 will flag all P units high, pulling C[4] instantly high
        A = 4'b1111; B = 4'b0000; Cin = 1'b1;
        #10;

        // Test Case 5: Zero condition verification
        A = 4'd0;   B = 4'd0;   Cin = 1'b0;
        #10;

        $finish;
    end

endmodule
```

### Simulation results
<img width="1895" height="278" alt="image" src="https://github.com/user-attachments/assets/ed62ef1a-19d2-4582-b937-d54b3f360d4b" />
