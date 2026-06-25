# Exercise 23 — Synchronous FIFO
## Add a count output showing the number of entries in the FIFO
### Truthtable

| wr_en | rd_en | full | empty | Qualified Write (wr_en && !full) | Qualified Read (rd_en && !empty) | Next Count (data_count) | Operation / Description |
|:-----:|:-----:|:----:|:-----:|:--------------------------------:|:--------------------------------:|:-----------------------:|:------------------------|
| 0 | 0 | X | X | 0 | 0 | data_count | Idle: No read or write requested |
| 1 | 0 | 0 | X | 1 | 0 | data_count + 1 | Valid Write: Element added |
| 1 | 0 | 1 | X | 0 | 0 | data_count | Blocked Write: FIFO is full; write ignored |
| 0 | 1 | X | 0 | 0 | 1 | data_count - 1 | Valid Read: Element removed |
| 0 | 1 | X | 1 | 0 | 0 | data_count | Blocked Read: FIFO is empty; read ignored |
| 1 | 1 | 0 | 0 | 1 | 1 | data_count | Simultaneous R/W: Stable (net change = 0) |
| 1 | 1 | 1 | 0 | 0 | 1 | data_count - 1 | R/W while Full: Write blocked, read succeeds |
| 1 | 1 | 0 | 1 | 1 | 0 | data_count + 1 | R/W while Empty: Read blocked, write succeeds |

### Code
**Design Code**
```verilog
module fifo_with_count #(
    parameter DATA_WIDTH = 8,
    parameter DEPTH      = 16,
    parameter ADDR_WIDTH = $clog2(DEPTH)
)(
    input  wire                  clk,
    input  wire                  rst_n,      // Active-low asynchronous reset
    input  wire                  wr_en,      // Write enable
    input  wire                  rd_en,      // Read enable
    input  wire [DATA_WIDTH-1:0] din,        // Data input
    output reg  [DATA_WIDTH-1:0] dout,       // Data output
    output wire                  full,       // FIFO full flag
    output wire                  empty,      // FIFO empty flag
    output reg  [ADDR_WIDTH:0]   data_count  
);

    reg [DATA_WIDTH-1:0] mem [0:DEPTH-1];

    reg [ADDR_WIDTH-1:0] wr_ptr;
    reg [ADDR_WIDTH-1:0] rd_ptr;

    assign full  = (data_count == DEPTH);
    assign empty = (data_count == 0);

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            wr_ptr <= 0;
        end else if (wr_en && !full) begin
            mem[wr_ptr] <= din;
            wr_ptr      <= wr_ptr + 1;
        end
    end

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            rd_ptr <= 0;
            dout   <= 0;
        end else if (rd_en && !empty) begin
            dout   <= mem[rd_ptr];
            rd_ptr <= rd_ptr + 1;
        end
    end

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            data_count <= 0;
        end else begin
            case ({wr_en && !full, rd_en && !empty})
                2'b10: data_count <= data_count + 1; // Write only
                2'b01: data_count <= data_count - 1; // Read only
                2'b11: data_count <= data_count;     // Simultaneous Read and Write (no net change)
                default: data_count <= data_count;   // Idle
            endcase
        end
    end

endmodule
```

**Testbench Code**
```verilog
        `timescale 1ns / 1ps
