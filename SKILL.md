---
name: video-translator
author: JimmyLi
description: 利用 FFmpeg 和 Whisper 提取英文视频或者音频中的语音内容，导出成 txt 文件的流程 (根据用户的选项也可以导出成 srt 字幕文件), 最后要求将 txt 文件中的内容翻译成中文并输出在控制台上;
disable-model-invocation: true
allowed-tools: ffmpeg, whisper-cli, Bash(ffmpeg*), Bash(whisper-cli*)
---

## 本地环境检查

* Skill 中包括了 FFmpeg，路径在 /Tools/ffmpeg/bin/中；
* Skill 中包括了 Whisper，路径在 /Tools/whisper/Release/中；
* Skill 中包括了 ggml-medium-q8_0 模型，路径在 /Tools/models/中;


## Skill 介绍

利用 ffmpeg 工具将用户提供的视频转换成音频文件，然后使用 Whisper 工具 + ggml-medium-q8_0 模型 将音频文件解析成全文本或者字幕文件，这个流程可以帮助用户快速获取视频中的语音内容，并且方便地进行翻译和观看。


## Skill 流程

定义 SKILL_PATH 为本 skill.md 所在的全路径，比如 SKILL_PATH=C:\Users\JimmyLi\Documents\Skills\video-translator，那么 ffmpeg 的全路径就是 {SKILL_PATH}/Tools/ffmpeg/bin/ffmpeg.exe，Whisper 的全路径就是 {SKILL_PATH}/Tools/whisper/Release/whisper-cli.exe，ggml-medium-q8_0 模型的全路径就是 {SKILL_PATH}/Tools/models/ggml-medium-q8_0.bin。

下面介绍工具使用时没有写工具的全路径，需要根据实际情况进行替换。

激活此 Skill 时：

1. 如果用户传入的是 mp3 资源，则直接进入步骤 3；
2. 如果用户传入的是其他音视频资源，则先使用 ffmpeg 转换成 mp3 音频文件，比如：
    ```
    ffmpeg -i 音视频资源全路径 -vn 临时路径/临时名字.mp3
    ```
3. 使用 Whisper 将 mp3 音频文件解析成字幕文件或者全文本，下面的例子同时输出了字幕文件 (-osrt) 和文本文件 (-otxt)：
    ```
    whisper-cli -m ggml-medium-q8_0.bin -t 8 -p 1 -fa -nf 临时路径/临时名字.mp3 -osrt -otxt
    ```

    > [重要] 必须用 `-p 1`（单处理器），**切勿用 `-p 2`**。`-p` 会把音频切成多段并行转写再拼接，长音频下拼接时的"分段时间戳偏移"会算错（曾出现 `split 1 - 00:-8:-34.-830` 负时间戳），导致 srt 时间轴中途重置、非单调、结尾时间远小于音频实际时长，第 6 步的字幕同步将无法完成。`-t 8` 已在单个上下文内做多线程并行，速度足够；若想更快可把 `-t` 调到机器物理核数（如 `-t 16`），但务必保持 `-p 1`。
4. 执行完毕后，就会在 mp3 文件所在目录下生成 txt 文件和 srt 字幕文件。注意：whisper-cli 是在**输入文件全名（含 `.mp3`）后面追加后缀**，所以 `12345.mp3` 生成的是 `12345.mp3.txt` 和 `12345.mp3.srt`（不是 `12345.txt` / `12345.srt`）。

    生成后做一次**时间轴校验**（防止 `-p` 误用等导致时间轴损坏）：用 `ffmpeg -i` 读出音频 Duration，再确认 srt 最后一条字幕的结束时间与之接近、且整段 srt 时间戳单调递增。
    ```
    ffmpeg -i 临时路径/临时名字.mp3          # 看 stderr 里的 Duration
    # 再核对 srt 文件最后一个 "-->" 右侧时间 ≈ Duration，且时间戳不回退
    ```
    若时间轴明显不对（末尾时间远小于音频时长、或中途出现时间回退），说明转写异常，需用 `-p 1` 重新转写后再继续。

5. 将 txt 文件中的内容翻译成中文并保存到同目录的另外一个 txt 文件中，同时将翻译后的文本输出在控制台上，翻译过程中有如下要求：
    * 翻译前先大概了解这是哪个领域的视频内容，比如游戏开发、电影、科技等，以便更好地理解和翻译文本内容；
    * txt 文件中的内容可能包含一些专业术语或者缩写，需要根据上下文进行合理的翻译，确保翻译结果准确且易于理解；
    * txt 文件中的内容可能一段内容被分成了很多行，翻译时需要根据内容分为合理的段落，而不是很多行；
    * 标点符合要符合中文技术文档的要求；
    * 要事无巨细的翻译，翻译时切勿丢掉任何内容;

6. [重要]将翻译后的内容同步到 srt 字幕文件中，确保字幕文件中的时间轴和翻译后的文本内容一致。

## 举例

假设用户传入了一个视频资源 "C:\Users\JimmyLi\Videos\12345.mp4"，那么 Skill 会先使用 ffmpeg 将其转换成 mp3 音频文件 "C:\Users\JimmyLi\Videos\12345.mp3"，然后使用 Whisper 将 mp3 文件解析成字幕文件 "C:\Users\JimmyLi\Videos\12345.mp3.srt" 和全文本 "C:\Users\JimmyLi\Videos\12345.mp3.txt"，用户可以根据需要选择使用哪个文件，最后 Skill 会将 "C:\Users\JimmyLi\Videos\12345.mp3.txt" 中的内容翻译成中文并输出在控制台上，同时保存到 "C:\Users\JimmyLi\Videos\12345_翻译.txt" 文件中，12345.mp3.srt 字幕文件中的内容也替换成了中文，并且没有更改时间轴。

```
ffmpeg -i "C:\Users\JimmyLi\Videos\12345.mp4" -vn "C:\Users\JimmyLi\Videos\12345.mp3"

whisper-cli -m ggml-medium-q8_0.bin  -t 8 -p 1 -fa -nf "C:\Users\JimmyLi\Videos\12345.mp3" -osrt -otxt
```