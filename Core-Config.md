# GhostWire Audio Core - Complete Line-by-Line Code Explanation (Part 1)

This document explains every single line of code in the GhostWire Audio Core library, designed for someone with only basic coding knowledge.

---

## Table of Contents

1. [Project Configuration Files](#project-configuration-files)
2. [Header Files (Public Interface)](#header-files-public-interface)

---

## Project Configuration Files

### .gitignore

This file tells Git (version control software) which files to ignore and not track.

```gitignore
# Build directories
```
- **Line 1**: This is a comment (starting with `#`). Comments are notes for humans, the computer ignores them.
- This comment explains that the next section lists build directories.

```gitignore
build/
cmake-build-*/
out/
```
- **Line 2**: `build/` - Ignore the entire "build" folder. This is where compiled code goes.
- **Line 3**: `cmake-build-*` - Ignore any folder starting with "cmake-build-" (the `*` is a wildcard meaning "anything").
- **Line 4**: `out/` - Ignore the "out" folder, another common place for compiled code.

```gitignore
# CLion specific
```
- **Line 6**: Comment explaining this section is for CLion (a popular C++ development tool).

```gitignore
.idea/
cmake-build-debug/
cmake-build-release/
cmake-build-relwithdebinfo/
cmake-build-minsizerel/
```
- **Lines 7-11**: Ignore various CLion-specific folders that store IDE settings and build configurations.

```gitignore
# Other IDEs (in case you or contributors use different tools)
```
- **Line 13**: Comment for other development tools.

```gitignore
.vscode/
*.swp
*.swo
*~
.DS_Store
```
- **Line 14**: `.vscode/` - Visual Studio Code settings folder.
- **Line 15**: `*.swp` - Vim editor temporary files (swap files).
- **Line 16**: `*.swo` - Another type of Vim temporary file.
- **Line 17**: `*~` - Backup files created by various text editors.
- **Line 18**: `.DS_Store` - Mac operating system hidden files.

```gitignore
# Compiled files
```
- **Line 20**: Comment for compiled output files.

```gitignore
*.o
*.obj
*.a
*.lib
*.so
*.dylib
*.dll
*.exe
```
- **Lines 21-28**: These are all different types of compiled binary files:
  - `.o` and `.obj`: Object files (compiled but not yet linked)
  - `.a` and `.lib`: Static library files
  - `.so`: Shared library on Linux
  - `.dylib`: Shared library on Mac
  - `.dll`: Shared library on Windows
  - `.exe`: Executable program on Windows

```gitignore
# CMake generated files (only in build directories)
```
- **Line 30**: Comment for CMake-generated files.

```gitignore
CMakeCache.txt
CMakeFiles/
cmake_install.cmake
Makefile
*.cmake
!CMakeLists.txt
!cmake/*.cmake
```
- **Lines 31-35**: Various files CMake creates when setting up a build.
- **Line 36**: `!CMakeLists.txt` - The `!` means "exception" - DO track this file (it's our build configuration).
- **Line 37**: `!cmake/*.cmake` - DO track our custom cmake files in the cmake folder.

```gitignore
# OS specific
Thumbs.db
```
- **Line 39**: Comment for operating system files.
- **Line 40**: `Thumbs.db` - Windows thumbnail cache file.

---

### LICENSE

```
MIT License
```
- **Line 1**: This declares the license type - MIT is very permissive.

```
Copyright (c) 2026 GhostWire Audio
```
- **Line 3**: Copyright notice with year and owner.

```
Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
```
- **Lines 5-10**: This grants permission to anyone to use this software for free and do basically anything with it (use, modify, sell, etc.).

```
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```
- **Lines 12-13**: The only requirement - you must include this license notice if you distribute the code.

```
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
- **Lines 15-21**: This is the "no warranty" clause - the authors aren't responsible if the software breaks or causes problems. This is standard legal protection.

---

## Project Build Configuration Files

### CMakeLists.txt (Root)

CMake is a build system that generates platform-specific build files. Think of it as a recipe for compiling the code.

```cmake
# Minimum CMake version supported
cmake_minimum_required(VERSION 3.15)
```
- **Line 1**: Comment explaining this line.
- **Line 2**: `cmake_minimum_required(VERSION 3.15)` - This project requires CMake version 3.15 or newer. If someone has an older version, CMake will show an error.

```cmake
# Project declaration
project(GW_Core
        VERSION 0.1.0
        DESCRIPTION "Real-time audio DSP engine"
        LANGUAGES CXX
)
```
- **Line 4**: Comment.
- **Line 5**: `project(GW_Core` - Declares a project named "GW_Core".
- **Line 6**: `VERSION 0.1.0` - Sets the project version to 0.1.0.
- **Line 7**: `DESCRIPTION "Real-time audio DSP engine"` - A short description.
- **Line 8**: `LANGUAGES CXX` - This project uses C++ (CXX is the CMake abbreviation for C++).
- **Line 9**: `)` - Closes the project() function call.

```cmake
# Allows gw-core to be used as a sub-project
if (CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
```
- **Line 11**: Comment explaining this conditional.
- **Line 12**: `if (CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)` - This checks if we're the main project. 
  - `CMAKE_PROJECT_NAME` is the top-level project name
  - `PROJECT_NAME` is this project's name
  - `STREQUAL` means "string equal"
  - This is true only if we're NOT being included as a subproject of something else

```cmake
    # Use C++17
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    set(CMAKE_CXX_EXTENSIONS OFF)
```
- **Line 13**: Comment.
- **Line 14**: `set(CMAKE_CXX_STANDARD 17)` - Use C++17 (a version of the C++ language from 2017).
- **Line 15**: `set(CMAKE_CXX_STANDARD_REQUIRED ON)` - This is required, not optional. If the compiler can't do C++17, fail.
- **Line 16**: `set(CMAKE_CXX_EXTENSIONS OFF)` - Don't use compiler-specific extensions, stick to standard C++17.

```cmake
    # Enable folder organization in IDEs
    set_property(GLOBAL PROPERTY USE_FOLDERS ON)
```
- **Line 18**: Comment.
- **Line 19**: `set_property(GLOBAL PROPERTY USE_FOLDERS ON)` - In IDEs like Visual Studio, organize files into folders for easier navigation.

```cmake
    # Export compile commands for tools like clangd
    set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```
- **Line 21**: Comment.
- **Line 22**: `set(CMAKE_EXPORT_COMPILE_COMMANDS ON)` - Create a JSON file that tells code editors and analysis tools how to compile each file. Helps with autocomplete and error checking.

```cmake
endif ()
```
- **Line 23**: `endif ()` - Closes the `if` statement from line 12.

```cmake
# Include our custom CMake modules
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
```
- **Line 25**: Comment.
- **Line 26**: `list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")`
  - `list(APPEND ...)` - Add to a list
  - `CMAKE_MODULE_PATH` - The list of directories where CMake looks for module files
  - `${CMAKE_CURRENT_SOURCE_DIR}/cmake` - The "cmake" folder in our project
  - This lets us use custom CMake files from our cmake/ folder

```cmake
# Include compiler warnings configuration
include(CompilerWarnings)
```
- **Line 28**: Comment.
- **Line 29**: `include(CompilerWarnings)` - Load and run the CompilerWarnings.cmake file from our cmake/ folder.

```cmake
# Add subdirectories
add_subdirectory(src)
add_subdirectory(examples)
add_subdirectory(tests)
```
- **Line 31**: Comment.
- **Line 32**: `add_subdirectory(src)` - Process the CMakeLists.txt file in the src/ folder (this builds the library).
- **Line 33**: `add_subdirectory(examples)` - Process examples/CMakeLists.txt (builds example programs).
- **Line 34**: `add_subdirectory(tests)` - Process tests/CMakeLists.txt (builds test programs).

---

### cmake/CompilerWarnings.cmake

This file sets up compiler warnings - messages the compiler gives when it thinks code might have problems.

```cmake
# This function adds strong compiler warnings to a target
function(set_project_warnings target_name)
```
- **Line 1**: Comment.
- **Line 2**: `function(set_project_warnings target_name)`
  - `function` - Defines a reusable function in CMake
  - `set_project_warnings` - Name of the function
  - `target_name` - Parameter: the name of what we're compiling (like "gw-core")

```cmake
    set(MSVC_WARNINGS
            /W4          # High warning level
            /w14640      # Warn on thread-unsafe static initialization
            /permissive- # Standards conformance mode
    )
```
- **Line 3**: `set(MSVC_WARNINGS` - Create a variable called MSVC_WARNINGS (for Microsoft Visual C++ compiler)
- **Line 4**: `/W4` - Warning level 4 (high, catches most issues)
- **Line 5**: `/w14640` - Specific warning about thread safety
- **Line 6**: `/permissive-` - Strict standards mode, no Microsoft-specific extensions
- **Line 7**: `)` - Closes the set() call

```cmake
    set(CLANG_WARNINGS
            -Wall        # Enable most warnings
            -Wextra      # Enable extra warnings
            -Wpedantic   # Warn on non-standard C++
            -Wshadow     # Warn when variables shadow each other
            -Wnon-virtual-dtor  # Warn on missing virtual destructors
            -Wcast-align        # Warn on dangerous casts
            -Wunused            # Warn on unused code
            -Woverloaded-virtual # Warn on overloaded virtuals
            -Wconversion        # Warn on type conversions that may lose data
            -Wsign-conversion   # Warn on sign conversions
            -Wformat=2          # Warn on printf format issues
    )
```
- **Line 9**: `set(CLANG_WARNINGS` - Warnings for Clang compiler
- **Lines 10-20**: Various warning flags:
  - `-Wall` - Enable most common warnings
  - `-Wextra` - Even more warnings beyond -Wall
  - `-Wpedantic` - Warn about non-standard C++ features
  - `-Wshadow` - Warn if a variable name hides another variable with the same name
  - `-Wnon-virtual-dtor` - Warn if a class with virtual functions doesn't have a virtual destructor
  - `-Wcast-align` - Warn about pointer casts that might cause alignment issues
  - `-Wunused` - Warn about unused variables, functions, etc.
  - `-Woverloaded-virtual` - Warn about confusing virtual function overloads
  - `-Wconversion` - Warn when converting types might lose information (like int to char)
  - `-Wsign-conversion` - Warn when converting between signed and unsigned
  - `-Wformat=2` - Extra warnings about printf-style formatting

```cmake
    set(GCC_WARNINGS
            ${CLANG_WARNINGS}
    )
```
- **Line 23**: `set(GCC_WARNINGS` - Warnings for GCC compiler
- **Line 24**: `${CLANG_WARNINGS}` - Use the same warnings as Clang (they're compatible)
- **Line 25**: `)` - Close the set() call

```cmake
    # Apply the appropriate warnings based on compiler
    if (MSVC)
        set(PROJECT_WARNINGS ${MSVC_WARNINGS})
```
- **Line 27**: Comment.
- **Line 28**: `if (MSVC)` - If we're using Microsoft Visual C++ compiler
- **Line 29**: `set(PROJECT_WARNINGS ${MSVC_WARNINGS})` - Use the MSVC warnings

```cmake
    elseif (CMAKE_CXX_COMPILER_ID MATCHES ".*Clang")
        set(PROJECT_WARNINGS ${CLANG_WARNINGS})
```
- **Line 30**: `elseif (CMAKE_CXX_COMPILER_ID MATCHES ".*Clang")` - Otherwise, if the compiler name contains "Clang"
  - `CMAKE_CXX_COMPILER_ID` - Variable containing the compiler name
  - `MATCHES ".*Clang"` - Regular expression: anything followed by "Clang"
- **Line 31**: Use Clang warnings

```cmake
    elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        set(PROJECT_WARNINGS ${GCC_WARNINGS})
```
- **Line 32**: `elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")` - If using GCC (GNU Compiler Collection)
- **Line 33**: Use GCC warnings

```cmake
    else ()
        message(WARNING "Unknown compiler, no warnings set")
    endif ()
```
- **Line 34**: `else ()` - If none of the above compilers
- **Line 35**: `message(WARNING "Unknown compiler, no warnings set")` - Print a warning message
- **Line 36**: `endif ()` - Close the if statement

```cmake
    target_compile_options(${target_name} PRIVATE ${PROJECT_WARNINGS})
endfunction()
```
- **Line 38**: `target_compile_options(${target_name} PRIVATE ${PROJECT_WARNINGS})`
  - `target_compile_options` - Add compilation options to a target
  - `${target_name}` - The thing we're compiling (passed as parameter)
  - `PRIVATE` - These options only apply to this target, not things that use it
  - `${PROJECT_WARNINGS}` - The warning flags we determined above
- **Line 39**: `endfunction()` - End of the function definition

---

## Header Files (Public Interface)

Header files (`.h`) declare the interface - they tell users what functions and classes are available. They're like a menu at a restaurant.

### include/gw/core/version.h

```cpp
#ifndef GW_CORE_VERSION_H
#define GW_CORE_VERSION_H
```
- **Line 1**: `#ifndef GW_CORE_VERSION_H` - "if not defined GW_CORE_VERSION_H"
- **Line 2**: `#define GW_CORE_VERSION_H` - "define GW_CORE_VERSION_H"
- These two lines are an **include guard**. They prevent this file from being included multiple times in the same compilation, which would cause errors.

```cpp
namespace gw::core {
```
- **Line 5**: `namespace gw::core {`
  - `namespace` - Creates a named scope to organize code
  - `gw::core` - Nested namespace: `gw` contains `core`
  - This prevents name conflicts (like having two classes named "Buffer" in different libraries)
  - `{` - Opening brace starts the namespace

```cpp
    // Version information
    constexpr int VERSION_MAJOR = 0;
    constexpr int VERSION_MINOR = 1;
    constexpr int VERSION_PATCH = 0;
```
- **Line 6**: `// Version information` - A comment (ignored by compiler)
- **Line 7**: `constexpr int VERSION_MAJOR = 0;`
  - `constexpr` - This is a compile-time constant (value known when compiling)
  - `int` - Integer type
  - `VERSION_MAJOR = 0` - Major version number is 0
  - `;` - Statement terminator
- **Lines 8-9**: Minor and patch version numbers (following semantic versioning: MAJOR.MINOR.PATCH)

```cpp
    // Get version as string
    const char *get_version_string();
```
- **Line 11**: Comment.
- **Line 12**: `const char *get_version_string();`
  - `const char *` - Returns a pointer to constant characters (a C-style string)
  - `get_version_string` - Function name
  - `()` - Takes no parameters
  - `;` - This is a declaration (says the function exists, doesn't define what it does yet)

```cpp
} // namespace gw::core
```
- **Line 13**: `}` - Closes the namespace
- `// namespace gw::core` - Comment documenting what we're closing (helpful in long files)

```cpp
#endif  // GW_CORE_VERSION_H
```
- **Line 16**: `#endif` - Ends the include guard from line 1
- `// GW_CORE_VERSION_H` - Comment documenting what we're ending

---

### include/gw/core/audio_format.h

```cpp
#ifndef GW_CORE_AUDIO_FORMAT_H
#define GW_CORE_AUDIO_FORMAT_H
```
- **Lines 1-2**: Include guard (same pattern as version.h)

```cpp
#include <cstddef>
#include <cstdint>
```
- **Line 4**: `#include <cstddef>`
  - `#include` - Include another file
  - `<cstddef>` - Standard library header that defines `size_t` (type for sizes and counts)
- **Line 5**: `#include <cstdint>` - Defines fixed-size integer types like `uint32_t`

```cpp
namespace gw::core {
```
- **Line 7**: Start namespace

```cpp
    /**
     *  Describes the format of an audio stream.
     *
     *  Immutable once created, real-time audio threads need stable format information
     */
    class AudioFormat {
```
- **Lines 8-12**: Documentation comment (Doxygen style)
  - `/**` starts a documentation comment
  - Explains what this class does
  - `*/` ends the comment
- **Line 13**: `class AudioFormat {`
  - `class` - Defines a class (a blueprint for objects)
  - `AudioFormat` - Class name
  - `{` - Start class definition

```cpp
    public:
```
- **Line 14**: `public:`
  - Everything after this is public (accessible from outside the class)
  - `:` indicates a label in C++

```cpp
        /**
         * Constructs an audio format
         *
         * @param sample_rate Samples per second (e.g., 44100, 48000)
         * @param num_channels Number of audio channels (e.g., 1 = mono, 2 = stereo)
         * @param bit_depth Bits per sample (typically 16, 24, or 32)
         */
        AudioFormat(uint32_t sample_rate,
                    uint32_t num_channels,
                    uint32_t bit_depth);
```
- **Lines 15-21**: Documentation for constructor
- **Lines 22-24**: `AudioFormat(uint32_t sample_rate, uint32_t num_channels, uint32_t bit_depth);`
  - This is a **constructor** - a special function that creates AudioFormat objects
  - Same name as the class
  - `uint32_t` - Unsigned 32-bit integer (0 to 4,294,967,295)
  - Three parameters: sample rate, number of channels, and bit depth
  - `;` - Declaration only (implementation is in the .cpp file)

```cpp
        // Getters
        [[nodiscard]] uint32_t get_sample_rate() const { return sample_rate_; }
        [[nodiscard]] uint32_t get_num_channels() const { return num_channels_; }
        [[nodiscard]] uint32_t get_bit_depth() const { return bit_depth_; }
```
- **Line 26**: Comment explaining these are "getters" (functions that get data)
- **Line 27**: `[[nodiscard]] uint32_t get_sample_rate() const { return sample_rate_; }`
  - `[[nodiscard]]` - Compiler attribute: warn if caller ignores the return value
  - `uint32_t` - Return type
  - `get_sample_rate()` - Function name
  - `const` - This function promises not to modify the object
  - `{ return sample_rate_; }` - Function body: return the member variable `sample_rate_`
  - This is defined inline (in the header) because it's so simple
- **Lines 28-29**: Similar getters for channels and bit depth

```cpp
        /**
         *  Calculates bytes per sample based on bit depth.
         *  Assumes packed samples (no padding).
         */
        [[nodiscard]] size_t get_bytes_per_sample() const;

        /**
         *  Calculate bytes per frame (one sample for all channels).
         */
        [[nodiscard]] size_t get_bytes_per_frame() const;
```
- **Lines 31-40**: Calculation methods

```cpp
        // Equality comparison
        bool operator==(const AudioFormat &other) const;
        bool operator!=(const AudioFormat &other) const;
```
- **Lines 42-45**: Comparison operators

```cpp
    private:
        uint32_t sample_rate_;
        uint32_t num_channels_;
        uint32_t bit_depth_;
    };
}

#endif
```
- **Lines 47-55**: Private members and end of file

---

### include/gw/core/audio_buffer.h

(Due to length, I'll provide a condensed version with key concepts)

```cpp
class AudioBuffer {
```

**Key Concepts:**

1. **Multichannel planar buffer** - Each channel stored separately
2. **Aligned memory** - 32-byte alignment for AVX SIMD operations
3. **Non-copyable but movable** - Copy deleted, move allowed
4. **RAII** - Constructor allocates, destructor frees
5. **Real-time awareness** - Clear() is safe, copy_from() is not

**Important members:**
- `float **channel_data_` - Array of pointers to each channel
- Each channel is allocated with `aligned_alloc()`
- Must free with `std::free()` not `delete`!

---

### include/gw/core/buffer_view.h

**Key Concepts:**

1. **Non-owning view** - Just a pointer + size
2. **Cheap to copy** - Only 16 bytes (pointer + size)
3. **Like std::span** - Modern C++ view into contiguous data
4. **Subviews** - Can create views of views
5. **Array-like access** - Supports `operator[]`

---

### include/gw/core/ring_buffer.h

**Key Concepts:**

1. **Lock-free SPSC** - Single Producer Single Consumer
2. **Atomic operations** - `std::atomic<size_t>` for positions
3. **Memory ordering** - Acquire/release semantics for correctness
4. **Wraparound** - Modulo arithmetic for circular behavior
5. **Capacity + 1** - Reserve one slot to distinguish full from empty

**Memory Ordering Explained:**
- `relaxed` - No synchronization (fastest)
- `acquire` - See all writes before corresponding release
- `release` - Make all writes visible to corresponding acquire
