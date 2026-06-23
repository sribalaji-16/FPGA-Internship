# Exercise 17 — 4-bit Shift Register
## Modify to support left-shift and right-shift with a direction control.

### Truthtable

| rst | dir | sin | q    |
|:---:|:---:|:---:|:----:|
| 1 | 0 | 1 | 0000 |
| 0 | 0 | 1 | 0000 |
| 0 | 0 | 1 | 1000 |
| 0 | 0 | 1 | 1100 |
| 0 | 0 | 1 | 1110 |
| 0 | 0 | 1 | 1111 |
| 0 | 1 | 0 | 1111 |
| 0 | 1 | 0 | 1110 |
| 0 | 1 | 0 | 1100 |
| 0 | 1 | 0 | 1000 |
| 0 | 1 | 0 | 0000 |

### Code
**Design Code**
```verilog
module bidir_shift_reg #(parameter W = 4) (
    input clk, rst, dir, sin,
    output reg [W-1:0] q
);
    always @(posedge clk or posedge rst) begin
        if (rst) q <= 0;
        else     q <= dir ? {q[W-2:0], sin} : {sin, q[W-1:1]};
    end
endmodule
```
**Testbench Code**
```verilog
module tb;
    reg clk=0, rst=1, dir=0, sin=1;
    wire [3:0] q;

    bidir_shift_reg #(4) uut (clk, rst, dir, sin, q);
    
    always #5 clk = ~clk;

    initial begin
        $dumpfile("tb.vcd");
        $dumpvars(0, tb);
        $monitor("Time=%0t | rst=%b | dir=%b | sin=%b | q=%b", $time, rst, dir, sin, q);
        #12 rst = 0;             // Release reset, starts shifting Right (dir=0)
        #40 dir = 1; sin = 0;    // Switch to shifting Left (dir=1) with new serial input
        #40 $finish;
    end
endmodule
```


### Simulation results

<img width="970" height="270" alt="image" src="https://github.com/user-attachments/assets/289d3189-94e9-4660-8d9d-63c4191d218e" />


## Implement a 4-bit PISO shift register.
### Truthtable

| rst | load | p_in | q    | sout |
|:---:|:----:|:----:|:----:|:----:|
| 1 | 0 | 0000 | 0000 | 0 |
| 0 | 1 | 1011 | 0000 | 0 |
| 0 | 1 | 1011 | 1011 | 1 |
| 0 | 0 | 1011 | 1011 | 1 |
| 0 | 0 | 1011 | 0110 | 0 |
| 0 | 0 | 1011 | 1100 | 1 |
| 0 | 0 | 1011 | 1000 | 1 |
| 0 | 0 | 1011 | 0000 | 0 |

### Code
**Design Code**
```verilog
module piso_4bit (
    input clk, rst, load,      // load=1 for Parallel Load, load=0 for Shift
    input [3:0] p_in,
    output sout
);
    reg [3:0] q;
    assign sout = q[3];        // MSB acts as the serial output pin

    always @(posedge clk or posedge rst) begin
        if (rst)     q <= 4'b0000;
        else if (load) q <= p_in;
        else         q <= {q[2:0], 1'b0}; // Shift left/up towards MSB, filling with 0
    end
endmodule
```

**Testbench Code**
```verilog
module tb_piso;
    reg clk=0, rst=1, load=0;
    reg [3:0] p_in = 4'b0;
    wire sout;

    piso_4bit uut (clk, rst, load, p_in, sout);
    always #5 clk = ~clk;

    initial begin
        $dumpfile("piso_4bit.vcd");
        $dumpvars(0, tb_piso);
        $monitor("Time=%0t | rst=%b | load=%b | p_in=%b | Internal Register(q)=%b | sout=%b", $time, rst, load, p_in, uut.q, sout);
        #12 rst = 0; p_in = 4'b1011; load = 1; // Load data 4'b1011
        #10 load = 0;                         // Shift mode for next 4 cycles
        #40 $finish;
    end
endmodule
```


### Simulation results

<img width="991" height="305" alt="image" src="https://github.com/user-attachments/assets/5afe9439-bb63-47a9-b4e4-9e653a6082fb" />


## Use the shift register to implement a 4-bit LFSR with feedback.
### Truthtable 

| rst_n | LFSR State (q) |
|:-----:|:--------------:|
| 0 | 0001 |
| 1 | 0001 |
| 1 | 0010 |
| 1 | 0100 |
| 1 | 1001 |
| 1 | 0011 |
| 1 | 0110 |
| 1 | 1101 |
| 1 | 1010 |
| 1 | 0101 |
| 1 | 1011 |
| 1 | 0111 |
| 1 | 1111 |
| 1 | 1110 |
| 1 | 1100 |
| 1 | 1000 |
| 1 | 0001 |
| 1 | 0010 |

### Code
**Design Code**
```verilog
module lfsr_4bit (
    input clk, rst_n, // Active-low reset
    output reg [3:0] q
);
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) q <= 4'b0001; // Preset to non-zero to avoid lockup
        else        q <= {q[2:0], q[3] ^ q[2]}; // Shift left, feed back XOR to LSB
    end
endmodule
```

**Testbench Code**
```verilog
module tb_lfsr;
    reg clk = 0, rst_n = 0;
    wire [3:0] q;

    lfsr_4bit uut (clk, rst_n, q);
    always #5 clk = ~clk;

    initial begin
        $dumpfile("lfsr_4bit.vcd");
        $dumpvars(0, tb_lfsr);
        $monitor("Time=%0t | rst_n=%b | LFSR State (q)=%b", $time, rst_n, q);
        #12 rst_n = 1;  // Release reset
        #160 $finish;   // Watch it cycle through all 15 states
    end
endmodule
```


### Simulation results

<img width="984" height="227" alt="image" src="https://github.com/user-attachments/assets/4b97925a-039b-4f3e-a84a-36539611c34f" />
