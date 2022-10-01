===============
NPU
===============

简介
=====
神经处理单元 (NPU) 或简称为 AI 加速器是一种专用电路，可实现执行机器学习算法所需的所有必要控制和算术逻辑
，是为深度学习算法设计的电子电路，通常具有单独的数据存储器和专用指令集架构。NPU的目标是为深度学习算法提供
比一般中央处理单元(CPU)更高的效率和性能。NPU使用大量计算组件来利用高数据级并行性，使用相对较大的缓冲区/内存
来利用数据重用模式，以及用于深度学习容错的有限数据宽度运算符。

主要特点
===========
- AI算力 :strong:`0.1 TOPS`
- 支持8-bit运算
- 兼容TensorFlow-lite/ONNX/Caffe/Mxnet/Darknet/Pytorch模型，可支持多类框架
- 提供PC端方便易用的开发工具，包含模型转换、模型量化、性能预估、精度验证
- 支持单层指令模式、多层指令模式
- 支持最大特征图4096 x 4096 x 4096(宽、高、深度)

功能列表
===========
+-----------------+-------------------+--------------------------+----------------+
| Type            | Operators         | Applicable Subset Spec.  | Processor      |
+=================+===================+==========================+================+
| Convolution     | Conv              | 1x1 up to 7x7            | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | Depthwise Conv    | 1x1 up to 7x7            | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | Pad               | same                     | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | Deconvolution     | use Upsample + Conv      | :strong:`NPU`  |     
+-----------------+-------------------+--------------------------+----------------+
| Pooling         | MaxPool(2x2)      | Stride 2                 | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | MaxPool(3x3)      | 3x3                      | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | AveragePool       | Stride 1, 2              | DSP            |
+                 +-------------------+--------------------------+----------------+
|                 | GlobalAveragePool |                          | DSP            |
+                 +-------------------+--------------------------+----------------+
|                 | GlobalMaxPool     |                          | DSP            |
+-----------------+-------------------+--------------------------+----------------+
| Activation      | Relu              |                          | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | LeakyRelu         |                          | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | Relu-n            | n > 0 (Relu6)            | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | Mish              | Look up table            | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | ELU               | Look up table            | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | PRelu             |                          | DSP            |
+                 +-------------------+--------------------------+----------------+
|                 | Sigmoid           |                          | DSP            |
+-----------------+-------------------+--------------------------+----------------+
| Other processing| BatchNormalization|                          | DSP            |
+                 +-------------------+--------------------------+----------------+
|                 | Add (shortcut)    |                          | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | Concat (route)    |                          | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | Fully Connected   |                          | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | Mul               |                          | :strong:`NPU`  |
+                 +-------------------+--------------------------+----------------+
|                 | Slice             |                          | DSP            |  
+-----------------+-------------------+--------------------------+----------------+

API参考
===========

void gen_npu_inst_layer(npu_layer* l, bool use_tflite, bool unsgn_input, bool img_in)
::

    /**
    * function     产生NPU指令
    * @param[in]   l: NPU层数据结构指标
    * @param[in]   use_tflite: 是否使用tflite数据格式
    * @param[in]   unsgn_input: 输入数据是否为uint8
    * @param[in]   img_in: 输入数据是否为YUV400
    * @return      空
    **/

bool check_BLAI_NPU_RUN(int type, int size, int stride, int dilation)
::

    /**
    * function     确认该层是否符合NPU加速条件
    * @param[in]   type: 该层类别 (enum LAYER_TYPE)
    * @param[in]   size: 卷积核大小
    * @param[in]   stride: 步伐大小
    * @param[in]   dilation: 空洞卷积核大小
    * @return      true: 符合NPU加速条件, false: 未符合NPU加速条件
    **/	

bool BLAI_MEM_alloc(npu_layer l, PSRAM_ctrl *ctrl)
::

    /**
    * function     自动产生NPU相关记忆体配置
    * @param[in]   l: NPU层数据结构指标
    * @param[in]   ctrl: 记忆体配置参数结构
    * @return      true: 记忆体配置成功, false: 记忆体配置失败
    **/	

