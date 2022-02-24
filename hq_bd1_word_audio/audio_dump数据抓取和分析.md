# Audio Dump 数据抓取和分析



## 修订记录

| 日期      | 修订版本 | 修改描述                                             | 作者   |
| --------- | -------- | ---------------------------------------------------- | ------ |
| 2022/2/22 | V1.0     | 高通、联发科、展锐平台的audio dump数据抓取和分析初版 | 胡锦灿 |



[toc]



## 概述

主要介绍qcom、mtk、unsioc平台怎么抓取aduio dump log及dump log的大致分析思路

**注意：**

1. **audio dump log指的是音频在传输过程中抓取的音频数据文件，其有助于我们分析各种杂音噪音无声问题**
2. **抓取Audio dump log前需关闭所有系统提示音，以免抓取的dump中有杂音**
3. **为了准确定位问题，抓取前请删除先前dump log**



## QCOM 平台 audio dump 数据抓取和分析

### BSP 版本

#### AP dump log 抓取

高通平台抓取AP侧 dump log 需要高通提供diff，若需要请提case给高通



#### Audio ADSP dump log 抓取

1. 打开 QXDM 工具
2. "Ctrl + M" 打开配置栏，勾选入下：

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/qcom_AudioDumpDcm_Configure.png)

3. 点击 "Save As DMC" 按钮，保存配置文件为 Audio.dmc，以便于后面使用
4. "Ctrl + O" 打开配置栏，选择Audio.dmc文件
5. 连接手机端口

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/qcom_qxdm_ConnectDevice.png)

6. 复现问题
7. 导出 hdf 文件

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/qcom_qxdm_SaveAudioLog.png)

8. 使用高通 QCAT 工具解析 hdf 文件

**注意**：对于需要抓取更多详细log，高通会给特殊的dmc配置文件

详细见如下视频：[链接](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/高通QXDM抓取AudioADSPLog.mp4)


<video id="video" controls="" preload="none"> <source id="mp4" src="https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/高通QXDM抓取AudioADSPLog.mp4" type="video/mp4"> </video>



#### Audio ADSP dump log 分析

Audio ADSP 音频数据流在高通平台有两个通路，需要根据场景选择通路再对应各节点前后音频数据进行分析

如下图是播放音乐场景下抓取的音频 dump

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/qcom_qxdm_AudioLog节点.png)  

根据Music场景选择 audio path，框选出来的十六进制数代表 audio path 中音频数据传输过程中的节点

如 `RF_test08-*.hdf.0x152Epcm.*rx.wav` 代表 Audio decoder input (0x152E) 节点的数据

`RF_test08-*.hdf.0x1586pcm.*rx.wav`代表 AFE Rx port Output (0x1586) 节点的数据，以此类推

我们根据这些节点的数据就能很容易分析出问题的所在

Audio path (Muisc/VOIP/Record/Playback... ) 通路

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/qcom_adsp_dumplog_AudioPatch.png) 

Voice patch (Modem call) 通路：

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/qcom_adsp_dumplog_VoicePatch.png)

### SEP 版本

1. 先打开USB串口，拨号盘输入暗码 `*#0808#*`，选择 RMNET+DM+MODEM+ADPL+ADB
2. 使用 QXDM 工具抓取log
3. 复现问题

**注**：抓取分析方式同 bsp 版本



## MTK 平台 audio dump 数据抓取和分析

### BSP 版本

#### Playback dump 抓取

1. :telephone_receiver:拨号键输入暗码 `*#*#3646633#*#*`进入Engineer Mode
2. Hardware Testing -> Audio -> Audio Logger
3. :ballot_box_with_check:OutPut 和 AudioMixer

<img src="https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/mtk_open_playback_dump.png" style="zoom: 50%;" /> 

4. 复现问题

5. 导出dump log

```shell
adb pull /data/vendor/audiohal/audio_dump/
adb pull /data/debuglogger/audio_dump/
```

 ![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/mtk_playback_dump.png)



#### Recording dump log抓取

