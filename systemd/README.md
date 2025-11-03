# Systemd Service Configuration Guide

## 概述

BuildTemplate的Daemon模板提供了完整的systemd集成支持，包括：

- 📝 自动生成systemd service文件
- 🔧 丰富的配置选项（资源限制、安全加固、依赖管理）
- 🚀 自动安装脚本（可选自动注册、启用、启动）
- 🔒 安全加固选项
- 📊 完整的生命周期管理

## 目录结构

```
BuildTemplate/systemd/
├── service.cmake.in           # Systemd service单元模板
├── preset.cmake.in            # Systemd预设配置模板
├── install-service.sh.in      # 服务安装脚本模板
└── uninstall-service.sh.in    # 服务卸载脚本模板
```

## 配置变量详解

### 基础配置

```cmake
# 启用systemd支持
set(ENABLE_DAEMON_WITH_SYSTEMD ON)

# 服务描述（显示在systemctl status中）
set(MODULE_DAEMONDESCRIPTION "My Application Daemon")
```

### 依赖管理

```cmake
# 在指定服务之后启动
set(MODULE_DAEMONAFTER "network.target postgresql.service")

# 在指定服务之前启动
set(MODULE_DAEMONBEFORE "nginx.service")

# 硬依赖（如果依赖服务失败，本服务也失败）
set(MODULE_DAEMONREQUIRE "postgresql.service")
```

### 服务行为

```cmake
# 服务类型
# - simple: 默认，ExecStart进程是主进程
# - forking: ExecStart进程fork子进程后退出
# - notify: 类似simple，但守护进程会通过sd_notify()通知systemd
# - dbus: 类似simple，但等待D-Bus名称出现
set(MODULE_DAEMON_SERVICE_TYPE "notify")

# 进程终止方式
# - process: 仅杀死主进程
# - control-group: 杀死控制组中所有进程
# - mixed: 主进程用SIGTERM，其他用SIGKILL
set(MODULE_DAEMON_KILL_MODE "control-group")

# 重启策略
# - no: 不重启
# - on-success: 仅在正常退出时重启
# - on-failure: 仅在异常退出时重启
# - on-abnormal: 异常信号或超时时重启
# - always: 总是重启
set(MODULE_DAEMON_RESTART "on-failure")

# 看门狗超时（秒）
set(MODULE_DAEMON_WATCHDOG_SEC "30")
```

### 资源限制

```cmake
# 核心转储大小（infinity表示无限制）
set(MODULE_DAEMON_CORE_DUMP "infinity")

# 最大打开文件数
set(MODULE_DAEMON_MAX_FILES "65536")

# 内存限制（支持K, M, G单位）
set(MODULE_DAEMON_MEMORY_LIMIT "1G")

# CPU配额（百分比）
set(MODULE_DAEMON_CPU_QUOTA "50%")
```

### 启动限制

```cmake
# 限制时间窗口内的重启次数
set(MODULE_DAEMON_START_LIMIT_INTERVAL "1min")
set(MODULE_DAEMON_START_LIMIT_BURST "5")
# 含义：1分钟内最多重启5次
```

### 安全加固

```cmake
# 启用安全加固
set(MODULE_DAEMON_ENABLE_SECURITY ON)
# 包括：
# - PrivateTmp=yes         (私有/tmp目录)
# - NoNewPrivileges=yes    (禁止提权)
# - ProtectSystem=strict   (只读系统目录)
# - ProtectHome=yes        (禁止访问用户家目录)

# 指定运行用户和组
set(MODULE_DAEMON_USER "myapp")
set(MODULE_DAEMON_GROUP "myapp")

# 工作目录
set(MODULE_DAEMON_WORKING_DIR "/var/lib/myapp")

# 环境变量
set(MODULE_DAEMON_ENVIRONMENT "MYAPP_CONFIG=/etc/myapp/config.ini")
```

### 安装行为

```cmake
# WantedBy目标（服务在哪个target下启用）
set(MODULE_DAEMON_WANTED_BY "multi-user.target")

# 服务别名
set(MODULE_DAEMON_ALIAS "myapp.service")

# 安装时自动注册到systemd
set(MODULE_DAEMON_AUTO_REGISTER ON)

# 安装时自动启用服务（systemctl enable）
set(MODULE_DAEMON_AUTO_ENABLE ON)

# 安装时自动启动服务（systemctl start）
set(MODULE_DAEMON_AUTO_START OFF)  # 生产环境建议OFF
```

## 使用示例

### 示例1: 简单守护进程

