
# 【Android RTMP】音频数据采集编码 ( FAAC 音频编码参数设置 | FAAC 编码器创建 | 获取编码器参数 | 设置 AAC 编码规格 | 设置编码器输入输出参数 ) #

本博客中讲解的是 , PCM 音频采样编码为 AAC 音频 , 如何设置 FACC 编码器参数 ;





## 一、 头文件、成员变量准备 ##

1 . 导入包 : 使用 FACC 编码器前 , 必须导入 facc.h 头文件 ;

	#include <faac.h>


2 . 成员变量定义 : 在初始化 FACC 编码器时 , 需要预先定义一些成员变量 , 这些变量在后续设置编码器参数 , 音频编码时都需要使用到 ;


① 输入样本个数 : 输入到 FAAC 编码器中的需要进行编码的 PCM 样本个数 , 表示FAAC 编码器最多一次可以接收的样本个数 ,

	unsigned long mInputSamples;

注意 : 是样本个数 , 不是字节数 , 如果采样位数是 8 88 位 , 那么 字 节 数 = 样 本 个 数 字节数 = 样本个数字节数=样本个数 , 如果采样位数是 16 1616 位 , 字 节 数 = 样 本 个 数 × 2 字节数 = 样本个数 \times 2字节数=样本个数×2 ;


② FAAC 编码器最大输出字节数 : 该参数 mMaxOutputBytes 与上面的 mInputSamples 都要传入 FAAC 编码器创建函数中 , 用于接收创建 FAAC 编码器后的返回值 , 创建之前这些值是不知道的 ;

	unsigned long mMaxOutputBytes;

③ FAAC 编码器 : 用于将 PCM 音频采样编码成 AAC 格式 ;

	faacEncHandle mFaacEncHandle;

④ FAAC 编码输出缓冲区 : FAAC 编码后的 AAC 裸数据, 存储到该缓冲区中 , 该缓冲区在初始化 FAAC 编码器时创建 ; 初始化完成后 , 知道 FAAC 最大输出缓冲区大小后 , 创建该输出缓冲区 , 其大小是 mMaxOutputBytes 字节 ;

	unsigned char* mFaacEncodeOutputBuffer;





## 二、 创建 FAAC 编码器 ##

1 . 调用 faacEncOpen 函数 , 创建 FAAC 编码器 ;


2 . 函数原型解析 :

```
faacEncHandle FAACAPI faacEncOpen(unsigned long sampleRate,
				  unsigned int numChannels,
				  unsigned long *inputSamples,
                                  unsigned long *maxOutputBytes
                                 );
```

① unsigned long sampleRate 参数 : 音频采样率 ;

② unsigned int numChannels 参数 : 音频通道参数 ;

③ unsigned long *mInputSamples 参数 : 输入样本个数, 需要进行编码的 PCM 音频样本个数 , FAAC 编码器最多一次可以接收的样本个数 ;

④ unsigned long *mMaxOutputBytes 参数 : 输出数据最大字节数 ;

⑤ faacEncHandle 返回值 : FAAC 音频编码器 ;


3 . 创建 FAAC 编码器 代码示例 :

```
mFaacEncHandle = faacEncOpen(sampleRateInHz, channelConfig, &mInputSamples, &mMaxOutputBytes);
```




## 三、 获取并设置 FAAC 编码器参数 ##

1 . 获取编码器参数 : 先获取参数, 然后设置参数, 最后再设置回去
```
faacEncConfigurationPtr configurationPtr = faacEncGetCurrentConfiguration(mFaacEncHandle);
```

2 . 设置编码器参数 : 为 mFaacEncHandle 音频编码器设置 configurationPtr 编码器参数
```
faacEncSetConfiguration(mFaacEncHandle, configurationPtr);
```

先获取 FAAC 编码器参数 faacEncConfigurationPtr 结构体 , 然后设置编码器参数 , 最后再将编码器参数 设置回 FAAC 编码器 FaacEncHandle ;





## 四、 设置 FAAC 编码器编码标准 ##

