# 第 12 章 命令行、Visual Studio 与补充练习

> 本章目标：补齐命令行入口参数，建立从创建项目到调试的 Visual Studio 操作闭环，并为原始代码目录中未单列的练习建立可复核索引。

## 本章知识地图

- 托管环境下常用的 C 程序入口是 `main(void)` 或 `main(int argc, char *argv[])`。
- `argc` 表示参数数量，`argv` 指向参数字符串，`argv[argc]` 保证为空指针。
- 命令行传入的是文本；把它当数值使用前必须转换并校验。
- Visual Studio 中 `.c` 扩展名决定源文件按 C 语言编译，项目模板名称出现 “C++” 不代表只能写 C++。
- 断点、单步、调用栈、局部变量、监视和内存窗口分别回答不同的调试问题。
- 独立练习应同时处理正常输入、相等/零值、越界和非法输入。

## 12.1 `main` 的两种常用形式

不接收命令行参数时：

```c
int main(void)
{
    return 0;
}
```

接收命令行参数时：

```c
int main(int argc, char *argv[])
{
    return 0;
}
```

第二个参数也可写成 `char **argv`，两种声明在函数参数位置等价。名称 `argc`、`argv` 是约定俗成而非关键字，但应保留这些名称以便阅读。

标准还允许由实现提供其他入口形式。例如部分 Windows 工具链支持额外参数或宽字符入口，但那属于平台扩展；跨平台示例优先使用上面两种形式。

| 参数/返回值 | 方向 | 作用 |
|---|---|---|
| `argc` | 输入 | 参数字符串数量，用它限制所有 `argv` 下标访问。 |
| `argv` | 输入 | 指向参数字符串指针数组；`argv[i]` 是第 `i` 个以 `\0` 结尾的字符串。 |
| `argv[argc]` | 哨兵 | 标准保证为空指针，不是普通参数。 |
| `main` 返回值 | 输出给运行环境 | 0 表示成功；非 0 表示失败类别，具体约定由程序定义。也可使用 `EXIT_SUCCESS/EXIT_FAILURE`。 |

## 12.2 `argc` 与 `argv`

假设程序名为 `show_args`，命令行为：

```text
show_args red "green apple" 42
```

典型参数关系如下：

```text
argc == 4
argv[0] -> "show_args"
argv[1] -> "red"
argv[2] -> "green apple"
argv[3] -> "42"
argv[4] == NULL
```

引号通常由命令行环境用于把包含空格的文本组织成一个参数；程序接收到的字符串一般不含外层引号。具体转义规则由 shell 或运行环境决定。

![图 12-01：argc、argv 参数字符串与空指针哨兵](<assets/图12-01_argc与argv.png>)

```c
#include <stdio.h>

int main(int argc, char *argv[])
{
    printf("argc = %d\n", argc);

    for (int i = 0; i < argc; ++i) {
        printf("argv[%d] = \"%s\"\n", i, argv[i]);
    }

    printf("sentinel is %s\n",
           argv[argc] == NULL ? "NULL" : "not NULL");
    return 0;
}
```

注意：

- `argc` 非负；在通常的托管环境中至少为 1。
- 若 `argc > 0`，`argv[0]` 表示程序调用名；实现无法提供时它可以是空字符串，不要假定它一定是完整路径。
- `argv[argc]` 是空指针，可作为遍历终止哨兵。
- 每个参数都是字符串，不会因为内容像 `42` 就自动变为整数。
- 读取 `argv[index]` 前先证明 `index < argc`。

## 12.3 安全转换命令行整数

`atoi` 无法可靠区分合法的零和转换失败。应使用 `strtol`，同时检查：

1. 是否至少转换了一个字符；
2. 是否完整消费字符串；
3. 是否发生范围错误；
4. 结果是否适合目标类型。

```c
#include <errno.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>

int parse_int(const char *text, int *out)
{
    char *end = NULL;
    long value;

    if (text == NULL || out == NULL || *text == '\0') {
        return 0;
    }

    errno = 0;
    value = strtol(text, &end, 10);

    if (errno == ERANGE || *end != '\0' ||
        value < INT_MIN || value > INT_MAX) {
        return 0;
    }

    *out = (int)value;
    return 1;
}

int main(int argc, char *argv[])
{
    int left;
    int right;

    if (argc != 3 ||
        !parse_int(argv[1], &left) ||
        !parse_int(argv[2], &right)) {
        fprintf(stderr, "usage: %s INTEGER INTEGER\n",
                argc > 0 ? argv[0] : "add");
        return 2;
    }

    long long sum = (long long)left + (long long)right;
    printf("%lld\n", sum);
    return 0;
}
```

示例：

```text
add 17 25
42
```

