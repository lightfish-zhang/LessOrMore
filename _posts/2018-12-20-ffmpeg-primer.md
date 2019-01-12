---
layout: post
title: ffmpeg音视频编程入门
date:   2018-12-20 20:30:00 +0800
categories: ffmpeg
tag: [codec]
---


* content
{:toc}

## 前言

本文以 ffmpeg 工具，讲述如何认识音视频编程，你可以了解到常见视频格式的大概样子，一步步学会如何使用 ffmpeg 的 C 语言 API

本文重于动手实践，代码仓库：[mpegUtil](https://github.com/lightfish-zhang/mpegUtil)

笔者的开发环境：Arch Linux 4.19.12, ffmpeg version n4.1

## 编码过程总览

以下是解码流程图，逆向即是编码流程

![](https://raw.githubusercontent.com/lightfish-zhang/mpegUtil/master/doc/decode_process.png)

本文是音视频编程入门篇，先略过传输协议层，主要讲格式层与编解码层的编程例子。

### 第一步，查看视频文件信息

料理食材的第一步，得先懂得食材的来源和特性。

- 来源，互联网在线观看（http/rtmp）、播放设备上存储的视频文件（file）。
- 格式，如何查看视频文件的格式呢，以下有 unix 命令行示例，至于 windows 系统，查看文件属性即可。


```shell
# linux 上查看视频文件信息
ffmpeg -i example.mp4
```

以某个mp4文件为例，输出：

```
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'example.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    encoder         : Wxmm_900012345
  Duration: 00:00:58.21, start: 0.000000, bitrate: 541 kb/s
    Stream #0:0(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p, 368x640, 487 kb/s, 24 fps, 24 tbr, 12288 tbn, 48 tbc (default)
    Metadata:
      handler_name    : VideoHandler
    Stream #0:1(und): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 48 kb/s (default)
    Metadata:
      handler_name    : SoundHandler
```

根据命令输出信息，视频文件中有两个 stream, 即 video 与 audio，视频流与音频流。

- stream 0, 是视频数据，编码格式为h264，24 fps 意为 24 frame per second，即每秒24帧，比特率487 kb/s,
- stream 1, 是音频数据，编码格式为acc，采样率44100 Hz，比特率48 kb/s

#### 【编程实操】读取音视频流的格式信息

在互联网场景中，在线观看视频才是常见需求，那么，计算机如何读取视频流的信息呢，下面以 ffmpeg 代码讲述

```c
    /* C代码例子，省略了处理错误的逻辑 */

    AVFormatContext *fmt_ctx = NULL; // AV 格式上下文
    AVIOContext *avio_ctx = NULL; // AV IO 上下文
    unsigned char *avio_ctx_buffer = NULL; // input buffer

    fmt_ctx = avformat_alloc_context();// 获得 AV format 句柄
    avio_ctx_buffer = (unsigned char *)av_malloc(data_size); // ffmpeg分配内存的方法，给输入分配缓存

    /* fread(file) or memcpy(avio_ctx_buffer, inBuf, n)  省略拷贝文件流的步骤 */

    // 给 av format context 分配 io 操作的句柄，必要传参：输入数据的指针、数据大小、write_flag＝０表明buffer不可写，其他参数忽略，置 NULL
    avio_ctx = avio_alloc_context(avio_ctx_buffer, data_size, 0, NULL, NULL, NULL, NULL);
    fmt_ctx->pb = avio_ctx;

    // 打开输入的数据流，读取 header 的格式内容，注意必须的后续处理 avformat_close_input()
    avformat_open_input(&fmt_ctx, NULL, NULL, NULL)

    // 获取音视频流的信息
    avformat_find_stream_info(fmt_ctx, NULL) 

```

ffmpeg 有一个方法直接打印音视频信息

```c
av_dump_format(fmt_ctx, 0, NULL, 0);
```

print: (源码的输出格式凌乱，笔者整理过)

```
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from '(null)':
Stream #0:0 : Video: h264 (High) (avc1 / 0x31637661), yuv420p, 368x640, 487 kb/s 24 fps,
Stream #0:1 : Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 48 kb/s
```

实践编程，获取音视频信息，当然需要细致地调用API，看下面代码

- 查看音视频流的索引、类型、解码器类型

```c
  avformat_find_stream_info(fmt_ctx, NULL);

  for(int i=0; i<fmt_ctx->nb_streams; i++){
      AVStream *stream = fmt_ctx->streams[i];
      AVCodecParameters *codec_par = stream->codecpar;
      av_log(NULL, AV_LOG_INFO, "find audio stream index=%d, type=%s, codec id=%d", 
        i, av_get_media_type_string(codec_par->codec_type), codec_par->codec_id);
  }
```

print:

```
find audio stream index=0, type=video, codec id=27
find audio stream index=1, type=audio, codec id=86018
```

- 看到没，上面代码只获得解码器的id值(枚举类型)，那么解码器的信息呢，加上下面代码

```c
  AVCodec *decodec = NULL;
  decodec = avcodec_find_decoder(codec_par->codec_id);
  av_log(NULL, AV_LOG_INFO, "find codec name=%s\t%s", decodec->name, decodec->long_name);
```

print:

```
find audio stream index=0, type=video, codec id=27
find codec name=h264    H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10
find audio stream index=1, type=audio, codec id=86018
find codec name=aac     AAC (Advanced Audio Coding)
```

- 关于视频，可以查看帧率(一秒有多少帧画面)

```c
  AVRational framerate = av_guess_frame_rate(fmt_ctx, stream, NULL);
  av_log(NULL, AV_LOG_INFO, "video framerate=%d/%d", framerate.num, framerate.den);
```

print:

```
video framerate=24/1
```

至此，我们掌握了如何利用 ffmpeg 的 C 语言 API 来读取音视频文件流的信息

### 解码



#### 转成 GIF 图片

为了让人有更直观的认识，转出个 gif 图片


## 参考文章

[雷霄骅的博客](https://blog.csdn.net/leixiaohua1020) (致敬英才)

