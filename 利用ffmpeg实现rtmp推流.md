
# 利用ffmpeg实现rtmp推流 #

1、首先下载ffmpeg和ffplay

13908708-d9f1366938a083b7.png

    http://ffmpeg.org/

官方下载链接为:http://ffmpeg.org/



2、cmd进入ffmpeg所在目录

13908708-fef42f75ed9e4347.png

cmd进入ffmpeg.exe所在目录


3、ffmpeg查看电脑设备

输入下面的语句即可列出电脑的设备

    ffmpeg -list_devices true -f dshow -i dummy

13908708-e20d8a39022dd71a.png

    ffmpeg -list_devices true -f dshow -i dummy

可以看到我电脑里面有USB2.0 PC CAMERA摄像头和一个乱码的麦克风

如果设备名称有中文，会出现乱码，想看设备原名，可以去设备管理器中查看，又可以利用第三方工具查看，推荐后者。

比如使用graphedit，打开程序后 图表-> 插入过滤器，就可以看到相应的设备名

13908708-98af70e6fc3ad39e.png
13908708-ddd08dfaeba48f05.png
13908708-070505010a6cf0ce.png

可以发现可用设备为USB2.0 PC CAMERA和麦克风 (2- USB2.0 MIC)
可以发现可用设备为USB2.0 PC CAMERA和麦克风 (2- USB2.0 MIC)



4、测试摄像头是否可用

cmd中输入下面语句并回车（USB2.0 PC CAMERA为摄像头名称）

    ffplay -f dshow -i video="USB2.0 PC CAMERA"  

或者

    ffplay -f vfwcap -i 0

13908708-4fe06a8ef30004cc.png

弹出的监控画面

如果成功弹出播放窗口，则代表设备可用，否则可能是设备不可用或者设备被占用



5、查看摄像头和麦克风信息

cmd中输入下面语句即可查询摄像头信息

    ffmpeg -list_options true -f dshow -i video="USB2.0 PC CAMERA"  

13908708-24fbd94f916e4c4a.png

    ffmpeg -list_options true -f dshow -i video="USB2.0 PC CAMERA"  

USB2.0 PC CAMERA摄像头信息



cmd中输入下面语句即可查询麦克风信息

`ffmpeg -list_options true -f dshow -i audio="麦克风 (2- USB2.0 MIC)"`  

13908708-89aeed189953ee29.png

麦克风(2- USB2.0 MIC)信息



6、本地视频的推流

先进行简单的本地视频推流模拟，我们在ffmpeg的目录下放置一个视频，然后cmd进入该目录，把视频推流至rtmp://127.0.0.1:1935/live/123（127.0.0.1:1935为rtmp服务器地址、live为nginx配置节点、123当做**，推流拉流地址一样即可播放），语句如下

    ffmpeg.exe -re -i demo.wmv -f flv rtmp://127.0.0.1:1935/live/123

13908708-0a9bdc89d0288f3f.png

推流中...

此时ffmpeg源源不断的把视频推流至服务器，如果地址没错，可以利用vlc或其他手段实现拉流，这里就先不解释如何拉流



7、摄像头推流

接下来正式把对摄像头进行推流，从前面我们知道摄像头名称为USB2.0 PC CAMERA，而且推流服务器ip为127.0.0.1:1935，关键字为live，所以cmd中输入以下语句：

    ffmpeg -f dshow -i video="USB2.0 PC CAMERA" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f flv rtmp://127.0.0.1:1935/live/123

13908708-5c714ab0c7582d0d.png

摄像头推流中

和本地视频推流一样，摄像头拍到的画面会实时推流出去（当然会有延迟而且现在是没有声音的），当地址正确时，可以实现拉流



8、麦克风推流

前面介绍了摄像头画面推流，可是没有声音，这次我们把麦克风声音推流出去，cmd中输入下面语句

    ffmpeg  -f dshow -i audio="麦克风 (2- USB2.0 MIC)" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f flv rtmp://127.0.0.1:1935/live/123

13908708-ca25894e87d59800.png

推送声音

和前面差不多，声音被推流出去了，通过vlc拉流可以听到录制的声音，但很明显不会有画面



9、摄像头&麦克风推流

终于到最激动人心的时刻了，我们这次要实现同时推流摄像头画面与声音，此时我们的语句应该如下

    ffmpeg -f dshow -i video="USB2.0 PC CAMERA" -f dshow -i audio="麦克风 (2- USB2.0 MIC)" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f flv rtmp://127.0.0.1:1935/live/123

或者

    ffmpeg -f dshow -i video="USB2.0 PC CAMERA":audio="麦克风 (2- USB2.0 MIC)" -vcodec libx264  -r 25  -preset:v ultrafast -tune:v zerolatency -f flv rtmp://127.0.0.1:1935/live/123

13908708-e1c2089f87021e88.png

监控画面与声音同步推流

很nice，和前面一样，画面与声音源源不断的被推流到服务器，接下来我们就应该正式的开发拉流了