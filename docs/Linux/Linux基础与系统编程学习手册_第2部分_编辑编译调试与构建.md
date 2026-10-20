<a id="part-2"></a>
# Linux 基础与系统编程学习手册 · 第 2 部分：编辑、编译、调试与构建

> 本册为《Linux 基础与系统编程学习手册》五册套装中的第 2 册（共 5 册），覆盖第 1—4 章。

## 分册导航

- [第 1 部分：Linux 命令与日常运维](Linux基础与系统编程学习手册_第1部分_Linux命令与日常运维.md)（第 1—11 章）
- **第 2 部分：编辑、编译、调试与构建（本册，第 1—4 章）**
- [第 3 部分：Linux 系统编程](Linux基础与系统编程学习手册_第3部分_Linux系统编程.md)（第 1—2 章）
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

<a id="chapter-12"></a>
## 第 1 章 Vi/Vim 文本编辑

### 1.1 Vi、Vim 与三种工作模式

Vi/Vim 是面向纯文本的编辑器，可以输入、删除、查找、替换和移动文本，也能通过配置扩展功能。它不是 Word、WPS 或 PowerPoint 一类排版软件；源代码、配置文件和日志等纯文本才是它的主要使用场景。它尤其适合只有终端的远程 Linux 环境，也能较高效地浏览较大的文本文件。Vim 是 Vi 的增强实现，许多系统中的 `vi` 命令也可能实际启动 Vim 或兼容实现。

```bash
vi 文件名
vim 文件名
```

- 文件已经存在：打开并编辑原文件。
- 文件不存在：进入一个新缓冲区；只有保存后才会真正创建文件。
- 如果本来想打开已有文件却看到空白缓冲区，应先检查路径、大小写和文件名。不要急着保存，否则可能因拼写错误创建一个多余文件。

Vi/Vim 的三个基本模式如下：

| 模式 | 作用 | 常见进入方式 |
|---|---|---|
| 普通模式（Normal mode） | 移动、删除、复制、粘贴、撤销等 | 启动编辑器时默认进入；其他模式按 `Esc` 返回 |
| 插入模式（文本输入模式） | 输入或修改文件内容 | 在普通模式按 `i`、`a`、`o` 等 |
| 命令行模式（Command-line mode，也常称末行模式） | 保存、退出、替换、分屏、执行外部命令等 | 在普通模式输入 `:` |

![Vi/Vim 三种基本模式及 i、a、o、冒号与 Esc 的切换方向](<assets/vim-basic-modes.png>)

Vim 的命令行模式位于编辑器底部，不是 Bash 终端。普通模式与命令行模式容易混淆，本文统一使用“普通模式”和“命令行模式”这两个名称。

### 1.2 配置 `~/.vimrc` 与显示行号

Vim 常见配置文件有：

- `/etc/vim/vimrc`：系统级配置，影响范围较大，通常需要管理员权限。
- `~/.vimrc`：当前用户配置，适合个人设置，通常会覆盖相冲突的系统级设置。

准备好 `vim.tar.gz` 后，可先把它放入当前用户家目录。对未知压缩包，先查看内容，再解压：

```bash
cd ~
tar tzvf vim.tar.gz
tar zxvf vim.tar.gz -C ~/
ls -la
vim ~/.vimrc
```

在 `~/.vimrc` 中加入以下任一写法可永久显示行号：

```vim
set number
```

`set nu` 是 `set number` 的缩写。临时开关可在 Vim 命令行模式执行：

```vim
:set number
:set nonumber
```

编辑配置前可先复制备份；解压前先检查文件列表，避免不可信归档覆盖家目录中的现有配置。配置文件的具体路径可能随发行版和 Vim 打包方式不同，可在 Vim 中用 `:version` 或 `:scriptnames` 辅助确认实际加载情况。

#### 1.2.1 Vim 配置归档的评估与采用（补充）

包含配置、插件和模板的 Vim 归档不是单一配置文件，解包内容可按用途分为：

| 分类 | 包内路径或代表文件 | 本手册处理 |
|---|---|---|
| 主配置 | `.vimrc` | 作为配置参考；解包前先备份现有 `~/.vimrc`，不直接覆盖 |
| 配色 | `.vim/colors/navajo-night.vim`、`vc.vim` | 可选视觉资源，不属于必须掌握的 Vim 命令 |
| 插件 | `srcexpl.vim`、`trinity.vim`、`c.vim`、`NERD_tree.vim`、`taglist.vim`、`supertab.vim` | 仅登记用途类别；版本较旧，启用前逐个检查兼容性和来源 |
| 文件类型配置 | `.vim/ftplugin/c.vim` | C 文件专用设置，可能与全局配置或其他插件冲突 |
| C/C++ 支持资源 | `.vim/c-support/rc/`、`wordlists/`、`scripts/`、`templates/` | 定制、词表、脚本和代码模板资源，按需选用 |
| 代码片段 | `.vim/c-support/codesnippets/` 中的 `main.c`、`main.cc`、Makefile 和数组示例 | 只作参考素材；使用前阅读源码、编译并核对，不直接复制到真实项目 |
| 文档 | `.vim/doc/`、`.vim/c-support/doc/`、`README.csupport` | 插件帮助和快捷键文档；安装后可用 `:helptags ~/.vim/doc` 生成帮助索引 |
| 状态文件 | `.VimballRecord`、`.netrwhist`、`.vim/doc/tags` | 安装/历史/索引状态，不作为教学内容；可由工具重新生成 |

归档中的可复用技术素材也已逐文件检查：

- `main.c`、`main.cc` 是带说明头的程序入口模板；它们一次性包含了 `errno.h`、`math.h`、`string.h` 等头文件，实际项目应按使用情况删减，而不是把模板头文件列表当成固定要求。
- `calloc_int_matrix.c`、`calloc_double_matrix.c` 用“一块行指针数组加一块连续数据区”表示动态二维数组；释放顺序是先释放 `*m` 指向的数据区，再释放 `m`。片段调用了 `assert`，嵌入独立文件时必须补齐 `<assert.h>`，并在真实程序中设计可恢复的分配失败路径，不能只依赖断言。
- 三个 `print_*_array` 片段按每行元素数格式化输出一维数组；调用者必须保证 `nrow > 0`，否则其中的取模运算无效。
- `Makefile` 与 `Makefile.multi-target.template` 演示单目标、多目标、编译选项和清理规则；它们是旧版插件的通用模板，不替代本章基于实际 `project/` 依赖关系编写的 Makefile。
- `wrapper.sh` 检查目标是否可执行，运行 `"${@}"`、保留退出状态并等待回车；它适合旧式 `xterm` 插件工作流。脚本中的参数仍来自调用者，使用时不要把不可信文本拼成命令，也不要为了运行它而放宽文件权限。
- `templates/`、`wordlists/`、`rc/` 和帮助文档分别提供代码模板、补全词表、定制配置和插件说明；属于按需查阅的编辑器资源，不是 Linux 或 C 语言本身的规范。

安全检查流程：

```bash
tar tzf network_4_03_vim.tar.gz
mkdir -p ~/vim-package-review
tar xzf network_4_03_vim.tar.gz -C ~/vim-package-review
find ~/vim-package-review -maxdepth 4 -type f -print
```

先解压到审查目录，阅读 `.vimrc`、插件和 Shell 脚本，再决定是否复制单个文件。Vim 配置和插件脚本都能执行命令；不要因为归档扩展名是 `.tar.gz` 就把它视为纯数据。对现代 Vim/Neovim 环境，更适合从仍维护的上游重新安装所需插件，并记录版本，而不是整体覆盖家目录。

### 1.3 输入、保存和退出

普通模式下的常用插入命令：

| 按键 | 作用 |
|---|---|
| `i` | 在光标前进入插入模式 |
| `I` | 在当前行第一个非空白字符前插入 |
| `a` | 在光标后进入插入模式 |
| `A` | 在行尾追加 |
| `o` | 在当前行下方新开一行并插入 |
| `O` | 在当前行上方新开一行并插入 |
| `Esc` | 从插入模式或命令行模式回到普通模式 |

六个插入命令的区别可以用同一行原文看清。约定缓冲区中只有一行 `hello world`，光标停在 `w` 上；`XY` 表示本次输入的文字，出现在结果中的位置就是插入点：

| 命令 | 插入点 | 结果 |
|---|---|---|
| `i` | 光标所在字符之前 | `hello XYworld` |
| `I` | 本行第一个非空白字符之前 | `XYhello world` |
| `a` | 光标所在字符之后 | `hello wXYorld` |
| `A` | 本行行尾 | `hello worldXY` |
| `o` | 在下方新建一行并插入 | `XY` 在 `hello world` 的下一行 |
| `O` | 在上方新建一行并插入 | `XY` 在 `hello world` 的上一行 |

三组命令最容易混淆：

- `i` 与 `a` 只差一个字符的位置：`i` 在光标前、`a` 在光标后。要在行尾追加内容，直接用 `A`，不必先按 `l` 把光标移到最后一个字符再 `i`。
- `I` 的落点是本行第一个非空白字符，不是第 0 列。对缩进行（如 `    int x;`）按 `I` 插入，文字会出现在缩进之后，正好符合写代码时在行首对齐位置输入的习惯。
- `o` 与 `O` 是六个命令中唯一新建行的两个，其余四个都在当前行内插入。

