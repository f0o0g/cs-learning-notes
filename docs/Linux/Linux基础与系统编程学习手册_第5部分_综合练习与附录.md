<a id="part-5"></a>
# 第五部分：综合练习与附录

> 本册为《Linux 基础与系统编程学习手册》五册套装中的第 5 册（共 5 册），覆盖第 1—5 章。

## 分册导航

- [第 1 部分：Linux 命令与日常运维](Linux基础与系统编程学习手册_第1部分_Linux命令与日常运维.md)（第 1—11 章）
- [第 2 部分：编辑、编译、调试与构建](Linux基础与系统编程学习手册_第2部分_编辑编译调试与构建.md)（第 1—4 章）
- [第 3 部分：Linux 系统编程](Linux基础与系统编程学习手册_第3部分_Linux系统编程.md)（第 1—2 章）
- [第 4 部分：操作系统内存管理](Linux基础与系统编程学习手册_第4部分_操作系统内存管理.md)（第 1 章）
- **第 5 部分：综合练习与附录（本册，第 1—5 章）**

## 阅读路径

- 初学命令行：依次学习[第一部分](Linux基础与系统编程学习手册_第1部分_Linux命令与日常运维.md)、[第二部分](Linux基础与系统编程学习手册_第2部分_编辑编译调试与构建.md)，再完成[第五部分](Linux基础与系统编程学习手册_第5部分_综合练习与附录.md)第 1 章对应练习。
- 学习系统编程：先掌握[第二部分](Linux基础与系统编程学习手册_第2部分_编辑编译调试与构建.md)第 2—4 章，再进入[第三部分](Linux基础与系统编程学习手册_第3部分_Linux系统编程.md)。
- 复习操作系统概念：阅读[第四部分](Linux基础与系统编程学习手册_第4部分_操作系统内存管理.md)，并用[第四部分](Linux基础与系统编程学习手册_第4部分_操作系统内存管理.md) 1.8 节和综合练习自测。

## 阅读约定

- 本手册按知识点组织。
- 标题中的 **“补充”** 表示可在掌握主线内容后选读的进阶、扩展或综合应用内容，不影响基础阅读路径。
- 命令中的 `$` 是普通用户提示符示意，不需要输入；`#` 常表示 root 提示符，也不需要输入。
- 示例默认在个人练习目录中执行。不要在 `/`、`/etc`、`/usr` 等系统目录中练习删除、移动或覆盖。
- Linux 命令和文件名区分大小写：`ls`、`LS`、`a.txt` 和 `A.txt` 不是同一个名字。
- 遇到不熟悉的命令，先运行 `man 命令`、`命令 --help`，再执行可能修改文件的操作。

<a id="chapter-19"></a>
## 第 1 章 综合练习（补充）

所有练习只在新建的个人目录中执行：

```bash
mkdir -p ~/linux-command-lab/session-01
cd ~/linux-command-lab/session-01
pwd
```

### 练习一：路径与补全

1. 创建 `project/src/include` 和 `project/docs`。
2. 分别用绝对路径和相对路径进入 `docs`。
3. 用 `pwd` 核对位置。
4. 只输入目录名前几个字符并使用 `Tab` 补全。
5. 用 `cd -` 返回上一目录。

### 练习二：文件、重定向和查看

```bash
touch notes.txt
echo "first line" > notes.txt
echo "second line" >> notes.txt
cat notes.txt
head -n 1 notes.txt
tail -n 1 notes.txt
wc -l -w -c -m notes.txt
```

观察字节数与字符数是否相同，并确保理解 `>` 覆盖、`>>` 追加。

### 练习三：复制、移动和安全删除

```bash
mkdir source backup archive
echo "demo" > source/demo.txt
cp -i source/demo.txt backup/
mv -i backup/demo.txt archive/demo-renamed.txt
ls -l source archive
rm -i archive/demo-renamed.txt
rmdir archive
```

确认复制后源仍存在，移动后原位置不再存在，并观察空目录与非空目录删除的差别。

### 练习四：目录与存储查看

```bash
tree -L 3 ~/linux-command-lab
du -sh ~/linux-command-lab
df -h ~/linux-command-lab
```

解释 `tree`、`du`、`df` 分别回答的问题。

### 练习五：权限与查找

```bash
mkdir -p ~/linux-command-lab/session-02/src
cd ~/linux-command-lab/session-02
echo "int main(void) { return 0; }" > src/main.c
ls -l src/main.c
chmod u+x src/main.c
ls -l src/main.c
find . -name "*.c"
grep -Rni "return" .
```

解释 `chmod u+x` 改变了哪一组权限，`find` 查的是文件名还是内容，`grep -Rni` 三个选项分别做什么。

### 练习六：压缩、进程和网络只读排查

```bash
tar zcvf session-02.tar.gz session-02
tar tzvf session-02.tar.gz
ps aux
jobs
ping 127.0.0.1
nslookup www.baidu.com
```

只观察输出，不对真实进程发送 `kill -9`，也不要关闭网卡。若网络不通，记录现象即可。

### 练习七：Vim 编辑与分屏

```bash
mkdir -p ~/linux-command-lab/session-03
cd ~/linux-command-lab/session-03
vim main.c
```

1. 按 `i` 输入一段至少 8 行的 C 代码，按 `Esc`，用 `:w` 保存。
2. 使用 `gg`、`G`、`5G` 在文件中跳转。
3. 在测试行上使用 `yy`、`p`、`dd`，再用 `u` 和 `Ctrl-r` 撤销、重做。
4. 使用 `/return` 查找，分别用 `n`、`N` 跳转。
5. 用 `V` 选择两行，按 `>` 缩进，再执行 `gg=G` 整理全文缩进。
6. 执行 `:vsp helper.c`，在新窗口输入内容并保存；用 `Ctrl-w h/l` 切换窗口。
7. 分别观察 `:q`、`:wq` 和 `:q!` 在“未修改”和“存在未保存修改”两种状态下的差别。

所有批量删除、替换都只对练习文件执行。确认能够解释为什么写错文件名会进入空白的新缓冲区。

### 练习八：GCC 编译、警告与条件编译

在 `session-03` 中保存本手册[第二部分](Linux基础与系统编程学习手册_第2部分_编辑编译调试与构建.md) 2.1 和 2.4 的 `hello.c`、`feature.c`，然后执行：

```bash
gcc --version
gcc -Wall hello.c -o hello
./hello
gcc -E feature.c -o feature.i
gcc -Wall feature.c -o app-default
gcc -Wall -DFEATURE_X feature.c -o app-feature
./app-default
./app-feature
gcc -MM feature.c
```

再向 `hello.c` 加入一个未使用的局部变量，对比不加警告选项和加 `-Wall -Wextra` 的输出；最终通过修正代码消除警告，不删除警告选项。使用 `-c` 分别生成目标文件，解释编译与链接的差别。

### 练习九：GDB 断点与变量观察

保存[第二部分](Linux基础与系统编程学习手册_第2部分_编辑编译调试与构建.md) 3.1 的 `qwpj.c`：