设置 FAAC 编码器编码标准 : 可以设置 MPEG2 , 或 MPEG4 , 目前一般设置 MPEG4 标准 ;
```
// 设置编码格式标准, 使用 MPEG4 新标准
configurationPtr->mpegVersion = MPEG4;
```




## 五、 设置 FAAC 编码器 AAC 编码规格 ##

1 . AAC 编码规格 : 9 99 种 ;

- MPEG-2 AAC LC低复杂度规格（Low Complexity）
- MPEG-2 AAC Main主规格
- MPEG-2 AAC SSR可变采样率规格（Scaleable Sample Rate）
- MPEG-4 AAC LC低复杂度规格（Low Complexity）
- MPEG-4 AAC Main主规格, MPEG-4 AAC SSR可变采样率规格（Scaleable Sample Rate）
- MPEG-4 AAC LTP长时期预测规格（Long Term Predicition）
- MPEG-4 AAC LD低延迟规格（Low Delay）
- MPEG-4 AAC HE高效率规格（High Efficiency）

2 . 这里选择低复杂度规格 : MPEG-4 AAC LC低复杂度规格（Low Complexity） 是最常用的 AAC 编码规格, 即兼顾了编码效率, 有保证了音质;

( 使用更高音质, 极大降低编码效率, 音质提升效果有限 ; 再提升编码效率, 会使音质降低很多 )

	configurationPtr->aacObjectType = LOW;




六、 设置 FAAC 编码器输入、输出格式

1 . 设置编码器的输入格式 : 这里设置输入的 PCM 的采样位数是 16 1616 位 ;

	configurationPtr->inputFormat = FAAC_INPUT_16BIT;

2 . 设置编码器的输出格式 : 这里设置输出格式 0, 就是 FAAC 将 PCM 采样进行编码, 编码出的格式是 AAC 原始数据 , 即没有解码信息的 ADIF 和 ADTS 的 AAC 纯样本裸数据 ;

	configurationPtr->outputFormat = 0;


参考 【Android RTMP】音频数据采集编码 ( AAC 音频格式解析 | FLV 音频数据标签解析 | AAC 音频数据标签头 | 音频解码配置信息 ) 博客的 AAC 格式音频解析 , AAC 有两种格式 :

- 音频数据交换格式 ( Audio Data Interchange Format )
- 音频数据传输流格式 ( Audio Data Transport Stream )

此处使用的不是上述两种格式的任意一种 , 而是 AAC 的纯样本裸数据 ;





## 七、 FAAC 设置音频编码参数代码 ##

1 . 成员变量定义代码 :

```
    /**
     * 输入样本个数, 需要进行编码的 PCM 音频样本个数
     * FAAC 编码器最多一次可以接收的样本个数
     * 传递下面两个数值的地址到 faacEncOpen 函数中, 用于当做返回值使用
     *
     * 该数据需要返回给 Java 层
     * Java 层每次从 AudioRecord 中读取 mInputSamples 个数据
     */
    unsigned long mInputSamples;

    /**
     * FAAC 编码器最多一次可以接收的样本个数
     * 传递下面两个数值的地址到 faacEncOpen 函数中, 用于当做返回值使用
     */
    unsigned long mMaxOutputBytes;

    /**
     * PCM 音频 FAAC 编码器
     * 将 PCM 采样数据编码成 FAAC 编码器
     */
    faacEncHandle mFaacEncHandle;

    /**
     * FAAC 编码输出缓冲区
     * FAAC 编码后的 AAC 裸数据, 存储到该缓冲区中
     * 该缓冲区在初始化 FAAC 编码器时创建
     */
    unsigned char* mFaacEncodeOutputBuffer;
```

2 . FACC 编码器参数初始化代码 :

