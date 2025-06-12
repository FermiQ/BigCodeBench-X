# try_install_packages.jl

## Overview

`try_install_packages.jl` is a Julia script designed to automate the installation of a list of Julia packages. It takes a single command-line argument: the path to a text file where each line contains the name of a Julia package. The script iterates through these names, attempting to install each package using Julia's built-in package manager, `Pkg`. It reports the success or failure of each installation attempt and provides a final summary.

This script is particularly useful in environments where Julia packages need to be dynamically provisioned or when testing the installability of a set of packages, as it gracefully handles errors for individual packages without halting the entire process.

## Key Components

The script is a single Julia file and uses core Julia functionalities.

-   **`using Pkg`**:
    -   This line at the beginning of the script imports Julia's standard package manager module, `Pkg`. All package management operations, primarily `Pkg.add()`, are performed using functions from this module.

-   **Command-Line Argument Processing**:
    -   The script checks the `ARGS` global array, which holds command-line arguments passed to the Julia interpreter.
    -   It expects exactly one argument: `if length(ARGS) != 1 error("Usage: julia try_install_packages.jl <packages_file>") end`.
    -   The first argument `ARGS[1]` is taken as the path to the file containing the list of package names.

-   **Package List Handling**:
    -   `packages_file = ARGS[1]`: Stores the input file path.
    -   `packages = readlines(packages_file)`: Reads all lines from `packages_file`. Each line is treated as a potential package name and stored as a string in the `packages` array.

-   **Installation Tracking**:
    -   `failures = String[]`: An empty array of strings, initialized to store the names of packages that could not be installed.
    -   `successes = String[]`: An empty array of strings, initialized to store the names of packages that were successfully installed.

-   **Package Installation Loop**:
    -   The script iterates through each `pkg` string in the `packages` array.
    -   `pkg = strip(pkg)`: Removes any leading or trailing whitespace from the package name.
    -   `if !isempty(pkg)`: Ensures that empty lines from the input file are skipped.
    -   **`try...catch` Block**: This block handles the installation attempt for each package:
        -   **`try`**:
            -   `println("Installing $pkg...")`: Prints a message to standard output indicating which package is currently being installed.
            -   `Pkg.add(pkg)`: This is the core command. It invokes Julia's package manager to download and install the specified package (`pkg`) and its dependencies from the configured package registries.
            -   `println("Successfully installed $pkg")`: If `Pkg.add()` completes without error, a success message is printed.
            -   `push!(successes, pkg)`: The package name is added to the `successes` array.
        -   **`catch e`**: If an exception `e` occurs during the `Pkg.add()` operation (e.g., package not found, network error, dependency resolution issue):
            -   `println("Failed to install $pkg: $(sprint(showerror, e))")`: A failure message is printed, including the package name and a detailed error message (the output of `sprint(showerror, e)` which captures the full error description as a string).
            -   `push!(failures, pkg)`: The package name is added to the `failures` array.

-   **Final Reporting**:
    -   After attempting to install all packages, the script prints a summary:
        -   First, it prints "Successes:", followed by each package name in the `successes` array on a new line.
        -   Then, it prints "Failures:", followed by each package name in the `failures` array on a new line.

## Important Variables/Constants

-   **`ARGS`**: Julia's built-in global variable array for command-line arguments. `ARGS[1]` is used to get the input file path.
-   **`packages_file::String`**: Stores the path to the input file listing packages.
-   **`packages::Vector{String}`**: An array holding the names of packages read from the input file.
-   **`failures::Vector{String}`**: Stores names of packages that failed to install.
-   **`successes::Vector{String}`**: Stores names of packages successfully installed.

## Usage Examples

**1. Create a Package List File:**
   Create a plain text file (e.g., `my_julia_packages.txt`) with one package name per line:
   ```text
   DataFrames
   JSON3
   CSV
   Makie  # A more complex package with many dependencies
   NonExistentPackageXYZ # This package will likely fail
   HTTP
   ```

**2. Run the Script:**
   Open your terminal or command prompt, navigate to the directory containing `try_install_packages.jl` and your package list file, and execute:
   ```bash
   julia try_install_packages.jl my_julia_packages.txt
   ```

**Expected Output:**
The script will print real-time status updates for each package:
```
Installing DataFrames...
Successfully installed DataFrames
Installing JSON3...
Successfully installed JSON3
Installing CSV...
Successfully installed CSV
Installing Makie...
# (Potentially a lot of output here as Makie and its dependencies are installed)
Successfully installed Makie
Installing NonExistentPackageXYZ...
Failed to install NonExistentPackageXYZ: ArgumentError: Package NonExistentPackageXYZ not found in current path...
Installing HTTP...
Successfully installed HTTP
Successes:
DataFrames
JSON3
CSV
Makie
HTTP
Failures:
NonExistentPackageXYZ
```
*(Actual output and error messages for `NonExistentPackageXYZ` may vary slightly based on Julia version and Pkg configuration).*

## Dependencies and Interactions

-   **Julia Language**: A working installation of the Julia programming language is required to execute the script.
-   **`Pkg` Module**: The script relies heavily on Julia's built-in `Pkg` module for all package management tasks. This module is part of the Julia standard library and does not need separate installation.
-   **Network Connectivity**: To download and install packages, `Pkg.add()` requires an active internet connection to access Julia's package registries (like the default General registry).
-   **File System**:
    -   The script reads package names from the input text file specified as a command-line argument.
    -   The `Pkg.add()` command downloads package source code and compiled files into Julia's environment-specific or global package directories (e.g., typically within `~/.julia/packages/` or a project-specific environment if one is active). The script itself does not directly manage these paths but relies on `Pkg`'s default behavior.
-   **Standard Output (STDOUT)**: All informational messages, per-package success/failure statuses, and the final summary report are printed to STDOUT. This includes detailed error messages caught during failed installations.
```
