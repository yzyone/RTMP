
# 【Android RTMP】音频数据采集编码 ( FAAC 编码器编码 AAC 音频解码信息 | 封装 RTMP 音频数据头 | 设置 AAC 音频数据类型 | 封装 RTMP 数据包 ) #

## 一、 FAAC 编码器编码 AAC 音频解码信息 ##

推流 AAC 音频数据之前 , 需要先将 AAC 格式音频的解码信息推流到服务器中 , AAC 音频解码信息用于指导播放器解码 AAC 音频数据 ; 其作用类似于 H.264 视频的 SPS 和 PPS 数据 , 用于指导播放器解码 H.264 视频帧数据 ;


1 . AAC 解码信息生成方法 : FAAC 编码器调用 faacEncGetDecoderSpecificInfo 方法 , 生成 AAC 音频解码信息 ;


2 . faacEncGetDecoderSpecificInfo 方法原型 :
```
#include <faac.h>

int FAACAPI faacEncGetDecoderSpecificInfo(faacEncHandle hEncoder, unsigned char **ppBuffer,
					  unsigned long *pSizeOfDecoderSpecificInfo);
```

① faacEncHandle hEncoder 参数 : FAAC 编码器 ;

② unsigned char **ppBuffer 参数 : 用于接收 FFAC 编码器编码生成的 AAC 解码信息 , 这是个二维指针 , 外部传入 , 当做返回值使用 ; 该值一般需要预先在外部定义 , 定义 unsigned char * 类型一维指针变量 , 或 unsigned char ** 类型二维指针变量 ;

③ unsigned long *pSizeOfDecoderSpecificInfo 参数 : 用于接收 FFAC 编码器编码生成的 AAC 解码信息 字节个数 , 当做返回值使用 ; 该值一般需要预先在外部定义 , 定义 unsigned long 类型变量 , 或 unsigned long* 类型变量 ;

后两个参数定义不同级别的指针类型 , 使用方法不同 , 但形式类似 , 都是用指针变量 , 传入地址作为参数 , 传入的指针当做返回值使用 ;


3 . 代码示例 :
```
    // 该指针用于接收获取的 FAAC 解码特殊信息
    unsigned char *pBuffer;
    // 该指针用于接收获取的 FAAC 解码特殊信息长度
    unsigned long sizeOfDecoderSpecificInfo;
    // 生成 FAAC 解码特殊信息数据
    faacEncGetDecoderSpecificInfo(mFaacEncHandle, &pBuffer, &sizeOfDecoderSpecificInfo);
```




## 二、 封装 RTMP 音频数据头 ##

1 . 封装第 1 11 字节数据 : 第一个字节中封装了 4 44 部分数据 , 音频格式 , 采样率 , 采样位数 , 音频通道 ; 一般情况下是 AE , 或者 AF ;


① AF 含义 : AAC 格式 , 44100 Hz 采样 , 16 位采样位数 , 立体声 ;

② AE 含义 : AAC 格式 , 44100 Hz 采样 , 16 位采样位数 , 单声道 ;


参考博客 【Android RTMP】音频数据采集编码 ( AAC 音频格式解析 | FLV 音频数据标签解析 | AAC 音频数据标签头 | 音频解码配置信息 ) 、四、 音频解码配置信息、 2. 第 11 字节 AF 数据解析 章节 , 有详细介绍这 8 88 位各代表的意义 ;


2 . 代码示例 :
```
    /*
        根据声道数生成相应的 文件头 标识
        AF / AE 头中的最后一位为 1 表示立体声, 为 0 表示单声道
        AF 是立体声
        AE 是单声道
     */
    rtmpPacket->m_body[0] = 0xAF;   //默认立体声

    if (mChannelConfig == 1) {
        // 如果是单声道, 将该值修改成 AE
        rtmpPacket->m_body[0] = 0xAE;
    }
```




## 三、 封装 RTMP 音频数据类型 ##

AAC 音频数据类型 : 如果是编码的音频采样数据 , 类型是 01 , 如果是 AAC 解码信息 , 类型是 00 ; 这里是 00 类型 , AAC 音频解码信息类型 ;
```
    // 编码出的声音 都是 0x01, 头信息是 AF 01 数据
    // 如果是AAC 音频解码数据 , 那么头信息是 AF 00 数据
    rtmpPacket->m_body[1] = 0x00;
```




## 四、 拷贝 AAC 音频数据到 RTMPPacket 数据包中 ##

之前调用 faacEncGetDecoderSpecificInfo 方法 , 生成了 AAC 音频解码信息 , 将生成的信息封装到 RTMPPacket 数据包中 , RTMP 数据包的大小是生成 AAC 解码信息大小 + 2 ; 多出的 2 字节数据是 AF 00 ;
```
    // 拷贝 AAC 音频数据到 RTMPPacket 数据包中
    memcpy(&rtmpPacket->m_body[2], pBuffer, sizeOfDecoderSpecificInfo);
```




