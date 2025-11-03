# LightAP CMake Build System# LightAP CMake Build System



**Version:** 1.1.0  ## 📋 概述

**License:** MIT  

**Language:** CMake 3.10.2+LightAP的CMake构建系统支持C++17优先，自动回退到C++14。所有C++17特性检测通过头文件中的`__cplusplus`宏完成，与Bitbake等构建系统完全兼容。



## 📋 概述## 🏗️ 系统架构



LightAP BuildTemplate 是一套现代化的CMake构建模板系统，为LightAP项目提供标准化、模块化的构建配置。系统支持：```

BuildTemplate/

- ✅ **现代CMake 3.x实践** - target_* commands, 导出目标, 组件化安装├── Config.cmake.in                     # 核心配置（包含辅助函数）

- ✅ **多C++标准支持** - C++14/17/20/23自动检测与回退├── SharedLibrary.cmake.in              # 共享库构建模板

- ✅ **模块化设计** - 可作为Git submodule独立使用├── Executable.cmake.in                 # 可执行文件构建模板

- ✅ **完整的构建类型** - 库、可执行文件、测试、守护进程、Protobuf├── Test.cmake.in                       # 测试构建模板

- ✅ **CTest集成** - 自动化测试框架支持├── Daemon.cmake.in                     # 守护进程构建模板

- ✅ **Systemd集成** - 守护进程服务管理└── README.md                           # 本文档

- ✅ **Find_package支持** - 导出CMake配置供其他项目使用```



## 🏗️ 系统架构## 🔧 核心特性



```### 1. C++标准自动检测

BuildTemplate/

├── Config.cmake.in              # 核心配置与辅助函数 (v1.1.0)**根CMakeLists.txt**负责检测编译器支持：

├── SharedLibrary.cmake.in       # 共享库构建模板 (.so)- ✅ C++17 → 编译器支持则使用

├── StaticLibrary.cmake.in       # 静态库构建模板 (.a)- ⚠️ C++14 → 自动回退

├── Executable.cmake.in          # 可执行文件构建模板

├── Test.cmake.in                # 单元测试与基准测试模板 (CTest)**特性检测**：在头文件中使用`__cplusplus`宏

├── Daemon.cmake.in              # 守护进程构建模板 (Systemd)- `__cplusplus >= 201703L` → 使用 std::optional/variant/filesystem

├── Protobuf.cmake.in            # Protobuf代码生成与库构建- 否则 → 使用 Boost 等价物

├── systemd/                     # Systemd服务配置模板

│   ├── service.cmake### 2. 辅助函数

│   ├── preset.cmake

│   └── mklink.sh.cmake`Config.cmake.in`提供两个辅助函数：

├── README.md                    # 本文档

├── STANDALONE.md                # 独立模块使用指南#### `lap_configure_cxx_target(TARGET name)`

└── build.sh                     # 标准构建脚本为可执行文件设置标准C++配置：

``````cmake

add_executable(my_app main.cpp)

## 🚀 快速开始lap_configure_cxx_target(TARGET my_app)

```

### 基本用法

#### `lap_configure_cxx_library(TARGET name)`

```cmake为库设置配置（包含PIC等）：

cmake_minimum_required(VERSION 3.10.2)```cmake

add_library(my_lib SHARED lib.cpp)

# 1. 包含核心配置lap_configure_cxx_library(TARGET my_lib)

include(BuildTemplate/Config.cmake.in)```



# 2. 定义项目### 3. 构建模板

project(MyModule VERSION 1.0.0)

所有`.cmake.in`模板自动应用辅助函数：

# 3. 配置模块变量- `SharedLibrary.cmake.in` → 自动配置共享库

set(MODULE_NAME "MyModule")- `Executable.cmake.in` → 自动配置可执行文件

set(MODULE_VERNO ${PROJECT_VERSION})- `Test.cmake.in` → 自动配置测试目标

set(MODULE_SOURCE_CXX_DIR ${CMAKE_CURRENT_SOURCE_DIR}/source/src)

set(MODULE_EXTERNAL_LIB Threads::Threads)## 📝 模块使用示例



# 4. 包含构建模板```cmake

set(ENABLE_BUILD_SHARED_LIBRARY ON)cmake_minimum_required(VERSION "3.10.2")

include(BuildTemplate/SharedLibrary.cmake.in)include(../../BuildTemplate/Config.cmake.in)

```

project(MyModule)

### 独立模块模式（Submodule）

set(MODULE_NAME "MyModule")

```bashset(MODULE_VERNO 1.0.0)

# 添加BuildTemplate为submoduleset(MODULE_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR})

git submodule add <BuildTemplate-repo-url> BuildTemplateset(MODULE_SOURCE_CXX_DIR ${MODULE_ROOT_DIR}/source)

git submodule update --init --recursiveset(MODULE_EXTERNAL_LIB Boost::filesystem)

set(ENABLE_BUILD_SHARED_LIBRARY ON)

# 构建

