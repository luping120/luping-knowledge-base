关于dockerfile的文件编写

关键字

FROM 
	镜像名
```dockerfile
FROM centos:latest
// 这个意思是拉取centos:latest作为构建镜像
// tag latest 的意思是最新版本
```
---
RUN 
	运行的命令
```dockerfile
// EXAM
RUN ls
// 代表运行ls这个命令
RUN yum update
// 代表进行yum缓存更新
// ...以此类推
```
接下来我们使用RUN参数进行更换yum源
```dockerfile
RUN 
```
---

CMD
	指定容器创建时的命令 （可以被覆盖）
	类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

- CMD 在docker run 时运行。
- RUN 是在 docker build 也就是镜像构建的时候。


```dockerfile // EXAM
CMD <shell 命令>
CMD ["<可执行的文件或命令>", <"param1">, <"param2">]
CMD [<"param1">, <"param2">] # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```

推荐使用第二种格式，执行过程比较明确