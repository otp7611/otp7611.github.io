h264视频编码

参考资料：

- h264的mp4封装规范H.264-AVC-ISO_IEC_14496-15
- h264标准文档T-REC-H.264-202108-S!!PDF-E
- https://w3c.github.io/webcodecs/

本文档以webcodec为具体的例子来讲解视频编译器。

# 编码器输出的两种格式

在配置编译器时，可以配置编译码器输出为两种格式，一种是annexB格式，另一种的是AVC格式。h264标准中规定是annexB，就是Annex B Byte stream format。AVC格式定义在h264的mp4封装规范。AVC格式主要用于mp4封装和rtmp传输。对rtmp协议深入分析就会发现，rtmp传输的都是文件存储格式的流和mp4中的流能对应上。

参考文档：

https://w3c.github.io/webcodecs/#dictdef-videoencoderconfig

https://w3c.github.io/webcodecs/codec_registry.html

https://www.w3.org/TR/webcodecs-avc-codec-registration/#avc-bitstream-format

```
enum AvcBitstreamFormat {
  "annexb",
  "avc",
};
```



## 实例

```c++
    const config = {
        codec: "avc1.4d002a",
        width: trackSettings.width,
        height: trackSettings.height,
        framerate: 20,
        bitrate: 200000,
        avc: {
            format: "annexb",
        },
    };
```

这个表示把编码器配置成annexb格式。webcodec中h265默认输出为avc.



## avc格式

https://www.w3.org/TR/webcodecs-avc-codec-registration/#videodecoderconfig-description

如果为avc格式，则编解码参数信息以带外参数返回。带外参数是指和原始编码流不是使用同一个接口。这个参数的格式为AVCDecoderConfigurationRecord。webcodec中就是以VideoDecoderConfig.description返回。而如果是ffmpeg则是对应AVCodecParameters字段extradata内容。

avc格式从编码器输出的编码流格式为

```
|length|nalu|length|nalu|...
```

length占用多少字节在AVCDecoderConfigurationRecord中的lengthSizeMinusOne中规定。如果lengthSizeMinusOne=3则length占用4字节。