```bash
gcc -Wall -Wextra -g -O0 qwpj.c -o qwpj
gdb ./qwpj
```

在 GDB 中依次练习：

```gdb
list 1
break qwpj
info breakpoints
run
next
step
print high
display day
display m
continue
undisplay 1
disable 1
enable 1
quit
```

根据实际停止位置调整 `next`、`step` 的使用，记录哪一条进入 `qwpj`、哪一条越过函数，并观察 `day`、`m` 在上爬和回落后的值。重新用不带 `-g` 的命令编译一个副本，只观察 GDB 的调试符号提示，不把文件大小当作唯一判断依据。

### 练习十：Makefile 增量构建

把[第二部分](Linux基础与系统编程学习手册_第2部分_编辑编译调试与构建.md) 4.2 节的 `project` 示例目录复制到个人练习目录，不要直接修改唯一原件。将[第二部分](Linux基础与系统编程学习手册_第2部分_编辑编译调试与构建.md) 4.6 的完整 Makefile 保存到 `project/src/Makefile`，确认 recipe 行使用 Tab，然后执行：

```bash
cd project/src
make --warn-undefined-variables
./myapp
make
```

第二次 `make` 应提示目标已经是最新状态。随后：

1. 修改 `add.c` 中的返回值并保存，再运行 `make`，观察是否只重建 `add.o` 后重新链接。
2. 执行 `make -n clean` 预览清理命令，确认只包含当前项目的 `.o`、`.d` 和 `myapp`。
3. 执行 `make output`，检查尖括号内的安装路径是否存在首尾空格。
4. 故意把 `DSTFILE` 的一处引用拼错，使用 `make --warn-undefined-variables` 和打印出的命令定位问题，然后恢复正确拼写。
5. 在练习副本中执行 `make clean`，再 `make` 完整重建。
6. 只使用 `make -n install`、`make -n uninstall` 预览系统级命令；没有管理员授权时不要真实安装或卸载。

解释 `$@`、`$^`、`$<` 在链接规则和模式规则中的实际值，并说明 `wildcard`、`patsubst` 分别解决了什么问题。

### 练习十一：文件描述符、复制与定位

新建 `~/linux-command-lab/session-04/fileio`，保存[第三部分](Linux基础与系统编程学习手册_第3部分_Linux系统编程.md)第 1 章的 `open.c`、`touch.c`、`copy.c` 和 `lseek.c`，只在这个新建练习目录中实验。

1. 准备 `aa`、`bb`、`cc`，编译运行 `open.c`，记录关闭前后描述符编号并解释为什么只能说“通常为 3、4、3”。
2. 记录 `umask`，运行 `./touch one two`，用 `stat -c '%a %n' one two` 验证 `0666 & ~umask`。
3. 用 `./copy` 分别复制 4 字节文件和大于 100 字节的文件，用 `cmp -s` 验证内容，并观察 100 字节缓冲区下大文件的循环读写。
4. 测试无参数运行、源文件不存在两个错误路径；结合[第三部分](Linux基础与系统编程学习手册_第3部分_Linux系统编程.md) 1.6 的纠错说明，解释为什么示例代码在同名源目标时会先清空源文件。
5. 编译运行 `lseek.c`，分别以短于 10 字节和长于 100 字节的文件作为 `bb` 观察输出。
6. 比较 `ulimit -n`、`/proc/sys/fs/nr_open` 和 `/proc/sys/fs/file-max`，只记录值，不修改系统配置。
7. 用 `strace -e trace=openat,read,write,lseek,close ./copy aa aa.copy`（系统已安装 `strace` 时）观察系统调用及返回值。

解释短读、短写、EOF、`EINTR`、`fsync` 和 `close` 各自解决或不解决的问题。

### 练习十二：阻塞与非阻塞终端读取

把[第三部分](Linux基础与系统编程学习手册_第3部分_Linux系统编程.md) 1.9 的 `block.c`、`nonblock.c` 保存到个人 `fileio` 练习目录：

```bash
gcc block.c -o block
gcc nonblock.c -o nonblock
```

1. 运行 `./block`，先等待几秒再输入一行，确认阻塞期间没有输出、输入后立即返回。
2. 再运行一次，输入超过 10 字节的行，观察本程序只处理前 10 字节及剩余输入对 Shell 的影响；只输入无害文本。
3. 运行 `./nonblock` 后暂不输入，观察程序每隔 5 秒继续等待的行为，并解释它没有处理 `EWOULDBLOCK`、`EINTR` 和短写的后果。
4. 在等待期间输入一行，观察休眠轮询可能带来的响应延迟。
5. 程序成功读出数据并退出后，解释为什么严谨写法应在退出前恢复标准输入原标志。
6. 解释为什么紧密 `while` 轮询会浪费 CPU，以及 `poll`/`select` 类接口解决的是什么问题。

这些行为依赖 Linux/POSIX 终端；不要用 Windows 控制台结果替代 Linux 验收。

### 练习十三：inode、目录项与链接

```bash
mkdir -p ~/linux-command-lab/session-05/links
cd ~/linux-command-lab/session-05/links
printf 'ccc\n' > source
ln source hard
ln -s source soft
ls -li source hard soft
stat -c '%n inode=%i links=%h type=%F size=%s blocks=%b' source hard soft
readlink soft
```

1. 解释 `source` 与 `hard` 为什么 inode 相同、链接数相同，`soft` 为什么不同。
2. 比较 `stat soft` 与 `stat -L soft`，说明命令默认是否跟随符号链接。
3. 删除 `source`，分别读取 `hard`、`soft`，解释结果；再创建新的 `source` 并重新读取。
4. 用 `df -h .`、`df -i .`、`du -sh .` 说明三条命令的统计对象。
5. 画出父目录项、inode 和数据块之间的指向关系，不把文件名画进 inode。
6. 通过只读描述符完成[第三部分](Linux基础与系统编程学习手册_第3部分_Linux系统编程.md) 2.7 的 `held` 实验，说明“删除名称”和“释放对象”的差别。
7. 清理时先用 `pwd`、`ls -la` 确认位于本练习目录，再逐个删除练习文件；不要对宽泛路径使用 `rm -rf`。

### 练习十四：VFS、共享偏移与标准输出重定向

在 `~/linux-command-lab/session-06/vfs` 中完成，所有目标文件均为新建练习文件：

1. 画出两个独立 `open("shared", O_WRONLY)` 产生两个打开文件描述的关系，并说明它们为什么有独立偏移。
2. 编写小程序：打开 `shared` 一次，再用 `dup` 得到第二个描述符；分别写入 `hello` 和 ` world`，用 `cat` 核对共享偏移的结果。
3. 在同一程序中对一个描述符执行 `lseek(fd, 0, SEEK_SET)`，观察另一描述符后续写入的位置。
4. 保存、编译并运行[第三部分](Linux基础与系统编程学习手册_第3部分_Linux系统编程.md) 2.16 的 `dup.c`，观察 `aa` 收到重定向文本并保留旧内容残留。
5. 把 `O_TRUNC` 改为 `O_APPEND` 后重复运行，解释文件结果差异；恢复源码后再继续。
6. 用 `ls -l /proc/$$/fd` 只读观察当前 Shell 描述符链接；不要关闭 Shell 的标准描述符。

