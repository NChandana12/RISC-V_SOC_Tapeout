# ⚡ Day 4: Gate-Level Simulation (GLS), Blocking vs. Non-Blocking, and Synthesis-Simulation Mismatch

Welcome to **Day 4** of the RTL Workshop! 🎉  
Today, we focus on three essential topics in digital design:

- **Gate-Level Simulation (GLS)** 🕹️  
- **Blocking vs. Non-Blocking Assignments in Verilog** 🔄  
- **Synthesis-Simulation Mismatch** ⚠️  

We will cover theory, practical examples, and labs to reinforce your understanding.

---

## 📑 Table of Contents

- [1️⃣ Gate-Level Simulation (GLS)](#1-gate-level-simulation-gls)  
- [2️⃣ Synthesis-Simulation Mismatch](#2-synthesis-simulation-mismatch)  
- [3️⃣ Blocking vs. Non-Blocking Assignments](#3-blocking-vs-non-blocking-assignments)  
  - [3.1 Blocking Statements](#31-blocking-statements)  
  - [3.2 Non-Blocking Statements](#32-non-blocking-statements)  
  - [3.3 Comparison Table](#33-comparison-table)  
- [4️⃣ Labs](#4-labs)  
- [5️⃣ Summary](#5-summary)  

---

## 1️⃣ Gate-Level Simulation (GLS)

**GLS** = **Gate-Level Simulation**. It simulates the **synthesized netlist** to validate:

- ✅ Functional correctness  
- ⏱️ Timing behavior  
- ⚡ Power estimation  
- 🧪 Test structures (scan chains, DFT)

### Why GLS?

- **Synthesis Validation:** Ensure RTL → gates translation is correct  
- **Timing Verification:** Check realistic delays (SDF annotated)  
- **Testability:** Confirm scan chains and other test features  

### When to Perform GLS?

- After synthesis (post-RTL)  
- Before physical design to catch early issues  

### Types of GLS

- **Functional GLS:** Logic-only simulation, zero/unit delays  
- **Timing GLS:** Includes annotated delays to detect timing violations  

---

## 2️⃣ Synthesis-Simulation Mismatch

A mismatch occurs when **RTL simulation** results differ from **gate-level simulation** or hardware. ⚠️  

**Common Causes:**  
- Non-synthesizable code (`initial`, `#delays`, etc.)  
- Incomplete or ambiguous coding (`else` missing, wrong sensitivity list)  
- Tool interpretation differences  

**Tip:** Always write **synthesizable, unambiguous RTL**. ✅

---

## 3️⃣ Blocking vs. Non-Blocking Assignments

Verilog has **two procedural assignment types**:

### 3.1 Blocking Statements (`=`) ⚡

- **Execution:** Sequential, immediate  
- **Use Case:** Combinational logic (`always @(*)`)  
- **Example:**  
```verilog
always @(*) y = a & b;
### 3.2 Non-Blocking Statements (`<=`)

- **Syntax:** `<=`
- **Execution:** Scheduled, executes concurrently at the end of the time step.
- **Suitable for:** Sequential logic (e.g., `always @(posedge clk)`).
- **Example:**  
  ```verilog
  always @(posedge clk) q <= d;
  ```

### 3.3 Comparison Table

| **Blocking (`=`)**                        | **Non-Blocking (`<=`)**                   |
|-------------------------------------------|--------------------------------------------|
| Uses `=` operator                         | Uses `<=` operator                         |
| Sequential, immediate execution           | Concurrent, scheduled at end of timestep   |
| Updates happen instantly in code order    | Updates applied after time step            |
| For combinational logic, temp variables   | For sequential logic, registers/flip-flops |
| Infers combinational logic (gates)        | Infers sequential logic (flip-flops)       |

---

## 4. Labs

### Lab 1: Ternary Operator MUX

Verilog code for a simple 2:1 multiplexer using a ternary operator:

```verilog
module ternary_operator_mux (input i0, input i1, input sel, output y);
  assign y = sel ? i1 : i0;
endmodule
```
- **Function:** `y = i1` if `sel = 1`; else `y = i0`.

![lab1](https://github.com/user-attachments/assets/3f5eb05a-1861-4bb8-940c-6ff9f2af87fb)

---

### Lab 2: Synthesis Using Yosys

Synthesize the above MUX using Yosys.  
_Follow the standard Yosys synthesis flow._

![lab2](https://github.com/user-attachments/assets/7a0cdc7c-cbbd-4943-bd3d-130a0d66b9b1)

---

### Lab 3: Gate-Level Simulation (GLS) of MUX

Run GLS for the synthesized MUX.  
Use this command (adjust paths as needed):

```shell
iverilog /path/to/primitives.v /path/to/sky130_fd_sc_hd.v ternary_operator_mux.v testbench.v
```

![lab3](https://github.com/user-attachments/assets/9acf45b3-2e42-4ac1-88ae-b4a494cc8d87)

---

### Lab 4: Bad MUX Example (Common Pitfalls)

Verilog code with intentional issues:

```verilog
module bad_mux (input i0, input i1, input sel, output reg y);
  always @ (sel) begin
    if (sel)
      y <= i1;
    else 
      y <= i0;
  end
endmodule
```

#### Issues:
- **Incomplete sensitivity list**: Should include `i0`, `i1`, and `sel`.
- **Non-blocking assignment in combinational logic**: Should use blocking assignments (`=`).

**Corrected version:**
```verilog
always @ (*) begin
  if (sel)
    y = i1;
  else
    y = i0;
end
```

![lab4](https://github.com/user-attachments/assets/4c2ede06-0605-4ff0-99cb-fc89844b89e4)

---

### Lab 5: GLS of Bad MUX

Perform GLS on the `bad_mux`.  
Expect simulation mismatches or warnings due to above issues.

![lab5](https://github.com/user-attachments/assets/2e698404-27b5-4c4a-a811-41b5fc13db77)

---

### Lab 6: Blocking Assignment Caveat

Verilog code:

```verilog
module blocking_caveat (input a, input b, input c, output reg d);
  reg x;
  always @ (*) begin
    d = x & c;
    x = a | b;
  end
endmodule
```

#### What’s wrong?
- The order of assignments causes `d` to use the old value of `x`—not the newly computed value.
- **Best Practice:** Assign intermediate variables before using them.

**Corrected order:**
```verilog
always @ (*) begin
  x = a | b;
  d = x & c;
end
```

![lab6](https://github.com/user-attachments/assets/42cac594-0008-4c7b-b415-43e6565b6081)

---

### Lab 7: Synthesis of the Blocking Caveat Module

Synthesize the corrected version of the module and observe the results.

![lab7](https://github.com/user-attachments/assets/833bfacc-3b76-40fa-814c-47f0d783a6e0)

---

## ✅ Summary

Focused on optimization strategies for combinational and sequential circuits

Covered:

Constant Propagation 🔧 – Simplify logic using constants

State Optimization 🧩 – Reduce states & encode efficiently

Cloning ⚡ – Duplicate modules to improve timing/load balance

Retiming ⏱️ – Move flip-flops to improve performance

---

> [!TIP]
>  Always simulate both your RTL and gate-level netlist, and review warnings from synthesis and simulation tools!

---
