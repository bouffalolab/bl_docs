===========
SEC ENG
===========

简介
=====
AES简介
-------------
高级加密标准(AES,Advanced Encryption Standard)为最常见的对称加密算法。在密码学中又称Rijndael加密法，是美国联邦政府采用的一种区块加密标准。

MD5简介
-------------
MD5信息摘要算法（英语：MD5 Message-Digest Algorithm），一种被广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值（hash value），用于确保信息传输完整一致。

SHA简介
-------------
SHA256是SHA-2下细分出的一种算法,SHA-2，名称来自于安全散列算法2（英语：Secure Hash Algorithm 2）的缩写，一种密码散列函数算法标准，由美国国家安全局研发，属于SHA算法之一，是SHA-1的后继者。
SHA-2下又可再分为六个不同的算法标准,包括了：SHA-224、SHA-256、SHA-384、SHA-512、SHA-512/224、SHA-512/256。其中SHA-512/224指结果取SHA-512的前224个bit，SHA-512/256指结果取SHA-512的前256个bit。

CRC简介
-------------
循环冗余校验（Cyclic Redundancy Check， CRC）是一种根据网络数据包或计算机文件等数据产生简短固定位数校验码的一种信道编码技术，主要用来检测或校验数据传输或者保存后可能出现的错误。它是利用除法及余数的原理来作错误侦测的。

GMAC简介
-------------
GMAC就是利用伽罗华域(Galois Field，GF，有限域)乘法运算来计算消息的MAC值。

主要特征
=========
- 支持aes-128，aes-192，aes-256加解密
- 支持crc-16，crc-32， md5，sha-256, sha-512
- 支持trng
- 支持gmac

原理描述
===========
AES对称加密算法也就是加密和解密用相同的密钥，加密流程如下图：

.. figure:: ../../picture/SecEngAes.svg
   :align: center

   AES加密流程图

下面简单介绍下各个部分的作用与意义：

 - 明文P：没有经过加密的数据。
 - 密钥K：用来加密明文的密码，在对称加密算法中，加密与解密的密钥是相同的。密钥为接收方与发送方协商产生，但不可以直接在网络上传输，否则会导致密钥泄漏，通常是通过非对称加密算法加密密钥，然后再通过网络传输给对方，或者直接面对面商量密钥。密钥是绝对不可以泄漏的，否则会被攻击者还原密文，窃取机密数据。
 - AES加密函数：设AES加密函数为E，则 C = E(K, P),其中P为明文，K为密钥，C为密文。也就是说，把明文P和密钥K作为加密函数的参数输入，则加密函数E会输出密文C。
 - 密文C:经加密函数处理后的数据。
 - AES解密函数:设AES解密函数为D，则 P = D(K, C),其中C为密文，K为密钥，P为明文。也就是说，把密文C和密钥K作为解密函数的参数输入，则解密函数会输出明文P。

MD5的实现：
-------------

MD5可以认为是基于block的算法：它要求每次处理的数据长度为512bits。但是实际中要处理的明文长度并不一定是512的整数倍，需要做数据补齐/填充（Padding）。假设原始明文消息的长度为K，MD5的Padding可以细分为2个子步骤：

 - 1.附加填充位（Append Padding Bits）：从原始明文消息的K位之后补100...一直到512-64=448位，填充位的规则是：只有第一个bit是1，之后都是0。
 - 2.附加长度（Append Length）：在第一步结果之后再填充上原消息的长度，可用来进行的存储长度为64位。整个Padding的示意图如下所示：

.. figure:: ../../picture/SecEngMd5Padding.svg
   :align: center

   MD5 Padding

准备需要用到的数据：

 - 4个常数： A = 0x67452301, B = 0x0EFCDAB89, C = 0x98BADCFE, D = 0x10325476;
 - 4个函数：F(X,Y,Z)=(X & Y) | ((~X) & Z); G(X,Y,Z)=(X & Z) | (Y & (~Z));  H(X,Y,Z)=X ^ Y ^ Z; I(X,Y,Z)=Y ^ (X | (~Z));

