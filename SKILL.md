---
name: sylixos-dev
description: >
  SylixOS / RealEvo 开发助手。当用户要搭建或复用 SylixOS workspace、生成 workspace/project/device/full
  JSON 配置或模板、初始化或导入工程、生成或修改 .sydev/Makefile、添加或管理设备、编译/clean/rebuild、
  上传产物、编辑 .reproject，或只提供芯片/平台名（如 rk3568、rk3588、x86、RISC-V、LoongArch、飞腾、龙芯）
  时触发。稳定入口是 sydev CLI。
---

# SylixOS / RealEvo Assistant

把 `sydev` 当成稳定入口，优先走 CLI 和 JSON 配置文件，不要依赖内部 TypeScript API 或交互式提示。

## 工作原则

- 优先非交互模式：完整参数、`--config`、`template apply ... -y`、`init --config`
- 执行动作时依赖退出码；读取状态时直接读 workspace 文件
- `build`、`clean`、`rebuild`、`upload`、`project list`、`template export` 默认要求当前目录就是 workspace 根目录
- 多工程上传或 `--all` 上传时，总是显式传 `--device`
- 用户要求“以后复用”时，优先生成 JSON 配置文件，再决定是否导入为全局模板
- 修改 `.sydev/Makefile` 或 `.reproject` 前，先读 `references/workspace-files.md`
- 如果文档、旧技能和当前实现冲突，以当前 `sydev` 仓库文档和 CLI 实现为准

## 开始前检查

先确认工具链和上下文：

```bash
which sydev && sydev --version
which rl && which rl-workspace && which rl-project
```

然后判断当前任务属于哪一类：

- 新建环境：收集 `cwd`、`basePath`、`version`、`platform(s)`、`os`、`debugLevel`、`createBase`、`build`
- 维护现有 workspace：确认当前目录存在 `.realevo/`
- 用户只给了芯片或板卡名：读取 `references/platform-mapping.md` 推断 `--platforms`；如果推断风险高，明确说明推断点并只追问缺失的架构信息

## 选工作流

### 1. 一次性初始化整套环境

适用于“帮我把 workspace、工程、设备都建好”。

- 优先写 `full-config.json`
- 执行 `sydev init --config full-config.json`
- 如果用户还想下次直接复用，再执行 `sydev template import full-config.json`

### 2. 分步搭环境

适用于只改某一部分，或者用户想显式控制每一步。

- `sydev workspace init`
- `sydev project create`
- `sydev device add`
- `sydev build`
- `sydev upload`

### 3. 模板化复用环境

适用于“这套环境以后还要再来一遍”。

- 从现有 workspace 导出：`sydev template export -o sydev-config.json`
- 导入全局模板库：`sydev template import sydev-config.json`
- 在新目录落地：`sydev template apply sydev-config.json --cwd <ws> --base-path <base> -y`

### 4. 日常开发维护

适用于已有 workspace 的编译、Makefile、上传和配置修补。

- 刷新 `.sydev/Makefile`：`sydev build init`
- 编译：`sydev build <project> --quiet -- -j$(nproc)`
- 清理：`sydev clean <project>`
- 重建：`sydev rebuild <project> -- -j$(nproc)`
- 上传：`sydev upload <project> --device <device> --quiet`

## 任务手册

### 搭建 workspace

- 用户想要可复用方案时，优先生成 `workspace.json`，不要先拼很长一条命令
- `workspace init --config` 使用“命令参数风格”字段，细节见 `references/config-schema.md`
- 用户只说芯片名时，先看 `references/platform-mapping.md`

常用写法：

```bash
sydev workspace init --config workspace.json
```

或：

```bash
sydev workspace init \
  --cwd <ws> \
  --base-path <ws>/.realevo/base \
  --version default \
  --platforms <platforms> \
  --os sylixos \
  --debug-level release \
  --create-base \
  --build
```

### 创建或导入工程

- 现有仓库走 `--mode import`
- 新建模板工程走 `--mode create`
- 多工程环境优先落到 `full-config.json`，比多条 shell 命令更稳定

常用写法：

```bash
sydev project create --config project.json
```

### 设备管理

- 推荐先生成 `device.json` 再 `sydev device add --config device.json`
- 设备结构化数据优先读取 `.realevo/devicelist.json`，缺失时回退 `.realevo/config.json`

### 维护 `.sydev/Makefile`

- 文件不存在或工程列表变化后，先跑 `sydev build init`
- 可以安全修改：
  - `cp-<name>` target
  - `__` 开头的用户模板 target
- 不要手改：
  - 头部注释
  - `export WORKSPACE_*`
  - `.PHONY`
- `sydev build init --default` 会整份重生，只有用户明确要求覆盖时才用

### 编辑 `.reproject`

- 默认设备来自 `DevName`
- 上传条目优先使用 `<file local="..." remote="..."/>`
- `$(WORKSPACE_<project>)` 中项目名要把 `-` 换成 `_`
- `$(Output)` 会按 `config.mk` 的 `DEBUG_LEVEL` 变成 `Debug` 或 `Release`
- 路径里包含 `libsylixos` 时，上传器会把 `$(WORKSPACE_xxx)` 替换成 base 路径
- 兼容旧格式 `<PairItem key="..." value="..."/>`，但新内容优先写 `<file>`

### 生成可复用配置

生成 JSON 前先看 `references/config-schema.md`，尤其要记住两套字段名：

- 单命令 `--config`：`platforms`、`createBase`
- full/template：`workspace.platform`、`workspace.createbase`

常见组合：

```bash
sydev workspace init --config workspace.json
sydev project create --config project.json
sydev device add --config device.json
sydev init --config full-config.json
sydev template import full-config.json
```

## 结构化状态读取

需要机器可读状态时直接读这些文件：

- `.realevo/workspace.json`
- `.realevo/config.json`
- `.realevo/devicelist.json`
- 一级子目录中同时包含 `.project` 和 `Makefile` 的项目目录
- `<project>/config.mk`
- `<project>/.reproject`
- `.sydev/Makefile`

不要把 CLI 的彩色输出、人类排版或提示文案当成稳定接口。

## 何时读取参考资料

- `references/sydev-commands.md`
  精确命令、参数和行为边界
- `references/config-schema.md`
  生成或校验 `workspace.json`、`project.json`、`device.json`、`full-config.json`
- `references/workspace-files.md`
  编辑 `.reproject`、`.sydev/Makefile`、理解 workspace 文件布局
- `references/platform-mapping.md`
  用户只给芯片、板卡或 CPU 架构时做保守推断
