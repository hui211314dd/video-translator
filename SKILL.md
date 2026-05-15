---
name: video-translator
author: JimmyLi
description: 利用 FFmpeg 和 Whisper 提取英文视频或者音频中的语音内容，导出成 txt 文件的流程 (根据用户的选项也可以导出成 srt 字幕文件), 最后要求将 txt 文件中的内容翻译成中文并输出在控制台上;
disable-model-invocation: true
allowed-tools: ffmpeg, whisper-cli, Bash(ffmpeg*), Bash(whisper-cli*)
---

## 本地环境要求

* 已经安装好 FFmpeg，并且能够在命令行中保证"ffmpeg -version"可用；
* 已经安装好 Whisper，并且能够在命令行中保证"whisper-cli -h"可用；
* 已经下载好 ggml-medium-q8_0 模型，并且模型文件路径存储在{WHISPER_MODEL_PATH}的环境变量中;

## 本地环境不满足要求时的提示
如果本地环境不满足要求，Skill 会提示用户安装 FFmpeg 和 Whisper，并且正确设置环境变量{WHISPER_MODEL_PATH}，以确保 Skill 能够正常运行。用户可以根据提示进行安装和配置，之后再次尝试激活 Skill。

* [FFmpeg](https://ffmpeg.org/download.html)，用户需要根据自己的操作系统选择合适的版本进行安装，并确保 ffmpeg 命令在命令行中可用；

* [Whisper](https://github.com/ggml-org/whisper.cpp/releases)，用户需要根据自己的操作系统选择合适的版本进行安装，并确保 whisper-cli 命令在命令行中可用；

* [HuggingFace 下载模型](https://huggingface.co/ggerganov/whisper.cpp/tree/main)，用户需要下载 ggml-medium-q8_0.bin 模型，并将其路径设置到环境变量{WHISPER_MODEL_PATH}中；

## Skill 介绍

利用 ffmpeg 工具将用户提供的视频转换成音频文件，然后使用 Whisper 工具 + ggml-medium-q8_0 模型 将音频文件解析成全文本或者字幕文件，这个流程可以帮助用户快速获取视频中的语音内容，并且方便地进行翻译和观看。


## Skill 流程

激活此 Skill 时：

1. 如果用户传入的是 mp3 资源，则直接进入步骤 3；
2. 如果用户传入的是其他音视频资源，则先使用 ffmpeg 转换成 mp3 音频文件，比如：
    ```
    ffmpeg -i 音视频资源全路径 -vn 临时路径/临时名字.mp3
    ```
3. 使用 Whisper 将 mp3 音频文件解析成字幕文件或者全文本，比如：
    ```
    whisper-cli -m 环境变量{WHISPER_MODEL_PATH}/ggml-medium-q8_0.bin 临时路径/临时名字.mp3 -osrt -otxt
    ```
4. 执行完毕后，就会在 mp3 文件所在的目录下生成一个同名的 txt 文件和 srt 字幕文件；

5. 将 txt 文件中的内容翻译成中文，并输出在控制台上，翻译过程中有如下要求：
    * 翻译前先大概了解这是哪个领域的视频内容，比如游戏开发、电影、科技等，以便更好地理解和翻译文本内容；
    * txt 文件中的内容可能包含一些专业术语或者缩写，需要根据上下文进行合理的翻译，确保翻译结果准确且易于理解；
    * txt 文件中的内容可能一段内容被分成了很多行，翻译时需要根据内容分为合理的段落，而不是很多行；
    * 标点符合要符合中文技术文档的要求；
    * 要事无巨细的翻译，翻译时切勿丢掉任何内容;

## 举例

假设用户传入了一个视频资源 "C:\Users\JimmyLi\Videos\12345.mp4"，那么 Skill 会先使用 ffmpeg 将其转换成 mp3 音频文件 "C:\Users\JimmyLi\Videos\12345.mp3"，然后使用 Whisper 将 mp3 文件解析成字幕文件 "C:\Users\JimmyLi\Videos\12345.srt" 和全文本 "C:\Users\JimmyLi\Videos\12345.txt"，用户可以根据需要选择使用哪个文件。

```
ffmpeg -i "C:\Users\JimmyLi\Videos\12345.mp4" -vn "C:\Users\JimmyLi\Videos\12345.mp3"

whisper-cli -m %WHISPER_MODEL_PATH%/ggml-medium-q8_0.bin "C:\Users\JimmyLi\Videos\12345.mp3" -osrt -otxt
```