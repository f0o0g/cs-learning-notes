<a id="part-3"></a>
# Linux 基础与系统编程学习手册 · 第 3 部分：Linux 系统编程

> 本册为《Linux 基础与系统编程学习手册》五册套装中的第 3 册（共 5 册），覆盖第 1—2 章。

## 分册导航

- [第 1 部分：Linux 命令与日常运维](Linux基础与系统编程学习手册_第1部分_Linux命令与日常运维.md)（第 1—11 章）
- [第 2 部分：编辑、编译、调试与构建](Linux基础与系统编程学习手册_第2部分_编辑编译调试与构建.md)（第 1—4 章）
- **第 3 部分：Linux 系统编程（本册，第 1—2 章）**
- [第 4 部分：操作系统内存管理](Linux基础与系统编程学习手册_第4部分_操作系统内存管理.md)（第 1 章）
- [第 5 部分：综合练习与附录](Linux基础与系统编程学习手册_第5部分_综合练习与附录.md)（第 1—5 章）

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

## 系统编程接口阅读约定

本部分把函数说明统一为“头文件与原型—参数—返回值—相关宏—示例与错误处理”。参数方向使用“输入”“输出”或“输入/输出”表示；指针参数还要说明所指对象的有效期和容量。系统调用或 POSIX 函数失败时，只有文档明确说明后才读取 `errno`，不能把成功调用之前残留的值当作本次结果。

C 程序入口统一写作：

```c
int main(int argc, char *argv[]);
```

| 参数 | 方向 | 含义 |
|---|---|---|
| `argc` | 输入 | 传入的参数字符串个数；普通 Shell 启动时包含程序启动名称，因此至少为 1 |
| `argv` | 输入 | 指向参数字符串指针数组；`argv[0]` 是启动名称，`argv[1]` 起是用户参数，`argv[argc]` 为空指针 |

**示例：如何从运行命令判断 argc 和 argv**

`argc` 和 `argv` 由程序启动环境准备并传给 `main`，不需要在 `main` 中自己给它们赋初值。下面以普通 Linux Shell 启动为例，假设已经将后面的程序编译为 `args_demo`，运行：

```bash
./args_demo apple "hello world" 42
```

可以按以下三步判断：

1. 先确定实际传入的字符串：启动名称为 `./args_demo`，三个用户参数依次为 `apple`、`hello world`、`42`。双引号使 `hello world` 保持为一个参数，外层引号由 Shell 移除，不进入字符串内容。
2. 再计数：包含启动名称，共有 **4 个字符串**，所以 `argc = 4`；用户参数个数为 `argc - 1 = 3`。不是按命令的字符数或原始空格数计数。
3. 最后编号：`argv` 从下标 0 开始，四个字符串对应 `argv[0]` 到 `argv[3]`；`argv[4]` 即 `argv[argc]` 为 `NULL`，这个结束标记不计入 `argc`。

| 表达式 | 指向的字符串内容或结束标记 |
|---|---|
| `argv[0]` | `./args_demo` |
| `argv[1]` | `apple` |
| `argv[2]` | `hello world` |
| `argv[3]` | `42` |
| `argv[4]` | `NULL`（空指针） |

这里每个有效的 `argv[i]` 都是指向字符串的指针。`argv[3]` 指向的 `42` 仍是两个字符组成的文本，不会自动变成整数 42；需要数值时再做转换。`argv[0]` 在本例中是 `./args_demo`，不能把它固定理解为程序的绝对路径。

**用程序核对**

创建 `args_demo.c`，完整内容如下：

```c
#include <stdio.h>

int main(int argc, char *argv[])
{
    printf("argc = %d\n", argc);
    for (int i = 0; i < argc; i++) {
        printf("argv[%d] = [%s]\n", i, argv[i]);
    }
    printf("argv[%d] == NULL: %s\n",
           argc, argv[argc] == NULL ? "yes" : "no");
    return 0;
}
```

编译并运行：

```bash
gcc -std=c11 -Wall -Wextra -pedantic args_demo.c -o args_demo
./args_demo apple "hello world" 42
```

本例的完整预期输出：

```text
argc = 4
argv[0] = [./args_demo]
argv[1] = [apple]
argv[2] = [hello world]
argv[3] = [42]
argv[4] == NULL: yes
```

输出中的方括号由 `printf` 添加，只用于标清字符串边界，不属于参数内容。循环使用 `i < argc`，只对有效字符串使用 `%s`；末尾的 `argv[argc]` 单独与 `NULL` 比较，不把空指针交给 `%s`。

**再看两种容易混淆的情况**

| 运行命令 | `argc` | 关键结果 |
|---|---|---|
| `./args_demo` | `1` | 没有用户参数，但启动名称仍占一项；`argv[1] == NULL` |
| `./args_demo ""` | `2` | `argv[1]` 指向空字符串，打印为 `argv[1] = []`；`argv[2] == NULL` |

空字符串长度虽然为 0，仍算一个参数：此时 `argv[1] != NULL`，而它指向的第一个字符是终止字符，即 `argv[1][0] == '\0'`。空字符串与用于结束数组的空指针是两回事。读取第一个用户参数前，应先确认 `argc > 1`。

“至少为 1”适用于上面的普通 Shell 启动方式；C 标准本身只保证 `argc` 非负。参数数组及末尾空指针的约定见 [C11 草案 §5.1.2.2.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=31)；引号移除和显式空参数的规则见 [GNU Bash：引号移除](https://www.gnu.org/software/bash/manual/html_node/Quote-Removal.html)与[词拆分](https://www.gnu.org/software/bash/manual/html_node/Word-Splitting.html)。

示例中的自定义辅助函数在首次出现处说明参数和返回约定；后文重复使用时不再复制同一张表。

常用 C/POSIX 辅助接口统一说明如下：

| 函数原型 | 参数解释 | 返回约定 |
|---|---|---|
| `int printf(const char *format, ...);` | `format` 是格式字符串，后续参数必须与转换说明匹配 | 返回输出字符数，失败返回负值 |
| `int fprintf(FILE *stream, const char *format, ...);` | `stream` 是目标流；其余参数同 `printf` | 返回输出字符数，失败返回负值 |
| `int fputs(const char *s, FILE *stream);` | `s` 是字符串；`stream` 是目标流 | 成功返回非负值，失败返回 `EOF` |
| `int puts(const char *s);` / `int putchar(int c);` | `s` 是字符串；`c` 是作为无符号字符输出的值 | 成功返回非负值，失败返回 `EOF` |
| `int strcmp(const char *s1, const char *s2);` | 两个参数均为以 `\0` 结尾的只读字符串 | 小于、等于或大于 0 分别表示字典序关系 |
| `size_t strlen(const char *s);` | `s` 必须是以 `\0` 结尾的有效字符串 | 返回终止空字符之前的字节数 |
| `unsigned long strtoul(const char *nptr, char **endptr, int base);` | `nptr` 是输入文本；`endptr` 接收停止位置；`base` 是 0 或 2—36 | 返回转换值；结合 `errno`、结束位置和范围判断失败 |
| `void *malloc(size_t size);` | `size` 是申请字节数 | 成功返回新内存，失败返回 `NULL` |
| `void free(void *ptr);` | `ptr` 必须为空或来自可释放分配且尚未释放 | 无返回值；调用后原指针不再有效 |
| `int snprintf(char *s, size_t n, const char *format, ...);` | `s` 是输出缓冲区；`n` 是容量；其余为格式参数 | 返回本应生成的长度；返回值大于等于 `n` 表示截断 |
| `unsigned int sleep(unsigned int seconds);` | `seconds` 是期望休眠秒数 | 正常完成返回 0；被信号中断时返回剩余秒数 |

`EXIT_SUCCESS` 和 `EXIT_FAILURE` 是 `<stdlib.h>` 定义的成功/失败退出状态宏；`NULL` 是空指针宏。`errno` 通常由 `<errno.h>` 以宏形式提供可修改的线程局部错误指示器，只能在接口报告失败后解释。

<a id="chapter-16"></a>
## 第 1 章 Linux 文件 I/O

Linux 文件 I/O 通常指由 `open`、`read`、`write`、`lseek`、`close` 等系统调用组成的低层接口。本章先说明接口与内核对象，再通过示例程序讲解完整操作流程。示例有意保留初学写法的常见问题（中文注释、拼写问题、未检查的返回值），纠错结论写在代码块外，常见错误统一汇总到纠错表。

### 1.1 文件 I/O、标准 I/O 与缓冲

#### 1.1.1 两套接口的关系

| 对比项 | 标准 I/O | Linux/POSIX 文件 I/O |
|---|---|---|
| 典型接口 | `fopen`、`fread`、`fwrite`、`fprintf`、`fclose` | `open`、`read`、`write`、`lseek`、`close` |
| 操作对象 | `FILE *` 流 | 非负整数文件描述符 |
| 缓冲 | C 库通常在用户空间缓冲 | 接口本身不提供 `FILE` 流缓冲，但内核仍可能缓存数据 |
| 可移植性 | 由 ISO C 定义，跨平台性通常更好 | 属于 POSIX/Linux 接口，移植到非 POSIX 系统时需适配 |
| 常见优势 | 格式化、按行处理、减少小块系统调用 | 更直接地控制描述符、标志、非阻塞与设备等对象 |

表中典型接口的用途可按下表对照理解：

| 用途 | 标准接口 | 低层接口 |
|---|---|---|
| 打开/创建对象 | `fopen`：按指定模式打开路径，成功返回 `FILE *` 流 | `open`：按标志打开或创建路径，成功返回文件描述符 |
| 读数据 | `fread`：从流按元素读入数据块 | `read`：从描述符按字节读入数据 |
| 写数据 | `fwrite`：向流按元素写出数据块 | `write`：向描述符按字节写出数据 |
| 格式化输出 | `fprintf`：把格式化文本写入流 | 无直接对应；通常先用 `snprintf` 格式化，再 `write` |
| 调整读写位置 | 对应 `fseek`/`ftell`（表中未列） | `lseek`：调整描述符的读写偏移 |
| 关闭 | `fclose`：刷新缓冲并关闭流 | `close`：关闭描述符，释放该编号 |

标准 I/O 往往在用户空间缓冲数据，缓冲区满足刷新条件时再调用 `write`。刷新条件可能包括缓冲区满、行缓冲遇到换行、显式 `fflush`、正常关闭流等；具体策略和容量由实现、流类型及运行状态决定。低层 I/O 不会因为“少一层库”就天然更快：大量小块 `write` 会增加系统调用次数，标准 I/O 的缓冲反而可能更高效。

“一切皆文件”强调许多内核对象可通过类似描述符的接口操作，例如普通文件、终端、管道、套接字和部分设备；它不是说所有对象都具有普通文件的全部语义。比如管道和套接字通常不能 `lseek`，不同对象对阻塞、权限和控制操作的支持也不同。

`FILE` 是实现相关的流对象，不能固定描述为“文件描述符、当前位置和一个 8192 字节缓冲区”的简单结构；8192 字节至多是某些实现中的常见值。标准流通常可通过 `fileno(stream)` 取得底层描述符，但不要直接猜测 `FILE` 的内部布局。混用同一打开文件上的标准 I/O 与低层 I/O 时，还要处理用户缓冲、文件偏移和刷新顺序，初学阶段应优先选定一套接口。

#### 标准 I/O 接口速查

```c
#include <stdio.h>
FILE *fopen(const char *restrict pathname, const char *restrict mode);
size_t fread(void *restrict ptr, size_t size, size_t nmemb, FILE *restrict stream);
size_t fwrite(const void *restrict ptr, size_t size, size_t nmemb, FILE *restrict stream);
int fflush(FILE *stream);
int fclose(FILE *stream);
int fileno(FILE *stream);
```

| 函数 | 参数说明 | 返回约定 |
|---|---|---|
| `fopen` | `pathname` 是路径；`mode` 是 `"r"`、`"w"`、`"a"` 及其 `+`/`b` 组合 | 成功返回 `FILE *`，失败返回 `NULL` |
| `fread`/`fwrite` | `ptr` 指向容量至少为 `size × nmemb` 的缓冲区；`stream` 是有效流 | 返回成功处理的元素数，不一定等于 `nmemb` |
| `fflush` | `stream` 指定输出流；传 `NULL` 表示刷新所有打开的输出流 | 成功为 0，失败为 `EOF` |
| `fclose` | `stream` 必须是尚未关闭的有效流 | 成功为 0，失败为 `EOF`；调用后不能再使用该流 |
| `fileno` | `stream` 是有效流 | 返回底层描述符，失败为 `-1`；描述符仍由流管理 |

`FILE` 是类型名，`EOF` 与 `NULL` 是 C 库宏：`EOF` 表示标准 I/O 的结束或失败返回标记，`NULL` 表示空指针常量。它们不等同于低层 `read` 返回 0 的语义。

### 1.2 进程与文件描述符

Linux 内核用进程控制块保存进程状态；在 Linux 实现中，核心对象是 `task_struct`（进程），其中关联的 `files_struct` 管理已打开文件。可把关系简化为：

![进程与文件描述符表、标准描述符及最小空闲编号复用](<assets/12_进程与文件描述符_4K.png>)

文件描述符（file descriptor，常简称 fd）就是这个表里的**下标编号**：一个小非负整数。它不是文件本身，也不存路径或权限信息，只是进程描述符表的一个索引——内核凭这个编号找到对应的已打开文件（记录着打开方式、当前偏移等），进而找到磁盘上的对象。因此：同一个文件被两次 `open` 会得到两个不同编号；编号只在当前进程内有意义，不同进程的 3 号指向的对象通常不同。程序终止或调用 `close` 后，编号被回收并可复用，但磁盘上的文件不受影响。描述符只是“凭据”，文件状态实际保存在内核的打开文件描述中，区别见 2.15。

程序应优先使用 `<unistd.h>` 中的符号，而不是散落的数字：

```c
STDIN_FILENO   /* 0 */
STDOUT_FILENO  /* 1 */
STDERR_FILENO  /* 2 */
```

`open` 成功时返回当前进程“最小的未使用文件描述符”。正常启动且三个标准描述符都存在时，第一个新描述符常是 3；关闭 3 后再次打开，3 通常会被复用。这只是分配规则的结果，不能假设任何一次调用必然返回固定数字，因为程序可能事先关闭标准流或打开了其他文件。

“4 GB 虚拟地址空间、3 GB 用户空间和 1 GB 内核空间”只适用于某些历史 32 位 Linux 配置；现代 64 位系统的地址空间布局、内核映射和安全随机化均不同。可复用的结论是：用户程序通过系统调用请求内核操作对象，而不是把某一种历史地址划分当成所有 Linux 的固定结构。

### 1.3 `open`、`close`、权限与错误

#### 接口用途

- `open`：打开已有对象，或在带 `O_CREAT` 时创建普通文件。
- `close`：释放当前进程中的描述符；之后该编号可以再次分配。

#### 头文件与原型

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode); /* 使用 O_CREAT 时 */
int close(int fd);
mode_t umask(mode_t mask);   /* 设置权限屏蔽位，返回修改前的掩码 */
```

| 参数 | 方向 | 含义 |
|---|---|---|
| `pathname` | 输入 | 以 `\0` 结尾的路径字符串，可以是绝对路径或相对当前工作目录的路径 |
| `flags` | 输入 | 一种访问模式与零个或多个状态/创建标志的按位或组合 |
| `mode` | 输入 | 仅在 `flags` 含 `O_CREAT` 或特定创建标志时提供，表示新对象的候选权限 |
| `fd` | 输入 | 当前进程中尚未关闭的文件描述符 |
| `mask` | 输入 | 新的权限屏蔽位；对应位会从后续创建对象的候选权限中清除 |

权限屏蔽位是进程级属性：`mask` 中为 `1` 的位表示创建新对象时要清除的权限位；屏蔽只影响新创建的对象，不改变已有对象的权限，其值由 `fork` 出的子进程继承。权限位构成、八进制写法与最终权限的计算式见下文 `mode_t mode` 的说明。

`open` 成功返回非负描述符，失败返回 `-1` 并设置 `errno`。返回的描述符是当前进程文件描述符表中“最小的未使用编号”：正常启动且三个标准描述符（0、1、2）都在时，第一次 `open` 通常返回 3；它是后续 `read`、`write`、`lseek`、`close` 操作该文件的唯一凭据，编号本身不含任何路径或权限信息。分配规则与复用现象见 1.2。

`close` 成功返回 0，失败返回 `-1`。`umask` 返回修改前的掩码且不会以 `-1` 报错。`close` 返回后不应再次使用或无条件重试同一编号，因为该编号可能已经被其他代码重新分配。

#### 访问模式和相关宏

`flags` 必须包含且只包含一种访问模式，再按位或上其他标志：

| 标志 | 含义 |
|---|---|
| `O_RDONLY` | 只读打开 |
| `O_WRONLY` | 只写打开 |
| `O_RDWR` | 读写打开 |
| `O_APPEND` | 每次写入前将写位置置于文件末尾 |
| `O_TRUNC` | 以可写方式打开已有普通文件时将长度截为 0 |
| `O_CREAT` | 文件不存在时创建；此时必须提供第三个 `mode` 参数 |
| `O_EXCL` | 与 `O_CREAT` 合用，目标已存在时失败，可避免无意覆盖 |
| `O_NONBLOCK` | 请求非阻塞语义；是否有效以及具体行为取决于对象类型 |
| `O_SYNC` | 请求写操作在同步完成要求满足后再返回，代价通常高于普通缓存写入 |

典型组合包括：`open("input.txt", O_RDONLY)` 只读打开输入；`open("app.log", O_WRONLY | O_CREAT | O_APPEND, 0666)` 创建或追加日志；`open("result.txt", O_WRONLY | O_CREAT | O_EXCL, 0600)` 仅在目标不存在时创建私有结果文件。所有调用都必须检查是否返回 `-1`，成功后还必须在所有控制路径上关闭描述符。完整错误处理见 1.4 的描述符复用程序和 1.6 的复制程序。

#### 权限位、umask 与最终权限

`mode_t mode` 只在创建新文件时决定候选权限。权限位按“属主/属组/其他用户 × 读/写/执行”共 9 位排列，每组内读 `4`（二进制 `100`）、写 `2`（二进制 `010`）、执行 `1`（二进制 `001`），习惯写成三位八进制；单个权限位对应的八进制值如下：

| 权限位 | 属主 | 属组 | 其他用户 |
|---|---|---|---|
| 读 r | `0400` | `0040` | `0004` |
| 写 w | `0200` | `0020` | `0002` |
| 执行 x | `0100` | `0010` | `0001` |

表中数值开头的 `0` 不是权限位，而是 C 语言八进制字面量的前缀：整数字面量以 `0` 开头时编译器按八进制解析，因此 `mode` 必须写成 `0644` 这类形式；若误写为 `644`，会被当作十进制六百四十四，换算成八进制是 `1204`，权限完全不同。Shell 命令里的 `chmod 644`、`umask 022` 由命令自身按八进制解析，不需要这个前缀。

一个 `mode` 值是若干权限位的按位或，例如 `0644` = 属主读+写（`0600`）加属组读（`040`）与其他用户读（`4`），即 `rw-r--r--`。父目录没有默认 ACL 时，新文件最终权限按下式受进程 `umask` 影响：

```text
最终权限 = mode & ~umask
```

例如 `mode` 为 `0666`、`umask` 为 `0022` 时，新文件通常得到 `0644`；`umask` 为 `0002` 时通常得到 `0664`。不要用普通程序永久调用 `umask(0000)` 来追求 `0666`，这会改变进程后续创建对象的权限策略；应让调用者的安全策略生效。

从设定到落定，权限位经历四个阶段。下面假设 `a.txt` 尚不存在、父目录没有默认 ACL，且创建成功；继承的 `0022` 仅为本例初始值：

```text
① 程序启动：继承父进程（本例为 Shell）的 umask = 0022   公式里 umask 的初始来源
        ↓