1. :telephone_receiver:拨号键输入暗码 `*#*#3646633#*#*`进入Engineer Mode
2. Hardware Testing -> Audio -> Audio Logger
3. :ballot_box_with_check:InPut 和 AudioMixer
4. 复现问题
5. 导出dump log

```shell
adb pull /data/vendor/audiohal/audio_dump/
adb pull /data/debuglogger/audio_dump/
```

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/mtk_recording_dump.png) 



#### Modem call dump log抓取

1. :telephone_receiver:拨号键输入暗码 `*#*#3646633#*#*`进入Engineer Mode
2. Hardware Testing -> Audio -> Speech Logger
3. :ballot_box_with_check:Enable speech log -> Normal-VM + EPL
4. 点击 "DUMP SPEECH DEBUG INFO" 按钮确保配置生效
4. 点击 "CLEAR HAL DUMP"按钮清除先前的log
5. Hardware Testing -> Audio -> Speech Enhancement
6. 确保Common Parameter -> Index 0 设置为6；Debug Info -> Index 0 设置为3

<img src="https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/mtk_audio_set_speechlog_enable.png" style="zoom:50%;" />                                      <img src="https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/mtk_audio_set_speech_enhancement.png" style="zoom:50%;" /> 

8. 复现问题

9. 导出VM log

```shell
adb pull /data/vendor/audiohal/audio_dump/
```

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/mtk_speech_dumplog.png) 



8. VM log解析

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/mtk_SpeechAnalyzer.png) 1、根据readme右键以管理员身份运行 run_as_administrator.bat；2、打开 SpeechAnalyzer 工具；3、选择vm文件；        					4、打开进行解析；5、等待解析完成

<img src="https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/run_mtk_SpeechAnalyzer_step.png"  /> 

<img src="https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/run_mtk_SpeechAnalyzer_step_2.png"  />

解析完成后如下图所示，按蓝色的播放按钮:play_or_pause_button:可播放解析后的音频，选择Play Mode可以只听下行或上行或一起听

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/run_mtk_SpeechAnalyzer_step_3.png)



**注意**

不同问题类型需要设置的Common Parameter0

| Common Parameter0 | 描述          |
| :---------------- | :------------ |
| Index 0 = 6       | 上下行场景    |
| Index 0 = 7       | 回声相关      |
| Index 0 = 11      | 双Mic降噪相关 |
| Index 0 = 12      | UL AGC相关    |



#### dump log 分析

**playback dump log音频处理节点参考如下**

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/mtk_ap_dump_节点.png)

`af_track_pcm.pcm` 是解码之后的数据（上层写给track的）

`mixer_drc_before.pcm` 和 `mixer_drc_after.pcm` 是DRC音频处理算法前后的数据

`af_mixer_resampler_in` 及 `af_mixer_resampler_out`是重采样前后的数据

`mixer_end.pcm` 是做完数字增益mixer等之后的数据

`af_mixer_pcm.pcm` 如果有音效处理的track，会进一步dump到这里，做进一步的音效处理

`af_mixer_write_write.pcm` 是Audio Framework 最终处理完的数据，会送到hal层做进一步的mtk处理

`streamout.pcm.AudioALSAPlaybackHandlerNormal.pcm` 是以32bit打开，但是有效的是24位，前8位要补回来，所以看这个节点的信号，需要整体放大48db去看

**注**：Recording dump log 的分析与 Playback 的相反，在此不多于赘述

**Modem call dump log**

Modem call 的上下行的各节点如下，但使用 SpeechAnalyzer 工具解析后最终只生成两个上下行的 wav 文件，如果听出上行或者下行有问题，请联系mtk进行分析

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/mtk_speech_dumplog节点对应关系.png) 

DL：网络下来的数据

DL0：网络下来的经过解码后的数据

DL1：解码后经过算法处理和的数据

UL：经过编码给到网络的数据

UL0：还没经过语音算法处理的数据

UL1：经过语音算法处理后的数据



### SEP (ENG & UD) 版本

#### 前置条件

