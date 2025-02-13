# 集群配置文件 clusterConfig.yaml

此 YAML 文件包含了集群的各项配置字段，安装之前必须先配置此文件。
该文件将定义部署的负载均衡类型、部署模式、集群节点信息等关键参数。默认位于 `offline/sample/` 目录。

## ClusterConfig 示例

以下是一个 ClusterConfig 文件示例。

```yaml title="clusterConfig.yaml"
apiVersion: provision.daocloud.io/v1alpha2
kind: ClusterConfig
metadata: 
spec:
  clusterName: my-cluster # (1)
  loadBalancer:
    type: metallb # (2)
    istioGatewayVip: xx.xx.xx.xx/32 # (3)
    insightVip: xx.xx.xx.xx/32 # (4)

  # privateKeyPath: /root/.ssh/id_rsa_sample  # (5)

  masterNodes:
    - nodeName: "g-master1" 
      ip: xx.xx.xx.xx
      ansibleUser: "***" 
      ansiblePass: "****"
    - nodeName: "g-master2" 
      ip: xx.xx.xx.xx
      ansibleUser: "***" 
      ansiblePass: "****"
    - nodeName: "g-master3" 
      ip: xx.xx.xx.xx
      ansibleUser: "***" 
      ansiblePass: "****"
  workerNodes:
    - nodeName: "g-worker1"
      ip: xx.xx.xx.xx
      ansibleUser: "***"
      ansiblePass: "****"
      nodeTaints: # (6)
        - "node.daocloud.io/es-only=true:NoSchedule"
    - nodeName: "g-worker2"
      ip: xx.xx.xx.xx
      ansibleUser: "***"
      ansiblePass: "****"
      nodeTaints:  # (7)
        - "node.daocloud.io/es-only=true:NoSchedule"
    - nodeName: "g-worker3"
      ip: xx.xx.xx.xx
      ansibleUser: "***"
      ansiblePass: "****"
      nodeTaints: # (8)
        - "node.daocloud.io/es-only=true:NoSchedule"

  ntpServer: # (9)
    - 0.pool.ntp.org
    - ntp1.aliyun.com
    - ntp.ntsc.ac.cn
  registry: 

    type: built-in # (10)
    # builtinRegistryDomainName: ${跟上述配置仓库地址一致。如果是 built-in，则填写火种节点 IP}。# (11)
    # type: external  # (12)
    # externalRegistry: external-registry.daocloud.io # (13)
    # externalRegistryUsername: admin   # (14)
    # externalRegistryPassword: Harbor12345  # (15)
    # externalScheme: https # (16)
    
    addonOfflinePackagePath: "Please-replace-with-Your-Real-Addon-Offline-Package-PATH-on-bootstrap-Node" # (17)

  # kubean 所需要的仓库配置
  imageConfig: 
    imageRepository: http://${IP_ADDRESS_OF_BOOTSTRAP_NODE} # (18)
    binaryRepository: http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean # (19)

  # RPM 或者 DEB 安装的源头
  repoConfig: 
    repoType: centos # (20)
    isoPath: "Please-replace-with-Your-Real-ISO-PATH-on-bootstrap-Node" # (21) 
    osPackagePath: "Please-replace-with-Your-Real-OS-Package-PATH-on-bootstrap-Node" # (22)
    dockerRepo: "http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/centos/$releasever/os/$basearch" 

    # dockerRepo: "" # (23)
    # dockerRepo: "http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/redhat/$releasever/os/$basearch" # (24)

    extraRepos:
      - http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/centos-iso/\$releasever/os/\$basearch  
      - http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/centos/\$releasever/os/\$basearch 
  k8sVersion: v1.24.7
  auditConfig:
    logPath: /var/log/audit/kube-apiserver-audit.log
    logHostPath: /var/log/kubernetes/audit
    # policyFile: /etc/kubernetes/audit-policy/apiserver-audit-policy.yaml
    # logMaxAge: 30
    # logMaxBackups: 10
    # logMaxSize: 100
    # policyCustomRules: >
    #   - level: None
    #     users: []
    #     verbs: []
    #     resources: []
  network:
    cni: calico
    clusterCIDR: 100.96.0.0/11
    serviceCIDR: 100.64.0.0/13
  cri:
    criProvider: containerd
    # criVersion: 1.6.8 # (25)
```

