#  &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;希姆计算TensorTurbo使用说明

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; V1.2.0.RC1

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp;&ensp; &ensp; &ensp; &ensp; &ensp; Stream Computing Inc. 

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp; 2022-02-24

 





![img](C:\Users\kai.yuan\Desktop\license.png)







# 版权声明

本文档版权为北京希姆计算科技有限公司所有（Copyright️2019-2022）。未经本公司书面许可，任何单位或个人不得以任何形式或通过任何途径，包括使用影印、录制在内的电子或机械手段对本文档的部分或者全部进行复制或传播。

# 警告和承诺

我们尽最大努力使本文档尽可能的完善和准确，但疏漏和缺陷之处在所难免。北京希姆计算科技有限公司保留随时修改更新文档的权利。

# 反馈信息

您的反馈意见将使我们受益匪浅。如果您对本文档有什么疑问、意见或建议，请与我们联系，感谢您对我们的支持和帮助。

# 一、TensorTurbo介绍

本章主要介绍了TensorTurbo的相关内容，主要内容包括TensorTurbo的主要功能特性、相关基本概念以及使用TensorTurbo的优势。

## 1.1 什么是TensorTurbo？

TensorTurbo是希姆计算提供的基于TVM二次开发的AI编译器，可以用于支持深度学习模型的推理和训练。开发和部署一个深度学习模型主要有以下三个步骤：

1. 模型训练。
2. 部署模型优化。
3. 模型部署。

TensorTurbo目前可以应用到部署模型优化和模型部署两个步骤中。TensorTurbo能够独立支持Tensorflow/Pytorch等格式的模型编译及运行，将来也可以集成到Tensorflow/Pytorch的后端，目前使能主流AI框架支持STCP920 AI芯片的推理。

## 1.2 基本概念

| **基本概念**                 | **说明**                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| NPU                          | 神经网络处理单元（Neural-network Processing Unit）。         |
| STCP920                      | 北京希姆计算科技有限公司基于NerualScale架构研发的产品，是一款AI推理卡，搭配一颗STCP920处理器，用于快速扩展在云端的推理能力，在图像视频分析、人脸识别、自然语言处理等场景中获得更好的推理性能。 |
| NPC                          | 神经网络处理核（Neural-network Processing Core），STCP920包含32个NPC。 |
| 本地内存 （L1）              | 每个NPC私有的高速内存，STCP920中的大小为1.25MB 。            |
| 共享内存 （LLB）             | 每个NPC Cluster私有的中速内存，被NPC Cluster内所有NPC共享，STCP920中的大小为8MB 。 |
| 全局内存 （DDR）             | 每个NPC Cluster私有的低速内存，被NPC Cluster内所有NPC共享，STCP920中 的大小为4GB 。 |
| 模型（Model）                | 泛指神经网络模型，例如ResNet50。                             |
| 算子（Operator）             | 泛指神经网络模型中的算子，是一个数学逻辑单元，算子组成一个有向无环图（Directed acyclic graph，DAG），构成一个模型。 |
| 元算子（Primitive Operator） | TensorTurbo架构中特有的概念，指类似加减乘除等基本的数学计算操作算子。 |
| 图算子（Graph Operator）     | TensorTurbo中特有的概念，是一个由元算子组成的子图，存在于图层抽象。例如softmax可以是一个图算子，定义在图层抽象中，在lower到算子层时，由元算子自动拼接编译得到。 |

## 1.3 使用TensorTurbo的优势

使用TensorTurbo可以：

- 帮助您快速部署您的AI模型到STCP920 AI芯片上。
- TensorTurbo支持自动的图优化和算子优化，最大限度的挖掘模型在STCP920 AI芯片运行的性能。
- TensorTurbo提供手动图调度的能力，满足您对功能和性能的定制化需求。

# 二、安装TensorTurbo

TensorTurbo支持在Spike模拟器、Gem模拟器和STCP920硬件环境下部署。目前仅提供在Linux环境下开发和使用TensorTurbo，推荐您在Debian9、Debian10、Ubuntu18.04、Ubuntu20.04系统环境下使用使用TensorTurbo。

安装TensorTurbo的详细步骤，请参见**希姆计算STCP920快速部署指南**。

# 三、使用TensorTurbo编译模型

使用TensorTurbo进行模型编译整体可拆分为以下步骤：

1. 模型导入。
2. 模型信息获取。
3. 模型编译。

