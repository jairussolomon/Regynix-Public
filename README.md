# Regynix

## Cadence Genus Retiming Synthesis Flow

### Lead Engineer
Jairus Samraj Solomon

---

# Detailed Flow Explanation

## What This Project Is Doing

This project demonstrates a **Cadence Genus retiming synthesis flow**.

The goal of the project is to:

- Read an RTL Verilog design (`testdesign.v`)
- Apply timing constraints (`testdesign.sdc`)
- Enable **retiming optimization**
- Synthesize the design through multiple stages
- Optimize timing and QoR (Quality of Results)
- Generate reports for timing, area, gates, and optimization results

The important concept here is **retiming**.

Retiming is an optimization technique where synthesis tools move flip-flops/registers across combinational logic to:

- Reduce critical path delay
- Improve timing closure
- Increase achievable clock frequency
- Balance logic between pipeline stages

Instead of manually changing RTL pipelines, Genus automatically relocates registers during synthesis.

---

# Important Files in the Project

## RTL Design

### `testdesign.v`

Contains the Verilog RTL design.

This is the actual hardware design being synthesized.

---

## Timing Constraints

### `Constraints/testdesign.sdc`

Contains:

- Clock definitions
- Input delays
- Output delays
- Timing exceptions
- Design constraints

The synthesis tool uses these constraints to optimize timing.

---

## Technology Libraries

### `LIB/*.lib`

Standard cell timing libraries.

Examples:

- `slow.lib`
- `pll.lib`
- `CDK_S128x16.lib`
- `CDK_S256x16.lib`
- `CDK_R512x16.lib`

These contain:

- Cell delays
- Power information
- Area information
- Timing arcs
- Sequential cell models

Genus maps generic logic into actual technology cells using these libraries.

---

# Step-by-Step Explanation of `run.tcl`

---

# 1. CPU Information Check

```tcl
if {[file exists /proc/cpuinfo]} {
  sh grep "model name" /proc/cpuinfo
  sh grep "cpu MHz"    /proc/cpuinfo
}
puts "Hostname : [info hostname]"
```

## What It Does

This section prints:

- CPU model
- CPU frequency
- Machine hostname

## Why It Is Used

Useful for:

- Logging runtime environment
- Debugging performance
- Comparing synthesis runtime on different machines

This step does not affect synthesis.

---

# 2. Runtime Statistics Start

```tcl
timestat START
```

## What It Does

Starts runtime tracking.

---

# 3. Library Setup

```tcl
set_attribute library { ./LIB/slow.lib ./LIB/pll.lib  ./LIB/CDK_S128x16.lib  ./LIB/CDK_S256x16.lib  ./LIB/CDK_R512x16.lib} /
```

## What It Does

Loads the technology libraries.

The `.lib` files provide timing, power, and area information required for synthesis.

---

# 4. Define Top Module

```tcl
set design testdesign
```

Defines the top-level module for synthesis.

---

# 5. Synthesis Effort Levels

```tcl
if ![info exists SYN_EFF] {set SYN_EFF medium}
if ![info exists MAP_EFF] {set MAP_EFF medium}
if ![info exists INC_EFF] {set INC_EFF medium}
if ![info exists PHYS_EFF] {set PHYS_EFF medium}
```

Controls optimization aggressiveness and runtime tradeoffs.

---

# 6. Report and Output Directories

```tcl
set LOG_PATH     logs/logs
set OUTPUTS_PATH outputs/outputs
set REPORTS_PATH reports/reports
```

Creates organized directories for logs, outputs, and reports.

---

# 7. HDL Array Naming Style

```tcl
set_attribute hdl_array_naming_style %s\[%d\] /
```

Improves readability of synthesized signal names.

---

# 8. Enable TNS Optimization

```tcl
set_attribute tns_opto true /
```

Optimizes Total Negative Slack (TNS) to reduce overall timing violations.

---

# 9. Enable Retiming Debug

```tcl
set_attr retime_debug 1 /
set_attr retime_debug_path 1 /
```

Enables detailed retiming debug information.

---

