开始我们的最后一本书《鸟哥的Linux私房菜》了，这本书由于过于扎实，与导师商量后决定不重要的部分就快速过一遍

该笔记根据章节分开记录

## 第0章 计算机概论

1. **外频**是计算机与外部组件进行数据传输时的速度，**倍频**是CPU内部用来加速内部性能的倍数，**两者相乘**即为CPU的频率速度

## 第1章 Linux是什么和如何学习

这章Linux历史居多，很多general的东西，没笔记

## 第2章 主机规划和磁盘分区

1. 扇区和磁道：这部分太老掉牙了，现在都是SSD，《Operation System》 3rd-ver还有专门几个小节讲，4th-ver就已经完全删去这部分内容了
2. 分区有利于把数据集中在部分扇区，加快查找速度；还有就是保障安全性
3. 分区有以下特性：
   1. 主引导记录（MBR）：安装启动程序的地方，有446字节
   2. 分区表：记录整块硬盘分区的状态，有64字节，最多四组记录，每组都记录了开始和结束的柱面号码。这四组信息被称为主要（primary）或扩展（extended）分区
   3. 所谓的分区就只是对64字节的分区表进行设置而已
   4. 扩展分区是分布在每个分区的最前面几个扇区来记录分区信息，本身并不能用来初始化，最多只能有一个，从extended分区还能划分出逻辑分区，逻辑分区和主要分区都能被格式化
   5. 逻辑分区sda从5开始，sda1~sda4是分配给primary或extended分区用的（多年的疑惑）
   6. 如果要**划定四个以上**的分区一定要有扩展分区。且考虑到磁盘的连续性，一般建议将扩展分区的柱面号码分配到最后面的柱面内
4. 启动流程中的BIOS与UFEI启动检测程序（面试问题）
   1. BIOS是写入主板的一个固件，BIO是计算机系统会主动执行的第一个程序
   2. 之后BIOS会到硬盘中读取**第一个扇区**的MBR位置，MBR 446字节的容量里放着基本的启动引导程序（boot loader）
   3. 这个boot loader是os安装时提供的，认识硬盘内的文件系统，因而可以读取内核文件，其任务：提供用户选择的启动选项，加载内核文件，或转交其他引导程序（引导程序可安装在每个分区的启动扇区，双系统的本质P77）
5. UFEI BIOS搭配GPT启动
   1. GPT可以通过64位寻址，也能提供较大的区块处理启动程序，但BIOS是一个16位程序，无法理解GPT，因此有了UFEI
   2. P78图，UFEI BIOS与BIOS的对比
   3. UFEI用polling处理硬件，BIOS用CPU中断，所以其实UFEI效率会低一些
   4. UFEI加入了secure boot的功能，防止hacker用BIOS启动阶段来破坏系统

## 第3章 安装CentOS 7.x

这里因为我一直都是用的ubuntu，而且在读到这部分时已经完成了ubuntu16虚拟机以及最终项目基础demo的运行，所以这部分就不看了

## 第4章 首次登录与在线求助

 这章主要是跟着做的

1. terminal命令的执行：
   1. 第一个被输入的字符**一定是命令或可执行文件**
   2. bc进入计算器，默认只有整型计算，小数点都被截掉，通过`scale=number`，number来决定保留几位小数
   3. 之前只用过`cd D[tab][tab]`来显示匹配的文件名，还可以直接`ca[tab][tab]`来显示符合前缀的命令
   4. `ctrl+d`可以退出命令行模式，可代替exit
   5. `[shift]+{[PageUp][PageDown]}`来上下翻看terminal页面信息
   6. `man+[command]`会显示这个command的详细信息，注意`[command(number)]`，这个number是有意义的，可以在P136查表。在man page可以通过`/+[string]`来匹配文本中的字符串
   7. `info`是由node组织的，进入node后的操作可在P141查到，也可以像man一样查到command的详细信息
2. nano指令（这个就不多说了，我比起vim更喜欢用nano）
3. `shutdown -h time`表示定时关机，-r表示reboot

## 第5章 linux权限和目录配置

用户组的概念其实在AWS以及Github中的配置中也有，大家思想都是类似的这里不赘述

1. `ls -al`的读写权限的解析（P153），如`drwxrw-r--`
   1. 第一个字符[d]表示这是一个directory，如果是文件则为[-]，[l]表示为链接文件，[b]表示设备里可供存储的周边设备，[c]表示为串行设备（如键盘，鼠标）
   2. 之后的都是三个字符一组，[r]是read，[w]是write，[x]是可执行
   3. 后面三组分别代表了**文件拥有者权限**，**加入此用户组账号权限**，**未加入用户组人的权限**
2. `chgrp`和`chown`分别是change group和change owner（P156），不是经常用到，所以主要讲`chmod`，因为就像上面提到的，权限三个一组，所以`7->111->rwx`，其他的以此类推，也就有了天天用的`chmod 777 file`这样的语句
3. 一个有意思的事：如果用户组没read权限，ls指令都会被permission denied，然后试了一下root下把文件chmod 000之后ls，发现其实root还是能看见的
4. 一般要读到directory底下的文件，都要拥有这个directory的`r-x`权限，r只是决定[tab]能否补全，x才是关键，决定能否打开
5. 文件种类和扩展名（这里只列举不算熟悉的，P163详细版本）：
   1. 常规文件：
      1. 纯文本文件（ASCII）：`cat`指令可以把文件内容读出来，ASCII文件可以被cat读出来
      2. 二进制文件（.bin）：一种可执行文件，`cat`自己就是一个二进制文件
      3. 数据文件（data）：可以使用`last`指令读出来，`cat`会读成乱码
   2. 链接文件[l]：类似于win下的快捷方式
   3. 数据接口（sockets，[s]）

## 第6章 Linux文件与目录管理

1. `pwd`（print working directory）显示当前文件夹，`cd -`返回刚刚的目录

2. `echo $PATH`打印执行文件路径的原理：`echo`就是显示，打印；$后跟变量

3. cp的copy过程默认源文件和目标文件的权限不同，目标文件的权限**是命令者自身的权限**，如果想要连同文件权限全盘copy，使用`cp -a [source dir] [target dir]`

4. `cp -l`和`cp -s`都会建立链接文件（link file），-l是hard链接，-s是symbol链接（有->指向链接文件，不增加源文件的link数量）

5. `rm -[fir]`：-f是force，强制删除；-i是interact，交互模式，会一个个询问是否删除；-r递归删除，十分危险（我经常用来删dir...）

6. `mv -[fiu]`：-f同样是force，如果重复了就直接覆盖；-i就是一个个询问是否覆盖；-u就是update，如果source比较新就会把原本的update。注意如果mv多个文件或目录，那么最后一个一定是目标目录

7. 文件内容查看
   1. tac从最后一行开始显示，是cat的倒写
   2. nl显示的时候，同时显示行号
   3. more一page一page地显示文件内容
   4. less和more类似，但可以向前翻页
   5. head/tail只看前/后几行（-n 表示行数）
      1. head可用-n -100表示忽略后面一百行，把100行前面的都打出来
      2. tail可用-n +100表示忽略前一百行，把100行后面的都打出来
   6. od以二进制形式读文件（读入二进制，-t决定以什么形式输出）
   
8. `touch [-acdmt] 文件`，如果文件存在，那么该文件的三个时间（atime(access time), ctime, mtime(modified time)）都被更新为当前时间（我一般都是不存在直接生成文件用的touch...）

9. 文件的默认权限可以使用`umask`查看，umask给出的例如`0002`结果，是后三位有效，该例中使用`umask -S`看到的真实默认权限为`u=rwx,g=rwx,o=rx`，能看出来其实就是掩码

   1. `chattr [+-=][ASacdistu]`（change attribute）可以配置文件隐藏特性，这里只介绍俩：`a`设置之后，文件**只能增加**不能删除或修改；`i`可以让一个文件**不能被删除，改名，设置link，也无法写入或增加数据**（甚至`rm -f`都删不掉）。这俩都是要root权限
   2. 可以用`lsattr`（list attribute）来看这些被设置的隐藏属性

