# Exercise 16 — T Flip-Flop
## Implement a T FF using a D FF and XOR gate.

### Truthtable

| T | Current State (Q<sub>present</sub>) | D = T ⊕ Q<sub>present</sub> | Next State (Q<sub>next</sub>) | Resulting Action |
|:-:|:----------------------------------:|:---------------------------:|:-----------------------------:|:-----------------|
| 0 | 0 | 0 | 0 | Hold (State stays 0) |
| 0 | 1 | 1 | 1 | Hold (State stays 1) |
| 1 | 0 | 1 | 1 | Toggle (0 → 1) |
| 1 | 1 | 0 | 0 | Toggle (1 → 0) |

### Code
**Design Code**
```verilog
module d_ff (
    input clk,
    input rst,
    input d,
    output reg q,
    output q_bar
);

    always @(posedge clk) begin
        if (rst)
            q <= 1'b0;
        else
            q <= d;
    end

    assign q_bar = ~q;

endmodule


module t_ff_using_dff (
    input clk,
    input rst,
    input t,
    output q,
    output q_bar
);

    wire d;
    wire q_int;

    // XOR gate: D = T ⊕ Q
    assign d = t ^ q_int;

    // D Flip-Flop
    d_ff dff (
        .clk(clk),
        .rst(rst),
        .d(d),
        .q(q_int),
        .q_bar(q_bar)
    );

    assign q = q_int;

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns/1ps
module tb_t_ff_using_dff;

    reg clk;
    reg rst;
    reg t;

    wire q;
    wire q_bar;

    // Instantiate the DUT
    t_ff_using_dff uut (
        .clk(clk),
        .rst(rst),
        .t(t),
        .q(q),
        .q_bar(q_bar)
    );

    // Clock generation (10 ns period)
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    // Stimulus
    initial begin
        // Generate VCD file
        $dumpfile("t_ff_using_dff.vcd");
        $dumpvars(0, tb_t_ff_using_dff);

        // Monitor signals
        $monitor("Time=%0t | RST=%b | T=%b | Q=%b | QB=%b",
                 $time, rst, t, q, q_bar);

        // Initialize
        rst = 1;
        t   = 0;

        #12;
        rst = 0;      // Release reset

        #10;
        t = 1;        // Toggle

        #20;
        t = 0;        // Hold

        #20;
        t = 1;        // Toggle

        #20;
        t = 1;        // Toggle again

        #20;
        t = 0;        // Hold

        #20;
        $finish;
    end

endmodule
```
 
### Simulation results

<img width="1898" height="272" alt="image" src="https://github.com/user-attachments/assets/bdde7aac-b932-4693-ba39-10d88847ab80" />


## Build a 4-bit ripple counter using 4 T flip-flops with T=1.

### Truthtable

| Clock Pulse | Q3 | Q2 | Q1 | Q0 | Decimal |
|:-----------:|:--:|:--:|:--:|:--:|:-------:|
| 0  | 0 | 0 | 0 | 0 | 0  |
| 1  | 0 | 0 | 0 | 1 | 1  |
| 2  | 0 | 0 | 1 | 0 | 2  |
| 3  | 0 | 0 | 1 | 1 | 3  |
| 4  | 0 | 1 | 0 | 0 | 4  |
| 5  | 0 | 1 | 0 | 1 | 5  |
| 6  | 0 | 1 | 1 | 0 | 6  |
| 7  | 0 | 1 | 1 | 1 | 7  |
| 8  | 1 | 0 | 0 | 0 | 8  |
| 9  | 1 | 0 | 0 | 1 | 9  |
| 10 | 1 | 0 | 1 | 0 | 10 |
| 11 | 1 | 0 | 1 | 1 | 11 |
| 12 | 1 | 1 | 0 | 0 | 12 |
| 13 | 1 | 1 | 0 | 1 | 13 |
| 14 | 1 | 1 | 1 | 0 | 14 |
| 15 | 1 | 1 | 1 | 1 | 15 |
| 16 | 0 | 0 | 0 | 0 | 0 (Repeats) |

### Code
**Design Code**
```verilog
module ripple_counter (
    input clk, rst, 
    output [3:0] q
);
    t_ff ff0 (clk,    rst, q[0]);
    t_ff ff1 (q[0],   rst, q[1]);
    t_ff ff2 (q[1],   rst, q[2]);
    t_ff ff3 (q[2],   rst, q[3]);
endmodule

module t_ff (
    input clk, rst, 
    output reg q
);
    always @(negedge clk or posedge rst) begin
        if (rst) q <= 1'b0;
        else     q <= ~q;
    end
endmodule
```

**Testbench Code**
```verilog
module tb_ripple_counter;
    reg clk = 0, rst = 1;
    wire [3:0] q;

    ripple_counter uut (clk, rst, q);

    always #5 clk = ~clk; // 10ns clock period

    initial begin
        $dumpfile("tb_ripple_counter.vcd");
        $dumpvars(0, tb_ripple_counter);
        $monitor("Time: %0t | q: %b", $time, q);
        #15 rst = 0;      // Release reset
        #170 $finish;     // Run long enough to see full 0-15 counting loop
    end
endmodule
```

### Simulation results

<img width="978" height="235" alt="image" src="https://github.com/user-attachments/assets/b458e8c2-d3da-46be-8461-df2626544928" />


## Create a frequency divider that outputs CLK/16.
### Truthtable

| Input Clock Cycles | Counter (Q3 Q2 Q1 Q0) | CLK/16 Output (Q3) |
|:------------------:|:---------------------:|:------------------:|
| 0  | 0000 | 0 |
| 1  | 0001 | 0 |
| 2  | 0010 | 0 |
| 3  | 0011 | 0 |
| 4  | 0100 | 0 |
| 5  | 0101 | 0 |
| 6  | 0110 | 0 |
| 7  | 0111 | 0 |
| 8  | 1000 | 1 |
| 9  | 1001 | 1 |
| 10 | 1010 | 1 |
| 11 | 1011 | 1 |
| 12 | 1100 | 1 |
| 13 | 1101 | 1 |
| 14 | 1110 | 1 |
| 15 | 1111 | 1 |
| 16 | 0000 | 0 (Repeats) |

### Code
**Design Code**
```verilog
module freq_div_16 (
    input clk,
    input rst,
    output clk_div16
);

    reg [3:0] count;

    always @(posedge clk) begin
        if (rst)
            count <= 4'b0000;
        else
            count <= count + 1'b1;
    end

    assign clk_div16 = count[3];

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns/1ps

module tb_freq_div_16;

    reg clk;
    reg rst;
    wire clk_div16;

    // DUT
    freq_div_16 uut (
        .clk(clk),
        .rst(rst),
        .clk_div16(clk_div16)
    );

    // Clock generation (10 ns period)
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    // Stimulus
    initial begin
        $dumpfile("freq_div_16.vcd");
        $dumpvars(0, tb_freq_div_16);

        $monitor("Time=%0t | rst=%b | clk=%b | clk_div16=%b",
                  $time, rst, clk, clk_div16);

        rst = 1;
        #12;
        rst = 0;

        // Run long enough to observe multiple divide-by-16 cycles
        #320;

        $finish;
    end

endmodule
```

### Simulation results

<img width="1898" height="235" alt="image" src="https://github.com/user-attachments/assets/904bba30-5964-4f0b-a645-e000b783eeb6" />
