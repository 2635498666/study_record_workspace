# RK3566 Linux SDK 学习笔记大纲

## 1. 学习背景与目标

### 1.1 当前环境

- 开发板：立创泰山派 RK3566
- WSL 发行版：Ubuntu 22.04
- SDK 根目录：`/home/zm/tsp-rk3566-build-env/tsp_rk3566`
- 编译方式：Docker 编译环境
- 当前板级配置：`device/rockchip/rk356x/BoardConfig-rk3566-tspi-v10.mk`
- 当前内核配置：`rockchip_linux_defconfig`
- 当前设备树：`kernel/arch/arm64/boot/dts/rockchip/tspi-rk3566-user-v10-linux.dts`
- 当前 Buildroot 配置：`rockchip_rk3566`

### 1.2 已完成内容

- [x] 搭建 Docker 编译环境
- [x] 执行 `./build.sh all`
- [x] 生成完整固件
- [x] 完成整体 `update.img` 下载
- [x] 验证开发板能够正常启动

### 1.3 后续学习目标

- [ ] 理解 SDK 的目录结构和构建流程
- [ ] 掌握各模块的独立编译
- [ ] 掌握各分区镜像的独立下载
- [ ] 掌握设备树的阅读、修改和验证
- [ ] 掌握 Buildroot 与根文件系统定制
- [ ] 掌握内核配置、驱动和模块编译
- [ ] 理解 U-Boot、Loader、Kernel 和 rootfs 的启动关系
- [ ] 掌握完整固件的重新打包和故障恢复

---

## 2. 实验记录规范

每次实验按以下模板记录，避免只记录命令而没有记录原因和结果。

### 2.1 实验信息

- 日期：
- 实验名称：
- 实验目标：
- 修改前现象：
- 预期结果：
- 使用的板级配置：
- 使用的 Git/repo 版本：

### 2.2 修改内容

- 修改文件：
- 修改节点或配置项：
- 修改原因：
- 修改前内容：
- 修改后内容：

### 2.3 编译与镜像

- 编译命令：
- 编译耗时：
- 编译结果：
- 关键日志：
- 生成镜像：
- 镜像时间和大小：

### 2.4 下载与验证

- 下载工具：
- 下载分区：
- 下载命令或操作：
- 串口启动日志：
- 板端验证命令：
- 实际结果：
- 是否达到预期：

### 2.5 问题与复盘

- 遇到的问题：
- 根本原因：
- 排查过程：
- 解决方法：
- 如何回退：
- 本次掌握的知识：
- 后续待确认内容：

---

## 3. 阶段一：理解 SDK、镜像与分区

### 3.1 学习目标

- 理解 `build.sh`、`BoardConfig` 和各子工程之间的关系。
- 区分“编译模块”“整理镜像”和“打包完整固件”。
- 建立修改内容、输出镜像和下载分区之间的对应关系。

### 3.2 重点目录

- `device/rockchip/`：板级配置、分区表和公共构建脚本
- `kernel/`：Linux 内核、驱动和设备树
- `u-boot/`：U-Boot、Loader 相关源码与产物
- `buildroot/`：根文件系统和软件包配置
- `app/`：应用程序源码
- `external/`：外部组件和驱动源码
- `rockdev/`：供下载和打包使用的镜像集合
- `tools/`：打包、升级和烧写工具
- `docs/`：Rockchip SDK 文档

### 3.3 构建命令的区别

| 命令 | 作用 | 是否重新编译源码 |
|---|---|---|
| `./build.sh all` | 编译 U-Boot、Kernel、rootfs、Recovery | 是 |
| `./build.sh firmware` | 整理各分区镜像到 `rockdev` | 通常否 |
| `./build.sh updateimg` | 将分区镜像封装成 `update.img` | 否 |
| `./build.sh allsave` | 完整编译、整理、打包并保存调试资料 | 是 |

### 3.4 镜像和分区关系

| 修改对象 | 推荐编译命令 | 主要镜像 | 下载分区 |
|---|---|---|---|
| U-Boot | `./build.sh uboot` | `rockdev/uboot.img` | `uboot` |
| Kernel/DTB | `./build.sh kernel` | `rockdev/boot.img` | `boot` |
| Buildroot/rootfs | `./build.sh rootfs` | `rockdev/rootfs.img` | `rootfs` |
| Recovery | `./build.sh recovery` | `rockdev/recovery.img` | `recovery` |
| 完整系统 | `./build.sh firmware updateimg` | `rockdev/update.img` | 整体升级 |