② mode_t old = umask(0077)：old = 0022，当前 umask = 0077   设置新屏蔽位并返回旧值；先设置，再调用 open
        ↓
③ open("a.txt", O_WRONLY | O_CREAT, 0666)：mode = 0666   mode 仍是你传的实参，umask() 不会修改它
        ↓
④ 最终权限 = 0666 & ~0077 = 0600（rw-------）   创建成功时记入 inode，后续修改 umask 不会改变已有文件权限
```

![umask 的继承与修改、mode 参数及文件最终权限](<assets/11_umask与文件创建权限_4K.png>)

图中阶段②使用的 `mode_t umask(mode_t mask)` 是设置屏蔽位并返回旧值的接口：`old` 保存原 umask `0022`，进程当前 umask 则已变为 `0077`。它并非只读查询；调用时会同时设置新值。

此后若要修改已落定的权限，只能用 `chmod()`/`fchmod()` 或 Shell 的 `chmod` 命令直接设置新权限位，不需要重新创建文件；两者的分工如下：

| | `umask(mask)` | `chmod()`/`fchmod()` |
|---|---|---|
| 改的对象 | 进程的屏蔽位（还没创建的文件） | 已存在文件的 inode 权限 |
| 生效时机 | 之后的所有创建调用 | 调用当时立即 |
| 是否经过 umask | —— | 不经过 |
| 返回值 | 修改前的旧 umask | 成功 0 / 失败 `-1` |

#### `errno` 与 `perror`

```c
void perror(const char *s);
```

`s` 是输出到标准错误前的上下文前缀；传空字符串或 `NULL` 时不输出前缀。`perror` 没有返回值，它根据当前 `errno` 追加对应错误文字，因此应紧接失败调用使用，避免中间调用改变 `errno`。

`errno` 是错误发生后供当前线程检查的错误编号；只有函数明确报告失败时才有意义，后续成功调用也不保证把它清零。它并不“保存在标准错误文件里”。`perror` 根据当前 `errno` 生成文字，并把前缀、冒号和错误说明写到标准错误流 `stderr`。

#### 常见错误

- `O_CREAT` 缺少第三个 `mode` 参数会导致未定义行为；`mode` 也不能代替访问模式。
- `O_TRUNC` 会在成功打开时立即清空文件。复制程序若先用它打开与源文件相同的目标，源数据会在比较之前丢失。
- `close` 失败也要记录，但某些错误后的重试语义与平台有关，不要无条件循环关闭同一编号。
- `write` 或 `close` 成功不等于数据已经持久化到物理介质；确有持久化要求时应根据协议使用 `fsync`，并处理它的返回值。

“先关闭标准输出 1，再立即 `open` 文件”虽然可能利用最小可用描述符规则取得 1，但调用之间若发生其他描述符分配就会失效。标准输出重定向应使用 `dup2`，完整实现见 2.16。Shell 的 `>` 也会完成重定向，但真实实现还要处理进程创建、描述符继承和错误传播。

### 1.4 准备文件、描述符复用与简化 `touch`

#### 准备实验目录

先建立实验目录和三个文件，再写入便于辨认的内容：

```bash
mkdir -p ~/linux-command-lab/session-04/fileio
cd ~/linux-command-lab/session-04/fileio/
touch aa bb cc
echo "aaa" > aa
echo "ccc" > cc
echo "bbb" > bb
vi open.c
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `mkdir -p ~/linux-command-lab/session-04/fileio` | 创建标准练习目录（含父目录） | 成功时无输出；`~/linux-command-lab` 下出现 `session-04/fileio` |
| 2 | `cd ~/linux-command-lab/session-04/fileio/` | 进入文件 I/O 实验目录 | 成功时无输出；后续命令均在 `fileio` 中执行 |
| 3 | `touch aa bb cc` | 创建三个空文件 | 成功时无输出，目录中出现 `aa`、`bb`、`cc` |
| 4 | `echo "aaa" > aa` | 把 `aaa` 和换行写入 `aa` | 成功时无终端输出，`aa` 内容被替换 |
| 5 | `echo "ccc" > cc` | 把 `ccc` 和换行写入 `cc` | 成功时无终端输出，`cc` 内容被替换 |
| 6 | `echo "bbb" > bb` | 把 `bbb` 和换行写入 `bb` | 成功时无终端输出，`bb` 内容被替换 |
| 7 | `vi open.c` | 打开或创建 `open.c` 进行编辑 | 进入 Vim 编辑界面，不产生普通终端输出 |

#### 描述符复用示例 `open.c`

`open.c` 先加入头文件和主函数骨架，再依次加入打开 `aa`、`bb` 和关闭 `aa` 后打开 `cc` 的代码。完整代码如下：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main()
{
    // 打开aa文件
    int fd_aa = open("aa", O_RDONLY);
    if (-1 == fd_aa) {
        perror("open aa fail");
        return -1;
    }

    // 打印aa的文件描述符
    printf("fd_aa = %d\n", fd_aa);
    // 打开bb文件
    int fd_bb = open("bb", O_RDONLY);
    if (-1 == fd_bb) {
        perror("open bb fail");
        close(fd_aa);
        return -1;
    }

    // 打印bb的文件描述符
    printf("fd_bb = %d\n", fd_bb);
    // 关闭aa文件
    close(fd_aa);
    // 打开cc文件
    int fd_cc = open("cc", O_RDONLY);
    if (-1 == fd_cc) {
        perror("open cc fail");
        close(fd_bb);
        return -1;
    }

    // 打印cc的文件描述符
    printf("fd_cc = %d\n", fd_cc);
    // 关闭bb文件和cc文件
    close(fd_bb);
    close(fd_cc);
    return 0;
}
```

编译、运行：

```bash
gcc open.c
ls -l
./a.out
```

示例输出为：

```text
fd_aa = 3
fd_bb = 4
fd_cc = 3
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `gcc open.c` | 编译 `open.c`，使用默认输出名 | 编译成功时无输出，生成 `a.out` |
| 2 | `ls -l` | 以长格式查看目录内容 | 显示文件权限、链接数、属主、大小和时间；这些数值是当时环境的实际值 |
| 3 | `./a.out` | 运行描述符复用示例 | 依次输出 `fd_aa = 3`、`fd_bb = 4`、`fd_cc = 3` |

三个数字对应描述符表的逐步变化：

```text
程序启动      0、1、2 被标准输入/输出/错误占用，最小未使用编号是 3
open("aa")    分配 3，打印 fd_aa = 3
open("bb")    3 已被占用，分配下一个最小未用编号 4，打印 fd_bb = 4
close(fd_aa)  释放 3；fd_bb 的 4 不受影响
open("cc")    最小未使用编号又变成 3，分配 3，打印 fd_cc = 3
```

`fd_cc = 3` 不代表它指向 `aa`：`close` 已清除编号 3 与 `aa` 的关联，新的 `open` 让同一编号指向新打开的 `cc`。上表现象只是“返回当前进程最小未使用描述符”规则的结果，不应把 3 当作固定返回值。示例代码没有检查 `close` 返回值，严谨写法应逐次检查，并用统一清理出口避免新增分支时遗漏描述符。

删除 `aa` 后再次运行，观察错误分支：

```bash
vi touch.c
rm -rf aa
./a.out
```

输出：

```text
open aa fail: No such file or directory
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `vi touch.c` | 开始编辑后续的简化 `touch` 程序 | 进入 Vim 编辑界面，不产生普通终端输出 |
| 2 | `rm -rf aa` | 强制递归删除路径 `aa` | 成功时无输出，`aa` 名称消失；该写法范围过宽，仅按示例保留，不应对不确定路径照搬 |
| 3 | `./a.out` | 再次运行 `open.c` 的编译产物 | 输出 `open aa fail: No such file or directory` |

对单个普通文件不需要递归强制删除，安全写法是 `rm -f -- aa`；若想在删除前确认，则使用 `rm -i -- aa`。错误文字来自 `perror` 对失败后 `errno` 的解释。

#### 简化版 `touch`

创建 `touch.c`，先检查命令行参数，再遍历多个文件名。示例代码如下：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argc, char* argv[])
{
    // 判断参数个数是否合法
    if (argc < 2) {
        printf("paramater error\n");
        return -1;
    }

    // 循环创建文件
    for (int i = 1; i < argc; i++) {
        int fd = open(argv[i], O_CREAT, 0666);
        if (-1 == fd) {
            perror("create file fail");
        } else {
            close(fd);
        }
    }
    return 0;
}
```

其中 `argc` 是包括程序名在内的参数个数，`argv[0]` 通常是启动程序时使用的名称，实际文件名从 `argv[1]` 开始。`paramater` 为示例保留的拼写，未改为 `parameter`。

常见错误及规范处理如下：

| 问题 | 原因 | 规范处理 |
|---|---|---|
| `open(path, O_CREAT, 0666)` | 缺少访问模式 | 使用 `O_WRONLY \| O_CREAT` |
| 无条件 `umask(0000)` | 改变进程后续创建策略 | 保留调用者的 `umask` |
| 只打开再关闭已有文件 | 不会实现 `touch` 的时间更新语义 | 对打开的文件调用 `futimens(fd, NULL)` |
| 忽略 `close` 返回值 | 延迟写入错误可能被遗漏 | 检查并报告关闭失败 |
| 使用 `rm -rf` 删除单文件 | 范围过宽 | 使用 `rm -i -- file` 或 `rm -f -- file` |

编译程序、创建多个文件并核对权限：

