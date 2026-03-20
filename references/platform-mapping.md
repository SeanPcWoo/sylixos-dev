# 平台映射与保守推断

用户经常只说芯片名、板卡名或架构名，不会直接给 `sydev --platforms`。这里给的是“保守推断”，目的是减少误判，而不是把所有名字都硬映射成一个平台。

## 基本规则

- 用户已经给了明确的 RealEvo 平台名时，直接用用户值，不要再替换
- 只在架构非常明确时才直接推断
- 不确定时，先问“32 位还是 64 位”或“是否已有现成平台名”
- 默认优先选择通用平台值，而不是过度优化到某个特定 core
- `*_SOFT` 平台只在用户明确要求软浮点或已有工具链要求时才选

## 当前 `sydev` 支持的常见平台族

- ARM32: `ARM_A5` `ARM_A7` `ARM_A8` `ARM_A9` `ARM_A15` `ARM_V7A`
- ARM64: `ARM64_A53` `ARM64_A55` `ARM64_A57` `ARM64_A72` `ARM64_GENERIC`
- x86: `x86_PENTIUM` `X86_64`
- RISC-V: `RISCV_GC32` `RISCV_GC64`
- LoongArch: `LOONGARCH64`
- 另外还支持 MIPS / PPC / SPARC / CSKY / SW6B 等

## 可以直接推断的常见映射

| 用户说法 | 默认平台 | 备注 |
| --- | --- | --- |
| `rk3568` | `ARM64_GENERIC` | 若用户明确要 A55 优化，可用 `ARM64_A55` |
| `rk3588` | `ARM64_GENERIC` | 大小核混合，默认不要猜特定 core |
| `rk3399` | `ARM64_GENERIC` | 只有明确要大核优化时才考虑 `ARM64_A72` |
| `cortex-a53` | `ARM64_A53` | |
| `cortex-a55` | `ARM64_A55` | |
| `cortex-a57` | `ARM64_A57` | |
| `cortex-a72` | `ARM64_A72` | |
| `armv7-a` | `ARM_V7A` | 通用 ARMv7-A |
| `imx6ull` | `ARM_A7` | 32 位，不要映射到 ARM64 |
| `allwinner t3` / `a40i` | `ARM_A7` | 32 位 A7 系列 |
| `x86_64` / `amd64` / `intel 64` / `amd 64` | `X86_64` | |
| `risc-v64` / `rv64` / `jh7110` / `c910` | `RISCV_GC64` | |
| `risc-v32` / `rv32` | `RISCV_GC32` | |
| `loongarch64` / `3a5000` / `3a6000` | `LOONGARCH64` | |
| `飞腾 ft2000/4` / `d2000` / `s2500` | `ARM64_GENERIC` | 默认先走通用 ARM64 |

## 需要确认后再选的平台

### 用户只说“龙芯”

不要直接映射。

- 新一代 LoongArch 机器通常走 `LOONGARCH64`
- 较老的龙芯 3A 系列可能更接近 `MIPS64_LS3A`

先确认是 LoongArch 还是旧 MIPS64 平台。

### 用户只说“RISC-V”

先确认是 32 位还是 64 位：

- `RISCV_GC32`
- `RISCV_GC64`

### 用户只说“x86”

默认可推 `X86_64`，但如果用户明确是 32 位老平台，再考虑 `x86_PENTIUM`。

### 用户只说厂商名

例如“海思”“全志”“NXP”“瑞芯微”。

不要只凭厂商名推平台，至少确认具体芯片或 CPU 架构。

## 多平台写法

`sydev` 支持多个平台同时传入，逗号分隔：

```bash
sydev workspace init --platforms ARM64_GENERIC,X86_64
```

JSON 里写数组：

```json
{
  "platforms": ["ARM64_GENERIC", "X86_64"]
}
```

或 full 配置：

```json
{
  "workspace": {
    "platform": ["ARM64_GENERIC", "X86_64"]
  }
}
```

## 实际使用建议

- 能用通用平台先用通用平台，除非用户明确要求按 core 优化
- 遇到模糊名字时，不要强猜；问 1 个架构问题比把环境搭错更便宜
- 如果用户已经有旧 workspace 或旧 `config.mk`，优先沿用现有平台名而不是重新猜
