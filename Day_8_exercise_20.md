# Exercise 20 — 4-bit Ring Counter
##  Implement a self-correcting ring counter that recovers from invalid states
### Truthtable

| Current State $Q_3 Q_2 Q_1 Q_0$ | Feedback Logic $\overline{Q_2 \lor Q_1 \lor Q_0}$ | Next State $Q_3^+ Q_2^+ Q_1^+ Q_0^+$ | Classification / Behavior |
| :---: | :---: | :---: | :--- |
| **0001** | $\overline{0 \lor 0 \lor 1} = 0$ | **0010** | **Valid State** (Advances normally) |
| **0010** | $\overline{0 \lor 1 \lor 0} = 0$ | **0100** | **Valid State** (Advances normally) |
| **0100** | $\overline{1 \lor 0 \lor 0} = 0$ | **1000** | **Valid State** (Advances normally) |
| **1000** | $\overline{0 \lor 0 \lor 0} = 1$ | **0001** | **Valid State** (Loops back to start) |
| *0000* | $\overline{0 \lor 0 \lor 0} = 1$ | **0001** | *Invalid State* $\rightarrow$ Recovers to Valid Sequence |
| *0011* | $\overline{0 \lor 1 \lor 1} = 0$ | **0110** | *Invalid State* $\rightarrow$ Transitions to `0110` |
| *0101* | $\overline{1 \lor 0 \lor 1} = 0$ | **1010** | *Invalid State* $\rightarrow$ Transitions to `1010` |
| *0110* | $\overline{1 \lor 1 \lor 0} = 0$ | **1100** | *Invalid State* $\rightarrow$ Transitions to `1100` |
| *0111* | $\overline{1 \lor 1 \lor 1} = 0$ | **1110** | *Invalid State* $\rightarrow$ Transitions to `1110` |
| *1001* | $\overline{0 \lor 0 \lor 1} = 0$ | **0010** | *Invalid State* $\rightarrow$ Recovers to Valid Sequence |
| *1010* | $\overline{0 \lor 1 \lor 0} = 0$ | **0100** | *Invalid State* $\rightarrow$ Recovers to Valid Sequence |
| *1011* | $\overline{0 \lor 1 \lor 1} = 0$ | **0110** | *Invalid State* $\rightarrow$ Transitions to `0110` |
| *1100* | $\overline{1 \lor 0 \lor 0} = 0$ | **1000** | *Invalid State* $\rightarrow$ Recovers to Valid Sequence |
| *1101* | $\overline{1 \lor 0 \lor 1} = 0$ | **1010** | *Invalid State* $\rightarrow$ Transitions to `1010` |
| *1110* | $\overline{1 \lor 1 \lor 0} = 0$ | **1100** | *Invalid State* $\rightarrow$ Transitions to `1100` |
| *1111* | $\overline{1 \lor 1 \lor 1} = 0$ | **1110** | *Invalid State* $\rightarrow$ Transitions to `1110` |

### Code
**Design Code**
```verilog
module self_correcting_ring_counter (
    input wire clk,
    input wire rst_n, // Active-low asynchronous reset
    output reg [3:0] q
);

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            q <= 4'b0001; 
        end else begin
            q <= {q[2:0], ~(q[2] | q[1] | q[0])};
        end
    end

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module tb_self_correcting_ring_counter;
    reg clk;
    reg rst_n;

    wire [3:0] q;

    self_correcting_ring_counter uut (
        .clk(clk),
        .rst_n(rst_n),
        .q(q)
    );

    always begin
        #10 clk = ~clk;
    end

    initial begin
        clk = 0;
        rst_n = 0;

        #40;
        rst_n = 1;
        
        #80;

        @(negedge clk);
        force uut.q = 4'b1010; // Force an invalid multi-token state
        
        @(negedge clk);
        release uut.q;         
        #100;
    
        @(negedge clk);
        force uut.q = 4'b0000;
        
        @(negedge clk);
        release uut.q;
        
        #100;
        $finish;
    end

    initial begin
        $dumpfile("tb_self_correcting_ring_counter.vcd");
        $dumpvars(0, tb_self_correcting_ring_counter);
        $monitor("Time = %0t ns | Reset = %b | Counter State (q) = %b", $time, rst_n, q);
    end

endmodule
```

### Simulation results

<img width="1900" height="309" alt="image" src="https://github.com/user-attachments/assets/eb99f34b-7ad3-4114-bf15-e30f76588762" />

## Build an 8-bit ring counter.
### Truthtable

| Clock Cycle | Current State ($Q_7 Q_6 Q_5 Q_4 Q_3 Q_2 Q_1 Q_0$) | Hex Value | Next State ($Q_7^+ Q_6^+ Q_5^+ Q_4^+ Q_3^+ Q_2^+ Q_1^+ Q_0^+$) | Description |
| :---: | :---: | :---: | :---: | :--- |
| **Reset** | `0000_0001` | `0x01` | `0000_0010` | Initial state with token at $Q_0$. |
| **1** | `0000_0010` | `0x02` | `0000_0100` | Token shifts left to $Q_1$. |
| **2** | `0000_0100` | `0x04` | `0000_1000` | Token shifts left to $Q_2$. |
| **3** | `0000_1000` | `0x08` | `0001_0000` | Token shifts left to $Q_3$. |
| **4** | `0001_0000` | `0x10` | `0010_0000` | Token shifts left to $Q_4$. |
| **5** | `0010_0000` | `0x20` | `0100_0000` | Token shifts left to $Q_5$. |
| **6** | `0100_0000` | `0x40` | `1000_0000` | Token shifts left to $Q_6$. |
| **7** | `1000_0000` | `0x80` | `0000_0001` | Token reaches $Q_7$ and wraps back around to $Q_0$. |
| **8** | `0000_0001` | `0x01` | `0000_0010` | Sequence repeats indefinitely. |

