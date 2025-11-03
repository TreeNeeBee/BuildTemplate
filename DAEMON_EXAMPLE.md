# Daemon with Systemd Auto-Installation Example

## 项目结构

```
MyDaemon/
├── CMakeLists.txt
├── BuildTemplate/          # Git submodule
├── daemon/
│   ├── main.cpp
│   └── daemon.hpp
└── README.md
```

## CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.10.2)

# 包含BuildTemplate
include(BuildTemplate/Config.cmake.in)

# 定义项目
project(MyDaemon VERSION 1.0.0 LANGUAGES CXX)

# 模块配置
set(MODULE_NAME "MyDaemon")
set(MODULE_VERNO ${PROJECT_VERSION})
set(MODULE_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR})

# 查找依赖
find_package(Threads REQUIRED)

# ============================================================================
# 守护进程配置 - 带自动systemd安装
# ============================================================================

set(ENABLE_BUILD_DAEMON ON)
set(ENABLE_DAEMON_WITH_SYSTEMD ON)
set(MODULE_DAEMON_DIR ${MODULE_ROOT_DIR}/daemon)
set(MODULE_EXTERNAL_DAEMON_LIB Threads::Threads)

# 基本信息
set(MODULE_DAEMONDESCRIPTION "My Application Daemon Service")
set(MODULE_DAEMONAFTER "network-online.target")

# 资源限制
set(MODULE_DAEMON_MEMORY_LIMIT "512M")
set(MODULE_DAEMON_CPU_QUOTA "50%")
set(MODULE_DAEMON_MAX_FILES "8192")

# 安全配置（生产环境）
set(MODULE_DAEMON_USER "daemon")
set(MODULE_DAEMON_GROUP "daemon")
set(MODULE_DAEMON_ENABLE_SECURITY ON)
set(MODULE_DAEMON_WORKING_DIR "/var/lib/mydaemon")

# 服务行为
set(MODULE_DAEMON_SERVICE_TYPE "notify")
set(MODULE_DAEMON_RESTART "on-failure")
set(MODULE_DAEMON_WATCHDOG_SEC "30")
set(MODULE_DAEMON_START_LIMIT_INTERVAL "10min")
set(MODULE_DAEMON_START_LIMIT_BURST "3")

# ============================================================================
# 关键配置：自动安装到systemd
# ============================================================================

# 方案1: 开发环境 - 完全自动化（自动注册、启用、启动）
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(MODULE_DAEMON_AUTO_REGISTER ON)
    set(MODULE_DAEMON_AUTO_ENABLE ON)
    set(MODULE_DAEMON_AUTO_START ON)   # 开发环境可以自动启动
    message(STATUS "Development mode: Daemon will auto-start on install")
endif()

# 方案2: 生产环境 - 谨慎策略（自动注册和启用，但不自动启动）
if(CMAKE_BUILD_TYPE STREQUAL "Release")
    set(MODULE_DAEMON_AUTO_REGISTER ON)
    set(MODULE_DAEMON_AUTO_ENABLE ON)
    set(MODULE_DAEMON_AUTO_START OFF)  # 生产环境需要手动启动
    message(STATUS "Production mode: Daemon registered but requires manual start")
endif()

# 包含Daemon模板
include(BuildTemplate/Daemon.cmake.in)
```

## daemon/main.cpp

```cpp
#include <iostream>
#include <thread>
#include <csignal>
#include <systemd/sd-daemon.h>

volatile sig_atomic_t g_running = 1;

void signal_handler(int signum) {
    g_running = 0;
}

int main() {
    // 注册信号处理
    std::signal(SIGTERM, signal_handler);
    std::signal(SIGINT, signal_handler);
    
    std::cout << "MyDaemon starting..." << std::endl;
    
    // 通知systemd服务已准备就绪
    sd_notify(0, "READY=1");
    
    // 主循环
    while (g_running) {
        // 执行守护进程工作
        std::this_thread::sleep_for(std::chrono::seconds(1));
        
        // 定期通知systemd看门狗
        sd_notify(0, "WATCHDOG=1");
    }
    
    std::cout << "MyDaemon stopping..." << std::endl;
    
    // 通知systemd正在停止
    sd_notify(0, "STOPPING=1");
    
    return 0;
}
```

## 构建和安装

### 开发环境（自动启动）

```bash
# 配置为Debug模式
cmake -B build -DCMAKE_BUILD_TYPE=Debug

# 编译
cmake --build build

# 安装（自动注册、启用并启动服务）
sudo cmake --install build

# 输出示例：
# =================================================================
# Installing systemd service: mydaemoned.service
# =================================================================
# Service file: /usr/lib/systemd/system/mydaemoned.service
# Install mode: system
# Reloading systemd daemon...
# Applying systemd preset...
# Enabling mydaemoned.service...
# Service enabled successfully.
# Starting mydaemoned.service...
# Service started successfully.
# 
# Service status:
# ● mydaemoned.service - My Application Daemon Service
#    Loaded: loaded (/usr/lib/systemd/system/mydaemoned.service; enabled)
#    Active: active (running) since ...
# =================================================================

# 验证服务运行
systemctl status mydaemoned.service
```

### 生产环境（手动启动）

```bash
# 配置为Release模式
cmake -B build -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=/usr

# 编译
cmake --build build

# 安装（自动注册和启用，但不启动）
sudo cmake --install build

