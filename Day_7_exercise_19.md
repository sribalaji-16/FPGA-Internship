# Exercise 19 — 4-bit Synchronous Down Counter
## Create an up/down counter with a direction control signal.
### Truthtable

| rst | clk           | dir | Present Count (Q) | Next Count (Q+) | Operation |
|-----|--------------|-----|-------------------|----------------|-----------|
| 1   | X            | X   | XXXX              | 0000           | Asynchronous Reset |
| 0   | No Edge      | X   | Q                 | Q              | Hold State |
| 0   | Falling Edge | X   | Q                 | Q              | No Change |
| 0   | Rising Edge  | 1   | 0000              | 0001           | Count Up |
| 0   | Rising Edge  | 1   | 0001              | 0010           | Count Up |
| 0   | Rising Edge  | 1   | 0010              | 0011           | Count Up |
| 0   | Rising Edge  | 1   | ...               | ...            | Count Up |
| 0   | Rising Edge  | 1   | 1110              | 1111           | Count Up |
| 0   | Rising Edge  | 1   | 1111              | 0000           | Overflow (Wrap Around) |
| 0   | Rising Edge  | 0   | 0000              | 1111           | Underflow (Wrap Around) |
| 0   | Rising Edge  | 0   | 0001              | 0000           | Count Down |
| 0   | Rising Edge  | 0   | 0010              | 0001           | Count Down |
| 0   | Rising Edge  | 0   | 0011              | 0010           | Count Down |
| 0   | Rising Edge  | 0   | ...               | ...            | Count Down |
| 0   | Rising Edge  | 0   | 1111              | 1110           | Count Down |    

### Code
**Design Code**
```verilog
module up_down_counter #(
    parameter WIDTH = 4  // Parameterized bit width (default is 4-bit, 0 to 15)
)(
    input  wire             clk,   // Clock signal
    input  wire             rst,   // Synchronous reset (Active High)
    input  wire             dir,   // Direction control: 1 = Up, 0 = Down
    output reg [WIDTH-1:0] count  // Counter output
);

    always @(posedge clk) begin
        if (rst) begin
            count <= {WIDTH{1'b0}}; // Reset counter to 0
        end else begin
            if (dir) begin
                count <= count + 1'b1; // Count Up
            end else begin
                count <= count - 1'b1; // Count Down
            end
        end
    end

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module tb_up_down_counter;

    parameter WIDTH = 4;

    reg clk;
    reg rst;
    reg dir;

    wire [WIDTH-1:0] count;

    up_down_counter #(
        .WIDTH(WIDTH)
    ) uut (
        .clk(clk),
        .rst(rst),
        .dir(dir),
        .count(count)
    );

    always #10 clk = ~clk;

    initial begin
        clk = 0;
        rst = 0;
        dir = 1;

        $dumpfile("tb_up_down_counter.vcd"); // Name of the waveform file
        $dumpvars(0, tb_up_down_counter);    // Dump all variables in this module

        rst = 1;
        #25;
        rst = 0;
        
        dir = 1;
        #150; // Let it count up

        dir = 0;
        #150; // Let it count down

        rst = 1;
        #20;
        rst = 0;
        #50;
        $finish;
    end
endmodule
```

### Simulation results

<img width="980" height="344" alt="image" src="https://github.com/user-attachments/assets/fd306c0e-be2b-41f9-a9ff-1fa344b53761" />

## Implement a programmable modulo-N down counter.

### Truthtable

| Reset (rst) | Load (load) | Clock (clk) | Current Count (Q) | Next Count (Q+) | Operation |
|-------------|-------------|-------------|-------------------|-----------------|-----------|
| 1 | X | X | XXXX | 0000 | Asynchronous Reset |
| 0 | 1 | X | XXXX | N - 1 | Load Modulus Value |
| 0 | 0 | No Edge | Q | Q | Hold State |
| 0 | 0 | Falling Edge (↓) | Q | Q | No Change |
| 0 | 0 | Rising Edge (↑) | Q > 0 | Q - 1 | Count Down |
| 0 | 0 | Rising Edge (↑) | Q = 0 | N - 1 | Reload and Wrap Around |

