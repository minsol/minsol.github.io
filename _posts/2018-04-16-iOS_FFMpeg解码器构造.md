---
layout:     post
title:      "iOS-FFmpeg解码流程"
subtitle:   "本文记录在实际开发中FFmpeg解码流程，做以备忘。"
date:       2018-04-16
author:     "minsol"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - 备忘录
---

```swift
1）创建
self.video = [[XYQMovieObject alloc] initWithVideo:@"rtmp://192.168.0.156:1935/rtmplive/test"];
2）开始事件
[video seekTime:0.0];
3）开启定时器
间隔：1 / video.fps
4）调用解码
[video stepFrame];
5）得到图片
video.currentImage;
```

```swift
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>
#include <libavcodec/avcodec.h>
#include <libavformat/avformat.h>
#include <libswscale/swscale.h>

@interface XYQMovieObject : NSObject

/* 解码后的UIImage */
@property (nonatomic, strong, readonly) UIImage *currentImage;

/* 视频的frame高度 */
@property (nonatomic, assign, readonly) int sourceWidth, sourceHeight;

/* 输出图像大小。默认设置为源大小。 */
@property (nonatomic,assign) int outputWidth, outputHeight;

/* 视频的长度，秒为单位 */
@property (nonatomic, assign, readonly) double duration;

/* 视频的当前秒数 */
@property (nonatomic, assign, readonly) double currentTime;

/* 视频的帧率 */
@property (nonatomic, assign, readonly) double fps;

/* 视频路径。 */
- (instancetype)initWithVideo:(NSString *)moviePath;

/* 切换资源 */
- (void)replaceTheResources:(NSString *)moviePath;

/* 重拨 */
- (void)redialPaly;

/* 从视频流中读取下一帧。返回假，如果没有帧读取（视频）。 */
- (BOOL)stepFrame;

/* 寻求最近的关键帧在指定的时间 */
- (void)seekTime:(double)seconds;

@end

```


