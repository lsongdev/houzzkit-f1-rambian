# Houzzkit F1 V2 硬件资源记录

2026-11-12 实测一台设备。原厂系统报告型号 `HOUZZkit F1 RK3568 Board V2`，设备树兼容标识为 `rpdzkj,rp-rk3568-v10`、`rockchip,rk3568`。实测使用 Armbian 26.11.0 Trixie、`6.18.59-ophub` 和 JL-RM01 设备树。

## Linux 中的资源分配

| 资源 | 这台 V2 的观察 |
| --- | --- |
| Zigbee | SoC 的 OHCI USB 控制器 `fd8c0000.usb` 直接枚举 CH340（`1a86:7523`）。驱动为 `ch341`，串口是 `/dev/ttyUSB0`，稳定路径为 `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0`。 |
| UART2 | `serial@fe660000`，对应 `/dev/ttyS2`，启动及登录控制台使用 1500000 波特。 |
| UART3 | `serial@fe670000`，启用时对应 `/dev/ttyS3`。当前设备树选用 M1，TX 为 `GPIO3_B7`，RX 为 `GPIO3_C0`。 |
| 双以太网 | `eth0` 对应 `ethernet@fe010000`，其 PHY 复位为 `GPIO3_B0`。 `eth1` 对应 `ethernet@fe2a0000`，其 PHY 复位为 `GPIO3_B7`。两根网线接上后均实测 1 Gbps 全双工。 |
| Wi-Fi | PCIe Realtek RTL8822CE，驱动 `rtw88_8822ce`。2.4 GHz 和 5 GHz 扫描均通过。 |
| 蓝牙 | USB Realtek `0bda:c822`，位于 `fd840000.usb`，驱动 `btusb`，`hci0` 的 RTL8822CU 固件已加载。BLE 扫描和配对未实测。 |
| 存储 | 32 GB eMMC（`/dev/mmcblk0`）承载系统。另有 128 GB M.2 NVMe（`/dev/nvme0n1`），数据分区已挂载。 |
| 显示、音频和 RTC | HDMI DRM 接口、音频和 RTC 设备均已枚举。HDMI 当时未接显示器。 |

上述地址和 Linux 接口对应当前镜像。外壳接口位置参见 [原厂快速上手](https://yinbaiyuan.github.io/started/)，主线移植的其他外设记录参见 [6.18 适配说明](../dts/mainline/README.md)。

## Zigbee 与 UART3 的实测

- 停掉 Zigbee 网关释放串口后，按 `115200 8N1` 打开 CH340，依次设 `DTR=false`、`RTS=false`，等待 4 秒，发送 ZNP `SYS_VERSION`。返回版本 `2.7.1`，修订号 `20230507`。网关使用同一串口正常运行。
- 原主线 DTB 同时给 UART3 TX 和 `eth1` 的 PHY 复位分配了 `GPIO3_B7`。内核引脚状态显示 UART3 的复用以及网口驱动对同一 GPIO 的申请。临时在 DTB 中禁用 `eth1` 后，UART3 独占 M1 引脚。
- 隔离状态下，UART3 在 115200 波特的 16550 内部回环逐字节收回了发送的 256 字节。退出内部回环后，ZNP 查询使 TX 计数增加 5 字节，RX 计数未变。此结果确认 UART 控制器和驱动能收发，板外 TX/RX 走线和对端用途仍未确认。
- 测试后恢复双网口，并用单独的测试 DTB 禁用 UART3。两个网口的千兆链路和 CH340 Zigbee 网关均已复测。

## 原厂资料和 8 针座

- 原厂拆机近照中，HDMI 一侧的 Zigbee 小板上可以看到 RF-star 模块和 `CC2652P` 标记。照片标出了该模块的天线位置，照片中各排孔或连接器的信号没有标注。这台 V2 拆机时未找到社区资料所说的 8 针 Zigbee 座。 [原厂硬件说明](https://yinbaiyuan.github.io/firmware/hardware.html) · [原厂 Zigbee 模块近照](https://yinbaiyuan.github.io/assets/image/houzzkit/houzzkit_f1_ipex_to_sma_connect_zigbee.jpeg) · [社区 JL-RM01 说明](https://github.com/mozzie1121/houzzkit-f1-rambian#设备信息)
- 原厂公开的 `rm01_rk3568_v10_lp4xd200p132sd6` 原理图，PDF 第 19 页把 `USB2_HOST3_DP/DM` 引到 `J64`，第 20 页标明 `GPIO3_B7` 可复用为 UART3 TX，第 35 页把该脚经 `R6743` 接入 PHY 复位电路。这份图能解释两个资源的分配，仍缺少本机 V2 的实物对照来确认 J64 与 CH340 的接法，以及 UART3 是否引出。 [原厂开源硬件资料](https://github.com/yinbaiyuan/houzzkit-f1-opensource/tree/main/houzzkit-f1-hardware) · [RK3568 USB 控制器设备树](https://kernel.googlesource.com/pub/scm/linux/kernel/git/thierry.reding/linux-pwm/+/for-next/arch/arm64/boot/dts/rockchip/rk356x.dtsi)