可以在 1.3 节末尾的 `note.txt` 上逐个验证：打开文件后用 `gg` 回到第一行，依次按 `i`、`a`、`I`、`A`、`o`、`O` 并输入几个字符，每次按 `Esc` 后用 `u` 撤销，再试下一个命令，观察插入点是否与上表一致。

普通模式输入 `:` 后，可执行：

| 命令 | 作用 |
|---|---|
| `:w` | 保存但不退出 |
| `:q` | 未修改文件时退出；有未保存修改时会拒绝退出 |
| `:wq` | 保存并退出 |
| `:x` | 有修改时保存并退出，无修改时直接退出 |
| `:q!` | 丢弃本次未保存修改并强制退出 |

`w` 来自 write，`q` 来自 quit。`q!` 会丢弃未保存内容：新缓冲区未保存时文件不会创建，已有文件则保留磁盘上的旧内容。重要修改应优先 `:w`，不要把 `:q!` 当作普通退出方式。

一个最小练习：

```bash
mkdir -p ~/linux-command-lab/session-03
cd ~/linux-command-lab/session-03
vim note.txt
```

进入后按 `i`，输入两行文字，按 `Esc`，输入 `:wq` 回车；重新执行 `vim note.txt` 应能看到刚才保存的内容。

### 1.4 移动、删除、撤销和缩进

以下按键都在普通模式使用。

**移动光标**：

| 按键 | 作用 |
|---|---|
| `h` / `j` / `k` / `l` | 左 / 下 / 上 / 右移动 |
| `w` / `b` | 向后 / 向前按单词移动 |
| `{` / `}` | 移到上一个 / 下一个段落 |
| `M` / `L` | 移到屏幕中间行 / 最后一行 |
| `Ctrl-d` / `Ctrl-u` | 向下 / 向上翻半屏 |
| `Ctrl-f` / `Ctrl-b` | 向下 / 向上翻一屏 |
| `gg` / `G` | 移到文件开头 / 文件末尾 |
| `行号G` | 移到指定行，例如 `25G` |

**删除文本**：

| 按键 | 作用 |
|---|---|
| `x` / `X` | 删除光标处字符 / 光标前字符 |
| `dd` | 删除当前行 |
| `数量dd` | 从当前行起向下删除指定行数，例如 `3dd` |
| `D` | 删除光标处到当前行末尾的内容 |
| `d0` | 删除行首到光标前的内容 |
| `dw` | 删除从光标开始的一个单词范围 |

删除多行时，将光标停在第一行再输入 `3dd`。插入模式中的 Backspace 更适合少量字符修正；批量操作优先使用普通模式命令。

**撤销、重复和缩进**：

| 按键 | 作用 |
|---|---|
| `u` | 撤销一次改动，可加数量 |
| `U` | 撤销最近被修改的那一行上的最新改动 |
| `Ctrl-r` | 重做已经撤销的改动 |
| `.` | 重复上一次修改操作 |
| `>>` / `<<` | 当前行右移 / 左移一个缩进层级 |
| `gg=G` | 从文件开头到末尾重新计算缩进 |

大写 `U` 不是“撤销打开文件后的全部操作”；它针对最近被修改的当前行，并且 `U` 自身也会成为一次可撤销的改动。`gg=G` 主要根据 Vim 的缩进规则整理代码缩进，不等于 Word 式排版，也不保证理解所有语言格式规范。

### 1.5 复制、可视选择、查找和替换

**复制与粘贴**：

| 按键 | 作用 |
|---|---|
| `yy` | 复制当前行 |
| `数量yy` | 从当前行起复制多行，例如 `4yy` |
| `p` | 在光标之后粘贴；按行复制时粘到当前行下方 |
| `P` | 在光标之前粘贴；按行复制时粘到当前行上方 |

例如要把第 2—5 行复制到第 10 行之后，可在第 2 行输入 `4yy`，再把光标移到第 10 行输入 `p`。不同分屏窗口共享寄存器时，也可在一个窗口复制、切换到另一个窗口粘贴。

Vim 并非不能选中一段文本。普通模式按 `v` 进入字符可视模式，按 `V` 进入行可视模式；移动光标选中后可配合：

- `d`：删除选区。
- `y`：复制选区。
- `>` / `<`：增加 / 减少选区缩进。

**替换与查找**：

| 按键或命令 | 作用 |
|---|---|
| `r字符` | 用一个字符替换光标处字符 |
| `R` | 进入替换模式，连续覆盖后续字符 |
| `/字符串` | 从当前位置向后查找 |
| `n` / `N` | 沿相同方向 / 相反方向跳到下一个匹配 |
| `:%s/abc/123/g` | 全文件把 `abc` 替换为 `123` |
| `:%s/abc/123/gc` | 全文件逐个确认替换 |

查找到文件末尾后是否从头继续取决于 `wrapscan` 设置，不能绝对认为一定循环。批量替换前先保存或备份；模式中包含 `/`、`.`、`*` 等特殊字符时需要了解转义和正则表达式规则。

**开发辅助操作**：

- 光标位于函数名时按 `K`：调用关键字帮助程序，常用于查看 Man Page；`3K` 可指定第 3 节。具体行为受 `keywordprg` 配置影响。
- `[d`：显示光标处宏定义的第一处匹配；通常要求相关头文件已被包含，并且 Vim 能找到它。
- `:!命令`：临时执行外部 Shell 命令，例如 `:!gcc --version`。

### 1.6 分屏操作

分屏适合同时查看多个源文件；小屏幕中过多分屏会压缩编辑区域，应按显示器大小控制窗口数量。

在 Vim 命令行模式中：

```vim
:sp other.c
:vsp other.c
```

- `:sp [文件名]`：上下分屏；不带文件名时显示当前缓冲区。
- `:vsp [文件名]`：左右分屏。
- 文件名存在时打开原文件，不存在时创建新缓冲区，仍需保存才会生成文件。

启动 Vim 时也可直接分屏：

```bash
vim -O main.c add.c       # 左右排列
vim -o main.c add.c       # 上下排列
vim -O3 main.c add.c      # 指定三个窗口；多余窗口为空
```

常用窗口命令都以 `Ctrl-w` 为前缀：

| 命令 | 作用 |
|---|---|
| `Ctrl-w w` | 切换到下一个窗口 |
| `Ctrl-w h/j/k/l` | 切换到左 / 下 / 上 / 右窗口 |
| `Ctrl-w c` | 关闭当前窗口 |
| `Ctrl-w q` | 退出当前窗口；最后一个窗口退出 Vim |
| `Ctrl-w s` / `Ctrl-w v` | 上下 / 左右分割当前窗口 |
| `Ctrl-w H/J/K/L` | 把当前窗口移到最左 / 最下 / 最上 / 最右 |
| `Ctrl-w +` / `Ctrl-w -` | 增加 / 减少当前窗口高度 |
| `Ctrl-w >` / `Ctrl-w <` | 增加 / 减少当前窗口宽度 |
| `Ctrl-w =` | 尽量平均分配窗口尺寸 |

数量可放在窗口命令前，例如 `30 Ctrl-w <` 将当前窗口宽度减少 30 列。光标在哪个窗口，编辑、保存、退出等命令就作用于哪个窗口；退出一个窗口不会自动保存其中的未保存修改。

<a id="chapter-13"></a>
## 第 2 章 GCC 编译

### 2.1 从 C 源文件生成可执行文件

GCC 驱动程序可以调用预处理器、编译器、汇编器和链接器，把一个或多个源文件生成目标文件或可执行文件。本节示例使用 `.c` 文件；C++ 源文件通常使用 `g++`，以便自动采用 C++ 链接规则。

![GCC 驱动的四阶段编译流程、停止选项与多文件链接](<assets/09_GCC编译流程_4K.png>)

**基本语法**：

```bash
gcc [选项...] 源文件... -o 输出文件
```

方括号 `[]` 和省略号 `...` 是命令行语法的惯用记法，不是需要输入的字符：方括号表示可选，省略号表示可以重复出现。三个部分的含义如下；对照 `gcc -Wall hello.c -o hello` 这条命令，`-Wall` 是选项，`hello.c` 是源文件，`hello` 是输出文件：

- `[选项...]`：以 `-` 开头的参数，控制编译行为，如 `-Wall`、`-std=c99`。可以一个不写（如 `gcc hello.c`），也可以写多个；顺序一般不影响结果。
- `源文件...`：要编译的 `.c` 文件，可以写一个，也可以写多个参与同一链接。
- `-o 输出文件`：指定编译产物的名字。`-o` 本身也是一个选项，紧随其后的参数是输出文件名；若省略，默认输出 `a.out`。

**源文件路径怎么写**：源文件可以带路径，GCC 按给出的路径逐个寻找，不要求都在当前目录：

```bash
gcc hello.c -o hello                                     # 同目录：直接写文件名
gcc src/hello.c -o src/hello                             # 相对路径：相对当前工作目录
gcc /home/ubuntu/lab/src/hello.c -o /home/ubuntu/lab/hello # 绝对路径
gcc src/main.c lib/add.c lib/sub.c -o app                # 多个源文件可分在不同目录
```

