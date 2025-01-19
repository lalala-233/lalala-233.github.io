# 部分 AMD GPU 使用 Stable Diffusion 出图

日期：1.19

## 早期方案

首先，说一下我的显卡：Rx5500xt。

AMD 官方的 Rocm 并不支持这张卡，最近几个月的 Rocm6.2 增强了一些支持，因此可以使用 ollama 运行 LLM 了。

然而，pytorch 还是不支持 Rx5500xt，当我试图用这张卡使用 pytorch 时，大概率出现花屏。

使用 `HSA_OVERRIDE_GFX_VERSION=10.3.0` 也没什么用。

具体可以看看这个 [Issue](https://github.com/ROCm/hipBLASLt/issues/648#issuecomment-2508806941)，我的发行版「Archlinux」中的官方构建有 Rocm 的集成，翻了 PKGBUILD 发现有对 gfx1012 的构建。

但安装好 pytorch 和 Rocm 后，使用 pytorch 会花屏（不知道为什么），最近好了一些，只会 core dumped 了。

之前的解决方案是使用 pytorch 的早期版本（2.0 左右），这时候的 torch-rocm 没有一些 5500xt 不支持的功能（大概）。

好在有人构建了 torch2.0 的 gfx1012 版本的 [docker](https://github.com/set-soft/sd_webui_rx5500)，你可以使用 docker。

虽然也并不是开箱即用，但配置已经非常简单了。

## 现在方案

docker image 有点大，如果有一个项目像 [llama.cpp](https://github.com/ggerganov/llama.cpp) 一样就好了。

[llama.cpp](https://github.com/ggerganov/llama.cpp) 是对 llama 推理的 cpp 实现，支持多种后端。

好在社区中确实有很多类似的项目，比如 [Stable-Diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp)。

其实这篇文章便是在 [Stable-Diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) 更新后有感而发写的。

时隔三周，[Stable-Diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) 一下子合并了 6 个 PR，修复了 Rocm 的构建，完善了部分模型检查。

## 介绍

[Stable-Diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) 是对 Stable Diffusion 推理的 cpp 实现，已经较为完善了。

不过之前 Rocm 的构建出了点 bug，现在改好了。

于是乎：

```bash
cmake .. -G "Ninja" -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DSD_HIPBLAS=ON -DCMAKE_BUILD_TYPE=Release -DAMDGPU_TARGETS=gfx1012
cmake --build . --config Release
```

走起。

随便测试了一下，发现有非常稳定的推理速度，1.42s/it 跳都不跳一下。

同时，对 sdxl、sd-turbo 等的支持也更好了一点（有些模型的预测模式和其他模型不同，这次更新添加了一部分检测）。

然而，遇到一个离谱的事情，Rocm 后端的推理速度（1.42s/it）不如 Vulkan 后端（1.27s/it），这就很抽象了。

说好的 Rocm 专注于 AI 计算，而 Vulkan 性能略低呢？

当然，仅供参考就是了。