```bash
gcc touch.c -o touch
./touch ww ee rr
ls -l
touch
umask
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `gcc touch.c -o touch` | 编译示例并把产物命名为 `touch` | 编译成功时无输出，生成当前目录下的 `touch` |
| 2 | `./touch ww ee rr` | 运行自编程序并依次处理三个文件名 | 成功时无输出，目录中出现 `ww`、`ee`、`rr` |
| 3 | `ls -l` | 查看新文件及其权限 | 示例运行显示三个新文件，大小为 0，权限受当时的 `umask` 影响 |
| 4 | `touch` | 运行系统 `touch` 但不提供文件参数 | 运行时报告缺少文件操作数并给出帮助提示；这是故意保留的错误命令 |
| 5 | `umask` | 显示当前 Shell 的文件创建掩码 | 示例输出 `0002` |

本次示例的 `umask` 为 `0002`，新文件得到 `0664`；随后程序中又加入了：

```c
umask(0000);
```

它用于演示掩码变化，不应成为普通工具的默认行为。`0666 & ~0002` 得到 `0664`；若在创建前临时改为 `0000`，新文件才可能得到 `0666`，但旧文件权限不会因此自动改变。

`futimens` 需要 `<sys/stat.h>`，原型为 `int futimens(int fd, const struct timespec times[2]);`。`fd` 是输入的已打开文件描述符；`times` 是输入数组，非空时 `times[0]` 和 `times[1]` 分别指定访问时间与修改时间，传 `NULL` 则把二者都设为当前时间。成功返回 0，失败返回 `-1` 并设置 `errno`。该示例只保证文件存在，不会像完整 `touch` 命令那样更新时间；需要时间语义时，在打开的文件上调用 `futimens(fd, NULL)`。

### 1.5 系统级与进程级文件限制

#### 文件描述符限制的三个层次

查看系统文件句柄配置和当前 Shell 的资源限制：

```bash
cat /proc/sys/fs/file-max
ulimit -a
ulimit -n
```

示例环境中 `/proc/sys/fs/file-max` 显示 `201232`，`ulimit -a` 中 `open files` 显示 `1024`。输出取决于当前系统配置，不是 Linux 固定常量：

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `cat /proc/sys/fs/file-max` | 读取系统范围的文件句柄上限配置 | 示例运行输出 `201232`，其他系统可能不同 |
| 2 | `ulimit -a` | 显示当前 Shell 的全部资源限制 | 示例运行的 `open files (-n)` 为 `1024`，其余项目也取决于环境 |

- `RLIMIT_NOFILE`：单个进程可分配描述符编号范围的资源限制，包含软限制和硬限制；Shell 的 `ulimit -n` 通常查看或调整它。

`ulimit -n` 只显示当前打开文件数限制；`ulimit -a` 的 `open files (-n)` 字段也给出这一数值。
- `/proc/sys/fs/nr_open`：内核允许提高 `RLIMIT_NOFILE` 的上限之一。
- `/proc/sys/fs/file-max`：系统范围可分配文件句柄的上限；当前使用情况可结合 `/proc/sys/fs/file-nr` 观察。

程序可用 `getrlimit(RLIMIT_NOFILE, ...)` 查询软硬限制。普通用户通常只能把软限制提高到硬限制以内；提高硬限制需要相应权限和系统策略。遇到 `EMFILE` 通常表示进程自身达到限制，`ENFILE` 表示系统范围的打开文件资源耗尽。

```c
#include <sys/resource.h>
int getrlimit(int resource, struct rlimit *rlim);
```

| 参数 | 方向 | 含义 |
|---|---|---|
| `resource` | 输入 | 要查询的资源类别；`RLIMIT_NOFILE` 表示文件描述符数量限制 |
| `rlim` | 输出 | 指向 `struct rlimit`，成功后 `rlim_cur` 是软限制，`rlim_max` 是硬限制 |

成功返回 0，失败返回 `-1` 并设置 `errno`。`RLIMIT_NOFILE`、`EMFILE`、`ENFILE` 分别是资源类别宏和错误宏，不是固定数值；`EMFILE` 表示进程级耗尽，`ENFILE` 表示系统级耗尽。

### 1.6 `read`、`write` 与可靠复制

#### 接口、参数与返回值

```c
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
int fstat(int fd, struct stat *statbuf);
int ftruncate(int fd, off_t length);
int fsync(int fd);
```

| 参数 | 方向 | 含义 |
|---|---|---|
| `fd` | 输入 | 已打开且具备相应读、写或查询权限的描述符 |
| `buf` | `read` 输出；`write` 输入 | 指向至少 `count` 字节的有效缓冲区；`write` 不修改缓冲区 |
| `count` | 输入 | 最多读取或请求写入的字节数；为 0 时仍可能执行参数合法性检查 |
| `statbuf` | 输出 | 指向可写 `struct stat`，成功后接收描述符对象的元数据 |
| `length` | 输入 | `ftruncate` 设置的目标逻辑长度；缩短会丢弃尾部数据，扩大区域读出为零 |

`fstat`、`ftruncate` 和 `fsync` 成功返回 0，失败返回 `-1`。`fsync` 请求把文件数据及相关元数据同步到持久化介质，但不能自动保证父目录项也已持久化。

- `read` 返回正数表示实际读取的字节数，返回 0 表示本次调用已经位于 EOF，返回 `-1` 表示失败。
- 正数小于 `count` 是“短读”，对终端、管道、套接字以及接近普通文件末尾都可能正常发生。
- `write` 返回正数表示本次实际接受的字节数，可能小于请求长度；可靠程序必须继续写剩余部分。
- 调用被信号中断并返回 `-1`、`errno == EINTR` 时，通常应按程序语义重试。

`EINTR` 表示调用被信号中断；`EIO` 表示一般性 I/O 错误。`STDIN_FILENO`、`STDOUT_FILENO`、`STDERR_FILENO` 分别表示标准输入、输出、错误描述符，标准值为 0、1、2。

自定义函数 `write_all(int fd, const char *buf, size_t count)` 是可靠写出的常见写法：`fd` 是目标描述符，`buf` 指向待写数据，`count` 是总字节数；全部写完返回 0，任何不可恢复错误返回 `-1` 并保留或设置 `errno`。

#### 复制程序 `copy.c`

`copy.c` 先检查参数个数，再依次打开源文件和目标文件，最后用 100 字节缓冲区循环读写。完整代码如下：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>

int main(int argc, char* argv[])
{
    if (3 != argc) {
        printf("paramater error\n");
        return -1;
    }

    // 打开源文件，以只读权限，不存在会报错
    int fd_src = open(argv[1], O_RDONLY);
    if (-1 == fd_src) {
        perror("open src fail");
        return -1;
    }

    // 打开目标文件，以写权限，长度截断为0，不存在就创建
    umask(0000);
    int fd_dst = open(argv[2], O_WRONLY|O_TRUNC|O_CREAT, 0666);
    if (-1 == fd_dst) {
        perror("open dst fail");
        close(fd_src);
        return -1;
    }

    // 循环从源文件读，成功就写入目标文件，0就跳出循环
    int len = 0;
    char buf[100] = "";
    while (1) {
        len = read(fd_src, buf, sizeof(buf));
        if (len > 0) {
            write(fd_dst, buf, len);
        } else if (0 == len) {
            break;
        } else {
            perror("read error");
            break;
        }
    }

    // 关闭源文件和目标文件
    close(fd_src);
    close(fd_dst);
    return 0;
}
```

调用形式是 `./copy SOURCE DEST`，因此 `argc` 应为 3。这个写法有几处需要纠正：`len` 应为 `ssize_t`；`write` 要处理短写和 `EINTR`；`O_TRUNC` 在打开时就会清空目标，若 `SOURCE` 与 `DEST` 是同一 inode 的不同路径，源数据会在复制前丢失，严谨写法应先用 `fstat` 比较设备号和 inode，再截断目标；也不应无条件 `umask(0000)`，应保留调用者的掩码。

编译和测试顺序为：

```bash
cat /proc/sys/fs/file-max
ulimit -a
vi copy.c
gcc copy.c -o copy
./copy
./copy copy.c bb
./copy cc uu
ls -l
cat uu
cat cc
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `cat /proc/sys/fs/file-max` | 再次查看系统文件句柄上限 | 示例运行输出 `201232` |
| 2 | `ulimit -a` | 再次查看当前 Shell 的资源限制 | 输出包含示例运行的 `open files (-n) 1024` |
| 3 | `vi copy.c` | 打开或创建复制程序源码 | 进入 Vim 编辑界面，不产生普通终端输出 |
| 4 | `gcc copy.c -o copy` | 编译复制程序并命名为 `copy` | 编译成功时无输出，生成 `copy` |
| 5 | `./copy` | 不带参数运行复制程序 | 输出 `paramater error` |
| 6 | `./copy copy.c bb` | 把 `copy.c` 复制到 `bb` | 成功时无输出；示例运行的 `bb` 大小为 989 字节 |
| 7 | `./copy cc uu` | 把 `cc` 复制到 `uu` | 成功时无输出；示例运行的 `uu` 大小为 4 字节 |
| 8 | `ls -l` | 查看源文件和复制结果 | 示例运行显示 `copy.c` 与 `bb` 均为 989 字节，`cc` 与 `uu` 均为 4 字节 |
| 9 | `cat uu` | 显示目标文件 `uu` 的内容 | 输出 `ccc` |
| 10 | `cat cc` | 显示源文件 `cc` 的内容 | 输出 `ccc` |

无参数运行时输出 `paramater error`。本次运行中 `copy.c` 和复制出的 `bb` 大小为 989 字节，`cc` 与 `uu` 都是 4 字节；`cat uu` 和 `cat cc` 均输出 `ccc`。文件大小相同只能作为快速检查，还应使用 `cmp -s SOURCE DEST` 并检查退出状态，或用校验和确认内容一致。

这个示例复制内容，不负责完整保留时间戳、所有权、扩展属性或稀疏布局。若业务协议要求断电后也确认数据落盘，应在关闭前调用并检查 `fsync(dst)`；这样代价较高，也不能替代目录项持久化等更完整的事务设计。

### 1.7 `lseek`、文件偏移和文件大小

#### 接口、参数与返回值

```c
#include <sys/types.h>
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
```

| 参数 | 方向 | 含义 |
|---|---|---|
| `fd` | 输入 | 关联可定位打开文件描述的描述符 |
| `offset` | 输入 | 有符号偏移量，如何解释由 `whence` 决定 |
| `whence` | 输入 | `SEEK_SET`、`SEEK_CUR` 或 `SEEK_END`，分别以文件开头、当前偏移、文件末尾为基准 |

`lseek` 修改打开文件描述关联的当前偏移，成功返回从文件开头算起的新偏移，失败返回 `(off_t)-1`：

| `whence` | 新位置 |
|---|---|
| `SEEK_SET` | `offset` |
| `SEEK_CUR` | 当前偏移加 `offset` |
| `SEEK_END` | 文件末尾加 `offset` |

普通文件可用 `lseek(fd, 0, SEEK_END)` 得到文件尾偏移，但这样会改变当前偏移；只想取得普通文件大小时，`fstat` 的 `st_size` 往往更直接。某些文件可能存在空洞，逻辑大小不等于实际占用空间。管道、FIFO 和套接字等不可定位对象通常返回 `-1` 并设置 `errno` 为 `ESPIPE`。

`SEEK_SET`、`SEEK_CUR`、`SEEK_END` 是定位基准宏；`ESPIPE` 是“对象不可定位”的错误宏。代码不依赖它们的具体数值。

#### 定位并读取指定偏移

`lseek.c` 先打开 `bb`，获取末尾偏移，再定位到第 10 字节读取并写标准输出。示例代码如下：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main()
{
    // 打开bb文件
    int fd = open("bb", O_RDONLY);
    if (-1 == fd) {
        perror("open bb fail");
        return -1;
    }

    // 把光标移动到文件尾
    int size = lseek(fd, 0, SEEK_END);
    printf("size = %d\n", size);

    // 从文件头移动10个字节
    lseek(fd, 10, SEEK_SET);

    // 读取100个字节
    char buf[100] = "";
    int len = read(fd, buf, sizeof(buf));
    if(-1 == len)
    {
        perror("read fail");
        close(fd);
        return -1;
    }

    // 写到标准输出
    write(STDOUT_FILENO, buf, len);
    close(fd);
    return 0;
}
```

偏移 10 表示从文件开头起跳过 10 字节，下一次读取从第 11 个字节开始。示例代码把 `off_t` 和 `ssize_t` 都写成了 `int`，也没有检查 `lseek`、`read`、`write` 的返回值；规范写法应使用匹配类型并逐次检查，不可定位对象失败时 `errno` 为 `ESPIPE`。

完成后的编译和运行命令为：

```bash
vi lseek.c
gcc lseek.c
./a.out
```

示例先输出：

```text
size = 989
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `vi lseek.c` | 打开或创建偏移实验源码 | 进入 Vim 编辑界面，不产生普通终端输出 |
| 2 | `gcc lseek.c` | 编译源码，使用默认输出名 | 编译成功时无输出，生成 `a.out` |
| 3 | `./a.out` | 运行偏移读取程序 | 先输出示例运行值 `size = 989`，随后显示 `bb` 从偏移 10 开始的内容 |

随后输出 `bb` 从偏移 10 开始读取到的最多 100 字节。此处的 `bb` 是先前复制出的 `copy.c`，所以输出从 `stdio.h` 开始，正好跳过了前 10 个字节 `#include <`。偏移按字节计算，文件第一个字节的偏移为 0，所以偏移 10 指向第 11 个字节。

### 1.8 阻塞、非阻塞与 `fcntl`

#### 1.8.1 什么是阻塞

读普通文件时，程序通常很快就能得到数据、EOF 或错误；但“普通文件永远不会让调用者等待”并不严谨：缺页、磁盘 I/O 和文件系统活动仍可能使调用短暂等待，而且 Linux 当前会忽略普通文件和块设备上的 `O_NONBLOCK`。阻塞/非阻塞最容易在终端、管道、FIFO、套接字和部分设备上观察。

当阻塞式 `read` 暂时没有数据可返回时，调用线程会睡眠，内核可调度其他可运行线程。数据到达后，线程才有机会恢复运行。睡眠等待不等于“整个系统停住”，也不等于线程一被唤醒就必然立即占有 CPU。

可运行状态可以从两个角度观察：线程正在 CPU 上执行，或已经就绪并在运行队列中等待调度。这两种情况都不需要等待外部事件。阻塞睡眠则是在等待数据、定时器或其他事件，条件满足后才重新变为可运行。

非阻塞描述符暂时无数据时，`read` 返回 `-1`，并把 `errno` 设为 `EAGAIN`；套接字接口也可能报告 `EWOULDBLOCK`。Linux 上两者值相同，但可移植程序应同时接受。非阻塞只表示本次调用不为等待数据而睡眠，不表示操作一定成功，也不表示所有对象都支持这种语义。

| 方式 | 暂时无数据时 | 优点 | 代价 |
|---|---|---|---|
| 阻塞 | 当前线程睡眠，等待事件 | 不需要反复查询，事件到达后响应通常及时 | 单线程一次只能等待当前调用 |
| 非阻塞 | 立即返回 `EAGAIN`/`EWOULDBLOCK` | 可继续处理其他任务或其他描述符 | 紧密轮询浪费 CPU；带休眠轮询会增加响应延迟 |

生产程序通常使用 `poll`、`select`、`epoll` 或事件循环同时等待多个描述符。这里的示例为了突出 `O_NONBLOCK` 的行为，使用带休眠的轮询，不提前展开这些接口。

#### 1.8.2 已打开描述符的状态标志