把消息分以512位为一分组进行处理，每一个分组进行4轮变换，以上面所说4个常数为起始变量进行计算，重新输出4个变量，以这4个变量再进行下一分组的运算，如果已经是最后一个分组，则这4个变量为最后的结果，即MD5值。

SHA256的实现：
--------------------------

和MD5有些类似，数据填充遵循的准则和MD5一样。SHA256算法中用到了8个哈希初值以及64个哈希常量。

其中，SHA256算法的8个哈希初值如下：

 - h0 := 0x6a09e667
 - h1 := 0xbb67ae85
 - h2 := 0x3c6ef372
 - h3 := 0xa54ff53a
 - h4 := 0x510e527f
 - h5 := 0x9b05688c
 - h6 := 0x1f83d9ab
 - h7 := 0x5be0cd19

这些初值是对自然数中前8个质数（2,3,5,7,11,13,17,19）的平方根的小数部分取前32bit而来。

在SHA256算法中，用到的64个常量如下：

- 428a2f98 71374491 b5c0fbcf e9b5dba5
- 3956c25b 59f111f1 923f82a4 ab1c5ed5
- d807aa98 12835b01 243185be 550c7dc3
- 72be5d74 80deb1fe 9bdc06a7 c19bf174
- e49b69c1 efbe4786 0fc19dc6 240ca1cc
- 2de92c6f 4a7484aa 5cb0a9dc 76f988da
- 983e5152 a831c66d b00327c8 bf597fc7
- c6e00bf3 d5a79147 06ca6351 14292967
- 27b70a85 2e1b2138 4d2c6dfc 53380d13
- 650a7354 766a0abb 81c2c92e 92722c85
- a2bfe8a1 a81a664b c24b8b70 c76c51a3
- d192e819 d6990624 f40e3585 106aa070
- 19a4c116 1e376c08 2748774c 34b0bcb5
- 391c0cb3 4ed8aa4a 5b9cca4f 682e6ff3
- 748f82ee 78a5636f 84c87814 8cc70208
- 90befffa a4506ceb bef9a3f7 c67178f2

和8个哈希初值类似，这些常量是对自然数中前64个质数 (2,3,5,7,11,13,17,19,23,29,31,37,41,43,47,53,59,61,67,71,73,79,83, 89,97…)的立方根的小数部分取前32bit而来。

SHA256散列函数:

 - Ch(x,y,z)=(x∧y)⊕(¬x∧z)
 - Ma(x,y,z)=(x∧y)⊕(x∧z)⊕(y∧z)
 - Σ0(x)=S \ :sup:`2` (x)⊕S \ :sup:`13` (x)⊕S \ :sup:`22` (x)
 - Σ1(x)=S \ :sup:`6` (x)⊕S \ :sup:`11` (x)⊕S \ :sup:`25` (x)
 - σ0(x)=S \ :sup:`7` (x)⊕S \ :sup:`18` (x)⊕R \ :sup:`3` (x)
 - σ1(x)=S \ :sup:`17` (x)⊕S \ :sup:`19` (x)⊕R \ :sup:`10` (x)

其中：

 - \∧    按位“与”
 - ¬     按位“补”
 - \⊕    按位“异或”
 - S \ :sup:`n` 循环右移n个bit
 - R \ :sup:`n` 右移n个bit

GMAC的原理
-------------
消息认证实际上是对消息本身产生的一个冗余的信息，即消息验证码（MAC）。消息认证码（Message authentication code）是一种确认完整性并进行认证的一种技术，简称MAC。密码学中，消息认证码指的是通信实体双方使用的一种验证机制，保证消息数据完整性的一种工具。
消息认证码是一种带密钥的哈希函数，它本质上是一个哈希函数，那为什么要带密钥呢？是因为消息在传输过程中是可以被篡改，哈希值也可以被篡改，因此为了保证这个哈希值的有效性，通过加密的方式将哈希值保护起来，这样在接收方接收到消息后就可以通过这个哈希值来判断整条消息的完整性，从而达到信息传递的目的。

消息认证码步骤如下图所示：

.. figure:: ../../picture/SecEngMac.svg
   :align: center

   消息认证码流程图

1）发送者与接收者事先共享密钥K（上图中的KEY1与KEY2值保持一致）。

