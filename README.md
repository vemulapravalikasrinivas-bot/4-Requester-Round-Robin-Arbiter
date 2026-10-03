# 4-Requester-Round-Robin-Arbiter
Designed and verified a synthesizable 4-requester Round-Robin Arbiter using Verilog-2001. Implements one-hot grants, fair priority rotation, wrap-around handling, starvation prevention, and asynchronous reset. Includes a self-checking testbench with directed, exhaustive, dynamic, and randomized verification.
# 4-Requester Round-Robin Arbiter

### RTL Design and Verification using Verilog-2001

A synthesizable **4-requester Round-Robin Arbiter** designed to control access to a shared resource while ensuring fair arbitration and preventing requester starvation.

This project was developed as part of the **Maven Silicon – Silicon Sprint Hackathon (VLSI Track)**.

---

## 📌 Project Overview

In systems where multiple requesters need access to a single shared resource, an arbiter determines which requester is granted access.

This project implements a **4-requester round-robin arbitration mechanism**.

The arbiter:

- Accepts four independent request signals.
- Grants access to at most one requester per clock cycle.
- Generates a one-hot grant signal.
- Maintains a rotating priority pointer.
- Skips inactive requesters.
- Supports priority wrap-around.
- Prevents starvation of continuously requesting clients.
- Uses an active-low asynchronous reset.
- Provides a combinational grant based on the current request and priority.
- Updates the priority pointer on the following rising clock edge.

---

## 🎯 Objectives

The primary objective is to design and verify a synthesizable 4-requester round-robin arbiter.

The design must ensure:

1. At most one requester is granted at a time.
2. A requester is granted only when its request is active.
3. The priority rotates after every successful grant.
4. Inactive requesters are skipped.
5. Priority wraps from requester 3 back to requester 0.
6. The priority remains unchanged when there is no valid request.
7. Continuous requests are serviced fairly.
8. No requester experiences starvation.

---

## 🧩 Design Specification

### Module

```verilog
module rr_arbiter (
    input        clk,
    input        rst_n,
    input  [3:0] req,
    output [3:0] grant,
    output       grant_valid
);
```

### Inputs

| Signal | Width | Description |
|---|---:|---|
| `clk` | 1 | Rising-edge clock |
| `rst_n` | 1 | Active-low asynchronous reset |
| `req` | 4 | Request signals from four requesters |

### Outputs

| Signal | Width | Description |
|---|---:|---|
| `grant` | 4 | One-hot grant signal |
| `grant_valid` | 1 | Indicates whether a valid grant exists |

---

## 🔄 Round-Robin Arbitration

The arbiter maintains an internal 2-bit priority pointer.

The pointer determines where the next search begins.

For example:

```text
Priority = 0

Search order:
0 → 1 → 2 → 3
```

If the priority is 2:

```text
Priority = 2

Search order:
2 → 3 → 0 → 1
```

The first active requester encountered in the circular search receives the grant.

---

## 🔁 Priority Update

After a successful grant, priority moves to the requester immediately following the granted requester.

| Granted Requester | Grant | Next Priority |
|---|---|---:|
| Requester 0 | `0001` | 1 |
| Requester 1 | `0010` | 2 |
| Requester 2 | `0100` | 3 |
| Requester 3 | `1000` | 0 |

If there is no valid request:

```text
grant       = 0000
grant_valid = 0
priority    = unchanged
```

---

## 🏗️ Architecture

The design consists of two major components:

```text
                  +-----------------------------+
                  |                             |
req[3:0] -------->| Combinational Arbitration  |
                  | Logic                       |
                  |                             |
priority_ptr ---->| Circular Priority Search   |
                  |                             |
                  +--------------+--------------+
                                 |
                       +---------+---------+
                       |                   |
                    grant             grant_valid
                       |
                       v
                  next_priority
                       |
                       v
              +-------------------+
clk --------->| Priority Register |
rst_n -------->|                   |
              +---------+---------+
                        |
                        |
                  priority_ptr
```

### Combinational Logic

The combinational section:

- Starts searching from `priority_ptr`.
- Checks requesters in circular order.
- Selects the first active requester.
- Generates a one-hot `grant`.
- Generates `grant_valid`.
- Calculates `next_priority`.

### Sequential Logic

The sequential priority register:

- Uses the rising edge of `clk`.
- Uses an active-low asynchronous reset.
- Resets priority to requester 0.
- Updates priority only when a valid grant occurs.

---

## 🧠 Internal Signals

| Signal | Width | Purpose |
|---|---:|---|
| `priority_ptr` | 2 | Current starting priority |
| `next_priority` | 2 | Priority after successful grant |
| `search_ptr` | 2 | Current requester being searched |
| `found` | 1 | Indicates that a requester has already been granted |
| `i` | integer | Loop counter |

---

## ⚙️ Example Arbitration

Suppose:

```text
req = 4'b1011
```

Therefore:

```text
Requester 3 = active
Requester 2 = inactive
Requester 1 = active
Requester 0 = active
```

