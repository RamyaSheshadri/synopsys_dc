# Synopsys Design Compiler: Brief Overview

**Synopsys Design Compiler** is a leading Electronic Design Automation (EDA) software tool used in the semiconductor industry for RTL (Register Transfer Level) synthesis. It translates digital logic designs written in hardware description languages (like Verilog or VHDL) into optimized gate-level representations ready for physical design and fabrication. Design Compiler enables engineers to:

- Convert behavioral or structural code to standard-cell implementations.
- Optimize designs for area, power, and speed.
- Check for and resolve design rule and timing violations.
- Generate standard output files (e.g., gate-level netlists, synthesis reports).

It is especially valued for handling complex ASIC and SoC (System on Chip) designs, supporting multiple technology libraries, and integrating with industry-standard EDA workflows.

# Half-Adder Project with Synopsys Design Compiler

## Overview
This project implements a simple **half-adder** using Verilog HDL and synthesizes it using **Synopsys Design Compiler**. It is intended as a beginner-level digital design project to get familiar with the ASIC synthesis workflow using industry-standard tools.

---
## Features

- Verilog design of a half adder
- Synthesis script in TCL for Synopsys DC
- Basic hands-on with Design Compiler environment
- Tried and tested on Linux systems with csh shell

---

## Getting Started

This guide walks you through setting up your environment and running your half-adder design with Synopsys Design Compiler.

### Prerequisites

Make sure you have the following:

- Access to **Synopsys Design Compiler** (installed via university or organization tools)
- Unix/Linux machine with terminal access
- Shell: **csh**
- Access/setup to the **Cadwork** directory (organization-specific project area)

---

## 🚀 Setup Instructions

### Step 1: Open Synopsys Directory

1. Open your **file manager**.
2. Navigate to: Home > Cadwork > Synopsys

### Step 2: Open Terminal
- Right-click inside the `Synopsys` folder or use system terminal to open a shell window in this directory.

### Step 3: Run Setup Commands
In your terminal, run the following setup commands one by one:
```text
csh
source .cshrc
mkdir 190
cd 190
```
> This changes your shell to C shell, sets up the Synopsys environment, creates a folder `190`, and navigates into it.
---

## File Structure
Your project folder should look like this after adding source files:
```
190/
├── half_adder.v # Verilog HDL for Half Adder
├── synthesize.tcl # TCL script to run synthesis
├── README.md # Project documentation
├── output/ # (Auto-generated synthesis reports, logs)
```

---
**type the command**: gedit ha.v
Type the code in the editor:
## Verilog Code
```
module ha(input a,b, output s,c);
assign s=a^b;
assign c=a&b;
endmodule
```
Similarly: ha_tb.v
```
`timescale 1ns/1ps

module half_adder_tb;
    // Testbench signals
    reg a, b;
    wire s,c;
    // Instantiate the Half Adder module
    ha uut ( .a(a),.b(b), .s(s),.c(c));
    initial begin
        // Display header
        $display("A B | SUM CARRY");
        $display("---------------");

        // Test all possible input combinations
        a = 0; b = 0; #10;
        $display("%b %b |  %b    %b", a, b, sum, carry);

        a = 0; b = 1; #10;
        $display("%b %b |  %b    %b", a, b, sum, carry);

        a = 1; b = 1; #10;
        $display("%b %b |  %b    %b", a, b, sum, carry);

        $finish;
    end
