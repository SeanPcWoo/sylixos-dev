# SylixOS 第三方库移植指南

本文档总结将第三方开源库（如 DPDK、网络协议栈、驱动框架等）移植到 SylixOS 的标准方法论。

## 移植标准流程

### 第一步：创建工程并初始化版本控制

```bash
# 1. 在 workspace 中创建工程
sydev project create --mode create --name lib<name> --template app --type realevo --debug-level release

# 2. 进入工程目录，git 初始化
cd lib<name>
git init
git add -A
git commit -m "Initial project from sydev template"
```

### 第二步：导入源码

```bash
# 3. 删除默认 src/ 目录，创建源码结构
rm -rf src/
mkdir -p <source_dir>/...

# 4. 从第三方源码树拷贝需要的文件
cp -r /path/to/upstream/src/* <source_dir>/

# 5. 提交原始未修改的源码
git add -A
git commit -m "Import <library> <version> source files (unmodified)"
```

**重点**：先提交原始源码，再做修改。这样 `git diff` 可以清晰地看到所有移植改动。

### 第三步：条件编译修改

**核心原则**：使用 `#ifdef SYLIXOS / #else / #endif` 保留原始实现。

```c
/* 示例：替换平台特定 API */
#ifdef SYLIXOS
    ncpu = (int)sysconf(_SC_NPROCESSORS_ONLN);
#else  /* !SYLIXOS - original FreeBSD/Linux implementation */
    sysctl(mib, 2, &ncpu, &len, NULL, 0);
#endif /* SYLIXOS */
```

**修改分类**：

| 类型 | 方法 | 示例 |
|------|------|------|
| 头文件差异 | `#ifdef SYLIXOS` 包裹 include | `<sys/sysctl.h>` → `<unistd.h>` |
| 系统调用替换 | 函数体内条件编译 | `sysctl()` → `sysconf()` |
| 类型差异 | typedef 条件编译 | `cpuset_t` → `cpu_set_t` |
| 整个函数替换 | 函数体级别条件编译 | kqueue → select+pipe |
| 宏/常量缺失 | 新建 compat 头文件提供 | `CPU_COUNT`, `MADV_NOCORE` |

### 第四步：处理 SylixOS 特有冲突

创建 `dpdk_sylixos_compat.h`（或 `<lib>_sylixos_compat.h`），通过 `-include` 强制包含：

**常见冲突及解决方案**：

| 冲突 | 原因 | 解决 |
|------|------|------|
| `optarg` 宏冲突 | SylixOS 定义为线程局部宏 | undef 后 extern 声明 |
| `SCHED_OTHER == SCHED_RR` | SylixOS 将 OTHER 映射为 RR | undef 重定义为不同值 |
| `CPU_COUNT` 缺失 | SylixOS 不提供 | 手动实现循环宏 |
| `flock()` 缺失 | SylixOS 可能不支持 | 提供 LOCK_EX/NB/UN 定义 |
| `-mgeneral-regs-only` 与浮点/NEON | 内核态编译选项 | 改用 `STATIC_LIBRARY_MK`（用户态） |

### 第五步：配置构建系统

**`config.mk`**：设置正确的 `SYLIXOS_BASE_PATH` 和 `PLATFORM_NAME`。

**`lib<name>.mk`**：

```makefile
# 目标：静态库
LOCAL_TARGET_NAME := lib<name>.a

# 源文件列表
LOCAL_SRCS := ...

# Include 路径：注意架构特定头文件优先于 generic
LOCAL_INC_PATH := \
-I"<arch_include>" \
-I"<generic_include>" \
...

# 宏定义：必须包含 -DSYLIXOS
LOCAL_DSYMBOL := \
-DSYLIXOS \
-DALLOW_EXPERIMENTAL_API \
-DALLOW_INTERNAL_API

# 编译选项：force-include 配置头文件和 compat 头文件
LOCAL_CFLAGS := \
-include <lib>_config.h \
-include <lib>_sylixos_compat.h

# 库类型选择
include $(STATIC_LIBRARY_MK)     # 用户态静态库（推荐，支持浮点/NEON）
# include $(KERNEL_LIBRARY_MK)   # 内核态库（限制 -mgeneral-regs-only）
```