1. 设置 --> 声音与振动 --> 系统声音/振动控制 --> 关闭所有功能

2. 将通知声音设置为静音
3. 关闭 DebugLoggerUI
4. 删除之前所有log

```shell
adb root
adb remount
adb shell rm -rf /data/log/audiopcm/*
adb shell rm -rf /data/debuglogger/*
adb shell rm -rf /data/vendor/audiohal/audio_dump/*
```

5. 打开动态log

```shell
adb shell setprop vendor.streamout.log 1
adb shell setprop vendor.streamin.log 1
```

#### Audio log 抓取

1. 打开DebugLoggerUI --> 设置中勾选MobileLog --> 点击开始录制log
2. 开启 audiohal dump

```shell
adb shell setenforce 0
adb shell "AudioSetParam AURISYS_SET_PARAM,HAL,ALL,MTKSE,ENABLE_LIB_DUMP,1=SET"
adb shell setprop vendor.streamout.pcm.dump 1
adb shell setprop vendor.streamin.pcm.dump 1
```

3. 开启 ss framework dump

`*#9900# --> AUDIOCORE DEBUG --> AUDIO PCM DUMP --> enable ALL`

4. 复现问题
5. 导出 log

```shell
adb pull /data/debuglogger/
adb pull /data/log/audiopcm/
adb pull /data/vendor/audiohal/audio_dump/
```

#### Speech log 抓取

1. 打开DebugLoggerUI --> 设置中勾选MobileLog + ModemLog --> 点击开始录制log

2. 开启 audiohal dump

```shell
adb shell setenforce 0
adb shell "AudioSetParam AURISYS_SET_PARAM,HAL,ALL,MTKSE,ENABLE_LIB_DUMP,1=SET"
adb shell setprop persist.vendor.audiohal.vm_cfg 1
```

3. 开启 ss framework dump

`*#9900# --> AudioCore Debug --> AUDIO PCM DUMP --> enable ALL
*#9900# --> AudioCore Debug --> AP CALL PCM DUMP --> 设置为开启状态`

4. 复现问题
5. 导出 log

```shell
adb pull /data/debuglogger/
adb pull /data/log/audiopcm/
adb pull /data/vendor/audiohal/audio_dump/
```

#### dump log 分析

audiohal dump 分析同上[链接](#dump log 分析)

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/mtk_SamsungSepHalAudioDump.png) 

ss framework dump 分析如下[链接](#SEP 版本 dump 数据分析)



## 展锐平台 audio dump 数据抓取和分析

### BSP 版本

#### AP dump log 抓取

##### 下行通道（playback）

1. 清除先前 log

```shell
adb root
adb remount
adb shell "rm -rf /data/vendor/local/media/*.pcm"
```

2. 重启 audioserver

```shell
adb shell stop audioserver
adb shell start audioserver
```

3. 打开 dump log

```shell
adb shell "echo set_dump_data_switch=0xffff > /data/vendor/local/media/mmi.audio.ctrl"
adb shell "echo 'vbc_pcm_dump=vbc_dac0' > /data/vendor/local/media/mmi.audio.ctrl"
```

4. 复现问题

5. 关闭 dump log

```shell
adb shell "echo set_dump_data_switch=0 > /data/vendor/local/media/mmi.audio.ctrl"
adb shell "echo 'vbc_pcm_dump=disbale' > /data/vendor/local/media/mmi.audio.ctrl"
```

6. 导出 dump log（pcm音频文件）

```shell
adb pull /data/vendor/local/media/
```

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/hal_vbc_dumplog.png) 

##### 上行通道（record）
1. 清除先前 log

```shell
adb root
adb remount
adb shell "rm -rf /data/vendor/local/media/*.pcm"
```

2. 重启 audioserver

```shell
adb shell stop audioserver
adb shell start audioserver
```

3. 打开 dump log

```shell
adb shell "echo set_dump_data_switch=0xffff > /data/vendor/local/media/mmi.audio.ctrl"
```

4. 复现问题
5. 关闭 dump log

