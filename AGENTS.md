# AGENTS.md

This file provides guidance to codeflicker when working with code in this repository.

## WHY: 项目目标

为 Ploopy Adept BLE 鼠标（3 针改装版）提供 ZMK 固件配置，基于 Seeeduino XIAO BLE (nRF52840)，实现 6 键布局 + 光学追踪 + BLE 无线连接。

## WHAT: 技术栈

- **固件框架**: ZMK Firmware (Zephyr RTOS)
- **硬件**: Seeeduino XIAO BLE (nRF52840) + PMW3610 光学传感器
- **依赖管理**: west (west.yml)
- **关键模块**: PMW3610 驱动 (efogdev)、报告率限制器 (badjeff)、指针加速 (nuovotaka)
- **CI/CD**: GitHub Actions 自动编译

## HOW: 核心开发流程（Podman 容器）

```bash
# 启动容器（宿主机）
podman run -it --rm --security-opt label=disable \
  --workdir /workspace \
  -v ~/kbd/zmk:/workspace \
  -v ~/kbd/zmk-config-adept:/workspaces/zmk-config \
  zmk-dev /bin/bash

# 容器内：更新依赖
west update

# 容器内：构建固件
west build -b xiao_ble -s zmk/app -- \
  -DZMK_CONFIG="/workspaces/zmk-config/config" \
  -DSHIELD=adept_board \
  '-DZMK_EXTRA_MODULES=/workspaces/zmk-config;/workspace/zmk-pmw3610-driver;/workspace/zmk-input-processor-report-rate-limit;/workspace/zmk-pointing-acceleration-alpha'

# 闪烧：双击 RESET 进入 Bootloader，拖拽 build/zephyr/zmk.uf2
```

主要修改文件：
- `config/adept.keymap` — 键盘映射与行为
- `config/adept.conf` — Kconfig 编译选项
- `boards/shields/adept/adept_board.overlay` — 硬件输入处理配置

## 渐进式文档

- `docs/agent/architecture.md` — 模块结构与架构模式
- `docs/agent/development_commands.md` — 完整构建与配置命令参考

**处理任务时，先判断哪些文档与当前任务相关，再按需查阅。**
