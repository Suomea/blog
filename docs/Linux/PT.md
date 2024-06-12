# FFmpeg
## 容器
视频文件本身就是一个容器，包括音频、视频和字幕等信息。
容器的格式
```
- MP4：一种通用的多媒体容器格式，支持视频、音频、字幕和静态图像等多种数据。
    
- AVI：是Windows平台上广泛使用的音频/视频文件格式。
    
- MKV：Matroska多媒体容器格式，可容纳几乎任何类型的音频、视频、字幕和附加元数据。
    
- MOV：由苹果公司开发的QuickTime容器格式，常用于存储视频和音频数据。
    
- FLV：Flash视频，主要用于流式传输媒体内容。
    
- WMV：Windows Media Video，一种由微软开发的视频容器格式。
    
- RMVB：RealMedia Variable Bitrate，由RealNetworks开发的流媒体格式，通常用于在线视频播放。
    
- WEBM：一种开放、免费的多媒体容器格式，主要用于网络视频。
```

查看 ffmpeg 支持的格式 `ffmpeg -formats`
## 编码
音视频都要经过编码存储，就是文本编码的 UTF8。
视频编码
```
- H.264/AVC：目前最为普遍的视频编码标准之一，用于压缩高质量视频，适用于各种应用场景，如在线视频、蓝光光盘等。
    
- H.265/HEVC：高效视频编码标准，相较于H.264，在相同画质下可以实现更高的压缩率，适用于高清和超高清视频。
    
- VP9：由Google开发的开放式视频编码格式，用于网络视频，具有高压缩率和高画质的特点。
    
- AV1：由Alliance for Open Media (AOMedia)开发的开放式视频编码格式，旨在提供更高的压缩效率和更好的视觉质量，适用于网络视频等场景。
    
- MPEG-2：早期广泛用于DVD、数字电视等应用的视频编码标准。
    
- MPEG-4 Part 2：包括Xvid、DivX等编码器，用于压缩视频以及流式传输媒体内容。
    
- VC-1：由微软开发的高压缩率视频编码标准，通常用于蓝光光盘、高清DVD等。
    
- Theora：开源的视频编码格式，主要用于网络视频。
```

音频编码
```
- MP3 是一种常见的音频编码格式，全称为 MPEG-1 Audio Layer III。它是一种有损压缩的音频格式，旨在通过去除人耳听不到的音频信号的部分来减小文件大小，从而实现高度压缩，同时保留足够的音质以供大多数人享受。
	
- FLAC：Free Lossless Audio Codec 是一种开源的无损音频编码格式，由 Xiph.Org Foundation 开发。
	
- APE：Monkey's Audio 是一种由 Matthew T. Ashland 开发的无损音频编码格式。
	
- AAC：Advanced Audio Coding，是一种高效的有损压缩音频格式，常用于iTunes音乐、YouTube等。
    
- WAV：Waveform Audio File Format，是一种无损音频格式，通常用于存储音频数据，保留了原始音频的高质量。
    
- FLAC：Free Lossless Audio Codec，是一种自由开放的无损音频编码格式，可以实现无损压缩，保留音频的原始质量。
    
- OGG：Ogg Vorbis，是一种开放、自由的有损音频编码格式，常用于音乐流媒体等场景。
    
- WMA：Windows Media Audio，是微软开发的一种有损音频编码格式，常用于Windows平台的音频文件。
    
- Opus：一种开放、免费的音频编码格式，旨在提供高质量的音频编码，适用于各种应用，如VoIP通话、网络音频流等。
    
- PCM：Pulse-code Modulation，是一种原始的数字音频编码格式，通常不进行压缩，用于存储音频数据或作为其他编码格式的基础。
```

查看 ffmpeg 支持的编码 `ffmpeg -codecs`

## 编码器
顾名思义，实现了编码格式的库文件。
查看 ffmpeg 已安装的编码器 `ffmpeg -encoders
`


# 文件名解析
The.Beekeeper.2024.2160p.UHD.Blu-ray.Remux.HEVC.TrueHD7.1.Atmos.iso
```
- The Beekeeper: 这可能是视频的标题，指示视频内容或者是电影、电视节目的名称。
    
- 2024: 这可能是指视频的发布年份，也可能是某个事件的年份，或者是某个版本的年份。
    
- 2160p: 这表示视频的分辨率，通常指的是4K分辨率。
    
- UHD: 这是Ultra High Definition的缩写，表示视频是超高清的。
    
- Blu-ray Remux: 这表明视频是从蓝光光盘中重新复制而来的，通常保留了原始视频的质量和特性。Remux 表示重制，一般仅仅删除原盘里面的多余内容，如花絮，仅保留主要的视频、音轨和部分语言字幕，画质和音质没有做任何修改。如果有WEB-DL/WEBRip 表示在线高清电影，一般是流媒体平台的格式，会有压缩和观感的损失。BDRip 采用更厉害的压缩算法将音视频进行有损压缩。
    
- HEVC: 这是High Efficiency Video Coding的缩写，指的是视频使用的编解码格式。
    
- TrueHD 7.1 Atmos: 这指出了视频的音频编解码格式和声道布局，其中TrueHD 7.1表示音频格式，而Atmos表示音频的立体声音效。
    
- .iso: 这可能是指文件的类型或者文件扩展名，"iso"通常表示光盘镜像文件。
```


