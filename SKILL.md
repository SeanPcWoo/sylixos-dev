---
name: sylixos-dev
description: >
  SylixOS / sydev 开发与调试助手。当用户要为 SylixOS 开发/编写/写程序（app、驱动、内核模块等）、
  搭建 SylixOS workspace、创建工程、添加设备、编译(build/clean/rebuild)、上传(upload)产物到
  设备、在板子上测试/验证/跑一下、telnet 调试、生成或应用 JSON 配置模板、编辑 .reproject 或
  .sydev/Makefile 时触发。用户只给出芯片名（rk3568、rk3588、飞腾、龙芯等）或平台架构
  （ARM64、RISC-V、LoongArch）时也触发。Also triggers for SylixOS cross-compilation,
  RTOS workspace setup, sydev CLI usage, embedded device deployment, and any mention of
  RealEvo or rl-workspace/rl-build/rl-project toolchain. 即使用户没有明确提到 sydev，
  只要涉及 SylixOS 相关的程序开发、嵌入式开发环境搭建、编译部署，都应该使用此技能。
  注意：为 SylixOS 开发任何程序都必须先通过 sydev 创建工程，不要手动创建 Makefile 和
  config.mk——sydev project create 会自动生成正确的工程骨架。创建工程默认使用非交互
  命令行参数（--mode create --name ... --template ... --type realevo），而非 --config，
  因为通常没有现成的 project JSON 文件。--type 默认值为 realevo。
---

# SylixOS / sydev 助手

`sydev`（v0.4.14）是 SylixOS 开发环境的 CLI 入口，覆盖 workspace 初始化、工程管理、
编译、上传和模板复用。优先通过 `sydev` 命令和 workspace 文件完成任务；只有 `sydev`
覆盖不到时才回退到 `rl-*` 或其他工具，并说明原因。

## 决策流程

收到请求后按以下顺序判断：

0. **开发/编写程序** → 这是一个完整闭环，按顺序执行：
   1. 创建工程（`sydev project create`）
   2. 编写代码
   3. 编译（`sydev build`）
   4. 上传（`sydev upload`）
   5. **询问用户是否需要上板调试验证**——如果用户确认，则 telnet 登录设备，运行程序，确认输出正确；如果需要在宿主机侧配合测试（如发送报文、构造请求），也应该自行完成
   步骤 1-4 自动执行，步骤 5 询问用户后决定。
1. **芯片/板卡名** → 读 `references/platform-mapping.md` 推断平台，推断不确定时追问
2. **搭建环境** → 生成 JSON 配置，走 `sydev init --config` 或分步命令
3. **编译/上传** → 确认在 workspace 根目录，直接执行
4. **编辑 Makefile / .reproject** → 先读 `references/workspace-files.md`
5. **调试/telnet** → 先读 `references/sylixos-debugging.md`
5.5. **移植第三方库** → 先读 `references/porting-guide.md`，按标准流程执行
6. **生成配置 / 模板** → 先读 `references/config-schema.md`
7. **查命令细节** → 读 `references/sydev-commands.md`
8. **SylixOS 系统问题** → 按 `references/official-doc-routing.md` 定位官方文档

## 核心命令速查

| 场景 | 命令 |
|------|------|
| 初始化 workspace | `sydev workspace init --cwd <path> --base-path <path> --version <ver> --platforms <plat> --os sylixos --debug-level release --create-base` |
| 创建工程 | `sydev project create --mode create --name <name> --template <tpl> --type realevo` |
| 添加设备 | `sydev device add --name <name> --ip <ip> --platforms <plat>` |
| 一键初始化全套 | `sydev init --config full.json`（全套初始化场景较复杂，仍推荐用 JSON） |
| 编译 | `sydev build <name> -- -j$(nproc)` |
| 清理 | `sydev clean <name>` |
| 重建 | `sydev rebuild <name> -- -j$(nproc)` |
| 上传 | `sydev upload <name> --device <dev>` |
| 批量上传 | `sydev upload --all --device <dev>` |
| 导出当前环境 | `sydev template export -o config.json` |
| 应用模板 | `sydev template apply config.json --cwd <ws> --base-path <base> -y` |
| 刷新 Makefile | `sydev build init` |

## 工作原则

