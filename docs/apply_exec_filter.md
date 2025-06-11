# apply_exec_filter.py

## Overview

The `apply_exec_filter.py` script is a command-line tool designed to process and merge data from two JSONL files. Its primary role is to combine information from an initial dataset (e.g., a list of tasks) with corresponding execution results, filter these results based on success or failure (exit code), and then save the consolidated data to a new JSONL file. This is typically used in a data processing pipeline where task definitions and their execution outcomes are stored separately and need to be reconciled.

## Key Components

- **`load_jsonl_ignoring_errors(path: str)`**:
  - Reads a JSONL (JSON Lines) file specified by `path`.
  - Parses each line as a JSON object.
  - If a line cannot be decoded as valid JSON, it prints an error message with the line number and content, then skips that line and proceeds with the next. This allows the script to handle potentially malformed entries in the input JSONL file without halting execution.
  - Yields parsed JSON objects one by one.

- **`main_with_args(file1: str, file2: str, include_failed: bool, output_file: str)`**:
  - This is the core function that orchestrates the data processing.
  - Reads `file1` (expected to be a JSONL file) into a pandas DataFrame (`df1`).
  - Reads `file2` (also a JSONL file, typically containing execution results) into a pandas DataFrame (`df2`) using the `load_jsonl_ignoring_errors` function to robustly handle malformed lines.
  - Merges `df1` and `df2` into a single DataFrame (`df`) based on a common `task_id` column. An inner merge is performed, meaning only `task_id`s present in both files are kept.
  - If `include_failed` is `False` (the default behavior if the `--include-failed` flag is not used):
    - Filters the merged DataFrame to retain only rows where the `exit_code` column is equal to 0 (indicating successful execution).
    - Drops several columns from the DataFrame: `exit_code`, `stderr`, `stdout`, `timeout`, and `reasoning`. These columns are typically related to the execution status and details of failed tasks.
  - Writes the final processed DataFrame to `output_file` in JSONL format, with each line being a JSON record.

- **`main()`**:
  - Sets up and parses command-line arguments using the `argparse` module.
  - Arguments:
    - `file1`: Path to the first input JSONL file.
    - `file2`: Path to the second input JSONL file.
    - `--include-failed`: An optional flag. If present, tasks that failed (non-zero `exit_code`) are included in the output.
    - `--output-file`: Path where the resulting JSONL file will be saved.
  - Calls `main_with_args` with the parsed arguments, effectively starting the data processing logic.

## Important Variables/Constants

The script primarily uses variables to manage file paths and pandas DataFrames. Key column names implicitly expected in the input files include:

- **`task_id`**: A common identifier in both `file1` and `file2` used for merging the two datasets.
- **`exit_code`**: Expected in `file2`. An integer indicating the exit status of a task (0 typically means success).
- **`stderr`**, **`stdout`**, **`timeout`**, **`reasoning`**: Expected in `file2` (or the merged DataFrame before filtering). These columns are dropped if `include_failed` is false.

## Usage Examples

The script is executed from the command line.

**Example 1: Merge and filter for successful tasks only**

```bash
python bigcodebench_multipl/src/apply_exec_filter.py path/to/input1.jsonl path/to/input2_exec_results.jsonl --output-file path/to/successful_merged_output.jsonl
```

**Example 2: Merge and include all tasks (successful and failed)**

```bash
python bigcodebench_multipl/src/apply_exec_filter.py path/to/input1.jsonl path/to/input2_exec_results.jsonl --include-failed --output-file path/to/all_merged_output.jsonl
```

## Dependencies and Interactions

- **External Libraries:**
  - `argparse`: (Python standard library) Used for parsing command-line arguments.
  - `pandas`: Used for efficient data manipulation via DataFrames. This is a core dependency for reading, merging, filtering, and writing data.
  - `json`: (Python standard library) Used by `load_jsonl_ignoring_errors` to parse JSON objects from each line of the input files.
  - `pathlib`: (Python standard library) Used by `load_jsonl_ignoring_errors` for object-oriented file path handling.

- **File System:**
  - Reads two input files in JSONL format.
  - Writes one output file in JSONL format.
  - The paths to these files are provided as command-line arguments.

- **Data Format Assumptions:**
  - Input files must be in JSONL format (one valid JSON object per line).
  - Both input files must contain a `task_id` field which can be used to join them.
  - The second input file (`file2`) is expected to contain an `exit_code` field, and potentially `stderr`, `stdout`, `timeout`, and `reasoning` fields, which are used for filtering and are dropped if only successful tasks are retained.
