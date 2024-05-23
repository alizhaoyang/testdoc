# ResNet50 模型部署手册

## **概述**

ResNet50 是一种非常强大的深度学习模型，主要用于图像处理领域。其通过独特的残差学习和深度结构，有效地提高了模型的准确率和性能。本文我们会演示如何使用 SOLA Runtime API 来开发一个简单的图片分类应用。

## 模型基本信息

- 输入: RGB 图片, 分辨率处理为 224(h)x224(w)
- 输出: 图片分类标签
- batch size: 4
- data type: int8

本示例参照 MLPerf Offline 模式运行，一次性加载所有图片，再进行分批推理。

MLPref Offline ResNet50 输入输出数据如下:

- 总图片数量: 50000
- 输入 query: [ 50000, 224, 224, 3 ] dtype=uint8, 以二进制格式保存
- 输出 query: [ 50000, 1024 ] dtype=int64, 以二进制格式保存

## 前提条件

请参见《SOLA Runtime 示例程序》完成基础环境配置。

## 使用**流程**

部署模型分为以下四个步骤：

> **说明**： 我们为以下每个步骤都提供了对应的脚本，您可以直接使用。

1. 下载模型和数据集：`prepare.sh`。
2. 编译模型：`build.sh`。
3. 运行模型：`run.sh`。
4. 验证运行结果：`verify.sh`。

你也可以参考以下步骤手动部署模型。

## **使用示例**

1. 执行以下命令，进入模型文件夹后下载模型和数据集，并自动对下载好的模型和数据集进行预处理。

   ```Bash
   $ cd resnet50
   $ ./prepare.sh
   ```

    如果脚本执行失败，您也可以通过以下链接手动下载模型和数据集，并解压：

    ```Bash
    $ wget https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-demo/resnet50/resnet50_demo.tar.gz
    $ tar -zxvf resnet50_demo.tar.gz   
    ```

    并手动执行预处理命令。

    ```Bash
    $ python -s scripts/gen.py --image_dir data/resnet50_demo/ILSVRC2012_img_val --val_map_path data/resnet50_demo/ILSVRC2012_validation_ground_truth_caffe.txt --cache_dir data/resnet50_demo/ --duplicate 1
    ```

2. 编译部署代码。

   ```Bash
   $./build.sh
   ```

3. 运行模型。

   ```Bash
   # 默认只执行device 0
   $./run.sh
   ```

   或者按照以下指令运行：

   ```Bash
   usage: ./build/resnet50 --module=string --inputs=string --labels=string [options] ... 
   options:
     -m, --module         module file path (string)
     -i, --inputs         input file path (string)
     -o, --labels         label file path (string)
     -d, --devices        select devices (string [=all])
     -f, --format         running mode: broadcast / split (string [=broadcast])
     -c, --count          count of duplication (int [=1])
     -b, --batch          run batch size (int [=4])
     -v, --verify_path    save verification result json path (string [=])
     -?, --help           print this message
   ```

    其中`-d`可以指定运行设备（使用`,`分割），`-f`指定运行模式（broadcast/split），`-c`指定运行次数，`-b` 指定需要运行的 batch size（4 的倍数），如：

    ```Bash
    # 在 device 0 上按照 batch size 4 进行推理
    ./build/resnet50 -m data/resnet50_demo/MLPerf_Compiler_RN50_V1.0.1.5.bin -i data/resnet50_demo/input_224_uint8_NHWC -o data/resnet50_demo/labels_224_uint8_NHWC -d 0 -f broadcast -c 1 -b 4 -v data/verification.json
    # 在 device 0 上按照 batch size 32 进行推理
    ./build/resnet50 -m data/resnet50_demo/MLPerf_Compiler_RN50_V1.0.1.5.bin -i data/resnet50_demo/input_224_uint8_NHWC -o data/resnet50_demo/labels_224_uint8_NHWC -d 0 -f broadcast -c 1 -b 32 -v data/verification.json
    # 在 device 0,1,2 上按照 batch size 4 进行推理
    ./build/resnet50 -m data/resnet50_demo/MLPerf_Compiler_RN50_V1.0.1.5.bin -i data/resnet50_demo/input_224_uint8_NHWC -o data/resnet50_demo/labels_224_uint8_NHWC -d 0,1,2 -f broadcast -c 1 -b 4 -v data/verification.json
    ```

4. 验证模型运行结果。

   ```Bash
   $./verify.sh
   ```

## 测试结果参考

以下为单 device 的测试结果参考：

| **model** | **data type** | **batch size** | **accuracy** | **performance** |
| --------- | ------------- | -------------- | ------------ | --------------- |
| ResNet50  | int8          | 4              | 76.43%       | 26501 FPS       |
| ResNet50  | int8          | 16             | 76.43%       | 25983 FPS       |
| ResNet50  | int8          | 32             | 76.43%       | 26115 FPS       |
| ResNet50  | int8          | 64             | 76.43%       | 26132 FPS       |
| ResNet50  | int8          | 128            | 76.43%       | 24883 FPS       |
| ResNet50  | int8          | 256            | 76.43%       | 24138 FPS       |
| ResNet50  | int8          | 256            | 76.43%       | 24138 FPS       |
| ResNet50  | int8          | 256            | 76.4280%     | 24138 FPS       |

## 更多信息

更多关于 Resnet50 模型的介绍，请参见 [Deep Residual Learning for Image Recognition](https://arxiv.org/abs/1512.03385) 。