`parse_int(text, out)` 中，`text` 是只读输入字符串，`out` 是非空输出参数；成功返回 1 并写入 `*out`，失败返回 0 且调用者不应使用旧结果。本例先把两个 `int` 转成 `long long` 再相加，避免 `int` 加法溢出。若业务要求结果仍必须落在 `int`，还要检查 `INT_MIN <= sum && sum <= INT_MAX`。

## 12.4 Visual Studio 2019：安装与创建 C 项目

课程资料以 Visual Studio 2019 为例。不同版本的菜单文字可能略有变化，但工作流一致。

### 12.4.1 安装

1. 运行 Visual Studio Installer。
2. 选择“使用 C++ 的桌面开发”（Desktop development with C++）工作负载。
3. 确认安装 MSVC 编译工具和 Windows SDK。
4. 完成安装后启动 Visual Studio。

Visual Studio 把 C 编译器放在 C++ 桌面开发工作负载中，因此安装界面没有单独的“C 语言”工作负载是正常现象。

### 12.4.2 创建空项目

1. 选择“创建新项目”。
2. 选择“空项目”，语言筛选可选 C++。
3. 指定项目名和保存位置。
4. 在解决方案资源管理器中右击“源文件”→“添加”→“新建项”。
5. 把源文件扩展名明确写成 `.c`，例如 `main.c`。
6. 写入 `main`，生成并运行。

MSVC 通常根据文件扩展名选择语言：`.c` 按 C 编译，`.cpp` 按 C++ 编译。若课程文件使用 `.cpp`，其中的 C 风格代码仍可能运行，但会受到 C++ 规则约束，不能据此判断代码是标准 C。

### 12.4.3 预编译头与安全函数提示

空项目通常不要求预编译头。若从其他模板创建项目并出现缺少 `pch.h` 的错误，可选择：

- 按模板要求在源文件第一行包含 `pch.h`；或
- 在项目属性中关闭该源文件/项目的预编译头。

MSVC 可能对 `scanf`、`strcpy` 等标准函数发出安全提示。学习时有三种合理处理方式：

- 使用标准接口并严格检查容量；
- 在确实理解参数规则后使用 MSVC 的 `_s` 接口；
- 对需要可移植的教学代码，在项目层面统一处理警告策略。

不要在每个源文件中随意屏蔽全部警告。警告是发现类型、格式符、转换和越界问题的重要线索。

## 12.5 生成、运行与定位编译错误

一个稳定的排错顺序是：

1. 先看“错误列表”中的第一条编译错误；
2. 双击跳到对应文件和行；
3. 同时查看“输出”窗口中的完整编译器消息；
4. 修正最前面的根因后重新生成；
5. 编译成功但链接失败时，检查函数是否只有声明没有定义、源文件是否加入项目、库是否正确配置；
6. 成功生成后再处理运行期行为。

常见阶段：

| 阶段 | 典型问题 |
|---|---|
| 预处理 | 头文件找不到、条件宏错误 |
| 编译 | 语法、类型、格式符、未声明标识符 |
| 链接 | 重复定义、无法解析的外部符号、库缺失 |
| 运行 | 输入错误、越界、空指针、资源泄漏 |

只看到“程序无法运行”还不够；必须先判断失败发生在哪个阶段。

## 12.6 断点与单步调试

### 12.6.1 常用动作

| 动作 | 常见快捷键 | 回答的问题 |
|---|---|---|
| 设置/取消断点 | `F9` | 程序应在哪里暂停 |
| 继续运行 | `F5` | 下一个断点或异常在哪里 |
| 逐过程 | `F10` | 当前函数下一条语句怎样变化 |
| 逐语句 | `F11` | 被调用函数内部怎样执行 |
| 跳出函数 | `Shift+F11` | 当前函数返回到哪里 |

快捷键可能随键盘布局和设置改变，应以菜单中显示的绑定为准。

### 12.6.2 多断点和条件断点

多个断点适合比较同一变量在不同阶段的状态。循环次数很大时，不必停数百次：

1. 在断点上打开设置；
2. 添加条件，例如 `i == 99`；
3. 或设置命中次数；
4. 继续运行到目标迭代。

条件表达式不应依赖会改变程序状态的副作用。

### 12.6.3 调试窗口各自的职责

| 窗口 | 用途 |
|---|---|
| 局部变量/自动窗口 | 查看当前栈帧附近的变量 |
| 监视 | 持续观察自选表达式，如 `p`、`*p`、`arr[i]` |
| 调用堆栈 | 查看谁调用了当前函数，并切换栈帧 |
| 内存 | 按地址查看原始字节，验证大小端、数组连续性和指针位置 |
| 断点 | 集中启用、禁用、删除和配置断点 |

观察指针时至少同时监视：

```text
p
*p
p + 1
```

