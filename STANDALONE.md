# LightAP Build Template

A reusable CMake build system template for C++17/C++14 projects with automatic feature detection.

## Overview

BuildTemplate provides standardized CMake configurations for building C++ projects with:
- Automatic C++17/C++14 detection and fallback
- Feature detection at compile-time using `__cplusplus` macro
- Support for shared libraries, executables, tests, and daemons
- Helper functions to reduce boilerplate
- Submodule support for standalone modules

## Features

✨ **Automatic C++ Standard Detection**
- Detects C++17 compiler support
- Falls back to C++14 if needed
- Runtime feature detection via `__cplusplus`

🔧 **Helper Functions**
- `lap_configure_cxx_target()` - Configure executables
- `lap_configure_cxx_library()` - Configure libraries with PIC

📦 **Build Templates**
- `SharedLibrary.cmake.in` - Shared library configuration
- `StaticLibrary.cmake.in` - Static library configuration
- `Executable.cmake.in` - Executable configuration
- `Test.cmake.in` - Test configuration with GTest
- `Daemon.cmake.in` - Daemon/service configuration
- `Protobuf.cmake.in` - Protobuf integration

🎯 **Bitbake Compatible**
- No CMake-specific feature macros
- All features detected at compile-time
- Standard `CMAKE_CXX_STANDARD` control

## Quick Start

### As Submodule (Recommended for Standalone Modules)

```bash
# Add to your module
cd YourModule
git submodule add https://github.com/yourorg/BuildTemplate.git BuildTemplate

# In CMakeLists.txt
include(BuildTemplate/Config.cmake.in)
```

### As Part of Multi-Module Project

```bash
# Directory structure
Project/
├── BuildTemplate/
└── modules/
    ├── ModuleA/
    └── ModuleB/

# In module CMakeLists.txt
include(../../BuildTemplate/Config.cmake.in)
```

## Usage Example

### Basic Module CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.10.2)

# Include BuildTemplate
include(BuildTemplate/Config.cmake.in)

project(MyModule)

# Module configuration
set(MODULE_NAME "MyModule")
set(MODULE_VERNO 1.0.0)
set(MODULE_SOURCE_CXX_DIR ${CMAKE_CURRENT_SOURCE_DIR}/source)
set(MODULE_EXTERNAL_LIB Boost::filesystem)
set(ENABLE_BUILD_SHARED_LIBRARY ON)

# Build library
include(BuildTemplate/SharedLibrary.cmake.in)

# Build tests
set(MODULE_TEST_DIR ${CMAKE_CURRENT_SOURCE_DIR}/test)
set(ENABLE_BUILD_UNITTEST ON)
set(MODULE_EXTERNAL_TEST_LIB 
    ${PLATFORM_SYSTEM_TARGET}_mymodule 
    GTest::GTest GTest::Main
)
include(BuildTemplate/Test.cmake.in)

# Custom targets with helper
add_executable(my_example examples/example.cpp)
lap_configure_cxx_target(TARGET my_example)
target_link_libraries(my_example PRIVATE ${PLATFORM_SYSTEM_TARGET}_mymodule)
```

### Dual-Mode Module (Standalone + Integrated)

```cmake
cmake_minimum_required(VERSION 3.10.2)

# Detect build mode
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    # Standalone mode
    project(MyModule VERSION 1.0.0)
    set(CMAKE_CXX_STANDARD 17)
    find_package(Threads REQUIRED)
    
    if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/BuildTemplate/Config.cmake.in")
        include("${CMAKE_CURRENT_SOURCE_DIR}/BuildTemplate/Config.cmake.in")
    else()
        message(FATAL_ERROR "BuildTemplate not found! Run: git submodule update --init")
    endif()
else()
    # Integrated mode
    include("../../BuildTemplate/Config.cmake.in")
endif()

# Module configuration continues...
```

## Configuration Options

### Common Options

| Variable | Default | Description |
|----------|---------|-------------|
| `PLATFORM_SYSTEM_TARGET` | "lap" | Platform prefix for targets |
| `CMAKE_CXX_STANDARD` | 17 | C++ standard (17 or 14) |
| `SDK_ROOT_DIR` | /usr/local | SDK installation directory |

### Build Type Options

| Option | Description |
|--------|-------------|
| `ENABLE_BUILD_SHARED_LIBRARY` | Build shared library |
| `ENABLE_BUILD_STATIC_LIBRARY` | Build static library |
| `ENABLE_BUILD_EXECUTABLE` | Build executable |
| `ENABLE_BUILD_UNITTEST` | Build unit tests |
| `ENABLE_BUILD_DAEMON` | Build daemon |

### Module Variables

| Variable | Purpose |
|----------|---------|
| `MODULE_NAME` | Module name (e.g., "Core") |
| `MODULE_VERNO` | Version number (e.g., "1.0.0") |
| `MODULE_SOURCE_CXX_DIR` | C++ source directory |
| `MODULE_EXTERNAL_LIB` | External library dependencies |
| `MODULE_TEST_DIR` | Test directory |
| `MODULE_EXTERNAL_TEST_LIB` | Test dependencies |

## Feature Detection

BuildTemplate uses compile-time feature detection:

```cpp
// In your headers
#if __cplusplus >= 201703L
    #include <optional>
    using Optional = std::optional;
#else
    #include <boost/optional.hpp>
    using Optional = boost::optional;
#endif
```

## Helper Functions

### lap_configure_cxx_target

Configures a C++ target with standard settings:

```cmake
add_executable(my_app main.cpp)
lap_configure_cxx_target(TARGET my_app)
# Sets: CXX_STANDARD, CXX_STANDARD_REQUIRED, CXX_EXTENSIONS
```

### lap_configure_cxx_library

Configures a library target with PIC:

```cmake
add_library(my_lib SHARED lib.cpp)
lap_configure_cxx_library(TARGET my_lib)
# Sets: All from above + POSITION_INDEPENDENT_CODE
```

## Bitbake Integration

BuildTemplate is fully compatible with Yocto/Bitbake:

```bash
# In your recipe
EXTRA_OECMAKE = "-DCMAKE_CXX_STANDARD=17"

# Or force C++14
EXTRA_OECMAKE = "-DCMAKE_CXX_STANDARD=14"
```

## Documentation

- [README.md](README.md) - Complete documentation with examples
- [Core Module](../modules/Core/) - Real-world usage example

## Example Projects

- **Core Module** - Complete standalone module implementation
  - See `modules/Core/CMakeLists.txt`
  - Supports both standalone and integrated builds
  - Includes tests, examples, and benchmarks

## License

Part of the LightAP project.

## Version

**v1.0.0** - 2025-11-03

## Changelog

- **2025-11-03**: Added standalone module submodule support
- **2025-10-29**: Initial CMake system with C++17/C++14 support
