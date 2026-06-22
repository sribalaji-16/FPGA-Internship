# Exercise 14 — D Flip-Flop

## Add a synchronous reset (sample rst on clock edge).

### Truthtable

| rst | d | q |
|:---:|:-:|:-:|
| 1 | 1 | x |
| 1 | 1 | 0 |
| 0 | 1 | 0 |
| 0 | 1 | 1 |
| 0 | 0 | 1 |
| 0 | 0 | 0 |

### Code
**Design Code**
```verilog
module dff_sync_rst (
    input clk, rst, d, 
    output reg q
);
    always @(posedge clk)
        q <= rst ? 1'b0 : d;
endmodule
```


**Testbench Code**
```verilog
module tb_dff;
    reg clk = 0, rst, d; 
    wire q;
    
    dff_sync_rst uut (clk, rst, d, q); // Instantiate design
    
    always #5 clk = ~clk; // 10ns clock period

    initial begin
        $dumpfile("dff_sync_rst.vcd");
        $dumpvars(0,tb_dff);
        $monitor("Time=%0t | rst=%b d=%b | q=%b", $time, rst, d, q);
        rst = 1; d = 1; #12; // Reset active during clock edge
        rst = 0; d = 1; #10; // Reset released, Q becomes 1
        d = 0;          #10; // Q becomes 0
        $finish;
    end
endmodule
```

### Simulation results

<img width="958" height="258" alt="image" src="https://github.com/user-attachments/assets/82c525f9-53db-4eb0-90f7-327d8a5148aa" />


## Add an active-low preset that forces Q=1 asynchronously

### Truthtable

| pre_n | rst | d | q |
|:-----:|:---:|:-:|:-:|
| 1 | 0 | 0 | x |
| 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 0 | 1 | 1 |
| 1 | 1 | 1 | 1 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 1 | 1 |

### Code
**Design Code**
```verilog
module dff_async_pre_sync_rst (
    input clk, preset_n, rst, d, 
    output reg q
);
    always @(posedge clk or negedge preset_n) begin
        if (!preset_n) 
            q <= 1'b1;      // Asynchronous active-low preset
        else           
            q <= rst ? 1'b0 : d; // Synchronous reset / Data transfer
    end
endmodule
```

**Testbench Code**
```verilog
module tb_dff;
    reg clk = 0, preset_n = 1, rst = 0, d = 0; 
    wire q;
    
    dff_async_pre_sync_rst uut (clk, preset_n, rst, d, q);
    always #5 clk = ~clk; 

    initial begin
        $dumpfile("tb_dff.vcd");
        $dumpvars(0, tb_dff);
        $monitor("Time=%0t | pre_n=%b rst=%b d=%b | q=%b", $time, preset_n, rst, d, q);
        #7  d = 1;              // Q becomes 1 at 10ns
        #10 rst = 1;            // Q resets to 0 at 20ns
        #5  preset_n = 0;       // Asynchronously forces Q=1 immediately at 22ns
        #10 preset_n = 1; rst = 0; // Release preset at 32ns
        #10 $finish;
    end
endmodule
```

### Simulation results

<img width="995" height="276" alt="image" src="https://github.com/user-attachments/assets/f8da0486-e33f-4065-a0de-a748001c59be" />


## Build a 4-bit register using four D flip-flops with shared CLK and RST.
### Truthtable

| rst | d    | q    |
|:---:|:----:|:----:|
| 1 | 1011 | xxxx |
| 1 | 1011 | 0000 |
| 0 | 1011 | 0000 |
| 0 | 1011 | 1011 |
| 0 | 0101 | 1011 |
| 0 | 0101 | 0101 |

### Code
**Design Code**
```verilog
module reg_4bit_dff (
    input clk, rst, 
    input [3:0] d, 
    output reg [3:0] q
);
    always @(posedge clk)
        q <= rst ? 4'b0000 : d;
endmodule
```

**Testbench Code**
```verilog
module tb_reg_4bit_dff;
    reg clk = 0, rst; 
    reg [3:0] d; 
    wire [3:0] q;
    
    reg_4bit_dff uut (clk, rst, d, q);
    always #5 clk = ~clk; // 10ns clock period

    initial begin
        $dumpfile("tb_reg_4bit_dff.vcd");
        $dumpvars(0, tb_reg_4bit_dff);
        $monitor("Time=%0t | rst=%b d=%b | q=%b", $time, rst, d, q);
        rst = 1; d = 4'b1011; #12; // Reset active: q becomes 0000
        rst = 0;              #10; // Reset released: q loads 1011
        d = 4'b0101;          #10; // q loads 0101
        $finish;
    end
endmodule
```

### Simulation results

<img width="960" height="260" alt="image" src="https://github.com/user-attachments/assets/b183fe72-61f1-422d-b91f-f9602644acf6" />