可以在 `open` 时加入 `O_NONBLOCK`，也可以用 `fcntl` 修改已经打开的描述符。`open("/dev/tty", O_RDONLY | O_NONBLOCK)` 会尝试以非阻塞方式打开当前进程的控制终端；程序仍需检查返回值，并在使用后检查 `close`。若要修改程序启动时已经打开的 `STDIN_FILENO`，使用 `F_GETFL` 取得旧标志，再以 `F_SETFL` 写回 `flags | O_NONBLOCK`。1.9 的程序展示了设置和使用过程；退出前恢复旧标志的要求在代码块外说明。

```c
#include <fcntl.h>

int fcntl(int fd, int op, ...);
```

| 参数 | 方向 | 含义 |
|---|---|---|
| `fd` | 输入 | 要查询或修改的有效描述符 |
| `op` | 输入 | 操作宏；此处 `F_GETFL` 读取状态标志，`F_SETFL` 设置可修改的状态标志 |
| 第三个参数 | 输入或输出 | 类型由 `op` 决定；`F_GETFL` 不需要，`F_SETFL` 传入 `int` 标志集合 |

`F_GETFL` 成功返回当前标志，`F_SETFL` 成功返回 0；失败均返回 `-1` 并设置 `errno`。`O_NONBLOCK` 表示避免因等待数据而阻塞，`O_APPEND` 表示每次写入都以文件末尾为定位基准；`EAGAIN`/`EWOULDBLOCK` 表示非阻塞对象当前不能完成操作。

`F_GETFL` 返回访问模式和文件状态标志；`F_SETFL` 在 Linux 上只能改变 `O_APPEND`、`O_ASYNC`、`O_DIRECT`、`O_NOATIME`、`O_NONBLOCK` 等有限标志，不能把只读描述符改为可写，也不能重新设置 `O_CREAT`、`O_TRUNC` 等创建标志。先取得旧值再按位或，是为了保留可修改的其他状态标志，不是因为 `F_SETFL` 可以改变读写访问模式。

经 `dup` 或 `fork` 得到的描述符可能共享同一个打开文件描述对象及状态标志。示例程序在标准输入上临时设置 `O_NONBLOCK`，退出前应恢复旧标志，避免把状态遗留给启动它的 Shell。

### 1.9 阻塞与非阻塞终端读取示例

本节给出阻塞版与非阻塞版示例代码，可严格编译的写法差异在代码块外说明。

#### 阻塞读取程序 `block.c`

示例代码如下：

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    char buf[10] = "";
    int len = read(STDIN_FILENO, buf, sizeof(buf));
    write(STDOUT_FILENO, buf, len);
    return 0;
}
```

进入已有的 `fileio` 目录，编辑、编译并运行：

```bash
cd ~/linux-command-lab/session-04/fileio/
ls -l
vi block.c
gcc block.c -o block
./block
```

输入 `hello world` 后，终端中连续显示为：

```text
hello world
hello worlubuntu@ubuntu:~/linux-command-lab/session-04/fileio$ d
d: 未找到命令
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `cd ~/linux-command-lab/session-04/fileio/` | 进入文件 I/O 实验目录 | 成功时无输出，终端路径进入 `fileio` |
| 2 | `ls -l` | 查看已有源码和程序 | 显示当时目录中的文件清单、权限、大小与时间 |
| 3 | `vi block.c` | 打开或创建阻塞读取源码 | 进入 Vim 编辑界面，不产生普通终端输出 |
| 4 | `gcc block.c -o block` | 编译程序并命名为 `block` | 编译成功时无输出，生成 `block` |
| 5 | `./block` | 运行只读取 10 字节的阻塞示例 | 输入 `hello world` 后程序写出 `hello worl`，余下的 `d` 被 Shell 当作下一条命令并报告未找到 |

程序的 10 字节缓冲区只读走 `hello worl`；余下的 `d` 和换行仍在终端输入队列。程序退出、Shell 恢复读取后，会把 `d` 当作下一条命令。这不是 GCC 错误，而是终端规范模式、读取上限和 Shell 继承剩余输入共同产生的结果。该程序没有检查 `read`/`write` 返回值：若 `read` 失败，负数 `len` 传给 `write` 时会转换为很大的无符号长度；严谨写法应用 `ssize_t` 保存返回值、处理 `EINTR`、按实际长度写出并检查每次调用。

#### 非阻塞读取程序 `nonblock.c`

`nonblock.c` 先用 `fcntl` 为已打开的标准输入设置非阻塞标志，再循环读取，区分“暂时无数据”和真正的读取错误。完整代码如下：

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>

int main()
{
    // 1、改变标准输入文件的属性为非阻塞
    // 1.1、获取旧的文件属性
    int flag = fcntl(STDIN_FILENO, F_GETFL);
    // 1.2、增加非阻塞属性
    flag = flag | O_NONBLOCK;
    // 1.3、把新的属性设置到文件上
    fcntl(STDIN_FILENO, F_SETFL, flag);

    // 2、循环非阻塞读取文件
    int len = 0;
    char buf[10] = "";
    while(1) {
        // 从标准输入文件读取数据
        len = read(STDIN_FILENO, buf, sizeof(buf));
        // 判断返回值，如果返回值是-1
        if (-1 == len) {
            // 判断errno是不是EAGAIN
            if (errno == EAGAIN) {
                // 如果errno是EAGAIN，就说明是暂时没有数据，休眠一会，再继续
                sleep(5);
                continue;
            } else {
                // 失败，打印错误日志，结束程序
                perror("read fail");
                return 1;
            }
        } else {
            // 读取数据成功，写入到标准输出文件中，结束程序
            write(STDOUT_FILENO, buf, len);
            return 0;
        }
    }
}
```

该版本在 `EAGAIN` 时休眠 5 秒再继续，其他错误用 `perror` 报告，成功时写出实际读取长度并结束。它没有处理 `EWOULDBLOCK`、`EINTR` 和短写，忽略了 `write` 返回值；两次 `fcntl` 都没有检查返回值，`read` 返回类型应为 `ssize_t`，退出前也应恢复标准输入原标志。严谨写法应逐项检查，接受 `EAGAIN` 与 `EWOULDBLOCK`，并在所有退出路径恢复旧标志。

### 1.10 `ioctl`：设备专用控制接口（补充）

`read`、`write`、`lseek` 等接口描述了许多文件对象共有的操作；设备还有一些无法自然表达为字节读写的控制请求，例如查询终端窗口大小、设置串口参数或控制网络接口。`ioctl` 为这类**依赖具体文件类型或驱动**的请求提供统一入口：

```c
#include <sys/ioctl.h>

int ioctl(int fd, unsigned long request, ...);
int isatty(int fd);
```

- `fd`：已经打开的文件描述符。
- `request`：请求码，决定第三参数是否存在以及它的真实类型。
- 第三个参数：可能是整数、指针或结构体地址，必须严格按对应设备/API 文档传递。
- 成功返回值由具体请求定义；失败通常返回 `-1` 并设置 `errno`。

`isatty(fd)` 检查 `fd` 是否关联终端：是终端返回 1，否则返回 0；返回 0 时可查看 `errno` 区分“不是终端”和无效描述符等情况。`TIOCGWINSZ` 是读取终端窗口尺寸的请求宏，第三参数必须是可写的 `struct winsize *`；成功后主要读取 `ws_row` 和 `ws_col`。`EINVAL` 表示参数或请求不合法，`ENOTTY` 表示对象不支持该终端请求，`EFAULT` 表示传入地址无效。

不能根据函数原型中的省略号猜测第三参数，也不能把一个驱动的请求码用于另一类设备。错误的指针类型、结构体大小或请求码会导致 `EINVAL`、`ENOTTY`、`EFAULT` 等错误，严重时还可能改变设备状态。

下面用只读请求 `TIOCGWINSZ` 查询当前终端窗口尺寸：

```c
#define _POSIX_C_SOURCE 200809L

#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/ioctl.h>
#include <unistd.h>

int main(void)
{
    struct winsize ws;

    if (!isatty(STDOUT_FILENO)) {
        fputs("standard output is not a terminal\n", stderr);
        return EXIT_FAILURE;
    }
    if (ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1) {
        perror("ioctl TIOCGWINSZ");
        return EXIT_FAILURE;
    }
    printf("rows=%u columns=%u\n",
           (unsigned)ws.ws_row, (unsigned)ws.ws_col);
    return EXIT_SUCCESS;
}
```

编译、运行并验证非终端输出分支：

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic ioctl_winsize.c -o ioctl_winsize
./ioctl_winsize
./ioctl_winsize > out.txt    # 标准输出不再是终端，程序应走失败分支
```

排查时依次确认：描述符指向什么对象、请求码属于哪个头文件和设备、第三参数类型是否完全匹配、调用是否需要权限、失败后的 `errno` 是什么。需要等待多个描述符就绪时，应优先学习 `select`、`poll` 或 `epoll`，不要用任意 `ioctl` 轮询替代事件等待机制。

### 1.11 本章复现与检查清单

在个人练习目录依次验证：

```bash
gcc open.c                     # 编译描述符复用示例，默认输出名 a.out
gcc touch.c -o touch           # -o 指定输出名，产物为 touch
gcc copy.c -o copy             # 编译复制程序，产物为 copy
gcc lseek.c -o lseek           # 指定输出名，避免默认的 a.out 覆盖 open.c 的产物
gcc block.c -o block           # 编译阻塞读取示例，产物为 block
gcc nonblock.c -o nonblock     # 编译非阻塞读取示例，产物为 nonblock

./a.out                        # 运行 open.c 的产物：依次打开 aa、bb、cc 并打印描述符编号
rm -f -- aa && ./a.out         # 删除 aa 后再运行，观察 open aa fail 错误分支
./touch ww ee rr               # 用简化 touch 新建 ww、ee、rr，供下一条观察权限
ls -l                          # 查看新建文件的权限，结合 umask 解释最终权限
umask                          # 查看当前权限屏蔽位，验证 0666 & ~umask
cat /proc/sys/fs/file-max      # 查看系统级可打开文件描述上限
ulimit -a                      # 查看资源限制全集，其中 open files (-n) 是描述符上限
ulimit -n                      # 单独查看进程描述符上限，与上一条对照
./copy                         # 无参数运行，观察参数检查错误分支
./copy copy.c bb               # 把较长的 copy.c 复制到已有文件 bb，观察循环读写
./copy cc uu                   # 把 cc 复制为新文件 uu
ls -l                          # 对比复制前后各文件的大小
cat uu                         # 验证 uu 内容与 cc 一致
cat cc                         # 确认源文件 cc 未被改动
./lseek                        # lseek 编译产物，确认 size = 989 与偏移 10 之后的输出
./block                        # 输入超过 10 字节的内容，观察剩余输入被 Shell 解释
./nonblock                     # 暂不输入，观察非阻塞轮询等待行为
```

本章示例保留 `%d` 打印大类型、隐式声明等问题，作为纠错表的反面案例，编译时可能出现警告；不要为了消除警告而改动示例代码。还应覆盖：大于缓冲区的文件、目标已存在、源文件缺失、源目标相同、空文件、短文件、偏移超过文件尾、非阻塞等待和输入到达。测试只在隔离目录执行，输入文件保持只读。

<a id="chapter-17"></a>
## 第 2 章 Linux 文件系统

本章使用 ext2 的经典磁盘布局建立基础模型。ext2、ext3、ext4 是 Linux 支持的文件系统，不是“Linux 文件系统”的唯一选择；现代系统还常见 XFS、Btrfs 等。ext3 在 ext2 思路上加入日志，ext4 又加入 extent、校验和、延迟分配等机制，因此不能把本章所有数值和布局直接套到任意文件系统。

### 2.0 文件系统接口阅读表

本表集中给出本章首次使用的接口。除特别说明外，整数返回函数成功返回 0、失败返回 `-1` 并设置 `errno`；调用后的对象生命周期仍以各节说明为准。

#### 状态、权限与所有权

```c
#include <sys/stat.h>
#include <unistd.h>

int stat(const char *restrict path, struct stat *restrict buf);
int fstat(int fd, struct stat *buf);
int lstat(const char *restrict path, struct stat *restrict buf);
int fstatat(int dirfd, const char *restrict path,
            struct stat *restrict buf, int flags);
int access(const char *path, int mode);
int chmod(const char *path, mode_t mode);
int fchmod(int fd, mode_t mode);
int chown(const char *path, uid_t owner, gid_t group);
int fchown(int fd, uid_t owner, gid_t group);
int lchown(const char *path, uid_t owner, gid_t group);
```

| 函数 | 参数解释 | 返回与注意事项 |
|---|---|---|
| `stat` | `path` 是路径；`buf` 是可写 `struct stat *` | 跟随符号链接，返回目标状态 |
| `fstat` | `fd` 是有效描述符；`buf` 接收状态 | 查询描述符关联对象 |
| `lstat` | 参数同 `stat` | 路径是符号链接时查询链接本身 |
| `fstatat` | `dirfd` 是相对路径基准；`path` 是路径；`buf` 接收状态；`flags` 控制跟随规则 | `AT_FDCWD` 以当前目录为基准，`AT_SYMLINK_NOFOLLOW` 查询链接本身 |
| `access` | `path` 是路径；`mode` 是 `F_OK` 或 `R_OK/W_OK/X_OK` 的按位或 | 按真实 UID/GID 检查；不能替代实际操作的返回值检查 |
| `chmod`/`fchmod` | `path` 或 `fd` 指定对象；`mode` 是新权限位 | 最终结果还受文件系统、所有权和权限规则约束 |
| `chown`/`fchown` | `path` 或 `fd` 指定对象；`owner`、`group` 是新 UID/GID | 对某一项传 `(uid_t)-1`/`(gid_t)-1` 表示保持不变 |
| `lchown` | 参数同 `chown` | 修改符号链接本身的所有权，不跟随目标 |

`F_OK` 表示只检查存在性；`R_OK`、`W_OK`、`X_OK` 分别检查读、写、执行/搜索能力。`S_ISREG(mode)`、`S_ISDIR(mode)`、`S_ISLNK(mode)` 分别判断普通文件、目录和符号链接；它们是类型测试宏，不应与底层位掩码结果混用。`S_IFMT` 是文件类型位掩码，`S_IFREG`、`S_IFDIR`、`S_IFLNK`、`S_IFCHR`、`S_IFBLK`、`S_IFIFO`、`S_IFSOCK` 是类型编码宏，不依赖其具体数值。

#### 时间、长度与路径

```c
#include <fcntl.h>
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <utime.h>
#include <unistd.h>

int utime(const char *filename, const struct utimbuf *times);
int utimensat(int dirfd, const char *path,
              const struct timespec times[2], int flags);
int truncate(const char *path, off_t length);
int ftruncate(int fd, off_t length);
int rename(const char *oldpath, const char *newpath);
int chdir(const char *path);
int fchdir(int fd);
char *getcwd(char *buf, size_t size);
long pathconf(const char *path, int name);
long fpathconf(int fd, int name);
```

