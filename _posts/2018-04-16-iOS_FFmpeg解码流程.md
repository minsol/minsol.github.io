---
layout:     post
title:      "iOS-FFmpeg解码H264的流程"
subtitle:   "本文记录在实际开发中FFmpeg解码H264的流程，做以备忘。"
date:       2018-04-16
author:     "minsol"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - 备忘录
---


### 前言
本文记录在实际开发中FFmpeg解码H264的流程
***



### 准备工作
>1. 注册所有容器格式和CODEC:`avcodec_register_all();`
>2. 查找对应的解码器AVCodec:`AVCodec *h264Codec = avcodec_find_decoder(AV_CODEC_ID_H264);`
>3. 通过解码器创建相应的AVCodecContext:`avcodec_alloc_context3(h264Codec);`
>4. 打开编解码器:`avcodec_open2(_h264Ctx, h264Codec, NULL);`
>5. 创建包含码流参数的AVFrame:`avframe = av_frame_alloc();`

```swift
- (void)ffmpegInit{
    avcodec_register_all();

    AVCodec *h264Codec = avcodec_find_decoder(AV_CODEC_ID_H264);
    AVCodec *mpeg4Codec = avcodec_find_decoder(AV_CODEC_ID_MPEG4);

    _h264Ctx = avcodec_alloc_context3(h264Codec);
    _mpeg4Ctx = avcodec_alloc_context3(mpeg4Codec);

    avcodec_open2(_h264Ctx, h264Codec, NULL);
    avcodec_open2(_mpeg4Ctx, mpeg4Codec, NULL);

    _avframe = av_frame_alloc();
}
```


###获取到H264的流数据后进行解码
```swift
unsigned char *frame ------>AVPacket------> AVFrame------>CVPixelBufferRef
```

>1. 创建AVPacket`int av_new_packet(AVPacket *pkt, int size);`得到AVPacket
>2. 将原始分组数据作为输入提供给解码器。`avcodec_send_packet(ctx, &pkt);`
>3. 从解码器返回已解码的输出数据。`avcodec_receive_frame(ctx, _avframe)` 得到AVFrame
>4. AVFrame得到的 YUV420P 转成 NV12
>

```swift
- (void)inputData:(const uint8_t *)data len:(int)len type:(int)type
{
    _running = YES;


    
    AVPacket pkt;
    AVCodecContext *ctx;
    if (type == 0) {
        ctx = _h264Ctx;
        
        if (*(uint32_t *)data == 0x01000000) {
            av_new_packet(&pkt, len);
            memcpy(pkt.data, data, len);
        } else {
            av_new_packet(&pkt, len + 4);
            pkt.data[0] = 0;
            pkt.data[1] = 0;
            pkt.data[2] = 0;
            pkt.data[3] = 1;
            memcpy(pkt.data + 4, data, len);
        }
    } else if (type == 1) {
        ctx = _mpeg4Ctx;

        static uint8_t mpeg4HeaderBuf[128];
        static int mpeg4HeaderBufLen = 0;
        if (*(uint32_t *)data == 0xb0010000) {
            if (len <= 128) {
                mpeg4HeaderBufLen = len;
                memcpy(mpeg4HeaderBuf, data, len);
                return;
            }
        }

        av_new_packet(&pkt, len + mpeg4HeaderBufLen);
        if (mpeg4HeaderBufLen) {
            memcpy(pkt.data, mpeg4HeaderBuf, mpeg4HeaderBufLen);
        }
        memcpy(pkt.data + mpeg4HeaderBufLen, data, len);
    } else {
        return;
    }

    
    avcodec_send_packet(ctx, &pkt);

    if (avcodec_receive_frame(ctx, _avframe) != 0) {
        av_packet_unref(&pkt);
        return;
    }
    
    
    
    av_packet_unref(&pkt);

    int width = ctx->width;
    int height = ctx->height;

    if (width != _width || height != _height) {
        _width = width;
        _height = height;
        if (_swsCtx) {
            sws_freeContext(_swsCtx);
            _swsCtx = NULL;
        }
        //YUV420P -> NV12
        _swsCtx = sws_getContext(width, height, AV_PIX_FMT_YUV420P, width, height, AV_PIX_FMT_NV12, SWS_FAST_BILINEAR, NULL, NULL, NULL);
    }

    if (_pixelBuffer) {
        CVPixelBufferRef tem = _pixelBuffer;
        _pixelBuffer = nil;
        CVPixelBufferRelease(tem);
    }

    CVPixelBufferRef pixelBuffer;

    CVPixelBufferCreate(NULL, width, height, kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange, (__bridge CFDictionaryRef)(@{(id)kCVPixelBufferIOSurfacePropertiesKey:@{}}), &pixelBuffer);

    CVPixelBufferLockBaseAddress(pixelBuffer, 0);

    uint8_t *yBuf = (uint8_t *)CVPixelBufferGetBaseAddressOfPlane(pixelBuffer, 0);
    uint8_t *uvBuf = (uint8_t *)CVPixelBufferGetBaseAddressOfPlane(pixelBuffer, 1);

    uint8_t *dst[8] = {yBuf, uvBuf};
    int dstStride[8] = {width, width};

    sws_scale(_swsCtx, (const uint8_t * const *)_avframe->data, _avframe->linesize, 0, height, dst, dstStride);

    CVPixelBufferUnlockBaseAddress(pixelBuffer, 0);

    _pixelBuffer = pixelBuffer;

    
    
//    CMSampleBufferRef sample;
//
//    CMVideoFormatDescriptionRef format;
//
//    CMVideoFormatDescriptionCreateForImageBuffer(NULL, pixelBuffer, &format);
//
//    CMSampleTimingInfo timingInfo = {CMTimeMake(3000, 90000), CMTimeMake(3000, 90000), CMTimeMake(3000, 90000)};
//
//    CMSampleBufferCreateReadyWithImageBuffer(kCFAllocatorDefault, pixelBuffer, format, &timingInfo, &sample);
//
//    CFArrayRef sampleAttachmentsArray = CMSampleBufferGetSampleAttachmentsArray(sample, true);
//    CFMutableDictionaryRef sampleAttachment = (CFMutableDictionaryRef)CFArrayGetValueAtIndex(sampleAttachmentsArray, 0);
//    CFDictionarySetValue(sampleAttachment, kCMSampleAttachmentKey_DisplayImmediately, kCFBooleanTrue);
//
//    [_displayLayer enqueueSampleBuffer:sample];
//    if (_displayLayer.status == AVQueuedSampleBufferRenderingStatusFailed) {
//        _running = NO;
//    } else {
//        _running = YES;
//    }
//
//    CFRelease(sample);
//
//    CFRelease(format);
//
//    CVPixelBufferRelease(pixelBuffer);
}


```










