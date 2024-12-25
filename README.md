# Mirror-acceleration
To help users in mainland China access container image repositories more quickly, this configuration file provides mirror acceleration configurations. It can significantly improve the speed of pulling images from repositories such as Docker Hub, GCR, Quay.io, and GitHub Container Registry.  

## Configuration solution
Just place the certs.d directory under the /etc/containerd/ path.  
```bash
pwd
/etc/containerd
root@h100-node:/etc/containerd# tree
.
├── certs.d
│   ├── docker.elastic.co
│   │   └── hosts.toml
│   ├── docker.io
│   │   └── hosts.toml
│   ├── gcr.io
│   │   └── hosts.toml
│   ├── ghcr.io
│   │   └── hosts.toml
│   ├── k8s.gcr.io
│   │   └── hosts.toml
│   ├── mcr.microsoft.com
│   │   └── hosts.toml
│   ├── nvcr.io
│   │   └── hosts.toml
│   ├── quay.io
│   │   └── hosts.toml
│   └── registry.k8s.io
│       └── hosts.toml
└── config.toml
```
## Remark
Containerd在启动时指定默认的配置文件路径为/etc/containerd/config.toml，后续所有镜像仓库相关的配置可以在该文件中进行热加载，无需重启Containerd。如果您使用不同的路径，可以根据实际情况进行调整。

### 1.确保配置文件包含`config_path`。
执行以下命令，确认默认配置文件中是否存在`config_path` 配置（例如`"/etc/containerd/cert.d"`）。

```bash
cat /etc/containerd/config.toml |grep config_path -C 5
```
如果不存在，您可以在默认配置文件中添加以下配置。

说明
若已有[plugins."io.containerd.grpc.v1.cri".registry]，则在下面只需添加config_path，注意缩进。若没有，则可以在任意地方写入。
```bash
...
   [plugins."io.containerd.grpc.v1.cri".registry]
     config_path = "/etc/containerd/certs.d"
```
### 2.检查`mirror`相关配置。
检查`/etc/containerd/config.toml`配置文件中是否存在与`mirror`相关的配置。如果已存在，请进行清理，以避免冲突。
```bash
...
[plugins."io.containerd.grpc.v1.cri".registry.mirrors]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
    endpoint = ["https://registry-1.docker.io"]
```
### 3.重启`containerd`。
如果默认配置文件发生更改，则可执行以下命令，重启`Containerd`。
```bash
systemctl restart containerd
```
若启动失败，执行以下命令。检查失败原因，通常是配置文件仍有冲突导致，您可以依据报错做相应调整。
```bash
journalctl -u containerd
```
### 4.创建镜像加速器配置文件
>将certs.d目录存放到/etc/containerd/ 路径下即可。
### 5.拉取Docker镜像验证加速是否生效
```bash
nerdctl pull redis:latest
crictl pull redis:latest
docker pull redis:latest
```