| 函数 | 参数解释 | 返回与注意事项 |
|---|---|---|
| `utime` | `filename` 指定路径；`times` 指向访问/修改秒级时间，传 `NULL` 表示当前时间 | 旧接口，精度和控制能力有限 |
| `utimensat` | `dirfd` 是相对路径基准；`path` 是路径；`times[0/1]` 是访问/修改时间；`flags` 控制链接处理 | `AT_FDCWD` 表示相对当前目录；可结合 `AT_SYMLINK_NOFOLLOW` |
| `truncate`/`ftruncate` | `path` 或 `fd` 指定文件；`length` 是目标逻辑长度 | 缩短会丢弃尾部数据，扩大区域读出为零 |
| `rename` | `oldpath` 是旧名称；`newpath` 是新名称 | 同一文件系统内按接口语义原子替换名称；跨文件系统通常报 `EXDEV` |
| `chdir`/`fchdir` | `path` 是目标目录；`fd` 是已打开目录描述符 | 成功后改变整个进程的当前工作目录 |
| `getcwd` | `buf` 是输出缓冲区；`size` 是容量 | 成功返回 `buf`；失败返回 `NULL` |
| `pathconf`/`fpathconf` | `path` 或 `fd` 指定对象；`name` 是 `_PC_*` 查询宏 | 返回限制值；返回 `-1` 时必须结合调用前清零的 `errno` 判断“无限制”还是失败 |

`AT_FDCWD` 表示以当前工作目录解析相对路径；`AT_SYMLINK_NOFOLLOW` 表示操作符号链接本身。`EXDEV` 表示跨设备/文件系统操作不受该接口支持。`UTIME_NOW`、`UTIME_OMIT` 可在纳秒接口中分别表示使用当前时间或保持该项不变。

#### 链接与目录

```c
#include <dirent.h>
#include <sys/stat.h>
#include <unistd.h>

int link(const char *oldpath, const char *newpath);
int symlink(const char *target, const char *linkpath);
ssize_t readlink(const char *restrict path, char *restrict buf, size_t bufsiz);
int unlink(const char *path);
int mkdir(const char *path, mode_t mode);
int rmdir(const char *path);
DIR *opendir(const char *name);
DIR *fdopendir(int fd);
struct dirent *readdir(DIR *dirp);
void rewinddir(DIR *dirp);
long telldir(DIR *dirp);
void seekdir(DIR *dirp, long loc);
int closedir(DIR *dirp);
FILE *tmpfile(void);
```

| 函数 | 参数解释 | 返回与生命周期 |
|---|---|---|
| `link` | `oldpath` 指定已有对象；`newpath` 是新硬链接名 | 成功新增目录项；通常不能跨文件系统 |
| `symlink` | `target` 是原样保存的目标字符串；`linkpath` 是新链接名 | 创建时不要求目标存在 |
| `readlink` | `path` 指定符号链接；`buf` 是输出缓冲区；`bufsiz` 是容量 | 返回写入字节数，不自动追加 `\0` |
| `unlink` | `path` 是要删除的非目录名称 | 删除目录项；对象是否释放取决于链接数和打开引用 |
| `mkdir` | `path` 是新目录路径；`mode` 是候选权限 | 最终权限受 `umask` 影响 |
| `rmdir` | `path` 是空目录路径 | 非空目录删除失败 |
| `opendir` | `name` 是目录路径 | 成功返回 `DIR *`，失败返回 `NULL` |
| `fdopendir` | `fd` 是已打开目录描述符 | 成功后目录流接管 `fd`，不应再单独关闭它 |
| `readdir` | `dirp` 是有效目录流 | 返回内部 `dirent *`；结束和失败都返回 `NULL`，用预先清零的 `errno` 区分 |
| `rewinddir` | `dirp` 是有效目录流 | 回到目录流开头，无返回值 |
| `telldir`/`seekdir` | `dirp` 是目录流；`loc` 必须是同一目录流先前返回的位置标记 | `loc` 不是通用字节偏移 |
| `closedir` | `dirp` 是尚未关闭的目录流 | 成功为 0；调用后流及底层描述符失效 |
| `tmpfile` | 无参数 | 创建自动清理的临时二进制流；成功返回 `FILE *`，失败返回 `NULL` |

`DIR` 是不透明目录流类型。`DT_UNKNOWN` 表示目录项没有提供可靠类型，程序应再用 `lstat` 或 `fstatat` 查询；其他 `DT_*` 值也只能作为文件系统提供的提示。Linux 的 `O_TMPFILE` 是创建无目录项临时 inode 的打开标志，只有部分文件系统支持，仍需按 `open` 规则提供访问模式和权限。

#### 描述符复制

```c
#include <unistd.h>
int dup(int oldfd);
int dup2(int oldfd, int newfd);
```

| 参数 | 方向 | 含义 |
|---|---|---|
| `oldfd` | 输入 | 要复制的有效描述符 |
| `newfd` | 输入 | `dup2` 指定的目标编号；若已打开，会先由内核原子替换 |

`dup` 返回最小可用新描述符；`dup2` 成功返回 `newfd`，失败返回 `-1`。新旧描述符共享同一打开文件描述、文件偏移和状态标志，但描述符自身的 close-on-exec 标志独立；`FD_CLOEXEC` 表示执行新程序时自动关闭该描述符。`F_DUPFD` 是 `fcntl` 的“从给定下界复制描述符”操作，`O_CLOEXEC` 使新描述符在创建时原子获得 close-on-exec 语义。

### 2.1 扇区、文件系统块与格式化

传统磁盘示意图把盘面划成磁道和扇区。以下采用“一个扇区 512 字节、一个文件系统块 4096 字节”的经典例子：

```text
1 个文件系统块 = 4096 字节
1 个传统逻辑扇区 = 512 字节
4096 / 512 = 8 个扇区
4096 × 8 = 32768 bit
```

这些不是所有设备的固定参数。现代磁盘可能使用 4096 字节物理扇区并向系统呈现 512 字节逻辑扇区；文件系统块大小也在创建文件系统时确定。`mke2fs -b` 可为 ext2/3/4 指定块大小，常见值为 1024、2048 或 4096 字节，实际默认值取决于工具、体系结构和文件系统规模。

分区和格式化是两件事：分区表描述存储设备上的区域，`mkfs`/`mke2fs` 在选定分区或设备上写入文件系统元数据。重新格式化会破坏该文件系统原有组织信息，不能把“只重装系统通常不影响其他分区”误解成“重新分区或格式化没有风险”。

文件的逻辑长度与实际分配空间也不同：

- 空普通文件通常有 inode，但不需要数据块。
- 非空文件通常按块分配空间，尾块即使只使用一部分，也会占用完整块。
- 稀疏文件的逻辑长度可以很大，但未写入区域不一定分配数据块。
- 压缩、去重、写时复制和内联数据等特性也会改变“长度与占用”的关系。

可用 `stat` 同时观察逻辑长度和已分配块数：

```bash
stat -c 'name=%n size=%s blocks=%b block-unit=%B io-block=%o' file
```

GNU `stat` 的 `%s` 是字节长度，`%b` 是已分配块数，`%B` 是 `%b` 使用的基本单位；它们不能简单等同。

### 2.2 ext2 块组与各区域

#### 经典 ext2 块组布局

![ext2 块组、位图、inode 表与数据块布局](<assets/ext2-block-group-corrected.png>)

下图用于辅助理解块组中各类元数据和数据区的相对关系。其核心关系可重建为：

```text
设备或分区
├─ 前 1024 字节：预留的启动记录区域
└─ ext2 文件系统
   ├─ block group 0
   │  ├─ superblock / 备份策略允许时的副本
   │  ├─ group descriptor table (GDT)
   │  ├─ block bitmap
   │  ├─ inode bitmap
   │  ├─ inode table
   │  └─ data blocks
   ├─ block group 1
   │  └─ 同类区域，具体备份布局由特性决定
   └─ ...
```

ext2 前 1024 字节为启动记录预留区；MBR/GPT 分区表并不能笼统说成“记录在每个文件系统的 boot block 中”。各区域作用如下：

| 区域 | 主要作用 |
|---|---|
| 超级块（superblock） | 记录整个文件系统的块大小、inode/块总量与空闲量、挂载和检查状态、特性标志等 |
| 块组描述符表（GDT） | 记录各块组的块位图、inode 位图、inode 表位置及空闲计数等 |
| 块位图 | 每一位描述本块组中相应块是否已分配 |
| inode 位图 | 每一位描述本块组中相应 inode 是否已分配 |
| inode 表 | 连续保存本块组的 inode 结构 |
| 数据块 | 保存普通文件内容、目录项、较长符号链接目标及部分元数据 |

不同对象使用数据区的方式不同：普通文件的数据放在数据块；目录的数据块保存目录项；较短符号链接目标可直接放在 ext inode 中，较长目标再分配数据块；设备文件通常把主/次设备号保存在 inode，FIFO 和套接字等特殊对象也不按普通文件方式保存内容数据块。

简化示意图把超级块和 GDT 画在每个块组中，便于解释备份目的；实际 ext2 可启用 `sparse_super`，只在选定块组保留备份。超级块描述的是同一个文件系统，不应把各块组误当成不同设备，也不应认为每份备份的块大小或文件系统版本互不相同。

块位图通常只需按位统计即可得到文件系统分配概况，因此 `df` 不必递归读取每个文件；`du` 要遍历指定目录树并读取目录项和 inode，所以大型目录中通常更慢。两者统计对象不同，结果不必完全一致。

inode 数量和数据块数量在创建文件系统时确定。`mke2fs -i` 可按“每多少字节配置一个 inode”调整创建策略，但格式化后不能把它当作普通运行参数随意修改。大量小文件可能先耗尽 inode，大文件为主时则可能先耗尽数据块。可分别检查：

```bash
df -h .
df -i .
du -sh .
```

### 2.3 inode、目录项与文件寻址

inode 保存文件对象的元数据和数据块映射，但不保存文件名。典型元数据包括类型、权限、UID、GID、链接数、逻辑长度、已分配块数以及访问/修改/状态变更时间。inode 大小由文件系统格式和创建参数决定；128 字节是早期 ext2 的常见值，不是所有 ext2/3/4 文件系统的固定值。

目录也是文件。目录的数据块保存“名称到 inode 号”的映射，因此同一 inode 可以由多个目录项引用。经典 ext 目录项至少要表达：

```text
inode number + record length + name length + file type（特性启用时）+ name
```

#### 目录项结构与类型编码

![目录项字段、inode 号与文件类型编码](<assets/03_目录项与inode_4K.png>)

图中给出名称、inode 号、记录长度和类型等字段。经典 ext 目录项的类型编码如下：

| 编码 | 文件类型 |
|---:|---|
| 0 | Unknown |
| 1 | Regular file |
| 2 | Directory |
| 3 | Character device |
| 4 | Block device |
| 5 | Named pipe |
| 6 | Socket |
| 7 | Symbolic link |

`Named pipe` 即 FIFO。更完整的经典 `ext*_dir_entry_2` 还需要名称长度；记录长度允许变长名称在块内紧凑排列。根目录 inode 号为 2 是 ext2/3/4 的约定，不应推广为所有文件系统的统一规则。目录可以占用一个或多个数据块，“目录大小总是 4096”同样只是特定环境中的常见现象。

用经典模型理解创建普通文件的步骤：

1. 在合适块组查找空闲 inode，设置 inode 位图相应位。
2. 初始化 inode 的类型、权限、属主、长度、时间和链接数等元数据。
3. 按实际需要分配数据块，设置块位图，并写入文件内容。
4. 在父目录数据块中加入“名称 → inode 号”的目录项。
5. 把数据块映射写入 inode，并以文件系统要求的顺序更新计数和元数据。

实际内核还要处理缓存、锁、日志或写回顺序，不能把上面的概括步骤当作真实内核调用顺序。读取路径时，内核逐级在目录中查找名称得到 inode，再根据 inode 的块映射读取内容。

文件的创建、删除与块寻址三类操作的整体关系如下图：

![ext2 文件的创建、删除与块寻址](<assets/ext2-add-delete-lookup.png>)

### 2.4 直接与间接数据块寻址

经典 ext2/3 inode 的 `i_block` 区域为 60 字节，可保存 15 个 4 字节块号：

```text
i_block[0..11]  ───────────────> 12 个直接数据块
i_block[12]     ──> 一级索引块 ──> 数据块
i_block[13]     ──> 二级索引块 ──> 一级索引块 ──> 数据块
i_block[14]     ──> 三级索引块 ──> 二级索引块 ──> 一级索引块 ──> 数据块
```

若块大小 `b = 4096` 字节、一个块号 4 字节，则一个索引块可放 `b / 4 = 1024` 个块号：

| 范围 | 可寻址数据量（仅按经典映射计算） |
|---|---:|
| 12 个直接块 | `12 × 4 KiB = 48 KiB` |
| 一级间接部分 | `1024 × 4 KiB = 4 MiB` |
| 二级间接部分 | `1024² × 4 KiB = 4 GiB` |
| 三级间接部分 | `1024³ × 4 KiB = 4 TiB` |

总映射能力还要加上前面的各级范围，实际最大文件长度会受到 on-disk 字段、内核实现和文件系统特性的其他限制。在此 4 KiB 经典模型中，12 个直接块覆盖 48 KiB，直接块用尽后才进入一级间接；“超过 4 KiB 就需要一级间接”的说法不正确。

访问直接块只需从 inode 取得块号；间接级数越多，需要经过的索引块越多，但缓存会减少实际磁盘读取。现代 ext4 的普通文件通常使用 extent 树描述连续区间，不再按上述 12+3 指针模型存放普通数据块映射；本节模型主要用于理解 ext2/3 和底层索引思想。

### 2.5 `stat` 命令与同名系统调用

GNU `stat` 命令报告文件或文件系统状态：

常用查询命令如下：

```bash
stat -c 'name=%n inode=%i links=%h type=%F size=%s blocks=%b' file
stat -L link-name
stat -f .
```

GNU `stat` 默认报告符号链接本身；`-L`/`--dereference` 才跟随链接。`stat -f` 报告文件系统信息。

C 接口名称相同但语义不同：

```c
#include <sys/stat.h>

int stat(const char *path, struct stat *buf);
int fstat(int fd, struct stat *buf);
int lstat(const char *path, struct stat *buf);
```

| 接口 | 对象来源 | 遇到符号链接 |
|---|---|---|
| `stat` | 路径 | 跟随到目标 |
| `lstat` | 路径 | 报告链接本身 |
| `fstat` | 已打开描述符 | 报告描述符引用的对象 |

`struct stat` 的常用成员包括：

| 成员 | 含义 |
|---|---|
| `st_dev`、`st_ino` | 所在设备和 inode 号，可共同识别同一文件对象 |
| `st_mode` | 文件类型和权限位 |
| `st_nlink` | 硬链接数 |
| `st_uid`、`st_gid` | 属主和属组 |
| `st_rdev` | 设备文件表示的设备号 |
| `st_size` | 逻辑长度（字节） |
| `st_blksize` | 推荐 I/O 块大小，不等于必然分配单位 |
| `st_blocks` | 已分配的 512 字节单位数量 |
| `st_atime` | 最近访问时间 |
| `st_mtime` | 最近内容修改时间 |
| `st_ctime` | 最近 inode 状态变更时间，不是创建时间 |

某些文件系统还提供 birth/creation time，但传统 `struct stat` 字段和工具支持并不统一，不能把 `ctime` 当创建时间。

### 2.6 硬链接与符号链接

