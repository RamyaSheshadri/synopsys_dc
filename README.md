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

