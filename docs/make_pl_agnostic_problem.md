# make_pl_agnostic_problem.py

## Overview

The `make_pl_agnostic_problem.py` script is a command-line tool that leverages language models (LMs) through the DSPy library to transform Python-specific programming problem statements into a more programming language-agnostic format. The primary goal is to remove Python-centric terminology, library references, and concepts while strictly preserving the problem's input/output behavior, as this is critical for existing test cases.

The script processes a dataset of problems (provided as a JSONL file), applies the transformation using a specified LM, and outputs a new JSONL file with the modified problem statements. Other fields from the input dataset are retained in the output.

## Key Components

### DSPy Signature and Module

-   **`PythonStdioToAgnostic(dspy.Signature)`**:
    -   Defines the task for the language model.
    -   **Instruction/Prompt**: "I will give you a problem statement for a Python programming problem, but I want to solve this problem in any programming language. Update the problem statement to remove references to Python, Python libraries, Python concepts, and Python implementation details. However, keep the description of the input and output identical. If the I/O description refers to reading or printing Python values, leave those unchanged. I have test-cases that rely on the I/O behavior being identical and I do not want to change them."
    -   **InputFields**:
        -   `python_problem: str`: The original problem statement containing Python-specific details.
    -   **OutputFields**:
        -   `neutral_problem: str`: The rewritten problem statement, intended to be language-neutral.

-   **`python_stdio_to_agnostic = dspy.ChainOfThought(PythonStdioToAgnostic)`**:
    -   A DSPy module created from the `PythonStdioToAgnostic` signature. Using `dspy.ChainOfThought` encourages the LM to generate a more reasoned and accurate transformation.

### Core Functions

-   **`main_loop(batch_size: int, input_field: str, output_field: str, input_dataset: list[dict]) -> Generator[dict, None, None]`**:
    -   **Purpose**: Handles the iterative processing of the input dataset in batches.
    -   **Processing**:
        1.  Converts each dictionary in `input_dataset` into a `dspy.Example`. The content of `item[input_field]` (from the input dictionary) is mapped to the `python_problem` field of the `dspy.Example`.
        2.  Uses `incremental_parallel` (from `dspy_util.py`) to apply the `python_stdio_to_agnostic` DSPy module to these examples in parallel batches of `batch_size`.
        3.  For each original `input_item` and its corresponding `prediction` from the LM:
            -   If the LM fails to provide a prediction for an item, a warning is issued, and the item is skipped.
            -   Otherwise, it constructs a new dictionary by copying all fields from the original `input_item` and then setting/overwriting the field specified by `output_field` with the `neutral_problem` text obtained from `prediction.neutral_problem`.
        4.  Yields this modified dictionary.

-   **`main_with_args(*, model_name: str, temperature: float, max_tokens: int, batch_size: int, input_field: str, output_field: str, input_path: str, output_path: Path)`**:
    -   **Purpose**: Orchestrates the entire workflow, including DSPy setup, file input/output, and calling `main_loop`.
    -   **Processing**:
        1.  Initializes and configures the DSPy language model (`dspy.LM`) using `model_name`, `temperature`, and `max_tokens`.
        2.  Reads the input JSONL file specified by `input_path` into a list of dictionaries using `pandas`.
        3.  Opens the `output_path` for writing.
        4.  Iterates through the dictionaries yielded by `main_loop` and writes each one as a separate line (JSON object) to the `output_path` file.

-   **`main()`**:
    -   **Purpose**: Parses command-line arguments and calls `main_with_args()`.
    -   Uses `argparse.ArgumentParser` to define and manage CLI arguments.
    -   Includes a check for the `OPENAI_API_KEY` environment variable and issues a warning if it's not set, as it might be required for certain LMs (like OpenAI models).

## Important Variables/Constants

The script's behavior is primarily controlled by command-line arguments:

-   **`--model-name` (str, default: "openai/gpt-4.1-mini")**: The identifier for the language model to be used by DSPy (often compatible with LiteLLM).
-   **`--temperature` (float, default: 0.6)**: The sampling temperature for the LM, influencing the randomness of its output.
-   **`--max-tokens` (int, default: 1000)**: The maximum number of tokens the LM should generate for its response (the neutral problem statement).
-   **`--batch-size` (int, default: 2)**: The number of problems to process in each parallel batch sent to the LM.
-   **`--input-field` (str, default: "prompt")**: The name of the JSON field in the `input_path` file that contains the original, Python-specific problem statement.
-   **`--output-field` (str, default: "prompt")**: The name of the JSON field in the `output_path` file where the transformed, language-agnostic problem statement will be stored. If this is the same as `input-field`, the original problem statement will be effectively replaced by the new one in the output.
-   **`input_path` (str, positional)**: The path to the input JSONL file containing the problems to be transformed.
-   **`output_path` (Path, positional)**: The path where the transformed dataset (as a JSONL file) will be saved.

*Note*: The script imports `extract_code_from_markdown` from `bcb_multipl_util` but does not appear to use it.

## Usage Examples

**Example: Transform Python-specific problems from `input_problems.jsonl` to language-agnostic problems in `agnostic_problems.jsonl`, using GPT-3.5-Turbo.**

```bash
# Set your OpenAI API key if using an OpenAI model
export OPENAI_API_KEY="your_openai_api_key"

python bigcodebench_multipl/src/make_pl_agnostic_problem.py \
    --model-name "openai/gpt-3.5-turbo" \
    --temperature 0.5 \
    --batch-size 5 \
    --input-field "python_prompt_text" \
    --output-field "agnostic_prompt_text" \
    input_problems.jsonl \
    agnostic_problems.jsonl
```

If `--input-field` and `--output-field` are identical (e.g., both "prompt"), the "prompt" field in the output items will contain the transformed text.

## Dependencies and Interactions

-   **External Libraries**:
    -   `dspy`: The core library for language model interaction, defining signatures (`PythonStdioToAgnostic`), and creating LM programs (`dspy.ChainOfThought`).
    -   `pandas`: Used for reading the input JSONL data file.
    -   `json` (Python standard library): Used for writing the output data in JSONL format.
    -   `warnings` (Python standard library): Used to issue alerts (e.g., missing API key, LM failing to produce a prediction).
    -   `argparse` (Python standard library): For parsing command-line arguments.
    -   `pathlib` (Python standard library): For object-oriented file path manipulation.
    -   `os` (Python standard library): Used to check for the `OPENAI_API_KEY` environment variable.
-   **Local Modules**:
    -   `dspy_util.incremental_parallel`: This function is imported and used to manage the batched, parallel execution of the DSPy module (`python_stdio_to_agnostic`) over the input examples.
-   **Language Models (LMs)**: The script requires a functional LM accessible through DSPy. For models like OpenAI's GPT series, this typically means having the `OPENAI_API_KEY` environment variable correctly set.
-   **Input Data Format**: The input (`input_path`) must be a JSONL file, where each line is a JSON object. Each object must contain the field specified by the `--input-field` argument, holding the Python-specific problem text.
-   **Output Data Format**: The output (`output_path`) will also be a JSONL file. Each JSON object will be a copy of the corresponding input object, with the language-agnostic problem statement stored in the field specified by `--output-field`.
```
