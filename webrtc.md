webrtc库

# 获取接收到的track

```
webrtc::PeerConnectionObserver::OnAddTrack
```

# 视频数据接收

webrtc::internal::Call::Call收到rtp后，通过webrtc::RtpStreamReceiverController::RtpStreamReceiverController进行rtp包的解复用webrtc::RtpDemuxer::OnRtpPacket。

解复用后，把数据传递给webrtc::RtpVideoStreamReceiver2::OnRtpPacket。RtpVideoStreamReceiver2负责视频帧的重新拼装。并把拼装后结果返回给webrtc::internal::VideoReceiveStream2::OnCompleteFrame

webrtc::internal::Call::Call创建webrtc::RtpStreamReceiverController对象和VideoReceiveStream2对象。VideoReceiveStream2创建RtpVideoStreamReceiver2对象，并把RtpVideoStreamReceiver2注册到webrtc::RtpStreamReceiverController。这样webrtc::RtpStreamReceiverController通过webrtc::RtpDemuxer直接与RtpVideoStreamReceiver2传递数据。

webrtc::internal::VideoReceiveStream2::CreateAndRegisterExternalDecoder创建解码器，并把解码器注册到VideoReceiver2::RegisterExternalDecoder，变量video_receiver里。video_receiver没有解码器，则会创建。

webrtc::internal::VideoReceiveStream2::OnCompleteFrame把重组后的编码帧发给VideoStreamBufferController 变量为buffer_, 当可以解一帧时，再把编码帧传递给VideoReceiveStream2::OnEncodedFrame。VideoReceiveStream2::OnEncodedFrame再把帧传递给VideoReceiver2变量为 video_receiver。

# 视频数据发送

采集的帧通过rtc::VideoBroadcaster::OnFrame传递到webrtc::VideoStreamEncoder::OnFrame。如果编码没有准备好就会缓存到pending_frame_ （webrtc::VideoStreamEncoder::MaybeEncodeVideoFrame）,准备好后会调用webrtc::VideoStreamEncoder::EncodeVideoFrame，如果有缓存的pending_frame_则会对它进行编码。

编码器得到编码帧后，调用到VideoStreamEncoder::OnEncodedImage。通过sink_->OnEncodedImage传递到VideoSendStreamImpl::OnEncodedImage，最后通过RtpVideoSender::OnEncodedImage发送出去。

VideoSendStreamImpl::VideoSendStreamImpl会创建RtpVideoSender，并把它设置到VideoStreamEncoder。

编码器的创建是在webrtc::VideoStreamEncoder::ReconfigureEncoder。

# 音频数据接收

RtpDemuxer收到数据后，会传递给webrtc::voe::(anonymous namespace)::ChannelReceive::OnRtpPacket。ChannelReceive会把数据给webrtc::NetEqImpl调用解码器进行解码webrtc::AudioDecoderOpusImpl::ParsePayload。

ChannelReceive可以进行播放控制playing_ 和得到rtp向sdp的映射表 payload_type_map_。

# 音频数据发送

音频编码器编译出数据后调用packetization_callback_->SendData，即webrtc::voe::(anonymous namespace)::ChannelSend::SendData发送数据.

# 详细代码

## 视频数据接收

