<!---
{
  "id": "a9c9305e-826b-41e2-8235-bb1315b07db9",
  "depends_on": ["2ad3a473-2965-40ad-ab86-891dcfeed508"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-12-04",
  "keywords": ["C++", "CMake", "std::filesystem", "Linking standard libraries", "Compiler flags"]
}
--->

# Using `std::filesystem` With Explicit Linking in CMake

> In this exercise you will learn how to use the C++17 `<filesystem>` standard library and how to explicitly link against `stdc++fs` when required by your compiler. Furthermore we will explore how to check compiler version requirements and conditionally link system libraries using CMake.

## Introduction

The `<filesystem>` header introduced in C++17 provides a robust, cross-platform API for navigating, querying, and modifying the file system. It includes types like `std::filesystem::path` and functions such as `exists()`, `create_directory()`, and `directory_iterator`. However, depending on your compiler and standard library version, using `<filesystem>` may require **explicit linking to an auxiliary library**, commonly `stdc++fs` (especially with GCC < 10).

In this exercise, you’ll create a small program that scans the current directory using `std::filesystem`, lists files and subdirectories, and prints their status. More importantly, you’ll configure your `CMakeLists.txt` to detect if the compiler requires explicit linking to `stdc++fs`, and add the necessary flags only when needed.

This practical example introduces CMake's generator expressions and version checks, preparing you for real-world projects where platform or compiler conditions dictate linking behavior.

### Further Readings and Other Sources

* C++17 Filesystem: [https://en.cppreference.com/w/cpp/filesystem](https://en.cppreference.com/w/cpp/filesystem)
* GCC bug tracker — Filesystem support: [https://gcc.gnu.org/bugzilla/show_bug.cgi?id=87321](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=87321)
* CMake: [https://cmake.org/cmake/help/latest/command/target_link_libraries.html](https://cmake.org/cmake/help/latest/command/target_link_libraries.html)
* YouTube: *C++17 Filesystem Crash Course* by The Cherno

---

## Tasks

### 1. Create project folders

```bash
mkdir cpp_filesystem
cd cpp_filesystem
mkdir -p src
```

### 2. Write the main program

Create `src/main.cpp`:

```cpp
#include <iostream>
#include <filesystem>

namespace fs = std::filesystem;

int main() {
    std::cout << "Listing contents of current directory:\n";
    for (const auto& entry : fs::directory_iterator(".")) {
        std::cout << (entry.is_directory() ? "[DIR]  " : "       ") << entry.path().filename() << "\n";
    }
    return 0;
}
```

This program uses `std::filesystem::directory_iterator` to print the contents of the current working directory.

### 3. Create the `CMakeLists.txt`

In the project root, add:

```cmake
cmake_minimum_required(VERSION 3.10)
project(FilesystemDemo VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(filesystemapp src/main.cpp)

# Explicitly link stdc++fs if using old GCC
if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 9.1)
        target_link_libraries(filesystemapp PRIVATE stdc++fs)
    endif()
endif()
```

Explanation:

* We use CMake version and compiler checks to determine if we need to link `stdc++fs`
* GCC 8 and early 9.x releases required manual linking for `<filesystem>`
* Clang and MSVC typically do not

### 4. Build and run the program

```bash
cmake -B build
cmake --build build
./build/filesystemapp
```

Expected output (sample):

```
Listing contents of current directory:
[DIR]  build
[DIR]  src
       CMakeLists.txt
```

---

## Questions

**1. Why does `std::filesystem` require special linking on some systems?**

<details>
<summary>Click to reveal answer</summary>

Older versions of libstdc++ (used by GCC) implemented `<filesystem>` in a separate library (`libstdc++fs.a`) that had to be linked manually. Newer versions integrate it directly into the standard library.

</details>

**2. What does `target_link_libraries(filesystemapp PRIVATE stdc++fs)` do?**

<details>
<summary>Click to reveal answer</summary>

It tells the linker to include the `stdc++fs` library, resolving any unresolved symbols for `std::filesystem` functions.

</details>

**3. How does CMake help make this portable across compilers?**

<details>
<summary>Click to reveal answer</summary>

CMake allows compiler identification (`CMAKE_CXX_COMPILER_ID`) and version checking (`VERSION_LESS`) so we can conditionally add linker flags or targets only when needed.

</details>

---

## Advice

As C++ evolves, standard libraries become more feature-rich, but compiler support often lags or differs in subtle ways. Always consult compiler documentation and test feature availability. Use CMake's ability to check compiler versions and properties to handle these inconsistencies in a maintainable way. This is especially important when sharing code across Linux, Windows, and macOS environments or when targeting CI pipelines with varying toolchains. Linking standard libraries explicitly may not be required in all environments, but learning to do so cleanly will help you avoid cryptic linker errors and ensure compatibility with older systems.
