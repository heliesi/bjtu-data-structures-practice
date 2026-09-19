# Lab 1：从一根指针开始——单链表、集合与猴子选大王

> 这是一份陪你从零写代码的实验指导。每一题按“读懂目标 → 先预测 → 动手实现 → 测试 → 解释原因”推进。第一次学习链表时，不需要一次读完全文；做到一个检查点，就停下来运行程序。

本讲义使用 **C++17**，默认你认识变量、`if`、`while` 和函数。指针、引用、头节点和内存释放都会在用到之前解释。核心算法留给你实现；代码框中的 `TODO` 表示需要你填写的部分。提示可以展开，建议先自己画图、尝试几分钟，再看第一层提示。

本文参考了 [CS61A 链表 lab](https://www-inst.eecs.berkeley.edu/~cs61a/fa23/lab/lab08/) 中的“预测运行结果、代码骨架、逐题测试”，以及 [CS61B 链表 lab](https://fa26.datastructur.es/labs/lab03/) 中的“方框箭头图、逐行观察、局部测试”。题目、C++ 示例和测试数据均为本次实验重新设计。更多参考链接及各自用途见文末；不需要安装它们的 Python/Java 测试工具。

## 导航与完成路线

| 阶段 | 你会完成什么 | 完成标志 |
| --- | --- | --- |
| [A：准备与指针](#part-a) | Q0～Q2 | 能解释指针指向哪里，能画出三个节点 |
| [B：单链表基础](#part-b) | Q3～Q6 | 能创建、追加、遍历和释放链表 |
| [C：集合运算](#part-c) | Q7～Q11 | 能排序、去重，求并集、交集和差集 |
| [D：循环链表](#part-d) | Q12～Q15 | 能建环、删节点，求猴王 |
| [E：整理与扩展](#part-e) | Q16～Q17 | 有可演示程序，按需完成随机密码版 |

建议分几次完成：第一次到 Q6，第二次到 Q11，第三次到 Q16。Q17 是图片中的加分扩展。耗时取决于你对 C++ 的熟悉程度，不必用速度衡量学习效果。

### 检测覆盖表

为了避免“写完了但不知道是否真的测到了”，本讲义把检测分成两类：

| 题目 | 检测方式 | 是否有可运行检查 |
| --- | --- | --- |
| Q0 | 编译并运行 `Hello, Lab 1!` | 有 |
| Q1～Q2 | 预测、画图、口头解释 | 手工检查 |
| Q3 | `assert` 检查头节点和空链表 | 有 |
| Q4 | 检查追加后的节点顺序和尾指针 | 有 |
| Q5 | 检查空表、一个节点、多个节点的长度和打印 | 有，见 Q5；释放在 Q6 补上 |
| Q6 | 检查清空、重新使用、销毁和重复销毁 | 有 |
| Q7 | 检查升序和排序前后长度 | 有，见 Q7 |
| Q8 | 检查连续重复值和重复调用 | 有 |
| Q9～Q11 | `CheckSets` 分别检查并集、交集、差集及输入不变 | 有 |
| Q12 | 检查空圈、自环、编号顺序和只走一圈 | 有 |
| Q13 | 检查删除 1 号、连续删除和释放最后节点 | 有 |
| Q14 | 纸上模拟报数位置 | 手工检查 |
| Q15 | 检查多个 N、M、非法参数和出圈顺序 | 有 |
| Q16 | 按固定输入手工演示端到端流程 | 手工检查 |
| Q17 | 固定密码的确定性测试，随机版检查范围和释放 | 有，见 Q17 |

自动检测并不能代替画图：它告诉你结果是否符合预期，画图才能帮助你理解哪个指针被改变了。

## 先把老师的要求读准确

两张图片描述的是同一个专题下的两个题目：

| 图片中的要求 | 本讲义对应内容 |
| --- | --- |
| 用 C 或 C++ 实现 | 使用 C++17，手写节点和连接操作 |
| 单链表表示集合 | Q3～Q8 |
| `Sort(slink *L)`：数据按值递增排序 | Q7；排序后允许相等值，再由 Q8 去重 |
| `Union`：两个有序集合的并集 | Q9 |
| 交集函数，图片拼写看起来为 `InerSect` | Q10，本文统一使用 `InerSect` |
| `Subs`：两个有序集合的差集 | Q11，明确计算第一个集合减第二个集合 |
| 输入正整数 N、M，循环链表模拟猴子选王 | Q12～Q15 |
| 输出猴王编号 | Q15～Q16 |
| `CircularLinkedList.cpp` 存基本操作，`main.cpp` 存主函数及选王函数，至少一个自定义头文件 | Q12、Q16，保持图片中的文件名 |
| 每只猴子额外保存随机生成的密码，出圈后改变下一轮报数上限 | Q17 |

第一题评分：基本功能 80%，演示与讲解 10%，代码规范与注释 10%。第二题评分：基本功能 70%，演示与讲解 10%，随机密码扩展 10%，代码规范与注释 10%。多文件结构的明确要求出现在第二张图片；本讲义为两个题目都采用多文件组织，便于学习与复用。

图片未明确规定“是否有头节点”“重复输入如何处理”“并交差是否破坏原集合”。我们作如下实现约定，后文都遵守它们：

- 单链表使用一个**不存有效集合元素的头节点**；循环链表不使用这种额外节点。
- 输入可以重复；先排序，再删除重复节点，使它成为集合。
- 并、交、差都产生独立的新结果，不改变 A、B。
- 差集 `Subs(A, B, C)` 表示 `C = A - B`，不是对称差，也不是 `B - A`。
- 基础版约瑟夫问题每轮从下一只猴子重新报 1，报到 M 的猴子退出。

这些是本讲义的设计选择。若老师另发了固定代码骨架或接口，应以那份明确要求调整。本文不使用 `std::list` 或 `std::set` 代替你的链表实现。

<a id="part-a"></a>

## A：先能运行，再理解指针

### Q0 · 让第一段程序运行起来

**目标：** 先确认“编辑 → 编译 → 运行”这条路通了。

在终端进入当前项目中的 `lab1` 文件夹。若你已经在项目根目录，只需执行：

```bash
cd lab1
clang++ --version
```

这里以 macOS 的 `clang++` 为例；已有 `g++` 的环境可把后文命令中的 `clang++` 换成 `g++`。如果提示找不到编译器，先配置 C++ 编译环境，再继续写链表。

在编辑器中新建 `main.cpp`，输入：

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, Lab 1!\n";
    return 0;
}
```

在 `lab1` 目录运行：

```bash
clang++ -std=c++17 -Wall -Wextra -pedantic -g main.cpp -o lab1_demo
./lab1_demo
```

其中 `-std=c++17` 选择语言标准，几个 `-W` 选项帮助发现可疑写法，`-g` 保留调试信息，`-o` 后面是生成的程序名称。

**检查点 A0：** 看到 `Hello, Lab 1!`。以后修改代码，需要重新编译后再运行；只运行旧程序不会自动包含新修改。

### Q1 · 节点是盒子，指针是地址

想象你有三张卡片，上面分别写着 10、20、30。除了数字，每张卡片还写了“下一张卡片放在哪里”。只要知道第一张的位置，就能顺着找到全部卡片。

在代码中，一张卡片就是一个节点：

```cpp
struct slink {
    int data;
    slink* next;
};
```

`struct` 把相关数据放在一起。`slink` 是我们定义的类型名；一个 `slink` 节点包含一个整数 `data`，以及一个保存“下一个节点地址”的指针 `next`。

```text
一个节点： [ data | next ]

一条普通单链表：
head ──→ [10 | •] ──→ [20 | •] ──→ [30 | nullptr]
```

箭头表示“保存了目标节点的地址”，不表示节点在内存中挨着放。`nullptr` 是空指针，表示没有指向任何节点。

先区分这几个表达式：

| 写法 | 含义 |
| --- | --- |
| `slink node{10, nullptr};` | 创建一个节点对象 |
| `slink* p = &node;` | `&node` 取得节点地址，`p` 保存这个地址 |
| `p->data` | 沿着 p 找到节点，访问它的 data |
| `p->next` | 沿着 p 找到节点，访问它的 next |
| `p = p->next;` | 改变 p 自己保存的地址，使 p 指向下一个节点 |
| `p->next = q;` | 改变节点内部的连接，使下一跳变成 q |

`p->data` 也可以写成 `(*p).data`：`*p` 找到地址对应的对象，`.` 访问对象的成员。遇到 `slink* p` 中的星号，它是在声明“p 是指针”；遇到 `*p`，它是在使用指针访问对象。

**先预测，再运行。** 暂时把以下代码放在 `main` 中，结构体定义放在 `main` 前面：

```cpp
slink a{10, nullptr};
slink b{20, nullptr};
a.next = &b;

slink* p = &a;
slink* q = p;
q->data = 99;
p = p->next;

std::cout << a.data << '\n';    // ① ?
std::cout << p->data << '\n';   // ② ?
std::cout << q->data << '\n';   // ③ ?
std::cout << (q == &a) << '\n'; // ④ ? 默认布尔输出为 0 或 1
```

<details>
<summary>预测题解析：写下答案以后再展开</summary>

四行输出依次是 `99`、`20`、`99`、`1`。

`q = p` 复制的是地址，没有复制节点。开始时 p、q 都指向 a，所以通过 q 修改数据，也就修改了 a。随后 `p = p->next` 只移动 p；q 仍然指向 a，a 到 b 的连接也没有改变。

```text
q ──→ [99 | •] ──→ [20 | nullptr]
                     ↑
                     p
```

</details>

**检查点 A1：** 用自己的话解释：为什么移动 p 不会带着 q 一起移动？为什么修改 `q->data` 却会改变 a？

### Q2 · 单链表与循环链表，到底差在哪里

“单”表示每个节点只有一个向后连接；“循环”表示最后一个节点连回起点。这是两个不同角度。因此，**循环链表也可以是单向的**。这次实验比较的是“普通单向链表”和“单向循环链表”。

```text
普通单向链表：
first ──→ [1] ──→ [2] ──→ [3] ──→ nullptr

单向循环链表：
first ──→ [1] ──→ [2] ──→ [3]
           ↑                │
           └────────────────┘
```

| 问题 | 普通单向链表 | 单向循环链表 |
| --- | --- | --- |
| 尾节点的 next | `nullptr` | 第一个节点 |
| 一趟遍历何时结束 | 指针到达 `nullptr` | 走了一圈，回到起点 |
| 能直接沿 next 回到前一个节点吗 | 不能 | 不能；绕一圈也需要多步 |
| 适合本实验的哪一部分 | 表示集合 | 反复报数、出圈 |

循环链表并不是“把循环条件删掉”。它只是把终点换成了起点，所以需要新的停止条件。

顺便对比数组：数组可以按下标直接定位，链表通常要逐个走过去。已知插入位置的前驱时，链表改连接是 O(1)；但**寻找这个前驱**可能需要 O(n)。单链表删除也一样：一般需要前驱，不能笼统地说“知道待删节点就一定能 O(1) 删除”。

这里的 O(n) 可以先理解为“工作量随节点数大致线性增长”，O(1) 表示操作步数不随链表长度增长。

**纸上练习：** 画出只有一个节点的两种链表。普通链表的 `next` 是什么？循环链表的 `next` 又指向谁？答案会在 Q12 用到。

<a id="part-b"></a>

## B：写出可以放心使用的单链表

### Q3 · 选定表示方式，创建空链表

普通单链表有“带头节点”和“不带头节点”两种常见写法。前面的概念图没有额外头节点；从本题起，集合部分统一增加一个头节点：

```text
空链表：
L ──→ [头节点 | nullptr]

保存 10、20 的链表：
L ──→ [头节点 | •] ──→ [10 | •] ──→ [20 | nullptr]
```

头节点是真实分配的节点，但其 `data` 不算集合元素。我们把它初始化为 0 只是方便；**0 仍然可以是后面的合法数据元素**。判断是否为头节点看位置，不看数值。

这样做的好处是：即使集合为空，L 仍指向一个稳定的入口。第一个数据节点总是 `L->next`。向开头插入、删除第一个数据节点时，可以统一修改 `L->next`，也容易对应图片中的 `slink* L` 接口。

先记住两个不同状态：

- `L != nullptr` 且 `L->next == nullptr`：已初始化的空链表，可以正常使用。
- `L == nullptr`：还没有链表对象，或者整条链表已经销毁。

在 `lab1` 下新建 `SinglyLinkedList.h`：

```cpp
#pragma once

struct slink {
    int data;
    slink* next;
};

slink* InitList();
void Append(slink* L, int value);
void PrintList(const slink* L);
int Length(const slink* L);
void ClearList(slink* L);
void DestroyList(slink*& L);

void Sort(slink* L);
void Unique(slink* L);

void Union(const slink* la, const slink* lb, slink* lc);
void InerSect(const slink* la, const slink* lb, slink* lc);
void Subs(const slink* la, const slink* lb, slink* lc);
```

`.h` 是接口清单，告诉其他文件“有什么函数、怎么调用”；`.cpp` 放函数实现。`#pragma once` 避免同一个头文件重复引入。`const slink*` 表示不能通过这个指针直接修改它指向的节点；它不是整条链表的深度只读保护，函数仍需遵守“不修改输入”的约定。

**接口必须逐字一致。** C++ 区分大小写，`ClearList` 和 `clearList` 是两个不同的函数；本文统一使用大写开头的 `ClearList`。如果你的代码写成了 `clearList`，请同时修改头文件、函数定义和所有调用处。`PrintList` 的声明和定义也必须都写成 `const slink*`：

```cpp
// SinglyLinkedList.h
void PrintList(const slink* L);
void ClearList(slink* L);

// SinglyLinkedList.cpp
void PrintList(const slink* L) {
    // ...
}

void ClearList(slink* L) {
    // ...
}
```

只在头文件里声明，或只修改函数名的一处，都不能算完成；编译时会出现“未定义函数”或“声明与定义冲突”。

新建 `SinglyLinkedList.cpp`，先只完成这个小骨架：

```cpp
#include "SinglyLinkedList.h"
#include <iostream>

slink* InitList() {
    // TODO：用 new 创建头节点，data 为 0，next 为 nullptr；返回其地址。
    return nullptr; // 占位，完成时替换
}
```

`new slink{值, 下一跳}` 会申请一个节点并返回地址。这个节点不会因为函数返回就自动消失；用完后要用 `delete` 释放。不要返回函数内部普通局部节点的地址，因为局部节点会在函数结束时失效。

把 `main.cpp` 改成：

```cpp
#include "SinglyLinkedList.h"
#include <cassert>
#include <iostream>

int main() {
    slink* L = InitList();
    assert(L != nullptr);
    assert(L->next == nullptr);
    delete L; // 目前仅有一个头节点，暂时这样清理
    L = nullptr;
    std::cout << "Q3 passed\n";
}
```

`assert(条件)` 的意思是“这里必须满足这个条件”；不满足就中止并提示位置。此处保留默认设置，不使用关闭断言的 `-DNDEBUG`。

从现在到 Q11，每次用这条命令编译：

```bash
clang++ -std=c++17 -Wall -Wextra -pedantic -g SinglyLinkedList.cpp main.cpp -o lab1_demo
./lab1_demo
```

头文件可以提前声明后续函数；只要当前还没有调用它们，就可以暂时不写定义。

**检查点 B0：** 能通过两个断言，并说清楚“空链表不等于 L 为空指针”。

### Q4 · 追加一个数据节点

**实现：** `void Append(slink* L, int value)`，把 value 加到末尾。不排序，不去重，也不改变 L 指向的头节点。

先在纸上画“空链表追加 7”和“已有 7 时追加 2”。无论哪一种，你都在找一个 `next == nullptr` 的节点，然后把它的 next 连向新节点。

```cpp
void Append(slink* L, int value) {
    slink* tail = L;
    // TODO 1：沿 next 找到尾节点。
    // TODO 2：创建新节点，新节点 next 为 nullptr。
    // TODO 3：连接到 tail 后面。
}
```

<details>
<summary>提示 1：应该从哪里开始找？</summary>

从头节点 L 开始。空链表的头节点同时也是尾节点，这样空表也能走同一套逻辑。

</details>

<details>
<summary>提示 2：什么时候停止移动？</summary>

当 `tail->next == nullptr` 时，tail 就是最后一个节点。不要让 tail 自己变成 `nullptr`，否则无法在它后面连接新节点。

</details>

在 Q3 的 `main` 中，删掉临时 `delete L` 及置空操作，在创建 L 后测试：

```cpp
Append(L, 7);
Append(L, 2);
Append(L, 5);
assert(L->next->data == 7);
assert(L->next->next->data == 2);
assert(L->next->next->next->data == 5);
assert(L->next->next->next->next == nullptr);
```

这一小步可以先测试连接，马上在 Q6 补齐完整释放。不要以为 `delete L` 会自动释放后面的三个节点：它只释放 L 指向的一个节点。

**检查点 B1：** 画出运行后的链表，包括头节点。说明为什么头节点的数据没有变成 7。

### Q5 · 遍历：让一个临时指针走完整条链表

**实现两个函数：** `PrintList` 输出所有数据，`Length` 返回数据节点个数，均不计头节点。

约定输出格式：

```text
空表：{}
非空表：{7, 2, 5}
```

遍历时使用临时指针 p：

```cpp
int Length(const slink* L) {
    int count = 0;
    const slink* p = L->next;
    // TODO：只要 p 指向数据节点，计数并移动。
    return count;
}
```

`PrintList` 使用相同的移动方式，把“加一”替换成“输出数据”。格式控制可以设置 `bool first = true`：如果当前不是第一个元素，先输出逗号，再输出数据。

实现 `PrintList` 时，函数头要和头文件完全一致：

```cpp
void PrintList(const slink* L) {
    // 只读取 L，不修改链表
}
```

不要在 `.cpp` 中写成 `void PrintList(slink* L)`。这会形成另一个重载函数，头文件中的 `const` 版本仍然没有实现；如果 main 调用的是 const 版本，链接时可能出现未定义函数。

**先预测：** 对于含三个数据节点的链表，下面两种条件分别会处理几个节点？

```cpp
// p 起初指向第一个数据节点
while (p != nullptr) { /* 处理并移动 p */ }
while (p->next != nullptr) { /* 处理并移动 p */ }
```

<details>
<summary>解析</summary>

第一种会处理 3 个节点；第二种只处理 2 个，因为遇到尾节点就停了。空表时，第二种还会访问空指针的 next，这是未定义行为。遍历“所有数据”与“寻找尾节点”的循环条件不同。

</details>

在 main 中加入下面的检测。它同时覆盖空表、一个节点、多个节点，以及“打印之后链表仍然存在”：

```cpp
{
    slink* empty = InitList();
    assert(Length(empty) == 0);
    PrintList(empty); // 预期：{}
    // DestroyList 属于 Q6；完成 Q6 后再取消下一行注释。
    // DestroyList(empty);
}

{
    slink* L = InitList();
    Append(L, 7);
    assert(Length(L) == 1);
    PrintList(L); // 预期：{7}

    Append(L, 2);
    Append(L, 5);
    assert(Length(L) == 3);
    PrintList(L); // 预期：{7, 2, 5}
    const slink* readOnly = L;
    PrintList(readOnly); // 预期仍是 {7, 2, 5}，同时检查 const 接口

    // DestroyList 属于 Q6；完成 Q6 后再取消下一行注释。
    // DestroyList(L);
}
```

Q5 的长度和打印断言可以在 Q5 完成后检查；这段完整代码中的 `DestroyList` 调用依赖 Q6。若你还没有完成 Q6，可以先把两行 `DestroyList` 暂时注释，或完成 Q6 后再运行整段测试。长期运行测试时不要保留注释，否则会泄漏本次测试创建的节点。

如果空表的 `Length` 或 `PrintList` 崩溃，优先检查是否在确认 `p != nullptr` 之前访问了 `p->next` 或 `p->data`。

**检查点 B2：** `PrintList` 没有输出头节点的 0，`Length` 没有多算一个节点。

### Q6 · 释放内存，并理解 `slink*&`

**实现：** `ClearList` 释放所有数据节点，但保留头节点；`DestroyList` 再释放头节点，并把调用者的 L 置为 `nullptr`。

```text
ClearList 前：L → [头] → [7] → [2] → nullptr
ClearList 后：L → [头] → nullptr
DestroyList 后：L = nullptr
```

删除节点之前，必须保留继续前进的路径。这里可以反复删除第一个数据节点：

```cpp
void ClearList(slink* L) {
    // 约定：L 必须是有效头节点。
    while (L->next != nullptr) {
        slink* victim = L->next;
        // TODO 1：让头节点越过 victim，连向 victim 的后继。
        // TODO 2：释放 victim。
    }
}

void DestroyList(slink*& L) {
    // TODO：若 L 已为 nullptr，直接返回。
    // TODO：清空数据节点，释放头节点，把 L 置为 nullptr。
}
```

不要写 `delete p; p = p->next;`：节点已释放，再读取其中的 next，就是访问已失效的内存。即便某次“看起来正常”，也不正确。

为什么两个参数长得不一样？

- `slink* L`：函数拿到地址的副本。可以沿地址修改节点，但在函数里给 L 重新赋值，不会改变外面的指针变量。
- `slink*& L`：函数拿到外面指针变量的引用。可以把它理解成外面变量的另一个名字，给它赋 `nullptr` 会改变外面的变量。

`Append`、`Sort` 可以通过稳定的头节点操作；`DestroyList` 需要修改外面的入口，所以使用引用。这里的 `&` 是引用声明，与表达式 `&node` 的取地址含义不同。

在 main 中追加下面的测试：

```cpp
ClearList(L);
assert(L != nullptr);
assert(L->next == nullptr);
assert(Length(L) == 0);
Append(L, 42); // 清空后仍可继续用
assert(Length(L) == 1);
DestroyList(L);
assert(L == nullptr);
DestroyList(L); // 约定：重复销毁空指针安全
```

**检查点 B3：** 创建、追加、遍历、清空、再次追加、销毁整条流程全部通过。完成本题后，后续每个测试都要释放自己创建的链表。

<a id="part-c"></a>

## C：把链表变成集合

### 先读：集合的“没有顺序”与存储的“有序”并不矛盾

数学上 `{2, 5, 7}` 与 `{7, 2, 5}` 是同一个集合。但程序可以选择按升序存储它们，方便比较。集合中的一个值只出现一次，所以重复输入需要处理。

本实验的处理流程是：

```text
输入 7, 2, 5, 2
    ↓ Append
链表 7 → 2 → 5 → 2
    ↓ Sort
链表 2 → 2 → 5 → 7
    ↓ Unique
集合 2 → 5 → 7
```

`Sort` 只负责排序，保留节点数；`Unique` 才负责删除重复节点。集合运算的输入必须已经完成这两步。把职责分开，出错时更容易定位。

### Q7 · 排序：先让最小的数站到前面

**实现：** `void Sort(slink* L)`。

本题先采用**选择排序并交换 data**。图片只要求按值排序，没有规定必须重新连接节点。交换整数容易入门；真正重连节点的插入排序放到文末额外练习。

想象你要排队：先从所有人中找出最矮的放在第一位，再从剩下的人中找出最矮的放在第二位。

对链表 `7 → 2 → 5`：

```text
第 1 轮：在 7、2、5 中找最小值 2，与第 1 个节点的数据交换
         2 → 7 → 5
第 2 轮：在 7、5 中找最小值 5，与第 2 个节点的数据交换
         2 → 5 → 7
```

用外层指针 p 表示“这轮要排好的位置”，用内层指针 q 扫描剩余节点，用 `minNode` 记录最小值所在节点。

```cpp
void Sort(slink* L) {
    for (slink* p = L->next; p != nullptr; p = p->next) {
        slink* minNode = p;
        // TODO 1：从 p 后面开始，找到本轮最小值所在的节点。
        // TODO 2：交换 p->data 与 minNode->data。
    }
}
```

<details>
<summary>提示 1：怎样保存当前最小值？</summary>

如果 `q->data < minNode->data`，就把 minNode 改成 q。它记录的是节点地址，不需要新建节点。

</details>

<details>
<summary>提示 2：怎样交换？</summary>

用一个临时 int 保存旧值，再交换两个节点的 data。不要交换 next，也不要让外面的 L 改指向最小数据节点。最小值就在 p 时，交换自己也无妨。

</details>

**测试：** 空表、单节点、已经有序、完全逆序、重复值、负数都要覆盖。完整测试工具见 Q8。

| 输入 | Sort 后 |
| --- | --- |
| `{}` | `{}` |
| `{5}` | `{5}` |
| `{1, 2, 3}` | `{1, 2, 3}` |
| `{3, 2, 1}` | `{1, 2, 3}` |
| `{7, 2, 5, 2}` | `{2, 2, 5, 7}` |
| `{0, -3, 4, -3}` | `{-3, -3, 0, 4}` |

把下面的测试放在 Q7 的实现之后、Q8 的 `ExpectList` 之前。它不依赖 Q8：

```cpp
{
    slink* L = InitList();
    for (int x : {7, 2, 5, 2, -1}) {
        Append(L, x);
    }

    int before = Length(L);
    Sort(L);
    assert(Length(L) == before); // Sort 只排序，不删节点

    const slink* p = L->next;
    while (p != nullptr && p->next != nullptr) {
        assert(p->data <= p->next->data);
        p = p->next;
    }

    DestroyList(L);
}
```

这段检测了“结果有序”和“节点数量没有改变”。它还包含重复值和负数，能避免只用 `{1, 2, 3}` 导致测试过于简单。

**检查点 C0：** 排序后长度没有改变。能解释“已经处理过的位置，保存了这一部分应有的最小值”。这种每轮成立的事实，叫作循环不变式。

### Q8 · 去重：相同的数现在挨在一起了

**实现：** `void Unique(slink* L)`。前提是链表已经升序排序。

```text
之前：头 → 2 → 2 → 2 → 5 → 5 → 7
之后：头 → 2 → 5 → 7
```

让 p 指向一个保留的节点，比较 p 与它的后继。如果相等，就删除后继；如果不等，p 才向后移动。

**思考：** 删除第二个 2 后，p 为什么不能立刻移动？请用三个连续的 2 解释。

<details>
<summary>提示</summary>

删除后，原来的第三个 2 变成了 p 的新后继，仍需继续检查。只有确认后继值不同，才能移动 p。访问 `p->next->data` 前，要同时保证 p 和 p->next 都非空。

本题只修改连接和释放重复节点，不需要申请新节点。

</details>

为了不靠肉眼判断，可以把下面这个**测试辅助函数**完整放进 `main.cpp`，位置在 main 前。它不替你实现算法，只检查结果：

```cpp
#include <initializer_list>

void ExpectList(const slink* L, std::initializer_list<int> expected) {
    assert(L != nullptr);
    const slink* p = L->next;
    for (int value : expected) {
        assert(p != nullptr);       // 实际元素不能少
        assert(p->data == value);   // 数值和顺序要对
        p = p->next;
    }
    assert(p == nullptr);           // 实际元素不能多，也不能连回前面
}
```

`{2, 5, 7}` 在这里是一小组预期值。`for (int value : expected)` 逐个取出这些值，不是你实现链表时需要使用的容器算法。

下面片段可以放进 main。使用一个独立的 `{ ... }` 代码块，避免与前面的变量重名：

```cpp
{
    slink* L = InitList();
    for (int x : {7, 2, 5, 2, 2}) {
        Append(L, x);
    }
    Sort(L);
    ExpectList(L, {2, 2, 2, 5, 7});
    Unique(L);
    ExpectList(L, {2, 5, 7});
    Unique(L); // 再去重一次，结果应不变
    ExpectList(L, {2, 5, 7});
    DestroyList(L);
}
```

再分别测试空表、`{6}`、`{6, 6, 6}`、`{-2, -2, 0, 0, 3}`。

**检查点 C1：** 排序且去重后，相邻数据严格递增。此时才把它交给集合运算。

### Q9 · 并集：两列已经排好序的队伍，合成一列

**实现：** `Union(la, lb, lc)`，把并集写入 lc。

三个集合运算统一遵守以下约定：

- la、lb、lc 都有有效头节点；la、lb 已有序且无重复。
- lc 与两个输入完全独立，不共享任何节点；不要调用 `Union(A, B, A)`。
- 可以重复使用同一个结果链表：函数开始先 `ClearList(lc)`。
- 读取输入中的值，申请新的结果节点；不把 A、B 的原节点接到 C 中。
- 结果有序、无重复，A、B 保持不变。

为什么要独立分配结果节点？如果直接把 B 的剩余链条接到 C 后面，B、C 就会共享节点；将来释放 B 后 C 可能失效，再释放 C 还可能重复释放同一块内存。

先计算：

```text
A = {1, 3, 5}
B = {2, 3, 4}
A ∪ B = {1, 2, 3, 4, 5}
```

给 A、B 各放一个指针 p、q，分别指向当前还没处理的最小元素：

| 情况 | 放入 C 的值 | 移动谁 |
| --- | --- | --- |
| `p->data < q->data` | p 的值 | p |
| `p->data > q->data` | q 的值 | q |
| 两者相等 | 这个值只放一次 | p、q 都移动 |

只要输入各自无重复，这样就不会让结果重复。某个指针走到末尾后，把另一个链表剩余的值逐个复制进结果。

**先做手算：** 在纸上写出下表后四行，再打开提示。

| p 的值 | q 的值 | 本步加入 | 已得到的 C |
| --- | --- | --- | --- |
| 1 | 2 | 1 | `{1}` |
| 3 | 2 | ? | ? |
| 3 | 3 | ? | ? |
| 5 | 4 | ? | ? |
| 5 | 到末尾 | ? | ? |

<details>
<summary>手算核对</summary>

后四步依次加入 2、3、4、5；相等时两个指针一起移动，因此 3 只出现一次。

</details>

**先准备一个 O(1) 尾插小工具。** Q4 的 Append 每次都从头寻找尾节点。集合运算连续添加很多结果时，如果反复这样查找，整体就会变慢。现在用 tail 始终记住结果末尾。

在 `SinglyLinkedList.cpp` 中、集合函数之前定义：

```cpp
static void AppendToTail(slink*& tail, int value) {
    // TODO：在 tail 后申请并连接新节点，然后把 tail 移到新节点。
}
```

这里 tail 必须指向有效节点，且 `tail->next == nullptr`。函数开头的 `static` 表示这个辅助函数只在本文件内使用；不必写入头文件。参数用引用，因为需要更新调用者保存的尾指针。

然后完成：

```cpp
void Union(const slink* la, const slink* lb, slink* lc) {
    ClearList(lc);
    const slink* p = la->next;
    const slink* q = lb->next;
    slink* tail = lc;

    // TODO 1：当 p、q 都非空时，按表格处理三种情况。
    // TODO 2：复制 p 尚未处理的值。
    // TODO 3：复制 q 尚未处理的值。
}
```

注意每次比较前保证两个指针都非空，不要访问 `nullptr->data`。

**测试：** Q11 后提供了三种运算共用的测试函数。现在也可以自己创建 A、B、C，先测试上述例子、两个空集、只有一个空集、两个相同集合。

**检查点 C2：** 结果正确，再打印 A、B，确认输入没有变化。最后分别销毁 A、B、C，没有崩溃。

### Q10 · 交集：只留下双方都有的元素

**实现：** `InerSect(la, lb, lc)`。本讲义沿用图片中看起来为 `InerSect` 的拼写；不要在声明处写这个名字，实现或调用时却写成 `InterSect`。

例子仍然是：

```text
A = {1, 3, 5}
B = {2, 3, 4}
A ∩ B = {3}
```

当 p 指向 1、q 指向 2，为什么可以直接跳过 1？因为 B 已经有序，q 后面只可能更大，1 不可能再匹配到 B 中的元素。

先自己填表，再编程：

| 情况 | 加入结果吗 | 移动谁 |
| --- | --- | --- |
| p 的值更小 | ? | ? |
| q 的值更小 | ? | ? |
| 值相等 | ? | ? |

<details>
<summary>提示 1：移动哪一边？</summary>

较小值的一侧向后移动，不加入结果。相等时加入一次，两个指针一起移动。

</details>

<details>
<summary>提示 2：要不要复制剩余部分？</summary>

不要。任意一方已经结束，剩余值就无法再找到双方配对。交集循环可以结束。

</details>

**实现步骤：** 清空 lc；建立 p、q、tail；只在双方均未结束时比较；按三种情况更新。可复用 `AppendToTail`，但请自己写出判断。

**检查点 C3：** 相同集合与自身求交得到原集合；不相交的集合求交得到空集；任意集合与空集求交得到空集。

### Q11 · 差集：A 里有，但 B 里没有

**实现：** `Subs(la, lb, lc)`，计算 `A - B`。

```text
A = {1, 3, 5}
B = {2, 3, 4}
A - B = {1, 5}
B - A = {2, 4}
```

差集有方向。交换参数通常会改变结果。

理解每一种比较比背代码更有用：

| 比较结果 | 你能确定什么 | 本步动作 |
| --- | --- | --- |
| p 的值小于 q | B 剩余元素不会再有 p 的值 | 加入 p，移动 p |
| p 的值等于 q | p 的值在 B 中出现 | 不加入，两个都移动 |
| p 的值大于 q | q 太小，但 B 后面还可能匹配 p | 只移动 q，保留 p 等待 |

**停止后处理：** B 先结束，A 的剩余值都应该保留；A 先结束，结果已经完整，B 剩下什么不影响结果。

<details>
<summary>提示：如何复用并集的骨架？</summary>

保留双方比较的循环，但改成上表的加入规则。循环后只复制 A 剩余值，不复制 B 剩余值。

</details>

#### 集合运算测试站

在 `main.cpp` 中、`ExpectList` 后面加入这个测试函数。它要等三个集合函数都实现后才能一起编译运行：

```cpp
void CheckSets(std::initializer_list<int> rawA,
               std::initializer_list<int> rawB,
               std::initializer_list<int> aExpected,
               std::initializer_list<int> bExpected,
               std::initializer_list<int> unionExpected,
               std::initializer_list<int> intersectionExpected,
               std::initializer_list<int> differenceExpected) {
    slink* A = InitList();
    slink* B = InitList();
    slink* C = InitList();
    for (int x : rawA) Append(A, x);
    for (int x : rawB) Append(B, x);
    Sort(A);
    Unique(A);
    Sort(B);
    Unique(B);
    ExpectList(A, aExpected);
    ExpectList(B, bExpected);

    Append(C, 999); // 验证函数确实清除了旧结果
    Union(A, B, C);
    ExpectList(C, unionExpected);
    ExpectList(A, aExpected);
    ExpectList(B, bExpected);

    InerSect(A, B, C);
    ExpectList(C, intersectionExpected);
    ExpectList(A, aExpected);
    ExpectList(B, bExpected);

    Subs(A, B, C);
    ExpectList(C, differenceExpected);
    ExpectList(A, aExpected);
    ExpectList(B, bExpected);

    DestroyList(A);
    DestroyList(B);
    ExpectList(C, differenceExpected); // 输入释放后，结果仍应有效
    DestroyList(C);
}
```

在 main 中运行以下测试。每次只新增一行调用，失败时更容易找到是哪组数据：

```cpp
CheckSets({}, {}, {}, {}, {}, {}, {});
CheckSets({}, {2, 1}, {}, {1, 2}, {1, 2}, {}, {});
CheckSets({2, 1}, {}, {1, 2}, {}, {1, 2}, {}, {1, 2});
CheckSets({1}, {1}, {1}, {1}, {1}, {1}, {});
CheckSets({1, 3}, {2, 4}, {1, 3}, {2, 4}, {1, 2, 3, 4}, {}, {1, 3});
CheckSets({7, 2, 5, 2}, {5, 8, 2, 8}, {2, 5, 7}, {2, 5, 8},
          {2, 5, 7, 8}, {2, 5}, {7});
CheckSets({0, -2, -2, 5}, {-2, 3, 0}, {-2, 0, 5}, {-2, 0, 3},
          {-2, 0, 3, 5}, {-2, 0}, {5});
CheckSets({1, 2}, {1, 2, 3}, {1, 2}, {1, 2, 3}, {1, 2, 3}, {1, 2}, {});
```

再自己补一组交换 A、B 的差集测试。不要只确认 `A - B`，也手算一次 `B - A`。

这些检测与题目的对应关系是：

| 检测场景 | 主要检查 |
| --- | --- |
| 两个空集 | Q9～Q11 的空指针移动和空结果 |
| A 为空或 B 为空 | 并集复制剩余部分，差集只保留 A 的剩余部分，交集为空 |
| 两个集合完全相同 | 并集不重复，交集完整，差集为空 |
| 完全不相交 | 并集完整，交集为空，差集等于 A |
| 输入有重复、无序、负数和 0 | Q7、Q8 的预处理，以及 Q9～Q11 的正常工作 |
| A、B 释放后仍检查 C | 结果是否真正申请了独立节点 |

如果 Q9、Q10 或 Q11 单独失败，不要只看最终输出。先把 `CheckSets` 中三个调用分开运行：先只保留 `Union`，再只保留 `InerSect`，最后只保留 `Subs`，这样能确定是哪一个合并循环出错。

**检查点 C4：** 能解释三种运算各自遇到“小于、等于、大于”时做什么，且能说明哪一种需要复制哪一边的剩余值。

#### 到这里，复杂度应该怎样讲

设 A 有 n 个数据节点，B 有 m 个数据节点，结果有 k 个数据节点。

| 操作 | 时间 | 原因 |
| --- | --- | --- |
| 本讲义的单次 Append | O(n) | 每次从头找尾部 |
| 用上述 Append 建 n 个节点 | O(n²) | 第一次找得短，后面越来越长 |
| Length、PrintList、ClearList | O(n) | 各节点处理一次 |
| 选择排序 Sort | O(n²) | 每一轮重新扫描未排好部分 |
| 已排序后的 Unique | O(n) | 只向后扫描 |
| 并、交、差的合并阶段 | O(n + m) | 两个输入指针各自只向前，尾指针追加为 O(1) |

并交差申请 O(k) 的结果节点，除结果外只用 O(1) 的辅助变量。如果 lc 原来已有 r 个节点，先清空旧结果另需 O(r) 时间。排序、建表预处理不包含在“合并阶段 O(n+m)”中。

如果集合函数反复调用从头找尾的 Append，结果构造最坏可能达到 O((n+m)²)，所以 Q9 专门引入了尾指针。理解之后，你也可以在批量建表时维护尾指针，把建表优化为 O(n)。

<a id="part-d"></a>

## D：把链表首尾接起来

### Q12 · 建立猴子围成的圈

集合用头节点简化操作；猴子圈使用**无额外头节点的单向循环链表**，每个节点恰好对应一只猴子，没有一个需要跳过的“假猴子”。

先新增 `CircularLinkedList.h`：

```cpp
#pragma once

struct MonkeyNode {
    int id;
    MonkeyNode* next;
};

MonkeyNode* CreateCircle(int n);
void PrintCircle(const MonkeyNode* tail);
int RemoveAfter(MonkeyNode*& prev);
void DestroyCircle(MonkeyNode*& entry);
```

本讲义约定 `CreateCircle(n)` 返回**尾节点**，而不是第一只猴子。建好后，尾节点编号为 n，第一只猴子是 `tail->next`。这样第一轮报数时，已经有了“1 号的前驱”，删除 1 号也方便。

```text
                     tail
                      ↓
         [1] → [2] → [3]
          ↑           │
          └───────────┘
```

请在 `CircularLinkedList.cpp` 中实现建环，步骤如下：

1. 若 `n <= 0`，返回 `nullptr`。正式程序会把这类输入当作无效输入；底层建环函数仍作保护。
2. 用 first、tail 两个指针，先建立 `1 → 2 → ... → n → nullptr`。
3. 第一只猴子创建时，让 first、tail 都指向它；之后用 tail 追加，避免反复找尾。
4. 全部创建后，令尾节点的 next 指向 first。
5. 返回 tail。

**先预测：** n 为 1 时应该是什么样？

<details>
<summary>提示</summary>

只有一个节点时，它的 next 指向自己。不是 `nullptr`，也不需要创建第二个节点。

</details>

#### 怎样遍历一整圈

`PrintCircle(tail)` 约定从 `tail->next` 开始，输出一整圈的数据；空指针输出 `{}`。对刚创建的圈应输出 `{1, 2, ..., n}`。

非空时先记住起点 start，再做：

```text
p = start
至少执行一次：
    处理 p
    p = p 的下一只
只要 p 还没有回到 start，就继续
```

可以使用 `do { ... } while (...);`。如果写 `p = start; while (p != start)`，循环一次也不会执行。

注意：只有传入约定的尾节点时，打印才从当前指定的第一只开始。后面的报数游标会移动；不能一直把它当作最初编号 n 的尾节点。

在 `main.cpp` 加上 `#include "CircularLinkedList.h"`。从本题开始，编译命令改为：

```bash
clang++ -std=c++17 -Wall -Wextra -pedantic -g SinglyLinkedList.cpp CircularLinkedList.cpp main.cpp -o lab1_demo
./lab1_demo
```

测试以下情况，完成 Q13 后给每次测试补上 `DestroyCircle`：

```cpp
assert(CreateCircle(0) == nullptr);
PrintCircle(nullptr); // 预期：{}
{
    MonkeyNode* tail = CreateCircle(1);
    assert(tail != nullptr);
    assert(tail->id == 1);
    assert(tail->next == tail);
    PrintCircle(tail); // {1}
    // Q13 完成后加 DestroyCircle(tail);
}
{
    MonkeyNode* tail = CreateCircle(4);
    assert(tail != nullptr);
    assert(tail->id == 4);
    const MonkeyNode* p = tail->next;
    for (int id = 1; id <= 4; ++id) {
        assert(p != nullptr);
        assert(p->id == id);
        p = p->next;
    }
    assert(p == tail->next);
    PrintCircle(tail); // {1, 2, 3, 4}
    // Q13 完成后加 DestroyCircle(tail);
}
```

**检查点 D0：** n 为 1、4 都能只打印一圈。解释为什么这里不能使用 `while (p != nullptr)`。

### Q13 · 删除 prev 后面的节点

**实现：** `int RemoveAfter(MonkeyNode*& prev)`。

函数约定：prev 指向圈内一个有效节点，删除它后面的那只猴子，返回被删猴子的编号。函数不负责输出。

```text
删除前：prev → [2] → [3] → [4]
删除后：prev → [2] ──────→ [4]
```

这是一个局部图，完整链表仍然首尾相连。需要删除的是 `prev->next`，不是 prev。

请按下面的思路实现：

1. 记下 `victim = prev->next`。
2. 保存 victim 的编号，用于返回。
3. 如果 victim 就是 prev，说明只剩一个节点：释放它，把 prev 置为 `nullptr`。
4. 否则，让 prev 的 next 跳过 victim，然后释放 victim；prev 本身仍指向原节点。
5. 返回保存的编号。

不要对空 prev 调用这个函数；可以用 `assert(prev != nullptr)` 表达这个前提。函数签名中的引用只在删除最后一个节点时用来把调用者指针置空。

**再实现：** `DestroyCircle(MonkeyNode*& entry)`，反复调用 `RemoveAfter(entry)`，直到 entry 为 `nullptr`。entry 可以指向圈里的任意一个节点；销毁不关心从哪个编号开始。

这解释了为什么这里不用普通链表的释放循环：普通链表靠 `nullptr` 停止，圈必须一边删除一边维护环，直到删除最后一个节点。

测试这段“删除下一只”的连续操作：

```cpp
{
    MonkeyNode* prev = CreateCircle(3); // 指向 3，它的下一只是 1
    assert(RemoveAfter(prev) == 1);
    assert(prev->id == 3);
    assert(prev->next->id == 2);
    assert(prev->next->next == prev);
    assert(RemoveAfter(prev) == 2);
    assert(prev->next == prev);
    assert(RemoveAfter(prev) == 3);
    assert(prev == nullptr);
    DestroyCircle(prev); // 空圈重复销毁应安全
}

{
    MonkeyNode* entry = CreateCircle(4);
    DestroyCircle(entry);
    assert(entry == nullptr);
    DestroyCircle(entry); // 再次销毁空圈也应安全
}
```

这组测试专门检查 `DestroyCircle` 是否能从非空循环链表释放到最后一个节点，并把调用者的指针设为 `nullptr`。回到 Q12，把创建圈的测试中暂缺的清理补上。

**检查点 D1：** 能正确删除第一个编号、连续删除到单节点，并最终释放最后一只猴子。没有保留并继续使用指向已删节点的旧指针。

### Q14 · 先手动选一次猴王

先不写函数，用 `N = 5，M = 3` 在纸上报数。每次删除后，紧邻下一只重新报 1。

| 轮次 | 本轮从谁报 1 | 报数路径 | 退出者 | 剩下的猴子，按编号列出 |
| --- | --- | --- | --- | --- |
| 1 | 1 | 1、2、3 | 3 | 1、2、4、5 |
| 2 | 4 | 4、5、1 | 1 | 2、4、5 |
| 3 | 2 | 2、4、5 | 5 | 2、4 |
| 4 | 2 | 2、4、2 | 2 | 4 |

最后是 4。最后一轮会绕过起点再次经过同一节点，这正是循环链表的用途。

现在引入一个始终要保持的约定：

> 每轮开始时，prev 指向“本轮报 1 的猴子”的前一只。也就是说，`prev->next` 才是本轮起点。

第一次 prev 是 CreateCircle 返回的尾节点，因此 `prev->next` 是 1 号。每向前走一步，相当于把“准备删除的候选猴子”向前推进一位。

**预测题：** M 为 3 时，prev 应该走 2 步还是 3 步，然后删除 `prev->next`？M 为 1 时应该走几步？

<details>
<summary>解析</summary>

M 为 3 时走 2 步。起初下一只已经是报 1 的猴子，走一步后下一只是报 2 的，再走一步后下一只是报 3 的。一般走 M−1 步。

M 为 1 时一步都不走，直接删除 `prev->next`。删除之后，它的新 next 恰好就是下一轮报 1 的猴子，prev 可以保持不动。

</details>

**检查点 D2：** 在纸上标出每次删除前的 prev。你能解释为什么删除后不额外前进一步吗？额外走一步会跳过本来该报 1 的猴子。

### Q15 · 实现约瑟夫问题

按图片要求，把选王函数放在 `main.cpp`，位于 main 前面；圈的创建、删除等基础操作仍放在 `CircularLinkedList.cpp`。

函数接口：

```cpp
int Josephus(int n, int m);
```

约定：n、m 必须为正数；不合法则抛出 `std::invalid_argument`。成功时返回猴王编号，并在返回前释放整个圈，包括最后获胜的节点。

在 `main.cpp` 增加 `#include <stdexcept>`，完成下面骨架：

```cpp
int Josephus(int n, int m) {
    if (n <= 0 || m <= 0) {
        throw std::invalid_argument("n and m must be positive");
    }

    MonkeyNode* prev = CreateCircle(n);
    // TODO：只要不止一个节点，就继续一轮报数。
    // 每轮：让 prev 前进 m - 1 步，然后 RemoveAfter(prev)。
    // 调试时可以输出 RemoveAfter 的返回值，观察出圈顺序。

    // TODO：只剩一只时，调用 RemoveAfter(prev) 删除最后节点，
    // 保存它返回的编号并返回。RemoveAfter 会把 prev 置为 nullptr。
    return 0; // 占位，完成后替换；合法猴王编号不会是 0
}
```

`throw` 表示报告非法参数；基础测试只传合法值。用户输入检查见 Q16，所以暂时不必学习复杂的异常处理。

<details>
<summary>提示 1：如何判断只剩一个节点？</summary>

检查 `prev->next == prev`。不要拿节点编号判断，最后剩下谁正是程序要算的答案。

</details>

<details>
<summary>提示 2：怎样收尾？</summary>

`RemoveAfter` 已经支持单节点情况，并返回被删除节点的编号。循环结束后调用一次 `RemoveAfter(prev)`，它返回的就是猴王编号，同时负责释放最后一个节点并把 prev 置为 `nullptr`。不要先 `delete prev`，再尝试读取 `prev->id`。

</details>

在 main 中测试：

```cpp
assert(Josephus(1, 1) == 1);
assert(Josephus(1, 99) == 1);
assert(Josephus(5, 1) == 5);
assert(Josephus(5, 2) == 3);
assert(Josephus(5, 3) == 4);
assert(Josephus(5, 7) == 4);
assert(Josephus(7, 3) == 4);
assert(Josephus(10, 3) == 4);
```

`N = 10，M = 3` 对应图片中的例子。调试输出应为：

```text
出圈顺序：3, 6, 9, 2, 7, 1, 8, 5, 10
猴王：4
```

`N = 7，M = 3` 则是：

```text
出圈顺序：3, 6, 2, 7, 5, 1
猴王：4
```

两组都剩 4，不代表算法一定正确，所以还要测 M 为 1、2 和 N 为 1。只看最终编号可能掩盖报数错误，调试时应一起核对出圈顺序。

再检查非法参数契约：

```cpp
auto rejects = [](int n, int m) {
    try {
        Josephus(n, m);
    } catch (const std::invalid_argument&) {
        return true;
    }
    return false;
};

assert(rejects(0, 3));
assert(rejects(3, 0));
assert(rejects(-1, 2));
```

`try/catch` 在这里的作用只是检查“函数是否拒绝了这个输入”。

**检查点 D3：** 输出编号正确，最后一个节点也被释放。基础模拟的时间为 O(NM)，建圈 O(N)，占用 O(N) 节点空间。

**可选优化：** 若当前还剩 r 只猴子，绕完整圈不会改变位置，因此本轮可只走 `(M - 1) % r` 步。先确保基础版正确，再维护 remaining 实现优化。不要因此改为递推公式求答案；本实验要求练习循环链表模拟。

<a id="part-e"></a>

## E：让程序能够演示，也能够解释

### Q16 · 整合输入与输出

现在才加入交互输入。前面用固定数据，是为了每次修改都能重现同一个问题。

最终由你逐步创建的源文件结构如下；这份指导本身只提供 Markdown 中的骨架与练习，并未预先生成这些源文件：

```text
lab1/
├── README.md                  ← 当前指导
├── SinglyLinkedList.h         ← 单链表结构与声明
├── SinglyLinkedList.cpp       ← 基础操作、排序、去重、集合运算
├── CircularLinkedList.h       ← 猴子节点与基础操作声明
├── CircularLinkedList.cpp     ← 建环、遍历、删除、销毁
└── main.cpp                   ← 测试、输入输出、Josephus、唯一的 main
```

项目中只保留一个 main。可以先把已通过的测试调用整理进 `void RunSelfTests()`，让 main 调用它，确认通过后再进入输入流程。`ExpectList`、`CheckSets` 等辅助函数仍放在 main 前面；不要为了演示永久删掉测试。

如果刚开始写交互不熟练，先分别调通下面两段流程，最后用菜单 `1：集合运算，2：猴子选王，0：退出` 调用它们。菜单只是便于演示的建议，图片没有要求特定菜单样式。

#### 集合运算输入流程

建议采用“先输入个数，再输入元素”，不要用 0 或 −1 当结束标志，因为它们可以是合法集合元素。

```text
输入 A 的元素个数：4
输入 A 的元素：7 2 5 2
输入 B 的元素个数：4
输入 B 的元素：5 8 2 8

A = {2, 5, 7}
B = {2, 5, 8}
A union B = {2, 5, 7, 8}
A intersection B = {2, 5}
A - B = {7}
```

实现顺序：

1. 读取元素个数，要求为非负整数，允许 0 表示空集。
2. 创建 A、B、C，按输入调用 Append。
3. 对 A、B 分别执行 Sort 和 Unique，然后打印标准化后的集合。
4. 调用 Union，打印 C；再调用 InerSect，打印 C；最后调用 Subs，打印 C。
5. 销毁三个链表。

如果元素读到一半出现非整数，也要清理已经创建的链表，再退出这个操作。可以先约定“输入错误就输出一条提示并结束程序”，不要一开始就加入复杂的重新输入逻辑。

#### 猴子选王输入流程

演示格式可以是：

```text
输入猴子数量 N 和报数上限 M：10 3
猴王编号：4
```

输入检查示例，放在返回 int 的 main 中：

```cpp
int n = 0;
int m = 0;
if (!(std::cin >> n >> m) || n <= 0 || m <= 0) {
    std::cerr << "N and M must be positive integers.\n";
    return 1;
}
std::cout << "Winner: " << Josephus(n, m) << '\n';
```

这一步在创建圈前就拒绝非法输入。若把逻辑移入返回 void 的辅助函数，失败时改为 `return;`，不要原样保留 `return 1;`。

**检查点 E0：** 你能向另一个人演示一组含重复值的集合运算，以及 `10 3`、`5 1` 两组猴子选王，并指出代码中对应的函数。

### 调试实验 · 在哪一步开始偏离你的预期

CS61B 的调试 lab 强调通过断点、单步执行和变量状态来定位问题。本实验把这一方法用于 C++：先写下“这一行后应该是什么”，再观察实际状态。参考：[CS61B Lab 2: Syntax Review, Debugging](https://fa26.datastructur.es/labs/lab02/)。

先阅读以下**故意写错的片段**，说明错在哪里；不要把它们加入正确实现。

**Bug A：插入时丢失旧后继。** 目标是在 p 后插入 node：

```cpp
p->next = node;
node->next = p->next;
```

<details>
<summary>分析与修复方向</summary>

第一行已经覆盖了旧后继地址，第二行拿到的是 node 自己，于是 node 指向自己。先保存旧的下一跳，或者先让 node 接向旧后继，再让 p 接向 node。

画出 `p → 原后继`，逐行擦掉、重画箭头，错误会很明显。

</details>

**Bug B：删除后才想起下一站。**

```cpp
delete p;
p = p->next;
```

<details>
<summary>分析与修复方向</summary>

第二行读取了已经释放的节点。先保存下一跳，再释放，最后移动指针；或者像 ClearList 一样，通过仍有效的前驱重新接好链。

</details>

**Bug C：循环链表一直走到空指针。**

```cpp
while (p != nullptr) {
    p = p->next;
}
```

<details>
<summary>分析与修复方向</summary>

合法的非空循环链表没有空的 next，这个循环不会结束。遍历一圈应记录起点；删除到只剩一个节点应检查自环。这是两个不同任务，对应两个不同停止条件。

</details>

**Bug D：报数多走了一步。**

```cpp
for (int i = 0; i < m; ++i) {
    prev = prev->next;
}
RemoveAfter(prev);
```

<details>
<summary>分析与修复方向</summary>

当前约定 `prev->next` 已经报 1，因此只走 m−1 步。用 `N=5, M=1` 检查最直观：正确行为是直接删除 1 号，错误代码却先走一步。

</details>

#### 在调试器中观察什么

使用你已有的 C++ 调试器，在 Append 接线前、Unique 删除前、Josephus 调用 RemoveAfter 前设置断点。重点观察：

| 位置 | 看什么 | 应满足什么 |
| --- | --- | --- |
| Append 接线前 | tail、tail->next | tail 有效，next 为空 |
| Unique 删除前 | p->data、p->next->data | 两个值相等，且两个节点都有效 |
| 集合比较前 | p、q、结果尾节点 | 仅当 p、q 都有效时读取 data |
| 约瑟夫删除前 | prev->id、prev->next->id | 下一只是本轮应该退出的猴子 |

单步执行时注意：断点通常暂停在该行执行之前。不要对空指针或已释放节点求 `p->data`，也不要把“两个节点值相同”误认为“它们是同一个节点”。

#### 用编译器帮忙发现非法内存访问

基础测试通过后，可以在支持以下选项的 Clang 环境使用：

```bash
clang++ -std=c++17 -Wall -Wextra -pedantic -g -O1 -fsanitize=address,undefined -fno-omit-frame-pointer SinglyLinkedList.cpp CircularLinkedList.cpp main.cpp -o lab1_check
./lab1_check
```

这会帮助发现越界、释放后访问、重复释放等问题。看到报告时，从报告中的源文件行号开始检查。它不能证明程序完全正确，也不保证在所有平台上都检查内存泄漏；macOS 上尤其不要把“没有报告”当作“没有泄漏”的证明。仍应确认每个 new 出来的节点最终都被释放一次。

### Q17 · 图片中的扩展：每只猴子有自己的密码

完成基础版之后再做本题。先保存基础版，再新增扩展函数，避免改坏已经通过的 `Josephus`。

把节点扩展为：

```cpp
struct MonkeyNode {
    int id;
    MonkeyNode* next;
    int password = 1;
};
```

password 放在末尾并有默认值，因此已有的 `new MonkeyNode{id, nullptr}` 初始化仍可使用。正式生成随机密码时，再给 password 赋值。

本讲义采用的规则是：初始报数上限由用户输入；某只猴子退出后，**用这只退出猴子的密码作为下一轮报数上限**，从它的下一只重新报 1。图片的简略描述没有完全展开密码的使用时机，答辩时应说明你的约定；如果老师有更明确的解释，以老师说明为准。

先别急着随机。用固定密码进行可重复测试：

| 编号 | 密码 |
| --- | --- |
| 1 | 3 |
| 2 | 1 |
| 3 | 4 |
| 4 | 2 |
| 5 | 5 |

为了让函数接口和检测明确，先在 `main.cpp` 中声明：

```cpp
int JosephusWithPasswords(
    int initialM,
    std::initializer_list<int> passwords);
```

这里 `passwords` 的长度就是猴子数量 N，列表中的第一个值属于 1 号猴子。约定 `initialM > 0`，密码列表非空，且每个密码都大于 0；不满足时抛出 `std::invalid_argument`。如果 main 中还没有使用过 `std::initializer_list`，请补上：

```cpp
#include <initializer_list>
```

初始报数上限为 2，请手算，再实现 `JosephusWithPasswords`。节点仍需由循环链表保存和删除。

<details>
<summary>固定密码例子的核对结果</summary>

先删除 2，它的密码是 1；下一轮直接删除 3，它的密码是 4；再从 4 开始数 4 下，删除 4，它的密码是 2；最后从 5 开始数 2 下，删除 1。猴王是 5。

出圈顺序是 `2, 3, 4, 1`，各轮报数上限是 `2, 1, 4, 2`。

</details>

把下面的确定性测试放在基础版测试之后：

```cpp
assert(JosephusWithPasswords(2, {3, 1, 4, 2, 5}) == 5);
```

这一个测试同时检查：初始报数上限、按照退出猴子的密码切换下一轮上限、循环链表的出圈顺序，以及最后一只猴子的释放。调试时应额外输出并核对：

```text
出圈顺序：2, 3, 4, 1
各轮上限：2, 1, 4, 2
猴王：5
```

再加入非法输入测试：

```cpp
bool rejected = false;
try {
    JosephusWithPasswords(0, {3, 1, 4});
} catch (const std::invalid_argument&) {
    rejected = true;
}
assert(rejected);
```

实现时只比基础版多一个关键步骤：**删除前**读取 `prev->next->password` 并保存下来，删除后再把当前报数上限更新为它。现有 RemoveAfter 仍然可以只返回编号。

固定测试通过后，再增加随机生成。你可以采用如下生成器配置，记得引入 `<random>`：

```cpp
std::mt19937 rng(2026);
std::uniform_int_distribution<int> passwordRange(1, 9);
// 为每个节点赋值：node->password = passwordRange(rng);
```

1～9 是本讲义为方便演示选择的范围，图片未指定范围；要保证密码为正数。随机生成器只初始化一次，再依次生成所有密码。固定种子便于在同一环境重现；不同标准库不保证这段分布代码给出完全相同的密码序列，因此跨环境的标准测试仍使用上面的固定密码表。

演示时先输出“编号—密码”表，再显示每轮退出者和下一轮报数上限。这样观众才能核对过程。

**检查点 E1：** 固定密码测试正确；随机版打印了实际使用的密码；所有节点依然正确释放。

## 遇到问题时，先看这里

| 现象 | 优先检查 |
| --- | --- |
| `Undefined symbols` / `undefined reference` | 声明了但尚未定义的函数是否被调用？编译命令是否漏了 `.cpp`？函数名、参数是否一致？ |
| 提示 main 重复定义 | 是否把多个测试文件中的 main 一起编译了？最终只保留一个入口 |
| 空表就崩溃 | 是否把 `L == nullptr` 与 `L->next == nullptr` 混淆？是否先解引用再判空？ |
| 打印多出一个 0 | 是否把头节点当数据输出了？ |
| 打印漏最后一个值 | 是否把遍历条件误写成 `p->next != nullptr`？ |
| 连续重复值只删了一部分 | 删除后是否错误地马上移动了 p？ |
| 集合结果夹着旧数据 | 每次求结果前有没有 ClearList(lc)？ |
| 销毁 A 后 C 失效 | 是否把输入节点直接接进了结果，而没有复制数据到新节点？ |
| 循环链表输出停不下来 | 是否还在等 `nullptr`？起点是否在遍历时也被移动了？ |
| 猴王不对，但程序不崩 | 是否多走了一步？删除后是否从紧邻下一只重新报 1？ |
| 修改后结果完全没变化 | 是否保存文件、重新编译，并运行了新生成的程序？ |

不要同时修改多个函数来“试试看”。先找出最小失败用例，比如空表、两个相同值，或 `N=3, M=1`；画出出错前一刻的连接，再对照每条赋值。

## 最后检查：代码与讲解都准备好了吗

基本功能完成后，逐项勾选：

- [ ] 我能解释节点、指针、头指针、头节点、首个数据节点和尾节点的区别。
- [ ] 我能解释为什么链表节点不要求连续存储。
- [ ] 单链表支持空集、负数、0、重复输入，打印不包含头节点。
- [ ] Sort 保持节点数；Unique 正确处理连续重复值。
- [ ] 三种集合运算只读取 A、B，产生独立、有序且去重的结果。
- [ ] 我能解释 `A - B` 与 `B - A` 的区别。
- [ ] 我知道循环链表的单节点状态是 next 指向自己。
- [ ] N=1、M=1、M>N 等情况都测试过；例如 `(1,99)` 覆盖了 M>N。
- [ ] 另外测试过多节点且 M>N 的情况，例如 `(5,7)`；先手算再核对结果。
- [ ] 非法输入有明确处理，不会进入错误的报数循环。
- [ ] 数据节点、头节点以及最终猴王节点都恰好释放一次。
- [ ] 多文件编译通过，只有一个 main，基础循环链表操作位于要求的文件中。
- [ ] 关键注释解释“为什么这样做”，不是逐行把 C++ 翻译成中文。
- [ ] 基础版稳定后，才按需加入随机密码扩展。

准备答辩时，请尝试不看代码回答：

1. 你的空链表在内存中是什么样？为什么还分配一个头节点？
2. `p = p->next` 与 `p->next = q` 分别改变了什么？
3. 为什么单链表删除一般需要前驱？
4. 为什么有序链表的交集可以跳过较小值？
5. 并集的 O(n+m) 是否包含排序？为什么维护结果尾指针很重要？
6. 猴子报 M 时为什么让 prev 走 M−1 步？
7. 一轮删除之后，哪个节点会在下一轮报 1？
8. 如果不释放最后的猴王节点，会发生什么？

你能画图讲清楚这些问题，就已经理解了这份实验最重要的内容。

## 额外练习：在基础版之后继续挑战

这些不属于图片明确要求的基本功能，可以按兴趣完成。

1. **重连节点的插入排序。** 保留 Q7 作为已通过版本，另写 SortByLinks。每次从原链表取下第一个节点，插入一个有序链表的合适位置。取节点前先保存后继；不要创建或删除数据节点。对 `{7, 2, 5, 2}` 测试有序性和长度，并确认所有旧节点仍存在。
2. **优化批量建表。** 用尾指针构建集合，避免对每个输入值都从头找尾；说明为什么建 n 个节点从 O(n²) 降到 O(n)。
3. **更多性质测试。** 核对 `A∪A=A`、`A∩A=A`、`A−A=空集`、`A∪B=B∪A`，以及“交集中的每个元素同时在 A、B 中”。这些性质可帮助发现只靠一组样例找不到的错误。

## 参考阅读与本讲义的改编方式

以下链接来自课程官方站点或官方历史课程站点，查阅日期为 2026-09-19。CS61A 历史页面在本次直连读取中有访问失败，相关题目结构通过搜索服务提供的官方页面内容核对；CS61B 页面可直接读取。课程网站后续可能调整路径。

- [CS61A Fall 2023 · Lab 8: Linked Lists](https://www-inst.eecs.berkeley.edu/~cs61a/fa23/lab/lab08/)：参考先预测输出、再完成小函数、配合例子与提示的练习组织；本讲义 Q1 使用原创 C++ 指针预测题。
- [CS61B Fall 2026 · Lab 3: Linked Lists](https://fa26.datastructur.es/labs/lab03/)：参考逐行画方框箭头图、区分修改原链表与生成新链表、明确头节点作用的方法；本讲义应用于 Q3、Q9 和循环链表练习。
- [CS61B Fall 2026 · Lab 2: Syntax Review, Debugging](https://fa26.datastructur.es/labs/lab02/)：参考小步运行测试、断点和单步观察；本讲义把观察对象换成 C++ 节点和指针。
- [CS61B Spring 2024 · Lab 3: Debugging, Part 2](https://sp24.datastructur.es/labs/lab03/)：参考分阶段排查、先尝试再看提示、从实际失败状态寻找原因的组织方式。

这些课程中的 Python/Java 库、提交平台、评分与课堂政策属于各自课程。本文的实验要求来自你提供的两张图片；这里的检查点和练习顺序是为本次 C++ 实训设计的。
