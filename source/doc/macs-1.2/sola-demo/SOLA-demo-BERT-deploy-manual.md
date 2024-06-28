# BERT 模型部署手册

## **概述**

BERT（Bidirectional Encoder Representations from Transformers）模型是一种强大的自然语言处理模型，它在预训练阶段采用了双向 Transformer 结构，使得模型可以同时考虑一个词的前后文信息，从而更准确地理解文本的含义。

## **模型**结构

| **Model**  | **Hidden layers** | **Hidden unit size** | **Attention heads** | **Feed-forward** **filter** **size** | **Max sequence length** | **Parameters** |
| ---------- | ----------------- | -------------------- | ------------------- | ------------------------------------ | ----------------------- | -------------- |
| BERT-Base  | 12 encoder        | 768                  | 12                  | 4 x 768                              | 512                     | 110M           |
| BERT-Large | 24 encoder        | 1024                 | 16                  | 4 x 1024                             | 512                     | 330M           |

## 模型信息

本程序使用的 BERT 模型信息：

- model: BERT-Large & BERT-Base
- batch size: 32
- data type: MixInt8Bf16

## 前提条件

请参见《SOLA Runtime 示例程序》完成基础环境配置。

## 使用**流程**

部署模型分为以下四个步骤：

> **说明**： 我们为以下每个步骤都提供了对应的脚本，您可以直接使用。

1. 下载模型和数据集：`prepare.sh`。
2. 编译模型：`build.sh`。
3. 运行模型：`run.sh`。
4. 验证运行结果：`verify.sh`。

## **使用示例**

1. 下载模型和数据集，下载完成后后会自动进行预处理。

   ```Bash
   $ cd bert
   $ ./prepare.sh
   ```

    如果脚本执行失败，您可以通过以下链接手动下载并解压：

    ```Bash
    $ wget https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-demo/bert/bert_demo.tar.gz
    $ tar -zxvf bert_demo.tar.gz 
    ```

    并手动执行预处理命令（需要 Python 环境，激活方式见主目录 README.md）

    ```Bash
    $ mkdir -p data/tmp/
    $ python -s scripts/pre_process.py --input_path=data/bert_demo/ --output_path=data/tmp/
    ```

2. 编译部署代码。

   ```Bash
   $ ./build.sh
   ```

3. 运行模型。

   ```Bash
   # 默认运行 bert base
   $ ./run.sh
   # 指定运行 bert base
   $ ./run.sh bert_base
   # 指定运行 bert large
   $ ./run.sh bert_large
   ```

    或者按照以下指令运行：

    ```Bash
    usage: ./build/bert --module=string --inputs=string --outputs=string [options] ... 
    options:
      -m, --module         module file path (string)
      -i, --inputs         input dir path, with input_x inside (string)
      -o, --outputs        result saving dir (string)
      -d, --devices        select devices (string [=all])
      -f, --format         running mode: broadcast / split (string [=broadcast])
      -c, --count          count of duplication (int [=1])
      -b, --batch          run batch size (int [=32])
      -v, --verify_path    save verification result json path (string [=])
      -?, --help           print this message
    ```

    其中`-d`可以指定运行设备（使用`,`分割），`-f`指定运行模式（broadcast/split），`-c`指定运行次数，`-b`指定需要运行的 batch size（32 的倍数），如：

    ```Bash
    # 在 device 0 上按照 batch size 32 进行 bert base 模型推理
    $ ./build/bert -m data/bert_demo/bert_base.bin -i data/tmp/ -o data/tmp/ -d 0 -f broadcast -c 1 -b 32 -v data/verification.json
    # 在 device 0 上按照 batch size 64 进行 bert large 模型推理
    $ ./build/bert -m data/bert_demo/bert_large.bin -i data/tmp/ -o data/tmp/ -d 0 -f broadcast -c 1 -b 64 -v data/verification.json
    # 在 device 0,1,2 上按照 batch size 32 进行 bert large 模型推理
    $ ./build/bert -m data/bert_demo/bert_large.bin -i data/tmp/ -o data/tmp/ -d 0,1,2 -f broadcast -c 1 -b 32 -v data/verification.json
    ```

    推理输出以二进制文件形式保存到`-o`指定的路径下，根据设备 id，保存文件名为`output_<id>`，然后参考 `run.sh` 中的命令执行精度验证的脚本。

4. 验证模型运行结果。

   ```Bash
    $ ./verify.sh

## **测试结果参考**

| **model**  | **data type** | **batch size** | **accuracy**                                               | **performance** |
| ---------- | ------------- | -------------- | ---------------------------------------------------------- | --------------- |
| ERT-Base   | MixInt8Bf16   | 32             | {"exact_match": 83.66130558183538, "f1": 90.8575190748761} | 2101 FPS        |
| BERT-Base  | MixInt8Bf16   | 64             | {"exact_match": 83.66130558183538, "f1": 90.8575190748761} | 2101 FPS        |
| BERT-Base  | MixInt8Bf16   | 128            | {"exact_match": 83.66130558183538, "f1": 90.8575190748761} | 2107 FPS        |
| BERT-Base  | MixInt8Bf16   | 256            | {"exact_match": 83.66130558183538, "f1": 90.8575190748761} | 2107 FPS        |
| BERT-Large | MixInt8Bf16   | 32             | {"exact_match": 83.74645222327341, "f1": 90.9330076613153} | 1009 FPS        |
| BERT-Large | MixInt8Bf16   | 64             | {"exact_match": 83.74645222327341, "f1": 90.9330076613153} | 1008 FPS        |
| BERT-Large | MixInt8Bf16   | 128            | {"exact_match": 83.74645222327341, "f1": 90.9330076613153} | 1009 FPS        |
| BERT-Large | MixInt8Bf16   | 256            | {"exact_match": 83.74645222327341, "f1": 90.9330076613153} | 1009 FPS        |

## 更多信息

MOFFETT’s BERT 为基于 [Google’s official implementation](https://github.com/google-research/bert) 的优化版本，更多信息，请参见

[BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding](https://arxiv.org/abs/1810.04805)。
