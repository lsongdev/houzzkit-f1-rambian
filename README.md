# Houzzkit F1 Armbian（rambian）

基于 [mozzie1121/amlogic-s9xxx-armbian](https://github.com/mozzie1121/amlogic-s9xxx-armbian)
（ophub 构建体系）适配 RK3568-JL-RM01（Houzzkit F1）的 Armbian 镜像项目，走 GitHub
Actions 云打包路线。

本仓库是**轻量 overlay 层**：不复制整个上游构建系统，云构建时先 checkout 上游，
再叠加 `overlay/` 里的设备文件，然后调用上游 action 用 `-b houzzkit-f1` 构建。

## 设备信息

| 项目 | 值 |
| --- | --- |
| SoC | Rockchip RK3568（3568A） |
| 型号 | Houzzkit F1 / SZZN RK3568-JL-RM01 |
| 内存/eMMC | 4GB LPDDR4X / 三星 32GB eMMC |
| 调试串口 | UART2（`serial@fe660000`，1500000） |
| Zigbee | 一台 F1 V2 实测经 CH340 USB 转串口（`/dev/ttyUSB0`），见 [V2 硬件资源记录](docs/F1-V2-HARDWARE.md) |
| 现有 bootchain | idbloader @ LBA 0x40 + U-Boot FIT @ LBA 0x4000（no-OPTEE） |
| Recovery | 按住 Recovery 上电进 RockUSB LOADER |

## 文档

- [适配方案](docs/ADAPTATION-PLAN.md)
- [云构建工作流](.github/workflows/build-armbian-houzzkit-f1.yml)
- [设备 boot 配置草稿](overlay/different-files/houzzkit-f1/bootfs/armbianEnv.txt)
- [设备注册行](overlay/model_database.row)
- [overlay 应用脚本](scripts/apply-overlay.sh)

## 目录

```text
overlay/
  different-files/houzzkit-f1/   # 设备 bootfs/rootfs 定制
  platform-files/                # dtb 等平台文件（后续放入 rk3568-jl-rm01.dtb）
  common-files/                  # 通用文件覆盖（可选）
  model_database.row             # model_database.conf 新增行
scripts/apply-overlay.sh         # 把 overlay 合并进上游 checkout
docs/ADAPTATION-PLAN.md          # 适配方案与里程碑
```