单个源文件把路径写清即可；多个源文件只要每个都带正确路径，GCC 就能分别找到并一起链接。相对路径以当前工作目录为基准，编译前可先用 `pwd` 确认位置；头文件与源文件不在同目录时，另用 `-I` 指出头文件目录（如 `-I../include`），这与源文件路径是两回事。

**`-o` 输出的都是可执行文件吗**：不一定是，取决于是否执行了链接。`gcc hello.c -o hello` 从源文件一步走到链接，输出的 `hello` 是可直接运行的可执行文件；若加了 `-c` 只编译不链接，`gcc -c add.c -o add.o` 输出的则是 `.o` 目标文件，不能直接运行，需再与其他目标文件链接成可执行文件（见 2.3 节分步编译与 2.5 节工具链）。

**动手编译第一个程序**：

```c
#include <stdio.h>

int main(void)
{
    puts("hello, gcc");
    return 0;
}
```

保存为 `hello.c` 后编译并运行，终端应输出 `hello, gcc`：

```bash
gcc hello.c -o hello
./hello
```

如果链接可执行文件时省略 `-o`，Unix/Linux 上 GCC 默认输出 `a.out`：

```bash
gcc hello.c
./a.out
```

**`a.out` 是什么**：它是链接器产出的**可执行文件**，与 `-o hello` 生成的 `hello` 内容性质相同，名字只是 GCC 的默认约定——源自早期 Unix 的 assembler output 缩写，与文件格式没有必然关系（现代 Linux 上它通常是 ELF 格式）。两点值得注意：

- 它不是中间产物，不是待链接的 `.o` 目标文件，而是可直接运行的最终产物；重名风险在于同目录下再编译另一个不带 `-o` 的程序会直接覆盖它。
- 它不带扩展名也不影响执行：Linux 判断能否运行靠文件内容与执行权限（`file a.out` 可验证），Windows 那种 `.exe` 后缀约定在这里不适用。

`./` 表示从当前目录寻找程序，因为当前目录通常不在 `PATH` 中。是否能运行还取决于编译是否成功、文件权限、目标平台和运行时依赖。

### 2.2 版本、语言和多文件编译

```bash
gcc --version
gcc -v
```

- `--version`：简洁显示 GCC 版本信息。
- `-v`：除版本外，还会显示配置和 GCC 调用的编译阶段命令；输出更详细。

版本会影响支持的语言标准、目标平台和选项。遇到新语法被报告为错误时，先确认文件扩展名、编译语言、`-std=` 选项和编译器版本，不要只凭版本号猜测。

多文件项目必须让 GCC 看到所有参与链接的源文件或目标文件。例如：

```text
calculator/
├── include/common.h
└── src/main.c add.c sub.c mul.c div.c
```

在 `src` 目录编译：

```bash
cd calculator/src
gcc -Wall -I../include main.c add.c sub.c mul.c div.c -o calculator
./calculator
```

`#include "common.h"` 已指定头文件名，`-I` 只需给出额外搜索目录。GCC 接受紧邻或分开的参数形式：

```bash
gcc -I../include main.c add.c -o calculator
gcc -I ../include main.c add.c -o calculator
```

若报 `common.h: No such file or directory`，先用 `pwd`、`ls ../include` 核对当前目录和相对路径。头文件能找到不代表实现函数一定参与链接；遗漏 `add.c` 或 `add.o` 还可能出现 undefined reference 链接错误。

### 2.3 GCC 常用选项

| 选项 | 作用 | 示例 |
|---|---|---|
| `-v` / `--version` | 查看详细配置与阶段信息 / 查看版本 | `gcc --version` |
| `-I目录` | 增加头文件搜索目录 | `gcc -I../include main.c` |
| `-c` | 只编译/汇编为 `.o`，不链接 | `gcc -c add.c` |
| `-g` | 生成供调试器使用的调试信息 | `gcc -g main.c -o app` |
| `-O0`—`-O3` | 从不优化到更强优化 | `gcc -O2 main.c -o app` |
| `-Wall` | 启用一组常用警告 | `gcc -Wall main.c -o app` |
| `-D名称[=值]` | 在编译时定义预处理宏 | `gcc -DDEBUG main.c -o app` |
| `-E` | 只做预处理，结果默认输出到终端 | `gcc -E main.c -o main.i` |
| `-M` | 输出源文件的依赖关系，包含系统头文件 | `gcc -M main.c` |
| `-MM` | 输出依赖关系，不包含系统头文件 | `gcc -MM main.c` |

`-On` 是泛指优化级别的占位写法，实际应使用 `-O0`、`-O1`、`-O2`、`-O3` 等具体级别；`n` 越大通常优化更积极，但不表示任何程序都会更快，也可能增加编译时间或改变调试体验。调试初学示例通常使用 `-O0`，也可了解为调试体验设计的 `-Og`。

`-Wall` 并不等于“打开所有警告”，只是启用一组常见且有价值的警告。项目常进一步使用 `-Wextra`，甚至把警告视为错误。出现警告时应理解原因并修改代码或经过评审做有针对性的处理，不应为了“清零”而删掉编译脚本中的警告选项。

`-I` 后面跟的是**目录**，不是头文件名字，也不需要把用到的头文件逐个列出。遇到 `#include "common.h"` 时，编译器会先在源文件所在目录找，再依次搜索 `-I` 指定的目录和系统默认目录，找到即止；只要把存放头文件的整个目录交给 `-I`（如 `-I../include`），其中的所有头文件都自动可被搜到。若目录写错或文件名大小写不符，报错通常是 `common.h: No such file or directory`。

分步编译示例（先各自编译为 `.o` 目标文件，再统一链接）：

```bash
gcc -Wall -I../include -c main.c -o main.o    # -c：只编译不链接；-o：指定目标文件名；-I：为预处理阶段提供头文件目录
gcc -Wall -I../include -c add.c -o add.o      # -c：只编译不链接；-o：指定目标文件名；-I：为预处理阶段提供头文件目录
gcc main.o add.o -o calculator                # 链接：不再写 -c，把两个 .o 连同启动代码、库拼成可执行文件
```

大型项目通常由 Make、CMake 或其他构建系统只重建发生变化的模块，不会每次手工输入成百上千个源文件。`-M`、`-MM` 生成的依赖信息可用于 Makefile；它们不是普通编译输出。

### 2.4 条件编译、不同目标与日志安全

`-D名称[=值]` 可以让同一套源码按编译参数选择不同分支：

```c
#include <stdio.h>

int main(void)
{
    int value = 10;
#ifdef FEATURE_X
    value += 100;
#else
    value += 200;
#endif
    printf("value=%d\n", value);
    return 0;
}
```

```bash
gcc feature.c -o app-default          # 不定义 FEATURE_X：走 #else 分支，value = 10 + 200
gcc -DFEATURE_X feature.c -o app-feature  # 定义 FEATURE_X：走 #ifdef 分支，value = 10 + 100
./app-default                        # 输出 value=210
./app-feature                        # 输出 value=110
```

GCC 同样接受 `-D FEATURE_X`，但紧邻写法更常见。条件编译适用于调试功能、产品变体、不同 CPU 或硬件平台的少量差异；公共代码保持一套，不同部分放在明确的宏分支中。宏过多会让可测试组合迅速增加，应由构建配置和自动化测试统一管理。

调试日志可放在宏控制的分支中：

```c
#ifdef DEBUG_LOG
fprintf(stderr, "state=%d\n", state);
#endif
```

```bash
gcc -DDEBUG_LOG -Wall app.c -o app-debug    # 定义 DEBUG_LOG：编译进 fprintf 日志，用于排查问题
gcc -Wall app.c -o app-release              # 不定义 DEBUG_LOG：日志分支在预处理阶段被删除，产物更小
```

发布版本不定义 `DEBUG_LOG` 时，上述语句不会进入编译结果。不要在发布前手工删除一批日志语句来维护另一套代码，这容易误删相邻逻辑并造成调试版、发布版长期分叉。日志量过大还会占用存储空间、增加 I/O 并干扰问题定位。

条件编译不能替代安全审查：任何版本的日志都不应输出密码、token、银行卡信息等敏感数据；还应检查第三方库、崩溃转储、配置文件和发布包中是否残留敏感信息。

在 Windows 上可以编辑 Linux 源码，也能通过交叉编译器生成 Linux 目标程序；但普通 Windows 编译器生成的原生 `.exe` 不能直接作为 Linux 可执行文件运行。应在目标 Linux 环境或使用目标兼容工具链编译、链接和测试。

### 2.5 GNU 工具链与 binutils（补充）

`gcc` 是编译驱动程序，它会按需要调用预处理器、编译器、汇编器和链接器。GNU binutils 则提供处理目标文件、库和可执行文件的一组工具。先建立一个隔离实验目录：

```bash
mkdir -p ~/linux-command-lab/toolchain
cd ~/linux-command-lab/toolchain
printf '%s\n' '#include <stdio.h>' 'int main(void){puts("hello");return 0;}' > hello.c
```

观察四个主要阶段：

```bash
gcc -E hello.c -o hello.i       # 预处理
gcc -S hello.i -o hello.s       # 编译为汇编文本
gcc -c hello.s -o hello.o       # 汇编为可重定位目标文件
gcc hello.o -o hello            # 链接为可执行文件
./hello
```

常用工具如下：

