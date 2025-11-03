# LightAP CMake Build System

**Version:** 1.1.0  
**License:** MIT  
**Language:** CMake 3.10.2+

## 概述

LightAP BuildTemplate 是一套现代化的CMake构建模板系统，为LightAP项目提供标准化、模块化的构建配置。

### 核心特性

- ✅ 现代CMake 3.x实践 - target_* commands, 导出目标, 组件化安装
- ✅ 多C++标准支持 - C++14/17/20/23自动检测与回退
- ✅ 模块化设计 - 可作为Git submodule独立使用
- ✅ 完整的构建类型 - 库、可执行文件、测试、守护进程、Protobuf
- ✅ CTest集成 - 自动化测试框架支持
- ✅ Systemd集成 - 守护进程服务管理与自动安装
- ✅ Find_package支持 - 导出CMake配置供其他项目使用

## 系统架构

```shell
BuildTemplate/
├── Config.cmake.in              # 核心配置与辅助函数 (v1.1.0)
├── SharedLibrary.cmake.in       # 共享库构建模板 (.so)
├── StaticLibrary.cmake.in       # 静态库构建模板 (.a)
├── Executable.cmake.in          # 可执行文件构建模板
├── Test.cmake.in                # 单元测试与基准测试模板 (CTest)
├── Daemon.cmake.in              # 守护进程构建模板 (Systemd)
├── Protobuf.cmake.in            # Protobuf代码生成与库构建
├── systemd/                     # Systemd服务配置模板
│   ├── service.cmake.in
│   ├── preset.cmake.in
│   ├── install-service.sh.in
│   ├── uninstall-service.sh.in
│   └── README.md
├── README.md                    # 本文档
├── STANDALONE.md                # 独立模块使用指南
├── DAEMON_EXAMPLE.md            # 守护进程完整示例
└── CHANGELOG.md                 # 版本变更记录
```

## 快速开始

### 基本用法

```cmake
cmake_minimum_required(VERSION 3.10.2)

# 1. 包含核心配置
include(BuildTemplate/Config.cmake.in)

# 2. 定义项目
project(MyModule VERSION 1.0.0)

# 3. 配置模块变量
set(MODULE_NAME "MyModule")
set(MODULE_VERNO ${PROJECT_VERSION})
set(MODULE_SOURCE_CXX_DIR ${CMAKE_CURRENT_SOURCE_DIR}/source/src)
set(MODULE_EXTERNAL_LIB Threads::Threads)

# 4. 包含构建模板
set(ENABLE_BUILD_SHARED_LIBRARY ON)
include(BuildTemplate/SharedLibrary.cmake.in)
```

### 独立模块模式（Submodule）

```bash
# 添加BuildTemplate为submodule
git submodule add <BuildTemplate-repo-url> BuildTemplate
git submodule update --init --recursive

# 构建
./build.sh          # Release构建
./build.sh debug    # Debug构建
```

详见 [STANDALONE.md](STANDALONE.md) 获取完整指南。

## 核心模板说明

### 1. Config.cmake.in - 核心配置系统 (v1.1.0)

提供全局配置、验证函数和辅助函数。

**主要功能：**
- C++标准自动检测（C++23/20/17/14）
- 编译器特定优化（GCC/Clang/MSVC）
- 目录和变量验证函数
- 源文件自动收集
- 配置摘要打印

**关键函数：**
- `lap_require_variable(var_name error_msg)` - 检查必需变量
- `lap_validate_directory(dir_path var_name)` - 验证目录存在
- `lap_print_config()` - 打印配置摘要
- `lap_configure_cxx_target(TARGET name)` - 配置C++目标
- `lap_configure_cxx_library(TARGET name)` - 配置库目标
- `lap_collect_sources(OUTPUT_VAR var DIRECTORIES ...)` - 收集源文件

### 2. SharedLibrary.cmake.in - 共享库构建

构建 `.so` 动态链接库，支持版本管理和导出配置。

**必需变量：**
```cmake
MODULE_NAME                    # 模块名
MODULE_VERNO                   # 版本号（X.Y.Z）
MODULE_SOURCE_CXX_DIR          # C++源码目录
ENABLE_BUILD_SHARED_LIBRARY    # 启用标志
```

**可选变量：**
```cmake
MODULE_INCLUDE_DIR             # 头文件目录
MODULE_EXTERNAL_LIB            # 外部依赖库
BUILD_WITH_STRIP               # Release模式strip符号
```

**生成文件：**
- `lib<module>.so.X.Y.Z` - 实际库文件
- `lib<module>.so.X` - 主版本符号链接
- `lib<module>.so` - 通用符号链接
- `<Module>Targets.cmake` - CMake导出配置

### 3. StaticLibrary.cmake.in - 静态库构建

构建 `.a` 静态链接库，用法与SharedLibrary相同。