### 3.5 实验一：不修改源码，单独编译 Kernel

- [ ] 记录编译前 `boot.img` 的时间和大小
- [ ] 执行 `./build.sh kernel`
- [ ] 确认目标 DTS 为 `tspi-rk3566-user-v10-linux`
- [ ] 检查 `kernel/boot.img`
- [ ] 检查 `rockdev/boot.img` 的符号链接和目标文件
- [ ] 单独下载 `boot` 分区
- [ ] 重启并验证系统正常启动

参考命令：

```bash
./build.sh kernel
ls -lh kernel/boot.img rockdev/boot.img
readlink -f rockdev/boot.img
./rkflash.sh boot
./rkflash.sh rd
```

### 3.6 阶段验收

- [ ] 能解释 `all`、`firmware`、`updateimg` 的区别
- [ ] 能根据修改内容判断需要编译哪个模块
- [ ] 能根据镜像判断应下载哪个分区
- [ ] 能独立完成一次无源码修改的 Kernel 编译和 `boot` 分区下载

---

## 4. 阶段二：设备树基础与第一次修改

### 4.1 学习目标

- 理解 DTS、DTSI、DTB 和 FIT `boot.img` 的关系。
- 理解设备树节点、属性、标签、引用和覆盖写法。
- 完成一个风险较低、结果可观察的设备树修改。

### 4.2 当前设备树继承关系

入口文件：

```text
kernel/arch/arm64/boot/dts/rockchip/tspi-rk3566-user-v10-linux.dts
```

主要包含文件：

```text
rk3566.dtsi
tspi-rk3566-core-v10.dtsi
tspi-rk3566-hdmi-v10.dtsi
tspi-rk3566-csi-v10.dtsi
```

学习原则：

- 优先在板级 DTS 的用户定义区域修改。
- 初期不直接修改 `rk3566.dtsi`。
- 每次只修改一个节点或一个属性。
- 修改前保留可用的 `boot.img`。

### 4.3 需要掌握的语法

- `/ { ... };` 根节点
- `node-name@address` 节点命名
- `label: node-name` 标签
- `&label { ... };` 引用和覆盖节点
- `compatible` 与驱动匹配
- `reg` 地址和长度
- `interrupts` 中断配置
- `clocks`、`resets`、`power-domains`
- `pinctrl-names`、`pinctrl-0`
- `status = "okay"` 与 `status = "disabled"`
- GPIO 编号、极性和上下拉

### 4.4 实验二：修改三色灯触发方式

实验建议：将蓝灯的 `linux,default-trigger` 从 `timer` 修改为 `heartbeat`。

- [ ] 备份当前可用的 `rockdev/boot.img`
- [ ] 找到 `rgb_led_b` 节点
- [ ] 只修改 `linux,default-trigger`
- [ ] 执行 `./build.sh kernel`
- [ ] 检查 DTS/DTB 编译是否出现错误或警告
- [ ] 单独下载 `boot.img`
- [ ] 观察 LED 行为
- [ ] 通过 sysfs 检查 LED trigger

板端验证命令：

```bash
cat /proc/device-tree/model
ls -l /sys/class/leds
cat /sys/class/leds/*/trigger
dmesg | tail -n 100
```

### 4.5 阶段验收

- [ ] 能找到实际生效的板级 DTS
- [ ] 能解释为什么修改设备树后下载 `boot` 分区
- [ ] 能通过编译日志发现 DTS 语法问题
- [ ] 能从 `/proc/device-tree` 或 sysfs 验证设备树是否生效
- [ ] 能恢复修改前的 `boot.img`

---

## 5. 阶段三：常用外设设备树

### 5.1 推荐学习顺序

1. `status` 启用和禁用
2. GPIO LED
3. GPIO 按键
4. pinctrl 与引脚复用
5. UART
6. I2C
7. SPI
8. PWM
9. 显示、摄像头和以太网
10. regulator、clock、reset 和 power-domain

### 5.2 GPIO 与 pinctrl 实验

- [ ] 阅读 GPIO bank 和 `RK_PA0` 等引脚命名规则
- [ ] 理解 `GPIO_ACTIVE_HIGH/LOW`
- [ ] 理解 `pcfg_pull_none/up/down`
- [ ] 添加或修改一个 GPIO LED
- [ ] 检查是否与其他外设存在 pinmux 冲突

### 5.3 UART 实验

- [ ] 阅读当前 `uart3` 节点
- [ ] 找到 `uart3m1_xfer` 的 pinctrl 定义
- [ ] 确认对应物理引脚和电平
- [ ] 编译并下载 `boot.img`
- [ ] 检查 `/dev/ttyS*`
- [ ] 完成串口收发测试

