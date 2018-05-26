# VFS

## 总结

1、VFS**只存在于内存中**，它在系统启动时被创建，系统关闭时注销。

2、VFS的作用就是屏蔽各类文件系统的差异，给用户、应用程序、甚至Linux其他管理模块提供统一的接口集合。

3、管理VFS数据结构的组成部分主要包括超级块和inode。

VFS是物理文件系统与服务之间的一个接口层，它对Linux的每个文件系统的所有细节进行抽象，使得不同的文件系统在Linux核心以及系统中运行的进程看来都是相同的。

**严格的说，VFS并不是一种实际的文件系统。它只存在于内存中，不存在于任何外存空间**。VFS在系统启动时建立，在系统关闭时消亡。

VFS使Linux同时安装、支持许多不同类型的文件系统成为可能。VFS拥有关于各种特殊文件系统的公共接口，**当某个进程调用了一个面向文件的系统调用时，内核将调用VFS中对应的函数，这个函数处理一些与物理结构无关的操作，并且把它重定向为真实文件系统中相应的函数调用，后者用来处理那些与物理结构相关的操作**。

下图就是逻辑上对VFS及其下层实际文件系统的组织图，可以看到**用户层只能于VFS打交道，而不能直接访问实际的文件系统**，比如EXT2、EXT3、PROC，换句话说，就是用户层不用也不能区别对待这些真正的文件系统，不过，**SOCKET虽然也属于VFS的管辖范围，但是有其特殊性，就是不能像打开大部分文件系统下的“文件”一样打开socket，它只能被创建，而且内核中对其有特殊性处理**。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/1.png)

![](http://oklbfi1yj.bkt.clouddn.com/VFS/6.jpg)

也就是说，**我们在用户空间所使用的针对文件系统的系统调用实际上是在和操作系统的VFS打交道**。不同的文件系统，如Ext2/3、XFS、FAT32等，具有不同的结构，假如用户调用open等文件IO函数去打开文件，具体的实现会非常不同。为了屏蔽这种差异，Linux引入了VFS的概念。相当于是Linux自建了一个新的贮存在内存中的文件系统（这个文件系统就是VFS）。所有其他文件系统都需要先转换成VFS的结构才能为用户所调用。

## VFS的构建

所谓VFS的构建就是加载实际文件系统的过程，也就是mount被调用的过程。如下图所示，以mount（挂载）一个ext2的文件系统为例。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/7.jpg)

VFS文件系统的基本结构是dentry结构体与inode结构体。

**Dentry代表一个文件目录中的一个点，可以是目录也可以是文件**。

Inode代表一个在磁盘上的文件，它与磁盘文件一一对应。

**Inode与dentry不一定一一对应，一个inode可能会对应多个dentry项。（hard link）**

Mount时，linux首先找到磁盘分区的super block，然后通过解析磁盘的inode table与file data，构建出自己的dentry列表与indoe列表。

VFS描述文件系统使用超级块和inode 的方式，所谓超级块就是对所有文件系统的管理机构，每种文件系统都要把自己的信息挂到super_blocks这么一个全局链表上。

内核中是分成2个步骤完成：首先每个文件系统必须通过register_filesystem函数将自己的file_system_type挂接到file_systems这个全局变量上，然后调用kern_mount函数把自己的文件相关操作函数集合表挂到super_blocks上。每种文件系统类型的读超级块的例程（get_sb）必须由自己实现。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/2.png)

文件系统由子目录和文件构成。每个子目录和文件只能由唯一的inode 描述。inode 是Linux管理文件系统的最基本单位，也是文件系统连接任何子目录、文件的桥梁。

VFS inode的内容取自物理设备上的文件系统，由文件系统指定的操作函数(i_op 属性指定)填写。VFS inode只存在于内存中，可通过inode缓存访问。

## VFS的结构

**VFS主要由dentry与inode构成**。Dentry用于维护VFS的目录结构，**每个dentry项就代表着我们用ls时看的的一项（每个目录和每个文件都对应着一个dentry项）**。**Inode为文件节点，它与文件一一对应**。Linux中，目录也是一种文件，所以dentry也会对应一个inode节点。

## super_block

### 相关的数据结构为