```swift
//
//  XYQMovieObject.m
//  FFmpeg_Test
//
//  Created by mac on 16/7/11.
//  Copyright © 2016年 xiayuanquan. All rights reserved.
//

#import "XYQMovieObject.h"

@interface XYQMovieObject()
@property (nonatomic, copy) NSString *cruutenPath;
@end

@implementation XYQMovieObject
{
    AVFormatContext     *XYQFormatCtx;
    AVCodecContext      *XYQCodecCtx;
    AVFrame             *XYQFrame;
    AVStream            *stream;
    AVPacket            packet;
    AVPicture           picture;
    int                 videoStream;
    double              fps;
    BOOL                isReleaseResources;
}

#pragma mark ------------------------------------
#pragma mark  初始化
- (instancetype)initWithVideo:(NSString *)moviePath {
    
    if (!(self=[super init])) return nil;
    if ([self initializeResources:[moviePath UTF8String]]) {
        self.cruutenPath = [moviePath copy];
        return self;
    } else {
        return nil;
    }
}
- (BOOL)initializeResources:(const char *)filePath {
    
    isReleaseResources = NO;
    AVCodec *pCodec;
    // 注册所有解码器
    avcodec_register_all();
    av_register_all();
    avformat_network_init();
    // 打开视频文件
    if (avformat_open_input(&XYQFormatCtx, filePath, NULL, NULL) != 0) {
        NSLog(@"打开文件失败");
        goto initError;
    }
    // 检查数据流
    if (avformat_find_stream_info(XYQFormatCtx, NULL) < 0) {
        NSLog(@"检查数据流失败");
        goto initError;
    }
    // 根据数据流,找到第一个视频流
    if ((videoStream =  av_find_best_stream(XYQFormatCtx, AVMEDIA_TYPE_VIDEO, -1, -1, &pCodec, 0)) < 0) {
        NSLog(@"没有找到第一个视频流");
        goto initError;
    }
    // 获取视频流的编解码上下文的指针
    stream      = XYQFormatCtx->streams[videoStream];
    XYQCodecCtx  = stream->codec;
#if DEBUG
    // 打印视频流的详细信息
    av_dump_format(XYQFormatCtx, videoStream, filePath, 0);
#endif
    if(stream->avg_frame_rate.den && stream->avg_frame_rate.num) {
        fps = av_q2d(stream->avg_frame_rate);
    } else { fps = 30; }
    // 查找解码器
    pCodec = avcodec_find_decoder(XYQCodecCtx->codec_id);
    if (pCodec == NULL) {
        NSLog(@"没有找到解码器");
        goto initError;
    }
    // 打开解码器
    if(avcodec_open2(XYQCodecCtx, pCodec, NULL) < 0) {
        NSLog(@"打开解码器失败");
        goto initError;
    }
    // 分配视频帧
    XYQFrame = av_frame_alloc();
    _outputWidth = XYQCodecCtx->width;
    _outputHeight = XYQCodecCtx->height;
    return YES;
initError:
    return NO;
}
- (void)seekTime:(double)seconds {
    AVRational timeBase = XYQFormatCtx->streams[videoStream]->time_base;
    int64_t targetFrame = (int64_t)((double)timeBase.den / timeBase.num * seconds);
    avformat_seek_file(XYQFormatCtx,
                       videoStream,
                       0,
                       targetFrame,
                       targetFrame,
                       AVSEEK_FLAG_FRAME);
    avcodec_flush_buffers(XYQCodecCtx);
}

- (BOOL)stepFrame {
    int frameFinished = 0;
    while (!frameFinished && av_read_frame(XYQFormatCtx, &packet) >= 0) {
        if (packet.stream_index == videoStream) {
            avcodec_decode_video2(XYQCodecCtx,
                                  XYQFrame,
                                  &frameFinished,
                                  &packet);
        }
    }
    if (frameFinished == 0 && isReleaseResources == NO) {
        [self releaseResources];
    }
    return frameFinished != 0;
}

- (void)replaceTheResources:(NSString *)moviePath {
    if (!isReleaseResources) {
        [self releaseResources];
    }
    self.cruutenPath = [moviePath copy];
    [self initializeResources:[moviePath UTF8String]];
}
- (void)redialPaly {
    [self initializeResources:[self.cruutenPath UTF8String]];
}
#pragma mark ------------------------------------
#pragma mark  重写属性访问方法
-(void)setOutputWidth:(int)newValue {
    if (_outputWidth == newValue) return;
    _outputWidth = newValue;
}
-(void)setOutputHeight:(int)newValue {
    if (_outputHeight == newValue) return;
    _outputHeight = newValue;
}
-(UIImage *)currentImage {
    if (!XYQFrame->data[0]) return nil;
    return [self imageFromAVPicture];
}
-(double)duration {
    return (double)XYQFormatCtx->duration / AV_TIME_BASE;
}
- (double)currentTime {
    AVRational timeBase = XYQFormatCtx->streams[videoStream]->time_base;
    return packet.pts * (double)timeBase.num / timeBase.den;
}
- (int)sourceWidth {
    return XYQCodecCtx->width;
}
- (int)sourceHeight {
    return XYQCodecCtx->height;
}
- (double)fps {
    return fps;
}
#pragma mark --------------------------
#pragma mark - 内部方法
- (UIImage *)imageFromAVPicture
{
    avpicture_free(&picture);
    //picture将会存储进行缩放后的数据，所以给其带的参数将在后面生成UIImage时予以指定，存储生成后的图片大小，将用来进行指定解码后的图像尺寸
    avpicture_alloc(&picture, AV_PIX_FMT_RGB24, _outputWidth, _outputHeight);
    // 下面这两步非常重要，是进行视频尺寸缩放的关键
    struct SwsContext * imgConvertCtx = sws_getContext(XYQFrame->width,
                                                       XYQFrame->height,
                                                       AV_PIX_FMT_YUV420P,
                                                       _outputWidth,
                                                       _outputHeight,
                                                       AV_PIX_FMT_RGB24,
                                                       SWS_FAST_BILINEAR,
                                                       NULL,
                                                       NULL,
                                                       NULL);
    
    // 下面使用前面初始化时设定的原始数据以及指定输出的UIImage尺寸大小进行缩放
    if(imgConvertCtx == nil) return nil;
    //把数据从pFrame写到picture之中
    sws_scale(imgConvertCtx,
              XYQFrame->data,
              XYQFrame->linesize,
              0,
              XYQFrame->height,
              picture.data,
              picture.linesize);
    sws_freeContext(imgConvertCtx);
    
    CGBitmapInfo bitmapInfo = kCGBitmapByteOrderDefault;
    CFDataRef data = CFDataCreate(kCFAllocatorDefault,
                                  picture.data[0],
                                  picture.linesize[0] * _outputHeight);
    
    CGDataProviderRef provider = CGDataProviderCreateWithCFData(data);
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    CGImageRef cgImage = CGImageCreate(_outputWidth,
                                       _outputHeight,
                                       8,                   //8位存储一个Component
                                       24,                  // RGB存储，只用三个字节，而不是像RGBA要用4个字节，所以这里一个像素点要3个8位来存储
                                       picture.linesize[0],
                                       colorSpace,
                                       bitmapInfo,
                                       provider,
                                       NULL,
                                       NO,
                                       kCGRenderingIntentDefault);
    UIImage *image = [UIImage imageWithCGImage:cgImage];
    CGImageRelease(cgImage);
    CGColorSpaceRelease(colorSpace);
    CGDataProviderRelease(provider);
    CFRelease(data);
    
    return image;
}

#pragma mark --------------------------
#pragma mark - 释放资源
- (void)releaseResources {
    NSLog(@"释放资源");
//    SJLogFunc
    isReleaseResources = YES;
    // 释放RGB
    avpicture_free(&picture);
    // 释放frame
    av_packet_unref(&packet);
    // 释放YUV frame
    av_free(XYQFrame);
    // 关闭解码器
    if (XYQCodecCtx) avcodec_close(XYQCodecCtx);
    // 关闭文件
    if (XYQFormatCtx) avformat_close_input(&XYQFormatCtx);
    avformat_network_deinit();
}
@end
```
