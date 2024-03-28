# Kubernetes Device Plugin 使用手册

## 简介 

MOFFETT Kubernetes Device Plugin 是运行在 Kubernetes 上的 DaemonSet，并自动提供以下支持：
* 向 Kubernetes 暴露集群中每个节点上的墨芯 AI 加速卡设备数量。
* 持续监控墨芯 AI 加速卡设备的健康状况。
* 在 Kubernetes 集群中运行使用墨芯 AI 加速卡的容器。

## 注意事项

  * 使用 v1beta1 版本的 Kubernetes device plugin API，支持 Kubernetes 1.10 以上的版本
  * plugin 目前需要以特权模式运行
  * 对墨芯 AI 加速卡的修改操作，如reboot等，之后都应该重启plugin

## 版本兼容

| plugin | SOLA ToolKit |
| :----: | :----------: |
| v0.1.0 |   >= 3.5.0   |

## 如何部署

### 准备

  1. 安装好 SOLA ToolKit 的宿主机（需注意 [plugin 和 SOLA 的兼容关系](#版本兼容)）

  2. Kubernetes version >= 1.10

  3. 获取 plugin 镜像:

     * 在线版本：

      ```shell
      docker pull moffett/k8s-device-plugin:v0.1.0
      docker images |grep k8s-device-plugin
      ```

     * 离线版本：

     ```shell
     wget https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/CloudNative/moffett-k8s-device-plugin-v0.1.0.tar
     docker load -i moffett-k8s-device-plugin-v0.1.0.tar
     docker images |grep k8s-device-plugin
     ```

### 部署

*  moffett-device-plugin.yml 示例

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: moffett-device-plugin-daemonset
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: moffett-device-plugin-ds
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: moffett-device-plugin-ds
    spec:
      tolerations:
      - key: moffett.ai/spu
        operator: Exists
        effect: NoSchedule
      # Mark this pod as a critical add-on; when enabled, the critical add-on
      # scheduler reserves resources for critical add-on pods so that they can
      # be rescheduled after a failure.
      # See https://kubernetes.io/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/
      priorityClassName: "system-node-critical"
      containers:
      - image: moffett/k8s-device-plugin:v0.1.0
        name: moffett-device-plugin-ctr
        securityContext:
          privileged: true
        volumeMounts:
        - name: device-plugin
          mountPath: /var/lib/kubelet/device-plugins
      volumes:
      - name: device-plugin
        hostPath:
          path: /var/lib/kubelet/device-plugins
```

*  使用yaml，在k8s中启动plugin

```shell
kubectl create -f moffett-device-plugin.yml
```

## 如何在k8s中使用墨芯AI加速卡

*  获取 SOLA 基础镜像
*  在k8s中启动镜像，下面是请求墨芯 AI 加速卡设备资源(`moffett.ai/spu`)的Pod示例，假设我们运行的镜像为 `sola:3.5.0`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: spu-pod
spec:
  restartPolicy: Never
  containers:
  - name: sola-container
    image: sola:3.5.0
    securityContext:
      privileged: true
    resources:
      limits:
        moffett.ai/spu: 2
  tolerations:
  - key: moffett.ai/spu
    operator: Exists
    effect: NoSchedule
#
# 这个 pod 需要两个 moffett.ai/spu 设备
# 而且只能够调度到满足需求的节点上
#
# 如果该节点中有 2 个以上的设备可用，其余的可供其他 Pod 使用
```
