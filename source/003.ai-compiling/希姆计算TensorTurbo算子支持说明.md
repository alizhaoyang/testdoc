# 希姆计算TensorTurbo算子支持说明

## 版本历史

| **文档版本** | **对应产品版本**   | **作者** | **日期**   | **描述**                                                     |
| ------------ | ------------------ | -------- | ---------- | ------------------------------------------------------------ |
| V1.12.0      | STCRP V1.4.0       | 希姆计算 | 2023-04-04 | 新增支持ONNX算子：Einsum<br />更新以下ONNX算子限制说明：AveragePool、Conv、MaxPool、Upsample<br />更新TensorTurbo算子层额外的算子depthwise_conv2d的使用限制说明<br />架构调整和编辑优化 |
| V1.11.0      | STCRP V1.3.0       | 希姆计算 | 2023-01-10 | 新增支持ONNX算子：Gemm、GRU、LSTM、IsInf、RNN、Shape更新Resize算子使用限制说明 |
| V1.10.0      | STCRP V1.2.0       | 希姆计算 | 2022-11-24 | 更新AveragePool 、Cast、Conv 、ConvTranspose 、LayerNormalization、Pow算子的使用限制说明。新增支持TensorTurbo算子层额外的算子：any。 |
| V1.9.0       | STCRP V1.1.0       | 希姆计算 | 2022-10-10 | 新增基于ONNX opset=9，统计出TensorTurbo目前暂不支持的ONNX算子列表。 |
| V1.8.0       | STCRP V1.1.0       | 希姆计算 | 2022-09-08 | 新增支持ONNX算子：IsNaN、LayerNormalization、MaxRoiPool、NonZero、 RoiAlign 新增支持TensorTurbo算子层额外的算子：floor_divide |
| V1.7.1       | TensorTurbo V1.7.0 | 希姆计算 | 2022-08-09 | 更新算子使用限制说明根据目前算子支持情况和算子性质进行重新编辑和排序 |
| V1.7.0       | TensorTurbo V1.7.0 | 希姆计算 | 2022-08-01 | 新增支持ONNX算子：Atanh、Asinh、Acosh、HardSigmoid、InstanceNormalization、MeanVarianceNormalization、Reciprocal、ReduceSumSquare、ReduceL1 新增支持TensorTurbo算子层额外的算子：negative |
| V1.6.0       | TensorTurbo V1.6.0 | 希姆计算 | 2022-06-30 | 新增支持ONNX算子：Acos、 Asin、Atan、GlobalLpPool、GlobalMaxPool、Hardmax、MaxUnpool、ReduceLogSum、ReduceProd、Tan |
| V1.5.0       | TensorTurbo V1.5.0 | 希姆计算 | 2022-06-02 | 新增支持ONNX算子：Ceil、Cos、Floor、LpPool、Pad、ReverseSequence、Round、ReduceLogSumExp、ReduceL2、Shrink、Sin |
| V1.4.0       | TensorTurbo V1.4.0 | 希姆计算 | 2022-04-29 | 新增支持ONNX算子：Cast、CumSum 、Elu、Gather、Log、Logsoftmax、Mod、SpaceToDepth、Softplus、Scatter、ScatterND新增支持TensorTurbo算子层额外的算子：argsort、sort |
| V1.3.0       | TensorTurbo V1.3.0 | 希姆计算 | 2022-03-31 | 初始对外版本。                                               |

## ONNX算子（Opset 9）

基于ONNX Opset 9，TensorTurbo支持的ONNX算子共计113个，算子列表如下：

