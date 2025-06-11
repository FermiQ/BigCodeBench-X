# completions.py

## Overview

The `completions.py` script is a command-line tool for generating code solutions for programming problems using language models (LMs) accessed via the `litellm` library. It processes a given benchmark (either a local JSONL file or a dataset from the Hugging Face Hub), sends problem prompts to a specified LM in batches, and extracts code from the LM's responses. The results, including the original problem data, the full response from the LM, and the extracted code snippet, are saved to an output file in JSONL format.

This tool is designed to automate the process of obtaining LM-generated solutions for a collection of programming tasks, which can then be used for evaluation or other downstream applications.

## Key Components

### Functions

-   **`load_benchmark(benchmark: str) -> list[dict]`**:
    -   **Purpose**: Loads programming problems from a specified benchmark source.
    -   **Input**: `benchmark` - A string that is either a path to a local JSONL file or a dataset name on the Hugging Face Hub.
    -   **Processing**:
        -   If `benchmark` corresponds to an existing file path, it reads the JSONL file using `pandas.read_json(..., lines=True)` and converts it to a list of dictionaries.
        -   Otherwise, it assumes `benchmark` is a Hugging Face dataset identifier and loads the "test" split using `datasets.load_dataset()`, returning it as a list.
    -   **Output**: A list of dictionaries, where each dictionary represents a problem item.

-   **`batches(items: Iterable, batch_size: int) -> Generator[list, None, None]`**:
    -   **Purpose**: Divides an iterable of items into smaller batches.
    -   **Input**:
        -   `items: Iterable`: The collection of items to batch.
        -   `batch_size: int`: The desired number of items per batch.
    -   **Processing**: A generator that yields lists, each containing `batch_size` items from the input `items`. It uses `tqdm` to display a progress bar for batch iteration.
    -   **Output**: A generator yielding batches (lists) of items.

-   **`make_prompt(item: dict, lang: str) -> list[dict]`**:
    -   **Purpose**: Constructs the prompt payload for the language model for a single problem.
    -   **Input**:
        -   `item: dict`: A dictionary representing a single problem, expected to have a "prompt" key containing the problem description.
        -   `lang: str`: The target programming language for the solution.
    -   **Processing**: Creates a list containing a single dictionary, structured for `litellm`'s chat completion interface. The "role" is "user", and the "content" is formed by appending a language directive (e.g., "\n\nSolve this problem using python.") to the original problem prompt (`item["prompt"]`).
    -   **Output**: A list of dictionaries representing the chat message(s) for the LM.

-   **`main_with_args(*, model: str, benchmark: str, temperature: float, max_tokens: int, top_p: float, batch_size: int, lang: str, output_path: Path)`**:
    -   **Purpose**: The main workflow function that orchestrates loading data, generating completions, and saving results.
    -   **Inputs**: Takes all necessary parameters, typically from command-line arguments (see "Important Variables/Constants" below).
    -   **Processing**:
        1.  Loads benchmark problems using `load_benchmark()`.
        2.  Opens the specified `output_path` for writing.
        3.  Iterates through the problems in batches using `batches()`:
            a.  For each batch, prepares a list of prompt messages using `make_prompt()` for every item in the batch.
            b.  Sends these prompts to the specified `model` using `litellm.batch_completion()`, along with `temperature`, `max_tokens`, and `top_p` parameters.
            c.  For each original problem item and its corresponding LM response:
                i.  Extracts the textual content from the LM's response.
                ii. Uses `extract_code_from_markdown()` (from the `bcb_multipl_util` module) to parse this text and extract the primary code block.
                iii.Creates a new dictionary by merging the original problem `item` with two new keys: `"response"` (the full LM response text) and `"program"` (the extracted code). If the original `item` had a "program" key, it will be overwritten by the newly extracted code.
                iv. Writes this combined dictionary as a JSON line to the `output_path`.
            d.  Flushes the output file stream after each batch to ensure data is written progressively.

-   **`main()`**:
    -   **Purpose**: Handles command-line argument parsing and invokes `main_with_args()`.
    -   Uses `argparse.ArgumentParser` to define and parse CLI arguments.
    -   Passes the parsed arguments as keyword arguments to `main_with_args()`.

## Important Variables/Constants

The script's behavior is primarily controlled by command-line arguments:

-   **`--model` (str, required)**: The identifier of the language model to be used with `litellm` (e.g., "gpt-3.5-turbo", "ollama/codellama").
-   **`--benchmark` (str, required)**: Specifies the source of programming problems. This can be:
    -   A path to a local JSONL file.
    -   A dataset name from the Hugging Face Hub (e.g., "nuprl/MultiPL-E-completions-CSharp").
-   **`--lang` (str, required)**: The target programming language in which the solutions should be generated (e.g., "python", "javascript", "C#"). This is appended to the prompt.
-   **`--temperature` (float, default: 0.2)**: The sampling temperature for the LM, controlling randomness. Lower values make the output more deterministic.
-   **`--max-tokens` (int, default: 5000)**: The maximum number of tokens the LM is allowed to generate for each response.
-   **`--top-p` (float, default: 0.95)**: The nucleus sampling parameter (top-p filtering).
-   **`--batch-size` (int, default: 10)**: The number of problems to include in each batch request to the LM.
-   **`--output-path` (Path, required)**: The file path where the results (JSONL format) will be saved.

## Usage Examples

**Example 1: Generate C# completions using GPT-3.5-Turbo from a Hugging Face dataset:**
```bash
python bigcodebench_multipl/src/completions.py \
    --model "gpt-3.5-turbo" \
    --benchmark "nuprl/MultiPL-E-completions-CSharp" \
    --lang "C#" \
    --temperature 0.2 \
    --batch-size 5 \
    --output-path "csharp_completions_output.jsonl"
```

**Example 2: Generate Python completions using a local Ollama model from a local JSONL file:**
```bash
python bigcodebench_multipl/src/completions.py \
    --model "ollama/codellama" \
    --benchmark "./my_custom_problems.jsonl" \
    --lang "python" \
    --temperature 0.1 \
    --batch-size 10 \
    --output-path "custom_py_completions.jsonl"
```

## Dependencies and Interactions

-   **External Libraries**:
    -   `litellm`: Crucial for interacting with a wide variety of language model APIs.
    -   `pandas`: Used if the input `benchmark` is a local JSONL file (for reading).
    -   `datasets` (Hugging Face): Used if the input `benchmark` is a Hugging Face dataset name.
    -   `argparse` (Python standard library): For parsing command-line arguments.
    -   `tqdm`: For displaying progress bars during batch processing.
    -   `json` (Python standard library): For writing output data in JSONL format.
    -   `pathlib` (Python standard library): For object-oriented file path manipulation.
-   **Local Modules**:
    -   `bcb_multipl_util.extract_code_from_markdown`: This utility function is imported and used to extract code snippets from the potentially Markdown-formatted responses generated by the LM.
-   **Language Models**: The script requires access to a language model configured and accessible through `litellm`. This might involve API keys or local model setups depending on the chosen `model`.
-   **Benchmark Data Format**: Input benchmark items (from file or HF dataset) are expected to be dictionaries containing at least a `"prompt"` key, which holds the problem description.
-   **File System**:
    -   Reads benchmark data if a local file path is provided.
    -   Writes all output (original problem data + LM response + extracted code) to the specified `--output-path` in JSONL format.
```