```
#6  0x0000555557ad6e69 in webrtc::RtpVideoStreamReceiver2::OnCompleteFrames(absl::InlinedVector<std::__Cr::unique_ptr<webrtc::RtpFrameObject, std::__Cr::default_delete<webrtc::RtpFrameObject> >, 3ul, std::__Cr::allocator<std::__Cr::unique_ptr<webrtc::RtpFrameObject, std::__Cr::default_delete<webrtc::RtpFrameObject> > > >) (this=0x7fffe00443b8, frames=<incomplete type>)
    at ../../video/rtp_video_stream_receiver2.cc:967
    
void RtpVideoStreamReceiver2::OnCompleteFrames(
    RtpFrameReferenceFinder::ReturnVector frames)收到数据后通过complete_frame_callback_->OnCompleteFrame进行处理
    
complete_frame_callback_是webrtc::internal::VideoReceiveStream2::OnCompleteFrame(std::__Cr::unique_ptr<webrtc::EncodedFrame, std::__Cr::default_delete<webrtc::EncodedFrame> >)
    (this=0x7fffe0043720, frame=...) at ../../video/video_receive_stream2.cc:718

webrtc::internal::VideoReceiveStream2处理了playout上的东本比如延时


#6  0x0000555557ad5d27 in webrtc::RtpVideoStreamReceiver2::OnRtpPacket(webrtc::RtpPacketReceived const&) (this=0x7fffe00443f8, packet=...) at ../../video/rtp_video_stream_receiver2.cc:766
#7  0x0000555557817326 in webrtc::RtpDemuxer::OnRtpPacket(webrtc::RtpPacketReceived const&) (this=0x7fffe00185b0, packet=...) at ../../call/rtp_demuxer.cc:271


#0  webrtc::RtpDemuxer::RtpDemuxer(bool) (this=0x7fffe0018438, use_mid=false) at ../../call/rtp_demuxer.cc:110
#1  0x0000555557828424 in webrtc::RtpStreamReceiverController::RtpStreamReceiverController() (this=0x7fffe00183d8) at ../../call/rtp_stream_receiver_controller.h:77
#2  0x000055555773a966 in webrtc::internal::Call::Call(webrtc::CallConfig, std::__Cr::unique_ptr<webrtc::RtpTransportControllerSendInterface, std::__Cr::default_delete<webrtc::RtpTransportControllerSendInterface> >) (this=0x7fffe00181b0, config=..., transport_send=...) at ../../call/call.cc:707
在call.cc webrtc::internal::Call::Call()有  RtpStreamReceiverController& receiver_controller =
      media_type == MediaType::AUDIO ? audio_receiver_controller_
                                     : video_receiver_controller_;



  VideoReceiveStream2(const Environment& env,
                      Call* call,
                      int num_cpu_cores,
                      PacketRouter* packet_router,
                      VideoReceiveStreamInterface::Config config,
                      CallStats* call_stats,
                      std::unique_ptr<VCMTiming> timing,
                      NackPeriodicProcessor* nack_periodic_processor,
                      DecodeSynchronizer* decode_sync);
                      
  VideoReceiveStream2* receive_stream = new VideoReceiveStream2(
      env_, this, num_cpu_cores_, transport_send_->packet_router(),
      std::move(configuration), call_stats_.get(),
      std::make_unique<VCMTiming>(&env_.clock(), trials()),
      &nack_periodic_processor_, decode_sync_.get());                      
```

```
webrtc::VideoReceiveStreamInterface* Call::CreateVideoReceiveStream(
    webrtc::VideoReceiveStreamInterface::Config configuration) {
  TRACE_EVENT0("webrtc", "Call::CreateVideoReceiveStream");
  RTC_DCHECK_RUN_ON(worker_thread_);

  EnsureStarted();

  env_.event_log().Log(std::make_unique<RtcEventVideoReceiveStreamConfig>(
      CreateRtcLogStreamConfig(configuration)));

  // TODO(bugs.webrtc.org/11993): Move the registration between `receive_stream`
  // and `video_receiver_controller_` out of VideoReceiveStream2 construction
  // and set it up asynchronously on the network thread (the registration and
  // `video_receiver_controller_` need to l ive on the network thread).
  // TODO(crbug.com/1381982): Re-enable decode synchronizer once the Chromium
  // API has adapted to the new Metronome interface.
  VideoReceiveStream2* receive_stream = new VideoReceiveStream2(
      env_, this, num_cpu_cores_, transport_send_->packet_router(),
      std::move(configuration), call_stats_.get(),
      std::make_unique<VCMTiming>(&env_.clock(), trials()),
      &nack_periodic_processor_, decode_sync_.get());
  // TODO(bugs.webrtc.org/11993): Set this up asynchronously on the network
  // thread.
  receive_stream->RegisterWithTransport(&video_receiver_controller_);
  video_receive_streams_.insert(receive_stream);
```