1. 集群名称
2. 有 3 个可选项：NodePort (default)、metallb、cloudLB (Cloud Controller)，建议生产环境使用 metallb
3. 当 loadBalancer.type 是 metallb 时必填，为 DCE 提供 UI 和 OpenAPI 访问权限
4. 别丢弃/32, 当 loadBalancer.type 是 metallb 时必填，用作 GLobal 集群的 Insight 数据采集入口，子集群的 insight-agent 可以向这个 VIP 报告数据
5. 指定 ssh 私钥，定义后无需再定义节点的 ansibleUser、ansiblePass
6. 对于 7 节点模式：至少 3 个工作节点应该被打上该污点，仅作为 ES 的工作节点
7. 对于 7 节点模式：至少 3 个工作节点应该被打上该污点，仅作为 ES 的工作节点
8. 对于 7 节点模式：至少 3 个工作节点应该被打上该污点，仅作为 ES 的工作节点
9. 可以使用自己搭建的 ntpServer
10. 支持 3 个选项：built-in, external, online
11. 可选。内置镜像仓库的域名，并在每个节点的 /etc/hosts 和 coredns 的 hosts 区域进行域名解析的配置
12. 使用已有的仓库，需要保证网络联通
13. 已有镜像仓库的 IP 地址或者域名
14. 只有 type: external 且推镜像时需要用户名和密码的情况下才需要定义此项
15. 只有 type: external 且推镜像时需要用户名和密码的情况下才需要定义此项
16. 这是一个占位符
17. addon 离线包文件的绝对路径，如果不需要 addon 离线化可以注释掉
18. 如果选择了已有仓库，需要填写外部镜像仓库地址
19. 如果选择了已有仓库，需要填写外部 MinIO 地址
20. `centos` 表示使用 CentOS、RedHat、kylin AlmaLinux 或 Fedora；`debian` 表示使用 Debian；`ubuntu` 表示使用 Ubuntu
21. 操作系统 ISO 文件的绝对路径，不得为空
22. 操作系统 osPackage 文件的绝对路径，不得为空
23. 如果是 kylin，安装器将会选择 containerd，所以需要将 dockerRepo 设置为空
24. 如果是 redhat
25. criVersion 仅在 online 模式下生效，请勿将其设置为 offline 模式

## 关键字段

该 YAML 文件中的关键字段说明，请参阅下表。

