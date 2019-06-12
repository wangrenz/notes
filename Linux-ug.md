#### NFS文件系统
网络文件系统（英语：Network File System，缩写作 NFS）是一种分布式文件系统协议，基于rpc协议之上。

+ Linux下服务端部署

安装必要的包，以CentOS 7为例：`yum install rpcbind nfs-utils`。

配置`/etc/exports`
```bash
/mnt *(ro,sync,insecure)
# 服务端共享目录， *: 所有客服端，或者指定IP段。 在内网挂载：如阿里云，单位内网。`insecure`参数一般不用加。
# 如果跨公网挂载，需要加`insecure`或将端口改为大于1024的端口
```
+ 启动服务

加入开机启动
```bash
systelctl enable rpcbind
systemctl enable nfs
```
开启服务
```bash
systelctl start rpcbind
systemctl start nfs
```

+ 客户端挂载
安装`rpcbind`并开启该服务。
手动挂载
```bash
mount -v -t nfs ip:/mnt /mnt 
```
加入开机启动`/etc/fstab`
```bash
ip:/mnt /mnt nfs ro 0 0
```
> 注意： 1. 在关闭nfs服务前务必先umount挂载点，再关闭nfs服务，最后关闭rpcbind。
避免由于nfs服务关闭而客户端还持续访问，造成客户端不能umount且负载过高的问题。