TensorTurbo支持从主流AI框架的不同格式导入AI模型。您可以在AI框架中定义模型，并使用TensorTurbo的前端转换API把不同AI框架定义的模型格式转换为统一的TensorTurbo IR module中间表示。目前TensorTurbo支持的AI框架包括TensorFlow、PyTorch、ONNX、MxNet、Keras等。

## 3.1  Python API

您可以使用TensorTurbo提供的Python API完成模型编译的整体流程。

### 3.1.1 整体流程示例

以ResNet50为例，在安装了TensorTurbo后，您可执行以下代码生成ResNet50在NPU卡上可执行的二进制文件。

```Python
import tensorturbo
import tb
# 为了减少tf warning log的输出
import os
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '3'

if __name__ == '__main__':
    inputs = ["input_tensor"] 
    outputs = ["softmax_tensor_fp16"]
    model_path = './resnet-50_v2.pb'
    out_path = './resnet-50_v2.stcobj'
    # 导入模型
    resnet50_v2 = tensorturbo.model.from_tensorflow(model_path, inputs, outputs)
    # 模型相关信息获得、修改
    print(resnet50_v2.get_input_info())
    print(resnet50_v2 .get_output_info())
    resnet50_v2.set_input_shape({"input_tensor": [16, 224, 224, 3]})
 
    #case1:编译模型（使用指定schedule）
    resnet50_v2.compile(
        output_file=out_path,
        schedule=tb.kernel.schedule.resnet50_schedule,
        required_pass=["FuseConvAddBnRelu"]
    )
    
    #case2:编译模型(schedule=None，则使用自动图调度AutoSchedule)
    resnet50_v2.compile(
        output_file=out_path,
        schedule=None,
        required_pass=[]
    )
```

下文将从导入模型、获取模型信息和编译模型三个方面介绍各个模块API的使用方式。

### 3.1.2 导入模型

TensorTurbo支持从各种AI框架导入模型。目前TensorTurbo支持通过TensorFlow pb文件、ONNX文件、MxNet模型文件、Keras的h5文件和TVM的relay文件进行导入模型。

#### 从TensorFlow pb文件导入模型

```
tensorturbo.model.from_tensorflow(path, inputs, outputs)
```

**参数说明**

| **参数名** | **类型**  | **可选参数** | **描述**                                                     |
| ---------- | --------- | ------------ | ------------------------------------------------------------ |
| path       | str       | 否           | pb模型文件路径。                                             |
| inputs     | list(str) | 是           | 模型中需要导入的子图的输入节点名称列表。如不提供，则默认使用整个模型的输入节点。 |
| outputs    | list(str) | 是           | 模型中需要导入的子图的输出节点名称列表。如不提供，则默认使用整个模型的输出节点。 |

**返回值**

```
tensorturbo.model.Model
```

**调用示例**

```Python
# tensorflow pb模型的输入和输出
inputs = ["input_tensor"] 
outputs = ["softmax_tensor_fp16"]
# pb文件路径
model_path = './resnet-50_v2.pb'

resnet50_v2 = tensorturbo.model.from_tensorflow(model_path, inputs, outputs)
```

#### 从ONNX文件导入模型

```
tensorturbo.model.from_onnx(path, shape)
```

**参数说明**

| **参数名** | **类型**             | **可选参数** | **描述**                                                     |
| ---------- | -------------------- | ------------ | ------------------------------------------------------------ |
| path       | str                  | 否           | ONNX模型文件路径。                                           |
| shape      | dict{str: list(int)} | 是           | 输入tensor的shape信息，如果模型中已有输入shape，那么可以不指定这个参数。 |

**返回值**

```
tensorturbo.model.Model
```

**调用示例**

```Python
model = tensorturbo.model.from_onnx("/path/to/model.onnx")
```

#### 从MXNet格式模型文件导入模型

```
tensorturbo.model.from_mxnet(prefix, epoch, shape=None)
```

**参数说明**

| **参数名** | **类型**             | **可选参数** | **描述**                                                     |
| ---------- | -------------------- | ------------ | ------------------------------------------------------------ |
| prefix     | str                  | 否           | MXNet模型文件前缀。                                          |
| epoch      | int                  | 否           | Epoch number。                                               |
| shape      | dict{str: list(int)} | 是           | 输入tensor的shape信息，如果模型中已有输入shape，那么可以不指定这个参数。 |

**返回值**