| 工具 | 主要用途 | 只读检查示例 |
|---|---|---|
| `file` | 判断文件类型和目标架构 | `file hello.o hello` |
| `nm` | 查看符号及其类型 | `nm -C hello.o` |
| `objdump` | 查看文件头、节、符号或反汇编 | `objdump -h hello.o`、`objdump -d hello` |
| `readelf` | 读取 ELF 文件头、节、段、符号和动态信息 | `readelf -h hello`、`readelf -S hello.o` |
| `size` | 汇总代码、数据和 BSS 大小 | `size hello.o hello` |
| `strings` | 提取可打印字符串，用于初步检查 | `strings hello \| less` |
| `ar` | 创建、更新和查看静态归档库 | `ar t libcalc.a` |
| `ranlib` | 为归档库生成/更新符号索引；GNU `ar -s` 也可完成 | `ranlib libcalc.a` |
| `objcopy` | 在目标文件格式间复制或增删节 | `objcopy --only-keep-debug hello hello.debug` |
| `strip` | 删除部分符号和调试信息以减小发布文件 | `strip --strip-unneeded hello.release` |

`strip` 和某些 `objcopy` 操作会修改文件。练习时先复制：

```bash
cp hello hello.release
strip --strip-unneeded hello.release
file hello hello.release
size hello hello.release
```

不要对唯一的调试构建直接执行 `strip`。发布程序需要可诊断性时，可把调试符号单独保存并按构建 ID 或版本归档。`strings` 只能显示可打印片段，不能证明程序安全，也不能替代源码审计。

### 2.6 静态库的创建与链接（补充）

静态库通常以 `.a` 结尾，本质上是多个 `.o` 文件组成的归档。建立最小示例：

```c
/* calc.h */
#ifndef CALC_H
#define CALC_H
int add(int a, int b);
int sub(int a, int b);
#endif
```

```c
/* add.c */
#include "calc.h"
int add(int a, int b) { return a + b; }
```

```c
/* sub.c */
#include "calc.h"
int sub(int a, int b) { return a - b; }
```

```c
/* main.c */
#include <stdio.h>
#include "calc.h"
int main(void)
{
    printf("%d %d\n", add(7, 3), sub(7, 3));
    return 0;
}
```

编译、建库并链接：

```bash
gcc -Wall -Wextra -c add.c sub.c
ar rcs libcalc.a add.o sub.o
ar t libcalc.a
nm -C libcalc.a
gcc -Wall -Wextra main.c -L. -lcalc -o calc-static
./calc-static
```

`-L.` 把当前目录加入本次链接的库搜索路径；`-lcalc` 寻找 `libcalc.so` 或 `libcalc.a`。对传统 Unix 链接器，库通常应写在使用其符号的目标文件之后，因此 `main.c -lcalc` 比 `-lcalc main.c` 更可靠。静态链接只是把需要的库成员复制进最终程序，不代表整个可执行文件一定不再依赖任何共享库；可用 `readelf -d` 或 `ldd` 检查其他依赖。

### 2.7 共享库、动态装载与排障（补充）

共享库通常以 `.so` 结尾。构建共享对象时，源文件通常需要位置无关代码：

```bash
gcc -Wall -Wextra -fPIC -c add.c sub.c
gcc -shared -Wl,-soname,libcalc.so.1 -o libcalc.so.1.0 add.o sub.o
ln -s libcalc.so.1.0 libcalc.so.1
ln -s libcalc.so.1 libcalc.so
gcc -Wall -Wextra main.c -L. -lcalc -o calc-shared
```

链接成功不等于运行时一定能找到库。先检查依赖和搜索信息：

```bash
readelf -d calc-shared
ldd ./calc-shared
```

在个人实验目录中可临时指定搜索路径：

```bash
LD_LIBRARY_PATH="$PWD" ./calc-shared
```

更适合可移动实验程序的做法是在链接时记录相对于可执行文件的运行路径：

```bash
gcc -Wall -Wextra main.c -L. -lcalc \
  -Wl,-rpath,'$ORIGIN' -o calc-shared
./calc-shared
```

`$ORIGIN` 由动态装载器解释为可执行文件所在目录，单引号用于防止 Shell 提前展开。系统级安装应遵循发行版的库目录与包管理规范；不要把个人实验库随意复制到 `/lib` 或覆盖系统库。`ldconfig` 用于维护受信系统库目录的缓存和链接，通常需要管理员权限，不适合拿来解决任意项目目录的依赖问题。

常见错误及排查：

| 现象 | 常见原因 | 检查方法 |
|---|---|---|
| 链接时报 `cannot find -lcalc` | 缺少 `libcalc.so`/`libcalc.a` 或 `-L` 路径错误 | `ls -l`、`gcc -print-search-dirs` |
| 链接时报 `undefined reference` | 实现未入库、符号名不匹配或链接顺序不当 | `nm -C libcalc.*`、检查命令顺序 |
| 运行时报 `cannot open shared object file` | 运行时搜索路径中没有库 | `readelf -d`、`ldd`、检查 RPATH/RUNPATH |
| 库更新后程序符号错误 | ABI 或 SONAME 不兼容 | 对比 `readelf -d`、导出符号和版本 |

不要对不可信程序运行 `ldd`；检查未知 ELF 文件时优先使用 `readelf -d` 或 `objdump -p`。共享库升级要考虑 ABI 兼容性，不能只保持文件名相同。

<a id="chapter-14"></a>
## 第 3 章 GDB 调试

### 3.1 准备带调试信息的程序

GDB 用于控制程序运行、设置断点、单步执行和查看变量。源码级调试前通常用 `-g` 生成调试信息，并在初学阶段关闭优化以减少代码重排。

> **什么是代码重排**：编译器开优化时会为了性能改变机器码的组织方式，常见手段包括把互不依赖的语句调换顺序、把循环内的重复计算提到循环外、用一条指令替代一段等价指令序列，甚至完全删掉它认为“结果没有被使用”的计算。重排后的机器码与源码不再逐行对应：单步执行时可能一行源码跳过去又跳回来，断点位置与预期不一致，某些变量被优化掉后无法查看或显示“值不可用”。这是正常现象，不是程序出错。因此在调试阶段用 `-O0` 保留源码与机器码的对应关系；性能问题排查完、发布优化版本时才用 `-O2` 等级别：

下面使用“青蛙爬井`qwpj.c`”程序观察循环变量，并采用规范的函数声明和排版：

```c
#include <stdio.h>

static int qwpj(int high)
{
    int day = 0;
    int m = 0;

    while (1) {
        ++day;
        m += 5;
        if (m >= high) {
            return day;
        }
        m -= 4;
    }
}

int main(void)
{
    int day = qwpj(10);
    printf("day = %d.\n", day);
    return 0;
}
```

```bash
gcc -Wall -Wextra -g -O0 qwpj.c -o qwpj   # -g：生成调试信息；-O0：关闭优化，避免代码重排干扰单步调试
./qwpj                     # 输出：day = 6.
gdb ./qwpj                 # 启动 GDB 调试；后续在 GDB 内设断点、单步、查看变量
```

调试时可分别停在 `qwpj` 入口、`m += 5`、`m -= 4` 和返回处，观察同一天内“先上爬、到达井口立即结束、未到达才回落”的顺序。若把回落放在到达判断之前，会得到不同且错误的模型。

**如何添加断点**：进入 GDB 后用 `break`（缩写 `b`）设置断点，可以按函数名或行号指定位置，程序会在断点行**即将执行前**停下：

```gdb
(gdb) break qwpj         # 在函数 qwpj 入口设断点，停在该函数第一条可执行语句处
(gdb) break 10           # 在当前源文件第 10 行（m += 5）设断点
(gdb) break 14           # 在 m -= 4 一行设断点
(gdb) info breakpoints   # 查看已设断点的编号、位置和命中次数
```

行号对应 `list` 显示的源码行，设断点前可先用 `list 1` 确认。断点设好后用 `run` 启动程序，命中第一个断点即停；循环内的断点每轮都会命中，配合 `continue` 可逐轮观察。断点的查看、禁用、启用等完整命令见 3.3 节。

进入 GDB 时若显示 `no debugging symbols found`，说明没有读到可用调试符号，源码行、局部变量和函数名等高级调试能力会明显受限。不能只比较可执行文件大小来判断：`-g` 往往让文件变大，但大小还受优化、剥离、链接库和平台影响。

### 3.2 查看源码与开始运行

| 命令 | 缩写 | 作用 |
|---|---|---|
| `list` | `l` | 列出当前位置附近的源码，默认约 10 行 |
| `list 1` | `l 1` | 查看第 1 行附近的源码 |
| `list 函数名` |  | 查看指定函数附近源码 |
| `start` |  | 从头运行，并在 C/C++ 的 `main` 入口附近设置临时断点后停止 |
| `run` | `r` | 从头运行；在可命中的断点处停止，没有断点则运行到结束或异常 |

`list 行号` 通常把该行放在显示范围附近，并非总是严格“从该行向下列十行”。源码较长时，可在另一终端或 Vim 分屏中查看完整文件，把 GDB 的 `list` 当作当前位置提示。

程序已在运行时再次执行 `run` 或 `start`，GDB 会询问是否从头重新启动；确认后当前进程会被重新运行。`start` 不是普通持久断点，它的核心效果是便于快速停到主过程开头。