If:

```text
priority = 0
```

the search is:

```text
0 → 1 → 2 → 3
```

Requester 0 is active, so:

```text
grant = 4'b0001
```

The next priority becomes:

```text
priority = 1
```

On the next arbitration cycle, the search begins from requester 1.

---

## 🔄 Wrap-Around

The arbiter supports circular priority.

For example:

```text
3 → 0 → 1 → 2 → 3 ...
```

If:

```text
priority = 3
req = 4'b1001
```

the search order is:

```text
3 → 0 → 1 → 2
```

The arbiter grants requester 3 first because it is active.

After requester 3 is served:

```text
next priority = 0
```

This provides continuous round-robin operation.

---

# 🧪 Verification

A self-checking Verilog testbench was developed to verify the RTL.

The testbench contains:

- A cycle-accurate reference model.
- Arbitration checking.
- Grant legality checking.
- Reset verification.
- Priority tracking.
- Directed tests.
- Exhaustive testing.
- Dynamic request testing.
- Randomized regression testing.

The testbench compares the DUT outputs against an independent reference model after each stimulus.

---

## 🔍 Verification Checks

The verification environment checks:

### 1. Arbitration correctness

Detects:

- Incorrect requester selection.
- Incorrect round-robin ordering.
- Incorrect priority handling.
- Incorrect wrap-around behavior.

### 2. Grant legality

Ensures:

- An inactive requester is never granted.
- Grant is one-hot.
- `grant_valid = 1` only when a grant exists.
- `grant_valid = 0` when `grant = 0000`.

### 3. Reset behavior

During reset:

```text
grant       = 0000
grant_valid = 0
priority    = 0
```

### 4. Priority hold

When there is no valid grant, the priority pointer must remain unchanged.

---

# 🧪 Test Scenarios

The testbench includes the following scenarios:

| Test | Description |
|---:|---|
| 1 | Reset behavior |
| 2 | No request |
| 3 | Single requester tests |
| 4 | Multiple simultaneous requests |
| 5 | All four requesters continuously asserted |
| 6 | Partial fairness |
| 7 | Request withdrawal |
| 8 | Priority hold |
| 9 | Priority wrap-around |
| 10 | Exhaustive priority/request testing |
| 11 | Dynamic request patterns |
| 12 | Randomized request patterns |

---

## 🔬 Exhaustive Verification

The testbench performs exhaustive testing of:

```text
4 priority states × 16 request combinations
```

giving:

```text
4 × 16 = 64
```

priority/request combinations.

This verifies the arbitration behavior across all possible priority states and 4-bit request patterns.

---

## 🎲 Randomized Verification

The testbench additionally applies:

```text
100 randomized request patterns
```

This provides regression coverage for changing request combinations that may not be explicitly included in the directed tests.

---

# ▶️ Simulation

The design can be simulated using **Icarus Verilog**.

### Compile

```bash
iverilog -o sim rtl/rr_arbiter.v tb/tb_rr_arbiter.v
```

### Run

```bash
vvp sim
```

## 📊 Expected Simulation Output

The testbench produces a final report containing:

```text
=================================================
 FINAL REPORT
=================================================
Total checks : 441  
Passed       : 441
Failed       : 0
=================================================
```

When all checks pass:

```text
===============================================
 VERIFICATION PASSED
===============================================


# 📁 Repository Structure

```text
4-Requester-Round-Robin-Arbiter/
│
├── README.md
│
├── rtl/
│   └── rr_arbiter.v
│
├── tb/
│   └── tb_rr_arbiter.v
│
├── schematic/
│   └── schematic_silicon.pdf
│ 
├── simulation/
│   └── DUT(s) results.pdf
```

---

# 🛠️ Tools and Technologies

- Verilog-2001
- RTL Design
- Digital Design
- Round-Robin Arbitration
- Functional Verification
- Self-Checking Testbench
- Icarus Verilog
- VLSI Design and Verification

---

# 📚 Key Concepts Demonstrated

This project demonstrates practical understanding of:

- Combinational RTL design
- Sequential logic
- Asynchronous reset
- Priority arbitration
- Round-robin scheduling
- One-hot encoding
- Circular priority logic
- Starvation prevention
- Fairness
- Reference-model-based verification
- Directed testing
- Exhaustive testing
- Randomized testing
- Self-checking testbenches
- Synthesizable Verilog coding

---

# 👥 Team

### Team DUT

**Team Leader:** Pravalika

**Team Members:**

- Srinath
- Rashmitha
- Naveen

**Event:** Maven Silicon – Silicon Sprint Hackathon

**Track:** VLSI

**Language:** Verilog-2001



## ⭐ Project Summary

A synthesizable 4-requester round-robin arbiter was implemented using Verilog-2001. The design provides one-hot grants, circular priority rotation, wrap-around handling, asynchronous reset, and starvation-free arbitration.

A self-checking verification environment was developed using a reference model, directed tests, exhaustive request/priority combinations, dynamic patterns, and randomized regression testing.
