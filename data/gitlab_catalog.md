# GitLab Repository Catalog

- GitLab Web: http://10.7.100.21:8000/
- SSH Clone: `ssh://git@10.7.100.21:16783/<group>/<repo>.git`
- Total: 2128 repositories in 143 groups
- Updated: 2026-03-03
- Full data: `data/gitlab_repos.json` (machine-readable, all 2128 repos)

## Core Groups

| Group | Repos | Description |
|-------|-------|-------------|
| `Test-Toolset` | 40 | Automated testing tools and CI scripts |
| `aerospace` | 9 | Aerospace/aviation adaptations |
| `bsp_sdk` | 130 | BSP SDK releases, manifests, test drivers |
| `cloudnative` | 40 | CI/CD, Jenkins, Armory, container infra |
| `custom` | 110 | Custom/OEM project repos |
| `document` | 33 | Online documentation repos |
| `driver_library` | 58 | Shared driver libraries (NIC, GPU, compat layers, CAN, USB) |
| `graphic` | 27 | GPU drivers, display frameworks |
| `industrial-control` | 18 | Industrial control adaptation projects |
| `industry` | 46 | Industrial control products (IGC, PLC) |
| `matrix653` | 19 | ARINC 653 partitioning OS |
| `matrix653_bsp` | 10 | Matrix653 board support packages |
| `middleware` | 286 | Third-party library and middleware ports |
| `ms-rtos` | 16 | MS-RTOS microcontroller kernel |
| `msrtos_bsp` | 29 | MS-RTOS BSP packages |
| `multimedia` | 26 | Audio, video, camera, display middleware |
| `net_test` | 38 | Network testing, armory packages |
| `openeuler` | 4 | openEuler Linux adaptations |
| `pci` | 5 | PCI/PCIe drivers and NIC cards |
| `project` | 385 | Customer project deliverables |
| `qt` | 22 | Qt 5.12 framework and QtCreator plugins |
| `quickvisor` | 29 | QuickVisor hypervisor, BSP, guest VM tools |
| `realevo-ide` | 67 | RealEvo IDE, VS Code plugin, Stream web |
| `solutions` | 11 | Solution packages and demos |
| `sylixos` | 94 | SylixOS kernel, ECS container, middleware core |
| `sylixos_bsp` | 144 | BSP packages (ARM, ARM64, x86, MIPS, SPARC, RISC-V, LoongArch) |
| `usb` | 42 | USB host/device drivers and tools |
| `wireless` | 32 | WiFi, Bluetooth, cellular drivers |
| *(personal repos)* | 358 | 115 users' individual repos |

## SylixOS BSP by Architecture

**ARM64** (13):
- `bspft2004_3rd` — Phytium FT2000-4 (ARMv8 64bit 4Core SMP) uboot & uefi code.
- `bspftd2000v` — Phytium D2000V (ARMv8 64bit SMP) packet BSP 
- `bspfte2000` — 目前 Phytium FT2000-4， FTD2000，E2000，D3000 都合并到 bspPhytium 仓库统
- `bspfts2500` — Phytium FTS2500 (ARMv8 64bit 64Core SMP) packet BSP

- `bspftxxxx` — Phytium FT1500A/FT2000 (ARMv8 64bit SMP) packet BSP

- `bsprk3328` — Rockchip 3328 BSP 
- `bsprk3399`
- `bsprk3562` — RK3562 标准 BSP
- `bsprk3568` — RK3568 BSP
- `bsprk3576` — RK3576 标准 BSP
- `bsprk3588` — Rockchip RK3588 Quad-core Cortex-A76 and Quad-core Cortex-A5
- `bspsemidrived9` — SemiDrive D9 (ARMv8 64bit SMP) packet BSP.
- `bspzcu102` — BSP of qemu-aarch64 zcu102 board.

**ARM** (50):
- `bspaarch64virt`
- `bspallwinnerh3` — Allwinner H3 Quad(Cortex-A7 SMP) packet BSP

- `bspallwinnerh6` — Allwinner H6 Quad(Cortex-A53 SMP) packet BSP

- `bspallwinnerh6_3rd`
- `bspallwinnert3` — Allwinner T3 Quad(Cortex-A7 SMP) packet BSP

- `bspallwinnert536` — 全志 T536 BSP 仓库
- `bspallwinnert7` — Allwinner T7 Hexa(Cortex-A7 SMP) packet BSP
- `bspam335x` — TI-AM335x(ARM Cortex-A8) packet BSP

- `bspam4378` — TI AM4378 (ARM Cortex-A9 SMP) packet BSP

- `bspam57xx` — TI AM57xx (ARM Cortex-A15 SMP) packet BSP

- `bspascend310b` — 华为自研芯片
- `bspasm9260` — AScale ASM9260 (ARM920T) packet BSP

