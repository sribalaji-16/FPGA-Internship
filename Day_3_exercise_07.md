# Exercise 07 — 4:1 Multiplexer
##  Implement 4:1 MUX using three 2:1 MUXes
### Truthtable 
| S1 | S0 | Selected Input | Output |
|:--:|:--:|:--------------:|:------:|
| 0 | 0 | I0 | I0 |
| 0 | 1 | I1 | I1 |
| 1 | 0 | I2 | I2 |
| 1 | 1 | I3 | I3 |

### Code 
**Design Code**
```verilog
module mux2to1 (
    input wire a,
    input wire b,
    input wire sel,
    output wire y
);
    assign y = sel ? b : a;
endmodule

module mux4to1 (
    input wire [3:0] i,
    input wire [1:0] sel,
    output wire y
);
    wire w[1:0];
    mux2to1 mux0 (.a(i[0]),.b(i[1]),.sel(sel[0]),.y(w[0]));
    mux2to1 mux1 (.a(i[2]),.b(i[3]),.sel(sel[0]),.y(w[1]));
    mux2to1 mux2 (.a(w[0]),.b(w[1]),.sel(sel[1]),.y(y));

endmodule
```

**Testbench Code**
```verilog
module tb_mux4to1_using_mux2to1;
    reg [3:0] in; 
    reg [1:0] sel; 
    wire out; 
    
    mux4to1 uut (.i(in),.sel(sel),.y(out));
    
    initial begin
        $dumpfile("mux4to1_using_mux2to1.vcd");
        $dumpvars(0, tb_mux4to1_using_mux2to1); 

        // Simplified monitor format for cleaner console reading
        $monitor("Time: %0t | in(3210): %b | sel: %b | out: %b", $time, in, sel, out);

        // --- TEST CASE 1: Select Channel 0 (sel = 00) ---
        sel = 2'b00;
        in  = 4'b0000; #5;  // Everything low -> out should be 0
        in  = 4'b0001; #5;  // in[0] goes high -> out should follow to 1
        in  = 4'b1111; #5;  // All other inputs go high -> out stays 1
        in  = 4'b1110; #5;  // in[0] drops low -> out should immediately drop to 0
        
        // --- TEST CASE 2: Select Channel 1 (sel = 01) ---
        sel = 2'b01;
        in  = 4'b0000; #5;  
        in  = 4'b0010; #5;  // in[1] goes high -> out should follow to 1
        in  = 4'b1101; #5;  // Toggle other channels -> out should ignore them and stay 0
        
        // --- TEST CASE 3: Select Channel 2 (sel = 10) ---
        sel = 2'b10;
        in  = 4'b0000; #5;  
        in  = 4'b0100; #5;  // in[2] goes high -> out should follow to 1
        in  = 4'b1011; #5;  // Toggle other channels -> out should ignore them and stay 0

        // --- TEST CASE 4: Select Channel 3 (sel = 11) ---
        sel = 2'b11;
        in  = 4'b0000; #5;  
        in  = 4'b1000; #5;  // in[3] goes high -> out should follow to 1
        in  = 4'b0111; #5;  // Toggle other channels -> out should ignore them and stay 0

        #10;
        $finish; 
    end
endmodule
```

### Simulation results
<img width="971" height="230" alt="image" src="https://github.com/user-attachments/assets/22e7313c-4597-4f8c-97e1-732fadc10185" />


## Build an 8:1 MUX from two 4:1 MUXes and one 2:1 MUX.

### Truth table 

| S2 | S1 | S0 | Output (Y) |
|:--:|:--:|:--:|:----------:|
| 0 | 0 | 0 | I0 |
| 0 | 0 | 1 | I1 |
| 0 | 1 | 0 | I2 |
| 0 | 1 | 1 | I3 |
| 1 | 0 | 0 | I4 |
| 1 | 0 | 1 | I5 |
| 1 | 1 | 0 | I6 |
| 1 | 1 | 1 | I7 |

