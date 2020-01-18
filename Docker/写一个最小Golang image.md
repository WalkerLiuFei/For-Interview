# Build smallest docker Image 

[build smalled docker image]() 

## Docker APP的框架

```
my-image/
├── assets
│   ├── entrypoint.sh
│   └── install.sh
├── build.sh
├── Dockerfile
├── README.md
└── VERSION
```

1. `build.sh` ： 通过这里执行 `docker build` 之前的本地命令，比如说这里的Golang应用，为了编译为最小的image,我需要先将应用编译好。
2. `Dockerfile` : 不必多说
3. `install.sh` : 这里是拷贝到container中并且运行命令的脚本，这个叫脚本用来替代`DockerFile`的`RUN`命令
4. `entrypoint` :  `container`的入口脚本

最后的example [echo-server](https://github.com/WalkerLiuFei/docker-image-example)

### 遇到的问题

1. Image编译好后，作为container的启动image无法启动，提示permission denial
2. strach 上面没有 chmod命令，所以需要在 builder的container上面完成 权限改变的操作
3. 启动之后马上就退出了，通过`docker inspect containerID` 查看推出原因
4. 发现推出的 exit code为1 ，说明是因为 应用的原因导致的container 退出
5. 发现代码的package 写错了，修复后重新build 依然有错
6. 通过 `alpine` 构建的image可以正常跑
7. 通过`scratch` 构建的image在编译``GOlang` binary时必须关闭 cgo
8. 关闭之后成功了

### 构建安全的image

1. 不要以`root`权限运行container内的应用
2. 在DockerFile 中使用`copy`文件夹时，路径末尾要加slash `/` 标明
3. 构建成功