## 第四章:首次登陆与线上求助

### 4.1首次登录系统

#### 4.1.3 x window与文字模式的切换

- linux默认提供6个 terminal
- 切换方式为 [Ctrl] + [Alt] + [F1]~[F6]
- 系统将 [F1]~[F6]命令为 tty1~tty6
- `startx`启动窗口界面，任何人都可以执行
- `systemctl setdefault graphical.target`将图形化界面设为默认
- `exit`登出系统

### 4.2 文字模式下指令的下达

#### 4.2.1 开始下达指令

```shell
# 格式
[linux@name ~]$ command [-options] parameter1 parameter2 ...
```

0. 一行指令中第一个输入的部分是  [指令 command] 或 [可执行档案 例: 可执行脚本 scrpit]
1. command为指令的名称
2. 中括号[]位置是加入选项设定，通常选项前会带 `-` 号，如 `-h`；也可以是完整指令，选项前带 `—`号，如 `--help`
3. parameter1 parameter2.. 为依附在选项后面的参数，或者是 command的参数
4. 指令，选项，参数之间用空格来区分，不论空几格shell都视为一格。空格是很重要的特殊字元
5. 按下 Enter按键后，该指令就立即执行。它代表着一行指令的开始启动
6. 指令太长，可以使用反斜杠 `/`来跳脱 Enter符号
7. linux系统区分 英文字母大小写

语系支持

- `locale`显示目前所支持的语系
- `LANG=en_US.utf8`、`export LC_ALL=en_US.utf8`，LANG只与输出信息有关系。若需要更改其他不同信息，还要同步更新LC_ALL

#### 4.2.2 基础指令的操作

- `date`显示日期与时间
- `cal` 显示日历
- `bc` 计算器

### 4.3 man page与 Info page

- man page

  | 代号 | 代表内容                                                     |
  | ---- | ------------------------------------------------------------ |
  | 1    | 用户可以在shell中操作的指令或可执行文件                      |
  | 2    | 系统核心可调用的函数与工具                                   |
  | 3    | 一些常用的函数(function)或函数库(library)，大部分为C的函数库(libc) |
  | 4    | 设备文件的说明，通常在/dev下的文件                           |
  | 5    | 配置文件或某些文件的格式                                     |
  | 6    | 游戏                                                         |
  | 7    | 惯例与协议，如Linux文件系统、网络协定、ASCII code            |
  | 8    | 系统管理员可用的管理指令                                     |
  | 9    | 跟 kernel有关的文件                                          |

### 4.4 文本编辑器：nano

`^`代表 `Ctrl`按键

### 4.5 正确关机方法

- `who` 查看目前有谁在线上
- `netsat -a` 查看网络的连线状态
- `ps -aux` 查看后台执行的程序
- `sync` 将数据同步写入硬盘中
- `shutdown` 常用关机指令
- `reboot、halt、poweroff` 重新开机，关机
- `shutdown` 在關機時會把系統的服務都關閉之後，才關閉電腦，而 `halt`、 `poweroff`指令則允許不管系統的狀態為何，直接停止電腦的運作
- `shutdown`、`reboot`、`halt`等指令均已经执行过 `sync`了

## 第五章:linux的文件权限与目录配置

- 文件的可存取身份：owner/group/others
- 三种身份各有 read/write/execute等权限

### 5.1 使用者与群组

linux使用者身份与群组记录的文件

- `/etc/passwd`这个文件中，记录着所有的系统上的账号与一般身份者及root的相关信息
- `/etc/shadow`记录着个人的密码
- `/etc/group` 记录着所有的群组名称

### 5.2 Linux 文件权限概念

#### 5.2.1 文件属性

```
					  连接数		 所属群组			创建时间或更新时间	
-rw-r--r--.  1  root  root  1864   May 4 18:01  initial-setup-ks.cfg
档案类型权限      owner			 档案容量										档案名字
```

```
	 -     r w x   	r w x  	---
档案类型 owner权限 群组权限 其他人的权限
r：可读     4
w: 可写     2
x: 可执行   1
```

- 第一栏是 文件的类型和权限
  - 第一字符代表文件的类型：目录[d]、文件[-]、链接文件[|]、设备文件里的可供储存的设备(可随机存取设备)[b]、设备文件里的序列埠设备(键盘等，一次性读取设备)[c]
  - 剩下的分别为 owner/group/others的权限
- 第二栏是 多少文件链接到此节点(i-node)
- 第三栏是 表示文件的owner
- 第四栏是 所属群组
- 第五栏是 文件size
- 第六栏是 文件的创建日期或 最近的修改日期
- 第七栏是 文件名

#### 5.2.2 如何改变文件属性与权限

- `chgrp` 改变文件所属群组
  - chgrp [ -R ] dirname/filename
  - -R 递归(recursive)执行
  - eg: chgrp users initial-setup-ks.cfg
- `chown` 改变文件拥有者
  - chown [ -R ] 账号名称  文件或目录 
  - chown [ -R ] 账号名称:群组名称  文件或目录
  - chown [ -R ] 账号名称.群组名称  文件或目录
  - eg: chown root:root initial-setup-ks.cfg
- `chmod` 改变文件的权限，SUID,SGID,SBIT等特性
  - chmod [ -R ] xyz  文件或目录。xyz为数字类型的权限属性(r: 4,w: 2,x: 1) 
  - chmod | u(user|owner) g(group) o(others) a(all) | + (加入) -(去除) =(设置) | r w x | 文件或目录
  - eg: 
    - chmod 777 .bashrc
    - chmod u=rwx,go=rx .bashrc
    - chmod a+w .bashrc
    - chmod a-x .bashrc

#### 5.2.3 目录与文件之权限意义

| 元件 | 内容          | 叠代物件   | r            | w            | x                     |
| ---- | ------------- | ---------- | ------------ | ------------ | --------------------- |
| 文件 | 详细数据 data | 文件数据夹 | 读到文件内容 | 修改文件内容 | 执行文件内容          |
| 目录 | 文件名        | 可分类抽屉 | 读到文件名   | 修改文件名   | 进入该目录的权限(key) |

#### 5.2.4 Linux文件种类与扩展名

- 文件种类
  - 正规文件(regular file)
    - 纯文本文件(ASCII)
    - 二进制(binary)
    - 数据格式文件(data)
  - 目录(directory)
  - 链接文件(link)
  - 设备与设备文件(device)
    - 区块(block)
    - 字符(character)
  - 数据接口文件(sockets)
  - 数据输送档(FIFO，pipe)
- 文件拓展名
  - 没有所谓的拓展名，全靠 权限的`x`来表示是否能执行
  - 常用拓展名
    - `.sh` 脚本或批处理文件(script)
    - `Z`，`.tar`，`.tar.gz`，`.zip` ，`.tgz` 经过打包的压缩文件
    - `.html`、`.php` 网页相关文件
- 文件长度限制
  - 单一文件或目录的最大容许文件名为 255 Bytes
  - 相当于 ASCII英文 255个字符
  - 相当于 中文(每个字符 2Bytes)，128个中文字
