# Exercise 22 — Clock Divider
## Design a divide-by-3 clock divider with 50% duty cycle.
### Truthtable

| Current State (Q1) | Current State (Q0) | Next State (Q1⁺) | Next State (Q0⁺) | Positive-Edge Clock (pos_clk) |
|:------------------:|:------------------:|:----------------:|:----------------:|:-----------------------------:|
| 0 | 0 | 0 | 1 | 1 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 0 | 1 |
| 1 | 1 | X (or 0) | X (or 0) | X (or 0) |

### Code
**Design Code**
```verilog
module clk_div3 (
    input clk, rst_n,
    output clk_out
);
    reg [1:0] q;
    reg neg_clk;
    wire pos_clk;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) q <= 2'b00;
        else        q <= (q == 2'd2) ? 2'b00 : q + 1'b1;
    end

    assign pos_clk = (q == 2'd0 || q == 2'd1);

    always @(negedge clk or negedge rst_n) begin
        if (!rst_n) neg_clk <= 1'b0;
        else        neg_clk <= pos_clk;
    end

    assign clk_out = pos_clk | neg_clk;
endmodule
```

**Testbench Code**
```verilog
`timescale 1ns/1ps
module tb_clk_div3;
    reg clk = 0;
    reg rst_n = 0;
    wire clk_out;

    clk_div3 uut (.clk(clk), .rst_n(rst_n), .clk_out(clk_out));

    always #5 clk = ~clk;

    initial begin
        $dumpfile("tb_clk_div3.vcd");
        $dumpvars(0, tb_clk_div3);
        $monitor("Time: %0t | clk: %b | rst_n: %b | clk_out: %b", $time, clk, rst_n, clk_out);
        #12 rst_n = 1; // Release reset
        #100 $finish;  // Run simulation
    end
endmodule
```

### Simulation Code

<img width="1896" height="230" alt="image" src="https://github.com/user-attachments/assets/bbbc1d2b-b4a0-4447-b9f0-33b6d5820a88" />

##  Build a parameterized divide-by-N clock divider.
### Truthtable

| pos_count (Current) | pos_count (Next) | pos_clk (Output) |
|:-------------------:|:----------------:|:----------------:|
| 0 | 1 | 1 (0 < 2) |
| 1 | 2 | 1 (1 < 2) |
| 2 | 3 | 0 (2 ≮ 2) |
| 3 | 0 | 0 (3 ≮ 2) |

### Code
**Design Code**
```verilog
module param_clk_divider #(
    parameter N = 3 // Default division factor
)(
    input  wire clk,
    input  wire rst_n,
    output wire clk_out
);
    localparam WIDTH = (N > 1) ? $clog2(N) : 1;

    reg [WIDTH-1:0] pos_count;
    reg [WIDTH-1:0] neg_count;
    wire pos_clk;
    wire neg_clk;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) 
            pos_count <= 0;
        else 
            pos_count <= (pos_count == N - 1) ? 0 : pos_count + 1'b1;
    end
    assign pos_clk = (pos_count < N/2);

    always @(negedge clk or negedge rst_n) begin
        if (!rst_n) 
            neg_count <= 0;
        else 
            neg_count <= (neg_count == N - 1) ? 0 : neg_count + 1'b1;
    end
    assign neg_clk = (neg_count < N/2);

    assign clk_out = (N == 1)   ? clk : 
                     (N % 2 == 0) ? pos_clk : 
                     (pos_clk | neg_clk);
endmodule
```

**Testbench Code**
```verilog
`timescale 1ns/1ps
module tb_param_clk_divider;
    reg clk = 0;
    reg rst_n = 0;
    wire clk_out;

    param_clk_divider #(.N(5)) uut (
        .clk(clk),
        .rst_n(rst_n),
        .clk_out(clk_out)
    );

    always #5 clk = ~clk;

    initial begin
        $dumpfile("tb_param_clk_divider.vcd");
        $dumpvars(0, tb_param_clk_divider);
        $monitor("Time: %0t | clk: %b | rst_n: %b | clk_out: %b", $time, clk, rst_n, clk_out);
        #15 rst_n = 1;   // Release Reset
        #200 $finish;    // Run simulation long enough to observe cycles
    end
endmodule
```

### Simulation results

<img width="1899" height="231" alt="image" src="https://github.com/user-attachments/assets/cec17632-e577-4411-9b89-f0d2c088fdee" />

## Implement a clock gating cell using AND gate + latch.
### Truthtable

| clk | en | te | Enet | en_latched(prev) | en_latched(next) | gated_clk |
|:---:|:--:|:--:|:----:|:----------------:|:----------------:|:---------:|
| 0 | 0 | 0 | 0 | 0 | 0 (transparent) | 0 |
| 0 | 0 | 1 | 1 | 0 | 1 (transparent) | 0 |
| 0 | 1 | 0 | 1 | 0 | 1 (transparent) | 0 |
| 0 | 1 | 1 | 1 | 1 | 1 (transparent) | 0 |
| 0 | X | 1 | 1 | 0 | 1 (transparent) | 0 |
| 0 | X | 1 | 1 | 1 | 1 (transparent) | 0 |
| 1 | X | X | X | 0 | 0 (latched/holding) | 0 (1 · 0) |
| 1 | X | X | X | 1 | 1 (latched/holding) | 1 (1 · 1) |

### Code
**Design Code**
```verilog
module clk_gate_cell (
    input  wire clk,
    input  wire en,
    input  wire te,
    output wire gated_clk
);
    wire en_net;
    reg  en_latched;

    assign en_net = en | te;

    always @(clk or en_net) begin
        if (!clk) begin
            en_latched <= en_net;
        end
    end

    assign gated_clk = clk & en_latched;
endmodule
```

**Testbench Code**
```verilog
`timescale 1ns/1ps
module tb_clk_gate_cell;
    reg clk = 0;
    reg en = 0;
    reg te = 0;
    wire gated_clk;

    clk_gate_cell uut (.clk(clk), .en(en), .te(te), .gated_clk(gated_clk));

    always #5 clk = ~clk;

    initial begin
        $dumpfile("tb_clk_gate_cell.vcd");
        $dumpvars(0, tb_clk_gate_cell);
        $monitor("Time: %0t | clk: %b | en: %b | te: %b | gated_clk: %b", $time, clk, en, te, gated_clk);
        #7  en = 1; 
        #5  en = 0;
        
        #3  en = 1; 
        #20 en = 0; 
        
        #10 te = 1; 
        #20 $finish;
    end
endmodule
```

### Simulations results

<img width="1903" height="267" alt="image" src="https://github.com/user-attachments/assets/f8ffc2bf-9356-402d-b8f1-d59c4513c055" />