### 5.4 I2C 实验

- [ ] 阅读当前 `i2c2` 或 `i2c3` 节点
- [ ] 确认总线引脚和 pinctrl
- [ ] 使用 `i2cdetect` 检查从设备地址
- [ ] 添加一个简单 I2C 设备节点
- [ ] 通过 `dmesg` 检查驱动匹配和 probe 结果

### 5.5 SPI 实验

- [ ] 阅读当前 `spi3` 节点
- [ ] 理解 CS、频率、工作模式和 DMA 属性
- [ ] 检查已有 `spi_test` 设备
- [ ] 确认 `/dev` 或 sysfs 中的设备节点
- [ ] 完成环回或外设通信测试

### 5.6 外设排障清单

- [ ] `status` 是否为 `okay`
- [ ] `compatible` 是否有对应驱动
- [ ] pinctrl 是否正确且没有复用冲突
- [ ] GPIO 极性是否正确
- [ ] 供电 regulator 是否启用
- [ ] clock 和 reset 是否完整
- [ ] 中断号和触发方式是否正确
- [ ] 驱动是否编进内核或已加载模块
- [ ] `dmesg` 中是否出现 probe defer 或错误

---

## 6. 阶段四：Buildroot、应用和 rootfs

### 6.1 学习目标

- 理解 Buildroot 配置、软件包、overlay 和 rootfs 镜像之间的关系。
- 掌握根文件系统的独立编译和下载。
- 掌握 SDK 中应用或外部组件的部分编译。

### 6.2 Rootfs 基础实验

- [ ] 找到当前 Buildroot defconfig
- [ ] 了解 overlay 目录
- [ ] 添加一个测试文件或启动脚本
- [ ] 执行 rootfs 编译
- [ ] 整理 `rockdev` 镜像
- [ ] 单独下载 `rootfs` 分区
- [ ] 启动后检查测试文件或服务

参考流程：

```bash
./build.sh rootfs
./build.sh firmware
ls -lh rockdev/rootfs.img
./rkflash.sh rootfs
./rkflash.sh rd
```

### 6.3 应用的部分编译

```bash
./build.sh app/<pkg>
./build.sh external/<pkg>
```

实验记录：

- 软件包名称：
- 源码目录：
- Buildroot package 定义：
- 是否在当前 `.config` 中启用：
- 部分编译输出：
- 是否重新生成 rootfs：
- 板端安装路径：
- 运行依赖：

推荐完整流程：

```bash
./build.sh app/<pkg>
./build.sh rootfs
./build.sh firmware
./rkflash.sh rootfs
./rkflash.sh rd
```

### 6.4 阶段验收

- [ ] 能向 rootfs 添加文件或启动服务
- [ ] 能独立编译并下载 rootfs
- [ ] 能区分“软件包编译完成”和“软件包已进入 rootfs”
- [ ] 知道 rootfs 与 userdata 的数据边界

---

## 7. 阶段五：内核配置、驱动与模块

### 7.1 学习目标

- 理解 defconfig、`.config` 和 Kconfig 的关系。
- 区分 built-in 驱动和 loadable module。
- 掌握内核模块编译、部署和验证。

### 7.2 重点知识

- `CONFIG_xxx=y`：编进内核镜像
- `CONFIG_xxx=m`：编译为内核模块
- `CONFIG_xxx` 未设置：不参与编译
- 模块必须和正在运行的 Kernel 版本及配置匹配
- 只执行 `./build.sh modules` 不等于模块已进入设备

### 7.3 实验计划

- [ ] 查看当前内核版本和配置
- [ ] 选择一个简单驱动配置项
- [ ] 分别理解 `y` 和 `m` 的产物差异
- [ ] 执行 `./build.sh modules`
- [ ] 找到生成的 `.ko`
- [ ] 部署模块到板端
- [ ] 执行 `depmod`、`modprobe` 或 `insmod`
- [ ] 使用 `lsmod` 和 `dmesg` 验证

板端验证命令：

```bash
uname -a
uname -r
lsmod
modinfo <module>
modprobe <module>
dmesg | tail -n 100
```

---

## 8. 阶段六：U-Boot 与启动链

### 8.1 启动链

```text
BootROM
  -> Loader/SPL
  -> U-Boot
  -> boot.img（Kernel + DTB）
  -> rootfs
  -> 用户空间服务和应用
```

### 8.2 学习目标