```
/**
 * 设置音频编码参数
 * @param sampleRateInHz    音频采样率
 * @param channelConfig     音频采样通道, 单声道 / 立体声
 */
void AudioChannel::setAudioEncoderParameters(int sampleRateInHz, int channelConfig) {
    // 设置音频通道参数, 单声道 / 立体声
    mChannelConfig = channelConfig;

    /*
        打开编码器
        faacEncHandle FAACAPI faacEncOpen(unsigned long sampleRate,
				  unsigned int numChannels,
				  unsigned long *mInputSamples,
                  unsigned long *mMaxOutputBytes);
        unsigned long sampleRate 参数 : 音频采样率
        unsigned int numChannels 参数 : 音频通道参数
        unsigned long *mInputSamples 参数 : 输入样本个数, 需要进行编码的 PCM 音频样本个数
            FAAC 编码器最多一次可以接收的样本个数
        unsigned long *mMaxOutputBytes 参数 : 输出数据最大字节数

        faacEncHandle 返回值 : FAAC 音频编码器

        注意上述 样本个数, 字节个数 区别 : 16 位采样位数, 一个样本有 2 字节
     */
    mFaacEncHandle = faacEncOpen(sampleRateInHz, channelConfig, &mInputSamples, &mMaxOutputBytes);

    // 获取编码器当前参数
    // 先获取参数, 然后设置参数, 最后再设置回去
    faacEncConfigurationPtr configurationPtr = faacEncGetCurrentConfiguration(mFaacEncHandle);

    // 设置编码格式标准, 使用 MPEG4 新标准
    configurationPtr->mpegVersion = MPEG4;

    /*
        设置 AAC 编码规格, 有 9 种
        MPEG-2 AAC LC低复杂度规格（Low Complexity）, MPEG-2 AAC Main主规格, MPEG-2 AAC SSR可变采样率规格（Scaleable Sample Rate）
        MPEG-4 AAC LC低复杂度规格（Low Complexity）, MPEG-4 AAC Main主规格, MPEG-4 AAC SSR可变采样率规格（Scaleable Sample Rate）
        MPEG-4 AAC LTP长时期预测规格（Long Term Predicition）, MPEG-4 AAC LD低延迟规格（Low Delay）, MPEG-4 AAC HE高效率规格（High Efficiency）
        这里选择低复杂度规格, 可选参数有 4 种
        MPEG-4 AAC LC低复杂度规格（Low Complexity） 是最常用的 AAC 编码规格, 即兼顾了编码效率, 有保证了音质;
        使用更高音质, 极大降低编码效率, 音质提升效果有限
        再提升编码效率, 会使音质降低很多
     */
    configurationPtr->aacObjectType = LOW;

    // 采样位数 16 位
    configurationPtr->inputFormat = FAAC_INPUT_16BIT;

    /*
        AAC 音频文件有两种格式 ADIF 和 ADTS
        AAC 文件解码时 : 音频解码信息定义在头部, 后续音频数据解码按照音频数据长度

        音频数据交换格式 ( Audio Data Interchange Format ) , 只有一份音频解码信息 , 存储在文件开头
        这种格式适合存储音频文件 , 节省空间 , 但是必须从开始播放才可以 , 从中间位置无法播放 ;

        音频数据传输流格式 ( Audio Data Transport Stream ) , 每隔一段音频数据
        就会有一份音频解码信息 , 这种格式适合音频流传输 , 可以在任何位置开始解码播放 ;

        RTMP 推流时, 不使用上述两种格式
        推流视频时, 先将 SPS, PPS 解码数据包的信息推流到服务器上
        推流音频时, 也是将解码相关的数据先推流到服务器中

        AAC 编码时, 会编码成 ADTS 数据
        但是推流音频时, 推流的是 AAC 裸数据, 需要将 ADTS 音频格式中的头信息去掉

        博客中截图 FLV 第一帧 AAC 音频数据标签 和 后续 AAC 音频数据标签

        这里设置输出格式 0, 就是 FAAC 将 PCM 采样进行编码, 编码出的格式是 AAC 原始数据
        即没有解码信息的 ADIF 和 ADTS 的 AAC 纯样本裸数据
     */
    configurationPtr->outputFormat = 0;

    // 为 mFaacEncHandle 音频编码器设置 configurationPtr 编码器参数
    faacEncSetConfiguration(mFaacEncHandle, configurationPtr);

    // 初始化输出缓冲区, 保存 FAAC 编码输出数据
    mFaacEncodeOutputBuffer = new unsigned char[mMaxOutputBytes];
}
```

————————————————

版权声明：本文为CSDN博主「韩曙亮」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/shulianghan/article/details/106817750