void fetch_BLAI_data_general(npu_layer *l, PSRAM_ctrl* ctrl, bool use_tflite, bool img_in)
::

    /**
    * function     自动产生NPU指令 (输入层数量最多为2)
    * @param[in]   l: NPU层数据结构指标
    * @param[in]   ctrl: 记忆体配置参数结构
    * @param[in]   use_tflite: 是否使用tflite数据格式
    * @param[in]   img_in: 输入数据是否为YUV400
    * @return      空
    **/	


void fetch_BLAI_data_route(npu_layer *l, PSRAM_ctrl* ctrl, bool use_tflite, bool img_in)
::

    /**
    * function     针对输入层数量大于两层的ROUTE层，自动产生NPU指令
    * @param[in]   l: NPU层数据结构指标
    * @param[in]   ctrl: 记忆体配置参数结构
    * @param[in]   use_tflite: 是否使用tflite数据格式
    * @param[in]   img_in: 输入数据是否为YUV400
    * @return      空
    **/	

bool BLAI_encode(npu_layer *l, PSRAM_ctrl* ctrl, int use_tflite, bool img_in)
::

    /**
    * function     自动产生NPU指令、NPU相关记忆体配置
    * @param[in]   l: NPU层数据结构指标
    * @param[in]   ctrl: 记忆体配置参数结构
    * @param[in]   use_tflite: 是否使用tflite数据格式
    * @param[in]   img_in: 输入数据是否为YUV400
    * @return      true: 符合NPU加速条件且记忆体配置成功, false: 未符合NPU加速条件或记忆体配置失败
    **/	

void Load_NPU_weights(npu_layer l, int8_t* WEI_buf, int* BIAS_buf, bool use_tflite)
::

    /**
    * function     将该层卷积参数照NPU指定方式存储入NPU专用参数记忆体
    * @param[in]   l: NPU层数据结构指标
    * @param[in]   WEI_buf: NPU专用参数记忆体
    * @param[in]   BIAS_buf: NPU专用参数记忆体
    * @param[in]   use_tflite: 是否使用tflite数据格式
    * @return      空
    **/	

void Store_tensor_data_to_NPU(npu_layer l, fixed_point_t* DATA_buf)
::

    /**
    * function     将运算图中的张量数据存储入NPU专用数据记忆体
    * @param[in]   l: NPU层数据结构指标
    * @param[in]   DATA_buf: NPU专用数据记忆体
    * @return      空
    **/	

void Load_NPU_data_to_tensor(npu_layer l, fixed_point_t* DATA_buf)
::

    /**
    * function     从NPU专用数据记忆体读取张量数据
    * @param[in]   l: NPU层数据结构指标
    * @param[in]   DATA_buf: NPU专用数据记忆体
    * @return      空
    **/	
	
void forward_NPU(npu_layer l, int8_t* DATA_buf, bool use_tflite)
::

    /**
    * function     使用指令运行NPU (需先使用gen_npu_inst_layer产生指令)
    * @param[in]   l: NPU层数据结构指标
    * @param[in]   DATA_buf: NPU专用数据记忆体
    * @param[in]   use_tflite: 是否使用tflite数据格式
    * @return      空
    **/	
	
数据结构参考
=============

:strong:`npu_layer数据结构：`

