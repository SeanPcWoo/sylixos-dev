# sydev JSON 配置参考

最容易出错的地方不是字段值，而是字段名风格。`sydev` 目前有两套 JSON 约定。

## 先记住两套字段名

### 1. 单命令 `--config`

用于：

- `sydev workspace init --config workspace.json`
- `sydev project create --config project.json`
- `sydev device add --config device.json`

这是“命令参数风格”：

```json
{
  "platforms": ["ARM64_GENERIC"],
  "createBase": true,
  "debugLevel": "release"
}
```

### 2. full 配置 / 模板

用于：

- `sydev init --config full-config.json`
- `sydev template import full-config.json`
- `sydev template apply full-config.json`
- `sydev template export`

这是“完整配置风格”：

```json
{
  "workspace": {
    "platform": ["ARM64_GENERIC"],
    "createbase": true,
    "debugLevel": "release"
  }
}
```

## 字段差异对照

| 单命令 `--config` | full / template | 说明 |
| --- | --- | --- |
| `platforms` | `workspace.platform` | 单复数不同 |
| `createBase` | `workspace.createbase` | 大小写不同 |
| `cwd` | `workspace.cwd` | full 配置嵌套在 `workspace` 下 |
| `basePath` | `workspace.basePath` | 同上 |

不要混用这两套字段名。

## 合并优先级

以下三类命令遵循同一规则：

- `workspace init --config ...`
- `project create --config ...`
- `device add --config ...`

```text
命令行参数 > JSON 文件 > 默认值
```

## `workspace init --config`

```bash
sydev workspace init --config workspace.json
```

```json
{
  "cwd": "/path/to/ws",
  "basePath": "/path/to/ws/.realevo/base",
  "version": "lts_3.6.5",
  "platforms": ["ARM64_GENERIC", "X86_64"],
  "os": "sylixos",
  "debugLevel": "release",
  "createBase": true,
  "build": false,
  "arm64PageShift": 12,
  "baseComponents": ["libsylixos", "openssl", "libcextern"]
}
```

`arm64PageShift` 仅在选择 ARM64 平台时有效，可选值：`12`(4K)、`14`(16K)、`16`(64K)。
`baseComponents` 中 `libsylixos` 必选；如果不传则使用默认全组件。

特殊版本附加字段：

```json
{
  "version": "custom",
  "customRepo": "ssh://git@example.com/custom-base.git",
  "customBranch": "main"
}
```

```json
{
  "version": "research",
  "researchBranch": "dev/rk3568"
}
```

## `project create --config`

### 导入模式

```json
{
  "mode": "import",
  "name": "bsp-rk3568",
  "source": "https://git.example.com/bsp/rk3568.git",
  "branch": "main",
  "makeTool": "make"
}
```

### 创建模式

```json
{
  "mode": "create",
  "name": "my-app",
  "template": "app",
  "type": "cmake",
  "debugLevel": "release",
  "makeTool": "ninja"
}
```

## `device add --config`

```json
{
  "name": "board1",
  "ip": "192.168.1.100",
  "platforms": ["ARM64_GENERIC"],
  "username": "root",
  "password": "root",
  "ssh": 22,
  "telnet": 23,
  "ftp": 21,
  "gdb": 1234
}
```

## `sydev init --config`

```bash
sydev init --config full-config.json
```

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
    {
      "name": "bsp-rk3568",
      "source": "https://git.example.com/bsp/rk3568.git",
      "branch": "main",
      "makeTool": "make"
    },
    {
      "name": "my-app",
      "template": "app",
      "type": "cmake",
      "debugLevel": "release",
      "makeTool": "make"
    }
  ],
  "devices": [
    {
      "name": "board1",
      "ip": "192.168.1.100",
      "platform": ["ARM64_GENERIC"],
      "username": "root",
      "password": "root",
      "ssh": 22,
      "telnet": 23,
      "ftp": 21,
      "gdb": 1234
    }
  ]
}
```

注意：

- `workspace.platform` 是数组，不是 `platforms`
- `workspace.createbase` 全小写，不是 `createBase`
- `workspace.cwd` / `workspace.basePath` 缺失时，`sydev init` 会提示输入

## `template import` / `template apply` 支持的 JSON

### 1. 原始 full 配置

也是 `template export` 的默认输出：

```json
{
  "schemaVersion": 1,
  "workspace": {
    "platform": ["ARM64_GENERIC"],
    "version": "lts_3.6.5",
    "createbase": true,
    "build": false,
    "debugLevel": "release",
    "os": "sylixos"
  },
  "projects": [],
  "devices": []
}
```

### 2. wrapped full 模板

```json
{
  "type": "full",
  "data": {
    "schemaVersion": 1,
    "workspace": {
      "platform": ["ARM64_GENERIC"],
      "version": "lts_3.6.5",
      "createbase": true,
      "build": false,
      "debugLevel": "release",
      "os": "sylixos"
    },
    "projects": [],
    "devices": []
  }
}
```

也兼容：

```json
{
  "type": "full",
  "full": {
    "schemaVersion": 1,
    "workspace": {
      "platform": ["ARM64_GENERIC"]
    }
  }
}
```

### 3. 单类型 wrapped 模板

```json
{
  "type": "workspace",
  "workspace": {
    "platform": ["ARM64_GENERIC"],
    "version": "default",
    "createbase": true
  }
}
```

```json
{
  "type": "project",
  "project": {
    "name": "my-app",
    "template": "app",
    "type": "cmake"
  }
}
```

```json
{
  "type": "device",
  "device": {
    "name": "board1",
    "ip": "192.168.1.100",
    "platform": ["ARM64_GENERIC"]
  }
}
```

## `template apply` 的路径覆盖规则

`template apply` 会重新决定 workspace 路径：

1. 命令行 `--cwd` / `--base-path`
2. 交互输入
3. `-y` 下的默认值：`process.cwd()` / `${cwd}/.realevo/base`

即使 JSON 文件里已经带了 `workspace.cwd` / `workspace.basePath`，`apply` 也会用本次命令解析出来的路径覆盖。

## `template export` 输出格式

```bash
sydev template export -o sydev-config.json
```

导出的就是“原始 full 配置”，可以直接继续用于：

- `sydev init --config sydev-config.json`
- `sydev template import sydev-config.json`
- `sydev template apply sydev-config.json --cwd ... --base-path ... -y`