### Code
**Design Code**
```verilog
module modulo_n_down_counter #(
    parameter WIDTH = 8 // Default bit-width allowing N up to 256
)(
    input wire clk,
    input wire rst,
    input wire load,                 // Signal to load a new modulo value
    input wire [WIDTH-1:0] mod_n,    // The programmable Modulo-N value
    output reg [WIDTH-1:0] count,
    output wire zero                 // High when counter reaches 0
);

    // Terminal count flag: asserted when counter reaches 0
    assign zero = (count == {WIDTH{1'b0}});

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            count <= {WIDTH{1'b0}};
        end else if (load) begin
            // Load the start value for Modulo-N (N-1)
            // If mod_n is 10, it counts from 9 down to 0 (10 states)
            count <= (mod_n > 0) ? (mod_n - 1'b1) : {WIDTH{1'b0}};
        end else begin
            if (zero) begin
                // Reload the wrap-around value when 0 is reached
                count <= (mod_n > 0) ? (mod_n - 1'b1) : {WIDTH{1'b0}};
            end else begin
                // Decrement counter
                count <= count - 1'b1;
            end
        end
    end

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module modulo_n_down_counter_tb;

    parameter WIDTH = 8;

    reg clk;
    reg rst;
    reg load;
    reg [WIDTH-1:0] mod_n;

    wire [WIDTH-1:0] count;
    wire zero;

    modulo_n_down_counter #(
        .WIDTH(WIDTH)
    ) uut (
        .clk(clk),
        .rst(rst),
        .load(load),
        .mod_n(mod_n),
        .count(count),
        .zero(zero)
    );

    always #10 clk = ~clk;

    initial begin
        clk = 0;
        rst = 0;
        load = 0;
        mod_n = 0;

        $dumpfile("modulo_n_down_counter_tb.vcd");
        $dumpvars(0, modulo_n_down_counter_tb);
        
        $monitor("Time: %0t | Count: %0d | Zero: %b | Modulo-N: %0d | Load: %b", $time, count, zero, mod_n, load);
        rst = 1;
        #25;
        rst = 0;
        #15;

        mod_n = 8'd5;
        load = 1;
        #20; // Hold load for 1 clock cycle
        load = 0;

        #140;

        mod_n = 8'd3;
        load = 1;
        #20;
        load = 0;

        #100;

        $finish;
    end
endmodule
```

### Simulation results

<img width="1897" height="408" alt="image" src="https://github.com/user-attachments/assets/ad8fbd8c-3dc0-4e11-9211-93b77259902a" />

##  Design a pulse-width modulator using an up-down counter.
### Truthtable

| Reset (`rst`) | Clock (`clk`) | Current Direction (`dir`) | Current Count (`count`) | Next Direction (`dir_next`) | Next Count (`count_next`) | Description / Operation |
| :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| `1` | X | X | X | `1` (UP) | `0` | **Asynchronous Reset**: Overrides everything immediately. |
| `0` | No Edge | X | X | `dir` | `count` | **No Clock**: State remains unchanged. |
| `0` | Up-Edge (↑) | `1` (UP) | < `MAX_COUNT - 1` | `1` (UP) | `count + 1` | **Counting Up**: Incrementing through the rising ramp. |
| `0` | Up-Edge (↑) | `1` (UP) | == `MAX_COUNT - 1` | `0` (DOWN) | `count + 1` | **Peak Threshold**: Reaches `MAX_COUNT`, switch direction to DOWN. |
| `0` | Up-Edge (↑) | `0` (DOWN) | > `1` | `0` (DOWN) | `count - 1` | **Counting Down**: Decrementing through the falling ramp. |
| `0` | Up-Edge (↑) | `0` (DOWN) | == `1` | `1` (UP) | `count - 1` | **Bottom Threshold**: Reaches `0`, switch direction back to UP. |

### Code
**Design Code**
```verilog
module pwm_up_down #(
    parameter WIDTH = 8,
    parameter MAX_COUNT = 8'd255 // Maximum peak of the triangle wave
)(
    input wire clk,
    input wire rst,
    input wire [WIDTH-1:0] duty_cycle, // Input to control brightness/speed
    output reg pwm_out
);

    reg [WIDTH-1:0] count;
    reg dir; // 1 for counting UP, 0 for counting DOWN

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            count <= {WIDTH{1'b0}};
            dir   <= 1'b1; // Start by counting UP
        end else begin
            if (dir == 1'b1) begin
                if (count == MAX_COUNT - 1'b1) begin
                    count <= count + 1'b1;
                    dir   <= 1'b0; // Turn around at peak
                end else begin
                    count <= count + 1'b1;
                end
            end else begin
                if (count == {WIDTH{1'b0}} + 1'b1) begin
                    count <= count - 1'b1;
                    dir   <= 1'b1; // Turn around at bottom
                end else begin
                    count <= count - 1'b1;
                end
            end
        end
    end

    always @(*) begin
        if (count < duty_cycle) begin
            pwm_out = 1'b1;
        end else begin
            pwm_out = 1'b0;
        end
    end

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module pwm_up_down_tb;

    parameter WIDTH = 8;
    parameter MAX_COUNT = 8'd15; // Kept small for clear visual waveform analysis

    reg clk;
    reg rst;
    reg [WIDTH-1:0] duty_cycle;

    wire pwm_out;

    pwm_up_down #(
        .WIDTH(WIDTH),
        .MAX_COUNT(MAX_COUNT)
    ) uut (
        .clk(clk),
        .rst(rst),
        .duty_cycle(duty_cycle),
        .pwm_out(pwm_out)
    );

    always #10 clk = ~clk;

    initial begin
        clk = 0;
        rst = 0;
        duty_cycle = 0;

        $dumpfile("pwm_up_down_tb.vcd");
        $dumpvars(0, pwm_up_down_tb);

        rst = 1;
        #25;
        rst = 0;
        #15;

        duty_cycle = 8'd4; 
        #650; // Run for slightly more than one full triangle period (30 clock cycles)

        duty_cycle = 8'd11;
        #650;

        $finish;
    end

endmodule
```
### Simulation results

<img width="1901" height="319" alt="image" src="https://github.com/user-attachments/assets/e15e63d5-4980-4da5-a8a9-48c44be87711" />

