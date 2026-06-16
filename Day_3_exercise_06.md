# Exercise 06 — 2:1 MultiplexerExercise 06 — 2:1 Multiplexer
##  Implement a 2:1 MUX using only NAND gates.

### Truth Table
| Select (S) | Input A | Input B | Output (Y) |
|:----------:|:-------:|:-------:|:----------:|
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 1 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 1 |
| 1 | 1 | 0 | 0 |
| 1 | 1 | 1 | 1 |

### Code 
**Design Code**
```verilog
module mux_using_nand (a, b, sel, y);
  input a, b, sel;
  output y;
  
  wire w[2:0];

  nand(w[0], sel, sel);
  nand(w[1], a, w[0]);
  nand(w[2], b, sel);
  nand(y, w[1], w[2]);
endmodule
```

**Testbench Code**
```verilog
module tb_mux_using_nand;

reg a, b, sel;
wire y;
mux_using_nand uut (
    .a(a),
    .b(b),
    .sel(sel),
    .y(y)
);

initial begin 
    $dumpfile("tb_mux_using_nand.vcd");
    $dumpvars(0, tb_mux_using_nand);

    $monitor("a=%b b=%b sel=%b y=%b", a, b, sel, y);

    // Test case 1: sel = 0, y should be a
    a = 0; b = 0; sel = 0; #10;

    // Test case 2: sel = 0, y should be a
    a = 1; b = 0; sel = 0; #10;

    // Test case 3: sel = 0, y should be a
    a = 0; b = 1; sel = 0; #10;

    // Test case 4: sel = 0, y should be a
    a = 1; b = 1; sel = 0; #10;

    // Test case 5: sel = 1, y should be b
    a = 0; b = 0; sel = 1; #10;

    // Test case 6: sel = 1, y should be b
    a = 1; b = 0; sel = 1; #10; 

    // Test case 7: sel = 1, y should be b
    a = 0; b = 1; sel = 1; #10;

    // Test case 8: sel = 1, y should be b
    a = 1; b = 1; sel = 1; #10;
    $finish;
end 
endmodule
```

### Simulation results

<img width="969" height="252" alt="image" src="https://github.com/user-attachments/assets/0e5f3ef1-7051-47e0-ade7-b4d0ed602c38" />


## Use 2:1 MUXes to build a 4:1 MUX (3 MUXes needed).
### Truth table
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


## Implement any 3-variable Boolean function using MUXes

### Truthtable

| Decimal Minterm | Input A (S2) | Input B (S1) | Input C (S0) | Count of 1s | Output (F) | MUX Input State |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **m0** | 0 | 0 | 0 | 0 | **0** | I0 = 0 |
| **m1** | 0 | 0 | 1 | 1 | **0** | I1 = 0 |
| **m2** | 0 | 1 | 0 | 1 | **0** | I2 = 0 |
| **m3** | 0 | 1 | 1 | 2 | **1** | I3 = 1 |
| **m4** | 1 | 0 | 0 | 1 | **0** | I4 = 0 |
| **m5** | 1 | 0 | 1 | 2 | **1** | I5 = 1 |
| **m6** | 1 | 1 | 0 | 2 | **1** | I6 = 1 |
| **m7** | 1 | 1 | 1 | 3 | **1** | I7 = 1 |

### Code
**Design Code**
```verilog
module mux8to1 (
    input wire [7:0] i,      // 8 data inputs
    input wire [2:0] sel,    // 3 select lines
    output wire y            // 1 output
);
    // Behavioral selection based on the 3-bit pointer
    assign y = i[sel];
endmodule


// --- Top-Level Module Implementing F = m(3,5,6,7) ---
module boolean_3var_fn (
    input wire a, b, c,      // Our 3 Boolean variables
    output wire f            // Function output
);

    // Internal wire vector to hold our hardcoded 1s and 0s
    wire [7:0] mux_inputs;

    // Index:      7  6  5  4  3  2  1  0
    assign mux_inputs = 8'b1_1_1_0_1_0_0_0; 

    // Instantiate the 8-to-1 MUX
    // Concatenate {a, b, c} to form the 3-bit select bus
    mux8to1 my_mux (
        .i(mux_inputs),
        .sel({a, b, c}),
        .y(f)
    );

endmodule
```
**Testbench Code**
```verilog
`timescale 1ns / 1ps

module tb_boolean_3var_fn;

    reg test_a;
    reg test_b;
    reg test_c;

    wire test_f;

    wire is_majority;
    assign is_majority = (test_f == 1'b1);

    boolean_3var_fn uut (.a(test_a),.b(test_b),.c(test_c),.f(test_f));

    initial begin

        $dumpfile("boolean_3var_fn_results.vcd"); 
        $dumpvars(0, tb_boolean_3var_fn);         

        // Print header layout to the terminal console
        $display("---------------------------------------");
        $display("Time | A  B  C | Output (F) | Evaluation");
        $display("---------------------------------------");
        
        // Monitor block automatically prints whenever any signal changes state
        //$monitor("%40t | %b  %b  %b |     %b      | %s", 
         //        $time, test_a, test_b, test_c, test_f,
          //       (test_f == 1'b1) ? "MATCH (>= 2 ones)" : "LOW (< 2 ones)");
        // this monitor doesn't work because the ternary operator is not synthesizable :( , so we will use a wire to evaluate the majority condition. 
        // Better try doing it with vivado simulator later, to know whether it work there?


        $monitor("%40t | %b  %b  %b |     %b      |      %b", 
                 $time, test_a, test_b, test_c, test_f, is_majority);
        // --- Appling all 8 binary combinations sequentially ---
        
        test_a = 0; test_b = 0; test_c = 0; #10; // Row 0: 000 -> Expect F=0
        test_a = 0; test_b = 0; test_c = 1; #10; // Row 1: 001 -> Expect F=0
        test_a = 0; test_b = 1; test_c = 0; #10; // Row 2: 010 -> Expect F=0
        test_a = 0; test_b = 1; test_c = 1; #10; // Row 3: 011 -> Expect F=1 *Majority*
        
        test_a = 1; test_b = 0; test_c = 0; #10; // Row 4: 100 -> Expect F=0
        test_a = 1; test_b = 0; test_c = 1; #10; // Row 5: 101 -> Expect F=1 *Majority*
        test_a = 1; test_b = 1; test_c = 0; #10; // Row 6: 110 -> Expect F=1 *Majority*
        test_a = 1; test_b = 1; test_c = 1; #10; // Row 7: 111 -> Expect F=1 *Majority*

        // End the simulation execution trace
        $display("---------------------------------------");
        $finish;
    end

endmodule
```
### Simulation Code
<img width="1154" height="277" alt="image" src="https://github.com/user-attachments/assets/d575a23a-1591-404f-9c67-e92757474443" />
