# SylixOS / sydev 开发助手

`sydev`（v0.4.14）是 SylixOS 开发环境的 CLI 入口，覆盖 workspace 初始化、工程管理、编译、上传和模板复用。优先通过 `sydev` 命令和 workspace 文件完成任务；只有 `sydev` 覆盖不到时才回退到 `rl-*` 或其他工具，并说明原因。

## 决策流程

收到请求后按以下顺序判断：

1. **芯片/板卡名** → 参考「平台映射」推断平台，推断不确定时追问
2. **搭建环境** → 生成 JSON 配置，走 `sydev init --config` 或分步命令
3. **编译/上传** → 确认在 workspace 根目录，直接执行
4. **编辑 Makefile / .reproject** → 参考「Workspace 文件」
5. **调试/telnet** → 参考「调试与验证」
6. **生成配置 / 模板** → 参考「JSON 配置参考」
7. **SylixOS 系统问题** → 参考「官方文档导航」

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
- **冲突时以代码为准**：文档和 CLI 实现冲突时，以当前 CLI 实现为准
- **失败看退出码和 stderr**：`sydev` 命令失败时检查退出码和 stderr 输出；编译失败会自动提取错误行摘要

---

## 搭建环境

### 路径一：一键初始化（推荐）

```bash
sydev init --config full-config.json
```

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

---

## JSON 配置参考

`sydev` 有两套 JSON 字段命名约定，不要混用。

### 单命令 `--config` 风格

用于 `workspace init --config`、`project create --config`、`device add --config`：

```json
{
  "cwd": "/path/to/ws",
  "basePath": "/path/to/ws/.realevo/base",
  "version": "lts_3.6.5",
  "platforms": ["ARM64_GENERIC"],
  "os": "sylixos",
  "debugLevel": "release",
  "createBase": true,
  "build": false,
  "arm64PageShift": 12,
  "baseComponents": ["libsylixos", "openssl"]
}
```

### full 配置风格

用于 `sydev init --config`、`template import`、`template apply`：

```json
{
  "schemaVersion": 1,
  "workspace": {
    "cwd": "/path/to/ws",
    "basePath": "/path/to/ws/.realevo/base",
    "platform": ["ARM64_GENERIC"],
    "version": "lts_3.6.5",
    "createbase": true,
    "build": false,
    "debugLevel": "release",
    "os": "sylixos",
    "arm64PageShift": 12,
    "baseComponents": ["libsylixos", "openssl"]
  },
  "projects": [
    { "name": "my-app", "template": "app", "type": "cmake", "debugLevel": "release", "makeTool": "make" }
  ],
  "devices": [
    { "name": "board1", "ip": "192.168.1.100", "platform": ["ARM64_GENERIC"], "username": "root", "password": "root" }
  ]
}
```

### 字段差异对照

| 单命令 `--config` | full / template | 说明 |
| --- | --- | --- |
| `platforms` | `workspace.platform` | 单复数不同 |
| `createBase` | `workspace.createbase` | 大小写不同 |
| `cwd` | `workspace.cwd` | full 配置嵌套在 `workspace` 下 |
| `basePath` | `workspace.basePath` | 同上 |

### project 配置

```json
{ "mode": "import", "name": "bsp-rk3568", "source": "https://git.example.com/repo.git", "branch": "main", "makeTool": "make" }
```

```json
{ "mode": "create", "name": "my-app", "template": "app", "type": "cmake", "debugLevel": "release", "makeTool": "make" }
```

模板可选值：`app`/`lib`/`common`/`ko`/`python_native_lib`/`uorb_pubsub`/`vsoa_pubsub`/`fast_dds_pubsub`

### device 配置

```json
{ "name": "board1", "ip": "192.168.1.100", "platforms": ["ARM64_GENERIC"], "username": "root", "password": "root", "ssh": 22, "telnet": 23, "ftp": 21, "gdb": 1234 }
```

---

## 命令详解

### workspace

```bash
sydev workspace init --config workspace.json
sydev workspace init --cwd /ws --base-path /ws/.realevo/base --version lts_3.6.5 --platforms ARM64_GENERIC --create-base --build
sydev workspace status
```

关键选项：`--version` (`default`/`ecs_3.6.5`/`lts_3.6.5`/`lts_3.6.5_compiled`/`research`/`custom`)、`--arm64-page-shift` (`12`/`14`/`16`)、`--base-components`（逗号分隔，`libsylixos` 必选）。

`version=custom` 需要 `--custom-repo` + `--custom-branch`；`version=research` 需要 `--research-branch`。

### project

```bash
sydev project create --config project.json
sydev project create --mode create --name my-app --template app --type cmake --make-tool make
sydev project create --mode import --name bsp --source <url> --branch main --make-tool make
sydev project list
```

### device

```bash
sydev device add --config device.json
sydev device add --name board1 --ip 192.168.1.100 --platforms ARM64_GENERIC --username root
sydev device list
```

### build / clean / rebuild

```bash
sydev build <project> --quiet -- -j$(nproc)     # 编译
sydev build __demo                                # 执行构建模板
sydev build init                                  # 增量更新 Makefile
sydev build init --default                        # 全量重生成 Makefile
sydev clean <project>                             # 清理
sydev rebuild <project> -- -j$(nproc)            # 重建
```

- `build` 先按工程名找，再按 `__` 模板名找
- 执行前自动同步工程 `config.mk` 中的 `SYLIXOS_BASE_PATH`
- `base` + `-j<N>` 时自动修补 jobserver 支持

### upload

```bash
sydev upload <project> --device board1 --quiet
sydev upload libcpu,libnet --device board1
sydev upload --all --device board1
```

