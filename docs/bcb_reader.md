# bcb_reader.py

## Overview

The `bcb_reader.py` module is responsible for loading and preprocessing the BigCodeBench dataset. It fetches the raw dataset, specifically version `v0.1.4` of `bigcode/bigcodebench` (likely from the Hugging Face Datasets Hub), and transforms each problem into a standardized format. This preprocessing involves extracting the core problem statement, constructing the complete canonical solution, and isolating the test code. The output format is designed for convenient use by other components within the BigCodeBench MultiPL framework, particularly for tasks involving code generation, translation, or evaluation.

## Key Components

### TypedDicts

- **`_OriginalBigCodeBenchProblem(TypedDict)`**:
  - Defines the expected structure of a single problem item as loaded directly from the `bigcode/bigcodebench` dataset source.
  - Fields:
    - `task_id: str`: Unique identifier for the task.
    - `complete_prompt: str`: The entire prompt given to a base model.
    - `instruct_prompt: str`: A prompt that includes instructions and a prefix of the solution.
    - `canonical_solution: str`: The remainder of the canonical solution code.
    - `code_prompt: str`: A prompt specifically for code generation.
    - `test: str`: String containing the test code for the problem.
    - `entry_point: str`: The entry point for the function/method to be implemented.
    - `doc_struct: str`: (Purpose not detailed in this script).
    - `libs: str`: (Purpose not detailed in this script, likely library dependencies).
  - *Note*: The script mentions that "BigCodeBench-Hard has a few extra columns," implying this TypedDict is for the standard version.

- **`BigCodeBenchProblem(TypedDict)`**:
  - Defines the structure of a problem *after* preprocessing by this script. This is the format intended for downstream consumption.
  - Fields:
    - `task_id: str`: Unique identifier for the task (same as original).
    - `problem: str`: The extracted natural language problem description/instruction.
    - `solution: str`: The complete canonical solution code, constructed by combining parts of the `instruct_prompt` and `canonical_solution`.
    - `tests: str`: The test code for the problem (taken directly from `test` field of original).

### Constants

- **`_PROMPT_BOILERPLATE: str`**:
  - Value: `"\nYou should write self-contained code starting with:\n```\n"`
  - Purpose: Used as a delimiter to split the `instruct_prompt` field of the raw dataset. It separates the problem description part from the beginning of the solution code prefix.

- **`_PROMPT_SUFFIX: str`**:
  - Value: `"```"`
  - Purpose: Represents the expected suffix of the solution prefix found in `instruct_prompt`. It's removed after splitting.

### Functions

- **`_prepare_bcb_problem(item: _OriginalBigCodeBenchProblem) -> BigCodeBenchProblem`**:
  - **Purpose**: Transforms a raw problem item from the BigCodeBench dataset into the standardized `BigCodeBenchProblem` format.
  - **Input**: `item` - an `_OriginalBigCodeBenchProblem` dictionary.
  - **Processing**:
    1. Takes the `instruct_prompt` from the input `item`.
    2. Splits `instruct_prompt` using `_PROMPT_BOILERPLATE` to separate the problem statement (`problem`) from the initial part of the solution (`solution_prefix`).
    3. Validates that `solution_prefix` ends with `_PROMPT_SUFFIX` and then removes this suffix.
    4. Constructs the full `solution` by concatenating the processed `solution_prefix` with the `canonical_solution` field from the input `item`.
    5. Assigns the `test` field from the input `item` directly to the `tests` field of the output.
    6. **Sanity Check**: Uses `ast.parse()` to attempt to parse both the generated `solution` string and the `tests` string. This helps ensure that the extracted code is syntactically valid Python. If parsing fails, an `SyntaxError` will be raised.
  - **Output**: A `BigCodeBenchProblem` dictionary containing `task_id`, `problem`, `solution`, and `tests`.

- **`load_bigcodebench() -> Generator[BigCodeBenchProblem, None, None]`**:
  - **Purpose**: The main public interface of the module. Loads the BigCodeBench dataset and yields processed problems one by one.
  - **Processing**:
    1. Uses `datasets.load_dataset("bigcode/bigcodebench", split="v0.1.4")` to load the specified version of the dataset.
    2. Iterates through each raw problem item (`_OriginalBigCodeBenchProblem`) in the loaded dataset.
    3. For each item, it calls `_prepare_bcb_problem()` to convert it into the `BigCodeBenchProblem` format.
    4. Yields the processed `BigCodeBenchProblem`.
  - **Output**: A generator that yields `BigCodeBenchProblem` dictionaries.

## Important Variables/Constants

- **Dataset Identifier**: The script is hardcoded to use `"bigcode/bigcodebench"` as the dataset name and `"v0.1.4"` as the specific split/version. Changes to these would require modifying `load_bigcodebench()`.
- **Prompt Structure**: The parsing logic in `_prepare_bcb_problem` heavily relies on the specific structure of `instruct_prompt` and the presence of `_PROMPT_BOILERPLATE` and `_PROMPT_SUFFIX` as delimiters.

## Usage Examples

```python
from bcb_reader import load_bigcodebench

# Iterate through the processed problems from BigCodeBench
problem_count = 0
for problem_data in load_bigcodebench():
    print(f"--- Task ID: {problem_data['task_id']} ---")
    print(f"Problem Description (first 100 chars): {problem_data['problem'][:100]}...")
    # print(f"Canonical Solution:\n{problem_data['solution']}")
    # print(f"Test Code:\n{problem_data['tests']}")
    problem_count += 1
    if problem_count >= 2: # Example: Stop after processing two problems
        break

print(f"\nProcessed {problem_count} problems.")
```

## Dependencies and Interactions

- **External Libraries**:
  - `datasets`: From Hugging Face (`huggingface_hub` package). This is essential for loading the `bigcode/bigcodebench` dataset.
  - `ast`: (Python standard library) Used for performing syntax checks on the Python code extracted for solutions and tests.
  - `typing`: (Python standard library) Utilized for `TypedDict` to define structured dictionary types and `Generator` for type hinting the output of `load_bigcodebench`.

- **Dataset Source**:
  - The script directly depends on the availability and structure of the `bigcode/bigcodebench` dataset, version `v0.1.4`, hosted on the Hugging Face Hub or available in a local Hugging Face datasets cache.

- **Downstream Consumption**:
  - The `BigCodeBenchProblem` dictionaries produced by `load_bigcodebench()` are intended to be consumed by other modules within the BigCodeBench MultiPL project for tasks such as model fine-tuning, code generation, program translation, and automated evaluation.
```
