===============
VENC
===============

简介
=====
VENC采用H264视频编码标准, 主要是以预测及运动补偿等方式进行压缩, 並以环路滤波提升画质, 兼顾码流传输和图像品质要求


主要特点
===========
- 1920x1080p@30fps + 640x480@30fps, BP/MP
- 输入: Semi-Planar YCbCr 4:2:0
- 输出: NALU(Network Abstract Layer Uint) in byte stream format
- CBR/VBR mode
- 最大8个ROI
- 最大16个OSD编码区域
- 支持软件模式和连动模式
- 可动态配置最大/最小量化參数
- 可动态配置I/P帧目标位元
- 可动态配置I帧距离
   
功能描述
============
软件模式和连动模式
----------
软件模式: 硬件以帧为单位和软件进行交互, 软件设定一个帧启动後硬件才压缩一帧, 完成後回传帧中断。输入帧內存需至少一帧。
连动模式: 软件设定H264_ENCODER_CTRL的位<CFG_S_ENC_SEQ_EN>或<CFG_ENC_SEQ_EN>來使能序列(sequence)編碼, 硬件自行侦测输入帧状态并进行压缩, 直到软件拉低序列使能, 硬件才停止压缩(结束於完整的一帧)並且回传序列中断。序列中每帧做完也会回传帧中断。

CBR/VBR模式
----------
CORE_REG23的<NUM_IMB_BITS>或<S_NUM_IMB_BITS>设0, 则该码流为VBR模式, 非0则为CBR模式; 在VBR模式需设定I/P帧固定量化参数(CORE_REG3/CORE_REG5), 不建议在硬件工作时切换模式。

双码流
----------
兩个码流的帧使能,帧中断,序列使能,序列中断是独立配置, 双流的帧率基本上需要一致, 但分辨率和其他压缩参数可以不同。

ROI(Region of Interest)
----------
单一码流最多可支援8个ROI区域,每个区域有各自的使能和起始/结束宏块位置设定, 需将H264_ROI_MODE的位<CFG_ROI_UPD>或H264_S_ROI_MODE的位<CFG_S_ROI_UPD>设1使新設定生效; 硬件則会每帧同步ROI设定。

OSD(On-Screen Display)编码区域
-------
硬件最多支持16个OSD区域, 软件设定H264_OSD_EN来决定每个码流OSD区域, OSD区域是以起始/结束宏块位置来表示。

动态配置最大/最小量化參数
-------
为提供更好的调控码率,软件设定CORE_REG31之後, 需设H264_ENCODER_CTRL(位<CFG_QR_UPD>或<CFG_S_QR_UPD>分别设定两个码流)使新设定值生效, 硬件会以GOP为单位来同步新设定值, 达成动态调整量化参数的范围。

动态配置I/P幀目标位元
-------
软件可以设定CORE_REG23/CORE_REG24来调整I/P帧的目标位元; CORE_REG23/CORE_REG24设定後需要再设H264_ENCODER_CTRL(位<CFG_QR_UPD>或<CFG_S_QR_UPD>]分别设定两个码流)让新的设定值生效, 硬件会以GOP为单位来同步新的设定值(CBR模式)。

动态配置I帧距离
-------
软件设定CORE_REG8之後再设定H264_ENCODER_CTRL(位<CFG_QR_UPD>或<CFG_S_QR_UPD>分别设定两个码流)让新的设定值生效, 硬件会以GOP为单位来同步新的设定值(CBR/VBR可用)。

视频序列中断
----------
在连动模式时使用, 当软件拉低序列使能後, 硬件处理当下该帧後发出序列中断(VDO_INT的位<S_SEQ_DONE_INT>或<SEQ_DONE_INT>), 此中断需要软件设定中断清除(VDO_INT_CLR的位<S_SEQ_DONE_INT_CLR>或<SEQ_DONE_INT_CLR>); 此中断可以使用遮挡(VDO_INT_MASK的位<S_SEQ_DONE_INT_MASK>或<SEQ_DONE_INT_MASK>)来忽略。

帧中断
----------
在软件模式和连动模式下只要硬件完成一帧压缩就会发出帧中断(VDO_INT的位<S_FRM_DONE_INT>或<FRM_DONE_INT>), 此中断需要软件设定中断清除(VDO_INT_CLR的位<S_FRM_DONE_INT_CLR>或<FRM_DONE_INT_CLR>); 此中断可以使用遮挡(VDO_INT_MASK的位<S_FRM_DONE_INT_MASK>或<FRM_DONE_INT_MASK>)来忽略。

输入缓存溢出状态警示
----------
如果发生视频压缩速度远低於Camera pixel输出时会发生缓存溢出情形, 软件可透过VDO_SRC_R_DBG(位<SRC_WR_OV_RD>)或VDO_S_SRC_R_DBG(位<S_SRC_WR_OV_RD>)的状态寄存器得知。