解释为什么 `dup2(target, STDOUT_FILENO)` 比“先 `close(1)` 再期待下一次 `open` 返回 1”更能避免并发描述符分配竞态，以及标准 I/O 缓冲为什么要在切换前后刷新。

### 练习十五：分页、分段与段页式地址转换

本练习只做纸面或文本计算，不修改系统内存配置：

1. 重算页大小 4096、页表 `{0→2, 1→3, 2→5, 3→7}` 下逻辑地址 4300 的页号、页内偏移和物理地址。
2. 重算页大小 1024、虚拟页 2 映射页框 8 时逻辑地址 2500 的物理地址。
3. 说明逻辑地址 1024 在 1 KiB 页面中为什么是页 1、偏移 0；区分“页框号 3”和“字节物理地址 3072”。
4. 使用[第四部分](Linux基础与系统编程学习手册_第4部分_操作系统内存管理.md) 1.6 的段表判断 `(0, 790)`、`(0, 810)`、`(4, 98)`、`(4, 120)` 是否越界，并计算有效地址。
5. 对给定访问串分别手算 3 个页框下的 FIFO 和 LRU 缺页次数，再解释为什么 OPT 只能作为知道未来序列的理论基线。
6. 画出 `(段号, 段内页号, 页内偏移)` 经段表、页表到物理页框的流程，并标出每一步可能发生的越界、无效页和缺页。

### 练习十六：工具链与 ELF 只读检查

在[第二部分](Linux基础与系统编程学习手册_第2部分_编辑编译调试与构建.md) 2.5 的隔离目录中生成 `hello.i`、`hello.s`、`hello.o` 和 `hello`，再完成：

1. 用 `file` 说明四个文件的类型差异。
2. 用 `readelf -h` 对比目标文件和可执行文件的 ELF 类型。
3. 用 `nm -C hello.o` 找到 `main` 与未解析的库符号。
4. 用 `objdump -d hello` 查看反汇编，只要求识别函数标签，不要求手工修改机器码。
5. 复制一份程序后再执行 `strip`，比较大小和 GDB 提示，不修改唯一原件。

### 练习十七：静态库与共享库

按[第二部分](Linux基础与系统编程学习手册_第2部分_编辑编译调试与构建.md) 2.6—2.7 构建 `libcalc.a`、`libcalc.so.1.0`、`calc-static` 和 `calc-shared`：

1. 用 `ar t`、`nm -C` 说明静态库里有哪些成员和符号。
2. 故意把 `-lcalc` 放到使用符号的源文件之前，记录当前链接器的结果，再恢复正确命令。
3. 用 `readelf -d` 和 `ldd` 检查两个程序的动态依赖。
4. 在隔离目录临时移走 `libcalc.so.1`，复现运行时找不到库，再恢复文件并验证；不要移动系统库。
5. 解释链接时搜索路径、运行时搜索路径和 SONAME 的区别。

### 练习十八：`ioctl` 终端查询

保存并编译[第三部分](Linux基础与系统编程学习手册_第3部分_Linux系统编程.md) 1.10 的 `ioctl_winsize.c`：

1. 在终端直接运行，记录行数和列数。
2. 调整终端窗口后重新运行，确认结果变化。
3. 把标准输出重定向到普通文件，确认 `isatty` 失败分支生效。
4. 解释为什么不能只看 `ioctl` 原型就猜第三参数类型。

### 练习十九：递归目录遍历

保存并编译[第三部分](Linux基础与系统编程学习手册_第3部分_Linux系统编程.md) 2.13.1 的 `tree_walk.c`：

1. 建立三层目录、普通文件和指向父目录的符号链接，确认程序不会跟随链接形成环。
2. 删除某个子目录的读取权限后，以普通用户运行并观察错误传播；完成后恢复权限再清理。
3. 把 `MAX_DEPTH` 临时改为 2，确认深度限制生效。
4. 解释为什么不能仅凭 `dirent.d_type` 判断所有文件类型，以及 `stat` 与 `lstat` 对符号链接的差异。

### 练习二十：服务器配置只读验收

只在可回滚的实验虚拟机中选择 SSH、NFS 或 FTP 之一练习：

1. 记录安装前后的服务状态、监听端口和配置文件备份路径。
2. 修改配置后先做语法或导出检查，再重载服务。
3. 从另一终端或客户端分别验证成功连接和预期失败连接。
4. 核对防火墙、监听地址、用户权限和日志，不设置匿名写入、`no_root_squash` 或 root 密码远程登录。
5. 写出停止服务、恢复配置和撤销测试文件的回滚步骤。

<a id="chapter-20"></a>
## 第 2 章 常见错误与排查（补充）

