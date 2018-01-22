title: 试水 Elixir
published_date: "2018-01-22 01:02:52 +0800"
layout: post.liquid
data:
  route: blog
---
# Why Elixir Rocks
我这个星期接触了 Elixir，遇到蛮多有趣的东西，于是写 blog 分享下。官方的介绍如下：
> Elixir is a dynamic, functional language designed for building scalable and maintainable applications.
要用一句话概括这语言给我的感觉就是：「皮是 Ruby, 肉是 Erlang, 骨是 Lisp」。

---

### Macro x Metaprogramming x DSL
讲 macro 之前项讲讲 AST。Elixir 的 AST 非常简单，要么是 literals（Atom, String, List, Number, Tuple with 2 element），要么是这样的`tuple`：
```elixir
{atom | tuple, list, list | atom}
```
- 第一部分是`atom`或者另外一个这样的`tuple`
- 第二部分是 metadata，一个 keyword list
- 第三部分要么是函数的参数列表，要么是一个`atom`。如果是`atom`表明这个 tuple 是个变量。

Elixir 提供了`quote/unquote`（类似 lisp 里的`quasiquote / unquote`)，quote 一个表达式即可得到它的 AST(Abstract Syntax Tree)。
所谓 macro 就是接受 AST 返回 AST 的函数，这样形式简单的 AST 给 macro 的编写带来了方便。
Elixir 只有少数几个 [Sepcial Forms](https://hexdocs.pm/elixir/Kernel.SpecialForms.html), 表面那些像 Ruby 的语法基本上都是用 macro 造出来的，这一点很像 Lisp。
`if/else`是宏，`defstruct`是宏，`|>`管道运算符也是一个宏，它用起来是这样的：
``` elixir
# without |>
foo(bar(baz, meow))
# with |>
baz |> bar(meow) |> foo
```

而所谓 metaprogramming 就是写生成代码的代码。macro 提供了把代码当数据处理的能力，于是乎可以用来做元编程。
元编程可以用来在语言里造 [Domain-specific languages (DSL)](https://en.wikipedia.org/wiki/Domain-specific_language) 来生成 boilerplate 代码，于是就可以写出这样的代码：

```elixir
# Imports only from/2 of Ecto.Query
import Ecto.Query, only: [from: 2]

# Create a query
query = from u in "users",
          where: u.age > 18,
          select: u.name

# Send the query to the repository
Repo.all(query)
```
ecto 是用来操作数据库的库，它提供了类似 SQL 的 DSL。`query`通过宏展开最后变成了一个 Elixir 的函数。

另外 Elixir 的 macro 是 hygienic 的，不像 C 的基于字符替换的宏，会污染调用方的 namespace。但是它也提供了 var! 来打破这个 hygiene，从而往调用方的塞变量。

### Pattern Matching
Elixir 到处都是 pattern matching，实际上`=`是 match 操作符。Rust 里面同样有 pattern matching，但是功能不如 Elixir 的强大。在 Rust 里我们通常用它来 match enum(tagged union)，`std::Result`就是一个好例子。elixr 这边还能 match 很多 Rust 不能 match 的类型，比如 list（Rust 的 slice pattern 还是 experimental)：
```elixir
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
```
比如字符串：
```elixir
iex> "he" <> rest = "hello"
"hello"
iex> rest
"llo"
```
这个其实有局限性，只有最后一部分可以是变量。
比如 map:
```elixir
iex> %{} = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> %{:a => a} = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> a
1
iex> %{:c => c} = %{:a => 1, 2 => :b}
** (MatchError) no match of right hand side value: %{2 => :b, :a => 1}
```

我学 Elixir 的时候用它做了个 telegram bot。用户发来的消息是这样格式的：
```
/command [optional data]
or
/command@bot_name [optional data]
```
这里就需要把不同的消息 route 到不同的 handler。在别的语言大概是需要维护一个 hash table，把 pattern 和 handler 注册进去，收到消息就对着 hash table 一项一项地 match。而我用的库很巧妙地利用 pattern matching 解决了这个问题：

```elixir
  defp generate_command(command, handler) do
    quote do
      def do_match_message(
            %{
              message: %{
                text: "/" <> unquote(command)
              }
            } = var!(update)
          ) do
        handle_message(unquote(handler), [var!(update)])
      end

      def do_match_message(
            %{
              message: %{
                text: "/" <> unquote(command) <> " " <> _
              }
            } = var!(update)
          ) do
        handle_message(unquote(handler), [var!(update)])
      end

      def do_match_message(
            %{
              message: %{
                text: "/" <> unquote(command) <> "@" <> unquote(@bot_name)
              }
            } = var!(update)
          ) do
        handle_message(unquote(handler), [var!(update)])
      end

      def do_match_message(
            %{
              message: %{
                text: "/" <> unquote(command) <> "@" <> unquote(@bot_name) <> " " <> _
              }
            } = var!(update)
          ) do
        handle_message(unquote(handler), [var!(update)])
      end
    end
  end
```
上面是`Router`核心的代码，`Poller`收到消息就调用`do_match_message`对其进行处理。
这个库把用户提供的`command`和`bot_name`来重载`do_match_message`函数，对`update`做 pattern matching，从而把消息 route 到相应的 handler。
用 macro 包装一下，用户需要写的代码就成了这样，减少了大量 boilerplate ：
```Elixir
  command ["hello", "hi"] do
    Logger.log :info, "Command /hello or /hi"

    send_message "Hello World!"
end
```


### Erlang/OTP
Elixir 是 Erlang VM 上的语言，自然是要跟 OTP 打交道的。但是 OTP 是什么？
> OTP (Open Telecom Platform) is a set of libraries that ships with Erlang. Erlang developers use OTP to build robust, fault-tolerant applications.
事实上很大程度上我用 Elixir 就是因为 Erlang/OTP，否则语言设计得再妙，没了 Erlang VM 的支撑我大概也不会去尝试。Erlang 提供了有如操作系统一般的 VM，对于分布式 / 并发的问题有非常独到的解决方法。而 Erlang 略显鬼畜的语法是我迟迟没有去接触它的原因，Elixir 让我有机会这一窥究竟。
这一部分我理解还是比较浅薄，而且 Erlang/OTP 确实比较复杂，推荐读陈天写的[介绍文章](https://zhuanlan.zhihu.com/p/26341437)。


### Mix
8102 年了，各种语言基本都把好用的构建工具和编译器一起 ship。mix 提供了**创建项目，包管理，编译，测试，文档，格式化代码**一条龙服务。
而且它是 extensible，用户可以自定义 task。下面提到的 linter 就是自定义的 task。
P.S. 我蛮喜欢 mix 这个名字，因为 Elixir 是炼金术术语，意思是万能药，mix 像是混合试剂的意思。

# 开发环境配置
### 安装编译器以及文档：
```bash
pacman -S elixir elixir-docs
```
`elixir`主要包含了编译器 (`elixirc`), REPL(`iex`) 以及构建工具 (`mix`).
`elixir-docs`在`archlinuxcn`源里（我打的包），包含了 Elixir 自带的库以及工具链的文档。
第三方库可以到 [hexdocs](https://hexdocs.pm) 查，或者用
```bash
mix hex.doc open PACKAGE [VERSION]
```

### 装 Vim 插件：
```VimL
Plug 'elixir-editors/vim-elixir'                    "语法高亮，缩进规则
Plug 'mhinz/vim-mix-format', { 'for': 'elixir' }    "格式化代码，需要 elixir1.6
Plug 'slashmili/alchemist.vim', { 'for': 'elixir' } "补全及跳转
```
BTW, Elixir 也有 LSP 实现（https://github.com/JakeBecker/elixir-ls）。Vim 对 LSP 的支持还不够好，用 VS Code 的话可以试试。

### Linter
Elixir 可以用 [credo](https://github.com/rrrene/credo) 做静态分析，在 mix.exs 的 deps 里加一行即可：
```elixir
defp deps do
  [
      {:credo, "~> 0.9.0-rc1", only: [:dev, :test], runtime: false}
  ]
end
```

### 其它
Elixir 的包托管在 [hex](https://hex.pm)，由于众所周知的问题，在中国大陆访问速度很慢。请心里默念「FUCK GFW」并在`~/.hex/hex.config`加一行代理：
```elixir
{http_proxy,«"http://127.0.0.1:1081"»}.
```

# 总结
Elixir 背靠 Erlang/OTP，同时语言设计精良，工具链完善，适合写健壮的，scalable 的应用，值得一试。