- **非交互优先**：自动化场景总是拼完整参数或 `--config`，不依赖交互提示
- **状态读文件**：需要机器可读的 workspace 状态时直接读 JSON 文件，不解析 CLI 彩色输出
- **当前目录 = workspace 根**：`build`/`clean`/`rebuild`/`upload`/`project list` 默认如此
- **多工程上传必须 --device**：多项目或 `--all` 上传时显式传 `--device`
- **复用走 JSON**：用户要"以后能复用"时，优先生成 JSON 配置文件
- **两套模板别混淆**：`sydev template` 管的是配置模板（`~/.sydev/templates/`），`sydev build __xxx` 执行的是 `.sydev/Makefile` 里的构建模板
- **冲突时以代码为准**：技能内容、仓库文档和 CLI 实现冲突时，以当前 CLI 实现为准
- **失败看退出码和 stderr**：`sydev` 命令失败时检查退出码和 stderr 输出；编译失败会自动提取错误行摘要
- **上传后主动询问是否验证**：upload 成功后询问用户是否需要 telnet 登录设备验证。如果用户确认，自动 telnet 登录、运行程序、确认输出符合预期。设备登录信息从 `.realevo/devicelist.json` 获取，缺失时回退 telnet=23、root/root

## 搭建环境

### 路径一：分步执行（默认，直接用命令行参数）

```bash
sydev workspace init \
  --cwd /path/to/ws \
  --base-path /path/to/ws/.realevo/base \
  --version lts_3.6.5 \
  --platforms ARM64_GENERIC \
  --os sylixos \
  --debug-level release \
  --create-base

sydev project create \
  --mode create \
  --name myapp \
  --template app \
  --type realevo

sydev device add \
  --name board1 \
  --ip 192.168.1.100 \
  --platforms ARM64_GENERIC
```

### 路径二：一键初始化（有现成 JSON 配置文件时）

```bash
sydev init --config full-config.json
```

JSON 格式见 `references/config-schema.md`。需要复用时追加 `sydev template import full-config.json`。

### 路径三：模板复用

```bash
sydev template export -o config.json
sydev template apply config.json --cwd <新路径> --base-path <新base> -y
```

## 日常开发循环

```bash
sydev build <project> --quiet -- -j$(nproc)
sydev upload <project> --device <device> --quiet
telnet <ip> <telnet-port>
```

## 调试与验证

upload 后需要上板验证时：

1. 从 `.realevo/devicelist.json`（优先）或 `.realevo/config.json` 获取设备信息
2. 默认回退值：telnet=23, username=root, password=root
3. `telnet <ip> <port>` 登录后做最小验证
4. 需要构造临时测试工程时用 `sydev project create`（app 或 ko 模板）

详细流程见 `references/sylixos-debugging.md`。

## Workspace 关键文件

| 文件 | 用途 |
|------|------|
| `.realevo/config.json` | base 路径、平台、设备 |
| `.realevo/workspace.json` | workspace 初始化参数 |
| `.realevo/devicelist.json` | 设备列表（rl-device 写入） |
| `.sydev/Makefile` | `sydev build init` 生成的构建脚本 |
| `<project>/config.mk` | `SYLIXOS_BASE_PATH` 自动同步 |
| `<project>/.reproject` | 上传配置（设备名 + 文件映射） |

编辑 `.reproject` 或 `.sydev/Makefile` 前先读 `references/workspace-files.md`。

## 第三方库移植

移植第三方开源库到 SylixOS 时，先读 `references/porting-guide.md`。标准流程：

1. **sydev 创建工程** → `sydev project create` + `git init`
2. **导入源码** → 拷贝到工程目录，提交原始代码
3. **条件编译修改** → `#ifdef SYLIXOS / #else / #endif` 保留原实现
4. **兼容头文件** → 创建 `xxx_sylixos_compat.h` 处理宏冲突（optarg、SCHED_OTHER 等）
5. **构建验证** → 配置 `.mk` 文件，`sydev build` 编译通过
6. **文档记录** → 创建 `PORTING.md` 记录所有改动

推荐移植基础：FreeBSD > Linux > POSIX（SylixOS 与 FreeBSD API 更接近）。

## 参考资料索引

| 文件 | 何时读取 |
|------|---------|
| `references/sydev-commands.md` | 需要精确命令选项、参数、行为边界 |
| `references/config-schema.md` | 生成或校验 JSON 配置文件 |
| `references/workspace-files.md` | 编辑 .reproject、.sydev/Makefile、理解文件布局 |
| `references/platform-mapping.md` | 用户只给芯片名或架构名，需要推断平台 |
| `references/sylixos-debugging.md` | upload 后调试、telnet、Shell 命令、构造测试工程 |
| `references/official-doc-routing.md` | SylixOS 官方文档（shell/app/drv/ecs）入口选路 |
| `references/porting-guide.md` | 移植第三方库到 SylixOS 的方法论和 API 对照表 |
