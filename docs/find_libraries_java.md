# find_libraries.py (Java Container)

## Overview

The `find_libraries.py` script, located in the `containers/java` directory, is a Python utility designed to heuristically identify potential Java library dependencies from source code and error messages. It processes a JSONL file containing generated code snippets (and optionally error logs), attempts to extract package names, and then appends these names to a specified output text file.

The script serves as a preliminary step in dependency discovery. As noted in its internal comments, the output (a list of package names) is not directly in Maven's `group:artifact:version` format and requires manual lookup or further processing to be consumable by Java build tools.

## Key Components

### Global Variables
-   **`IGNORED = ["nothing"]`**:
    -   A list of strings. Any potential library name matching an entry in this list will be disregarded by the `find_libraries` function.

### Functions

-   **`find_libraries(code: str) -> Set[str]`**:
    -   **Purpose**: Attempts to guess library dependencies by parsing import statements from a given string of source code.
    -   **Input**: `code` - A string containing source code.
    -   **Processing**:
        -   Returns an empty set if the input `code` is not a string.
        -   Splits the code into lines.
        -   Identifies lines starting with "using " or "import ". While "import" is standard Java, "using" is more common in languages like C# or Julia; its inclusion suggests this function might be based on a generic template.
        -   Extracts the part of the line after "using " or "import ".
        -   Splits this part by commas (to handle multiple imports on one line, e.g., `import java.util.List, java.util.Set;` though less common for distinct top-level packages in Java).
        -   Each extracted part is stripped of whitespace.
        -   **Filtering**:
            -   Ignores parts starting with "Base" (more relevant to languages like Julia).
            -   Ignores parts containing spaces or found in the global `IGNORED` list, issuing a warning for such cases.
        -   Adds valid identified strings to a set to ensure uniqueness.
    -   **Output**: A `Set[str]` containing unique potential library/package names found in the code.
    -   *Note*: For Java, this function will primarily pick up on `import` statements. The extracted strings are typically fully qualified package names or class names (e.g., `java.util.List`, `org.apache.commons.lang3.StringUtils`).

-   **`extract_missing_packages(error_text: Optional[str]) -> set[str]`**:
    -   **Purpose**: Extracts Java package names from error messages that indicate a missing package.
    -   **Input**: `error_text` - A string, typically from `stderr`, containing compiler or runtime error messages.
    -   **Processing**:
        -   Returns an empty set if `error_text` is not a string.
        -   Uses the regular expression `r"error: package ([\w\.]+) does not exist"` to find matches. This pattern is characteristic of Java compiler errors when a package cannot be resolved.
        -   Collects all matched package names (the first capturing group `[\w\.]+`) into a set.
    -   **Output**: A `Set[str]` containing unique package names identified from the error messages.

-   **`main_with_args(generations_jsonl: Path, output_txt: Path)`**:
    -   **Purpose**: The main workflow function that reads generated data, finds libraries, and writes them to an output file.
    -   **Processing**:
        1.  If `output_txt` already exists, its content (assumed to be a list of library names, one per line) is read into a set (`existing_libs`). Otherwise, `existing_libs` is initialized as an empty set.
        2.  Reads the JSONL file specified by `generations_jsonl` into a pandas DataFrame. Each line in the JSONL file is expected to be a JSON object.
        3.  Applies the `find_libraries` function to the "program" column of the DataFrame. The results (sets of libraries for each program) are stored in a new "libs" column.
            *(Note: A line to apply `extract_missing_packages` to an "stderr" column is commented out in the source but indicates an intended feature).*
        4.  Aggregates all library sets from the "libs" column into a single set `new_libs` to get all unique libraries found in the current run.
        5.  Merges `new_libs` with `existing_libs` to get a combined set of all libraries.
        6.  Sorts the combined list of libraries alphabetically (for consistent output) and writes them to `output_txt`, with each library on a new line.

-   **`main()`**:
    -   **Purpose**: Parses command-line arguments.
    -   Uses `argparse.ArgumentParser` to define two positional arguments: `generations_jsonl` (input file path) and `output_txt` (output file path).
    -   Calls `main_with_args` with the parsed arguments.

## Important Variables/Constants

-   **Command-Line Arguments**:
    -   `generations_jsonl` (Positional, `pathlib.Path`): The path to the input JSONL file. This file should contain objects with a "program" key holding the source code string.
    -   `output_txt` (Positional, `pathlib.Path`): The path to the text file where the list of identified library/package names will be written or updated.
-   **`IGNORED` (Global list)**: Contains strings that, if matched as potential library names, will be excluded.
-   **Regex in `extract_missing_packages`**: `r"error: package ([\w\.]+) does not exist"` is tailored to Java's "package does not exist" error messages.

## Usage Examples

**1. Prepare an input JSONL file (e.g., `generated_java_code.jsonl`):**
   ```json
   {"task_id": "task1", "program": "import java.util.ArrayList;\nimport org.apache.commons.math3.analysis.UnivariateFunction;\n\npublic class Solution1 { // ... }"}
   {"task_id": "task2", "program": "import com.google.gson.Gson;\n\npublic class Solution2 { // ... }", "stderr": "src/Solution2.java:3: error: package com.another.missing does not exist\nimport com.another.missing.SomeClass;"}
   ```

**2. Run the script:**
   ```bash
   python containers/java/find_libraries.py generated_java_code.jsonl discovered_java_libs.txt
   ```

**3. Check `discovered_java_libs.txt`:**
   The file might contain (order may vary before sorting, but output is sorted):
   ```
   com.google.gson.Gson
   java.util.ArrayList
   org.apache.commons.math3.analysis.UnivariateFunction
   ```
   *(If the `extract_missing_packages` part were active for an "stderr" field, `com.another.missing` might also appear).*

## Dependencies and Interactions

-   **Python Version**: Assumed to be Python 3.
-   **Standard Libraries**: `argparse`, `pathlib`, `typing`, `warnings`, `re`.
-   **`pandas`**: A third-party library required for reading and processing the JSONL input file. It needs to be installed in the Python environment where the script is run (e.g., `pip install pandas`).
-   **File System**:
    -   Reads the input `generations_jsonl` file.
    -   Reads from and writes to (or creates) the `output_txt` file.
-   **Input Data Structure**: The script expects `generations_jsonl` to be a valid JSONL file. Each JSON object on a new line must contain a key (by default "program") whose value is a string of source code. If the error extraction logic were used, a key for error messages (e.g., "stderr") would also be expected.
-   **Output Data**: The `output_txt` file will contain a list of unique, sorted library/package names, one per line. This list is an aggregation of previously existing entries (if the file existed) and newly found names.
-   **Java Specificity**: While `find_libraries` has some generic parsing elements, the `extract_missing_packages` function is specifically tuned for Java error messages. The script's location within `containers/java` strongly implies its intended use is for Java projects. The output requires further mapping to be useful with Maven or Gradle.
```
