# DSA Coder & Robust Solution Optimiser

This project is an automated, end-to-end pipeline designed to solve, test, and optimise Data Structures and Algorithms (DSA) problems using **Llama-3 8B** via the **Ollama** framework. It transforms raw, unstructured problem descriptions into high-performance, validated Python code through a self-correcting feedback loop.

---

## 🚀 The Pipeline: Iterative Solving with Feedback

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

## 📂 Component Breakdown

### 🔍 Preprocessing (`structured_preprocess`)
Uses a **Regex-first approach** paired with a two-pass LLM cycle. Regex strips UI noise (e.g., "Topics", "Attempted", "premium lock icon") and reliably isolates the `class Solution` block. Two focused LLM calls then handle description and constraints separately — each pass has a single responsibility, preventing the hallucination and field-bleed that plagued single-prompt extraction.

### 🧪 Test Case Logic (`build_testcases`)
A boundary-aware regex pattern extracts Input/Output pairs directly from the problem's hand-written examples. The pattern terminates capture at `Explanation:`, `Constraints:`, or the next `Example` marker, so explanation text never leaks into expected values.

### ⚡ The Solver (`solve_with_retry`)
The LLM generates an initial `class Solution` which is immediately executed in a sandboxed environment against the extracted sample tests. On failure, the full Python traceback is captured and fed back with explicit fix instructions. The system retries up to **3 times** and keeps the attempt with the highest pass rate as the base solution.

### 🧬 Stress Test Generator (`generate_all_tests`)
The core of this stage is **Hypothesis**, Python's property-based testing library. Rather than relying solely on the LLM to imagine edge cases, Hypothesis takes the extracted `problem_text`, the parsed `constraints`, and the validated `sample_testcases` as its inputs — and uses them to infer the shape, bounds, and types of valid inputs for the problem. From that, it automatically generates a large, diverse set of inputs that probe the solution's behaviour as a property rather than a fixed expected output. The LLM then supplements this with semantically meaningful edge cases that require understanding the problem's intent. Every candidate test passes through a validation gate that checks for required fields, forbidden expressions, structural consistency, and duplicates.

### ⚡ The Optimiser (`optimize_solution`)
Receives the base solution and any failing stress tests as evidence. Instructs the LLM to perform a **Complexity Audit** — replacing O(N²) patterns with O(N log N) or O(N) equivalents such as two-pointer, sliding window, binary search, or hash-based lookups.

### 🎓 Tutor (`llm_compare_solutions`)
The final stage compares the naive and optimised solutions:
* **Complexity Comparison:** Analysis of time and space complexity shifts (e.g., O(2ⁿ) vs O(n)).
* **Impact Analysis:** Determining if the change is a **True Optimisation** (asymptotic improvement) or a **Refactor** (equivalent complexity, cleaner code).

---

## 🧪 Resilience-Driven Test Generation

Once the base solution is validated against sample tests, the generator pressures it with eight distinct methodologies:

* **Boundary Testing:** Handles minimum/maximum limits (e.g., n=0, empty arrays, single elements).
* **Extreme Value Testing:** Verifies correctness under very large or uniform value distributions (e.g., elements at 10⁹).
* **Pattern Testing:** Validates logic on sorted, reversed, or repeating sequences.
* **Random Testing:** Acts as a "chaos monkey" to catch obscure logical gaps through unpredictable inputs.
* **Stress Testing:** Evaluates the solution near maximum constraint sizes (10⁵+ elements).
* **Deterministic Validation:** Mandates exact mathematical precision against strict expected results.
* **Failure Visibility Testing:** Logs exhaustive execution states to fuel the iterative repair engine.
* **Error Classification Testing:** Categorises failures into logic, runtime, or performance issues.

---

## 🛠 Technical Architecture: Generic & Robust

Built to be truly generic, this architecture avoids the trap of hardcoding for specific problems:

* **Signature-Based Discovery:** Uses Python's `inspect` and `dir` modules to identify the correct entry point by matching argument count, then deprioritises helper methods (`dfs`, `bfs`, `helper`, `util`) so the main solver is always invoked.
* **Dynamic Input Mapping:** Maps parsed variable names directly to function signature parameters rather than positional slicing, handling any number of arguments at any type correctly.
* **Regex Pattern Matching:** Core extraction of Input/Output samples uses boundary-aware patterns that prevent leakage from Explanation blocks.
* **Property-Based Testing via Hypothesis:** Stress inputs are generated from constraint definitions rather than hardcoded lists, producing a wider and more principled coverage surface.

---

## 🔧 Bottlenecks Addressed from the Previous Version

### Input Parsing Was Brittle
The old approach sliced a flat number list positionally, silently breaking on any problem with a different argument shape. The parser now walks the input respecting bracket depth and maps each variable by name directly to the function's parameter list.

### Explanation Blocks Leaked into Test Outputs
The original regex over-captured, pulling explanation text into expected output fields. A boundary condition now terminates extraction at the first occurrence of `Explanation:`, `Constraints:`, or the next example marker.

### In-Place Functions Always Failed
Problems that modify their input and return `None` (like Next Permutation) were always logged as failures. The runner now detects a `None` return and falls back to comparing the mutated argument state instead.

### Generated Tests Were Structurally Invalid
The LLM occasionally produced test cases containing Python expressions instead of literal values, or with missing fields. A validation layer now rejects such tests and retries, with a soft fallback to raw output rather than silently dropping a test category.

### Preprocessing Hallucinated Constraints
A single-prompt extraction pass caused constraint hallucination and field bleed. Three isolated passes now handle description, constraints, and code separately — the code pass uses regex only, with no LLM involvement.

---

## 🖥 Usage

Paste any LeetCode-style problem into `problem_text` at the top of the notebook and run all cells:

```
problem_text = """
53. Maximum Subarray
Given an integer array nums, find the subarray with the largest sum, and return its sum.
Example 1: Input: nums = [-2,1,-3,4,-1,2,1,-5,4] Output: 6
Example 2: Input: nums = [1] Output: 1
Constraints: 1 <= nums.length <= 10^5, -10^4 <= nums[i] <= 10^4
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
"""
```

The pipeline handles everything from there — preprocessing, solving, stress testing, optimising, and reporting.

---

## 🛠 Future Improvements

* **Model Scaling:** Evaluating larger reasoning models (34B / 70B) for harder edge case generation and optimisation quality.
* **Type Hint Support:** Integrating `TreeNode` and `ListNode` parsers to support Binary Tree and Linked List problems.
* **Memory Profiling:** Adding `memory_profiler` for empirical space complexity data alongside the theoretical audit.
* **Multi-Model Support:** Adding comparison and benchmarking between **Llama3**, **Mixtral**, and **Phi-3**.

---

## ⚠️ Known Limitations

* **Test Case Execution:** Generated test cases occasionally fail to map correctly to function signatures due to structured output limitations at 8B scale. The optimisation loop functions independently of this.
* **Problem Scope:** Currently supports standard array, string, and integer problems. Binary Tree and Linked List support is in progress.

---

## ⚠️ Safety & Constraints

* **Execution Safety:** The project uses `exec()` to run LLM-generated code. **Must** be run in a sandboxed environment (like a Colab VM).
* **Deterministic Logic:** All generation tasks are set to `temperature: 0` to ensure logical code output rather than creative prose.
