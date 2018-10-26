---
layout:     post
title:      "iOS通过CVImageBuffer生成CGImage"
subtitle:   "本文记录在实际开发中使用CVImageBuffer生成CGImage。"
date:       2018-02-07
author:     "minsol"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - iOS开发记录
---

```swift

方法1
        CVPixelBufferLockBaseAddress(imageBuffer, .readOnly)
        
        let width = CVPixelBufferGetWidth(imageBuffer)
        let height = CVPixelBufferGetHeight(imageBuffer)
        let bytesPerRow = CVPixelBufferGetBytesPerRow(imageBuffer)
        let bufferSize = CVPixelBufferGetDataSize(imageBuffer)
        let baseAddress = CVPixelBufferGetBaseAddress(imageBuffer)!
        let cfData = CFDataCreateWithBytesNoCopy(nil, baseAddress.assumingMemoryBound(to: UInt8.self), bufferSize, kCFAllocatorNull)!
        let dataProvider = CGDataProvider(data: cfData)!
        let bitmapInfo = CGBitmapInfo.byteOrder32Little.union(CGBitmapInfo(rawValue: CGImageAlphaInfo.noneSkipFirst.rawValue))
        let cgImage = CGImage(width: width, height: height, bitsPerComponent: 8, bitsPerPixel: 32, bytesPerRow: bytesPerRow, space: colorSpace, bitmapInfo: bitmapInfo, provider: dataProvider, decode: nil, shouldInterpolate: true, intent: .defaultIntent)!

        CVPixelBufferUnlockBaseAddress(imageBuffer, .readOnly)
        
        
方法2
    lazy var context: CIContext = {
        let eaglContext = EAGLContext(api: EAGLRenderingAPI.openGLES2)
        let options = [kCIContextWorkingColorSpace : NSNull()]
        return  CIContext(eaglContext: eaglContext!, options: options)
    }()
    
    
        let outputImage = CIImage(cvImageBuffer: imageBuffer)
        _ = context.createCGImage(outputImage, from: outputImage.extent)

```
