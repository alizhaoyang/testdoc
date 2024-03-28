BLOOM 7B 部署手册
===============

## BLOOM 模型简介
BLOOM (BigScience Large Open-science Open-access Multilingual Language Model) 是在46种自然语言和13种编程语言上训练的1760亿参数语言模型，论文地址：[https://arxiv.org/pdf/2211.05100.pdf](https://arxiv.org/pdf/2211.05100.pdf)。

本文档介绍如何在 MOFFETT AI加速卡上利用 SOLA 部署 BLOOM 7B 模型。

### 模型架构
![BLOOM ARCH](data/bloom_arch.png)


### 模型配置
- num_layers: 24
- num_heads: 32
- hidden_size: 4096
- vocab_size: 250680
- batch_size: 1
- token_num: 1
- max_seq_len: 256

### 模型输入

- word embedding
- alibi position embedding
- gather_H_index
- scatter_Gcb_index
- scatter_W_index

### 系统要求

- 至少 12 个 MOFFETT Antoum 芯片
- 支持 avx512f 的 CPU

## BLOOM 部署

部署模型分为四个步骤：下载、编译、运行、验证，每个步骤都提供了对应的默认脚本。

下载：`prepare.sh`

编译：`build.sh`

运行：`run.sh`

验证：`verify.sh`

你也可以参考以下步骤手动部署模型。

### 部署步骤

1. 下载模型依赖

    可以执行以下脚本下载模型依赖：
    ```bash
    ./prepare.sh
    ```
    若脚本执行失败，也可以通过以下链接手动下载并解压：
    ```text
    https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-demo/bloom7b/bloom7b_demo.tar.gz    ```
    ```
   
2. 编译部署代码

    ```bash
    ./build.sh
    ```
   
3. 运行
    
    可以执行以下脚本验证精度和性能：
    ```bash
    ./run.sh
    ```
    也可以手动运行，程序接受两个参数，第一个参数为模型路径，第二个参数为模式，模式是可选的：
    ```bash
    # 问答模式
    ./build/bloom data/model/bloom7b_demo/
    # 无限问答模式
    ./build/bloom data/model/bloom7b_demo/ inf_mode
    # 自动问答模式
    ./build/bloom data/model/bloom7b_demo/ data/questions.txt
    # 验证模式
    ./build/bloom data/model/bloom7b_demo/ verify
    ```

4. 验证模式下验证结果

    ```bash
    ./verify.sh
    ```

### 性能指标参考

测试环境：
- 2x Intel(R) Xeon(R) Platinum 8380 CPU @ 2.30GHz
- 16x 64GiB DDR4 3200 MHz
- 12x MOFFETT Antoum 芯片

|   throughput   |     latency     |
|:--------------:|:---------------:|
| 52.300 token/s | 19.120 ms/token |