- 文件名称限制
  - 避免使用```?><;&![]|\'"`(){}```
  - `.`开头的文件为隐藏文件
  - 避免使用 `+ -`

### 5.3 Linux 目录配置

#### 5.3.1 目录配置的依据 — FHS

- FHS将目录分为四种交互作用的形态

  |                    | 可分享的(shareable)        | 不可分享的(unshareable) |
  | ------------------ | -------------------------- | ----------------------- |
  | 不变的(static)     | /usr (软件放置处)          | /etc (配置文件)         |
  |                    | /opt (第三方协力软件)      | /boot (开机与核心档)    |
  | 可变动的(variable) | /var/mail (使用者邮件信箱) | /var/run (程序相关)     |
  |                    | /var/spool/news (新闻群组) | /var/lock (程序相关)    |

- FHS定义的三个主目录

  - `/`(root，根目录)：与开机系统有关
    - 根目录是最重要的目录
    - 所有的目录都是由根目录衍生出来的
    - 与开机/还原/系统修复等有关
  - `/usr`(unix software resource): 与软件安装/执行有关
    - `/usr`放置的数据属于可分享与不可变动的(shareable, static)
    - 是 Unix Software Resource(Unix操作系统软件资源)所放置的目录
  - `/var`(variable): 与系统运行过程有关
    - `/var`目录主要针对常态性变动的文件，包括高速缓存(cache)、登录文件(log file)
    - 还有某些软件运行所产生的文件，包括程序文件(lock file, run file)，或者 MySQL数据库文件

#### 5.3.2 目录树 (directory tree)

- 目录树的起始点为根目录(/, root)
- 每一目录不止能使用本地端的 partition文件系统，可以用使用网络上的filesystem
- 每个文件在此目录树中的文件名(含完整路径)都是独一无二的

## 第六章:Linux文件与目录管理

### 6.1 目录与路径

#### 6.1.1 相对路径与绝对路径

#### 6.1.2 目录的相关操作

- `cd` 变化目录(change directory)
- `pwd` 显示当前的目录
- `mkdir` 创建一个新的目录
- `rmdir` 删除一个空的目录

#### 6.1.3 关于可执行文件路径的变量: $PATH

- `echo $PATH` 显示当前的 PATH
- 这个变量的内容是由一堆目录所组成的
- 每个目录之间用冒号`:`来隔开
- 每个目录是有顺序之分的
- 不同身份使用者默认的PATH不同，能够执行的指令也不同
- PATH可以被修改
- 使用绝对路径或相对路径直接执行某个指令的文件名，会比搜寻的PATH正确
- 指令应该放到正确的目录下
- 本目录(.)最好不要放到PATH中

### 6.2 文件与目录管理

#### 6.2.1 文件与目录的检视: ls

`ll`在大多数distribution的默认情况下都是 `ls -l`的意思

#### 6.2.2 复制、删除、移动: cp、rm、mv

- `cp` 复制文件或目录
- `rm` 移除文件或目录
- `mv` 移动文件与目录，或更名
  - 使用 `-u`来测试新旧文件，看是否需要搬运

#### 6.2.3 取得路径的文件名称与目录名称

- `basename` 取得最后的文件名
- `dirname` 取得的目录名

### 6.3 文件内容查阅

- `cat` 由第一行开始显示文件内容
- `tac` 由最后一行开始显示
- `nl` 显示的时候，输出行号
- `more` 分页显示文件内容
- `less` 分页显示文件内容，可向前翻页
- `head` 只看头几行
- `tail` 只看最后几行
- `od` 以二进制的方式读取文件内容

#### 6.3.1 直接检视文件内容

- `cat` Concatenate(连续)的简写
  - 将一个文件的内容连续打印在屏幕上
  - `cat [-AbEnTv]`
- `tac` 由最后一行到第一行显示在屏幕上
- `nl` 添加行号打印
  - `nl [-bnw]` 文件

#### 6.3.2 可翻页检视

- `more` 分页翻动
  - 空格键：向下翻页
  - Enter：向下翻一行
  - /字串：向下搜索输入的字符
  - `:f`：立刻显示出文件名及目前显示的行数
  - `q`：退出
  - `b或[ctrl]-b`：向回翻页，只对文件有效，对管线无效
- `less` 分页翻动，man指令就是调用`less`来显示说明文档
  - 空格键：向下翻页
  - `[pagedown]`：向下翻页
  - `[pageup]`：向上翻页
  - `/字串`：向下搜索输入的字符
  - `?字串`：向上搜索输入的字符
  - `n`：重复前一次搜索
  - `N`：反向的重复前一次搜索
  - `g`：前进到数据的第一行
  - `G`：前进到数据的最后一行
  - `q`：退出

#### 6.3.3 数据撷取

- `head` 取出前面几行
  - head [-n number] 文件
  - -n 后面接数字，代表显示几行
  - 默认显示前十行
  - eg: head -n 20 /etc/man_db.conf
  - eg: head -n -100 /etc/man_db.conf (除了最后100行，其他都显示)
- `tail` 取出最后几行
  - tail [-n number] 文件
  - -n  后面数字，代表显示几行
  - -f  持续侦测后面的文件
  - 默认显示最后十行
  - eg: tail -n 20 /etc/man_db.conf
  - eg: tail -n +100 /etc/man_db.conf (只列出100行之后的内容)

#### 6.3.4 非纯文本文件：od

- `od [-t TYPE] 文件`
- 可执行文件通常是 binary file，od读取不会出现乱码

#### 6.3.5 修改文件时间或创建新文件：touch

- 三个主要的时间参数
  - `modification time (mtime)` 文件的内容数据变更时，更新该时间
  - `status time (ctime)` 文件的状态（权限或属性）变更时，更新该时间
  - `access time (atime)` 文件的内容被取用时，更新该时间
- `touch [-acdmt] 文件`
  - -a 仅修改 atime
  - -c 仅修改 ctime，文件不存在则不创建新文件
  - -d 后面可以接欲修订的日期而不用当前的日期，等同 `—date="日期或时间"`
  - `-m` 仅修改 mtime
  - -t 后面可以接欲修订的日期而不用当前的日期，格式为 `[YYYYMMDDhhmm]`
- 使用场景
  - 创建一个 空文件
  - 将某个文件日期修改为当前 (mtime和 atime)

### 6.4 文件与目录的默认权限与隐藏权限

#### 6.4.1 文件默认权限：umask

unmaks就是指定 目前使用者再创建文件或目录时候的权限默认值

- 查阅的方式
  - `umask`
  - `umask -S(Symbolic)`
- 设置的方式
  - 直接 umask number
  - eg: `umask 002`

#### 6.4.2 文件隐藏属性

- `chattr` 设置文件隐藏属性
  - `chattr [+-=][ASacdistu] 文件或目录名称` 
- `lsattr` 显示文件隐藏属性

#### 6.4.3 文件特殊权限：SUID,SGID,SBIT

- Set UID
  - 当 s 出现在文件拥有者的 x权限上时，就被称为Set UID，简称为 SUID的特殊权限
  - 限制与功能
    - SUID权限仅对二进制程序(binary program)有效，不能用在 shell script上
    - 执行者对于该程序需要有 x权限
    - 只在执行该程序的过程中有效(run-time)
    - 执行者将具有该程序的拥有者(owner)权限
- Set GID
  - 当 s 出现在群组的 x权限上时，就被成为Set GID，SGID
  - 功能
    - SGID对二进制程序有用
    - 可以是针对文件或目录进行设置
    - 程序执行者对于该程序来说，需要具备 x权限
    - 执行者在执行的过程中将获得该程序群组的支持
- Sticky Bit
  - SBIT 只针对目录有效，对文件没有效果
  - 对目录的作用
    - 当使用者对于此目录具有w,x权限，亦具有写入的权限
    - 当使用者在该目录下创建文件或目录时，仅有自己与 root才有权利删除
- SUID/SGID/SBIT 权限设置
  - 三个数字组合
    - 4 为 SUID
    - 2 为 SGID
    - 1 为 SBIT
  - eg: `chmod 4755 filename` , 4 为特殊权限的值之和
  - 大写的S/T，具有空的SUID/SGID权限

#### 6.4.4 观察文件类型: file

- 可以知道某个文件的基本数据
- 该文件没有使用到动态函数库(share library)
- eg: `file ~/.bashrc`

### 6.5 指令与文件的搜寻

#### 6.5.1 指令文件名的搜寻

- `which` 寻找可执行文件
- `which [-a] command`
- `-a` 将所有由 PATH目录中可以找到的指令列出，而不止第一个被找到的
- 根据 PATH环境变量所规范的路径，去搜寻可执行的文件名

#### 6.5.2 文件文件名的搜寻

- `whereis` 由一些特定的目录中寻找文件文件名
  - `whereis [-bmsu] 文件或目录名`
- `locate/updatedb`
  - `locate [-ir] keyword`
  - 寻找的数据由 已创建的数据库/var/lib/mlocate/ 里面的数据所搜寻到，不是直接搜索硬盘
  - 数据库的创建默认每天执行一次，有可能会找不到文件
  - `updatedb` 会读取 /etc/updatedb.conf这个配置，然后去搜寻文件名，最后更新数据库文件
- `find`
  - `find [PATH] [option] [action]`
  - 跟时间有关的选项
    - +4 代表大于等于 5天前的文件名：ex>find /var -mtime +4
    - -4 代表小于等于 4天内的文件文件名：ex>find /var -mtime -4
    - 4 代表 4-5那一天的文件文件名：ex>find /var -mtime 4
  - 跟使用者或群组名称有关的参数
  - 与文件权限及名称有关的参数

## 第七章:Linux磁盘与文件系统的管理

### 7.1 认识Linux文件系统

#### 7.1.1 磁盘组成与分区的复习

- 磁盘组成
  - 原型的盒片（记录数据）
  - 机械手臂及磁头（读写盘片）
  - 主轴马达，转动盘片
- 磁盘的文件名
  - 所有的文件名被仿真为`/dev/sd[a-p]`，第一颗磁盘文件名为 `/dev/sda`
  - 分区的文件名为 `/dev/sda[1-128]`
  - 虚拟机的磁盘通常为 `/dev/vd[a-p]`
  - 软件磁盘阵列为 `/dev/md[0-128]`
  - 使用的是LVM时，文件名为 `/dev/VGNAME/LVNAME`等格式
  - `/dev/sd[a-p][1-128]`：为实体磁盘的磁盘文件名
  - `/dev/vd[a-d][1-128]`：为虚拟磁盘的磁盘文件名

#### 7.1.2 文件系统特性

- 磁盘分区，进行格式化之后，成为操作系统能够使用的 文件系统格式(filesystem)
  - LVM，软件磁盘阵列(software raid)将一个分区格式化为多个文件系统
  - LVM，PAID可以将多个分区合成一个文件系统
- 文件系统通常会将文件权限和文件属性分别到存放到不同的区块，权限与属性放置在 inode中,实际数据放置到 data block区块中，还有一个超级区块(superblock)会记录整个文件系统的整体信息
  - `superblock`：记录filesystem整体信息，包括inode/block的总量、使用量、剩余量，以及文本系统的格式与相关信息
  - `inode`：记录文件的属性，一个文件占用一个inode，同时记录此文件的数据所在的block号码
  - `block`：实际记录文件的内容，若文件太大时，会占用多个block
- 索引式文件系统（indexed allocation）
- 磁盘重组

#### 7.1.3 Linux的 EXT2文件系统(inode)

Ex2在格式化的时候基本上是区分为多个区块群组（block group）的，每个区块群组都有独立的 inode/block/superblock系统，每个区块群组有6个主要内容：

- `data block` 数据区块
  - 原则上，除了重新格式化，block的大小与数量在格式化完都不能改变了
  - 每个block内最多只能放一个文件的数据
  - 如果文件大于 block的大小，则一个文件会占用多个block
  - 若文件小于block，则剩余的容量不能再被使用
- `inode table` inode表格
  - inode记录的文件数据
    - 该文件的存取模式（read/write/excute）
    - 该文件的owner和group
    - 文件的容量
    - 文件的创建或状态改变的时间（ctime）
    - 最近一次的读取时间（atime）
    - 最近修改的时间（mtime）
    - 定义文件特性的flag，如SetUID...
    - 该文件真正内容的指向（pointer）
  - 特性
    - 每个 inode大小固定为 128Bytes(新的ext4和xfs可设置为 256B)
    - 每个文件仅占用一个 inode
    - 文件系统能够创建的文件数量与 inode的数量有关
    - 系统先找到 inode，分析inode中的权限是否符合使用者，符合的话，才开始实际读取block
- `Superblock` 超级区块
  - Superblock是记录整个filesystem相关信息的地方，记录的信息：
    - block与 inode的总量
    - 未使用与已使用的 block与 inode数量
    - block(1,2,4K)与 inode(128,256Bytes)的大小
    - filesystem的挂载时间、最近一次写入数据的时间、最近一次检验磁盘（fsck）的时间等
    - 一个 vaild bit数值，若文件系统被挂载，vaild bit为0，否则是1
  - 每个block group都可能有superblock，其他的是第一个的备份
- `Filesystem Descripition` 文件系统描述说明
  - 这个区段描述每个block group的开始和结束的block号码
  - 说明每个区段分别介于哪个block号码之间
- `block bitmap` 区块对照表
  - 得知哪个是空的 block
  - 删除文件后，相关block被释放，bitmap中的该block号码被标记为 未使用
- `inode bitmap` inode对照表
  - 记录使用与未使用的 inode号码
- `dumpe2fs`：查询 Ext家族 superblock信息的指令

#### 7.1.4 与目录树的关系

- 目录
  - 创建一个目录，文件系统会分配一个inode和一个block，其中 inode记录该目录的相关权限与属性及分配的block号码，而block中则是记录这个目录下的文件名与该文件名占用的inode号码数据
  - `ls -i` 来显示文件所占用的 inode号码
- 文件
  - inode 仅有12个直接指向
  - 一个间接
  - 一个双间接
  - 一个三间接记录区
- 目录树读取
  - 由于文件名的记录是在目录的block中，所以新增/删除/重命名与目录的w权限有关
  - 目录树由根目录开始读起

#### 7.1.5 EXT2/EXT3/EXT4 文件的存取与日志式文件系统的功能

- 新增文件的行为
  1. 确定使用者是否有 w和 x权限
  2. 根据inode bitmap找到没有使用的 inode号码，将新文件的权限/属性写入
  3. 根据block bitmap找到没有使用的 block号码，将实际数据写入block中，并将新的 inode的block指向数据
  4. 更新同步 inode bitmap, block bitmap, superblock
- 数据存放区域：inode table, data block
- 中介数据（metadata，涉及到操作，经常会变动）：superblock, block bitmap, inode bitmap
- 数据的不一致（inconsistent）状态
- 日志式文件系统(Journaling filesystem)
  - 预备：当系统要写入一个文件，会现在日志记录区块中记录某个文件准备写入的信息
  - 实际写入：开始写入文件的权限和数据，开始更新 metadata数据
  - 结束：完成数据与 metadata的更新，在日志记区块中完成该文件的记录

#### 7.1.6 Linux文件系统的运行

- 系统载入一个文件到内存后
- 该文件没有被更动过，被标记为 干净(clean)
- 被更动过，会被标记为 脏的(dirty)
- 所有动作暂不会被写入到磁盘
- 系统不定时将 dirty数据写回磁盘

#### 7.1.7 挂载点的意义(mount point)

- 挂载点一定是目录，该目录为进入该文件系统的入口
- 并不是任何文件系统都可以使用，必须挂载目录树的某个目录后，才能使用该文件系统

#### 7.1.8 其他Linux支持的文件系统与 VFS

- 常见支持的文件系统
  - 传统文件系统：ext2/minix/MS-DOS/FAT(用 vfat模块)/iso9960(光盘)
  - 日志式文件系统：ext3/ext4/ReiserFS/Window's NTFS/IBM's JFS/SGI's XFS/ZFS
  - 网络文件系统：NFS/SMBFS
- 查看linux支持和的文件系统，可查看这个目录
  - `ls -l /lib/modules/$(uname -r)/kernel/fs`
- 系统目前已载入内存中的文件系统
  - `cat /proc/filesystems`
- Linux VFS(Virtual Filesystem Switch)
  - 整个 Linux认识的 filesystem都是VFS 进行管理

#### 7.1.9 XFS 文件系统简介

- EXT家族当前：支持度最广，但格式化超慢
- XFS文件系统的配置
  - 数据区(data section)
    - 类似于 ext家族的 block group
    - block和inode有不同的容量进行设置
  - 文件系统活动登录区(log section)
    - 主要记录文件系统的变化
  - 实时运行区(realtime section)
    - 文件被创建时，xfs会在这个区段找到一/数个extent区块，将文件放置到这个区块后，等分配完，在写入到data section的 inode与 block
- XFS文件系统的描述数据观察
  - `xfs_info 挂载点 文件设备名` 

 ### 7.2 文件系统的简单操作

#### 7.2.1 磁盘与目录的容量

- `df` 列出文件系统的整体磁盘使用量
  - `df [-ahikHTm] [目录或文件名]`
  - 读取的范围主要是在 Superblock内的信息
  - 特别留意根目录的容量
- `du` 评估文件系统的磁盘使用量（常用在推估目录所占容量）
  - `du [-ahskm] 文件或目录名称`
  - 直接到文件系统内搜寻所有的文件数据

#### 7.2.2 实体链接与符号链接：ln

- Hard Link (实体链接，硬式链接或实际链接)
  - 只是在某个目录下新增一笔文件名链接到某 inode号码的关联记录
  - 一般情况下，使用hard link设置链接文件时，磁盘的空间与 inode的数目都不会改变
  - 不能跨 Filesystem
  - 不能 Link目录
- Symbolic Link (符号链接，亦即捷径)
  - 创建一个独立的文件，而这个文件会让数据的读取指向他link的那个文件的文件名
- `ln [-sf] 来源文件 目标文件`
- 关于目录的link数量

### 7.3 磁盘的分区、格式化、检验与挂载

新增磁盘步骤

1. 分区，创建可用的 partition
2. 对该 partition格式化(format)，创建可用的 filesystem
3. 可以选择对 filesystem进行检验
4. 创建挂载点(目录)，并挂载

#### 7.3.1 观察磁盘分区状态

- `lsblk` 列出系统中的所有磁盘列表
  - `lsblk [-dfimpt] [device]`
  - `-f` 列出文件系统与设备的 UUID数据
  - `MAJ:MIN` 主要、次要设备代码
- `blkid` 列出设备的UUID等参数
  - UUID是全域单一识别码(universally unique identifier)
- `parted` 列出磁盘的分区表类型与分区信息
  - `parted device_name print`

#### 7.3.2 磁盘分区：gdisk/fdisk

MBR分区表使用 fdisk分区，GPT分区表使用 gdisk分区

- `gdisk`
  - `gdisk 设备名称`
  - `w` 立即执行
  - `q` 离开
  - `partprobe` 更新 Linux核心的分区表信息
  - `partprobe [-s]` -s打印信息
  - `cat /proc/partitions` 核心的分区资料
- `fdisk`
  - `fdisk 设备名称`
  - `m` 获得指令介绍

#### 7.3.3 磁盘格式化（创建文件系统）

- XFS文件系统 mkfs.xfs
  - `mkfs.xfs [-b bsize] [-d parms] [-i parms] [-l parms] [-L label] [-f] [-r parms] 设备名称`
  - eg: `mkfs.xfs /dev/vda4`
  - `grep 'processor' /proc/cpuinfo` 找出系统的cpu核数
  - `mkfs.xfs -f -d agcount=2 /dev/vda4` 设置 agcount数值
- XFS文件系统 for RAID性能优化（optional）
  - 磁盘阵列（RAID）
- EXT4文件系统mkfs.ext4
  - 格式化为 ext4的传统 linux文件
  - `mkfs.ext4 [-b size] [-L label] 设备名称`
- 其他文件系统 mkfs
  - mkfs(make filesystem)是个综合指令
  - `mkfs[tab][tab]` 查看支持的文件系统的格式化功能

#### 7.3.4 文件系统检验

- `xfs_repair` 处理 XFS文件系统
  - `xfs_repair [-find] 设备名称`
- `fsck.ext4` 处理 EXT4文件系统
  - `fsck.ext4 [-pf] [-b superblock] 设备名称`

#### 7.3.5 文件系统挂载与卸载

- 挂载点是目录，这个目录是进入磁盘分区（文件系统）的入口，挂载前的检查：
  - 单一文件系统不应该重复被挂载在不同的挂载点(目录)中
  - 单一目录不应该重复挂载多个文件系统
  - 要做为挂载点的目录，理论应该是空目录
- `mount` 挂载指令
  - `mount -a`  依照配置文件[/etc/fstab]将磁盘挂载
  - `mount [-l]` 显示Label名称
  - `mount [-t 文件系统] LABEL='' 挂载点`
  - `mount [-t 文件系统] UUID='' 挂载点`
  - `mount [-t 文件系统] 设备文件名 挂载点`
- 哪些类型的 filesystem需要进行挂载测试，参考：
  - `/etc/filesystems` 系统指定的测试挂载文件系统类型的优先顺序
  - `/proc/filesystems` linux系统已经载入的文件系统类型
- linux支持的文件系统之驱动程序所在目录
  - `/lib/moduels/$(uname -r)/kernel/fs/`
- 挂载 xfs/ext4/vfat等文件系统
  - `blkid /dev/vda5` 找出 /dev/vda5的UUID
  - `mount UUID="~~" /data/ext4`  挂载
- 挂载 CD或 DVD光盘
- 挂载 vfat中文U盘（USB磁盘）
- 重新挂载根目录与挂载不特定目录
- `umount` 将设备文件卸载
  - `umount [-fn] 设备文件名或挂载点`

#### 7.3.6 磁盘/文件系统参数修订

修改一下目前文件系统的一些相关信息

- mknod 

  - `mknod 设备文件名 [bcp] [Major] [Minor]`

- `xfs_admin` 修改 XFS文件系统的UUID与Label name

  - `xfs_admin [-lu] [-L label] [-U uuid] 设备文件名`

- `tune2fs` 修改 ext4的 label name与UUID

  - `tune2fs [-l] [-L label] [-U uuid] 设备文件名`

### 7.4 设置开机挂载

#### 7.4.1 开机挂载 /etc/fstab及 /etc/mtab

- 系统挂载的限制
  - 根目录`/` 是一定要挂载的，还要先于其他 mount point挂载
  - 其他 mount point必须是已创建目录，要遵守 FHS
  - 所有的 mount point在同一时间内，只能挂载一次
  - 所有的 partition在同一时间内，只能挂载一次
  - 卸载前，需要将工作目录移到 mount point之外
- mount指令挂载时，将所有的选项写入`/etc/fstab`(filesystem table)
- `/etc/fstab`文件内容共有 6个字段
  - 磁盘设备文件名/UUID/LABEL name
  - 挂载点(mount point)
  - 磁盘分区的文件系统
  - 文件系统参数
  - 是否被 dump备份指令作用
  - 是否以 fsck检验扇区

####  7.4.2 特殊设备loop挂载

- 挂载光盘/DVD镜像文件
  - `mount -o loop /tmp/Centod-7.0-DVD.iso /data/centos_dvd`
- 创建大文件以制作loop设备文件
- 创建大型文件
  - `dd if=/dev/zero of=/srv/loopdev bs=1M count=512`
- 大型文件的格式化
  - `mkfs.xfs -f /srv/loopdev`
  - `blkid /srv/loopdev`
- 挂载
  - `mount -o loop UUID="7ddasfasdfaf" /mnt`
  - `df /mnt`

### 7.5 内存交换空间（swap）之创建

swap是 解决内存不足，可以暂时把内存中的程序拿到内存中暂放的内存交换空间

#### 7.5.1 使用实体分区创建swap

- 创建步骤
  1. 分区
  2. 格式化
  3. 使用
  4. 通过 free与 swapon -s观察内存的用量

#### 7.5.2 使用文件创建swap

1. 使用 dd指令新增一个 128MB的文件在 /tmp
   - `dd if=/dev/zero of=/tmp/swap bs=1M count=128`
2. 使用 mkswap将 /tmp/swap的文件格式化为 swap的格式
   - `mkswap /tmp/swap`
3. 使用 swapon将 /tmp/swap启动
   - `swapon /tmp/swap`
   - `swapon -s`
4. 使用 swapoff 关掉 swap file,并设置自启动

### 7.6 文件系统的特殊观察与操作

#### 7.6.1 磁盘空间之浪费问题

#### 7.6.2 利用 GNU的 parted进行分区行为（Optional）

- 同时支持 GPT和 MBR的分区指令 -> `parted`
- 可以在一行命令中完成分区
- `parted [设备] [指令 [参数]]`
- 列出目前本机的分区表数据
  - `parted /dev/vda print`

##  第八章:文件与文件系统的压缩，打包与备份

### 8.1 压缩文件的用途与技术

- 目前计算机系统中是使用所谓的 Bytes单位来计量
- 1Bytes = 8bits
- WWW网站压缩技术

### 8.2 Linux系统常见的压缩指令

- 压缩文件拓展名
  - `*.z`  compress程序压缩的文件
  - `*.zip`  zip程序压缩的文件
  - `*.gz`  gzip程序压缩的文件
  - `*.bzz`  bzip2程序压缩的文件
  - `*.xz`  xz程序压缩的文件
  - `*.tar`  tar程序打包的数据
  - `*.tar.gz`  tar程序打包的文件，经过 gzip压缩
  - `*.tar.bzz`  tar程序打包的文件，经过 bzip2压缩
  - `*.tar.xz`  tar程序打包的文件，经过 xz压缩

#### 8.2.1 gzip,zcat/zmore/zless/zgrep

- gzip可以解开 compress,zip,gzip等软件压缩的文件
- `gzip [-cdtv#] 文件名`
  - -c 将压缩的数据输出到屏幕上，可通过数据流重导向来处理
  - -d 解压缩
  - gunzip 解压缩
  - -t 可以用来检验一下压缩文件的一致性，检查错误
  - -v 可以显示元文件/压缩文件的压缩比等信息
  - -# #是数字，代表压缩等级，1~9(压缩比最好)，默认为6
- gzip 压缩的文件可以在 window系统上被解压
- zcat/zmore/zless 可以读取纯文本文件压缩后的文件
- zgrep 来搜寻压缩文件中的关键字

#### 8.2.2 bzip2,bzcat/bzmore/bzless/bzgrep

- bzip2为了取代 gzip来提供更好的压缩比
- `bzip2 [-cdkzv#] 文件名`
  - -k  保留原始文件
  - -z  压缩的参数，默认值
  - 其他参数类似 gzip的参数
- bzcat/bzmore/bzless 可以读取纯文本文件压缩后的文件
- bzgrep来搜寻压缩文件中的关键字

#### 8.2.3 xz,xzcat/xzmore/xzless/xzgrep

- xz比 bzip2的压缩比要更高，但时间更久
- `xz [-dtlkc#] 文件名`
  - -l 列出压缩文件的相关信息
  - 其他参数类似 gzip, bzip2的参数

### 8.3 打包指令: tar

将多个文件或目录包成一个大文件的指令功能就是打包指令

#### 8.3.1 tar

- 常用 tar命令及参数
  - `tar [-z | -j | -J] [cv] [-f 待创建的新文件名] filename`  打包
  - `tar [-z | -j | -J] [tv] [-f 既有的 tar文件名]`  察看
  - `tar [-z | -j | -J] [xv] [-f 既有的 tar文件名] [-C 目录]`  解压
  - `-c`  创建打包文件，可搭配 -v来查看过程中的被打包文件名
  - `-t`  查看打包文件的内容有哪些文件名
  - `-x`  解打包或解压缩，搭配 -C在特定目录解开
  - `-c, -t, -x` 不可同时出现在一行命令中
  - `-z`  通过 gzip来压缩或解压，文件名：*.tar.gz
  - `-j`  通过 bzip2来压缩或解压，文件名：*.tar.bz2
  - `-J`  通过 xz来压缩或解压，文件名：*.tar.xz
  - `-z, -j, -J` 不可同时出现在一行命令中
  - `-v`  在压缩或解压过程中，将正在处理的文件名显示出来
  - `-f filename`  -f后面直接接要被处理的文件名
  - `-C 目录`  在特定目录进行解压
  - `-p`  保留备份数据的原本权限与属性
  - `-P`  保留绝对路径，允许备份数据包含根目录
- 简单的使用方式
  - `tar -jcv -f filename.tar.bz2 要被压缩的目录或目录名称`  压缩
  - `tar -jtv -f filename.tar.bz2`  查询
  - `tar -jxv -f filename.tar.bz2 -C 欲解压的目录`  解压
- 打包某目录，但不包含该目录下的某些文件
  - `--exclude`  不包含
  - eg: `tar -jcv -f /root/system.tar.bz2 --exclude=/root/etc*`
- 仅备份比某个时刻还要新的文件
  - `--newer-mtime`  后续日期只包含 mtime
  - `--newer`  后续日期包含 mtime和 ctime
  - eg: `tar -jcv -f /root/etc.newer.then.passwd.tar.bz2 --newer-mtime="2019/08/13" /etc/*`
- 基本名称 tarfile, tarball
  - tar打包出来的文件没有进行压缩，叫作 tarfile
  - tar打包出来的文件有进行压缩，叫作 tarball
- 特殊应用: 利用管线命令与数据流
  - eg: `tar -cvf - /etc | tar -xvf -` 将 /etc的数据压缩并解压到 当前所在目录

### 8.4 XFS文件系统的备份与还原

xfs文件系统中，xfsdump和 xfsrestore

#### 8.4.1 XFS文件系统备份 xfsdump

- xfsdump 可以进行文件系统的完整备份(full backup)，还可以进行累积备份(Incremental backup)，备份有变化的文件
- 限制
  - xfsdump只备份已挂载的文件系统
  - 只有 root权限才能操作
  - 只能备份 XFS文件系统
  - 备份下来的数据只能通过 xfsrestore解析
  - 可以通过文件系统的 UUID来分辨备份文件，不能备份两个相同 UUID的文件系统
- 指令
  - `xfsdump [-L S_label] [-M M_label] [-l #] [-f 备份文件] 待备份数据`
  - `xfsdump -I` 从 /var/lib/xfsdump/inventory 列出目前备份的信息状态

#### 8.4.2 XFS文件系统还原 xfsrestore

- xfsdump的复原使用的是 xfsrestore指令
- `xfsrestore -I`  查看备份文件数据
  - xfsfdump和 xfsrestore都会从 /var/lib/xfsdump/inventory/里面取数据
- `xfsrestore [-f 备份文件] [-L S_label] [-s] 待复原项目` 
  - 需要被复原的那个文件
  - 该文件的 session label name
  - eg: `xfsrestore -f /srv/boot.dump -L boot_all /boot`
  - 只想要复原某个目录或文件，直接加上 `-s 目录`
- `xfsrestore [-f 备份文件] -r 待复原文件`  通过累积备份文件来复原系统
  - 如果备份数据是由 level0 -> level1 -> level2...去进行的
  - 复原也需要相同的流程
- `xfsrestore [-f 备份文件] -i 待复原文件`  进入互动模式
  - `-s` 可以接部分数据还原
  - `-i` 可以以互动的形式来还原

### 8.5 光盘写入工具

文字模式的烧录行为的通常做法

- 利用 mkisofs指令先将文件创建成为 镜像文件(iso)
- 利用 cdrecord指令来将镜像文件烧录至光盘或DVD

#### 8.5.1 mkisofs: 创建镜像文件

#### 8.5.2 cdrecord: 光盘烧录工具

### 8.6 其他常见的压缩与备份工具

#### 8.6.1 dd

- dd 不只是制作一个文件，更重要的是在 备份
- 可以读取磁盘设备的内容（几乎是直接读取扇区 sector）
- 没有用到的扇区也会写入备份文件，xfsdump只会备份使用到的部分
- 做出来的文件跟原本的数据大小一样
- 新分区的 partition不需要格式化，因为 dd直接将 sector表面的数据都复制来了

#### 8.6.2 cpio

- cpio可以备份任何东西，包括设备文件
- 不会主动的去找文件备份
- 配合 find等可以找到文件名的指令来使用
- `cpio -ovcB > [file|device]`  备份
- `cpio -ivcdu < [file|device]`  还原
- `cpio -ivct < [file|device]`  查看
- eg: `find boot | cpio -ocvB > /tmp/boot.cpio` 

## 第九章:vim程序编辑器

- vim是一种程序编辑器

### 9.1 vi与vim

- 在 Linux中的大部分配置文件都是以 ASCII的纯文本形态存在的

#### 9.1.1 为何要学 vim

- 使用的原因
  - 所有的 Unix Like系统都内置 vi
  - 很多软件的编辑接口都主动调用 vi
  - vim具有程序编辑的能力，能用字体颜色分辨语法的正确性
  - 运行速度快，程序简单
- vim是 vi的进阶版本

### 9.2 vi的使用

- 一般指令模式(command mode)
- 编辑模式(insert mode)
  - `i,I,o,O,a,A,r,R` 进入编辑模式
  - Esc退出
- 命令行命令模式(command-line mode)

#### 9.2.1 简易执行范例

1. 使用 “vi filename”进入一般指令模式
   - eg: `/bin.vi welcome.txt`
   - 在 centos7中，一般账号 vim取代了 vi，所以使用绝对路径执行
2. 按下 i进入编辑模式，开始编辑文字
3. 按下 [Esc]回到一般指令模式
4. 进入命令行界面，文件储存并离开 vi环境

#### 9.2.2 按键说明

- 第一部分：一般指令模式可用的按键

| 移动光标的方法        |                                            |
| --------------------- | ------------------------------------------ |
| h或 <-                | 向左移动一个字符                           |
| j或 向下方向键        | 向下移动一个字符                           |
| k或 向上方向键        | 向上移动一个字符                           |
| l或 ->                | 向右移动一个字符                           |
| Ctrl + f              | 向下翻一页                                 |
| Ctrl + b              | 向上翻一页                                 |
| Ctrl + d              | 向下翻半页                                 |
| Ctrl + u              | 向上翻半页                                 |
| +                     | 光标移动到非空白字符的下一列               |
| -                     | 光标移动到非空白字符的上一列               |
| n<space>              | 移动到光标所在行的第几个字符               |
| 0或[home]             | 移动到这一列的最前面字符                   |
| $或[End]              | 移动到这一列的最后面字符                   |
|                       | 光标移动到屏幕最上方列的第一个字符         |
| M                     | 光标移动到屏幕中央列的第一个字符           |
| L                     | 光标移动到屏幕最下方列的第一个字符         |
| G                     | 光标移动到这个文件的最后一列               |
| nG                    | 移动到文件的第 n列                         |
| gg                    | 移动到文件的第一列                         |
| n<Enter>              | 向下移动 n列                               |
| /word                 | 向光标之下寻找字符串                       |
| ?word                 | 向光标之上寻找字符串                       |
| n                     | 英文按键，重复前一次向上的寻找动作         |
| N                     | 英文按键，重复前一次向下的寻找动作         |
| :n1,n2s/word1/word2/g | n1,n2为数字,代表行数,搜寻word1,用word2替换 |
| :1,$s/word1/word2/g   | 从第一行搜寻到最后一行，并替换             |
| :1,$s/word1/word2/gc  | 搜寻并提示用户是否取代                     |
| x,X                   | x向后删除一个字符，X向前删除一个字符       |
|                       | n为数字，连续向后删除n个字符               |
| dd                    | 删除光标所在行                             |
| ndd                   | n为数字，删除光标所在的向下n行             |
| d1G                   | 删除光标所在到第一行的所有数据             |
| dG                    | 删除光标所在到最后一行的所有数据           |
| d$                    | 删除光标所在到该行的最后一个字符           |
| d0                    | 数字0，删除光标所在到该行的第一个字符      |
| yy                    | 复制光标所在行                             |
| nyy                   | 复制光标所在的向下 n行                     |
| y1G                   | 复制光标所在到第一行的所有数据             |
| yG                    | 复制光标所在到最后一行的所有数据           |
| y0                    | 复制光标所在字符到行首的数据               |
| y$                    | 复制光标所在字符到行尾的数据               |
| p,P                   | p在下一行粘贴，P在上一行粘贴               |
| J                     | 合并光标所在行与下一行                     |
| c                     | 重复删除多个数据，如10cj,向下删除10行      |
| u                     | 复原上一个操作                             |
| Ctrl + r              | 重做上一个操作                             |

- 第二部分：一般指令模式切换到编辑模式的可用按钮说明

| 进入插入或取代的编辑模式 |                                                          |
| ------------------------ | -------------------------------------------------------- |
| i,I                      | 进入 insert mode，在光标处开始输入                       |
| a,A                      | 进入 insert mode，在光标所在行的最后一个字符开始输入     |
| o,O                      | 进入 insert mode，在光标所在行的下面插入一个新行开始输入 |
| r,R                      | 进入 insert mode，取代光标处的字符，开始输入             |
| [Esc]                    | 退出 insert mode，进入一般指令模式                       |

- 第三部分: 一般指令模式切换到命令行界面的可用按钮说明

| 命令行界面的储存、离开等指令 |                                          |
| ---------------------------- | ---------------------------------------- |
| :w                           | 储存数据                                 |
| :w!                          | 强制储存数据，跟权限有关系               |
| :q                           | 离开 vi                                  |
| :wq                          | 储存数据并退出，:wq!强制退出，不保留修改 |
| ZZ                           | 有更改就储存退出，没有更改就不储存但退出 |
| :w[filename]                 | 另存为 新文件                            |
| :r[filename]                 | 读入另一个文件的数据，添加到光标后面     |
| :n1,n2 w[filename]           | 截取 n1,n2的内容储存成新文件             |
| :! command                   | 强制执行命令                             |
| :set nu                      | 显示行号                                 |
| :set nonu                    | 不显示行号                               |

#### 9.2.4 vim的暂存盘、救援回复与打开时的警告讯息

- 用 vim编辑时，vim会在被编辑的文件目录下，创建一个 .filename.swp的文件
- 该文件会记录 对所编辑文件的动作
- 可以用作救援功能

### 9.3 vim的额外功能

#### 9.3.1 区块选择(Visual Block)

| 区块选择的按键意义 |                                      |
| ------------------ | ------------------------------------ |
| v                  | 字符选择，会将光标经过的地方反白选择 |
| V                  | 行选择，会将光标经过的行反白选择     |
| Ctrl + v           | 区块选择，可以用长方形的方式选择数据 |
| y                  | 将反白的地方复制起来                 |
| d                  | 将反白的地方删除掉                   |
| p                  | 将刚刚复制的区块，在光标处贴上       |

#### 9.3.2 多文件编辑

| 多文本编辑的按键 |                            |
| ---------------- | -------------------------- |
| :n               | 编辑下一个文件             |
| :N               | 编辑上一个文件             |
| :files           | 列出目前 vim打开的所有文件 |

#### 9.3.3 多窗口功能

| 多窗口情况下的按键功能 |                                |
| ---------------------- | ------------------------------ |
| :sp filename           | 打开有个新窗口，后面是文件名   |
| Ctrl + w + j/向下箭头  | 分别按下对应按键移动到下方窗口 |
| Ctrl + w + k/向上箭头  | 分别按下对应按键移动到上方窗口 |
| Ctrl + w + q           | 关闭保存 下方窗口              |

#### 9.3.4 vim的挑字补全功能

| 组合按钮             | 补齐的内容                                                 |
| -------------------- | ---------------------------------------------------------- |
| Ctrl + x -> Ctrl + n | 通过目前正在编辑的这个“文件的内容文字”作为关键字，予以补齐 |
| Ctrl + x -> Ctrl + f | 以当前目录内的“文件名”作为关键字，予以补齐                 |
| Ctrl + x -> Ctrl + o | 以拓展名作为语法补充，以vim内置的关键字，予以补齐          |

#### 9.3.5 vim环境设置与记录:~/.vimrc,~/viminfo

- `~/viminfo`
  - 记录曾经做过的行为
- `~/.vimrc`
  - 默认不存在，需自行创建
  - 个人的配置项
- `/etc/vimrc`
  - 全局的 vim配置项

#### 9.3.6 vim常用指令示意图

- 一般模式
- 编辑模式
- 指令模式

## 第十章:认识与学习 BASH

### 10.1 认识 BASH这个 Shell

#### 10.1.1 硬件、核心与Shell

用户 --->  使用者界面(Shell, KDE, application)  --->  核心(kernel)  --->  硬件(Hardware)

#### 10.1.2 为何要学命令行的shell

- 各家 distributions使用的 bash都一样
- 远端管理：命令行的传输速度比较快

#### 10.1.3 系统的合法shell与/etc/shells功能

- Shell版本
  - Bourne Shell(sh)
  - Sun的 C Shell
  - 商业上的 K Shell
  - TCSH
  - Bourne Again Shell(bash)
- /etc/shells 可以看到可用的 shells 

#### 10.1.4 Bash shell的功能

- 命令编修能力(history)
  - 默认记录 1000个
  - 在 .bash_history中存储
  - 这一次的命令行会暂存在内存中
- 命令与文件补全功能( [tab]按键的好处)
  - 命令补全
  - 文件补齐
- 命令别名设置功能(alias)
  - `alias lm='ls -a'`
- 工作控制、前景背景控制(job control, foreground, background)
- 程序化脚本(shell scripts)
- 万用字符(Wildcard)

#### 10.1.5 查询指令是否为Bash shell的内置命名: type

- 查看某个指令是来自于外部指令(指非 bash所提供的命令)，还是内置 bash的指令
- `type [-tpa] name`
- 也可以作为类似 which指令的用途，找指令用的

#### 10.1.6 指令的下达与快速编辑按钮

- `\[Enter]` 换行输入

- | 组合键            | 功能与示范                                 |
  | ----------------- | ------------------------------------------ |
  | [ctrl]+u/[ctrl]+k | 分别是从光标处 向前删除或 向后删除指令串   |
  | [ctrl]+a/[ctrl]+e | 分别是让光标移动到指令串的 最前面或 最后面 |

### 10.2 Shell的变量功能

变量对于 bash非常重要

#### 10.2.1 什么是变量

- 让一个特定字符串代表不固定的内容
- 变量的可变性与方便性
- 影响 bash环境操作的变量
- 脚本程序设计(shell script) 的好帮手

#### 10.2.2 变量的取用与设置:echo,变量设置规则,unset

- 变量的取用
  - `echo $variable`
  - `echo $PATH`
  - `echo ${PATH}`
- 设置
  - `myname=abc`
  - `echo ${myname}`
- 变量的设置规则
  - 变量与变量内容以一个等号"="来链接
  - 等号两边不能直接接空白字符
  - 变量名称只能是英文字母与数字，但是开头字符不能是数字
  - 变量内容若有空白字符可使用双引号或 单引号将变量内容结合起来
    - 双引号中可识别变量
    - 单引号中仅为一般字符串
  - 可用跳脱字符`\`将特殊字符变成一般字符（转义字符）
  - 在一串指令中的执行中，还需要由其他额外的指令所提供的信息，可以使用 反单引号或 $(指令)
  - 若该变量为扩增变量内容时，可用 `$变量名称` 或 `${变量}`累加内容
  - 若该变量需要在其他子程序执行，则需要以 export来使变量变成环境变量
  - 通常大写字符为系统默认变量，自行设置变量可使用小写字符
  - 取消变量的方法是使用 unset: `unset 变量名称`

#### 10.2.3 环境变量的功能

- 用 `env`观察环境变量与常见环境变量说明
  - `env`
  - 列出所有的环境变量
  - 包括 export导出的变量
- 用 `set`观察所有变量(含环境变量与 自订变量)
- `PS1` 观察字符的设置
- `$` 关于本shell的PID
- `?` 关于上个执行命令的回传值
- OSTYPE,HOSTTYPE,MACHTYPE 主机硬件与核心的等级
- `export` 自订变量转成环境变量

#### 10.2.4 影响显示结果的语系变量(locale)

- 查询 linux支持的语系
  - `locale -a`
- 整体系统的默认语系定义在 locale.conf

#### 10.2.5 变量的有效范围

- 环境变量=全域变量
- 自订变量=区域变量

#### 10.2.6 变量键盘的读取、阵列与宣告: read,array,declare

- `read`
  - 读取来自键盘输入的变量
  - `read [-pt] variable`
- `declare/typeset`
  - 宣告变量的类型
  - `declare -x sum` 将 sum变成环境变量
  - `declare -r sum` 将 sum变成只读属性
  - `declare +x sum` 将 sum取消环境变量
  - `declare -p sum` 单独列出变量的类型
- bash对变量的定义
  - 变量类型默认为 字符串
  - bash环境中的数值运算，默认最多仅能达到整数形态，1/3 = 0
- 阵列(array) 变量类型
  - `var[index]=content` 设置阵列的每一个 item
  - `echo "${var[1]},${var[2]},${var[3]}"`

#### 10.2.7 与文件系统及程序的限制关系: ulimit

- `ulimit [-SHacdfltu] [配额]`
- 限制使用者的某些系统资源
- `ulimit -a` 列出目前身份的所有限制数据数值
- `ulimit -f 10240` 限制使用者仅能创建 10M以下容量的文件

#### 10.2.8 变量内容的删除、取代与替换

- `path=${PATH}`

- `echo ${path}`

- `echo ${path#/*local/bin:}`  删除 path变量中的 local/bin

- `#` 代表从前面开始删除，仅删除最短的那个，`*` 来取代 0到无穷多个字符

- `##` 代表删除最长的那个

- `:` 代表截止位置

- `%` 代表从后面向前删除变量内容

- | 变量设置方式                                  | 说明                                                         |
  | --------------------------------------------- | ------------------------------------------------------------ |
  | ${变量#关键字} \${变量##关键字}               | 若变量内容从头开始的数据符合“关键字”，则将符合的最短数据删除，若变量内容从头开始的数据符合“关键字”，则将符合的最长数据删除 |
  | ${变量%关键字} \${变量%%关键字}               | 若变量内容从尾向前的数据符合“关键字”，则将符合的最短数据删除，若变量内容从未向前的数据符合“关键字”，则将符合的最长数据删除 |
  | ${变量/旧字串/新字串} \${变量//旧字串/新字串} | 若变量内容符合“旧字串”则第一个旧字串会被新字串取代，若变量内容符合“旧字串”则全部的旧字串会被新字串取代 |

- 变量的测试与内容替换

  - `echo ${username} `  -->  ''

- `username=${username-root}`

  - `echo ${username}`  -->  root

- `username="test"`

  - `username=${username-root}`   -->  test

### 10.3 命令别名与历史命令

#### 10.3.1 命令别名设置:alias,unalias

- `alias lm='ls -al | more'`
- `alias rm='rm -i'`
- `alias`
- `unalias lm`

#### 10.3.2 历史命令:history

- `alias h='history'`
- `history [n]` 列出最近 n条命令
- `history [-c]` 清除所有 history
- `history [-raw] hisfiles`
- 历史命令的读取与记录过程
  - 以 bash登录主机，会从 ~/.bash_history读取命令，条数跟 HISTFILESIZE有关
  - 登出时，会将历史记录更新到 ~/.bash_history
  - `history -w`强制写入
- 同一账号同时多次登录的 history写入问题
  - 会记录最后登出的那个 bash
  - 其他 bash命令也会被记录，但是会被最后一个覆盖更新
- 无法记录时间
  - 可以通过 ~/.bash_logout来进行 history的记录，并加上 date来增加时间参数

### 10.4 Bash Shell的操作环境

#### 10.4.1 路径与指令搜寻顺序

- 指令运行的顺序
  1. 以相对/绝对路径执行指令，eg: `/bin/ls` 或 `./ls`
  2. 由 alias找到该指令来执行
  3. 由 bash内置的(builtin)指令来执行
  4. 通过 $PATH这个变量的顺序搜寻到的第一个指令来执行
  5. 先 alias再 builtin再由 $PATH找到 /bin/echo

#### 10.4.2 bash的进站与欢迎讯息: /etc/issue, /etc/motd

-  终端机接口(tty1~tty6)登录时所提示的文字在 ` /etc/issue`中
-  `/etc/issue.net`是提供给 telnet这个远端登录程序用的
-  `/etc/motd` 记录着 当登陆后，告诉登录者的消息

#### 10.4.3 bash的环境配置文件

- login与 non-login shell
  - login shell
    - 取得 bash时需要完整的登录流程
    - login shell会读取两个配置文件
      1. `/etc/profile` 系统整体的设置
         - 利用使用者的 UID来决定很多重要的变量数据
         - 设置的变量还有 PATH, MAIL, USER, HOSTNAME, HISTSIZE, umask
         - 调用外部的数据
         - `/etc/profile.d/*.sh` 所有拓展名为 .sh的，并且用户有 r权限，那么该文件都会被 /etc/profile调用进来
         - `/etc/locale.conf` 由 /etc/profile.d/lang.sh调用进来，最重要的是 LANG/LC_ALL
         - `/usr/share/bash-comletion/completions/*` 由 /etc/profile.d/bash_completion.sh载入，主要是命令补齐，文件名补齐
      2. `~/.bash_profile或 ~/.bash_login或 ~/.profile` 属于使用者个人设置
         - bash在读完 /etc/profile，再依次读取个人配置文件 ~/.bash_profile, ~/.bash_login, ~/.profile
         - bash的 login shell设置只会读取 其中一个文件，顺序如上所述
         - `~/.bash_profile` 通过 sourse指令来读取，最后会读取 `~/.bashrc`的内容
    - source 读入环境配置文件的指令
      - 修改配置文件之后，不需要先登出再登录来生效设置
      - 直接读取配置文件到当前的环境中
  - non-login shell: 取得 bash接口的方法不需要重复登录
    - 仅会读取 `~/.bashrc`
    - 还会主动调用`/etc/bashrc`，其内容：
      - 依据不同的 UID规范出 umask的值
      - 依据不同的 UID规范出提示字符（S1变量）
      - 调用 /etc/profile.d/*.sh的设置
    - 是 RedHat系统特有的，其他 distributions可能有不同的文件名
  - 其他配置文件
    - `/etc/man_db.conf` 规范使用 man
    - `~/.bash_history` 与 HISTFIOESIZE变量有关
    - `~/.bash_logout` 记录了 登出bash需要做的动作

#### 10.4.4 终端机的环境设置: stty,set

- linux distributions已经提供了 使用者环境

- 某些 Unix like的机器中，需要自己来配置一些 按键

- stty (setting tty终端机的意思)可以帮助设置终端机的输入按键代表意义

- `stty [-a]` 将目前所有的 stty参数列出来

  - intr: interrupt中断的讯号给正在 run的程序
  - quit: quit的讯号给正在 run的程序
  - erase: 向后删除字符
  - kill: 删除在目前命令行上的文字
  - eof: End of file，结束输入
  - start: 在某个程序停止后，重新启动它的 output
  - stop: 停止目前屏幕的输出
  - susp: terminal stop的讯号给正在 run的程序

- set 除了来显示一些 变量，还可以设置这个指令输出/输入的环境

  - `set [-uvCHhmBx]`
  - `echo $-` $-变量内容就是 set的所有设置

- bash的默认组合键

  | 组合按键 | 执行结果                     |
  | -------- | ---------------------------- |
  | Ctrl + C | 终止目前命令                 |
  | Ctrl + D | 输入结束 EOF                 |
  | Ctrl + M | Enter                        |
  | Ctrl + S | 暂停屏幕的输出               |
  | Ctrl + Q | 恢复屏幕的输出               |
  | Ctrl + U | 在提示字符下，将整列命令删除 |
  | Ctrl + Z | 暂停目前的命令               |

#### 10.4.5 万用字符与特殊字符

- 万用字符（wildcard）

  | 符号 | 意义                                                |
  | ---- | --------------------------------------------------- |
  | *    | 代表 "0个到无穷多个"任意字符                        |
  | ?    | 代表 "一定有一个"任意字符                           |
  | []   | 同样代表 "一定有一个在括号内"的字符（非任意字符）   |
  | [-]  | 若有减号在中括号内时，代表 "在编码顺序内的所有字符" |
  | [^]  | 若中括号内的第一个字符为指数符号，表示 "反向选择"   |

- 特殊符合

  | 符号  | 意义                                               |
  | ----- | -------------------------------------------------- |
  | #     | 注释符号                                           |
  | \     | 跳脱符号：将 特殊字符或万用字符还原成一般字符      |
  | \|    | 管线（pipe）:分隔两个管线命令的界定                |
  | ;     | 连续指令下达分隔符号：连续性命令的界定             |
  | ~     | 使用者的主目录                                     |
  | $     | 取用变量前置字符：亦即是变量之前需要加的变量取代值 |
  | &     | 工作控制（job control）：将指令变成背景下工作      |
  | !     | 逻辑运算意义上的非 not的意思                       |
  | /     | 目录符号：路径分隔的符号                           |
  | >, >> | 数据流重导向：输出导向，分别是 取代 和 累加        |
  | <, << | 数据流重导向：输入导向                             |
  | ''    | 单引号，不具有变量置换的功能（$变为纯文本）        |
  | ""    | 具有变量置换的功能（$可保留相关功能）              |
  | ``    | 中间为可执行的指令，亦可使用 $()                   |
  | ()    | 在中间的为 子shell的起始与结束                     |
  | {}    | 在中间为命令区块的组合                             |

### 10.5 数据流重导向

redirect(数据流重导向)，就是将某个指令执行后应该要出现在屏幕上的数据，给他传输到其他的地方

#### 10.5.1 什么是数据流重导向

standard output和 standard error output分别代表 标准输出（STDOUT）与 标准错误输出（STDERR）。标准输出指的是 指令执行所回传的正确的讯息，而标准错误输出是 指令执行失败后，所回传的错误讯息

- 数据流重导向可以将 standard output和 standard error output分别传送到其他的文件或设备去，而分别传送所用的特殊字符如下：
  - 标准输入（stdin）：代码为 0，使用 <或 <<
  - 标准输出（stdout）：代码为 1，使用 >或 >>
  - 标准错误输出（stderr）：代码为 2，使用 2>或 2>>

- `ll / > ~/rootfile`

  - 该文件 rootfile若不存在，系统会自动的将它创建起来

  - 当这个文件存在的时候，那么系统就会先把这个文件内容清空，然后再将数据写入

  - 若以 `>`输出到一个已存在的文件中，那个文件将会被覆盖掉

  - 不想删除旧数据，可以使用 `>>`来累加数据，文件不存在就创建，存在就累加数据

  - `1>`：以覆盖的方法将正确的数据输出到指定的文件或设备上

  - `1>>`：以累加的方法将正确的数据输出到...

  - `2>`：以覆盖的方法将错误的数据输出到...

  - `2>>`：以累加的方法将错误的数据输出到...

    

- 将正确的与错误的数据分别存入到不同的文件中

  `find /home -name .bashrc > list_right 2> list_error`

- `/dev/null`垃圾桶黑洞设备和特殊写法

  /dev/null可以吃掉任何导向这个设备的信息，可以实现将错误讯息忽略掉而不显示或存储

  `find /home -name .bashrc 2> /dev/null`

- 将正确和错误的数据写入同一个文件

  `find /home -name .bashrc > list 2>&1`

  `find /home -name .bashrc &> list`

- standard input：<与 <<

  将原本需要由键盘输入的数据，改由文件内容来取代

  ```shell
  cat > catfile
  	testing
  	cat file test 
  	< [ctrl] + d
  ```

  用某个文件的内容来取代键盘的敲击

  `cat > catfile < ~/.bashrc`

  `<<`表示结束的输入字符，eg: 用 cat直接将输入的信息输出到 catfile中，且当键盘输入 eof时，该次输入就结束

  ```shell
  cat > catfile << 'eof'
  	this is a test
  	ok now stop
  	eof < 输入关键字，立刻结束而不需要输入 [ctrl] + d
  ```

- `2>&1 1>&2`

#### 10.5.2 命令执行的判断依据  ;   &&   || 

- cmd;cmd 不考虑指令相关性的连续指令下达

  一次执行多个指令，分号前的指令执行完后就会立刻接着执行后面的指令

-  $? （指令回传值）与 && 或 ||

  两个指令之间有相依性，而这个相依性主要判断的地方在于前一个指令执行的结果是否正确。若前一个指令执行的结果为正确，在 linux下面会回传一个 $? = 0的值

  - cmd1 && cmd2 ： 若 cmd1执行完毕且正确执行（`$?=0`），则开始执行 cmd2，若 cmd1执行完毕且为错误（`$?≠0`）,则 cm2不执行
  - cmd1 || cmd2：cmd1执行完毕且正确执行，cmd2不执行。若 cmd1执行完毕且为错误，则开始执行 cmd2

- `ls /tmp/abc || mkdir /tmp/abc && touch /tmp/abc/hehe`

- ## 第四章:首次登陆与线上求助

  ### 4.1首次登录系统

  #### 4.1.3 x window与文字模式的切换

  - linux默认提供6个 terminal
  - 切换方式为 [Ctrl] + [Alt] + [F1]~[F6]
  - 系统将 [F1]~[F6]命令为 tty1~tty6
  - `startx`启动窗口界面，任何人都可以执行
  - `systemctl setdefault graphical.target`将图形化界面设为默认
  - `exit`登出系统

  ### 4.2 文字模式下指令的下达

  #### 4.2.1 开始下达指令

  ```
  # 格式
  [linux@name ~]$ command [-options] parameter1 parameter2 ...
  ```

  1. 一行指令中第一个输入的部分是  [指令 command] 或 [可执行档案 例: 可执行脚本 scrpit]
  2. command为指令的名称
  3. 中括号[]位置是加入选项设定，通常选项前会带 `-` 号，如 `-h`；也可以是完整指令，选项前带 `—`号，如 `--help`
  4. parameter1 parameter2.. 为依附在选项后面的参数，或者是 command的参数
  5. 指令，选项，参数之间用空格来区分，不论空几格shell都视为一格。空格是很重要的特殊字元
  6. 按下 Enter按键后，该指令就立即执行。它代表着一行指令的开始启动
  7. 指令太长，可以使用反斜杠 `/`来跳脱 Enter符号
  8. linux系统区分 英文字母大小写

  语系支持

  - `locale`显示目前所支持的语系
  - `LANG=en_US.utf8`、`export LC_ALL=en_US.utf8`，LANG只与输出信息有关系。若需要更改其他不同信息，还要同步更新LC_ALL

  #### 4.2.2 基础指令的操作

  - `date`显示日期与时间
  - `cal` 显示日历
  - `bc` 计算器

  ### 4.3 man page与 Info page

  - man page

    | 代号 | 代表内容                                                     |
    | ---- | ------------------------------------------------------------ |
    | 1    | 用户可以在shell中操作的指令或可执行文件                      |
    | 2    | 系统核心可调用的函数与工具                                   |
    | 3    | 一些常用的函数(function)或函数库(library)，大部分为C的函数库(libc) |
    | 4    | 设备文件的说明，通常在/dev下的文件                           |
    | 5    | 配置文件或某些文件的格式                                     |
    | 6    | 游戏                                                         |
    | 7    | 惯例与协议，如Linux文件系统、网络协定、ASCII code            |
    | 8    | 系统管理员可用的管理指令                                     |
    | 9    | 跟 kernel有关的文件                                          |

  ### 4.4 文本编辑器：nano

  `^`代表 `Ctrl`按键

  ### 4.5 正确关机方法

  - `who` 查看目前有谁在线上
  - `netsat -a` 查看网络的连线状态
  - `ps -aux` 查看后台执行的程序
  - `sync` 将数据同步写入硬盘中
  - `shutdown` 常用关机指令
  - `reboot、halt、poweroff` 重新开机，关机
  - `shutdown` 在關機時會把系統的服務都關閉之後，才關閉電腦，而 `halt`、 `poweroff`指令則允許不管系統的狀態為何，直接停止電腦的運作
  - `shutdown`、`reboot`、`halt`等指令均已经执行过 `sync`了

  ## 第五章:linux的文件权限与目录配置

  - 文件的可存取身份：owner/group/others
  - 三种身份各有 read/write/execute等权限

  ### 5.1 使用者与群组

  linux使用者身份与群组记录的文件

  - `/etc/passwd`这个文件中，记录着所有的系统上的账号与一般身份者及root的相关信息
  - `/etc/shadow`记录着个人的密码
  - `/etc/group` 记录着所有的群组名称

  ### 5.2 Linux 文件权限概念

  #### 5.2.1 文件属性

  ```
              连接数    所属群组     创建时间或更新时间 
  -rw-r--r--.  1  root  root  1864   May 4 18:01  initial-setup-ks.cfg
  档案类型权限      owner      档案容量                   档案名字
     -     r w x    r w x   ---
  档案类型 owner权限 群组权限 其他人的权限
  r：可读     4
  w: 可写     2
  x: 可执行   1
  ```

  - 第一栏是 文件的类型和权限
    - 第一字符代表文件的类型：目录[d]、文件[-]、链接文件[|]、设备文件里的可供储存的设备(可随机存取设备)[b]、设备文件里的序列埠设备(键盘等，一次性读取设备)[c]
    - 剩下的分别为 owner/group/others的权限
  - 第二栏是 多少文件链接到此节点(i-node)
  - 第三栏是 表示文件的owner
  - 第四栏是 所属群组
  - 第五栏是 文件size
  - 第六栏是 文件的创建日期或 最近的修改日期
  - 第七栏是 文件名

  #### 5.2.2 如何改变文件属性与权限

  - `chgrp` 改变文件所属群组
    - chgrp [ -R ] dirname/filename
    - -R 递归(recursive)执行
    - eg: chgrp users initial-setup-ks.cfg
  - `chown` 改变文件拥有者
    - chown [ -R ] 账号名称  文件或目录 
    - chown [ -R ] 账号名称:群组名称  文件或目录
    - chown [ -R ] 账号名称.群组名称  文件或目录
    - eg: chown root:root initial-setup-ks.cfg
  - `chmod` 改变文件的权限，SUID,SGID,SBIT等特性
    - chmod [ -R ] xyz  文件或目录。xyz为数字类型的权限属性(r: 4,w: 2,x: 1) 
    - chmod | u(user|owner) g(group) o(others) a(all) | + (加入) -(去除) =(设置) | r w x | 文件或目录
    - eg: 
      - chmod 777 .bashrc
      - chmod u=rwx,go=rx .bashrc
      - chmod a+w .bashrc
      - chmod a-x .bashrc

  #### 5.2.3 目录与文件之权限意义

  | 元件 | 内容          | 叠代物件   | r            | w            | x                     |
  | ---- | ------------- | ---------- | ------------ | ------------ | --------------------- |
  | 文件 | 详细数据 data | 文件数据夹 | 读到文件内容 | 修改文件内容 | 执行文件内容          |
  | 目录 | 文件名        | 可分类抽屉 | 读到文件名   | 修改文件名   | 进入该目录的权限(key) |

  #### 5.2.4 Linux文件种类与扩展名

  - 文件种类
    - 正规文件(regular file)
      - 纯文本文件(ASCII)
      - 二进制(binary)
      - 数据格式文件(data)
    - 目录(directory)
    - 链接文件(link)
    - 设备与设备文件(device)
      - 区块(block)
      - 字符(character)
    - 数据接口文件(sockets)
    - 数据输送档(FIFO，pipe)
  - 文件拓展名
    - 没有所谓的拓展名，全靠 权限的`x`来表示是否能执行
    - 常用拓展名
      - `.sh` 脚本或批处理文件(script)
      - `Z`，`.tar`，`.tar.gz`，`.zip` ，`.tgz` 经过打包的压缩文件
      - `.html`、`.php` 网页相关文件
  - 文件长度限制
    - 单一文件或目录的最大容许文件名为 255 Bytes
    - 相当于 ASCII英文 255个字符
    - 相当于 中文(每个字符 2Bytes)，128个中文字
  - 文件名称限制
    - 避免使用`?><;&![]|\'"`(){}`
    - `.`开头的文件为隐藏文件
    - 避免使用 `+ -`

  ### 5.3 Linux 目录配置

  #### 5.3.1 目录配置的依据 — FHS

  - FHS将目录分为四种交互作用的形态

    |                    | 可分享的(shareable)        | 不可分享的(unshareable) |
    | ------------------ | -------------------------- | ----------------------- |
    | 不变的(static)     | /usr (软件放置处)          | /etc (配置文件)         |
    |                    | /opt (第三方协力软件)      | /boot (开机与核心档)    |
    | 可变动的(variable) | /var/mail (使用者邮件信箱) | /var/run (程序相关)     |
    |                    | /var/spool/news (新闻群组) | /var/lock (程序相关)    |

  - FHS定义的三个主目录

    - `/`(root，根目录)：与开机系统有关
      - 根目录是最重要的目录
      - 所有的目录都是由根目录衍生出来的
      - 与开机/还原/系统修复等有关
    - `/usr`(unix software resource): 与软件安装/执行有关
      - `/usr`放置的数据属于可分享与不可变动的(shareable, static)
      - 是 Unix Software Resource(Unix操作系统软件资源)所放置的目录
    - `/var`(variable): 与系统运行过程有关
      - `/var`目录主要针对常态性变动的文件，包括高速缓存(cache)、登录文件(log file)
      - 还有某些软件运行所产生的文件，包括程序文件(lock file, run file)，或者 MySQL数据库文件

  #### 5.3.2 目录树 (directory tree)

  - 目录树的起始点为根目录(/, root)
  - 每一目录不止能使用本地端的 partition文件系统，可以用使用网络上的filesystem
  - 每个文件在此目录树中的文件名(含完整路径)都是独一无二的

  ## 第六章:Linux文件与目录管理

  ### 6.1 目录与路径

  #### 6.1.1 相对路径与绝对路径

  #### 6.1.2 目录的相关操作

  - `cd` 变化目录(change directory)
  - `pwd` 显示当前的目录
  - `mkdir` 创建一个新的目录
  - `rmdir` 删除一个空的目录

  #### 6.1.3 关于可执行文件路径的变量: $PATH

  - `echo $PATH` 显示当前的 PATH
  - 这个变量的内容是由一堆目录所组成的
  - 每个目录之间用冒号`:`来隔开
  - 每个目录是有顺序之分的
  - 不同身份使用者默认的PATH不同，能够执行的指令也不同
  - PATH可以被修改
  - 使用绝对路径或相对路径直接执行某个指令的文件名，会比搜寻的PATH正确
  - 指令应该放到正确的目录下
  - 本目录(.)最好不要放到PATH中

  ### 6.2 文件与目录管理

  #### 6.2.1 文件与目录的检视: ls

  `ll`在大多数distribution的默认情况下都是 `ls -l`的意思

  #### 6.2.2 复制、删除、移动: cp、rm、mv

  - `cp` 复制文件或目录
  - `rm` 移除文件或目录
  - `mv` 移动文件与目录，或更名
    - 使用 `-u`来测试新旧文件，看是否需要搬运

  #### 6.2.3 取得路径的文件名称与目录名称

  - `basename` 取得最后的文件名
  - `dirname` 取得的目录名

  ### 6.3 文件内容查阅

  - `cat` 由第一行开始显示文件内容
  - `tac` 由最后一行开始显示
  - `nl` 显示的时候，输出行号
  - `more` 分页显示文件内容
  - `less` 分页显示文件内容，可向前翻页
  - `head` 只看头几行
  - `tail` 只看最后几行
  - `od` 以二进制的方式读取文件内容

  #### 6.3.1 直接检视文件内容

  - `cat` Concatenate(连续)的简写
    - 将一个文件的内容连续打印在屏幕上
    - `cat [-AbEnTv]`
  - `tac` 由最后一行到第一行显示在屏幕上
  - `nl` 添加行号打印
    - `nl [-bnw]` 文件

  #### 6.3.2 可翻页检视

  - `more` 分页翻动
    - 空格键：向下翻页
    - Enter：向下翻一行
    - /字串：向下搜索输入的字符
    - `:f`：立刻显示出文件名及目前显示的行数
    - `q`：退出
    - `b或[ctrl]-b`：向回翻页，只对文件有效，对管线无效
  - `less` 分页翻动，man指令就是调用`less`来显示说明文档
    - 空格键：向下翻页
    - `[pagedown]`：向下翻页
    - `[pageup]`：向上翻页
    - `/字串`：向下搜索输入的字符
    - `?字串`：向上搜索输入的字符
    - `n`：重复前一次搜索
    - `N`：反向的重复前一次搜索
    - `g`：前进到数据的第一行
    - `G`：前进到数据的最后一行
    - `q`：退出

  #### 6.3.3 数据撷取

  - `head` 取出前面几行
    - head [-n number] 文件
    - -n 后面接数字，代表显示几行
    - 默认显示前十行
    - eg: head -n 20 /etc/man_db.conf
    - eg: head -n -100 /etc/man_db.conf (除了最后100行，其他都显示)
  - `tail` 取出最后几行
    - tail [-n number] 文件
    - -n  后面数字，代表显示几行
    - -f  持续侦测后面的文件
    - 默认显示最后十行
    - eg: tail -n 20 /etc/man_db.conf
    - eg: tail -n +100 /etc/man_db.conf (只列出100行之后的内容)

  #### 6.3.4 非纯文本文件：od

  - `od [-t TYPE] 文件`
  - 可执行文件通常是 binary file，od读取不会出现乱码

  #### 6.3.5 修改文件时间或创建新文件：touch

  - 三个主要的时间参数
    - `modification time (mtime)` 文件的内容数据变更时，更新该时间
    - `status time (ctime)` 文件的状态（权限或属性）变更时，更新该时间
    - `access time (atime)` 文件的内容被取用时，更新该时间
  - `touch [-acdmt] 文件`
    - -a 仅修改 atime
    - -c 仅修改 ctime，文件不存在则不创建新文件
    - -d 后面可以接欲修订的日期而不用当前的日期，等同 `—date="日期或时间"`
    - `-m` 仅修改 mtime
    - -t 后面可以接欲修订的日期而不用当前的日期，格式为 `[YYYYMMDDhhmm]`
  - 使用场景
    - 创建一个 空文件
    - 将某个文件日期修改为当前 (mtime和 atime)

  ### 6.4 文件与目录的默认权限与隐藏权限

  #### 6.4.1 文件默认权限：umask

  unmaks就是指定 目前使用者再创建文件或目录时候的权限默认值

  - 查阅的方式
    - `umask`
    - `umask -S(Symbolic)`
  - 设置的方式
    - 直接 umask number
    - eg: `umask 002`

  #### 6.4.2 文件隐藏属性

  - `chattr` 设置文件隐藏属性
    - `chattr [+-=][ASacdistu] 文件或目录名称` 
  - `lsattr` 显示文件隐藏属性

  #### 6.4.3 文件特殊权限：SUID,SGID,SBIT

  - Set UID
    - 当 s 出现在文件拥有者的 x权限上时，就被称为Set UID，简称为 SUID的特殊权限
    - 限制与功能
      - SUID权限仅对二进制程序(binary program)有效，不能用在 shell script上
      - 执行者对于该程序需要有 x权限
      - 只在执行该程序的过程中有效(run-time)
      - 执行者将具有该程序的拥有者(owner)权限
  - Set GID
    - 当 s 出现在群组的 x权限上时，就被成为Set GID，SGID
    - 功能
      - SGID对二进制程序有用
      - 可以是针对文件或目录进行设置
      - 程序执行者对于该程序来说，需要具备 x权限
      - 执行者在执行的过程中将获得该程序群组的支持
  - Sticky Bit
    - SBIT 只针对目录有效，对文件没有效果
    - 对目录的作用
      - 当使用者对于此目录具有w,x权限，亦具有写入的权限
      - 当使用者在该目录下创建文件或目录时，仅有自己与 root才有权利删除
  - SUID/SGID/SBIT 权限设置
    - 三个数字组合
      - 4 为 SUID
      - 2 为 SGID
      - 1 为 SBIT
    - eg: `chmod 4755 filename` , 4 为特殊权限的值之和
    - 大写的S/T，具有空的SUID/SGID权限

  #### 6.4.4 观察文件类型: file

  - 可以知道某个文件的基本数据
  - 该文件没有使用到动态函数库(share library)
  - eg: `file ~/.bashrc`

  ### 6.5 指令与文件的搜寻

  #### 6.5.1 指令文件名的搜寻

  - `which` 寻找可执行文件
  - `which [-a] command`
  - `-a` 将所有由 PATH目录中可以找到的指令列出，而不止第一个被找到的
  - 根据 PATH环境变量所规范的路径，去搜寻可执行的文件名

  #### 6.5.2 文件文件名的搜寻

  - `whereis` 由一些特定的目录中寻找文件文件名
    - `whereis [-bmsu] 文件或目录名`
  - `locate/updatedb`
    - `locate [-ir] keyword`
    - 寻找的数据由 已创建的数据库/var/lib/mlocate/ 里面的数据所搜寻到，不是直接搜索硬盘
    - 数据库的创建默认每天执行一次，有可能会找不到文件
    - `updatedb` 会读取 /etc/updatedb.conf这个配置，然后去搜寻文件名，最后更新数据库文件
  - `find`
    - `find [PATH] [option] [action]`
    - 跟时间有关的选项
      - +4 代表大于等于 5天前的文件名：ex>find /var -mtime +4
      - -4 代表小于等于 4天内的文件文件名：ex>find /var -mtime -4
      - 4 代表 4-5那一天的文件文件名：ex>find /var -mtime 4
    - 跟使用者或群组名称有关的参数
    - 与文件权限及名称有关的参数

  ## 第七章:Linux磁盘与文件系统的管理

  ### 7.1 认识Linux文件系统

  #### 7.1.1 磁盘组成与分区的复习

  - 磁盘组成
    - 原型的盒片（记录数据）
    - 机械手臂及磁头（读写盘片）
    - 主轴马达，转动盘片
  - 磁盘的文件名
    - 所有的文件名被仿真为`/dev/sd[a-p]`，第一颗磁盘文件名为 `/dev/sda`
    - 分区的文件名为 `/dev/sda[1-128]`
    - 虚拟机的磁盘通常为 `/dev/vd[a-p]`
    - 软件磁盘阵列为 `/dev/md[0-128]`
    - 使用的是LVM时，文件名为 `/dev/VGNAME/LVNAME`等格式
    - `/dev/sd[a-p][1-128]`：为实体磁盘的磁盘文件名
    - `/dev/vd[a-d][1-128]`：为虚拟磁盘的磁盘文件名

  #### 7.1.2 文件系统特性

  - 磁盘分区，进行格式化之后，成为操作系统能够使用的 文件系统格式(filesystem)
    - LVM，软件磁盘阵列(software raid)将一个分区格式化为多个文件系统
    - LVM，PAID可以将多个分区合成一个文件系统
  - 文件系统通常会将文件权限和文件属性分别到存放到不同的区块，权限与属性放置在 inode中,实际数据放置到 data block区块中，还有一个超级区块(superblock)会记录整个文件系统的整体信息
    - `superblock`：记录filesystem整体信息，包括inode/block的总量、使用量、剩余量，以及文本系统的格式与相关信息
    - `inode`：记录文件的属性，一个文件占用一个inode，同时记录此文件的数据所在的block号码
    - `block`：实际记录文件的内容，若文件太大时，会占用多个block
  - 索引式文件系统（indexed allocation）
  - 磁盘重组

  #### 7.1.3 Linux的 EXT2文件系统(inode)

  Ex2在格式化的时候基本上是区分为多个区块群组（block group）的，每个区块群组都有独立的 inode/block/superblock系统，每个区块群组有6个主要内容：

  - `data block` 数据区块
    - 原则上，除了重新格式化，block的大小与数量在格式化完都不能改变了
    - 每个block内最多只能放一个文件的数据
    - 如果文件大于 block的大小，则一个文件会占用多个block
    - 若文件小于block，则剩余的容量不能再被使用
  - `inode table` inode表格
    - inode记录的文件数据
      - 该文件的存取模式（read/write/excute）
      - 该文件的owner和group
      - 文件的容量
      - 文件的创建或状态改变的时间（ctime）
      - 最近一次的读取时间（atime）
      - 最近修改的时间（mtime）
      - 定义文件特性的flag，如SetUID...
      - 该文件真正内容的指向（pointer）
    - 特性
      - 每个 inode大小固定为 128Bytes(新的ext4和xfs可设置为 256B)
      - 每个文件仅占用一个 inode
      - 文件系统能够创建的文件数量与 inode的数量有关
      - 系统先找到 inode，分析inode中的权限是否符合使用者，符合的话，才开始实际读取block
  - `Superblock` 超级区块
    - Superblock是记录整个filesystem相关信息的地方，记录的信息：
      - block与 inode的总量
      - 未使用与已使用的 block与 inode数量
      - block(1,2,4K)与 inode(128,256Bytes)的大小
      - filesystem的挂载时间、最近一次写入数据的时间、最近一次检验磁盘（fsck）的时间等
      - 一个 vaild bit数值，若文件系统被挂载，vaild bit为0，否则是1
    - 每个block group都可能有superblock，其他的是第一个的备份
  - `Filesystem Descripition` 文件系统描述说明
    - 这个区段描述每个block group的开始和结束的block号码
    - 说明每个区段分别介于哪个block号码之间
  - `block bitmap` 区块对照表
    - 得知哪个是空的 block
    - 删除文件后，相关block被释放，bitmap中的该block号码被标记为 未使用
  - `inode bitmap` inode对照表
    - 记录使用与未使用的 inode号码
  - `dumpe2fs`：查询 Ext家族 superblock信息的指令

  #### 7.1.4 与目录树的关系

  - 目录
    - 创建一个目录，文件系统会分配一个inode和一个block，其中 inode记录该目录的相关权限与属性及分配的block号码，而block中则是记录这个目录下的文件名与该文件名占用的inode号码数据
    - `ls -i` 来显示文件所占用的 inode号码
  - 文件
    - inode 仅有12个直接指向
    - 一个间接
    - 一个双间接
    - 一个三间接记录区
  - 目录树读取
    - 由于文件名的记录是在目录的block中，所以新增/删除/重命名与目录的w权限有关
    - 目录树由根目录开始读起

  #### 7.1.5 EXT2/EXT3/EXT4 文件的存取与日志式文件系统的功能

  - 新增文件的行为
    1. 确定使用者是否有 w和 x权限
    2. 根据inode bitmap找到没有使用的 inode号码，将新文件的权限/属性写入
    3. 根据block bitmap找到没有使用的 block号码，将实际数据写入block中，并将新的 inode的block指向数据
    4. 更新同步 inode bitmap, block bitmap, superblock
  - 数据存放区域：inode table, data block
  - 中介数据（metadata，涉及到操作，经常会变动）：superblock, block bitmap, inode bitmap
  - 数据的不一致（inconsistent）状态
  - 日志式文件系统(Journaling filesystem)
    - 预备：当系统要写入一个文件，会现在日志记录区块中记录某个文件准备写入的信息
    - 实际写入：开始写入文件的权限和数据，开始更新 metadata数据
    - 结束：完成数据与 metadata的更新，在日志记区块中完成该文件的记录

  #### 7.1.6 Linux文件系统的运行

  - 系统载入一个文件到内存后
  - 该文件没有被更动过，被标记为 干净(clean)
  - 被更动过，会被标记为 脏的(dirty)
  - 所有动作暂不会被写入到磁盘
  - 系统不定时将 dirty数据写回磁盘

  #### 7.1.7 挂载点的意义(mount point)

  - 挂载点一定是目录，该目录为进入该文件系统的入口
  - 并不是任何文件系统都可以使用，必须挂载目录树的某个目录后，才能使用该文件系统

  #### 7.1.8 其他Linux支持的文件系统与 VFS

  - 常见支持的文件系统
    - 传统文件系统：ext2/minix/MS-DOS/FAT(用 vfat模块)/iso9960(光盘)
    - 日志式文件系统：ext3/ext4/ReiserFS/Window's NTFS/IBM's JFS/SGI's XFS/ZFS
    - 网络文件系统：NFS/SMBFS
  - 查看linux支持和的文件系统，可查看这个目录
    - `ls -l /lib/modules/$(uname -r)/kernel/fs`
  - 系统目前已载入内存中的文件系统
    - `cat /proc/filesystems`
  - Linux VFS(Virtual Filesystem Switch)
    - 整个 Linux认识的 filesystem都是VFS 进行管理

  #### 7.1.9 XFS 文件系统简介

  - EXT家族当前：支持度最广，但格式化超慢
  - XFS文件系统的配置
    - 数据区(data section)
      - 类似于 ext家族的 block group
      - block和inode有不同的容量进行设置
    - 文件系统活动登录区(log section)
      - 主要记录文件系统的变化
    - 实时运行区(realtime section)
      - 文件被创建时，xfs会在这个区段找到一/数个extent区块，将文件放置到这个区块后，等分配完，在写入到data section的 inode与 block
  - XFS文件系统的描述数据观察
    - `xfs_info 挂载点 文件设备名` 

  ### 7.2 文件系统的简单操作

  #### 7.2.1 磁盘与目录的容量

  - `df` 列出文件系统的整体磁盘使用量
    - `df [-ahikHTm] [目录或文件名]`
    - 读取的范围主要是在 Superblock内的信息
    - 特别留意根目录的容量
  - `du` 评估文件系统的磁盘使用量（常用在推估目录所占容量）
    - `du [-ahskm] 文件或目录名称`
    - 直接到文件系统内搜寻所有的文件数据

  #### 7.2.2 实体链接与符号链接：ln

  - Hard Link (实体链接，硬式链接或实际链接)
    - 只是在某个目录下新增一笔文件名链接到某 inode号码的关联记录
    - 一般情况下，使用hard link设置链接文件时，磁盘的空间与 inode的数目都不会改变
    - 不能跨 Filesystem
    - 不能 Link目录
  - Symbolic Link (符号链接，亦即捷径)
    - 创建一个独立的文件，而这个文件会让数据的读取指向他link的那个文件的文件名
  - `ln [-sf] 来源文件 目标文件`
  - 关于目录的link数量

  ### 7.3 磁盘的分区、格式化、检验与挂载

  新增磁盘步骤

  1. 分区，创建可用的 partition
  2. 对该 partition格式化(format)，创建可用的 filesystem
  3. 可以选择对 filesystem进行检验
  4. 创建挂载点(目录)，并挂载

  #### 7.3.1 观察磁盘分区状态

  - `lsblk` 列出系统中的所有磁盘列表
    - `lsblk [-dfimpt] [device]`
    - `-f` 列出文件系统与设备的 UUID数据
    - `MAJ:MIN` 主要、次要设备代码
  - `blkid` 列出设备的UUID等参数
    - UUID是全域单一识别码(universally unique identifier)
  - `parted` 列出磁盘的分区表类型与分区信息
    - `parted device_name print`

  #### 7.3.2 磁盘分区：gdisk/fdisk

  MBR分区表使用 fdisk分区，GPT分区表使用 gdisk分区

  - `gdisk`
    - `gdisk 设备名称`
    - `w` 立即执行
    - `q` 离开
    - `partprobe` 更新 Linux核心的分区表信息
    - `partprobe [-s]` -s打印信息
    - `cat /proc/partitions` 核心的分区资料
  - `fdisk`
    - `fdisk 设备名称`
    - `m` 获得指令介绍

  #### 7.3.3 磁盘格式化（创建文件系统）

  - XFS文件系统 mkfs.xfs
    - `mkfs.xfs [-b bsize] [-d parms] [-i parms] [-l parms] [-L label] [-f] [-r parms] 设备名称`
    - eg: `mkfs.xfs /dev/vda4`
    - `grep 'processor' /proc/cpuinfo` 找出系统的cpu核数
    - `mkfs.xfs -f -d agcount=2 /dev/vda4` 设置 agcount数值
  - XFS文件系统 for RAID性能优化（optional）
    - 磁盘阵列（RAID）
  - EXT4文件系统mkfs.ext4
    - 格式化为 ext4的传统 linux文件
    - `mkfs.ext4 [-b size] [-L label] 设备名称`
  - 其他文件系统 mkfs
    - mkfs(make filesystem)是个综合指令
    - `mkfs[tab][tab]` 查看支持的文件系统的格式化功能

  #### 7.3.4 文件系统检验

  - `xfs_repair` 处理 XFS文件系统
    - `xfs_repair [-find] 设备名称`
  - `fsck.ext4` 处理 EXT4文件系统
    - `fsck.ext4 [-pf] [-b superblock] 设备名称`

  #### 7.3.5 文件系统挂载与卸载

  - 挂载点是目录，这个目录是进入磁盘分区（文件系统）的入口，挂载前的检查：
    - 单一文件系统不应该重复被挂载在不同的挂载点(目录)中
    - 单一目录不应该重复挂载多个文件系统
    - 要做为挂载点的目录，理论应该是空目录
  - `mount` 挂载指令
    - `mount -a`  依照配置文件[/etc/fstab]将磁盘挂载
    - `mount [-l]` 显示Label名称
    - `mount [-t 文件系统] LABEL='' 挂载点`
    - `mount [-t 文件系统] UUID='' 挂载点`
    - `mount [-t 文件系统] 设备文件名 挂载点`
  - 哪些类型的 filesystem需要进行挂载测试，参考：
    - `/etc/filesystems` 系统指定的测试挂载文件系统类型的优先顺序
    - `/proc/filesystems` linux系统已经载入的文件系统类型
  - linux支持的文件系统之驱动程序所在目录
    - `/lib/moduels/$(uname -r)/kernel/fs/`
  - 挂载 xfs/ext4/vfat等文件系统
    - `blkid /dev/vda5` 找出 /dev/vda5的UUID
    - `mount UUID="~~" /data/ext4`  挂载
  - 挂载 CD或 DVD光盘
  - 挂载 vfat中文U盘（USB磁盘）
  - 重新挂载根目录与挂载不特定目录
  - `umount` 将设备文件卸载
    - `umount [-fn] 设备文件名或挂载点`

  #### 7.3.6 磁盘/文件系统参数修订

  修改一下目前文件系统的一些相关信息

  - mknod 
    - `mknod 设备文件名 [bcp] [Major] [Minor]`
  - `xfs_admin` 修改 XFS文件系统的UUID与Label name
    - `xfs_admin [-lu] [-L label] [-U uuid] 设备文件名`
  - `tune2fs` 修改 ext4的 label name与UUID
    - `tune2fs [-l] [-L label] [-U uuid] 设备文件名`

  ### 7.4 设置开机挂载

  #### 7.4.1 开机挂载 /etc/fstab及 /etc/mtab

  - 系统挂载的限制
    - 根目录`/` 是一定要挂载的，还要先于其他 mount point挂载
    - 其他 mount point必须是已创建目录，要遵守 FHS
    - 所有的 mount point在同一时间内，只能挂载一次
    - 所有的 partition在同一时间内，只能挂载一次
    - 卸载前，需要将工作目录移到 mount point之外
  - mount指令挂载时，将所有的选项写入`/etc/fstab`(filesystem table)
  - `/etc/fstab`文件内容共有 6个字段
    - 磁盘设备文件名/UUID/LABEL name
    - 挂载点(mount point)
    - 磁盘分区的文件系统
    - 文件系统参数
    - 是否被 dump备份指令作用
    - 是否以 fsck检验扇区

  #### 7.4.2 特殊设备loop挂载

  - 挂载光盘/DVD镜像文件
    - `mount -o loop /tmp/Centod-7.0-DVD.iso /data/centos_dvd`
  - 创建大文件以制作loop设备文件
  - 创建大型文件
    - `dd if=/dev/zero of=/srv/loopdev bs=1M count=512`
  - 大型文件的格式化
    - `mkfs.xfs -f /srv/loopdev`
    - `blkid /srv/loopdev`
  - 挂载
    - `mount -o loop UUID="7ddasfasdfaf" /mnt`
    - `df /mnt`

  ### 7.5 内存交换空间（swap）之创建

  swap是 解决内存不足，可以暂时把内存中的程序拿到内存中暂放的内存交换空间

  #### 7.5.1 使用实体分区创建swap

  - 创建步骤
    1. 分区
    2. 格式化
    3. 使用
    4. 通过 free与 swapon -s观察内存的用量

  #### 7.5.2 使用文件创建swap

  1. 使用 dd指令新增一个 128MB的文件在 /tmp
     - `dd if=/dev/zero of=/tmp/swap bs=1M count=128`
  2. 使用 mkswap将 /tmp/swap的文件格式化为 swap的格式
     - `mkswap /tmp/swap`
  3. 使用 swapon将 /tmp/swap启动
     - `swapon /tmp/swap`
     - `swapon -s`
  4. 使用 swapoff 关掉 swap file,并设置自启动

  ### 7.6 文件系统的特殊观察与操作

  #### 7.6.1 磁盘空间之浪费问题

  #### 7.6.2 利用 GNU的 parted进行分区行为（Optional）

  - 同时支持 GPT和 MBR的分区指令 -> `parted`
  - 可以在一行命令中完成分区
  - `parted [设备] [指令 [参数]]`
  - 列出目前本机的分区表数据
    - `parted /dev/vda print`

  ## 第八章:文件与文件系统的压缩，打包与备份

  ### 8.1 压缩文件的用途与技术

  - 目前计算机系统中是使用所谓的 Bytes单位来计量
  - 1Bytes = 8bits
  - WWW网站压缩技术

  ### 8.2 Linux系统常见的压缩指令

  - 压缩文件拓展名
    - `*.z`  compress程序压缩的文件
    - `*.zip`  zip程序压缩的文件
    - `*.gz`  gzip程序压缩的文件
    - `*.bzz`  bzip2程序压缩的文件
    - `*.xz`  xz程序压缩的文件
    - `*.tar`  tar程序打包的数据
    - `*.tar.gz`  tar程序打包的文件，经过 gzip压缩
    - `*.tar.bzz`  tar程序打包的文件，经过 bzip2压缩
    - `*.tar.xz`  tar程序打包的文件，经过 xz压缩

  #### 8.2.1 gzip,zcat/zmore/zless/zgrep

  - gzip可以解开 compress,zip,gzip等软件压缩的文件
  - `gzip [-cdtv#] 文件名`
    - -c 将压缩的数据输出到屏幕上，可通过数据流重导向来处理
    - -d 解压缩
    - gunzip 解压缩
    - -t 可以用来检验一下压缩文件的一致性，检查错误
    - -v 可以显示元文件/压缩文件的压缩比等信息
    - -# #是数字，代表压缩等级，1~9(压缩比最好)，默认为6
  - gzip 压缩的文件可以在 window系统上被解压
  - zcat/zmore/zless 可以读取纯文本文件压缩后的文件
  - zgrep 来搜寻压缩文件中的关键字

  #### 8.2.2 bzip2,bzcat/bzmore/bzless/bzgrep

  - bzip2为了取代 gzip来提供更好的压缩比
  - `bzip2 [-cdkzv#] 文件名`
    - -k  保留原始文件
    - -z  压缩的参数，默认值
    - 其他参数类似 gzip的参数
  - bzcat/bzmore/bzless 可以读取纯文本文件压缩后的文件
  - bzgrep来搜寻压缩文件中的关键字

  #### 8.2.3 xz,xzcat/xzmore/xzless/xzgrep

  - xz比 bzip2的压缩比要更高，但时间更久
  - `xz [-dtlkc#] 文件名`
    - -l 列出压缩文件的相关信息
    - 其他参数类似 gzip, bzip2的参数

  ### 8.3 打包指令: tar

  将多个文件或目录包成一个大文件的指令功能就是打包指令

  #### 8.3.1 tar

  - 常用 tar命令及参数
    - `tar [-z | -j | -J] [cv] [-f 待创建的新文件名] filename`  打包
    - `tar [-z | -j | -J] [tv] [-f 既有的 tar文件名]`  察看
    - `tar [-z | -j | -J] [xv] [-f 既有的 tar文件名] [-C 目录]`  解压
    - `-c`  创建打包文件，可搭配 -v来查看过程中的被打包文件名
    - `-t`  查看打包文件的内容有哪些文件名
    - `-x`  解打包或解压缩，搭配 -C在特定目录解开
    - `-c, -t, -x` 不可同时出现在一行命令中
    - `-z`  通过 gzip来压缩或解压，文件名：*.tar.gz
    - `-j`  通过 bzip2来压缩或解压，文件名：*.tar.bz2
    - `-J`  通过 xz来压缩或解压，文件名：*.tar.xz
    - `-z, -j, -J` 不可同时出现在一行命令中
    - `-v`  在压缩或解压过程中，将正在处理的文件名显示出来
    - `-f filename`  -f后面直接接要被处理的文件名
    - `-C 目录`  在特定目录进行解压
    - `-p`  保留备份数据的原本权限与属性
    - `-P`  保留绝对路径，允许备份数据包含根目录
  - 简单的使用方式
    - `tar -jcv -f filename.tar.bz2 要被压缩的目录或目录名称`  压缩
    - `tar -jtv -f filename.tar.bz2`  查询
    - `tar -jxv -f filename.tar.bz2 -C 欲解压的目录`  解压
  - 打包某目录，但不包含该目录下的某些文件
    - `--exclude`  不包含
    - eg: `tar -jcv -f /root/system.tar.bz2 --exclude=/root/etc*`
  - 仅备份比某个时刻还要新的文件
    - `--newer-mtime`  后续日期只包含 mtime
    - `--newer`  后续日期包含 mtime和 ctime
    - eg: `tar -jcv -f /root/etc.newer.then.passwd.tar.bz2 --newer-mtime="2019/08/13" /etc/*`
  - 基本名称 tarfile, tarball
    - tar打包出来的文件没有进行压缩，叫作 tarfile
    - tar打包出来的文件有进行压缩，叫作 tarball
  - 特殊应用: 利用管线命令与数据流
    - eg: `tar -cvf - /etc | tar -xvf -` 将 /etc的数据压缩并解压到 当前所在目录

  ### 8.4 XFS文件系统的备份与还原

  xfs文件系统中，xfsdump和 xfsrestore

  #### 8.4.1 XFS文件系统备份 xfsdump

  - xfsdump 可以进行文件系统的完整备份(full backup)，还可以进行累积备份(Incremental backup)，备份有变化的文件
  - 限制
    - xfsdump只备份已挂载的文件系统
    - 只有 root权限才能操作
    - 只能备份 XFS文件系统
    - 备份下来的数据只能通过 xfsrestore解析
    - 可以通过文件系统的 UUID来分辨备份文件，不能备份两个相同 UUID的文件系统
  - 指令
    - `xfsdump [-L S_label] [-M M_label] [-l #] [-f 备份文件] 待备份数据`
    - `xfsdump -I` 从 /var/lib/xfsdump/inventory 列出目前备份的信息状态

  #### 8.4.2 XFS文件系统还原 xfsrestore

  - xfsdump的复原使用的是 xfsrestore指令
  - `xfsrestore -I`  查看备份文件数据
    - xfsfdump和 xfsrestore都会从 /var/lib/xfsdump/inventory/里面取数据
  - `xfsrestore [-f 备份文件] [-L S_label] [-s] 待复原项目` 
    - 需要被复原的那个文件
    - 该文件的 session label name
    - eg: `xfsrestore -f /srv/boot.dump -L boot_all /boot`
    - 只想要复原某个目录或文件，直接加上 `-s 目录`
  - `xfsrestore [-f 备份文件] -r 待复原文件`  通过累积备份文件来复原系统
    - 如果备份数据是由 level0 -> level1 -> level2...去进行的
    - 复原也需要相同的流程
  - `xfsrestore [-f 备份文件] -i 待复原文件`  进入互动模式
    - `-s` 可以接部分数据还原
    - `-i` 可以以互动的形式来还原

  ### 8.5 光盘写入工具

  文字模式的烧录行为的通常做法

  - 利用 mkisofs指令先将文件创建成为 镜像文件(iso)
  - 利用 cdrecord指令来将镜像文件烧录至光盘或DVD

  #### 8.5.1 mkisofs: 创建镜像文件

  #### 8.5.2 cdrecord: 光盘烧录工具

  ### 8.6 其他常见的压缩与备份工具

  #### 8.6.1 dd

  - dd 不只是制作一个文件，更重要的是在 备份
  - 可以读取磁盘设备的内容（几乎是直接读取扇区 sector）
  - 没有用到的扇区也会写入备份文件，xfsdump只会备份使用到的部分
  - 做出来的文件跟原本的数据大小一样
  - 新分区的 partition不需要格式化，因为 dd直接将 sector表面的数据都复制来了

  #### 8.6.2 cpio

  - cpio可以备份任何东西，包括设备文件
  - 不会主动的去找文件备份
  - 配合 find等可以找到文件名的指令来使用
  - `cpio -ovcB > [file|device]`  备份
  - `cpio -ivcdu < [file|device]`  还原
  - `cpio -ivct < [file|device]`  查看
  - eg: `find boot | cpio -ocvB > /tmp/boot.cpio` 

  ## 第九章:vim程序编辑器

  - vim是一种程序编辑器

  ### 9.1 vi与vim

  - 在 Linux中的大部分配置文件都是以 ASCII的纯文本形态存在的

  #### 9.1.1 为何要学 vim

  - 使用的原因
    - 所有的 Unix Like系统都内置 vi
    - 很多软件的编辑接口都主动调用 vi
    - vim具有程序编辑的能力，能用字体颜色分辨语法的正确性
    - 运行速度快，程序简单
  - vim是 vi的进阶版本

  ### 9.2 vi的使用

  - 一般指令模式(command mode)
  - 编辑模式(insert mode)
    - `i,I,o,O,a,A,r,R` 进入编辑模式
    - Esc退出
  - 命令行命令模式(command-line mode)

  #### 9.2.1 简易执行范例

  1. 使用 “vi filename”进入一般指令模式
     - eg: `/bin.vi welcome.txt`
     - 在 centos7中，一般账号 vim取代了 vi，所以使用绝对路径执行
  2. 按下 i进入编辑模式，开始编辑文字
  3. 按下 [Esc]回到一般指令模式
  4. 进入命令行界面，文件储存并离开 vi环境

  #### 9.2.2 按键说明

  - 第一部分：一般指令模式可用的按键

  | 移动光标的方法        |                                            |
  | --------------------- | ------------------------------------------ |
  | h或 <-                | 向左移动一个字符                           |
  | j或 向下方向键        | 向下移动一个字符                           |
  | k或 向上方向键        | 向上移动一个字符                           |
  | l或 ->                | 向右移动一个字符                           |
  | Ctrl + f              | 向下翻一页                                 |
  | Ctrl + b              | 向上翻一页                                 |
  | Ctrl + d              | 向下翻半页                                 |
  | Ctrl + u              | 向上翻半页                                 |
  | +                     | 光标移动到非空白字符的下一列               |
  | -                     | 光标移动到非空白字符的上一列               |
  | n<space>              | 移动到光标所在行的第几个字符               |
  | 0或[home]             | 移动到这一列的最前面字符                   |
  | $或[End]              | 移动到这一列的最后面字符                   |
  |                       | 光标移动到屏幕最上方列的第一个字符         |
  | M                     | 光标移动到屏幕中央列的第一个字符           |
  | L                     | 光标移动到屏幕最下方列的第一个字符         |
  | G                     | 光标移动到这个文件的最后一列               |
  | nG                    | 移动到文件的第 n列                         |
  | gg                    | 移动到文件的第一列                         |
  | n<Enter>              | 向下移动 n列                               |
  | /word                 | 向光标之下寻找字符串                       |
  | ?word                 | 向光标之上寻找字符串                       |
  | n                     | 英文按键，重复前一次向上的寻找动作         |
  | N                     | 英文按键，重复前一次向下的寻找动作         |
  | :n1,n2s/word1/word2/g | n1,n2为数字,代表行数,搜寻word1,用word2替换 |
  | :1,$s/word1/word2/g   | 从第一行搜寻到最后一行，并替换             |
  | :1,$s/word1/word2/gc  | 搜寻并提示用户是否取代                     |
  | x,X                   | x向后删除一个字符，X向前删除一个字符       |
  |                       | n为数字，连续向后删除n个字符               |
  | dd                    | 删除光标所在行                             |
  | ndd                   | n为数字，删除光标所在的向下n行             |
  | d1G                   | 删除光标所在到第一行的所有数据             |
  | dG                    | 删除光标所在到最后一行的所有数据           |
  | d$                    | 删除光标所在到该行的最后一个字符           |
  | d0                    | 数字0，删除光标所在到该行的第一个字符      |
  | yy                    | 复制光标所在行                             |
  | nyy                   | 复制光标所在的向下 n行                     |
  | y1G                   | 复制光标所在到第一行的所有数据             |
  | yG                    | 复制光标所在到最后一行的所有数据           |
  | y0                    | 复制光标所在字符到行首的数据               |
  | y$                    | 复制光标所在字符到行尾的数据               |
  | p,P                   | p在下一行粘贴，P在上一行粘贴               |
  | J                     | 合并光标所在行与下一行                     |
  | c                     | 重复删除多个数据，如10cj,向下删除10行      |
  | u                     | 复原上一个操作                             |
  | Ctrl + r              | 重做上一个操作                             |

  - 第二部分：一般指令模式切换到编辑模式的可用按钮说明

  | 进入插入或取代的编辑模式 |                                                          |
  | ------------------------ | -------------------------------------------------------- |
  | i,I                      | 进入 insert mode，在光标处开始输入                       |
  | a,A                      | 进入 insert mode，在光标所在行的最后一个字符开始输入     |
  | o,O                      | 进入 insert mode，在光标所在行的下面插入一个新行开始输入 |
  | r,R                      | 进入 insert mode，取代光标处的字符，开始输入             |
  | [Esc]                    | 退出 insert mode，进入一般指令模式                       |

  - 第三部分: 一般指令模式切换到命令行界面的可用按钮说明

  | 命令行界面的储存、离开等指令 |                                          |
  | ---------------------------- | ---------------------------------------- |
  | :w                           | 储存数据                                 |
  | :w!                          | 强制储存数据，跟权限有关系               |
  | :q                           | 离开 vi                                  |
  | :wq                          | 储存数据并退出，:wq!强制退出，不保留修改 |
  | ZZ                           | 有更改就储存退出，没有更改就不储存但退出 |
  | :w[filename]                 | 另存为 新文件                            |
  | :r[filename]                 | 读入另一个文件的数据，添加到光标后面     |
  | :n1,n2 w[filename]           | 截取 n1,n2的内容储存成新文件             |
  | :! command                   | 强制执行命令                             |
  | :set nu                      | 显示行号                                 |
  | :set nonu                    | 不显示行号                               |

  #### 9.2.4 vim的暂存盘、救援回复与打开时的警告讯息

  - 用 vim编辑时，vim会在被编辑的文件目录下，创建一个 .filename.swp的文件
  - 该文件会记录 对所编辑文件的动作
  - 可以用作救援功能

  ### 9.3 vim的额外功能

  #### 9.3.1 区块选择(Visual Block)

  | 区块选择的按键意义 |                                      |
  | ------------------ | ------------------------------------ |
  | v                  | 字符选择，会将光标经过的地方反白选择 |
  | V                  | 行选择，会将光标经过的行反白选择     |
  | Ctrl + v           | 区块选择，可以用长方形的方式选择数据 |
  | y                  | 将反白的地方复制起来                 |
  | d                  | 将反白的地方删除掉                   |
  | p                  | 将刚刚复制的区块，在光标处贴上       |

  #### 9.3.2 多文件编辑

  | 多文本编辑的按键 |                            |
  | ---------------- | -------------------------- |
  | :n               | 编辑下一个文件             |
  | :N               | 编辑上一个文件             |
  | :files           | 列出目前 vim打开的所有文件 |

  #### 9.3.3 多窗口功能

  | 多窗口情况下的按键功能 |                                |
  | ---------------------- | ------------------------------ |
  | :sp filename           | 打开有个新窗口，后面是文件名   |
  | Ctrl + w + j/向下箭头  | 分别按下对应按键移动到下方窗口 |
  | Ctrl + w + k/向上箭头  | 分别按下对应按键移动到上方窗口 |
  | Ctrl + w + q           | 关闭保存 下方窗口              |

  #### 9.3.4 vim的挑字补全功能

  | 组合按钮             | 补齐的内容                                                 |
  | -------------------- | ---------------------------------------------------------- |
  | Ctrl + x -> Ctrl + n | 通过目前正在编辑的这个“文件的内容文字”作为关键字，予以补齐 |
  | Ctrl + x -> Ctrl + f | 以当前目录内的“文件名”作为关键字，予以补齐                 |
  | Ctrl + x -> Ctrl + o | 以拓展名作为语法补充，以vim内置的关键字，予以补齐          |

  #### 9.3.5 vim环境设置与记录:~/.vimrc,~/viminfo

  - `~/viminfo`
    - 记录曾经做过的行为
  - `~/.vimrc`
    - 默认不存在，需自行创建
    - 个人的配置项
  - `/etc/vimrc`
    - 全局的 vim配置项

  #### 9.3.6 vim常用指令示意图

  - 一般模式
  - 编辑模式
  - 指令模式

  ## 第十章:认识与学习 BASH

  ### 10.1 认识 BASH这个 Shell

  #### 10.1.1 硬件、核心与Shell

  用户 --->  使用者界面(Shell, KDE, application)  --->  核心(kernel)  --->  硬件(Hardware)

  #### 10.1.2 为何要学命令行的shell

  - 各家 distributions使用的 bash都一样
  - 远端管理：命令行的传输速度比较快

  #### 10.1.3 系统的合法shell与/etc/shells功能

  - Shell版本
    - Bourne Shell(sh)
    - Sun的 C Shell
    - 商业上的 K Shell
    - TCSH
    - Bourne Again Shell(bash)
  - /etc/shells 可以看到可用的 shells 

  #### 10.1.4 Bash shell的功能

  - 命令编修能力(history)
    - 默认记录 1000个
    - 在 .bash_history中存储
    - 这一次的命令行会暂存在内存中
  - 命令与文件补全功能( [tab]按键的好处)
    - 命令补全
    - 文件补齐
  - 命令别名设置功能(alias)
    - `alias lm='ls -a'`
  - 工作控制、前景背景控制(job control, foreground, background)
  - 程序化脚本(shell scripts)
  - 万用字符(Wildcard)

  #### 10.1.5 查询指令是否为Bash shell的内置命名: type

  - 查看某个指令是来自于外部指令(指非 bash所提供的命令)，还是内置 bash的指令
  - `type [-tpa] name`
  - 也可以作为类似 which指令的用途，找指令用的

  #### 10.1.6 指令的下达与快速编辑按钮

  - `\[Enter]` 换行输入

  - | 组合键            | 功能与示范                                 |
    | ----------------- | ------------------------------------------ |
    | [ctrl]+u/[ctrl]+k | 分别是从光标处 向前删除或 向后删除指令串   |
    | [ctrl]+a/[ctrl]+e | 分别是让光标移动到指令串的 最前面或 最后面 |

  ### 10.2 Shell的变量功能

  变量对于 bash非常重要

  #### 10.2.1 什么是变量

  - 让一个特定字符串代表不固定的内容
  - 变量的可变性与方便性
  - 影响 bash环境操作的变量
  - 脚本程序设计(shell script) 的好帮手

  #### 10.2.2 变量的取用与设置:echo,变量设置规则,unset

  - 变量的取用
    - `echo $variable`
    - `echo $PATH`
    - `echo ${PATH}`
  - 设置
    - `myname=abc`
    - `echo ${myname}`
  - 变量的设置规则
    - 变量与变量内容以一个等号"="来链接
    - 等号两边不能直接接空白字符
    - 变量名称只能是英文字母与数字，但是开头字符不能是数字
    - 变量内容若有空白字符可使用双引号或 单引号将变量内容结合起来
      - 双引号中可识别变量
      - 单引号中仅为一般字符串
    - 可用跳脱字符`\`将特殊字符变成一般字符（转义字符）
    - 在一串指令中的执行中，还需要由其他额外的指令所提供的信息，可以使用 反单引号或 $(指令)
    - 若该变量为扩增变量内容时，可用 `$变量名称` 或 `${变量}`累加内容
    - 若该变量需要在其他子程序执行，则需要以 export来使变量变成环境变量
    - 通常大写字符为系统默认变量，自行设置变量可使用小写字符
    - 取消变量的方法是使用 unset: `unset 变量名称`

  #### 10.2.3 环境变量的功能

  - 用 `env`观察环境变量与常见环境变量说明
    - `env`
    - 列出所有的环境变量
    - 包括 export导出的变量
  - 用 `set`观察所有变量(含环境变量与 自订变量)
  - `PS1` 观察字符的设置
  - `$` 关于本shell的PID
  - `?` 关于上个执行命令的回传值
  - OSTYPE,HOSTTYPE,MACHTYPE 主机硬件与核心的等级
  - `export` 自订变量转成环境变量

  #### 10.2.4 影响显示结果的语系变量(locale)

  - 查询 linux支持的语系
    - `locale -a`
  - 整体系统的默认语系定义在 locale.conf

  #### 10.2.5 变量的有效范围

  - 环境变量=全域变量
  - 自订变量=区域变量

  #### 10.2.6 变量键盘的读取、阵列与宣告: read,array,declare

  - `read`
    - 读取来自键盘输入的变量
    - `read [-pt] variable`
  - `declare/typeset`
    - 宣告变量的类型
    - `declare -x sum` 将 sum变成环境变量
    - `declare -r sum` 将 sum变成只读属性
    - `declare +x sum` 将 sum取消环境变量
    - `declare -p sum` 单独列出变量的类型
  - bash对变量的定义
    - 变量类型默认为 字符串
    - bash环境中的数值运算，默认最多仅能达到整数形态，1/3 = 0
  - 阵列(array) 变量类型
    - `var[index]=content` 设置阵列的每一个 item
    - `echo "${var[1]},${var[2]},${var[3]}"`

  #### 10.2.7 与文件系统及程序的限制关系: ulimit

  - `ulimit [-SHacdfltu] [配额]`
  - 限制使用者的某些系统资源
  - `ulimit -a` 列出目前身份的所有限制数据数值
  - `ulimit -f 10240` 限制使用者仅能创建 10M以下容量的文件

  #### 10.2.8 变量内容的删除、取代与替换

  - `path=${PATH}`

  - `echo ${path}`

  - `echo ${path#/*local/bin:}`  删除 path变量中的 local/bin

  - `#` 代表从前面开始删除，仅删除最短的那个，`*` 来取代 0到无穷多个字符

  - `##` 代表删除最长的那个

  - `:` 代表截止位置

  - `%` 代表从后面向前删除变量内容

  - | 变量设置方式                                 | 说明                                                         |
    | -------------------------------------------- | ------------------------------------------------------------ |
    | ${变量#关键字} ${变量##关键字}               | 若变量内容从头开始的数据符合“关键字”，则将符合的最短数据删除，若变量内容从头开始的数据符合“关键字”，则将符合的最长数据删除 |
    | ${变量%关键字} ${变量%%关键字}               | 若变量内容从尾向前的数据符合“关键字”，则将符合的最短数据删除，若变量内容从未向前的数据符合“关键字”，则将符合的最长数据删除 |
    | ${变量/旧字串/新字串} ${变量//旧字串/新字串} | 若变量内容符合“旧字串”则第一个旧字串会被新字串取代，若变量内容符合“旧字串”则全部的旧字串会被新字串取代 |

  - 变量的测试与内容替换

    - `echo ${username}`  -->  ''

  - `username=${username-root}`

    - `echo ${username}`  -->  root

  - `username="test"`

    - `username=${username-root}`   -->  test

  ### 10.3 命令别名与历史命令

  #### 10.3.1 命令别名设置:alias,unalias

  - `alias lm='ls -al | more'`
  - `alias rm='rm -i'`
  - `alias`
  - `unalias lm`

  #### 10.3.2 历史命令:history

  - `alias h='history'`
  - `history [n]` 列出最近 n条命令
  - `history [-c]` 清除所有 history
  - `history [-raw] hisfiles`
  - 历史命令的读取与记录过程
    - 以 bash登录主机，会从 ~/.bash_history读取命令，条数跟 HISTFILESIZE有关
    - 登出时，会将历史记录更新到 ~/.bash_history
    - `history -w`强制写入
  - 同一账号同时多次登录的 history写入问题
    - 会记录最后登出的那个 bash
    - 其他 bash命令也会被记录，但是会被最后一个覆盖更新
  - 无法记录时间
    - 可以通过 ~/.bash_logout来进行 history的记录，并加上 date来增加时间参数

  ### 10.4 Bash Shell的操作环境

  #### 10.4.1 路径与指令搜寻顺序

  - 指令运行的顺序
    1. 以相对/绝对路径执行指令，eg: `/bin/ls` 或 `./ls`
    2. 由 alias找到该指令来执行
    3. 由 bash内置的(builtin)指令来执行
    4. 通过 $PATH这个变量的顺序搜寻到的第一个指令来执行
    5. 先 alias再 builtin再由 $PATH找到 /bin/echo

  #### 10.4.2 bash的进站与欢迎讯息: /etc/issue, /etc/motd

  - 终端机接口(tty1~tty6)登录时所提示的文字在 `/etc/issue`中
  - `/etc/issue.net`是提供给 telnet这个远端登录程序用的
  - `/etc/motd` 记录着 当登陆后，告诉登录者的消息

  #### 10.4.3 bash的环境配置文件

  - login与 non-login shell
    - login shell
      - 取得 bash时需要完整的登录流程
      - login shell会读取两个配置文件
        1. `/etc/profile` 系统整体的设置
           - 利用使用者的 UID来决定很多重要的变量数据
           - 设置的变量还有 PATH, MAIL, USER, HOSTNAME, HISTSIZE, umask
           - 调用外部的数据
           - `/etc/profile.d/*.sh` 所有拓展名为 .sh的，并且用户有 r权限，那么该文件都会被 /etc/profile调用进来
           - `/etc/locale.conf` 由 /etc/profile.d/lang.sh调用进来，最重要的是 LANG/LC_ALL
           - `/usr/share/bash-comletion/completions/*` 由 /etc/profile.d/bash_completion.sh载入，主要是命令补齐，文件名补齐
        2. `~/.bash_profile或 ~/.bash_login或 ~/.profile` 属于使用者个人设置
           - bash在读完 /etc/profile，再依次读取个人配置文件 ~/.bash_profile, ~/.bash_login, ~/.profile
           - bash的 login shell设置只会读取 其中一个文件，顺序如上所述
           - `~/.bash_profile` 通过 sourse指令来读取，最后会读取 `~/.bashrc`的内容
      - source 读入环境配置文件的指令
        - 修改配置文件之后，不需要先登出再登录来生效设置
        - 直接读取配置文件到当前的环境中
    - non-login shell: 取得 bash接口的方法不需要重复登录
      - 仅会读取 `~/.bashrc`
      - 还会主动调用`/etc/bashrc`，其内容：
        - 依据不同的 UID规范出 umask的值
        - 依据不同的 UID规范出提示字符（S1变量）
        - 调用 /etc/profile.d/*.sh的设置
      - 是 RedHat系统特有的，其他 distributions可能有不同的文件名
    - 其他配置文件
      - `/etc/man_db.conf` 规范使用 man
      - `~/.bash_history` 与 HISTFIOESIZE变量有关
      - `~/.bash_logout` 记录了 登出bash需要做的动作

  #### 10.4.4 终端机的环境设置: stty,set

  - linux distributions已经提供了 使用者环境

  - 某些 Unix like的机器中，需要自己来配置一些 按键

  - stty (setting tty终端机的意思)可以帮助设置终端机的输入按键代表意义

  - `stty [-a]` 将目前所有的 stty参数列出来

    - intr: interrupt中断的讯号给正在 run的程序
    - quit: quit的讯号给正在 run的程序
    - erase: 向后删除字符
    - kill: 删除在目前命令行上的文字
    - eof: End of file，结束输入
    - start: 在某个程序停止后，重新启动它的 output
    - stop: 停止目前屏幕的输出
    - susp: terminal stop的讯号给正在 run的程序

  - set 除了来显示一些 变量，还可以设置这个指令输出/输入的环境

    - `set [-uvCHhmBx]`
    - `echo $-` $-变量内容就是 set的所有设置

  - bash的默认组合键

    | 组合按键 | 执行结果                     |
    | -------- | ---------------------------- |
    | Ctrl + C | 终止目前命令                 |
    | Ctrl + D | 输入结束 EOF                 |
    | Ctrl + M | Enter                        |
    | Ctrl + S | 暂停屏幕的输出               |
    | Ctrl + Q | 恢复屏幕的输出               |
    | Ctrl + U | 在提示字符下，将整列命令删除 |
    | Ctrl + Z | 暂停目前的命令               |

  #### 10.4.5 万用字符与特殊字符

  - 万用字符（wildcard）

    | 符号 | 意义                                                |
    | ---- | --------------------------------------------------- |
    | *    | 代表 "0个到无穷多个"任意字符                        |
    | ?    | 代表 "一定有一个"任意字符                           |
    | []   | 同样代表 "一定有一个在括号内"的字符（非任意字符）   |
    | [-]  | 若有减号在中括号内时，代表 "在编码顺序内的所有字符" |
    | [^]  | 若中括号内的第一个字符为指数符号，表示 "反向选择"   |

  - 特殊符合

    | 符号  | 意义                                               |
    | ----- | -------------------------------------------------- |
    | #     | 注释符号                                           |
    | \     | 跳脱符号：将 特殊字符或万用字符还原成一般字符      |
    | \|    | 管线（pipe）:分隔两个管线命令的界定                |
    | ;     | 连续指令下达分隔符号：连续性命令的界定             |
    | ~     | 使用者的主目录                                     |
    | $     | 取用变量前置字符：亦即是变量之前需要加的变量取代值 |
    | &     | 工作控制（job control）：将指令变成背景下工作      |
    | !     | 逻辑运算意义上的非 not的意思                       |
    | /     | 目录符号：路径分隔的符号                           |
    | >, >> | 数据流重导向：输出导向，分别是 取代 和 累加        |
    | <, << | 数据流重导向：输入导向                             |
    | ''    | 单引号，不具有变量置换的功能（$变为纯文本）        |
    | ""    | 具有变量置换的功能（$可保留相关功能）              |
    | ``    | 中间为可执行的指令，亦可使用 $()                   |
    | ()    | 在中间的为 子shell的起始与结束                     |
    | {}    | 在中间为命令区块的组合                             |

  ### 10.5 数据流重导向

  redirect(数据流重导向)，就是将某个指令执行后应该要出现在屏幕上的数据，给他传输到其他的地方

  #### 10.5.1 什么是数据流重导向

  standard output和 standard error output分别代表 标准输出（STDOUT）与 标准错误输出（STDERR）。标准输出指的是 指令执行所回传的正确的讯息，而标准错误输出是 指令执行失败后，所回传的错误讯息

  - 数据流重导向可以将 standard output和 standard error output分别传送到其他的文件或设备去，而分别传送所用的特殊字符如下：
    - 标准输入（stdin）：代码为 0，使用 <或 <<
    - 标准输出（stdout）：代码为 1，使用 >或 >>
    - 标准错误输出（stderr）：代码为 2，使用 2>或 2>>

  - `ll / > ~/rootfile`

    - 该文件 rootfile若不存在，系统会自动的将它创建起来

    - 当这个文件存在的时候，那么系统就会先把这个文件内容清空，然后再将数据写入

    - 若以 `>`输出到一个已存在的文件中，那个文件将会被覆盖掉

    - 不想删除旧数据，可以使用 `>>`来累加数据，文件不存在就创建，存在就累加数据

    - `1>`：以覆盖的方法将正确的数据输出到指定的文件或设备上

    - `1>>`：以累加的方法将正确的数据输出到...

    - `2>`：以覆盖的方法将错误的数据输出到...

    - `2>>`：以累加的方法将错误的数据输出到...

      

  - 将正确的与错误的数据分别存入到不同的文件中

    `find /home -name .bashrc > list_right 2> list_error`

  - `/dev/null`垃圾桶黑洞设备和特殊写法

    /dev/null可以吃掉任何导向这个设备的信息，可以实现将错误讯息忽略掉而不显示或存储

    `find /home -name .bashrc 2> /dev/null`

  - 将正确和错误的数据写入同一个文件

    `find /home -name .bashrc > list 2>&1`

    `find /home -name .bashrc &> list`

  - standard input：<与 <<

    将原本需要由键盘输入的数据，改由文件内容来取代

    ```
    cat > catfile
      testing
      cat file test 
      < [ctrl] + d
    ```

    用某个文件的内容来取代键盘的敲击

    `cat > catfile < ~/.bashrc`

    `<<`表示结束的输入字符，eg: 用 cat直接将输入的信息输出到 catfile中，且当键盘输入 eof时，该次输入就结束

    ```
    cat > catfile << 'eof'
      this is a test
      ok now stop
      eof < 输入关键字，立刻结束而不需要输入 [ctrl] + d
    ```

  - `2>&1 1>&2`

  #### 10.5.2 命令执行的判断依据  ;   &&   || 

  - cmd;cmd 不考虑指令相关性的连续指令下达

    一次执行多个指令，分号前的指令执行完后就会立刻接着执行后面的指令

  -  $? （指令回传值）与 && 或 ||

    两个指令之间有相依性，而这个相依性主要判断的地方在于前一个指令执行的结果是否正确。若前一个指令执行的结果为正确，在 linux下面会回传一个 $? = 0的值

    - cmd1 && cmd2 ： 若 cmd1执行完毕且正确执行（`$?=0`），则开始执行 cmd2，若 cmd1执行完毕且为错误（`$?≠0`）,则 cm2不执行
    - cmd1 || cmd2：cmd1执行完毕且正确执行，cmd2不执行。若 cmd1执行完毕且为错误，则开始执行 cmd2

  - `ls /tmp/abc || mkdir /tmp/abc && touch /tmp/abc/hehe`

    如果某个目录不存在，就创建该目录，并且在目录下创建一个文件

  - `ls /tmp/abc && echo 'exist' || echo 'no exist'`

  - 假设判断式有三个

    `command1 && command2 || command3`