```cmake
# CMakeLists.txt
set(ENABLE_BUILD_DAEMON ON)
set(ENABLE_DAEMON_WITH_SYSTEMD ON)
set(MODULE_DAEMON_DIR ${CMAKE_CURRENT_SOURCE_DIR}/daemon)
set(MODULE_EXTERNAL_DAEMON_LIB lap_core)
set(MODULE_DAEMONDESCRIPTION "Simple Application Daemon")
include(BuildTemplate/Daemon.cmake.in)
```

生成的service文件：
```ini
[Unit]
Description=Simple Application Daemon
After=network.target

[Service]
Type=notify
ExecStart=/usr/bin/cored
Restart=on-failure
...

[Install]
WantedBy=multi-user.target
```

### 示例2: 数据库依赖的守护进程

```cmake
set(ENABLE_BUILD_DAEMON ON)
set(ENABLE_DAEMON_WITH_SYSTEMD ON)
set(MODULE_DAEMON_DIR ${CMAKE_CURRENT_SOURCE_DIR}/daemon)
set(MODULE_EXTERNAL_DAEMON_LIB lap_core pq)

# 依赖配置
set(MODULE_DAEMONDESCRIPTION "Database Application Daemon")
set(MODULE_DAEMONAFTER "network.target postgresql.service")
set(MODULE_DAEMONREQUIRE "postgresql.service")

# 资源限制
set(MODULE_DAEMON_MEMORY_LIMIT "2G")
set(MODULE_DAEMON_MAX_FILES "10000")

include(BuildTemplate/Daemon.cmake.in)
```

### 示例3: 生产环境安全加固

```cmake
set(ENABLE_BUILD_DAEMON ON)
set(ENABLE_DAEMON_WITH_SYSTEMD ON)
set(MODULE_DAEMON_DIR ${CMAKE_CURRENT_SOURCE_DIR}/daemon)
set(MODULE_EXTERNAL_DAEMON_LIB lap_core lap_log)

# 基本配置
set(MODULE_DAEMONDESCRIPTION "Production Application Daemon")
set(MODULE_DAEMONAFTER "network-online.target")

# 安全配置
set(MODULE_DAEMON_USER "appuser")
set(MODULE_DAEMON_GROUP "appgroup")
set(MODULE_DAEMON_WORKING_DIR "/var/lib/myapp")
set(MODULE_DAEMON_ENABLE_SECURITY ON)

# 资源限制
set(MODULE_DAEMON_MEMORY_LIMIT "512M")
set(MODULE_DAEMON_CPU_QUOTA "25%")
set(MODULE_DAEMON_MAX_FILES "4096")
set(MODULE_DAEMON_CORE_DUMP "0")  # 禁用核心转储

# 重启策略
set(MODULE_DAEMON_RESTART "always")
set(MODULE_DAEMON_START_LIMIT_INTERVAL "10min")
set(MODULE_DAEMON_START_LIMIT_BURST "3")

# 自动安装（仅注册和启用，不自动启动）
set(MODULE_DAEMON_AUTO_REGISTER ON)
set(MODULE_DAEMON_AUTO_ENABLE ON)
set(MODULE_DAEMON_AUTO_START OFF)

include(BuildTemplate/Daemon.cmake.in)
```

### 示例4: 开发环境快速启动

```cmake
set(ENABLE_BUILD_DAEMON ON)
set(ENABLE_DAEMON_WITH_SYSTEMD ON)
set(MODULE_DAEMON_DIR ${CMAKE_CURRENT_SOURCE_DIR}/daemon)
set(MODULE_EXTERNAL_DAEMON_LIB lap_core)

# 开发环境：自动启动
set(MODULE_DAEMONDESCRIPTION "Development Daemon")
set(MODULE_DAEMON_AUTO_REGISTER ON)
set(MODULE_DAEMON_AUTO_ENABLE ON)
set(MODULE_DAEMON_AUTO_START ON)  # 开发环境可以自动启动

include(BuildTemplate/Daemon.cmake.in)
```

## 服务管理

### 自动管理（推荐）

如果设置了 `MODULE_DAEMON_AUTO_REGISTER ON`，安装时会自动执行注册：

```bash
cmake --build build
sudo cmake --install build

# 服务已自动注册
# 如果AUTO_ENABLE=ON，已自动启用
# 如果AUTO_START=ON，已自动启动
```

### 手动管理

如果未启用自动注册，使用安装脚本：

```bash
# 注册并启用服务
sudo /usr/lib/systemd/scripts/install-cored.sh

# 卸载服务
sudo /usr/lib/systemd/scripts/uninstall-cored.sh
```

### 标准systemctl命令

