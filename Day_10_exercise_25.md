# Exercise 25 — Sequence Detector — Moore FSM
## State transition table for Sequence Detector for `1011`

**Overlapping Sequence Detector - `1011`**
| Current State | Next State (X = 0) | Next State (X = 1) | Output (Y) |
|:-------------:|:------------------:|:------------------:|:----------:|
| S0 (Reset) | S0 | S1 | 0 |
| S1 (1) | S2 | S1 | 0 |
| S2 (10) | S0 | S3 | 0 |
| S3 (101) | S2 | S4 | 0 |
| S4 (1011) | S2 | S1 | 1 |

**Non-Overlapping Sequence Detector - `1011`**
| Current State | Next State (X = 0) | Next State (X = 1) | Output (Y) |
|:-------------:|:------------------:|:------------------:|:----------:|
| S0 (Reset) | S0 | S1 | 0 |
| S1 (1) | S2 | S1 | 0 |
| S2 (10) | S0 | S3 | 0 |
| S3 (101) | S2 | S4 | 0 |
| S4 (1011) | S0 | S1 | 1 |

## Add an overlap detection mode where S4 transitions to S1 (not S0) on x=1.
### State transition table

| Current State | Next State (X = 0)<br>Non-Overlapping | Next State (X = 0)<br>Overlapping | Next State (X = 1)<br>Both Modes | Output (Y) |
|:-------------:|:----------------------------------------:|:-----------------------------------:|:----------------------------------:|:----------:|
| S0 (Reset) | S0 | S0 | S1 | 0 |
| S1 (1) | S2 | S2 | S1 | 0 |
| S2 (10) | S0 | S0 | S3 | 0 |
| S3 (101) | S2 | S2 | S4 | 0 |
| S4 (1011) | S0 | S2 | S1 | 1 |

### Code
**Design Code**
```verilog
module seq_detector_1011_moore (
    input  wire clk,
    input  wire rst_n,    // Active-low asynchronous reset
    input  wire x,        // Serial input bit
    output reg  y         // Sequence detected output
);

    parameter OVERLAP_MODE = 1;

    localparam [2:0] S0 = 3'b000,
                     S1 = 3'b001,
                     S2 = 3'b010,
                     S3 = 3'b011,
                     S4 = 3'b100;

    reg [2:0] current_state, next_state;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            current_state <= S0;
        end else begin
            current_state <= next_state;
        end
    end

    always @(*) begin
        case (current_state)
            S0: begin
                if (x) next_state = S1;
                else   next_state = S0;
            end
            
            S1: begin
                if (x) next_state = S1;
                else   next_state = S2;
            end
            
            S2: begin
                if (x) next_state = S3;
                else   next_state = S0;
            end
            
            S3: begin
                if (x) next_state = S4;
                else   next_state = S2;
            end
            
            S4: begin
                if (x) begin
                    next_state = S1;
                end else begin
                    if (OVERLAP_MODE)
                        next_state = S2; // Overlapping: Retain '10'
                    else
                        next_state = S0; // Non-overlapping: Complete reset
                end
            end
            
            default: next_state = S0;
        endcase
    end

    always @(*) begin
        case (current_state)
            S4:      y = 1'b1;
            default: y = 1'b0;
        endcase
    end

endmodule
```

**Testbench Code**
```verilog
module tb_seq_detector;
    reg clk;
    reg rst_n;
    reg x;
    wire y;

    seq_detector_1011_moore #(.OVERLAP_MODE(1)) uut (
        .clk(clk),
        .rst_n(rst_n),
        .x(x),
        .y(y)
    );

    always #5 clk = ~clk;

    initial begin
        $dumpfile("tb_seq_detector.vcd");
        $dumpvars(0, tb_seq_detector);
        $monitor("Time: %0t | clk: %b | rst_n: %b | x: %b | y: %b", $time, clk, rst_n, x, y);
        clk = 0; rst_n = 0; x = 0;
        #15 rst_n = 1;
        
        @(posedge clk); x = 1;
        @(posedge clk); x = 0;
        @(posedge clk); x = 1;
        @(posedge clk); x = 1; // First detection happens here (state becomes S4)
        @(posedge clk); x = 0; // S4 transitions to S2 because of overlap
        @(posedge clk); x = 1;
        @(posedge clk); x = 1; // Second detection
        @(posedge clk); x = 0;
        
        #40 $finish;
    end
endmodule
```

### Simulation results

<img width="1898" height="254" alt="image" src="https://github.com/user-attachments/assets/32df6414-8f43-4b00-b6e4-a17bdeda6c27" />

## Encode the state using one-hot encoding and compare synthesis results.
### State transition table

