# MINI PROJECTS
## Digital Safe System

### Truthtable - 4-Bit Digital Safe System

| S3 | S2 | S1 | S0 | Input Password | L1 (Correct Password) | L2 (Wrong Password) |
|:--:|:--:|:--:|:--:|:--------------:|:---------------------:|:-------------------:|
| 0 | 0 | 0 | 0 | 0000 | 0 | 1 |
| 0 | 0 | 0 | 1 | 0001 | 0 | 1 |
| 0 | 0 | 1 | 0 | 0010 | 0 | 1 |
| 0 | 0 | 1 | 1 | 0011 | 0 | 1 |
| 0 | 1 | 0 | 0 | 0100 | 0 | 1 |
| 0 | 1 | 0 | 1 | 0101 | 0 | 1 |
| 0 | 1 | 1 | 0 | 0110 | 0 | 1 |
| 0 | 1 | 1 | 1 | 0111 | 0 | 1 |
| 1 | 0 | 0 | 0 | 1000 | 0 | 1 |
| 1 | 0 | 0 | 1 | 1001 | 0 | 1 |
| 1 | 0 | 1 | 0 | 1010 | 1 | 0 |
| 1 | 0 | 1 | 1 | 1011 | 0 | 1 |
| 1 | 1 | 0 | 0 | 1100 | 0 | 1 |
| 1 | 1 | 0 | 1 | 1101 | 0 | 1 |
| 1 | 1 | 1 | 0 | 1110 | 0 | 1 |
| 1 | 1 | 1 | 1 | 1111 | 0 | 1 |

### Code
**Design File**
```verilog
module digital_safe(
    input clk,
    input [3:0] sw,
    output led_unlock,
    output led_wrong,
    output reg [6:0] seg,
    output reg [3:0] an
);

parameter PASSWORD = 4'b1010;

wire unlock;

assign unlock = (sw == PASSWORD);

assign led_unlock = unlock;
assign led_wrong  = ~unlock;

reg [19:0] refresh_counter = 20'd0;

always @(posedge clk)
    refresh_counter <= refresh_counter + 1;

wire [1:0] digit = refresh_counter[19:18];

always @(*) begin

    an  = 4'b1111;
    seg = 7'b1111111;

    case(digit)

    // RIGHTMOST (AN0)
    2'b00: begin
        an = 4'b1110;
        if(unlock)
            seg = 7'b0101011;   
        else
            seg = 7'b1111111;   
    end

    // AN1
    2'b01: begin
        an = 4'b1101;
        if(unlock)
            seg = 7'b0000110;  
        else
            seg = 7'b0001110;   
    end

    // AN2
    2'b10: begin
        an = 4'b1011;
        if(unlock)
            seg = 7'b0001100;  
        else
            seg = 7'b0001110;  
    end

    // LEFTMOST (AN3)
    2'b11: begin
        an = 4'b0111;
        if(unlock)
            seg = 7'b1000000;   
        else
            seg = 7'b1000000; 
    end

    endcase

end
endmodule
```

