# Exercise 15 — JK Flip-Flop
## Convert a JK FF to a D FF by adding external logic
### Truthtable

| D | Q | QB |
|:-:|:-:|:--:|
| 0 | x | x |
| 0 | 0 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |
| 0 | 1 | 0 |
| 0 | 0 | 1 |

### Code
**Design Code**
```verilog
module d_ff_from_jk (
    input d, clk, 
    output reg q, 
    output qb
);
    assign qb = ~q;
    always @(posedge clk) begin
        case ({d, ~d})
            2'b01: q <= 1'b0; 
            2'b10: q <= 1'b1; 
        endcase
    end
endmodule
```

**Testbench Code**
```verilog
module tb_d_ff;
    reg d, clk = 0; 
    wire q, qb;
    
    d_ff_from_jk uut (d, clk, q, qb);
    always #5 clk = ~clk; // 10ns clock period

    initial begin
        $dumpfile("d_ff_from_jk.vcd");
        $dumpvars(0, tb_d_ff);
        $monitor("Time=%0d | D=%b | Q=%b | QB=%b", $time, d, q, qb);
        d = 0; #12;
        d = 1; #10;
        d = 0; #10;
        $finish;
    end
endmodule
```

### Simulation results

<img width="961" height="259" alt="image" src="https://github.com/user-attachments/assets/be44f5aa-be0c-4c88-ba5a-1535f55ebbf2" />


## Convert a JK FF to a T FF.
### Truthtable

| RST | T | Q | QB |
|:---:|:-:|:-:|:--:|
| 1 | 0 | 0 | 1 |
| 0 | 0 | 0 | 1 |
| 0 | 1 | 0 | 1 |
| 0 | 1 | 1 | 0 |
| 0 | 1 | 0 | 1 |
| 0 | 0 | 0 | 1 |

### Code
**Design Code**
```verilog
module t_ff_from_jk (
    input t, clk, rst,  // <-- Added rst here
    output reg q, 
    output qb
);
    assign qb = ~q;
    always @(posedge clk or posedge rst) begin
        if (rst) 
            q <= 1'b0; // Clear the 'x' state on reset
        else begin
            case ({t, t})
                2'b11: q <= ~q;  // Toggle
                default: q <= q; // Hold
            endcase
        end
    end
endmodule
```

**Testbench Code**
```verilog
module tb_t_ff;
    reg t;
    reg clk = 0; 
    reg rst;           // <-- Added reg rst here
    wire q, qb;
    
    t_ff_from_jk uut (
        .t(t), 
        .clk(clk), 
        .rst(rst),     // <-- Connected rst here
        .q(q), 
        .qb(qb)
    );

    always #5 clk = ~clk; 

    initial begin
        $dumpfile("t_ff_from_jk.vcd");
        $dumpvars(0, tb_t_ff);
        $monitor("Time=%0d | RST=%b | T=%b | Q=%b | QB=%b", $time, rst, t, q, qb);
        
        rst = 1; t = 0; #7; 
        rst = 0;         #5; // Reset released, now active
        
        t = 1; #20; // Toggles smoothly
        t = 0; #10; // Holds smoothly
        $finish;
    end
endmodule
```

### Simulation results

<img width="977" height="269" alt="image" src="https://github.com/user-attachments/assets/a20f580e-460d-4da5-9d61-116c2c95c9a8" />


## Build a 3-bit binary counter using JK flip-flops.
### Truthtable

| Reset | Q2 Q1 Q0 | Decimal |
|:-----:|:--------:|:-------:|
| 1 | 000 | 0 |
| 0 | 000 | 0 |
| 0 | 001 | 1 |
| 0 | 010 | 2 |
| 0 | 011 | 3 |
| 0 | 100 | 4 |
| 0 | 101 | 5 |
| 0 | 110 | 6 |
| 0 | 111 | 7 |
| 0 | 000 | 0 |
| 0 | 001 | 1 |

### Code
**Design Code**
```verilog
module jk_ff (
    input clk, rst, 
    output reg q
);
    always @(negedge clk or posedge rst)
        if (rst) q <= 1'b0;
        else     q <= ~q; 
endmodule

module counter_3bit (
    input clk, rst, 
    output [2:0] q
);
    jk_ff ff0 (clk,    rst, q[0]); // LSB: clocked by main CLK
    jk_ff ff1 (q[0],   rst, q[1]); // Bit 1: clocked by q[0]
    jk_ff ff2 (q[1],   rst, q[2]); // MSB: clocked by q[1]
endmodule
```

**Testbench Code**
```verilog
module tb_counter;
    reg clk = 0, rst; 
    wire [2:0] q;

    counter_3bit uut (clk, rst, q);
    always #5 clk = ~clk; // 10ns clock cycle

    initial begin
        $dumpfile("counter_3bit.vcd");
        $dumpvars(0, tb_counter);
        $monitor("Time=%0d | Reset=%b | Counter Output (Q2 Q1 Q0) = %b (%0d)", $time, rst, q, q);
        rst = 1; #7;  // Assert reset to clear X states
        rst = 0; #90; // Let it cycle past 7 and roll over

        $finish;
    end
endmodule
```


### Simulation results

<img width="983" height="267" alt="image" src="https://github.com/user-attachments/assets/ece8a46d-5995-4914-b82d-af682fc602f5" />
