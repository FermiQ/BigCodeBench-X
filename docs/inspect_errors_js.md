# inspect_errors.py (JavaScript Container)

## Overview

The `inspect_errors.py` script is a Python command-line utility designed for analyzing and summarizing errors from JavaScript execution logs, particularly those originating from the Node.js runtime. It processes a JSONL file where each record contains execution details (like exit codes and standard error streams). The script filters these records to identify Node.js specific errors, prints these full error messages, and then performs a targeted extraction of module names from "Cannot find module" errors.

This tool is useful for quickly diagnosing common issues in batch executions of JavaScript code, especially for identifying missing npm packages.

## Key Components

### Functions

-   **`main_with_args(src_file: str)`**:
    -   **Purpose**: The core function that handles the entire error inspection workflow.
    -   **Input**: `src_file` - A string representing the path to the input JSONL file containing execution logs.
    -   **Processing**:
        1.  **Data Loading**: Reads the JSONL data from `src_file` into a pandas DataFrame.
        2.  **Error Filtering**:
            -   Selects records where the `exit_code` is not equal to `0` (indicating an error occurred).
            -   Further filters to include only records where `stderr` is not null/NaN.
            -   Narrows down to errors likely from Node.js by selecting records where the `stderr` string contains "Node.js".
        3.  **Printing Node.js Errors**:
            -   Iterates through each filtered error record.
            -   Prints the full content of the `stderr` field for each error, followed by a horizontal separator line (`--------------------`).
        4.  **Summary Count**: Prints the total number of identified Node.js errors.
        5.  **Missing Module Extraction**:
            -   Filters the Node.js error DataFrame to find records where `stderr` contains the specific string "Cannot find module".
            -   Uses a regular expression (`r"Cannot find module '(.*)'"`) to extract the module name from these error messages. The first captured group `(.*)` is assumed to be the module name.
            -   Prints a unique list of all such extracted module names, each on a new line.

-   **`main()`**:
    -   **Purpose**: Handles command-line argument parsing.
    -   Uses `argparse.ArgumentParser` to define one positional argument: `src_file` (the path to the input JSONL file).
    -   Calls `main_with_args` with the parsed `src_file` argument.

## Important Variables/Constants

-   **Command-Line Arguments**:
    -   `src_file` (Positional, `str`): The path to the input JSONL file. This file should contain records of execution attempts, with each record being a JSON object.
-   **Expected Input JSONL Fields**:
    -   `"exit_code"` (integer): The exit status of the executed program. A non-zero value typically indicates an error.
    -   `"stderr"` (string, can be null): The standard error output captured during the execution.
-   **Key String Literals for Filtering**:
    -   `"Node.js"`: Used to identify errors as originating from the Node.js runtime environment.
    -   `"Cannot find module"`: Used to pinpoint specific errors related to missing modules.
-   **Regular Expression for Module Extraction**:
    -   `r"Cannot find module '(.*)'"`: This regex is used to capture the name of the module that Node.js failed to find.

## Usage Examples

**1. Prepare an Input JSONL File (e.g., `js_run_logs.jsonl`):**
   Each line should be a JSON object representing one execution result.
   ```json
   {"task_id": "t1", "exit_code": 0, "stderr": null, "stdout": "Completed successfully."}
   {"task_id": "t2", "exit_code": 1, "stderr": "SyntaxError: Unexpected token\n    at Object.compileFunction (node:vm:360:18)\n    at Node.js an_issue"}
   {"task_id": "t3", "exit_code": 1, "stderr": "Error: Cannot find module 'express'\nRequire stack:\n- /app/server.js\n    at Function.Module._resolveFilename (node:internal/modules/cjs/loader:933:15)\n    at Node.js error"}
   {"task_id": "t4", "exit_code": 1, "stderr": "Error: Cannot find module 'lodash'\nRequire stack:\n- /app/utils.js\n    at Node.js another_error"}
   {"task_id": "t5", "exit_code": 1, "stderr": "Error: Cannot find module 'express'\nRequire stack:\n- /app/app.js\n    at Node.js a_third_error"}
   ```

**2. Run the Script:**
   ```bash
   python containers/js/inspect_errors.py js_run_logs.jsonl
   ```

**3. Expected Console Output:**
   The script will print the `stderr` for each filtered Node.js error, followed by a summary and the list of unique missing modules.
   ```
   SyntaxError: Unexpected token
       at Object.compileFunction (node:vm:360:18)
       at Node.js an_issue
   ----------------------------------------------------------------------------------------------------
   Error: Cannot find module 'express'
   Require stack:
   - /app/server.js
       at Function.Module._resolveFilename (node:internal/modules/cjs/loader:933:15)
       at Node.js error
   ----------------------------------------------------------------------------------------------------
   Error: Cannot find module 'lodash'
   Require stack:
   - /app/utils.js
       at Node.js another_error
   ----------------------------------------------------------------------------------------------------
   Error: Cannot find module 'express'
   Require stack:
   - /app/app.js
       at Node.js a_third_error
   ----------------------------------------------------------------------------------------------------
   Found 4 errors that are from Node.js
   express
   lodash
   ```

## Dependencies and Interactions

-   **Python Version**: Assumed to be Python 3.
-   **Standard Libraries**: `argparse`.
-   **`pandas`**: A third-party Python library essential for reading the JSONL input file and performing data filtering and manipulation. It must be installed in the Python environment (e.g., `pip install pandas`).
-   **File System**: The script reads from the input file specified by the `src_file` command-line argument. It does not write any files; all output is directed to the standard output stream.
-   **Input Data Structure**: The script expects `src_file` to be a valid JSONL file. Each JSON object on a new line must contain an `"exit_code"` field (integer) and an `"stderr"` field (string, which can be null or missing if no error output occurred).
-   **Node.js Error Message Format**: The accuracy of the missing module extraction (`Cannot find module 'module_name'`) depends on the consistency of Node.js error message formatting. Changes to this format in future Node.js versions could affect the script's behavior.
```