第一项是地址，第二项是该地址对应的对象，第三项显示指向类型如何决定移动步长。

## 12.7 调用栈与实参求值

逐语句进入函数后，调用堆栈会显示当前函数和调用者。切换到上层栈帧可以查看调用点的局部变量，但不要把“调试器当前显示顺序”误认为 C 标准规定的实参求值顺序。

```c
f(g(), h());
```

C 语言并不普遍保证 `g()` 与 `h()` 谁先执行。若顺序重要，应拆开：

```c
int g_value = g();
int h_value = h();
f(g_value, h_value);
```

这也是课堂“调用栈”截图需要与语言标准规则配合理解的地方。

## 12.8 提取函数与项目文件管理

### 12.8.1 提取函数

IDE 若提供“提取函数”快速操作，可用于生成初始结构，但仍要人工检查：

- 输入值应成为参数还是全局变量；
- 结果应通过返回值还是输出参数传回；
- 数组参数是否同时传入长度；
- 新函数是否只承担一个职责；
- 声明是否放入合适的头文件。

没有自动操作时，手工步骤是：

1. 选定一段职责完整的流程；
2. 列出它读取和修改的外部变量；
3. 设计参数与返回值；
4. 在调用前声明函数；
5. 把代码移入函数定义；
6. 用原有输入验证重构前后行为一致。

### 12.8.2 头文件与源文件

- 对外声明放在 `.h`。
- 函数和对象定义放在 `.c`。
- 每个头文件使用包含保护或 `#pragma once`。
- 不要仅把文件复制到磁盘目录；还要确认它已加入当前项目并参与生成。
- 移动或重命名文件后，检查 `#include` 路径和项目筛选器。
- 解决方案资源管理器中的“筛选器”主要用于视图整理，不一定等同于磁盘上的真实目录。

## 12.9 补充练习索引与参考实现

本节对应原始代码目录中没有以独立小节出现的案例。它们大多复用前文知识，因此采用“考点 + 改进版参考实现”的形式。

### 12.9.1 天平判断

考点：二选一扩展为三种关系，不能漏掉相等。

```c
#include <stdio.h>

int main(void)
{
    int left;
    int right;

    if (scanf("%d%d", &left, &right) != 2) {
        return 1;
    }

    if (left > right) {
        puts("左边重");
    } else if (left < right) {
        puts("右边重");
    } else {
        puts("平衡");
    }
    return 0;
}
```

### 12.9.2 打印八个条目

原练习用八条 `printf` 训练顺序结构。数据与流程分离后更便于修改：

```c
#include <stdio.h>

int main(void)
{
    const char *items[] = {
        "条目一", "条目二", "条目三", "条目四",
        "条目五", "条目六", "条目七", "条目八"
    };

    for (size_t i = 0; i < sizeof items / sizeof items[0]; ++i) {
        puts(items[i]);
    }
    return 0;
}
```

若练习目标只是初识顺序结构，逐条 `puts` 并没有错；改成数组循环则是在同一题上继续练习“数据驱动流程”。

### 12.9.3 计算十进制位数

`0` 有一位；负号不是数字位。下面的 `do...while` 同时处理正数、负数和零，且不需要对最小负数取绝对值：

```c
#include <stdio.h>

int main(void)
{
    long long number;
    int digits = 0;

    if (scanf("%lld", &number) != 1) {
        return 1;
    }

    do {
        ++digits;
        number /= 10;
    } while (number != 0);

    printf("%d\n", digits);
    return 0;
}
```

### 12.9.4 `switch` 判断星期

```c
#include <stdio.h>

int main(void)
{
    int day;
    const char *name;

    if (scanf("%d", &day) != 1) {
        return 1;
    }

    switch (day) {
    case 1: name = "星期一"; break;
    case 2: name = "星期二"; break;
    case 3: name = "星期三"; break;
    case 4: name = "星期四"; break;
    case 5: name = "星期五"; break;
    case 6: name = "星期六"; break;
    case 7: name = "星期日"; break;
    default:
        puts("请输入 1 到 7");
        return 1;
    }

    puts(name);
    return 0;
}
```

### 12.9.5 求 1 到 100 的和

循环写法用于训练累加器：

```c
int sum = 0;
for (int i = 1; i <= 100; ++i) {
    sum += i;
}
printf("%d\n", sum);
```

输出：

```text
5050
```

也可用等差数列公式 `100 * 101 / 2`。两种写法表达的是同一数学关系：循环适合一般化过程，公式在已证明时更直接。

### 12.9.6 随机数组筛选

原文件名虽为 `随机数去重.c`，实际代码内容是“生成随机整数，把符合条件的值变为负数，再把负数筛选到新数组”，并不是去重算法。该知识点已经完整收入第 3 章 4.8 节“随机筛选：把负数复制到新数组”。

