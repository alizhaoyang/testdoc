GPT-J 6B部署手册
===============

## GPT-J 6B模型简介
GPT-J 6B是一个由EleutherAI研究小组创建的开源自回归语言模型。它是OpenAI的GPT-3的最先进替代品之一，在各种自然语言任务（如聊天、摘要和问答等）方面表现良好。

本文档介绍如何在 MOFFETT AI加速卡上利用 SOLA 部署 GPT-J 6B 模型。

### 模型配置
- num_layers: 28
- num_heads: 16
- hidden_size: 4096
- vocab_size: 50401
- batch_size: 8
- token_num: 32
- max_seq_len: 256

### 系统要求

- 至少 1 个 MOFFETT Antoum 芯片
- 支持 avx512f 的 CPU

## 模型部署

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
    https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-demo/gptj_mlperf/gptj_mlperf.tar.gz
    ```

2. 编译部署代码

    ```bash
    ./build.sh
    ```
   
3. 运行
   
    可以执行以下脚本运行一段summarize例子：
    ```bash
    ./run.sh
    ```
    也可以手动运行，指定第一个参数为模型路径：
    ```bash
    ./build/gptj <model_path>
    ```
   
4. 验证结果

    ```bash
    ./verify.sh
    ```

### 性能指标参考

测试环境：
- 2x Intel(R) Xeon(R) Platinum 8380 CPU @ 2.30GHz
- 16x 64GiB DDR4 3200 MHz
- 24x MOFFETT Antoum 芯片

|    throughput     |    latency     |
|:-----------------:|:--------------:|
| 1532.002 tokens/s | 0.653 ms/token |