2）发送者根据消息计算MAC值（使用密钥KEY1对原始消息计算MAC1）。

3）发送者将原始消息和MAC1发送给接收者。

4）接收者根据收到的原始消息计算MAC2（使用密钥KEY2）。

5）接收者将自己计算出的MAC2与从发送者收到的MAC1比对。

6）如果MAC一致，接收者可以判定消息的确来自接收者（认证成功）且没有被篡改或者出现传输出错的情况；如果不一致，可判断消息不是来自发送方（认证失败）。

注意：建议发送方和接收方将密钥KEY存放于硬件安全模块中，计算MAC值的过程最好也放到硬件安全模块中完成，这样可以保证密钥的安全，例如放到加密芯片中。

GMAC就是利用伽罗华域(Galois Field，GF，有限域)乘法运算来计算消息的MAC值。

功能描述
===========

AES 加速器
-------------
1 AES加速器支持 AES-128/192/256 加解密6种运算。

 - 配置寄存器se_aes_0_ctrl中se_aes_0_mode和se_aes_0_dec_en，如下图所示：

.. figure:: ../../picture/SecEngAesMode.svg
   :align: center

   AES运算模式图

配置寄存器se_aes_0_ctrl中se_aes_0_block_mode选择不同的加密方式，目前支持ECB、CTR、CBC、XTS模式。

2 密钥、明文、密文、初始化向量

 - 寄存器se_aes_0_msa存放明文或者密文的地址。
 - 寄存器se_aes_0_mda存放密文或者明文的地址。
 - 寄存器se_aes_0_iv_0~se_aes_0_iv_3存放IV。
 - 寄存器se_aes_0_key_0~se_aes_0_key_7存放密钥。

3 软硬件加密流程

 - 配置寄存器se_aes_0_endian，其中包括se_aes_0_dout_endian、se_aes_0_din_endian、se_aes_0_key_endian、se_aes_0_iv_endian、se_aes_0_twk_endian

   备注：0:little-endian 1:big-endian

 - 配置寄存器se_aes_0_ctrl中se_aes_0_block_mode
 - 配置寄存器se_aes_0_ctrl中se_aes_0_mode
 - 配置寄存器se_aes_0_ctrl中se_aes_0_dec_en
 - 配置寄存器se_aes_0_ctrl中se_aes_0_dec_key_sel 0:new key 1:same key as last one
 - 配置寄存器se_aes_0_ctrl中se_aes_0_iv_sel 0:new iv 1:same iv as last one
 - 配置寄存器se_aes_0_ctrl中se_aes_0_en enable aes
 - 配置寄存器se_aes_0_iv_0~se_aes_0_iv_3 set IV。

   填写顺序备注：MSB：se_aes_0_iv_0~se_aes_0_iv_3；LSB：se_aes_0_iv_3~se_aes_0_iv_0
 - 配置寄存器se_aes_0_key_0~se_aes_0_key_7 set key。

   填写顺序备注：MSB：se_aes_0_key_0~se_aes_0_key_7；LSB：se_aes_0_key_7~se_aes_0_key_0。

   填写个数备注：AES-128取前4个，AES-196取前6个，AES-256取8个。

 - 配置寄存器se_aes_0_msa set msa addr
 - 配置寄存器se_aes_0_mda set mda addr
 - 配置寄存器se_aes_0_ctrl中se_aes_0_msg_len set msg len
 - 配置寄存器se_aes_0_ctrl中se_aes_0_trig_1t Trigger AES Engine
 - 结果输出存储在寄存器se_aes_0_mda对应的地址中

SHA 加速器
-------------
1 SHA 加速器支持7种标准运算：

 - SHA-1、SHA-224、SHA-256、SHA-512、SHA-384、SHA-512/224、SHA-512/256，同时还支持MD5、CRC16、CRC32。

寄存器se_sha_0_ctrl中se_sha_0_mode:  0:SHA-256 1:SHA-224 2:SHA-1 3:SHA-1 4:SHA-512 5:SHA-384 6:SHA-512/224 7:SHA-512/256

寄存器se_sha_0_ctrl中se_sha_0_mode_ext: hash mode extention; 0:SHA 1:MD5 2:CRC-16 3:CRC-32

