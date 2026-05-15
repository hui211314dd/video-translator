# video-translator-skill

利用 FFmpeg 和 Whisper 提取英文视频或者音频中的语音内容，导出成 txt 文件的流程 (根据用户的选项也可以导出成 srt 字幕文件), 最后要求将 txt 文件中的内容翻译成中文并输出在控制台上;

TODO: 将来考虑把所需的工具比如 ffmpeg、whisper-cli 以及 model 都放在 skill 里面;