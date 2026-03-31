---
name: sylixos-dev
description: >
  SylixOS / sydev 开发与调试助手。当用户要搭建 SylixOS workspace、创建工程、添加设备、
  编译(build/clean/rebuild)、上传(upload)产物到设备、telnet 调试、生成或应用 JSON 配置
  模板、编辑 .reproject 或 .sydev/Makefile 时触发。用户只给出芯片名（rk3568、rk3588、
  飞腾、龙芯等）或平台架构（ARM64、RISC-V、LoongArch）时也触发。
---

# SylixOS / sydev 助手

`sydev`（v0.4.14）是 SylixOS 开发环境的 CLI 入口，覆盖 workspace 初始化、工程管理、
编译、上传和模板复用。优先通过 `sydev` 命令和 workspace 文件完成任务；只有 `sydev`
覆盖不到时才回退到 `rl-*` 或其他工具，并说明原因。

## 决策流程

收到请求后按以下顺序判断：

1. **芯片/板卡名** → 读 `references/platform-mapping.md` 推断平台，推断不确定时追问
2. **搭建环境** → 生成 JSON 配置，走 `sydev init --config` 或分步命令
3. **编译/上传** → 确认在 workspace 根目录，直接执行
4. **编辑 Makefile / .reproject** → 先读 `references/workspace-files.md`
5. **调试/telnet** → 先读 `references/sylixos-debugging.md`
6. **生成配置 / 模板** → 先读 `references/config-schema.md`
7. **查命令细节** → 读 `references/sydev-commands.md`
8. **SylixOS 系统问题** → 按 `references/official-doc-routing.md` 定位官方文档

## 核心命令速查

| 场景 | 命令 |
|------|------|
| 初始化 workspace | `sydev workspace init --config ws.json` |
| 创建工程 | `sydev project create --config proj.json` |
| 添加设备 | `sydev device add --config dev.json` |
| 一键初始化全套 | `sydev init --config full.json` |
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

## 搭建环境

### 路径一：一键初始化（推荐）

生成 `full-config.json` 后一条命令完成 workspace + 工程 + 设备：

```bash
sydev init --config full-config.json
```

JSON 格式见 `references/config-schema.md`。需要复用时追加 `sydev template import full-config.json`。

### 路径二：分步执行

```bash
sydev workspace init --config workspace.json
sydev project create --config project.json
sydev device add --config device.json
```

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

## 参考资料索引

| 文件 | 何时读取 |
|------|---------|
| `references/sydev-commands.md` | 需要精确命令选项、参数、行为边界 |
| `references/config-schema.md` | 生成或校验 JSON 配置文件 |
| `references/workspace-files.md` | 编辑 .reproject、.sydev/Makefile、理解文件布局 |
| `references/platform-mapping.md` | 用户只给芯片名或架构名，需要推断平台 |
| `references/sylixos-debugging.md` | upload 后调试、telnet、Shell 命令、构造测试工程 |
| `references/official-doc-routing.md` | SylixOS 官方文档（shell/app/drv/ecs）入口选路 |
