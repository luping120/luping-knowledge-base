关于dockerfile的文件编写

关键字
---
## FROM 
	镜像名
```dockerfile
FROM centos:latest
# 这个意思是拉取centos:latest作为构建镜像
# tag latest 的意思是最新版本
```
---
## RUN 
	运行的命令
```dockerfile
# EXAM
RUN ls
# 代表运行ls这个命令
RUN yum update
# 代表进行yum缓存更新
# ...以此类推
```
接下来我们使用RUN参数进行更换yum源
```dockerfile
# 这是方法一
FROM centos:8

# 将原有的 mirrorlist 注释掉，并将 baseurl 指向 CentOS 官方归档源（vault）
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-* && \
    sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

# 清理缓存并生成新的 YUM 缓存
RUN yum clean all && yum makecache -y
```
如果镜像里面没有repo配置文件呢？
```dockerfile
# 这是方法二
FROM centos:8

# 1. 删除可能存在的其他 repo 文件（如果目录为空可忽略）
RUN rm -rf /etc/yum.repos.d/*.repo

# 2. 创建 CentOS-Base.repo 并写入可用源（使用 vault.centos.org 归档源）
RUN cat > /etc/yum.repos.d/CentOS-Base.repo <<EOF
[base]
name=CentOS-8 - Base
baseurl=http://vault.centos.org/8.5.2111/BaseOS/\$basearch/os/
gpgcheck=1
gpgkey=http://vault.centos.org/8.5.2111/BaseOS/\$basearch/os/RPM-GPG-KEY-CentOS-Official

[extras]
name=CentOS-8 - Extras
baseurl=http://vault.centos.org/8.5.2111/extras/\$basearch/os/
gpgcheck=1
gpgkey=http://vault.centos.org/8.5.2111/extras/\$basearch/os/RPM-GPG-KEY-CentOS-Official

[appstream]
name=CentOS-8 - AppStream
baseurl=http://vault.centos.org/8.5.2111/AppStream/\$basearch/os/
gpgcheck=1
gpgkey=http://vault.centos.org/8.5.2111/AppStream/\$basearch/os/RPM-GPG-KEY-CentOS-Official
EOF

# 3. 生成 yum 缓存
RUN yum clean all && yum makecache -y

```
---

## CMD
	指定容器创建时的命令 （可以被覆盖）
	类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

- CMD 在docker run 时运行。
- RUN 是在 docker build 也就是镜像构建的时候。


```dockerfile // EXAM
CMD <shell 命令>

CMD ["<可执行的文件或命令>", <"param1">, <"param2">]

CMD [<"param1">, <"param2">] 
# 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```

推荐使用第二种格式，执行过程比较明确
```dockerfile
# 实例一 启动一个交互式shell
FROM centos8:latest

CMD ["/bin/bash"]
```
**效果**：`docker run -it centos8:custom` 会直接进入 bash 终端；若运行 `docker run -it centos8:custom cat /etc/os-release` 则覆盖 CMD 执行 `cat`。

```dockerfile
# 实例二 执行一次性任务
FROM centos8:latest

CMD ["yum","list","installed"]
```
**效果**：默认列出所有已安装的包，适合简单的验证镜像。

```dockerfile
# 实例三 启动一个简单的 HTTP 服务（假设已安装 `httpd`）
FROM centos8:latest

RUN yum install -y httpd
# 代表开放80端口
EXPOSE 80 

CMD ["/usr/sbin/httpd", "-DFOREGROUND"]

```
**效果**：前台运行 Apache，保持容器存活