| **算子**                  | **限制说明**                                                 |
| ------------------------- | ------------------------------------------------------------ |
| Abs                       | 无限制。                                                     |
| Acos                      | 输入范围[-1,1]。                                             |
| Acosh                     | 输入数值范围为[1, 16950]。                                   |
| Add                       | 无限制。                                                     |
| And                       | 无限制。                                                     |
| ArgMax                    | 只支持axis为一根轴。                                         |
| ArgMin                    | 只支持axis为一根轴。                                         |
| Asin                      | 输入范围[-1,1]。                                             |
| Asinh                     | 输入数值范围为[-255.9, 255.9]。                              |
| Atan                      | 无限制。                                                     |
| Atanh                     | 输入数值范围为[-1, 1)。                                      |
| AveragePool               | 支持2D与3D。pad参与计算。                                    |
| BatchNormalization        | 无限制。                                                     |
| Cast                      | 无限制。                                                     |
| Ceil                      | 无限制。                                                     |
| Clip                      | 无限制。                                                     |
| Concat                    | 无限制。                                                     |
| Constant                  | 无限制。                                                     |
| ConstantOfShape           | 无限制。                                                     |
| Conv                      | dilations在各维度的取值应相同。kernel大小不超过9*9。         |
| ConvTranspose             | 不支持auto_pad与dilations。kernel大小不超过9*9。             |
| Cos                       | 输入数值范围为[-6.7, 6.7]。                                  |
| Cosh                      | 无限制。                                                     |
| DepthToSpace              | 无限制。                                                     |
| Div                       | 无限制。                                                     |
| Einsum                    | 目前该算子equation仅支持以下场景： ibh,hnd->ibndibnd,jbnd->bnijibnd,snd->ibnsijbs,ibns->bnijijbn->bnijbnij,jbnd->ibndibnd,hnd->ibh |
| Elu                       | 无限制。                                                     |
| Equal                     | 无限制。                                                     |
| Erf                       | 无限制。                                                     |
| Exp                       | 无限制。                                                     |
| Expand                    | 无限制。                                                     |
| EyeLike                   | 无限制。                                                     |
| Flatten                   | 无限制。                                                     |
| Floor                     | 无限制。                                                     |
| Gather                    | 无限制。                                                     |
| Gemm                      | 无限制。                                                     |
| GlobalAveragePool         | 仅支持2D（Layout为NHWC）。                                   |
| GlobalLpPool              | 仅支持p=1和p=2, 仅支持2D（Layout为NHWC）。                   |
| GlobalMaxPool             | 仅支持2D（Layout为NHWC）。                                   |
| Greater                   | 无限制。                                                     |
| GRU                       | 无限制。                                                     |
| Hardmax                   | 无限制。                                                     |
| HardSigmoid               | 所有输入数据均为FP16类型。                                   |
| Identity                  | 无限制。                                                     |
| InstanceNormalization     | 输入数值范围为[-5, 5]，reduce轴size小于128*128。             |
| IsInf                     | 无限制。                                                     |
| IsNaN                     | 无限制。                                                     |
| LeakyRelu                 | 无限制。                                                     |
| Less                      | 无限制。                                                     |
| Log                       | 输入数值范围为[4.2e-5, 33900]。                              |
| LogSoftmax                | 无限制。                                                     |
| LpNormalization           | 仅支持p=1和p=2，输入数值范围为[-5,5]，reduce轴size小于4096。 |
| LpPool                    | 仅支持p=1和p=2, 仅支持2D（Layout为NHWC）。                   |
| LSTM                      | 不支持sequence_lens输入。                                    |
| MatMul                    | 无限制。                                                     |
| Max                       | 仅支持二元输入。                                             |
| MaxPool                   | 支持2D与3D。                                                 |
| MaxUnpool                 | 无限制。                                                     |
| Mean                      | 输入数值范围为[-0.2, 0.2]，输入个数小于128。                 |
| MeanVarianceNormalization | 输入数值范围为[-5, 5]，reduce轴size小于128*128。             |
| Min                       | 仅支持二元输入。                                             |
| Mul                       | 无限制。                                                     |
| MaxRoiPool                | 第2个输入rois对应的数据按batch_index升序排列。               |
| Neg                       | 无限制。                                                     |
| Not                       | 无限制。                                                     |
| NonZero                   | 无限制。                                                     |
| OneHot                    | 输入depth参数最大支持320000。                                |
| Or                        | 无限制。                                                     |
| Pad                       | 仅支持constant和edge模式。仅支持四维 （NHWC）和二维（HW）输入的H和W轴上的扩展。 |
| Pow                       | 第二个输入（指数）是值为0.5、1、2或3的常数。或限定第一个输入（底数）范围[0.6, 1.7], 第二个输入（指数）范围[-1.7, 1.7] |
| PRelu                     | 无限制。                                                     |
| Reciprocal                | 无限制。                                                     |
| ReduceL1                  | 输入数值范围为[-0.2, 0.2]，reduce轴size小于128。             |
| ReduceL2                  | 输入数值范围为[-0.2, 0.2]，reduce轴size小于128。             |
| ReduceLogSum              | 输入数值范围为[0, 0.2]，reduce轴size小于128。                |
| ReduceLogSumExp           | 无限制。                                                     |
| ReduceMax                 | 无限制。                                                     |
| ReduceMean                | 输入数值范围为[-0.2, 0.2]，reduce轴size小于128。             |
| ReduceMin                 | 无限制。                                                     |
| ReduceProd                | 输入数值范围为[-1.4, 1.4]，reduce轴size小于32。              |
| ReduceSum                 | 输入数值范围为[-0.2, 0.2]，reduce轴size小于128。             |
| ReduceSumSquare           | 输入数值范围为[-0.45, 0.45]，reduce轴size小于128。           |
| Relu                      | 无限制。                                                     |
| Reshape                   | 无限制。                                                     |
| RNN                       | 不支持sequence_lens输入。                                    |
| Scatter/ScatterElements   | 仅支持indices.shape = updates.shape，且indices与data的shape除了axis轴外，其他维度的大小相同。不支持indices包含重复值。 |
| Selu                      | 无限制。                                                     |
| Shape                     | 无限制。                                                     |
| Shrink                    | 无限制。                                                     |
| Sigmoid                   | 无限制。                                                     |
| Sign                      | 无限制。                                                     |
| Sin                       | 输入数值范围为[-5.2, 5.2]。                                  |
| Sinh                      | 无限制。                                                     |
| Size                      | 无限制。                                                     |
| Slice                     | 无限制。                                                     |
| Softmax                   | 无限制。                                                     |
| Softplus                  | 无限制。                                                     |
| Softsign                  | 无限制。                                                     |
| SpaceToDepth              | 无限制。                                                     |
| Split                     | 无限制。                                                     |
| Sqrt                      | 无限制。                                                     |
| Squeeze                   | 无限制。                                                     |
| Sub                       | 无限制。                                                     |
| Sum                       | 输入数值范围为[-0.2, 0.2]，输入个数小于128。                 |
| Tan                       | 输入数值范围为[-1.48, 1.48]。                                |
| Tanh                      | 无限制。                                                     |
| Tile                      | 无限制。                                                     |
| TopK                      | 无限制。                                                     |
| Transpose                 | 无限制。                                                     |
| Unsqueeze                 | 无限制。                                                     |
| Upsample                  | 支持2D与3D。在最新的ONNX算子中已弃用，但在ONNX opset=9中该算子仍正常使用。 |
| Where                     | 无限制。                                                     |
| Xor                       | 无限制。                                                     |

