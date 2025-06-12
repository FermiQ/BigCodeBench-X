# find_libraries.py (Julia Container)

## Overview

The `find_libraries.py` script, located within the `containers/jl` directory, is a Python utility used to heuristically identify Julia package (library) dependencies from snippets of Julia source code. It reads a JSONL file where each entry is expected to contain Julia code, parses this code to find `using` or `import` statements, and extracts potential package names. The unique, sorted list of these names is then written to a specified output text file.

This script is described as a "hack" in its docstring, suggesting it's a practical approach for getting an initial list of dependencies, likely for pre-populating a Julia environment within a container based on model-generated code.

## Key Components

### Global Variables
-   **`IGNORED = ["nothing"]`**:
    -   A list of strings. Any potential package name matching an entry in this list will be disregarded by the `find_libraries` function.

### Functions

-   **`find_libraries(code: str) -> Set[str]`**:
    -   **Purpose**: Parses a string of Julia code to identify declared package dependencies.
    -   **Input**: `code` - A string containing Julia source code.
    -   **Processing**:
        -   Splits the input `code` into individual lines.
        -   Iterates through each line, stripping leading whitespace.
        -   Identifies lines that begin with Julia's package loading keywords: "using " or "import ".
        -   Extracts the remainder of the line after the keyword.
        -   Splits this remainder by commas (`,`) to handle multiple packages declared on a single line (e.g., `using PkgA, PkgB`).
        -   Each potential package name is then stripped of surrounding whitespace.
        -   **Filtering**:
            -   Package names starting with "Base" are ignored, as "Base" is Julia's core module and does not require explicit installation or separate listing.
            -   Package names containing spaces or found in the global `IGNORED` list are also ignored. A warning is issued via `warnings.warn()` for these ignored names.
        -   Valid package names are added to a Python `set` to ensure uniqueness.
    -   **Output**: A `Set[str]` containing the unique, valid package names identified from the code.

-   **`main_with_args(generations_jsonl: Path, output_txt: Path)`**:
    -   **Purpose**: The main workflow function that drives the process of reading input, finding packages, and writing the output.
    -   **Processing**:
        1.  Reads the JSONL file specified by `generations_jsonl` into a pandas DataFrame. It expects each JSON object in the file to have a field named "result_program" containing the Julia source code.
        2.  Applies the `find_libraries` function to each string in the "result_program" column of the DataFrame. The resulting set of packages for each code snippet is stored in a new column named "libs".
        3.  Iterates through the "libs" column (which contains sets of packages) and aggregates all unique package names found across all input code snippets into a single Python `set` called `all_libs`.
        4.  Sorts the `all_libs` alphabetically to ensure consistent output.
        5.  Writes the sorted list of package names to the file specified by `output_txt`, with each package name on a new line. This file will be overwritten if it already exists.

-   **`main()`**:
    -   **Purpose**: Handles command-line argument parsing.
    -   Uses `argparse.ArgumentParser` to define two positional arguments: `generations_jsonl` (the input JSONL file path) and `output_txt` (the output text file path).
    -   Calls `main_with_args` with the parsed arguments.

## Important Variables/Constants

-   **Command-Line Arguments**:
    -   `generations_jsonl` (Positional, `pathlib.Path`): Path to the input JSONL file. Each JSON object in this file must contain a key named "result_program" with a string value representing Julia source code.
    -   `output_txt` (Positional, `pathlib.Path`): Path to the output text file where the discovered Julia package names will be written.
-   **`IGNORED` (Global list)**: A list of package names that should be explicitly ignored during parsing.
-   **`"result_program"` (String literal)**: The hardcoded key name expected in the input JSONL objects to find the Julia source code.

## Usage Examples

**1. Prepare an input JSONL file (e.g., `julia_code_samples.jsonl`):**
   ```json
   {"id": "sample1", "result_program": "using DataFrames\nusing CSV\n\nfunction process_data(df::DataFrame)\n  # ...\nend"}
   {"id": "sample2", "result_program": "import HTTP\nusing JSON, Base.Threads\n\n# Make web request"}
   {"id": "sample3", "result_program": "using Genie, DataFrames\n # Web app code"}
   ```

**2. Run the script:**
   ```bash
   python containers/jl/find_libraries.py julia_code_samples.jsonl discovered_julia_pkgs.txt
   ```

**3. Check `discovered_julia_pkgs.txt`:**
   The file will contain the unique, sorted list of identified packages:
   ```
   CSV
   DataFrames
   Genie
   HTTP
   JSON
   ```
   *(Note: `Base.Threads` is ignored because it starts with "Base").*

## Dependencies and Interactions

-   **Python Version**: Assumed to be Python 3.
-   **Standard Libraries**: `argparse`, `pathlib`, `typing`, `warnings`.
-   **`pandas`**: A third-party Python library required for reading and efficiently processing the input JSONL file. It must be installed in the Python environment where the script is executed (e.g., via `pip install pandas`).
-   **File System**:
    -   Reads the input `generations_jsonl` file.
    -   Writes to (or creates) the `output_txt` file.
-   **Input Data Structure**: The script strictly expects the input file (`generations_jsonl`) to be in JSONL format, where each line is a valid JSON object. Each of these objects *must* contain a key named "result_program", and its value must be a string containing Julia source code.
-   **Output Data**: The `output_txt` file will contain a plain text list of unique Julia package names, sorted alphabetically, with one name per line. This list can then be used as a basis for installing dependencies in a Julia environment, for example, using Julia's built-in package manager (Pkg).
```
