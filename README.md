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

### 💻 Prerequisites

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
190/
├── half_adder.v # Verilog HDL for Half Adder
├── synthesize.tcl # TCL script to run synthesis
├── README.md # Project documentation
├── output/ # (Auto-generated synthesis reports, logs)


---

## 🧠 Verilog Code

Here's a basic half adder module in Verilog:
```
module ha(input a,b, output s,c);
assign s=a^b;
assign c=a&b;
endmodule
```