### Code
**Design Code**
```verilog
module mux4to1 (
    input wire [3:0] data_in, // 4-bit input data
    input wire [1:0] sel,     // 2-bit select signal
    output wire data_out       // Output data
);
    assign data_out = (sel == 2'b00) ? data_in[0] :
                      (sel == 2'b01) ? data_in[1] :
                      (sel == 2'b10) ? data_in[2] :
                      (sel == 2'b11) ? data_in[3] : 1'b0; // Default case
endmodule

module mux2to1 (
    input wire [1:0] data_in, // 2-bit input data
    input wire sel,           // 1-bit select signal
    output wire data_out       // Output data
);
    assign data_out = (sel == 1'b0) ? data_in[0] : data_in[1]; // Select between the two inputs
endmodule

module mux8to1 (
    input wire [7:0] data_in, // 8-bit input data
    input wire [2:0] sel,     // 3-bit select signal
    output wire data_out       // Output data
);
    wire mux1_out, mux2_out;

    mux4to1 mux1 (
        .data_in(data_in[3:0]), // First 4 bits of input data
        .sel(sel[1:0]),         // Lower 2 bits of select signal
        .data_out(mux1_out)     // Output from first mux
    );

    mux4to1 mux2 (
        .data_in(data_in[7:4]), // Last 4 bits of input data
        .sel(sel[1:0]),         // Lower 2 bits of select signal
        .data_out(mux2_out)     // Output from second mux
    );

    mux2to1 mux3 (
        .data_in({mux2_out, mux1_out}), // Combine outputs from previous muxes
        .sel(sel[2]),           // High bit of select signal
        .data_out(data_out)     // Final output
    );

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module testbench;
    reg [7:0] data_in; // 8-bit input data
    reg [2:0] sel;     // 3-bit select signal
    wire data_out;     // Output data

    mux8to1 uut (
        .data_in(data_in),
        .sel(sel),
        .data_out(data_out)
    );

    initial begin

        $dumpfile("mux8to1_testbench.vcd"); // Create a VCD file for waveform analysis
        $dumpvars(0, testbench); // Dump all variables in the testbench

        $monitor("Time: %0t | data_in: %b | sel: %b | data_out: %b", $time, data_in, sel, data_out);
        // Test case 1
        data_in = 8'b11111010; // Input data
        sel = 3'b000;          // Select first input (data_in[0])
        #10;                   // Wait for 10 time units

        // Test case 2
        sel = 3'b001;          // Select second input (data_in[1])
        #10;

        // Test case 3
        sel = 3'b010;          // Select third input (data_in[2])
        #10;

        // Test case 4
        sel = 3'b011;          // Select fourth input (data_in[3])
        #10;

        // Test case 5
        sel = 3'b100;          // Select fifth input (data_in[4])
        #10;

        // Test case 6
        sel = 3'b101;          // Select sixth input (data_in[5])
        #10;

        // Test case 7
        sel = 3'b110;          // Select seventh input (data_in[6])
        #10;

        // Test case 8
        sel = 3'b111;          // Select eighth input (data_in[7])
        #10;

        $finish;              // End simulation
    end
endmodule
```

### Simulation results
<img width="1157" height="238" alt="image" src="https://github.com/user-attachments/assets/5f9c1e46-fbaf-48bf-8748-3542c3ea7fd1" />

## Use a 4:1 MUX to implement Y = A'B + AB' (XOR)

### Truthtable
| S1 | S0 | Output (Y) |
|:--:|:--:|:----------:|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |
### Code
**Design Code**
```verilog
module xor_using_mux4to1 (
    input wire [1:0] sel,     // 2-bit select signal
    output reg data_out       // Output data
);
    always @(*) begin
        data_out = (sel == 2'b00) ? 1'b0 :
                      (sel == 2'b01) ? 1'b1 :
                      (sel == 2'b10) ? 1'b1 : 1'b0;
    end
endmodule
```

**Testbench Code**
```verilog
module test_xor_using_mux4to1;
    reg [1:0] sel;     // 2-bit select signal
    wire data_out;     // Output data

    // Instantiate the xor_using_mux4to1 module
    xor_using_mux4to1 uut (
        .sel(sel),
        .data_out(data_out)
    );

    initial begin

        $dumpfile("xor_using_mux4to1.vcd"); // Create a VCD file for waveform viewing
        $dumpvars(0, test_xor_using_mux4to1); // Dump all variables in the testbench

        // Test case 1: Select data_in[0]
        sel = 2'b00;       // Select data_in[0]
        #10;              // Wait for 10 time units
        $display("Select: %b, Output: %b", sel, data_out);

        // Test case 2: Select data_in[1]
        sel = 2'b01;       // Select data_in[1]
        #10;              // Wait for 10 time units
        $display("Select: %b, Output: %b", sel, data_out);

        // Test case 3: Select data_in[2]
        sel = 2'b10;       // Select data_in[2]
        #10;              // Wait for 10 time units
        $display("Select: %b, Output: %b", sel, data_out);

        // Test case 4: Select data_in[3]
        sel = 2'b11;       // Select data_in[3]
        #10;              // Wait for 10 time units
        $display("Select: %b, Output: %b", sel, data_out);

        $finish;          // End the simulation
    end
endmodule   
```

### Simulation results
<img width="1010" height="202" alt="image" src="https://github.com/user-attachments/assets/c9a2b856-0251-45b2-9115-64ddd5d0b90a" />
