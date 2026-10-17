# Mu酱自备头像源码 · 验收清单（单一事实源）

版本 v1 · 2026-10-17

这份清单是「什么样的 `muse_pixel.c` 能被 Mu酱工坊接受」的唯一事实
源，三个消费方共用同一份：

1. **服务端静态检查**（现行）：`api/_lib.js` 的 `validAvatarCode` +
   `avatarCodeSafetyOk`、`api/avatar-code.js` 的 `findUndefinedCalls`
   （LINK_OK_EXTERN 白名单）。下方 B/C/D 三节的名单与代码常量逐项
   对应；`tools/gen-avatar-skill.py` 生成发布物时会逐项比对代码
   常量，对不上就拒绝生成——清单与检查器漂移在发布环节会被拦下。
2. **自备源码上传检查**（`userAvatarSourceOk`，随自备源码通道上线）：
   在现行检查之上加 E 节收紧层，名单以本清单为准。
3. **「Mu酱头像制作」Skill**（站内工坊区/Q&A 复制块 + GitHub 公开仓）：
   规格部分由本清单 + 官方配方（SDK `tools/muse/avatar_prompt.md`）
   生成，照 Skill 做出的文件按设计必过 1+2 的全部检查。

## A. 文件形状

- 单个完整的 C 文件（`muse_pixel.c`），纯文本，无 Markdown 围栏残留。
- 长度 1,000–120,000 字符。
- 必须包含渲染入口 `muse_pixel_render`，并实现 muse_pixel.h 的全部
  4 个函数：`muse_pixel_render`、`muse_pixel_accent`、
  `muse_pixel_scale`、`muse_pixel_set_size`。
- 文件开头保留 `// Copyright (c) Meta Platforms, Inc. and affiliates.`
  一行，其下写一段注释说明画的是什么形象（名字、形状、颜色、性格）。

## B. #include 白名单（只许这 6 个，逐字一致）

- `"muse_pixel.h"`
- `<math.h>`
- `<stdbool.h>`
- `<stdint.h>`
- `<stdlib.h>`
- `<string.h>`

## C. 禁用面（出现即拒）

- 禁用调用（词边界 + 后随括号）：`system`、`popen`、`fork`、`execve`、
  `execl`、`execlp`、`execle`、`execv`、`execvp`、`execvpe`、`socket`、
  `connect`、`fopen`、`freopen`、`open`、`creat`、`remove`、`rename`、
  `unlink`、`getenv`、`setenv`、`malloc`、`calloc`、`realloc`、`free`、
  `alloca`。
- 禁用构造：`asm`、`__asm__`、`__attribute__`、`__builtin_*`、`#undef`。
- 含义：只许静态内存——不许动态内存、文件、进程、网络、环境变量。

## D. 调用白名单（链接层）

全文出现的每个函数调用，目标只能是以下三类之一：

1. 在本文件内有定义的函数（含函数式宏）。
2. muse_pixel.h 的 4 个 API：`muse_pixel_render`、`muse_pixel_accent`、
   `muse_pixel_scale`、`muse_pixel_set_size`。
3. libc/libm 纯函数集（逐字名单）：
   `sin`、`cos`、`tan`、`asin`、`acos`、`atan`、`atan2`、`sinh`、`cosh`、
   `tanh`、`sqrt`、`pow`、`exp`、`log`、`log2`、`log10`、`floor`、`ceil`、
   `round`、`trunc`、`fabs`、`fmod`、`hypot`、`fmin`、`fmax`、`abs`、
   `labs`、`sinf`、`cosf`、`tanf`、`asinf`、`acosf`、`atanf`、`atan2f`、
   `sqrtf`、`powf`、`expf`、`logf`、`floorf`、`ceilf`、`roundf`、`truncf`、
   `fabsf`、`fmodf`、`hypotf`、`fminf`、`fmaxf`、`memset`、`memcpy`、
   `memmove`、`memcmp`、`strlen`、`strcmp`、`strncmp`、`strnlen`、
   `rand`、`srand`。

注：2026-10-17 实测——工坊正常产物普遍用 `sinf/expf/fabsf/fminf` 等
f 变体与 `expf`，本名单以工坊产物实测为准（早期设计稿的紧名单缺
f 变体，会误杀正常配方产物，已按本清单修正）。

## E. 收紧层（自备源码上传专用，与上传检查器同文）

- `extern` 声明全禁。
- 一切 `__` 前缀标识符全禁（实现保留名）。
- 预处理只许：`#include`（目标限 B 节）、`#define`、`#ifndef`、
  `#endif`、`#if defined(...)` 形式、`#else`。禁 `#pragma`、`#error`、
  `#undef`、`#warning`（`#ifdef` 请改写为 `#if defined(...)`）。
- 字符串字面量单个 ≤80 字符，且不得含 `://`。

2026-10-17 实测：官方原版 `muse_pixel.c`（38,363 字节）与工坊产物
样本均全过 A–E（原版仅用 `#define`/`#include`，无 extern、无 `__`
标识符，白名单外调用为零）。

## F. 渲染规格（官方配方要点，做图时遵守）

- 画布 64×64 调色板索引帧缓冲，索引 0 = 纯黑背景；调色板 ≤32 色
  （含背景），角色色表固定 + 每模式 glow 阶梯/accent 的模式配色表。
- 接口与姿态结构以 `muse_pixel.h` 为准：`muse_pose_t{mode, t,
  mode_t, level, happy}`；模式枚举 BOOT/IDLE/LISTENING/THINKING/
  SPEAKING/ERROR/OFF 共 7 个 + happy 摸头反应（ERROR 除外）。
- 角色约 32–36 px 宽、46–48 px 高，居中 x=32、脚底 y≈56.5；
  4×4 Bayer 有序抖动、1 px 硬轮廓、左上光源、地面抖动影。
- 逐像素循环用整型定点（Q12），逐像素循环内禁浮点与
  `pow/sqrt/sin/cos`（用预建查表）；浮点只许逐帧/逐部件用。
- 静态内存 only（与 C 节一致）；帧耗时目标 S3 <10 ms。
- 每个模式的动画节拍（呼吸/眨眼/视线游移/火花为全模式共有；
  BOOT 弹出睁眼、LISTENING 竖耳环圈随 level、THINKING 托腮三点、
  SPEAKING 口型随 level、ERROR X 眼红配色、OFF 挥手淡出、
  happy 跳跃爱心）与默认模式配色表见官方配方原文
  （SDK `esp32/tools/muse/avatar_prompt.md`），Skill 块内全量内嵌。

## G. 交付前自检（逐项打勾，全过才交付）

1. 文件是单个完整 `muse_pixel.c`，长度在 1,000–120,000 字符内，
   4 个 API 函数都实现了，开头有版权行 + 形象说明注释。
2. `#include` 逐条比对 B 节：只有白名单 6 个。
3. 全文搜索 C 节禁用词：零命中（注意 `free`/`open` 等词边界）。
4. 全文每个函数调用逐个核对 D 节：文件内定义 / 4 API / libc 名单，
   三类之外零个（最常见翻车：调用了从未定义的 helper）。
5. 无 `extern`、无 `__` 开头标识符；预处理只有 E 节允许的几种，
   `#if` 都是 `defined(...)` 形式；字符串都 ≤80 字符、无 `://`。
6. 调色板条目数 ≤32（含背景索引 0）；逐像素循环里无浮点运算与
   禁用数学函数（循环外逐帧用浮点可以）。
7. 7 个模式都有动画、happy 反应有（ERROR 除外），配色表 7 行齐。
