# TryInstallDependencies.java

## Overview

`TryInstallDependencies.java` is a command-line Java utility designed to automate the process of downloading a list of specified Java dependencies using Apache Maven. It reads dependency coordinates (e.g., "groupId:artifactId:version") from an input file, then attempts to download each dependency's JAR file into a predefined local directory (`/java_deps` by default). The program provides real-time feedback by printing Maven's output and concludes with a summary of successfully downloaded and failed dependencies.

This utility is likely used within a containerized build or testing environment where Java dependencies need to be provisioned dynamically based on a list.

## Key Components

### Class: `TryInstallDependencies`

This is the sole class in the file and contains all the program logic.

-   **`private static final String JAVA_DEPS_PATH = "/java_deps";`**:
    -   A constant string defining the hardcoded path where Maven will be instructed to copy the downloaded dependency JAR files.

-   **`public static void main(String[] args)`**:
    -   The main entry point of the application.
    -   **Argument Parsing**: Expects exactly one command-line argument: the path to a text file listing the dependencies to be installed (one dependency per line). If the argument is not provided correctly, it prints a usage error to `System.err` and exits.
    -   **Dependency Processing**:
        -   Reads each line (dependency coordinate) from the input file.
        -   For each non-empty dependency string, it calls the `installDependency()` method.
        -   Maintains two lists: `successes` and `failures`, to track the outcome for each dependency.
        -   Prints an immediate status message ("Successfully installed: ..." or "Failed to install: ...") for each dependency.
    -   **Reporting**: After attempting to install all dependencies, it prints a summary to `System.out`, listing all successfully installed dependencies followed by all those that failed.
    -   **Error Handling**: Catches `IOException` that might occur during the reading of the dependencies file and exits with an error message.

-   **`private static boolean installDependency(String dependency)`**:
    -   **Purpose**: Attempts to download a single specified dependency using Maven.
    -   **Input**: `dependency` - A string representing the Maven coordinates of the dependency (e.g., "group:artifact:version").
    -   **Maven Invocation**:
        -   Constructs a Maven command using `ProcessBuilder`:
            ```
            mvn dependency:copy -Dartifact=<dependency_string> -DoutputDirectory=<JAVA_DEPS_PATH>
            ```
        -   This command tells Maven to find the specified `artifact` and copy its JAR file to the `outputDirectory`.
        -   `pb.redirectErrorStream(true)` is used to merge Maven's standard error into its standard output stream, ensuring all console output from Maven is captured together.
    -   **Process Execution and Output Handling**:
        -   Starts the Maven process.
        -   Reads and prints each line of output from the Maven process to `System.out` in real-time. This allows monitoring of Maven's progress and any issues.
        -   Waits for the Maven process to complete, with a timeout of 60 seconds using `process.waitFor(60, TimeUnit.SECONDS)`.
    -   **Success Determination**: The installation is considered successful if the Maven process:
        1.  Completes within the 60-second timeout.
        2.  Exits with a status code of `0`.
    -   Prints a specific success or failure message for the current dependency.
    -   **Return Value**: Returns `true` if the dependency was installed successfully, `false` otherwise.
    -   **Exception Handling**: Catches any general `Exception` during the process (e.g., if `mvn` command is not found) and treats it as an installation failure, printing an exception message.

## Important Variables/Constants

-   **`JAVA_DEPS_PATH`**: The hardcoded directory (`/java_deps`) where dependency JARs are downloaded. For the program to work as intended, this directory must be writable by the user running the program, or the path needs to be modified in the source code.
-   **Input File (via `args[0]`)**: The path to a text file where each line contains a Maven dependency coordinate string.

## Usage Examples

**1. Compilation:**
   Ensure you have a Java Development Kit (JDK) installed and accessible.
   ```bash
   javac TryInstallDependencies.java
   ```

**2. Prepare Dependencies File:**
   Create a text file (e.g., `my_dependencies.txt`) with content like:
   ```
   org.apache.commons:commons-lang3:3.12.0
   com.google.code.gson:gson:2.9.0
   # This one might fail if it doesn't exist or there's a typo
   nonexistent:dependency:1.0
   ```

**3. Ensure Target Directory Exists (Optional but Recommended):**
   By default, Maven will try to create the `outputDirectory` if it doesn't exist. However, permissions might be an issue.
   ```bash
   # Optional: create /java_deps if it doesn't exist and you have permissions
   # sudo mkdir /java_deps
   # sudo chown your_user:your_group /java_deps
   # Or, modify JAVA_DEPS_PATH in the source code to a path you have write access to.
   ```

**4. Running the Program:**
   Ensure Apache Maven (`mvn`) is installed and in your system's PATH.
   ```bash
   java TryInstallDependencies my_dependencies.txt
   ```
   The program will output the progress of each download attempt (including Maven's console output) and then a summary of successes and failures.

## Dependencies and Interactions

-   **Java Development Kit (JDK)**: Required to compile and execute the `.java` file.
-   **Apache Maven**: This is a critical runtime dependency. The program constructs and executes `mvn` commands. It specifically relies on the `dependency:copy` goal. Maven must be installed and configured correctly on the system where this program is run.
-   **File System**:
    -   Reads the input dependency list from the file path provided as a command-line argument.
    -   The Maven process invoked by this program will write the downloaded JAR files to the directory specified by the `JAVA_DEPS_PATH` constant (defaulting to `/java_deps`).
-   **Network Connectivity**: Maven requires internet access to download dependencies from remote repositories (e.g., Maven Central, JCenter, or other configured repositories).
-   **Operating System**: The program uses `ProcessBuilder` to execute external commands, which is a standard Java feature but interacts with the underlying OS to run Maven.
-   **Standard Output/Error Streams**:
    -   Prints detailed logs, including the entire output from Maven, to `System.out`.
    -   Prints final summaries of successful and failed installations to `System.out`.
    -   Prints usage instructions and file error messages to `System.err`.
```