./build.sh          # Release构建# 自动应用C++标准配置

./build.sh debug    # Debug构建include(../../BuildTemplate/SharedLibrary.cmake.in)

```

# 测试

详见 [STANDALONE.md](STANDALONE.md) 获取完整指南。set(MODULE_TEST_DIR ${MODULE_ROOT_DIR}/test)

set(ENABLE_BUILD_UNITTEST ON)

## 🔧 核心特性详解set(MODULE_EXTERNAL_TEST_LIB 

    ${PLATFORM_SYSTEM_TARGET}_mymodule 

### 1. Config.cmake.in - 核心配置系统    GTest::GTest GTest::Main

)

**Version:** 1.1.0  include(../../BuildTemplate/Test.cmake.in)

**功能：** 提供全局配置、验证函数和辅助函数

# 自定义目标

#### 1.1 验证函数add_executable(my_example examples/example.cpp)

lap_configure_cxx_target(TARGET my_example)

```cmake```

# 检查必需变量是否定义

lap_require_variable(MODULE_NAME "MODULE_NAME is required")## 💡 头文件中的特性检测



# 验证目录是否存在```cpp

lap_validate_directory("${MODULE_SOURCE_DIR}" "MODULE_SOURCE_DIR")// CTypedef.hpp - 自动检测C++版本

#if __cplusplus >= 201703L

# 打印配置摘要    #include <optional>

lap_print_config()    template<typename T>

```    using Optional = ::std::optional<T>;

#else

#### 1.2 辅助函数    #include <boost/optional.hpp>

    template<typename T>

```cmake    using Optional = ::boost::optional<T>;

# 为可执行文件/库配置C++标准和编译选项#endif

lap_configure_cxx_target(TARGET my_target)```



# 为库配置（包含POSITION_INDEPENDENT_CODE等）## 🔍 调试

lap_configure_cxx_library(TARGET my_lib)

构建时会输出：

# 收集源文件（支持多种C++扩展名）```

lap_collect_sources(-- C++17 support detected

    OUTPUT_VAR SOURCES-- [LightAP] Using C++ standard: 17

    DIRECTORIES ${SRC_DIR1} ${SRC_DIR2}-- [LightAP] Configured target 'lap_core' with C++17

)```

```

## 🎯 与Bitbake集成

#### 1.3 C++标准自动检测

系统设计完全兼容Bitbake：

系统自动检测编译器支持并选择最高可用标准：1. **不依赖CMake宏定义** - 所有特性检测在头文件中

2. **标准C++版本控制** - 只需设置`CMAKE_CXX_STANDARD`

```cmake3. **灵活的编译器选项** - 通过`CMAKE_CXX_FLAGS`控制

# 检测顺序：C++23 → C++20 → C++17 → C++14

-- [LightAP] Detected C++17 supportBitbake recipe示例：

-- [LightAP] Using C++ standard: 17```bash

```EXTRA_OECMAKE = "-DCMAKE_CXX_STANDARD=17"

# 或强制C++14

在代码中检测特性：EXTRA_OECMAKE = "-DCMAKE_CXX_STANDARD=14"

```

```cpp

#if __cplusplus >= 201703L## � 独立模块使用（Submodule模式）

    #include <optional>

    using std::optional;BuildTemplate支持作为Git submodule被独立模块引用，实现模块的独立发布和构建。

#else

    #include <boost/optional.hpp>### 使用场景

    using boost::optional;

#endif当你想发布独立模块（如Core模块）时：

```1. 模块可以独立编译，不依赖整个LightAP项目

2. BuildTemplate作为submodule被引入

#### 1.4 编译器特定优化3. 模块的CMakeLists.txt自动检测构建模式（独立/集成）



自动检测编译器并应用最佳实践警告选项：### 设置Submodule



- **GCC/Clang**: `-Wall -Wextra -Wpedantic -Wshadow -Wconversion`在独立模块目录中：

- **MSVC**: `/W4 /WX`

```bash

### 2. SharedLibrary.cmake.in - 共享库构建# 添加BuildTemplate作为submodule

cd modules/Core

**用途：** 构建 `.so` 动态链接库git submodule add ../../BuildTemplate BuildTemplate

git submodule update --init --recursive

#### 必需变量```



```cmake### 模块CMakeLists.txt示例

set(MODULE_NAME "Core")                          # 模块名

set(MODULE_VERNO "1.0.0")                       # 版本号```cmake

set(MODULE_SOURCE_CXX_DIR "source/src")         # C++源码目录cmake_minimum_required(VERSION 3.10.2)

set(ENABLE_BUILD_SHARED_LIBRARY ON)            # 启用共享库构建

```# ==============================================================================

# 检测构建模式

#### 可选变量# ==============================================================================

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)

```cmake    # 独立模式

set(MODULE_INCLUDE_DIR "source/inc")            # 头文件目录    set(MODULE_STANDALONE_BUILD ON)