| 现象 | 常见原因 | 排查方式 |
|---|---|---|
| `command not found` | 拼写错误、命令未安装、`PATH` 不含程序目录 | 检查拼写，使用 `which` 检查外部命令路径，确认软件是否安装 |
| `No such file or directory` | 路径错误、大小写错误、相对路径基准错误 | `pwd`、`ls -la`，使用 Tab 补全 |
| `Permission denied` | 缺少文件权限或父目录权限 | `ls -ld` 查看权限；不要盲目提升权限，先确认目标和授权 |
| `Directory not empty` | 用 `rmdir` 删除非空目录 | 先 `ls -la` 核对内容；确认后再考虑递归删除 |
| 文件被意外覆盖 | 使用了 `>`、默认覆盖的 `cp/mv` | 重要操作使用 `-i` 或备份；追加使用 `>>` |
| `tree` 不存在 | 软件包未安装 | 从发行版官方软件源安装 |
| `which cd` 无结果 | `cd` 是 Shell 内建命令 | 这是正常现象；`which` 主要查找 PATH 中的外部程序 |
| `du` 与 `df` 不一致 | 统计对象不同、已删除但仍打开文件、保留块等 | 分别确认目录占用和文件系统空间，必要时查进程 |
| `Permission denied` 后加 `sudo` 仍失败 | 目标路径错误、文件被占用、用户不在 sudoers、命令本身限制 | 先确认路径和权限，再读错误信息；不要连续盲试 |
| `grep` 找不到明明存在的内容 | 大小写不一致、搜错目录、文件不是普通文本 | 使用 `grep -Rni` 缩小范围，确认文件编码和搜索路径 |
| `umount` 提示目标正忙 | 当前终端或程序仍在挂载点目录中 | 退出挂载点目录，关闭正在访问该目录的程序后再卸载 |
| 删除用户失败 | 用户仍有登录会话或进程正在运行 | 用 `ps` 查对应进程，退出会话或结束进程后再处理 |
| `ping` 超时 | IP 写错、路由不通、防火墙限制、目标不在线 | 先 ping 本机和网关，再查目标地址和网络配置 |
| Vim 打开已有文件却显示空白 | 文件名、大小写或相对路径写错 | 先 `:q!` 退出，不要保存；用 `pwd`、`ls -la` 核对后重开 |
| Vim 提示 `No write since last change` | 有未保存修改却执行了 `:q` | 需要保留时用 `:wq`；确认丢弃时才用 `:q!` |
| Vim 提示交换文件已存在 | 上次异常退出、文件仍在其他 Vim 中打开或确有残留交换文件 | 先确认没有其他编辑会话，再按提示恢复或谨慎清理；不要直接覆盖 |
| GCC 报头文件不存在 | `-I` 目录错误、当前目录判断错误或文件名大小写不一致 | 用 `pwd`、`ls` 核对目录；`-I` 填目录而不是具体头文件 |
| GCC 链接报 `undefined reference` | 实现源文件或目标文件未参与链接、符号名不一致 | 确认所有实现文件已编译并加入最终链接命令 |
| GCC 有警告但仍生成程序 | `-Wall` 发现可疑代码，警告默认不一定阻止链接 | 理解并修正警告原因；不要通过删除警告选项掩盖问题 |
| GDB 提示没有调试符号 | 编译时未加 `-g`、符号被剥离或加载了错误文件 | 用 `gcc -g -O0` 重建并确认 GDB 打开的可执行文件路径 |
| GDB 变量显示 `<optimized out>` | 优化改变或移除了变量 | 调试构建使用 `-O0` 或 `-Og`，并确保当前停在变量作用域内 |
| GDB 断点没有停住 | 断点被禁用、代码未执行到、源码与可执行文件不一致 | 用 `info breakpoints` 查状态和位置，重新构建后再 `run` |
| `No targets specified and no makefile found` | 当前目录没有默认名称的 Makefile | 用 `pwd`、`ls -la` 核对位置，或使用 `make -C`、`make -f` |
| `missing separator` | recipe 前使用了空格或全角字符，而不是 Tab | 在编辑器中显示不可见字符，重新输入真正的 Tab |
| `No rule to make target ...` | 前置条件拼错、不存在，且没有能生成它的规则 | 核对文件名、大小写、`wildcard` 结果和模式规则 |
| `make` 显示的 GCC 命令缺少文件或参数 | 变量名拼错或变量展开为空 | 使用 `make --warn-undefined-variables`，检查 `SRCFILE`、`DSTFILE` 等拼写 |
| 修改头文件后没有重编译 | Makefile 未记录头文件依赖 | 显式添加头文件前置条件，或使用 `-MMD -MP` 和 `-include` |
| `make` 总是全部重编译 | 目标未正确生成、时间戳异常，或所有源文件直接依赖最终目标 | 把编译和链接拆开，检查依赖图与目标文件时间 |
| `make clean` 没有执行 | 存在名为 `clean` 的文件且目标未声明为 phony | 添加 `.PHONY: clean` |
| 拼接后的安装/删除路径出现空格 | 变量赋值行在注释前留下尾随空格 | 用 `printf '<%s>\n'` 显示边界，删除尾随空格并给路径加引号 |
| `open` 创建文件后权限或行为异常 | `O_CREAT` 缺少 `mode`，或只写了 `O_CREAT` 而没有访问模式 | 使用 `O_WRONLY \| O_CREAT` 等明确访问模式，并提供八进制 `mode`；再结合 `umask` 计算权限 |
| `open` 返回的第一个描述符不是 3 | 标准描述符曾关闭，或程序此前已经打开其他对象 | 根据“最小未使用描述符”规则分析，不把 3 写成固定结论 |
| 复制时源文件突然变成空文件 | 源和目标是同一文件，目标以 `O_TRUNC` 过早打开 | 截断前用 `fstat` 比较 `st_dev`、`st_ino`；同一文件立即报错 |
| 复制结果偶尔不完整 | 未处理短写、`EINTR` 或读取错误 | 使用 `ssize_t`，循环写完每一块；仅在 `read` 返回 0 时认定 EOF |
| 错误文字与失败原因不符 | 在失败检查之前又调用了可能改变 `errno` 的函数 | 在函数返回失败后立即保存或使用 `errno`，再调用 `perror`/`strerror` |
| 长时间运行后出现 `Too many open files` | 某些错误路径遗漏 `close`，或达到 `RLIMIT_NOFILE` | 审查所有退出路径，查看 `/proc/PID/fd` 和 `ulimit -n`；不要只提高限制掩盖泄漏 |
| `lseek` 返回 `Illegal seek` | 描述符指向管道、FIFO 或套接字等不可定位对象 | 检查对象类型和 `errno == ESPIPE`，改用顺序读写协议 |
| 大文件长度显示负数或被截断 | 用 `int` 保存 `off_t`/`ssize_t`，或格式说明符不匹配 | 使用正确类型；将 `off_t` 转为 `intmax_t` 后用 `%jd` 输出 |
| `write` 成功后断电仍丢数据 | 把进入内核缓存误认为已经持久化 | 按业务协议调用并检查 `fsync`，必要时同时处理目录项持久化 |
| 非阻塞 `read` 返回 `-1` 就被当作故障退出 | 没有区分 `EAGAIN`/`EWOULDBLOCK` 与真正错误 | 只在调用报告失败后检查 `errno`；暂时无数据时等待事件或稍后重试 |
| 非阻塞循环占满一个 CPU 核心 | 在无数据时执行紧密轮询 | 使用 `poll`/`select`/`epoll`；简单演示程序至少加入有界等待，避免无限空转 |
| `fcntl(F_SETFL)` 后标准输入行为异常 | 覆盖了旧状态标志、忽略调用失败或退出前未恢复 | 先用 `F_GETFL` 保存旧值，按位加入 `O_NONBLOCK`，检查返回值并在清理路径恢复 |
| 把普通文件设为 `O_NONBLOCK` 后仍会等待 | Linux 当前对普通文件和块设备忽略该标志 | 不依赖普通文件的非阻塞语义；对适用对象使用异步或事件接口 |
| `df -h` 还有空间却不能创建文件 | inode 已耗尽，常见于大量小文件 | 用 `df -i` 检查 inode；清理或重新规划文件系统，而不是只看字节容量 |
| `stat` 与 `stat -L` 显示不同 inode | 前者查看符号链接本身，后者跟随到目标 | 根据要检查“链接”还是“目标”选择是否解引用 |
| 删除原文件后硬链接仍能读取 | 硬链接只是另一个名称，inode 链接数尚未归零 | 用 `ls -li`、`stat` 查看 inode 和链接数；不要把某个名称视为唯一原件 |
| 删除原文件后符号链接变红或读取失败 | 符号链接保存的目标路径已无法解析 | 用 `readlink` 检查保存的字符串，修复目标路径或重新创建链接 |
| `ln` 报 `Invalid cross-device link` | 尝试创建跨文件系统硬链接 | 在同一文件系统内创建硬链接，跨文件系统需求改用符号链接或复制 |
| `readlink` 打印出尾部乱码 | 返回内容没有自动添加字符串终止符 | 给缓冲区预留一字节，按返回长度手动写入 `\0` |
| `rm` 后磁盘空间没有立刻增加 | 文件仍被进程打开，或统计/写回尚未更新 | 查找持有描述符的进程；让程序正常关闭，不要只反复删除名称 |
| 把普通删除当作安全擦除 | 删除名称不等于按介质要求完成数据净化 | 按介质、敏感级别和认可标准选择 clear、purge、加密擦除或合规销毁 |
| `printf("%d", st.st_ino)` 出现格式警告 | `ino_t`/`off_t` 不一定是 `int` | 转成 `uintmax_t`/`intmax_t` 后用 `%ju`/`%jd`，不要忽略警告 |
| 权限参数 `0899` 被程序接受 | 用 `atoi` 或自写循环却没有验证八进制数字 | 用 `strtoul(text, &end, 8)`，检查 `errno`、尾字符和允许范围 |
| `access` 成功后实际打开仍失败 | 检查与操作之间状态变化，或真实/有效身份语义不同 | 不把 `access` 当授权；直接执行目标操作并检查它自己的返回值 |
| `readdir` 返回 `NULL` 被一律当作正常结束 | 没有用 `errno` 区分目录末尾与读取错误 | 调用前清零 `errno`，循环结束后检查；始终关闭目录流 |
| `d_name` 用 `%d` 打印出现警告或异常 | 文件名是字符数组而非整数 | 使用 `%s`；开启 `-Wall -Wextra -Wpedantic -Werror` 尽早发现 |
| `strcmp` 出现隐式声明警告 | 忘记包含 `<string.h>` | 添加正确头文件和函数声明，不依赖旧式隐式声明 |
| 自写目录列表与 `ls` 顺序不同 | `readdir` 不保证排序，自写程序也未排版 | 若需求要求稳定顺序，收集名称后显式排序；颜色和列宽另行实现 |
| 两个描述符写同一文件时结果与预期不同 | 没有区分独立 `open` 与 `dup` 共享的打开文件描述 | 画出描述符到打开文件描述的指向，检查是否共享偏移和 `O_APPEND` |
| `printf` 重定向后部分文字去了错误目标 | 切换 `STDOUT_FILENO` 前后没有刷新 `stdout` 缓冲 | 在 `dup2` 切换前和恢复前调用并检查 `fflush(stdout)` |
| 用 `close(1); open(...)` 重定向偶发失效 | 两次调用之间其他线程或信号处理程序取得了描述符 1 | 使用 `dup2(target, STDOUT_FILENO)` 原子替换目标描述符 |
| 分页地址换算相差一个页大小 | 把页号、页框号或页内偏移混用 | 依次计算除法得页号、取余得偏移、查页表得页框，再组合物理地址 |
| 分段地址计算得到段外位置 | 未先检查段内偏移是否小于段长 | 先做界限检查，只有有效时才计算 `基址 + 段内偏移` |
| 把缺页一律当成程序错误 | 混淆合法但未驻留页面与未映射/无权限地址 | 由操作系统检查页表状态；合法缺页可调入，非法访问才向进程报告错误 |
| `cannot find -lxxx` | 库文件名不符合 `libxxx.a/.so`、缺少 `-L` 或目录错误 | `ls -l` 核对文件，检查 `-L` 和 `-l` 的对应关系 |
| `error while loading shared libraries` | 运行时找不到共享库或 SONAME 链接缺失 | `readelf -d`、`ldd`，检查 RUNPATH/RPATH 和库链接；不要复制到系统目录碰运气 |
| `ioctl` 返回 `ENOTTY` | 描述符对象不支持该请求，或请求码不属于该设备 | 用 `isatty`、`fstat` 确认对象，查对应设备和请求码文档 |
| 递归遍历无限循环 | 没跳过 `.`/`..` 或跟随符号链接回到祖先目录 | 使用 `lstat` 和明确的链接策略，必要时记录已访问的设备号/inode |
| SSH 修改后无法新建连接 | 配置语法、端口、防火墙、监听地址或认证出错 | 保留旧会话，先 `sshd -t`，查看 `ss`、服务状态和日志后再重载 |