- `bspat91rm9200` — ATMEL AT91RM9200 (ARM920T) packet BSP

- `bspat91sam9x25` — Atmel at91sam9x25 (ARM ARM926) packet BSP

- `bspatsama5d2` — Atmel atsama5d2 (ARM Cortex-A5) packet BSP

- `bspcv186ah` — 算能 cv186ah
- `bspcyclonev`
- `bspdks001` — DKS001 (ARM Cortex-A9 SMP) packet BSP
- `bspfmql`
- `bspfmql100tai` — FMSH ARM MPSOC(Cortex-A53 SMP + FPGA) packet BSP
- `bspfmsh` — 复旦微（FMSH）系列代码整合仓库
- `bspft2004` — 目前 Phytium FT2000-4， FTD2000，E2000，D3000 都合并到 bspPhytium 仓库统
- `bspftd2000` — 目前 D2000 代码已经合并到 FT2004 仓库里，新项目或者测试请不要使用此仓库的代码！！！
- `bspftm7004v` — 科大7004v芯片arm端bsp
- `bspfts5000` — 飞腾 S5000C SylixOS BSP （浪潮服务器板卡）
- `bsphksa9201` — 631s自研芯片HKSA9201的bsp
- `bspimx283` — EasyARM-i.MX283(ARM926EJ-S) packet BSP

- `bspimx6` — Freescale i.MX6 Quad(Cortex-A9 SMP) packet BSP

- `bspimx6ul` — Freescale i.MX6UL(Cortex-A7 SMP) packet BSP

- `bspimx7d` — NXP i.MX7 Dual(Cortex-A7 SMP) packet BSP
- `bspimx8m` — NXP i.MX8M Mini (Cortex-A53 4Core SMP) packet BSP
- `bspjsimx6q`
- `bspjyxz7` — 保利YX4S045适配
- `bspls102xa` — NXP LS102XA Dual(Cortex-A7 SMP) packet BSP
- `bspls1043a`
- `bspls1046`
- `bsplsocam0201` — Xi'an Microelectronic Technology Institute LSoCAM0201 (ARM C
- `bsplxka200` — Lynxi Ka200 Octa(Cortex-A53 SMP) packet BSP
- `bspmini2440` — Mini2440(ARM920T) packet BSP

- `bspnanopim3` — nanopim3 bsp
- `bspnuc970` — Nuvoton 970 (ARM ARM920T) packet BSP

- `bspphytium` — 飞腾处理器 BSP
- `bsprockchip` — 作为 rk3562/rk3568/rk3576/rk3588 等瑞芯微系列处理器的总仓库，整合所有外设功能。 
- `bspsemidrivex9`
- `bspsm15d325` — [国微]SM15D325TMAL
- `bspsmart210` — Smart210(ARM Cortex-A8) packet BSP

- `bspsq9728`
- `bspyulong810a` — Orbita YuLong 810A (ARM Cortex-A9 SMP ) packet BSP
- `bspzynq7000` — Xilinx ARM Soc(Cortex-A9 SMP + FPGA) packet BSP

- `bspzynqmpsoc` — Xilinx ARM Soc(Cortex-A53 SMP + FPGA) packet BSP


**x86** (1):
- `bspx86` — x86 packet BSP


**MIPS** (13):
- `bsphr2` — HUA RUI 2 HAO(MIPS64) packet BSP 
- `bsphr3` — HUA RUI 3 HAO(MIPS64) packet BSP
- `bspjz4780` — Ingenic JZ4780(MIPS32 SMP) packet BSP

- `bspjzx1000` — Ingenic X1000 (MIPS32 AMP) BSP
- `bspjzx1600` — Ingenic X1600 (MIPS32 AMP) BSP
- `bspjzx2000` — Ingenic X2000/M300 (MIPS32 SMP) BSP
- `bspls1e300` — Loongson-1E300(MIPS64 SMP) packet BSP

- `bspls1x` — Loongson-1X(MIPS32) packet BSP

- `bspls2h` — Loongson-2H(MIPS32) packet BSP

- `bspls2k` — Loongson-2K(MIPS64 SMP) packet BSP

- `bspls3x` — Loongson-3X(MIPS64 SMP) packet BSP

- `bspmipsr4k` — MIPS R4K(MIPS32/MIPS64 qemu) packet BSP

- `libls2xdrv` — Loongson-2x(MIPS32) driver library


**SPARC** (8):
- `bspbm3803` — BM3803 (772) SPARCv8 packet BSP

- `bspbm3823` — 772S BM3823 SPARCv8 packet BSP
- `bspbm3883` — 772S BM3883 Octa(SPARCv8 SMP) packet BSP
- `bsplc3233` — LCSOC3233 (771) SPARCv8 packet BSP

