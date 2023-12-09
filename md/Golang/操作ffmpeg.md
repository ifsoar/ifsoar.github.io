引入ffmpeg库

```go
"github.com/floostack/transcoder/ffmpeg"
```

获取媒体信息
```go
// 获取视频元信息  
func testMetadata() {  
  
   //可以只配置ffprobe库  
   ffmpegConf := &ffmpeg.Config{  
      //FfmpegBinPath:  "D:\\ffmpeg-2020-12-27-git-bff6fbead8-full_build\\bin\\ffmpeg.exe",  
      FfprobeBinPath: "D:\\ffmpeg-2020-12-27-git-bff6fbead8-full_build\\bin\\ffprobe.exe",  
   }  
  
   metadata, err := ffmpeg.  
      New(ffmpegConf).  
      Input("/test.mp4").  
      GetMetadata()  
   if err != nil {  
      log.Fatal(err)  
   }  
   metadata.GetFormat()  
   data, _ := json.Marshal(metadata)  
   fmt.Println(string(data))  
   //meta结构见json  
   /*      "duration": "2759.876000"  时长秒值  
      "size": "643491783"          文件大小  
      "width": 1920           视频分辨率  
      "height": 1080      "display_aspect_ratio": "16:9"    宽高比  
      "avg_frame_rate": "25/1"         帧率  
   */}
```

将视频转码成ts并切割成m3u8
```go
func testFfmpeg() {  
   inputOpts := ffmpeg.Options{}  
  
   hlsSegmentDuration := 10                 //每个ts片的时长秒值  
   hlsSegmentFilename := "/out/part%04d.ts" //ts片段保存的文件名，%04d代表用四位数字表示序号，不足部分补零  
   hlsListSize := 0                         //m3u8文件保存切片信息的数量，0代表保存全部  
   outputFormat := "hls"                    //输出格式，hls代表输出位hls的ts流  
   overWrite := true                        //是否覆盖已经存在的输出文件  
   videoFilter := "scale=iw/1.5:ih/1.5"     //分辨率缩放，支持固定值【scale=320:240】、支持在原始尺寸上缩放【scale=iw/1.5:ih/1.5】代表原始宽高除以1.5，也可以乘以  
   videoBitRate := "1.5M"                   //码率限制，支持小数点  
   //frameRate := 30                          //帧率，常见有25、30、60  
   threads := 4 //使用的cpu核心数，可以用该值来限制cpu占用，否则会吃满cpu  
   outputOpts := ffmpeg.Options{  
      HlsSegmentDuration: &hlsSegmentDuration,  
      HlsSegmentFilename: &hlsSegmentFilename,  
      HlsListSize:        &hlsListSize,  
      OutputFormat:       &outputFormat,  
      Overwrite:          &overWrite,  
      VideoFilter:        &videoFilter,  
      VideoBitRate:       &videoBitRate,  
      //FrameRate:          &frameRate,  
      Threads: &threads,  
   }  
  
   ffmpegConf := &ffmpeg.Config{  
      FfmpegBinPath:   "D:\\ffmpeg-2020-12-27-git-bff6fbead8-full_build\\bin\\ffmpeg.exe",  
      FfprobeBinPath:  "D:\\ffmpeg-2020-12-27-git-bff6fbead8-full_build\\bin\\ffprobe.exe",  
      ProgressEnabled: true, //启用进度channel消息  
   }  
  
   progress, err := ffmpeg.  
      New(ffmpegConf).  
      Input("/test.mp4").  
      WithOptions(inputOpts).  
      Output("/out/index.m3u8").  
      Start(outputOpts) //不会阻塞  
  
   if err != nil {  
      log.Fatal(err)  
   }  
   //for循环读取进度消息  
   for msg := range progress {  
      //格式   {FramesProcessed:1 CurrentTime:00:00:00.42 CurrentBitrate:N/A Progress:0.015218075014964439 Speed:23.7x}      
      log.Printf("%+v", msg)  
   }  
}
```