- 理解 BootROM、Loader、U-Boot、Kernel 和 rootfs 的职责。
- 学会查看和修改非关键 U-Boot 配置。
- 学会单独编译、下载和恢复 `uboot` 分区。

### 8.3 实验计划

- [ ] 记录正常启动时的完整串口日志
- [ ] 区分 Loader、U-Boot 和 Kernel 的日志边界
- [ ] 查看 U-Boot 环境变量
- [ ] 理解 bootcmd、bootargs 和 FIT 加载流程
- [ ] 执行 `./build.sh uboot`
- [ ] 备份当前可用固件
- [ ] 单独下载 `uboot.img`
- [ ] 验证启动过程

参考命令：

```bash
./build.sh uboot
ls -lh u-boot/uboot.img rockdev/uboot.img
./rkflash.sh uboot
./rkflash.sh rd
```

### 8.4 风险边界

- 初期不修改 DDR 初始化参数。
- 初期不修改 Loader/SPL。
- 不随意烧写 `parameter.txt`。
- 不在不理解影响时修改启动分区地址。
- 保留可用的完整 `update.img`。
- 掌握 Loader 和 Maskrom 恢复方式后再进行高风险实验。

---

## 9. 阶段七：完整固件打包与恢复

### 9.1 完整构建流程

```bash
./build.sh all
./build.sh firmware
./build.sh updateimg
ls -lh rockdev
```

### 9.2 打包检查清单

- [ ] 板级配置是否正确
- [ ] Kernel DTS 是否正确
- [ ] `boot.img` 是否为最新版本
- [ ] `rootfs.img` 是否为最新版本
- [ ] `uboot.img` 是否正确
- [ ] `recovery.img` 是否存在
- [ ] `parameter.txt` 是否对应当前分区设计
- [ ] `update.img` 的生成时间和大小是否合理
- [ ] 完整固件是否经过一次冷启动验证

### 9.3 故障恢复能力

- [ ] 能进入 Loader 模式
- [ ] 能进入 Maskrom 模式
- [ ] 能识别升级工具中的设备状态
- [ ] 能使用已验证的 `update.img` 恢复开发板
- [ ] 能保存并分析完整串口日志
- [ ] 能区分 U-Boot 故障、Kernel 故障和 rootfs 故障

---

## 10. 常用检查命令

### 10.1 编译环境

```bash
./build.sh -h
./build.sh -h kernel
readlink -f device/rockchip/.BoardConfig.mk
readlink -f device/rockchip/.target_product
```

注意：当前 SDK 的 `./build.sh info` 会重新构建设备树，不是纯只读命令。

### 10.2 镜像检查

```bash
ls -lh rockdev
readlink -f rockdev/boot.img
readlink -f rockdev/rootfs.img
sha256sum rockdev/boot.img
```

### 10.3 板端系统检查

```bash
uname -a
cat /proc/cmdline
cat /proc/device-tree/model
mount
df -h
lsblk
dmesg | less
```

### 10.4 外设检查

```bash
ls /sys/class/gpio
ls /sys/class/leds
ls /dev/ttyS*
ls /dev/i2c-*
ls /dev/spidev*
cat /proc/interrupts
cat /sys/kernel/debug/pinctrl/*/pinmux-pins
```

---

## 11. 学习进度总表

| 阶段 | 内容 | 状态 | 完成日期 | 主要问题 |
|---|---|---|---|---|
| 0 | Docker 全量编译和整体下载 | 已完成 |  |  |
| 1 | 镜像、分区和 Kernel 部分编译 | 未开始 |  |  |
| 2 | 设备树基础和 LED 实验 | 未开始 |  |  |
| 3 | GPIO/UART/I2C/SPI/PWM | 未开始 |  |  |
| 4 | Buildroot、应用和 rootfs | 未开始 |  |  |
| 5 | 内核配置、驱动和模块 | 未开始 |  |  |
| 6 | U-Boot 和启动链 | 未开始 |  |  |
| 7 | 完整打包和故障恢复 | 未开始 |  |  |

---

## 12. 待整理资料

- [ ] 开发板原理图及引脚复用表
- [ ] RK3566 Technical Reference Manual
- [ ] SDK 自带 Kernel、GPIO、I2C、SPI、PWM 文档
- [ ] Rockchip 分区和固件下载文档
- [ ] 当前开发板进入 Loader/Maskrom 模式的方法
- [ ] 当前可用完整固件的版本、哈希和保存位置
- [ ] 常见编译错误及解决方法
- [ ] 常见启动错误及串口日志特征

