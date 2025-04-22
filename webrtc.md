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

