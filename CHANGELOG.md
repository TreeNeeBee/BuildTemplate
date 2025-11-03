# BuildTemplate Changelog

All notable changes to the LightAP BuildTemplate system will be documented in this file.

## [1.1.0] - 2024-01

### Major Refactoring - Modern CMake 3.x Practices

This release represents a comprehensive modernization of the entire BuildTemplate system, applying modern CMake practices, improving validation, and enhancing documentation.

### Added - Config.cmake.in

- **New validation functions:**
  - `lap_require_variable()` - Check if required variables are defined
  - `lap_validate_directory()` - Validate directory existence
  - `lap_print_config()` - Print configuration summary

- **New helper functions:**
  - `lap_collect_sources()` - Collect source files with multiple extensions

- **Enhanced features:**
  - Automatic compiler detection (GCC, Clang, MSVC)
  - Compiler-specific warning flags configuration
  - Platform-specific configuration support
  - Multiple inclusion guard (LAP_BUILD_TEMPLATE_INCLUDED)
  - CMake version validation
  - Enhanced `lap_configure_cxx_target()` with compiler warnings

### Changed - Library Templates

**SharedLibrary.cmake.in:**
- Migrated to modern CMake `target_*` commands
- Extended source file support: .cpp, .cxx, .cc, .c++, .c
- Extended header file support: .h, .hpp, .hxx
- Added export targets for `find_package()` support
- Implemented component-based installation (Runtime, Development)
- Added comprehensive variable validation
- Improved documentation with detailed header comments

**StaticLibrary.cmake.in:**
- Applied same modernization as SharedLibrary
- Changed to PUBLIC linking for dependency propagation
- ARCHIVE-based installation instead of LIBRARY
- Export targets for static library integration
- Consistent structure with SharedLibrary

### Changed - Executable Template

**Executable.cmake.in:**
- Fixed bugs in original implementation (target_link_directories syntax)
- Simplified installation rules (removed unnecessary header installation)
- Added optional config directory installation support
- Improved source file collection
- Modern CMake target_* commands throughout

### Changed - Test Template

**Test.cmake.in:**
- **Major Feature: Full CTest Integration**
  - `include(CTest)` for test framework
  - `add_test()` automatic registration
  - `set_tests_properties()` configuration
  
- **Test Configuration:**
  - Unit tests: 300s timeout, "unittest" label
  - Benchmarks: 600s timeout, "benchmark" + "performance" labels
  - Working directory configuration
  
- Enhanced source collection with GLOB_RECURSE
- Support for both unittest and benchmark in single template
- Comprehensive status messages

### Changed - Daemon Template

**Daemon.cmake.in:**
- Applied modern CMake practices
- **Major Enhancement: Automatic Systemd Service Registration**
  - Auto-generate installation/uninstallation scripts
  - Optional auto-register on `cmake --install`
  - Optional auto-enable and auto-start service
  - Complete lifecycle management support
- Enhanced systemd integration with 20+ configuration options:
  - Service behavior (Type, Restart, Watchdog)
  - Resource limits (Memory, CPU, Files)
  - Security hardening (User, Group, PrivateTmp, etc.)
  - Dependency management (Before, After, Requires)
  - Start rate limiting
- Improved error handling and status messages
- Added validation for all required variables
- Generated scripts: `install-<daemon>.sh`, `uninstall-<daemon>.sh`
- Consistent code style with other templates

**Systemd Templates:**
- `service.cmake.in` - Enhanced with 20+ configurable parameters
- `preset.cmake.in` - Auto-enable service support
- `install-service.sh.in` - NEW: Automatic service registration script
- `uninstall-service.sh.in` - NEW: Clean service removal script
- `systemd/README.md` - NEW: Comprehensive systemd configuration guide (11KB)

### Changed - Protobuf Template

**Protobuf.cmake.in:**
- Modernized protobuf integration
- Enhanced error handling and validation
- Support for proto import directories
- Export targets for find_package support
- Component-based installation (Development)
- Improved generated file structure
- Better status messages and debugging output

### Changed - Documentation

**README.md:**
- Complete rewrite with comprehensive documentation
- Added detailed API reference for all functions
- Best practices guide with recommended project structure
- Troubleshooting guide with common issues and solutions
- Bitbake/Yocto integration examples
- Component-based installation documentation
- Export targets and find_package usage examples
- CTest integration documentation
- All new v1.1.0 features documented

### Migration Guide from 1.0.0 to 1.1.0

#### Breaking Changes
None - v1.1.0 is fully backward compatible with v1.0.0

#### Recommended Updates

1. **Add enable_testing() for CTest:**
   ```cmake
   # In your top-level CMakeLists.txt
   enable_testing()
   include(CTest)
   ```

2. **Use validation functions:**
   ```cmake
   lap_require_variable(MODULE_NAME "MODULE_NAME is required")
   lap_validate_directory("${MODULE_SOURCE_DIR}" "MODULE_SOURCE_DIR")
   ```

3. **Leverage export targets:**
   ```cmake
   # Other projects can now use:
   find_package(YourModule REQUIRED)
   target_link_libraries(app PRIVATE YourModule::your_lib)
   ```

4. **Use component-based installation:**
   ```bash
   cmake --install . --component Runtime      # For end users
   cmake --install . --component Development  # For developers
   ```

### Statistics

- **Files Refactored:** 7 CMake templates + 1 README
- **Lines Added:** ~1,500+ (including comprehensive documentation)
- **Functions Added:** 6 new helper/validation functions
- **Features Added:** CTest integration, export targets, component installation
- **Bugs Fixed:** Executable.cmake.in target_link_directories syntax
- **Documentation:** Increased from ~250 lines to ~1,200 lines

## [1.0.0] - 2023-10

### Initial Release

- Basic CMake build template system
- C++14/17 automatic detection
- Submodule mode support
- Core templates: SharedLibrary, Executable, Test, Daemon, Protobuf
- Basic documentation

---

**Note:** Semantic versioning follows MAJOR.MINOR.PATCH:
- MAJOR: Incompatible API changes
- MINOR: Backward-compatible functionality additions
- PATCH: Backward-compatible bug fixes