**关键差异：**
- 外部库使用PUBLIC链接（传递给使用者）
- 安装为ARCHIVE组件

### 4. Executable.cmake.in - 可执行文件构建

构建可执行二进制程序。

**必需变量：**
```cmake
MODULE_NAME
MODULE_EXECUTABLE_DIR          # 可执行文件源码目录
MODULE_EXTERNAL_EXECUTABLE_LIB # 链接库
ENABLE_BUILD_EXECUTABLE        # 启用标志
```

**可选变量：**
```cmake
MODULE_EXECUTABLE_TARGET       # 自定义目标名
MODULE_INSTALL_CONFIG_DIR      # 配置文件目录
BUILD_WITH_STRIP               # Strip符号表
```

### 5. Test.cmake.in - 测试框架 (CTest集成)

构建单元测试和基准测试，完整集成CTest。

**单元测试配置：**
```cmake
ENABLE_BUILD_UNITTEST ON
MODULE_TEST_DIR "test/unittest"
MODULE_EXTERNAL_TEST_LIB "lap_core" "GTest::GTest" "GTest::Main"
```

**基准测试配置：**
```cmake
ENABLE_BUILD_BENCHMARK ON
MODULE_BENCHMARK_DIR "test/benchmark"
MODULE_EXTERNAL_BENCHMARK_LIB "lap_core" "benchmark::benchmark"
```

**运行测试：**
```bash
ctest                     # 运行所有测试
ctest -L unittest         # 只运行单元测试
ctest -L benchmark        # 只运行基准测试
ctest --verbose           # 详细输出
```

### 6. Daemon.cmake.in - 守护进程构建 (Systemd集成)

构建守护进程并提供完整的systemd服务集成，支持自动注册、启用和启动。

**基本配置：**
```cmake
ENABLE_BUILD_DAEMON ON
MODULE_DAEMON_DIR "daemon"
MODULE_EXTERNAL_DAEMON_LIB "lap_core" "lap_log"
```

**Systemd自动安装（推荐）：**
```cmake
ENABLE_DAEMON_WITH_SYSTEMD ON
MODULE_DAEMON_AUTO_REGISTER ON      # 安装时自动注册到systemd
MODULE_DAEMON_AUTO_ENABLE ON        # 自动启用服务
MODULE_DAEMON_AUTO_START OFF        # 不自动启动（生产环境推荐）
```

**高级配置（可选）：**
```cmake
# 服务行为
MODULE_DAEMON_SERVICE_TYPE "notify"          # simple/forking/notify
MODULE_DAEMON_RESTART "on-failure"           # 重启策略
MODULE_DAEMON_WATCHDOG_SEC "30"              # 看门狗超时

# 资源限制
MODULE_DAEMON_MEMORY_LIMIT "1G"              # 内存限制
MODULE_DAEMON_CPU_QUOTA "50%"                # CPU配额
MODULE_DAEMON_MAX_FILES "65536"              # 最大文件数

# 安全加固
MODULE_DAEMON_USER "daemon"                  # 运行用户
MODULE_DAEMON_GROUP "daemon"                 # 运行组
MODULE_DAEMON_ENABLE_SECURITY ON             # 启用安全选项
```

**生成的文件：**
- `<module>d` - 守护进程可执行文件
- `<module>d.service` - Systemd服务单元文件
- `98-<module>d.preset` - Systemd预设配置
- `install-<module>d.sh` - 自动安装脚本
- `uninstall-<module>d.sh` - 自动卸载脚本

**systemd命令：**
```bash
# 服务管理
sudo systemctl start cored.service
sudo systemctl stop cored.service
sudo systemctl restart cored.service
sudo systemctl enable cored.service
sudo systemctl disable cored.service

# 查看状态和日志
systemctl status cored.service
journalctl -u cored.service -f              # 实时日志
journalctl -u cored.service --since today   # 今天的日志
```

详见：[systemd/README.md](systemd/README.md) | [DAEMON_EXAMPLE.md](DAEMON_EXAMPLE.md)

### 7. Protobuf.cmake.in - Protobuf支持

从`.proto`文件生成C++代码并构建静态库。

**基本配置：**
```cmake
ENABLE_BUILD_PROTOBUF ON
MODULE_PROTO_DIR "proto"
MODULE_VERSION "1.0.0"
```

**可选配置：**
```cmake
MODULE_PROTOBUF_TARGET "custom_proto"       # 自定义库名
PROTOBUF_IMPORT_DIRS "/path/to/imports"     # Proto导入路径
```

## 完整的CMakeLists.txt示例

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

# ============ 测试 ============
find_package(GTest)
if(GTest_FOUND)
    enable_testing()
    set(ENABLE_BUILD_UNITTEST ON)
    set(MODULE_TEST_DIR ${MODULE_ROOT_DIR}/test/unittest)
    set(MODULE_EXTERNAL_TEST_LIB mymodule GTest::GTest GTest::Main)
    include(BuildTemplate/Test.cmake.in)