```swift
解码使用的是ffmpeg，解码后得到的是AVFrame，就需要把AVFrame转成CVPixelbuffer在送给AVSampleBufferDisplayLayer渲染。
如何进行转化呢？如下：
- (void)dispatchAVFrame:(AVFrame*) frame{  
    if(!frame || !frame->data[0]){  
        return;  
    }  
  
    CVReturn theError;  
    if (!self.pixelBufferPool){  
        NSMutableDictionary* attributes = [NSMutableDictionary dictionary];  
        [attributes setObject:[NSNumber numberWithInt:kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange] forKey:(NSString*)kCVPixelBufferPixelFormatTypeKey];  
        [attributes setObject:[NSNumber numberWithInt:frame->width] forKey: (NSString*)kCVPixelBufferWidthKey];  
        [attributes setObject:[NSNumber numberWithInt:frame->height] forKey: (NSString*)kCVPixelBufferHeightKey];  
        [attributes setObject:@(frame->linesize[0]) forKey:(NSString*)kCVPixelBufferBytesPerRowAlignmentKey];  
        [attributes setObject:[NSDictionary dictionary] forKey:(NSString*)kCVPixelBufferIOSurfacePropertiesKey];  
        theError = CVPixelBufferPoolCreate(kCFAllocatorDefault, NULL, (__bridge CFDictionaryRef) attributes, &_pixelBufferPool);  
        if (theError != kCVReturnSuccess){  
            NSLog(@"CVPixelBufferPoolCreate Failed");  
        }  
    }  
      
    CVPixelBufferRef pixelBuffer = nil;  
    theError = CVPixelBufferPoolCreatePixelBuffer(NULL, self.pixelBufferPool, &pixelBuffer);  
    if(theError != kCVReturnSuccess){  
        NSLog(@"CVPixelBufferPoolCreatePixelBuffer Failed");  
    }  
      
    CVPixelBufferLockBaseAddress(pixelBuffer, 0);  
    size_t bytePerRowY = CVPixelBufferGetBytesPerRowOfPlane(pixelBuffer, 0);  
    size_t bytesPerRowUV = CVPixelBufferGetBytesPerRowOfPlane(pixelBuffer, 1);  
    void* base = CVPixelBufferGetBaseAddressOfPlane(pixelBuffer, 0);  
    memcpy(base, frame->data[0], bytePerRowY * frame->height);  
    base = CVPixelBufferGetBaseAddressOfPlane(pixelBuffer, 1);  
    memcpy(base, frame->data[1], bytesPerRowUV * frame->height/2);  
    CVPixelBufferUnlockBaseAddress(pixelBuffer, 0);  
      
    [self dispatchPixelBuffer:pixelBuffer];  
}  
这里的前提是AVFrame中yuv的格式是nv12；但如果AVFrame是yuv420p，就需要把frame->data[1]和frame->data[2]的每一个字节交叉存储到pixelBUffer的plane1上，即把原来的uuuu和vvvv，保存成uvuvuvuv，如下：

uint32_t size = frame->linesize[1] * frame->height;  
uint8_t* dstData = new uint8_t[2 * size];  
for (int i = 0; i < 2 * size; i++){  
    if (i % 2 == 0){  
        dstData[i] = frame->data[1][i/2];  
    }else {  
        dstData[i] = frame->data[2][i/2];  
    }  
}  

这样，只要把dstData中的内容拷贝到pixelBUffer的plane1上即可。
这里使用CVPixelBufferPool的原因是，如果每次都从一个AVFrame去构造一个新的CVPixelbuffer将会非常占用cpu，也比较耗时。
```