# 输出示例：
# =================================================================
# Installing systemd service: mydaemoned.service
# =================================================================
# ...
# Enabling mydaemoned.service...
# Service enabled successfully.
# Service not started (use: systemctl start mydaemoned.service)
# 
# Service management commands:
#   Start:   systemctl start mydaemoned.service
#   Stop:    systemctl stop mydaemoned.service
#   Status:  systemctl status mydaemoned.service
#   Logs:    journalctl -u mydaemoned.service -f
# =================================================================

# 手动启动（在验证配置后）
sudo systemctl start mydaemoned.service

# 查看状态
systemctl status mydaemoned.service

# 查看日志
journalctl -u mydaemoned.service -f
```

### 不自动注册（传统方式）

```cmake
# CMakeLists.txt中不设置AUTO_REGISTER
# 或显式禁用
set(MODULE_DAEMON_AUTO_REGISTER OFF)
```

```bash
# 安装后手动注册
sudo cmake --install build

# 使用生成的安装脚本
sudo /usr/lib/systemd/scripts/install-mydaemoned.sh

# 或使用标准systemctl命令
sudo systemctl daemon-reload
sudo systemctl enable mydaemoned.service
sudo systemctl start mydaemoned.service
```

## 卸载服务

```bash
# 方法1: 使用生成的卸载脚本
sudo /usr/lib/systemd/scripts/uninstall-mydaemoned.sh

# 方法2: 手动卸载
sudo systemctl stop mydaemoned.service
sudo systemctl disable mydaemoned.service
sudo rm /usr/lib/systemd/system/mydaemoned.service
sudo systemctl daemon-reload
```

## 生成的文件

安装后会生成以下文件：

```
/usr/bin/mydaemoned                                    # 守护进程可执行文件
/usr/lib/systemd/system/mydaemoned.service            # Systemd服务单元
/usr/lib/systemd/system-preset/98-mydaemoned.preset   # 预设配置
/usr/lib/systemd/scripts/install-mydaemoned.sh        # 安装脚本
/usr/lib/systemd/scripts/uninstall-mydaemoned.sh      # 卸载脚本
```

## 服务管理

```bash
# 启动服务
sudo systemctl start mydaemoned.service

# 停止服务
sudo systemctl stop mydaemoned.service

# 重启服务
sudo systemctl restart mydaemoned.service

# 重新加载配置（如果支持SIGHUP）
sudo systemctl reload mydaemoned.service

# 查看服务状态
systemctl status mydaemoned.service

# 查看是否已启用
systemctl is-enabled mydaemoned.service

# 查看是否运行中
systemctl is-active mydaemoned.service

# 启用开机自启
sudo systemctl enable mydaemoned.service

# 禁用开机自启
sudo systemctl disable mydaemoned.service

# 查看服务日志
journalctl -u mydaemoned.service

# 实时跟踪日志
journalctl -u mydaemoned.service -f

# 查看最近100行日志
journalctl -u mydaemoned.service -n 100

# 查看今天的日志
journalctl -u mydaemoned.service --since today

# 查看从某个时间点的日志
journalctl -u mydaemoned.service --since "2024-01-01 10:00:00"
```

## 验证配置

```bash
# 检查service文件语法
systemd-analyze verify /usr/lib/systemd/system/mydaemoned.service

# 查看服务依赖树
systemctl list-dependencies mydaemoned.service

# 查看服务配置
systemctl cat mydaemoned.service

# 查看服务属性
systemctl show mydaemoned.service
```

## 故障排查

```bash
# 查看详细状态
systemctl status mydaemoned.service -l

# 查看最近的错误日志
journalctl -u mydaemoned.service -p err

# 查看完整的启动日志
journalctl -u mydaemoned.service -b

# 重新加载systemd配置
sudo systemctl daemon-reload

# 重置失败状态
sudo systemctl reset-failed mydaemoned.service

# 查看资源使用情况
systemctl status mydaemoned.service --no-pager
```

## 最佳实践

### 1. 开发环境

- ✅ 使用 `AUTO_START=ON` 快速迭代
- ✅ 不设置资源限制或设置较宽松的值
- ✅ 不启用安全加固（方便调试）

### 2. 生产环境

- ✅ 使用 `AUTO_REGISTER=ON` + `AUTO_ENABLE=ON` + `AUTO_START=OFF`
- ✅ 设置合理的资源限制
- ✅ 启用安全加固
- ✅ 使用专用用户运行
- ✅ 配置适当的重启策略
- ✅ 手动验证后再启动服务

### 3. 持续集成/部署

```bash
# CI/CD脚本示例
cmake -B build -DCMAKE_BUILD_TYPE=Release \
      -DMODULE_DAEMON_AUTO_REGISTER=ON \
      -DMODULE_DAEMON_AUTO_ENABLE=ON \
      -DMODULE_DAEMON_AUTO_START=OFF

cmake --build build
sudo cmake --install build

# 运行健康检查
if ! systemctl is-enabled mydaemoned.service; then
    echo "Error: Service not enabled"
    exit 1
fi

# 手动启动并验证
sudo systemctl start mydaemoned.service
sleep 5

if ! systemctl is-active mydaemoned.service; then
    echo "Error: Service failed to start"
    journalctl -u mydaemoned.service -n 50
    exit 1
fi

echo "Service deployed and started successfully"
```

## 参考

- BuildTemplate Daemon.cmake.in 文档
- systemd/README.md - 完整systemd配置指南
- systemd.service(5) - Systemd服务单元文档
