# Exercise 10 — 3:8 Decoder
## Build a 4:16 decoder using two 3:8 decoders and logic

### Truthtable

| A3 | A2 | A1 | A0 | Output |
|:--:|:--:|:--:|:--:|:-------------:|
| 0 | 0 | 0 | 0 | Y0  |
| 0 | 0 | 0 | 1 | Y1  |
| 0 | 0 | 1 | 0 | Y2  |
| 0 | 0 | 1 | 1 | Y3  |
| 0 | 1 | 0 | 0 | Y4  |
| 0 | 1 | 0 | 1 | Y5  |
| 0 | 1 | 1 | 0 | Y6  |
| 0 | 1 | 1 | 1 | Y7  |
| 1 | 0 | 0 | 0 | Y8  |
| 1 | 0 | 0 | 1 | Y9  |
| 1 | 0 | 1 | 0 | Y10 |
| 1 | 0 | 1 | 1 | Y11 |
| 1 | 1 | 0 | 0 | Y12 |
| 1 | 1 | 0 | 1 | Y13 |
| 1 | 1 | 1 | 0 | Y14 |
| 1 | 1 | 1 | 1 | Y15 |

## Decoder Selection Logic

- When **A3 = 0**, the **first 3-to-8 decoder** is enabled.
  - Outputs `Y0` to `Y7` are active.
- When **A3 = 1**, the **second 3-to-8 decoder** is enabled.
  - Outputs `Y8` to `Y15` are active.

| A3 | Enabled Decoder | Possible Active Outputs |
|:--:|:---------------:|:----------------------:|
| 0 | Decoder 0 | Y0 – Y7 |
| 1 | Decoder 1 | Y8 – Y15 |
Improve accuracy for strate

### Code
**Design Code**
```verilog
module decoder_3to8 (
    input [2:0] in,
    input enable,
    output reg [7:0] out
);
    
    always @(*) begin
        if (enable) begin
            out = 8'd1 << in; 
        end else begin
            out = 8'b00000000;
       end
    end 
endmodule

module decoder_4to16 (
    input [3:0] in,
    output wire [15:0] out
);
    decoder_3to8 dec0 (
        .in(in[2:0]),
        .enable(~in[3]),
        .out(out[7:0])
    );
    decoder_3to8 dec1 (
        .in(in[2:0]),
        .enable(in[3]),
        .out(out[15:8])
    );
endmodule
```

**Testbench Code**
```verilog
`timescale 1ns
module tb_decoder_4to16;
    reg [3:0] in;
    wire [15:0] out;

    decoder_4to16 uut (
        .in(in),
        .out(out)
    );
    //integer i;

    initial begin
        $dumpfile("decoder_4to16.vcd");
        $dumpvars(0, tb_decoder_4to16);
        // Test all possible inputs
        $monitor("Time %0t: Input: %b, (%2d), Output: %b", $time, in, in, out);

        //in = 4'b0000; #10;
        //integer i;
        for (integer i = 0; i < 16; i = i + 1) begin
            in = i; #10;
            //in = {in[3], in[2], in[1], in[0]} #10;
            
            //if (in == 15) begin
            //    #10;
            //    $finish;
            //end
        end
    end
endmodule
```

### Simulation results

<img width="1152" height="210" alt="image" src="https://github.com/user-attachments/assets/a81a8161-6a01-422c-ac3a-c5427f629f37" />


## Implement with active-low enable and active-low outputs.
### Truthtable 

| EN | IN | OUT  |
|:--:|:--:|:----:|
| 1  | 00 | 1111 |
| 0  | 00 | 1110 |
| 0  | 01 | 1101 |
| 0  | 10 | 1011 |
| 0  | 11 | 0111 |

### Code
**Design Code**
```verilog
module decoder_2to4_active_low(
    input [1:0] in,
    input en,
    output wire[3:0] out
);
    assign out = en ? 4'b1111 : ~(4'b0001 << in);
endmodule
```
**Testbench Code**
```verilog
module tb_decoder_2to4_active_low;
    reg [1:0] in;
    reg en;
    wire [3:0] out;

    decoder_2to4_active_low uut (
        .in(in),
        .en(en),
        .out(out)
    );

    initial begin
        $dumpfile("decoder_2to4_active_low_tb.vcd");
        $dumpvars(0, tb_decoder_2to4_active_low);
        // Test case 1: Enable = 0, all outputs should be high
        en = 1; in = 2'b00; #10;
        $display("Test case 1: en=1, in=00, out=%b", out);
        // Test case 2: Enable = 1, in = 00, out[0] should be low
        en = 0; in = 2'b00; #10;
        $display("Test case 2: en=0, in=00, out=%b", out);

        // Test case 3: Enable = 1, in = 01, out[1] should be low
        in = 2'b01; #10;
        $display("Test case 3: en=0, in=01, out=%b", out);

        // Test case 4: Enable = 1, in = 10, out[2] should be low
        in = 2'b10; #10;
        $display("Test case 4: en=0, in=10, out=%b", out);

        // Test case 5: Enable = 1, in = 11, out[3] should be low
        in = 2'b11; #10;
        $display("Test case 5: en=0, in=11, out=%b", out);

        $finish;
    end
endmodule
```


### Simulation results

<img width="982" height="237" alt="image" src="https://github.com/user-attachments/assets/28b18339-b5f8-457c-88b6-a38dc0bef9b7" />


## Use the decoder to implement a full-adder truth table.

### Truthtable

| A | B | Cin | Sum | Cout |
|:-:|:-:|:---:|:---:|:----:|
| 0 | 0 |  0  |  0  |  0   |
| 0 | 0 |  1  |  0  |  0   |
| 0 | 1 |  0  |  0  |  0   |
| 0 | 1 |  1  |  0  |  0   |
| 1 | 0 |  0  |  0  |  0   |
| 1 | 0 |  1  |  0  |  0   |
| 1 | 1 |  0  |  0  |  0   |
| 1 | 1 |  1  |  0  |  0   |

### Code
**Design Code**
```verilog
module fa_using_demux(
    input a,
    input b,
    input cin,
    input en,
    output sum,
    output cout
);
    wire [7:0] y_n;

    // Continuous assignment mimicking the active-low Demux logic
    assign y_n = en ? 8'b11111111 : ~(1'b1 << {a, b, cin});

    assign sum  = ~(y_n[1] & y_n[2] & y_n[4] & y_n[7]);
    assign cout = ~(y_n[3] & y_n[5] & y_n[6] & y_n[7]);
endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module tb_fa_using_demux;
    reg a, b, cin, en;
    wire sum, cout;

    fa_using_demux uut (
        .a(a),
        .b(b),
        .cin(cin),
        .en(en),
        .sum(sum),
        .cout(cout)
    );

    initial begin
        $dumpfile("fa_using_demux_tb.vcd"); 
        $dumpvars(0, tb_fa_using_demux);
        // Test all combinations of inputs
        for (integer i = 0; i < 8; i = i + 1) begin
            {a, b, cin} = i; // Assign a, b, cin based on the loop index
            en = 1; // Enable the circuit
            #10; // Wait for 10 time units
            $display("a=%b, b=%b, cin=%b => sum=%b, cout=%b", a, b, cin, sum, cout);
        end
    end
endmodule
```

### SImulation results


<img width="1135" height="305" alt="image" src="https://github.com/user-attachments/assets/9fecdf92-6330-4567-bb57-f4022af40f4c" />