```
      rtp_video_stream_receiver_(env_,
                                 call->worker_thread(),
                                 &transport_adapter_,
                                 call_stats->AsRtcpRttStats(),
                                 packet_router,
                                 &config_,
                                 rtp_receive_statistics_.get(),
                                 &stats_proxy_,
                                 &stats_proxy_,
                                 nack_periodic_processor,
                                 this,  // OnCompleteFrameCallback
                                 std::move(config_.frame_decryptor),
                                 std::move(config_.frame_transformer))
                                 
                                 
void VideoReceiveStream2::RegisterWithTransport(
    RtpStreamReceiverControllerInterface* receiver_controller) {
  RTC_DCHECK_RUN_ON(&packet_sequence_checker_);
  RTC_DCHECK(!media_receiver_);
  RTC_DCHECK(!rtx_receiver_);
  receiver_controller_ = receiver_controller;

  // Register with RtpStreamReceiverController.
  media_receiver_ = receiver_controller->CreateReceiver(
      remote_ssrc(), &rtp_video_stream_receiver_);
  if (rtx_ssrc() && rtx_receive_stream_ != nullptr) {
    rtx_receiver_ = receiver_controller->CreateReceiver(
        rtx_ssrc(), rtx_receive_stream_.get());
  }
}                       

从video_receiver_controller_收到数据后传递给rtp_video_stream_receiver_，就就是webrtc::RtpDemuxer::OnRtpPacket调用webrtc::RtpVideoStreamReceiver2::OnRtpPacket的原因。而没有VideoReceiveStream2但对象确实在VideoReceiveStream2中创建。

RtpVideoStreamReceiver2把重组好的包传递给webrtc::internal::VideoReceiveStream2::OnCompleteFrame。再调用std::optional<int64_t> VideoStreamBufferController::InsertFrame(
    std::unique_ptr<EncodedFrame> frame) {
```

## 视频数据发送

```
SctMrtcH264EncoderImpl::Encode
  -> encoded_image_callback_->OnEncodedImage(encoded_images_[i],&codec_specific)
  -> VideoStreamEncoder::OnEncodedImage
   ->   EncodedImageCallback::Result result =  sink_->OnEncodedImage(image_copy, codec_specific_info);
   -> VideoSendStreamImpl::OnEncodedImage
   -> RtpVideoSender::OnEncodedImage
   

webrtc::SctMrtcH264EncoderImpl::RegisterEncodeCompleteCallback
  <- webrtc::VideoStreamEncoder::ReconfigureEncoder()
  
VideoStreamEncoder::SetSink
  <- webrtc::internal::VideoSendStreamImpl::VideoSendStreamImpl
  
VideoSendStreamImpl::VideoSendStreamImpl
  -> rtp_video_sender_(transport->CreateRtpVideoSender())
```

### 视频采集到编码

