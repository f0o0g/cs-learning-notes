# Git 与 GitHub 学习文档

## 阅读方法与目录

先理解第 1～3 章，再按第 4～8 章完成操作，最后用第 9 章完善项目说明。没有 GitHub 账号或暂时无法联网时，可以先完成本地提交，再做文件管理和本地分支练习；网络步骤留待连接恢复后完成。

全文代码块分为**需要执行的命令**、**示例输出**和**文件内容**。命令中的 `YOUR-USERNAME`、`you@example.com` 等占位值必须先替换。每个操作块执行后先检查结果，再继续下一步；条件示例只在符合其前提时执行。

1. [认识 Git 与 GitHub](#chapter-1)
2. [认识 GitHub 项目](#chapter-2)
3. [准备环境与 SSH 认证](#chapter-3)
4. [第一次本地提交](#chapter-4)
5. [第一次推送到 GitHub](#chapter-5)
6. [下载与同步项目](#chapter-6)
7. [修改、删除与忽略文件](#chapter-7)
8. [分支与冲突入门](#chapter-8)
9. [用 Markdown 编写 README](#chapter-9)
10. [综合练习与常见问题](#chapter-10)
11. [附录 A：课程易错点修正](#appendix-a)
12. [附录 B：原始材料索引](#appendix-b)
13. [附录 C：官方参考与验证说明](#appendix-c)

<a id="chapter-1"></a>

## 1. 认识 Git 与 GitHub

### 1.1 为什么需要版本控制

假设你在写一个项目，每次修改都另存为“最终版”“最终版2”“真正最终版”。时间一长，就很难回答：哪一版修复了问题？昨天删除的内容在哪里？两个人的改动怎样合在一起？

**版本控制**负责记录文件随时间发生的变化。Git 让你把有意义的一组改动保存为一次提交，之后可以查看历史、比较差异、找回以前保存的内容，也可以同时尝试不同方向的修改。

Git 记录的是你主动提交的内容。编辑器里按下保存键，不会自动产生 Git 提交；做了本地提交，也不会自动上传到 GitHub。

### 1.2 Git、GitHub 和 Git Bash 分别是什么

| 名称 | 作用 | 例子 |
| --- | --- | --- |
| Git | 安装在电脑上的分布式版本控制工具 | 在本地记录一次修改、查看昨天的版本 |
| GitHub | 托管 Git 仓库并支持项目协作的网站 | 保存远程仓库、浏览代码、跟踪问题 |
| Git Bash | Git for Windows 提供的 Bash 终端环境 | 输入 `git status`、`cd` 等命令 |
| Markdown | 使用纯文本符号描述文档结构的轻量级标记语言 | 编写项目的 `README.md` |

Git 可以离线完成初始化、提交、查看历史和本地合并。连接 GitHub 时才涉及网络和相应权限。GitHub 也不是 Git 唯一能使用的远程托管服务。

Git Bash 提供了一些常见命令，但并不等于完整的 Linux 系统，也不保证已经安装 C/C++ 编译器或项目所需依赖。

### 1.3 仓库、提交、分支与 HEAD

| 概念 | 准确理解 |
| --- | --- |
| 仓库（repository） | 保存版本历史及相关管理信息；普通本地仓库的管理数据通常在隐藏的 `.git` 目录中 |
| 提交（commit） | 对当时暂存内容形成的一次版本记录，包含快照、作者、说明及父提交等信息 |
| 提交编号 | 用于标识提交的哈希值，日志里常显示其缩写；每个人的编号通常不同 |
| 分支（branch） | 指向某个提交、会随新提交向前移动的引用，用于组织不同的开发路线 |
| HEAD | 表示当前检出的版本；在本文的正常分支操作中，它通常指向当前分支 |
| 默认分支 | GitHub 仓库默认展示及协作使用的分支，由仓库设置决定 |

一个账号可以拥有多个仓库。常见做法是一个项目放一个仓库，但这不是 Git 强制规定。

**分支不是把不同文件分别装入抽屉。** 同一个项目的两个分支可以共享大量历史，只在部分修改上不同。切换分支时，Git 会相应更新工作区中的文件。

课堂使用 `master` 作为主分支名，本文新建练习仓库明确使用 `main`。两者都是分支名称，没有“其中一个版本更高级”的区别；操作已有项目时应使用它实际的分支名。

### 1.4 四个位置：文件改动到底在哪里

| 位置 | 存放什么 | 你能怎样观察 |
| --- | --- | --- |
| 工作区 | 当前可以打开和编辑的项目文件 | 编辑器、资源管理器、`git diff` |
| 暂存区（index / staging area） | 准备放入下一次提交的内容 | `git diff --staged` |
| 本地仓库 | 已经提交的版本历史 | `git log` |
| 远程仓库 | GitHub 等位置保存的仓库与历史 | 仓库网页、远程分支信息 |

课堂中的“缓冲区”“缓存区”在这些操作语境中统一称为**暂存区**。它不是普通的临时缓存，而是决定下一次提交包含什么内容的区域。

```text
编辑文件          git add          git commit          git push
   ↓                 ↓                  ↓                  ↓
工作区 ───────────→ 暂存区 ───────────→ 本地仓库 ───────────→ 远程仓库
  ↑                                      ↑
  └──── merge 后更新文件 ────────────────┘

远程仓库 ── git fetch ──→ 本地远程跟踪引用（例如 origin/main）
本地远程跟踪引用 ── git merge ──→ 当前本地分支及工作区
```

`origin/main` 是本地记录的“上次获知的远程 main 状态”，并不是随时联网更新的实时视图。第 6 章会实际演示。

**自检：** 如果文件已经修改，但还没运行 `git add`，它是否会自动进入下一次普通的 `git commit`？答案是不会。下一次提交读取的是暂存区的内容。

<a id="chapter-2"></a>

## 2. 认识 GitHub 项目

### 2.1 仓库地址与可见性

仓库网页地址通常具有 `https://github.com/所有者/仓库名` 的结构。所有者可以是个人，也可以是组织；仓库地址与本地文件夹名不必相同。

| 可见性 | 含义 |
| --- | --- |
| Public（公开） | 其他人通常可以浏览和克隆；不代表其他人可以直接推送 |
| Private（私有） | 只有获得相应访问权限的用户可以访问 |

**读取权限与写入权限是两件事。** 能看到公开项目、下载它并在本地修改，并不代表能把修改推回原作者的仓库。

先注册并登录自己的 GitHub 账号。第 5 章再创建用于练习的远程仓库；本章先了解网页中的内容。

### 2.2 进入一个项目后先看什么

| 位置 | 重点阅读内容 |
| --- | --- |
| Code | 源代码、配置、文档及目录结构；确认当前查看的分支 |
| README.md | 项目用途、运行条件、安装方式、示例和限制 |
| LICENSE | 使用、修改、分发项目时需要遵守的许可条件 |
| Issues | 已报告的缺陷、功能请求和相关讨论；可先搜索有没有同样的问题 |
| 提交历史 | 最近改动、维护情况，以及具体修改了哪些行 |
| Releases（如有） | 作者发布的版本说明和附件，可能提供可直接运行的程序 |

不要把“下载源代码”和“安装软件”混为一谈。源码项目可能还需要编译、解释器或依赖；是否能直接运行，要看 README。也不是所有 GitHub 项目都只提供源码，有些会在 Releases 发布安装包。

### 2.3 搜索示例与学习资料

`sample`、`demo`、`tutorial` 是可以组合使用的普通关键词，不是必须遵守的特殊“资料标签”。它们常出现在仓库名称、描述和 README 中，含义大致如下：

| 关键词 | 常见含义 |
| --- | --- |
| `sample` | 示例代码，展示某个库、框架或 API 的基本用法，通常体量较小、只覆盖核心功能 |
| `demo` | 演示项目，重在展示运行效果，通常是可以跑起来的完整小项目 |
| `tutorial` | 教程，按步骤讲解并配有说明文档，适合跟着操作学习 |

简单的使用例子（在搜索栏输入，并在结果页选择 **Repositories**）：

```text
socket sample language:C++  # 找 C++ 的 socket 示例代码
python tutorial             # 找讲解 Python 的教程仓库
git topic:tutorial          # 用 topic: 限定仓库主题为“教程”
```

代码块中 `#` 及其后的内容是本文添加的注释，实际搜索时不需要输入。第一行在普通关键词 `sample` 的基础上用 `language:` 限定编程语言；第二行只用普通关键词搜索；第三行的 `topic:` 用于限定仓库主题（作者主动打的标签），匹配方式与普通关键词不同。无论用哪种方式搜索，结果仍需检查 README、依赖要求和更新时间，不能仅凭名称判断是否适合自己。参见 [GitHub 仓库搜索语法](https://docs.github.com/en/search-github/searching-on-github/searching-for-repositories)。

### 2.4 许可证入门

公开展示代码不等于授予任意用途的许可。项目缺少许可证时，不应直接把它当作“可以任意复制、修改和分发”的开源项目。许可范围应以实际许可证文件为准。参见 [GitHub：为仓库提供许可](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository)。

课堂提到的三种许可证，可以先这样区分：

| 许可证 | 入门时需要记住的要点 |
| --- | --- |
| [MIT](https://choosealicense.com/licenses/mit/) | 较宽松，允许商业使用等用途，但需要保留相应版权和许可声明 |
| [Apache-2.0](https://choosealicense.com/licenses/apache-2.0/) | 较宽松，包含明确的专利授权；分发时需要遵守保留声明、标注修改等要求，并按条件处理 NOTICE |
| [GPL-3.0](https://choosealicense.com/licenses/gpl-3.0/) | 允许商业使用；分发受其约束的软件或衍生作品时，涉及提供对应源码及相同许可等要求 |

这张表用于识别基本差异，不代替许可证全文。是否收费、是否公开可见、允许怎样使用，应分别判断。本文的练习项目不需要先替自己选择开源许可证。

### 2.5 分享项目与仓库管理

分享作业时，通常复制浏览器地址栏中的仓库网页链接即可。分享私有仓库链接不会自动赋予接收者读取权限。

**课堂补充：** 仓库设置和账号设置属于不同范围。删除仓库通常在该仓库的 **Settings → General → Danger Zone** 中；注销账号在个人账号设置中。删除前要看清对象与确认提示，这两项不属于本教程练习步骤。删除 GitHub 仓库不会自动删除电脑里的克隆；删除本地文件夹也不会自动删除 GitHub 仓库。具体流程见 [删除仓库](https://docs.github.com/en/repositories/creating-and-managing-repositories/deleting-a-repository) 和 [删除个人账号](https://docs.github.com/en/account-and-profile/how-tos/account-management/deleting-your-personal-account)。

<a id="chapter-3"></a>

## 3. 准备环境与 SSH 认证

### 3.1 安装并检查 Git

从 [Git for Windows 官方网站](https://gitforwindows.org/) 获取安装程序。完成安装后，从开始菜单打开 **Git Bash**；也可以在目标文件夹中右键选择 **Open Git Bash here**，Windows 11 中该菜单有时位于“显示更多选项”内。

输入：

```bash
git --version
```

预期输出以 `git version` 开头，后面是版本号。本文本地验证环境为 `2.55.0.windows.4`；示例使用的 `git init -b main` 需要 Git 2.28 或以上版本，旧版本可先升级。安装与基础配置可参考 [GitHub：设置 Git](https://docs.github.com/en/get-started/git-basics/set-up-git)。

### 3.2 先学会定位文件夹和输入命令

| 命令或符号 | 含义 |
| --- | --- |
| `pwd` | 显示当前所在目录 |
| `ls` / `ls -a` | 列出文件 / 同时列出隐藏项目 |
| `cd 目录` | 进入指定目录 |
| `cd ..` | 返回上一级目录 |
| `~` | 当前用户的主目录 |
| `mkdir 目录名` | 新建目录 |
| Tab | 尝试补全命令或文件路径 |
| ↑ / ↓ | 浏览以前输入的命令 |
| Ctrl+C | 中断当前终端操作，不能保证撤销已经完成的动作 |

例如，Windows 路径 `C:\Users\你的用户名\Desktop`，在 Git Bash 中可以写成 `/c/Users/你的用户名/Desktop`。路径包含空格时，用英文引号包起来。本文把练习放在 `~/git-practice`，不用依赖桌面的实际位置。

输入命令时注意：

- 使用英文半角空格、引号和减号；不要把 `--global` 写成中文破折号。
- 代码块没有包含终端提示符，直接复制命令即可，不要额外输入 `$`。
- 在 Git Bash 中可用右键菜单或 Shift+Insert 粘贴；多行块按顺序执行，确认每步没有异常。
- `git`、`ssh-keygen` 是命令名，`--global`、`-t` 是参数；参数之间用空格分开。
- `git log`、`git diff` 可能进入分页器，按 `q` 退出。若意外进入 Vim，见第 10 章。

### 3.3 配置“提交作者”，不是登录 GitHub

将下面的姓名和邮箱替换为自己的信息后执行。姓名可以是你希望展示的署名，不必与 GitHub 用户名相同。

```bash
git config --global user.name "你的署名"
git config --global user.email "you@example.com"
git config --global --get user.name
git config --global --get user.email
```

`--global` 表示设置当前电脑用户使用的默认值。后两条命令用于检查。若希望提交关联到 GitHub 账号，使用该账号已添加的邮箱；如需隐藏个人邮箱，可以使用 GitHub 邮箱设置页提供的 noreply 地址。参见 [设置 Git 用户名](https://docs.github.com/en/get-started/git-basics/setting-your-username-in-git) 和 [设置提交邮箱](https://docs.github.com/en/account-and-profile/how-tos/email-preferences/setting-your-commit-email-address)。

以下是按需使用的配置查询方式：

```bash
git config --list
git config --show-origin --get user.email
```

第一条列出有效配置；第二条帮助判断邮箱来自哪个配置文件。在某个仓库里设置不带 `--global` 的 `user.name`、`user.email`，可以覆盖该仓库使用的默认值。

**课堂命令说明：** `git config --global --unset user.email` 会删除全局邮箱配置，不是“退出登录”。它不属于首次配置的必做步骤。改错了邮箱时，重新运行设置命令通常就够了。

### 3.4 SSH 认证解决什么问题

提交身份回答“这次提交署名是谁”；SSH 认证回答“连接 GitHub 的用户能证明自己持有哪个密钥”。配置姓名和邮箱不能代替认证。

SSH 密钥是一对配合使用的密钥：

| 文件 | 用途 |
| --- | --- |
| `~/.ssh/id_ed25519` | 私钥，保存在自己电脑上，用来证明持有相应密钥 |
| `~/.ssh/id_ed25519.pub` | 公钥，可以添加到 GitHub 的 SSH keys 中 |

密钥不是从网卡 MAC 地址或 IP 地址提取的“设备身份证”。SSH 使用密钥进行身份认证，并建立加密连接；公钥能证明的关系与用户对私钥的控制有关。

### 3.5 检查并生成密钥

先在 Git Bash 中查看是否已有密钥：

```bash
ls -al ~/.ssh
```

如果提示目录不存在，首次使用时很正常。如果已经有准备使用的密钥，可以复用并跳到下一节。不要为了跟着教程而覆盖已有私钥。

**仅在需要新密钥时执行：** 替换邮箱后，生成 Ed25519 密钥。

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

- `-t ed25519`：指定密钥类型。
- `-C`：添加便于识别的注释，示例用邮箱；它本身不会完成账号绑定。
- 询问保存路径时，没有同名密钥才接受默认路径；发现覆盖提示应停止并检查。若另取文件名，后续读取和添加密钥时也要使用那个文件名。
- 询问 passphrase 时，为私钥设置保护口令并重复输入。终端不显示输入字符是正常现象；它与 GitHub 登录密码不是一回事。

课堂使用的是 RSA。旧系统不支持 Ed25519 时，官方示例是 `ssh-keygen -t rsa -b 4096 -C "you@example.com"`。这两种生成方式择一即可；采用 RSA 时默认文件名相应为 `id_rsa`。参见 [GitHub：生成 SSH 密钥](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)。

### 3.6 在 Git Bash 中加载密钥并添加公钥

下面针对 Git for Windows 自带的 SSH 工具，在当前 Git Bash 会话启动 agent，并让它管理私钥口令：

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

第一条通常输出 agent 的进程编号，第二条可能要求输入刚才设置的私钥口令，然后提示密钥已添加。关闭终端后，下次会话如有需要可重新加载。

Windows 也有系统自带的 OpenSSH 服务，不能假定它与 Git Bash 的 agent 互通。如果你已有不同的 SSH 配置，应按所用客户端排查；本教程的命令不需要修改 Windows 服务。参见 [GitHub：SSH agent 与 Windows 客户端说明](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#troubleshooting-ssh-agent-conflicts-in-windows)。

查看要添加到 GitHub 的**公钥**：

```bash
cat ~/.ssh/id_ed25519.pub
```

复制完整公钥这一行，包括开头的密钥类型及末尾注释。然后在 GitHub 中操作：

1. 点击头像，进入 **Settings**。
2. 打开 **SSH and GPG keys → New SSH key**。
3. Title 填可识别的名称，例如“个人电脑”；Key type 选择 **Authentication Key**。
4. 在 Key 中粘贴公钥内容并添加，按网页要求完成身份确认。

添加的是 `.pub` 文件内容，不是私钥内容，也不是文件路径。参见 [GitHub：添加 SSH 公钥](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)。

### 3.7 验证 SSH 连接

```bash
ssh -T git@github.com
```

这里的 `git` 是固定的 SSH 用户名，不能换成你的 GitHub 用户名。首次连接可能询问是否信任服务器：将提示中的主机密钥指纹与 [GitHub 官方指纹](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints) 对照，一致后再输入 `yes`。

成功时会显示类似：

```text
Hi YOUR-USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

这表示认证成功；后半句表示 GitHub 不提供交互式 shell，并不是认证失败。此测试成功时退出码仍可能为 `1`，应结合上述响应判断。再确认问候中的账号确实是目标账号。参见 [GitHub：测试 SSH 连接](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection)。

如果看到 `Permission denied (publickey)`，检查公钥、私钥与 agent；如果是连接超时，先检查网络。认证成功也不代表你对任意仓库都有写权限。

**本章完成标志：** Git 可以运行，提交署名已配置，使用 SSH 时能收到正确账号的认证成功提示。

<a id="chapter-4"></a>

## 4. 第一次本地提交

本章所有操作只发生在电脑上，不需要 GitHub 或网络。`diff`、`log` 及暂存快照演示属于**日常必需补充**。

### 4.1 创建独立练习目录并初始化

在 Git Bash 中执行：

```bash
mkdir -p ~/git-practice
cd ~/git-practice
mkdir git-learning-demo
cd git-learning-demo
git init -b main
pwd
git status
```

`mkdir -p` 可以创建父目录，父目录已存在时也可以继续。第二次 `mkdir` 刻意不加 `-p`：如果同名练习目录已经存在，先确认它的内容；不要继续把本节的文件生成命令用于已有项目。

`git init -b main` 在当前目录建立本地仓库，并把初始分支名指定为 `main`。它不会联网，不会创建 GitHub 仓库，也不会生成 SSH 密钥。

`git status` 此时应该说明当前分支是 `main`，还没有提交。运行 `ls -a` 可以看到 `.git` 目录。不要手动修改或删除它；其中包含本地版本历史与配置。

初始化不等于已经有了第一个版本。第一次成功提交后，`main` 才有一个可以指向的提交。

### 4.2 创建第一个文件

为了让所有人的练习内容一致，先通过命令生成 README：

```bash
printf '%s\n' '# Git Learning Demo' '' 'A repository for learning Git.' > README.md
cat README.md
git status --short
```

`printf` 是 Bash 环境中的文本输出命令，不是 Git 命令。`%s\n` 表示把每段文字分别输出并换行；空字符串会产生空行。`cat` 用于查看文件。

`>` 会创建或覆盖文件，`>>` 会在文件末尾追加。本文只对指定练习文件使用这些写入方式；也可以在编辑器中保存同样的内容。保存为 UTF-8，确认文件名没有误变成 `README.md.txt`。

预期状态：

```text
?? README.md
```

`??` 表示未跟踪。文件已经保存在工作区，但尚未加入暂存区。

### 4.3 添加到暂存区，并理解“添加时的内容”

```bash
git add README.md
git status
git diff --staged
```

`git add README.md` 将这个文件当前的内容加入暂存区。`git diff --staged` 显示暂存区相对上一次提交的变化；首次提交前没有旧版本，因此这里显示新文件的内容。

现在故意在暂存后再修改一次工作区：

```bash
printf '\n%s\n' 'Goal: learn Git step by step.' >> README.md
git status --short
git diff
git diff --staged
```

预期 `git status --short` 中出现：

```text
AM README.md
```

这里第一列 `A` 表示相对 HEAD，文件已作为新增内容暂存；第二列 `M` 表示工作区又比暂存区多了修改。`??` 是未跟踪文件的特殊表示。

| 命令 | 比较的内容 | 本次应该看见什么 |
| --- | --- | --- |
| `git diff` | 工作区与暂存区 | 刚追加的目标说明 |
| `git diff --staged` | 暂存区与 HEAD | 第一次 `add` 时的 README，还没有目标说明 |

如果此时直接提交，目标说明不会被包含。重新暂存，让下一次提交包含最新内容：

```bash
git add README.md
git diff
git diff --staged
```

这次 `git diff` 应没有差异，`git diff --staged` 应包含全部 README 内容。普通 `git diff` 默认不展示未跟踪文件的内容，查看新文件还需要结合 `git status` 和编辑器。

### 4.4 创建第一次提交

```bash
git commit -m "初始化学习项目"
git status
git log --oneline
```

`-m` 后的内容是提交说明。它应该描述这次做了什么，便于以后查找；中文和英文都可以。

完成后，`git status` 应说明工作区干净，例如 `nothing to commit, working tree clean`。`git log --oneline` 应看到一条“初始化学习项目”，前面的提交编号因人而异。

这次提交保存在**本地仓库**中，尚未推送到 GitHub。它会使用本地磁盘空间。

### 4.5 再提交一次，观察历史

新增学习记录文件：

```bash
printf '%s\n' 'Today: git add and git commit.' > learning.txt
git status
git add learning.txt
git diff --staged
git commit -m "添加第一条学习记录"
git log --oneline
git status
```

现在日志应有两条提交，最新的在前。每条提交保存的是整个版本的逻辑快照，不只是提交说明里的那个文件。

想进一步查看最近一次提交，可执行：

```bash
git show --stat HEAD
git log -1
```

`--stat` 展示改动文件及行数摘要，`-1` 限制只显示最近一次提交。完整差异通常用 `+` 标记新增、`-` 标记删除；不要只依赖颜色，终端主题可能不同。

**提交习惯：** 一次提交尽量表达一件完整、容易解释的事情。修复一个小问题或完善一段文档都可以提交，不必等到“重大功能上线”；也不需要把每次按保存键都变成一个提交。

**本章完成标志：** 当前目录为练习仓库，分支是 `main`，日志有两条提交，工作区干净，并且能解释 `add` 与 `commit` 的区别。

<a id="chapter-5"></a>

## 5. 第一次推送到 GitHub

### 5.1 在网页创建空远程仓库

前提：已完成第 3 章 SSH 认证和第 4 章两次本地提交。

在 GitHub 的新建仓库页面操作：

1. 所有者选择自己的账号，Repository name 填 `git-learning-demo`。
2. Description 可以填写“Git 学习练习”。个人练习可以选择 **Private**；需要公开展示时选择 **Public**。
3. 本次**不要初始化 README、.gitignore 或 LICENSE**，让远程仓库保持空白。
4. 创建后，在快速设置区域选择 **SSH**，复制仓库地址。

这里保持远程为空，是因为本地已经有独立创建的提交。若你的 GitHub 仓库已有 README 或其他提交，优先采用第 6 章的“克隆已有仓库后再添加文件”方式，不要直接照搬本章空仓库流程。参见 [GitHub：创建仓库](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository)。

### 5.2 为远程地址设置名称 origin

回到原练习仓库，将 `YOUR-USERNAME` 换成仓库所有者的真实用户名：

```bash
cd ~/git-practice/git-learning-demo
git remote add origin git@github.com:YOUR-USERNAME/git-learning-demo.git
git remote -v
```

`origin` 是当前本地仓库给远程地址起的名称，惯例如此，并非必须如此。地址也不是 GitHub 的个人主页地址。

预期会列出 origin 的 fetch 和 push 地址；在本例中二者相同。`git remote add` 只是保存地址，不会立即上传，也不能证明地址和权限一定有效。

**地址写错时才执行：** 修改已有 origin 的地址。

```bash
git remote set-url origin git@github.com:YOUR-USERNAME/git-learning-demo.git
git remote -v
```

课堂中的 `git remote remove origin` 是删除当前本地仓库的远程配置及相关跟踪信息，不会删除 GitHub 仓库。仅修正地址时，用 `set-url` 更直接。

### 5.3 第一次推送并建立上游关系

```bash
git branch --show-current
git push -u origin main
git branch -vv
```

第一条应显示 `main`。推送命令可以拆开理解：

| 部分 | 含义 |
| --- | --- |
| `push` | 向远程传送所需对象并请求更新远程引用 |
| `origin` | 目标远程仓库名称 |
| `main` | 本例将本地 main 推送到远程同名分支 |
| `-u` | 成功推送时，为本地分支设置对应的上游关系 |

上游（upstream）是本地分支默认关联的远程分支。本例设置成功后，`git branch -vv` 应显示 `main` 对应 `[origin/main]`。

刷新 GitHub 仓库网页，应该能看到 `README.md`、`learning.txt` 和提交历史。GitHub 会渲染仓库中的 README。

`push` 不会把工作区里的全部文件自动上传：未暂存或未提交的改动不会因为推送而成为远程提交。Git 推送也不是向网盘上传整个 `.git` 文件夹。

### 5.4 后续修改：还是先提交，再推送

```bash
printf '%s\n' 'Today: git push.' >> learning.txt
git diff
git add learning.txt
git diff --staged
git commit -m "记录首次推送练习"
git push origin main
git status
```

预期 GitHub 上的 `learning.txt` 多出一行，并新增对应提交。打开该提交可以查看行级变化。

本教程为方便辨认，继续使用显式的 `git push origin main`。在本文的普通配置及上游关系下，后续也可以使用 `git push`。

### 5.5 为什么 push 不等于自动合并

对于普通分支，推送通常要求远程原有提交已经包含在你要推送的历史中，即可以**快进**。若其他人先推送了你没有的提交，Git 会拒绝覆盖那段历史。此时要先获取并整合远程修改，而不是认为“分支同名就会自动合并”。参见 [Git push 官方说明](https://git-scm.com/docs/git-push)。

```text
可以快进：        A──B          远程在 A，本地在 B，远程可前进到 B

已经分叉：          B           本地
                  /
                 A
                  \
                   C           远程；需要先整合 B 与 C
```

**本章完成标志：** GitHub 上出现练习文件与三条提交，当前分支关联 `origin/main`，本地工作区干净。

<a id="chapter-6"></a>

## 6. 下载与同步项目

本章的 ZIP、克隆来自课堂；远程跟踪引用、拉取策略和双副本同步属于**日常必需补充**。

### 6.1 ZIP、clone 和 pull 有什么不同

| 操作 | 得到什么 | 适合什么情况 |
| --- | --- | --- |
| Code → Download ZIP | 所选版本的文件快照，通常不包含 `.git` 历史 | 只想查看某一版本的文件 |
| `git clone` | 新的本地仓库，通常包含可获取的历史，并检出一个分支 | 第一次把已有仓库拿到本地并继续维护 |
| `git fetch` | 获取远程对象，更新相应的远程跟踪引用 | 已有本地仓库，先了解远程变化 |
| `git pull --ff-only` | 获取远程变化，并在能够快进时更新当前分支 | 已有本地仓库，希望安全地跟上远程进度 |

ZIP 解压后不会自动拥有原仓库的版本历史；`git clone` 则会建立 Git 仓库。克隆通常会自动配置 `origin` 并建立初始分支的上游关系，因此无需再次 `git init` 或 `git remote add origin`。参见 [Git clone 官方说明](https://git-scm.com/docs/git-clone)。

如果远程已经有提交，推荐先克隆，再把自己的新文件复制到克隆目录，检查差异并提交。复制时不要把另一个仓库的 `.git` 目录带进去。

### 6.2 HTTPS 与 SSH 都能用于 Git 传输

| 地址形式 | 示例 | 认证说明 |
| --- | --- | --- |
| HTTPS | `https://github.com/YOUR-USERNAME/git-learning-demo.git` | 公开仓库读取通常无需认证；私有读取或写入需要相应认证，可由凭据管理器处理 |
| SSH | `git@github.com:YOUR-USERNAME/git-learning-demo.git` | 使用已配置的 SSH 密钥进行账号认证，访问私有仓库或写入仍受权限限制 |

HTTPS 不是“只能下载”，SSH 也不是“只能上传”。GitHub 的 HTTPS Git 操作不能用普通账号密码代替所需凭据；需要时使用凭据管理器或合适权限的令牌，令牌不要写进仓库文件。本文继续用已经配置好的 SSH。参见 [GitHub：远程仓库地址](https://docs.github.com/en/get-started/git-basics/about-remote-repositories)。

### 6.3 克隆第二个工作副本

前提：第 5 章推送成功。为观察同步，在同一电脑上再克隆一份，名字叫 `git-learning-demo-copy`。这两个目录各有自己的工作区、暂存区和 `.git`，指向同一个 GitHub 远程仓库。

```bash
cd ~/git-practice
git clone git@github.com:YOUR-USERNAME/git-learning-demo.git git-learning-demo-copy
cd git-learning-demo-copy
git status
git remote -v
git log --oneline
```

预期该副本也有前三次提交，当前分支为 `main`，状态干净。如果目录已经存在，先检查，不要把克隆结果和旧目录混在一起。

不要在原仓库里再克隆一个子仓库。本例先切回父目录，让两个目录并排存放。

### 6.4 远程领先：先 fetch，再快进更新

先在**副本目录**提交一个新文件，模拟另一台电脑完成了一次修改：

```bash
cd ~/git-practice/git-learning-demo-copy
printf '%s\n' 'A note from the second working copy.' > teammate.txt
git add teammate.txt
git commit -m "从第二个副本添加记录"
git push origin main
```

回到**原目录**，先获取信息：

```bash
cd ~/git-practice/git-learning-demo
git fetch origin
git status
git log --oneline main..origin/main
git diff main origin/main
ls
```

此时应该观察到：

- `origin/main` 已指向新提交，本地 `main` 仍停留在原位置，状态可能显示落后一个提交。
- `main..origin/main` 列出远程跟踪分支有、当前 main 没有的提交。
- `git diff main origin/main` 显示两个版本的文件差异。
- 工作区暂时还没有 `teammate.txt`，因为 `fetch` 没有替你更新当前分支和文件。

确认没有未提交修改后，进行快进更新：

```bash
git pull --ff-only origin main
cat teammate.txt
git status
```

预期现在能看到文件内容，当前分支与上游同步。

`--ff-only` 只在不需要创建合并提交时前进分支；双方有各自的新提交时，它会拒绝整合，不会替你决定合并方式。即使此前已执行 `fetch`，再执行 `pull` 也没问题，只是会再获取一次远程状态。参见 [Git pull 官方说明](https://git-scm.com/docs/git-pull)。

### 6.5 双方都有提交：处理推送被拒绝

本节刻意制造一次失败，用来理解非快进推送。接着上一节操作，两份副本现在都包含 `teammate.txt`。

先在**副本目录**创建远程的新提交：

```bash
cd ~/git-practice/git-learning-demo-copy
printf '%s\n' 'A new note published by the second copy.' > remote-note.txt
git add remote-note.txt
git commit -m "副本发布远程笔记"
git push origin main
```

再回到**原目录**，不拉取，直接创建自己的提交：

```bash
cd ~/git-practice/git-learning-demo
printf '%s\n' 'A new note committed in the original copy.' > local-note.txt
git add local-note.txt
git commit -m "原目录添加本地笔记"
git push origin main
```

最后一条命令预期被拒绝，提示可能含有 `rejected`、`fetch first` 或 `non-fast-forward`。这是因为原目录不知道副本刚推送的提交，远程历史并不是本地新历史的完整前缀。

试着快进拉取，同样应该被拒绝：

```bash
git pull --ff-only origin main
```

这不是 SSH 配置突然失效。先检查并获取历史，再明确合并：

```bash
git status
git fetch origin
git log --oneline --graph --all --decorate
git merge -m "合并远程学习笔记" origin/main
git push origin main
git status
```

合并前应保持工作区干净。本例在两侧新增的是不同文件，通常可以自动合并，合并后同时保留 `local-note.txt` 与 `remote-note.txt`，再推送就可以成功。

`git merge origin/main` 的含义是把已经获取到本地的远程跟踪分支历史整合进**当前分支**。`-m` 为需要创建的合并提交提供说明，避免本例进入编辑器。如果出现内容冲突，暂停在合并步骤，按第 8 章的方法处理后再推送。

不要为了绕过这类拒绝直接使用强制推送；先保留并整合双方的历史。仓库若有特定团队协作规范，再按团队要求操作。

### 6.6 以后每天怎样同步

日常开始修改前，在目标仓库查看状态；工作区干净、当前为目标分支时，再拉取：

```bash
git status
git branch --show-current
git pull --ff-only origin main
```

随后编辑文件、查看差异、暂存、提交，最后推送。`git status` 显示干净，只说明当前文件与暂存区没有待提交变化；没有新执行 `fetch` 或 `pull` 时，不能仅凭它认定服务器没有新提交。

遇到“本地修改会被覆盖”时，先整理并提交自己需要保留的修改，或把尚不准备提交的内容备份到仓库外，再决定下一步。不要为了成功拉取而无意丢弃内容。

**本章完成标志：** 原目录中同时有两份笔记，能够解释“克隆”“获取”“拉取”，并成功经历一次推送被拒绝后的合并与重推。

<a id="chapter-7"></a>

## 7. 修改、删除与忽略文件

课堂包含添加、删除和恢复文件。本章对取消暂存、从提交恢复以及忽略规则的展开属于**日常必需补充**。

### 7.1 先分清自己想撤销哪一步

以下是命令说明，先阅读，再执行后面的具体练习。本文这些恢复示例都建立在**已有至少一次提交**的仓库中。

| 目的 | 命令形式 | 对工作区文件的影响 |
| --- | --- | --- |
| 取消暂存，保留自己写的内容 | `git restore --staged -- 文件` | 文件内容保留；暂存区恢复为 HEAD 中对应状态 |
| 放弃尚未暂存的修改 | `git restore -- 文件` | 用暂存区内容覆盖工作区对应文件 |
| 从上一次提交恢复工作区文件 | `git restore --source=HEAD -- 文件` | 用 HEAD 内容覆盖工作区；不改暂存区 |
| 删除文件，并把删除暂存 | `git rm -- 文件` | 工作区文件被删除，删除动作进入暂存区 |
| 停止跟踪，保留本地文件 | `git rm --cached -- 文件` | 文件保留在本地，暂存从版本中移除的操作 |

`--` 用来结束选项，让后面的值按文件路径处理。`restore` 需要指定恢复目标，并不是输入一个空的 `git restore` 就能自动找回所有内容。

**会覆盖内容的恢复命令没有通用“撤销按钮”。** 执行前先看 `git diff`，只对确认可以丢弃的练习改动操作。已经提交的历史、暂存内容和从未记录的文件，能恢复的依据并不相同。恢复语义见 [Git restore 官方说明](https://git-scm.com/docs/git-restore)。

### 7.2 取消暂存，但保留编辑内容

回到原目录，确认状态干净后执行：

```bash
cd ~/git-practice/git-learning-demo
git status
printf '%s\n' 'Temporary practice line.' >> learning.txt
git add learning.txt
git diff --staged
git restore --staged -- learning.txt
git status --short
git diff
```

预期：`git diff --staged` 中原先能看到新增行；取消暂存后，该行仍在文件里，`git diff` 能看到它，状态变为未暂存的修改。取消暂存没有删除你的输入。

现在这行只是练习草稿，接着演示丢弃它：

```bash
git restore -- learning.txt
git diff
git status
```

预期临时行消失、差异为空、工作区干净。如果你想保留这行，应该重新暂存并提交，而不是执行这一步。

### 7.3 误删已跟踪文件：暂存区还在时怎样找回

仅删除本节练习文件，模拟在资源管理器中误删：

```bash
rm learning.txt
git status --short
git restore -- learning.txt
cat learning.txt
git status
```

这里的 `rm` 是普通文件删除命令，未把删除暂存；暂存区中仍有文件内容，因此 `restore` 可以恢复它。普通 `rm` 通常不经过 Windows 回收站，不要替换成自己的重要文件来试。

如果文件从未暂存、从未提交，Git 通常没有可供恢复的内容，不能把这个示例理解为“Git 能找回任意误删文件”。

### 7.4 正式删除文件，以及从提交中找回

先创建一个有历史记录的专用练习文件：

```bash
printf '%s\n' 'A disposable tracked file.' > obsolete.txt
git add obsolete.txt
git commit -m "添加删除练习文件"
```

删除并观察暂存状态：

```bash
git rm -- obsolete.txt
git diff --staged
git status --short
```

此时 `obsolete.txt` 已从工作区和暂存区移除，但 **HEAD 中仍有它**。单独 `git restore -- obsolete.txt` 不能从当前暂存区取回已移除的路径。如果尚未提交删除，想取消删除，可执行：

```bash
git restore --staged -- obsolete.txt
git restore -- obsolete.txt
git status
```

第一条先从 HEAD 恢复暂存区，第二条再从暂存区恢复工作区。预期文件回来，工作区干净。

接着真正提交删除，并用紧邻的上一个提交演示找回：

```bash
git rm -- obsolete.txt
git commit -m "删除练习文件"
git restore --source=HEAD~1 -- obsolete.txt
cat obsolete.txt
git status --short
git add obsolete.txt
git commit -m "从历史恢复练习文件"
```

本例的 `HEAD~1` 是当前提交的第一个父提交，恰好含有删除前的文件。恢复后文件先以未跟踪状态回到工作区，需要重新 `add` 和 `commit` 才形成新的恢复记录。

真实项目中应先用 `git log --oneline -- 文件名` 找到正确版本，再使用相应提交编号作为 `--source`。不要假定每个场景的 `HEAD~1` 都是想要的版本。**删除文件不会自动抹去历史提交中已有的内容。**

### 7.5 用 .gitignore 排除不应跟踪的文件

练习创建忽略规则，只在本示例尚无 `.gitignore` 时运行下面的生成命令；已有规则应在编辑器中补充。

```bash
printf '%s\n' '# Local-only files' '.env' 'local-settings.txt' 'build/' '*.log' > .gitignore
mkdir -p build
printf '%s\n' 'temporary build output' > build/output.txt
printf '%s\n' 'temporary log' > practice.log
git status --short
git check-ignore -v build/output.txt practice.log
git add .gitignore
git commit -m "添加忽略规则"
```

预期普通 `status` 只显示需要提交的 `.gitignore`；`build/output.txt`、`practice.log` 被忽略。`check-ignore -v` 会说明命中了哪个文件中的哪条规则。

| 规则 | 本例含义 |
| --- | --- |
| `.env` | 忽略名为 .env 的本地配置文件 |
| `local-settings.txt` | 忽略本地个人设置 |
| `build/` | 忽略匹配的构建输出目录 |
| `*.log` | 忽略日志文件 |

`.gitignore` 一般应作为项目规则提交。它主要影响未跟踪文件，不会自动停止跟踪已经提交的文件；也不会删除旧提交中的数据。参见 [Git ignore 官方说明](https://git-scm.com/docs/gitignore)。

### 7.6 已跟踪文件如何保留在本地、停止进入新版本

为了看清区别，使用一份**不含密码的模拟设置**。下面的 `-f` 仅用于刻意把刚才已忽略的演示文件加入版本，不是日常添加被忽略文件的建议。

```bash
printf '%s\n' 'theme=light' > local-settings.txt
git add -f local-settings.txt
git commit -m "添加模拟个人设置以演示停止跟踪"
git rm --cached -- local-settings.txt
git diff --staged
cat local-settings.txt
git commit -m "停止跟踪个人设置"
git ls-files -- local-settings.txt
git status --short
```

预期：文件仍在本地，可以 `cat` 查看；`git ls-files` 不再列出它，状态中也不再提示它，因为已经有对应忽略规则。这只是让文件退出后续版本，并未清除先前提交的内容。

如果误提交的是真实密钥或密码，单纯添加忽略规则或 `rm --cached` 不足以处理已泄露凭据，应先使凭据失效，再按项目流程处理历史。练习文件只使用上面的虚构设置。

本章练习结束后，把产生的提交推送到已有远程仓库：

```bash
git push origin main
git status
```

如果暂时只做离线练习，可稍后推送。Git 通常不跟踪空目录；需要目录出现在版本中时，应在其中放入有实际用途的文件。

**本章完成标志：** 能区分取消暂存、丢弃修改、删除文件和停止跟踪，并理解恢复文件时必须有明确的恢复来源。

<a id="chapter-8"></a>

## 8. 分支与冲突入门

> 日常必需补充：课堂介绍了分支概念，本章补齐创建、切换、合并及解决冲突的实际操作。

### 8.1 为什么要新建分支

假设 main 上是当前可用版本，你想先写一份新说明。可以从 main 的当前提交创建功能分支，先在新分支提交，确认后再合回 main。

```text
创建功能分支后：
                 B──C   feature-notes
                /
               A        main

如果 main 没有新增提交，合并时可以快进：
               A──B──C  main、feature-notes
```

分支名不是文件名。创建分支通常很轻量，不会把整个项目复制成另一个目录；在同一工作区切换分支时，你看到的文件会随版本变化。

### 8.2 新建分支、修改并合回 main

从原练习目录开始，确认工作区干净。如果有自己的未提交改动，先整理好再切换。

```bash
cd ~/git-practice/git-learning-demo
git status
git switch main
git switch -c feature-notes
git branch
printf '%s\n' 'Check git status before and after each operation.' > tips.txt
git add tips.txt
git commit -m "添加操作检查提示"
```

`git switch -c feature-notes` 创建并切换到新分支。`git branch` 列出本地分支，当前分支前有 `*`。如果分支已经存在，使用 `git switch 分支名` 切换，不要重复创建。

现在切回 main，观察文件，再合并：

```bash
git switch main
ls
git merge -m "合并学习提示" feature-notes
cat tips.txt
git branch -d feature-notes
git push origin main
```

合并前，main 中还没有 `tips.txt`；合并后，该文件及对应提交进入 main。因为本例 main 没有独立前进，通常输出 `Fast-forward`，**不会额外产生一条合并提交**，`-m` 的说明也就没有用到。

`git branch -d feature-notes` 删除的是本地分支名称，不会把已经合入 main 的文件删掉；它也不会删除远程同名分支。本文没有推送过该功能分支，因此只需要处理本地名称。若 `-d` 提示尚未合并，先检查历史，不要直接改为强制删除。

### 8.3 制造一次确定的内容冲突

先在 main 创建一个所有分支共享的基础文件：

```bash
git switch main
printf '%s\n' 'color=blue' > theme.txt
git add theme.txt
git commit -m "添加颜色配置基础版本"
git switch -c feature-color
printf '%s\n' 'color=red' > theme.txt
git add theme.txt
git commit -m "功能分支选择红色"
```

然后回到 main，把同一行改成另一个值：

```bash
git switch main
printf '%s\n' 'color=green' > theme.txt
git add theme.txt
git commit -m "主分支选择绿色"
git merge -m "合并颜色选择" feature-color
```

最后一条命令预期报告 `CONFLICT`，并停下来等待你解决。这是因为同一基础版本的一行被两侧改成不同内容，Git 无法替你选择想保留哪种含义。

此时查看：

```bash
git status
cat theme.txt
```

在默认冲突展示方式下，文件类似：

```text
<<<<<<< HEAD
color=green
=======
color=red
>>>>>>> feature-color
```

`HEAD` 一侧是当前 main 的内容，另一侧来自正在合入的 `feature-color`。标记行用来表示待解决范围，不是应该保留的配置。使用不同冲突显示配置时，文件也可能包含共同祖先信息。

此时历史形状是：

```text
             R   feature-color：红色
            /
基础版本 B
            \
             G   main：绿色，合并尚未完成
```

### 8.4 编辑结果、标记解决并完成合并

本例决定将最终颜色设置为紫色。这说明解决冲突并不局限于“保留一侧”，也可以写出兼顾需求的新内容。

```bash
printf '%s\n' 'color=purple' > theme.txt
git add theme.txt
git diff --staged
git status
git commit -m "解决颜色冲突并统一为紫色"
git log --oneline --graph --all --decorate
git branch -d feature-color
git push origin main
git status
```

实际项目中，应在编辑器中检查每个冲突块并删除标记；本例文件只有一行，因此可以直接写入确定内容。`git add` 表示你已把这个文件整理为期望结果，`git commit` 才完成这次合并。

如果有多个冲突文件，可以用 `git diff --name-only --diff-filter=U` 列出尚未解决的路径。逐一处理，全部确认后提交；还需根据项目检查程序或文档是否正确，标记消失并不保证业务含义正确。

完成后的合并提交有两个父提交：

```text
             R────────┐
            /         │
基础版本 B            M   main：紫色
            \         │
             G────────┘
```

与第 8.2 节的快进不同，这次会产生新的合并提交 M。参见 [Git merge 官方说明](https://git-scm.com/docs/git-merge)。

### 8.5 暂时不想解决：中止合并

**条件路线，不是上一步之后继续执行的命令：** 如果你停留在第 8.3 节的冲突现场，尚未提交合并结果，可以选择中止：

```bash
git merge --abort
git status
cat theme.txt
```

本例从干净状态开始合并，中止后应回到合并前的 main，`theme.txt` 恢复为 `color=green`，功能分支上的红色提交仍在。

中止不是删除分支，也不是撤销已经完成的合并提交。已经完成第 8.4 节时，不需要执行 `--abort`。如果在合并前就带着未提交改动，Git 不一定能完整重建原状态，因此要养成合并前先检查状态的习惯。

选择了中止、之后又想继续本次练习时，重新运行下面的命令，再按第 8.4 节解决：

```bash
git merge -m "再次合并颜色选择" feature-color
```

**本章完成标志：** 能创建并切换分支，区分快进和合并提交，识别冲突标记，并独立完成一次冲突解决。

<a id="chapter-9"></a>

## 9. 用 Markdown 编写 README

Markdown 可以用任意纯文本编辑器编写，不依赖特定收费编辑器。文件通常保存为 `.md`；GitHub 使用的语法在基础 Markdown 上还支持表格、任务列表等扩展。

本章标为 `markdown` 的代码块展示的是**文件内容**，不是终端命令。可以将示例复制到编辑器查看预览，不要直接粘贴到 Git Bash 执行。

### 9.1 标题与文档层级

```markdown
# 项目名称

正文简介。

## 安装方法

说明安装步骤。

### Windows

说明 Windows 下的具体步骤。
```

`#` 后面有一个空格；标题支持 1～6 级。层级表达文档结构，不应只为追求字体大小而跳级。一篇文档通常使用一个一级标题，主要章节用二级标题。

### 9.2 段落、换行和转义

空一行表示新段落：

```markdown
这是第一段。

这是第二段。
```

在 GitHub 的 `.md` 文档中，仅在源码里按一次 Enter，通常不会强制产生可见换行。如果需要在同一段内换行，可以在行尾加两个空格，或使用行尾反斜杠：

```markdown
第一行末尾有反斜杠。\
这是紧接着的第二行。
```

也可以使用 `<br>`；普通行内反斜杠常用于转义，而不是任意位置都表示换行：

```markdown
第一行<br>
第二行

\# 这会显示井号，而不是成为标题。
\*这对星号按普通字符显示\*
```

GitHub 的评论输入框和 `.md` 文件对普通换行的处理可能不同，应在实际发布位置预览。参见 [GitHub：段落与换行](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#line-breaks)。

### 9.3 粗体、斜体、删除线和行内代码

| 写法 | 效果或用途 |
| --- | --- |
| `**重点**` | **重点** |
| `*强调*` | *强调* |
| `***特别强调***` | ***特别强调*** |
| `~~旧说明~~` | ~~旧说明~~ |
| 单个反引号包住命令 | 例如 `git status`，表示行内代码 |

强调标记应紧贴内容，写成 `**重点**`，不要照搬手写转写中的 `** 重点 **`。行内代码适合文件名、参数和短命令，不只是给普通词语加背景。

如果想在正文显示含反引号的文字，可用更长的反引号包裹，例如 `` `git status` ``。分隔线可以独占一行写成 `---` 或 `***`，前后留空行。

### 9.4 列表与任务清单

```markdown
- Git 基础
  - 查看状态
  - 创建提交
- GitHub 基础
  - 推送项目

1. 编辑文件。
2. 检查差异。
3. 暂存并提交。

- [x] 完成首次提交
- [ ] 完成冲突练习
```

无序列表使用 `-`、`*` 或 `+` 加空格；有序列表使用数字、英文句点和空格。嵌套内容按父级结构缩进，示例用空格，不依赖编辑器对 Tab 的不同处理。

列表用于组织并列项和步骤，不会自动生成可点击目录。目录需要链接到相应标题或锚点；例如本文开头的目录。

### 9.5 引用块

```markdown
> 先查看状态，再决定下一步操作。
>
> 这是同一个引用块中的另一段。

> 一级引用。
>> 嵌套引用。
```

`>` 后面加空格，多个 `>` 可以形成嵌套。引用别人的内容时，还应写明出处；引用块也可以用于简短说明。

### 9.6 超链接与图片

```markdown
[Git 官方网站](https://git-scm.com/)
[GitHub](https://github.com/ "打开 GitHub")
[查看学习提示](tips.txt)

![练习截图](images/practice.png "练习结果")
```

链接和图片都使用英文半角 `[]`、`()`。图片前面多一个 `!`；方括号里的文字是图片的替代文本，后面的可选引号内容是悬停提示。

**仓库内图片可以使用相对路径，不必上传到图床。** 例如 README 在仓库根目录，图片放在同一仓库的 `images/practice.png`，就可以使用上面的写法；图片文件本身也需要提交并推送。相对路径以当前 Markdown 文件位置为基准，文件名大小写应一致。参见 [GitHub：相对链接和图片](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#relative-links)。

`C:\Users\某人\Desktop\图片.png` 是某台电脑的本地绝对路径，GitHub 无法据此读取那台电脑上的图片。公开可访问的网络图片 URL 也可以使用，但能否显示取决于该地址是否持续可用、是否要求登录等。

上面的 `images/practice.png` 是语法示例，主线练习没有创建该文件。没有实际图片时，不要把这个占位图片链接直接留在 README 中。

### 9.7 表格与对齐

```markdown
| 命令 | 作用 | 是否访问远程 |
| :--- | :---: | ---: |
| git status | 查看状态 | 否 |
| git push | 推送提交 | 是 |
```

第一行是表头，第二行是分隔与对齐设置，后面是数据。每列分隔单元格至少写三个 `-`：`:---` 表示左对齐，`:---:` 居中，`---:` 右对齐。列之间用 `|` 分隔，表格前后留空行。

单元格要显示竖线时通常写 `\|`，避免它被当成新的一列。表格适合比较命令，不适合塞入很长的多段正文。参见 [GitHub：创建表格](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/organizing-information-with-tables)。

### 9.8 围栏代码块

下面的外层代码框用于展示 Markdown 源码；复制源码后，三反引号之间的内容会显示为代码块：

````markdown
```bash
git status
git log --oneline
```

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, Git!\n";
    return 0;
}
```
````

开头和结尾的三个反引号需要配对。开头的 `bash`、`cpp`、`python`、`sql` 等用于声明语言，以便渲染器做语法高亮，不会执行代码，也不会决定固定的背景色。颜色由平台与主题控制。

如果代码内容自身含有三反引号，可以用四反引号作为外层围栏，本文就是这样展示代码块写法的。参见 [GitHub：创建代码块](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks)。

### 9.9 为练习项目写一份完整 README

完成第 8 章后，打开**原练习目录**中的 `README.md`，用下面的内容替换原来的简短介绍。此示例仅引用主线练习中实际存在的文件，不包含图片占位。

````markdown
# Git Learning Demo

一个用于练习 Git 与 GitHub 的学习项目。

## 学习目标

- 区分工作区、暂存区、本地仓库和远程仓库。
- 完成提交、推送、克隆和同步。
- 使用分支，并解决一次内容冲突。

## 环境

- Windows
- Git for Windows / Git Bash

## 查看项目

进入自己的本地项目目录，在 Git Bash 中执行：

```bash
git status
git log --oneline --graph --all
```

## 文件说明

| 文件 | 用途 |
| --- | --- |
| README.md | 项目介绍 |
| learning.txt | 日常学习记录 |
| tips.txt | 操作检查提示 |
| theme.txt | 分支冲突练习，最终颜色为 purple |
| .gitignore | 忽略本地配置、日志和构建输出 |

## 学习记录

- [查看学习日志](learning.txt)
- [查看操作提示](tips.txt)

## 已完成练习

- [x] 第一次本地提交
- [x] 推送与拉取
- [x] 分支合并
- [x] 冲突解决

> 每次提交前先检查差异，提交后再确认仓库状态。
````

保存文件，检查内容和文件名，再执行：

```bash
cd ~/git-practice/git-learning-demo
git diff -- README.md
git add README.md
git diff --staged
git commit -m "完善项目学习说明"
git push origin main
git status
```

刷新 GitHub 仓库首页，检查标题、列表、表格、代码块及文件链接是否正常。主线未完成的练习，应把 README 对应的 `[x]` 改回 `[ ]`，不要把模板文字当成已经发生的事实。

**本章完成标志：** README 能在 GitHub 正确显示，并清楚说明项目用途、环境、文件和学习成果。

<a id="chapter-10"></a>

## 10. 综合练习与常见问题

### 10.1 独立练习：给项目增加一页复习笔记

在已经完成主线的 `git-learning-demo` 中完成下面的任务。先自己尝试，遇到困难再回看对应章节。

1. 回到 main，检查状态并拉取远程更新。
2. 创建并切换到 `feature-review` 分支。
3. 新建 `review.md`，写出 Git 与 GitHub 的区别、四个区域的关系，以及三条最容易混淆的命令。
4. 在 README 中添加到 `review.md` 的相对链接。
5. 查看未暂存差异，将两个文件暂存，再查看暂存差异并提交。
6. 切回 main，将复习分支合入，删除已合并的本地复习分支并推送。
7. 在 GitHub 打开 README 中的链接，确认能访问复习笔记和对应提交。

建议提交说明写成“添加 Git 复习笔记”，让别人只看说明就能知道这次提交的目的。

**验收清单：**

- [ ] 能解释 `add`、`commit` 和 `push` 分别改变哪个位置。
- [ ] main 的工作区干净，复习内容已在 GitHub 可见。
- [ ] README 链接可以打开，Markdown 没有断裂的代码块或表格。
- [ ] 日志能看到复习提交，临时功能分支已合并。
- [ ] 能重新演示取消暂存但保留文件内容。
- [ ] 能解释为什么非快进推送会被拒绝，以及如何保留双方改动。
- [ ] 能识别冲突文件，并在解决后完成提交。

### 10.2 五个理解题

| 问题 | 参考答案 |
| --- | --- |
| `git add` 后又改了一行，直接 `commit` 会包含那行吗？ | 普通提交不会；需要再次暂存该修改 |
| 不联网可以提交吗？ | 可以。本地提交与联网推送是不同步骤 |
| `fetch` 成功后为什么文件没有变化？ | 它更新了远程跟踪信息，尚未整合进当前分支和工作区 |
| 设置了 `user.email`，为什么仍无法推送？ | 提交邮箱不是认证凭据，还需检查认证、仓库权限和历史状态 |
| 文件加入 `.gitignore` 后，历史中的秘密消失了吗？ | 没有，忽略规则不会清理既有历史 |

### 10.3 看报错时先定位失败阶段

不要看到失败就从头重新初始化仓库或重新生成密钥。先判断失败发生在：终端与目录、配置与提交、连接与认证、仓库权限，还是历史整合。

| 提示或现象 | 常见原因 | 检查与处理 |
| --- | --- | --- |
| 找不到 `git` / `command not found` | 未安装、终端未重新打开或环境路径不正确 | 打开 Git Bash，运行 `git --version`；必要时检查安装 |
| `not a git repository` | 不在仓库中，或该目录没有 Git 管理信息 | 用 `pwd` 确认位置，切入正确目录；不要在不确定的目录随意 init |
| `Author identity unknown` | 没有有效的提交署名配置 | 检查 `user.name`、`user.email` 及其配置来源 |
| `nothing to commit` | 没有已暂存的新变化，也可能选错目录或文件被忽略 | 结合 `status`、`diff`、`diff --staged`、`check-ignore` 判断 |
| `remote origin already exists` | 已经配置过 origin，clone 后也会如此 | 用 `remote -v` 查看，地址错误时 `remote set-url origin 地址` |
| `src refspec main does not match any` | 尚无首次提交，或没有名为 main 的分支 | 查看 `git log --oneline` 和 `git branch --show-current`，按实际分支处理 |
| 没有 upstream / tracking information | 当前分支未设置上游 | 初次推送实际分支时使用 `git push -u origin 分支名` |
| `Permission denied (publickey)` | SSH 没用上对应私钥，或账号没添加对应公钥 | 检查公钥添加情况、`ssh-add -l` 及 `ssh -T git@github.com` |
| `Repository not found` | 地址错误、仓库不存在或当前账号无权限 | 检查所有者与仓库名、登录账号及私有仓库授权，不能仅判断为仓库不存在 |
| `403` 或写入权限被拒绝 | 凭据、账号或仓库规则不允许写入 | 核对推送地址、使用的账号与权限；组织仓库还可能有规则限制 |
| `Connection timed out`、无法解析主机 | 网络、DNS、代理或端口连通性问题 | 区分域名解析与端口连接问题；SSH 超时不靠重新填邮箱解决 |
| `rejected` / `non-fast-forward` | 远程已有本地未包含的历史 | 按第 6.5 节 fetch、检查、merge 后再推送 |
| `Not possible to fast-forward` | 双方历史分叉，不能仅移动当前分支指针 | 检查历史后采用明确的整合方式；本文使用 merge |
| `local changes ... would be overwritten` | 未提交修改可能被拉取、合并或切换覆盖 | 先整理并提交需要保留的改动，或另外备份后再处理 |
| `refusing to merge unrelated histories` | 两边通常分别独立初始化，没有共同历史 | 初学练习优先克隆已有远程，再复制自己的文件并提交；保留原目录，不用强制推送掩盖问题 |
| `CONFLICT` / `unmerged paths` | 合并尚待人工解决 | 查看冲突文件，编辑结果，add 后 commit；或在未完成时 abort |
| LF / CRLF 换行警告 | Windows 与其他环境的行尾表示不同 | 警告本身不一定表示提交失败；结合结果检查，项目已有行尾规则时遵守规则 |

如果 SSH 的 22 端口确实被当前网络阻止，可以进一步按 [GitHub：通过 HTTPS 端口使用 SSH](https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port) 测试官方的 443 端口连接方式；不要把它当成所有网络或权限错误的统一修复。

### 10.4 分页器与编辑器怎样退出

| 当前界面 | 操作 |
| --- | --- |
| `git log` / `git diff` 的分页器 | 按 `q` 返回命令行 |
| Vim 编辑器，准备保存并退出 | 先按 Esc，再输入 `:wq` 并回车；也可按大写 `ZZ` |
| Vim 编辑器，想让当前编辑任务以失败退出 | 先按 Esc，再输入 `:cq` 并回车；随后用 `git status` 检查 Git 状态 |
| Nano 编辑器 | Ctrl+O 保存，按提示确认文件名，再 Ctrl+X 退出 |

Vim 的 `:q!` 是放弃本次编辑，不应一概理解为“取消 Git 操作”：如果已经有有效的默认说明，Git 仍可能继续。本文提交使用 `-m`，通常不需要进入提交说明编辑器。

### 10.5 按阶段检查自己的结果

| 刚完成的操作 | 检查命令或位置 | 应关注什么 |
| --- | --- | --- |
| 编辑文件 | `git status`、`git diff` | 改的是不是预期文件，有无意外删除 |
| 暂存 | `git diff --staged` | 下一次提交将包含哪些内容 |
| 提交 | `git log -1`、`git status` | 提交说明是否正确，有无遗漏修改 |
| 推送 | GitHub 网页和提交历史 | 目标仓库、分支、文件内容是否正确 |
| 获取远程 | `git log --oneline main..origin/main` | 远程有哪些自己尚未整合的提交 |
| 解决冲突 | `git status`、文件内容和项目检查 | 有无未解决路径，最终内容是否符合预期 |

<a id="appendix-a"></a>

## 附录 A：课程易错点修正

下表针对原始转写和手写笔记中的易误解表述整理。部分错误可能来自语音识别，而不是授课者的原意；正文使用核实后的解释，不要求逐字复现原稿。

| 原稿中的表述或可能造成的误解 | 本文采用的解释 | 对应章节 |
| --- | --- | --- |
| 分支是资源存储单元，可把项目文件拆进不同抽屉 | 分支是指向提交并可移动的引用，用于组织开发路线 | 1、8 |
| 主分支一定叫 master | 默认分支名可变；本文新建示例显式使用 main | 1、4、5 |
| `git init` 建立设备认证通道 | init 创建本地仓库；SSH 认证另外完成 | 3、4 |
| Git 姓名、邮箱就是登录凭据，姓名必须等于账号名 | 它们是提交署名配置，认证与权限另行处理 | 3 |
| SSH 公钥里包含 MAC 地址、IP 地址等设备指纹 | 密钥用于证明私钥持有关系，不是设备信息提取结果 | 3 |
| SSH 密钥生成时一路回车，旧密钥可以直接覆盖 | 先检查已有密钥，明确文件路径和保护口令；不要覆盖正在使用的私钥 | 3 |
| add 后文件再改变，也会自动包含在提交中 | 暂存记录的是 add 时的内容，需要再次 add 才纳入新修改 | 4 |
| status 只查看缓冲区；红色、绿色就是是否能恢复 | status 同时报告工作区、暂存区等状态，颜色不是恢复能力判断依据 | 4、7 |
| 本地永远比远程新 | 本地、远程可以任一方领先，也可以分叉 | 6 |
| 同名分支推送时自动合并、替换 | 普通推送更新远程引用，通常要求快进；分叉先获取并整合 | 5、6 |
| clone 只下载资源文件，推拉只有原作者才能使用 | 普通 clone 创建本地仓库并获取历史；读取与写入由各自权限决定 | 2、6 |
| HTTPS 是下载地址，SSH 是上传地址 | 两种协议都可用于 Git 传输，认证方式不同 | 6 |
| commit 只消耗 GitHub 空间，能自动实时备份一切 | commit 在本地保存主动暂存的内容，占本地空间；push 才涉及远程 | 4、5 |
| 只有重大功能才提交 | 一个完整、可说明的小改动也适合提交 | 4 |
| `git rm` 后所有历史都无法恢复 | 它移除工作区和暂存内容；已提交的历史仍可能包含文件 | 7 |
| `git rm --cached` 等于取消暂存 | 它用于停止跟踪；保留改动但取消暂存用 `restore --staged` | 7 |
| 已跟踪文件写进 .gitignore 就消失了 | 忽略规则不会自动移除已跟踪文件，更不会清理旧历史 | 7 |
| GitHub 图片必须用图床网络地址 | 同仓库图片可以用相对路径；他人无法访问的是本机绝对路径 | 9 |
| 粗体符号内要加空格；链接可以用中文圆括号 | 强调标记紧贴文字，链接和图片使用英文半角括号 | 9 |
| 代码块的语言名决定固定的背景和字体颜色 | 语言名用于高亮识别，外观由渲染平台与主题决定 | 9 |
| 知名许可证均可随便使用，学生身份免除许可条件 | 各许可证有不同条件，是否适用不以学生身份判断 | 2 |

<a id="appendix-b"></a>

## 附录 B：原始材料索引

本教程使用的三份原始材料如下。“资料”文件夹未参与整理，原始文稿未修改。

| 原始材料 | 主要内容 | 本文位置 |
| --- | --- | --- |
| 2026-09-09 课堂转写（原始材料未随仓库发布） | GitHub、仓库与分支、搜索、项目页面、许可证、Git 配置和 SSH | 第 1～3 章 |
| 2026-09-10 课堂转写（原始材料未随仓库发布） | 远程地址、暂存与提交、推送、下载、仓库管理和 Markdown | 第 4～7、9 章，及相关补充 |
| 手写笔记转写（原始材料未随仓库发布） | 5 页笔记中的 Git 命令、流程和 Markdown 语法 | 全文交叉核对 |

录音转写中的时间标记可用于回查上下文，不代表每个小主题恰好从该秒开始：

| 课程日期 | 时间标记附近 | 回查内容 |
| --- | --- | --- |
| 09-09 | 00:03:04 | 仓库、分支与项目搜索 |
| 09-09 | 00:23:21 | Code、Issues、README 和许可证 |
| 09-09 | 00:39:05 | 初始化仓库与提交身份配置 |
| 09-09 | 00:52:23 | SSH 密钥与连接认证 |
| 09-10 | 00:00:00 | 远程地址和 origin 名称 |
| 09-10 | 00:07:08、00:16:17 | 暂存、提交、推送流程与状态 |
| 09-10 | 00:27:00 | 实际推送、提交历史与差异 |
| 09-10 | 00:46:00 | ZIP 下载与克隆 |
| 09-10 | 00:54:15 | 仓库管理、账号管理及分享作业 |
| 09-10 | 00:58:54 起 | Markdown 概览、图片与各类语法 |

整理时删除了重复口语、等候网络、课堂闲聊以及与 Git/Markdown 主线无关的编辑器安装激活过程。手写转写中无法确认的旁注，没有擅自补成技术结论。

<a id="appendix-c"></a>

## 附录 C：官方参考与验证说明

正文各主题附近已放置直接参考链接，下面保留便于继续学习的主要入口：

| 主题 | 官方资料 |
| --- | --- |
| 安装 Git | [Git for Windows](https://gitforwindows.org/) |
| 系统学习 | [Pro Git 中文版](https://git-scm.com/book/zh/v2) |
| 查询命令 | [Git 命令参考](https://git-scm.com/docs) |
| SSH 接入 | [GitHub：通过 SSH 连接](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) |
| 克隆仓库 | [git clone](https://git-scm.com/docs/git-clone) |
| 获取与拉取 | [git fetch](https://git-scm.com/docs/git-fetch)、[git pull](https://git-scm.com/docs/git-pull) |
| 推送与合并 | [git push](https://git-scm.com/docs/git-push)、[git merge](https://git-scm.com/docs/git-merge) |
| 恢复与删除 | [git restore](https://git-scm.com/docs/git-restore)、[git rm](https://git-scm.com/docs/git-rm) |
| 忽略文件 | [gitignore](https://git-scm.com/docs/gitignore) |
| Markdown | [GitHub 基础书写语法](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) |
| 许可证 | [GitHub 许可证说明](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository) |

### 本文验证范围

命令与文档结构的验证记录将在校验完成后补全。