<a id="chapter-21"></a>
## 第 3 章 命令速查表（补充）

| 命令 | 核心用途 | 常用示例 |
|---|---|---|
| `env` | 查看环境变量 | `env` |
| `echo` | 输出文本或变量 | `echo "$PATH"` |
| `history` | 查看命令历史 | `history` |
| `man` | 查看手册 | `man ls` |
| `cd` | 切换目录 | `cd ~/projects` |
| `pwd` | 输出当前目录 | `pwd` |
| `which` | 在 PATH 中定位外部命令 | `which ls` |
| `ls` | 列出目录内容 | `ls -lah` |
| `touch` | 创建空文件/更新时间 | `touch note.txt` |
| `mkdir` | 创建目录 | `mkdir -p a/b` |
| `rmdir` | 删除空目录 | `rmdir empty` |
| `rm` | 删除文件/目录 | `rm -i file` |
| `mv` | 移动或重命名 | `mv -i old new` |
| `cp` | 复制文件/目录 | `cp -R src backup` |
| `cat` | 输出/连接文件 | `cat file` |
| `more` | 分页查看 | `more file` |
| `less` | 可前后翻页查看 | `less file` |
| `head` | 查看开头 | `head -n 5 file` |
| `tail` | 查看结尾/跟踪日志 | `tail -f app.log` |
| `clear` | 清理终端显示 | `clear` |
| `tree` | 树形显示目录 | `tree -L 2 .` |
| `wc` | 统计换行/词/字符/字节 | `wc -l file` |
| `od` | 按底层格式显示数据 | `od -tcx1 file` |
| `du` | 估算目录树占用 | `du -sh dir` |
| `df` | 查看文件系统空间 | `df -h` |
| `whoami` | 查看当前用户名 | `whoami` |
| `chmod` | 修改权限 | `chmod 755 run.sh` |
| `chown` | 修改属主/属组 | `sudo chown root:root file` |
| `sudo` | 临时提权执行命令 | `sudo apt-get install tree` |
| `umask` | 查看/设置默认权限掩码 | `umask` |
| `find` | 按文件名等条件查找 | `find . -name "*.c"` |
| `grep` | 按内容检索 | `grep -Rni "main" .` |
| `apt-get` | 软件源安装/删除 | `sudo apt-get install tree` |
| `dpkg` | 安装/删除 deb 包 | `sudo dpkg -i package.deb` |
| `mount` | 挂载设备 | `sudo mount /dev/sdb1 /mnt` |
| `umount` | 卸载挂载点 | `sudo umount /mnt` |
| `tar` | 打包/压缩/解压 | `tar zcvf dir.tar.gz dir` |
| `zip` | 创建 zip 压缩包 | `zip -r dir.zip dir` |
| `unzip` | 解压 zip 包 | `unzip dir.zip` |
| `ps` | 查看进程 | `ps aux` |
| `jobs` | 查看当前 Shell 作业 | `jobs` |
| `fg` | 作业调到前台 | `fg %1` |
| `bg` | 作业放到后台继续 | `bg %1` |
| `kill` | 向进程发送信号 | `kill -15 PID` |
| `useradd` | 创建用户 | `sudo useradd -m name` |
| `passwd` | 设置密码 | `sudo passwd name` |
| `su` | 切换用户 | `su name` |
| `userdel` | 删除用户 | `sudo userdel -r name` |
| `ifconfig` | 查看/配置网卡 | `ifconfig` |
| `ping` | 测试网络连通 | `ping 127.0.0.1` |
| `netstat` | 查看连接/端口 | `netstat -an` |
| `nslookup` | 查询域名解析 | `nslookup www.baidu.com` |
| `alias` | 设置命令别名 | `alias ll='ls -alF'` |
| `date` | 查看时间 | `date` |
| `free` | 查看内存 | `free -m` |
| `reboot` | 重启系统 | `sudo reboot` |
| `poweroff` | 关闭系统 | `sudo poweroff` |
| `vi` / `vim` | 编辑纯文本文件 | `vim main.c` |
| `gcc` | 编译并链接 C 程序 | `gcc -Wall main.c -o app` |
| `file` | 判断文件和目标格式 | `file app app.o` |
| `nm` | 查看目标文件或库中的符号 | `nm -C app.o` |
| `objdump` | 查看目标文件信息或反汇编 | `objdump -d app` |
| `readelf` | 读取 ELF 结构和动态信息 | `readelf -d app` |
| `ar` | 创建或查看静态归档库 | `ar rcs libcalc.a add.o` |
| `ldd` | 查看受信程序的共享库依赖 | `ldd ./app` |
| `gdb` | 调试可执行程序 | `gdb ./app` |
| `make` | 按 Makefile 依赖图增量构建 | `make --warn-undefined-variables` |
| `open` / `close` | 打开对象并释放文件描述符（C 接口） | `open("data", O_RDONLY)` |
| `read` / `write` | 按字节从描述符读取/写入（C 接口） | `read(fd, buf, sizeof buf)` |
| `lseek` | 修改或查询可定位对象的文件偏移（C 接口） | `lseek(fd, 0, SEEK_END)` |
| `fcntl` | 获取或修改描述符属性（C 接口） | `fcntl(fd, F_SETFL, flags)` |
| `ioctl` | 发出设备或文件类型专用控制请求（C 接口） | `ioctl(fd, TIOCGWINSZ, &ws)` |
| `dup` / `dup2` | 复制描述符或把指定编号原子改指向同一打开文件描述（C 接口） | `dup2(fd, STDOUT_FILENO)` |
| `stat` | 查看文件、链接或文件系统状态 | `stat -c '%i %h %n' file` |
| `stat` / `fstat` / `lstat` | 读取 inode 主要信息（C 接口） | `lstat(path, &st)` |
| `ln` | 创建硬链接或符号链接 | `ln -s target link` |
| `readlink` | 读取符号链接保存的目标路径 | `readlink link` |
| `link` / `symlink` / `unlink` | 创建链接或删除名称（C 接口） | `unlink("name")` |
| `access` | 按真实身份检查路径可访问性（C 接口） | `access(path, F_OK)` |
| `chmod` / `fchmod` | 修改路径或描述符对象的权限（C 接口） | `chmod(path, 0644)` |
| `chown` / `fchown` / `lchown` | 修改所有者或组（C 接口） | `lchown(link, uid, gid)` |
| `utime` / `utimensat` | 修改访问和内容修改时间（C 接口） | `utimensat(AT_FDCWD, path, ts, 0)` |
| `truncate` / `ftruncate` | 把文件逻辑长度设为指定值（C 接口） | `ftruncate(fd, 4096)` |
| `rename` | 原子改变同一文件系统内的名称（C 接口） | `rename("old", "new")` |
| `chdir` / `getcwd` | 修改或取得当前工作目录（C 接口） | `getcwd(buf, sizeof buf)` |
| `pathconf` / `fpathconf` | 查询路径相关限制（C 接口） | `pathconf(path, _PC_NAME_MAX)` |
| `opendir` / `readdir` / `closedir` | 打开、遍历和关闭目录流（C 接口） | `while ((e = readdir(dir)) != NULL)` |
| `ulimit -n` | 查看或设置当前 Shell 的打开文件软限制 | `ulimit -n` |
| `getrlimit` | 在程序中查询软硬资源限制（C 接口） | `getrlimit(RLIMIT_NOFILE, &lim)` |
| `cmp` | 按字节比较两个文件 | `cmp -s source copy` |

