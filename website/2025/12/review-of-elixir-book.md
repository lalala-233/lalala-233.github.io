# 《函数式编程入门：使用 Elixir》读后感

书籍的 ISBN：978-7-5680-6171-1。

## 读后感

这本书通过介绍 Elixir 语言，讲解了一些函数式编程的基础概念，如不可变值、模式匹配、递归函数、高阶函数等内容。

与其说是在介绍函数式编程，这本书更像在介绍 Elixir 这门语言。

## 一些有意思的地方

这里介绍了一些 Elixir 中有意思的部分，或许可以看成 feature？

### 模式匹配

模式匹配允许你少写一些 if 语句或使用匹配守卫（Match Guard），一定程度上减小了心智负担。

有意思的是，Elixir 将调用函数也视为一种模式匹配，并提供了一种多分句函数的写法，从某种程度上看这有点像函数重载。

```elixir
defmodule Geometry do
  def area({:rectangle, width, height}) do
    width * height
  end
  
  def area({:circle, radius}) do
    3.14 * radius * radius
  end
  
  def area({:triangle, base, height}) do
    0.5 * base * height
  end
end
```

Rust 也将函数参数的匹配也视为模式匹配，允许你解构参数，但没有这种多分句函数的写法。

AI 总结：

1. 多分句函数：同一个函数名，不同参数模式。
2. 守卫子句：when 关键字添加额外约束。
3. 结构化匹配：直接匹配 Map、List 的结构。
4. 二进制模式：匹配二进制数据的特定模式。
5. 变量绑定：在匹配的同时提取值。

### 不可变数据

Elixir 的所有数据默认不可变，又不像 Rust 一样有所有权的概念，因此为了避免频繁复制，Elixir 有着所谓的写时复制（Copy on Write）特性，不同的数据可以共享同一块内存，只在修改时才会额外复制。

> 据 AI 所言，写时复制特性来源于 Elixir 底层的 Erlang VM，即 BEAM。

## 一些神秘的地方

这里介绍了 Elixir 一些神秘或神奇的部分，或许会令人困惑？

### 函数命名

Elixir 的函数命名也很有意思，一个函数名由名字和参数的数量组成，如 `Enum.map/2`，这代表 `map` 这个函数接收两个参数。

某种意义上这种做法也提供了函数重载的部分功能。

> 据 AI 所言，这种命名约定来自于 Erlang 标准库。

同时，一些函数以 `?` 或 `!` 结尾，如 `Enum.empty?/1`、`File.write!/2`，问号表示函数返回的是 bool 值，而感叹号表示函数比较危险（？），即可能抛出异常或有破坏性。

> 据 AI 所言，问号的命名来自于 Lisp，感叹号的命名可能来自 Ruby。

### 多种控制语句

> 爱来自 AI

Elixir 提供了多种条件控制结构：

|结构|用途|例子|
|:-:|:-:|:-:|
|`if/unless`|单一条件检查|`if x > 0, do: :positive`|
|`cond`|多个条件分支|检查多个条件表达式|
|`case`|模式匹配|匹配不同模式|
|`with`|链式成功路径|多个可能失败的操作|

> 据 AI 所言，if 和 unless 在 Elixir 中是宏，不是语言关键字。

### 模块

将代码模块化当然是个好主意，但是……

Elixir 的模块略显抽象，或许更像 cpp 里的 namespace，和 Rust 中的 mod 区别还是很大的。

### 冗余（？）

Elixir 中有个名为「协议」的神秘机制，允许实现多态。

但是，Elixir 中还有个名为「行为」的神秘机制，也允许实现多态。

Elixir 有个错误处理机制，提供了 throw/catch。

但是，Elixir 还有个错误处理机制，提供了 raise/rescue。

诚然，这些机制的侧重点是不同的，但这种允许用多种方式实现同一功能的冗余体现在了很多方面。

### 递归

递归是个好东西。

