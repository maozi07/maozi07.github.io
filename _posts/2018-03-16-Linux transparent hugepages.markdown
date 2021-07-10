---
layout: post
---
### 关于THP(transparent hugepages)

Transparent Huge Pages 在centos6中默认启动。内核会尝试申请大页（2M pages普通一般为64K），来减小TLB压力；启用THP在某些情况下可提高10%性能。

### 在Hadoop环境中的问题

在Hadoop的环境中，如果检测到CPU system占用超过30%，个人经验一般情况下很容易CPU system达到100%，一般来说就是由于启用THP造成的。

### centos7下禁用THP

- 禁用 tuned 服务

```
1.确保tuned服务已经启动
systemctl start tuned
2.关闭tuned服务
tuned-adm off
3.确保没有启动的profiles
tuned-adm list
输出应该如下：
No current active profile
4.关闭tuned服务并禁止开机启动
systemctl stop tuned
systemctl disable tuned
```

- 禁用THP

```
1.查看THP状态
# cat /sys/kernel/mm/redhat_transparent_hugepage/enabled
# cat /sys/kernel/mm/redhat_transparent_hugepage/defrag
输出说明：
always   -  always use THP
never    -  disable THP
2.暂时禁用THP，重启失效
/etc/rc.local中添加下面两行
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
3.永久禁用THP
/etc/rc.local中添加下面两行
# echo never > /sys/kernel/mm/transparent_hugepage/enabled
# echo never > /sys/kernel/mm/transparent_hugepage/defrag

# chmod +x /etc/rc.local

修改/etc/sysconfig/grub文件，在GRUB_CMDLINE_LINUX中添加transparent_hugepage=never，修改后例子：
GRUB_CMDLINE_LINUX="rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap vconsole.font=latarcyrheb-sun16 vconsole.keymap=us transparent_hugepage=never"

重新生成grub.cfg
# grub2-mkconfig -o /boot/grub2/grub.cfg

```

### 查看THP状态等

- 查看系统上THP使用状况

```
# grep AnonHugePages /proc/meminfo
AnonHugePages:    632832 kB
```

- 查看每个进程使用THP使用状况

```
# grep -e AnonHugePages  /proc/*/smaps | awk  '{ if($2>4) print $0} ' |  awk -F "/"  '{print $0; system("ps -fp " $3)} '
/proc/4110/smaps:AnonHugePages:     12288 kB
UID        PID  PPID  C STIME TTY          TIME CMD
yarn      4110  3030  2 Mar15 ?        00:34:33 /usr/java/jdk1.8.0//bin/java -Dproc_nodemanager -Xmx1000m -Djava.net.preferIPv4Stack=true -

```

- 确定HugePages是否已禁用

```
禁用:

如果HugePages_Total 值为"0" 代表HugePages已经禁用.
# grep -i HugePages_Total /proc/meminfo
HugePages_Total:       0

如果/proc/sys/vm/nr_hugepages 或者vm.nr_hugepages sysctl 为"0" 同样代表HugePages已禁用:
# cat /proc/sys/vm/nr_hugepages
0
# sysctl vm.nr_hugepages
vm.nr_hugepages = 0

启用:

如果HugePages_Total的值大于"0", 意味着HugePages已启用:
# grep -i HugePages_Total /proc/meminfo
HugePages_Total:       1024

如果/proc/sys/vm/nr_hugepages 或者vm.nr_hugepages sysctl 大于"0" 同样代表HugePages已禁用:
# cat /proc/sys/vm/nr_hugepages
1024
# sysctl vm.nr_hugepages
vm.nr_hugepages = 1024
```

### 参考资料
> 1. [cloudera Disabling Transparent Hugepages (THP)](https://www.cloudera.com/documentation/enterprise/latest/topics/cdh_admin_performance.html#cdh_performance__section_nt5_sdf_jq)
> 2. [How to use, monitor, and disable transparent hugepages in Red Hat Enterprise Linux 6 and 7?](https://access.redhat.com/solutions/46111)
