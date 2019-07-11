# liverecord
record.sh为自动录播脚本，支持youtube频道、twitcast频道、twitch频道、openrec频道、mirrativ频道、reality频道、niconico生放送、niconico社区、niconico频道（支持登陆niconico账号进行录制）、bilibili直播间、其它streamlink支持的直播网址和ffmpeg支持的m3u8地址。  
bilibili录制支持在youtube频道、twitcast频道、twitch频道、openrec频道、mirrativ频道、reality频道有直播时不进行录制，从而简单的排除转播的录制。
支持按照录制时长分段。  
支持选择自动备份到onedrive或者百度云。支持选择是否根据上传结果保留本地文件。  
如果因为偶发的检测异常导致没有直播时开始录制，进而产生没有相应录像文件的log文件，脚本将会自动删除这个没有对应录像文件的log文件。

感谢[live-stream-recorder](https://github.com/printempw/live-stream-recorder)、[GiGaFotress/Vtuber-recorder](https://github.com/GiGaFotress/Vtuber-recorder)。  

# 环境依赖
自动录播需要curl，[ffmpeg](https://github.com/FFmpeg/FFmpeg)，[streamlink](https://github.com/streamlink/streamlink)(基于python3)，[livedl](https://github.com/himananiito/livedl)(基于go)。  
其中livedl为可选，目的是支持twitcast高清录制和niconico相关的录制， __请将编译完成的livedl文件放置于运行时目录的livedl/文件夹内__  。如果不希望使用livedl可以选择twitcastffmpeg参数而非twitcast参数进行twitcast的录制，无法进行niconico的录制。  

onedrive自动备份功能需要[OneDrive for Business on Bash](https://github.com/0oVicero0/OneDrive)，在服务器获取授权后即可使用。  
百度云自动备份功能需要[BaiduPCS-Go](https://github.com/iikira/BaiduPCS-Go)，在服务器登陆后即可使用。如果上传不稳定建议尝试修改设置为使用https方式上传。  

# 自动录播使用方法
### 方法
`record.sh youtube|youtubeffmpeg|youtubecurl|youtubecurlffmpeg|twitcast|twitcastffmpeg|twitch|openrec|mirrativ|reality|nicolv[:用户名,密码]|nicoco[:用户名,密码]|nicoch[:用户名,密码]|bilibili|streamlink|m3u8" "频道号码" [best|其他清晰度] [loop|once|视频分段时间] [10|其他监视间隔] ["record_video/other|其他本地目录"] [nobackup|onedrive|baidupan|both|onedrivekeep|baidupankeep|bothkeep|onedrivedel|baidupandel|bothdel] ["noexcept|排除转播的youtube频道号码"] ["noexcept|排除转播的twitcast频道号码"] ["noexcept|排除转播的twitch频道号码"] ["noexcept|排除转播的openrec频道号码"] ["noexcept|排除转播的streamlink支持的频道网址"]`  
### 示例
```
record.sh youtube "UCWCc8tO-uUl_7SJXIKJACMw"   #录制https://www.youtube.com/channel/UCWCc8tO-uUl_7SJXIKJACMw

record.sh youtubeffmpeg "UCWCc8tO-uUl_7SJXIKJACMw" 1080p,720p,480p,360p,worst once loop 30 "record_video/mea" both   #录制https://www.youtube.com/channel/UCWCc8tO-uUl_7SJXIKJACMw，使用ffmpeg，依次获取1080p 720p 480p 360p worst中第一个可用的清晰度，在检测到直播并进行一次录制后终止，间隔30秒检测，录像保存于record_video/mea文件夹中，录制完成后自动上传到onedrive和百度云相同路径并在上传结束后删除本地录像，如果上传失败则会保留本地录像

nohup record.sh bilibili "12235923" best 7200 30 "record_video/mea_bilibili" bothkeep "UCWCc8tO-uUl_7SJXIKJACMw" "kaguramea" "kagura0mea" "KaguraMea" > mea_bilibili.log &   #后台录制https://www.twitch.tv/kagura0mea，最高清晰度，循环检测并在录制进行7200秒时分段，间隔30秒检测，录像保存于record_video/mea文件夹中，录制完成后自动上传到onedrive和百度云相同路径并在上传完成后保留本地录像，log记录保存于mea_bilibili.log文件
 ```
### 参数说明
第一个参数必选，选择录制平台。可选参数为youtube、youtubeffmpeg、youtubecurl、youtubecurlffmpeg、twitcast、twitcastffmpeg、twitch、openrec、mirrativ、reality、nicolv、nicoco、nicoch、bilibili、streamlink、m3u8，如果需要登陆nico账号进行nico相关的录制请使用`nicolv:用户名,密码` 、 `nicoco:用户名,密码` 、 `nicoch:用户名,密码`。其中youtube与twitch参数使用streamlink判断直播状态系统占用较多，youtubecurl与youtubecurlffmpeg参数使用curl解析网页的方法判断直播状态可以减少系统占用。youtubeffmpeg与youtubecurlffmpegg参数为使用ffmpeg录制youtube。twitcastffmpeg参数为使用ffmpeg录制twitcasting，无法录制高清流但不需要livedl。nicolv、nicoco、nicoch参数都需要livedl。 __注意依赖于livedl的录制请尽量不要重复录制同一个直播（即使用重复的多个livedl进程录制同一个直播），使用livedl录制同一个twitcast直播虽然都能正常进行录制但文件名将带-2后缀，使用livedl重复录制同一个niconico直播文件名相同会导致数据库锁定（可以在调用livedl时通过`-nico-format 文件名`参数指定文件名避免，本脚本已设置此参数所以应该不会出现文件名冲突）和相同账号重复登陆会导致websocket冲突（即同一直播同一账号只能进行一个录制，可以通过不使用登陆功能避免，即同一直播可以同时进行多个未登陆的录制，但只有登陆才能使用timeshift和会员限定这样的功能）。__  
第二个参数必选，选择频道号码。其中youtube、twitcast、twitch、openrec、mirrativ为对应网站个人主页网址中的ID部分，reality为频道名称(如果为部分名字则匹配含有这些文字的其中一个频道)或vlive_id(获取方法可于脚本内查找)，nicolv为niconico生放送号码(如lv320447549)，nicoco为niconico社区号码(如co41030)，nicoch为niconico频道号码(如macoto2525)，bilibili为直播间号码，streamlink为直播网址，m3u8为直播m3u8文件网址。  

第三个参数可选，选择清晰度，默认为best。可以用,分隔来指定多个清晰度，将会依次获取尝试直到获取第一个可用的清晰度。 __注意使用ffmpeg录制youtube直播仅支持1080p以下的清晰度，请不要使用best或1080p60及以上的参数__ 。  
第四个参数可选，选择是否循环或者录制分段时间，默认为loop。如果指定为once则会在检测到直播并进行一次录制后终止，如果指定为数字则会在录制进行相应秒数时分段，使用视频分段功能时为loop模式。 __注意分段时可能会导致十秒左右的视频缺失__ 。  
第五个参数可选，选择监视间隔，默认为10秒。  
第六个参数可选，选择本地录像存放目录，默认为record_video/other文件夹。  
第七个参数可选，选择是否自动备份，默认为nobackup。可选参数为nobackup、onedrive、baidupan、both、onedrivenot、baidupannot、bothnot、onedrivedel、baidupandel、bothdel。其中onedrive、baidupan、both分别指上传onedrive、上传百度云、同时上传，在一次录制完成后开始上传，上传路径与本地路径相同，如果上传成功则删除本地文件，上传失败将会保留本地文件。带有keep的参数即使上传成功也会保留本地文件。带有del的参数即使上传失败也会删除本地文件。  

第八到十四个参数可选，默认为noexcept。按照顺序分别为选择排除转播的youtube频道号码、排除转播的twitcast频道号码、排除转播的twitch频道号码、noexcept|排除转播的openrec频道号码、排除转播的mirrativ频道号码、排除转播的reality频道号码、排除转播的streamlink支持的频道网址，相应频道正在直播时不进行录制。排除转播功能仅支持bilibili录制。  
