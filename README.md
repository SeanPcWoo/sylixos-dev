# sylixos-dev

一个 [Claude Code](https://claude.ai/code) 技能（Skill），为 SylixOS 嵌入式实时操作系统提供开发辅助。

## 功能

基于 `sydev` CLI 工具，覆盖 SylixOS 开发的完整工作流：

- **环境搭建** — workspace 初始化、Base 编译、设备配置
- **工程管理** — 创建 app / lib / ko 等类型的工程
- **编译上传** — 一键 build、upload 到目标板
- **调试验证** — telnet 登录设备，上板运行验证
- **第三方库移植** — 标准化的移植流程与 API 对照
- **平台映射** — 根据芯片名（rk3568、飞腾、龙芯等）自动推断目标平台

## 支持的平台

ARM32 / ARM64 / x86_64 / RISC-V / LoongArch / MIPS / PPC / SPARC / CSKY 等。

## 项目结构

```
├── SKILL.md              # 技能定义与决策流程
├── AGENTS.md             # Agent 配置
└── references/
    ├── sydev-commands.md       # sydev 命令详解
    ├── config-schema.md        # JSON 配置格式参考
    ├── workspace-files.md      # Workspace 文件结构说明
    ├── platform-mapping.md     # 芯片 → 平台映射表
    ├── sylixos-debugging.md    # 调试与验证流程
    ├── porting-guide.md        # 第三方库移植指南
    └── official-doc-routing.md # SylixOS 官方文档导航
```

## 安装

```bash
claude skill install-from-git <repo-url>
```

安装后，当对话涉及 SylixOS 开发相关内容时，技能会自动触发。

## 相关链接

- [SylixOS 官方文档](https://docs.acoinfo.com/sylixos/)
- [Claude Code](https://claude.ai/code)