10. 文件特殊权限：SUID, SGID, SBIT

    1. Set UID

       当s这个标志出现在owner的x权限的位置上时（`ls -l /usr/bin/passwd`可以看到），就被称为SUID，有以下限制和功能：

       1. 仅对binary程序有效（不能用在shell脚本或目录上）
       2. 执行者对于该程序需要有x的权限（比如执行者和passwd的关系）
       3. 本权限仅在执行过程中有效
       4. 执行者将具有**该程序**的owner权限（passwd的owner权限）

       书上P198给了一个例子，这就好比使用passwd命令可以让用户暂时获得passwd的owner的权限，并支持在程序执行期间对其他文件（书上的shadow文件，原本权限是`----------`）进行passwd的owner权限级别的修改

    2. Set GID

       当s这个标志出现在group的x权限的位置上时（书上说`ls -l /usr/bin/locate`可以看到，但ubuntu16上并没有），就被称为SGID，有以下功能：

       1. SGID对binary程序有用
       2. 执行者对于该程序需要具备x权限
       3. 执行者在执行过程中获得**该程序**的group支持（这里对应的是把执行者转换为目标文件的用户组，而不是passwd的group权限）

       基本上和SUID同理

    3. Sticky Bit

       可通过`ls -l /tmp`看到带t的权限，虚拟机里是`VMwareDnD`

       1. 只针对目录有效
       2. 当用户对于此目录有w，x的权限时，有写入能力
       3. 当用户在此目录下建立文件或目录时，**仅有自己和root**才有权力**删除**该文件

       说白了就是有w和x的话，谁都可以写，但是一旦写了就只有写的人以及root可以删除

    4. 可以在原本的三个数字的权限前加上一个数字，就表示以上三种特殊权限。

       1. 4为SUID
       2. 2为SGID
       3. 1为SBIT

       比如把权限改为`-rwsr-xr-x`时，可用`chmod 4755 filename`来实现

11. `which [-a] command`可以使用`which`来找到command的执行文件放在哪里，`-a`会把PATH目录里所有找到的命令都列出来而不是只列出一个

12. 一般都是先用`whereis`和`locate`来找文件，因为`whereis`只在限定的目录下找，而`locate`是在mlocate的数据库里找，所以快；找不到再用`find`，`find`可根据时间，权限，使用者，大小等查找P203

## 第7章 Linux磁盘与文件系统管理

1. 一个可挂载的数据为一个文件系统而非一个分区，通常把**文件权限（rwx）和文件属性（拥有者，用户组，时间参数）**与数据分开放置。权限和属性放在inode中，数据放在数据区块里

   1. superblock超级区块：记录此系统的整体信息，包括inode与数据区块的总量，使用量，格式等信息
   2. inode：记录文件属性，一个文件占用一个inode，同时记录此文件数据所在的区块号码
   3. 数据区块：实际记录文件内容，一个大文件可占用多个区块

2. 索引式文件系统参见P212图，这种系统会有inode去记录具体的区块，非索引式如FAT需要把所有的磁盘读出来（顺序读取和hashmap的区别），因此需要时不时碎片整理

3. ext2是一种索引式文件系统，支持的区块有1K, 2K, 4K三种：

   1. 区块的大小和数量在格式化后不能更改，除非重新格式化
   2. 每个区块内最多放置一个文件的数据
   3. 但一个文件可占用多个区块
   4. 如果一个文件大小小于区块大小，那么就会浪费磁盘空间

4. inode表记录的信息见P213，就是权限，时间，使用者，大小，block等

5. superblock（超级块）记录：

   1. 数据区块与inode总量
   2. 未使用和已使用的block以及inode数量
   3. block和inode大小（block 1k,2k,4k；inode128B, 256B）
   4. 文件系统挂载时间，最近一次写入数据时间，最近一次检验磁盘（fsck）时间等信息
   5. 一个有效位，0已被挂载，1未被挂载

   可以使用`dumpe2fs`来查询（dump ext2 file system，用过了，很详细）

6. 因为可能出现数据不一致的问题，所以又出现了**日志式文件系统**，原本不一致时check超级块中的有效位和文件状态，日志式只要看日志记录的区块就可以了，快很多

7. Linux把常用文件数据放进内存的cache buffer，可以使用`sync`强制把dirty数据写入磁盘（正常关机时，系统就调用这个命令）

8. 将文件系统和目录树结合的操作称为挂载，**挂载点一定是目录**，是进入该文件系统的入口（以前双系统扩容用到过）

9. `df`(display filesystem)看整个文件系统的磁盘使用（block版本，可以用`df -h`看正常占用版本），`du`看文件系统的磁盘使用（常用于看目录）

10. 所以前面讲了这么多，终于说到了硬链接（hard link），本质就是在某个目录下**新增一个文件名链接到inode**（类似传指针，即使源文件被删了，也能通过链接文件找到数据），不能跨文件系统，不能链接目录（消耗过大，不支持），不占用额外区块

11. 相应的，符号链接（symbolic link）就像是快捷方式，一旦源文件被删，那么就找不到了。是独立的文件，会占用inode和区块

12. `ln [-sf] sourcefile targetfile`，如果不加参数默认硬链接，`-s`就是符号链接，`-f`如果目标文件存在就直接删了再建立

13. 每个新的目录会有2链接，一个是`/tmp/testing也叫/tmp/testing/.`，另一个是返回上级目录`tmp/testing/..`，如果新建了一个目录，比如新建`/tmp/testing`，那么`/tmp`目录的链接数量也会+1

14. `lsblk`（list block device），看以看到所有磁盘和磁盘内的分区信息，具体见P232

15. `blkid`（block id）列出设备的UUID，`parted device_name print`可以列出磁盘的分区表类型和分区信息

16. 用`gdisk`可以处理分区（P235），但新增分区使用w写入后，重新启动才有效，但可以通过`partprobe`来更新分区信息

17. `mkfs`是创建文件系统，也是通常说的格式化（P238）

18. `fsck.ext4`可以对ext4文件系统进行检查，但注意必须是在文件系统有问题时使用该命令，被检查的硬盘分区务必**不可挂载到系统上**，即需要在卸载的状态

19. `mount`挂载（P243），启动挂载（P248）

20. 内存交换区[swap]，以前内存不足时，可以将内存中的数据暂时放到硬盘中，这部分硬盘就是内存交换区。（P252，ubuntu16已经自带了，这里看了一下没有平常能用到的）

## 第8章 文件与文件系统的压缩

简单的来说，因为文件有相当多未填满空间，压缩就是把这些空间填满来使得整个文件的占用容量下降

常见压缩命令：

1. `gzip`（替代compress）：
   1. 可解开compress，zip和gzip的压缩文件
   2. 自己压缩的文件以.gz结尾
   3. 如果原本是文本文件，可以通过`zcat/zmore/zless`来分别以`cat/more/less`读取被压缩的文本
   4. 如果从文字压缩文件中找关键字可以通过`zgrep`
2. `bzip2`（替代gzip，提供更好的压缩比）：
   1. 压缩时间比gzip更久，时间换空间
   2. 生成.bz2结尾的压缩文件
   3. 有相应的`bzcat/bzmore/bzless`
3. `xz`（替代bzip2，更高的压缩比）
   1. 生成.xz结尾的压缩文件
   2. 对应的`xzcat/xzmore/xzless`
4. 打包命令`tar`
   1. 前面的压缩指令虽然可以对目录进行压缩，但本质是对所有文件分别进行压缩的操作
   2. `tar`的几个常用option：
      1. `-c`是compress压缩；`-t`是查看文件名；`-x`是解压
      2. `-z`通过gzip解压/压缩；`-j`通过bzip2；`-J`通过xz
      3. `-v`压缩/解压过程中显示正在处理的文件
      4. `-f`表示要被处理的文件，`-C`用在解压缩
      5. `-p`保留备份数据的原本权限和属性；`-P`保留绝对路径

