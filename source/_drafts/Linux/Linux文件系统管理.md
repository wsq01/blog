



# mount命令：挂载Linux系统外的文件
所有的硬件设备必须挂载之后才能使用，只不过，有些硬件设备（比如硬盘分区）在每次系统启动时会自动挂载，而有些（比如 U 盘、光盘）则需要手动进行挂载。

我们可以对挂载的含义进行引申，挂载指的是将硬件设备的文件系统和 Linux 系统中的文件系统，通过指定目录（作为挂载点）进行关联。而要将文件系统挂载到 Linux 系统上，就需要使用`mount`挂载命令。

`mount`命令的常用格式有以下几种：
```
[root@localhost ~]# mount [-l]
```
单纯使用 mount 命令，会显示出系统中已挂载的设备信息，使用 -l 选项，会额外显示出卷标名称。
```
[root@localhost ~]# mount -a
```
-a 选项的含义是自动检查 /etc/fstab 文件中有无疏漏被挂载的设备文件，如果有，则进行自动挂载操作。`/etc/fstab`文件是自动挂载文件，系统开机时会主动读取`/etc/fstab`这个文件中的内容，根据该文件的配置，系统会自动挂载指定设备。
```
[root@localhost ~]# mount [-t 系统类型] [-L 卷标名] [-o 特殊选项] [-n] 设备文件名 挂载点
```
各选项的含义分别是：
-t 系统类型：指定欲挂载的文件系统类型。Linux 常见的支持类型有 EXT2、EXT3、EXT4、iso9660（光盘格式）、vfat、reiserfs 等。如果不指定具体类型，挂载时 Linux 会自动检测。
-L 卷标名：除了使用设备文件名（例如 /dev/hdc6）之外，还可以利用文件系统的卷标名称进行挂载。
-n：在默认情况下，系统会将实际挂载的情况实时写入 /etc/mtab 文件中，但在某些场景下（例如单人维护模式），为了避免出现问题，会刻意不写入，此时就需要使用这个选项；
-o 特殊选项：可以指定挂载的额外选项，比如读写权限、同步/异步等，如果不指定，则使用默认值（defaults）。具体的特殊选项参见表 1；

表 1 mount 命令选项及功能
选项	功能
rw/ro	是否对挂载的文件系统拥有读写权限，rw 为默认值，表示拥有读写权限；ro 表示只读权限。
async/sync	此文件系统是否使用同步写入（sync）或异步（async）的内存机制，默认为异步 async。
dev/nodev	是否允许从该文件系统的 block 文件中提取数据，为了保证数据安装，默认是 nodev。
auto/noauto	是否允许此文件系统被以 mount -a 的方式进行自动挂载，默认是 auto。
suid/nosuid	设定文件系统是否拥有 SetUID 和 SetGID 权限，默认是拥有。
exec/noexec	设定在文件系统中是否允许执行可执行文件，默认是允许。
user/nouser	设定此文件系统是否允许让普通用户使用 mount 执行实现挂载，默认是不允许（nouser），仅有 root 可以。
defaults	定义默认值，相当于 rw、suid、dev、exec、auto、nouser、async 这 7 个选项。
remount	重新挂载已挂载的文件系统，一般用于指定修改特殊权限。
## 示例
### 1.
```
[root@localhost ~]# mount
#查看系统中已经挂载的文件系统，注意有虚拟文件系统
/dev/sda3 on / type ext4 (rw)  <--含义是，将 /dev/sda3 分区挂载到了 / 目录上，文件系统是 ext4，具有读写权限
proc on /proc type proc (rw)
sysfe on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw, gid=5, mode=620)
tmpfs on /dev/shm type tmpfs (rw)
/dev/sda1 on /boot type ext4 (rw)
none on /proc/sys/fe/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfe/rpc_pipefs type rpc_pipefs (rw)
```
### 2.
修改特殊权限。通过例 1 我们查看到，/boot 分区已经被挂载了，而且采用的是 defaults 选项。这里我们重新挂载分区，并采用 noexec 权限禁止执行文件执行，看看会出现什么情况（注意不要用 / 分区做实验，否则系统命令也就不能执行了。
```
[root@localhost ~]# mount -o remount noexec /boot
#重新挂载 /boot 分区，并使用 noexec 权限
[root@localhost sh]# cd /boot
#写一个 shell 脚本，看是否会运行
[root@localhost boot]#vi hello.sh
#!/bin/bash
echo "hello!!"
[root@localhost boot]# chmod 755 hello.sh
[root@localhost boot]# ./hello.sh
-bash:./hello.sh:权限不够
#虽然赋予了hello.sh执行权限，但是仍然无法执行
[root@localhost boot]# mount -o remount exec /boot
#记得改回来，否则会影响系统启动
```
对于特殊选项的修改，除非特殊场景下需要，否则不建议大家随意修改，非常容易造成系统出现问题，而且还找不到问题的根源。
### 3.挂载分区
```
[root@localhost ~]# mkdir /mnt/disk1
#建立挂载点目录
[root@localhost ~]# mount /dev/sdb1 /mnt/disk1
#挂载分区
```
`/dev/sdb1`分区还没有被划分。我们在这里只看看挂载分区的方式，非常简单，甚至不需要使用`-ext4`命令指定文件系统，因为系统可以自动检测。

其实，硬盘分区（设备）挂载和卸载（使用`umount 命令）的概念源自 UNIX，UNIX 系统一般是作为服务器使用的，系统安全非常重要，特别是在网络上，最简单有效的方法就是“不使用的硬盘分区（设备）不挂载”，因为没有挂载的硬盘分区是无法访问的，这样系统也就更安全了。

另外，这样也可以减少挂载的硬盘分区数量，相应地，也就可以减少系统维护文件的规模，当然也就减少了系统的开销，即提高了系统的效率。

# umount命令：卸载文件系统
# fsck命令：检测和修复文件系统
# dumpe2fs命令：查看文件系统信息
# fdisk命令：给硬盘分区
# parted命令用法：创建分区
# mkfs命令详解:格式化分区（为分区写入文件系统）
