# find_libraries.py (R Container)

## Overview

The `find_libraries.py` script, located in the `containers/r` directory, is a Python utility intended to identify potential R package dependencies from source code. It reads a JSONL file containing code snippets (expected under a "result_program" key), attempts to parse these snippets to guess package names, and then writes these unique names to a specified output text file.

**Important Note on Current Implementation**: The parsing logic within the `find_libraries` function in this script appears to be a direct copy from a version intended for the Julia language. It specifically looks for lines starting with "using " or "import " and applies filtering rules (like ignoring "Base" prefixes) that are relevant to Julia but **not to R**. Consequently, in its current state, this script is **not effective** at accurately identifying common R package dependencies (e.g., those loaded via `library()` or `require()`).

The documentation below describes the script as it is currently written. For effective R dependency discovery, the `find_libraries` function would need to be substantially revised to recognize R's package loading patterns.

## Key Components

### Global Variables
-   **`IGNORED = ["nothing"]`**:
    -   A list of strings. Any potential package name matching an entry in this list will be disregarded by the `find_libraries` function (though this has little effect given the current parsing for R).

### Functions

-   **`find_libraries(code: str) -> Set[str]`**:
    -   **Purpose (Intended for Julia, not R)**: Parses a string of source code to identify declared package dependencies based on "using" or "import" statements.
    -   **Input**: `code` - A string containing source code.
    -   **Processing (Julia-centric)**:
        -   Splits the input `code` into lines.
        -   Identifies lines starting with "using " or "import ".
        -   Extracts and processes potential library names from these lines, including splitting by commas.
        -   Filters out names starting with "Base", containing spaces, or listed in `IGNORED`.
    -   **Output**: A `Set[str]` containing unique potential library names found.
    -   *R Relevance*: This logic will generally fail to identify R packages, as R uses `library(packageName)` or `require(packageName)`. It might incorrectly pick up strings if R code happens to have comments or strings that coincidentally match the "import " or "using " patterns.

-   **`main_with_args(generations_jsonl: Path, output_txt: Path)`**:
    -   **Purpose**: The main workflow function: reads input, applies the (currently Julia-focused) `find_libraries` function, and writes output.
    -   **Processing**:
        1.  Reads the JSONL file specified by `generations_jsonl` into a pandas DataFrame. It expects each JSON object to have a field named "result_program" containing the source code.
        2.  Applies the `find_libraries` function to each string in the "result_program" column. The results are stored in a new "libs" column.
        3.  Aggregates all unique strings found from the "libs" column into a single Python `set` called `all_libs`.
        4.  Sorts `all_libs` alphabetically.
        5.  Writes the sorted list to `output_txt`, with each string on a new line. The file is overwritten if it exists.

-   **`main()`**:
    -   **Purpose**: Handles command-line argument parsing.
    -   Uses `argparse.ArgumentParser` for two positional arguments: `generations_jsonl` (input) and `output_txt` (output).
    -   Calls `main_with_args` with the parsed arguments.

## Important Variables/Constants

-   **Command-Line Arguments**:
    -   `generations_jsonl` (Positional, `pathlib.Path`): Path to the input JSONL file. Each JSON object in this file must contain a key "result_program" with a string value (intended to be R source code).
    -   `output_txt` (Positional, `pathlib.Path`): Path to the output text file where the discovered strings (misinterpreted as packages by the current logic) will be written.
-   **`IGNORED` (Global list)**: List of strings to be ignored if found.
-   **`"result_program"` (String literal)**: The hardcoded key name expected in input JSONL objects for the source code.

## Ideal R Parsing (Not Implemented)

For effective R package discovery, the `find_libraries` function should ideally look for patterns such as:
-   `library(packageName)`
-   `library("packageName")`
-   `require(packageName)`
-   `require("packageName")`
-   `packageName::function_name` (to identify packages used via namespace access)
It should then extract `packageName` from these statements. This often involves using regular expressions tailored to these R-specific patterns.

## Usage Examples (Based on Current Script)

**1. Prepare an input JSONL file (e.g., `r_code.jsonl`):**
   ```json
   {"id": "script1", "result_program": "library(dplyr)\ndata <- data.frame(x = 1:5, y = 6:10)\nresult <- data %>% filter(x > 2) %>% select(y)"}
   {"id": "script2", "result_program": "require(ggplot2)\nlibrary(data.table)\n\nplot <- ggplot(aes(x=x,y=y)) + geom_point()"}
   ```

**2. Run the script:**
   ```bash
   python containers/r/find_libraries.py r_code.jsonl discovered_r_strings.txt
   ```

**3. Check `discovered_r_strings.txt`:**
   With the current (Julia-focused) logic, this file would likely be **empty** or contain incorrect entries, as R code using `library()` or `require()` would not be matched by the existing parsing rules.

## Dependencies and Interactions

-   **Python Version**: Assumed to be Python 3.
-   **Standard Libraries**: `argparse`, `pathlib`, `typing`, `warnings`.
-   **`pandas`**: A third-party Python library required for reading the JSONL input file. Must be installed (e.g., `pip install pandas`).
-   **File System**:
    -   Reads the input `generations_jsonl` file.
    -   Writes to (or creates) the `output_txt` file.
-   **Input Data Structure**: Expects `generations_jsonl` to be in JSONL format, with each JSON object containing a "result_program" key holding a string of source code.
-   **Output Data**: The `output_txt` file will contain a plain text list of unique strings extracted by the current parsing logic (which is not suited for R), sorted alphabetically, one per line.
```