**Constraint file**
```xdc
## This file is a general .xdc for the Basys3 rev B board
## Tailored for the digital_safe module

## Clock signal
set_property -dict { PACKAGE_PIN W5    IOSTANDARD LVCMOS33 } [get_ports clk]
create_clock -add -name sys_clk_pin -period 10.00 -waveform {0 5} [get_ports clk]


## Switches (Using the 4 leftmost switches sw[3:0] for the password)
set_property -dict { PACKAGE_PIN V17   IOSTANDARD LVCMOS33 } [get_ports {sw[0]}]
set_property -dict { PACKAGE_PIN V16   IOSTANDARD LVCMOS33 } [get_ports {sw[1]}]
set_property -dict { PACKAGE_PIN W16   IOSTANDARD LVCMOS33 } [get_ports {sw[2]}]
set_property -dict { PACKAGE_PIN W17   IOSTANDARD LVCMOS33 } [get_ports {sw[3]}]


## LEDs (Using LED 0 for Unlock status and LED 1 for Wrong Password status)
set_property -dict { PACKAGE_PIN U16   IOSTANDARD LVCMOS33 } [get_ports led_unlock]
set_property -dict { PACKAGE_PIN E19   IOSTANDARD LVCMOS33 } [get_ports led_wrong]


##7 Segment Display
set_property -dict { PACKAGE_PIN W7    IOSTANDARD LVCMOS33 } [get_ports {seg[0]}]
set_property -dict { PACKAGE_PIN W6    IOSTANDARD LVCMOS33 } [get_ports {seg[1]}]
set_property -dict { PACKAGE_PIN U8    IOSTANDARD LVCMOS33 } [get_ports {seg[2]}]
set_property -dict { PACKAGE_PIN V8    IOSTANDARD LVCMOS33 } [get_ports {seg[3]}]
set_property -dict { PACKAGE_PIN U5    IOSTANDARD LVCMOS33 } [get_ports {seg[4]}]
set_property -dict { PACKAGE_PIN V5    IOSTANDARD LVCMOS33 } [get_ports {seg[5]}]
set_property -dict { PACKAGE_PIN U7    IOSTANDARD LVCMOS33 } [get_ports {seg[6]}]

set_property -dict { PACKAGE_PIN U2    IOSTANDARD LVCMOS33 } [get_ports {an[0]}]
set_property -dict { PACKAGE_PIN U4    IOSTANDARD LVCMOS33 } [get_ports {an[1]}]
set_property -dict { PACKAGE_PIN V4    IOSTANDARD LVCMOS33 } [get_ports {an[2]}]
set_property -dict { PACKAGE_PIN W4    IOSTANDARD LVCMOS33 } [get_ports {an[3]}]


## Configuration options, can be used for all designs
set_property CONFIG_VOLTAGE 3.3 [current_design]
set_property CFGBVS VCCO [current_design]

## SPI configuration mode options for QSPI boot, can be used for all designs
set_property BITSTREAM.GENERAL.COMPRESS TRUE [current_design]
set_property BITSTREAM.CONFIG.CONFIGRATE 33 [current_design]
set_property CONFIG_MODE SPIx4 [current_design]
```


### Results

https://github.com/user-attachments/assets/624b36e0-cb11-4944-ac69-4b3ec6422e80

## Car Park Occupied Slot Counting System

### Truthtable

| s0 | s1 | s2 | c1 | c0 |
|:--:|:--:|:--:|:--:|:--:|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 | 1 |
| 0 | 1 | 0 | 0 | 1 |
| 0 | 1 | 1 | 1 | 0 |
| 1 | 0 | 0 | 0 | 1 |
| 1 | 0 | 1 | 1 | 0 |
| 1 | 1 | 0 | 1 | 0 |
| 1 | 1 | 1 | 1 | 1 |

### Code
**Design File**
```verilog
module car_park(
    input s0,
    input s1,
    input s2,
    output c1,
    output c0
);

assign c1 = (s0 & s1) | (s1 & s2) | (s0 & s2);

assign c0 = s0 ^ s1 ^ s2;

endmodule
```

**Constraints File**
```xdc
############################
## Switches
############################

set_property PACKAGE_PIN V17 [get_ports s0]
set_property IOSTANDARD LVCMOS33 [get_ports s0]

set_property PACKAGE_PIN V16 [get_ports s1]
set_property IOSTANDARD LVCMOS33 [get_ports s1]

set_property PACKAGE_PIN W16 [get_ports s2]
set_property IOSTANDARD LVCMOS33 [get_ports s2]

############################
## LEDs
############################

set_property PACKAGE_PIN U16 [get_ports c0]
set_property IOSTANDARD LVCMOS33 [get_ports c0]

set_property PACKAGE_PIN E19 [get_ports c1]
set_property IOSTANDARD LVCMOS33 [get_ports c1]
```

### Results

https://github.com/user-attachments/assets/19167356-df54-4ca1-a219-919a7717a7fc