# 10. Read RTL Files

```tcl
read_hdl -v2001 testdesign.v
```

Reads the Verilog RTL into Genus.

---

# 11. Elaborate Design

```tcl
elaborate $design
```

Builds the internal design hierarchy and logic representation.

---

# 12. Check Design

```tcl
check_design
```

Checks for RTL and connectivity issues.

---

# 13. Read SDC Constraints

```tcl
read_sdc Constraints/${design}.sdc
```

Loads timing constraints used during optimization.

---

# 14. Timing Lint Reports

```tcl
report timing -lint -verbose
```

Checks timing setup quality and constraint issues.

---

# 15. Critical Timing Paths Report

```tcl
report timing -endpoints -num_paths 10
```

Reports the worst timing paths in the design.

---

# 16. Enable Retiming

```tcl
set_attr retime true /des*/*
```

The most important step in the project.

Allows Genus to move registers across combinational logic to improve timing.

---

# 17. Reset Retiming Optimization

```tcl
set_attr retime_optimize_reset true /
```

Allows retiming even when reset logic exists.

---

# 18. Generic Synthesis (`syn_gen`)

```tcl
syn_gen
```

Converts RTL into generic technology-independent logic.

---

# 19. Generic Reports

```tcl
generate_reports -outdir $REPORTS_PATH -tag gen
```

Generates generic synthesis timing, area, and QoR reports.

---

# 20. Summary Table

```tcl
summary_table -outdir $REPORTS_PATH
```

Creates overall QoR summaries.

---

# 21. Generic Timing Report

```tcl
report timing -endpoints -num_paths 10 > TimingPath_Gen.rpt
```

Shows timing after generic synthesis.

---

# 22. Save Generic Database

```tcl
write_db -to_file ${design}_generic.db
```

Saves the intermediate Genus database.

---

# 23. Technology Mapping (`syn_map`)

```tcl
syn_map
```

Maps generic logic into actual standard cells from the technology library.

---

# 24. Mapping Reports

Generated using:

```tcl
generate_reports -tag map
```

Includes area, timing, and QoR reports after mapping.

---

# 25. Incremental Optimization (`syn_opt`)

```tcl
syn_opt
```

Performs final timing and QoR optimization.

---

# 26. Final Timing Reports

```tcl
report timing -endpoints -num_paths 10 > TimingPath_Opt.rpt
```

Shows the final optimized timing paths.

---

# 27. Save Final Database

```tcl
write_db -to_file ${design}_incr.db
```

Saves the final optimized design database.

---

# Understanding the Final Reports

## `gen_qor.rpt`

QoR report after generic synthesis.

Contains:

- Slack
- TNS
- Area estimates
- Cell count

---

## `map_qor.rpt`

QoR after technology mapping using real standard cells.

---

## `TimingPath_Opt.rpt`

Most important final timing report.

Shows:

- Critical paths
- Slack
- Arrival time
- Required time

---

## `map_area.rpt`

Shows total synthesized area.

---

## `map_gates.rpt`

Shows gate usage statistics and cell counts.

---

# What Retiming Improved

Retiming likely:

- Reduced critical path delay
- Improved slack
- Balanced pipeline stages
- Increased achievable clock frequency

---

# Overall Flow Summary

```text
RTL Verilog
     ↓
Read HDL
     ↓
Elaboration
     ↓
Constraint Loading
     ↓
Design Checks
     ↓
Enable Retiming
     ↓
Generic Synthesis (syn_gen)
     ↓
Technology Mapping (syn_map)
     ↓
Incremental Optimization (syn_opt)
     ↓
Timing/Area/QoR Reports
     ↓
Final Optimized Netlist Database
```

---

# Final Understanding of the Project

Regynix demonstrates how Cadence Genus performs retiming-based optimization during RTL synthesis.

The project showcases:

- RTL synthesis flow
- Constraint-driven optimization
- Register retiming
- Timing closure improvement
- Technology mapping
- QoR analysis

The main educational objective is understanding how automatic retiming improves timing performance without manually modifying RTL pipelines.