## 五、 设置数据包大小 ##

该数据包大小是 2 字节 , 加上 faacEncGetDecoderSpecificInfo 方法生成 的 AAC 解码数据大小 ;

2 字节是 AF 00 ;
```
    // 设置 RTMP 数据包大小
    rtmpPacket->m_nBodySize = rtmpPackagesize;
```




## 六、 设置绝对时间、数据类型、RTMP 通道、头类型 ##

这些数据设置基本都是格式化的 , 按照如下设置即可 ;
```
    // 设置绝对时间, 一般设置 0 即可
    rtmpPacket->m_hasAbsTimestamp = 0;
    // 设置 RTMP 数据包大小
    rtmpPacket->m_nBodySize = rtmpPackagesize;
    // 设置 RTMP 包类型, 视频类型数据
    rtmpPacket->m_packetType = RTMP_PACKET_TYPE_AUDIO;
    // 分配 RTMP 通道, 该值随意设置, 建议在视频 H.264 通道基础上加 1
    rtmpPacket->m_nChannel = 0x11;
    // // 设置头类型, 随意设置一个
    rtmpPacket->m_headerType = RTMP_PACKET_SIZE_LARGE;
```




## 七、 FAAC 编码器编码代码示例 ##

```
/**
 * 获取音频解码信息
 * 推流音频数据时, 先发送解码信息包, 再推流 AAC 音频采样包
 * @return 音频解码数据包
 */
RTMPPacket *AudioChannel::getAudioDecodeInfo() {

    /*
        下面的数据信息用于指导 AAC 数据如何进行解码
        类似于 H.264 视频信息中的 SPS 与  PPS 数据

        int FAACAPI faacEncGetDecoderSpecificInfo(
                        faacEncHandle hEncoder,
                        unsigned char **ppBuffer,
					    unsigned long *pSizeOfDecoderSpecificInfo);


     */
    // 该指针用于接收获取的 FAAC 解码特殊信息
    unsigned char *pBuffer;
    // 该指针用于接收获取的 FAAC 解码特殊信息长度
    unsigned long sizeOfDecoderSpecificInfo;
    // 生成 FAAC 解码特殊信息数据
    faacEncGetDecoderSpecificInfo(mFaacEncHandle, &pBuffer, &sizeOfDecoderSpecificInfo);


    // 组装 RTMP 数据包

    /*
        数据的大小 :
        前面有 2 字节头信息
        音频解码配置信息 : 前两位是 AF 00 , 指导 AAC 数据如何解码 ( 是这个 )
        音频采样信息 : 前两位是 AF 01 , 实际的 AAC 音频采样数据
     */
    int rtmpPackagesize = 2 + sizeOfDecoderSpecificInfo;

    // 创建 RTMP 数据包对象
    RTMPPacket *rtmpPacket = new RTMPPacket;

    // 为 RTMP 数据包分配内存
    RTMPPacket_Alloc(rtmpPacket, rtmpPackagesize);

    /*
        根据声道数生成相应的 文件头 标识
        AF / AE 头中的最后一位为 1 表示立体声, 为 0 表示单声道
        AF 是立体声
        AE 是单声道
     */
    rtmpPacket->m_body[0] = 0xAF;   //默认立体声

    if (mChannelConfig == 1) {
        // 如果是单声道, 将该值修改成 AE
        rtmpPacket->m_body[0] = 0xAE;
    }

    // 编码出的声音 都是 0x01, 本方法是对音频数据进行编码的方法, 头信息肯定是 AF 01 数据
    // 数据肯定是 AAC 格式的采样数据
    rtmpPacket->m_body[1] = 0x00;

    // 拷贝 AAC 音频数据到 RTMPPacket 数据包中
    memcpy(&rtmpPacket->m_body[2], pBuffer, sizeOfDecoderSpecificInfo);

    // 设置绝对时间, 一般设置 0 即可
    rtmpPacket->m_hasAbsTimestamp = 0;
    // 设置 RTMP 数据包大小
    rtmpPacket->m_nBodySize = rtmpPackagesize;
    // 设置 RTMP 包类型, 视频类型数据
    rtmpPacket->m_packetType = RTMP_PACKET_TYPE_AUDIO;
    // 分配 RTMP 通道, 该值随意设置, 建议在视频 H.264 通道基础上加 1
    rtmpPacket->m_nChannel = 0x11;
    // // 设置头类型, 随意设置一个
    rtmpPacket->m_headerType = RTMP_PACKET_SIZE_LARGE;

    // 调用回调接口, 将该封装好的 RTMPPacket 数据包放入 native-lib 类中的 线程安全队列中
    // 这是个 RTMPPacketPackUpCallBack 类型的函数指针
    // 这里不回调, 直接返回 rtmpPacket
    //mRtmpPacketPackUpCallBack(rtmpPacket);

    return rtmpPacket;
}
```
————————————————

版权声明：本文为CSDN博主「韩曙亮」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/shulianghan/article/details/106826659