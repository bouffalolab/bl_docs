===========
SEC ENG
===========

简介
=====

SEC ENG 内置了多种运算模块，包括 AES、SHA、CRC、GMAC、PKA、TRNG。

主要特征
=========
- AES

   - 支持 128 位、192位和256位密钥长度
   - 支持多种链接模式的加密和解密（ECB/CBC/CTR/XTS） 
   - 独有的 AES LINK 功能

- SHA

   - 支持SHA1/SHA224/SHA256/SHA384/SHA512
   - 独有的 SHA LINK 功能

- 支持MD5，CRC-16，CRC-32

功能描述
===========

AES 加速器
-------------

密钥
^^^^^^^^^^

通过配置寄存器se_aes_0_ctrl中的se_aes_0_mode和se_aes_0_dec_en 可以选择加密模式和解密模式时需要的密钥长度。

.. figure:: ../../picture/SecEngAesMode.svg
   :align: center

   AES运算模式图

通过配置寄存器 se_aes_0_ctrl 中的 aes_0_hw_key_en 选择是否开启硬件密钥，如果使用的是软件密钥，则还需要配置寄存器se_aes_0_key_0~se_aes_0_key_7用于存放密钥，每个寄存器存放4个字节的密钥。

链接模式
^^^^^^^^^^

通过配置寄存器se_aes_0_ctrl中的se_aes_0_block_mode可以选择不同的链接模式，目前支持ECB、CBC、CTR、XTS模式。

明文、密文
^^^^^^^^^^^^^^^^^^

- 明文或者密文需要是 16 的倍数
- 寄存器se_aes_0_msa 存放加密时输入的明文地址或者解密时输入的密文地址
- 寄存器se_aes_0_mda 存放加密时输出的密文地址或者解密时输出的明文地址
- 寄存器se_aes_0_ctrl中的 se_aes_0_msg_len 用来设定密文或者明文长度（以 16字节为单位）

初始化向量
^^^^^^^^^^^^^^^^^^

寄存器se_aes_0_iv_0~se_aes_0_iv_3存放初始化向量 IV，通过配置 se_aes_0_ctrl 中的 aes_0_iv_sel 选择是否使用新的 iv，第一次配置 iv 时需要清0，后续一直使用此 iv 或者自动更新 iv 则需要置1。

加解密配置流程
^^^^^^^^^^^^^^^^^^

- 配置寄存器se_aes_0_ctrl中的 se_aes_0_en 使能 AES
- 配置寄存器se_aes_0_endian，其中包括se_aes_0_dout_endian、se_aes_0_din_endian、se_aes_0_key_endian、se_aes_0_iv_endian、se_aes_0_twk_endian，若值为0表示little-endian，值为1表示big-endian
- 配置寄存器se_aes_0_ctrl中的se_aes_0_block_mode 选择链接模式
- 配置寄存器se_aes_0_ctrl中的se_aes_0_mode 选择密钥长度
- 使用软件密钥则配置寄存器se_aes_0_key_0~se_aes_0_key_7 存放密钥，使用硬件密钥则置位寄存器 se_aes_0_ctrl 中的 aes_0_hw_key_en
- 配置寄存器se_aes_0_iv_0~se_aes_0_iv_3设置IV，MSB时填写顺序为se_aes_0_iv_0~se_aes_0_iv_3，LSB时填写顺序为se_aes_0_iv_3~se_aes_0_iv_0
- 配置寄存器se_aes_0_ctrl中的se_aes_0_dec_en 选择加密或者解密模式
- 配置寄存器se_aes_0_msa设置待处理数据的源地址
- 配置寄存器se_aes_0_mda设置处理结果存放的目的地址
- 配置寄存器se_aes_0_ctrl中的se_aes_0_msg_len设置待处理数据的长度，以16字节为单位
- 配置寄存器se_aes_0_ctrl中的se_aes_0_trig_1t触发AES运行

结果会输出到 se_aes_0_mda 指定的目标地址。

SHA 加速器
-------------

SHA 模式
^^^^^^^^^^

- 寄存器se_sha_0_ctrl中的se_sha_0_mode用于选择SHA的具体模式: 0:SHA-256 1:SHA-224 2:SHA-1 3:SHA-1 4:SHA-512 5:SHA-384 6:SHA-512/224 7:SHA-512/256
- 寄存器se_sha_0_ctrl中的se_sha_0_mode_ext由于选择扩展模式，如果不使用扩展模式应设为0：0:SHA 1:MD5 2:CRC-16 3:CRC-32

明文、密文
^^^^^^^^^^^^^^^^^^

- 寄存器se_sha_0_msa存放明文地址。
- 寄存器se_sha_0_hash_l_0~se_sha_0_hash_l_7存放密文。

配置流程
^^^^^^^^^^^^^^^^^^

 - 配置寄存器se_sha_0_ctrl中的se_sha_0_mode选择SHA的具体模式
 - 配置寄存器se_sha_0_ctrl中的se_sha_0_en使能SHA
 - 配置寄存器se_sha_0_ctrl中的se_sha_0_hash_sel，0表示开始新的HASH计算，1表示沿用上一次的结果进行HASH计算
 - 配置寄存器se_sha_0_msa 设置输入数据的地址
 - 配置寄存器se_sha_0_ctrl中的se_sha_0_msg_len待处理数据的长度，SHA-1、SHA-224、SHA-256以512-bit为单位，SHA-512、SHA-384、SHA-512/224、SHA-512/256以1024-bit为单位
 - 配置寄存器se_sha_0_ctrl中的se_sha_0_trig_1t触发SHA运行
 - 结果输出存储在se_sha_0_hash_l_0~se_sha_0_hash_l_7中（默认MSB），MSB:se_sha_0_hash_l_0~se_sha_0_hash_l_7，LSB:se_sha_0_hash_l_7~se_sha_0_hash_l_0

TRNG
-----------------

SEC ENG 内置一个TRNG(真随机数发生器)，其生成的随机数可作为加密等操作的基础。

- 真随机数：真正的随机数是使用物理现象产生的：比如掷钱币、骰子、转轮、使用电子元件的噪音、核裂变等等，这样的随机数发生器叫做物理性随机数发生器，它们的缺点是技术要求比较高。
- 伪随机数：真正意义上的随机数（或者随机事件）在某次产生过程中是按照实验过程中表现的分布概率随机产生的，其结果是不可预测的，是不可见的。而计算机中的随机函数是按照一定算法模拟产生的，其结果是确定的，是可见的。我们可以认为这个可预见的结果其出现的概率是100%。所以用计算机随机函数所产生的“随机数”并不随机，是伪随机数。

配置流程
^^^^^^^^^^^^^^^^^^

 - 配置寄存器se_trng_0_ctrl_0中的se_trng_0_en使能TRNG
 - 配置寄存器se_trng_0_ctrl_0中的se_trng_0_trig_1t触发TRNG运行
 - 结果输出存储在se_trng_0_dout_0~se_trng_0_dout_7中