配置寄存器se_sha_0_ctrl中se_sha_0_mode选择不同的SHA运算,配置寄存器se_sha_0_ctrl中se_sha_0_mode_ext可选择MD5、CRC16、CRC32。

 - 当se_sha_0_mode_ext为0时，se_sha_0_mode有效。
 - 当se_sha_0_mode_ext不为0时，se_sha_0_mode无效。

2 明文、密文

 - 寄存器se_sha_0_msa存放明文地址。
 - 寄存器se_sha_0_hash_l_0~se_sha_0_hash_l_7存放密文。

 获取顺序备注：
 MSB:se_sha_0_hash_l_0~se_sha_0_hash_l_7；
 LSB:se_sha_0_hash_l_7~se_sha_0_hash_l_0

3 运算流程

 - 配置寄存器se_sha_0_ctrl中se_sha_0_mode set SHA operation type
 - 配置寄存器se_sha_0_ctrl中se_sha_0_en enable sha
 - 配置寄存器se_sha_0_ctrl中se_sha_0_hash_sel 0:new hash 1:accumulate last hash
 - 配置寄存器se_sha_0_msa set mda addr
 - 配置寄存器se_sha_0_ctrl中se_sha_0_msg_len set msg len
 - 配置寄存器se_sha_0_ctrl中se_sha_0_trig_1t Trigger SHA Engine
 - 结果输出存储在se_sha_0_hash_l_0~se_sha_0_hash_l_7 MSB:se_sha_0_hash_l_0~se_sha_0_hash_l_7 LSB:se_sha_0_hash_l_7~se_sha_0_hash_l_0

随机数发生器
-------------
1.内置一个真随机数发生器，其生成的随机数可作为加密等操作的基础。

 - 真随机数：真正的随机数是使用物理现象产生的：比如掷钱币、骰子、转轮、使用电子元件的噪音、核裂变等等，这样的随机数发生器叫做物理性随机数发生器，它们的缺点是技术要求比较高。
 - 伪随机数：真正意义上的随机数（或者随机事件）在某次产生过程中是按照实验过程中表现的分布概率随机产生的，其结果是不可预测的，是不可见的。而计算机中的随机函数是按照一定算法模拟产生的，其结果是确定的，是可见的。我们可以这样认为这个可预见的结果其出现的概率是100%。所以用计算机随机函数所产生的“随机数”并不随机，是伪随机数。

2.输出

 - 寄存器SE_TRNG_0_DOUT_0~SE_TRNG_0_DOUT_7存放输出的随机数。

3.使用流程

 - 配置寄存器se_trng_0_ctrl_0中se_trng_0_en enable trng
 - 配置寄存器se_trng_0_ctrl_0中se_trng_0_trig_1t trigger trng engine
 - 结果输出存储在se_trng_0_dout_0~se_trng_0_dout_7

GMAC(link模式)
--------------------------
1.GMAC_link_Table结构体定义

 - Word0:

     + [9]:se_gmac_0_int_clr_1t
     + [10]:se_gmac_0_int_set_1t
     + [31:16]:se_gmac_0_msg_len

 - Word1:se_gmac_0_msa
 - Word2、Word3、Word4、Word5：se_gmac_0_h
 - Word6、Word7、Word8、Word9：se_gmac_0_tag

2.使用流程

 - 配置寄存器se_gmac_0_ctrl_0中se_gmac_0_x_endian、se_gmac_0_h_endian、se_gmac_0_t_endian 0:little-endian 1:big-endian
 - 将GMAC_link_Table结构体的起始地址写到寄存器se_gmac_0_lca中
 - 将原始消息地址赋给GMAC_link_Table结构体中Word1中se_gmac_0_msa
 - 将原始消息长度赋给GMAC_link_Table结构体中Word0中se_gmac_0_msg_len，128bit消息长度对应se_gmac_0_msg_len中的1
 - 配置寄存器se_gmac_0_ctrl_0中se_gmac_0_trig_1t trigger gmac engine
 - 结果输出存储在GMAC_link_Table结构体中Word6~Word9

.. only:: html

   .. include:: sec_register.rst

.. raw:: latex

   \input{../../zh_CN/content/sec}