```shell
adb shell "echo set_dump_data_switch=0 > /data/vendor/local/media/mmi.audio.ctrl"
```

6. 导出 dump log（pcm音频文件）

```shell
adb pull /data/vendor/local/media/
```

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/hal_record_dumplog.png) 

##### VOIP

抓取方式同下行通道

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/voip_dumplog.png) 

**注意：再次抓取dump log需要重启audioserver或机器重启**



#### AP dump log 分析

下行通道（playback）：

playback_hal_*.pcm：是音乐播放场景下AudioFlinger通过out_write函数(audio_hw.c)写入AudioHal的dump数据

playback_vbc_*.pcm：是AudioHal通过normal_out_write函数写给Alsa driver的dump数据

vbc_dump_vbc_dac0_*.pcm：是经过vbc硬件固化算法后的dump数据，将发送给codec或Smart PA

在Audio系统框架图中分析：

1. playback_hal可以理解为通道1的数据，此数据可能经过第三方的算法处理（看算法是加在抓取dump前还是后），若此数据有问题且经过第三方算法处理，可咨询展锐添加第三方算法处理前的dump抓取代码来排查算法问题
2. playback_vbc可以理解为通道2的数据，此数据是在out_write中调用normal_out_write写给Alsa driver的dump数据，其与playback_hal基本没有区别（不过在OT9上傅里叶算法是加在playback_hal后、playback_vbc前的，可以以此来区分算法差异）
3. vbc_dump可以理解为通道3的数据，是最接近PA并可dump出来的数据，数据再往下就被编codec解析成模拟信号并通过功放、播放设备播放出来

**Audio系统框架图**

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack@master/hq_bd1_img/Audio系统框架图_dump通路.jpg)

**UMS512 Audio 硬件结构简图**

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/Audio 硬件结构简图.png)

**注**

1. **pcm文件可通过Adobe Audition软件打开，可查看语音波形是否有断续、无声、杂音，并通过上图定位问题发生点**
2. **目前展锐的FAST给low-latency模式使用；PRIMARY给非low-latency模式使用；也就是说短小声音走fast其他均走primary**
3. **目前展锐的FAST，是ap直接送数据到vbc(voice band controler)，不经过dsp**
4. **如何查看当前的模式**

```shell
adb logcat -v threadtime|egrep -i "set_usecase"
02-17 07:05:35.308   475 11258 I audio_hw_control: set_usecase cur :0x0 usecase=0x100  on
02-17 07:05:38.611   475   504 I audio_hw_control: set_usecase cur :0x100 usecase=0x100  off
02-17 07:05:45.952   475 11034 I audio_hw_control: set_usecase cur :0x0 usecase=0x8  on
02-17 07:05:52.822   475   504 I audio_hw_control: set_usecase cur :0x8 usecase=0x8  off

02-17 07:33:38.236   475 11034 I audio_hw_control: set_usecase cur :0x1000 usecase=0x2  on
02-17 07:33:49.513   475   506 I audio_hw_control: set_usecase cur :0x1002 usecase=0x1000  off
02-17 07:33:49.640   475 11034 I audio_hw_control: set_usecase cur :0x2 usecase=0x2  off

usecase：0x200表示deepbuffer；0x100表示fast；0x8表示primary output；0x400表示compress offload；0X20普通录音
VOIP场景0x2 + 0x1000.
```



#### DSP dump log 抓取

打开ADSP log

方法一：

1. 打开 Ylog
2. 在出现的 "Ylog" 界面，选择 "Setting"，场景选择语音场景
3. 返回 "Ylog" 界面，注意此时语音场景按钮应为高亮状态

方法二：

```shell
# 打开log
adb root
adb shell cplogctl setcplog agdsp on
# 复现问题
# 导出log
adb pull /sdcard/log/cp/audio/
```

解析log

1. 打开展锐的Logel工具

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/logel.png) 

2. 点击:play_or_pause_button:按钮对log进行回放

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/unsioc_audio_adsplog解析_1.png)

