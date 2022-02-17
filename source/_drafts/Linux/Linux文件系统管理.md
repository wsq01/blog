

# df用法：查看文件系统硬盘使用情况
`df`命令，用于显示 Linux 系统中各文件系统的硬盘使用情况，包括文件系统所在硬盘分区的总容量、已使用的容量、剩余容量等。

与整个文件系统有关的数据，都保存在 Super block（超级块）中，而`df`命令主要读取的数据几乎都针对的是整个文件系统，所以`df`命令主要是从各文件系统的 Super block 中读取数据。
```
[root@localhost ~]# df [选项] [目录或文件名]
```
`df`命令常用选项及作用：

| 选项	| 作用 |
| :--: | :--: |
| -a	|  显示所有文件系统信息，包括系统特有的 /proc、/sysfs 等文件系统； |
| -m	|  以 MB 为单位显示容量； |
| -k	|  以 KB 为单位显示容量，默认以 KB 为单位； |
| -h	|  使用人们习惯的 KB、MB 或 GB 等单位自行显示容量； |
| -T	|  显示该分区的文件系统名称； |
| -i	|  不用硬盘容量显示，而是以含有 inode 的数量来显示。 |

```
[root@localhost ~]# df
Filesystem      1K-blocks      Used Available Use% Mounted on
/dev/hdc2         9920624   3823112   5585444  41% /
/dev/hdc3         4956316    141376   4559108   4% /home
/dev/hdc1          101086     11126     84741  12% /boot
tmpfs              371332         0    371332   0% /dev/shm
```
不使用任何选项的`df`命令，默认会将系统内所有的文件系统信息，以 KB 为单位显示出来。

由`df`命令显示出的各列信息的含义分别是：
* `Filesystem`：表示该文件系统位于哪个分区，因此该列显示的是设备名称；
* `1K-blocks`：此列表示文件系统的总大小，默认以 KB 为单位；
* `Used`：表示用掉的硬盘空间大小；
* `Available`：表示剩余的硬盘空间大小；
* `Use%`：硬盘空间使用率。如果使用率高达 90% 以上，就需要额外注意，因为容量不足，会严重影响系统的正常运行；
* `Mounted on`：文件系统的挂载点，也就是硬盘挂载的目录位置。

```
[root@localhost ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/hdc2             9.5G  3.7G  5.4G  41% /
/dev/hdc3             4.8G  139M  4.4G   4% /home
/dev/hdc1              99M   11M   83M  12% /boot
tmpfs                 363M     0  363M   0% /dev/shm
```
```
[root@localhost ~]# df -h /etc
Filesystem            Size  Used Avail Use% Mounted on
/dev/hdc2             9.5G  3.7G  5.4G  41% /
```
这里在`df`命令后添加了目录名，在这种情况下，df 命令会自动分析该目录所在的分区，并将所在分区的有关信息显示出来。由此，我们就可以知道，该目录下还可以使用多少容量。
```
[root@localhost ~]# df -aT
Filesystem    Type 1K-blocks    Used Available Use% Mounted on
/dev/hdc2     ext3   9920624 3823112   5585444  41% /
proc          proc         0       0         0   -  /proc
sysfs        sysfs         0       0         0   -  /sys
devpts      devpts         0       0         0   -  /dev/pts
/dev/hdc3     ext3   4956316  141376   4559108   4% /home
/dev/hdc1     ext3    101086   11126     84741  12% /boot
tmpfs        tmpfs    371332       0    371332   0% /dev/shm
none   binfmt_misc         0       0         0   -  /proc/sys/fs/binfmt_misc
sunrpc  rpc_pipefs         0       0         0   -  /var/lib/nfs/rpc_pipefs
```
注意，使用`-a`选项，会将很多特殊的文件系统显示出来，这些文件系统包含的大多是系统数据，存在于内存中，不会占用硬盘空间，因此你会看到，它们所占据的硬盘总容量为 0。
# du命令：统计目录或文件所占磁盘空间大小
`du`是统计目录或文件所占磁盘空间大小的命令。

需要注意的是，使用`ls -r`命令是可以看到文件的大小的。但是大家会发现，在使用`ls -r`命令査看目录大小时，目录的大小多数是 4KB，这是因为目录下的子目录名和子文件名是保存到父目录的`block`（默认大小为 4KB）中的，如果父目录下的子目录和子文件并不多，一个`block`就能放下，那么这个父目录就只占用了一个`block`大小。

但是我们在统计目录时，不是想看父目录下的子目录名和子文件名到底占用了多少空间，而是想看父目录下的子目录和子文件的总磁盘占用量大小，这时就需要使用`du`命令才能统计目录的真正磁盘占用量大小。
```
[root@localhost ~]# du [选项] [目录或文件名]
```
选项：
* `-a`：显示每个子文件的磁盘占用量。默认只统计子目录的磁盘占用量
* `-h`：使用习惯单位显示磁盘占用量，如 KB、MB 或 GB 等；
* `-s`：统计总磁盘占用量，而不列出子目录和子文件的磁盘占用量

```
[root@localhost ~]# du
#统计当前目录的总磁盘占用量大小，同时会统计当前目录下所有子目录的磁盘占用量大小，不统计子文件
#磁盘占用量的大小。默认单位为KB
20 ./.gnupg
#统计每个子目录的大小
24 ./yum.bak
8 ./dtest
28 ./sh
188
#统计当前目录总大小
```
```
[root@localhost ~]# du -a
#统计当前目录的总大小，同时会统计当前目录下所有子文件和子目录磁盘占用量的大小。默认单位为 KB

4 ./.bashjogout
36 ./install.log
4 ./.bash_profile
4 ./.cshrc
…省略部分输出…
188
```
```
[root@localhost ~]# du -sh
#只统计磁盘占用量总的大小，同时使用习惯单位显示
188K .
```
## du命令和df命令的区别
使用`du`命令和`df`命令去统计分区的使用情况时，得到的数据是不一样的。那是因为`df`命令是从文件系统的角度考虑的，通过文件系统中未分配的空间来确定文件系统中已经分配的空间大小。也就是说，在使用`df`命令统计分区时，不仅要考虑文件占用的空间，还要统计被命令或程序占用的空间（最常见的就是文件已经删除，但是程序并没有释放空间）。

而`du`命令是面向文件的，只会计算文件或目录占用的磁盘空间。也就是说，`df`命令统计的分区更准确，是真正的空闲空间。
# mount命令：挂载Linux系统外的文件
所有的硬件设备必须挂载之后才能使用，只不过，有些硬件设备（比如硬盘分区）在每次系统启动时会自动挂载，而有些（比如 U 盘、光盘）则需要手动进行挂载。

挂载指的是将硬件设备的文件系统和 Linux 系统中的文件系统，通过指定目录（作为挂载点）进行关联。而要将文件系统挂载到 Linux 系统上，就需要使用`mount`挂载命令。

`mount`命令的常用格式有以下几种：
```
[root@localhost ~]# mount [-l]
```
单纯使用`mount`命令，会显示出系统中已挂载的设备信息，使用`-l`选项，会额外显示出卷标名称。
```
[root@localhost ~]# mount -a
```
`-a`选项的含义是自动检查`/etc/fstab 文件中有无疏漏被挂载的设备文件，如果有，则进行自动挂载操作。`/etc/fstab`文件是自动挂载文件，系统开机时会主动读取`/etc/fstab`这个文件中的内容，根据该文件的配置，系统会自动挂载指定设备。
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