| Current State Name | Current State Vector (Q4 Q3 Q2 Q1 Q0) | Next State Vector (X = 0)<br>Non-Overlapping | Next State Vector (X = 0)<br>Overlapping | Next State Vector (X = 1)<br>Both Modes | Output (Y) |
|:------------------:|:-------------------------------------:|:--------------------------------------------:|:-----------------------------------------:|:------------------------------------------:|:----------:|
| S0 (Reset) | 00001 | 00001 (S0) | 00001 (S0) | 00010 (S1) | 0 |
| S1 (1) | 00010 | 00100 (S2) | 00100 (S2) | 00010 (S1) | 0 |
| S2 (10) | 00100 | 00001 (S0) | 00001 (S0) | 01000 (S3) | 0 |
| S3 (101) | 01000 | 00100 (S2) | 00100 (S2) | 10000 (S4) | 0 |
| S4 (1011) | 10000 | 00001 (S0) | 00100 (S2) | 00010 (S1) | 1 |

### Synthesis Comparison: Binary vs. One-Hot

| Feature / Metric | Binary / Sequential Encoding (Previous) | One-Hot Encoding (Current) |
|:-----------------|:-----------------------------------------|:---------------------------|
| **State Vector Width** | 3 bits (`⌈log₂(5)⌉`) | 5 bits (1 bit per state) |
| **Flip-Flop Count** | 3 Registers | 5 Registers |
| **Combinational Logic / Gate Count** | Higher: Requires complex decoding logic to decode 3 bits into next-state transitions. | Lower: Next-state equations depend on single bits, resulting in simpler excitation equations. |
| **Output Decoding Depth** | Requires multi-input AND/OR gates to decode state bits (e.g., `Y = Q₂ · Q̅₁ · Q̅₀`). | Zero Delay: The output can directly tap the corresponding state bit register (`Y = Q₄`). |
| **Maximum Frequency (F<sub>max</sub>)** | Lower potential due to deeper combinational critical paths in next-state decoding. | Higher potential: Shorter combinational paths between registers enable faster clock speeds. |
| **Target Hardware Suitability** | Excellent for CPLDs / ASICs where flip-flops are costly relative to combinational gates. | Excellent for FPGAs where LUTs are paired heavily with abundant, "free" flip-flops. |

### Code
**Design Code**
```verilog
module seq_detector_1011_one_hot (
    input  wire clk,
    input  wire rst_n,
    input  wire x,
    output reg  y
);

    parameter OVERLAP_MODE = 1;

    localparam [4:0] S0 = 5'b00001,
                     S1 = 5'b00010,
                     S2 = 5'b00100,
                     S3 = 5'b01000,
                     S4 = 5'b10000;

    reg [4:0] current_state, next_state;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            current_state <= S0;
        end else begin
            current_state <= next_state;
        end
    end

    always @(*) begin
        case (current_state)
            S0: begin
                if (x) next_state = S1;
                else   next_state = S0;
            end
            
            S1: begin
                if (x) next_state = S1;
                else   next_state = S2;
            end
            
            S2: begin
                if (x) next_state = S3;
                else   next_state = S0;
            end
            
            S3: begin
                if (x) next_state = S4;
                else   next_state = S2;
            end
            
            S4: begin
                if (x) begin
                    next_state = S1;
                end else begin
                    if (OVERLAP_MODE)
                        next_state = S2;
                    else
                        next_state = S0;
                end
            end
            
            default: next_state = S0;
        endcase
    end

    always @(*) begin
        y = current_state[4]; 
    end

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns/1ps
module tb_seq_detector_one_hot;

    reg clk;
    reg rst_n;
    reg x;
    wire y;

    seq_detector_1011_one_hot #(.OVERLAP_MODE(1)) uut (
        .clk(clk),
        .rst_n(rst_n),
        .x(x),
        .y(y)
    );

    always #5 clk = ~clk;

    initial begin
        $dumpfile("tb_seq_detector_one_hot.vcd");
        $dumpvars(0, tb_seq_detector_one_hot);
        $monitor("Time = %0dt | rst_n = %b | Input(x) = %b | Current State = %5b | Output(y) = %b", 
                 $time, rst_n, x, uut.current_state, y);
    end

    initial begin
        clk = 0;
        rst_n = 0;
        x = 0;
        
        #15 rst_n = 1;
        
        @(posedge clk); x = 1; // Transitions to S1
        @(posedge clk); x = 0; // Transitions to S2
        @(posedge clk); x = 1; // Transitions to S3
        @(posedge clk); x = 1; // Transitions to S4 -> y goes High (First Detection)
        
        @(posedge clk); x = 0; // Overlap check: Transitions to S2
        @(posedge clk); x = 1; // Transitions to S3
        @(posedge clk); x = 1; // Transitions to S4 -> y goes High (Second Detection)
        
        @(posedge clk); x = 0; // Transitions to S2
        
        @(posedge clk);
        #10;
        $finish;
    end

endmodule
```


### simulation results

<img width="1904" height="257" alt="image" src="https://github.com/user-attachments/assets/e73ba0de-b980-443c-97e2-117e529450f0" />

## Formally verify equivalence between the Mealy and Moore implementations.

### The Core Functional Difference

| Feature | Mealy Machine | Moore Machine |
|:--------|:--------------|:--------------|
| **Output Function** | Output depends on the **Current State** and **Current Input** (`Y = f(S, X)`). | Output depends **only on the Current State** (`Y = f(S)`). |
| **Output Timing** | Output transitions to **1 immediately** when the final input bit (`1`) arrives. | Output goes high **one clock cycle later**, after entering the detection state. |
| **States Required** | 4 States | 5 States |