以 3.1 节的 `qwpj` 为例，这几个命令的典型用法：

```gdb
(gdb) start               # 启动并自动停在 main 入口附近（临时断点，不是 break 设的）
(gdb) list                # 显示当前位置附近源码，确认自己停在哪
(gdb) list qwpj           # 切换到 qwpj 函数附近查看
(gdb) step                # 从 main 进入自编写的 qwpj 函数
(gdb) next                # 执行一行：++day，函数调用会整体越过
(gdb) next                # 执行一行：m += 5
(gdb) print m             # 停下时查看变量，确认上一行已生效
(gdb) finish              # 不再逐行看：直接执行到 qwpj 返回，停在调用者
(gdb) continue            # 继续运行到程序结束，输出 day = 6.
```

可以看出分工：`start`/`run` 负责启动，`list` 负责定位，`step`/`next`/`finish` 负责控制停在逐行、跨函数或到返回三种粒度，`continue` 负责放手运行。连续单步时直接按回车可重复上一条命令。

### 3.3 断点、继续运行和退出

| 命令 | 缩写 | 作用 |
|---|---|---|
| `break 行号` | `b 行号` | 在当前源文件指定行设置断点 |
| `break 函数名` | `b 函数名` | 在函数入口设置断点 |
| `info breakpoints` | `info b` | 查看断点编号、位置、启用状态和命中次数 |
| `continue` | `c` | 继续运行到下一个断点、信号或程序结束 |
| `disable 编号` |  | 临时禁用断点但保留定义 |
| `enable 编号` |  | 重新启用断点 |
| `quit` | `q` | 退出 GDB；程序仍在运行时可能要求确认 |

完整操作示例（仍用 `qwpj` 程序）：

```gdb
(gdb) list 1              # 从头查看源码，确认各函数所在行号
(gdb) break qwpj          # 断点 1：qwpj 入口
(gdb) break main          # 断点 2：main 入口
(gdb) info breakpoints    # 查看两个断点的编号、位置和命中次数
(gdb) run                 # 从头运行，先命中断点 2 停在 main
(gdb) next                # 执行 int day = qwpj(10); 一行，越过函数调用
(gdb) print day           # 查看函数返回结果：应显示 6
(gdb) continue            # 继续运行，命中断点 1 停在 qwpj 入口
(gdb) disable 2           # 临时禁用断点 2（main），保留定义以便重用
(gdb) info breakpoints    # 可看到断点 2 已变为禁用状态
(gdb) enable 2            # 重新启用断点 2
(gdb) continue            # 继续到下一个停止事件：循环内断点每轮命中，直到程序结束
(gdb) quit                # 退出 GDB；程序仍在运行时会要求确认
```

断点编号和 `display` 编号属于不同列表，不要混用。循环中的断点可能反复命中；`continue` 不是“固定走到下一行”，而是继续到下一个停止事件。调试结束时若 GDB 询问是否终止仍在运行的被调试程序，应确认是否确实可以结束后再回答。

### 3.4 单步、进入函数和运行到返回

| 命令 | 缩写 | 作用 |
|---|---|---|
| `next` | `n` | 执行当前源码行；函数调用作为整体越过 |
| `step` | `s` | 执行当前源码行；遇到有可调试源码的函数时进入函数 |
| `finish` |  | 继续执行，直到当前函数返回并停在调用者 |

GDB 停止时显示的源码行通常是**下一条将要执行的行**，不是已经执行完的行。不要把 `next` 和 `step` 记反：`next` 越过函数调用，`step` 进入可调试函数。

在 `qwpj(10)` 上使用 `step` 可以进入自己编写的 `qwpj`；在 `printf`、`puts` 等库函数调用上通常使用 `next`。若库函数没有源码或调试符号，盲目 `step` 可能进入汇编代码、动态链接跳板或陌生实现。误入函数后可用 `finish` 尝试执行到当前函数返回，但函数异常退出或不返回时需改用断点等方式。

仍以 `qwpj` 为例，三个命令的差别可这样观察（设已停在 `qwpj` 入口）：

```gdb
(gdb) next                # 执行 ++day；处在 main 时遇到 qwpj(10) 也会整体越过，不进函数
(gdb) step                # 执行 m += 5；若当前行是函数调用且可调试，则进入函数体停住
(gdb) finish              # 放弃逐行：一直执行到 qwpj 返回，停在 main 的调用处
Run till exit; use `continue` to restart the session if you need to run again.
Value returned is $1 = 6
```

三者在 `qwpj(10)` 调用行的直观区别：`next` 执行完整个 `qwpj` 后停在下一行，函数内部一闪而过；`step` 停在 `qwpj` 第一条可执行语句，可逐行看循环；`finish` 则用于已经进入 `qwpj`、不想再逐行看的时候，直接跳到返回。配合 3.5 节的 `print`，停在循环内时可随时查看 `day`、`m` 的值。

如果本次命令与上一次相同，可直接按回车重复上一条 GDB 命令，例如连续执行多次 `next`。重复前先确认当前停靠位置，避免一次走过关键状态。

### 3.5 查看和持续监控变量

| 命令 | 缩写 | 作用 |
|---|---|---|
| `print 表达式` | `p 表达式` | 单次计算并显示表达式的当前值 |
| `display 表达式` |  | 每次程序停止时自动显示表达式 |
| `undisplay 编号` |  | 按显示项编号取消自动显示 |

示例：

```gdb
(gdb) start               # 启动并停在 main 入口附近
(gdb) break qwpj          # 在 qwpj 入口设断点，准备观察循环变量
(gdb) continue            # 继续运行，命中断点后停在 qwpj 第一条可执行语句
(gdb) print high          # 单次查看参数：应显示 10
(gdb) display day         # 自动显示项 1：此后每次停止都列出 day
(gdb) display m           # 自动显示项 2：此后每次停止都列出 m
(gdb) next                # 执行 ++day，停止时自动列出 day 和 m 的新值
(gdb) next                # 执行 m += 5，再次看到 day、m 自动更新
(gdb) undisplay 1         # 取消自动显示项 1（day）；编号来自 display 时的输出，不是变量名
```

`display` 会为表达式分配编号，`undisplay` 后面接的是这个编号，不是变量名。变量离开作用域时可能暂时无法显示；启用优化后，变量还可能显示为 `<optimized out>`，这也是调试版本常使用 `-O0` 或 `-Og` 的原因。

<a id="chapter-15"></a>
## 第 4 章 GNU Make 与 Makefile

### 4.1 为什么使用 Makefile

直接输入 `gcc` 适合少量源文件；项目变大后，命令会越来越长，还可能包含多个模块、生成步骤、头文件目录和库。Makefile 把这些构建规则写成可重复执行的脚本，再由 `make` 根据依赖关系决定需要执行哪些命令。

Makefile 主要有三个优点：

1. **一次书写，重复使用**：不必每次手工重输长编译命令；项目结构、模块或选项变化时仍需维护 Makefile，并非写完后永远不改。
2. **复用性较强**：把文件列表动态生成、把易变配置集中为变量后，相似项目可以复制并少量修改；不同目录结构、语言或工具链仍需适配，不能保证一个文件无修改通用于所有项目。
3. **节省后续构建时间**：第一次完整构建通常不能节省编译工作；以后只重新生成过期的目标文件，再参与链接。

C 程序从源码到可执行文件通常经历：
![C 程序的编译期、链接期与三路链接输入](<assets/10_C程序编译与链接_4K.png>)

入门时可以把“预处理、编译、汇编”合称编译期，把最后一步称为链接期；在不同语境中，“编译”有时也泛指生成最终程序的整个构建过程。链接并非在所有项目中都一定比编译快，大型程序的链接也可能耗时。

`make` 主要比较**目标的最后修改时间**与其前置条件的最后修改时间：目标不存在，或任一前置条件比目标更新时，规则需要重新执行。例如 `add.c` 比 `add.o` 新，就应重建 `add.o`。它不是比较“创建时间”，系统时钟跳变、复制文件时保留时间戳或遗漏头文件依赖，都可能使判断不符合预期。

假设项目有 5 个 `.c` 文件：第一次要生成 5 个 `.o` 再链接；之后只修改 `add.c` 时，正确的依赖图只需重新生成 `add.o`，再用 5 个 `.o` 重新链接，从而省去另外 4 个文件的编译时间。

### 4.2 项目结构、文件名与规则三要素

示例目录如下，其中 `des.c` 是实际文件名：

```text
project/
├── include/
│   └── common.h
└── src/
    ├── add.c
    ├── des.c
    ├── main.c
    ├── mul.c
    └── sub.c
```

归档内六个文件的接口和实现如下：`common.h` 声明 `add`、`sub`、`mul`、`des`；四个同名源文件分别执行整数加、减、乘、除；`main.c` 只调用 `add(2, 3)` 并输出 `2+3 = 5`。`des` 是归档原名，不静默改成 `div`。其实现直接计算 `a / b`，没有处理除数为 0，也没有处理有符号整数除法的极端溢出情形，因此只能作为分离编译和 Makefile 依赖关系的教学素材，不能直接当成健壮计算器。

