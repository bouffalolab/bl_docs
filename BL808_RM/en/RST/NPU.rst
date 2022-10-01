===============
NPU
===============

Overview
=====
The Neural Processing Unit (NPU), also called AI accelerator, is an electronic circuit dedicated to deep learning algorithms. It can implement all the control and arithmetic logic necessary to execute machine learning algorithms, usually with a separate data memory and special instruction set architecture. NPU aims to provide higher efficiency and performance for deep learning algorithms than the general central processing unit (CPU). NPU uses a large number of computing modules to utilize high data-level parallelism, a relatively large buffer/memory to apply the data reuse patterns, and a finite data width operator for deep learning fault tolerance.

Features
===========
- AI computing power: strong:`0.1 TOPS`
- Supports 8-bit operation
- Compatible with TensorFlowlite/ONNX/Caffe/Mxnet/Darknet/Pytorch model, and multiple frameworks
- Provide PC-side easy-to-use development tools, including model conversion, model quantification, performance prediction, and accuracy verification
- Supports single-layer instruction mode and multi-layer instruction mode
- Supports the maximum feature graph 4096 × 4096 × 4096 (width × height × depth)

Feature List
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

API Reference
===========

void gen_npu_inst_layer(npu_layer* l, bool use_tflite, bool unsgn_input, bool img_in)
::

    /**
    * function     Generate NPU instructions
    * @param[in]   l: NPU layer data structure indicators
    * @param[in]   use_tflite: whether to use tflite data format
    * @param[in]   unsgn_input: whether the input data is uint8
    * @param[in]   img_in: whether the input data is YUV400
    * @return      Null
    **/

bool check_BLAI_NPU_RUN(int type, int size, int stride, int dilation)
::

    /**
    * function     Confirm whether the layer is eligible for NPU acceleration
    * @param[in]   type: The layer type (enum LAYER_TYPE)
    * @param[in]   size: convolution kernel size
    * @param[in]   stride: step size
    * @param[in]   dilation: Atrous convolution kernel size
    * @return      true: NPU acceleration conditions are met, false: NPU acceleration conditions are not met
    **/	

bool BLAI_MEM_alloc(npu_layer l, PSRAM_ctrl *ctrl)
::

    /**
    * function     Automatically generate NPU related memory configuration
    * @param[in]   l: NPU layer data structure indicators
    * @param[in]   ctrl: memory configuration parameter structure
    * @return      true: memory configuration succeeded, false: memory configuration failed
    **/	

void fetch_BLAI_data_general(npu_layer *l, PSRAM_ctrl* ctrl, bool use_tflite, bool img_in)
::

    /**
    * function     Automatically generate NPU instructions (the maximum number of input layers is 2)
    * @param[in]   l: NPU layer data structure indicators
    * @param[in]   ctrl: memory configuration parameter structure
    * @param[in]   use_tflite: whether to use tflite data format
    * @param[in]   img_in: whether the input data is YUV400
    * @return      null
    **/	


void fetch_BLAI_data_route(npu_layer *l, PSRAM_ctrl* ctrl, bool use_tflite, bool img_in)
::

    /**
    * function     For ROUTE layers with more than two input layers, NPU commands are automatically generated
    * @param[in]   l: NPU layer data structure indicators
    * @param[in]   ctrl: memory configuration parameter structure
    * @param[in]   use_tflite: whether to use tflite data format
    * @param[in]   img_in: whether the input data is YUV400
    * @return      null
    **/	

bool BLAI_encode(npu_layer *l, PSRAM_ctrl* ctrl, int use_tflite, bool img_in)
::

    /**
    * function     Automatically generate NPU instructions, NPU-related memory configuration
    * @param[in]   l: NPU layer data structure indicators
    * @param[in]   ctrl: memory configuration parameter structure
    * @param[in]   use_tflite: whether to use tflite data format
    * @param[in]   img_in: whether the input data is YUV400
    * @return      true: NPU acceleration conditions are met and memory configuration is successful, false: NPU acceleration conditions are not met or memory configuration fails
    **/	

void Load_NPU_weights(npu_layer l, int8_t* WEI_buf, int* BIAS_buf, bool use_tflite)
::

    /**
    * function     The convolution parameters of this layer are stored in the NPU-specific parameter memory in the way specified by the NPU.
    * @param[in]   l: NPU layer data structure indicators
    * @param[in]   WEI_buf: NPU dedicated parameter memory
    * @param[in]   BIAS_buf: NPU dedicated parameter memory
    * @param[in]   use_tflite: whether to use tflite data format
    * @return      null
    **/	

void Store_tensor_data_to_NPU(npu_layer l, fixed_point_t* DATA_buf)
::

    /**
    * function     Store the tensor data in the operation graph into the dedicated data memory of the NPU
    * @param[in]   l: NPU layer data structure indicators
    * @param[in]   DATA_buf: NPU dedicated data memory
    * @return      null
    **/	

void Load_NPU_data_to_tensor(npu_layer l, fixed_point_t* DATA_buf)
::

    /**
    * function     Read tensor data from NPU dedicated data memory
    * @param[in]   l: NPU layer data structure indicators
    * @param[in]   DATA_buf: NPU dedicated data memory
    * @return      null
    **/	
	
void forward_NPU(npu_layer l, int8_t* DATA_buf, bool use_tflite)
::

    /**
    * function     Use the command to run the NPU (you need to use gen_npu_inst_layer to generate the command first)
    * @param[in]   l: NPU layer data structure indicators
    * @param[in]   DATA_buf: NPU dedicated data memory
    * @param[in]   use_tflite: whether to use tflite data format
    * @return      null
    **/	
	
Data Structure Reference
=============

:strong:`npu_layer data structure:`

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
        