
# Linux文件系统详解
硬盘是用来存储数据的，可以将其想象成柜子，只不过柜子是用来存储衣物的。新买来的硬盘，通常要对其进行分区并格式化，分区就如同把一个大柜按照要求分割成几个小柜子（组合衣柜)；格式化就好比在每个小柜子中打入隔断，决定每个隔断的大小和位置，然后在柜门上贴上标签，标签中写清楚每件衣服保存的隔断的位置和这件衣服的一些特性（比如衣服是谁的，衣服的颜色、大小等）。

很多人认为，对硬盘进行格式化，只是清除了硬盘中的数据，其实不然，格式化过程中还向硬盘中写入了文件系统。因为不同的操作系统，管理系统中文件的方式也不尽相同（给文件设定的属性和权限也不完全一样），因此，为了使硬盘有效存放当前系统中的文件数据，就需要将硬盘进行格式化，令其使用和操作系统一样（或接近）的文件系统格式。

各操作系统使用的文件系统并不相同，例如，Windows 98 以前的微软操作系统使用 FAT（FAT16）文件系统，Windows 2000 以后的版本使用 NTFS 文件系统，而 Linux 的正统文件系统是 Ext2。

既然格式化的真实目的是为了写入文件系统，那么，Linux 中的文件系统到底是什么，又是如何运作的呢？

在 CentOS 6.3 系统中，默认的文件系统是 Ext4，它是 Ext3（Ext2）文件系统的升级版，在性能、伸缩性和可靠性方面进行了大量改进，变化可以说是翻天覆地的，比如：
* 向下兼容 Ext3；
* 最大 1EB 文件系统和 16TB 文件；
* 无限数量子目录；
* Extents 连续数据块概念；
* 多块分配、延迟分配、持久预分配；
* 快速 FSCK、日志校验、无日志模式、在线碎片整理、inode 增强、默认启用 barrier 等；

不同的文件系统，其运作模式和操作系统的文件数据有关。拿 Linux 操作系统中的文件为例，文件数据不仅包括文件中的内容，还包含非常多的文件属性，例如文件的 rwx 权限以及文件所有者、所属组、创建时间等。

通常情况下，文件系统会将文件的实际内容和属性分开存放：
文件的属性保存在 inode 中（i 节点）中，每个 inode 都有自己的编号。每个文件各占用一个 inode。不仅如此，inode 中还记录着文件数据所在 block 块的编号；
文件的实际内容保存在 block 中（数据块），类似衣柜的隔断，用来真正保存衣物。每个 block 都有属于自己的编号。当文件太大时，可能会占用多个 block 块。
另外，还有一个 super block（超级块）用于记录整个文件系统的整体信息，包括 inode 和 block 的总量、已经使用量和剩余量，以及文件系统的格式和相关信息等。
由此我们可以推断出，只要能找到文件 inode 所在的位置，自然就能知道这个文件存放数据的 block 号，从而找到文件的实际数据。整个过程如图 1 所示。

图 1 中，文件系统先格式化出 inode 和 block 块，假设某文件的权限和属性信息存放到 inode 4 号位置，这个 inode 记录了实际存储文件数据的 block 号有 4 个，分别为 2、7、13、15，由此，操作系统就能快速地找到文件数据的存储位置。

这种管理文件的系统称为索引式文件系统，Linux 文件系统（Ext 系列）就属于索引式文件系统。

注意，inode 节点并不存储文件的文件名，因为文件名是文件所在目录的数据，所以会保存在上一级目录的 block 块中。前面章节在讲权限命令的时候说过，要对文件的上一级目录拥有 w 权限，才能删除目录中的文件，就是因为文件名是保存在目录的 block 中的。
## Linux支持的常见文件系统
Linux 系统能够支持的文件系统非常多，除 Linux 默认文件系统 Ext2、Ext3 和 Ext4 之外，还能支持 fat16、fat32、NTFS（需要重新编译内核）等 Windows 文件系统。也就是说，Linux 可以通过挂载的方式使用 Windows 文件系统中的数据。Linux 所能够支持的文件系统在 "/usr/src/kemels/当前系统版本/fs" 目录中（需要在安装时选择），该目录中的每个子目录都是一个可以识别的文件系统。

| 文件系统	| 描 述 |
| :--: | :--: |
| Ext      | Linux 中最早的文件系统，由于在性能和兼容性上具有很多缺陷，现在已经很少使用|
| Ext2	   | 是 Ext 文件系统的升级版本，Red Hat Linux 7.2 版本以前的系统默认都是 Ext2 文件系统。于 1993 年发布，支持最大 16TB 的分区和最大 2TB 的文件|
| Ext3	   | 是 Ext2 文件系统的升级版本，最大的区别就是带日志功能，以便在系统突然停止时提高文件系统的可靠性。支持最大 16TB 的分区和最大 2TB 的文件|
| Ext4	   | 是 Ext3 文件系统的升级版。Ext4 在性能、伸缩性和可靠性方面进行了大量改进。Ext4 的变化可以说是翻天覆地的，比如向下兼容 Ext3、最大 1EB 文件系统和 16TB 文件、无限数量子目录、Extents 连续数据块 概念、多块分配、延迟分配、持久预分配、快速 FSCK、日志校验、无日志模式、在线碎片整理、inode 增强、默认启用 barrier 等。它是 CentOS 6.3 的默认文件系统 |
| swap	   | swap 是 Linux 中用于交换分区的文件系统（类似于 Windows 中的虚拟内存)，当内存不够用时，使用交换分区暂时替代内存。一般大小为内存的 2 倍，但是不要超过 2GB。它是 Linux 的必需分区 |
| NFS	     | NFS 是网络文件系统（Network File System）的缩写，是用来实现不同主机之间文件共享的一种网络服务，本地主机可以通过挂载的方式使用远程共享的资源 |
| iso9660	 | 光盘的标准文件系统。Linux 要想使用光盘，必须支持 iso9660 文件系统 |
| fat      | 就是 Windows 下的 fatl6 文件系统，在 Linux 中识别为 fat |
| vfat	   | 就是 Windows 下的 fat32 文件系统，在 Linux 中识别为 vfat。支持最大 32GB 的分区和最大 4GB 的文件 |
| NTFS	   | 就是 Windows 下的 NTFS 文件系统，不过 Linux 默认是不能识别 NTFS 文件系统的，如果需要识别，则需要重新编译内核才能支持。它比 fat32 文件系统更加安全，速度更快，支持最大 2TB 的分区和最大 64GB 的文件 |
| ufs	     | Sun 公司的操作系统 Solaris 和 SunOS 所采用的文件系统 |
| proc	   | Linux 中基于内存的虚拟文件系统，用来管理内存存储目录 /proc |
| sysfs	   | 和 proc —样，也是基于内存的虚拟文件系统，用来管理内存存储目录 /sysfs |
| tmpfs	   | 也是一种基于内存的虚拟文件系统，不过也可以使用 swap 交换分区 |

# Linux系统是如何识别硬盘设备和硬盘分区的
Linux 系统初始化时，会根据 MBR 来识别硬盘设备。

MBR，全称`Master Boot Record`，可译为硬盘主引导记录，占据硬盘 0 磁道的第一个扇区。MBR 中，包括用来载入操作系统的可执行代码，实际上，此可执行代码就是 MBR 中前 446 个字节的`boot loader`程序（引导加载程序），而在`boot loader`程序之后的 64 个（16×4）字节的空间，就是存储的分区表（`Partition table`）相关信息。如图 1 所示。


在分区表中，主要存储的值息包括分区号、分区的起始磁柱和分区的磁柱数量。所以 Linux 操作系统在初始化时就可以根据分区表中以上 3 种信息来识别硬盘设备。其中，常见的分区号如下：
* `0x5`（或`0xf`）：可扩展分区（Extended partition）。
* `0x82`：Linux 交换区（Swap partition）。
* `0x83`：普通 Linux 分区（Linux partition）。
* `0x8e`：Linux 逻辑卷管理分区（Linux LVM partition）。
* `0xfd`：Linux 的 RAID 分区（Linux RAID auto partition）。

由于 MBR 留给分区表的磁盘空间只有 64 个字节，而每个分区表的大小为 16 个字节，所以在一个硬盘上最多可以划分出 4 个主分区。如果想要在一个硬盘上划分出 4 个以上的分区时，可以通过在硬盘上先划分出一个可扩展分区的方法来增加额外的分区。

不过，在 Linux 的 Kernel 中所支持的分区数量有如下限制：
一个 IDE 的硬盘最多可以使用 63 个分区；
一个 SCSI 的硬盘最多可以使用 15 个分区。