```bash
# 启动服务
sudo systemctl start cored.service

# 停止服务
sudo systemctl stop cored.service

# 重启服务
sudo systemctl restart cored.service

# 重新加载配置（不重启）
sudo systemctl reload cored.service

# 查看服务状态
systemctl status cored.service

# 启用开机自启
sudo systemctl enable cored.service

# 禁用开机自启
sudo systemctl disable cored.service

# 查看服务日志
journalctl -u cored.service

# 实时跟踪日志
journalctl -u cored.service -f

# 查看最近100行日志
journalctl -u cored.service -n 100

# 查看今天的日志
journalctl -u cored.service --since today
```

## 故障排查

### 服务启动失败

```bash
# 查看详细状态
systemctl status cored.service -l

# 查看完整日志
journalctl -u cored.service -xe

# 检查service文件语法
systemd-analyze verify /lib/systemd/system/cored.service

# 查看依赖树
systemctl list-dependencies cored.service
```

### 常见问题

#### 1. 权限问题

```bash
# 检查可执行文件权限
ls -l /usr/bin/cored

# 检查service文件权限
ls -l /lib/systemd/system/cored.service

# 确保正确的所有者
sudo chown root:root /lib/systemd/system/cored.service
sudo chmod 644 /lib/systemd/system/cored.service
```

#### 2. 环境变量问题

systemd服务运行在受限环境中，需要在service文件中显式设置：

```cmake
set(MODULE_DAEMON_ENVIRONMENT "PATH=/usr/local/bin:/usr/bin:/bin LD_LIBRARY_PATH=/usr/local/lib")
```

#### 3. 工作目录问题

如果守护进程依赖特定工作目录：

```cmake
set(MODULE_DAEMON_WORKING_DIR "/var/lib/myapp")
```

并确保目录存在且有正确权限：

```bash
sudo mkdir -p /var/lib/myapp
sudo chown myappuser:myappgroup /var/lib/myapp
```

#### 4. 依赖服务未启动

```bash
# 检查依赖服务状态
systemctl status postgresql.service

# 手动启动依赖
sudo systemctl start postgresql.service

# 或修改依赖配置
set(MODULE_DAEMONREQUIRE "")  # 移除硬依赖
set(MODULE_DAEMONAFTER "network.target")  # 仅等待网络
```

## 最佳实践

### 1. 生产环境配置

```cmake
# ✅ 使用专用用户运行
set(MODULE_DAEMON_USER "appuser")
set(MODULE_DAEMON_GROUP "appgroup")

# ✅ 启用安全加固
set(MODULE_DAEMON_ENABLE_SECURITY ON)

# ✅ 设置资源限制
set(MODULE_DAEMON_MEMORY_LIMIT "1G")
set(MODULE_DAEMON_CPU_QUOTA "50%")

# ✅ 配置合理的重启策略
set(MODULE_DAEMON_RESTART "on-failure")
set(MODULE_DAEMON_START_LIMIT_INTERVAL "10min")
set(MODULE_DAEMON_START_LIMIT_BURST "3")

# ✅ 不自动启动（手动验证后启动）
set(MODULE_DAEMON_AUTO_START OFF)
```

### 2. 开发环境配置

```cmake
# ✅ 快速迭代
set(MODULE_DAEMON_AUTO_REGISTER ON)
set(MODULE_DAEMON_AUTO_ENABLE ON)
set(MODULE_DAEMON_AUTO_START ON)

# ✅ 宽松的资源限制
# 不设置或设置较大值

# ✅ 简单的重启策略
set(MODULE_DAEMON_RESTART "always")
```

### 3. 依赖管理

```cmake
# ✅ 明确声明依赖
set(MODULE_DAEMONAFTER "network-online.target database.service")

# ⚠️ 谨慎使用硬依赖
# 只在真正必须时使用Requires=
set(MODULE_DAEMONREQUIRE "critical-service.service")

# ✅ 使用Before协调启动顺序
set(MODULE_DAEMONBEFORE "web-frontend.service")
```

### 4. 日志管理

```cmake
# ✅ 配置日志环境变量
set(MODULE_DAEMON_ENVIRONMENT "LOG_LEVEL=INFO LOG_FILE=/var/log/myapp/daemon.log")
```

配合logrotate：

```bash
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    notifempty
    create 0640 appuser appgroup
    sharedscripts
    postrotate
        systemctl reload cored.service > /dev/null 2>&1 || true
    endscript
}
```

## 参考资料

- [systemd.service(5)](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
- [systemd.unit(5)](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)
- [systemd.exec(5)](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)
- [systemd.resource-control(5)](https://www.freedesktop.org/software/systemd/man/systemd.resource-control.html)