本节把 Makefile 放在 `project/src/`。GNU Make 默认在**当前目录**依次查找 `GNUmakefile`、`makefile`、`Makefile`；团队项目最常见的是 `Makefile`。

**如何创建 Makefile**：它就是一个普通文本文件，用任何编辑器创建即可，例如在 `project/src/` 目录中：

```bash
cd project/src
vim Makefile        # 或 nano Makefile；保存后文件名就是 Makefile，无扩展名
```

注意两点：文件名必须完整（`Makefile`，不是 `Makefile.txt` 或 `makefile.txt`，若编辑器自动加了 `.txt` 后缀 make 会找不到）；文件里 recipe 行的缩进必须是 Tab 字符（Vim 中默认按 Tab 键即可，注意不要被编辑器配置成自动展开为空格）。

**Makefile 放在哪个目录**：没有强制位置，常见两种选择。本节放在 `project/src/`，与源文件同目录，进入该目录后直接 `make` 即可，规则中的相对路径（如 `../include`）也最直观。小型项目也可放在项目根目录 `project/`，此时规则里的路径要相应改为 `src/main.c`、`-Iinclude`。选哪种都可以，选定后规则中所有相对路径都以 Makefile 所在目录为基准。

若文件名不是默认名，或 Makefile 不在当前目录，需用选项显式指定：

```bash
make -f build.mk            # -f：指定其他文件名的 Makefile，不找默认的 Makefile
make -C project/src         # -C：先切到 project/src 目录再执行那里的 Makefile，结束后回到原目录
```

一条规则包含三个组成部分：

```make
目标: 前置条件1 前置条件2
	命令
```

| 部分 | 含义 |
|---|---|
| 目标（target） | 要生成或更新的文件，也可以是 `clean` 一类动作名称 |
| 前置条件（prerequisites，也常称依赖） | 生成目标需要的源文件、目标文件、头文件、库或其他目标 |
| 命令（recipe） | 更新目标时交给 Shell 执行的命令 |

默认语法要求每条 recipe 行以真正的 **Tab 字符**开头，不能用几个空格冒充；GNU Make 可通过 `.RECIPEPREFIX` 改前缀，但普通项目不建议为此偏离惯例。目标后的冒号应使用英文半角 `:`。Makefile 中 `#` 开始注释；recipe 内的注释还会受到 Shell 语法影响。

最直接的一步构建版本：

```make
myapp: main.c add.c des.c mul.c sub.c ../include/common.h
	gcc -I../include -Wall -g main.c add.c des.c mul.c sub.c -o myapp
```

在 `project/src` 执行：

```bash
make
./myapp
```

这个版本能生成程序，但任一源文件或头文件变化都会重新编译全部源文件，尚未实现对象文件级增量构建。

### 4.3 分离编译、链接与依赖图

要复用未变化的 `.o` 文件，应把“生成目标文件”和“链接可执行文件”拆成不同规则：

```make
# 终极目标：可执行文件；前置条件是 5 个 .o，缺谁或不新鲜就先重建谁
myapp: main.o add.o des.o mul.o sub.o
	gcc main.o add.o des.o mul.o sub.o -o myapp    # 链接：只有 .o，不再写 -c 和 -I

# main 依赖源文件和头文件：common.h 改动也会触发重编 main.o
main.o: main.c ../include/common.h
	gcc -I../include -Wall -g -c main.c -o main.o  # -c 只编译不链接；-g 带调试信息

# 其余模块不包含 common.h，前置条件只列各自的 .c
add.o: add.c
	gcc -I../include -Wall -g -c add.c -o add.o    # 未用到的模块不会被重编，实现增量构建
des.o: des.c
	gcc -I../include -Wall -g -c des.c -o des.o
mul.o: mul.c
	gcc -I../include -Wall -g -c mul.c -o mul.o
sub.o: sub.c
	gcc -I../include -Wall -g -c sub.c -o sub.o
```

`-c` 只生成 `.o`，不链接。头文件在预处理/编译阶段使用；链接阶段读取 `.o` 和库，不再解析 C 源文件里的 `#include`。但头文件仍应出现在相应 `.o` 的前置条件中，否则头文件修改后 `make` 可能错误地认为对象文件仍是最新的。

recipe 里的 `-o` 与 2.1 节 `-o 输出文件` 是同一个选项：指定本次产物的名字，告诉 GCC 把结果写到哪个文件。

有了 `-c` 为什么还要 `-o`：两个选项分工不同，不重复。`-c` 决定**这一步做什么**——只编译不链接，产物是目标文件；`-o` 决定**产物叫什么名字**——写到哪个文件。`-c` 并不负责命名：不带 `-o` 时，GCC 的默认规则是把源文件主名替换扩展名（`add.c` → `add.o`），且路径会随源文件位置（`src/add.c` → `src/add.o`）；带上 `-o` 则完全由指定的名字决定，甚至可以改名输出（如 `-o add_x86.o`）。分两步走：先问“这一步要做什么类型的事”（`-c`），再问“结果写到哪里”（`-o`）。

两种场合的用法：

- **链接规则**（`gcc main.o ... -o myapp`）：`-o` 指定可执行文件名。不写则默认写 `a.out`；显式命名为 `myapp` 后，产物名与规则目标名一致，`make` 才能靠时间戳判断 `myapp` 是否需要重建。
- **编译规则**（`gcc ... -c add.c -o add.o`）：`-o` 指定目标文件名。省略时结果恰好一致（`add.c` → `add.o`）；显式写出则不依赖默认规则，也让产物名与规则目标 `add.o` 严格对应。

简言之，`-o` 把“产物叫什么”固定下来，使命令产物与 Makefile 规则的目标名对得上，这是增量构建能正常工作的前提之一。

执行 `make` 并不是简单地从上到下运行全部命令。未指定目标时，GNU Make 先选择默认目标，再递归检查它的前置条件：

1. 默认目标 `myapp` 需要 5 个 `.o`。
2. 某个 `.o` 不存在或比对应 `.c`、头文件旧时，查找能更新它的规则。
3. 所有 `.o` 就绪后，若 `myapp` 不存在或比任一 `.o` 旧，再执行链接命令。
4. 所有目标均为最新时，通常提示 `is up to date` 或 `Nothing to be done`，不重复构建。

规则顺序通常不决定依赖执行顺序，但会影响默认目标：在没有 `.DEFAULT_GOAL` 时，第一个符合条件的普通目标通常成为默认目标。把最终目标写在前面是清晰的入门习惯，并不是说 Makefile 必须严格按文本顺序执行。

### 4.4 自动变量与模式规则

三个常用自动变量为：

| 自动变量 | 含义 |
|---|---|
| `$@` | 当前规则的目标文件名 |
| `$^` | 当前规则的全部前置条件，重复项通常只保留一次 |
| `$<` | 当前规则的第一个前置条件 |

它们由 `make` 在执行具体规则的 recipe 时自动赋值，不能当作普通全局变量在任意位置使用。链接规则可改成：

```make
myapp: main.o add.o des.o mul.o sub.o
	gcc $^ -o $@    # $^ 展开为全部前置条件（5 个 .o），$@ 展开为 myapp
```

> 展开后等价于 4.3 节的 `gcc main.o add.o des.o mul.o sub.o -o myapp`：`$^` 与 `$@` 不是新命令，只是 make 在执行 recipe 时把变量替换成规则里的具体名字，好处是前置条件增减时 recipe 不用改。

对象文件规则的目标和源文件名称只有后缀不同，可用一个模式规则替代多条重复规则：

```make
%.o: %.c
	gcc -I../include -Wall -g -c $< -o $@
```

**`%` 是什么**：`%` 是模式规则里的通配符，单独一个 `%` 匹配任意非空字符串；同一个规则里出现的多个 `%` 必须**匹配出相同的内容**，这个内容就是“主干”（stem）。`%.o: %.c` 读作“任何名字的 `.o`，都能由同名的 `.c` 生成”。

**匹配从哪里开始**：`%` 不是主动去磁盘上找文件，而是拿 make **需要构建的目标名**去撞目标模式。驱动整个匹配的是需求：

```text
make 决定需要重建 add.o（因为 myapp 的前置条件里列了它）
        │
        ▼
拿目标名 add.o 对照目标模式 %.o
        │  末尾 .o 对上字面 .o，剩下的 add 就是主干
        ▼
把主干代入前置条件模式 %.c → add.c
        │
        ▼
执行 recipe：$@=add.o，$<=add.c
```

也就是说，匹配的“输入”始终是 make 在依赖图中需要的目标名（如 `myapp` 的前置条件 `add.o`），而不是当前目录下的文件清单；代入得到的 `add.c` 只是推导出的前置条件名，真正执行前 make 才检查它是否存在。

具体例子：

- 要 `add.o` 时，目标模式 `%.o` 中的 `%` 匹配出主干 `add`，再代入前置条件模式 `%.c` 得到 `add.c`；
- 要 `mul.o` 时，同样匹配出主干 `mul`，得到 `mul.c`。

由此，一条模式规则就顶替了 4.3 节里 `add.o`、`des.o`、`mul.o`、`sub.o` 四条仅文件名不同的重复规则。

匹配出主干后自动变量随之确定，此时 `$@` 是 `add.o`，`$<` 是 `add.c`。模式规则只说明“需要某个 `.o` 时怎样从对应 `.c` 生成”，不会自动让所有 `.c` 都参与最终程序；最终目标的前置条件列表仍决定需要哪些对象文件。