```
tensorturbo.model.Model
```

**调用示例**

```Python
model = tensorturbo.model.from_mxnet("model", 10)
```

#### 从Keras模型文件导入模型

```
tensorturbo.model.from_keras(path, shape=None)
```

**参数说明**

| **参数名** | **类型**             | **可选参数** | **描述**                                                     |
| ---------- | -------------------- | ------------ | ------------------------------------------------------------ |
| path       | str                  | 否           | Keras模型文件路径。                                          |
| shape      | dict{str: list(int)} | 是           | 输入tensor的shape信息，如果模型中已有输入shape，那么可以不指定这个参数。 |

**返回值**

```
tensorturbo.model.Model
```

**调用示例**

```Python
 model = tensorturbo.model.from_keras("/path/to/model.h5")
```

#### 从TVM Relay IR的文本格式导入模型

```
tensorturbo.model.from_relay(relay_file, **args)
```

**参数说明**

| **参数名** | **类型** | **可选参数** | **描述**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| path       | str      | 否           | relay文件路径。                                              |
| args       | dict     | 是           | 模型参数，例如{"BATCH": 8}，导入过程中Relay文本文件中出现的所有`BATCH`将会被替换为`8`。 |

**返回值**

```
tensorturbo.model.Model
```

**调用示例**

```Python
 model = tensorturbo.model.from_relay("/path/to/model.relay", BATCH=8)
```

### 3.1.3 获取模型信息

在完成导入模型工作后，您可以通过以下接口来获取模型输入输出的信息，或者更改输入shape，例如更改batch size。

#### 获取模型输入数据的类型信息

```
tensorturbo.model.Model.get_input_info()
```

您可以使用模型导入后返回值来调用接口，上述返回值的类型为`tensorturbo.model.Model`。

**参数说明**

无

**返回值**

```
dict: {str: {"shape" : list(int), "dtype", str}}
```

**调用示例**

```Python
 print(resnet50_v2.get_input_info())
 
 #输出结果
 {"input_tensor": {"shape": [16, 224, 224, 3], "dtype": "float32"}}
```

#### 获取模型输出数据的类型信息

```
tensorturbo.model.Model.get_output_info()
```

**参数说明**

无

**返回值**

```
dict: {str: {"shape" : list(int), "dtype", str}}
```

**调用示例**

```Python
 print(resnet50_v2.get_output_info())
 
 #输出结果
 {'softmax_tensor_fp16': {'shape': [16, 1001], 'dtype': 'float16'}}
```

#### 修改模型输入数据的类型信息

```
tensorturbo.model.Model.set_input_shape(input_shape)
```

编译器根据修改后的类型信息重新推导整个模型的静态类型,用于更改batch size。

**参数说明**

| **参数名**  | **类型**              | **可选参数** | **描述**              |
| ----------- | --------------------- | ------------ | --------------------- |
| input_shape | dict:{str: list(int)} | 否           | 指定每个输入的shape。 |

**返回值**

无

**调用示例**

```Python
resnet50_v2.set_input_shape({"input_tensor": [16, 224, 224, 3]})
```

### 3.1.4 编译模型

通过模型导入成功构建TensorTurbo计算图之后，需要编译计算图，编译的结果是一个TensorTurbo包含runtime module的二进制代码，您可以直接通过模型运行相关的API开始推理。

#### 编译模型

```
tb.model.Model.compile(output_file, schedule, **config)
```

将TensorTurbo模型编译为stcobj二进制文件

**参数说明**

| **参数名**  | **类型** | **可选参数** | **描述**                                       |
| ----------- | -------- | ------------ | ---------------------------------------------- |
| output_file | str      | 否           | 输出stcobj文件路径。                           |
| schedule    | function | 是           | 图层调度函数。如果不指定，则启用自动图层调度。 |
| config      | dict     | 是           | 编译配置参数，详见下表。                       |

#### **编译配置参数**

在编译模型时，您可根据不同模型配置不同的参数，包括dump文件，使能/关闭Relay IR的pass以及pass config。以“tb“开头的参数表示pass config，用于对特定pass的功能进行开关控制。