<a id="chapter-22"></a>
## 第 4 章 常见误区与纠错摘要（补充）

1. Linux 是内核；Ubuntu、Red Hat 等是 Linux 发行版。
2. Xshell 是远程终端客户端；Shell 是命令解释器。
3. 日常学习应使用普通用户，不建议长期处于 root 高权限环境。
4. `/bin/sh` 不一定等于 Bash；现代 macOS 默认 Shell 也不再是 Bash。
5. `PATH` 只负责按目录搜索不含斜杠的命令名；`cd` 等内建命令不依赖单独的可执行文件。
6. `echo text > file` 是 Shell 重定向，`>` 覆盖，`>>` 追加。
7. `ls -l` 通常显示最后修改时间，不应笼统称为创建时间；目录大小也不恒为 4096。
8. `rmdir` 的“只删空目录”是保护特性；`rm -rf` 不是普通删除的推荐默认值。
9. `mv`、`cp` 的目标解释和覆盖行为由目标状态、类型、选项、权限与文件系统共同决定。
10. `wc -l` 统计换行符，`-w` 按空白分隔统计词，`-c` 统计字节，`-m` 统计字符。
11. `du` 递归估算文件/目录占用，`df` 报告文件系统空间，`ls` 不能替代它们。
12. `od`、`head`、`tail`、`more`、`less` 都有实际用途；它们属于当前阶段的非重点内容，但不等于命令本身无用。
13. `chmod` 改出执行权限不等于文件一定可执行；还要看文件格式、解释器和执行环境。
14. `umask` 是拿走权限位，不是简单十进制或八进制减法。
15. `apt-get update` 更新软件包索引，不等于升级全部软件。
16. `kill` 是发送信号，`kill -9` 是强制终止，应作为最后手段。
17. `ping 127.0.0.1` 只能证明本机回环可用，不代表外部网络一定可达。
18. 示例配置中的用户名、路径、IP、token 和密码不能机械照抄，必须替换成真实环境值。
19. Vim 的普通模式不是 Bash 命令行；插入模式才把普通按键当作文件内容。
20. Vim 可以通过 `v`、`V` 进入可视模式选择文本，并非只能逐字符删除。
21. Vim 大写 `U` 撤销最近修改行上的最新改动，不是撤销本次打开后的全部操作；`Ctrl-r` 才是重做。
22. `gg=G` 重新计算代码缩进，不是通用文档排版，也不能保证符合所有项目格式规范。
23. GCC 的 `-I`、`-D` 可使用紧邻参数的常见写法，也接受分开的参数；不要把任一示例理解为唯一形式。
24. `-Wall` 只启用一组常见警告，不代表启用 GCC 的全部警告；警告应分析修正，不应通过删除选项掩盖。
25. GCC 优化选项应写成 `-O0`、`-O1`、`-O2`、`-O3` 等；`-On` 只是泛指优化级别的占位写法。
26. GDB 的 `next` 越过函数调用，`step` 进入可调试函数；不要用“逐语句/逐过程”反向记忆。
27. 含调试信息的文件通常较大，但文件大小不能证明一定存在可用调试符号。
28. Windows 可以通过目标兼容工具链编译 Linux 程序，但普通 Windows 原生可执行文件不能直接当作 Linux 程序运行。
29. 条件编译可移除某些调试日志，却不能替代敏感信息审计、发布包检查和安全测试。
30. `make` 按目标和前置条件组成的依赖图更新文件，不是简单从 Makefile 第一行依次执行到末尾。
31. 第一次完整构建通常不能省去编译步骤；后续构建才能复用未过期的 `.o`，而大型项目的链接也不一定总比编译快。
32. Make 主要比较目标与前置条件的最后修改时间；遗漏头文件依赖会导致错误地跳过重建。
33. GNU Make 默认可查找 `GNUmakefile`、`makefile`、`Makefile`；其他名称也能通过 `make -f` 使用。
34. 变量名只使用大写字母、数字和下划线且不以数字开头是推荐规范，不是 GNU Make 的绝对语法限制。
35. `-I`、`-D宏名` 按惯例放在 `CPPFLAGS`，`-Wall`、`-g` 等放在 `CFLAGS`；单独一个 `-D` 是不完整选项。
36. `clean` 等功能目标应声明为 `.PHONY`；它们可以拥有前置条件，`install: $(TARGET)` 就是合理写法。
37. 已知对象文件和单个程序不需要 `rm -rf`；不要把 `sudo rm -rf` 与未经检查的变量拼接组合使用。
38. Make 变量赋值中的尾随空格可能进入变量值，路径使用前应打印边界、引用并验证非空。
39. 动态文件列表能提高 Makefile 复用性，但复制到目录结构、语言或工具链不同的项目仍需修改和测试。
40. 未定义的 Make 变量常展开为空字符串而不是立即报错，应结合 `--warn-undefined-variables` 和实际 recipe 输出排查。
41. 低层文件 I/O 不一定比标准 I/O 更快；标准 I/O 的用户空间缓冲常能减少小块系统调用。
42. `FILE` 是实现相关的流对象，不能固定描述为只含描述符、位置和 8192 字节缓冲区。
43. “一切皆文件”描述统一接口思想，不代表管道、套接字和设备都支持普通文件的全部操作；它们通常不能随意 `lseek`。
44. `open` 返回当前进程最小的未使用描述符；3 是常见首个结果，不是固定保证。
45. 3 GB/1 GB 只可作为某些历史 32 位 Linux 地址空间配置示例，不能推广到现代 64 位系统。
46. `O_CREAT` 需要访问模式和第三个 `mode` 参数；新文件权限按 `mode & ~umask` 计算，不应为追求宽松权限随意设置 `umask(0000)`。
47. `errno` 是失败后检查的线程局部错误编号，不存放在标准错误流中；`perror` 才把解释文字写到 `stderr`。
48. 文件描述符上限不是固定 1024；应区分进程的 `RLIMIT_NOFILE`、`/proc/sys/fs/nr_open` 和系统文件句柄上限。
49. `read` 返回 0 表示本次调用位于 EOF；短读不必然是错误，`write` 成功也可能只写入一部分数据。
50. `write` 和 `close` 成功不能单独证明数据已经持久化；有明确持久化要求时按协议使用并检查 `fsync`。
51. `lseek` 返回 `off_t` 而非通用 `int`；获取文件尾偏移会改变当前位置，管道和套接字等对象通常不可定位。
52. 复制程序不能在确认源目标不同之前用 `O_TRUNC` 打开目标，否则同一文件会被提前清空。
53. 阻塞等待时当前线程睡眠、CPU 可以运行其他任务；这不表示整个进程或系统必然完全不使用 CPU。
54. `O_NONBLOCK` 主要影响终端、管道、FIFO、套接字和部分设备；Linux 当前会忽略普通文件和块设备上的该标志。
55. `F_GETFL` 可取得访问模式和状态标志，但 `F_SETFL` 只能修改有限状态标志，不能把只读描述符改成可写。
56. 非阻塞 `read` 的 `-1` 不总是永久错误；`EAGAIN`/`EWOULDBLOCK` 表示暂时会阻塞，`EINTR` 表示被信号中断。
57. 休眠轮询会降低 CPU 空转，却会增加响应延迟；真实多路 I/O 通常应使用 `poll`、`select`、`epoll` 或事件循环。
58. 512 字节扇区、4 KiB 文件系统块和 128 字节 inode 都是具体模型或格式化结果，不是所有介质和文件系统的固定常量。
59. inode 保存对象元数据和数据块映射，不保存文件名；文件名与 inode 号的对应关系保存在父目录的目录项中。
60. ext2 的 12 个直接指针和三级间接寻址是经典块映射模型；现代 ext4 普通文件通常使用 extent 树。
61. 硬链接不是“原文件的副本”，而是同一文件系统内另一个目录项引用同一 inode；各名称地位相同。
62. 符号链接保存目标路径字符串并有自己的 inode，不与目标共用 inode 或普通文件内容；目标可不存在或跨文件系统。
63. `unlink` 删除名称；只有最后一个链接和最后一个打开描述符都消失后，文件对象及空间才可最终释放。
64. GNU `stat` 命令默认查看符号链接本身，`stat -L` 才跟随；`stat(2)` 默认跟随，`lstat(2)` 查看链接本身。
65. 普通删除、格式化或简单写满空间都不是适用于所有 HDD、SSD、闪存和移动设备的安全擦除方案；介质处置应遵循认可标准并验证结果。
66. `ino_t`、`off_t` 等系统类型不保证等于 `int`；打印前应使用合适的转换和格式说明符。
67. 数值在内存中没有“十进制整数”和“八进制整数”两种存储；进制属于源码记法或文本解析规则。
68. `access` 使用真实 UID/GID，不能替代实际操作的权限判断，也不能消除检查后路径被替换的竞态。
69. 改变文件所有者不一定只能由 root 完成；Linux 还要结合所有权、组成员关系和相关 capability 判断。
70. `truncate` 延长文件后新增范围读取为零，但物理块可能按稀疏文件方式延迟分配。
71. `pathconf` 返回 `-1` 可能表示错误，也可能表示限制不确定；必须结合事先清零后的 `errno` 判断。
72. `readdir` 返回 `NULL` 可能是目录末尾或错误，返回顺序也不保证等同于 `ls` 的排序结果。
73. `d_type` 是可选提示，可能为 `DT_UNKNOWN`；需要可靠类型时应再用 `fstatat`/`stat` 查询。
74. VFS 是内核中的统一文件系统接口和对象模型，不等于某一种磁盘格式，也不应把所有具体文件系统实现简单称为硬件驱动。
75. 文件描述符是进程表中的整数索引，打开文件描述保存偏移和状态，inode 表示文件系统对象；三者不能混为同一结构。
76. `dup`、`dup2` 得到的描述符共享同一打开文件描述、偏移和文件状态标志；分别调用 `open` 通常创建独立打开文件描述。
77. 已打开的文件名称仍可被 `unlink`；最后名称消失后，对象会持续到最后打开引用释放，而不是“正在使用就删不了”。
78. `dup2` 会原子关闭并替换指定的新描述符；“先关 1 再打开文件”在多线程或信号环境中存在描述符被抢占的竞态。
79. `printf` 使用标准 I/O 缓冲，切换或恢复 `STDOUT_FILENO` 前应检查 `fflush`，否则缓冲数据可能写到意外目标。
80. 指针越界不保证立刻崩溃；内存保护通常按页工作，不能识别 C 对象边界，也不会自动阻止或修复内存泄漏。
81. 虚拟地址由处理器地址转换硬件依据操作系统页表转换，TLB 可缓存结果；不能把虚拟地址简单说成没有实际意义的“假地址”。
82. 页大小不是只允许 1 KiB、2 KiB、4 KiB，内存容量选择也不是为了被 4 KiB 整除；页大小取决于体系结构和操作系统配置。
83. OPT 需要未来访问序列，只是理论基线；FIFO 可能出现 Belady 异常，LRU 的精确维护也有成本。
84. 现代 Windows 虚拟内存支持按需分页，不能把“整个程序必须一次装入内存”当作分页的固定缺点。
85. 分段先检查 `段内偏移 < 段长` 再加基址；段式典型地产生外部碎片，分页典型地产生末页内部碎片。
86. 经典段页式使用页框后通常主要保留末页内部碎片，不能机械写成必然同时产生物理内存的内部和外部碎片。
87. 页表、段表的理论访存次数会受到 TLB、多级页表和缓存影响；“页式两次、段页式三次”不是现代系统固定耗时。
88. 段页式不是只供大型机使用的绝对分类；应区分教学模型、处理器机制和现代 64 位 Windows 的分页策略。

