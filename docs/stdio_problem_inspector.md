# stdio_problem_inspector.py

## Overview

The `stdio_problem_inspector.py` script provides tools for working with STDIN/STDOUT-based programming problems, particularly those generated and evaluated within the BigCodeBench-MultiPL ecosystem. It serves two main purposes, accessible via command-line subcommands:

1.  **`upload`**: This utility merges a file containing synthesized STDIN/STDOUT problem definitions with a corresponding file of execution results for those problems. The output is a single JSONL file, which is a necessary preprocessing step for inspection or further analysis.
2.  **`dataset-inspector`**: This launches an interactive web application built with Gradio. The application loads a processed dataset (typically the output of the `upload` command, hosted on the Hugging Face Hub) and allows developers to visually inspect individual problems. It displays the generated problem details (prompt, program, test suite, execution outputs, model reasoning) side-by-side with the original BigCodeBench problem from which it was derived. Users can filter problems based on execution status (e.g., timeouts, successes, errors).

The script's introductory comments indicate that a version of the inspector tool is already hosted on Hugging Face Spaces, but it can also be run locally.

## Key Components

### Copied from `bcb_reader.py`

For the purpose of fetching and displaying original BigCodeBench problem details, several components are directly incorporated from `bcb_reader.py`:
-   **TypedDicts**: `_OriginalBigCodeBenchProblem` and `BigCodeBenchProblem`.
-   **Constants**: `_PROMPT_BOILERPLATE` and `_PROMPT_SUFFIX`.
-   **Functions**: `_prepare_bcb_problem(item)` and `load_bigcodebench()`.
These enable the inspector to show the source problem alongside the generated/evaluated one.

### `upload` Subcommand

-   **`upload(problems_path: Path, results_path: Path, output_path: Path)` Function**:
    -   Reads problems from `problems_path` (JSONL, e.g., output of `make_stdio_problem.py`).
    -   Reads execution results from `results_path` (JSONL).
    -   Merges these two datasets into a single pandas DataFrame using `task_id` as the key (left merge).
    -   Asserts that the merged DataFrame has a specific, expected set of columns: `reasoning`, `prompt`, `program`, `test_suite`, `task_id`, `timeout`, `exit_code`, `stdout`, `stderr`. This serves as a basic schema validation.
    -   Writes the merged DataFrame to `output_path` as a JSONL file.

### `dataset-inspector` Subcommand

-   **`dataset_inspector(dataset_name: str, data_dir: str)` Function**:
    -   This function sets up and launches the Gradio web application.
    -   **Data Loading**:
        -   Loads the target dataset (synthesized problems + results) from the Hugging Face Hub using `datasets.load_dataset(dataset_name, data_dir=data_dir, split="test")`.
        -   Loads the original BigCodeBench problems using the bundled `load_bigcodebench()` function.
        -   Merges these two datasets on `task_id` to allow side-by-side comparison.
    -   **Gradio UI Helper Functions**:
        -   `get_filtered_data(predicate)`: Filters the merged DataFrame based on the current UI filter settings (timeout, success, error).
        -   `format_problem_display(row, predicate)`: Formats a single problem's data (both generated and original parts, including code, prompts, execution details, and optionally reasoning) into Markdown for display.
        -   `update_display(current_index, predicate)`: Core UI update logic; refreshes displayed content and button states.
        -   `go_prev()`, `go_next()`: Handle navigation between problems in the filtered list.
        -   `on_filter_change()`: Resets the view when filter criteria are modified.
        -   `update_predicate()`: Updates the internal state dictionary (`predicate`) that stores filter settings.
    -   **Gradio UI Layout (`gr.Blocks`)**:
        -   **State Variables**: `current_index` (integer for current problem) and `predicate` (dictionary for filter states).
        -   **Controls**:
            -   "Previous" and "Next" buttons for navigation.
            -   A status textbox showing "current_problem / total_filtered_problems".
            -   Checkboxes: "Filter by timeout = True", "Show successes (exit_code == 0)", "Show errors (exit_code != 0)", "Show reasoning".
        -   **Display Panes**: Two `gr.Markdown` components (`generated_display`, `original_display`) to show the formatted content of the generated problem and its original counterpart.
    -   **Event Handling**: Connects UI controls (buttons, checkboxes) to their respective Python handler functions.
    -   Launches the interactive application using `demo.launch(share=True)`.