### 4.5 自定义变量、`wildcard` 与 `patsubst`

自定义变量把容易变化或重复出现的文本集中定义：

```make
TARGET := myapp    # 项目自定义变量：最终目标名，后续用 $(TARGET) 引用
CC := gcc           # 覆盖 make 内置变量：编译器命令，规则里用 $(CC) 代替写死 gcc
```

`TARGET` 是项目自定变量，换目标名只改这一处；`CC` 是 make 预先内置的变量，默认值为 `cc`，这里重新赋值为 `gcc`，规则可写 `$(CC) ...`，将来换编译器（如交叉编译的 `arm-linux-gnueabihf-gcc`）也只改这一行。`:=` 表示立即展开赋值。

引用时使用 `$(变量名)`，例如 `$(CC)`。Make 变量本质上保存文本，没有 C 语言的 `int`、`long`、`float`、`double` 等数据类型。

“只能包含字母、数字、下划线且不能以数字开头”更适合作为团队命名规范，并非 GNU Make 的语法限制。GNU Make 实际允许更广泛的变量名，但特殊字符会显著降低可读性。项目自定义变量通常使用大写字母、数字和下划线，并避免数字开头；大写是惯例，不是为了区分所谓“高级语言”而强制要求。

常见赋值形式：

- `NAME = value`：递归展开，引用时再展开右侧内容。
- `NAME := value`：立即展开，定义时就计算右侧内容。
- `NAME ?= value`：变量尚未定义时才给默认值，允许命令行覆盖。
- `NAME += value`：在原值后追加。

四种写法的对比示例（重点看 `=` 与 `:=` 的差别）：

```make
A = $(B)          # 递归展开：定义时不看 B，引用 $(A) 那一刻才去取 B 的值
B := hello

C := $(B) world   # 立即展开：定义时就展开 $(B)，此时 B 已是 hello → C = "hello world"

OUTDIR ?= build   # 默认值：命令行没给 OUTDIR 时用 build；make OUTDIR=release 可覆盖
OBJS := main.o    # 先定义初值
OBJS += add.o     # += 追加：在原值后累积，现在 OBJS = "main.o add.o"
```

`A` 递归展开的后果是：引用 `$(A)` 时取的是**当时**的 `$(B)`，定义顺序无关；若引用了不存在的变量则得到空串。`=` 适合右侧引用尚未定义的变量，`:=` 更直观且能避免多次展开的意外，项目里通常默认用 `:=`，只在需要延迟展开时改用 `=`。

未定义变量通常展开为空字符串，不一定立即报语法错误，因此拼错变量名可能只表现为最终命令缺少参数。可使用 `make --warn-undefined-variables` 辅助发现问题。

`wildcard` 获取匹配的文件名：

```make
SRCFILE := $(wildcard *.c)    # wildcard 是 make 函数：把当前目录匹配 *.c 的文件名展开成列表
                              # 设有 main.c add.c 等 → SRCFILE = "main.c add.c ..."
```

它把当前工作目录中匹配 `*.c` 的名称保存到 `SRCFILE`。如果从错误目录运行，或源文件在子目录而模式没有写路径，结果可能为空。

`patsubst` 按模式替换字符串：

```make
DSTFILE := $(patsubst %.c,%.o,$(SRCFILE))   # 参数依次为：原模式、替换模式、待处理文本；
                                            # 把每个 .c 的主干套到 %.o 上 → "main.o add.o ..."
```

参数顺序是“原模式、替换模式、待处理文本”，必须使用英文逗号。它把 `SRCFILE` 中每个 `.c` 名称转换为对应 `.o` 名称并保存到 `DSTFILE`。例如：

```text
SRCFILE = add.c des.c main.c mul.c sub.c
DSTFILE = add.o des.o main.o mul.o sub.o
```

**`patsubst` 的匹配逻辑**分四步，对列表中的**每个单词**各自执行一遍：

1. **取词**：把 `$(SRCFILE)` 按空白拆成单词，逐个处理（空格分隔的列表不是一整段文本）。
2. **试匹配**：拿单词去对照原模式 `%.c`——末尾必须是字面 `.c`，其余部分作为主干。`add.c` 匹配出主干 `add`；`add.h`、`Makefile` 末尾不是 `.c`，不匹配。
3. **套替换模式**：匹配成功才替换——把主干代入替换模式 `%.o` 得 `add.o`；替换模式里的字面部分（`.o`）原样保留，只有 `%` 的位置被主干填上。
4. **不匹配则原样保留**：单词与 `%.c` 不匹配时不替换、不报错、原样放入结果列表。

以 `add.c readme.txt` 为例：`add.c` 匹配成功变成 `add.o`，`readme.txt` 不匹配原样保留，结果是 `add.o readme.txt`。注意模式匹配从**词尾**找字面后缀：`my.c.backup` 末尾是 `.backup` 不是 `.c`，不匹配而原样保留；而 `dir/add.c` 的主干是整段 `dir/add`，结果是 `dir/add.o`——替换在完整单词上进行，不是只在文件名部分。

**`*.c` 与 `%.c` 的区别**：两者角色完全不同。`*.c` 是通配符，面向**磁盘事实**——执行时扫描真实文件名，空目录得到空列表；`%.c` 是模式占位符，面向**依赖需求**——输入是 make 需要构建的目标名（如 `myapp` 的前置条件 `add.o`，或命令行 `make add.o` 指定的目标），只做纯文本匹配，不关心文件是否存在：

| | `*.c` | `%.c` |
|---|---|---|
| 出现场景 | `wildcard` 函数、shell、终端命令 | 模式规则（`%.o: %.c`）、`patsubst` |
| 展开时机 | 执行时扫描磁盘，列出真实文件 | 匹配规则或替换文本时纯文本操作 |
| 匹配含义 | `*` 匹配任意字符串（含空） | `%` 匹配任意非空字符串，即“主干” |
| 一句话概括 | 事实查询：“现在有哪些 `.c`” | 逻辑推导：“对应目标的 `.c` 叫什么” |

这解释了两者失败方式的不同：空目录里 `$(wildcard *.c)` 得到空列表，不报错；而 `%.o: %.c` 规则本身永远成立，只当某个 `.o` 被需要且同名 `.c` 不存在时，才在构建阶段报 `No rule to make target 'x.c', needed by 'x.o'`。

使用 `*` 时另有两点限制：其一，默认不匹配以 `.` 开头的隐藏文件，也不跨目录，收集子目录需写 `$(wildcard src/*.c)`；其二，裸写在规则前置条件里（如 `myapp: *.c`）会被当作字面文件名处理，行为不可靠，收集文件列表应通过 `$(wildcard ...)`。另注意 recipe 行中的 `*.c`（如 `gcc *.c -o app`）由 shell 在命令执行时展开，属于 shell 通配符而非 make 语法。

`SRCFILE`、`DSTFILE` 也可命名为更常见的 `SOURCES`、`OBJECTS`；一旦选定就要保持拼写一致。

### 4.6 常用变量与完整 Makefile

常用变量如下：

| 变量 | 典型用途 | 注意事项 |
|---|---|---|
| `TARGET` | 最终可执行文件名，如 `myapp` | 每个项目通常需要确认 |
| `CC` | C 编译器命令，如 `gcc` | C++ 项目通常另用 `CXX` / `g++` |
| `INCLUDE_PATH` | 头文件目录 | 通过 `-I` 放入 `CPPFLAGS` 更符合惯例 |
| `LIBRARY_PATH` | 库文件搜索目录 | 仅定义变量不会生效，要通过 `-L` 放入 `LDFLAGS` |
| `INSTALL_PATH` | 安装目录 | 示例使用 `/usr/bin`；手工安装项目通常更适合 `/usr/local/bin` |
| `CFLAGS` | C 编译选项 | 常放 `-Wall`、`-Wextra`、`-g`、优化级别等 |
| `CPPFLAGS` | 预处理选项 | 常放 `-I目录` 和完整的 `-D宏名[=值]`；单独一个 `-D` 不完整 |
| `LDFLAGS` | 链接器搜索路径等链接选项 | 如 `-L$(LIBRARY_PATH)` |
| `LDLIBS` | 要链接的库 | 如 `-lm`、`-lpthread` |

以下示例值为 `LIBRARY_PATH = ./lib`、`INSTALL_PATH = /usr/bin`；实际值必须按 Makefile 所在目录和部署规范调整。本节 Makefile 位于 `project/src/`，因此相邻的项目库目录应写成 `../lib`。

把前面的组成部分合成一个可复用版本：