set(MODULE_EXTERNAL_LIB "Threads::Threads")     # 外部依赖库    project(MyModule VERSION 1.0.0 LANGUAGES CXX)

set(MODULE_EXTERNAL_INCLUDE_DIR "/usr/include") # 外部头文件    

set(BUILD_WITH_STRIP ON)                        # Release模式下strip符号    # 配置C++标准和依赖

```    set(CMAKE_CXX_STANDARD 17)

    find_package(Threads REQUIRED)

#### 特性    # ... 其他依赖

    

- ✅ 自动源文件收集（.cpp, .cxx, .cc, .c++, .c）    # 包含BuildTemplate（从submodule）

- ✅ 头文件收集（.h, .hpp, .hxx）    if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/BuildTemplate/Config.cmake.in")

- ✅ SOVERSION管理（从版本号提取主版本）        include("${CMAKE_CURRENT_SOURCE_DIR}/BuildTemplate/Config.cmake.in")

- ✅ 导出CMake配置（`find_package`支持）        set(BUILD_TEMPLATE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/BuildTemplate")

- ✅ 组件化安装（Runtime, Development）    else()

        message(FATAL_ERROR "BuildTemplate not found! Run: git submodule update --init")

#### 生成文件    endif()

else()

```    # 集成模式（作为LightAP的一部分）

lib<module>.so          → 主要符号链接    set(MODULE_STANDALONE_BUILD OFF)

lib<module>.so.1        → 兼容版本符号链接    include("../../BuildTemplate/Config.cmake.in")

lib<module>.so.1.0.0    → 实际库文件    set(BUILD_TEMPLATE_DIR "../../BuildTemplate")

<Module>Targets.cmake   → CMake导出配置endif()

```

# 模块配置

### 3. StaticLibrary.cmake.in - 静态库构建set(MODULE_NAME "MyModule")

set(MODULE_VERNO 1.0.0)

**用途：** 构建 `.a` 静态链接库# ...



用法与SharedLibrary相同，仅需设置：# 使用BuildTemplate模板

include("${BUILD_TEMPLATE_DIR}/SharedLibrary.cmake.in")

```cmake```

set(ENABLE_BUILD_STATIC_LIBRARY ON)

include(BuildTemplate/StaticLibrary.cmake.in)### 独立构建流程

```

```bash

**关键差异：**# 克隆独立模块（包含submodule）

- 安装为 `ARCHIVE` 而非 `LIBRARY`git clone --recursive https://github.com/yourorg/MyModule.git

- 外部库使用 `PUBLIC` 链接（传递给使用者）cd MyModule



### 4. Executable.cmake.in - 可执行文件构建# 或者克隆后初始化submodule

git clone https://github.com/yourorg/MyModule.git

**用途：** 构建可执行二进制程序cd MyModule

git submodule update --init --recursive

#### 必需变量

# 构建

```cmakemkdir build && cd build

set(MODULE_NAME "MyApp")cmake ..

set(MODULE_EXECUTABLE_DIR "source/src")make -j$(nproc)

set(MODULE_EXTERNAL_EXECUTABLE_LIB "lap_core")

set(ENABLE_BUILD_EXECUTABLE ON)# 运行测试

```ctest



#### 可选变量# 安装

sudo make install

```cmake```

set(MODULE_EXECUTABLE_TARGET "myapp")           # 自定义目标名

set(MODULE_INSTALL_CONFIG_DIR "config")         # 配置文件目录### 优势

set(BUILD_WITH_STRIP ON)                        # Strip符号表

```✅ **独立发布** - 模块可以独立发布到GitHub等平台  

✅ **统一构建** - 使用相同的构建模板和标准  

#### 特性✅ **灵活集成** - 既可独立使用，也可作为LightAP的一部分  

✅ **版本控制** - BuildTemplate版本通过submodule精确控制  

- ✅ 自动源文件收集✅ **易于维护** - 构建系统更新自动同步

- ✅ 可选配置文件安装

- ✅ 仅安装Runtime组件（无Development头文件）### 实际案例

- ✅ 与库分离的包含路径管理

参考 **Core模块** 的实现：

### 5. Test.cmake.in - 测试框架- `modules/Core/CMakeLists.txt` - 支持独立/集成双模式

- `modules/Core/.gitmodules` - Submodule配置

**用途：** 构建单元测试和性能基准测试（集成CTest）- `modules/Core/build.sh` - 独立构建脚本



#### 单元测试配置## 📚 更多信息



```cmake- Core模块独立构建示例: `../modules/Core/build.sh`

set(ENABLE_BUILD_UNITTEST ON)- Core模块文档: `../modules/Core/README.md`

set(MODULE_TEST_DIR "test/unittest")

set(MODULE_EXTERNAL_TEST_LIB ## 📅 更新历史

    lap_core 

    GTest::GTest - **2025-11-03**: 添加独立模块submodule支持

    GTest::Main  - 支持模块独立发布和构建

)  - CMakeLists.txt自动检测构建模式

include(BuildTemplate/Test.cmake.in)  - 提供完整的独立构建方案

```- **2025-10-29**: 简化CMake系统

  - 移除独立的配置文件，合并到`Config.cmake.in`

