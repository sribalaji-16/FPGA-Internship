# Exercise 13 — SR Latch
##  Implement an SR latch using NAND gates (active-low inputs).

### Truthable 

| Sn | Rn | Q | Qn |
|:--:|:--:|:-:|:--:|
| 0 | 0 | 1 | 1 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 0 | 1 |
| 1 | 1 | 0 | 1 |

### Code
**Design Code**
```verilog
module sr_latch_using_nand (
    input Sn, Rn, 
    output Q, Qn
);
    assign Q  = ~(Sn & Qn);
    assign Qn = ~(Rn & Q);
endmodule
```

**Testbench Code**
```verilog
module tb_sr_latch_using_nand;
    reg Sn, Rn; 
    wire Q, Qn;

    integer i;

    sr_latch_using_nand uut (Sn, Rn, Q, Qn);

    initial begin
        $dumpfile("sr_latch_using_nand.vcd");
        $dumpvars(0, tb_sr_latch_using_nand);
        $monitor("Sn=%b Rn=%b | Q=%b Qn=%b", Sn, Rn, Q, Qn);
        for(i = 0; i < 4; i = i + 1) begin
            {Sn, Rn} = i; #10;
        end
    end
endmodule
```

### Simulation results

<img width="999" height="281" alt="image" src="https://github.com/user-attachments/assets/079fbe57-d62c-442f-9ffe-55b8560b3071" />


## Add a Clock enable to create a gated SR latch

### Truthable

| E | S | R | Q | Qn |
|:-:|:-:|:-:|:-:|:--:|
| 1 | 0 | 1 | 0 | 1 |
| 0 | 0 | 0 | 0 | 1 |
| 0 | 0 | 1 | 0 | 1 |
| 0 | 1 | 0 | 0 | 1 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 0 | 1 |
| 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 0 | 1 | 0 |
| 1 | 1 | 1 | 1 | 1 |

### Code
**Design Code**
```verilog
module gated_sr_latch (
    input S, R, E, 
    output Q, Qn
);
    assign Q  = ~(~(S & E) & Qn);
    assign Qn = ~(~(R & E) & Q);
endmodule
```

**Testbench Code**
```verilog
module tb_gated_sr_latch;
    reg S, R, E; 
    wire Q, Qn;
    
    integer i;

    gated_sr_latch uut (S, R, E, Q, Qn);

    initial begin
        $dumpfile("gated_sr_latch.vcd");
        $dumpvars(0, tb_gated_sr_latch);
        $monitor("E=%b S=%b R=%b | Q=%b Qn=%b", E, S, R, Q, Qn);
        
        // 1. Force an initial valid state (Reset the latch)
        E = 1; S = 0; R = 1; #10; 
        
        // 2. Now run your loop to test all states
        for(i = 0; i < 8; i = i + 1) begin
            {E, S, R} = i; #10;
        end
    end
endmodule
```


### Simulation results

<img width="985" height="304" alt="image" src="https://github.com/user-attachments/assets/68c78ca2-1096-45b4-9629-e4e8c40f7db5" />


## Explain why S=R=1 is forbidden and show simulation waveform.

### Truthtable

| Enable (E) | Set (S) | Reset (R) | Output (Q) | Complement (Q̅) | Observed State Description |
|:----------:|:-------:|:---------:|:----------:|:---------------:|:---------------------------|
| **1** | 0 | 1 | **0** | **1** | **Reset State** |
| **1** | 1 | 0 | **1** | **0** | **Set State** |
| **1** | 1 | 1 | **1** | **1** | ❌ **Forbidden / Invalid State** (Outputs are equal) |

### Code
**Design Code**
```verilog
module gated_sr_latch (
    input S, R, E, 
    output Q, Qn
);
    assign Q  = ~(~(S & E) & Qn);
    assign Qn = ~(~(R & E) & Q);
endmodule
```

**Testbench Code**
```verilog
module tb_gated_sr_latch_forbidden_state;
    reg S, R, E; 
    wire Q, Qn;

    gated_sr_latch uut (S, R, E, Q, Qn);

    initial begin
        $dumpfile("dump.vcd"); 
        $dumpvars(0, tb_gated_sr_latch_forbidden_state);
        $monitor("E=%b S=%b R=%b | Q=%b Qn=%b", E, S, R, Q, Qn);
        
        E=1; S=0; R=1; #10; // Initialize to Reset (Q=0)
        E=1; S=1; R=0; #10; // Set State (Q=1, Qn=0)
        E=1; S=1; R=1; #10; // FORBIDDEN STATE (Q=1, Qn=1)
        E=1; S=0; R=0; #10; // Race Condition / Unstable Hold
    end
endmodule
```

### Simulation results

<img width="978" height="289" alt="image" src="https://github.com/user-attachments/assets/6390b5c7-4fd1-4b64-a571-dcb4c1879676" />