这本书简单介绍了一下递归的思想，以及减治法和分治法的概念。

同时，这本书简单介绍了一下尾递归和体递归的区别，Elixir 应该会尽可能保证尾递归被优化。

一个有意思的地方是无界递归函数，采用了一种延迟计算的思想，只在需要时才计算。

一个神秘的地方是其中有一章叫「递归调用匿名函数」，这章引入了函数式编程中的 Y 组合子的思想，但并未过多介绍，比较神秘。

不过 Y 组合子比较学术化，在生产中几乎不会这么写，有兴趣的可以自己去搜搜相关知识。

### 列表

与 c/cpp 这种需要手动管理内存的语言不同，Elixir 可以自动地管理你的内存。

其实和 Python 没多大区别，不就是把列表作为一种语法、且可以动态创建列表嘛……

但是配合上 Elixir 的语法，写点排序算法很有意思。

快速排序：

```elixir
defmodule QuickSort do
  def sort([]), do: []
  def sort([pivot | rest]) do
    {lesser, greater} = Enum.split_with(rest, &(&1 <= pivot))
    sort(lesser) ++ [pivot] ++ sort(greater)
  end
end
```

归并排序：

```elixir
defmodule MergeSort do
  def sort([]), do: []
  def sort([x]), do: [x]
  
  def sort(list) do
    {left, right} = split(list)
    merge(sort(left), sort(right))
  end
  
  defp split(list) do
    mid = div(length(list), 2)
    Enum.split(list, mid)
  end
  
  defp merge([], right), do: right
  defp merge(left, []), do: left
  
  defp merge([l | left], [r | right]) when l <= r do
    [l | merge(left, [r | right])]
  end
  
  defp merge([l | left], [r | right]) do
    [r | merge([l | left], right)]
  end
end
```

### 动态语言

作为一门动态语言，Elixir 有着很神秘的性质：热更新（热代码升级）。

你可以在不停止系统的情况下更新代码，并在运行时直接切换版本。

同时，还有神秘的类型标注和静态分析，有助于让代码更加规范，减小出错的可能。

最后，Elixir 还可以编译成字节码给虚拟机执行，进一步提高性能（虽然仍会在运行时进行类型检查）。

> 据 AI 所言，Elixir 热更新的能力来源于 Erlang，而 Erlang 热更新的能力来源于爱立信公司在电信领域的特殊需求。
>
> Elixir 能热更新的原因：
>
> - 历史需求：源自电信系统 24/7 运行要求
> - 虚拟机设计：BEAM 的进程隔离和代码版本管理
> - 函数式核心：不可变数据 + 纯函数 = 安全的热更新
> - OTP 框架：提供了标准化的热升级流程

## 一些令人困惑的地方

这里介绍了一些 Elixir 中可能令人困惑、或这本书中并未深入讨论的内容。

### 原生数据类型

> 据 AI 所言，Elixir 继承了 Erlang 的衣钵，而 Erlang 诞生于爱立信的电信交换机系统，这些数据类型在针对特定场景进行了优化。

Elixir 有着一些原生的数据类型，如列表、元组、映射表等，并为此提供了一些特殊的表示方式，如：

- 列表：`[1, 2, 3]`
- 元组：`{:ok, result}`
- 映射表：`%{key: value}`

同时，还有一些神秘的类型，如原子（如 `:ok`）。

新手记住这些特殊的表示可能需要花费一些精力。

### 神秘语法

可能作者以为他想通过 Elixir 来介绍函数式编程，并未过多的去讲语法相关概念。

虽说没怎么出现不讲语法直接给代码的问题，但是 Elixir 的部分语法还是过于神秘了。

```elixir
# 标注类型
@spec parse(String.t()) :: {:ok, term()} | {:error, String.t()}
def parse(str) when is_binary(str) do
  # ...
end

# 定义自定义类型
@type user_id :: integer()
@type user :: %User{id: user_id(), name: String.t()}
```