#### 基准测试配置  - 特性检测完全基于`__cplusplus`宏

  - 与Bitbake完全兼容

```cmake

set(ENABLE_BUILD_BENCHMARK ON)

set(MODULE_BENCHMARK_DIR "test/benchmark")
set(MODULE_EXTERNAL_BENCHMARK_LIB 
    lap_core 
    benchmark::benchmark
)
```

#### CTest集成特性

自动生成CTest测试注册：

```cmake
add_test(NAME core_test COMMAND core_test)
set_tests_properties(core_test PROPERTIES
    TIMEOUT 300
    LABELS "unittest"
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
)
```

运行测试：

```bash
cd build
ctest                     # 运行所有测试
ctest -L unittest         # 只运行单元测试
ctest -L benchmark        # 只运行基准测试
ctest --verbose           # 详细输出
ctest --rerun-failed      # 重跑失败的测试
```

### 6. Daemon.cmake.in - 守护进程构建

**用途：** 构建守护进程并集成systemd服务管理

#### 基本配置

```cmake
set(ENABLE_BUILD_DAEMON ON)
set(MODULE_DAEMON_DIR "daemon")
set(MODULE_EXTERNAL_DAEMON_LIB "lap_core" "lap_log")
```

#### Systemd集成（基础）

```cmake
set(ENABLE_DAEMON_WITH_SYSTEMD ON)
set(MODULE_DAEMONDESCRIPTION "Core Module Daemon")
set(MODULE_DAEMONAFTER "network.target")
set(MODULE_DAEMONREQUIRE "")
set(MODULE_DAEMONBEFORE "")
```

#### Systemd高级配置

```cmake
# 服务行为
set(MODULE_DAEMON_SERVICE_TYPE "notify")        # simple/forking/notify/dbus
set(MODULE_DAEMON_RESTART "on-failure")         # no/on-success/on-failure/always
set(MODULE_DAEMON_WATCHDOG_SEC "30")            # 看门狗超时

# 资源限制
set(MODULE_DAEMON_MEMORY_LIMIT "1G")            # 内存限制
set(MODULE_DAEMON_CPU_QUOTA "50%")              # CPU配额
set(MODULE_DAEMON_MAX_FILES "65536")            # 最大文件数

# 安全加固
set(MODULE_DAEMON_USER "daemon")                # 运行用户
set(MODULE_DAEMON_GROUP "daemon")               # 运行组
set(MODULE_DAEMON_ENABLE_SECURITY ON)           # 启用安全选项
set(MODULE_DAEMON_WORKING_DIR "/var/lib/app")   # 工作目录

# 自动安装（重要！）
set(MODULE_DAEMON_AUTO_REGISTER ON)             # 安装时自动注册systemd
set(MODULE_DAEMON_AUTO_ENABLE ON)               # 自动启用服务
set(MODULE_DAEMON_AUTO_START OFF)               # 自动启动（生产环境建议OFF）
```

#### 生成文件

- `<module>d` - 守护进程可执行文件
- `<module>d.service` - Systemd服务单元文件
- `98-<module>d.preset` - Systemd预设配置
- `install-<module>d.sh` - 服务安装脚本（自动注册到systemd）
- `uninstall-<module>d.sh` - 服务卸载脚本

#### 自动安装到Systemd

**方式1: 安装时自动注册（推荐）**

```cmake
# CMakeLists.txt中配置
set(MODULE_DAEMON_AUTO_REGISTER ON)
set(MODULE_DAEMON_AUTO_ENABLE ON)
```

```bash
# 安装时自动完成注册
sudo cmake --install build
# 服务已自动注册到systemd，并已启用
```

**方式2: 手动注册**

```bash
# 安装后手动执行注册脚本
sudo /usr/lib/systemd/scripts/install-cored.sh

# 或使用标准systemctl命令
sudo systemctl daemon-reload
sudo systemctl enable cored.service
sudo systemctl start cored.service
```

#### Systemd命令

```bash
# 启动/停止/重启
sudo systemctl start cored.service
sudo systemctl stop cored.service
sudo systemctl restart cored.service

# 启用/禁用开机自启
sudo systemctl enable cored.service
sudo systemctl disable cored.service

# 查看状态和日志
systemctl status cored.service
journalctl -u cored.service -f          # 实时日志
journalctl -u cored.service -n 100      # 最近100行
journalctl -u cored.service --since today  # 今天的日志
```

#### 完整示例 - 生产环境守护进程