endmodule
```
## Purpose of `.sdc` and `.tcl` Files

### What is a `.sdc` File?

- **Stands for:** Synopsys Design Constraint.
- **Purpose:** Contains timing and physical constraints used by synthesis and place-and-route tools.
- **Content Examples:**
  - Clock definitions and constraints
  - Input/output delay specifications
  - Design environment parameters (e.g., loading, operating conditions)
- **Importance:** Ensures that the synthesized design meets real-world performance requirements by guiding the tool on critical paths, timing budgets, and other physical considerations.

## Why Use an SDC File for Combinational Circuits?

Even though a half adder is **purely combinational** (no clocks or sequential logic), including an SDC (Synopsys Design Constraints) file is a common best practice in professional synthesis flows:

- **Tool Compliance:** Many synthesis and timing tools require or expect an SDC file, even for combinational logic, to maintain consistency.
- **Modeling Inputs/Outputs:** SDC lets you specify attributes like input/output delay, load, or driving cell, modeling real-world or interfacing logic.
- **Virtual Clock Concept:** For I/O timing, a virtual clock can be defined—even if your design has no actual clock—to let you use input/output delay constraints.
- **Max Delay Constraints:** You can still guide the tool's optimization by constraining the maximum allowable delay between specific input/output combinations, as an example of target timing.

**Summary Table**

| SDC Constraint      | Use in Combinational?                           |
|---------------------|------------------------------------------------|
| Input/Output Delay  | Model I/O interface timing                     |
| Virtual Clock       | Enable timing constraints on I/O               |
| Load/Drive         | Simulate physical characteristics for outputs   |
| Max Delay           | Impose timing limits between I/Os              |

---

## SDC File for Half Adder


```
## 📄 half_adder.sdc

# half_adder.sdc
# Minimal SDC for simple combinational half adder

# Define a virtual clock (10ns period, not connected to module)
create_clock -name virt_clk -period 10

# Set typical input delays (relative to virtual clock)
set_input_delay 2.0 -clock virt_clk [get_ports {a b}]

# Set typical output delays (relative to virtual clock)
set_output_delay 2.0 -clock virt_clk [get_ports {sum carry}]

# Optionally set load for outputs (represents capacitance)
set_load 0.01 [get_ports sum]
set_load 0.01 [get_ports carry]

# End of SDC
```

### What is a `.tcl` File?

- **Stands for:** Tool Command Language (pronounced “tickle”).
- **Purpose:** Contains scripts automating tool operations—such as reading design files, invoking synthesis, reporting results, and managing file outputs.
- **Content Examples:**
  - Commands to read HDL files and libraries
  - Flow control and automation (e.g., for batch synthesis, running multiple commands)
  - Report generation (timing, area, power)
  - Custom tool behaviors or batch processes
- **Importance:** Enables reproducible, automated design flows and reduces human error. TCL scripting is widely supported across EDA tools and is a key skill for VLSI/ASIC engineers.

### TCL script for half adder:
```

# Clean up any previous designs
remove_design -all

# ============================
# 🔧 Setup Libraries and Paths
# ============================
set search_path [list ../rtl ../lib]
set link_library  "* saed32rvt_tt1p05v25c.db"
set target_library "saed32rvt_tt1p05v25c.db"

#Read RTL
analyze -format verilog half_adder.v
elaborate half_adder -architecture verilog -library WORK

# Define Current Design
current_design half_adder

#Link Design
link

#Check Design
check_design

#Compile Design
compile -map_effort medium -area_effort high

#Export Synthesized Netlist
write_file -f verilog -hier -output half_adder_netlist.v

