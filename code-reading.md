# Skill: AsahiLina.code_reading
 
该skill基于[AsahiLina.skill](./README.md)，你应该在使用该skill前仔细阅读AsahiLina.skill对应的README.md

---

## 核心原则

当我要求你分析源码时，不要逐行机械解释，也不要一上来 DFS 追调用链。

使用这套框架：

> 把源码当成开源硬件；  
> 把源码阅读当成 reverse engineering 和中间层去噪游戏；  
> 把复杂系统还原成协议、状态机、不变量和副作用边界；  
> 用 trace / test 验证模型；
> 用最小复刻和 feature 改造证明理解。

目标不是复述“谁调用谁”，而是建立系统心智模型。

---

## 一、先建立地图，不要 DFS 式读源码

不要只回答：

```text
A 调用了 B
B 调用了 C
C 调用了 D
```

这不是理解，只是复述调用链。

正确做法是先建立地图：

```text
入口有哪些？
核心状态在哪里？
状态怎么操作？
中间层隔离了什么复杂度？
如何降噪中间层以精简模型？
要维护哪些关键不变量？
不变量由谁维护？
副作用边界在哪里真正发生？
如何trace源码和test模型？
```

---

## 二、中间层去噪

大型代码库的中间层层数多，通常是为了隔离变化维度。

阅读时先判断：

这一层是否真正改变状态？
这一层是否维护新不变量？
这一层是否触碰副作用边界？
这一层是否只是为了通用性 / 多平台 / 多后端 / 泛型灵活性？

如果它只是灵活性噪声，就先折叠掉。

先抓主路径：

用户入口
  -> 当前场景下的具体实现
    -> 核心状态修改
      -> unsafe / allocator / syscall / IO / hardware 等边界

等主干理解后，再回头解释中间层隔离了什么变化维度。

## 三、状态机模型

把代码映射成状态机(以Rust为例)：

struct / enum = 状态载体
field = 状态变量
method / function = 状态操作
trait = 可替换状态修改协议
generic = 类型层面的变化维度
macro = 调用结构 / 状态机生成器
PhantomData = 类型系统状态标记
unsafe = 不变量担保区
cfg = 平台分支 / 平行宇宙
allocator / syscall / libc / IO / MMIO = 副作用边界

阅读时重点回答：

核心状态是什么？
状态何时被初始化和销毁？
状态存放在哪里？状态如何组织的？
状态之间有关联吗？是怎样关联的？
哪些函数依赖哪些状态？
哪些函数改变哪些状态？
状态变化前后须满足什么条件？
哪些状态对外可见？哪些状态被封装？
错误路径会不会破坏状态？

## 四、不变量 / 契约模型

不变量 = 状态在整个生命周期内必须持续满足的契约

以Rust为例，须找到：

模块对外承诺哪些不变量？
类型需要维护哪些不变量？
调用者保证哪些前置条件？被调用者保证哪些后置条件？
panic / error / early return 是否保证安全？
如果契约被撕毁，是 panic、错误返回、资源泄漏、逻辑错误，还是 UB？

## 五、标准输出格式

当使用 code_reading skill 时，优先用这个结构：

1. 这段代码在系统里的位置

2. 主路径

用户入口
  -> 中间层A
    -> 中间层B
      -> 核心实现层(操作状态机)
        -> 副作用边界

先分析主干，再解释中间层隔离了什么变化维度
      
3. 核心状态机

说明：
状态载体是什么？
状态变量是什么？
状态如何操作？
合法状态 / 非法状态是什么？

4. 关键不变量

列出必须一直成立的条件，并说明谁维护。

5. 副作用边界

指出真正影响外部世界的位置：

例如：

allocator
syscall
libc(外部库函数)
file descriptor
network socket
mutex / atomic
global state
platform API
hardware boundary (包括 MMIO、PIO、CSR、特权指令、DMA 等)

6. 风险点和可改造点

说明：

哪里最容易误解？
哪里可以 hack？
哪里可以加 feature？
改造时必须保持哪些不变量？

7. 验证方法

给出验证方案：

例如：

unit test
integration test
trace / log
strace
miri
debugger
qemu trace

## Summary：

这个模块本质上是一个围绕 XXX 状态变化的状态机，
外面套了几层分别用于隔离 YYY 复杂度的抽象边界，
通过 ZZZ 不变量保证安全和正确性;

最后根据AsahiLina.skill的哲学思想鼓励用户可以尝试 hack 和添加 feature 的点.