```cmake
set(ENABLE_BUILD_DAEMON ON)
set(ENABLE_DAEMON_WITH_SYSTEMD ON)
set(MODULE_DAEMON_DIR ${CMAKE_CURRENT_SOURCE_DIR}/daemon)
set(MODULE_EXTERNAL_DAEMON_LIB lap_core lap_log)

# 基本信息
set(MODULE_DAEMONDESCRIPTION "Core Module Production Daemon")
set(MODULE_DAEMONAFTER "network-online.target postgresql.service")

# 安全配置
set(MODULE_DAEMON_USER "appuser")
set(MODULE_DAEMON_GROUP "appgroup")
set(MODULE_DAEMON_WORKING_DIR "/var/lib/myapp")
set(MODULE_DAEMON_ENABLE_SECURITY ON)

# 资源限制
set(MODULE_DAEMON_MEMORY_LIMIT "512M")
set(MODULE_DAEMON_CPU_QUOTA "25%")
set(MODULE_DAEMON_MAX_FILES "4096")

# 重启策略
set(MODULE_DAEMON_RESTART "always")
set(MODULE_DAEMON_START_LIMIT_INTERVAL "10min")
set(MODULE_DAEMON_START_LIMIT_BURST "3")

# 自动安装配置（生产环境：注册并启用，但不自动启动）
set(MODULE_DAEMON_AUTO_REGISTER ON)
set(MODULE_DAEMON_AUTO_ENABLE ON)
set(MODULE_DAEMON_AUTO_START OFF)

include(BuildTemplate/Daemon.cmake.in)
```

**详细文档：** 参见 [systemd/README.md](systemd/README.md) 获取完整systemd配置指南

### 7. Protobuf.cmake.in - Protobuf支持

**用途：** 从`.proto`文件生成C++代码并构建静态库

#### 配置

```cmake
set(ENABLE_BUILD_PROTOBUF ON)
set(MODULE_PROTO_DIR "proto")
set(MODULE_VERSION "1.0.0")
include(BuildTemplate/Protobuf.cmake.in)
```

#### 可选变量

```cmake
set(MODULE_PROTOBUF_TARGET "custom_proto")      # 自定义库名
set(PROTOBUF_IMPORT_DIRS "/path/to/imports")    # Proto导入路径
set(ENABLE_BUILD_WITH_PLATFORM_PREX ON)         # 平台特定命名
set(PLATFORM_SYSTEM_TARGET "x86_64")
```

#### 工作流程

1. 查找 Protobuf 包
2. 收集所有 `.proto` 文件
3. 使用 `protoc` 生成 `.pb.h` 和 `.pb.cc`
4. 编译为静态库（带 `-fPIC`）
5. 安装头文件和库

#### 生成结构

```
build/
├── message.pb.h
├── message.pb.cc
└── ...

install/include/ModuleName/protobuf/
└── message.pb.h

install/lib/
└── libmodulename_proto.a
```

## 📦 安装组件

BuildTemplate使用组件化安装策略：

### Runtime组件

用户运行应用所需的最小文件：

```bash
cmake --install . --component Runtime
```

包含：
- 共享库（`.so*`）
- 可执行文件
- 守护进程
- Systemd服务文件

### Development组件

开发者链接和编译所需文件：

```bash
cmake --install . --component Development
```

包含：
- 头文件（`.h`, `.hpp`）
- 静态库（`.a`）
- CMake配置文件（`*Targets.cmake`）
- Protobuf生成的头文件

### 完整安装

```bash
cmake --install .  # 安装所有组件
```

## 🔍 导出目标与Find_package支持

所有库模板自动生成CMake配置文件，支持其他项目通过`find_package`使用：

### 使用示例

```cmake
# 在另一个项目中
find_package(Core REQUIRED)
target_link_libraries(my_app PRIVATE Core::lap_core)
```

### 导出文件结构

```
install/lib/cmake/Core/
├── CoreTargets.cmake
├── CoreTargets-release.cmake
└── CoreProtobufTargets.cmake  # 如果启用Protobuf
```

## 💡 最佳实践

### 1. 模块目录结构

推荐的模块组织方式：

```
MyModule/
├── CMakeLists.txt
├── BuildTemplate/              # Submodule
├── source/
│   ├── inc/                   # 公共头文件
│   │   └── MyModule/
│   │       └── API.hpp
│   └── src/                   # 源文件和私有头文件
│       ├── impl.cpp
│       └── internal.hpp
├── test/
│   ├── unittest/
│   │   └── test_api.cpp
│   └── benchmark/
│       └── bench_perf.cpp
├── daemon/                     # 可选
│   └── main.cpp
├── proto/                      # 可选
│   └── messages.proto
└── examples/                   # 可选
    └── example.cpp
```

### 2. CMakeLists.txt组织

