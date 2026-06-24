# Exercise 21 — 4-bit Johnson Counter
## Decode the Johnson counter outputs to obtain 1-of-8 signals
### Truthtable 

| Clock Pulse | Q0 | Q1 | Q2 | Q3 | Active Output | Decoding Expression |
|:-----------:|:--:|:--:|:--:|:--:|:-------------:|:-------------------|
| 0 (Reset) | 0 | 0 | 0 | 0 | Y0 | Q̅0 · Q̅3 |
| 1 | 1 | 0 | 0 | 0 | Y1 | Q0 · Q̅1 |
| 2 | 1 | 1 | 0 | 0 | Y2 | Q1 · Q̅2 |
| 3 | 1 | 1 | 1 | 0 | Y3 | Q2 · Q̅3 |
| 4 | 1 | 1 | 1 | 1 | Y4 | Q0 · Q3 |
| 5 | 0 | 1 | 1 | 1 | Y5 | Q̅0 · Q1 |
| 6 | 0 | 0 | 1 | 1 | Y6 | Q̅1 · Q2 |
| 7 | 0 | 0 | 0 | 1 | Y7 | Q̅2 · Q3 |

### Code
**Design Code**
```verilog
module johnson_decode(input clk, rst, output [7:0] y);
    reg [3:0] q;
    // 4-bit twisted ring shift register
    always @(posedge clk or posedge rst)
        q <= rst ? 4'b0000 : {~q[0], q[3:1]}; 

    // Continuous assignment for 1-of-8 decoded outputs
    assign y[0] = ~q[3] & ~q[2]; // Simplified or direct match from truth table
    assign y[1] =  q[3] & ~q[2];
    assign y[2] =  q[2] & ~q[1];
    assign y[3] =  q[1] & ~q[0];
    assign y[4] =  q[3] &  q[2]; // Note: index ordering depends on shift direction. 
    assign y[5] = ~q[3] &  q[2]; // This matches a right-shifting sequence:
    assign y[6] = ~q[2] &  q[1]; // rst->1000->1100->1111->0111->0011->0001
    assign y[7] = ~q[1] &  q[0];
endmodule
```

**Testbench Code**
```verilog
module tb;
    reg clk = 0, rst;
    wire [7:0] y;

    johnson_decode uut (clk, rst, y);

    always #5 clk = ~clk; // 10ns clock period

    initial begin
        $dumpfile("johnson_decode_tb.vcd");
        $dumpvars(0, tb);
        rst = 1; #10 rst = 0;
        #90 $finish; // Run long enough to see all 8 states decoded
    end

    initial $monitor("Time=%0t | rst=%b | Decoded Output Y(7:0)=%b", $time, rst, y);
endmodule
```

### Simulation results

<img width="983" height="234" alt="image" src="https://github.com/user-attachments/assets/15310dab-d3c0-4898-b37f-2944809a4ac8" />

## Build a Johnson counter with programmable length using a parameter
### Truthtable

| State / Clock | Q0 | Q1 | ... | QN-1 | Decoded Active Condition |
|:-------------:|:--:|:--:|:---:|:----:|:------------------------|
| 0 (Reset) | 0 | 0 | ... | 0 | Q̅0 · Q̅N-1 |
| 1 | 1 | 0 | ... | 0 | Q0 · Q̅1 |
| 2 | 1 | 1 | ... | 0 | Q1 · Q̅2 |
| ... | ... | ... | ... | ... | ... |
| N | 1 | 1 | ... | 1 | QN-2 · QN-1 |
| N+1 | 0 | 1 | ... | 1 | Q̅0 · Q1 |
| ... | ... | ... | ... | ... | ... |
| 2N−1 | 0 | 0 | ... | 1 | Q̅N-2 · QN-1 |

### Code
**Design Code**
```verilog
module programmable_johnson #(parameter N = 4) (
    input clk, rst,
    output reg [N-1:0] q
);
    always @(posedge clk or posedge rst)
        q <= rst ? {N{1'b0}} : {q[N-2:0], ~q[N-1]};
endmodule
```

**Testbench Code**
```verilog
module tb;
    reg clk = 0, rst;
    wire [2:0] q;

    programmable_johnson #(.N(3)) uut (clk, rst, q);

    always #5 clk = ~clk;

    initial begin
        $dumpfile("programmable_johnson_tb.vcd");
        $dumpvars(0, tb);
        rst = 1; #12 rst = 0;
        #70 $finish;
    end

    initial $monitor("Time=%0t | rst=%b | Counter State (q)=%b", $time, rst, q);
endmodule
```

### Simulation results

<img width="983" height="238" alt="image" src="https://github.com/user-attachments/assets/9ff9a542-3fe2-44e0-8ed2-79367132a767" />

## Compare Gray code counter with Johnson counter state sequences.

### Truthtable

| Clock Pulse | Gray Code Sequence (G2 G1 G0) | Johnson Sequence (Q2 Q1 Q0) |
|:-----------:|:-----------------------------:|:----------------------------:|
| 0 (Reset) | 000 | 000 |
| 1 | 001 | 100 |
| 2 | 011 | 110 |
| 3 | 010 | 111 |
| 4 | 110 | 011 |
| 5 | 111 | 001 |
| 6 | 101 | 000 (Repeats) |
| 7 | 100 | — |

### Code
**Design Code**
```verilog
module counter_comparison(
    input clk, rst,
    output reg [2:0] johnson,
    output wire [2:0] gray
);
    reg [2:0] binary;

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            johnson <= 3'b000;
            binary  <= 3'b000;
        end else begin
            johnson <= {johnson[1:0], ~johnson[2]}; // Right shift with inverted feedback
            binary  <= binary + 1;                  // Internal binary tracking for Gray
        end
    end

    assign gray = binary ^ (binary >> 1);
endmodule
```

**Testbench Code**
```verilog
module tb;
    reg clk = 0, rst;
    wire [2:0] johnson, gray;

    counter_comparison uut (clk, rst, johnson, gray);

    always #5 clk = ~clk; // Generated clock signal

    initial begin
        $dumpfile("counter_comparison.vcd");
        $dumpvars(0, tb);
        rst = 1; #10 rst = 0;
        #90 $finish;
    end

    initial $monitor("Time=%0t | Johnson=%b | Gray=%b", $time, johnson, gray);
endmodule
```

### Simulation results

<img width="1033" height="250" alt="image" src="https://github.com/user-attachments/assets/72e3d7dd-9bad-497a-83b8-3a5daa023945" />