```
webrtc::SctMrtcH264EncoderImpl::Encode
 <- webrtc::VideoStreamEncoder::EncodeVideoFrame  video/video_stream_encoder.cc:2047
 <- VideoStreamEncoder::OnBitrateUpdated   如果有pending_frame_，则会编码发送
 <- webrtc::internal::VideoSendStreamImpl::OnBitrateUpdated  video/video_send_stream_impl.cc:942
 
VideoStreamEncoder::EncodeVideoFrame
 <- webrtc::VideoStreamEncoder::MaybeEncodeVideoFrame  可能会配置pending_frame_  video/video_stream_encoder.cc
 <- webrtc::VideoStreamEncoder::OnFrame
 <- webrtc::VideoStreamEncoder::CadenceCallback::OnFrame  调整帧时间信息
 <- webrtc::(anonymous namespace)::PassthroughAdapterMode::OnFrame   video/frame_cadence_adapter.cc:79
 <- FrameCadenceAdapterImpl::OnFrameOnMainQueue
 <- webrtc::(anonymous namespace)::FrameCadenceAdapterImpl::OnFrame(webrtc::VideoFrame const&)
 <- rtc::VideoBroadcaster::OnFrame(webrtc::VideoFrame const&)  media/base/video_broadcaster.cc
 <- webrtc::test::TestVideoCapturer::OnFrame(webrtc::VideoFrame const&)
 <- webrtc::test::SctMrtcFrameGeneratorCapturer::InsertFrame()  mrtc/framegeneratorcapturer.cpp:113
 
webrtc::VideoStreamEncoder::VideoStreamEncoder
 <- webrtc::internal::(anonymous namespace)::CreateVideoStreamEncoder   video/video_send_stream_impl.cc:355
 <- webrtc::internal::VideoSendStreamImpl::VideoSendStreamImpl  video/video_send_stream_impl.cc:411
 <- webrtc::internal::Call::CreateVideoSendStream  call/call.cc:970
 <- cricket::WebRtcVideoSendChannel::WebRtcVideoSendStream::RecreateWebRtcStream()   media/engine/webrtc_video_engine.cc:2648
 <- cricket::WebRtcVideoSendChannel::WebRtcVideoSendStream::SetCodec  media/engine/webrtc_video_engine.cc:2034
 <- cricket::WebRtcVideoSendChannel::WebRtcVideoSendStream::SetSenderParameters
 <- cricket::WebRtcVideoSendChannel::ApplyChangedParams(cricket::WebRtcVideoSendChannel::ChangedSenderParameters const&)
 <- cricket::WebRtcVideoSendChannel::SetSenderParameters(cricket::VideoSenderParameters const&)
 <- cricket::VideoChannel::SetRemoteContent_w   pc/channel.cc:1227
 <- cricket::BaseChannel::SetRemoteContent 
 <- webrtc::SdpOfferAnswerHandler::PushdownMediaDescription   pc/sdp_offer_answer.cc:4938
 
RtpPayloadParams::GetRtpVideoHeader
  <- RtpVideoSender::OnEncodedImage
  <- VideoStreamEncoder::OnEncodedImage
 
```

## 音频数据接收

```
webrtc::AudioDecoder::Decode   api/audio_codecs/audio_decoder.cc:93
 <- webrtc::OpusFrame::Decode  modules/audio_coding/codecs/opus/audio_coder_opus_common.h:66 
 <- webrtc::NetEqImpl::DecodeLoop  modules/audio_coding/neteq/neteq_impl.cc:1428
 <- webrtc::NetEqImpl::Decode
 <- NetEqImpl::GetAudioInternal
 <- NetEqImpl::GetAudio
 <- webrtc::voe::(anonymous namespace)::ChannelReceive::GetAudioFrameWithInfo
 <- ChannelReceive::GetAudioFrameWithInfo
 <- AudioReceiveStreamImpl::GetAudioFrameWithInfo
 这里往下是播放了
 
webrtc::OpusFrame::OpusFrame modules/audio_coding/codecs/opus/audio_coder_opus_common.h:42
 <- webrtc::AudioDecoderOpusImpl::ParsePayload audio_decoder_opus.cc:57
 <- webrtc::NetEqImpl::InsertPacketInternal(webrtc::RTPHeader const&, rtc::ArrayView<unsigned char const, -4711l>, webrtc::RtpPacketInfo const&)  modules/audio_coding/neteq/neteq_impl.cc:607
 把包都缓存在packet_buffer_了。
 
 <- webrtc::NetEqImpl::InsertPacket(webrtc::RTPHeader const&, rtc::ArrayView<unsigned char const, -4711l>, webrtc::RtpPacketInfo const&) modules/audio_coding/neteq/neteq_impl.cc:191
 <- webrtc::voe::(anonymous namespace)::ChannelReceive::OnReceivedPayloadData  audio/channel_receive.cc:379
 <- webrtc::voe::(anonymous namespace)::ChannelReceive::ReceivePacket audio/channel_receive.cc:777
 <- webrtc::voe::(anonymous namespace)::ChannelReceive::OnRtpPacket( audio/channel_receive.cc:717
 <- webrtc::RtpDemuxer::OnRtpPacket  call/rtp_demuxer.cc:271
 这里开始是rtp处理
 
 
webrtc::AudioDecoderOpus::MakeAudioDecoder api/audio_codecs/opus/audio_decoder_opus.h:43
 <- webrtc::audio_decoder_factory_template_impl::CreateDecoder  api/audio_codecs/audio_decoder_factory_template.h:67
 <- webrtc::audio_decoder_factory_template_impl::Helper api/audio_codecs/audio_decoder_factory_template.h:107
 <- webrtc::audio_decoder_factory_template_impl::AudioDecoderFactoryT audio_decoder_factory_template.h:129
 <- webrtc::DecoderDatabase::DecoderInfo::GetDecoder() const  modules/audio_coding/neteq/decoder_database.cc:66
 <- webrtc::DecoderDatabase::DecoderInfo::SampleRateHz() const modules/audio_coding/neteq/decoder_database.h:62
 这里创建decoder. sdp中有sample rate 配置。webrtc::DecoderDatabase::DecoderInfo中有decoder所有配置信息。
...
 <- webrtc::NetEqImpl::InsertPacketInternal modules/audio_coding/neteq/neteq_impl.cc:488
```

