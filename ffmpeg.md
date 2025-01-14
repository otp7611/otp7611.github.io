使用ffmpeg命令行工具

# 转码

https://ffmpeg.org/ffmpeg-filters.html

转码就是从一种格式换为另一种格式，filter模块可以很方便的改变采样格式，另外还能做一些处理，如加水印，去除反相声轨。

filter语句，整体上具有如下规律：

- filter间用逗号(,)分开，filter参数以\<filterName>=<args…..>提供，以第一个＝分隔。参数间以冒号(:)分隔。
- 以逗号分开filter，前一个输出是下一个的输入。
- 一条filter处理管道用分号(;)分开

## 硬件转码

这里描述的硬件指nvidia显卡。

```shell
ffmpeg -t 60 -hwaccel_device 0 -gpu 0 -c:v h264_cuvid -i ~/share/keep/audioVideo.offset.mp4 -i ~/download/0.png -filter_complex '[0:v]hwupload_cuda=device=0,scale_cuda=1920:1080:format=yuv420p[base];[1:v]format=pix_fmts=rgba,hwupload_cuda[overlay_video];[base][overlay_video]overlay_cuda=x=0:y=0' -gpu 0 -c:v h264_nvenc -b:v 2500K -ac 1 -c:a aac -bsf:a aac_adtstoasc -hls_playlist_type vod -hls_flags discont_start -strict -2 -y a.mp4
```

-t 单位为秒，ffmpeg有时间的表示方法，参考https://ffmpeg.org/ffmpeg-utils.html#Time-duration

参数-hwaccel_device 0，参数-gpu 0，参考hwupload_cuda=device=0描述使用哪个gpu核处理。索引应该要保持一致。可以通过nvidia-smi确认。

查看一个进程是用了哪个gpu可以通过nvidia-smi查，如果一个进程同时使用了0，1那么nivdia-smi会输出两个相同pid的记录，一个使用0，一个使用1.

-c:v h264_cuvid 指定解码器，注意如果要指定在哪个gpu核处理，-gpu参数一定要放在-c:v前。

-i 表示输入，允许有多个。在filter中可以通过[0:v]来引用，这里表示使用第一个文件(0)的视频(v)。

-filter_complex 表示是多输入多输出的复杂filter, 它与-vf, -af, 参数是兼容的，也就是说-vf, -af直接换成-filter_complex时，原有命令还是正常的。如果有多个输入，一定要用-filter_complex。

[0:v]使用第一个文件(0)的视频(v)，[1:v]表示使用第二个文件(1)的视频(v)

hwupload_cuda=device=0表示把帧提交到硬件中，软件帧变为硬件帧。hwdownload就是把硬件帧变为软件帧。通过使用hwupload_cuda和hwdownload可以方便在硬件filter和软件filter间做切换。否则软件帧是不能输入到硬件filter中的。这里device=0会产生副作用，表示编码的gpu核也会使用0号核，后面的-gpu 0是无效的。

[base];  filter处理管可以在前后加入输入和输出label, 使得其它filter能够引用。

-c:v h264_nvenc指定编码器

-hls_playlist_type vod表示如果输出为hls，则为回放模式(#EXT-X-ENDLIST结尾)，否则为实时模式

-hls_flags discont_start表示m3u8的第一个ts段要加上#EXT-X-DISCONTINUITY

## 软件转码

```
ffmpeg -t 60 -i ~/share/keep/audioVideo.offset.mp4 -i http://d.lanrentuku.com/down/png/1712/23haidiyulei-png/haidiyuleipng-021.png -filter_complex 'scale=1920:1080,format=pix_fmts=yuv420p[base];[1:v]format=pix_fmts=rgba[logo-in],[logo-in][base]scale2ref=w=oh*mdar:h=ih/10[logo-out][video-out];[video-out][logo-out]overlay=x=0:y=0' -c:v libx264 -b:v 2500K -ac 1 -c:a aac -bsf:a aac_adtstoasc -hls_playlist_type vod -hls_flags discont_start -strict -2 -y a.mp4
```

注意硬件的scale_cuda和软件的scale参数是不一样的。

# 直接播放PCM数据

```shell
ffplay -f f32le -ar 48k -ac 1 frame.flt5
```

# 查看流信息

```shell
ffprobe -show_frames 'a.m3u8' | grep pict_type
```

-of csv表示输出为csv格式

# 从多个输入文件中选择流，合并成新文件

```shell
ffmpeg -i a.mp4 -i v.mp4 -map 0:0 -map 1:0 -c copy my.mp4
```

# 给视频添加pts时间显示

```shell
ffmpeg -y -i ~/share/keep/audioVideo.offset.mp4 -vf "drawtext=fontsize=36:fontcolor=yellow:text='%{pts\:hms}':x=20:y=20" audioVideo.timestamp.mp4
```

# 检测反相

```shell
ffmpeg -i invert.mp4 -af "aphasemeter=video=0:phasing=1:angle=150:duration=1:tolerance=0.001,ametadata=print:file=inwav-phase.txt" -vn -f null -
```

# 增加探测内容范围

```shell
ffprobe -analyzeduration 20M a.mp4
```



# 显示完整的帮助选项

```shell
ffmpeg -h full
```