```make
# 默认目标显式指定为 all，不依赖“第一个目标”惯例
.DEFAULT_GOAL := all

# ---- 配置区：集中定义，改一处全局生效 ----
TARGET := myapp                    # 最终可执行文件名
CC := gcc                          # 编译器命令，换编译器只改这里
INCLUDE_PATH := ../include         # 头文件目录（本 Makefile 在 project/src/ 下）
LIBRARY_PATH := ../lib             # 库目录：仅定义不生效，需经 -L 放入 LDFLAGS
INSTALL_PATH ?= /usr/local/bin     # 安装目录；?= 允许命令行 make INSTALL_PATH=... 覆盖

# ---- 编译/链接选项：按用途分入约定变量 ----
CPPFLAGS := -I$(INCLUDE_PATH) -MMD -MP   # 预处理选项：头文件目录 + 生成 .d 依赖文件
CFLAGS := -Wall -Wextra -g                # C 编译选项：警告 + 调试信息
LDFLAGS :=                               # 链接选项：如 -L$(LIBRARY_PATH)，此处暂空
LDLIBS :=                                # 要链接的库：如 -lm，此处暂空

# ---- 文件列表：自动收集并推导 ----
SRCFILE := $(wildcard *.c)         # 扫描当前目录所有 .c 源文件
DSTFILE := $(patsubst %.c,%.o,$(SRCFILE))   # 对应的 .o 目标文件列表
DEPFILE := $(patsubst %.c,%.d,$(SRCFILE))   # 对应的 .d 依赖文件列表

# 声明这些名字是伪目标：不对应文件，避免与同名文件冲突或被误判为“已最新”
.PHONY: all clean install uninstall disclean output

all: $(TARGET)                     # 默认目标：只依赖 myapp，构建链由此展开

$(TARGET): $(DSTFILE)              # 链接：所有 .o 就绪后生成 myapp
	$(CC) $(LDFLAGS) $^ $(LDLIBS) -o $@

%.o: %.c                           # 模式规则：任何 x.o 由 x.c 编译，替代逐文件写规则
	$(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

-include $(DEPFILE)                # 读入 .d 依赖文件：头文件变化也能触发重编；
                                   # 首次构建前 .d 不存在，前缀 - 便 include 不报错

clean:                             # 清理：删除中间产物 .o/.d 和可执行文件
	$(RM) $(DSTFILE) $(DEPFILE) $(TARGET)   # $(RM) 是 make 内置，默认 rm -f

output:                            # 只打印安装位置，不实际安装，便于先确认再执行
	@printf 'install target=<%s>\n' "$(INSTALL_PATH)/$(TARGET)"

install: $(TARGET)                 # 安装前先确保已构建；@ 前缀：命令本身不回显
	@test -n "$(TARGET)" || { echo 'TARGET is empty'; exit 1; }        # 变量为空则中止
	@test -n "$(INSTALL_PATH)" || { echo 'INSTALL_PATH is empty'; exit 1; }
	sudo install -m 0755 "$(TARGET)" "$(INSTALL_PATH)/$(TARGET)"     # 复制并设为可执行

uninstall:                         # 卸载：先做安全检查再删除
	@test -n "$(TARGET)" || { echo 'TARGET is empty'; exit 1; }
	@test -n "$(INSTALL_PATH)" || { echo 'INSTALL_PATH is empty'; exit 1; }
	@test "$(INSTALL_PATH)" != "/" || { echo 'refuse root directory'; exit 1; }   # 拒绝根目录
	@printf 'remove=<%s>\n' "$(INSTALL_PATH)/$(TARGET)"   # 先打印将删除的路径
	sudo rm -f -- "$(INSTALL_PATH)/$(TARGET)"

# 兼容笔记中的名称；更通用的名称是 uninstall
disclean: uninstall                # 别名：执行 disclean 等同于 uninstall
```

这个版本使用 `-MMD -MP` 生成 `.d` 依赖文件，并通过 `-include` 读取，因此实际包含的项目头文件变化时，对应 `.o` 也会重建。`-MMD -MP` 是必要补充：只比较 `.c` 与 `.o` 会漏掉头文件变化。

如果项目确实有库目录，可按需要设置：

```make
LDFLAGS := -L$(LIBRARY_PATH)
LDLIBS := -lmylib
CPPFLAGS += -DDEBUG
```

`CPPFLAGS = -D` 必须补上宏名；`CFLAGS = -I $(INCLUDE_PATH) -Wall -g` 虽然通常也能工作，但按约定把 `-I`、`-D` 放入 `CPPFLAGS` 更容易与编译选项分工。

### 4.7 功能目标、默认目标与删除安全

常见功能目标可以分为四类：

- `clean`：删除 `.o`、依赖文件和最终目标，以便完整重建。
- `install`：把 `$(TARGET)` 安装到 `$(INSTALL_PATH)`。
- `disclean`：疑似想表达 `uninstall`，这里保留为兼容别名。
- `output`：打印拼接后的安装目标路径，人工检查变量内容。

调用方式：

```bash
make                 # 构建默认目标
make clean
make output
make install
make uninstall
make disclean        # 与上例中的 uninstall 相同
```

这类动作目标通常不对应真实文件，应加入 `.PHONY`，否则目录中一旦存在名为 `clean` 的文件，`make clean` 可能被误判为无需执行。“只有目标和命令、没有依赖”对 `clean`、`output` 常成立，但不是语法要求；安全的 `install` 应依赖 `$(TARGET)`，确保安装前程序已经构建。

Recipe 在语法上可以执行 `cp`、`mv`、`chmod` 等任意可用的 Shell 命令，但目标名称应准确描述行为：`clean` 应只清理构建产物，不要暗中复制、移动或修改无关文件，否则维护者无法仅凭目标名判断影响范围。

把功能目标放在最终目标下面可以避免它们意外成为默认目标。更稳妥的写法是显式设置 `.DEFAULT_GOAL := all`；设置后，功能目标在文件中的物理位置不再决定默认构建对象。

基础的清理和卸载思路是：

```make
clean:                             # 清理构建产物：删除 .o 列表和可执行文件
	rm -rf $(DSTFILE) $(TARGET)    # -r 递归、-f 强制；对已知的普通文件其实用 rm -f 即够

install:                           # 安装：把可执行文件复制到 INSTALL_PATH
	sudo cp $(TARGET) $(INSTALL_PATH)   # 需 root 权限写入系统目录；裸 cp 不设权限，
                                    # 更稳妥的是 install -m 0755

disclean:                          # 卸载（原笔记名称，通称 uninstall）：删除安装副本
	sudo rm -rf $(INSTALL_PATH)/$(TARGET)  # 变量拼接 + 递归删除，风险高（见下文）
	rm -rf /usr/bin/myapp           # 写死路径：绕开了变量统一配置，且不该用递归

output:                            # 打印安装目标路径，供人工核对
	echo $(INSTALL_PATH)/$(TARGET)  # 无引号，变量含空格时会被拆开；printf 加边界标记更稳
```

这些目标已被完整覆盖，但不应直接照抄到真实系统：

- 已知的 `.o` 和单个可执行文件不需要递归删除，使用 `$(RM)` 或 `rm -f --` 即可。
- `sudo rm -rf` 与变量拼接组合风险极高；变量为空、拼错或包含意外空格时，Shell 可能把一条路径拆成多个删除目标。写死的 `rm -rf /usr/bin/myapp` 同样不应使用递归选项，也绕开了 `INSTALL_PATH`、`TARGET` 的统一配置。
- `/usr/bin` 由系统包管理器维护，手工安装的软件通常优先考虑 `/usr/local/bin`，并确认团队部署规范。
- `install -m 0755` 能同时复制并设置权限，比裸 `cp` 更适合安装可执行文件。
- `echo` 能打印路径；用 `printf '<%s>\n'` 加边界标记更容易发现首尾空格。

Make 变量赋值中，值末尾到 `#` 注释前的空格可能成为变量内容。例如：

```make
INSTALL_PATH := /usr/bin    # 这些空格可能进入变量值
```

拼接后可能变成 `/usr/bin /myapp`。未加引号时 Shell 会把它拆成 `/usr/bin` 和 `/myapp` 两个参数；若再配合 `sudo rm -rf`，后果可能是破坏系统目录。应把说明写到独立注释行、避免变量值尾随空格，并在删除前执行：

```bash
make output                             # 先打印拼接后的安装路径，人工核对有无空格、错拼
make -n uninstall                       # -n：只显示将执行的删除命令，不真删，预览危险操作
make --warn-undefined-variables output  # 变量拼错/未定义时告警，而不是静默展开为空
```

`make -n` 只打印将执行的 recipe，适合检查危险操作；它不是所有情形下的绝对安全沙箱，recipe 中的递归 `make` 等特殊命令仍可能执行，因此仍要阅读输出和规则。

### 4.8 排错与维护习惯

Makefile 通常没有类似 C 编译器的完整静态检查。变量名拼错时，未定义变量常展开为空，`make` 可能直到执行生成的 Shell 命令才报错。排查顺序：

1. 查看 `make` 打印的实际命令，确认缺少的是文件、选项还是变量值。
2. 用 `make --warn-undefined-variables` 查未定义变量。
3. 用 `make -n 目标` 预览 recipe，尤其是 `clean`、`install`、`uninstall`。
4. 核对 `SRCFILE`、`DSTFILE`、路径和 `patsubst` 参数顺序。
5. 检查 recipe 开头是真正的 Tab，而不是空格。
6. 修改 Makefile 后重新构建和运行程序，不能只看 `make` 是否返回成功。

工作中更常见的是阅读和修改已有构建脚本：新增模块时要找到源文件列表、依赖、编译选项、链接选项和安装目标的正确位置。Make、CMake 和其他构建系统语法不同，但都围绕“目标、依赖、构建命令和增量更新”组织；理解依赖图比死记某一份脚本更重要。


