## Opus 音频库（libopus_codec.z.so）

## 简介
`Opus` 是一个开源、高效、低延迟的音频编解码库，支持高质量语音和音乐的编码与解码。它是第三方音频库，既可以被应用或服务独立调用，也可以作为 OpenHarmony AVCodec 音频链路中的软编解码能力被框架按需调用。

- **独立调用**：应用或服务可以直接调用 `libopus_codec.z.so` 提供的 Opus 编解码接口。
- **框架集成**：AVCodec 在进入 Opus 音频处理路径时，通过运行时动态装载方式调用 `libopus_codec.z.so`。

## 1. Opus 自身调用方式（独立使用）
Opus 作为第三方库具备独立调用能力，不依赖 AVCodec 框架即可完成音频编码或解码。

### 调用流程
1. 加载库：`dlopen("libopus_codec.z.so")`
2. 获取符号：通过 `dlsym` 获取 Opus 编码或解码相关接口。
3. 创建实例：初始化编码器或解码器上下文。
4. 编解码音频数据：输入编码数据或 PCM 数据，执行 Opus 编码/解码。
5. 获取输出：解码场景输出 PCM，编码场景输出 Opus 码流。

## 2. 系统架构（AVCodec 集成）
Opus 在 AVCodec 音频链路中的调用关系如下：

应用层 → AVCodec 公共接口层 → Framework 实现层 → 工厂/插件适配层 → `libopus_codec.z.so` → 音频输出层

- **应用层**：上层播放器、录音、通话或音频处理业务。
- **AVCodec 公共接口层**：AVCodec API，对外提供统一音频编解码能力入口。
- **Framework 实现层**：负责参数解析、实例创建、Buffer 管理及状态流转。
- **工厂/插件适配层**：根据 MIME、编解码类型等信息选择 Opus 处理路径。
- **第三方库层**：`libopus_codec.z.so`，在进入 Opus 路径时按需动态装载。
- **音频输出层**：解码场景输出 PCM Buffer，编码场景输出 Opus 码流。

## 3. AVCodec 运行时调用流程
1. 应用通过 AVCodec API 创建音频编解码实例，并设置 Opus 格式参数。
2. Framework 实现层接收配置并初始化编解码请求。
3. `codec_factory` / `engine factory` 根据格式选择 Opus 处理路径。
4. 插件适配层识别目标格式为 Opus。
5. 调用 `dlopen("libopus_codec.z.so")` 装载 Opus 动态库。
6. 调用 `dlsym` 获取 Opus 编解码接口符号。
7. 创建 Opus 编码器或解码器实例，并送入输入 Buffer。
8. 执行音频处理，并将输出 Buffer 返回上层。

<div align="center">
  <img src="zh-cn_image_opus.png" alt="OPUS调用流程图" />
  <br>
  <b>图 1</b> OPUS调用流程图
</div>

**说明**：Opus 的动态库装载发生在运行期，通常在首次进入 Opus 处理路径或首次创建 Opus 实例时触发。未进入 Opus 路径时，`libopus_codec.z.so` 通常不会被提前装载。

## 4. 仓目录结构
```text
/foundation/multimedia/third_party_opus
├── BUILD.gn             # OpenHarmony 编译配置
├── README.OpenSource    # OpenHarmony 开源合规说明
├── bundle.json          # OpenHarmony 部件描述文件
├── OAT.xml              # OpenHarmony OAT 扫描配置
├── src/                 # Opus 源码及 demo
├── include/             # Opus 头文件
└── test/                # 测试代码
```

## 5. 编译构建
### 独立构建 Opus
```bash
./build.sh --product-name {product_name} --ccache --build-target opus
```

### 编译 64 位 ARM 目标
```bash
./build.sh --product-name {product_name} --ccache --target-cpu arm64 --build-target opus
```

> `{product_name}` 为当前支持的平台名称，例如 `rk3568`。

### 编译产物
构建完成后生成 Opus 动态库：

```text
libopus_codec.z.so
```

AVCodec 使用 Opus 时，不通过 BUILD.gn 直接 deps 到主链路，而是在运行时通过 `dlopen` 按需装载 `libopus_codec.z.so`。

## 6. 使用示例
- **独立使用**：详见 `src/opus_demo.c`。
- **AVCodec 使用**：详见 `av_codec demo`。

## 7. 注意事项
- `libopus_codec.z.so` 为 Opus 在 OpenHarmony 中的动态库产物名称。
- Opus 是第三方音频库，不仅可以被 AVCodec 调用，也可以被其他应用或服务独立调用。
- AVCodec 音频软编解码链路为本地调用链路，不经过 IPC。
- Opus 在 AVCodec 中采用运行时动态装载方式，需要处理 `dlopen` 失败和 `dlsym` 符号解析失败。
- 未触发 Opus 音频处理路径时，`libopus_codec.z.so` 通常不会被提前装载。
- **OpenHarmony 特有文件**：仓库中存在 `bundle.json`、`OAT.xml`、`README.OpenSource`、`BUILD.gn`，这些文件为 OpenHarmony 社区添加或修改，用于部件描述、OAT 扫描、开源合规说明和 GN 构建集成。
