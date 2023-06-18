---
title: "下载youtube播放列表视频"
date: 2023-04-26T09:30:59+08:00
# draft: true
---

# 从youtube下载视频  
  
## 工具
[yt-dlp](https://github.com/yt-dlp/yt-dlp)  

### 用法
下载工具之后，windows下将其所在路径添加至系统环境变量PATH中  
```bash
yt-dlp.exe -i -N 10 --playlist-start 10 -f 'bestvideo[height<=2160] +bestaudio/best[height<=2160]' --verbose -o '%(playlist)s/%(playlist_index)s - %(title)s.%(ext)s' https://www.youtube.com/playlist?list=******  
```
### 参数说明  
-i： 忽略错误  
-N: 多线程下载  
--playlist-start：播放列表开始下载位置  
-f：分辨率  
--verbose：输出过程详细信息  
-o：指定下载输出路径和文件名格式  
最后的是url，即视频播放列表链接  