| **配置参数名**          | **类型**  | **默认值** | **说明**                                                     |
| ----------------------- | --------- | ---------- | ------------------------------------------------------------ |
| tb.dump_file            | str       | None       | 将模型编译生成的C Intrinsics代码保存为.cc文件。              |
| required_pass*          | list(str) | None       | 使能pass列表，常用需要控制开关的pass有："FuseConvAddBnRelu"，在ResNet50模型中需要设置。 |
| disabled_pass*          | list(str) | None       | 关闭pass列表，常用需要控制开关的pass有：“FuseLargeOps”，DSSM模型中需要设置，否则会影响性能。 |
| tb.fuse_transpose       | bool      | False      | 融合transpose算子，需在bert_128模型中设置为True。            |
| tb.fuse_reshape         | bool      | False      | 融合reshape算子，需在bert_128模型中设置为True。              |
| tb.bisection_sum        | bool      | False      | 二分累加，规避大数加小数，提高累加精度，默认关闭，打开后会造成性能下降。 |
| tb.CSE_skip_scalar      | bool      | False      | 用于跳过scalar，默认关闭，您可在Bert、Xlnet中打开该选项。    |
| tb.insn_schedule        | bool      | True       | 指令调度，默认打开。影响Bert large的内存分配，在该网络中设置关闭。 |
| tb.graph_schedule_level | int       | 3          | 图层自动调度优化等级，仅在schedule为None时生效。数值为0-3，各级别优化内容如下： O0：把所有输入输出还有中间计算结果都保存在DDR上。 O1：尽量把所有输入输出还有中间计算结果都保存在LLB上。 O2：尽量把所有输入输出还有中间计算结果都保存在L1上，不能放到L1上的就尽量存放到LLB上。 O3：使用计算顺序调整、切分算子、合并算子等操作，通过启发式方法尽可能得到最优计算顺序以尽量把中间结果放在L1上。 |
| quant_config            | dict      | None       | 量化相关参数，如果不指定则不做量化，更多信息请参见下文量化参数部分介绍。 |