```c
struct super_block
{
struct list_head s_list;/* Keep this first */// 连接super_block的链表
dev_t s_dev;/* search index; _not_ kdev_t */
unsignedlong s_blocksize;
unsignedlong s_old_blocksize;
unsignedchar s_blocksize_bits;
unsignedchar s_dirt;
unsignedlonglong s_maxbytes;/* Max file size */
struct file_system_type *s_type;// 所表示的文件系统的类型
struct super_operations *s_op;// 文件相关操作函数集合表
struct dquot_operations *dq_op;//
struct quotactl_ops *s_qcop;//
struct export_operations *s_export_op;//
unsignedlong s_flags;//
unsignedlong s_magic;//
struct dentry *s_root;// Linux文件系统中某个索引节点(inode)的链接
struct rw_semaphore s_umount;//
struct semaphore s_lock;//
int s_count;//
int s_syncing;//
int s_need_sync_fs;//
atomic_t s_active;//
void*s_security;//
struct xattr_handler **s_xattr;//
struct list_head s_inodes;/* all inodes */// 链接文件系统的inode
struct list_head s_dirty;/* dirty inodes */
struct list_head s_io;/* parked for writeback */
struct hlist_head s_anon;/* anonymous dentries for (nfs) exporting */
struct list_head s_files;// 对于每一个打开的文件,由file对象来表示。链接文件系统中file
struct block_device *s_bdev;//
struct list_head s_instances;//
struct quota_info s_dquot;/* Diskquota specific options */
int s_frozen;//
wait_queue_head_t s_wait_unfrozen;//
char s_id[32];/* Informational name */
void*s_fs_info;/* Filesystem private info */
/**
* The next field is for VFS *only*. No filesystems have any business
* even looking at it. You had been warned.
*/
struct semaphore s_vfs_rename_sem;/* Kludge */
/* Granuality of c/m/atime in ns.
Cannot be worse than a second */
u32 s_time_gran;
};
```

super_block存在于两个链表中,一个是系统所有super_block的链表, 一个是对于特定的文件系统的super_block链表。所有的super_block都存在于 super_blocks(VFS管理层) 链表中：

![](http://oklbfi1yj.bkt.clouddn.com/VFS/3.png)

对于特定的文件系统(文件系统层的具体文件系统), 该文件系统的所有的super_block 都存在于file_sytem_type中的fs_supers链表中，而所有的文件系统,都存在于file_systems链表中.这是通过调用register_filesystem接口来注册文件系统的：

```c++
int register_filesystem(struct file_system_type * fs);
```

![](http://oklbfi1yj.bkt.clouddn.com/VFS/4.png)

## inode

inode译成中文就是索引节点，它用来存放档案及目录的基本信息，包含时间、档名、使用者及群组等。

inode 是 UNIX 操作系统中的一种数据结构，其本质是结构体，它包含了与文件系统中各个文件相关的一些重要信息。在 UNIX 中创建文件系统时，同时将会创建大量的 inode 。通常，文件系统磁盘空间中大约百分之一空间分配给了 inode 表。

dentry（dir entry）的中文名称是目录项，是Linux文件系统中某个索引节点(inode)的链接。这个索引节点可以是文件的，也可以是目录的。

inode对应于物理磁盘上的具体对象，**dentry是一个内存实体，其中的d_inode成员指向对应的inode**。

inode存在于两个双向链表中:
一个是inode所在文件系统的super_block的 s_inodes 链表中
一个是根据inode的使用状态存在于以下三个链表中的某个链表中:
未用的 inode_unused 链表，正在使用的 inode_in_use 链表，脏的 super block中的s_dirty 链表。另外，还有一个重要的链表 inode_hashtable(这个暂不介绍)。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/5.PNG)

有时，人们使用了一些不同的术语，如 **inode** 和 **索引编号** (inumber)。这两个术语非常相似，并且相互关联，但它们所指的并不是同样的概念。 **inode 指的是数据结构；而索引编号实际上是 inode 的标识编号**，因此**也称其为 inode 编号** 或者索引编号。索引编号只是文件相关信息中一项重要的内容。

**inode 表**包含一份清单，其中**列出了对应文件系统的所有 inode 编号**。当用户搜索或者访问一个文件时，UNIX 系统通过 inode 表查找正确的 inode 编号。**在找到 inode 编号之后，相关的命令才可以访问该 inode ，并对其进行适当的更改**。 

例如，使用 `vi` 来编辑一个文件。当您键入 `vi <filename>` 时，在 inode 表中找到 inode 编号之后，才允许您打开该 inode 。在 `vi` 的编辑会话期间，更改了该 inode 中的某些属性，当您完成操作并键入 `:wq`时，将关闭并释放该 inode 。通过这种方式，**如果两个用户试图对同一个文件进行编辑， inode 已经在第一个编辑会话期间分配给了另一个用户 ID (UID)，因此第二个编辑任务就必须等待，直到该 inode 释放为止**。

### inode 的结构

从根本上讲， **inode 中包含有关文件的所有信息（除了文件的实际名称以及实际数据内容之外）**。

与其他的操作系统相比，UNIX 系统中的目录和文件可能看起来有所不同，但事实并非如此。**在 UNIX 中，目录本身就是文件，只是在它们的 inode 中使用了一些附加的设置。目录 本质上就是一个包含了其他文件的文件**。**另外，其模式信息中设置了一些相应的标志，以告知系统该文件实际上是一个目录**。

### 使用 inode

当您在 UNIX 中创建一个文件系统时，将为 inode 表分配大约百分之一的总磁盘空间。每次在文件系统中创建一个文件时，都会为该文件分配一个 inode 。通常，与一个文件系统相关联的 inode 的数目足够多，但耗尽 inode 的可能性始终存在。

如果由于某种原因，**某个文件系统 inode 的使用率达到百分之百，那么您将无法在该文件系统中创建更多的文件、设备、目录等等**。对于这种情况，一种解决方案是通过 `smitty chfs` 命令**为该文件系统添加更多的空间**。

## dentry

dentry的中文名称是目录项，是Linux文件系统中某个索引节点(inode)的链接。这个索引节点可以是文件的，也可以是目录的。

dentry对象存在于三个双向链表中:
所有未用的目录项 dentry_unused 链表，正在使用的目录项对应inode的 i_dentry 链表，表示父子目录结构的链表。另外,还有一个重要的链表: inode_hashtable(这个暂不介绍)。

## dentry cache

每个文件都要对应一个inode节点与至少一个dentry项。

为了避免资源浪费，VFS采用了dentry cache的设计。

**当有用户用ls命令查看某一个目录或用open命令打开一个文件时，VFS会为这里用的每个目录项与文件建立dentry项与inode，即“按需创建”**。然后维护一个LRU（Least Recently Used）列表，当Linux认为VFS占用太多资源时，VFS会释放掉长时间没有被使用的dentry项与inode项。

## 进程对文件的管理

进程控制块task_struct中有两个变量与文件有关：fs与files。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/8.jpg)