#Reports 
report_area > reports/half_adder_area.rpt
report_timing > reports/half_adder_timing.rpt
report_power > reports/half_adder_power.rpt
```


  ## What is dc_shell?

**dc_shell** is the **command-line interface for Synopsys Design Compiler**, one of the most widely used RTL synthesis tools in the semiconductor industry. When you launch `dc_shell`, you enter a text-based environment where you can input commands interactively or execute scripts to synthesize and analyze digital hardware designs (such as those written in Verilog or VHDL)

- It allows you to run synthesis commands, set up and manage libraries, elaborate designs, optimize for timing, area, and power, and generate reports and netlists.
- You can operate it interactively (typing commands one by one) or by running a **TCL** script (`-t` mode) to automate your flow.
- `dc_shell` works alongside Design Compiler’s graphical interface (Design Vision), but is preferred for automation and scripting in most professional flows.
- To start Design Compiler, you type `dc_shell` (or `dc_shell-t` for TCL mode) at the Unix/Linux terminal prompt; you will then see the `dc_shell>` prompt.

**In short:**  
`dc_shell` is where you control the entire synthesis process for your digital design—setting up technology libraries, reading in RTL code, applying constraints, running optimization, and outputting your gate-level netlist and synthesis reports.

** In this specific workflow,we used dc_shell (or dc_shell-t -f ha.tcl) primarily to run your .tcl script for synthesis and report generation.
### Summary Table

| File Type  | Full Form                         | Main Purpose                                     | Example Use                           |
|------------|-----------------------------------|--------------------------------------------------|---------------------------------------|
| `.sdc`     | Synopsys Design Constraints       | Specify design timing/physical constraints       | Set clock, IO delay, environment      |
| `.tcl`     | Tool Command Language script      | Automate tool operations and flow control        | Run synthesis, generate reports, etc. |

Both `.sdc` and `.tcl` files are critical for ensuring that the chip design is functionally correct, adheres to timing requirements, and that design flows are efficient and automated.


## To view waveform:

### 1. Compile Design + Testbench with vcs:
```
vcs mux2.v mux2_tb.v -R -debug_all
```
* -R: Runs the simulation after compilation
* -debug_all: Enables all signals for waveform viewing in DVE

### 2. Open Waveforms in GUI:
```
dve -vpd vcdplus.vpd &
```
- DVE will open → Use the GUI to:
 - Add signals from the hierarchy
 - Zoom, pan, and inspect transitions
 - Save waveforms for reports

<img width="1895" height="1028" alt="image (13)" src="https://github.com/user-attachments/assets/134d4f7c-3615-4847-b17f-cfa79bcc496c" />

## Output waveform:

<img width="1851" height="1019" alt="image (12)" src="https://github.com/user-attachments/assets/1c2ea3da-31ba-44eb-b5d6-9805719e50bb" />

## Synthesis Report Inferences

This section provides **inferences and observations** drawn from the synthesis reports (`.rpt` files) generated by Synopsys Design Compiler for the Half Adder design. Each report gives insight into the design's power, area, and timing characteristics.


# What is `saed32rvt_tt1p05v25c`?

## Overview

**`saed32rvt_tt1p05v25c`** refers to a specific **standard cell library** used in digital ASIC design workflows. This library is based on the Synopsys Academic 32nm process (SAED32), and is configured for typical process parameters:

- **rvt**: Regular Voltage Threshold (RVT) transistors
- **tt**: Typical-Typical process corner (standard process/voltage/temperature)
- **1p05v**: 1.05 Volts supply
- **25c**: 25°C operating temperature

## Is This a `.lib` File?

Yes, `saed32rvt_tt1p05v25c` is **represented by a `.lib` file** (and in compiled flows, a `.db` file). Here's how it fits in:

- **`.lib` files** are standard text-based format files describing the characteristics (timing, power, functionality, pin info) of all standard cells in a library, under the specified conditions. These are known as **Liberty files**.
- The name `saed32rvt_tt1p05v25c.lib` identifies a **Liberty file for the 32nm SAED RVT library at 1.05V and 25°C** operating point.

## Where You Use It

You specify this file in your synthesis tools (like Synopsys Design Compiler) as the **target and link library**. This tells the tool which gates and timing/power models to use:

- Example:
```
set target_library "path/to/saed32rvt_tt1p05v25c.lib"
```
- Some flows use a compiled `.db` version (binary) for efficiency, but both originate from the `.lib` text file.

## Why It Matters

- The choice of library affects timing, area, and power results—because every gate’s performance is modeled according to the exact process (e.g., 32nm, regular Vt, 1.05V, 25°C).
- These libraries are crucial for accurate analysis and implementation during synthesis and later design phases.

## Summary Table

| Name Example                | File Type | Technology | V<sub>DD</sub> | Temp | Use                    |
|-----------------------------|-----------|------------|----------------|------|------------------------|
| saed32rvt_tt1p05v25c.lib    | .lib      | SAED32 RVT | 1.05V          | 25°C | Synthesis, STA, P&R    |
| saed32rvt_tt1p05v25c.db     | .db       | SAED32 RVT | 1.05V          | 25°C | Compiled binary (faster)|

**In short:**  
`saed32rvt_tt1p05v25c` is a Liberty-format cell library file—most commonly named `saed32rvt_tt1p05v25c.lib`—providing cell definitions, timing, and power data for 32nm ASIC synthesis and analysis at typical (1.05V, 25°C) conditions.

---

### 1. Power Report (`ha_power.rpt`)

- **Library used:** `saed32rvt_tt1p05v25c`
- **Operating Voltage:** 1.05V
- **Total Cell Internal Power:** 681.8254 nW
- **Net Switching Power:** 0.02534 nW
- **Total Dynamic Power:** 681.85073 nW (mostly internal)
- **Cell Leakage Power:** 486.9799 nW
- **Total Power:** ≈ 1.2065 μW

<img width="1920" height="1053" alt="power rpt 2" src="https://github.com/user-attachments/assets/7d8533d8-9ac1-494c-a524-9fa49e66729b" />


**Inference:**  
The half adder's power consumption is extremely low with negligible switching power (<0.01% of total). The overall power is mainly split between internal and leakage components, reflecting a simple design with static gates and no clocked elements. This is expected for such a small combinational circuit.

---

### 2. Area Report (`ha_area.rpt`)

- **Number of ports:** 4
- **Number of nets:** 4
- **Number of cells:** 2
- **Combinational cells:** 2
- **No sequential cells, macros, or black boxes**
- **Combinational Area:** 6.335860
- **Buf/Inv Area, Macro, etc.:** 0
- **Total cell area:** 6.335860
- **Total area:** 6.888930

<img width="1920" height="1053" alt="area rpt" src="https://github.com/user-attachments/assets/3feb9e0c-c558-4830-898f-7e2263c2b08e" />


**Inference:**  
- "Number of cells: 2" and "Combinational cells: 2" specifically refer to the two logic gates used in the synthesized design:
    - 1 XOR gate: Implements the sum output (sum = a ^ b)
    - 1 AND gate: Implements the carry output (carry = a & b)
- The area metrics confirm the minimal nature of the circuit, consisting of just two combinational cells corresponding to the XOR (sum) and AND (carry) logic of a half adder. There are no sequential or buffer/inverter elements, which aligns with the pure combinational logic.

---

### 3. Timing Report (`ha.qor.rpt`)

- **Levels of Logic:** 1.00
- **Critical Path Length:** 0.09 (unit not specified, likely ns)
- **No violating paths, no hold violations**
- **Combinational Area:** 6.335860
- **Total Cell Area:** 6.335860

  <img width="1920" height="1053" alt="timing rpt" src="https://github.com/user-attachments/assets/c5c60098-d472-481d-b77e-deaa1705ea5b" />


**Inference:**  
The design critical path is very short (low delay), with only a single logic level, showing outstanding timing performance and no negative slack or violations. This is ideal for elementary logic cells like a half adder.

---
## What is QoR (Quality of Results)?

**QoR (Quality of Results)** is a collection of synthesis and implementation metrics used to evaluate how good your hardware design is in terms of **timing, area, power, and performance**.

It is an essential part of any synthesis or implementation flow, especially when using tools like **Synopsys Design Compiler**, **Vivado**, or **Yosys**.

---
##  What Does QoR Measure?

The QoR report typically includes:

|  Metric           |  Description                                      |
|---------------------|-----------------------------------------------------|
| **Slack (ns)**      | Timing margin before a path becomes violating       |
| **Critical Path**   | Longest delay path in the design (limits frequency) |
| **Max Frequency**   | Inverse of critical path (how fast your design runs)|
| **Area (μm²)**      | Total silicon area used by the design               |
| **Power (μW/mW)**   | Estimated power consumption                         |
| **Gate/Cell Count** | Total logic elements used (e.g., AND, XOR gates)    |
| **Levels of Logic** | Number of gate "layers" between input and output    |

You get this QoR table after synthesis or optimization, and it is used to compare how efficient and optimized the final design is.

---

## Why is QoR Important?

- **Benchmark Your Design:** Know how well your design performs on key metrics.
- **Identify Bottlenecks:** Slack issues? Too much area? Too slow? QoR tells you.
- **Compare Runs:** Use QoR to compare two different synthesis strategies or code versions.
- **Guide Optimization:** "Does this version improve area without hurting timing?"

<img width="1920" height="1053" alt="qor rpt" src="https://github.com/user-attachments/assets/ea2007e8-70ed-4f2e-bb84-cd9f84366525" />
<img width="1920" height="1053" alt="qor rpt 2" src="https://github.com/user-attachments/assets/44f6cc69-a75e-487c-b05a-844837e6af47" />

- **QoR = the synthesis tool’s report card for your design.**
- QoR = A summary of the most important outputs from the synthesis stage.
- It’s not a separate report — it’s a collected view that includes key parts of:
  - Timing Report
  - Area Report
  - Power Report
  - (Sometimes: DRC Summary, Gate Count, Logic Levels, etc.)
- It tells you **how fast**, **how small**, and **how efficient** your circuit is.
- Every time you see or generate a `.rep` file after synthesis (`report_qor`, `report_timing`, etc.), you're looking at the **QoR summary**.
- Tracking QoR across different designs lets you compare and decide which version to keep or optimize further. It becomes a powerful part of your flow — especially in VLSI placements and real-world tapeouts!

> *These reports confirm that the synthesized Half Adder is highly efficient in terms of area, power, and timing, with no timing violations and an extremely compact gate-level implementation.*

# 3 bit Synchronous Counter:
### Overview:
This project demonstrates synthesizing a 3-bit synchronous counter using Synopsys Design Compiler (DC). The process takes Verilog RTL, applies synthesis with timing constraints, and generates a gate-level netlist suitable for ASIC design flow.

### counter.v:
```
module counter(input clk,rst,
output reg y);
always@(posedge clk)
begin
    if(rst)
        y<=0;
    else
        y<=y+1