```cmake
cmake_minimum_required(VERSION 3.10.2)

# ============ 项目配置 ============
include(BuildTemplate/Config.cmake.in)
project(MyModule VERSION 1.0.0 LANGUAGES CXX)

# ============ 模块变量 ============
set(MODULE_NAME "MyModule")
set(MODULE_VERNO ${PROJECT_VERSION})
set(MODULE_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR})

# ============ 查找依赖 ============
find_package(Threads REQUIRED)

# ============ 库构建 ============
set(MODULE_SOURCE_CXX_DIR ${MODULE_ROOT_DIR}/source/src)
set(MODULE_INCLUDE_DIR ${MODULE_ROOT_DIR}/source/inc)
set(MODULE_EXTERNAL_LIB Threads::Threads)
set(ENABLE_BUILD_SHARED_LIBRARY ON)
include(BuildTemplate/SharedLibrary.cmake.in)

# ============ 可执行文件 ============
set(MODULE_EXECUTABLE_DIR ${MODULE_ROOT_DIR}/examples)
set(MODULE_EXTERNAL_EXECUTABLE_LIB mymodule)
set(ENABLE_BUILD_EXECUTABLE ON)
include(BuildTemplate/Executable.cmake.in)

# ============ 测试 ============
find_package(GTest)
if(GTest_FOUND)
    enable_testing()  # CTest支持
    set(ENABLE_BUILD_UNITTEST ON)
    set(MODULE_TEST_DIR ${MODULE_ROOT_DIR}/test/unittest)
    set(MODULE_EXTERNAL_TEST_LIB mymodule GTest::GTest GTest::Main)
    include(BuildTemplate/Test.cmake.in)
endif()

# ============ 守护进程（可选） ============
if(BUILD_DAEMON)
    set(ENABLE_BUILD_DAEMON ON)
    set(ENABLE_DAEMON_WITH_SYSTEMD ON)
    set(MODULE_DAEMON_DIR ${MODULE_ROOT_DIR}/daemon)
    set(MODULE_EXTERNAL_DAEMON_LIB mymodule)
    include(BuildTemplate/Daemon.cmake.in)
endif()
```

### 3. 头文件中的特性检测

不依赖CMake宏定义，使用标准C++宏：

```cpp
// Platform.hpp
#pragma once

// C++17特性检测
#if __cplusplus >= 201703L
    #define HAS_CPP17 1
    #include <optional>
    #include <variant>
    #include <filesystem>
    namespace fs = std::filesystem;
    template<typename T>
    using Optional = std::optional<T>;
#else
    #define HAS_CPP17 0
    #include <boost/optional.hpp>
    #include <boost/variant.hpp>
    #include <boost/filesystem.hpp>
    namespace fs = boost::filesystem;
    template<typename T>
    using Optional = boost::optional<T>;
#endif

// C++20特性检测
#if __cplusplus >= 202002L
    #include <span>
    #include <ranges>
#endif
```

### 4. 变量命名约定

```cmake
# 模块全局变量 - 大写带MODULE_前缀
MODULE_NAME                    # 模块名称
MODULE_VERSION/MODULE_VERNO    # 版本号
MODULE_ROOT_DIR                # 模块根目录

# 构建类型特定 - 功能描述
MODULE_SOURCE_CXX_DIR          # C++源文件目录
MODULE_INCLUDE_DIR             # 头文件目录
MODULE_TEST_DIR                # 测试目录
MODULE_DAEMON_DIR              # 守护进程目录

# 依赖配置 - 用途明确
MODULE_EXTERNAL_LIB            # 库依赖
MODULE_EXTERNAL_TEST_LIB       # 测试依赖
MODULE_EXTERNAL_DAEMON_LIB     # 守护进程依赖

# 控制开关 - ENABLE_前缀
ENABLE_BUILD_SHARED_LIBRARY    # 构建共享库
ENABLE_BUILD_UNITTEST          # 构建单元测试
ENABLE_DAEMON_WITH_SYSTEMD     # 启用Systemd
```

## 🐛 故障排查

### 常见问题

#### 1. "No .proto files found"

**问题：** Protobuf.cmake.in 找不到proto文件

**解决：**
```cmake
# 确认目录正确
set(MODULE_PROTO_DIR "${CMAKE_CURRENT_SOURCE_DIR}/proto")
lap_validate_directory("${MODULE_PROTO_DIR}" "MODULE_PROTO_DIR")
```

#### 2. "Undefined reference" 链接错误

**问题：** 目标找不到符号

**解决：**
```cmake
# 检查链接顺序（依赖应在后面）
set(MODULE_EXTERNAL_LIB 
    my_module        # 你的库在前
    Threads::Threads # 系统库在后
)

# 对于静态库，使用PUBLIC确保传递依赖
target_link_libraries(my_static_lib PUBLIC ${DEPS})
```

#### 3. CTest找不到测试

**问题：** `ctest`不显示测试

**解决：**
```cmake
# 确保在顶层CMakeLists.txt调用
enable_testing()
include(CTest)

# 然后包含测试模板
include(BuildTemplate/Test.cmake.in)
```

#### 4. Systemd服务无法启动

**问题：** 守护进程服务失败

**检查步骤：**
```bash
# 查看服务状态
systemctl status mymoduled.service

# 查看日志
journalctl -u mymoduled.service -xe

# 验证可执行文件路径
cat /lib/systemd/system/mymoduled.service
which mymoduled
```