**files中存储着root与pwd两个指向dentry项的指针**。用户确定路径时，绝对路径会通过root进行定位；相对路径会的通过pwd进行定位。（**一个进程的root不一定是文件系统的根目录。比如ftp进程的根目录不是文件系统的根目录，这样才能保证用户只能访问ftp目录下的内容**）

fs是一个file object列表，其中每一个节点对应着一个被打开了的文件。当进程定位到文件时，会构造一个file object，并**通过f_inode 关联到inode节点**。**文件关闭时（close），进程会释放对应对应file object**。File object中的f_mode是打开时选择的权限，f_pos为读写位置。**当打开同一个文件多次时，每次都会构造一个新的file object**。每个file object中有独立的f_mode与f_pos。

## open的过程

首先建立一个文件管理结构，如下图所示，该进程已经打开了两个文件。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/9.jpg)

接下来我们再打开一个新文件。

### 第一步：找到文件

找到了inode节点也就找到了文件。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/10.jpg)

### 第二步：建立file object

建立一个新的file object对象，放入file object对象列表，并把它指向inode节点。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/11.jpg)

### 第三步：建立file descriptor

file descriptor（文件描述符）就是进程控制块task_struct中files中维护的fd_array。因为是数组，所以file descriptor实际上已经预先分配好空间了，这里这是需要把某个空闲的file descriptor与file object关联起来。这个file descriptor在数组中的索引号就是open文件时得到的文件fd。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/12.jpg)

同一个文件是可以open多次的，结构如下图所示。每次open都会建立一个新的file descriptor与file object。然后指向同一个文件的inode节点。下图中，假设open的文件与fd1指向的是同一个文件，则新创建的file object 2与fd1的file  object 2会指向同一个inode2节点。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/13.jpg)

## dup

Linux还提供了dup功能，用于复制file descriptor。使用dup不会建立新的file object，所以新建立的file descriptor会与原filedescriptor同时指向同一个file object。下图中，我们通过dup(fd1)得到了fd2，则fd2与fd1指向了同一个file object2。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/14.jpg)

**两次open后由于会生成新的object，所以文件读写属性、文件读写位置（f_pos，这个变量是存在文件对象中的）等信息都是独立的**。使用dup复制file descriptor后，由于没有独立的object，所以修改某个fd的属性或文件读写位置后，另一个fd也会随之变化。

## Fork对打开文件的影响

fork一个子进程与Dup的的操作类似。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/15.jpg)

使用fork后的结构如下。

![](http://oklbfi1yj.bkt.clouddn.com/VFS/16.jpg)

同样是**没有创建新的file object**。**因此当对parent process中的fd1进行文件指针的移动时（如读写），child process中的fd1也会受影响**。**也就是说opened files list不是进程的一部分，因此不会被复制**。Opened files list应该是一个全局性的资源链表，**进程维护的是一个指针列表fd table，所以被复制的只是指针列表fd table，而不是opened files list**。











