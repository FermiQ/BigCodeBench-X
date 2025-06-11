# bcb_multipl_util.py

## Overview

The `bcb_multipl_util.py` script provides utility functions for the BigCodeBench MultiPL environment. The primary function in this version of the file is designed to extract code snippets embedded within Markdown text. This is useful for isolating executable or analyzable code from problem descriptions, documentation, or other Markdown-formatted content. The file notes that the code is adapted from an internal repository `github.com/nuprl/prl_ml`.

## Key Components

- **`extract_code_from_markdown(markdown: str) -> Optional[str]`**:
  - **Purpose**: Extracts the first complete code block found within a given Markdown string.
  - **Input**:
    - `markdown: str`: A string containing text formatted in Markdown.
  - **Processing**:
    1. Searches for the initial triple backticks (```) that signify the start of a code block.
    2. If found, it then searches for the closing triple backticks.
    3. The text content between these delimiters is extracted.
    4. The extracted code is then `strip()`ped of leading/trailing whitespace.
    5. A specific check is performed: if the string `"# Example usage:"` is present in the extracted code, the code block is truncated to exclude this comment and anything following it. This helps in cleaning up example code that might include illustrative usage directly within the snippet.
    6. It attempts to remove the language identifier (e.g., "python", "javascript") if it's present on the first line of the code block (e.g., ` ```python\n...`).
    7. The final processed code string is `strip()`ped again.
  - **Output**:
    - `Optional[str]`: Returns the cleaned, extracted code as a string if a valid code block is found. Returns `None` if no code block (opened and closed with ```) is detected in the input `markdown` string.

## Important Variables/Constants

The function `extract_code_from_markdown` uses internal string indices and temporary string slices during its operation (e.g., `code_block_start`, `code_start`, `code_block_end`, `first_newline`). There are no module-level constants defined that affect its behavior externally.

## Usage Examples

```python
from bcb_multipl_util import extract_code_from_markdown

markdown_text_1 = """
This is a description.
```python
def greet(name):
    print(f"Hello, {name}!")

# Example usage:
# greet("World")
```
More text here.
"""

code_snippet_1 = extract_code_from_markdown(markdown_text_1)
if code_snippet_1:
    print("--- Snippet 1 ---")
    print(code_snippet_1)
    # Expected Output:
    # --- Snippet 1 ---
    # def greet(name):
    #     print(f"Hello, {name}!")

markdown_text_2 = """
```
print("Just code, no language tag.")
```
"""
code_snippet_2 = extract_code_from_markdown(markdown_text_2)
if code_snippet_2:
    print("\n--- Snippet 2 ---")
    print(code_snippet_2)
    # Expected Output:
    # --- Snippet 2 ---
    # print("Just code, no language tag.")

markdown_text_no_code = "This string has no code block."
code_snippet_3 = extract_code_from_markdown(markdown_text_no_code)
if code_snippet_3 is None:
    print("\n--- Snippet 3 ---")
    print("No code found, as expected.")
    # Expected Output:
    # --- Snippet 3 ---
    # No code found, as expected.

markdown_text_unclosed_block = """
```python
print("This block is not closed."
"""
code_snippet_4 = extract_code_from_markdown(markdown_text_unclosed_block)
if code_snippet_4 is None:
    print("\n--- Snippet 4 ---")
    print("Unclosed code block not extracted, as expected.")
    # Expected Output:
    # --- Snippet 4 ---
    # Unclosed code block not extracted, as expected.
```

## Dependencies and Interactions

- **External Libraries**: None. The script relies solely on standard Python string manipulation methods and the `typing.Optional` type hint.
- **Interactions**: This utility function is likely called by other modules within the BigCodeBench MultiPL framework that need to process text inputs (e.g., problem statements, model outputs) containing Markdown. It helps in isolating the code parts for further processing, execution, or analysis.
