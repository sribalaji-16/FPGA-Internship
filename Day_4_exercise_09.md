# Exercise 09 — Encoder
##  Implement a 4:2 encoder
### Truthtable

| D3 | D2 | D1 | D0 | Y1 | Y0 |
|:--:|:--:|:--:|:--:|:--:|:--:|
| 0 | 0 | 0 | 1 | 0 | 0 |
| 0 | 0 | 1 | 0 | 0 | 1 |
| 0 | 1 | 0 | 0 | 1 | 0 |
| 1 | 0 | 0 | 0 | 1 | 1 |

### Code
**Design Code**
```verilog
module encoder_4to2_dataflow (
    input [3:0] D,   // 4-bit input vector
    output [1:0] Y   // 2-bit output vector Y = {Y1, Y0}
);

    // Continuous assignment using bitwise OR operators
    assign Y[1] = D[2] | D[3];
    assign Y[0] = D[1] | D[3];

endmodule
```

**Testbench Code**
```verilog
module tb_encoder_4to2;

    reg [3:0] D;
    wire [1:0] Y;

    encoder_4to2_dataflow uut (.D(D),.Y(Y));

    initial begin
        $dumpfile("encoder_4to2.vcd");
        $dumpvars(0, tb_encoder_4to2);
        // Monitor outputs in the console
        $monitor("Time=%0t | Input D=%b | Output Y=%b", $time, D, Y);
        
        // Stimulus mapping the truth table
        D = 4'b0001; #10;
        D = 4'b0010; #10;
        D = 4'b0100; #10;
        D = 4'b1000; #10;
        
        $finish;
    end

endmodule
```

### Simulation results

<img width="992" height="200" alt="image" src="https://github.com/user-attachments/assets/f62ede70-e26e-4874-b97d-e1811af0ffba" />


## Add an enable input to control encoding operation.
### Truthtable

This result obtained is given as truthtable.

| EN | D3 | D2 | D1 | D0 | Y1 | Y0 |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 0 | 1 | 0 | 0 | 0 |
| 1 | 0 | 0 | 0 | 1 | 0 | 0 |
| 1 | 0 | 0 | 1 | 0 | 0 | 1 |
| 1 | 0 | 1 | 0 | 0 | 1 | 0 |
| 1 | 1 | 0 | 0 | 0 | 1 | 1 |
| 0 | 0 | 1 | 0 | 0 | 0 | 0 |

### Code
**Design Code**
```verilog
module encoder_4to2_en(
    input [3:0] D,
    input EN,
    output [1:0] Y
);

    assign Y[1] = EN & (D[2] | D[3]);
    assign Y[0] = EN & (D[1] | D[3]);

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module tb_encoder_4to2_en;

    reg [3:0] D;
    reg EN;

    wire [1:0] Y;

    // Instantiate Unit Under Test (UUT)
    encoder_4to2_en uut (.D(D),.EN(EN),.Y(Y));

    initial begin

        $dumpfile("encoder_4to2_en.vcd");
        $dumpvars(0, tb_encoder_4to2_en);

        $monitor("Time=%0t | EN=%b | D=%b | Y=%b", $time, EN, D, Y);
        
        // --- Test Case 1: Encoder Disabled ---
        EN = 0; D = 4'b1000; #10;
        D = 4'b0010; #10;
        
        // --- Test Case 2: Encoder Enabled ---
        EN = 1;
        D = 4'b0001; #10;
        D = 4'b0010; #10;
        D = 4'b0100; #10;
        D = 4'b1000; #10;
        
        // --- Test Case 3: Turn off again ---
        EN = 0; D = 4'b0100; #10;

        $finish;
    end
endmodule
```
### Simulation results

<img width="1137" height="227" alt="image" src="https://github.com/user-attachments/assets/9927be08-5f85-49dc-8fd2-85b148671c85" />


##  Write a testbench applying all valid one-hot input combinations.
### Truthtable

| EN | D3 | D2 | D1 | D0 | Y1 | Y0 |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| 1 | 1 | 0 | 0 | 0 | 1 | 1 |
| 1 | 0 | 0 | 0 | 1 | 0 | 0 |
| 1 | 0 | 0 | 1 | 0 | 0 | 1 |
| 1 | 0 | 1 | 0 | 0 | 1 | 0 |
| 1 | 1 | 0 | 0 | 0 | 1 | 1 |
| 0 | 0 | 1 | 0 | 0 | 0 | 0 |

### Code
**Testbench Code**
```verilog
`timescale 1ns / 1ps

module tb_encoder_one_hot;

    reg [3:0] D;
    reg EN;

    wire [1:0] Y;

    // Instantiate the 4:2 Dataflow Encoder with Enable inline
    // (This maps directly to our logic expressions)
    assign Y[1] = EN & (D[2] | D[3]);
    assign Y[0] = EN & (D[1] | D[3]);

    initial begin
        $dumpfile("tb_encoder_one_hot.vcd");
        $dumpvars(0, tb_encoder_one_hot);
        // Display a formatted header in the simulation console
        $display("\n--- Starting 4:2 Encoder One-Hot Test ---");
        $monitor("Time = %0t ns | EN = %b | Input D = %b | Output Binary Y = %b", $time, EN, D, Y);
        
        // -------------------------------------------------------------
        // STEP 1: Verify Disabled State (Outputs must stay 00)
        // -------------------------------------------------------------
        EN = 0; 
        D = 4'b0001; #10; // Apply one-hot D0 while disabled
        D = 4'b1000; #10; // Apply one-hot D3 while disabled
        
        // -------------------------------------------------------------
        // STEP 2: Enable System and Loop Through All Valid One-Hot Inputs
        // -------------------------------------------------------------
        EN = 1; #5;
        
        D = 4'b0001; #10; // 1st One-Hot: D0 active -> Expected Y = 00
        D = 4'b0010; #10; // 2nd One-Hot: D1 active -> Expected Y = 01
        D = 4'b0100; #10; // 3rd One-Hot: D2 active -> Expected Y = 10
        D = 4'b1000; #10; // 4th One-Hot: D3 active -> Expected Y = 11

        // -------------------------------------------------------------
        // STEP 3: Return to Disabled State
        // -------------------------------------------------------------
        EN = 0;
        D = 4'b0100; #10;
        
        $display("--- Test Complete --- \n");
        $finish; // End simulation
    end

endmodule
```

### Simulation results

<img width="1135" height="236" alt="image" src="https://github.com/user-attachments/assets/51fbe141-0635-4a49-b123-abc3faf02f1e" />
