# Kubernetes Device Plugin 使用说明

## **概述**

MOFFETT Kubernetes Device Plugin 是运行在 Kubernetes 上的 DaemonSet，具备以下核心功能：

- 自动将集群中每个节点上的墨芯 AI 加速卡设备数量暴露给 Kubernetes，实现设备资源的透明化管理和动态调度。
- 实时监控墨芯 AI 加速卡设备的健康状态，确保设备稳定运行，及时发现并处理潜在问题。
- 允许在 Kubernetes 集群中无缝运行依赖墨芯 AI 加速卡的容器，提升容器化应用的性能与效率。

通过这一插件，Kubernetes 能够更好地管理和利用墨芯 AI 加速卡资源，为 AI 应用提供强大的算力支持。

## **前提条件**

- 已安装墨芯 AI 计算卡，且能被系统正确识别。
- 在宿主机上安装好 SOLA ToolKit，且 SOLA ToolKit 的版本>= 3.5.0。
- 系统已安装 Docker，且 Docker 的版本大于 18.09。
- Kubernetes 的版本不低于 Kubernetes V1.10。

## **部署** **Device Plugin**

> **注意**：plugin 目前需要以特权模式运行，且对墨芯 AI 加速卡的修改操作后都应该重启 plugin。

1. **获取 plugin 镜像：** plugin 镜像分为在线版和离线版，获取方式如下：

   1. 获取在线版本 device plugin 镜像：

      ```Bash
       $ docker pull moffett/k8s-device-plugin:v0.1.0
       $ docker images | grep k8s-device-plugin
      ```

   2. 获取离线版本 device plugin 镜像：

      ```Bash
       $ wget https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/CloudNative/moffett-k8s-device-plugin-v0.1.0.tar
       $ docker load -i moffett-k8s-device-plugin-v0.1.0.tar
       $ docker images | grep k8s-device-plugin
      ```

2. **新建部署文件**： 在本地新建一个文件，名为 moffett-device-plugin.yml，用于部署 device plugin，参考内容示例如下：

   ```Bash
   #moffett-device-plugin.yml 
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

3. 在 Yaml 文件所在目录，执行以下命令，在 Kubernetes 集群中部署 plugin。

   ```Bash
     $ kubectl create -f moffett-device-plugin.yml
   ```

## **使用** Device Plugin

1. 获取 SOLA 基础镜像。

2. 在 k8s 中启动镜像。 下面是请求墨芯 AI 加速卡设备资源（`moffett.ai/spu`）的 Pod 示例，假设我们运行的镜像为 `sola:3.5.0`。

   ```Bash
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
     # 这个 pod 需要两个 moffett.ai/spu 设备
     # 而且只能够调度到满足需求的节点上
     # 如果该节点中有 2 个以上的设备可用，其余的可供其他 Pod 使用
   ```