其余的如同xfs文件系统以及光盘刻录，一个从未用过，一个已经过时，这里不再记录了

## 第9章 vim程序编辑器

很重要，虽然用的多，但还是仔细看看有没有遗漏的

1. vi三模式：
   1. 一般命令模式（command mode）
      1. 进入就是
      2. 可以删除字符或整行
      3. 可以使用复制黏贴
      4. [0或home]可回到这一行的最开始，[$或end]可回到这一行的最后
      5. `/word`和`?word`分别从光标的之下和之上找word的匹配
      6. 范围替换
         1. `:n1,n2s/word1/word2/g`从n1行到n2行(包括n1和n2)，所有word1替换成word2
         2. `:1,$s/word1/word2/gc`从第一到最后一行的替换，最后的c表示用户confirm是否替换(会逐行询问)
      7. 删除，复制和粘贴
         1. x等于del，X等于backspace
         2. `dd`剪切掉光标这一行，`ndd`剪切掉光标向下的n行
         3. `yy`复制所在的光标哪一行，`nyy`复制光标向下的n行，p和P把复制内容粘贴到光标下一行
      8. u恢复前一个操作，`ctrl+r`重做上一个操作
   2. 编辑模式（insert mode）
      1. 一般模式下无法编辑，按下`[i, I, o, O, a, A, r, R]`后进入（我一般I进入）
      2. Esc键退出编辑模式
   3. 命令行模式（command-line mode）
      1. 一般模式下输入`[:, /, ?]`任何进入
      2. 可以读取保存（最喜欢:wq了），批量替换，退出vi，显示行号等
      3. `:w!`强制写入，`:q!`强制退出不保存
      4. `:set nu`设置行号，`:set nonu`取消行号
2. vim会在编辑文件的目录建立一个以.filename.swap命名的暂时副本，如果vim因为不正常的原因中断（如一般命令下被ctrl+z中断），之后再打开就可以继续上次的进度
3. 之前也碰见过vim警告问题，Delete（D）可以删除之前的.swap文件，之后再创建一个新的.swap文件
4. 用`v`多行选中，`ctrl+v`矩形选中，选中后：y复制，d删除，p粘贴
5. vim+多个文件名打开多文件后：
   1. `:n`编辑下个文件
   2. `:N`编辑上个文件
   3. `:files`列出被这个vim开启的所有文件
6. vim多窗口（居然还这功能我是没想到的）
   1. `sp {filename}`打开俩文件
   2. `sp`直接sp可以打开同文件里的两个窗口
   3. 打开之后通过`ctrl+w+按键`来控制，`:q`退出光标在的page
7. vim关键字补全（佛了，这也有）
   1. `ctrl+x -> ctrl+n`使用在文件中的文本进行关键字补全
   2. `ctrl+x -> ctrl+n`使用在当前目录内的文件名进行补全
   3. `ctrl+x -> ctrl+o`使用扩展名（vim内置）进行补全

## 第10章 认识与学习BASH

1. 可以通过`type [-tpa] name`来看command是内部命令（bulletin），别名（alias）还是外部命令（file）