- `bspleon3` — Cobham Gaisler LEON3(SPARCv8 qemu) packet BSP

- `bsps698pm` — Myorbita S698PM Quad(SPARCv8 SMP) packet BSP

- `bspyulong810a` — Orbita YuLong 810A ( SPARCv8 SMP ) packet BSP
- `sparcl1boot` — Level 1 bootloader for LCSOC3233 SPARCv8 CPU


**RISC-V** (6):
- `bspallwinnerd1`
- `bspandesv5` — Andes V5 packet BSP

- `bspcreativek210` — Canaan-creative k210 packet BSP

- `bspnuclein900`
- `bspsifiveu500` — SiFive U500 packet BSP

- `bspsm90d325tnail` — 国微 sm90d325tnail_SylixOS_SDK_64bit 项目 BSP 工程源码

**LoongArch** (7):
- `bspls2k0300`
- `bspls2k0500`
- `bspls2k1000la`
- `bspls2k1500` — 龙芯 2k1500 
- `bspls2k2000`
- `bspls2k3000` — 龙芯 2k3000 bsp 仓库
- `bspls3a5000`

**C-SKY** (4):
- `bspfuxih` — FUXI-H 系列 BSP 工程
- `bspjw037` — C-SKY JW037(C-SKY CK810MF) packet BSP

- `bspsc8925` — C-SKY SC8925(C-SKY CK810MF) packet BSP

- `bspxt860` — C-SKY 玄铁860(C-SKY CK860MP) packet BSP

**PowerPC** (16):
- `bspccp908t` — CCP908T BSP
- `bspmpc5125` — Freescale MPC5125(E300) packet BSP

- `bspmpc750` — IBM RAD750(qemu) packet BSP

- `bspmpc750_804`
- `bspmpc750_demo` — bspmpc750 demo board bsp
- `bspmpc8309` — NXP MPC8309 (PowerPC E300) packet BSP

- `bspmpc8313` — Freescale MPC8313(E300) packet BSP

- `bspmpc8323` — NXP MPC8323 (PowerPC E300) packet BSP

- `bspmpc8377` — Freescale MPC8377(E300) packet BSP

- `bspp1022`
- `bspp2010` — NXP P2010 (PowerPC E500MC) packet BSP

- `bspp4080` — Freescale P4080 8 Core(E500MC SMP) packet BSP

- `bspppc460` — IBM PowerPC 460 packet BSP

- `bspt1022` — NXP T1022 (PowerPC E5500) packet BSP

- `bspt1042` — NXP T1040 (PowerPC E5500) packet BSP

- `bspt2080` — NXP T2080

**Lite (MCU)** (12):
- `bspcc3220sf` — TI cc3220sf (ARM Cortex-M4) packet BSP

- `bspgd32f470`
- `bspgd32f4xx` — gd32f4xx 支持多板卡lite bsp仓库
- `bspimxrt1050` — NXP i.MX-RT1050(Cortex-M7) packet BSP

- `bsplpc54xxx` — NXP LPC54000 Series (Cortex-M4) packet BSP

- `bspsc5654a` — C-SKY sc5654a (C-SKY CK803) packet BSP.

- `bspscm6xx` — State Grid SCM6xx (Cortex-M4) packet BSP

- `bspsemidrived9_r5` — SemiDrive D9 Pro 平台的 Cortex-R5 的 SylixOS BSP
- `bspsmartfusion2` — Microsemi-smartfusion2 (Cortex-M3) packet BSP

- `bspstm32f767` — ST STM32F767(Cortex-M7) packet BSP

- `bspstm32h7xx` — ST STM32H7xx (ARM Cortex-M7) packet BSP.

- `bsptms570lc43xx` — TI TMS570lc43xx(Cortex-R5) packet BSP


**Other** (14):
- `bspsw831`
- `ft1500aboot`
- `grub4dos` — grub4dos
- `imxrt1050boot` — i.MX RT1050 bootloader
- `lc3233boot` — Bootloader for LCSOC3233 SPARCv8 CPU

- `ppc750boot_804` — ppc750boot_804
- `x86_ap_boot` — x86 application process boot
- `bsp6678` — TI c6678 (C66 Mulit DSP AMP) packet BSP

- `bsp66ak2x-arm`
- `bsp66ak2x-dsp`
- `bsp_ftm6678` — FT M6678 (C66 Mulit DSP AMP) packet BSP

- `bspftm6603`
- `bspftm66ak`
- `bspftm8024v` — FT M8024V (C66 Mulit DSP AMP) packet BSP

## Searching

To search the full repo list programmatically:
```bash
# Search by keyword
python3 -c "import json; data=json.load(open('data/gitlab_repos.json')); [print(r['path'],r['ssh']) for g in data['groups'].values() for r in g if 'rk3568' in r['path']]"
```
