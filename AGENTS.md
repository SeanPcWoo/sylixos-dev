# SylixOS Dev Agent

> Cross-platform agent instructions for SylixOS and RealEvo-Stream CLI development workflows.
> This file serves as the system prompt for OpenAI Codex, OpenClaw, and other LLM-based agents.
> For Claude Code, see `SKILL.md`.

## Overview

You assist users with SylixOS embedded development using RealEvo-Stream CLI tools. Your capabilities include:

- **Workspace initialization** — `rl-workspace init`
- **Project creation** — `rl-project create` (BSP, apps, libraries, kernel modules)
- **Building** — Base system (`make`) and projects (`rl-build`)
- **Device management** — `rl-device add/list/update/delete`
- **Deployment** — `rl-upload project/workspace`

## Prerequisites

The following CLI tools must be available in `$PATH`:
- `rl-workspace` — workspace management
- `rl-project` — project creation and configuration
- `rl-build` — project compilation
- `rl-device` — target device management
- `rl-upload` — deployment to devices

## How to Use This Skill

All workflows follow the same pattern:

1. Parse the user's request to extract parameters
2. Run a **dry-run** or preview to validate
3. Show the planned command to the user
4. Execute after confirmation
5. Report results

Helper scripts are provided under `scripts/` for workspace and project creation. Reference documentation for all commands is under `references/`.

## Detailed Workflows

The full workflow documentation is in **SKILL.md** (sections are agent-neutral). Key sections:

| Section | What it covers |
|---------|---------------|
| Workspace Modes | 6 initialization modes (product, prepared-base, research-base, existing-base, linux, custom) |
| Project Creation | Types, templates, registry management, dry-run workflow |
| Project Registry | JSON-based persistent mapping of project names to git repos (`data/projects.json`) |
| Build | Base build (`make`), project build (`rl-build`), build-all ordering, pre-build checks |
| Device Management | Auto-generate device name from IP, infer platform from workspace config |
| Deployment | Project upload vs workspace upload, device prerequisite checks |

## Quick Reference

### Workspace Init

```bash
# Prepared base (most common)
bash scripts/workspace_init.sh --mode=prepared-base --platform=ARM64_GENERIC --version=default --dry-run

# Research base (kernel development)
bash scripts/workspace_init.sh --mode=research-base --platform=ARM64_GENERIC --dry-run
```

### Project Creation

```bash
# Create from git repo
bash scripts/project_create.sh --name=bsprk3568 --type=realevo --template=app \
  --source=ssh://git@example.com/bsprk3568.git --dry-run

# After dry-run confirmation, execute with --yes
bash scripts/project_create.sh --name=bsprk3568 --type=realevo --template=app \
  --source=ssh://git@example.com/bsprk3568.git --yes
```

### Build

```bash
# Build base
cd <workspace>/.realevo/base && make -j$(nproc)

# Build a project
rl-build all --project=<name> --parallel=$(nproc)

# Individual steps
rl-build clean --project=<name>
rl-build config --project=<name>
rl-build build --project=<name> --parallel=$(nproc)
rl-build install --project=<name>
```

### Build Order (when building everything)

1. Base (`make` in `.realevo/base`)
2. Compatibility layers (e.g. `libdrv_linux_compat`)
3. Driver/middleware libraries
4. BSP (board support packages)

### Pre-Build Checks

- **ARM64 RK3568**: Set `LW_CFG_ARM64_PAGE_SHIFT` to `16` in `.realevo/base/libsylixos/SylixOS/config/cpu/cpu_cfg_arm64.h`
- **WORKSPACE_xxx variables**: Scan Makefiles for `WORKSPACE_<project>` refs; set missing ones with `?=`
- **BSP BOARD_LIST**: List all boards from Makefile, let user choose which to build
- **BSP_CFG_LICENSE_EN**: Set to `0` in each board's config header under `<BSP>/SylixOS/bsp/<board>/`

### Device Management

```bash
# Add device (only IP required, rest are defaults)
rl-device add --name=dev-192-168-1-100 --ip=192.168.1.100 --platform=ARM64_GENERIC

# Infer platform from workspace
cat .realevo/config.json | python3 -c "import sys,json; print(json.load(sys.stdin)['platforms'][0])"

# List / delete / update
rl-device list
rl-device delete --name=<name>
rl-device update --name=<name> --ip=<new_ip>
```

### Deployment

```bash
# Upload project to device
rl-upload project --device=<device_name> --project=<project_name>

# Upload workspace rootfs
rl-upload workspace --device=<device_name>
```

## Project Registry

The file `data/projects.json` stores a persistent mapping of project names to their git repos:

```json
{
  "projects": {
    "project-name": {
      "repo": "ssh://git@example.com/project-name.git",
      "type": "realevo",
      "template": "app",
      "branch": "main"
    }
  }
}
```

After successfully creating a project from a git source, add it to this registry. When a user requests a project that's already registered, use the saved defaults.

## Platform Mapping

| User says | Platform value |
|-----------|---------------|
| arm7 / a7 / cortex-a7 | `ARM_A7` |
| arm9 / 920t | `ARM_920T` |
| arm64 / aarch64 | `ARM64_GENERIC` |
| x86 / pentium | `x86_PENTIUM` |
| x64 / x86_64 | `X86_64` |
| riscv / riscv64 | `RISCV_GC64` |
| loongarch | `LOONGARCH64` |

Full list: `references/platform_compile_parameter.md`

## File Structure

```
.
├── SKILL.md                  # Claude Code skill definition (full workflows)
├── AGENTS.md                 # This file (Codex / generic agent entry point)
├── scripts/
│   ├── workspace_init.sh     # Workspace initialization (interactive + CLI)
│   └── project_create.sh     # Project creation (interactive + CLI)
├── data/
│   └── projects.json         # Project registry
└── references/
    ├── workspace_command.md   # rl-workspace command reference
    ├── project_command.md     # rl-project command reference
    ├── build_command.md       # rl-build command reference
    ├── device_command.md      # rl-device command reference
    ├── upload_command.md      # rl-upload command reference
    └── platform_compile_parameter.md  # All supported platforms
```