为什么要将一个硬盘划分成多个分区，而不是直接使用整个硬盘呢？其主要有如下原因：
1. 方便管理和控制
首先，可以将系统中的数据（也包括程序）按不同的应用分成几类，之后将这些不同类型的数据分别存放在不同的磁盘分区中。由于在每个分区上存放的都是类似的数据或程序，这样管理和维护就简单多了。
2. 提高系统的效率
给硬盘分区，可以直接缩短系统读写磁盘时磁头移动的距离，也就是说，缩小了磁头搜寻的范围；反之，如果不使用分区，每次在硬盘上搜寻信息时可能要搜寻整个硬盘，所以速度会很慢。另外，硬盘分区也可以减轻碎片（文件不连续存放）所造成的系统效率下降的问题。
3. 使用磁盘配额的功能限制用户使用的磁盘量
由于限制用户使用磁盘配额的功能，只能在分区一级上使用，所以，为了限制用户使用磁盘的总量，防止用户浪费磁盘空间（甚至将磁盘空间耗光），最好将磁盘先分区，然后在分配给一般用户。
4. 便于备份和恢复
硬盘分区后，就可以只对所需的分区进行备份和恢复操作，这样的话，备份和恢复的数据量会大大地下降，而且也更简单和方便。

# df用法：查看文件系统硬盘使用情况
`df`命令，用于显示 Linux 系统中各文件系统的硬盘使用情况，包括文件系统所在硬盘分区的总容量、已使用的容量、剩余容量等。