硬链接是在目录中增加一个新名称，使它和已有名称指向同一 inode。两个名称地位相同，无法从 inode 判断哪个是“原文件”；硬链接通常不能跨文件系统，也不允许普通用户给目录创建硬链接。

符号链接是拥有自己 inode 的特殊文件，其内容是一个目标路径字符串。相对目标按符号链接所在目录解析；目标可以不存在，也可以在另一文件系统。较短目标在 ext 文件系统中常直接保存在 inode 的 `i_block` 区域，较长目标才需要数据块，因此不能说符号链接与目标“共用同一份文件内容”。

| 对比项 | 硬链接 | 符号链接 |
|---|---|---|
| 创建命令 | `ln TARGET LINK` | `ln -s TARGET LINK` |
| inode | 与已有名称相同 | 自己拥有 inode |
| 保存内容 | 新目录项引用同一 inode | 保存目标路径字符串 |
| 跨文件系统 | 不可以 | 可以 |
| 目标名称删除后 | 仍能访问同一 inode | 通常变成悬空链接 |
| 重新建立同名目标 | 不受影响，仍引用旧 inode | 会解析到新目标 |

#### 验证硬链接计数

依次执行以下命令：

```bash
cat uu
ln uu uu_h
ls -l
```

`cat uu` 输出：

```text
ccc
```

`ls -l` 的输出如下，列表在 `uu_h` 之后被截断：

```text
总用量 80
-rwxrwxr-x 1 ubuntu ubuntu 7612 8月  7 11:41 a.out
-rw-rw-r-- 1 ubuntu ubuntu  989 8月  7 11:07 bb
-rwxrwxr-x 1 ubuntu ubuntu 7424 8月  8 09:30 block
-rw-rw-r-- 1 ubuntu ubuntu  168 8月  8 09:30 block.c
-rw-rw-r-- 1 ubuntu ubuntu    4 8月  7 09:53 cc
-rwxrwxr-x 1 ubuntu ubuntu 7608 8月  7 11:06 copy
-rw-rw-r-- 1 ubuntu ubuntu  989 8月  7 11:06 copy.c
-rw-rw-rw- 1 ubuntu ubuntu    0 8月  7 10:14 ee
-rw-rw-r-- 1 ubuntu ubuntu  696 8月  7 11:41 lseek.c
-rwxrwxr-x 1 ubuntu ubuntu 7588 8月  8 09:51 nonblock
-rw-rw-r-- 1 ubuntu ubuntu 1027 8月  8 09:51 nonblock.c
-rw-rw-r-- 1 ubuntu ubuntu  791 8月  7 09:59 open.c
-rw-rw-rw- 1 ubuntu ubuntu    0 8月  7 10:14 rr
-rwxrwxr-x 1 ubuntu ubuntu 7488 8月  7 10:14 touch
-rw-rw-r-- 1 ubuntu ubuntu  450 8月  7 10:14 touch.c
-rw-rw-rw- 2 ubuntu ubuntu    4 8月  7 11:08 uu
-rw-rw-rw- 2 ubuntu ubuntu    4 8月  7 11:08 uu_h
```

`uu` 与 `uu_h` 长度同为 4，硬链接数都从 1 变为 2。

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `cat uu` | 显示原文件内容 | 输出 `ccc` |
| 2 | `ln uu uu_h` | 为 `uu` 创建硬链接名称 `uu_h` | 成功时无输出 |
| 3 | `ls -l` | 查看创建前后的目录项元数据 | 示例运行显示 `uu` 与 `uu_h` 大小均为 4、链接数均为 2；用户名、时间和其他文件大小均为现场值 |

上述实验只验证硬链接；符号链接、删除原名称和重建同名目标可用下面的扩展实验继续验证：

```bash
mkdir -p ~/linux-command-lab/session-05/links
cd ~/linux-command-lab/session-05/links
printf 'ccc\n' > uu

ln uu uu_h
ln -s uu uu_s
ls -li uu uu_h uu_s
stat -c '%n inode=%i links=%h type=%F size=%s' uu uu_h uu_s
readlink uu_s

rm uu
cat uu_h
cat uu_s                 # 目标名称不存在，应失败

printf 'new\n' > uu
cat uu_h                 # 仍为旧 inode 中的 ccc
cat uu_s                 # 现在解析到新建的 uu，输出 new
stat -c '%n inode=%i links=%h type=%F' uu uu_h uu_s
stat -L -c '%n -> inode=%i type=%F' uu_s
```

inode 号由当前文件系统分配，每次运行可能不同，应以实际输出为准。符号链接大小通常等于目标路径字符串的字节数，而不是笼统的“文件名长度”。

对应的系统调用为：

```c
#include <unistd.h>

int link(const char *oldpath, const char *newpath);
int symlink(const char *target, const char *linkpath);
ssize_t readlink(const char *path, char *buf, size_t bufsiz);
int unlink(const char *path);
```

`link` 在 Linux 上不能跨挂载点，目标名称已存在时也不会覆盖。`symlink` 创建链接本身，不要求目标当时存在。`readlink` 返回写入缓冲区的字节数且不会自动补 `\0`；把结果当 C 字符串前必须为终止符预留一字节，检查返回值不超过缓冲区容量，并在成功位置写入 `\0`。本节的命令实验可直接用 `readlink uu_s` 验证目标文本。

### 2.7 删除、打开文件与介质清理

`unlink` 删除的是名称。若该名称是最后一个硬链接且没有进程打开文件，文件对象才会被删除，相关空间变为可重用；若仍有打开描述符，文件会一直存在到最后一个描述符关闭。若路径指向符号链接，删除的是链接本身而不是目标。

可在隔离目录观察“名称已删除但描述符仍有效”：

```bash
printf 'still readable\n' > held
exec 3< held
rm held
cat <&3
exec 3<&-
```

底层释放会涉及目录项、链接计数、inode、数据块、位图、缓存和可能的日志更新，不能概括成“立刻把几个位清零”。释放空间通常不等于立刻覆写全部旧数据，但“删除的文件一定能恢复”同样错误：后续写入、日志、TRIM、闪存磨损均衡、加密以及文件系统实现都会影响恢复可能性。

出售、返修或报废设备时，不要把“下载文件写满一次”、普通格式化、泡水或随意破坏设备当作通用安全方案。应根据介质类型、数据敏感度和组织制度，使用厂商与适用标准认可的 clear、purge、cryptographic erase 或 destroy 流程，并验证结果；不熟悉的物理销毁应交给合规服务方。

### 2.8 文件状态查询实验

`stat`、`fstat`、`lstat` 成功返回 `0`，失败返回 `-1` 并设置 `errno`。在 Vim 中把光标放到函数名上，可用 `2K` 打开第 2 节系统调用手册；库函数通常查第 3 节，可用 `3K`。退出手册页按 `q`。

`ino_t` 和 `off_t` 不保证等同于 `int`，不能直接用 `%d` 输出。规范代码把它们转换到足够宽的标准整数类型，并使用匹配的格式说明符。

| 常见问题 | 规范处理 |
|---|---|
| `stat` 与 `lstat` 的对象混淆 | 明确前者跟随符号链接、后者查询链接本身 |
| 用 `%d` 输出 inode 和文件长度 | 使用 `<inttypes.h>` 与明确转换 |
| 仅打印结果、不检查失败 | 每次调用都检查 `-1` 并调用 `perror` |

示例程序 `stat.c` 如下。它先跟随 `aa_l` 查询目标文件，再查询符号链接本身：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

int main()
{
    struct stat st;
    if (-1 == stat("aa_l", &st)) {
        perror("stat aa_l fail");
        return 1;
    }
    printf("ino=%d, size = %d\n", st.st_ino, st.st_size);
    if (-1 == lstat("aa_l", &st)) {
        perror("stat aa_l fail");
        return 1;
    }
    printf("ino=%d, size = %d\n", st.st_ino, st.st_size);
    return 0;
}
```

编译和运行过程如下。`ino_t` 在该环境中是 `long unsigned int`，`off_t` 是 `long int`，因此 `%d` 产生四条格式警告；程序仍生成默认可执行文件 `a.out`：

```text
$ ls -l
总用量 8
-rw-rw-r-- 1 ubuntu ubuntu  20 8月 10 09:22 aa
lrwxrwxrwx 1 ubuntu ubuntu   2 8月 10 09:22 aa_l -> aa
-rw-rw-r-- 1 ubuntu ubuntu 390 8月 10 09:28 stat.c
$ gcc stat.c
stat.c: In function ‘main’:
stat.c:13:9: warning: format ‘%d’ expects argument of type ‘int’, but argument 2
    has type ‘__ino_t {aka long unsigned int}’ [-Wformat=]
    printf("ino=%d, size = %d\n", st.st_ino, st.st_size);
           ^
stat.c:13:9: warning: format ‘%d’ expects argument of type ‘int’, but argument 3
    has type ‘__off_t {aka long int}’ [-Wformat=]
stat.c:18:9: warning: format ‘%d’ expects argument of type ‘int’, but argument 2
    has type ‘__ino_t {aka long unsigned int}’ [-Wformat=]
    printf("ino=%d, size = %d\n", st.st_ino, st.st_size);
           ^
stat.c:18:9: warning: format ‘%d’ expects argument of type ‘int’, but argument 3
    has type ‘__off_t {aka long int}’ [-Wformat=]
$ ./a.out
ino=828843, size = 20
ino=828844, size = 2
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `ls -l` | 查看 `aa`、`aa_l` 和 `stat.c` 的元数据 | 示例运行显示 `aa` 为 20 字节、`aa_l` 指向 `aa` 且链接文本长度为 2；时间、用户名为现场值 |
| 2 | `gcc stat.c` | 编译状态查询程序 | 产生四条 `%d` 与 `ino_t`/`off_t` 类型不匹配警告，但仍生成 `a.out` |
| 3 | `./a.out` | 运行 `stat`/`lstat` 对比程序 | 示例运行输出目标 inode/大小 `828843, 20`，以及链接 inode/大小 `828844, 2`；inode 号不具有通用性 |

同一运行结果随后再次显示，并开始编辑权限示例：

```text
$ vi chmod.c
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `vi chmod.c` | 打开或创建权限修改示例源码 | 进入 Vim 编辑界面，不产生普通终端输出 |

`aa` 的逻辑长度是 20 字节；`aa_l` 保存目标路径 `aa`，所以 `lstat` 报告链接本身长度为 2 字节。不同运行中的 inode 号会变化，不能把上述数值当作固定结果。消除类型不匹配的规范写法是把 `st_ino`、`st_size` 转换到 `<inttypes.h>` 的 `uintmax_t`、`intmax_t`，再用 `%ju`、`%jd` 输出，例如：

```c
printf("inode=%ju size=%jd\n",
       (uintmax_t)st.st_ino, (intmax_t)st.st_size);
```

### 2.9 `access`、`chmod` 与权限参数

相关接口为：

```c
#include <unistd.h>
#include <sys/stat.h>

int access(const char *path, int mode);
int chmod(const char *path, mode_t mode);
int fchmod(int fd, mode_t mode);
```

`access` 的 `mode` 可以是 `F_OK`，或 `R_OK | W_OK | X_OK` 的组合。它按调用进程的真实 UID/GID 检查，而实际 `open` 等操作通常按有效身份检查；因此它主要回答“发起程序的用户是否有权”，不能把“先 `access`、后操作”当作安全授权方案，两次调用之间路径可能被替换。真正操作仍必须检查自己的返回值。

权限解析和修改的常见错误如下：

| 问题 | 规范处理 |
|---|---|
| 用 `atoi` 接收权限文本 | 使用 `strtoul(text, &end, 8)` 并检查完整消费 |
| 自写逐位“十进制转换” | 整数没有十/八进制存储之分，直接按基数 8 解析 |
| 先用 `access` 再假定后续成功 | 实际 `chmod` 仍独立检查返回值，避免竞态假设 |
| 缺少函数声明或返回值 | 在调用前声明，并覆盖所有返回路径 |

完整代码如下：

```c
#include <stdio.h>
#include <sys/stat.h>
#include <unistd.h>

// 权限转换成十进制
int convert(int o)
{
    int count = 1;
    int res = 0;
    // 循环，每次取出最低位，把最低位删掉
    while (o > 0) {
        res += (o % 10) * count;
        o = o / 10;
        count *= 8;
    }
    return res;
}

int main(int argc, char* argv[])
{
    // 判断参数个数是否合法 (=3)
    if (3 != argc) {
        printf("paramater error\n");
        return 1;
    }
    // 判断被修改文件是否存在
    if (-1 == access(argv[2], F_OK)) {
        perror("access fail");
        return 1;
    }
    // 权限转换成十进制
    int mod = convert(atoi(argv[1]));
    // 修改文件权限
    if (-1 == chmod(argv[2], mod)) {
        perror("chmod fail");
        return 1;
    }

    return 0;
}
```

这里的“转换成十进制”只是把文本中每个八进制数字解释成数值：`count *= 8` 按八进制加权，整数在内存中并没有“十进制存储”或“八进制存储”的区别。程序没有包含 `atoi` 所需的 `<stdlib.h>`，会产生隐式声明警告；规范写法直接用 `strtoul(text, &end, 8)` 解析并检查完整消费，不使用 `atoi`。

运行命令和结果为：

```text
$ ls -l
总用量 28
-rw-rw-r-- 1 ubuntu ubuntu   20 8月 10 09:22 aa
lrwxrwxrwx 1 ubuntu ubuntu    2 8月 10 09:22 aa_l -> aa
-rwxrwxr-x 1 ubuntu ubuntu 7584 8月 10 09:29 a.out
-rwxrwxr-x 1 ubuntu ubuntu 7512 8月 10 09:46 chmod
-rw-rw-r-- 1 ubuntu ubuntu  718 8月 10 09:46 chmod.c
-rw-rw-r-- 1 ubuntu ubuntu  390 8月 10 09:28 stat.c
$ ./chmod
paramater error
$ ./chmod 0642 xx
access fail: No such file or directory
$ ./chmod 0642 aa
$ ls -l
总用量 28
-rw-r---w- 1 ubuntu ubuntu   20 8月 10 09:22 aa
lrwxrwxrwx 1 ubuntu ubuntu    2 8月 10 09:22 aa_l -> aa
-rwxrwxr-x 1 ubuntu ubuntu 7584 8月 10 09:29 a.out
-rwxrwxr-x 1 ubuntu ubuntu 7512 8月 10 09:46 chmod
-rw-rw-r-- 1 ubuntu ubuntu  718 8月 10 09:46 chmod.c
-rw-rw-r-- 1 ubuntu ubuntu  390 8月 10 09:28 stat.c
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `ls -l` | 查看修改前的权限和目录状态 | 示例运行中 `aa` 权限为 `rw-rw-r--`，其余大小、用户名和时间均为现场值 |
| 2 | `./chmod` | 不带参数运行自编权限程序 | 输出 `paramater error` |
| 3 | `./chmod 0642 xx` | 尝试修改不存在的 `xx` | 输出 `access fail: No such file or directory` |
| 4 | `./chmod 0642 aa` | 把 `aa` 的权限参数设为示例输入的 `0642` | 成功时无输出 |
| 5 | `ls -l` | 查看修改后的权限 | 示例运行中 `aa` 变为 `rw-r---w-` |