真正的去重任务需要明确：

- 是否保留第一次出现顺序；
- 输入是否已经排序；
- 能否修改原数组；
- 新数组容量；
- 时间和空间复杂度要求。

无序小数组可在写入结果前做线性查重，时间复杂度为 \(O(n^2)\)；更大数据可先排序或使用哈希结构。

### 12.9.7 结构体工资表排序

考点：结构体数组、多字段排序、总工资计算和稳定的比较规则。

```c
#include <stdio.h>

typedef struct Salary {
    int basic;
    int performance;
    int bonus;
    int adjustment;
} Salary;

int total_salary(const Salary *salary)
{
    return salary->basic +
           salary->performance +
           salary->bonus +
           salary->adjustment;
}

int comes_after(const Salary *left, const Salary *right)
{
    if (left->basic != right->basic) {
        return left->basic > right->basic;
    }
    return left->performance > right->performance;
}

int main(void)
{
    Salary salaries[] = {
        {8000, 5200, 1000, -2000},
        {5000, 4230,  800,   500},
        {9000, 3200, 2300,   260},
        {5000,  500,  100,  -200},
        {5000, 3800,  500,   500},
        {8000, 1350,  500,  -600}
    };
    const size_t count = sizeof salaries / sizeof salaries[0];

    for (size_t i = 0; i < count; ++i) {
        for (size_t j = i + 1; j < count; ++j) {
            if (comes_after(&salaries[i], &salaries[j])) {
                Salary temp = salaries[i];
                salaries[i] = salaries[j];
                salaries[j] = temp;
            }
        }
    }

    puts(" basic   performance bonus adjustment total");
    for (size_t i = 0; i < count; ++i) {
        printf("%6d %11d %5d %10d %5d\n",
               salaries[i].basic,
               salaries[i].performance,
               salaries[i].bonus,
               salaries[i].adjustment,
               total_salary(&salaries[i]));
    }
    return 0;
}
```

`&&` 的优先级高于 `||`，但多字段比较仍应加括号或提取为比较函数，避免读者误判逻辑。内层循环从 `i + 1` 开始即可，不需要让元素与自己比较。

## 12.10 本章小结与易错点

| 易错点 | 正确理解 |
|---|---|
| 直接读取 `argv[1]` | 先检查 `argc` |
| 用 `atoi` 判断参数是否合法 | 用 `strtol` 检查结束位置和范围 |
| 认为 VS 的“C++ 空项目”不能写 C | 使用 `.c` 扩展名并确认编译选项 |
| 只看错误列表最后一条 | 从第一条根因错误开始修复 |
| 把 `F10` 和 `F11` 混用 | 前者越过调用，后者进入被调用函数 |
| 认为调试器显示顺序就是语言保证 | 以 C 标准规则为准，重要顺序显式拆语句 |
| 文件存在磁盘上就一定参与构建 | 确认文件已加入项目和生成配置 |
| 十进制位数循环漏掉零 | 用 `do...while` 至少执行一次 |
| 多字段比较依赖读者记优先级 | 加括号或提取比较函数 |

---

## 12.11 综合题库（含答案与解析）

### 12.11.1 Visual Studio 快捷键

| 题目 | 答案 | 解析 |
|---|---|---|
| VS2019 中 C 代码编写完成后，怎样“开始执行（不调试）”？ | `Ctrl+F5`。 | 它生成并直接运行程序，不附加调试器；需要断点调试时使用 `F5`。 |
| `F10` 与 `F11` 的区别是什么？ | `F10` 单步跳过函数调用，`F11` 单步进入函数。 | 两者都会执行当前语句，但下一次暂停位置不同。 |

### 12.11.2 创建并设置启动项目

作业多次要求在同一解决方案下创建 `Todo_C_1_1`、`Todo_C_1_2`、`Todo_C_1_4` 等项目，并设为启动项目。统一答案如下：

1. 在解决方案资源管理器中右击解决方案，选择“添加 → 新建项目”。
2. 选择“空项目”，填写题目指定的项目名和位置。
3. 右击新项目的“源文件”，选择“添加 → 新建项”，创建扩展名为 `.c` 的文件。
4. 一个项目只能保留一个参与链接的 `main`。若每道题都建立一个源文件，应将其它含 `main` 的文件“从生成中排除”，或为每题建立独立项目。
5. 右击需要运行的项目，选择“设为启动项目”。
6. 使用 `Ctrl+F5` 验证普通运行；需要观察变量、调用栈或内存时设置断点并按 `F5`。

**解析：**“设为启动项目”只决定解决方案运行哪个项目，不会自动解决同一项目中多个 `main` 的重复定义。磁盘上已有文件也不代表它已加入当前项目和生成配置。
