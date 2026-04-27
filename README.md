# DSA_Coder_and_robust_solution_optimiser

This project is an end-to-end system designed to solve, test, and optimize Data Structures and Algorithms (DSA) problems using **Llama-3 7B** via the **Ollama** framework. It transforms raw, unstructured problem descriptions into high-performance, validated Python code through a self-correcting feedback loop. Built on the observation that LLM first-pass solutions are rarely optimal, this system goes beyond generation by closing the loop with execution, validation, and iterative repair to produce better complexity outcomes.

---

## 🚀 The Beta Engine: Iterative Solving with Feedback
This pipeline features a sophisticated Iterative Solving engine that acts as a self-correcting developer. Rather than a single "guess," the system ensures a working base solution is established before proceeding to optimization.

* **Automated Infrastructure:** Scripted setup for Ollama and Llama-3 7B within Linux or Google Colab environments.
* **Hybrid Initial Processing:** The system uses a combination of **Regular Expressions (Regex)** and LLMs for initial extraction. Regex handles the deterministic patterns like input/output extraction and code cleaning, ensuring high accuracy before passing structured data to the model.
* **Llama-3 7B Reasoning:** The pipeline currently utilizes the **Llama-3 7B** model. This specific model is targeted for its balance of speed and performance in local environments, especially compared to significantly larger versions like the 70B or 405B models.
* **Execution Sandbox:** Code is executed in a sandboxed environment against sample tests extracted directly from the problem description using specialized Regex patterns.
* **Traceback Feedback:** If the code fails or crashes, the full Python traceback or logic error is captured.
* **The 3-Retry Loop:** The error log is fed back to Llama-3 7B with explicit instructions to fix the bug. The system iterates up to 3 times to reach a stable, passing base solution.

---

## 📂 Component Breakdown

### 🔍 Preprocessing (`structured_preprocess`)
Uses a robust **Regex-first approach** paired with a two-pass LLM cycle. Regex is used to reliably isolate the `class Solution` block and strip UI noise (e.g., "Topics", "Attempted"). The LLM passes then formalize the cleaned description and constraints into a workable JSON format.

### 🧪 Test Case Logic (`generate_all_tests`)
Unlike simple runners, this system generates a **Type-2 Validation Suite**. It uses an LLM to generate JSON-formatted test cases that adhere to a specific schema, ensuring the `input_parsed` data matches the expected function arguments.

### ⚡ The Optimizer (`optimize_solution`)
The optimizer receives the `base_code` and a JSON object containing `failed_cases`. It is instructed to perform a **Complexity Audit** to replace $O(N^2)$ approaches with $O(N \log N)$ or $O(N)$ wherever possible.

### 🎓 Tutor (`llm_compare_solutions`)
This acts as the final stage, providing a comparison between the initial "Naive" solution and the "Optimized" solution. It provides:
* **Complexity Comparison:** Analysis of time and space complexity shifts (e.g., $O(2^n)$ vs $O(n)$).
* **Impact Analysis:** Determining if the change is a "True Optimization" or just a "Refactor."

---

## 🧪 Resilience-Driven Test Generation
Once a base solution is validated against samples, the generator pressures the code with eight distinct testing methodologies:

* **Boundary Testing:** Handles minimum/maximum limits (e.g., $n=0$ or empty arrays).
* **Extreme Value Testing:** Verifies correctness under massive value distributions (e.g., elements at $10^9$).
* **Pattern Testing:** Validates logic on sorted, reversed, or repeating sequences.
* **Random Testing:** Acts as a "chaos monkey" to catch obscure logical gaps through unpredictable inputs.
* **Stress Testing:** Evaluates the solution near maximum constraint sizes ($10^5+$ elements).
* **Deterministic Validation:** Mandates exact mathematical precision against strict expected results.
* **Failure Visibility Testing:** Logs exhaustive execution states to fuel the Iterative Solving engine.
* **Error Classification Testing:** Categorizes failures into logic, runtime, or performance issues.

---

## 🛠 Technical Architecture: Generic & Robust
Built to be truly generic, this architecture avoids the trap of hardcoding for specific problems:

* **Signature-Based Discovery:** Uses Python's `inspect` and `dir` modules to identify the "Main" entry point by analyzing parameter counts and internal dependencies rather than alphabetical picking.
* **Dynamic Input Mapping:** Avoids brittle slicing (like `numbers[:-1]`) by mapping JSON keys directly to function signature requirements.
* **Regex Pattern Matching:** Core extraction of Input/Output samples from problem text is powered by highly tuned Regular Expressions, preventing leakage from "Explanation" blocks.

---

## 🖥 Usage Example
To run the pipeline, provide a `problem_text` variable containing the description and a starter class:

```python
problem_text = """
53. Maximum Subarray
Given an integer array nums, find the subarray with the largest sum.
Example 1: Input: nums = [-2,1,-3,4,-1,2,1,-5,4] Output: 6
...
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
"""
# The pipeline handles the rest
structured_data = structured_preprocess(problem_text)
tests = build_testcases(problem_text)
final_code = full_pipeline(
    base_solution=code,
    description=structured_data["description"],
    constraints=structured_data["constraints"],
    tests=tests,
    validated_tests=generated_tests
)
```

---

## 🛠 Future Improvements
* **Model Scaling:** Currently using the **Llama-3 7B** model, which occasionally struggles with generating all possible edge cases due to possible reasoning limitations. Future versions supporting latest models for more complex reasoning.
* **Type Hint Support:** Integrating `TreeNode` and `ListNode` parsers to support Binary Tree and Linked List problems.
* **Memory Profiling:** Adding `memory_profiler` for empirical space complexity data.
* **Multi-Model Support:** Adding comparison and benchmarking between **Llama3**, **Mixtral**, and **Phi-3**.

---

## ⚠️ Known Limitations
* **Test Case Execution:** Generated test cases occasionally fail to map correctly to function signatures due to structured output limitations at 7B scale. The optimization loop functions independently of this.
* **Problem Scope:** Currently supports standard array, string, and integer problems. Binary Tree and Linked List support is in progress.
---

## ⚠️ Safety & Constraints
* **Execution Safety:** The project uses `exec()` to run LLM-generated code. **Must** be run in a sandboxed environment (like a Colab VM).
* **Deterministic Logic:** All generation tasks are set to `temperature: 0` to ensure logical code output rather than creative prose.