## 音频数据的发送

```
webrtc::AudioEncoderOpusImpl::EncodeImpl modules/audio_coding/codecs/opus/audio_encoder_opus.cc:585
 <- webrtc::AudioEncoder::Encode api/audio_codecs/audio_encoder.cc:53
 <- webrtc::(anonymous namespace)::AudioCodingModuleImpl::Encode modules/audio_coding/acm2/audio_coding_module.cc:238
 <- webrtc::(anonymous namespace)::AudioCodingModuleImpl::Add10MsData modules/audio_coding/acm2/audio_coding_module.cc:325 
 <- webrtc::voe::(anonymous namespace)::ChannelSend::ProcessAndEncodeAudio audio/channel_send.cc:870
 <- webrtc::internal::AudioSendStream::SendAudioData  audio/audio_send_stream.cc:386
 <- webrtc::AudioTransportImpl::SendProcessedData  audio/audio_transport_impl.cc:200
 <- webrtc::AudioTransportImpl::RecordedDataIsAvailable audio/audio_transport_impl.cc:180
 number_of_frames=441, bytes_per_sample=2, number_of_channels=1, sample_rate=44100, audio_delay_milliseconds=11
 <- webrtc::AudioDeviceBuffer::Del iverRecordedData() modules/audio_device/audio_device_buffer.cc:318
 <- webrtc::AudioDeviceLinuxPulse::ProcessRecordedData modules/audio_device/linux/audio_device_pulse_linux.cc:1968
 <- webrtc::AudioDeviceLinuxPulse::ReadRecordedData(void const*, unsigned long)   modules/audio_device/linux/audio_device_pulse_linux.cc:1915
 
 
AudioCodingModuleImpl::Encode
 -> packetization_callback_->SendData
 -> webrtc::voe::(anonymous namespace)::ChannelSend::SendData
 -> ChannelSend::SendRtpAudio
 
 
ChannelSend::ChannelSend
 创建编码器，并把回调设置为自己
 -> audio_coding_->RegisterTransportCallback(this)
 
AudioEncoderOpusImpl::AudioEncoderOpusImpl()
 -> webrtc::AudioEncoderOpus::MakeAudioEncoder  audio_codecs/opus/audio_encoder_opus.cc:50
 -> webrtc::audio_encoder_factory_template_impl::CreateEncoder<webrtc::AudioEncoderOpus, void>  api/audio_codecs/audio_encoder_factory_template.h:68
 -> webrtc::internal::AudioSendStream::SetupSendCodec(webrtc::AudioSendStream::Config const&) audio/audio_send_stream.cc:566
 -> webrtc::internal::AudioSendStream::ReconfigureSendCodec   audio/audio_send_stream.cc:657
 -> AudioSendStream::ConfigureStream audio/audio_send_stream.cc:305
 -> webrtc::internal::AudioSendStream::Reconfigure  audio/audio_send_stream.cc:180
 -> cricket::WebRtcVoiceSendChannel::WebRtcAudioSendStream::ReconfigureAudioSendStream  media/engine/webrtc_voice_engine.cc:1235
 -> cricket::WebRtcVoiceSendChannel::WebRtcAudioSendStream::SetSendCodecSpec media/engine/webrtc_voice_engine.cc:906
 -> cricket::WebRtcVoiceSendChannel::SetSendCodecs media/engine/webrtc_voice_engine.cc:1523
// Scan through the list to figure out the codec to use for sending. 
 -> cricket::WebRtcVoiceSendChannel::SetSenderParameters media/engine/webrtc_voice_engine.cc:1351
webrtc_voice_engine.cc:1319: WebRtcVoiceMediaChannel::SetSenderParameters: {codecs: [AudioCodec[111:opus:48000:0:2], AudioCodec[63:red:48000:0:2
], AudioCodec[110:telephone-event:48000:0:1]], extensions: [{uri: http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01, id: 3}, {uri: http://www.webrtc.org/experiments/r
tp-hdrext/abs-send-time, id: 2}, {uri: urn:ietf:params:rtp-hdrext:sdes:mid, id: 4}, {uri: urn:ietf:params:rtp-hdrext:ssrc-audio-level, id: 1}], extmap-allow-mixed: true, max_bandwidth_bps: -1, mid: 0, options: AudioOptions {}, rtcp: {reduced_size:true, remote_estimate:false}}

 -> cricket::VoiceChannel::SetRemoteContent_w pc/channel.cc:967
 -> cricket::BaseChannel::SetRemoteContent pc/channel.cc:291
 

AudioEncoderOpusImpl::SdpToConfig
这里已经保证opus编码器是48000Hz和双声道
 <- webrtc::AudioEncoderOpusImpl::AppendSupportedEncoders modules/audio_coding/codecs/opus/audio_encoder_opus.cc:215
 <- webrtc::AudioEncoderOpus::AppendSupportedEncoders api/audio_codecs/opus/audio_encoder_opus.cc:34
 <- webrtc::audio_encoder_factory_template_impl::Helper
 <- webrtc::audio_encoder_factory_template_impl::AudioEncoderFactoryT
 <- cricket::WebRtcVoiceEngine::Init media/engine/webrtc_voice_engine.cc:508
 
webrtc::CreateOpusAudioDecoderFactory()
 -> AudioDecoderOpus::AppendSupportedDecoders
 AudioDecoderOpus::SdpToConfig
 这里已经保证opus解码编码器是48000Hz和双声道
 
 
webrtc::AudioDeviceLinuxPulse::AudioDeviceLinuxPulse
 <- webrtc::AudioDeviceModuleImpl::CreatePlatformSpecificObjects
 <- webrtc::AudioDeviceModule::CreateForTest
 <- webrtc::AudioDeviceModule::Create
 <- cricket::WebRtcVoiceEngine::Init    media/engine/webrtc_voice_engine.cc:525
 <- cricket::CompositeMediaEngine::Init
 <- webrtc::ConnectionContext::ConnectionContext pc/connection_context.cc:175
 
webrtc::AudioDeviceLinuxPulse::AttachAudioBuffer(webrtc::AudioDeviceBuffer*)
 <- webrtc::AudioDeviceModuleImpl::AttachAudioBuffer modules/audio_device/audio_device_impl.cc:274
 
AudioDeviceModuleImpl::RegisterAudioCallback
 <- cricket::WebRtcVoiceEngine::Init() media/engine/webrtc_voice_engine.cc:551
 
cricket::WebRtcVoiceEngine::WebRtcVoiceEngine media/engine/webrtc_voice_engine.cc:456
如果adm为空，就会创建默认的音频设备。
 <- webrtc::(anonymous namespace)::MediaFactoryImpl::CreateMediaEngine api/enable_media.cc:57
 adm是由deps.adm传入的参数。
 <- webrtc::ConnectionContext::ConnectionContext pc/connection_context.cc:104
 <- webrtc::ConnectionContext::Create pc/connection_context.cc:83
 <- webrtc::PeerConnectionFactory::Create    pc/peer_connection_factory.cc:81
 <- webrtc::CreateModularPeerConnectionFactory    pc/peer_connection_factory.cc:67
 <- sct::mrtc::PeerConnection::onInit(  
```