endif()

# ============ 守护进程 ============
if(BUILD_DAEMON)
    set(ENABLE_BUILD_DAEMON ON)
    set(ENABLE_DAEMON_WITH_SYSTEMD ON)
    set(MODULE_DAEMON_AUTO_REGISTER ON)
    set(MODULE_DAEMON_AUTO_ENABLE ON)
    set(MODULE_DAEMON_DIR ${MODULE_ROOT_DIR}/daemon)
    set(MODULE_EXTERNAL_DAEMON_LIB mymodule)
    include(BuildTemplate/Daemon.cmake.in)
endif()
```

## 变量命名约定

```cmake
# 模块全局变量 - 大写带MODULE_前缀
MODULE_NAME                    # 模块名称
MODULE_VERNO                   # 版本号
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

## 安装组件

### Runtime组件（用户运行时）

```bash
cmake --install . --component Runtime
```

包含：
- 共享库（`.so*`）
- 可执行文件
- 守护进程
- Systemd服务文件

### Development组件（开发者）

```bash
cmake --install . --component Development
```

包含：
- 头文件（`.h`, `.hpp`）
- 静态库（`.a`）
- CMake配置文件（`*Targets.cmake`）

### 完整安装

```bash
cmake --install .  # 安装所有组件
```

## Find_package支持

所有库模板自动生成CMake配置文件：

```cmake
# 在其他项目中使用
find_package(Core REQUIRED)
target_link_libraries(my_app PRIVATE Core::lap_core)
```

## 常见问题

### 1. Protobuf找不到文件

```cmake
# 确认目录正确
set(MODULE_PROTO_DIR "${CMAKE_CURRENT_SOURCE_DIR}/proto")
lap_validate_directory("${MODULE_PROTO_DIR}" "MODULE_PROTO_DIR")
```

### 2. 链接错误

```cmake
# 检查链接顺序（依赖应在后面）
set(MODULE_EXTERNAL_LIB 
    my_module        # 你的库在前
    Threads::Threads # 系统库在后
)
```

### 3. CTest找不到测试

```cmake
# 确保调用enable_testing()
enable_testing()
include(BuildTemplate/Test.cmake.in)
```

### 4. Systemd服务无法启动

```bash
# 查看服务状态和日志
systemctl status mymoduled.service
journalctl -u mymoduled.service -xe
```

## 调试技巧

```bash
# CMake调试
cmake -B build --debug-output

# 编译详细输出
cmake --build build --verbose

# 测试详情
ctest --verbose

# 查看所有目标
cmake --build . --target help

# 列出所有测试
ctest -N
```

## 与Bitbake/Yocto集成

```bash
DESCRIPTION = "LightAP Core Module"
LICENSE = "MIT"
inherit cmake

# C++标准控制
EXTRA_OECMAKE += "-DCMAKE_CXX_STANDARD=17"

# 组件化打包
PACKAGES =+ "${PN}-dev ${PN}-daemon"

FILES:${PN} = "${libdir}/lib*.so.*"
FILES:${PN}-dev = "${includedir} ${libdir}/lib*.so ${libdir}/cmake"
FILES:${PN}-daemon = "${bindir}/*d ${systemd_system_unitdir}"

SYSTEMD_SERVICE:${PN}-daemon = "cored.service"

do_install:append() {
    cmake --install . --component Runtime --prefix ${D}${prefix}
    cmake --install . --component Development --prefix ${D}${prefix}
}
```

## 相关文档

- **[STANDALONE.md](STANDALONE.md)** - 独立模块使用指南（Submodule模式）
- **[DAEMON_EXAMPLE.md](DAEMON_EXAMPLE.md)** - 守护进程完整示例
- **[systemd/README.md](systemd/README.md)** - Systemd配置详细说明
- **[CHANGELOG.md](CHANGELOG.md)** - 版本变更记录

## 更新历史

### v1.1.0 (2024-01)

**重大更新：全面现代化CMake构建系统**

- ✨ Config.cmake.in v1.1.0 - 新增验证函数和辅助函数
- 🏗️ SharedLibrary/StaticLibrary - 应用现代CMake实践，组件化安装
- ⚡ Executable.cmake.in - 修复bug，改进安装规则
- 🧪 Test.cmake.in - CTest完全集成
- 🔧 Daemon.cmake.in - **重大增强：自动Systemd服务注册**
- 🔌 Protobuf.cmake.in - 现代化protobuf集成
- 📖 完整文档更新

### v1.0.0 (2023-10)

- 初始版本发布
- C++14/17自动检测
- 基础构建模板
- Submodule模式支持

---

**Version:** 1.1.0  
**Last Updated:** 2024-01  
**Maintainer:** LightAP Team

