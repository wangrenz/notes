### 程序并行，集群使用记录
>程序的并行一般有两种方式，多线程和多进程。即OpenMP(共享内存)和MPI(分布式)。OpenMP为编译器自带，加上对应参数即可。
如：`gfortran -fopenmp`，`ifort -qopenmp`。MPI并行库需要额外提供，有`Intel MPI，OpenMPI，mvapich`(基于mpich增加了IB网络的支持)。
气象海洋模式的大规模并行计算都是用到上百个节点，而程序里并行代码都已经给定。我们做的就是根据具体的服务器，编译环境，并行库，
测试出一个计算速度最快，使用核数适中的方案。

#### 1 编译器使用
现在大部分超算都是`Intel`处理器，而`Intel`编译器针对它做了一定优化。所以最好使用`Intel compiler`。但是部分版本`Intel compiler`编译出的模式存在一些问题，比如运行时`core dump`，以及并行效率不高等等。
所以建议使用最新update版本，比如`2017update5,2018update3`等。对于一些必要的库，如`hdf5,netcdf`。我的想法是静态编译进去，这样模式很容易迁移到其他集群上去。
不用去安装编译器和库来重新编译（前提是glibc一致）。具体做法为：
1. `WRF`模式
修改`configure.wrf`中的`LIB_EXTERNAL`，改为如下形式：
```bash
LIB_EXTERNAL    = \
                  -L$(WRF_SRC_ROOT_DIR)/external/io_netcdf -lwrfio_nf /netcdfpath/lib/libnetcdff.a \
                  /netcdfpath/lib/libnetcdf.a /hdf5path/lib/libhdf5_fortran.a /hdf5path/lib/libhdf5_hl.a \
                  /hdf5path/lib/libhdf5.a /usr/lib64/libz.a -ldl 
```
其中`netcdfpath`，`hdf5path`分别为`netcdf`，`hdf5`的安装路径。
修改`external/io_netcdf/makefile`中的`LIBS`和`LIBFFS`，改为如下形式：
```bash
LIBS    = $(LIB_LOCAL) -L$(NETCDFPATH)/lib /netcdfpath/lib/libnetcdff.a /netcdfpath/lib/libnetcdf.a \
          /hdf5path/lib/libhdf5_fortran.a /hdf5path/lib/libhdf5_hl.a \
          /hdf5path/lib/libhdf5.a /usr/lib64/libz.a -lm -ldl
LIBFFS  = $(LIB_LOCAL) -L$(NETCDFPATH)/lib /netcdfpath/lib/libnetcdff.a /netcdfpath/lib/libnetcdf.a \
          /hdf5path/lib/libhdf5_fortran.a /hdf5path/lib/libhdf5_hl.a \
          /hdf5path/lib/libhdf5.a /usr/lib64/libz.a -lm -ldl
```
得到的`wrf.exe`依赖如下：
```bash
wang@localhost$ ldd wrf.exe
        linux-vdso.so.1 =>  (0x00007ffd7618c000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00002b5ff2eda000)
        libmpifort.so.12 => /app/intel/compilers_and_libraries_2018.3.222/linux/mpi/intel64/lib/libmpifort.so.12 (0x00002b5ff30de000)
        libmpi.so.12 => /app/intel/compilers_and_libraries_2018.3.222/linux/mpi/intel64/lib/libmpi.so.12 (0x00002b5ff3487000)
        librt.so.1 => /lib64/librt.so.1 (0x00002b5ff4116000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00002b5ff431e000)
        libm.so.6 => /lib64/libm.so.6 (0x00002b5ff453a000)
        libiomp5.so => /app/intel/compilers_and_libraries_2018.3.222/linux/compiler/lib/intel64/libiomp5.so (0x00002b5ff483c000)
        libc.so.6 => /lib64/libc.so.6 (0x00002b5ff4c17000)
        libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00002b5ff4fe4000)
        /lib64/ld-linux-x86-64.so.2 (0x00002b5ff2cb6000)
```

