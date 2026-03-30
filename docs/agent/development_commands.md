# 开发命令参考

## 本地环境初始化（不使用容器）

```bash
pip install west
west init -l config/
west update
west zephyr-export
```

## Podman 容器环境（推荐）

官方文档：https://zmk.dev/docs/development/local-toolchain/setup/container

### 实际目录结构（已验证）

```
~/kbd/
├── zmk/                                          # west workspace 根目录（含 .west/config）
│   ├── zmk/                                      # 官方 ZMK 源码（west update 拉取）
│   ├── zephyr/                                   # Zephyr RTOS（west update 拉取）
│   ├── zmk-pmw3610-driver/                       # PMW3610 驱动（west update 拉取）
│   ├── zmk-input-processor-report-rate-limit/   # 报告率限制（west update 拉取）
│   ├── zmk-pointing-acceleration-alpha/          # 指针加速（west update 拉取）
│   └── modules/                                  # 其他 Zephyr 模块
└── zmk-config-adept/                             # 本项目
```

注意：`~/kbd/zmk/.west/config` 中 `manifest.path` 需指向 `zmk-config-adept/config`。

### 容器镜像（已构建）

```bash
# 首次构建镜像（仅需一次）
podman build -t zmk-dev -f ~/kbd/zmk/.devcontainer/Dockerfile ~/kbd/zmk/.devcontainer
```

### 启动容器

```bash
podman run -it --rm \
  --security-opt label=disable \
  --workdir /workspace \
  -v ~/kbd/zmk:/workspace \
  -v ~/kbd/zmk-config-adept:/workspaces/zmk-config \
  zmk-dev /bin/bash
```

### 容器内初始化（首次或更新依赖）

```bash
# 切换 manifest 到本项目的 west.yml（首次需要）
west config manifest.path /workspaces/zmk-config/config
west config manifest.file west.yml

# 更新所有依赖（包括 zmk、zephyr、外部模块）
west update

# 注册 Zephyr CMake 包（首次需要）
west zephyr-export
```

### 容器内构建固件

```bash
# 构建主固件
west build -b xiao_ble -s zmk/app -- \
  -DZMK_CONFIG="/workspaces/zmk-config/config" \
  -DSHIELD=adept_board \
  '-DZMK_EXTRA_MODULES=/workspaces/zmk-config;/workspace/zmk-pmw3610-driver;/workspace/zmk-input-processor-report-rate-limit;/workspace/zmk-pointing-acceleration-alpha'

# 构建 settings_reset 固件（蓝牙配对清除）
west build -b xiao_ble -s zmk/app -- \
  -DZMK_CONFIG="/workspaces/zmk-config/config" \
  -DSHIELD="adept_board settings_reset" \
  '-DZMK_EXTRA_MODULES=/workspaces/zmk-config;/workspace/zmk-pmw3610-driver;/workspace/zmk-input-processor-report-rate-limit;/workspace/zmk-pointing-acceleration-alpha'

# 强制全量重新构建（当 build 缓存出错时，先在宿主机 mv ~/kbd/zmk/build ~/kbd/zmk/build.bak）
west build -b xiao_ble -s zmk/app -- \
  -DZMK_CONFIG="/workspaces/zmk-config/config" \
  -DSHIELD=adept_board \
  '-DZMK_EXTRA_MODULES=/workspaces/zmk-config;/workspace/zmk-pmw3610-driver;/workspace/zmk-input-processor-report-rate-limit;/workspace/zmk-pointing-acceleration-alpha'
```

固件产物：`~/kbd/zmk/build/zephyr/zmk.uf2`（宿主机可直接访问）
west build -p -b xiao_ble -s app -- \
  -DZMK_CONFIG="/workspaces/zmk-config/config" \
  -DSHIELD=adept_board
## 闪烧固件

容器内无法直接 `west flash`，使用 UF2 方式：
1. 双击鼠标上的 RESET 按钮进入 Bootloader
2. 设备挂载为 USB 存储盘
3. 将 `build/zephyr/zmk.uf2` 拖拽到挂载盘

## Podman 容器管理

```bash
podman ps                        # 列出运行中容器
podman stop "<container-id>"     # 停止容器
podman rm "<container-id>"       # 删除容器
podman volume ls                 # 列出卷
podman volume rm zmk-config      # 删除卷
```

## 依赖管理

```bash
west update      # 更新到 west.yml 指定版本
west list        # 查看依赖状态
```

## CI/CD

GitHub Actions 自动在每次 push/PR 时构建固件：
- 工作流文件：`.github/workflows/build.yml`
- 构建目标由 `build.yaml` 定义
- 产物：`zmk.uf2` 固件文件

## 主要配置文件修改指南

| 需求 | 修改文件 |
|---|---|
| 修改按键绑定、组合键 | `config/adept.keymap` |
| 调整加速曲线参数 | `boards/shields/adept/adept_board.overlay` → `&pointer_accel {}` |
| 修改传感器 DPI | `boards/shields/adept/pmw3610.dtsi` → `cpi = <600>` |
| 修改报告率上限 | `config/adept.conf` → `CONFIG_PMW3610_REPORT_INTERVAL_MIN` |
| 修改 BLE 速率限制 | `boards/shields/adept/adept_board.overlay` → `zip_ble_report_rate_limit` |
| 调整传感器休眠时间 | `config/adept.conf` → `CONFIG_PMW3610_REST*` |
| 修改去抖动时间 | `config/adept.conf` → `CONFIG_ZMK_KSCAN_DEBOUNCE_*_MS` |