2. 变量的使用和设置：

   1. echo可以输出变量，如`echo ${PATH}`，man看了一下，本质是展示一行文本

   2. 变量可以直接通过`var=abc`来定义，但注意以下：

      1. 变量两边不能有空格
      2. 变量名称**只能是英文和数字，且不可以以数字开头**
      3. 双引号内使用$可以保持原本的特性（类似于py里的插入）
      4. 单引号内就是纯文本（内部可用转义符）

   3. 子程序在继承父进程时只能继承环境变量，想使用父进程自定义变量使用`export PATH`

   4. 系统定义的变量一般大写，自定义小写

   5. 取消变量定义用`unset var`

   6. 反单引号`` `的意思是先执行内部的命令，比如

      ```shell
      ls -l `name` 
      ls -l ${name}
      ```

      以上俩命令本质是一样的

3. `env`命令可以看到所有的环境变量，`set`可以看所有包括自定义的变量，其中比较重要的几个变量

   1. PS1（P323）是命令提示字符
   2. $有关于本shell的PID，可用`echo $$`查看
   3. ?:是关于上个执行命令的返回值，一般成功执行返回值都是0

4. 可以用`read`指令来读取键盘或文件中的变量

5. `declare`默认把所有变量都print出来，可以`[-aixr]`来限定转换成什么类型，变量类型默认字符串

6. `ulimit`限制开启的资源总量

7. 变量内容的删除，取代与替换：

   1. 删除
      1. 书上例子`${变量#关键字} -> echo ${path#/*local/bin:}`会把**最短的匹配这个模式的字符串**删掉，*这里是通配符。##是把**最长的匹配这模式的字符串**删掉
      2. `${变量%关键字}`%就是从后向前匹配，找最短匹配这个模式字符串删掉，%%找最长的
   2. 替换：
      1. `${变量/旧字符串/新字符串}`把第一个旧字符串替换掉
      2. `${变量//旧字符串/新字符串}`把所有的旧字符串替换掉
      3. `${name-内容}`可以做到通过`-`，如果name被设置就无事发生，如果没被设置就使用内容

8. 通过`history`打印出`~/.bash_history`的内容，依序记录，但无法记录时间

9. Bash命令的执行顺序，

   1. 以绝对/相对路径执行命令
   2. 由alias找到命令
   3. 由bash内置的bulletin命令执行
   4. 通过$PATH顺序找到的第一个指令执行

10. bash的环境配置文件

    1. login shell指取得bash需要完整的登录流程，而non-login shell不需要重复操作（比如子进程）
    2. login shell会读两个文件：
       1. /etc/profile：系统整体设置，不要更改
       2. ~/.bash_profile或~/.bash_login或~/.profile，用户个人的设置，可以修改，这三个文件有先后顺序，但bash只读取其中一个，其余两个不论存在与否都不再读取
       3. 书上说~/.bash_profile会先找~/.bashrc(但ubuntu16里压根没有profile，直接就是bashrc哈哈哈哈)
    3. `source`命令读取环境配置文件，比如我就常用`source ~/.bashrc`
    4. `~/.bash_logout`记录了当注销bash后，系统还会帮我做什么操作

11. `stty`指令可以帮助查看**当前的按键内容**

12. 通配符：*表示有0或无穷多个任意字符，？表示一个任意字符，[]（如[abcd]）表示一定有一个括号内的字符，[-]（如[0-9]）表示**编码顺序内的所有字符**，[^]（如【^abc】）表示除了a，b，c以外的任意一个

13. 数据流重定向：

    1. 就像C++里，输入符号为<或<<，输出符号为>或>>，stderr为2>或2>>
    2. 比如`ls -l >> testFile`就会把原本输出到terminal的数据都输出到testFile里，如果testFile不存在也会自动创建一个（书上说会覆盖，但我多试了几次后发现ubuntu16并不会覆盖，而是直接在先用文件后面接着写）
    3. 如果想直接丢弃，可以使用黑洞设备`dev/null`如`2>> /dev/null`
    4. 如果正确和错误的信息都想写到同一个文件，可以用`2>&1`或`&>`

14. 管道命令pipe（也就是`|`符号）

    1. 管道仅能处理stdout，stderr会被忽略
    2. 必须接受来自前一个命令的stdout作为stdin继续处理才行（所以叫管道，一环扣一环）
    3. 选取命令`cut`：
       1. `cut -d '分隔字符' -f fields`，其中'分隔字符'就是类似java split方法的字符，fields就是具体取用分割后的哪几块显示
       2. `-c number`表示以number为单位取出固定的字符串区间
    4. 选取命令`grep`：
       1. cut是将某行信息中我们想要的部分分割拿出来，grep是分析一行信息，如果匹配就把整行拿出来
       2. 底层是正则，很多语法见P352
    5. 排序命令`sort`:
       1. 啥都不加就是每行为一个元素，然后比较从小到大排序
       2. 其他的见P353
    6. 排序命令`uniq`：
       1. 把重复的数据（同样按行）仅列出一个显示（底层应该是hash set）
       2. `-c`进行计数，`-i`忽略大小写字母的不同
    7. 排序命令`wc`：
       1. 计算文件中的行/word/字符
       2. 分别用`-l`行；`-w`word；`-m`字符数
    8. 双重重定向`tee`：
       1. 把数据流既给文件又给screen
       2. 用法`tee filename`，如果不加`-a`就会覆盖（这次是真的会覆盖）
    9. 字符转换命令`tr`：
       1. 比如`tr [a-z] [A-Z]`表示把所有小写变成大写
       2. `tr -d '字符'`表示把字符都删掉
    10. 字符转换命令`col -x`可以把tab都转换为空格
    11. 字符转换命令`join`可以把两个相关文件的相同部分组合在一起（P356），注意最好先sort
    12. 字符转换命令`paste`就是简单地把两行贴在一起，中间用tab隔开（可用`-d`修改分隔符）
    13. 字符转换命令`expand [-t number] `，可以指定把tab换成number个空格（tab可以被`cat -A`指令显示成`^I`）
    14. 划分命令`split`:
        1. `-b`按照文件大小分割，单位如`b,k,m`等
        2. `-l`按照行数分割
    15. 参数代换`xargs`：
        1. 就是产生某个命令的参数的意思
        2. 可以见P358的例子，好比说`id`这个命令不是管道命令，如果直接用管道的写法接收不到stdin，这时就需要xargs来作为管道命令接受stdin并将参数传递给id
    16. 用到tar时，stdin和stdout可以用-来替换

## 第11章 正则表达式与文件格式化处理

1. 语系会影响到正则的结果，因为不同语系对字符的编码排序顺序可能不同，一般使用`LANG=C`
2. `grep`的进阶：
   1. `-A`指after，后接一个数字n表示除了列出这行，还列出后续的n行
   2. `-B`指before，后接一个数字n表示除了列出这行，还列出前面的n行
   3. `-n`后接匹配用关键字
   4. `-v`表示verse，也就是打印除了匹配的以外行
   5. `-i`表示匹配时无视大小写
   6. 可以在中括号内进行查找集合字符的操作，如`'[^g]oo'`表示不要以g开头的oo形式的字符串，`[[:lower:]]123`表示只接受以小写开头后接123的字符
   7. 注意`[^g]oo`表示不要g开头，`^[a-z]oo`这里^表示行首，也就是^在[]内外的意义不同
   8. `str$`表示以str结尾的那行
   9. `.`表示一定有一个任意字符，`*`表示重复前一个字符0-无数次（这里有个trick，如果要找任意数字，可以`'[0-9][0-9]*'`，来表示一个或多个连续的digit）
   10. 通过{}限定重复范围，比如`'go\{2,5\}g'`表示重复2-5次的o
   11. P372含有这些总结
3. `sed`工具
   1. 本身是一个pipe命令，功能很多，一个个来
   2. d表示删除，如`cat test | sed '2,5d'`表示把2-5行都删掉
   3. a表示增加，如`cat test | sed '2a Test'`会在第二行后加上**新的一行**，内容为Test；如果把a换成i，那就会在第二行的前面加上新的一行
   4. c表示替换，如`cat test | sed '2,5c replacement'`表示把删掉的2-5行都替换成replacement
   5. 范围取用使用-n（几乎等同于`head -n num1 | tail -n num2`）和正则p，如`cat test | sed -n  '2,5p'`表示取用2-5行
   6. 关键词查找与替换`sed 's/要被替换的的字符/新的字符/g'`，里面用正则就可以
   7. 可以用`-i`直接对文件修改
4. 正则的RE字符
   1. +重复**一个或以上**的前一个RE字符，也就是之前的`oo*`这么一个写法等同于`o+`
   2. ?表示**零个或一个**前一个RE字符
   3. |表示用**或**列出数个匹配用字符串（有一个匹配上就成功）
   4. ()找出**群组**字符串，比如`egrep -n 'g(la|oo)d' test`表示glad或good都算匹配字符串
   5. ()+表示多个重复群组的判别，比如`echo 'AxyzxyzC' | egrep 'A(xyz)+C'`表示开头A，结尾C，中间任意个重复xyz就算匹配
5. `awk`工具
   1. 相比于sed往往对一整行处理，awk往往是把一整行分成数个字段来处理，适合小文本
   2. 三个内置变量：
      1. NF：每一行（$0）拥有的字段总数（一整行是$0，$1是分割出的第一个字段，剩下的都是以此类推）
      2. NR：目前awk处理的是第几行
      3. FS：目前的分隔字符，默认为空格，修改分隔符可以用，如`{FS=':'}`
   3. 注意awk后续所有的操作都要在''中，用到printf中的格式需要用“”括住
   4. 注意awk的操作如`cat /etc/passwd | awk '{FS=":"} $3 < 10 {print $1 "\t " $3}'`改变分隔符从第二行开始才起效，如果希望从第一行就起效，需要加上BEGIN关键字，如`cat /etc/passwd | awk 'BEGIN {FS=":"} $3 < 10 {print $1 "\t " $3}'`
   5. awk还能写条件分支，如P381例子，和C的语法类似，就是不需要if，如同上例，直接把条件和{}连在一起就是一个条件分支
6. `diff`
   1. 对比文件间的差异，以行为单位，一般用在**同一文件或软件的新旧版本上**
   2. `diff [-bBi] from-file to-file`
   3. -b忽略多空格差异，-B忽略空白行差异，-i忽略大小写差异
7. `cmp`以字节为单位，把不同处的字节序号和行数打出来
8. `patch`，diff可以通过>制作一个.patch文件，patch命令可以使用.patch文件把旧文件更新或把新文件退版本(-R)

## 第12章 学习shell脚本

1. shell脚本就是用shell的功能写一个程序

2. shell的一些内部事项和C类似，但注意如果分行写需要`\[Enter]`而不是像C一样直接`[Enter]`就行的，注释使用`#`

3. `/bin/sh`就是`bin/bash`的链接文件，使用`sh shell.sh`就是告诉系统用bash来执行文件中的命令，此时shell.sh只要有r权限就能执行

4. 在shell文件中，一般

   1. 第一行注释表明该脚本使用的shell名称
   2. 第二行开始注释写明程序内容（功能，版本，作者信息，日期，历史记录等）
   3. 主要环境变量的声明，这样程序执行时可以直接用外部命令而不需要再写绝对路径
   4. 主要程序部分
   5. 执行结果告知，即定义返回值（这里用exit代替C中的return）

5. bash shell只支持整数运算，可以使用`var=$((计算式))`来完成计算

6. 如果需要小数点计算，可以使用bc命令帮助，比如`echo "12.12*1.34" | bc`

7. 脚本的执行方式差异：

   1. 直接执行方式（./script或sh(bash) script）：都会使用一个**新的bash环境**来执行脚本内的命令（本质是在子进程的bash环境中执行），但**当子进程结束后，在子进程中的各项变量并不会传回父进程中**
   2. 使用source来执行脚本：这就是在父进程中直接执行了

8. `test`命令测试P396，可以像C中的?:语句一样对于不同情况用不同的echo语句，具体可以使用&&和||符号

9. 判断符号[  ]，因为[]经常和正则与通配符一起使用，所以用作判断时**需要在判断式和中括号的两边加上空格**

10. shell的默认变量

    1. `$#`表示后接参数的个数，书上例子为4
    2. `$0`一般就是脚本名字
    3. `$@`代表["$1" "$2" "$3" "$4"]，每个变量独立
    4. `$*`表示["$1 $2 $3 $4"]
    5. 可以用`shift [num]`命令直接移动变量，移动会把前面num个参数隐藏（就像移动指针）

11. 条件判断式

    1. `if...then`

       1. 大概的判断写法如下：

          ```sh
          if [条件判断，可用||和&&]; then
          	...
          elif [条件判断]; then
          	...
          else
          	...
          fi
          ```

          总的来说和py写法相近

    2. `case...esac`判断

       1. 格式如下：

          ```shell
          case $var in
            "第一个变量内容") # 每个变量内容都用""括起来，并加上右圆括号
            	程序段
            	;;			  # 每个程序段结束需要加上;;
            "第二个变量内容")
            	程序段
            	;;
            *)			  # *类似于default
            	exit 1
            	;;
          esac
          ```

    3. `function`功能

       1. 在shell脚本中执行从上至下从左至右，所以function代码块必须在程序最前面，如：

          ```shell
          function fname() { 
          	程序段
          }
          ```

          注意这个地方的()，在ubuntu上使用`sh file.sh`不行，`./file.sh`可以，[stackoverflow](https://askubuntu.com/questions/834853/ubuntu-bash-functions-syntax-error-or-unexpected)上的解释是使用sh时ubuntu会使用`/bin/sh`而不是bash

          通过`./show123-2.sh one`来使用

       2. function内有内置变量，`$0`是函数名，`$1,$2...`是后面接的输入变量

12. 循环

    1. `while do done`以及`until do done`(不定循环)

       ```shell
       while [ condition ]
       do
       	...
       done
       #========
       until[ condition ]
       do
       	...
       done
       ```

    2. `for...do...done`（固定循环）

       ```shell
       for var in con1 con2 con3 ...
       do
       	...
       done
       ```

       连续的数字可以用`for var in $(seq 1 100)`

       也可以C-like的写法`for ((初始值; 限制值; 赋值运算))`

13. debug时可以使用`sh -x script.sh`命令，可以精准定位到哪一行出问题了

## 第13章 Linux账号管理与ACL权限设置

1. 用户账号的一些特点：
   1. UID 0 是系统管理员，不一定只有root
   2. UID 0~999是系统账号，和其他的非root账号相比**没有特权**，更多是一个习惯
   3. UID 1000~60000可登录账号
   4. 家目录就是用户登陆以后的第一个目录
2. `head -n 4 /etc/passwd`可以看到第二个部分都是x，那是密码，其实都放在了`/etc/shadoow`中，shadow中九个字段的用途见P421
3. 用户组可在`/etc/group`中查询，意义见P424
4. `groups`可以看当前用户有那些用户组
5. 看有效用户组就建立一个文件，然后看权限就好了
6. `newgrp`可以完成有效用户组的切换（必须切换到已有支持的用户组）
7. 账号管理：
   1. 新增用户`useradd [-u UID] [-g 初始用户组] [-G 次要用户组] [-nM] [-c 说明栏] [-d 目录绝对路径] [-s shell] 使用者账号`。注意系统账号不会有家目录，一般账号会有
   2. `passwd`用法
      1. `-S`可以看当前的帐号状态
      2. `-x [num]`可以决定num天以后密码失效
      3. `-i [num]`可以决定在密码过期后num天账号失效
      4. 想要让一个账号暂时无法登录，可以让他变得不合法`passwd -l usr`，通过`passwd -S usr`可以看到状态变为L（LOCK，书上是LK，系统不同），这时看`grep usr /etc/passwd`可以发现密码加上了一个!（书上是俩!!）
   3. `chage`可以更详细的显示密码参数，除此之外还可以**让用户第一次登陆的时候，强制要先修改密码**，通过`chage -d 0 usr`，让usr的初始密码设置在1970-1-1，强制需要更改一次
   4. `usermod`命令来修改，具体见P433
   5. `userdel`命令可以删除用户的相关数据，`-r`可以连同home目录一起删
8. ubuntu16虽然默认不支持`finger`命令，但支持`chfn`(change finger)命令，进去以后就是和书上一样设置phone number之类的参数。一般在主机有很多用户时才会使用
9. `groupadd`，`groupmod`，`groupdel`就和上面的user那一套类似
10. `gpasswd`可以使用用户组管理员功能
    1. `gpasswd [-A user1,...] [-M user3,...] groupname`
    2. `-A`可以设置管理员，可以有多个管理
    3. `-M`把账号加入这个用户组
    4. 更多详细功能见P438
11. ACL
    1. Access Control List
    2. `setfacl`来设置权限
       1. 可以用`setfacl -m u:账户:rwx file`来改变文件的权限（`g:用户组:rwx`改用户组）
       2. 使用acl改变完权限后参数后会有一个+号，需要使用`getfacl`才能看到真实的权限
    3. `getfacl`获取某个目录的ACL规范
       1. 可以用`getfacl file`直接看
12. PAM模块
    1. 三个字段组成一个验证过程
    2. 第一个字段验证类别Type（有先后验证顺序）：auth, account, session, password
    3. 第二个字段验证的控制标识（control flag）：
       1. required：成功就带有success标识，失败就带有failure标识，但不中断后续的验证流程，有利于log，最常用
       2. requisite：失败立刻返回failure标识，终止验证流程
       3. sufficient：成功就立刻返回success标识，失败继续流程，和requisite刚好相反
       4. optional：大都是为了显示信息而不在控制验证上
    4. P455有详细的pam文件的说明
13. 可以用`w`，`who`查看系统上的所有用户，`lastlog`看最后登录时间
14. P458用户对谈的用法

## 第14章 磁盘配额（Quota）与高级文件系统管理

1. 软限制与硬限制（soft/hard）：hard表示用户一定不会超过这个值，超过直接锁定用户的磁盘使用权；soft表示超过了soft设定的值以后会给一定时间的grace time让用户自己降低使用量，如果grace time过了仍然没有降低，那么soft设定的量就会直接变成hard（也就是锁定使用权）
2. 这里书上的是xfs文件系统，ubuntu16是ext4，这里的配额我就看了个大概
3. 软件磁盘阵列（software RAID），根据level不同功能也不同：
   1. RAID 0(性能最佳)：先将磁盘切割出等量的数据块chunk，一般大小在4KB-1MB，当文件写入RAID时，会根据chunk的大小切割好再依序放入RAID。数据会被**等量地放在各个磁盘上**。这样也会导致**越多磁盘组成的RAID 0性能会越好，但只要任何一块磁盘损坏，在RAID上面所有的数据都会遗失无法读取**
   2. RAID 1（镜像模式，mirror）：就是同一份数据会保存两份，虽然有备份，但容量减半
   3. RAID 1+0, RAID 0+1：综合上两种方法，先让两个磁盘组成RAID 1，并设置两组；再将两RAID 1组成一个RAID 0（目前最推荐）
   4. RAID 5：性能和备份的综合考虑。需要三块以上磁盘组成，文件写入就像RAID0，但每次写入时，每块都会写进一个**奇偶校验数据parity**，如果一块磁盘损坏了，能够靠另外两个的奇偶校验检查码来重建。但如果三块坏了两块就不行了。这种方法读取速度可以，但**写入需要计算校验码所以慢**
   5. RAID 6：就是RAID 5的进阶。它直接从总体中拿两块磁盘作为写入奇偶校验数据的载体
   6. Spare Disk热备份磁盘：让系统实时地在硬盘坏掉时重建
4. 软件RAID中，依靠软件而不是硬盘本身来实现RAID地算法特性，centOS中自带`mdadm`命令，但ubuntu中并没有，这里我只做一个了解，因为现在硬件其实越来越便宜，软件实现往往不是那么有必要
   1. `mdadm --create /dev/md[0-9] --auto=yes --level[015] --chunk=NK`
5. 逻辑卷管理器（Logical Volume Manager）
   1. 重点在于可**弹性地调整文件系统的容量**，整合多个物理分区，使其使用起来就像一个磁盘一样
   2. 一些概念：
      1. 物理卷（Physical Volume， PV）：最底层的卷
      2. 卷组（Volume Group，VG）：LVM大磁盘就是把多个PV整合成这个VG，64位Linux上VG几乎没有大小的限制
      3. 物理扩展块（Physical Extent， PE）：PE就类似于文件系统中的block大小，会影响到LVM的最大容量
      4. 逻辑卷（Logical Volume，LV）：最终VG会被切割成LV，LV就是最后可以被格式化作为类似分区使用的东西
   3. 上述概念从底层到顶层的抽象层次，P486直接提供了流程所需的方法，这里就不多赘述了

## 第15章 计划服务（crontab）

1. 计划服务种类
   1. `at`是突发性的，执行一次就结束（ubuntu16需要手动安装）
   2. `crontab`是例行性的，每隔一定周期都会完成
2. 仅一次的计划任务
   1. 在使用前，需要确认atd服务已经启动，`systemctl status atd`
   2. 并非所有用户都能用at，防止hacker：
      1. 如果`/etc/at.allow`存在，可以看这个文件中的用户，在内部才能用at
      2. 如果`/etc/at.deny`存在，可以看这个文件中的用户，**不在**文件内才能用at
      3. 如果俩都不存在，那么就是只有root可以使用at
   3. 注意使用at时，最好用绝对路径来执行命令，因为at在运行时，会跑到执行at命令的那个工作目录
   4. 由于系统会将该项at任务独立出你的bash环境，直接交给atd程序接管，因此执行at任务后就可以脱机了
   5. 管理at任务：`atq`查看at任务队列，`atrm (jobnum)`来rm掉at任务
   6. `batch`命令，用法和at类似，可以让系统有空时（负载小于0.8）再执行我们的后台任务
3. 循环任务：
   1. 类似于at，可以查看`/etc/cron.allow`或`/etc/cron.deny`来确定用户是否能够使用cron（奇怪的是ubuntu16上能看at但看不到cron的）
   2. 具体时间可以参见P507的特殊字符表和数字代表的时间（分时日月周），这就组成了语法部分（*表示任意时间，/后面跟间隔，都很重要）
   3. 个人操作使用`crontab -e`，能保证任务不会被大家看到
   4. 系统维护管理使用`vim /etc/crontab`，便于管理，容易追踪
   5. 自己开发软件用`vim /etc/cron.d/newfile`
4. `anacron`可以检测因为某些原因而超过时间却未执行的任务，流程如下
   1. 由`/etc/anacrontab`分析出cron.daily这项任务天数为一天
   2. 由`var/spool/anacron/cron.daily`取出最近一次执行anacron的timestamp
   3. 如果差异时间超过一天，就准备执行命令
   4. 根据`/etc/anacrontab`的设置延迟后执行
   5. 执行结束后，anacron程序结束

## 第16章 进程管理与SELinux初探

1. 登录并执行bash时，系统已经根据用户给了一个PID
2. fork and exec程序调用的流程：
   1. Linux的程序调用一般就是fork-and-exec流程
   2. 进程借由父进程fork一个子进程，子进程再以exec的方式来执行实际执行的进程
3. 系统或网络服务常驻于内存中，在后台启动并一直持续运行， 常被称为**daemon（服务）**
4. `cp file1 file2 &`&就可以把命令挂在后台执行（面试的时候才知道的，当时还以为是nohup命令的一部分）
5. 执行任务管理的操作中，每个任务都是当前bash的子进程，彼此间有关联，我们无法用任务管理的方式由tty1环境去管理tty2的bash
6. 默认状态下使用`ctrl+z`会使job暂停，可以使用`jobs -l`命令看到，带有+号的是默认使用任务，带-号的是第二个被放到后台的任务，第三个以后的任务就不再带有+，-号了
7. `fg`（foreground命令）可以把后台任务拿到前台处理，`fg %jobnumber`，也可以`fg +/-`来指定第一个和第二个进后台的程序拿到前台
8. `bg %jobnumber`可以后台跑后台程序，可以观察到被跑的任务后加上了一个&表示在后台运行
9. `kill`起到给信号管理后台任务的作用（`kill -l`看信号，32稳定32不稳定），特别留意-9是强制结束任务，-15（默认值）是按正常步骤结束一个任务。用法`kill -signal %jobnumber`
10. 但目前用&挂起的后台本质是bash的后台而非系统的后台，这意味着如果远程连linux服务器然后脱机任务还是会被中断，所以需要`nohup [命令与参数]`注意nohup不支持bash内部命令，需要用外部命令来，脱机不会被打断，输出会被重定向到`nohup.out`中
11. 查看进程：
    1. 静态的`ps`：
       1. 把某个时间点的进程运行情况截取
       2. `ps -l`看自己bash进程，`ps aux`看系统运行进程
       3. `ps -l`显示的信息解析见P525
       4. `ps aux`显示的信息解析见P526
       5. 僵尸进程除了进程标识为Z以外，进程CMD后会出现`<defunct>`，因为父进程无法完整的结束子进程
    2. 动态的`top`：
       1. top可以持续监测进程运行的状态
       2. `top [-d 数字] | top [bnp]`，-d后接多少秒更新一次，-b通常与重定向有关，-n通常和-b一起，表示把多少次的top结果重定向，-p指定PID来进行检测
       3. P527有详细操作，进入top界面由很多操作，比如面试重点M按内存使用率排序，r修改nice值等
    3. `pstree`
       1. `pstree [-A|U] [up]`
       2. -A让进程树的链接使用ascii码，-U则是unicode字符（某些终端可能会乱码）
       3. -p同时显示各个PID，-u同时显示各个进程所属账号名称
12. 因为kill命令需要知道PID，往往和ps，pstree等命令一起使用。如果只想对命令产生的进程发信号，可以用`killall [-iIe] [command name]`指令（见P531），注意发送的信号会对系统中所有执行这个command的进程起作用
13. PRI(priority)由内核动态调整，用户无法直接调整，但用户可以通过调整nice值**影响**PRI：
    1. nice范围-20~19
    2. 当nice为负值时，进程会降低PRI，使其较优先被处理
    3. root可以随意调整自己或他人进程的nice值
    4. 一般用户只能调整自己的nice值，且只能在0-19范围之内（避免一般用户抢占系统资源）
    5. 一般用户仅能**将nice值越调越高**
    6. 一开始就给进程一个特定nice值，使用`nice [-n 数字] command`
    7. 后续调整用`renice [number] PID`
14. `free`查看内存使用情况
15. `uname`看系统和内核相关信息（具体见P534，一般用`uname -a`看所有信息）
16. `uptime`看系统启动时间和任务负载
17. `netstat [-atunlp]`追踪网络或socket文件，这个比较重要，还是在这里记一下：
    1. `-a`列出所有链接，监听，socket
    2. `-t`是TCP网络封包信息，`-u`是UDP网络封包信息
    3. `-n`是不以进程的服务名称，以端口号来显示
    4. `-l`列出目前正在listen的服务
    5. `-p`列出网络进程的PID
    6. internet的情况我也比较熟悉这里就不赘述
    7. 除了internet，linux的进程可以通过socket文件通信，输出的字段有
       1. Proto：一般是unix
       2. RefCnt：连接到此socket的进程数量
       3. Flags：连接的标识
       4. Type：socket存取的类型，主要有确认连接的STREAM和不需要确认的DGRAM两种
       5. State：若为CONNECTED则表示多个进程之间已经建立连接
       6. Path：链接该socket相关进程的路径或相关输出的路径
18. `dmesg`分析内核产生的信息
19. `vmstat [-a] [延迟[总计检测次数]]`检测系统资源变化（CPU/内存/磁盘IO状态等，具体使用见P536），字段如下：
    1. procs字段
       1. r：等待中进程数量；b：不可唤醒的进程数量
       2. 这两越多代表进程越繁忙
    2. memory字段
       1. swpd：虚拟内存被使用的数量
       2. free：未被使用的内存容量
       3. buff：缓冲存储器的数容量
       4. cache：用于高速缓存的容量
    3. swap字段：
       1. si表示由磁盘中将进程取出的容量
       2. so表示由于内存不足而写入swap的容量
    4. 磁盘读写I/O字段：
       1. bi表示由磁盘读入的区块数量
       2. bo表示写入磁盘的区块数量
       3. 这俩越高表示系统I/O越忙碌
    5. 系统system字段：
       1. in：每秒被中断的进程次数
       2. cs：每秒执行的事件切换次数
       3. 数值越大，表示系统与外界设备沟通越频繁
    6. CPU字段
       1. us：非内核层的CPU使用状态
       2. sy：内核层的CPU使用状态
       3. id：闲置的状态
       4. wa：等待I/O所耗费的CPU状态
       5. st：被虚拟机所使用的CPU状态
20. SUID/SGID的权限
    1. SUID
       1. 其实与进程相关性非常大
       2. SUID只能对二进制bin文件起效
       3. 执行者对程序必须有x权限
       4. 本权限仅在该程序的执行过程中有效（runtime）
       5. 执行期间，执行者具有该程序owner的权限
       6. 比如passwd以后就有了root权限，这是因为在触发passwd后，会获得一个**新的进程和PID**
       7. 可以通过`find / -perm /6000`来找整个系统中的SUID/SGID文件
21. `/proc/*`的意义
    1. 各个进程的PID都是以目录形式存在于`/proc`中
    2. 每个进程的目录里都有对应的files，这些文件的含义见P539
22. `fuser [-umv] [-k [i] [-signal]] file/dir`可以看到正在使用文件或dir的进程
23. `lsof`则可以列出进程正在使用的文件名称
24. SELINUX：
    1. 就是security enhanced Linux，因为企业发现系统出问题往往是内部员工的资源误用
    2. 传统的文件权限与账号为自主访问控制（DAC），之前的那些根据进程拥有者和rwx权限的那些就是DAC
    3. SE引入了强制访问控制（Mandatory Access Control，MAC），可以针对特性的进程和文件资源管理权限，打个比方就是你是root也不能为所欲为
    4. 主体subject：进程；目标object：一般是文件系统；策略policy：规定资源读写的rule
    5. 三种策略：
       1. targeted：针对网络服务的较多，针对本机限制少，默认策略
       2. minimum：由target自定义而来，仅对选择的进程进行保护
       3. mls：完整的SELinux限制，比较严格
    6. `ls -Z`可以看到三字段`Identity:role:type`
       1. Identity：一般有unconfined_u（不受限user）；system_u（系统用户）
       2. Role：object_r（代表文件或目录，最常见）；system_r（代表进程或一般用户）
       3. Type（最重要）：在文件资源Object上被称为Type；在主题进程Subject上被称为Domain。Domain需要和Type搭配来读取文件资源
       4. （ubuntu16里只能看到一个字段...）
    7. 三种模式：
       1. Enforcing：强制模式，表示已经正确开始限制domain/type
       2. Permissive：宽容模式，仅有警告信息不影响实际读写，可用作SE的debug
       3. Disabled：关闭模式
    8. `setenforce`指令可以在Permissive和Enforcing之间转换，`getenforce`指令可以查看
    9. ubuntu16的selinux启动和书上的大相径庭，详见这个链接[enable SELinux](https://linuxconfig.org/how-to-disable-enable-selinux-on-ubuntu-20-04-focal-fossa-linux)
    10. 对SELinux里的规则查看和修改分别用`getsebool`和`setsebool`
    11. **因为这里我尝试着往ubuntu上安装selinux并且重启后，机子直接裂开了**，我这里参考了[stackoverflow sol1](https://askubuntu.com/questions/1180868/after-installed-selinux-system-is-stuck-and-not-booting)和[stackoverflow sol2](https://askubuntu.com/questions/141606/how-to-fix-the-system-is-running-in-low-graphics-mode-error)才终于解决了。看大佬们说**ubuntu和selinux天生八字不合**，这之后SELinux部分我就没跟着搞了

## 第17章 认识系统服务（daemon）

1. systemd的好处（见P566）
2. systemd将过去所谓的daemon执行脚本统统称为一个服务单位（unit），根据功能划分为不同的类型，包括：系统服务，数据监听与交换的socket文件服务，存储系统状态的快照类型，提供不同运行级别的操作环境（target）等，都放在以下目录中：
   1. `/usr/lib/systemd/system`：每个服务最主要的启动脚本设置，有点类似以前的`/etc/init.d`
   2. `run/systemd/system`：执行过程中产生的脚本，优先级**高于**前一种
   3. `/etc/systemd/system`：管理员根据主机需求建立的执行脚本，优先级又更高一些
3. 可以根据扩展名判断unit的类型：
   1. .service：一般服务类型，一般是系统服务和网络服务
   2. .socket：内部程序数据交换的socket服务（IPC）
   3. .target：执行环境类型，一群unit的集合
   4. .mount/.automount：文件挂载相关的服务
   5. .path：检测特定文件或目录类型（某些服务需要特定目录来提供队列服务，如打印服务）
   6. .timer：循环执行的服务，类似anacrontab，但由systemd提供，更有弹性
4. 除了kill命令外，`systemctl [command] [unit]`也是管理unit的良好手段（P568罗列了command）。比如我们可以使用`systemctl stop atd.service`来关闭，一个正常的服务不该用kill指令关闭，因为关闭后systemctl**无法继续监控该服务了**
5. Active状态：
   1. running：正有一个或多个服务正在系统中运行
   2. exited：仅执行一次就结束的服务
   3. waiting：正在运行中，不过还需要等待其他事件发生才能继续运行（打印服务）
6. daemon默认状态：
   1. enabled：这个daemon将在开机时被运行
   2. disabled：这个daemon开机时不会被运行
   3. static：这个daemon不可以自己启动（不能enable），但可能被其他enabled的服务唤醒（依赖属性的服务）
   4. mask：这个daemon无论如何都无法启动，因为已经被强制注销（非删除）。可被`systemctl unmask`方法改回默认
7. 查看系统上所有的服务：
   1. 命令`system [command] [--type=TYPE] [--all]`
   2. command：
      1. `list-units`依据unit显示目前启动的unit（加上--all可以把没启动的也列出来）
      2. `list-unit-files`依据`usr/lib/systemd/system/`内的文件
   3. 可以根据type来调整，比如我只想看daemon，那么可以`type=service`
8. target unit：
   1. 可以通过`systemctl list-units --type=target`来查看
   2. 正常模式使用`mutil-user.target`和`graphical.target`
   3. 恢复模式用`rescue.target`（systemd启动时增加一个额外的临时系统，可以取得root来维护原系统）和`emergency.target`（更紧急）
   4. 修改可提供登陆的tty数量，可修改`getty.target`
   5. command参数：
      1. get-default：获得目前的target
      2. set-default：更改后面接的target为默认
      3. isolate：切换到后面接的模式（service部分用的start，stop，restart等参数）
9. 通过systemctl看服务之间的依赖性：
   1. `systemctl list-dependencies [unit] [--reverse]`，reverse指反向追踪谁依赖这个unit
10. P578举了一个vsftpd的配置文件目录意义的例子，P579则提供了unit部分参数意义表，P580提供了Service的参数意义表，如果需要配置service可以参考
11. getty的@是1-6（范式）的一种缩写，如果需要修改tty的个数，可以参见P583对`/etc/systemd/logind.conf`进行修改

## 第18章 认识和分析日志文件

1. 针对日志文件需要的服务和程序：

   1. `systemd-journald.service`：最主要的信息记录者，由systemd提供
   2. `rsyslog.service`：主要收集登陆系统和网络等服务的信息
   3. `logrotate`：主要进行日志文件的轮循功能（旧日志满了开个新的空的日志，等一段时间后没被用到，旧日志文件就会被删）

2. 日志内容的一般模式：

   1. 事件发生的日期与时间
   2. 发生此事件的主机名
   3. 启动此事件的服务名称（systemd，crond等）或命令与函数名称（如su，login等）
   4. 该信息的实际内容

3. `/etc/rsyslog.conf`就是rsyslog的配置文件，规定了

   1. 什么服务（见P599表，或使用`man 3 syslog`查询）

   2. 的什么等级的信息（见P600表，从0-6严重性依次降低，debug（7）和none是两个特殊等级，用作debug和隐藏部分错误信息的调试）

      等级信息前还有连接符号：

      1. `.`代表比后面严重的信息（>=），就会被记录
      2. `.=`代表所需要的仅仅是后面接的等级，其他的不要
      3. `.!`代表除了后面接的等级，其他都要

   3. 需要被记录在哪里（设备或文件），在目标文件前加上减号`-`意味着由于信息产生较多，希望能先存储在速度较快的buffer中，数据量够大了一次性写入磁盘

4. 对于logrotate而言，`/etc/logrotate.conf`才是主要的参数文件，`/etc/logrotate.d/`下的文件会被主动读入.conf中使用（这点和ubuntu上的rsyslog的布置是一样的）

5. 可以通过`cat /etc/logrotate.conf`看轮循的配置文件，参数意义如下：

   1. 第一个参数是周期，`weekly`或`monthly`都有
   2. 第二个参数是创建一个新的日志文件create语句
   3. 第三个`rotate number`表示最大由number个日志文件
   4. 还有一个compress参数如果文件太大可以用

6. 通过`cat /etc/logrotate.d/syslog`来观察如何设置它的轮循(书上的，ubuntu16的文件内容大相径庭，压根没的syslog)

   1. 根据这个网站[ubuntu logrotate](https://www.digitalocean.com/community/tutorials/how-to-manage-logfiles-with-logrotate-on-ubuntu-16-04)，其实书上的说法也相近，只是.conf中的那些参数也可以一并写进logrotate.d的文件的{}中

   2. 正确写法为

      1. 文件名：被处理的日志文件的绝对路径，可用空格分开多个文件
      2. 文件后接轮循的参数，要用{}括起来
      3. 执行脚本需要和`sharedscripts`以及`endscripts`结合使用，可用的环境为
         1. prerotate：在启动logrotate前进行的命令（如修改文件属性）
         2. postrotate：在做完logrotate之后的命令（如重新启动`kill -HUP`某服务）

   3. ubuntu的示例如下：

      ```
      /var/log/example-app/*.log {
          daily
          missingok
          size                            // 规定大于某个大小开始轮循
          rotate 14
          compress
          notifempty
          create 0640 www-data www-data
          sharedscripts                   // 执行脚本的开始
          postrotate
              systemctl reload example-app
          endscript
      }
      ```

7. `journalctl`查看登录信息的详解见P611，这里不多赘述

## 第19章 启动流程、模块管理与Loader

1. 流程大致如下：

   1. 加载BIOS的硬件信息和进行自检，根据设置取得第一个可以启动的设备
   2. 读取并执行第一个可启动设备中的MBR引导程序（第一章提到过，也就是grub2等程序）
   3. 根据启动引导程序的设置加载kernel，kernel会开始检测硬件和加载驱动程序
   4. 硬件驱动成功后，kernel会主动调用systemd程序，并以default.target流程启动
      1. systemd执行sysinit.target初始化系统以及basic.target准备操作系统
      2. systemd启动multi-user.target下的本机与服务器服务
      3. systemd执行multi-user.target下的/etc/rc.d/rc.local文件
      4. systemd执行multi-user.target下的getty.target及登陆服务
      5. systemd执行graphical需要的服务（昨天ubuntu装SELinux就是倒在了这一步，后面靠着tty1起死回生）

2. 启动引导程序就是boot loader，BIOS通过INT 13终端功能读取，最主要的功能是识别操作系统的文件格式（和第一章重复的部分就不多说了），最终目的是**加载内核文件**

3. 当内核文件经由boot loader完成读取后，Linux会把内核**解压到**内存中，然后开始测试与驱动各个设备（BIOS后又一次自检），这次之后内核就完全接管了BIOS的工作了

4. Linux内核通过动态加载内核module，这些放在`/lib/module`内（/lib和/从来不能放在俩分区），因此在启动过程中**内核必须要挂载根目录**，这样才能读取内核module提供的加载驱动程序的功能。

   但USB,SATA,SCSI等设备的驱动往往也是module，如果不先读取这些驱动的module，内核连这些设备是啥都不知道，遑论挂载根目录了

   所以有了虚拟文件系统（Initial RAM Disk/Filesystem，在ubuntu16里是`initrd.img-4.15.0-136-generic`），特点是能够通过boot loader加载到内存中，然后这个文件**会被解压并模拟成一个根目录**，此模拟在内存中的文件系统能够提供一个可执行程序，可通过该程序来加载**启动过程中最需要的module**，比如USB, SATA, SCSI等

   加载完成后会帮助内核重新调用systemd，最终释放虚拟文件系统

5. 第一个程序systemd分析

   1. 首先连接到`usr/lib/systemd/system`里读取`multi-user.target`或`graphical.target`中的任意一个，如果读到graphical，那么根据unit.wants，系统会发现必须先启动multi-user，multi-user又依赖于`basic.target`，basic又在`sysinit.target`之后，所以最后还是一个顺序，可以通过`systemctl list-dependencies graphical.target`看这个顺序（Ubuntu16和书上的CentOS差的挺大的...）：

      ```c++
      ● ├─accounts-daemon.service
      ● ├─apport.service
      ● ├─grub-common.service
      ● ├─irqbalance.service
      ● ├─lightdm.service
      ● ├─ondemand.service
      ● ├─speech-dispatcher.service
      ● ├─systemd-update-utmp-runlevel.service
      ● ├─ureadahead.service
      ● ├─vmware-tools-thinprint.service
      ● ├─vmware-tools.service
      ● └─multi-user.target
      ●   ├─anacron.service
      ●   ├─apport.service
      ●   ├─atd.service
      ●   ├─avahi-daemon.service
      ●   ├─cron.service
      ●   ├─cups-browsed.service
      ●   ├─cups.path
      ●   ├─dbus.service
      ●   ├─dns-clean.service
      ●   ├─grub-common.service
      ```

   2. 所以把unit enable的本质就是把unit放到`multi-user.target.wants`的目录中做链接

   3. 系统完成启动后，还想让其完成**额外的程序执行**，可以写入`/etc/rc.local`中

6. 两个地方处理模块加载：

   1. `/etc/modules-load.d/*.conf`：单纯要内核加载模块的位置（无参数）
   2. `/etc/modprobe.d/*.conf`：可以加上模块**参数***的位置

7. 内核模块放置于`/lib/modules/4.15.0-136-generic/kernel/`中，具体文件的含义见P631，可以用`depmod`建立一个文件，内含模块的依赖性。

8. `lsmod`可以看目前内核加载了多少module，显示名称，大小，和used by。看module的具体信息用`modinfo [-adln]`（author，description，license，路径）

9. 模块的加载与删除：

   1. `modprobe`命令（推荐）：自动查找module.dep的记录，能够解决模块依赖的问题
   2. `insmod`加载/`rmmod`卸载：需要自己找到完整的文件名，并且一旦产生依赖性问题，无法加载或删除模块

10. 因为MBR只有446B太小了，往往MBR仅安装boot loader的最小主程序，第二步再利用boot loader去加载所有的配置文件和环境参数文件（系统定义与主要配置文件grub.cfg）

11. grub2对硬盘代号的设置：

    1. (hd0,1)：一般的默认语法，由grub2自行判断
    2. (hd0,msdos1)：传统的MBR式分区
    3. (hd0,gpt1)：GPT模式
    4. 硬盘代号用()括起来，硬盘就是hd+number，以查找顺序作为number，第一个被找到的是0，第二个是1。但每块硬盘的第一个分区代号为1（参见P636表的例子，很好理解）

12. 因为grub2不支持直接对grub.cfg进行编辑，可以通过编辑`/etc/default/grub`这个环境配置文件，之后再使用`grub2-mkconfig`来重建grub.cfg，参数意义见P638。可以用`menuentry`来划定一个系统的参数，然后在`GRUB_DEFAULT`中设定使用哪套menuentry的参数

13. P641展示了通过chainloader交出loader控制权的方法（可以作为先安装linux后安装win的一个解决方法）

14. 可以通过在`/etc/grub.d/*`下创建一个`01_users`的文件来加入不同用户需要输入密码获得不同操作系统的判断（详见P648），然后再在40_custom中创建menuentry时加上`--user`的选项

15. 

