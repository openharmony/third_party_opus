# 三方开源软件 opus

- [三方开源软件 opus](#三方开源软件-opus)
    - [1. opus 简介](#1-opus-简介)
    - [2. 引入背景简述](#2-引入背景简述)
    - [3. 使用场景](#3-使用场景)
    - [4. 为 OpenHarmony 带来的价值](#4-为-openharmony-带来的价值)
    - [5. 如何使用](#5-如何使用)
        - [5.1 系统架构（AVCodec 集成）](#51-系统架构avcodec-集成)
        - [5.2 编译构建](#52-编译构建)
        - [5.3 使用示例](#53-使用示例)
        - [5.4 仓目录结构](#54-仓目录结构)
        - [5.5 注意事项](#55-注意事项)

## 1. opus 简介

Opus 是一个开源、高效、低延迟的音频编解码库，支持高质量语音和音乐的编码与解码。本仓库是 Opus 在 OpenHarmony 中的适配版本，应用无需直接集成此库，通过 AVCodec Kit API 即可调用到本库提供的 Opus 编解码能力。

Opus 官方主页：[opus-codec.org](https://opus-codec.org/)，了解更多关于 Opus 项目的信息。

源代码参考资料可以访问：[Opus GitHub 社区代码仓库](https://github.com/xiph/opus)。

**框架集成**：AVCodec 通过 BUILD.gn deps 链接方式将 opus 编解码库纳入依赖图，运行时由动态链接器在进程装载阶段自动完成库装载，无需运行期手动 `dlopen` / `dlsym`。

## 2. 引入背景简述

OpenHarmony 媒体编解码AVCodec Kit需要提供丰富的音频编解码能力，其中 Opus 是一种广泛使用的低延迟、高质量音频编解码格式，适用于语音通话、音乐流媒体等场景。Opus 是业界成熟、经过广泛验证的开源音频编解码实现方案，引入该库可快速补齐 OpenHarmony 在 Opus 编解码方面的能力缺口。

## 3. 使用场景

- 语音通话、视频会议等实时通讯场景中需要低延迟的音频编解码。
- 音乐流媒体、播客、录音机等应用场景中需要高质量的音频编码与播放。
- 音频转码工具需要将其他编码格式转换为 Opus 码流，或将 Opus 码流解码为 PCM。

## 4. 为 OpenHarmony 带来的价值

1. 补充 AVCodec Kit的 Opus 编解码能力，完善 OpenHarmony 多媒体音频编解码生态。
2. 满足生态伙伴产品对 Opus 格式的兼容性需求，降低接入成本。

## 5. 如何使用

### 5.1 系统架构（AVCodec 集成）

Opus 在 AVCodec 音频链路中的调用关系如下：

应用层 → AVCodec Kit 接口层 → Framework 实现层 → 工厂/插件适配层 → opus → 音频输出。

- **应用层**：上层播放器、录音、通话或音频处理业务。
- **AVCodec Kit接口层**：AVCodec API，对外提供统一的音频编解码能力入口。
- **Framework实现层**：负责参数解析、实例创建、Buffer 管理及状态流转。
- **插件适配层**：根据 MIME（Multipurpose Internet Mail Extensions）、编码类型等信息选择 Opus 处理路径。
- **第三方库层**：opus，提供 Opus 编解码核心实现。
- **AVCodec Kit接口输出**：解码场景输出 PCM Buffer，编码场景输出 Opus 码流。

### 5.2 编译构建

编译 64 位 ARM 目标：

```bash
./build.sh --product-name {product_name} --ccache --target-cpu arm64 --build-target opus
```

> `{product_name}` 为当前支持的平台名称，例如 `rk3568`。

### 5.4 使用示例

- **AVCodec Kit使用**：详见 [AVCodec 音频编码指南](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/media/avcodec/audio-encoding.md)和[AVCodec 音频解码指南](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/media/avcodec/audio-decoding.md)。

### 5.5 仓目录结构

```text
third_party_opus/
├── BUILD.gn             # OpenHarmony 编译配置
├── README.OpenSource    # OpenHarmony 开源合规说明
├── bundle.json          # OpenHarmony 部件描述文件
├── OAT.xml              # OpenHarmony OAT 扫描配置
├── src/                 # Opus 源码
├── include/             # Opus 头文件
└── test/                # 测试代码
```

### 5.6 注意事项

- AVCodec 音频软编解码链路为本地调用链路，不经过 IPC。
- Opus 通过 `BUILD.gn` deps 纳入依赖图，运行期无需显式 `dlopen` / `dlsym`。
- **OpenHarmony 特有文件**：仓库中存在 `bundle.json`、`OAT.xml`、`README.OpenSource`、`BUILD.gn`、`README_zh.md`，这些文件为 OpenHarmony 社区添加或修改，用于部件描述、OAT 扫描、开源合规说明和 GN 构建集成。