module tb_fifo_with_count;

    parameter DATA_WIDTH = 8;
    parameter DEPTH      = 16;
    parameter ADDR_WIDTH = 4; // $clog2(16)

    reg                  clk;
    reg                  rst_n;
    reg                  wr_en;
    reg                  rd_en;
    reg  [DATA_WIDTH-1:0] din;
    wire [DATA_WIDTH-1:0] dout;
    wire                  full;
    wire                  empty;
    wire [ADDR_WIDTH:0]   data_count;

    fifo_with_count #(
        .DATA_WIDTH(DATA_WIDTH),
        .DEPTH(DEPTH)
    ) dut (
        .clk(clk),
        .rst_n(rst_n),
        .wr_en(wr_en),
        .rd_en(rd_en),
        .din(din),
        .dout(dout),
        .full(full),
        .empty(empty),
        .data_count(data_count)
    );

    always #10 clk = ~clk;

    integer i;

    initial begin
        $dumpfile("fifo_with_count_tb.vcd");
        $dumpvars(0, tb_fifo_with_count);
        $monitor("Time: %0t | wr_en: %b | rd_en: %b | din: %h | dout: %h | full: %b | empty: %b | data_count: %d", 
                  $time, wr_en, rd_en, din, dout, full, empty, data_count);
        clk   = 0;
        rst_n = 0;
        wr_en = 0;
        rd_en = 0;
        din   = 0;

        #25;
        rst_n = 1; // Release reset
        #10;
        
        for (i = 0; i < DEPTH; i = i + 1) begin
            @(posedge clk);
            wr_en = 1;
            din   = i + 10; // Input data: 10, 11, 12...
            #1; // Small delay to observe stable signals after clock edge
        end

        @(posedge clk);
        din = 99;
        #1;
        
        @(posedge clk);
        wr_en = 0; // Stop writing

        for (i = 0; i < DEPTH; i = i + 1) begin
            @(posedge clk);
            rd_en = 1;
            #1;
        end

        @(posedge clk);
        #1;

        @(posedge clk);
        rd_en = 0; // Stop reading
        #20;

        @(posedge clk);
        wr_en = 1; din = 8'hAA;
        @(posedge clk);
        wr_en = 0;
        #1;

        @(posedge clk);
        wr_en = 1;
        rd_en = 1;
        din   = 8'hBB;
        #1;
        
        @(posedge clk);
        wr_en = 0;
        rd_en = 0;
        #20;

        $finish;
    end

endmodule
```

### Simulation results

<img width="1897" height="472" alt="image" src="https://github.com/user-attachments/assets/b0893b6d-1ed1-438c-9e1f-f24289b73953" />

## Implement an almost-full/almost-empty threshold flag.
### Truthtable

| wr_en | rd_en | Current data_count Range | Next data_count | empty | almost_empty | almost_full | full | Operation Description |
|:-----:|:-----:|:------------------------:|:---------------:|:-----:|:------------:|:-----------:|:----:|:----------------------|
| X | X | 0 (Reset/Idle) | 0 | 1 | 1 | 0 | 0 | Empty: Counter holds at 0 |
| 0 | 0 | 1 to 15 | data_count | 0 | See Note | See Note | 0 | Absolute Idle: No operations requested |
| 0 | 0 | 16 | 16 | 0 | 0 | 1 | 1 | Full Idle: No operations requested |
| 1 | 0 | 0 to 15 | data_count + 1 | 0 | See Note | See Note | 0 | Valid Write: Count increments |
| 1 | 0 | 16 | 16 | 0 | 0 | 1 | 1 | Blocked Write: Prevented by overflow protection |
| 0 | 1 | 0 | 0 | 1 | 1 | 0 | 0 | Blocked Read: Prevented by underflow protection |
| 0 | 1 | 1 to 16 | data_count - 1 | See Note | See Note | See Note | 0 | Valid Read: Count decrements |
| 1 | 1 | 0 | 1 | 1 | 1 | 0 | 0 | R/W at Empty: Read blocked, write executes. Count +1 |
| 1 | 1 | 1 to 15 | data_count | 0 | See Note | See Note | 0 | Simultaneous R/W: Operations cancel out. Count stable |
| 1 | 1 | 16 | 15 | 0 | 0 | 1 | 1 | R/W at Full: Write blocked, read executes. Count -1 |

### Code
**Design Code**
```verilog
module fifo_with_thresholds #(
    parameter DATA_WIDTH       = 8,
    parameter DEPTH            = 16,
    parameter ALMOST_FULL_VAL  = 12,  // Assert flag when count >= 12
    parameter ALMOST_EMPTY_VAL = 4,   // Assert flag when count <= 4

    parameter ADDR_WIDTH       = $clog2(DEPTH)
)(
    input  wire                  clk,
    input  wire                  rst_n,
    input  wire                  wr_en,
    input  wire                  rd_en,
    input  wire [DATA_WIDTH-1:0] din,
    output reg  [DATA_WIDTH-1:0] dout,
    output wire                  full,
    output wire                  empty,
    output reg                   almost_full,  // New output flag
    output reg                   almost_empty, // New output flag
    output reg  [ADDR_WIDTH:0]   data_count  
);

    reg [DATA_WIDTH-1:0] mem [0:DEPTH-1];

    reg [ADDR_WIDTH-1:0] wr_ptr;
    reg [ADDR_WIDTH-1:0] rd_ptr;


    assign full  = (data_count == DEPTH);
    assign empty = (data_count == 0);


    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            wr_ptr <= 0;
        end else if (wr_en && !full) begin
            mem[wr_ptr] <= din;
            wr_ptr      <= wr_ptr + 1;
        end
    end


    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            rd_ptr <= 0;
            dout   <= 0;
        end else if (rd_en && !empty) begin
            dout   <= mem[rd_ptr];
            rd_ptr <= rd_ptr + 1;
        end
    end


    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            data_count <= 0;
        end else begin
            case ({wr_en && !full, rd_en && !empty})
                2'b10: data_count <= data_count + 1;
                2'b01: data_count <= data_count - 1;
                default: data_count <= data_count; 
            endcase
        end
    end

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            almost_full  <= 1'b0;
            almost_empty <= 1'b1;  
        end else begin
            almost_full  <= (data_count >= ALMOST_FULL_VAL);
            
            almost_empty <= (data_count <= ALMOST_EMPTY_VAL);
        end
    end