无参数运行输出 `paramater error`（示例保留的拼写）；目标不存在时 `access` 分支报错；`0642` 成功后 `aa` 的权限从 `rw-rw-r--` 变为 `rw-r---w-`。

随后还使用了这些命令：

```text
$ man utime
$ man pathconf
$ vi ls.c
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `man utime` | 打开 `utime` 手册页 | 进入手册阅读界面，不产生持久文件变化 |
| 2 | `man pathconf` | 打开 `pathconf` 手册页 | 进入手册阅读界面，不产生持久文件变化 |
| 3 | `vi ls.c` | 打开或创建简化 `ls` 源码 | 进入 Vim 编辑界面，不产生普通终端输出 |

另一次多路径 `ls` 的可见输出为：

```text
$ ls / ~ /dh
ls: 无法访问'/dh': 没有那个文件或目录
/:
bin    dev   initrd.img  media  proc  sbin  sys  var
boot   etc   lib         mnt    root  snap  tmp  vmlinuz
cdrom  home  lost+found  opt    run   srv   usr

/home/ubuntu:
linux-command-lab           公共的  图片  音乐
network_4_03_vim.tar.gz     模板    文档  桌面
vmware-tools-distrib        视频    下载
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `ls / ~ /dh` | 同时列出根目录、主目录和不存在的 `/dh` | 先报告 `/dh` 不存在，再显示示例运行环境中的 `/` 与 `/home/ubuntu` 清单；用户名和文件集合依环境而异 |

验证时可分别测试合法模式 `0642`、目标不存在、缺失参数和权限不足，并用 `stat -c '%a %n'` 核对实际结果。

### 2.10 所有权、时间、长度与路径接口

本组接口覆盖文件所有者、时间戳、逻辑长度和路径操作：

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <utime.h>

int chown(const char *path, uid_t owner, gid_t group);
int fchown(int fd, uid_t owner, gid_t group);
int lchown(const char *path, uid_t owner, gid_t group);
int utime(const char *path, const struct utimbuf *times);
int truncate(const char *path, off_t length);
int ftruncate(int fd, off_t length);
int rename(const char *oldpath, const char *newpath);
int chdir(const char *path);
int fchdir(int fd);
char *getcwd(char *buf, size_t size);
long pathconf(const char *path, int name);
long fpathconf(int fd, int name);
```

`utime` 使用的时间结构包含两个 `time_t` 成员：

```c
struct utimbuf {
    time_t actime;   /* access time */
    time_t modtime;  /* modification time */
};
```

- `chown` 跟随符号链接，`lchown` 修改链接本身，`fchown` 操作已打开描述符。能否改变 UID/GID 受所有权、组成员关系和 `CAP_CHOWN` 等权限规则约束，不能简单概括成“只有 root 才能调用”。
- `utime` 修改访问时间和修改时间；传入 `NULL` 时请求使用当前时间。新程序通常优先用纳秒精度的 `utimensat`/`futimens`。`touch` 对不存在的路径可创建文件，对已存在文件则更新时间。
- `truncate`/`ftruncate` 把逻辑长度精确设为 `length`。缩短会丢弃尾部；延长部分读取为零，文件系统可以用稀疏洞实现。`open(..., O_TRUNC)` 则把可写普通文件截为 0 字节。
- `rename` 在同一文件系统中完成名称移动；替换目标时具有原子可见性，但跨挂载点通常以 `EXDEV` 失败。`mv` 还需处理跨文件系统复制后删除等情况。
- `chdir`/`fchdir` 改变进程当前工作目录，`getcwd` 取得绝对路径。改变工作目录会影响以后相对路径的解析。
- `pathconf`/`fpathconf` 查询路径相关限制，如 `_PC_LINK_MAX`、`_PC_NAME_MAX`、`_PC_PATH_MAX`、`_PC_PIPE_BUF`。返回 `-1` 既可能表示错误，也可能表示限制不确定；调用前应令 `errno = 0`，返回后结合 `errno` 判断。

### 2.11 链接删除与匿名临时文件

`link` 原子地增加一个指向同一 inode 的目录项，但硬链接不能跨挂载点，跨文件系统会以 `EXDEV` 失败。此前 2.6 的 `symlink`、`readlink`、`unlink` 实验同样适用于本节。

创建临时文件后立即 `unlink` 是一种常见模式：名称马上消失，进程仍可通过已打开描述符读写；描述符全部关闭后，对象即可释放。进程崩溃或系统掉电时，具体回收由文件系统的一致性与孤儿 inode 恢复机制处理，不能描述成“下次启动由某个应用扫描链接数并删除”。优先考虑 `tmpfile()`、Linux 的 `O_TMPFILE`，或 `mkstemp` 后立即 `unlink`，并检查每一步错误。

### 2.12 目录创建与目录流

```c
#include <sys/stat.h>
#include <unistd.h>
#include <dirent.h>

int mkdir(const char *path, mode_t mode);
int rmdir(const char *path);
DIR *opendir(const char *name);
DIR *fdopendir(int fd);
struct dirent *readdir(DIR *dirp);
void rewinddir(DIR *dirp);
long telldir(DIR *dirp);
void seekdir(DIR *dirp, long loc);
int closedir(DIR *dirp);
```

`mkdir` 的最终权限还受 `umask` 影响；`rmdir` 只删除空目录。`opendir`/`fdopendir` 打开目录流；`fdopendir` 成功后由目录流接管传入描述符。`readdir` 每次返回一个目录项指针，该存储可被下一次调用覆盖且不应由调用者释放；`closedir` 同时关闭底层描述符。`rewinddir` 回到开头；`telldir` 返回不透明位置，只有把它原样交给同一目录流的 `seekdir` 才有意义。

Linux 常见的 `struct dirent` 字段如下：

| 字段 | 含义与边界 |
|---|---|
| `d_ino` | inode 号 |
| `d_off` | 下一目录项的位置标记，不应当作普通字节偏移 |
| `d_reclen` | 本记录长度 |
| `d_type` | 文件类型提示；并非所有文件系统都支持，可能为 `DT_UNKNOWN` |
| `d_name` | 以 `\0` 结尾的文件名 |

`readdir` 返回 `NULL` 既可能是正常到达末尾，也可能是错误。严谨写法是在循环前把 `errno` 设为 0，结束后再检查。返回顺序不是排序结果；真正的 `ls` 通常还要排序、排版、着色，并依据选项处理隐藏项。

### 2.13 简化版 `ls`

实现简化版 `ls` 时的常见错误如下：

| 问题 | 规范处理 |
|---|---|
| `prinFile`/`printFile` 混用 | 统一声明和调用名称 |
| `%d` 输出 `d_name` | 文件名是字符串，使用 `%s` |
| `strcmp` 隐式声明 | 包含 `<string.h>` |
| 假定 `readdir` 顺序等同 `ls` | 如需稳定顺序，收集后显式排序 |
| 把 `d_type` 当成总是有效 | `DT_UNKNOWN` 时使用 `lstat`/`fstatat` |

| 自定义函数 | 参数 | 返回约定 |
|---|---|---|
| `walkDir(path)` | `path` 是要读取的目录路径 | 无返回值；`opendir` 失败时打印错误后返回 |
| `printFile(path)` | `path` 是待判断路径 | 无返回值；普通文件直接输出，目录交给 `walkDir` |

`ls.c` 用 `walkDir` 读取并打印目录项，用 `printFile` 区分普通文件与目录，并在 `main` 中按参数个数选择处理方式。完整代码如下：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <dirent.h>

// 进入文件夹
void walkDir(char* path){
    // 打开文件夹
    DIR* dr = opendir(path);
    if (NULL == dr) {
        perror("opendir fail");
        return;
    }
    // 循环读取文件夹记录项
    struct dirent* di = NULL;
    while (1) {
        di = readdir(dr);
        if (NULL == di) {
            break;
        } else if (strcmp(".", di->d_name) == 0 || strcmp("..", di->d_name) == 0) {
            continue;
        } else {
            printf("%s  ", di->d_name);
        }
    }
    // 关闭文件
    closedir(dr);
}

// 打印指定路径
void printFile(char* path) {
    // 判断路径是否存在，不存在就报错
    struct stat st;
    if (-1 == stat(path, &st)) {
        perror("stat fail");
        return;
    }
    // 判断是否是文件夹，如果是文件就直接打印文件名
    if ((st.st_mode & S_IFMT) != S_IFDIR) {
        printf("%s|", path);
    } else {
        // 如果是文件夹，就进入文件
        walkDir(path);
    }
}

int main(int argc, char* argv[])
{
    // 判断，只有一个参数，就打印当前路径
    if (1 == argc) {
        prinFile(".");
    } else if (2 == argc) {
        // 有2个参数，就打印指定路径
        printFile(argv[1]);
    } else {
        // 有超过2个参数，循环打印每一个路径
        for (int i = 1; i < argc; i++) {
            printf("%s:\n", argv[i]);
            prinFile(argv[i]);
            printf("\n\n");
        }
    }
    printf("\n");
    return 0;
}
```

`main` 中两处 `prinFile` 是示例保留的拼写，与 `printFile` 并不一致，会导致链接错误。类型判断用 `(st.st_mode & S_IFMT) != S_IFDIR`，其掩码取值可参考手册：

```text
The following mask values are defined for the file type component of the
st_mode field:

S_IFMT    0170000   bit mask for the file type bit field
S_IFSOCK  0140000   socket
S_IFLNK   0120000   symbolic link
S_IFREG   0100000   regular file
S_IFBLK   0060000   block device
S_IFDIR   0040000   directory
S_IFCHR   0020000   character device
S_IFIFO   0010000   FIFO

Thus, to test for a regular file (for example), one could write:

stat(pathname, &sb);
if ((sb.st_mode & S_IFMT) == S_IFREG) {
    /* Handle regular file */
}
```

若误用 `%d` 打印 `d_name`，编译警告为：

```text
$ gcc ls.c -o ls
ls.c: In function ‘walkDir’:
ls.c:22:11: warning: format ‘%d’ expects argument of type ‘int’, but argument 2 has type ‘char *’ [-Wformat=]
    printf("%d  ", di->d_name);
           ^
```

改为 `%s` 后编译运行，与系统 `ls` 对比：

```text
$ ./ls
a.out  chmod.c  ..  ls  ls.c  chmod  aa  aa_l  stat.c  .
$ ls
aa  aa_l  a.out  chmod  chmod.c  ls  ls.c  stat.c
```

随后还使用 Vim 打开当前目录：

```text
$ vi .
```

再加入过滤 `.` 和 `..` 的 `strcmp` 判断后，由于尚未包含 `<string.h>`，出现隐式声明警告：

```text
$ gcc ls.c -o ls
ls.c: In function ‘walkDir’:
ls.c:21:14: warning: implicit declaration of function ‘strcmp’ [-Wimplicit-function-declaration]
    } else if (strcmp(".", di->d_name) == 0 || strcmp("..", di->d_name) == 0) {
              ^
$ ./ls
a.out  chmod.c  ls  ls.c  chmod  aa  aa_l  stat.c
$ ls
aa  aa_l  a.out  chmod  chmod.c  ls  ls.c  stat.c
$ ./ls /
home  media  root  boot  dev  cdrom  run  sbin  vmlinuz  usr  lib  tmp  snap  proc  etc
sys  var  srv  initrd.img  lost+found  bin  mnt  opt
$ ls /
bin    cdrom  etc   initrd.img  lost+found  mnt  proc  run   snap  sys  usr  vmlinuz
boot   dev    home  lib         media       opt  root  sbin  srv   tmp  var
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `gcc ls.c -o ls` | 编译简化版 `ls` 并命名为 `ls` | 早期版本先出现 `%d` 与字符串参数不匹配警告；加入 `strcmp` 后又出现缺少声明的警告，两个开发阶段均按示例保留 |
| 2 | `./ls` | 运行当前目录中的自编程序 | 先显示包含 `.`、`..` 的未过滤结果，后续版本显示过滤结果；返回顺序未排序 |
| 3 | `ls` | 运行系统 `ls` 作为现场对照 | 显示排序后的当前目录清单；具体文件集合为当时环境的实际值 |
| 4 | `vi .` | 用 Vim 打开当前目录 | 进入 Vim 目录浏览界面，不产生普通终端输出 |
| 5 | `./ls /` | 用自编程序列出根目录 | 显示未排序的根目录条目；集合和次序为当时环境的实际值 |
| 6 | `ls /` | 用系统 `ls` 列出根目录作对照 | 显示排版、排序后的根目录条目；集合为当时环境的实际值 |

两次输出对比说明：`readdir` 的返回顺序不是排序结果；真正的 `ls` 通常还要排序、排版、着色，并依据选项处理隐藏项。规范写法应包含 `<string.h>`，先声明函数再调用，用 `S_ISDIR(st.st_mode)` 判断目录，检查 `closedir` 返回值，并用预先清零的 `errno` 区分 `readdir` 的结束和错误；如需稳定顺序，收集后显式排序；`d_type` 可能为 `DT_UNKNOWN`，此时用 `lstat`/`fstatat` 复查。

#### 2.13.1 递归目录遍历（补充）

递归遍历是在单层 `readdir` 基础上，对发现的子目录再次执行同一过程。完整实现至少要处理：

1. 跳过 `.` 和 `..`，否则会立即形成递归环。
2. 正确拼接父路径和名称，不能只把 `d_name` 传给 `lstat`。
3. 使用 `lstat` 判断符号链接，并明确是否跟随。下面选择**不跟随符号链接**，避免链接环和越界访问其他目录树。
4. 检查 `opendir`、`readdir`、内存分配、`lstat` 和 `closedir` 的错误。
5. 限制递归深度；生产工具还应考虑文件系统边界、并发变化、权限、超长路径和输出排序。

下面的示例可编译运行，并把错误状态传播给 `main`：

| 自定义函数 | 参数 | 返回约定 |
|---|---|---|
| `print_indent(depth)` | `depth` 是当前递归层数 | 无返回值，只输出缩进 |
| `join_path(parent, name)` | 两个参数均为只读字符串 | 成功返回新分配路径，调用者负责 `free`；失败返回 `NULL` |
| `walk_tree(path, depth)` | `path` 是当前对象路径；`depth` 是当前层数 | 成功返回 0；本节点或任一后代失败后返回 `-1` |

`MAX_DEPTH` 是本示例定义的递归上限常量；`_DEFAULT_SOURCE` 是 glibc 功能测试宏，用于在严格语言模式下暴露所需接口声明，不是运行时开关。`SIZE_MAX` 是 `<stdint.h>` 提供的 `size_t` 最大值宏，路径长度相加前用它检查溢出；`ENAMETOOLONG` 是 `<errno.h>` 中表示路径或名称过长的错误宏。

```c
#define _DEFAULT_SOURCE
#include <dirent.h>
#include <errno.h>
#include <stdio.h>
#include <stdint.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>

enum { MAX_DEPTH = 64 };

