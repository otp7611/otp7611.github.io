rtmp和传输h265视频

参考

标准版rtmp rtmp_specification_1.0.pdf

加强版rtmp https://veovera.org/docs/enhanced/enhanced-rtmp-v1

# 加强版rtmp

实践中adobe rtmp传输的一般是h264+aac,所以扩展此协议时，只要在传输h264格式时adobe规定的格式上扩展就可以了。

rtmp协议和flv一样，负载其实是各种tag.加强版rtmp自定义了新的tag就是ExVideoTagHeader，它可以与标准的rtmp VideoTagHeader兼容，准确来讲是与标准的rtmp传输h264时格式兼容。

## ExVideoTagHeader

在实践中，改造现有rtmp到加强版rtmp，只需要处理好ExVideoTagHeader就行了。其它不用改，因为它是兼容标准的rtmp。

参考https://veovera.org/docs/enhanced/enhanced-rtmp-v1#defining-additional-video-codecs

## VideoTagHeader与ExVideoTagHeader对比

|                 | VideoTagHeader（H264）                         | ExVideoTagHeader（HEVC）                               |
| --------------- | ---------------------------------------------- | ------------------------------------------------------ |
| 长度            | 包括了3字节的[CompositionTime Offset]，共5字节 | 不包括包括3字节的[CompositionTime Offset]，它也是5字节 |
| FrameType       | 枚举值相同                                     | 枚举值相同                                             |
| 编码器类型      | CodecId（4比特）                               | Video FourCC （32比特）                                |
| PacketType      | AVCPacketType                                  | PacketType                                             |
| CompositionTime | 在头部                                         | 不在头部                                               |

注意，VideoTagHeader是包括CompositionTime Offset。

# 生成AVCDecoderConfigurationRecord

```
ff_isom_write_avcc
```

# 生成HEVCDecoderConfigurationRecord

仔细研究就会发现rtmp传输的音频和视频编码格式与mp4上一样的。所以h265在rtmp上传输时，就是会用到HEVCDecoderConfigurationRecord。但是从编码器拿到了数据一般都是annexB格式。所以就需要去生成这个东西。参考[视频编码初探](/media)

这个是二进制的东西，自己搞太麻烦了，用ffmpeg吧。

```
extern "C" {
int ff_isom_write_hvcc(AVIOContext *pb, const uint8_t *data,
                       int size, int ps_array_completeness);
}
```

ff_isom_write_hvcc这个参数并没有暴露出来，直接用extern就可以了。