end
endmodule
```

### counter_tb.v
```
modult counter_tb();
reg clk,rst;
wire y;
counter uut(clk,rst,y);
initial
    begin
        clk=0;
    end
always #5 clk=~clk;

initial
begin
rst=0;
#10 rst=1;
#10 rst=0;
#100 $finish;
end

 initial begin
    $dumpfile("dump.vcd");         // VCD file name
    $dumpvars(0, counter_tb);      // Dump all variables in this module
  end
endmodule
```
**To view waveform:**
- first compile using: vcs -l run.log -R -debug_access+all +vcs+vcdpluson +vpd counter.v counter_tb.v
- then: dve -vpd vcdplus.vpd &

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3808bb29-33ef-4032-93aa-0c8413985277" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/38d4354f-9b80-4f35-8541-9f001eaef785" />

### SDC(Synopsys Design Compiler) top.sdc
This file is used to define **timing constraints** for your counter module during synthesis with Synopsys Design Compiler. Each section of the script plays a specific role to guide the synthesis tool in analyzing the circuit's timing.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ab93316a-11d3-4c68-9b04-8a86c82af623" />

Below is an explanation:
### 1. Define the Clock Signal
```tcl
create_clock -name clk -period 10 [get_ports clk]
```
- **Purpose:** Defines a clock named `clk` with a period of 10 ns (corresponding to a frequency of 100 MHz).
- **Effect:** All timing analysis and constraints will be referenced to this clock.

### 2. Set Input Delays

```tcl
set_input_delay 2 -clock clk [get_ports reset]
```
- **Purpose:** Sets the input arrival time for the `reset` port to 2 ns relative to the clock.
- **Effect:** Accounts for the delay from the external signal source (like a testbench or other logic) to the input pin of the design.

### 3. Set Output Delays

```tcl
set_output_delay 2 -clock clk [get_ports q]
```
- **Purpose:** Sets output required time for the `q` port (which is the 3-bit counter output) to 2 ns relative to the clock.
- **Effect:** Informs the tool how much time downstream logic has to latch the signal after it is generated.

### 4. Setup and Hold Times for Flip-Flops

```tcl
set_max_delay 5 -from [get_pins uut/q[3]] -to [get_pins uut/q[0]]
set_min_delay 1 -from [get_pins uut/q[3]] -to [get_pins uut/q[0]]
```
- **Purpose:** Specifies maximum and minimum delays between specific flip-flop pins, which could correspond to setup and hold requirements.
- **Effect:** Helps in analyzing internal paths between flip-flops, enforcing them to be between 1 and 5 ns.
- **Note:** The actual pin names and constraints may need adjustment depending on your cell library and design.

### 5. Multi-cycle and Special Path Constraints

(Comments in your file, but here’s what they mean)
- For signals like reset, if they are synchronized to the clock, you might specify a multi-cycle path.
- Example (commented in your file):

```tcl
set_multicycle_path 2 -setup -from [get_ports reset] -to [get_pins uut/q]
```
**Effect:** If uncommented, instructs DC that the path from reset to q stretches across 2 clock cycles (multi-cycle path).

### 6. False Paths

```tcl
set_false_path -from [get_ports reset]
```
- **Purpose:** Tells the tool to ignore timing analysis for the path from the `reset` port.
- **Use Case:** Useful for asynchronous resets, which don’t need traditional timing checking.

### 7. Reporting

```tcl
report_timing -path full -delay max
```
- **Purpose:** Generates a detailed timing report, showing the maximum delay paths through the design.
- **Effect:** This helps you check for violations and see if your constraints are being met.

## Summary

**This SDC file provides essential clock, input/output, and internal timing constraints,** which enable Synopsys Design Compiler to correctly analyze and optimize your design for timing. These commands help ensure your synthesized counter will function reliably at your specified clock frequency (100 MHz) and interface cleanly with the rest of your digital system.

### TCL Script:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9d0e3c33-76cd-4954-b438-adcac5f1a8c9" />

**This script automates the standard ASIC synthesis steps:**
- Setup library
- Read and elaborate RTL design
- Apply constraints
- Synthesize (compile)
- Export mapped netlist
- Generate area/timing/power/QoR reports
You can launch Synopsys DC, load this script, and it will run all steps for your counter design in one go, provided all paths and module names are correct.

Yes, you are correct! To run your synthesis Tcl script (such as `counter.tcl`), you need to launch the Synopsys Design Compiler shell (dc_shell). Here’s how you do it:

## Steps to Run Your Tcl Script in Synopsys DC

1. **Open a terminal** on your system.
2. **Start the Design Compiler shell** by typing:
   ```bash
   dc_shell
   ```
3. **Source your Tcl script** inside the DC shell:
   ```tcl
   source counter.tcl
   ```
## What Happens Next
- The Design Compiler will execute the commands in your Tcl script.
- It will read your Verilog design, apply SDC constraints, perform synthesis, and generate reports and netlists as specified in your script.

# Explanation of Synthesis Reports:

## 1. Power Report
**Purpose:**  
Estimates how much power your synthesized counter will consume in operation.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/51b3e635-5504-4615-9c61-c8eabba68579" />

### Key Sections & Meaning

- **Global Operating Voltage:**  
  The design operates at 1.05V (supply voltage).

- **Total Dynamic Power (2.9572μW):**  
  Power consumed due to the logic switching activities. This is further split into:
  - **Internal Power (2.7340μW):**  
    Energy dissipated inside logic gates as they switch.
  - **Net Switching Power (0.2232μW):**  
    Energy from charging/discharging load capacitance on nets.

- **Cell Leakage Power (4.5986nW):**  
  Leakage current drawn from the supply even when the logic isn’t switching (standby loss).

- **Total Power (7.5558μW):**  
  The sum of dynamic and leakage power—this is the complete power profile for your counter.

#### How to interpret:
- These values are LOW, as expected for a small 3-bit sequential counter in 32nm technology.
- The vast majority is dynamic power. Leakage is much less critical at this size, but would matter for larger chips.

## 2. Area Report

**Purpose:**  
Shows how much silicon area your synthesized logic occupies.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/78d5aa2d-b3d1-4742-a99c-4f6bcc3e7718" />

### Important Fields

- **Number of Ports, Nets, Cells:**  
  - *Ports:* 5 (inputs + outputs)
  - *Nets:* 12 (wires connecting logic)
  - *Cells:* 10 (standard cell components)
  - *Sequential Cells:* 7 (e.g., flip-flops for each of the 3 counter bits, and possibly a few for control)
  - *Combinational Cells:* 3 (e.g., logic gates for incrementing the counter)

- **Combinational Area (18.04μm²):**  
  Area used by gates (AND, OR, XOR).

- **Non-Combinational Area (19.82μm²):**  
  Area for sequential elements (D flip-flops, latches).

- **Total Cell Area (37.87μm²):**  
  Combined standard cells’ area.

- **Total Area (39.19μm²):**  
  Includes interconnect (metal wiring) overhead. The difference with “cell area” is the routing overhead.

#### How to interpret:
- This is a *very compact* and highly optimized digital block—just a few dozen square microns.
- The cell ratio shows your design is dominated about equally by combinational and sequential logic.

## 3. Timing Report

**Purpose:**  
Checks if your counter meets the required clock speed and that all signals arrive in time.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0b5189e9-2b5d-40c1-8678-eb9fb3467fb4" />

### Key Metrics

- **Clock Period:**  
  You specified 10ns (implies 100MHz operation).

- **Data Required Time (9.97ns):**  
  The maximum allowed time for data to propagate from one register (flip-flop) to another and still meet setup timing.

- **Data Arrival Time (0.29ns):**  
  How long it actually takes for data to traverse the critical path.

- **Slack (9.68ns):**  
  Margin between required and actual arrival time.  
  - *Positive slack* (which you have) means the design is FAST—signals arrive much earlier than required.
  - *Negative slack* would indicate a timing violation (design too slow).

#### How to interpret:
- Your counter is “over-designed” for the constraint: it can work at much higher frequencies if needed.
- Typical for simple counters; adding complexity or more bits would reduce the slack.

## Putting It All Together

- **Your 3-bit counter is tiny, low-power, and fast.**
- You are well within safe limits for area, power, and speed.
- To optimize for larger designs, trade-offs may appear (e.g., area vs. speed, power vs. performance).

## Why Do These Reports Matter?

- **Power:** Essential if battery life, thermal limits, or energy budget matters.
- **Area:** Drives cost per chip; large area means higher cost and possibly lower yield.
- **Timing:** Ensures your design runs at the intended frequency without glitches or errors.

## Understanding the Role of SDC File vs. Area and Power Reports

### What the SDC File Controls

- The SDC (Synopsys Design Constraints) file specifies **timing constraints** (like clocks, input delays, output delays, false paths) for your design.
- Its main goal is to tell the synthesis tool how fast your circuit needs to operate, which inputs/outputs are related to which clocks, and which signals should be ignored for timing.

### How Area and Power Are Determined

- **Area and power are not set directly by the SDC file.**  
  Instead, they are influenced by your netlist (RTL code), the library you use (cell area, power models), and the synthesis tool's optimizations performed to meet the constraints you provide.
- When you set a **tight timing constraint** (shorter clock period), the tool may optimize the design to be faster—potentially increasing area and dynamic power to meet timing.
- If timing constraints are **relaxed** (longer clock period), the tool might use smaller, slower, or fewer gates—reducing area and power.

### How to Judge Whether Area and Power Are "Proper"

- You can tell if your area and power are **reasonable** only by interpreting the reports generated after synthesis.
  - **Area report** (e.g., `counter_area.rpt`): Shows the total cell and total area in μm².
  - **Power report** (e.g., `counter_power.rpt`): Details dynamic and leakage power.
- What is "correct" depends on:
  - The logic complexity (for a 3-bit counter, both area and power should be extremely low).
  - The technology node/library being used.
  - Meeting your timing constraints **without extra, unnecessary logic or excessive switching activity**.

#### Example: For a simple 3-bit counter, single-digit μW power and under 50μm² area are typical in small nodes like 32nm, and large positive slack in timing reports confirms no excessive effort is wasted to just barely meet timing.

### Key Points

- **You do not set area and power in the SDC**—the tool tries to meet the required timing with minimal hardware.
- **Reports are "proper" if:**
  - Your timing constraints are satisfied (large, positive slack).
  - Area and power values match expectations for your counter size and target library.
  - No "over-design" (unreasonably large area/power for the function) or "under-design" (unable to meet timing, or missing logic).
- For advanced optimization, you can add **area or power constraints** using tool-specific commands in your synthesis script (not SDC), but for most student flows, reviewing the reports is sufficient.

**In summary:**  
Your SDC file guides the tool to synthesize for speed; the synthesis tool balances this with area/power. The generated area and power reports show you the actual cost, and you determine if they are "proper" by comparing with typical values for your design and technology.

## Quality of Results (QoR) summary report generated by Synopsys Design Compiler after synthesizing 3-bit counter. 
- This type of report gives a concise snapshot of your design’s key implementation metrics—timing, area, resource usage, design rule compliance, and tool runtime.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/27465c64-3467-4230-9d5a-bd795a153563" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a06abeb4-918e-43bb-b48a-9e72e69c3467" />

## Synthesis QoR (Quality of Results) Summary

The attached QoR report summarizes the results of synthesizing the 3-bit counter RTL with Synopsys Design Compiler:
- **Timing:**  
  - The critical path delay is **0.29ns**, yielding a very large slack of **9.71ns** against the 10ns clock constraint, meaning the design far exceeds timing requirements.
  - **No setup or hold violations** are present.

- **Area:**  
  - The total standard cell area consumed is **~37.87µm²**, with a total design area (including interconnect) of **~39.91µm²**, indicating a very compact implementation.

- **Cell Usage:**  
  - The design consists of **10 basic (leaf) cells**:  
    - **7 sequential cells** (e.g., flip-flops for each bit of the counter plus any required control),  
    - **3 combinational cells** (for the counter logic),  
    - Plus a small number of buffers/inverters.

- **Design Rule Check:**  
  - **No violations** reported in net, transition, or capacitance rules: the synthesized netlist is physically valid for downstream implementation.

- **Synthesis Runtime:**  
  - Compile and optimization completed in approximately **1.17 seconds**.

**Summary:**  
This QoR report confirms that the synthesized 3-bit counter meets and greatly exceeds all specified timing, area, and design rule constraints, making it an efficient and robust result for further ASIC flow steps.

### QoR Outcome (Footer)
- All critical metrics are summarized again:
 - No setup or hold violations.
 - No violating timing paths.
 - Design has been fully checked and passes essential physical, timing, and logic requirements.

### What Does a QoR Report Tell You?
- Implementation Sanity: Your synthesized design is correct and fits your constraints.
- Design Health: You can tell if you are meeting area and timing budgets, have electrical rule errors, or are wasting hardware.
- Next Steps: If all metrics are green (as here), you’re ready for downstream flow (e.g., place & route or handoff).
