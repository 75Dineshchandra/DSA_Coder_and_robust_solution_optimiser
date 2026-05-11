# 🧠 DSA Coder & Robust Solution Optimiser

An end-to-end automated pipeline that transforms raw Data Structures & Algorithms (DSA) problem statements into **validated, stress-tested, and iteratively improved Python solutions** using Llama 3 (via Ollama) and property-based testing. This is a working prototype built for experimentation and system design exploration of LLM-driven code generation and testing pipelines.

The system combines:
- LLM-based code generation
- Symbolic + regex-based parsing
- Sandboxed execution
- Property-based testing (Hypothesis)
- Iterative repair loops
- Complexity-aware solution comparison

---

## 🚀 High-Level Pipeline

```
Raw Problem Text
      │
      ▼
┌──────────────────────────┐
│  structured_preprocess   │  Regex + 2-pass LLM → description, constraints, starter code
└──────────────────────────┘
      │
      ▼
┌──────────────────────────┐
│     build_testcases      │  Regex extracts Example Input/Output pairs from problem text
└──────────────────────────┘
      │
      ▼
┌──────────────────────────┐
│    solve_with_retry      │  LLM generates solution → exec sandbox → traceback → retry (×3)
└──────────────────────────┘
      │
      ▼
┌──────────────────────────┐
│   generate_all_tests     │  Hypothesis + LLM: 8-type property-based stress suite
└──────────────────────────┘
      │
      ▼
┌──────────────────────────┐
│    optimize_solution     │  Complexity audit: O(N²) → O(N log N) / O(N)
└──────────────────────────┘
      │
      ▼
┌──────────────────────────┐
│  llm_compare_solutions   │  Naive vs Optimised: complexity shift + "True Optimization" verdict
└──────────────────────────┘
      │
      ▼
  Final Validated Code + Tutor Report
```


---

## 🧩 Core Components

### 1. Structured Preprocessing
Uses a hybrid approach:
- Regex removes UI noise (e.g., “Topics”, “Attempted”, formatting artifacts)
- LLM extracts:
  - Problem description
  - Constraints
- Code extraction is fully regex-driven (no hallucination risk)

This ensures clean separation of reasoning inputs.

---

### 2. Sample Test Extraction
Extracts `Input → Output` pairs from problem statements using boundary-aware regex.

Key design rule:
- Stops capture at:
  - `Explanation`
  - `Constraints`
  - next `Example`

This prevents contamination of expected outputs.

---

### 3. Solver with Retry Loop
LLM generates initial solution:
- Executed in a sandbox (`exec`)
- Validated against sample tests
- On failure:
  - Full traceback is fed back to LLM
  - System retries up to 3 attempts
- Best-performing version is retained

This forms a **self-correcting generation loop**.

---

### 4. Execution Engine
Handles:
- Dynamic function discovery
- Class/method resolution (`Solution.method`)
- In-place mutation handling (important for problems like *Next Permutation*)
- Robust input parsing with bracket-depth tracking

---

### 5. Property-Based Stress Testing (Hypothesis)

Uses :contentReference[oaicite:0]{index=0} to generate structured random inputs derived from constraints.

Instead of relying only on fixed test cases, the system:
- Infers input domains from constraints
- Generates randomized edge cases
- Validates correctness as a **property**, not just examples

Test types include:
- Boundary cases
- Randomized stress inputs
- Extreme values
- Pattern-based sequences
- Adversarial distributions

This improves robustness beyond typical LeetCode-style testing.

---

### 6. Constraint Parsing Engine
Parses constraints into structured formats:
- Array size limits
- Value bounds
- Implicit defaults when missing

Used to guide Hypothesis strategy generation.

---

### 7. Optimisation Module
Second LLM pass performs:
- Complexity analysis
- Detection of inefficiencies (e.g., O(n²))
- Suggestion of optimized patterns:
  - Sliding window
  - Two pointers
  - Hash maps
  - Binary search

---

### 8. Solution Comparator
Compares:
- Base solution
- Optimised solution

Evaluates:
- Correctness equivalence
- Time complexity shift
- Space complexity impact

Classifies result as:
- **True Optimization** → asymptotic improvement
- **Refactor Only** → same complexity, cleaner code
- **Regression Risk** → correctness or performance issues

---

## 🧠 Design Philosophy

This project is built on three principles:

### 1. Separation of Concerns
- Parsing ≠ solving ≠ testing ≠ optimisation
- Each stage is independently verifiable

### 2. Deterministic Execution First
LLM output is never trusted directly:
- Always validated via execution
- Always stress-tested

### 3. Property over Example Testing
Example-based validation is insufficient for DSA systems.
This pipeline prioritizes:
- Constraint-driven input generation
- Property-based correctness checks

---

## ⚙️ Architecture Highlights

- AST-based signature extraction
- Dynamic function resolution
- Safe execution sandbox (`exec` isolation)
- Retry-based self-healing generation loop
- Hybrid regex + LLM preprocessing
- Constraint-to-strategy compiler for test generation

---

## 📦 Current Scope

Supported problem types:
- Arrays
- Strings
- Integers
- Standard LeetCode-style functions

Partially supported / experimental:
- In-place mutation problems
- Multi-parameter signatures

Not yet supported:
- Trees (`TreeNode`)
- Linked lists (`ListNode`)
- Graph problems with complex state

---

## 📈 If Scaled Further, What This Becomes

At larger scale, this system can evolve into:

### 1. Automated Interview Problem Solver
- End-to-end coding interview assistant
- Real-time solution generation + validation

### 2. LLM Code Reliability Benchmarking Engine
- Compare models (Llama, GPT, Mixtral, etc.)
- Measure:
  - correctness
  - robustness
  - optimization quality

### 3. AI Coding Tutor System
- Explains:
  - why solution works
  - where inefficiency exists
  - how to improve complexity

### 4. Synthetic Dataset Generator for DSA Training
- Generates:
  - edge-case datasets
  - adversarial test suites
  - constraint-based benchmarks

### 5. Agentic Code Optimization Framework
- Multi-agent system:
  - Solver agent
  - Tester agent
  - Adversarial test generator
  - Complexity critic

---

## ⚠️ Known Limitations

- Uses `exec()` → must run in sandboxed environment (Colab/isolated VM)
- Hypothesis generation depends on constraint parsing accuracy
- LLM may occasionally produce structurally valid but suboptimal solutions
- Tree/graph parsing not fully implemented

---

## 🔒 Safety Note

All generated code is executed dynamically:
- Never run outside a controlled environment
- No external system access is required
- No file/network operations are used by default

---

## 🧭 Summary

This is not just a “DSA solver”.

It is a **closed-loop system for generating, testing, and improving algorithmic code using LLMs + property-based validation.**

The project reflects how I approached the problem of “just making code work” and moved toward understanding why it works and where it breaks. While building it, I had to strengthen my Python and data structure fundamentals in a practical way instead of isolated problem solving.

It also made me realize that in a future where basic code generation is automated, the real value is not in writing boilerplate solutions, but in validating, refining, and debugging those outputs using a strong grasp of system design, constraints, and core computational thinking.

