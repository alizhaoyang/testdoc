# BERT 部署手册

## 概述

BERT Bidirectional Encoder Representations from Transformers, 参考

 [BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding](https://arxiv.org/abs/1810.04805)

paper. MOFFETT's BERT 为基于 [Google's official implementation](https://github.com/google-research/bert) 的优化版本。

## 模型结构

| **Model**  | **Hidden layers** | **Hidden unit size** | **Attention heads** | **Feed-forward filter size** | **Max sequence length** | **Parameters** |
| ---------- | ----------------- | -------------------- | ------------------- | ---------------------------- | ----------------------- | -------------- |
| BERT-Base  | 12 encoder        | 768                  | 12                  | 4 x 768                      | 512                     | 110M           |
| BERT-Large | 24 encoder        | 1024                 | 16                  | 4 x 1024                     | 512                     | 330M           |

## 环境依赖

以下为运行 BERT 所需要的环境依赖：

- 编译器: g++ >= 7.5
- Python 环境: 请参照 requirements.txt

## 流程介绍

部署模型分为四个步骤：下载、编译、运行、验证，每个步骤都提供了对应的默认脚本。

下载：`prepare.sh`

编译：`build.sh`

运行：`run.sh`

验证：`verify.sh`

你也可以参考以下步骤手动部署模型。

### 部署步骤

1. 下载模型和数据集

   可以执行以下脚本下载模型依赖，下载后会自动进行预处理：
    ```bash
    ./prepare.sh
    ```
   若脚本执行失败，也可以通过以下链接手动下载并解压：
    ```text
    https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-demo/bert/bert_demo.tar.gz    ```
    ```
   并手动执行预处理命令（需要 python 环境，激活方式见主目录 README.md）
    ```bash
    mkdir -p data/tmp/
    python -s scripts/pre_process.py --input_path=data/bert_demo/ --output_path=data/tmp/
    ```

2. 编译部署代码

    ```bash
    ./build.sh
    ```

3. 运行

   可以执行以下脚本运行：
    ```bash
    ./run.sh
    ```
   其中 -d 可以指定运行设备 -f 指定运行模式（broadcast/split） -c 指定运行次数

   推理输出为 truncated output，以二进制文件形式保存到 -o 路径下，根据设备id，保存结果为 output_id

   后处理部分，后处理完的数据会保存为 mlperf loadgen 可加载的形式，保存到 mlperf_log_accuracy.json

   最后执行 mlperf 精度验证脚本，可以看到精度打印，也可以在 predictions.json 中看到具体预测结果

4. 验证结果

    ```bash
    ./verify.sh
    ```
## 精度

```bash
Evaluating predictions...
{"exact_match": 83.66130558183538, "f1": 90.8575190748761}
```