说明：基于ONNX Opset 9，统计出TensorTurbo目前暂不支持的ONNX算子共12个，包括Compress、Dropout、If、LRN、Loop、Multinomial、RandomNormal、RandomNormalLike、RandomUniform、RandomUniformLike、Scan、TfidfVectorizer。

## ONNX算子（其他Opset版本）

基于ONNX的其他Opset版本，TensorTurbo支持的ONNX算子共计12个，算子列表如下：

| **算子**           | **限制说明**                                                 |
| ------------------ | ------------------------------------------------------------ |
| CumSum             | 不支持axis=None，输入数值范围为[-0.2, 0.2]，reduce轴size小于128。 |
| Mod                | 双输入均为整数，输入数值范围为[-256, 256]，第二个输入非0。   |
| GatherND           | 仅支持batch_dims=0。                                         |
| GatherElements     | output.shape = indices.shape，且indices维度小于或等于data维度。 |
| GreaterOrEqual     | 无限制。                                                     |
| LessOrEqual        | 无限制。                                                     |
| LayerNormalization | 仅支持axis=-1，输入范围[-0.2,0.2]，reduce轴size小于4096。    |
| Resize             | 支持2D与3D。目前支持的插值模式有nearest_neighbor和linear两种模式。目前支持的坐标变换模式有half_pixel 、align_corners和asymmetric。 |
| ReverseSequence    | 仅支持sequence_lens输入为常数，且其值与input输入的shape在time_axis轴上值相等。 |
| Round              | 无限制。                                                     |
| RoiAlign           | 第2个输入rois对应的数据按batch_index升序排列。               |
| ScatterND          | 不支持indices包含重复值。                                    |

## TensorTurbo算子层额外支持的算子

TensorTurbo支持的非ONNX算子共计15个，算子列表如下：

| **算子**            | **限制说明**                                           |
| ------------------- | ------------------------------------------------------ |
| argsort             | 仅支持axis=-1（最后一个轴排序）。                      |
| any                 | 不支持reduce轴绑核。                                   |
| broadcast_to        | 无限制。                                               |
| deformable_conv2d   | 仅支持batch size=1。在H轴与W轴上的大小不超过2048。     |
| depthwise_conv2d    | dilations在各维度的取值应相同。kernel大小不超过9*9。   |
| full                | 无限制。                                               |
| floor_divide        | 双输入均为整数，输入范围[-1024, 1024]，第二个输入非0。 |
| gelu                | 无限制。                                               |
| mish                | 无限制。                                               |
| non_max_suppression | 无限制。                                               |
| not_equal           | 无限制。                                               |
| opencv_magnitude    | 无限制。                                               |
| relu6               | 无限制。                                               |
| swish               | 无限制。                                               |
| sort                | 仅支持axis=-1（最后一个轴排序）。                      |