3. 点击Tool->Audio->Audio Dsp Transfer 对log进行解析

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/unsioc_audio_adsplog解析_2.png)

4. 等待数秒解析成功后会自动打开AudioDsp文件夹，此时已解析成功

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/unsioc_audio_adsplog解析_3.png)



#### DSP dump log 分析

下面的 0001/0002/… 表示DSP的点，16k表示采样率，需要注意各个点位的文件大小要一致，否则有掉点

data-0001-16k-0000-time 是与时间打点的设置，该文件的大小可以不考虑。

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/unsioc_adsp_dump_log.png)

对于音频处理流程在ADSP中的平台，音频处理节点如下。

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack/hq_bd1_img/UMS512_DSPlog节点对应关系.png)

data-000A-16k-0000.pcm即为上图中的 10 点，代表副麦克输入数据

data-0001-16k-0000.pcm代表通话时从modem过来的语音数据

data-0002-16k-0000.pcm代表经过Decode解码之后的语音数据

data-0003-16k-0000.pcm代表经过CVS之后的语音数据

data-0004-16k-0000.pcm代表主麦克输入数据

data-0005-16k-0000.pcm代表经过算法后即将被Encode的语音数据

data-0006-16k-0000.pcm代表被Encode编码后将发送给modem的语音数据

data-0008-16k-0000.pcm代表经过CVS回声算法处理过后的语音数据（如果没问题的话这个点的数据明显感知到回声被消除了）

data-0009-16k-0000.pcm代表为回声参考信号

### SEP 版本

#### AP dump log 抓取

1. audio hal dump

   ENG 及 UD 版本 audiohal dump 抓取同 bsp [链接](#AP dump log 抓取-2)

   USER 版本无 root 权限无法抓取

2. ss framework dump

   抓取方法通用，如下所示 [链接](#SS framework dump 抓取)

#### DSP dump log 抓取

方法一

```shell
# 打开log
adb root
adb shell cplogctl setcplog agdsp on
# 复现问题
# 导出log
adb pull /sdcard/log/cp/audio/
```

方法二

```shell
# *#9900# --> Click 'SILENT LOG:OFF' and select 'ALL' to start logging
# 复现问题
# 导出log
adb pull /sdcard/log/cp/audio/
```



## SEP 版本 dump 数据抓取与分析

### SS framework dump 抓取

1. 清除先前的log

```shell
# ENG & UD 版本清除方式
adb root
adb remount
adb shell rm -rf /data/log/audiopcm/*
```

```shell
# User 版本清除方式
# *#9900# --> AUDIOCORE DEBUG --> AUDIO PCM DUMP --> Delete All Files
```

2. 开启 ss framework dump

`*#9900# --> AUDIOCORE DEBUG --> AUDIO PCM DUMP --> enable ALL`

3. 复现问题
4. 关闭 ss framework dump

`*#9900# --> AUDIOCORE DEBUG --> AUDIO PCM DUMP --> touch to turn off "All"`

5. 导出 log

```shell
# ENG & UD 版本导出方式
adb pull /data/log/audiopcm/
```

```shell
# User 版本导出方式
# *#9900# --> AUDIOCORE DEBUG --> AUDIO PCM DUMP --> Copy to SDCard
adb pull /sdcard/log/audiopcm
```

### dump 分析

**ss framework dump**

![](https://cdn.jsdelivr.net/gh/Herscon/Image-shack@master/hq_bd1_img/SamsungSepApAudioDump.png) 

`playback_proxy_audiotrack_session_*.pcm` 是App中 AudioTrack 的数据，即将发送给 Framework

`playback_track_session_*.pcm` 是App下发后的数据，在 Framework 层中 Audio Flinger 创建的 Track 中处理的数据

`playback_resampler_session_*_before_*.pcm` 及 `playback_resampler_session_*_after_*.pcm` 是重采样前后的数据

`playback_effects_dolby_fx_session_*_before_*.pcm` 及 `playback_effects_dolby_fx_session_*_after_*.pcm` 是杜比音效处理前后的数据

