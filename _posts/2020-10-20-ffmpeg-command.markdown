---
layout:     post
title:      "FFmpeg常用命令"
subtitle:   "ffmpeg version 4.3"
date:       2020-10-20 09:27:55
author:     "Z1hgq"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - 音视频
    - FFmpeg
    - shell
---

> “生活可以没有诗歌，但不能没有诗意；行进中可以没有路，但不能没有前进的脚步；工作中可以没有经验，但不能没有学习；人生中可以没有闪光，但不能有污迹。”

---

## 文件操作

1、在.mp3（320 kbps）上保持高质量，并将.flac文件中的元数据转换为ID3v2格式

```shell
ffmpeg -i input.flac -ab 320k -map_metadata 0 -id3v2_version 3 output.mp3
```

2、Convert MP3 to FLV with ffmpeg

```shell
ffmpeg -y -i audio.mp3 -f flv -acodec libmp3lame -ab 160k -ac 1 audio.flv
```

3、分离视频音频流

```shell
ffmpeg -i input_file -vcodec copy -an output_file_video　　//分离视频流
ffmpeg -i input_file -acodec copy -vn output_file_audio　　//分离音频流
```

4、视频解复用

```shell
ffmpeg –i test.mp4 –vcodec copy –an –f m4v test.264
ffmpeg –i test.avi –vcodec copy –an –f m4v test.264
```

5、视频转码

```shell
ffmpeg –i test.mp4 –vcodec h264 –s 352*278 –an –f m4v test.264              //转码为码流原始文件
ffmpeg –i test.mp4 –vcodec h264 –bf 0 –g 25 –s 352*278 –an –f m4v test.264  //转码为码流原始文件
ffmpeg –i test.avi -vcodec mpeg4 –vtag xvid –qsame test_xvid.avi            //转码为封装文件
//-bf B帧数目控制，-g 关键帧间隔控制，-s 分辨率控制
```

6、视频封装

```shell
ffmpeg –i video_file –i audio_file –vcodec copy –acodec copy output_file
```

7、视频录制

```shell
ffmpeg –i rtsp://192.168.3.205:5555/test –vcodec copy out.avi
```

8、YUV序列播放

```shell
ffplay -f rawvideo -video_size 1920x1080 input.yuv
```

9、YUV序列转AVI

```shell
ffmpeg –s w*h –pix_fmt yuv420p –i input.yuv –vcodec mpeg4 output.avi
```
**主要参数**： -i 设定输入流 -f 设定输出格式 -ss 开始时间 视频参数： -b 设定视频流量，默认为200Kbit/s -r 设定帧速率，默认为25 -s 设定画面的宽与高 -aspect 设定画面的比例 -vn 不处理视频 -vcodec 设定视频编解码器，未设定时则使用与输入流相同的编解码器 音频参数： -ar 设定采样率 -ac 设定声音的Channel数 -acodec 设定声音编解码器，未设定时则使用与输入流相同的编解码器 -an 不处理音频

## 采集直播

1、将文件当做直播送至live

```shell
ffmpeg -re -i localFile.mp4 -c copy -f flv rtmp://server/live/streamName
```

2、将直播媒体保存至本地文件

```shell
ffmpeg -i rtmp://server/live/streamName -c copy dump.flv
```

3、将其中一个直播流，视频改用h264压缩，音频不变，送至另外一个直播服务流

```shell
ffmpeg -i rtmp://server/live/originalStream -c:a copy -c:v libx264 -vpre slow -f flv rtmp://server/live/h264Stream
``` 

4、将其中一个直播流，视频改用h264压缩，音频改用faac压缩，送至另外一个直播服务流

```shell
ffmpeg -i rtmp://server/live/originalStream -c:a libfaac -ar 44100 -ab 48k -c:v libx264 -vpre slow -vpre baseline -f flv rtmp://server/live/h264Stream
```

5、将其中一个直播流，视频不变，音频改用faac压缩，送至另外一个直播服务流

```shell
ffmpeg -i rtmp://server/live/originalStream -acodec libfaac -ar 44100 -ab 48k -vcodec copy -f flv rtmp://server/live/h264_AAC_Stream
```

6、将一个高清流，复制为几个不同视频清晰度的流重新发布，其中音频不变

```shell
ffmpeg -re -i rtmp://server/live/high_FMLE_stream -acodec copy -vcodec x264lib -s 640×360 -b 500k -vpre medium -vpre baseline rtmp://server/live/baseline_500k -acodec copy -vcodec x264lib -s 480×272 -b 300k -vpre medium -vpre baseline rtmp://server/live/baseline_300k -acodec copy -vcodec x264lib -s 320×200 -b 150k -vpre medium -vpre baseline rtmp://server/live/baseline_150k -acodec libfaac -vn -ab 48k rtmp://server/live/audio_only_AAC_48k
```

7、功能一样，只是采用-x264opts选项

```shell
ffmpeg -re -i rtmp://server/live/high_FMLE_stream -c:a copy -c:v x264lib -s 640×360 -x264opts bitrate=500:profile=baseline:preset=slow rtmp://server/live/baseline_500k -c:a copy -c:v x264lib -s 480×272 -x264opts bitrate=300:profile=baseline:preset=slow rtmp://server/live/baseline_300k -c:a copy -c:v x264lib -s 320×200 -x264opts bitrate=150:profile=baseline:preset=slow rtmp://server/live/baseline_150k -c:a libfaac -vn -b:a 48k rtmp://server/live/audio_only_AAC_48k
```

8、将当前摄像头及音频通过DSSHOW采集，视频h264、音频faac压缩后发布

```shell
ffmpeg -r 25 -f dshow -s 640×480 -i video=”video source name”:audio=”audio source name” -vcodec libx264 -b 600k -vpre slow -acodec libfaac -ab 128k -f flv rtmp://server/application/stream_name
```

9、将一个JPG图片经过h264压缩循环输出为mp4视频

```shell
ffmpeg.exe -i INPUT.jpg -an -vcodec libx264 -coder 1 -flags +loop -cmp +chroma -subq 10 -qcomp 0.6 -qmin 10 -qmax 51 -qdiff 4 -flags2 +dct8x8 -trellis 2 -partitions +parti8x8+parti4x4 -crf 24 -threads 0 -r 25 -g 25 -y OUTPUT.mp4
```

10、将普通流视频改用h264压缩，音频不变，送至高清流服务(新版本FMS live=1)

```shell
ffmpeg -i rtmp://server/live/originalStream -c:a copy -c:v libx264 -vpre slow -f flv “rtmp://server/live/h264Stream live=1〃
```