Because of this structural difference, the cycle-by-cycle equivalence is:

```text
Y_Moore(t) = Y_Mealy(t − 1)
```

---

### State Mapping & Bisimulation Relation

| History Tracked | Moore State (S) | Mealy State (M) | Equivalence Condition |
|:---------------:|:---------------:|:---------------:|:---------------------:|
| Reset / ∅ | S0 | M0 | S0 ≡ M0 |
| 1 | S1 | M1 | S1 ≡ M1 |
| 10 | S2 | M2 | S2 ≡ M2 |
| 101 | S3 | M3 | S3 ≡ M3 |
| 1011 | S4 (Transient) | — | Maps to the registered output of **M3** when **X = 1** |

---

### S4 Transition Proof

#### Mealy Machine

| Current State | Input (X) | Next State | Output (Y) |
|:-------------:|:---------:|:----------:|:----------:|
| M3 (101) | 1 | M1 | 1 |

---

#### Moore Machine

| Current State | Input (X) | Next State | Output (Y) |
|:-------------:|:---------:|:----------:|:----------:|
| S3 (101) | 1 | S4 | 0 |
| S4 | — | — | 1 |

From **S4**:

| Next Input (X) | Next State | Explanation |
|:--------------:|:----------:|:------------|
| 0 | S2 (Overlapping) | Continues matching the overlapping pattern. |
| 1 | S1 | Starts a new sequence with the received `1`. |

This matches the behavior of the transitions originating from **Mealy state M1** in the subsequent clock cycle, thereby proving functional equivalence with a **one-clock-cycle output delay**.

### Code
**Design Code**
```verilog
module seq_detector_1011_mealy (
    input  wire clk,
    input  wire rst_n,
    input  wire x,
    output reg  y
);

    localparam [1:0] M0 = 2'b00, // Got nothing
                     M1 = 2'b01, // Got '1'
                     M2 = 2'b10, // Got '10'
                     M3 = 2'b11; // Got '101'

    reg [1:0] current_state, next_state;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) current_state <= M0;
        else        current_state <= next_state;
    end

    always @(*) begin
        y = 1'b0;
        next_state = current_state;

        case (current_state)
            M0: begin
                if (x) next_state = M1;
                else   next_state = M0;
            end
            M1: begin
                if (x) next_state = M1;
                else   next_state = M2;
            end
            M2: begin
                if (x) next_state = M3;
                else   next_state = M0;
            end
            M3: begin
                if (x) begin
                    next_state = M1;
                    y = 1'b1; // Output goes high immediately when x=1 in M3
                end else begin
                    next_state = M2;
                    y = 1'b0;
                end
            end
            default: next_state = M0;
        endcase
    end

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns/1ps
module formal_equivalence_miter_tb;

    reg clk;
    reg rst_n;
    reg x;

    wire y_moore;
    wire y_mealy;
    
    reg y_mealy_delayed;

    seq_detector_1011_moore moore_inst (
        .clk(clk),
        .rst_n(rst_n),
        .x(x),
        .y(y_moore)
    );

    seq_detector_1011_mealy mealy_inst (
        .clk(clk),
        .rst_n(rst_n),
        .x(x),
        .y(y_mealy)
    );

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            y_mealy_delayed <= 1'b0;
        end else begin
            y_mealy_delayed <= y_mealy;
        end
    end

    always @(posedge clk) begin
        if (rst_n) begin // Only check after coming out of reset
            if (y_moore !== y_mealy_delayed) begin
                $display("FORMAL ERROR: Equivalence Violated at time %0t!", $time);
                $display("Moore Output = %b | Mealy Output (Delayed) = %b", y_moore, y_mealy_delayed);
                $finish; // Terminate simulation if designs diverge
            end
        end
    end

    always #5 clk = ~clk;

    initial begin
        $dumpfile("formal_equivalence_miter_tb.vcd");
        $dumpvars(0, formal_equivalence_miter_tb);
        clk = 0;
        rst_n = 0;
        x = 0;
        
        #15 rst_n = 1; // Release reset

        @(posedge clk); x = 1;
        @(posedge clk); x = 0;
        @(posedge clk); x = 1;
        @(posedge clk); x = 1; // Sequence 1 detected
        
        @(posedge clk); x = 0;
        @(posedge clk); x = 1;
        @(posedge clk); x = 1; // Sequence 2 detected (via overlap)
        
        @(posedge clk); x = 0;
        @(posedge clk); x = 0; // Break sequence tracking entirely
        
        @(posedge clk); x = 1;
        @(posedge clk); x = 1; // Distort sequence preamble
        
        repeat (5) @(posedge clk);
        
        $display("SUCCESS: Moore and Mealy implementations are formally equivalent!");
        $finish;
    end

endmodule
```

### Simulation results

<img width="1902" height="307" alt="image" src="https://github.com/user-attachments/assets/e2c5d4d7-598d-4775-83ad-c61db85fe9bc" />