endmodule
```

**Testbench Code**
```verilog
`timescale 1ns / 1ps
module tb_fifo_with_thresholds;

    parameter DATA_WIDTH       = 8;
    parameter DEPTH            = 16;
    parameter ALMOST_FULL_VAL  = 12;
    parameter ALMOST_EMPTY_VAL = 4;
    parameter ADDR_WIDTH       = 4;

    reg                  clk;
    reg                  rst_n;
    reg                  wr_en;
    reg                  rd_en;
    reg  [DATA_WIDTH-1:0] din;
    wire [DATA_WIDTH-1:0] dout;
    wire                  full;
    wire                  empty;
    wire                  almost_full;
    wire                  almost_empty;
    wire [ADDR_WIDTH:0]   data_count;

    fifo_with_thresholds #(
        .DATA_WIDTH(DATA_WIDTH),
        .DEPTH(DEPTH),
        .ALMOST_FULL_VAL(ALMOST_FULL_VAL),
        .ALMOST_EMPTY_VAL(ALMOST_EMPTY_VAL)
    ) dut (
        .clk(clk),
        .rst_n(rst_n),
        .wr_en(wr_en),
        .rd_en(rd_en),
        .din(din),
        .dout(dout),
        .full(full),
        .empty(empty),
        .almost_full(almost_full),
        .almost_empty(almost_empty),
        .data_count(data_count)
    );

    always #10 clk = ~clk;

    integer i;

    initial begin
        $dumpfile("fifo_with_thresholds_tb.vcd");
        $dumpvars(0, tb_fifo_with_thresholds);
        $monitor("Time: %0t | wr_en: %b | rd_en: %b | din: %h | dout: %h | full: %b | empty: %b | almost_full: %b | almost_empty: %b | data_count: %d", 
                  $time, wr_en, rd_en, din, dout, full, empty, almost_full, almost_empty, data_count);
        clk   = 0;
        rst_n = 0;
        wr_en = 0;
        rd_en = 0;
        din   = 0;

        #25;
        rst_n = 1; // Release reset
        #10;
        
        for (i = 1; i <= DEPTH; i = i + 1) begin
            @(posedge clk);
            wr_en = 1;
            din   = i + 100; // Sample data
            #1; // Wait for registered outputs to update
               
        end

        @(posedge clk);
        wr_en = 0; // Stop writing
        #20;

        for (i = 1; i <= DEPTH; i = i + 1) begin
            @(posedge clk);
            rd_en = 1;
            #1; // Wait for registered outputs to update
        end

        @(posedge clk);
        rd_en = 0; // Stop reading
        #20;
        $finish;
    end

endmodule
```

