---
layout: post
---
#### ramfs与tmpfs都可挂载内存为硬盘上的目录，通俗的就是内存当作磁盘使用

### ramfs
1. ramfs制使用物理内存，挂载时不可限制大小
2. 写入的文件都会保存在内存中，内存耗尽时Linux系统会触发OOM killer（Out Of Memory killer）
3. 手动删除文件可释放空间

因此使用时要特别小心以防触发OOM killer，OOM killer触发时系统会很卡接近死机，停止向内存中写数据系统会慢慢恢复

可使用下面命令将内存挂在为硬盘上一个文件使用：

```
mkdir /ramtest
mount -t ramfs ramfs /ramtest
```

### tmpfs
1. tmpfs使用物理内存与交换空间，默认50%内存＋交换空间大小，可指定文件系统大小
2. 文件写入超出指定大小会无法写入，与向一个磁盘写数据直到超出磁盘容量情况相同
3. 手动删除文件可释放空间

例子：

如有200M大小tmpfs，先写入test1 150M后写入test2 100M 最后test2只有部分写入，test2实际文件大小50M文件被部分截断

可使用下面命令挂载tmpfs，_-o size=200m_ 参数用于指定文件最大容量,大小可使用k,m,g指定

```
mount -t tmpfs tmpfs /ramtest -o size=200m
```

### 参考资料

> [https://www.kernel.org/doc/Documentation/filesystems/ramfs-rootfs-initramfs.txt](https://www.kernel.org/doc/Documentation/filesystems/ramfs-rootfs-initramfs.txt)