### `main()` Function

-   Configures `argparse` to handle command-line arguments and subcommands.
-   **`upload` arguments**:
    -   `--problems-path` (Path, required): Path to the problems file (e.g., from `make_stdio_problem.py`).
    -   `--results-path` (Path, required): Path to the execution results file.
    -   `--output-path` (Path, required): Path to save the merged JSONL file.
-   **`dataset-inspector` arguments**:
    -   `--dataset-name` (str, default: "nuprl/BigCodeBench-MultiPL-Results"): Name of the dataset on Hugging Face Hub.
    -   `--data-dir` (str, default: "python_stdio"): Directory within the HF dataset (often language-specific).
-   If no subcommand is specified, it defaults to running `dataset_inspector` with its default arguments.
-   Calls the selected function (`upload` or `dataset_inspector`) with the parsed arguments.

## Important Variables/Constants

-   **Column Schema for `upload`**: The `upload` function expects a specific set of columns after merging, used in an assertion: `["reasoning", "prompt", "program", "test_suite", "task_id", "timeout", "exit_code", "stdout", "stderr"]`.
-   **Default Dataset for Inspector**:
    -   `dataset-name`: "nuprl/BigCodeBench-MultiPL-Results"
    -   `data-dir`: "python_stdio"
-   **Gradio `predicate` State**: This dictionary (`{'filter_timeout': bool, 'filter_successes': bool, 'filter_errors': bool, 'show_reasoning': bool}`) is crucial for controlling the filtering and display logic in the Gradio app.

## Usage Examples

(Assumes `uv` is used for environment management as per the script's docstring)

**1. Prepare dataset for inspection (merge problems and results):**
```bash
uv run python3 -m bigcodebench_multipl.stdio_problem_inspector upload \
    --problems-path path/to/your_stdio_problems.jsonl \
    --results-path path/to/your_execution_results.jsonl \
    --output-path path/to/merged_problems_for_inspection.jsonl
```

**2. Upload `merged_problems_for_inspection.jsonl` to Hugging Face Hub:**
   - Create a new dataset on Hugging Face Hub (e.g., `your-username/my-inspection-data`).
   - Create a directory structure like `python_stdio/test.jsonl` (if your data is for Python STDIN/STDOUT problems) within that dataset repository, and place your `merged_problems_for_inspection.jsonl` file as `test.jsonl`.

**3. Run the dataset inspector locally, pointing to your HF dataset:**
```bash
uv run python3 -m bigcodebench_multipl.stdio_problem_inspector dataset-inspector \
    --dataset-name "your-username/my-inspection-data" \
    --data-dir "python_stdio"
```
A URL for the local Gradio app will be printed to the console.

**4. Run the dataset inspector with default (pre-configured) dataset (if accessible):**
```bash
uv run python3 -m bigcodebench_multipl.stdio_problem_inspector dataset-inspector
# or simply:
# uv run python3 -m bigcodebench_multipl.stdio_problem_inspector
```

## Dependencies and Interactions

-   **External Libraries**:
    -   `pandas`: Used for data manipulation in `upload` and internally by `dataset_inspector`.
    -   `gradio` (`gr`): The core library for building the interactive web UI in `dataset_inspector`.
    -   `datasets` (Hugging Face): Used by `dataset_inspector` to load data from Hugging Face Hub, and by the bundled `bcb_reader` code.
    -   `ast` (Python standard library): Used by the bundled `bcb_reader` code for syntax checking.
    -   `argparse` (Python standard library): For command-line argument parsing.
    -   `pathlib` (Python standard library): For object-oriented path handling.
-   **Hugging Face Hub**: The `dataset_inspector` is primarily designed to consume datasets hosted on the Hugging Face Hub. The `upload` command prepares data for this purpose.
-   **File System**: The `upload` subcommand reads two input JSONL files and writes one output JSONL file. The `dataset_inspector` relies on data that originates from files, typically accessed via the `datasets` library.
-   **`bcb_reader.py` (Bundled Code)**: The inspector's ability to show original BigCodeBench problems relies on the correct functioning of the code copied from `bcb_reader.py`.
-   **User's Web Browser**: For interacting with the `dataset-inspector` UI.
```