- 单工程可不传 `--device`（从 `.reproject` 读取），多工程/`--all` 必须传
- 上传通过 FTP，路径从 `.reproject` 解析

### template

```bash
sydev template create                             # 交互式创建
sydev template list [--type full]                 # 列表
sydev template show <id>                          # 详情
sydev template apply <source> --cwd /ws --base-path /base -y
sydev template export -o config.json              # 导出当前 workspace
sydev template import config.json                 # 导入
sydev template delete <id>
```

### init

```bash
sydev init --config full-config.json
```

一次性初始化 workspace + projects + devices。使用 full 配置风格。

---

## Workspace 文件结构

```
<workspace>/
├── .realevo/
│   ├── config.json          # base 路径、平台、设备
│   ├── workspace.json       # workspace 初始化参数
│   ├── devicelist.json      # 设备列表（rl-device 写入）
│   └── projects.json
├── .sydev/
│   └── Makefile             # sydev build init 生成
├── <project>/
│   ├── .project + Makefile  # 项目识别标志
│   ├── config.mk            # SYLIXOS_BASE_PATH 自动同步
│   └── .reproject           # 上传配置
```

- workspace 判定：目录下存在 `.realevo/`
- 项目判定：一级子目录同时包含 `.project` 和 `Makefile`
- 设备加载优先级：`devicelist.json` > `config.json`

### .reproject 格式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<reproject>
  <DeviceSetting DevName="board1"/>
  <upload>
    <file local="$(WORKSPACE_libcpu)/$(Output)/lib/libcpu.so"
          remote="/system/lib/libcpu.so"/>
  </upload>
</reproject>
```

宏替换规则：
- `$(WORKSPACE_<project>)` → 项目绝对路径（`-` 换成 `_`）
- `$(Output)` → `Debug` 或 `Release`（取决于 `config.mk` 的 `DEBUG_LEVEL`）
- 路径含 `libsylixos` 时自动替换为 base 路径

### .sydev/Makefile 可编辑区域

| 区域 | 可否手改 |
|------|---------|
| 头部注释、`export WORKSPACE_*`、`.PHONY` | 否（自动维护） |
| 工程 block 的 build/clean/rebuild | 谨慎（增量更新保留，`--default` 覆盖） |
| `cp-<name>` | 是（产物复制路径） |
| `__` 用户模板区域 | 是（多目标编译模板） |

---

## 调试与验证

### 最小调试闭环

```bash
sydev build <project> --quiet -- -j$(nproc)
sydev upload <project> --device <device> --quiet
telnet <ip> <telnet-port>
```

### 设备信息解析

优先级：`.realevo/devicelist.json` > `.realevo/config.json`

| 含义 | devicelist.json | config.json |
| --- | --- | --- |
| 用户名 | `user` | `username` |
| 平台 | `platform`（字符串） | `platforms`（数组） |
| 端口 | 字符串 | 数字 |

默认回退值：telnet=`23`，username=`root`，password=`root`

### 登录后验证

- 确认上传目录和文件存在：`ls`、`cat`
- 进程/网络：`ps`、`ifconfig`、`ping`
- 模块：`insmod`、`rmmod`、`lsmod`

### 构造临时测试工程

```bash
# 用户态应用
sydev project create --mode create --name dbg-app --template app --type cmake --debug-level debug --make-tool make

# 内核模块
sydev project create --mode create --name dbg-ko --template ko --type cmake --debug-level debug --make-tool make
```

- 验证用户态逻辑/二进制/共享库 → `app`
- 验证内核态/模块装载 → `ko`

---

## 平台映射

用户只给芯片名时的保守推断规则：

| 用户说法 | 默认平台 |
| --- | --- |
| `rk3568`/`rk3588`/`rk3399` | `ARM64_GENERIC` |
| `cortex-a53` | `ARM64_A53` |
| `cortex-a55` | `ARM64_A55` |
| `cortex-a72` | `ARM64_A72` |
| `imx6ull` / `allwinner a40i` | `ARM_A7` |
| `x86_64`/`amd64` | `X86_64` |
| `risc-v64`/`jh7110`/`c910` | `RISCV_GC64` |
| `loongarch64`/`3a5000`/`3a6000` | `LOONGARCH64` |
| `飞腾 ft2000/d2000` | `ARM64_GENERIC` |

需要确认的情况：
- "龙芯" → LoongArch 还是旧 MIPS64？
- "RISC-V" → 32 位还是 64 位？
- 只给厂商名 → 至少确认芯片型号或 CPU 架构

支持的平台族：ARM32（A5/A7/A8/A9/A15/V7A）、ARM64（A53/A55/A57/A72/GENERIC）、x86（PENTIUM/X86_64）、RISC-V（GC32/GC64）、LoongArch（LOONGARCH64）、MIPS、PPC、SPARC、CSKY、SW6B 等。

---

## SylixOS 官方文档导航

| 主题 | 入口 | 适用场景 |
|------|------|---------|
| Shell | https://docs.acoinfo.com/sylixos/shell/ | Shell 命令、文件系统、网络配置、进程管理 |
| App | https://docs.acoinfo.com/sylixos/app/ | 用户态应用开发、IPC、动态装载 |
| Drv | https://docs.acoinfo.com/sylixos/drv/ | 内核态驱动、BSP、设备树、模块 |
| ECS | https://docs.acoinfo.com/sylixos/ecs/ | 容器命令、配置、镜像、日志 |

选路规则：Shell 命令 → `shell`；用户态应用 → `app`；驱动/模块 → `drv`；容器 → `ecs`。跨域问题按主问题选入口。