与整个文件系统有关的数据，都保存在`Super block`（超级块）中，而`df`命令主要读取的数据几乎都针对的是整个文件系统，所以`df`命令主要是从各文件系统的`Super block`中读取数据。
```bash
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

```bash
[root@localhost ~]# df
Filesystem      1K-blocks      Used Available Use% Mounted on
/dev/hdc2         9920624   3823112   5585444  41% /
/dev/hdc3         4956316    141376   4559108   4% /home
/dev/hdc1          101086     11126     84741  12% /boot
tmpfs              371332         0    371332   0% /dev/shm
```
不使用任何选项的`df`命令，默认会将系统内所有的文件系统信息，以`KB`为单位显示出来。

由`df`命令显示出的各列信息的含义分别是：
* `Filesystem`：表示该文件系统位于哪个分区，因此该列显示的是设备名称；
* `1K-blocks`：此列表示文件系统的总大小，默认以`KB`为单位；
* `Used`：表示用掉的硬盘空间大小；
* `Available`：表示剩余的硬盘空间大小；
* `Use%`：硬盘空间使用率。如果使用率高达 90% 以上，就需要额外注意，因为容量不足，会严重影响系统的正常运行；
* `Mounted on`：文件系统的挂载点，也就是硬盘挂载的目录位置。

```bash
[root@localhost ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/hdc2             9.5G  3.7G  5.4G  41% /
/dev/hdc3             4.8G  139M  4.4G   4% /home
/dev/hdc1              99M   11M   83M  12% /boot
tmpfs                 363M     0  363M   0% /dev/shm
```
```bash
[root@localhost ~]# df -h /etc
Filesystem            Size  Used Avail Use% Mounted on
/dev/hdc2             9.5G  3.7G  5.4G  41% /
```
这里在`df`命令后添加了目录名，在这种情况下，`df`命令会自动分析该目录所在的分区，并将所在分区的有关信息显示出来。由此，我们就可以知道，该目录下还可以使用多少容量。
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

需要注意的是，使用`ls -r`命令是可以看到文件的大小的。但是大家会发现，在使用`ls -r`命令査看目录大小时，目录的大小多数是`4KB`，这是因为目录下的子目录名和子文件名是保存到父目录的`block`（默认大小为 4KB）中的，如果父目录下的子目录和子文件并不多，一个`block`就能放下，那么这个父目录就只占用了一个`block`大小。

但是我们在统计目录时，不是想看父目录下的子目录名和子文件名到底占用了多少空间，而是想看父目录下的子目录和子文件的总磁盘占用量大小，这时就需要使用`du`命令才能统计目录的真正磁盘占用量大小。
```bash
[root@localhost ~]# du [选项] [目录或文件名]
```
选项：
* `-a`：显示每个子文件的磁盘占用量。默认只统计子目录的磁盘占用量
* `-h`：使用习惯单位显示磁盘占用量，如 KB、MB 或 GB 等；
* `-s`：统计总磁盘占用量，而不列出子目录和子文件的磁盘占用量

```bash
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
```bash
[root@localhost ~]# du -a
#统计当前目录的总大小，同时会统计当前目录下所有子文件和子目录磁盘占用量的大小。默认单位为 KB

4 ./.bashjogout
36 ./install.log
4 ./.bash_profile
4 ./.cshrc
…省略部分输出…
188
```
```bash
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
```bash
[root@localhost ~]# mount [-l]
```
单纯使用`mount`命令，会显示出系统中已挂载的设备信息，使用`-l`选项，会额外显示出卷标名称。
```bash
[root@localhost ~]# mount -a
```
`-a`选项的含义是自动检查`/etc/fstab`文件中有无疏漏被挂载的设备文件，如果有，则进行自动挂载操作。`/etc/fstab`文件是自动挂载文件，系统开机时会主动读取`/etc/fstab`这个文件中的内容，根据该文件的配置，系统会自动挂载指定设备。
```bash
[root@localhost ~]# mount [-t 系统类型] [-L 卷标名] [-o 特殊选项] [-n] 设备文件名 挂载点
```
各选项的含义分别是：
* `-t`系统类型：指定欲挂载的文件系统类型。Linux 常见的支持类型有 EXT2、EXT3、EXT4、iso9660（光盘格式）、vfat、reiserfs 等。如果不指定具体类型，挂载时 Linux 会自动检测。
* `-L`卷标名：除了使用设备文件名（例如`/dev/hdc6`）之外，还可以利用文件系统的卷标名称进行挂载。
* `-n`：在默认情况下，系统会将实际挂载的情况实时写入`/etc/mtab`文件中，但在某些场景下（例如单人维护模式），为了避免出现问题，会刻意不写入，此时就需要使用这个选项；
* `-o`特殊选项：可以指定挂载的额外选项，比如读写权限、同步/异步等，如果不指定，则使用默认值（`defaults`）。

`mount`命令选项及功能

| 选项	      | 功能 |
| :--: | :--: |
| rw/ro	      | 是否对挂载的文件系统拥有读写权限，rw 为默认值，表示拥有读写权限；ro 表示只读权限。 |
| async/sync	| 此文件系统是否使用同步写入（sync）或异步（async）的内存机制，默认为异步 async。 |
| dev/nodev	  | 是否允许从该文件系统的 block 文件中提取数据，为了保证数据安装，默认是 nodev。 |
| auto/noauto	| 是否允许此文件系统被以 mount -a 的方式进行自动挂载，默认是 auto。 |
| suid/nosuid	| 设定文件系统是否拥有 SetUID 和 SetGID 权限，默认是拥有。 |
| exec/noexec	| 设定在文件系统中是否允许执行可执行文件，默认是允许。 |
| user/nouser	| 设定此文件系统是否允许让普通用户使用 mount 执行实现挂载，默认是不允许（nouser），仅有 root 可以。 |
| defaults	  | 定义默认值，相当于 rw、suid、dev、exec、auto、nouser、async 这 7 个选项。 |
| remount	    | 重新挂载已挂载的文件系统，一般用于指定修改特殊权限。 |

## 示例
### 1.
```bash
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
```bash
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
```bash
[root@localhost ~]# mkdir /mnt/disk1
#建立挂载点目录
[root@localhost ~]# mount /dev/sdb1 /mnt/disk1
#挂载分区
```
`/dev/sdb1`分区还没有被划分。我们在这里只看看挂载分区的方式，非常简单，甚至不需要使用`-ext4`命令指定文件系统，因为系统可以自动检测。

其实，硬盘分区（设备）挂载和卸载（使用`umount`命令）的概念源自 UNIX，UNIX 系统一般是作为服务器使用的，系统安全非常重要，特别是在网络上，最简单有效的方法就是“不使用的硬盘分区（设备）不挂载”，因为没有挂载的硬盘分区是无法访问的，这样系统也就更安全了。

另外，这样也可以减少挂载的硬盘分区数量，相应地，也就可以减少系统维护文件的规模，当然也就减少了系统的开销，即提高了系统的效率。

# Linux挂载光盘（使用mount命令）
在 Windows 中，如果我们想要使用光盘，只需要将光盘放入光驱即可。但在Linux 系统中，将光盘放入光驱后，还需要将光盘中的文件系统手动挂载到 Linux 系统中，还可以使用。

同样，用完光盘后，Windows 系统可以直接弹出光驱并取出光盘，但 Linux 系统不行，必须先卸载才能取出光盘，这确实不如 Windows 方便，不过这也只是一个操作习惯，习惯了就好。

将光盘放入光驱之后，需执行如下挂载命令：
```
[root@localhost ~]# mkdir/mnt/cdrom/
\#建立挂载点
[root@localhost ~]# mount -t iso9660 /dev/cdrom /mnt/cdrom/
\#挂载光盘
```
光盘的文件系统是 iso9660，不过这个文件系统可以省略不写，系统会自动检测。因此，挂在命令也可以写为如下的方式：
```
[root@localhost ~]# mount /dev/cdrom /mnt/cdrom/
\#挂载光盘。两个挂载光盘的命令使用一个就可以了
[root@localhost ~]# mount
\#查看已经挂载的设备
…省略部分输出…
/dev/srO on /mnt/cdrom type iso9660 (ro)
\#光盘已经挂载了，但是挂载的设备文件名是/dev/sr0
```
我们知道，挂载就是把光驱的设备文件和挂载点连接起来。挂载点`/mnt/cdrom`是我们手工建立的空目录，我个人习惯把挂载点建立在 /mnt/ 目录中，因为我们在学习 Linux 的时候是没有`/media/`目录的，大家要是愿意也可以建立`/media/cdrom`作为挂载点，只要是已经建立的空目录都可以作为挂载点。那么`/dev/cdrom`就是光驱的设备文件名，不过注意`/dev/cdrom`只是一个软链接（如同 Windows 系统中的文件快捷方式）。 命令如下：
```
[root@localhost ~]#ll /dev/cdrom
lrwxrwxrwx 1 root root 3 1月31 01:13/dev/cdrom ->sr0
```
`/dev/cdrom`的源文件是`/dev/sr0`。`/dev/sr0`是光驱的真正设备文件名，代表 SCSI 接口或 SATA 接口的光驱，所以刚刚查询挂载时看到的光驱设备文件命令是`/dev/sr0`。也就是说，挂载命令也可以写成这样：
```
[root@localhost ~]# mount /dev/sr0 /mnt/cdrom/
```
其实光驱的真正设备文件名是保存在`/proc/sys/dev/cdrom/info`文件中的，所以可以通过查看这个文件来查询光盘的真正设备文件名：
```
[root@localhost ~]# cat /proc/sys/dev/cdrom/info
CD-ROM information, ld: cdrom.c 3.20 2003/12/17
drive name: sr0
…省略部分输出…
```
# Linux挂载U盘（使用mount命令）
挂载 U 盘和挂载光盘的方式是一样的，只不过光盘的设备文件名是固定的（`/dev/sr0`或`/dev/cdrom`），而 U 盘的设备文件名是在插入 U 盘后系统自动分配的。

因为 U 盘使用的是硬盘的设备文件名，而每台服务器上插入的硬盘数量和分区方式都是不一样的，所以 U 盘的设备号需要单独检测与分配，以免和硬盘的设备文件名产生冲突。

U 盘的设备文件名是系统自动分配的，我们只要查找出来然后挂载可以了。首先把 U 盘插入 Linux 系统中，这里需要注意的是，如果是虚拟机，则需要先把鼠标点入虚拟机再插入 U 盘。

通过使用`fdisk`命令，即可查看到 U 盘的设备文件名，执行命令如下：
```
[root@localhost ~]# fdisk -l
Disk /dev/sda: 21.5GB, 21474836480 bytes
\#系统硬盘
…省略部分输出…
Disk/dev/sdb: 8022 MB, 8022654976 bytes
\#这就是识别的U盘，大小为8GB
94 heads, 14 sectors/track, 11906 cylinders
Units = cylinders of 1316 * 512 = 673792 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000
Device Boot Start End Blocks Id System
/dev/sdb1 1 11907 7834608 b W95 FAT32
\#系统给U盘分配的设备文件名
```
查看到 U 盘的设备文件名，接下来就要创建挂载点了。
```
[root@localhost ~]# mkdir /mnt/usb
```
然后就是挂载了：
```
[root@localhost ~]# mount -t vfat /dev/sdb1 /mnt/usb/
挂载U盘。因为是Windows分区，所以是vfat文件系统格式
[root@localhost ~]# cd /mnt/usb/
\#去挂载点访问U盘数据
[root@localhost usb]# ls
\#输出为乱码
\#之所以出现乱码，是因为编码格式不同
```
之所以出现乱码，是因为 U 盘是 Windows 中保存的数据，而 Windows 中的中文编码格式和 Linux 中的不一致，只需在挂载的时候指定正确的编码格式就可以解决乱码问题，命令如下：
```
[root@localhost ~]# mount -t vfat -o iocharset=utf8 /dev/sdb1 /mnt/usb/
\#挂载U盘，指定中文编码格式为UTF-8
[root@localhost ~]# cd /mnt/usb/
[root@localhost usb]# ls
1111111年度总结及计划表.xls ZsyqlHL7osKSPBoGshZBr6.mp4 协议书
12月21日.doc 恭喜发财(定）.mp4 新年VCR(定).mp4
\#可以正确地查看中文了
```
因为我们的 Linux 在安装时采用的是 UTF-8 编码格式，所以要让 U 盘在挂载时也指定为 UTF-8 编码格式，才能正确显示。
```
[root@localhost ~]# echo $LANG
zh_CN.UTF-8
\#查看一下Linux默认的编码格式
```
注意，Linux 默认是不支持 NTFS 文件系统的，所以默认是不能挂载 NTFS 格式的移动硬盘的。要想让 Linux 支持移动硬盘，主要有三种方法：

重新编译内核，加入 ntfs 模块，然后安装 ntfs 模块即可；
不自己编译内核，而是下载已经编译好的内核，直接安装即可；
安装 NTFS 文件系统的第三方插件，也可以支持 NTFS 文件系统；

# Linux开机自动挂载硬件设备（配置etcfatab文件）
Linux 通过`/etc/fstab`配置文件来实现开机自动挂载某个硬件设备，这个配置文件对所有用户可读，但只有`root`用户有权修改此文件。
```
[root@localhost ~]# vi /etc/fstab
UUID=c2ca6f57-b15c-43ea-bca0-f239083d8bd2 / ext4 defaults 1 1
UUID=0b23d315-33a7-48a4-bd37-9248e5c44345 /boot ext4 defaults 1 2
UUID=4021be19-2751-4dd2-98cc-383368c39edb swap swap defaults 0 0
\#只有这三个是真正的硬盘分区，下面的都是虚拟文件系统或交换分区
tmpfs /dev/shm tmpfs defaults 0 0
devpts /dev/pts devpts gid=5, mode=620 0 0
sysfs /sys sysfe defaults 0 0
proc /proc proc defaults 0 0
```
目前，大家可以忽略`tmpfs、devpts、sysfs`和`proc`这几行，它们分别是与共享内存、终端窗口、设备信息和内核参数相关联的特殊设备。

可以看到，在`fstab`文件中，每行数据都分为了 6 个字段，它们的含义分别是：
* 用来挂载每个文件系统的分区设备文件名或 UUID（用于指代设备名）；
* 挂载点；
* 文件系统的类型；
* 各种挂载参数；
* 指定分区是否被 dump 备份；
* 指定分区是否被 fsck 检测；

## /etc/fstab文件各字段的含义
首先介绍第一个字段，什么是 UUID 呢？UUID 即通用唯一标识符，是一个 128 位比特的数字，可以理解为就是硬盘的 ID，UUID 由系统自动生成和管理。

这个字段在 CentOS 5.5 系统中是写入分区的卷标名或分区设备文件名的，现在变成了硬盘的 UUID。这样做的好处是当硬盘増加了新的分区，或者分区的顺序改变，或者内核升级后，仍然能够保证分区能够正确地加载，而不至于造成启动障碍。

那么，每个分区的 UUID 到底是什么呢？用 dumpe2fs 命令就可以查看到，具体执行命令如下：
```
[root@localhost ~]# dumpe2fs /dev/sdb5
dumpe2fs 1.41.12 (17-May-2010)
Filesystem volume name: test_label
Last mounted on: <not available>
Filesystem UUID: 63f238f0-a715-4821-8ed1-b3d18756a3ef
\#UUID
...省略部分输出...
```
另外，也可以通过查看每个硬盘UUID的链接文件名来确定UUID：
```
[root@localhost ~]# ls -l /dev/disk/by-uuid/
总用量 0
Irwxrwxrwx. 1 root root 10 4 月 11 00:17 0b23d315-33a7-48a4-bd37-9248e5c44345
-> ../../sdal
Irwxrwxrwx. 1 root root 10 4 月 11 00:17 4021 be19-2751 -4dd2-98cc-383368c39edb
-> ../../sda2
Irwxrwxrwx. 1 root root 10 4 月 11 00:17 63f238f0-a715-4821-8ed1-b3d18756a3ef
-> ../../sdb5
Irwxrwxrwx. 1 root root 10 4月 11 00:17 6858b440-ad9e-45cb-b411 -963c5419e0e8
-> ../../sdb6
Irwxrwxrwx. 1 root root 10 4月 11 00:17 c2ca6f57-b15c-43ea-bca0-f239083d8bd2
-> ../../sda3
```
需要强调的是，挂载点一定要是已经建立的空目录。

第三个字段为文件系统名称，CentOS 6.3 的默认文件系统应该是 ext4。

第四个字段是挂载参数，这个参数和`mount`命令的挂载参数一致。

第五个字段表示“指定分区是否被`dump`备份”，0 代表不备份，1 代表备份，2 代表不定期备份。

第六个字段表示“指定分区是否被`fsck`检测”，0 代表不检测，其他数字代表检测的优先级，1 的优先级比 2 高。所以先检测 1 的分区，再检测 2 的分区。一般分区的优先级是 1，其他分区的优先级是 2。

配置 /etc/fatab 文件
能看懂这个文件了吧？我们把 /dev/sdb5 和 /dev/sdb6 两个分区加入 /etc/fstab 文件，执行命令如下：
```
[root@localhost ~]# vi /etc/fstab
UUID=c2ca6f57-b15c-43ea-bca0-t239083d8bd2 ext4 defaults 1 1
UUID=0b23d315-33a7-48a4-bd37-9248e5c44345 I boot ext4 defaults 1 2
UUID=4021be19-2751-4dd2-98cc-383368c39edb swap swap defaults 0 0
tmpfs /dev/shm tmpfs defaults 0 0
devpts /dev/pts devpts gid=5, mode=620 0 0
sysfs /sys sysfs defaults 0 0
proc /proc proc defaults 0 0
/dev/sdb5 /disk5 ext4 defaults 1 2
/dev/sdb6 /disk6 ext4 defaults 1 2
```
可以看到，这里并没有使用分区的 UUID，而是直接写入分区设备文件名，也是可以的。不过，如果不写 UUID，就要注意，在修改了磁盘顺序后，/etc/fstab 文件也要相应的改变。

这里直接使用分区的设备文件名作为此文件的第一个字段，当然也可以写分区的 UUID。只不过 UUID 更加先进，设备文件名稍微简单一点。

至此，分区就建立完成了，接下来只要重新启动，测试一下系统是否可以正常启动就可以了。只要 /etc/fstab 文件修改正确，就不会出现任何问题。
# 修改etcfstab文件出错导致Linux不能启动，该怎么办
如果把`/etc/fstab`文件修改错了，也重启了，系统崩溃启动不了了，那该怎么办？比如：
```
[root@localhost ~]# vi /etc/fstab
UUID=c2ca6f57-b15c-43ea-bca0-f239083d8bd2 ext4 defaults 1 1
UUID=0b23d315-33a7-48a4-bd37-9248e5c44345 boot ext4 defaults 12
UUID=4021be19-2751-4dd2-98cc-383368c39edb swap swap defaults 0 0
tmpfs /dev/shm tmpfs defaults 0 0
devpts /dev/pts devpts gid=5, mode=620 00
sysfs /sys sysfs defaults 0 0
proc /proc proc defaults 0 0
/dev/sdb5 /disk5 ext4 defaults 1 2
/dev/sdb /disk6 ext4 defaults 1 2
\#故意把/dev/sdb6写成了 /dev/sdb
```
我们重新启动系统，真的报错了，如图 1 所示。

先别急，仔细看看，系统提示输入 root 密码，我们输入密码试试，如图 2 所示。


我们又看到了系统提示符，赶快把 /etc/fstab 文件修改回来吧。又报错了，如图 3 所示。

别慌，分析一下原因提示是没有写权限，那么只要把 / 分区重新挂载上读写权限不就可以修改了吗？命令如下：
```
[root@localhost ~]#mount-oremount, rw/
```
再去修改 /etc/fstab 文件，把它改回来就可以正常启动了。


# umount命令：卸载文件系统
前面介绍了如何将光盘和 U 盘挂载在系统中，而在使用完成后，需要先将其与挂载点取消关联，然后才能成功卸载。不过，硬盘分区是否需要卸载，取决于你下次是否还需要使用，一般不对硬盘分区执行卸载操作。

`umount`命令用于卸载已经挂载的硬件设备：
```
[root@localhost ~]# umount 设备文件名或挂载点
```
注意，卸载命令后面既可以加设备文件名，也可以加挂载点，不过只能二选一，比如：
```
[root@localhost ~]# umount /mnt/usb
#卸载U盘
[root@localhost ~]# umount /mnt/cdrom
#卸载光盘
[root@localhost ~]# umount /dev/sr0
#命令加设备文件名同样是可以卸载的
```

如果加了两个（如下所示），从理论上分析，会对光驱卸载两次，当然，卸载第二次的时候就会报错。
```
[root@localhost ~]# mount /dev/sr0 /mnt/cdrom/
```

另外，我们在卸载时有可能会出现以下情况：
```
[root@localhost ~]# cd /mnt/cdrom/
#进入光盘挂载点
[root@localhost cdrom]# umount /mnt/cdrom/
umount: /mnt/cdrom: device is busy.
#报错，设备正忙
```
这种报错是因为我们已经进入了挂载点，因此，如果要卸载某硬件设备，在执行`umount`命令之前，用户须退出挂载目录。

卸载硬件设备成功与否，除了执行`umount`命令不报错之外，还可以使用`df`命令或`mount -l`来查看目标设备是否还挂载在系统中。
# fsck命令：检测和修复文件系统
计算机难免会由于某些系统因素或人为误操作（突然断电）出现系统异常，这种情况下非常容易造成文件系统的崩溃，严重时甚至会造成硬件损坏。这也是我们一直在强调的“服务器一定要先关闭服务再进行重启”的原因所在。

那么，如果真出现了文件系统损坏的情况，有办法修复吗？可以的，对于一些小问题，使用`fsck`命令就可以很好地解决。

`fsck`命令用于检查文件系统并尝试修复出现的错误。
```
[root@localhost ~]# fsck [选项] 分区设备文件名
```
表 1 罗列出了该命令常用的选项以及各自的功能。

表 1 fsck命令常用选项及其功能
选项	功能
-a	自动修复文件系统，没有任何提示信息。
-r	采取互动的修复模式，在修改文件前会进行询问，让用户得以确认并决定处理方式。
-A（大写）	按照 /etc/fstab 配置文件的内容，检查文件内罗列的全部文件系统。
-t 文件系统类型	指定要检查的文件系统类型。
-C（大写）	显示检查分区的进度条。
-f	强制检测，一般 fsck 命令如果没有发现分区有问题，则是不会检测的。如果强制检测，那么不管是否发现问题，都会检测。
-y	自动修复，和 -a 作用一致，不过有些文件系统只支持 -y。
此命令通常只有身为 root 用户且文件系统出现问题时才会使用，否则，在正常状况下使用 fsck 命令，很可能损坏系统。另外，如果你怀疑已经格式化成功的硬盘有问题，也可以使用此命令来进行检查。
使用 fsck 检查并修复文件系统是存在风险的，特别是当硬盘错误非常严重的时候，因此，当一个受损文件系统中包含了非常有价值的数据时，务必首先进行备份！

需要注意的是，在使用 fsck 命令修改某文件系统时，这个文件系统对应的磁盘分区一定要处于卸载状态，磁盘分区在挂载状态下进行修复是非常不安全的，数据可能会遭到破坏，也有可能会损坏磁盘。

这里，给大家举个例子，如果想要修复某个分区，则只需执行如下命令：
```
[root@localhost ~]#fsck -r /dev/sdb1
#采用互动的修复模式
```
fsck 命令在执行时，如果发现存在没有文件系统依赖的文件或目录，就会提示用户是否把它们找回来，因为这些没有文件系统依赖的文件或目录对用户来说是看不到的，换句话说，用户根本无法使用，这通常是由文件系统内部结构损坏导致的。如果用户同意找回（输入 y），fsck 命令就会把这些孤立的文件或目录放到 lost+found 目录中，并用这些文件自己对应的 inode 号来命名，以便用户查找自己丢失的文件。

因此，当用户在利用 fsck 命令修复磁盘分区以后，如果发现分区中有文件丢失，就可以到对应的 lost+found 目录中去查找，但由于无法通过文件名称分辨各个文件，这里可以利用 file 命令查看文件系统类型，进而判断出哪个是我们需要的文件。
# dumpe2fs命令：查看文件系统信息
了解文件系统之后，我们可以使用`dumpe2fs`命令来查看文件系统的详细信息：
```
[root@www ~]# dumpe2fs [-h] 文件名
```
-h 选项的含义是仅列出 superblock（超级块）的数据信息；

例如，通过 df 命令找到根目录硬盘的文件名，然后使用 dump2fs 命令观察文件系统的详细信息，执行命令如下：
```
[root@localhost ~]# df   <==这个命令可以叫出目前挂载的装置
Filesystem    1K-blocks      Used Available Use% Mounted on
/dev/hdc2       9920624   3822848   5585708  41% /
/dev/hdc3       4956316    141376   4559108   4% /home
/dev/hdc1        101086     11126     84741  12% /boot
tmpfs            371332         0    371332   0% /dev/shm

[root@localhost ~]# dumpe2fs /dev/hdc2
dumpe2fs 1.39 (29-May-2006)
Filesystem volume name:   /1             <==这个是文件系统的名称(Label)
Filesystem features:      has_journal ext_attr resize_inode dir_index
  filetype needs_recovery sparse_super large_file
Default mount options:    user_xattr acl <==默认挂载的参数
Filesystem state:         clean          <==这个文件系统是没问题的(clean)
Errors behavior:          Continue
Filesystem OS type:       Linux
Inode count:              2560864        <==inode的总数
Block count:              2560359        <==block的总数
Free blocks:              1524760        <==还有多少个 block 可用
Free inodes:              2411225        <==还有多少个 inode 可用
First block:              0
Block size:               4096           <==每个 block 的大小啦！
Filesystem created:       Fri Sep  5 01:49:20 2008
Last mount time:          Mon Sep 22 12:09:30 2008
Last write time:          Mon Sep 22 12:09:30 2008
Last checked:             Fri Sep  5 01:49:20 2008
First inode:              11
Inode size:               128            <==每个 inode 的大小
Journal inode:            8              <==底下这三个与下一小节有关
Journal backup:           inode blocks
Journal size:             128M

Group 0: (Blocks 0-32767) <==第一个 data group 内容, 包含 block 的启始/结束号码
  Primary superblock at 0, Group descriptors at 1-1  <==超级区块在 0 号 block
  Reserved GDT blocks at 2-626
  Block bitmap at 627 (+627), Inode bitmap at 628 (+628)
  Inode table at 629-1641 (+629)                     <==inode table 所在的 block
  0 free blocks, 32405 free inodes, 2 directories    <==所有 block 都用完了！
  Free blocks:
  Free inodes: 12-32416                              <==剩余未使用的 inode 号码
Group 1: (Blocks 32768-65535)
#由于数据量非常的庞大，这里省略了一部分输出信息
```
可以看到，使用 dumpe2fs 命令可以查询到非常多的信息，以上信息大致可分为 2 部分。前半部分显示的是超级块的信息，包括文件系统名称、已使用以及未使用的 inode 和 block 的数量、每个 block 和 inode 的大小，文件系统的挂载时间等。

另外，Linux 文件系统（EXT 系列）在格式化的时候，会分为多个区块群组（block group），每 个区块群组都有独立的 inode/block/superblock 系统。此命令输出结果的后半部分，就是每个区块群组的详细信息（如 Group0、Group1）。
# fdisk命令：给硬盘分区
我们在安装操作系统的过程中已经对系统硬盘进行了分区，但如果新添加了一块硬盘，想要正常使用，难道需要重新安装操作系统才可以分区吗？

当然不是，在 Linux 中有专门的分区命令`fdisk`和`parted`。其中 fdisk 命令较为常用，但不支持大于 2TB 的分区；如果需要支持大于 2TB 的分区，则需要使用 parted 命令，当然 parted 命令也能分配较小的分区。我们先来看看如何使用 fdisk 命令进行分区。

fdisk 命令的格式如下：
```
[root@localhost ~]# fdisk ~l
#列出系统分区
[root@localhost ~]# fdisk 设备文件名
#给硬盘分区
```
注意，千万不要在当前的硬盘上尝试使用`fdisk`，这会完整删除整个系统，一定要再找一块硬盘，或者使用虚拟机。这里给大家举个例子：
```
[root@localhost ~]# fdisk -l
#查询本机可以识别的硬盘和分区
Disk /dev/sda:32.2 GB, 32212254720 bytes
#硬盘文件名和硬盘大小
255 heads, 63 sectors/track, 3916 cylinders
#共255个磁头、63个扇区和3916个柱面
Units = cylinders of 16065 *512 = 8225280 bytes
#每个柱面的大小
Sector size (logical/physical): 512 bytes/512 bytes
#每个扇区的大小
I/O size (minimum/optimal): 512 bytes/512 bytes
Disk identifier: 0x0009e098
Device Boot Start End Blocks ld System
/dev/sda1 * 1 26 204800 83 Linux
Partition 1 does not end on cylinder boundary.
#分区1没有占满硬盘
/dev/sda2 26 281 2048000 82 Linux swap / Solaris
Partition 2 does not end on cylinder boundary
#分区2没有占满硬盘
/dev/sda3 281 3917 29203456 83 Linux
#设备文件名启动分区 起始柱面 终止柱面容量 ID 系统
Disk /dev/sdb: 21.5 GB, 21474836480 bytes #第二个硬盘识别，这个硬盘的大小
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes/512 bytes Disk identifier: 0x00000000
```
使用 "fdisk -l" 查看分区信息，能够看到我们添加的两块硬盘（/dev/sda 和 /dev/sdb）的信息。我们解释一下这些信息，其上半部分态是硬盘的整体状态，/dev/sda 硬盘的总大小是 32.2 GB，共有 3916 个柱面，每个柱面由 255 个磁头读/写数据，每个磁头管理 63 个扇区。每个柱面的大小是 8225280 Bytes，每个扇区的大小是 512 Bytes。

信息的下半部分是分区的信息，共 7 列，含义如下：
* `Device`：分区的设备文件名。
* `Boot`：是否为启动引导分区，在这里 /dev/sda1 为启动引导分区。
* `Start`：起始柱面，代表分区从哪里开始。
* `End`：终止柱面，代表分区到哪里结束。
* `Blocks`：分区的大小，单位是 KB。
* `id`：分区内文件系统的 ID。在 fdisk 命令中，可以 使用 "i" 查看。
* `System`：分区内安装的系统是什么。

如果这个分区并没有占满整块硬盘，就会提示 "Partition 1 does not end on cyl inder boundary"，表示第一个分区没有到硬盘的结束柱面。大家发现了吗？/dev/sda 已经分配完了分区，没有空闲空间了。而第二块硬盘`/dev/sdb`已经可以被识别了，但是没有可分区。

我们以硬盘`/dev/sdb`为例来做练习:
```
[root@localhost ~]# fdisk /dev/sdb
#给/dev/sdb分区
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0xed7e8bc7.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.
Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)
WARNING: DOS-compatible mode is deprecated.it's strongly recommended to switch off the mode (command 'c') and change display units to sectors (command 'u').
Command (m for help):m
#交互界面的等待输入指令的位置，输入 m 得到帮助
Command action
#可用指令
a toggle a bootable flag
b edit bsd disklabel
c toggle the dos compatibility flag
d delete a partition
I list known partition types m print this menu
n add a new partition
o create a new empty DOS partition table
p print the partition table
q quit without saving changes
s create a new empty Sun disklabel
t change a partition's system id
u change display/entry units
v verity the partition table
w write table to disk and exit
x extra functionality (experts only)
```
注意这里的分区命令是 "fdisk /dev/sdb"，这是因为硬盘并没有分区，使用 fdisk 命令的目的就是建立分区。

在 fdisk 交互界面中输入 m 可以得到帮助，帮助里列出了 fdisk 可以识别的交互命令，我们来解释一下这些命令，如表 1 所示。

表 1 fdisk 交互
命令	说 明
a	设置可引导标记
b	编辑 bsd 磁盘标签
c	设置 DOS 操作系统兼容标记
d	删除一个分区
1	显示已知的文件系统类型。82 为 Linux swap 分区，83 为 Linux 分区
m	显示帮助菜单
n	新建分区
0	建立空白 DOS 分区表
P	显示分区列表
q	不保存退出
s	新建空白 SUN 磁盘标签
t	改变一个分区的系统 ID
u	改变显示记录单位
V	验证分区表
w	保存退出
X	附加功能（仅专家）

# Linux fdisk创建分区（主分区、扩展分区和逻辑分区）过程详解
我们实际建立一个主分区，看看过程是什么样子的。
```
[root@localhost ~]# fdisk /dev/sdb
…省略部分输出…
Command (m for help): p
\#显示当前硬盘的分区列表
Disk /dev/sdb: 21.5 GB, 21474836480 bytes 255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 *512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xb4b0720c
Device Boot Start End Blocks id System
\#目前一个分区都没有
Command (m for help): n
\#那么我们新建一个分区
Command action
\#指定分区类型
e extended
\#扩展分区
p primary partition (1-4)
\#主分区
p
\#这里选择p，建立一个主分区
Partition number (1-4): 1
\#选择分区号，范围为1~4，这里选择1
First cylinder (1 -2610, default 1):
\#分区的起始柱面，默认从1开始。因为要从硬盘头开始分区，所以直接回车
Using default value 1
\#提示使用的是默认值1
Last cylinder, +cylinders or +size{K, M, G}(1-2610, default 2610): +5G
\#指定硬盘大小。可以按照柱面指定(1-2610)。我们对柱面不熟悉，那么可以使用size{K, M, G}的方式指定硬盘大小。这里指定+5G，建立一个5GB大小的分区
Command (m for help):
\#主分区就建立了，又回到了fdisk交互界面的提示符
Command (m for help): p
\#查询一下新建立的分区
Disk /dev/sdb: 21.5GB, 21474836480 bytes
255 heads,63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes 1512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xb4b0720c
Device Boot Start End Blocks id System
/dev/sdb1 1 654 5253223+ 83 [linux](http://www.beylze.cn/linux/)
\#dev/sdb1已经建立了吧
```
总结，建立主分区的过程就是这样的："fdisk 硬盘名 -> n(新建)->p(建立主分区) -> 1(指定分区号) -> 回车（默认从 1 柱面开始建立分区）-> +5G(指定分区大小)"。当然，我们的分区还没有格式化和挂载，所以还不能使用。

主分区和扩展分区加起来最多只能建立 4 个，而扩展分区最多只能建立 1 个。

扩展分区的建立命令如下：
```
[root@localhost ~]# fdisk /dev/sdb
…省略部分输出…
Command (m for help): n
\#新建立分区
Command action
e extended
p primary partition (1-4)
e
\#这次建立扩展分区
Partition number (1-4): 2
\#给扩展分区指定分区号2
First cylinder (655-2610, default 655):
\#扩展分区的起始柱面。上节建立的主分区1已经占用了1~654个柱面，所以我们从655开始建立，注意：如果没有特殊要求，则不要跳开柱面建立分区，应该紧挨着建立分区
Using default value 655
提示使用的是默认值655
Last cylinder, +cylinders or +size{K, M, G} (655-2610, default 2610):
\#这里把整块硬盘的剩余空间都建立为扩展分区
Using default value 2610
\#提示使用的是默认值2610
```
这里把 /dev/sdb 硬盘的所有剩余空间都建立为扩展分区，也就是建立一个主分区，剩余空间都建立成扩展分区，再由扩展分区中建立逻辑分区。
扩展分区是不能被格式化和直接使用的，所以还要在扩展分区内部再建立逻辑分区。

我们来看看逻辑分区的建立过程：
```
[root@localhost ~]# fdisk /dev/sdb
…省略部分输出…
Command (m for help): n
\#建立新分区
Command action
l logical (5 or over)
\#由于在前面章节中，扩展分区已经建立，所以这里变成了l(logic)
p primary partition (1-4)
l
\#建立逻辑分区
First cylinder (655-2610, default 655):
\#不用指定分区号，默认会从5开始分配，所以直接选择起始柱面
\#注意：逻辑分区是在扩展分区内部再划分的，所以柱面是和扩展分区重叠的
Using default value 655
Last cylinder, +cylinders or +size{K, M, G} (655-2610, default 2610):+2G
\#分配2GB大小
Command (m for help): n
\#再建立一个逻辑分区
Command action
l logical (5 or over)
p primary partition (1-4)
l
First cylinder (917-2610, default 917):
Using default value 917
Last cylinder, +cylinders or +size{K, M, G} (917-2610, default 2610):+2G
Command (m for help): p
\#查看一下已经建立的分区
Disk /dev/sdb: 21.5GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 *512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes Disk identifier: 0xb4b0720c
Device Boot Start End Blocks id System
/dev/sdb1 1 654
5253223+ 83 Linux
\#主分区
/dev/sdb2 655 2610 15711570
5 Extended
\#扩展分区
/dev/sdb5 655 916
2104483+ 83 Linux
\#逻辑分区 1
/dev/sdb6 917 1178
2104483+ 83 Linux
\#逻辑分区2
Command (m for help): w
\#保存并退出
The partition table has been altered!
Calling ioctl。to re-read partition table.
Syncing disks.
[root@localhost -]#
\#退回到提示符界面
```
所有的分区立过程中如果不保存并退出是不会生效的，所以建立错了也没有关系，使用 q 命令不保存退出即可。如果使用了 w 命令，就会保存退出。有时因为系统的分区表正忙，所以需要重新启动系统才能使新的分区表生效。命令如下：
```
Command (m for help): w
\#保存并退出
The partition table has been altered!
Calling ioctl() to re-read partition table.
WARNING: Re-reading the partition table failed with error 16:
Device or resource busy.
The kernel still uses the old table.
The new table will be used at the next reboot.
\#要求重新启动，才能格式化
Syncing disks.
```
看到了吗？必须重新启动！可是重新启动很浪费时间。如果不想重新启动，则可以使用 partprobe 命令。这个命令的作用是让系统内核重新读取分区表信息，这样就可以不用重新启动了。命令如下：
```
[root@localhost ~]# partprobe
```
如果这个命令不存在，则请安装 parted-2.1-18.el6.i686 这个软件包。partprobe 命令不是必需的，如果没有提示重启系统，则直接格式化即可。
# parted命令用法：创建分区
虽然我们可以使用 fdisk命令对硬盘进行快速的分区，但对高于 2TB 的硬盘分区，此命令却无能为力，此时就需要使用`parted`命令。

`parted`命令是可以在命令行直接分区和格式化的，不过`parted`交互模式才是更加常用的命令方式，进入交互模式的方法如下：
```
[root@localhost ~]# parted 硬盘设备文件名
#进入交互模式
```
例如：
```
[root@localhost ~]# parted /dev/sdb
#打算继续划分/dev/sdb硬盘
GNU Parted 2.1
```
## 使用/dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)   <--parted 的等待输入交互命令的位置，输入 help，可以看到在交互模式下支持的所有命令

parted 交互命令比较多，我们介绍常见的命令，如表 1 所示。

表 1 parted常见的交互命令
parted交互命令	说 明
check NUMBER	做一次简单的文件系统检测
cp [FROM-DEVICE] FROM-NUMBER TO-NUMBER	复制文件系统到另一个分区
help [COMMAND]	显示所有的命令帮助
mklabel,mktable LABEL-TYPE	创建新的磁盘卷标（分区表）
mkfs NUMBER FS-TYPE	在分区上建立文件系统
mkpart PART-TYPE [FS-TYPE] START END	创建一个分区
mkpartfs PART-TYPE FS-TYPE START END	创建分区，并建立文件系统
move NUMBER START END	移动分区
name NUMBER NAME	给分区命名
print [devices|free|list,all|NUMBER]	显示分区表、活动设备、空闲空间、所有分区
quit	退出
rescue START END	修复丢失的分区
resize NUMBER START END	修改分区大小
rm NUMBER	删除分区
select DEVICE	选择需要编辑的设备
set NUMBER FLAG STATE	改变分区标记
toggle [NUMBER [FLAG]]	切换分区表的状态
unit UNIT	设置默认的单位
Version	显示版本
【例 1】查看分区表
```
(parted) print
#进入print指令
Model: VMware, VMware Virtual S (scsi)
#硬盘参数，是虚拟机
Disk/dev/sdb: 21.5GB
#硬盘大小
Sector size (logical/physical): 512B/512B
#扇区大小
Partition Table: msdos
#分区表类型，是MBR分区表
Number Start End Size Type File system 标志
1 32.3kB 5379MB 5379MB primary
2 5379MB 21.5GB 16.1GB extended
5 5379MB 7534MB 2155MB logical ext4
6 7534MB 9689MB 2155MB logical ext4
#看到了我们使用fdisk命令创建的分区，其中1分区没被格式化；2分区是扩展分区，不能被格式化
```
使用 print 命令可以査看分区表信息，包括硬盘参数、硬盘大小、扇区大小、分区表类型和分区信息。分区信息共有 7 列，分别如下：
Number：分区号，比如，1号就代表 /dec/sdb1；
Start：分区起始位置。这里不再像 fdisk 那样用柱面表示，使用字节表示更加直观；
End：分区结束位置；
Size：分区大小；
Type：分区类型，有 primary、extended、logical 等类型；
Filesystem：文件系统类型；
标志：分区的标记。

【例 2】修改成 GPT 分区表
```
(partcd) mklabel gpt
#修改分区表命令
警告：正在使用/dev/sdb上的分区。由于/dev/sdb分区已经挂载，所以有警告。注意，如果强制修改，那么原有分区及数据会消失
忽略/Ignore/放弃/Cancel? ignore
#输入ignore忽略报错
警告：The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to continue?
是/Yes/否/No? yes
#输入 yes
警告：WARNING: the kernel failed to re-read the partition table on /dev/sdb (设 备或资源忙）.As a result, it may not reflect all of your changes until after reboot.
#下次重启后才能生效
(parted) print
#查看一下分区表
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
#分区表已经变成 GPT
Number Start End Size File system Name 标志
#所有的分区都消失了
```
修改了分区表，如果这块硬盘上已经有分区了，那么原有分区和分区中的数据都会消失，而且需要重启系统才能生效。

另外，我们转换分区表的目的是支持大于 2TB 的分区，如果分区并没有大于 2TB，那么这一步是可以不执行的。
注意，一定要把 /etc/fstab 文件和原有分区中的内容删除才能重启，否则会报错。


【例 3】建立分区
因为修改过了分区表，所以/dev/sdb硬盘中的所有数据都消失了，我们就可以重新对这块硬盘分区了。不过，在建立分区时，默认文件系统就只能是 ext2 了。命令如下：
```
(parted)mkpart
#输入创建分区命令，后面不要参数，全部靠交互
指定
分区名称？ []?disk1
#分区名称，这里命名为disk 1
文件系统系统？ [ext2]?
#文件系统类型，直接回车，使用默认文件系统ext2
起始点？ 1MB
#分区从1MB开始
结束点？5GB分区到5GB结束
#分区完成
(parted) print
#查看一下
Model: VMware, VMware Virtual S (scsi)
Disk/dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B Partition Table: gpt
Number Start End Size Rle system Name 标志
1 1049kB 5000MB 4999MB disk1
#分区1已经出现
```
不知道大家有没有注意到，我们现在用 print 查看的分区和第一次查看 MBR 分区表的分区时有些不一样了，少了 Type 这个字段，也就是分区类型字段，多了 Name（分区名）字段。分区类型是用于标识主分区、扩展分区和逻辑分区的，不过这种标识只在 MBR 分区表中使用，现在已经变成了 GPT 分区表，所以就不再有 Type 类型了。

【例 4】建立文件系统
分区分完后，还需要进行格式化。我们知道，如果使用 parted 交互命令格式化，则只能格式化成 ext2 文件系统。我们在这里要演示一下 parted 命令的格式化方法，所以就格式化成 ext2 文件系统。命令如下：
```
(parted) mkfs
#格式化命令（很奇怪，也是mkfs，但是这只是parted的交互命令）
WARNING: you are attempting to use parted to operate on (mkfs) a file system.
parted's file system manipulation code is not as robust as what you'll find in
dedicated, file-system-specific packages like e2fsprogs. We recommend
you use parted only to manipulate partition tables, whenever possible.
Support for performing most operations on most types of file systems
will be removed in an upcoming release.
警告：The existing file system will be destroyed and all data on the partition will be lost. Do you want to continue?
是/Yes/否/No? yes
#警告你格式化丟失，没关系，已经丢失过了
```
分区编号？ 1
文件系统类型 [ext2]?
#指定文件系统类型，写别的也没用，直接回车
(parted) print #格式化完成，查看一下
Model: VMware, VMware Virtual S (scsi)
Disk/dev/sdb: 21,5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number Start End Size File system Name标志
1 1049kB 5000MB 4999MB ext2 diski
#拥有了文件系统

如果要格式化成 ext4 文件系统，那么请 mkfs 命令帮忙吧（注意：不是 parted 交互命令中的 mkfs，而是系统命令 mkfs)。

【例 5】调整分区大小
parted 命令还有一大优势，就是可以调整分区的大小（在 Windows 中也可以实现，不过要么需要转换成动态磁盘，要么需要依赖第三方工具，如硬盘分区魔术师）。起始 Linux 中 LVM 和 RAID 是可以支持分区调整的，不过这两种方法也可以看成动态磁盘方法，使用 parted 命令调整分区更加简单。

注意，parted 调整已经挂载使用的分区时，是不会影响分区中的数据的，也就是说，数据不会丢失。但是一定要先卸载分区，再调整分区大小，否则数据是会出现问题的。另外，要调整大小的分区必须已经建立了文件系统（格式化），否则会报错。

命令如下：
(parted) resize
分区编号？ 1
#指定要修改的分区编号
起始点？ [1049kB]? 1MB
#分区起始位置
结束点？ [5000MB]? 6GB
分区结束位置
(parted) print
#查看一下
Model: VMware, VMware Virtual S (scsi)
Disk/dev/sdb: 21,5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number Start End Size File system Name标志
1 1049kB 6000MB 5999MB ext2 diski
#分区大小改变


删除分区

命令如下：
```
(parted) rm
#删除分区命令
分区编号？ 1
#指定分区编号
(parted) print
#查看一下
Model: VMware, VMware Virtual S (scsi)
Disk/dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number Start End Size File system Name 标志 #分区消失
```
要注意的是，parted 中所有的操作都是立即生效的，没有保存生效的概念。这一点和 fdisk 交互命令明显不同，所以做的所有操作大家要加倍小心。

那么，到底是使用 fdisk 命令，还是使用 parted 命令进行分区呢？这完全看个人习惯，我们更加习惯使用 fdisk 命令。
# mkfs命令详解:格式化分区（为分区写入文件系统）
分区完成后，如果不格式化写入文件系统，则是不能正常使用的。这时就需要使用 mkfs 命令对硬盘分区进行格式化。

mkfs 命令格式如下：
```
[root@localhost ~]# mkfs [-t 文件系统格式] 分区设备文件名
```
-t 文件系统格式：用于指定格式化的文件系统，如 ext3、ext4；

前面章节中，我们建立了 /dev/sdb1（主分区）、/dev/sdb2（扩展分区）、/dev/sdb5（逻辑分区）和 /dev/sdb6（逻辑分区）这几个分区，其中 /dev/sdb2 不能被格式化。剩余的三个分区都需要格式化之后使用，这里我们以格式化 /dev/sdb6 分区作为演示，其余分区的格式化方法一样。

格式化 /dev/sdb6 分区的执行命令如下：
```
[root@localhost ~]# mkfs -t ext4 /dev/sdb6
mke2fs 1.41.12 (17-May-2010)
Filesystem label=  <--这里指的是卷标名，我们没有设置卷标
OS type：Linux
Block size=4096 (log=2) <--block 的大小配置为 4K
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
131648 inodes, 526120 blocks <--由此配置决定的inode/block数量
26306 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=541065216 17 block groups
32768 blocks per group, 32768 fragments per group
7744 inodes per group
Superblock backups stored on blocks:
32768, 98304, 163840, 229376, 294912
Writing inode tables: done
Creating journal (16384 blocks):done
Writing superblocks and filesystem accounting information:done
This filesystem will be automatically checked every 39 mounts or 180 days, whichever comes first. Use tune2fs -c or -i to override.
# 这样就创建起来所需要的 Ext4 文件系统了！简单明了！
```
虽然 mkfs 命令非常简单易用，但其不能调整分区的默认参数（比如块大小是 4096 Bytes），这些默认参数除非特殊清况，否则不需要调整。如果想要调整，就需要使用 mke2fs 命令重新格式化。

`mkfs`命令为硬盘分区写入文件系统时，无法手动调整分区的默认参数（比如块大小是 4096 Bytes），如果想要调整，就需要使用`mke2fs`命令。

mke2fs 命令的基本格式如下：
```
[root@localhost ~]# mke2fs [选项] 分区设备文件名
```
表 1 罗列出了 mke2fs 命令常用的几个选项及各自的功能。

表 1 mke2fs命令常用选项及功能
选项	功能
-t 文件系统	指定格式化成哪个文件系统， 如 ext2、ext3、ext4；
-b 字节	指定 block 的大小；
-i 字节	指定"字节 inode "的比例，也就是多少字节分配一个 inode；
-j	建立带有 ext3 日志功能的文件系统；
-L 卷标名	给文件系统设置卷标名，就不使用 e2label 命令设定了；
为了更好的对比 mkfs 命令，这里我们依旧以格式化 /dev/sdb6 为例，不过，这次使用的是 mke2fs 命令，执行命令如下：
```
[root@localhost ~]# mke2fs -t ext4 -b 2048 /dev/sdb6
#格式化分区，并指定block的大小为2048 Bytes
mke2fe 1.41.12 (17-May-2010)
Filesystem label=
OS type：Linux
Block size=2048 (log=1)  <--block 的大小配置为 2K
Fragment size=2048 (log=1)
Stride=0 blocks, Stripe width=0 blocks 131560
inodes,1052240 blocks 52612 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=538968064 65 block groups
16384 blocks per group, 16384 fragments per group
2024 inodes per group
Superblock backups stored on blocks:
16384, 49152, 81920, 114688, 147456, 409600, 442368, 802816
Writing inode tables: done
Creating journal (32768 blocks):done
Writing superblocks and filesystem accounting information:done
This filesystem will be automatically checked every 38 mounts or 180 days, whichever comes first. Use tune2fs -c or-i to override.
```
如果没有特殊需要，建议使用 mkfs 命令对硬盘分区进行格式化。
# 虚拟内存和物理内存
我们都知道，直接从内存读写数据要比从硬盘读写数据快得多，因此更希望所有数据的读取和写入都在内存中完成，然而内存是有限的，这样就引出了物理内存与虚拟内存的概念。

物理内存就是系统硬件提供的内存大小，是真正的内存。相对于物理内存，在 Linux 下还有一个虚拟内存的概念，虚拟内存是为了满足物理内存的不足而提出的策略，它是利用磁盘空间虚拟出的一块逻辑内存。用作虚拟内存的磁盘空间被称为交换空间（又称 swap 空间）。

作为物理内存的扩展，Linux 会在物理内存不足时，使用交换分区的虚拟内存，更详细地说，就是内核会将暂时不用的内存块信息写到交换空间，这样一来，物理内存得到了释放，这块内存就可以用于其他目的，当需要用到原始的内容时，这些信息会被重新从交换空间读入物理内存。

Linux 的内存管理采取的是分页存取机制，为了保证物理内存能得到充分的利用，内核会在适当的时候将物理内存中不经常使用的数据块自动交换到虚拟内存中，而将经常使用的信息保留到物理内存。

要深入了解 Linux 内存运行机制，需要知道下面提到的几个方面：
首先，Linux 系统会不时地进行页面交换操作，以保持尽可能多的空闲物理内存，即使并没有什么事情需要内存，Linux 也会交换出暂时不用的内存页面，因为这样可以大大节省等待交换所需的时间。
其次，Linux 进行页面交换是有条件的，不是所有页面在不用时都交换到虚拟内存，Linux 内核根据“最近最经常使用”算法，仅仅将一些不经常使用的页面文件交换到虚拟内存。

有时我们会看到这么一个现象，Linux 物理内存还有很多，但是交换空间也使用了很多，其实这并不奇怪。例如，一个占用很大内存的进程运行时，需要耗费很多内存资源，此时就会有一些不常用页面文件被交换到虚拟内存中，但后来这个占用很多内存资源的进程结束并释放了很多内存时，刚才被交换出去的页面文件并不会自动交换进物理内存（除非有这个必要），那么此时系统物理内存就会空闲很多，同时交换空间也在被使用，就出现了刚才所说的现象了。

最后，交换空间的页面在使用时会首先被交换到物理内存，如果此时没有足够的物理内存来容纳这些页面，它们又会被马上交换出去，如此一来，虚拟内存中可能没有足够的空间来存储这些交换页面，最终会导致 Linux 出现假死机、服务异常等问题。Linux 虽然可以在一段时间内自行恢复，但是恢复后的系统己经基本不可用了。

因此，合理规划和设计 Linux 内存的使用是非常重要的，关于物理内存和交换空间的大小设置问题，取决于实际所用的硬盘大小，但大致遵循这样一个基本原则：
如果内存较小（根据经验，物理内存小于 4GB），一般设置 swap 分区大小为内存的 2 倍；
如果物理内存大于 4GB，而小于 16GB，可以设置 swap 分区大小等于物理内存；
如果内存大小在 16GB 以上，可以设置 swap 为 0，但并不建议这么做，因为设置一定大小的 swap 分区是有一定作用的。

# swap分区及作用详解
swap 分区通常被称为交换分区，这是一块特殊的硬盘空间，即当实际内存不够用的时候，操作系统会从内存中取出一部分暂时不用的数据，放在交换分区中，从而为当前运行的程序腾出足够的内存空间。

也就是说，当内存不够用时，我们使用`swap`分区来临时顶替。这种“拆东墙，补西墙”的方式应用于几乎所有的操作系统中。

使用`swap`交换分区，显著的优点是，通过操作系统的调度，应用程序实际可以使用的内存空间将远远超过系统的物理内存。由于硬盘空间的价格远比 RAM 要低，因此这种方式无疑是经济实惠的。当然，频繁地读写硬盘，会显著降低操作系统的运行速率，这也是使用 swap 交换分区最大的限制。

相比较而言，Windows 不会为 swap 单独划分一个分区，而是使用分页文件实现相同的功能，在概念上，Windows 称其为虚拟内存，从某种意义上将，这个叫法更容易理解。因此，初学者将 swap 交换分区理解为虚拟内存是没有任何问题的。

具体使用多大的 swap 分区，取决于物理内存大小和硬盘的容量。一般来讲，swap 分区容量应大于物理内存大小，建议是内存的两倍，但不超过 2GB。但是，有时服务器的访问量确实很大，有可能出现 swap 分区不够用的情况，所以我们需要学习 swap 分区的构建方法。

建立新的 swap 分区，只需要执行以下几个步骤。
* 分区：不管是 fdisk 命令还是 parted 命令，都需要先区。
* 格式化：格式化命令稍有不同，使用 mkswap 命令把分区格式化成 swap 分区。
* 使用 swap 分区。

下面我们来逐一实现。
## 建立swap分区第一步：分区
```
[root@localhost ~]# fdisk /dev/sdb
\#以/dev/sdb分区为例
WARNING: DOS-compatible mode is deprecated.It's strongly recommended to switch off the mode (command 'c') and change display units to sectors (command 'u').
Command (m for help): n
\#新建
Command action e extended p primary partition (1-4)
P
\#主分区
Partition number (1-4): 1
\#分区编号
First cylinder (1-2610, default 1):
\#起始柱面
Using default value 1
Last cylinder, +cylinders or +size{K, M, G} (1-2610, default 2610): +500M
\#大小
Command (m for help): p
\#查看一下
Disk /dev/sdb: 21.5GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 *512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes 1512 bytes
Disk identifier: OxOOOOOebd
Device Boot Start End Blocks Id System
/dev/sdb1 1 65 522081 83 [linux](http://www.beylze.cn/linux/)
\#刚分配的分区ID是83，是Linux分区，我们在这里要分配swap分区
Command (m for help): t
\#修改分区的系统ID
Selected partition 1
\#只有一个分区，所以不用选择分区了
Hex code (type L to list codes): 82
\#改为swap分区的ID
Changed system type of partition 1 to 82 (Linux swap / Solaris)
Command (m for help): p
\#再查看一下
Disk /dev/sdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 *512 = 8225280 bytes Sector size (logical/physical): 512 bytes / 512 bytes I/O size (minimum/optimal): 512 bytes 1512 bytes Disk identifier: OxOOOOOebd
Device Boot Start End Blocks Id System
/dev/sdb1 1 65 522081 82 Linux swap / Solaris
\#修改过来了
Command (m for help): w
\#记得保存退出
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.
```
仍以 /dev/sdb 分区作为实验对象。不过，如果分区刚刚使用 parted 命令转变为 GPT 分区表，则记得转换回 MBR 分区表，fdisk 命令才能识别，否则干脆新添加一块硬盘做实验。
## 建立 swap 分区第二步：格式化
因为要格式化成 swap 分区，所以格式化命令是 mkswap。
```
[root@localhost ~]# mkswap /dev/sdb1
Setting up swapspace version 1, size = 522076 KiB
no label, UUID=c3351 dc3-f403-419a-9666-c24615e170fb
```
## 使用swap分区
在使用 swap 分区之前，我们先来说说 free 命令。
```
[root@localhost ~]#free
total used free shared buffers cached
Mem: 1030796 130792 900004 0 15292 55420
-/+ buffers/cache: 60080 970716
Swap: 2047992 0 2047992
```
free 命令主要是用来查看内存和 swap 分区的使用情况的，其中：

total：是指总数；
used：是指已经使用的；
free：是指空闲的；
shared：是指共享的；
buffers：是指缓冲内存数；
cached：是指缓存内存数，单位是KB；
我们需要解释一下 buffers（缓冲）和 cached（缓存）的区别。简单来讲，cached 是给读取数据时加速的，buffers 是给写入数据加速的。cached 是指把读取出来的数据保存在内存中，当再次读取时，不用读取硬盘而直接从内存中读取，加速了数据的读取过程；buffers 是指在写入数据时，先把分散的写入操作保存到内存中，当达到一定程度后再集中写入硬盘，减少了磁盘碎片和硬盘的反复寻道，加速了数据的写入过程。

我们已经看到，在加载进新的`swap`分区之前，`swap`分区的大小是 2000MB，接下来只要加入`swap`分区就可以了，使用命令`swapon`。命令格式如下：
```
[root@localhost ~]# swapon 分区设备文件名
```
```
[root@localhost ~]# swapon /dev/sdb1
swap分区已加入，我们查看一下。
[root@localhost ~]#free
total used free shared buffers cached
Mem: 1030796 131264 899532 0 15520 55500
-/+ buffers/cache: 60244 970552
Swap: 2570064 0 2570064
```
swap 分区的大小变成了 2500MB，加载成功了。如果要取消新加入的 swap 分区，则也很简单，命令如下：
```
[root@localhost ~]# swapoff /dev/sdb1
```
如果想让`swap`分区开机之后自动挂载，就需要修改`/etc/fstab`文件，命令如下：
```
[root@localhost ~]#vi /etc/fstab
UUID=c2ca6f57-b15c-43ea-bca0-f239083d8bd2 / ext4 defaults 1 1
UUID=0b23d315-33a7-48a4-bd37-9248e5c443451 boot ext4 defaults 1 2
UUID=4021be19-2751-4dd2-98cc-383368c39edb swap swap defaults 0 0
tmpfs /dev/shm
tmpfs defaults 0 0
devpts /dev/pts
devpts gid=5, mode=620 0 0
sysfs /sys
sysfs defaults 0 0
proc /proc
proc defaults 0 0
/dev/sdb1 swap swap
defaults 0 0
\#加入新swap分区的相关内容，这里直接使用分区的设备文件名，也可以使用UUID。
```