### Code
**Design Code**
```verilog
module ring_counter_8bit (
    input wire clk,
    input wire rst_n, // Active-low asynchronous reset
    output reg [7:0] q
);

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            q <= 8'b0000_0001; 
        end else begin
            q <= {q[6:0], q[7]};
        end
    end

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module tb_ring_counter_8bit;
    reg clk;
    reg rst_n;

    // Outputs
    wire [7:0] q;

    ring_counter_8bit uut (
        .clk(clk),
        .rst_n(rst_n),
        .q(q)
    );

    always begin
        #10 clk = ~clk;
    end

    initial begin
        clk = 0;
        rst_n = 0;

        #40;
        rst_n = 1; 
        
        #240;
        
        $finish;
    end

    initial begin
        $dumpfile("ring_counter_8bit_tb.vcd");
        $dumpvars(0, tb_ring_counter_8bit);
        $monitor("Time = %0t ns | Reset_n = %b | Counter State (q) = %b (Hex: 0x%h)", $time, rst_n, q, q);
    end

endmodule
```

### Simulation results

<img width="1895" height="298" alt="image" src="https://github.com/user-attachments/assets/cf5658c8-c2be-4d80-bbc0-c9c4f194bf5f" />

## Use a ring counter to generate non-overlapping 4-phase clock signals
### Truthtable

| Clock Cycle | CLK | Internal Counter State ($Q_3 Q_2 Q_1 Q_0$) | Phase 3 ($Ph_3$) | Phase 2 ($Ph_2$) | Phase 1 ($Ph_1$) | Phase 0 ($Ph_0$) | Functional Status / Output State |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **Reset** | 0 | `0001` | 0 | 0 | 0 | 0 | Reset active / All phases low |
| **1 (High)** | 1 | `0001` | 0 | 0 | 0 | **1** | **Phase 0 Active** |
| **1 (Low)** | 0 | `0001` | 0 | 0 | 0 | 0 | *Non-overlap Gap (All Low)* |
| **2 (High)** | 1 | `0010` | 0 | 0 | **1** | 0 | **Phase 1 Active** |
| **2 (Low)** | 0 | `0010` | 0 | 0 | 0 | 0 | *Non-overlap Gap (All Low)* |
| **3 (High)** | 1 | `0100` | 0 | **1** | 0 | 0 | **Phase 2 Active** |
| **3 (Low)** | 0 | `0100` | 0 | 0 | 0 | 0 | *Non-overlap Gap (All Low)* |
| **4 (High)** | 1 | `1000` | **1** | 0 | 0 | 0 | **Phase 3 Active** |
| **4 (Low)** | 0 | `1000` | 0 | 0 | 0 | 0 | *Non-overlap Gap (All Low)* |
| **5 (High)** | 1 | `0001` | 0 | 0 | 0 | **1** | **Sequence Loops Back to Phase 0** |

### Code
**Design Code**
```verilog
module non_overlapping_4phase (
    input wire clk,       // Master high-frequency clock
    input wire rst_n,     // Active-low asynchronous reset
    output wire ph0,      // Phase 0 signal
    output wire ph1,      // Phase 1 signal
    output wire ph2,      // Phase 2 signal
    output wire ph3       // Phase 3 signal
);

    reg [3:0] q;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            q <= 4'b0001; // Initialize with a single token
        end else begin
            q <= {q[2:0], q[3]}; // Circular shift left (q[3] loops to q[0])
        end
    end

    assign ph0 = q[0] & clk;
    assign ph1 = q[1] & clk;
    assign ph2 = q[2] & clk;
    assign ph3 = q[3] & clk;

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module tb_non_overlapping_4phase;

    reg clk;
    reg rst_n;

    wire ph0;
    wire ph1;
    wire ph2;
    wire ph3;

    non_overlapping_4phase uut (
        .clk(clk),
        .rst_n(rst_n),
        .ph0(ph0),
        .ph1(ph1),
        .ph2(ph2),
        .ph3(ph3)
    );

    always begin
        #5 clk = ~clk;
    end

    initial begin
        clk = 0;
        rst_n = 0;

        #20;
        rst_n = 1;

        #200;

        $finish;
    end

    initial begin
        $dumpfile("tb_non_overlapping_4phase.vcd");
        $dumpvars(0, tb_non_overlapping_4phase);
        $monitor("Time = %0t ns | clk = %b | Phases [3:0] = %b%b%b%b", 
                 $time, clk, ph3, ph2, ph1, ph0);
    end

endmodule
```

### Simulation results

<img width="1902" height="352" alt="image" src="https://github.com/user-attachments/assets/98fbae73-da6e-4a52-bb78-237b685743c6" />
