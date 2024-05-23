# ChatGLMv2 6B 部署手册

本文档介绍了如何在墨芯 AI 加速卡上使用 SOLA 部署 ChatGLMv2 6B 模型。

## **模型简介**

ChatGLMv2 6B 是一个性能强大、易于部署的开源中英双语对话大模型，可用于各种自然语言处理任务。在初代模型的基础上进行了全面升级，在 MMLU、CEval、GSM8K、BBH 等多个数据集上取得了显著性能提升，具有对话流畅、部署门槛较低等优点。

## **模型配置**

- num_layers: 28
- num_heads: 32
- hidden_size: 13696
- vocab_size: 65024
- batch_size: 32
- token_num: 1
- max_seq_len: 512
- data_type: Bf16

## **系统要求**

- 至少需要 4 个设备
- 支持 AVX512f 或 SSE4 的 CPU

## 前提条件

请参见《SOLA Runtime 示例程序》完成基础环境配置。

## 使用**流程**

部署模型分为以下四个步骤：

> **说明**： 我们为以下每个步骤都提供了对应的脚本，您可以直接使用。

1. 下载模型和数据集：`prepare.sh`。
2. 编译模型：`build.sh`。
3. 运行模型：`run.sh`。
4. 验证运行结果：`verify.sh`。

您也可以参考以下步骤手动部署模型。

**部署步骤**

1. 下载模型依赖。您可以执行以下脚本下载模型依赖：

   ```Bash
      $ cd chatglmv2-6b
      $ ./prepare.sh
   ```

    若脚本执行失败，也可以通过以下链接手动下载并解压：

   ```Plain
      https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-demo/chatglm2/chatglmv2_serving_splitkv.tar.gzhttps://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-demo/tokenizer/chatglm2-6b.tar.gz
   ```

    将`chatglmv2_serving_splitkv.tar.gz`放到`data/model`目录下，`chatglm2-6b.tar.gz`放到 `data/tokenizer`。

2.  编译部署代码。

    ```Bash
    $ ./build.sh
    ```

3. 运行模型。 您可以执行以下脚本验证精度和性能：

    ```Bash
    $ ./run.sh
    ```

   也可以手动运行，模式是可选的：

   ```Bash
   # 问答模式
   $ export PYTHONPATH="$PYTHONPATH:$PWD/test"python3 test/chat_chatglmv26b.py --mode="qa"
   # 自动问答模式，问题可以手动指定，默认使用`data/questions.txt`
   $ python3 test/chat_chatglmv26b.py --mode="auto-qa" --questions="data/questions.txt"
   ```

## **性能指标参考**

测试环境：

- 2x Intel(R) Xeon(R) Platinum 8380 CPU @ 2.30GHz
- 16x 64GiB DDR4 3200 MHz

| **throughput** | **latency**     |
| -------------- | --------------- |
| 5.306 token/s  | 188.45 ms/token |