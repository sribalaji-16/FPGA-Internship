# Exercise 24 — Sequence Detector — Mealy FSM
## Draw the complete state transition diagram with all arc labels
### State transition diagram

<img width="1600" height="1346" alt="image" src="https://github.com/user-attachments/assets/911f68af-358e-4b4f-9999-8fcf7d5cc2b0" />

### State transition table

**Overlapping Sequence Detector (1011)**

| Current State | Input (x) | Next State | Output (y) | Rationale / Sequence Tracked |
|:-------------:|:---------:|:----------:|:----------:|:-----------------------------|
| S0 (IDLE) | 0 | S0 | 0 | No progress (0) |
| S0 (IDLE) | 1 | S1 | 0 | Started sequence (1) |
| S1 (got 1) | 0 | S2 | 0 | Progressing (10) |
| S1 (got 1) | 1 | S1 | 0 | Keeps the first bit active (11 → last bit is 1) |
| S2 (got 10) | 0 | S0 | 0 | Sequence broken (100). Reset to IDLE |
| S2 (got 10) | 1 | S3 | 0 | Progressing (101) |
| S3 (got 101) | 0 | S2 | 0 | Pattern becomes 1010. The last two bits are 10, so go to S2 |
| S3 (got 101) | 1 | S1 | 1 | Sequence **1011** detected! Overlaps; last **1** becomes the first **1** of the next sequence |

**Non-Overlapping Sequence Detector (1011)**

| Current State | Input (x) | Next State | Output (y) | Rationale / Sequence Tracked |
|:-------------:|:---------:|:----------:|:----------:|:-----------------------------|
| S0 (IDLE) | 0 | S0 | 0 | No progress (0) |
| S0 (IDLE) | 1 | S1 | 0 | Started sequence (1) |
| S1 (got 1) | 0 | S2 | 0 | Progressing (10) |
| S1 (got 1) | 1 | S1 | 0 | Keeps the first bit active (11 → last bit is 1) |
| S2 (got 10) | 0 | S0 | 0 | Sequence broken (100). Reset to IDLE |
| S2 (got 10) | 1 | S3 | 0 | Progressing (101) |
| S3 (got 101) | 0 | S2 | 0 | Pattern becomes 1010. The last two bits are 10, so go to S2 |
| S3 (got 101) | 1 | S0 | 1 | Sequence **1011** detected! No overlap allowed; reset to IDLE |


## Modify to detect the overlapping sequence '1011' (allow overlap).
### Code
**Design Code**
```verilog
module sequence_detector_mealy (
    input  wire clk,
    input  wire rst,
    input  wire x,  
    output reg  y   
);

    parameter S0_IDLE = 2'b00,
              S1_GOT1 = 2'b01,
              S2_GOT10 = 2'b10,
              S3_GOT101 = 2'b11;

    reg [1:0] current_state, next_state;

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            current_state <= S0_IDLE;
        end else begin
            current_state <= next_state;
        end
    end

    always @(*) begin
        next_state = current_state;
        y = 1'b0;

        case (current_state)
            S0_IDLE: begin
                if (x == 1'b1) begin
                    next_state = S1_GOT1;
                end else begin
                    next_state = S0_IDLE;
                end
            end

            S1_GOT1: begin
                if (x == 1'b0) begin
                    next_state = S2_GOT10;
                end else begin
                    next_state = S1_GOT1;
                end
            end

            S2_GOT10: begin
                if (x == 1'b1) begin
                    next_state = S3_GOT101;
                end else begin
                    next_state = S0_IDLE;  
                end
            end

            S3_GOT101: begin
                if (x == 1'b1) begin
                    next_state = S1_GOT1; 
                    y = 1'b1;             
                end else begin
                    next_state = S2_GOT10;
                end
            end

            default: begin
                next_state = S0_IDLE;
                y = 1'b0;
            end
        endcase
    end

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module tb_sequence_detector_mealy;

    reg clk;
    reg rst;
    reg x;

    wire y;

    sequence_detector_mealy uut (
        .clk(clk),
        .rst(rst),
        .x(x),
        .y(y)
    );

    always begin
        #5 clk = ~clk;
    end

    initial begin
        $dumpfile("tb_sequence_detector_mealy.vcd");
        $dumpvars(0, tb_sequence_detector_mealy);
        clk = 0;
        rst = 1;
        x = 0;

        $monitor("Time = %0dtns | rst = %b | Input (x) = %b | Output (y) = %b | State = %b", 
                 $time, rst, x, y, uut.current_state);

        #15;
        rst = 0;
        
        @(posedge clk); x = 1; // Bit 1
        @(posedge clk); x = 0; // Bit 0
        @(posedge clk); x = 1; // Bit 1
        @(posedge clk); x = 1; // Bit 1 -> First '1011' completed here! (y should go high)
        
        @(posedge clk); x = 0; // Bit 0 -> Moving from S1 to S2 due to overlap
        @(posedge clk); x = 1; // Bit 1
        @(posedge clk); x = 1; // Bit 1 -> Second '1011' completed here! (y should go high)

        @(posedge clk); x = 0; // Bit 0
        @(posedge clk); x = 0; // Bit 0 -> Breaks pattern completely, pushes FSM to S0_IDLE
        @(posedge clk); x = 1; // Bit 1
        
        #20;
        $finish;
    end
      
endmodule
````

### Simulation results

<img width="1901" height="291" alt="image" src="https://github.com/user-attachments/assets/8ceb66c0-4934-452b-901e-684531fd5af2" />

## Compare latency with the equivalent Moore FSM implementation
| Feature | Mealy FSM (1011 Sequence Detector) | Moore FSM (1011 Sequence Detector) |
|:--------|:-----------------------------------|:-----------------------------------|
| **Output Latency** | **0 Clock Cycles (Immediate):** The output goes high in the exact same clock cycle that the final bit arrives. | **1 Clock Cycle Delay:** The output goes high on the next active clock edge after the final bit arrives. |
| **Output Dependence** | Depends on both the **current state** and the **current input (`x`)**. | Depends **only on the current state**. |
| **Total States Required** | **4 States:** `S0`, `S1`, `S2`, `S3`. | **5 States:** `S0`, `S1`, `S2`, `S3`, and a dedicated detection state `S4`. |
| **Detection Timing** | The target pattern is detected on the **transition** leaving the partial-match state (`S3`). | The target pattern is detected only after the FSM **enters the final detection state (`S4`)**. |
| **Hardware Vulnerability** | **Prone to glitches:** Asynchronous input changes or noise on `x` can directly affect the output `y` within a clock cycle. | **Glitch-free / Synchronous:** Outputs change only on the active clock edge, making them highly stable. |