1. `FVCOM`模式
思路和`WRF`模式编译一致，链接时写静态库的全路径。具体做法如下：
修改`make.inc`中的`IOLIBS`：
```bash
IOLIBS  = -L. /netcdfpath/lib/libnetcdff.a /netcdfpath/lib/libnetcdf.a \
          /hdf5path/lib/libhdf5_fortran.a /hdf5path/lib/libhdf5_hl.a 
          /hdf5path/lib/libhdf5.a /usr/lib64/libz.a -lm -ldl
```
得到的`fvcom`为：
```bash
wang@localhost$ ldd fvcom
        linux-vdso.so.1 =>  (0x00007ffd96156000)
        libm.so.6 => /lib64/libm.so.6 (0x00002b0ec362b000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00002b0ec392d000)
        libmpifort.so.12 => /app/intel/compilers_and_libraries_2018.3.222/linux/mpi/intel64/lib/libmpifort.so.12 (0x00002b0ec3b31000)
        libmpi.so.12 => /app/intel/compilers_and_libraries_2018.3.222/linux/mpi/intel64/lib/libmpi.so.12 (0x00002b0ec3eda000)
        librt.so.1 => /lib64/librt.so.1 (0x00002b0ec4b69000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00002b0ec4d71000)
        libc.so.6 => /lib64/libc.so.6 (0x00002b0ec4f8d000)
        libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00002b0ec535a000)
        /lib64/ld-linux-x86-64.so.2 (0x00002b0ec3407000)

```

#### 2 数值模式的计算方案
`WRF`之类有限差分模式，主要体现在空间差分上，所以他所使用的核数一般和最外层的格点数相同。而`FVCOM`有限体积模式，
并行时需要把空间分为几个区域（区域数即核数），这就和模拟的空间大小分辨率等有很大关系，需要具体测试。

#### 3 HPC结构
##### 3.1 集群体系结构
集群中一个节点相当于一台计算机，按功能其类型主要有，用户节点，控制节点，管理节点，存储节点，计算节点。
1. 用户节点

即登录节点，用户在这些节点提交作业。一般可以访问外网，下载数据。而且安装了许多实用的小工具。譬如：wget,axel,imagemagick,htop。
如果没有装，后面讲在集群安装软件的方法。

2. 控制节点

调度计算节点上的作业，通常集群的作业调度程序(如PBS)应该运行在这个节点上。但大多集群的登录节点，或管理节点同时负责作业调度。

3. 管理节点

也就是mgt节点，监控集群中各个节点和网络的运行状况。通常的集群的管理软件也运行在这个节点上。一般只能root才能登录，没有挂载并行文件系统。

4. 存储节点

也叫IO节点，这些节点上挂载了TB级的并行文件系统，然后共享到其他节点。如果这些节点挂载出问题，那么其他节点并行文件系统也就不能用了。

5. 计算节点

计算节点是整个集群系统的计算核心。它的功能就是执行计算。一般一个节点10几到20几核不等。

##### 3.2 集群网络结构
节点间一般是通过以太网来实现tcp通信。但是如果并行计算，这种千兆以太网的速度是远远不够的。所以现在大多集群并行计算都是通过IB网络来通信，
速度达到了56Gb/s。并行程序执行时的通信结构有如下方式：`shm|dapl|tcp|tmi|ofa|ofi`。shm为共享内存，tcp为以太网，ofa为IB网络。

以`Intel MPI`为例。`FVCOM`模式运行时，设置环境变量`export I_MPI_FABRICS=shm:ofa`。节点内共享内存，节点间IB网络。
经测试这种搭配是效率最高的。`WRF`模式对这个设置不敏感，`I_MPI_FABRICS=shm:ofa`和`I_MPI_FABRICS=ofa`速度基本一致，后者为节点内，节点间都用IB网络。

#### 4 PBS作业系统
常用的命令，适用于PBS-Pro,torque。
1. `qmgr -c "p s"`

查看队列配置

2. `qstat`

查看作业状态，可用参数`-a,-n,s`

3. `showq`

查看作业状态，为maui软件中的命令

4. `qsub`

提交作业

5. `pbsnodes`

`pbsnodes -a`，查看各节点状态，衍生命令为`pestat`

5. `qdel`

删除作业，`qdel + 作业号`
