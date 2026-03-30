# 架构文档

## 项目结构

```
zmk-config-adept/
├── config/                          # ZMK 主配置
│   ├── adept.keymap                 # 键盘映射（层、行为、组合键）
│   ├── adept.conf                   # Kconfig 编译选项
│   ├── info.json                    # ZMK Studio 键位元数据
│   └── west.yml                     # 依赖清单
├── boards/shields/adept/            # 硬件 Shield 定义
│   ├── adept_board.overlay          # 核心硬件集成配置（输入处理流水线）
│   ├── pmw3610.dtsi                 # 光学传感器 SPI 配置
│   ├── adept.dtsi                   # 基础设备树
│   ├── adept-layouts.dtsi           # 物理键位布局
│   ├── adept.zmk.yml                # Shield 元数据
│   ├── Kconfig.shield               # 条件编译配置
│   └── Kconfig.defconfig            # 默认 Kconfig 值
├── zephyr/module.yml                # Zephyr 模块注册
├── build.yaml                       # 构建目标（board/shield/snippet）
└── .github/workflows/build.yml      # GitHub Actions CI/CD
```

## 配置层次

```
adept.conf (Kconfig)         ← 编译期功能开关
       ↓
adept_board.overlay          ← 硬件初始化 & 输入处理
       ├── pmw3610.dtsi      ← 传感器 SPI 驱动配置
       ├── adept.dtsi        ← 基础设备树
       └── adept-layouts.dtsi ← 物理键位布局
       ↓
adept.keymap                 ← 运行期行为 & 键盘映射
```

## 输入处理流水线

`adept_board.overlay` 中定义的 `trackball_listener` 流水线：

**默认鼠标层：**
```
PMW3610 传感器原始数据
  → pointer_accel（加速曲线：0.8x~3.0x，阈值 900-1800 cps）
  → zip_ble_report_rate_limit（8ms 限速，约 125Hz 有效）
  → zip_xy_transform（XY 轴交换，修正传感器方向）
```

**滚动层（Layer 1）：**
```
PMW3610 传感器原始数据
  → zip_xy_transform（XY 交换 + Y 反转）
  → zip_xy_scaler（1:16 缩放）
  → zip_xy_to_scroll_mapper（转换为滚动事件）
```

## 键盘映射

**4 层结构：**

| 层 | 名称 | 激活方式 | 功能 |
|---|---|---|---|
| 0 | 鼠标层 | 默认 | MB1-5 + Ctrl+PageUp/Down |
| 1 | 滚动层 | Combo (键2+键3) | 媒体控制 + 滚轮 |
| Extra1/Extra2 | 保留 | — | — |
| 4 | Studio 解锁 | — | ZMK Studio 权限 |

**组合键（8 个）：**

| 组合 | 键位索引 | 功能 |
|---|---|---|
| 中键 | 4+5 | MB3 |
| 滚动层 | 2+3 | 切换 Layer 1 |
| ESC | 0+1 | Escape |
| 关闭标签 | 1+2 | Ctrl+W |
| 切换应用 | 0+4 | Alt+Tab |
| 关闭应用 | 0+3 | Alt+F4 |
| BT 清除 | 0+1+2+3 | BT_CLR |
| 进入 Bootloader | 0+3+4+5 | Bootloader |

## 外部依赖（west.yml）

| 模块 | 来源 | 分支 | 作用 |
|---|---|---|---|
| zmk | zmkfirmware/zmk | main | ZMK 主框架 |
| zmk-pmw3610-driver | efogdev | zephyr-4.1 | 光学传感器驱动 |
| zmk-input-processor-report-rate-limit | badjeff | main | 报告率限制 |
| zmk-pointing-acceleration-alpha | nuovotaka | main | 指针加速算法 |

## 硬件配置要点

- **MCU**: nRF52840 (Seeeduino XIAO BLE)
- **传感器**: PMW3610，600 DPI，SPI 2MHz，GPIO 中断
- **BLE TX**: +8dBm（最高档）
- **报告率上限**: 250Hz（CONFIG_PMW3610_REPORT_INTERVAL_MIN=4）
- **有效 BLE 报告率**: ~125Hz（受 8ms 速率限制器约束）
- **传感器休眠**: 三级（REST1: 20ms，REST2: 200ms，REST3: 300ms）