#### 5. 头文件找不到

**问题：** 编译时报 "No such file or directory"

**解决：**
```cmake
# 使用 lap_collect_sources 确保包含所有头文件目录
set(LOCAL_LIB_INCLUDE_DIRS "")
foreach(_headerFile ${LOCAL_LIB_HEADERS})
    get_filename_component(_dir ${_headerFile} DIRECTORY)
    list(APPEND LOCAL_LIB_INCLUDE_DIRS ${_dir})
endforeach()
list(REMOVE_DUPLICATES LOCAL_LIB_INCLUDE_DIRS)

# 或显式添加
set(MODULE_INCLUDE_DIR "source/inc")
```

### 调试技巧

#### 启用详细输出

```bash
cmake -B build --debug-output     # CMake调试
cmake --build build --verbose     # 编译命令
ctest --verbose                   # 测试详情
```

#### 检查生成的配置

```cmake
# 在CMakeLists.txt中添加
lap_print_config()  # 打印所有LightAP配置

message(STATUS "Module name: ${MODULE_NAME}")
message(STATUS "Sources: ${LOCAL_LIB_SOURCES}")
```

#### 验证目标属性

```bash
# 在build目录
cmake --build . --target help     # 查看所有目标
ctest -N                          # 列出所有测试
```

## 🔗 与Bitbake/Yocto集成

### Recipe示例

```bash
DESCRIPTION = "LightAP Core Module"
LICENSE = "MIT"

SRC_URI = "git://github.com/myorg/lightap-core.git;protocol=https;branch=main"

DEPENDS = "boost protobuf gtest"

inherit cmake

# C++标准控制
EXTRA_OECMAKE += "-DCMAKE_CXX_STANDARD=17"

# 可选特性
EXTRA_OECMAKE += "-DBUILD_DAEMON=ON"
EXTRA_OECMAKE += "-DENABLE_BUILD_UNITTEST=${@bb.utils.contains('PTEST_ENABLED', '1', 'ON', 'OFF', d)}"

# 组件化打包
PACKAGES =+ "${PN}-dev ${PN}-daemon"

FILES:${PN} = "${libdir}/lib*.so.*"
FILES:${PN}-dev = "${includedir} ${libdir}/lib*.so ${libdir}/cmake"
FILES:${PN}-daemon = "${bindir}/*d ${systemd_system_unitdir}"

SYSTEMD_SERVICE:${PN}-daemon = "cored.service"

do_install:append() {
    # 只安装Runtime组件到主包
    cd ${B}
    cmake --install . --component Runtime --prefix ${D}${prefix}
    
    # Development组件到-dev包
    cmake --install . --component Development --prefix ${D}${prefix}
}
```

### 关键点

1. **不依赖CMake定义特性** - 所有`__cplusplus`检测在头文件
2. **标准CMake变量** - 只使用`CMAKE_CXX_STANDARD`等标准变量
3. **组件化安装** - Runtime/Development分离便于打包

## 📚 API参考

### Config.cmake.in 函数

| 函数 | 参数 | 描述 |
|------|------|------|
| `lap_require_variable` | `var_name` `error_msg` | 检查变量是否定义 |
| `lap_validate_directory` | `dir_path` `var_name` | 验证目录存在 |
| `lap_print_config` | 无 | 打印LightAP配置摘要 |
| `lap_configure_cxx_target` | `TARGET target_name` | 配置C++目标 |
| `lap_configure_cxx_library` | `TARGET target_name` | 配置库目标 |
| `lap_collect_sources` | `OUTPUT_VAR var` `DIRECTORIES dirs...` | 收集源文件 |

### 全局变量

#### 必需（所有模板）

| 变量 | 类型 | 示例 | 描述 |
|------|------|------|------|
| `MODULE_NAME` | String | `"Core"` | 模块名称 |
| `MODULE_VERNO`/`MODULE_VERSION` | String | `"1.0.0"` | 版本号（X.Y.Z） |

#### 共享/静态库

| 变量 | 必需 | 描述 |
|------|------|------|
| `MODULE_SOURCE_CXX_DIR` | ✅ | C++源文件目录 |
| `MODULE_INCLUDE_DIR` | ❌ | 头文件目录 |
| `MODULE_EXTERNAL_LIB` | ❌ | 链接库列表 |
| `MODULE_EXTERNAL_INCLUDE_DIR` | ❌ | 外部头文件目录 |
| `BUILD_WITH_STRIP` | ❌ | Release模式strip |

#### 可执行文件

| 变量 | 必需 | 描述 |
|------|------|------|
| `MODULE_EXECUTABLE_DIR` | ✅ | 可执行文件源码目录 |
| `MODULE_EXTERNAL_EXECUTABLE_LIB` | ✅ | 链接库 |
| `MODULE_EXECUTABLE_TARGET` | ❌ | 自定义目标名 |
| `MODULE_INSTALL_CONFIG_DIR` | ❌ | 配置文件目录 |

