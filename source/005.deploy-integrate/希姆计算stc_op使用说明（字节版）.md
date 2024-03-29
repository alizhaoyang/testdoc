# 希姆计算stc_op使用说明
## 版本历史

| **文档版本** | **对应产品版本**  | **作者** | **日期**   | **说明**                      |
| ------------ | ----------------- | -------- | ---------- | ----------------------------- |
| V1.4.0       | TensorTurbo 1.4.0 | 希姆计算 | 2022-04-29 | API说明中新增参数`op_index`。 |
| V1.3.0       | TensorTurbo 1.3.0 | 希姆计算 | 2022-03-31 | 初始对外版本。                |

## 简介

stc_op可用于拼接模型并导出为TensorFlow的.pb文件。

使用希姆计算TensorTurbo编译模型得到.stcobj文件后，可以直接使用NPU的Runtime接口加载运行，但单次只能加载一个文件，在涉及较多模型文件的任务中相对不便。

stc_op可以将.stcobj文件封装为算子，并将多个封装好的算子构造新的TensorFlow的计算图。您只需执行该计算图即可完成相应的任务，简化了执行推理的操作。

## 注意事项

本工具专为字节推荐系统（DSSM）的粗排、精排场景定制，模型参数仅支持字节粗排模型和精排模型编译得到的.stcobj文件。

## 操作步骤

### 前提条件

- 服务器已安装HPE。详细的安装步骤，请参见[希姆计算异构环境安装指南](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/)。
- 服务器已安装Python，且Python版本不低于Python3.6。
- 服务器已安装TensorFlow1.15。
- 联系您的希姆计算接口人获取stc_op的whl包和源码。

### 操作步骤

1. 执行以下命令，安装stc_op。

   > 注意：请将命令中的文件名称替换为您实际拿到的安装包名称。

   ```bash
   $ pip install stc_op-0.0.1-cp37-cp37m-linux_x86_64.whl
   ```
   
2. 使用stc_op。完成stc_op安装后，调用接口拼接模型即可。详细说明，请参见*API说明*和*使用示例*章节。


## API说明

```Python
def stc_dssm( 
    inputs, 
    part1_stcobj_path, 
    part2_stcobj_path, 
    part2_max_rounds=8,
    op_index=0, 
)
```

| **参数**          | **说明**                                                     | **是否必选** |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| inputs            | 接收TensorFlow placeholder 或TensorFlow Tensor对象，拼接graph的有向无环图。 | 是                                                           |
| part1_stcobj_path | 粗排模型编译后生成的.stcobj文件的路径。                      | 是                                                           |
| part2_stcobj_path | 精排模型编译后生成的.stcobj文件的路径。                      | 是                                                           |
| part2_max_rounds  | 如果kernel2的.stcobj文件中的静态Tensor的输入size是$(x_0, x_1...)$，那么该kernel可以支持的最大动态shape是 $part2_max_rounds*(x_0,x_1...)$。 | 否                                                           |
| op_index          | 当前构建的自定义算子，是graph中的自定义算子的索引标识。      | 否                                                           |

##  使用示例

在下文示例中，使用TensorFlow的接口调用stc_op算子生成一个新的模型，并保存该模型的.pb文件到本地。执行以下代码时，流程如下：

1. 生成stc_op的输入信息，即粗排模型的输入。
2. 传入输入shape信息，构造stc_op算子。
3. 调用TensorFlow的方法，将计算图导出成一个.pb文件。

```Python
import stc_op

def dssm(datasets_path):
    with tf.Graph().as_default() as g:
        u_feature = tf.placeholder(
            shape=[1, 2400], dtype=tf.float16, name="u_features")
        u_idx = tf.placeholder(shape=[1, 50], dtype=tf.int32, name="u_idx")
        u_clk_cid = tf.placeholder(
            shape=[1, 50, 177], dtype=tf.float16, name="u_clk_cid"
        )
        u_clk_c1 = tf.placeholder(
            shape=[1, 50, 177], dtype=tf.float16, name="u_clk_c1")
        u_clk_c2 = tf.placeholder(
            shape=[1, 50, 177], dtype=tf.float16, name="u_clk_c2")
        u_clk_c3 = tf.placeholder(
            shape=[1, 50, 177], dtype=tf.float16, name="u_clk_c3")
        u_clk_src = tf.placeholder(
            shape=[1, 50, 177], dtype=tf.float16, name="u_clk_src"
        )
        u_clk_rdt = tf.placeholder(
            shape=[1, 50, 177], dtype=tf.float16, name="u_clk_rdt"
        )
        i_idx = tf.placeholder(
            shape=[
                None,
            ],
            dtype=tf.uint8,
            name="i_idx",
        )
        i_features_0 = tf.placeholder(
            shape=[None, 176], dtype=tf.float16, name="i_features_0"
        )

        stc_op.stc_dssm(
            inputs=[
                u_feature,
                u_idx,
                u_clk_cid,
                u_clk_c1,
                u_clk_c2,
                u_clk_c3,
                u_clk_src,
                u_clk_rdt,
                i_idx,
                i_features_0,
            ],
            part1_stcobj_path=datasets_path + "/dssm_part1_modify.stcobj",
            part2_stcobj_path=datasets_path + "/dssm_part2_modify.stcobj",
        )
        g.finalize()
    with tf.gfile.FastGFile("./dssm.pb", mode="wb") as f:
        f.write(g.as_graph_def().SerializeToString())
```