::

    /** NPU layer information of dynamic fixed point format(CMSIS/NMSIS) */                 
    
    struct npu_layer {

        ///////////////////////////////////////////////
        /////    Begin of user define region      /////
        ///////////////////////////////////////////////

        /** operation type (enum LAYER_TYPE)*/
        uint8_t type;

        /** activation type (enum ACTIVATION)*/
        uint8_t activation;

        /** layer width */
        uint16_t w;

        /** layer height */
        uint16_t h;

        /** layer channel (1) */
        uint16_t c;

        /** extra layer channel (2-8) */
        uint16_t cn[7];

        /** layer output width */
        uint16_t out_w;

        /** layer output height */
        uint16_t out_h;

        /** layer output channel */
        uint16_t out_c;

        /** number of input layers*/
        uint8_t input_num;

        /** CONV kernel size */
        uint8_t size;

        /** CONV groups size */
        uint16_t groups;

        /** CONV dilation size */
        uint8_t dilation;

        /** stride size */
        uint8_t stride;

        /** input tensor type*/
        uint8_t input_type;

        /** True: combined layer need to keep output data */
        bool mid_out;

        /** True: input image is 1-channel */
        bool img_in;

        /** dynamic fixed point format(CMSIS/NMSIS) for input data */
        int8_t fdata;

        /** dynamic fixed point format(CMSIS/NMSIS) for weight */
        int8_t fweight;

        /** dynamic fixed point format(CMSIS/NMSIS) for bias */
        int8_t fbias;

        /** dynamic fixed point format(CMSIS/NMSIS) for output data */
        int8_t fout;

        /** dynamic fixed point format(CMSIS/NMSIS) for route input data (1) */
        int8_t froute1;

        /** dynamic fixed point format(CMSIS/NMSIS) for route input data (2) */
        int8_t froute2;

        /** dynamic fixed point format(CMSIS/NMSIS) for route input data (3-8) */
        int8_t frouten[6];

        /** Tensorflow-Lite input offset (1) */
        uint8_t tf_input1_offset;

        /** Tensorflow-Lite input offset (2) */
        uint8_t tf_input2_offset;

        /** Tensorflow-Lite input offset (3-8) */
        uint8_t tf_input_offset_extra[6];

        /** Tensorflow-Lite output offset */
        uint8_t tf_output_offset;

        /** Tensorflow-Lite input shift (1) */
        int8_t tf_input1_shift;

        /** Tensorflow-Lite input shift (2) */
        int8_t tf_input2_shift;

        /** Tensorflow-Lite input shift (3-8) */
        int8_t tf_input_shift_extra[6];

        /** Tensorflow-Lite output shift */
        int8_t tf_output_shift;

        /** Tensorflow-Lite quantized_activation_min */
        int16_t quantized_activation_min;

        /** Tensorflow-Lite quantized_activation_max */
        int16_t quantized_activation_max;

        /** Tensorflow-Lite input multiplier (1) */
        int tf_input1_multiplier;

        /** Tensorflow-Lite input multiplier (2) */
        int tf_input2_multiplier;

        /** Tensorflow-Lite input multiplier (3-8) */
        int tf_input_multiplier_extra[6];

        /** Tensorflow-Lite output multiplier */
        int tf_output_multiplier;

        /** Tensorflow-Lite route input multiplier */
        int tf_route_input_multiplier;

        /** Tensorflow-Lite route input multiplier */
        int tf_route_input_shift;

        /** Tensorflow-Lite left shift */
        int8_t tf_left_shift;

        /** pointer of input data buffers */
        int8_t* input_i8[8];

        /** pointer of output data buffer */
        int8_t* output_i8;

        /** pointer of combined layer output data buffer */
        int8_t* mid_output_i8;

        /** pointer of weight buffer */
        int* weights;

        /** pointer of bias buffer */
        int* biases;

        ///////////////////////////////////////////////
        /////     End of user define region       /////
        ///////////////////////////////////////////////	

        /** pointer of NPU instruction buffer */
        uint8_t* NPU_inst;

        /** flag of NPU processor */
        bool NPU_on;

        /** memory patch size*/
        uint32_t DRAM_patch_size;

        /** memory patch location for input data */
        uint16_t DRAM_in[8];

        /** memory patch location for output data */
        uint16_t DRAM_out[8];

        /** memory patch location for mid output data */
        uint16_t DRAM_mid_out;

        /** memory patch location for weight data */
        uint16_t DRAM_weight;

        /** size of weight data */
        int DRAM_nweight;

        /** memory patch location for bias data */
        uint16_t DRAM_bias;

        /** size of bias data */
        int DRAM_nbias;

        /** uint8_t input data */
        bool unsgn_input;

        /** number of instruction */
        uint8_t inst_cnt;
	};
        