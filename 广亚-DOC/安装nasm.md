# 为什么编译 FFmpeg 需要 nasm / yasm / pkg-config

## nasm / yasm — 汇编器

FFmpeg 为了极致性能，大量核心算法（解码器、滤镜、色彩空间转换等）都用**手写汇编**实现，而不是纯 C。这些 `.asm` 文件需要汇编器来编译。

- **nasm**（Netwide Assembler）— 现代主流汇编器，FFmpeg 4.x 主要用它
- **yasm** — 早期 FFmpeg 用的汇编器，4.x 仍有部分代码兼容它

**没有它们会怎样？**
`configure` 会检测到缺失，自动**禁用所有 SIMD 优化**（SSE、AVX、NEON 等）。代码能编过，但性能会大幅下降——同样的视频转码，可能慢 3~10 倍。

---

## pkg-config — 库信息查询工具

编译 FFmpeg 时需要链接大量第三方库（x264、x265、fdk-aac、openssl……）。每个库安装路径、头文件位置、链接参数各不相同。

`pkg-config` 的作用是**统一查询这些信息**：

```bash
# 比如查询 x264 的编译参数
pkg-config --cflags --libs x264
# 输出：-I/usr/local/include -L/usr/local/lib -lx264
```

FFmpeg 的 `configure` 脚本内部大量调用 `pkg-config` 来探测你系统上装了哪些库、怎么链接它们。

**没有它会怎样？**
`configure` 无法自动发现第三方库，即使你已经 `brew install x264`，FFmpeg 也会报"找不到 x264"而跳过，最终编出来的 ffmpeg 功能残缺。

---

## 总结

| 工具 | 作用 | 缺少后果 |
|------|------|----------|
| `nasm` | 编译 FFmpeg 内部的 x86/x64 汇编优化代码 | 禁用 SIMD，性能骤降 |
| `yasm` | 同上（兼容旧汇编代码） | 部分优化缺失 |
| `pkg-config` | 帮 configure 自动找到系统里的第三方库 | 第三方编解码器全部探测失败 |

> 这三个都是**编译工具链**，不会打包进最终的 ffmpeg 二进制，只在编译过程中使用。
