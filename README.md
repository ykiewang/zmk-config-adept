# zmk-config-adept

ZMK config for 3-pin modded Ploopy Adept BLE

Features: ZMK Studio | Scroll layer | Cursor acceleration | 125Hz report rate | KeyBeacon status over BLE

---

**[English](#english) | [中文](#中文)**

---

## English

### Physical Layout

6 buttons in two rows:

```
 [0]  [1]  [2]  [3]      ← Upper buttons (0 & 3 slightly lower than 1 & 2)
 [4]            [5]      ← Lower buttons (left click, right click)
```

Trackball sits in the center area.

### Layer 0: Mouse Layer (Default)

Trackball controls cursor with acceleration curve (1.0x ~ 3.0x).

| Key | Function | Note |
|-----|----------|------|
| 0 | Ctrl+PageUp | Keyboard shortcut |
| 1 | Mouse Back (MB4) | Mouse button 4 |
| 2 | Mouse Forward (MB5) | Mouse button 5 |
| 3 | **Tap**: Ctrl+PageDown / **Hold**: Temp scroll layer | Layer-Tap behavior |
| 4 | Left Click (MB1) | — |
| 5 | Right Click (MB2) | — |

> Key 3 is a Layer-Tap: tap sends Ctrl+PageDown, hold temporarily activates scroll layer (returns on release).

### Layer 1: Scroll Layer

Trackball converts to scroll wheel (XY swap + Y invert + 1:16 scaling). Left/right click unchanged.

| Key | Function | Note |
|-----|----------|------|
| 0 | Play/Pause | Media control |
| 1 | Volume Down | Media control |
| 2 | Volume Up | Media control |
| 3 | Back to Mouse Layer | Permanently returns to Layer 0 |
| 4 | Left Click (MB1) | — |
| 5 | Right Click (MB2) | — |

**How to enter (two ways):**
- **Temporary**: Hold key 3 on Layer 0, auto-returns on release
- **Locked**: Press key 2 + key 3 simultaneously (combo) on Layer 0, press key 3 to return

### Layer 2 / 3: Reserved

Marked as `reserved`, no bindings. Configurable via ZMK Studio or keymap file.

### Layer 4: ZMK Studio Unlock

Only key 0 is bound to `studio_unlock`, others pass through. Used for ZMK Studio permission unlock.

### Combos

Press specified keys simultaneously to trigger. Works across layers without switching.

| Action | Key Combo | Layers | Note |
|--------|-----------|--------|------|
| Middle Click (MB3) | 4 + 5 | 0, 1 | Press left + right click together |
| Switch to Scroll Layer | 2 + 3 | 0 | Locked switch to Layer 1 |
| ESC | 0 + 1 | 0, 1 | — |
| Close Tab | 1 + 2 | 0, 1 | Ctrl+W |
| Switch App | 0 + 4 | 0, 1 | Alt+Tab |
| Close App | 0 + 3 | 0, 1 | Alt+F4 |
| Clear BLE Pairing | 0 + 1 + 2 + 3 | 0 | All 4 upper keys |
| Enter Bootloader | 0 + 3 + 4 + 5 | 0 | For firmware flashing |

### Acceleration Curve Tuning

The cursor speeds up when you move the trackball faster. Edit `&pointer_accel` in `boards/shields/adept/adept_board.overlay` to adjust. Rebuild and flash after changes.

**Common adjustments:**

| I want to... | Parameter | Plain explanation |
|--------------|-----------|-------------------|
| Cursor moves too slow overall | Increase `sensitivity` (e.g. 800 → 1000) | Think of it as a global speed dial for the cursor |
| Cursor moves too fast overall | Decrease `sensitivity` (e.g. 800 → 600) | Turns down the base speed |
| Fast flicks don't go far enough | Increase `max-factor` (e.g. 3000 → 4000) | Raises the top speed ceiling when you flick hard |
| Fast flicks are too jumpy | Decrease `max-factor` (e.g. 3000 → 2000) | Lowers the top speed ceiling so it doesn't fly away |
| Acceleration kicks in too early | Increase `speed-threshold` (e.g. 900 → 1200) | You need to move faster before the speed boost starts |
| Acceleration kicks in too late | Decrease `speed-threshold` (e.g. 900 → 600) | Even gentle movements get a speed boost |
| Want near-zero acceleration (linear feel) | Set `min-factor` and `max-factor` to the same value | Cursor moves at a constant ratio regardless of speed |

> Values use 1/1000 scale: 1000 = 1.0x, 800 = 0.8x, 3000 = 3.0x

**Example: "I want less acceleration and more precision"**

In `boards/shields/adept/adept_board.overlay`, find the `&pointer_accel` section and change:

```dts
&pointer_accel {
    // ...
    sensitivity = <800>;              // was 800, keep as is
    max-factor = <2000>;              // was 3000 → lower top speed for less jumpiness
    speed-threshold = <1200>;         // was 900  → need faster movement to trigger acceleration
    speed-max = <2400>;               // was 1800 → wider ramp-up range, smoother transition
    min-factor = <1000>;              // was 1000, keep as is
    // ...
};
```

### Quick Reference

- **Browser tabs**: Key 0 (prev) / Tap key 3 (next)
- **Browser nav**: Key 1 (back) / Key 2 (forward)
- **Scroll**: Hold key 3 + move trackball, or combo 2+3 to lock scroll mode
- **Middle click**: Press left + right (4+5) together
- **Media**: Enter scroll layer, key 0/1/2 = play-pause / vol down / vol up
- **BLE re-pair**: Hold all 4 upper keys
- **Flash firmware**: Combo 0+3+4+5 for bootloader, or open the back cover and double-tap the RESET button on the MCU

### KeyBeacon status over BLE

The firmware ships [KeyBeacon](https://github.com/ykiewang/keybeacon) wired in: the Adept reports
its live layer, held modifiers, host/profile connection state and battery over the host Bluetooth
link, so a desktop app can show them without adding a screen to the keyboard.

- Module: [`zmk-keybeacon`](https://github.com/ykiewang/zmk-keybeacon), pinned in
  `config/west.yml`; the Kconfig switches live in `boards/shields/adept/adept_board.conf`.
- Adept is unibody, so it is always the host-link role — no split-role setup needed.
- Grab the app from the [KeyBeacon releases](https://github.com/ykiewang/keybeacon/releases)
  (macOS / Windows). Hosts paired before the upgrade must re-pair once, because the GATT database
  now carries the KeyBeacon service.

### How to Modify, Build and Flash

1. **Fork** this repo (or clone it directly if you have write access)
2. **Edit** the config files on GitHub or locally:
   - `config/adept.keymap` — key bindings, combos, layers
   - `config/adept.conf` — Kconfig options (report rate, BLE, sensor sleep, etc.)
   - `boards/shields/adept/adept_board.overlay` — acceleration curve, input pipeline
3. **Commit & Push** — GitHub Actions will automatically build the firmware
4. **Download** — go to the repo's **Actions** tab, click the latest successful run, download the `firmware` artifact (contains `.uf2` files)
5. **Flash** — enter bootloader (combo 0+3+4+5, or open back cover and double-tap MCU RESET), drag `zmk.uf2` to the mounted USB drive

---

## 中文

### 物理布局

鼠标共 6 个按键，分上下两排：

```
 [0]  [1]  [2]  [3]      ← 上排按键（0、3 略低于 1、2）
 [4]            [5]      ← 下排按键（左键、右键）
```

轨迹球位于按键中央区域。

### Layer 0：鼠标层（默认）

轨迹球控制光标移动，带加速曲线（1.0x ~ 3.0x）。

| 键位 | 功能 | 说明 |
|------|------|------|
| 0 | Ctrl+PageUp | 键盘快捷键 |
| 1 | 鼠标后退 (MB4) | 鼠标第 4 键 |
| 2 | 鼠标前进 (MB5) | 鼠标第 5 键 |
| 3 | **点按**: Ctrl+PageDown / **长按**: 临时切换到滚动层 | Layer-Tap 行为 |
| 4 | 左键 (MB1) | — |
| 5 | 右键 (MB2) | — |

> 键 3 是 Layer-Tap 行为：轻点发送 Ctrl+PageDown，按住不放则临时进入滚动层（松手自动回来）。

### Layer 1：滚动层

轨迹球转换为滚轮（XY 交换 + Y 反转 + 1:16 缩放），左右键保持不变。

| 键位 | 功能 | 说明 |
|------|------|------|
| 0 | 播放/暂停 | 媒体控制 |
| 1 | 音量减 | 媒体控制 |
| 2 | 音量加 | 媒体控制 |
| 3 | 切回鼠标层 | 永久切回 Layer 0 |
| 4 | 左键 (MB1) | — |
| 5 | 右键 (MB2) | — |

**进入方式（二选一）：**
- **临时切换**：在 Layer 0 长按键 3，松手自动回 Layer 0
- **锁定切换**：在 Layer 0 同时按键 2 + 键 3（组合键），按键 3 切回

### Layer 2 / 3：预留层

当前标记为 `reserved`，无绑定。可通过 ZMK Studio 或修改 keymap 文件配置。

### Layer 4：ZMK Studio 解锁层

仅键 0 绑定 `studio_unlock`，其余透传。用于 ZMK Studio 在线配置时的权限解锁。

### 组合键

同时按下指定键位触发，无需切层。

| 操作 | 按键组合 | 可用层 | 说明 |
|------|----------|--------|------|
| 中键 (MB3) | 4 + 5 | 0, 1 | 同时按左右键 |
| 切换到滚动层 | 2 + 3 | 0 | 锁定进入 Layer 1 |
| ESC | 0 + 1 | 0, 1 | — |
| 关闭标签页 | 1 + 2 | 0, 1 | Ctrl+W |
| 切换应用 | 0 + 4 | 0, 1 | Alt+Tab |
| 关闭应用 | 0 + 3 | 0, 1 | Alt+F4 |
| 清除蓝牙配对 | 0 + 1 + 2 + 3 | 0 | 上排四键全按 |
| 进入 Bootloader | 0 + 3 + 4 + 5 | 0 | 用于刷固件 |

### 加速曲线调参

光标会在快速滑动轨迹球时自动加速。修改 `boards/shields/adept/adept_board.overlay` 中的 `&pointer_accel` 节点来调整手感，改完需重新编译刷入固件。

**常见调整：**

| 我想要... | 怎么改 | 通俗解释 |
|-----------|--------|----------|
| 光标整体太慢 | 调大 `sensitivity`（如 800 → 1000） | 相当于全局速度旋钮，越大光标越快 |
| 光标整体太快 | 调小 `sensitivity`（如 800 → 600） | 把基础速度调低 |
| 快速甩动不够远 | 调大 `max-factor`（如 3000 → 4000） | 提高猛甩时的速度上限，甩得更远 |
| 快速甩动太飘 | 调小 `max-factor`（如 3000 → 2000） | 降低速度上限，猛甩也不会飞太远 |
| 加速太灵敏，轻轻动就飞 | 调大 `speed-threshold`（如 900 → 1200） | 要滑得更快才会触发加速 |
| 加速太迟钝，要很用力才加速 | 调小 `speed-threshold`（如 900 → 600） | 轻轻滑就能触发加速 |
| 不想要加速，想要线性手感 | 把 `min-factor` 和 `max-factor` 设成一样 | 不管滑多快，光标速度倍率都一样 |

> 数值用千分比：1000 = 1.0 倍，800 = 0.8 倍，3000 = 3.0 倍

**示例：「我想要少一点加速、多一点精确」**

打开 `boards/shields/adept/adept_board.overlay`，找到 `&pointer_accel` 部分，修改为：

```dts
&pointer_accel {
    // ...
    sensitivity = <800>;              // 原 800，保持不变
    max-factor = <2000>;              // 原 3000 → 降低极速，甩动不会飞太远
    speed-threshold = <1200>;         // 原 900  → 要滑得更快才开始加速
    speed-max = <2400>;               // 原 1800 → 加速过渡更平缓
    min-factor = <1000>;              // 原 1000，保持不变
    // ...
};
```

修改后需重新构建固件并刷入。

### 日常使用速查

- **浏览器标签页切换**：键 0（上一页）/ 点按键 3（下一页）
- **浏览器前进后退**：键 1（后退）/ 键 2（前进）
- **滚动网页**：长按键 3 + 滚轨迹球，或按组合键 2+3 锁定滚动模式
- **中键点击**：同时按左右键（4+5）
- **媒体控制**：进入滚动层后，键 0/1/2 分别为播放暂停/音量减/音量加
- **蓝牙重新配对**：上排四键同时按住
- **刷固件**：同时按键 0+3+4+5 进入 Bootloader，或打开后盖双击主控上的 RESET 按钮

### 通过 BLE 上报 KeyBeacon 状态

固件已内置 [KeyBeacon](https://github.com/ykiewang/keybeacon)：Adept 会通过主机蓝牙链路实时上报
当前层、按住的修饰键、主机/配对槽连接状态与电量，桌面端 app 即可显示，不需要给键盘加屏幕。

- 模块：[`zmk-keybeacon`](https://github.com/ykiewang/zmk-keybeacon)，在 `config/west.yml` 中锁版本；
  Kconfig 开关位于 `boards/shields/adept/adept_board.conf`。
- Adept 是一体式键盘，天然就是 host-link 角色，无需分体角色配置。
- app 见 [KeyBeacon releases](https://github.com/ykiewang/keybeacon/releases)（macOS / Windows）。
  刷入新固件后，之前配对过的主机需要重新配对一次，因为 GATT 数据库新增了 KeyBeacon 服务。

### 如何修改代码、构建并刷入固件

1. **Fork** 本仓库（或直接 clone，如果你有写权限）
2. **修改**配置文件（在 GitHub 网页上或本地编辑）：
   - `config/adept.keymap` — 按键绑定、组合键、层
   - `config/adept.conf` — Kconfig 选项（报告率、BLE、传感器休眠等）
   - `boards/shields/adept/adept_board.overlay` — 加速曲线、输入处理流水线
3. **提交并推送** — GitHub Actions 会自动构建固件
4. **下载固件** — 进入仓库的 **Actions** 页面，点击最新一次成功的构建，下载 `firmware` 产物（包含 `.uf2` 文件）
5. **刷入** — 进入 Bootloader（组合键 0+3+4+5，或打开后盖双击主控 RESET），将 `zmk.uf2` 拖入弹出的 USB 存储盘
