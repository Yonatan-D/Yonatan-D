## Redis: 查看占用内存

```bash
redis-cli
> info

找到
# Memory
used_memory:705656
used_memory_human:689.12K

计算得出：705656/1024/1024 = 0.6M,当然直接读_human就好了
```

## Redis: `<jemalloc>: Unsupported #system page size`

背景：在uos上运行项目集成镜像，内置redis服务启不来。这个镜像的redis是在64位树莓派上编译的，系统是Raspbian（基于debian）

原因：在arm64环境下，centos的pagesize是64k，ubuntu的pagesize是4k，一般来说64k下编译的镜像是可以在小于或者等于64k的环境下运行的，但如果是在4k下编译的镜像，那么是不能在pagesize大于4k的环境下运行的。这个是jemalloc造成的。如果在编译构建镜像时使用libc就不会有这种问题。

在构建的Dockerfile的RUN make xxxx指令前加上ENV USE_JEMALLOC no即可使得编译redis时不使用jemalloc。

当然，jemalloc在某些场景下性能要优于libc，这个在选择用何种方式编译redis时，需要考虑清楚。

```bash
# 查看内存页面大小
getconf PAGESIZE

uos的是65536，是64k，而我们这边环境都是4k
```

解决思路：

1. 在arm64 centos机器中重新编译redis可执行文件 ~~（或者修改pagesize为64k）~~

2. 将编译好的文件复制到容器中
3. 重新commit成一个新镜像发布

> 阅读：
> 
> [大内存时代——为什么PageSize仍不建议选择16KB或64KB？其实我们有更好的选择](https://zhuanlan.zhihu.com/p/269906327)
