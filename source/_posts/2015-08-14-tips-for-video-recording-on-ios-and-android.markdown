---
layout: post
title: "iOS和Android视频录制播放注意事项"
date: 2015-08-14 01:15:19 +0800
comments: true
categories: 
---

iOS使用AVFoundation录制的视频默认格式是mov的，在Android下无法播放，需要使用AVAssetExportSession转码/压缩之后Android才能播放，转码代码如下：

```objective-c
- (void)compressionVideoWithURL:(NSURL *)URL
{
    AVURLAsset *avAssert = [AVURLAsset URLAssetWithURL:URL options:nil];
    NSArray *compatiblePresets = [AVAssetExportSession exportPresetsCompatibleWithAsset:avAssert];
    if ([compatiblePresets containsObject:AVAssetExportPresetLowQuality]) {
        AVAssetExportSession *exportSession = [[AVAssetExportSession alloc] initWithAsset:avAssert presetName:AVAssetExportPresetPassthrough];
       
        NSString *docDir = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
        NSString *outputFilePath = [NSString stringWithFormat:@"%@/%@", docDir, [AppManager zhidaoVideoCompressedFileName]];
       
        // can not replace the exists file,so delete it first
        NSFileManager *fileManager = [NSFileManager defaultManager];
        if ([fileManager fileExistsAtPath:outputFilePath]) {
            [fileManager removeItemAtPath:outputFilePath error:nil];
        }
       
        NSLog(@"resultPath:%@", outputFilePath);
       
        exportSession.outputURL = [NSURL fileURLWithPath:outputFilePath];
        exportSession.outputFileType = AVFileTypeMPEG4;
        exportSession.shouldOptimizeForNetworkUse = YES;
        [exportSession exportAsynchronouslyWithCompletionHandler:^{
            switch (exportSession.status) {
                case AVAssetExportSessionStatusUnknown:
                    NSLog(@"AVAssetExportSessionStatusUnknown");
                    break;
                case AVAssetExportSessionStatusWaiting:
                    NSLog(@"AVAssetExportSessionStatusWaiting");
                    break;
                case AVAssetExportSessionStatusExporting:
                    NSLog(@"AVAssetExportSessionStatusExporting");
                    break;
                case AVAssetExportSessionStatusCompleted:{
                    NSLog(@"AVAssetExportSessionStatusCompleted");
                    self.videoLocalURL = [NSURL fileURLWithPath:outputFilePath];
                    break;
                }
                case AVAssetExportSessionStatusFailed:
                    NSLog(@"AVAssetExportSessionStatusFailed");
                    NSLog(@"Export failed: %@", [[exportSession error] localizedDescription]);
                    break;
                case AVAssetExportSessionStatusCancelled:
                    NSLog(@"AVAssetExportSessionStatusCancelled");
                    break;
            }
        }];
       
    }
}
```

Android下录制的时候如果不指定音频格式有些手机录制出来的视频中的音频编码可能是AMR（H.264，AMR Narrowband）在iOS >= 4.3的iOS设备上将无法播放，所以Android录制视频需要指定音频编码为AAC