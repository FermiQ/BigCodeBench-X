# install_packages.R

## Overview

The `install_packages.R` script is a simple R program designed to install a predefined, hardcoded list of R packages. Its sole purpose is to invoke the `install.packages()` function with this specific list, thereby ensuring that these packages (and their dependencies) are installed into the R environment where the script is executed.

This script is typically used for setting up an R environment, for example, within a Docker container or a new R installation, to ensure that a specific set of commonly used or required packages is available.

## Key Components

The script consists of a single, direct command:

-   **`install.packages(c("quanteda", "blake3", "gridExtra", "openssl", "aTSA", "KernSmooth", "moments", "matrixStats", "Rmpfr", "randomForest", "SuppDists", "naturalsort", "sentimentr", "ggpolar", "XML", "RColorBrewer"))`**
    -   **`install.packages()`**: This is the standard R function used to download and install packages from configured repositories (usually CRAN - the Comprehensive R Archive Network - by default).
    -   **`c(...)`**: This creates a character vector in R. The elements of this vector are the names of the packages to be installed.
    -   **Hardcoded Package List**: The script explicitly lists the following packages for installation:
        -   `quanteda`
        -   `blake3`
        -   `gridExtra`
        -   `openssl`
        -   `aTSA`
        -   `KernSmooth` (often a base/recommended package, but explicit install doesn't hurt)
        -   `moments`
        -   `matrixStats`
        -   `Rmpfr`
        -   `randomForest`
        -   `SuppDists`
        -   `naturalsort`
        -   `sentimentr`
        -   `ggpolar`
        -   `XML`
        -   `RColorBrewer`

When this script is executed, R will attempt to download and install each of these packages along with their respective dependencies.

## Important Variables/Constants

This script does not define or use any variables. The list of packages to be installed is hardcoded directly within the call to `install.packages()`.

## Usage Examples

To use this script, you need to have R installed on your system.

**1. From the Command Line (using `Rscript`):**
   Navigate to the directory containing `install_packages.R` and run:
   ```bash
   Rscript install_packages.R
   ```
   R will then proceed to install the listed packages. Progress messages, warnings, and any errors from the installation process will be printed to the console.

**2. From an Interactive R Session:**
   You can also run the script from within an R interactive session:
   ```R
   # Assuming install_packages.R is in your current working directory
   source("install_packages.R")
   ```
   This will execute the `install.packages()` command as if you typed it directly into the console.

## Dependencies and Interactions

-   **R Installation**: A functional R installation is required to execute the script.
-   **Package Repositories (CRAN)**: The `install.packages()` function, by default, downloads packages from CRAN. Therefore, an active internet connection is necessary for the script to successfully download and install packages that are not already cached or available locally.
-   **System Dependencies**: Some R packages (e.g., `XML`, `openssl`) may depend on external system libraries (like `libxml2-dev` or `libssl-dev` on Linux systems). If these underlying system dependencies are not met, the installation of the corresponding R packages may fail. This script does not manage or install system-level dependencies.
-   **Write Permissions**: The user executing the R script must have write permissions to the R library directory where packages are to be installed. This location can vary based on the R setup (system-wide, user-specific, or project-local with tools like `renv`).
-   **Console Output**: The `install.packages()` function provides verbose output to the R console regarding the download progress, compilation (if any), and installation status of each package. Any warnings or errors encountered during the process will also be displayed there. The script itself does not include additional error handling or output redirection beyond what `install.packages()` provides.
```