| 字段                               | 说明                                                         | 默认值                                                      |
| ---------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- |
| clusterName                        | 在 KuBean Cluster 里的 Global 集群命名                       | -                                                           |
| loadBalancer.type                  | 所使用的 LoadBalancer 的模式，物理环境用 metallb，POC 用 NodePort，公有云和 SDN CNI 环境用 cloudLB | NodePort (default)、metallb、cloudLB (Cloud Controller)     |
| loadBalancer.istioGatewayVip       | 如果负载均衡模式是 metallb，则需要指定一个 VIP，供给 DCE 的 UI 界面和 OpenAPI 访问入口 | -                                                           |
| loadBalancer.insightVip            | 如果负载均衡模式是 metallb，则需要指定一个 VIP，供给 GLobal 集群的 insight 数据收集入口使用，子集群的 insight-agent 可上报数据到这个 VIP | -                                                           |
| privateKeyPath                     | kuBean 部署集群的 SSH 私钥文件路径，如果填写则不需要定义ansibleUser、ansiblePass | -                                                           |
| masterNodes                        | Global 集群：Master 节点列表，包括 nodeName/ip/ansibleUser/ansiblePass 几个关键字段 | -                                                           |
| workerNodes                        | Global 集群：Worker 节点列表，包括 nodeName/ip/ansibleUser/ansiblePass 几个关键字段 | -                                                           |
| registry.type                      | k8s 组件和 DCE 组件的镜像拉取仓库的类型，有 online (在线环境)、built-in (使用火种节点内置的仓库)、external (使用已有的外置仓库) | online                                                      |
| registry.builtinRegistryDomainName | 如果使用 built-in (使用火种节点内置的仓库)，如果需要自动化植入仓库域名到各个节点的 /etc/hosts 和 CoreDNS 的 hosts，可指定域名 | -                                                           |
| registry.externalRegistry          | 如果 registry.type = external 则需要定义该字段需要指定 external 仓库的 IP 或者域名(使用已有的外置仓库)，如果使用 Harbor，需要提前创建相应 Project | -                                                           |
| registry.externalRegistryUsername  | 如果 registry.type = external 则需要定义该字段外置仓库的用户名，用于推送镜像 | -                                                           |
| registry.externalRegistryPassword  | 如果 registry.type = external 则需要定义该字段外置仓库的密码，用于推送镜像 | -                                                           |
| imageConfig.imageRepository        | 如果是离线安装，kuBean 安装集群时的本地镜像仓库来源          | -                                                           |
| imageConfig.binaryRepository       | 如果是离线安装，kuBean 安装集群时的本地二进制仓库来源        | [https://files.m.daocloud.io](https://files.m.daocloud.io/) |
| repoConfig                         | RPM 或者 DEB 安装的源头，如果离线模式下,是安装器启动的 MinIO | -                                                           |
| repoConfig.`repoType`              | 支持 `centos（包含CentOS, RedHat,kylin AlmaLinux or Fedora）、debian、ubuntu` |                                                             |
| repoConfig.`dockerRepo`            | 如果是 kylin，安装器将会选择 containerd，所以需要将 dockerRepo 设置为空 |                                                             |
| repoConfig.isoPath                 | 操作系统 ISO 文件的路径 ，离线模式下不能为空                 | -                                                           |
| k8sVersion                         | kuBean 安装集群的 K8s 版本必须跟 KuBean 和离线包相匹配       | -                                                           |
| ntpServer                          | 可用的 NTP 服务器，供给新节点同步时间                        | -                                                           |
| network.cni                        | CNI 选择，比如 Calico、Cilium                                | calico                                                      |
| network.clusterCIDR                | Cluster CIDR                                                 | -                                                           |
| network.serviceCIDR                | Service CIDR                                                 | -                                                           |
| auditConfig                        | k8s api-server 的审计日志配置                                | 默认关闭                                                    |

## 场景配置说明

### 负载均衡选择

支持 3 个选项：NodePort、metallb、cloudLB

```yaml
  loadBalancer:
    type: metallb # (1)
    istioGatewayVip: xx.xx.xx.xx/32 # (2)
    insightVip: xx.xx.xx.xx/32  # (3)

	  # 如果 loadBalancer 是 NodePort
    type: NodePort

	  # cloudLB 特性目前处于 todo 状态
```

1. 支持 3 个选项：NodePort(default), metallb, cloudLB (Cloud Controller)
2. 如果 loadBalancer != metallb，请移除这一行
3. 记住要保留 /32

### 1 / 4 / 7 节点模式

```yaml
  # all in one 模式
  masterNodes:
    - nodeName: "g-master1" 
      ip: xx.xx.xx.xx
      ansibleUser: "root"
      ansiblePass: "dangerous"

  # 3 节点模式
  masterNodes:
    - nodeName: "g-master1" 
      ip: xx.xx.xx.xx
      ansibleUser: "root"
      ansiblePass: "dangerous"
    - nodeName: "g-master1" 
      ip: xx.xx.xx.xx
      ansibleUser: "root"
      ansiblePass: "dangerous"
    - nodeName: "g-master1" 
      ip: xx.xx.xx.xx
      ansibleUser: "root"
      ansiblePass: "dangerous"

  # 7 节点模式
  masterNodes:
    - nodeName: "g-master1" 
      ip: xx.xx.xx.xx
      ansibleUser: "root"
      ansiblePass: "dangerous"
    - nodeName: "g-master1" 
      ip: xx.xx.xx.xx
      ansibleUser: "root"
      ansiblePass: "dangerous"
    - nodeName: "g-master1" 
      ip: xx.xx.xx.xx
      ansibleUser: "root"
      ansiblePass: "dangerous"
  workerNodes:
    - nodeName: "g-worker1"
      ip: xx.xx.xx.xx
      ansibleUser: "root"
      ansiblePass: "dangerous"
      nodeTaints:   # (1)
        - "node.daocloud.io/es-only=true:NoSchedule"
    - nodeName: "g-worker2"
      ip: xx.xx.xx.xx
      ansibleUser: "root"
      ansiblePass: "dangerous"
      nodeTaints:   # (2)
        - "node.daocloud.io/es-only=true:NoSchedule"
    - nodeName: "g-worker2"
      ip: xx.xx.xx.xx
      ansibleUser: "root"
      ansiblePass: "dangerous"
      nodeTaints:   # (3)
        - "node.daocloud.io/es-only=true:NoSchedule"
```

1. 对于 7 节点模式：至少 3 个工作节点应该被打上该污点，仅作为 ES 的工作节点
2. 对于 7 节点模式：至少 3 个工作节点应该被打上该污点，仅作为 ES 的工作节点
3. 对于 7 节点模式：至少 3 个工作节点应该被打上该污点，仅作为 ES 的工作节点

### 镜像仓库模式

支持三种模式：online, built-in, external

```yaml
  registry: 

    type: online # (1)

    type: built-in # (2)
    builtinRegistryDomainName: # (3)

    type: external # (4)
    externalRegistry: external-registry.daocloud.io # (5)
    externalRegistryUsername: admin      # (6)
    externalRegistryPassword: Harbor12345  # (7)
    externalScheme: https # (8)
```

1. 在线模式，设置为在线模式后，无需定义 spec.imageConfig、spec.repoConfig
2. 使用内置的仓库，由安装器进行部署安装
3. 可选。内置镜像仓库的域名，并在每个节点的 /etc/hosts 和 coredns 的 hosts 区域进行域名解析的配置
4. 使用已有的仓库，需要保证网络联通
5. 已有镜像仓库的 IP 地址或者域名
6. 只有 type: external 且推镜像时需要用户名和密码的情况下才需要定义此项
7. 只有 type: external 且推镜像时需要用户名和密码的情况下才需要定义此项
8. 这是一个占位符

### Kubean 组件安装集群配置

```yaml
  # kubean 所需要的仓库配置
  imageConfig: 
    imageRepository: http://${IP_ADDRESS_OF_BOOTSTRAP_NODE} 或者 上述“自定义的内置域名”} # (1)
    binaryRepository: http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean # (2)

  # RPM 或者 DEB 安装的源头
  repoConfig: 
    repoType: centos # (3)
    isoPath: "Please-replace-with-Your-Real-ISO-PATH-on-bootstrap-Node" # (4)
    osPackagePath: "Please-replace-with-Your-Real-OS-Package-PATH-on-bootstrap-Node" # (5)
    dockerRepo: "http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/centos/$releasever/os/$basearch" 

    # dockerRepo: "" # (6)
    # dockerRepo: "http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/redhat/$releasever/os/$basearch" # (7)
    
    extraRepos:
      - http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/centos-iso/\$releasever/os/\$basearch 
      - http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/centos/\$releasever/os/\$basearch
 
      #  如果系统是 RedHat 8，需要使用以下参数
      #- http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/redhat-iso/\$releasever/os/\$basearch/AppStream
      #- http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/redhat-iso/\$releasever/os/\$basearch/BaseOS
      #- http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/redhat/\$releasever/os/\$basearch

      #  如果系统是 kylin，需要使用以下参数
      #- http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/kylin-iso/\$releasever/os/\$basearch
      #- http://${IP_ADDRESS_OF_BOOTSTRAP_NODE}:9000/kubean/kylin/\$releasever/os/\$basearch
```

1. 如果选择了已有的仓库，需要填写外部镜像仓库地址
2. 如果选择了已有的仓库，需要填写外部 MinIO 地址
3. `centos` 表示使用 CentOS、RedHat、kylin AlmaLinux 或 Fedora；`debian` 表示使用 Debian；`ubuntu` 表示使用 Ubuntu
4. 操作系统 ISO 文件路径，不得为空
5. 操作系统 osPackage 文件的绝对路径，不得为空
6. 如果是 kylin，安装器将会选择 containerd，所以需要将 dockerRepo 设置为空
7. 如果是 redhat