static void print_indent(unsigned depth)
{
    for (unsigned i = 0; i < depth; ++i) {
        fputs("  ", stdout);
    }
}

static char *join_path(const char *parent, const char *name)
{
    size_t parent_len = strlen(parent);
    size_t name_len = strlen(name);
    int need_slash = parent_len != 0 && parent[parent_len - 1] != '/';
    size_t extra = (size_t)need_slash + 1; /* 可选斜杠和结尾 \0 */
    size_t size;
    if (parent_len > SIZE_MAX - name_len ||
        parent_len + name_len > SIZE_MAX - extra) {
        errno = ENAMETOOLONG;
        return NULL;
    }
    size = parent_len + name_len + extra;
    char *result = malloc(size);

    if (result == NULL) {
        return NULL;
    }
    (void)snprintf(result, size, "%s%s%s",
                   parent, need_slash ? "/" : "", name);
    return result;
}

static int walk_tree(const char *path, unsigned depth)
{
    struct stat st;
    DIR *dir;
    int status = 0;

    if (depth > MAX_DEPTH) {
        fprintf(stderr, "%s: maximum depth exceeded\n", path);
        return -1;
    }
    if (lstat(path, &st) == -1) {
        perror(path);
        return -1;
    }

    print_indent(depth);
    printf("%s%s\n", path, S_ISLNK(st.st_mode) ? " [symlink]" : "");

    if (!S_ISDIR(st.st_mode)) {
        return 0;
    }
    dir = opendir(path);
    if (dir == NULL) {
        perror(path);
        return -1;
    }

    for (;;) {
        struct dirent *entry;
        char *child;

        errno = 0;
        entry = readdir(dir);
        if (entry == NULL) {
            if (errno != 0) {
                perror("readdir");
                status = -1;
            }
            break;
        }
        if (strcmp(entry->d_name, ".") == 0 ||
            strcmp(entry->d_name, "..") == 0) {
            continue;
        }

        child = join_path(path, entry->d_name);
        if (child == NULL) {
            perror("join_path");
            status = -1;
            break;
        }
        if (walk_tree(child, depth + 1) == -1) {
            status = -1;
        }
        free(child);
    }

    if (closedir(dir) == -1) {
        perror("closedir");
        status = -1;
    }
    return status;
}

int main(int argc, char *argv[])
{
    const char *root = argc > 1 ? argv[1] : ".";

    if (argc > 2) {
        fprintf(stderr, "usage: %s [directory]\n", argv[0]);
        return EXIT_FAILURE;
    }
    return walk_tree(root, 0) == 0 ? EXIT_SUCCESS : EXIT_FAILURE;
}
```

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic tree_walk.c -o tree_walk
mkdir -p tree-lab/a/b tree-lab/c
touch tree-lab/root.txt tree-lab/a/a.txt
ln -s ../ tree-lab/a/back-to-root
./tree_walk tree-lab
```

符号链接 `back-to-root` 应只显示为链接而不继续进入，因此不会形成递归环。该程序用于理解目录遍历，不等同于成熟的 `find` 或 `tree`；目录在遍历期间可能变化，输出顺序也取决于文件系统返回顺序。

### 2.14 VFS：统一的文件系统接口

VFS（Virtual File System，也称 Virtual Filesystem Switch）是 Linux 内核中的文件系统抽象层。应用程序仍然调用统一的 `open`、`read`、`write`、`stat`、`chmod` 等接口；VFS 根据路径所在挂载点和对象类型，转到具体文件系统或设备实现。这样，ext 系列、FAT、NTFS 驱动、网络文件系统和伪文件系统可以同时出现在同一棵目录树中。

可以把第一张图中的关系转成下面的文字结构：

```text
用户空间：open / read / write
              │
              ▼
Linux 内核：  VFS（统一接口和对象模型）
              │
        ┌─────┼──────────┐
        ▼     ▼          ▼
   具体文件系统实现    其他文件系统实现
        │                 │
       NTFS            ext2/ext3        FAT32 ...
```

图中写有 `ntfs_open`、`ext3_open`，它们表达“不同实现响应同一类操作”的教学意图，不应据此假定每个文件系统都必须导出这些固定名字的函数。真实 Linux 内核通过已注册的操作表、路径查找和挂载关系完成分派，具体成员与实现会随内核版本变化。

![VFS 向不同文件系统分派统一接口](<assets/02_VFS统一接口与文件系统_4K.png>)

当一次 `open` 成功时，用户得到的是当前进程文件描述符表中的一个整数索引；该表项引用内核中的打开文件描述对象，Linux 内核实现中通常对应 `struct file`。适合初学者的稳定关系是：

```text
进程 files_struct
  └─ 文件描述符表
       ├─ fd 3 ─┐
       └─ fd 4 ─┴─> 打开文件描述（struct file）
                         ├─ 当前文件偏移 f_pos
                         ├─ 文件状态标志 f_flags
                         ├─ 文件操作表 f_op
                         ├─ 路径、dentry 与 inode 关联
                         └─ 引用计数
```

- `struct file` 表示一次打开形成的状态，不等于磁盘上的 inode，也不等于文件名。
- `f_op` 指向 `struct file_operations`。常见操作包括定位、读、写、迭代读写、打开初始化以及最后引用释放；不同文件系统或设备可提供不同实现。
- 路径查找通过 dentry 缓存把名称解析到 inode。inode 操作表负责查找、创建、链接、删除名称和修改属性等对象级操作。
- 图中的 `f_dentry`、`open/close/read/write` 操作表和“3G～4G 内核层”是简化或历史版本表示。现代内核常通过 `f_path` 关联 dentry，关闭过程对应 `flush`、`release` 等语义；3 GB/1 GB 也只适用于部分历史 32 位地址空间布局。
- 文件系统实现不必直接等同于“硬件驱动”。普通磁盘文件系统、网络文件系统、`procfs`、`sysfs` 和设备对象都可接入 VFS，但它们的后端和语义不同。

图中还能看到文件描述符 `3`、`4` 指向同一 `file`，`5` 指向另一个 `file`，以及 `write(3, ...)`、`chmod`、`link`、`unlink`、`open("abc") A`、`open("abc") B` 等标识。其关键不是编号本身，而是“描述符引用哪个打开文件描述”。

![文件描述符表、struct file 与文件操作表](<assets/01_VFS文件访问结构_4K.png>)

> 纠错：一个文件正在被进程打开，并不妨碍目录权限允许的其他进程执行 `unlink` 删除名称。打开文件描述仍引用原对象，进程可以继续访问；只有链接计数归零且最后一个打开引用也释放后，对象和空间才可最终回收。

### 2.15 打开文件描述与共享偏移

每次独立调用 `open` 通常创建新的打开文件描述，因此即使路径最终指向同一个 inode，也有各自的当前偏移和文件状态标志：

```text
open("abc", ...) -> fd_a -> file A（偏移 0） ─┐
                                                ├─> 同一 inode
open("abc", ...) -> fd_b -> file B（偏移 0） ─┘
```

如果 A 先从偏移 0 写入 `hello`，B 仍从自己的偏移 0 写入 `world`，后一次写入会覆盖相同范围，结果取决于写入长度、顺序、标志和并发情况，不能笼统保证为某个固定字符串。需要每次追加时应使用 `O_APPEND`，并继续处理写错误和并发协议。

`dup`、`dup2`、`fcntl(..., F_DUPFD, ...)` 或 `fork` 继承得到的描述符可以引用同一个打开文件描述：

```text
fd 3 ─┐
      ├─> 同一打开文件描述（共享偏移和文件状态标志） -> inode
fd 4 ─┘
```

若共享偏移最初为 0，先通过 fd 3 写入 `hello` 后偏移移到 5，再通过 fd 4 写入 ` world` 会从偏移 5 继续，形成 `hello world`。偏移是随成功读写或 `lseek` 更新的，不是“每次写都自动回到文件头”。

共享范围要区分清楚：

| 状态 | 同一打开文件描述的重复描述符 |
|---|---|
| 当前文件偏移 | 共享 |
| `O_APPEND`、`O_NONBLOCK` 等文件状态标志 | 共享 |
| `FD_CLOEXEC` 等文件描述符标志 | 各描述符独立 |
| 描述符整数值 | 可以不同 |

### 2.16 `dup`、`dup2` 与标准输出重定向

接口声明在 `<unistd.h>`：

```c
#include <unistd.h>

int dup(int oldfd);
int dup2(int oldfd, int newfd);
```

- `dup(oldfd)` 成功时返回当前进程最小的未使用描述符，新描述符引用与 `oldfd` 相同的打开文件描述。
- `dup2(oldfd, newfd)` 让 `newfd` 引用 `oldfd` 的打开文件描述；若二者不同且 `newfd` 已打开，关闭和替换由内核原子完成。
- 二者失败都返回 `-1` 并设置 `errno`。`dup1` 不是该接口族中的函数名，正确名称是 `dup`。
- `dup2(oldfd, oldfd)` 在 `oldfd` 有效时不做修改并返回该值。Linux 还提供 `dup3`，可原子设置 `O_CLOEXEC`。

示例程序 `dup.c` 仍调用 `printf`，但用 `dup` 保存原标准输出，再用 `dup2` 把标准输出改指向文件 `aa`：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main()
{
    int fd_aa = open("aa", O_RDWR);
    if (-1 == fd_aa) {
        perror("open aa fail");
        return 1;
    }

    int fd_save = dup(STDOUT_FILENO);
    dup2(fd_aa, STDOUT_FILENO);

    printf("this is a test\n");
    close(fd_aa);
    close(fd_save);
    return 0;
}
```

在存有 `aa` 的练习目录中编译、运行：

```text
$ cd ~/linux-command-lab/session-05/filesys/
$ ls -l
-rw-r--r-- 1 ubuntu ubuntu   20 8月 10 09:22 aa
lrwxrwxrwx 1 ubuntu ubuntu    2 8月 10 09:22 aa_l -> aa
-rwxrwxr-x 1 ubuntu ubuntu 7584 8月 10 09:29 a.out
-rwxrwxr-x 1 ubuntu ubuntu 7512 8月 10 09:46 chmod
-rw-rw-r-- 1 ubuntu ubuntu  718 8月 10 09:46 chmod.c
-rwxrwxr-x 1 ubuntu ubuntu 7784 8月 10 11:10 ls
-rw-rw-r-- 1 ubuntu ubuntu 1379 8月 10 11:09 ls.c
-rw-rw-r-- 1 ubuntu ubuntu  390 8月 10 09:28 stat.c
$ cat aa
aaaaaaaaaaaaaaaaaaaa
$ vi dup.c
$ gcc dup.c
$ ./a.
bash: ./a.: 没有那个文件或目录
$ ./a.out
$ cat aa
this is a test
aaaaa
```

| 步骤 | 命令 | 命令作用 | 输出或状态变化 |
|---:|---|---|---|
| 1 | `cd ~/linux-command-lab/session-05/filesys/` | 进入文件系统实验目录 | 成功时无输出，终端路径进入 `filesys` |
| 2 | `ls -l` | 查看实验目录的文件、权限和大小 | 显示示例运行时的目录清单；用户名、时间和二进制大小均为现场值 |
| 3 | `cat aa` | 在运行程序前显示 `aa` | 首次输出 20 个 `a` |
| 4 | `vi dup.c` | 打开或创建重定向示例源码 | 进入 Vim 编辑界面，不产生普通终端输出 |
| 5 | `gcc dup.c` | 编译源码，使用默认输出名 | 编译成功时无输出，生成 `a.out` |
| 6 | `./a.` | 误把可执行文件名输成 `./a.` | 输出 `bash: ./a.: 没有那个文件或目录`；错误命令按示例保留 |
| 7 | `./a.out` | 运行标准输出重定向示例 | 终端无普通输出，字符串被写入 `aa` |
| 8 | `cat aa` | 显示运行后的 `aa` | 输出分属两行的 `this is a test` 与 `aaaaa` |

其中 `./a.` 是终端误输，可执行文件是 `./a.out`。运行后 `printf` 的输出经重定向进入 `aa`；由于用 `O_RDWR` 打开且未截断，`this is a test\n` 共写入 15 字节（换行占 1 字节），原有 20 字节内容的末尾仍保留 5 个 `a`。终端中的 `this is a test` 与 `aaaaa` 分属两行，不能合并。

该示例有几处需要纠正：`dup`、`dup2` 和 `printf` 的返回值都没有检查；`printf` 属于带缓冲的标准 I/O，重定向前后不刷新可能使缓冲数据写到意外目标，严谨写法应在切换描述符前和写入后调用并检查 `fflush`；程序结束前没有把标准输出恢复到 `fd_save`，会影响同一进程后续输出；打开 `aa` 时未用 `O_TRUNC`，旧内容会残留在新写入文本之后，若想清空应使用 `O_TRUNC`，想追加则明确改用 `O_APPEND`。这些更改只影响当前进程的描述符表，不会把整个系统或启动它的 Shell 永久重定向。

Shell 的 `>`、`2>`、管道以及日志程序都可利用相同的“在执行目标程序前布置描述符”思想，但真实 Shell 还要处理进程创建、管道端点、并发、关闭继承描述符和错误传播。

### 2.17 本章复现与检查清单（补充）

1. 用 `stat`、`df -h`、`df -i`、`du -sh` 区分逻辑长度、分配空间、块和 inode 使用量。
2. 画出超级块、GDT、两个位图、inode 表和数据块的关系，并说明上述 4 KiB 经典模型的数值假设。
3. 计算 4 KiB 块下直接、一级、二级和三级间接部分的理论数据量。
4. 用 `stat`、`stat -L`、`ls -li` 核对硬链接和符号链接的 inode 与链接数。
5. 删除原名称，验证硬链接仍可读、符号链接悬空；重建同名目标后再次验证。
6. 用只读描述符验证最后一个名称删除后对象仍可访问，关闭描述符后结束实验。
7. 用 `stat`/`lstat` 对比符号链接目标与链接本身，并用与 `ino_t`、`off_t` 匹配的类型和格式打印。
8. 用 `access` 观察存在性和权限，但让实际操作独立检查返回值；复现 `chmod` 示例程序的 `./chmod`、`./chmod 0642 xx`、`./chmod 0642 aa`，并核对 `ls -l` 权限位。
9. 观察 `utime`、`truncate`、`rename`、`chdir`、`getcwd`、`pathconf` 的成功与错误分支。
10. 用目录流程序列出普通文件、目录和多路径，确认 `readdir` 原始顺序不等于 `ls` 排序结果。
11. 画出“描述符表—打开文件描述—dentry/inode—具体文件系统操作”的 VFS 关系，不把文件描述符、`struct file` 和 inode 混为同一对象。
12. 分别用两次 `open` 和 `dup` 打开同一练习文件，观察独立偏移与共享偏移的差别。
13. 编译运行 `dup.c`，观察 `aa` 收到重定向文本并保留旧内容残留，解释严谨写法为什么要检查 `dup`/`dup2` 返回值、刷新标准 I/O 缓冲并恢复标准输出。
14. 所有实验只在 `~/linux-command-lab` 下的新目录执行，不在系统目录、存放文档的目录或真实数据目录中练习删除。