<a id="chapter-23"></a>
## 第 5 章 参考与继续学习（补充）

以下权威文档适合用于核对命令选项、接口原型、实现边界和安全建议：

- [GNU Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [Vim Reference Manual](https://vimhelp.org/)
- [Using the GNU Compiler Collection](https://gcc.gnu.org/onlinedocs/gcc/)
- [Debugging with GDB](https://sourceware.org/gdb/current/onlinedocs/gdb.html/)
- [GNU Make Manual](https://www.gnu.org/software/make/manual/make.html)
- [GNU Binutils Manual](https://sourceware.org/binutils/docs/binutils/)
- [Ubuntu Server：OpenSSH server](https://documentation.ubuntu.com/server/how-to/security/openssh-server/)
- [Ubuntu Server：Network File System (NFS)](https://documentation.ubuntu.com/server/how-to/networking/install-nfs/)
- [Linux man-pages：ioctl(2)](https://man7.org/linux/man-pages/man2/ioctl.2.html)
- [Linux man-pages：readdir(3)](https://man7.org/linux/man-pages/man3/readdir.3.html)
- [Linux man-pages：open(2)](https://man7.org/linux/man-pages/man2/open.2.html)
- [Linux man-pages：read(2)](https://man7.org/linux/man-pages/man2/read.2.html)
- [Linux man-pages：write(2)](https://man7.org/linux/man-pages/man2/write.2.html)
- [Linux man-pages：lseek(2)](https://man7.org/linux/man-pages/man2/lseek.2.html)
- [Linux man-pages：utimensat 与 futimens](https://man7.org/linux/man-pages/man3/futimens.3.html)
- [Linux man-pages：fcntl(2)](https://man7.org/linux/man-pages/man2/fcntl.2.html)
- [Linux man-pages：F_GETFL/F_SETFL](https://man7.org/linux/man-pages/man2/F_SETFL.2const.html)
- [Linux man-pages：dup(2)](https://man7.org/linux/man-pages/man2/dup.2.html)
- [Linux man-pages：stat(2)](https://man7.org/linux/man-pages/man2/stat.2.html)
- [Linux man-pages：link(2)](https://man7.org/linux/man-pages/man2/link.2.html)
- [Linux man-pages：symlink(2)](https://man7.org/linux/man-pages/man2/symlink.2.html)
- [Linux man-pages：readlink(2)](https://man7.org/linux/man-pages/man2/readlink.2.html)
- [Linux man-pages：unlink(2)](https://man7.org/linux/man-pages/man2/unlink.2.html)
- [Linux man-pages：symlink(7)](https://man7.org/linux/man-pages/man7/symlink.7.html)
- [Linux man-pages：errno(3)](https://man7.org/linux/man-pages/man3/errno.3.html)
- [Linux man-pages：perror(3)](https://man7.org/linux/man-pages/man3/perror.3.html)
- [Linux man-pages：getrlimit(2)](https://man7.org/linux/man-pages/man2/getrlimit.2.html)
- [Linux 内核：fs sysctl](https://www.kernel.org/doc/html/latest/admin-guide/sysctl/fs.html)
- [Linux 内核：The Second Extended Filesystem](https://www.kernel.org/doc/html/v6.6/filesystems/ext2.html)
- [Linux 内核：ext4 inode.i_block](https://www.kernel.org/doc/html/latest/filesystems/ext4/ifork.html)
- [Linux 内核：ext4 Directory Entries](https://docs.kernel.org/filesystems/ext4/directory.html)
- [Linux 内核：Overview of the Linux Virtual File System](https://docs.kernel.org/filesystems/vfs.html)
- [Microsoft：Virtual Address Space](https://learn.microsoft.com/en-us/windows/win32/memory/virtual-address-space)
- [Microsoft：Virtual Address Space and Physical Storage](https://learn.microsoft.com/en-us/windows/win32/memory/virtual-address-space-and-physical-storage)
- [Microsoft：Virtual Address Spaces](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/virtual-address-spaces)
- [GNU Coreutils：stat invocation](https://www.gnu.org/software/coreutils/manual/html_node/stat-invocation.html)
- [NIST SP 800-88 Rev.2：Guidelines for Media Sanitization](https://csrc.nist.gov/pubs/sp/800/88/r2/final)
- [Ubuntu：The Linux command line for beginners](https://documentation.ubuntu.com/desktop/en/latest/tutorial/the-linux-command-line-for-beginners/)
- [Filesystem Hierarchy Standard 3.0](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html)
