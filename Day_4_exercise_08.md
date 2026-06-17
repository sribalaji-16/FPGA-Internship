# Exercise 08 — 1:4 Demultiplexer
##  Add an active-low enable input to the DEMUX

### Truthtable

| Enable | Select (S) | Input (I) | Out[0] | Out[1] |
|:------:|:----------:|:---------:|:------:|:------:|
|   0    |     0      |     1     |    1   |    0   |
|   0    |     1      |     1     |    0   |    1   |
|   1    |     0      |     1     |    0   |    0   |
|   1    |     1      |     1     |    0   |    0   |
|   0    |     0      |     0     |    0   |    0   |
|   0    |     1      |     0     |    0   |    0   |
|   1    |     0      |     0     |    0   |    0   |
|   1    |     1      |     0     |    0   |    0   |

### Code
**Design Code**
```verilog
module demux_1to2 (
    input wire sel,
    input wire enable,
    input wire in,
    output wire [1:0] out
);
    assign out[0] = ~enable & ~sel & in;
    assign out[1] = ~enable & sel & in;
endmodule
```

**Testbench Code**
```verilog
module tb_demux_1to2;
    reg sel;
    reg enable;
    reg in;
    wire [1:0] out;

    demux_1to2 uut (
        .sel(sel),
        .enable(enable),
        .in(in),
        .out(out)
    );

    initial begin
        $dumpfile("demux_1to2_tb.vcd");
        $dumpvars(0, tb_demux_1to2);

        $monitor("Time: %0t | enable: %b | sel: %b | in: %b | out[0]: %b | out[1]: %b", $time, enable, sel, in, out[0], out[1]);    

        // Test case 1: enable = 0, sel = 0, in = 1
        enable = 0; sel = 0; in = 1;
        #10;

        // Test case 2: enable = 0, sel = 1, in = 1
        enable = 0; sel = 1; in = 1;
        #10;

        // Test case 3: enable = 1, sel = 0, in = 1
        enable = 1; sel = 0; in = 1;
        #10;

        // Test case 4: enable = 1, sel = 1, in = 1
        enable = 1; sel = 1; in = 1;
        #10;

        // Test case 5: enable = 0, sel = 0, in = 0
        enable = 0; sel = 0; in = 0;
        #10;

        // Test case 6: enable = 0, sel = 1, in = 0
        enable = 0; sel = 1; in = 0;
        #10;

        // Test case 7: enable = 1, sel = 0, in = 0
        enable = 1; sel = 0; in = 0;
        #10;

        // Test case 8: enable = 1, sel = 1, in = 0
        enable = 1; sel = 1; in = 0;
        #10;
    end
endmodule
```

### Simulation results

<img width="985" height="254" alt="image" src="https://github.com/user-attachments/assets/ead85b79-77f9-4c71-9e73-d6f56911bd35" />


## Build a 1:8 DEMUX using two 1:4 DEMUXes.
### Truthtable

| S2 | S1 | S0 | Input (I) | Y0 | Y1 | Y2 | Y3 | Y4 | Y5 | Y6 | Y7 |
|:--:|:--:|:--:|:---------:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 0 | 0 | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| 1 | 1 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| 1 | 1 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 |

### Code
**Design Code**
```verilog
module demux_1to4 (
    input wire [1:0] sel,
    input wire en,
    input wire in,
    output reg [3:0] out
);
    always @(*) begin
        out[0] = ~en & ~sel[1] & ~sel[0] & in;
        out[1] = ~en & ~sel[1] & sel[0] & in;
        out[2] = ~en & sel[1] & ~sel[0] & in;
        out[3] = ~en & sel[1] & sel[0] & in;
    end
endmodule

module demux_1to8 (
    input wire [2:0] sel,
    input wire in,
    output wire [7:0] out
);
    demux_1to4 demux_low (
        .sel(sel[1:0]),
        .en(sel[2]),
        .in(in),
        .out(out[3:0])
    );
    demux_1to4 demux_high (
        .sel(sel[1:0]),
        .en(~sel[2]),
        .in(in),
        .out(out[7:4])
    );
endmodule
```
**Testbench Code**
```verilog
module tb_demux_1to8;
    reg [2:0] sel;
    reg in;
    wire [7:0] out;

    integer i,j;
    
    demux_1to8 uut (.sel(sel),.in(in),.out(out));

    initial begin
        $dumpfile("demux_1to8.vcd");    
        $dumpvars(0, tb_demux_1to8);

        $monitor("Time: %0t | sel: %b | in: %b | out: %b", $time, sel, in, out);
        // Test all combinations of sel and in
        for (i = 0; i < 8; i = i + 1) begin
            for (j = 0; j < 2; j = j + 1) begin
                sel =i;
                in = j;
                #10; // Wait for 10 time units
            end
        end
        $finish;
    end
endmodule
```

### Simulation results

<img width="1003" height="275" alt="image" src="https://github.com/user-attachments/assets/8f056bf2-dc1b-4f80-9b17-5448037074c3" />


## Use a DEMUX as a function generator for a truth table.
### Function expression

Function Generator $$F = Y_1 + Y_3 + Y_6 + Y_7$$

### Truthtable

| A | B | C | F |
|:-:|:-:|:-:|:-:|
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 |
| 0 | 1 | 0 | 0 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 |

### Code
**Design Code**
```verilog
module demux_function_generator (
    input wire A, B, C,
    output wire F
);
    wire [7:0] y; // Wires to capture DEMUX minterm outputs

    // Instantiate a standard 1:8 DEMUX
    // Pass {A, B, C} as the select lines and tie 'in' to 1'b1
    demux_1to8 master_demux (
        .sel({A, B, C}),
        .in(1'b1),
        .out(y)
    );

    // Function F = Sum of Minterms (1, 3, 6, 7)
    assign F = y[1] | y[3] | y[6] | y[7];

endmodule

module demux_1to8 (
    input wire [2:0] sel,
    input wire in,
    output wire [7:0] out
);
    // Shift the 1-bit input 'in' left by 'sel' positions
    assign out = (in << sel);
    // Such an amazing implementation of a demux, umaaahhhh :)
endmodule
```

**Testbench Code**
```verilog
module tb_demux_function_generator;
    reg A, B, C;
    wire F;

    // Instantiate the demux_function_generator
    demux_function_generator uut (
        .A(A),
        .B(B),
        .C(C),
        .F(F)
    );

    initial begin

        $dumpfile("demux_function_generator.vcd");
        $dumpvars(0, tb_demux_function_generator);

        // Test all combinations of A, B, C
        $display("A B C | F");
        $display("-------------");
        for (integer i = 0; i < 8; i = i + 1) begin
            {A, B, C} = i; // Assign values to A, B, C
            #10; // Wait for 10 time units
            $display("%b %b %b | %b", A, B, C, F);
        end
        $finish;
    end
endmodule
```

### Simulation results

<img width="972" height="256" alt="image" src="https://github.com/user-attachments/assets/e8ca13e7-7de0-46b7-86f3-1215e8c21e6c" />