### Simulation results

<img width="1894" height="565" alt="image" src="https://github.com/user-attachments/assets/523866c0-e122-4af7-ba71-4d69f5c1958b" />

## Write a testbench that fills, then drains the FIFO, checking data integrity.
### Code
**Design Code**
Refer the fifo_with_thresholds.v file
**Testbench Code**
```verilog
`timescale 1ns / 1ps
module tb_fifo_data_integrity;

    parameter DATA_WIDTH       = 8;
    parameter DEPTH            = 16;
    parameter ALMOST_FULL_VAL  = 12;
    parameter ALMOST_EMPTY_VAL = 4;
    parameter ADDR_WIDTH       = 4;

    reg                  clk;
    reg                  rst_n;
    reg                  wr_en;
    reg                  rd_en;
    reg  [DATA_WIDTH-1:0] din;
    wire [DATA_WIDTH-1:0] dout;
    wire                  full;
    wire                  empty;
    wire                  almost_full;
    wire                  almost_empty;
    wire [ADDR_WIDTH:0]   data_count;

    fifo_with_thresholds #(
        .DATA_WIDTH(DATA_WIDTH),
        .DEPTH(DEPTH),
        .ALMOST_FULL_VAL(ALMOST_FULL_VAL),
        .ALMOST_EMPTY_VAL(ALMOST_EMPTY_VAL)
    ) dut (
        .clk(clk),
        .rst_n(rst_n),
        .wr_en(wr_en),
        .rd_en(rd_en),
        .din(din),
        .dout(dout),
        .full(full),
        .empty(empty),
        .almost_full(almost_full),
        .almost_empty(almost_empty),
        .data_count(data_count)
    );

    always #10 clk = ~clk;

    reg [DATA_WIDTH-1:0] test_vector [0:DEPTH-1];
    integer i;
    integer errors = 0;

    initial begin
        $dumpfile("fifo_waves.vcd");
        $dumpvars(0, tb_fifo_data_integrity);
        $monitor("Time: %0t | wr:%b rd:%b | din:0x%h dout:0x%h | count:%0d | E:%b AE:%b AF:%b F:%b", 
                 $time, wr_en, rd_en, din, dout, data_count, empty, almost_empty, almost_full, full);
    end

    initial begin
        clk   = 0;
        rst_n = 0;
        wr_en = 0;
        rd_en = 0;
        din   = 0;

        #25;
        rst_n = 1; 
        #10;
        
        for (i = 0; i < DEPTH; i = i + 1) begin
            @(posedge clk);
            if (!full) begin
                wr_en = 1;
                din   = 8'hA0 + i;        
                test_vector[i] = 8'hA0 + i; 
                #1; 
            end
        end

        @(posedge clk);
        wr_en = 0;
        #20;

        for (i = 0; i < DEPTH; i = i + 1) begin
            @(posedge clk);
            if (!empty) begin
                rd_en = 1;
                @(posedge clk); 
                #1; 
                
                if (dout !== test_vector[i]) begin
                    errors = errors + 1;
                end
            end
        end

        @(posedge clk);
        rd_en = 0;
        #20;

        $finish;
    end

endmodule
```

### Simulation results

<img width="1895" height="611" alt="image" src="https://github.com/user-attachments/assets/afe3c93d-635d-4231-9616-db9adf286ccb" />