如果需要进一步理解relay pass 里面的相关配置和概念，请参见[tvm](https://tvm.apache.org/docs/arch/pass_infra.html#)的官方文档。

#### **量化参数(quant_config)**

量化参数可选一个可选功能模块，您可按需选择。更多介绍请参见**希姆计算-量化文档**。

| **量化参数**           | **参数列表**                               | **说明**                                                     |
| ---------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| calibration_set        | [{"input_name": input}]                    | 校准数据集，默认为空。 `input_name`为模型输入tensor名称，`input`为校准数据。 |
| cpu_target             | "llvm"                                     | Host侧CPU类型，默认为llvm。                                  |
| "llvm -mcpu=core-avx2" | 如CPU支持，开启avx向量运算。               |                                                              |
| calibrate_mode         | “percentile”                               | input校准方式，默认为“percentile”                            |
| “kl_divergence”        | kl散度校准input方式。                      |                                                              |
| weight_scale           | "power2"                                   | 当前默认为"power2",当前仅支持该方式，暂可不用设置该参数。    |
| preset_scale           | {"layer_name":[input_scale, weight_scale]} | `layer_name`为量化层名称，`input_scale`和`weight_scale`为预设的scale参数。默认为空。 |

当使用Tensorturbo计算scale时，需设置`calibration_set`、`cpu_target`、`calibrate_mode`参数，`weight_scale`暂不需设置。

当使用预设scale时，需设置`preset_scale`参数。

**返回值**

无

**调用示例**

```Python
out_path = './resnet-50_v2.stcobj'
resnet50_v2.compile(
    output_file=out_path,
    """
    目前tb的scheduel里提供resnet50_schedule和bert_schedule
    schedule=tb.kernel.schedule.resnet50_schedule,
    如果不设置，就默认使用自动图调度，此时应required_pass=[]
    """
    schedule= tb.kernel.schedule.resnet50_schedule,
    required_pass=["FuseConvAddBnRelu"],
)
```

## 3.2 模型部署

目前希姆提供两种部署方式：

- C++ API bare mental的部署
- Tensorflow custom op的部署方案

具体说明请参见**STCP920 Runtime API手册**(TODO)。

# 四、模型图调度

通过模型图调度，在整个网络上充分发掘算子间L1/LLB数据重用的机会，对网络中的算子进行拆分、合并和计算顺序重排，从而提高图程序的局部性，使上一个算子的输出数据可以缓存在L1/LLB上，减少网络对LLB带宽和DDR带宽的需求，从而使希姆计算卡在深度模型上发挥出最好的性能。

## 4.1 模型自动图调度

当不传入schedule给compile API时，默认使用自动图调度。

目前希姆已完成支持的模型列表如下，模型均已完成自动图调度的支持（性能与手写图调度几乎一致）。

| **CV类**     | resnet50      | resnet34      | resnet101     | mobilenet  | densenet | resnet50_v1.5 | -          |
| ------------ | ------------- | ------------- | ------------- | ---------- | -------- | ------------- | ---------- |
| **NLP类**    | bert-base128  | bert-base 256 | bert-base 512 | bert-large | XLNet    | albert        | video-bert |
| **推荐场景** | wide and deep | deepfm        | dlrm          | dssm       | mmoe     | dlrm_1M       | -          |

一般来说，在用户自己设计的模型上，往往达不到NPU的最好性能。如果您想进一步优化性能，可使用手动图调度方式。

## 4.2 模型图层手动调度（进阶版）

TensorTurbo图层手工调度可以在整个网络上充分发掘算子间L1/LLB数据重用的机会，通过对网络中的算子进行拆分、合并和计算顺序重排，从而提高图程序的局部性，使上一个算子的输出数据可以缓存在L1/LLB上，减少网络对LLB带宽和DDR带宽的需求。

没有经过调度的原网络结构，计算以算子为单位执行，每个算子的所有batch执行完成后再执行下一个算子，如下图所示。

![img](C:\Users\kai.yuan\Desktop\427d4a64-80a7-4b0a-8a98-07845bb9ed71.png)

此时算子计算的中间结果包含所有batch的数据，大小超出L1/LLB存储限制，缓存在DDR上。

通过对原网络中的算子选择不同的拆分因子进行拆分，每个算子可以在计算完部分数据后就进入下一个算子的计算。拆分和调度后的网络计算顺序如下图所示。

![img](C:\Users\kai.yuan\Desktop\93ee63e8-0e50-4199-8ef5-1086986335e0.png)

TensorTurbo 提供了对图进行手工调度的API，您可手动指定算子拆分和调度方案。

- **NPU kernel索引**

```Python
tb.relay.graph_schedule.GetNPUSubFunctions(function)
```

该API用以从主函数中索引NPU kernel函数。TensorTurbo根据算子的属性把主函数划分为若干NPU kernel子函数，用于表示在NPU上的计算子图，NPU kernel之外的算子则在CPU上执行。

GetNPUSubFunctions函数从主函数索引NPU kernel函数，返回一个list，其中每个元素是一个NPU kernel函数对象，顺序按主函数中DAG后序遍历的顺序进行组织。

- **算子索引**

```Python
tb.relay.graph_schedule.ListOps(function)
```

该API用以从kernel中索引每一个算子，返回一个dict，其中key是算子的名字，value是一个list，顺序存储了输入function中的每个算子。

您可以通过索引这个dict，找到function中的某个算子。例如索引function中的第二个卷积算子。

```Python
op = ListOps(function)['nn.conv2d'][1]
```

**注意**

- 在Relay IR中，只存在DAG表示算子之间的数据依赖关系，并没有实际确定的算子计算顺序**。**ListOps返回的算子list中的算子顺序是DAG上的深度优先后序遍历，而非实际计算顺序。
- IR变换pass会使之前索引到的算子失效，在经过优化等可能会改变Relay IR的操作后，需要重新调用ListOps获取新的算子索引。

**算子分组**

TensorTurbo引入Group的概念，用于帮助您操作计算图的拆分和调度。

Group是一个多输入、单输出的DAG，可以描述整个计算图中的一个子图。您可通过以下方式来创建一个Group。

```Python
tb.relay.graph_schedule.Group(output, list[input])
```

group子图用来描述一组拆分逻辑和调度参数相同的算子，后续的拆分API可以以group为操作对象，从而避免枚举子图中的每个算子分别拆分。

```Python
tb.relay.graph_schedule.GroupOps(list[group])
```

调用GroupOps 对整个module进行指定的group变换，把创建的group子图的相关信息保存在IR上。

- **算子融合**

 有了算子Group的定义之后，您可以对一个group进行算子融合的操作。把指定的算子group融合成不可拆分的function， 减少不必要的算子中间结果数据传输。

```Python
tb.relay.graph_schedule.FuseTargetOps(list[group])
```

示例如下。

```Python
targetops = [
    Group(ops["nn.leaky_relu"][0], [ops["add"][100]]),
    Group(ops["tanh"][0], [ops["nn.leaky_relu"][0]]),
]
mod = FuseTargetOps(targetops)(mod)
```

- **子图拆分**

把子图计算拆分成更小规模的计算，减少算子输入输出数据大小，从而提高子图计算的局部性，使子图计算的中间结果可以在L1重用。

子图拆分必须保证子图的计算可以在L1上完成，即其输入输出和中间结果的大小不超过L1的大小限制。如果子图拆分不满足这个条件，您可通过以下方式尝试修改拆分策略：

- 增大`split_factor`，使每个子图计算的数据更小。
- 重新划分子图为更细粒度的子图，直到满足中间结果在L1上重用的条件。

```Python
tb.relay.graph_schedule.SplitGroups(dict{group : split_factor})
```

SplitGroups API以group子图为操作对象，把整个子图中的输入输出和其中的每个算子按指定factor进行拆分。

- **算子调度**

为了进一步提高整个图程序的局部性，使group子图之间的计算结果最大程度利用L1/LLB片上缓存，您需要为整个网络中每个group和group之外的算子指定以下信息：

- group/算子之间的计算顺序。
- group/算子输出数据的存储位置（L1/LLB/DDR）。
- group/算子的多核计算划分。

算子调度的目标就是为整个网络中的所有group/算子指定以上三个属性，包含这些属性的group/算子集合称为一个调度。

调度信息

```Python
tb.relay.graph_schedule.ScheduleInfo(op_or_group, mem_scope, core_bind)
```

ScheduleInfo数据结构用于保存一个group/算子的调度信息。

- `mem_scope`：输出数据所在的存储位置，可选值为`NPUMemScope.[DDR|LLB|L1]`。

您需要仔细考虑全网中所有group/算子计算结果的存储位置，在利用片上存储缓存中间结果的同时也要考虑到L1/LLB的容量限制。如果`mem_scope`的指定不合理，可能会导致编译器后端存储分配失败。

- `core_bind`：NDArray类型的数据，表示计算在核间数据分布情况 。

Array类型的数据表示计算输出数据在L1上的核间分布。例如[[0, 1], [2, 3], [4, 5], [6, 7]]，表示该op/group按batch拆分为4份，第一个batch输出在0-1核上，第二个batch输出在2-3核上，依次类推。所有核的数据组合起来才构成了完整的输出结果。

 计算核分配时，应尽量减少核间数据传输。假设一个L1->L1的copy算子的输入core_bind=[[0, 1], [2, 3], [4, 5], [6, 7]]，输出core_bind=[0, 1, 2, 3]，表示每两个核的数据汇总到一个核上。如果按照这种核分配策略，[0,1]核的数据需要汇总到0核上，此时0核上的数据不需要传输，1核的数据需要传到0核上；[2, 3]核的数据都需要传输到1核上，总共需要进行7次核间传输。如果更换输出的core_bind=[0, 2, 4, 6]，则只需要进行4次核间传输；再进一步考虑到NOC网络的拓扑结构，可以找到一种对NOC网络带宽需求最小的最优分配方案。

```Python
tb.relay.graph_schedule.Schedule(list[ScheduleInfo])
```

Schedule API根据您指定的list中的group/算子计算顺序，重新组织IR，并把每个group/算子对应的ScheduleInfo保存在IR上。

# 五、支持OP列表

DDR、LLB、L1按照由大到小，由慢到快的顺序排列。

- Elementwise(Unary)

| **OP**     | **memory scope (DDR)** | **memory scope (LLB)** | **memory scope  (L1)** |
| ---------- | ---------------------- | ---------------------- | ---------------------- |
| Abs        | Yes                    | Yes                    | Yes                    |
| Cast       | Yes                    | Yes                    | Yes                    |
| Clip       | Yes                    | Yes                    | Yes                    |
| Cosh       | Yes                    | Yes                    | Yes                    |
| Erf        | Yes                    | Yes                    | Yes                    |
| Exp        | Yes                    | Yes                    | Yes                    |
| LogicalNot | Yes                    | Yes                    | Yes                    |
| Sign       | Yes                    | Yes                    | Yes                    |
| Sinh       | Yes                    | Yes                    | Yes                    |
| Sqrt       | Yes                    | Yes                    | Yes                    |
| Square     | Yes                    | Yes                    | Yes                    |
| Tanh       | Yes                    | Yes                    | Yes                    |

- Elementwise (Binary)

| **OP**       | **memory scope (DDR)** | **memory scope (LLB)** | **memory scope  (L1)** |
| ------------ | ---------------------- | ---------------------- | ---------------------- |
| Add          | Yes                    | Yes                    | Yes                    |
| Bias_add     | Yes                    | Yes                    | Yes                    |
| Divide       | Yes                    | Yes                    | Yes                    |
| Equal        | Yes                    | Yes                    | Yes                    |
| Greater      | Yes                    | Yes                    | Yes                    |
| GreaterEqual | Yes                    | Yes                    | Yes                    |
| Less         | Yes                    | Yes                    | Yes                    |
| LessEqual    | Yes                    | Yes                    | Yes                    |
| LogicalAnd   | Yes                    | Yes                    | Yes                    |
| LogicalOr    | Yes                    | Yes                    | Yes                    |
| LogicalXor   | Yes                    | Yes                    | Yes                    |
| Magnitude    | Yes                    | Yes                    | Yes                    |
| Maximum      | Yes                    | Yes                    | Yes                    |
| Minimum      | Yes                    | Yes                    | Yes                    |
| Multiply     | Yes                    | Yes                    | Yes                    |
| NotEqual     | Yes                    | Yes                    | Yes                    |
| Subtract     | Yes                    | Yes                    | Yes                    |

- Reduce

| **OP** | **memory scope (DDR)** | **memory scope (LLB)** | **memory scope  (L1)** |
| ------ | ---------------------- | ---------------------- | ---------------------- |
| ArgMax | Yes                    | Yes                    | Yes                    |
| ArgMin | Yes                    | Yes                    | Yes                    |
| Max    | Yes                    | Yes                    | Yes                    |
| Min    | Yes                    | Yes                    | Yes                    |
| Mean   | Yes                    | Yes                    | Yes                    |
| Sum    | Yes                    | Yes                    | Yes                    |

- Neural Networks

| **OP**          | **memory scope (DDR)** | **memory scope (LLB)** | **memory scope  (L1)** |
| --------------- | ---------------------- | ---------------------- | ---------------------- |
| Broadcast       | Yes                    | Yes                    | Yes                    |
| Expand_dims     | Yes                    | Yes                    | Yes                    |
| Full            | Yes                    | Yes                    | Yes                    |
| Take            | Yes                    |                        | Yes                    |
| Concat          | Yes                    | Yes                    | Yes                    |
| Resize2D        | Yes                    | Yes                    | Yes                    |
| GatherND        | Yes                    | Yes                    | Yes                    |
| Reshape         | Yes                    | Yes                    | Yes                    |
| Select          | Yes                    | Yes                    | Yes                    |
| Slice           | Yes                    | Yes                    | Yes                    |
| Split           | Yes                    | Yes                    | Yes                    |
| Squeeze         | Yes                    | Yes                    | Yes                    |
| AvgPool         | Yes                    | Yes                    | Yes                    |
| GlobalAvgPol    | Yes                    | Yes                    | Yes                    |
| BatchNorm       | Yes                    | Yes                    | Yes                    |
| LayerNrom       | Yes                    | Yes                    | Yes                    |
| MaxPool         | Yes                    | Yes                    | Yes                    |
| OneHot          | Yes                    | Yes                    | Yes                    |
| Gelu            | Yes                    | Yes                    | Yes                    |
| Relu            | Yes                    | Yes                    | Yes                    |
| LeakyRelu       | Yes                    | Yes                    | Yes                    |
| PRelu           | Yes                    | Yes                    | Yes                    |
| Relu6           | Yes                    | Yes                    | Yes                    |
| Mish            | Yes                    | Yes                    | Yes                    |
| Sigmoid         | Yes                    | Yes                    | Yes                    |
| Swish           | Yes                    | Yes                    | Yes                    |
| Softmax         | Yes                    | Yes                    | Yes                    |
| Roialign        | Yes                    |                        | Yes                    |
| Roipooling      | Yes                    |                        | Yes                    |
| Transpose       | Yes                    | Yes                    | Yes                    |
| Conv3d          | Yes                    | Yes                    | Yes                    |
| DepthwiseConv2d | Yes                    | Yes                    | Yes                    |
| NMS             | Yes                    |                        | Yes                    |
| Conv2d          | Yes                    | Yes                    | Yes                    |
| Matmul          | Yes                    | Yes                    | Yes                    |
| TopK            | Yes                    |                        | Yes                    |