#### 测试

| 变量 | 必需 | 描述 |
|------|------|------|
| `MODULE_TEST_DIR` | ✅ | 单元测试目录 |
| `MODULE_EXTERNAL_TEST_LIB` | ✅ | 测试库（含GTest） |
| `MODULE_BENCHMARK_DIR` | ❌ | 基准测试目录 |
| `MODULE_EXTERNAL_BENCHMARK_LIB` | ❌ | 基准测试库 |

#### 守护进程

| 变量 | 必需 | 描述 |
|------|------|------|
| `MODULE_DAEMON_DIR` | ✅ | 守护进程源码目录 |
| `MODULE_EXTERNAL_DAEMON_LIB` | ✅ | 守护进程依赖库 |
| `ENABLE_DAEMON_WITH_SYSTEMD` | ❌ | 启用Systemd |
| `MODULE_DAEMONDESCRIPTION` | ❌ | 服务描述 |
| `MODULE_DAEMONAFTER` | ❌ | After= 依赖 |

#### Protobuf

| 变量 | 必需 | 描述 |
|------|------|------|
| `MODULE_PROTO_DIR` | ✅ | .proto文件目录 |
| `MODULE_PROTOBUF_TARGET` | ❌ | 自定义库名 |
| `PROTOBUF_IMPORT_DIRS` | ❌ | Proto导入路径 |

## 📖 示例项目

完整示例见 `modules/Core/` 目录：

```bash
# 查看Core模块的完整实现
cat modules/Core/CMakeLists.txt
cat modules/Core/build.sh
cat modules/Core/.gitmodules
```

## 🔗 独立模块使用（Submodule模式）

BuildTemplate支持作为Git submodule被独立模块引用，实现模块的独立发布和构建。

详细文档请参考：[STANDALONE.md](STANDALONE.md)

### 快速示例

```bash
# 添加BuildTemplate作为submodule
cd modules/Core
git submodule add ../../BuildTemplate BuildTemplate
git submodule update --init --recursive

# 构建
./build.sh
```

## 📄 许可证

MIT License - 详见项目根目录 LICENSE 文件

## 🤝 贡献

欢迎提交Issue和Pull Request！

开发新特性时请：
1. 遵循现有代码风格
2. 更新相关文档
3. 添加测试用例
4. 更新本README

## 📮 联系方式

- **项目主页:** [GitHub - LightAP](https://github.com/yourorg/LightAP)
- **Issue追踪:** [GitHub Issues](https://github.com/yourorg/LightAP/issues)

## 📅 更新历史

### v1.1.0 (2024-01)

**重大更新：全面现代化CMake构建系统**

- ✨ **Config.cmake.in v1.1.0**
  - 新增验证函数：`lap_require_variable`, `lap_validate_directory`, `lap_print_config`
  - 新增辅助函数：`lap_collect_sources`
  - 增强`lap_configure_cxx_target`：编译器特定警告
  - 自动编译器检测（GCC/Clang/MSVC）
  - 平台特定配置支持
  - 防止重复包含保护

- 🏗️ **SharedLibrary.cmake.in**
  - 应用现代CMake `target_*` commands
  - 扩展源文件扩展名支持（.cpp/.cxx/.cc/.c++/.c）
  - 扩展头文件扩展名支持（.h/.hpp/.hxx）
  - 导出CMake目标用于`find_package`
  - 组件化安装（Runtime, Development）
  - 完善的变量验证

- 📦 **StaticLibrary.cmake.in**
  - 与SharedLibrary一致的现代化改造
  - PUBLIC链接传递依赖给使用者
  - ARCHIVE组件安装
  - 导出目标支持

- ⚡ **Executable.cmake.in**
  - 修复原有bug（target_link_directories语法）
  - 简化安装规则（移除不必要的头文件安装）
  - 可选配置文件目录安装
  - 改进源文件收集

- 🧪 **Test.cmake.in**
  - **CTest完全集成**
  - 自动测试注册（`add_test`）
  - 测试属性配置（超时、标签）
  - 单元测试支持（300s超时，"unittest"标签）
  - 基准测试支持（600s超时，"benchmark"/"performance"标签）
  - 改进源文件收集

- 🔧 **Daemon.cmake.in**
  - 应用现代CMake实践
  - 增强Systemd集成
  - 改进验证和错误处理
  - 一致的代码风格

- 🔌 **Protobuf.cmake.in**
  - 现代化protobuf集成
  - 增强错误处理
  - 支持proto导入路径
  - 导出目标支持
  - 组件化安装

- 📖 **文档更新**
  - 全面的README.md（本文档）
  - 详细的API参考
  - 最佳实践指南
  - 故障排查指南
  - Bitbake/Yocto集成示例

### v1.0.0 (2023-10)

- 初始版本发布
- C++14/17自动检测
- 基础构建模板
- Submodule模式支持

---

**最后更新:** 2024-01  
**版本:** 1.1.0  
**维护者:** LightAP Team