**库类型选择指南**：
- `STATIC_LIBRARY_MK`：用户态静态库，可用浮点和 NEON，适合 DPDK 等数据面库
- `KERNEL_LIBRARY_MK`：内核态库，有 `-mgeneral-regs-only` 限制，适合驱动
- `APPLICATION_MK`：用户态可执行程序

### 第六步：编译验证

```bash
sydev build lib<name> -- -j$(nproc)
```

常见编译错误及修复模式：

| 错误 | 原因 | 修复 |
|------|------|------|
| `duplicate case value` | SylixOS 宏值与其他平台相同 | compat 头文件中 undef 重定义 |
| `Symbol is not public ABI` | 调用了内部 API | 加 `-DALLOW_INTERNAL_API` |
| `incomplete element type` | include 路径顺序错误 | 架构头文件优先于 generic |
| `conflicting types` | SylixOS 宏替换了函数参数名 | compat 头文件中 undef |
| `-mgeneral-regs-only` 与浮点 | 内核态编译限制 | 改为 `STATIC_LIBRARY_MK` |

### 第七步：编写文档并提交

```bash
# 编写移植文档
cat > PORTING.md << 'EOF'
# <Library> SylixOS 移植文档
## 修改的文件
## 关键适配点
## 已知限制
EOF

git add -A
git commit -m "Port <library> to SylixOS with documentation"
```

## 选择移植基础平台

| 被移植库特征 | 推荐基础 | 原因 |
|-------------|---------|------|
| 有 FreeBSD 支持 | FreeBSD | SylixOS 与 FreeBSD 同为 BSD 系，API 更接近 |
| 仅有 Linux | Linux | 直接从 Linux 移植，注意 epoll→select 等差异 |
| 仅有 POSIX | 直接用 | SylixOS 有良好的 POSIX 支持 |
| 有 RTOS 支持 | RTOS 版本 | FreeRTOS/VxWorks 实现可能更接近 |

## SylixOS 与 FreeBSD 的关键 API 差异

| 功能 | FreeBSD | SylixOS |
|------|---------|---------|
| 事件通知 | kqueue/kevent | select/poll |
| CPU 集合 | cpuset_t | cpu_set_t |
| 线程 ID | thr_self() | pthread_self() |
| 线程命名 | pthread_set_name_np() | pthread_setname_np() |
| CPU 计数 | sysctl HW_NCPU | sysconf(_SC_NPROCESSORS_ONLN) |
| 系统配置 | sysctlbyname() | sysconf() / 固定配置 |
| 连续内存 | contigmem 驱动 | mmap MAP_ANONYMOUS |
| I/O 特权 | /dev/io | MMIO（ARM64 不需要） |
| 文件锁 | flock() | fcntl() F_SETLK |
| core dump 控制 | MADV_NOCORE | 不支持（定义为空操作） |

## SylixOS 与 Linux 的关键 API 差异

| 功能 | Linux | SylixOS |
|------|-------|---------|
| 事件通知 | epoll | select/poll |
| 大页 | hugetlbfs / mmap MAP_HUGETLB | mmap MAP_ANONYMOUS |
| 物理地址 | /proc/self/pagemap | 需内核驱动 |
| UIO | /dev/uioN | 不支持 |
| VFIO | /dev/vfio | 不支持 |
| 线程命名 | pthread_setname_np() | pthread_setname_np()（相同） |
| core dump | MADV_DONTDUMP | 不支持 |
| optarg | extern 变量 | 线程局部宏（需 undef） |
| SCHED_OTHER | 独立值 | == SCHED_RR（需重定义） |
