# 猜猜看

> [ch02-00-guessing-game-tutorial.md](https://github.com/rust-lang/book/blob/master/src/ch02-00-guessing-game-tutorial.md)
> <br>
> commit 7c1c935560190fcd64c0851e75dbeabf75fedd19

让我们通过自己动手的方式一起完成一个项目来快速上手 Rust！本章通过展示如何在真实的项目中运用的方式向你介绍一些常用的 Rust 概念。你将会学到`let`、`match`、方法、关联函数、使用外部 crate 等更多的知识！接下来的章节会探索这些概念的细节。在这一章，我们练习基础。

我们会实现一个经典新手编程问题：猜猜看游戏。它是这么工作的：程序将会随机生成一个 1 到 100 之间的随机整数。接着它会提示玩家输入一个猜测。当输入了一个猜测后，它会告诉提示猜测是太大了还是太小了。猜对了，它会打印出祝贺并退出。

## 准备一个新项目

要创建一个新项目，进入你在第一章创建的**项目**目录，并使用 Cargo 创建它，像这样：

```sh
$ cargo new guessing_game --bin
$ cd guessing_game
```

第一个命令，`cargo new`，获取项目的名称（`guessing_game`）作为第一个参数。`--bin`参数告诉 Cargo 创建一个二进制项目，与第一章类似。第二个命令进入到新创建的项目目录。

看一样生成的 *Cargo.toml* 文件：

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
```

如果 Cargo 从环境中获取的作者信息不正确，修改这个文件并再次保存。

正如第一章那样，`cargo new`生成了一个“Hello, world!”程序。查看 *src/main.rs* 文件：

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

现在让我们使用`cargo run`在相同的步骤编译并运行这个“Hello, world!”程序：


```sh
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
     Running `target/debug/guessing_game`
Hello, world!
```

`run`命令在你需要快速迭代项目时就派上用场了，而这个游戏就正是这么一个项目：我们需要在进行下一步之前快速测试每次迭代。

重新打开 *src/main.rs* 文件。我们将会在这个文件编写全部的代码。

## 处理一次猜测

程序的第一部分会请求用户输入，处理输入，并检查输入是否为期望的形式。首先，允许玩家输入一个猜测。在 *src/main.rs* 中输入列表 2-1 中的代码。

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

<figcaption>

Listing 2-1: Code to get a guess from the user and print it out

</figcaption>
</figure>

这些代码包含很多信息，所以让我们一点一点地过一遍。为了获取用户输入并接着打印结果作为输出，我们需要将`io`（输入/输出）库引入作用域中。`io`库来自于标准库（也被称为`std`）：

```rust,ignore
use std::io;
```

Rust 默认只在每个程序的 [*prelude*][prelude]<!-- ignore --> 中引用很少的一些类型。如果想要使用的类型并不在 prelude 中，你必须使用一个`use`语句显式的将其引入到作用域中。使用`std::io`库将提供很多`io`相关的功能，接受用户输入的功能。

[prelude]: https://doc.rust-lang.org/std/prelude/

正如第一章所讲，`main`函数是程序的入口点：

```rust,ignore
fn main() {
```

`fn`语法声明了一个新函数，`()`表明没有参数，`{`作为函数体的开始。

第一章也讲到了，`println!`是一个在屏幕上打印字符串的宏：

```rust,ignore
println!("Guess the number!");

println!("Please input your guess.");
```

这些代码仅仅打印一个提示，说明游戏的内容并请求用户输入。

### 用变量储存值

接下来，创建一个地方储存用户输入，像这样：

```rust,ignore
let mut guess = String::new();
```

现在程序开始变得有意思了！这一小行代码发生了很多事。注意这是一个`let`语句，用来创建**变量**。这里是另外一个例子：

```rust,ignore
let foo = bar;
```

这行代码会创建一个叫做`foo`的新变量并把它绑定到值`bar`上。在 Rust 中，变量默认是不可变的。下面的例子展示了如何在变量名前使用`mut`来使一个变量可变：

```rust
let foo = 5; // immutable
let mut bar = 5; // mutable
```

> 注意：`//` 开始一个注释，它持续到本行的结尾。Rust 忽略注释中的所有内容。

现在我们知道了`let mut guess`会引入一个叫做`guess`的可变变量。等号（`=`）的另一边是`guess`所绑定的值，它是`String::new`的结果，这个函数会返回一个`String`的新实例。[`String`][string]<!-- ignore -->是一个标准库提供的字符串类型，它是可增长的、UTF-8 编码的文本块。

[string]: ../std/string/struct.String.html

`::new`那一行的`::`语法表明`new`是`String`类型的一个**关联函数**（*associated function*）。关联函数是针对类型实现的，在这个例子中是`String`，而不是`String`的某个特定实例。一些语言中把它称为**静态方法**（*static method*）。

`new`函数创建了一个新的空的`String`，你会在很多类型上发现`new`函数，因为这是创建某个类型新值的常用函数名。

总结一下，`let mut guess = String::new();`这一行创建了一个可变变量，目前它绑定到一个`String`新的、空的实例上。哟！

回忆一下我们在程序的第一行使用`use std::io;`从标准库中引用输入/输出功能。现在在`io`上调用一个关联函数，`stdin`：

```rust,ignore
io::stdin().read_line(&mut guess)
    .expect("Failed to read line");
```

如果我们在程序的开头没有`use std::io`这一行，我们可以把函数调用写成`std::io::stdin`这样。`stdin`函数返回一个 [`std::io::Stdin`][iostdin]<!-- ignore -->的实例，这是一个代表终端标准输入句柄的类型。

[iostdin]: ../std/io/struct.Stdin.html

代码的下一部分，`.read_line(&mut guess)`，调用 [`read_line`][read_line]<!-- ignore --> 方法从标准输入句柄获取用户输入。我们还向`read_line()`传递了一个参数：`&mut guess`。

[read_line]: ../std/io/struct.Stdin.html#method.read_line

`read_line`的工作是把获取任何用户键入到标准输入的字符并放入一个字符串中，所以它获取字符串作为一个参数。这个字符串需要是可变的，这样这个方法就可以通过增加用户的输入来改变字符串的内容。

`&`表明这个参数是一个**引用**（*reference*），它提供了一个允许多个不同部分的代码访问同一份数据而不需要在内存中多次拷贝的方法。引用是一个复杂的功能，而 Rust 的一大优势就是它是安全而优雅操纵引用。完成这个程序并不需要知道这么多细节：第四章会更全面的解释引用。现在，我们只需知道它像变量一样，默认是不可变的。因此，需要写成`&mut guess`而不是`&guess`来使其可变。

这行代码还没有分析完。虽然这是单独一行代码，但它只是一个逻辑上代码行（虽然换行了但仍是一个语句）的第一部分。第二部分是这个方法：

```rust,ignore
.expect("Failed to read line");
```

当使用`.foo()`语法调用方法时，明智的选择是换行并留出空白（缩进）来把长的代码行拆开。我们可以把代码写成这样：

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

不过，过长的代码行难以阅读，所以最好拆开来写，两行代码两个方法调用。现在来看看这行代码干了什么。

### 使用`Result`类型来处理潜在的错误

之前提到过，`read_line`将用户输入放入到传递给它字符串中，不过它也返回一个值————一个[`io::Result`][ioresult]<!-- ignore -->。Rust 标准库中有很多叫做`Result`的类型。一个[`Result`][result]<!-- ignore -->泛型以及对应子模块的特定版本，比如`io::Result`。

[ioresult]: ../std/io/type.Result.html
[result]: ../std/result/enum.Result.html

`Result`类型是 [*枚举*（*enumerations*）][enums]<!-- ignore -->，通常也写作 *enums*。枚举拥有固定值集合的类型，而这些值被称为枚举的**成员**（*variants*）。第六章会更详细的介绍枚举。

[enums]: ch06-00-enums.html

对于`Result`，它的成员是`Ok`或`Err`，`Ok`表明操作成功了，同时`Ok`成员之中包含成功生成的值。`Err`意味着操作失败，`Err`之中包含操作是为什么或如何失败的信息。

`Result`类型的作用是编码错误处理信息。`Result`类型的值，正如其他任何类型，拥有定义于其上的方法。`io::Result`的实例拥有[`expect`方法][expect]<!-- ignore -->可供调用。如果`io::Result`实例的值是`Err`，`expect`会导致程序崩溃并显示显示你作为参数传递给`expect`的信息。如果`io::Result`实例的值是`Ok`，`expect`会获取`Ok`中的值并原原本本的返回给你，这样就可以使用它了。在本例中，返回值是用户输入到标准输入的一些字符。

[expect]: ../std/result/enum.Result.html#method.expect

如果不使用`expect`，程序也能编译，不过会出现一个警告：

```sh
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
src/main.rs:10:5: 10:39 warning: unused result which must be used,
#[warn(unused_must_use)] on by default
src/main.rs:10     io::stdin().read_line(&mut guess);
                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Rust 警告说我们没有使用`read_line`返回的值`Result`，表明程序没有处理一个可能的错误。消除警告的正确方式是老实编写错误处理，不过因为我们仅仅希望程序出现问题就崩溃，可以使用`expect`。你会在第九章学习从错误中恢复。

### 使用`println!`占位符打印值

除了位于结尾的大括号，目前为止编写的代码就只有一行代码值得讨论一下了，就是这一行：

```rust,ignore
println!("You guessed: {}", guess);
```

这行代码打印出存储了用户输入的字符串。这对`{}`是一个在特定位置预留值的占位符。可以使用`{}`打印多个值：第一个`{}`对应格式化字符串之后列出的第一个值，第二个对应第二个值，以此类推。用一个`println!`调用打印多个值应该看起来像这样：

```rust
let x = 5;
let y = 10;

println!("x = {} and y = {}", x, y);
```

这行代码会打印出`x = 5 and y = 10`。

### 测试第一部分代码

让我们来测试下猜猜看游戏的第一部分。使用`cargo run`运行它：

```sh
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

至此为止，游戏的第一部分已经完成：我们从键盘获取了输入并打印了出来。

## 生成一个秘密数字

接下来，需要生成一个秘密数字，用户会尝试猜测它。秘密数字应该每次都不同，这样多玩几次才会有意思。生成一个 1 到 100 之间的随机数这样游戏也不会太难。Rust 标准库中还未包含随机数功能。然而，Rust 团队确实提供了一个[`rand` crate][randcrate]。

[randcrate]: https://crates.io/crates/rand

## 使用 crate 来增加更多功能

记住 *crate* 是一个 Rust 代码的包。我们正在构建的项目是一个**二进制 crate**，它生成一个可执行文件。 `rand` crate 是一个 *库 crate*，它包含意在被其他程序使用的代码。

Cargo 对外部 crate 的运用是其真正闪光的地方。在我们可以使用`rand`编写代码之前，需要编辑 *Cargo.toml* 来包含`rand`作为一个依赖。现在打开这个文件并在`[dependencies]`部分标题（Cargo 为你创建了它）的下面添加如下代码：

<span class="filename">Filename: Cargo.toml</span>

```toml
[dependencies]

rand = "0.3.14"
```

在 *Cargo.toml* 文件中，任何标题之后的内容都是属于这个部分的，一直持续到直到另一个部分开始。`[dependencies]`部分告诉 Cargo 项目依赖了哪个外部 crate 和需要的 crate 版本。在这个例子中，我们使用语义化版本符号`0.3.14`来指定`rand`crate。Cargo 理解[语义化版本（Semantic Versioning）][semver]<!-- ignore -->（有时也称为 *SemVer*），这是一个编写版本号的标准。版本号`0.3.14`事实上是`^0.3.14`的缩写，它的意思是“任何与 0.3.14 版本公有 API 相兼容的版本”。

[semver]: http://semver.org

现在，不用修改任何代码，构建项目，如列表 2-2：

<figure>

```text
$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rand v0.3.14
 Downloading libc v0.2.14
   Compiling libc v0.2.14
   Compiling rand v0.3.14
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
```

<figcaption>

Listing 2-2: The output from running `cargo build` after adding the rand crate
as a dependency

</figcaption>
</figure>

可能会出现不同的版本号（不过多亏了语义化版本，它们与代码是兼容的！），同时显示顺序也可能会有所不同。

现在我们有了一个外部依赖，Cargo 从 *registry* （[Crates.io][cratesio]）上获取了一份（兼容的）最新版本代码的拷贝。Crates.io 是 Rust 生态环境中的人们向他人贡献他们的开源 Rust 项目的地方。

[cratesio]: https://crates.io

在更新完 registry （索引）后，Cargo 检查`[dependencies]`部分并下载还不存在部分。在这个例子中，虽然只列出了`rand`一个依赖，Cargo 也获取了一份`libc`的拷贝，因为`rand`依赖`libc`来正常工作。在下载他们之后，Rust 编译他们接着用这些依赖编译项目。

如果不做任何修改就立刻再次运行`cargo build`，则不会有任何输出。Cargo 知道它已经下载并编译了依赖，同时 *Cargo.toml* 文件中也没有任何相关修改。Cargo 也知道代码没有做任何修改，所以它也不会重新编译代码。因为无事可做，它简单的退出了。如果打开 *src/main.rs* 文件，并做一些普通的修改，保存并再次构建，只会出现一行输出：

```sh
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
```

这一行表明 Cargo 只构建了对 *src/main.rs* 文件做出的微小修改。依赖没有被修改，所以 Cargo 知道可以复用已经为此下载并编译的代码。它只是重新构建了部分（项目）代码。

#### The *Cargo.lock* 文件确保构建是可重现的

Cargo 有一个机制来确保每次任何人重新构建代码都会生成相同的成品：Cargo 只会使用你指定的依赖的版本，除非你又手动指定了别的。例如，如果下周`rand` crate 的`v0.3.15`版本出来了，而它包含一个重要的 bug 修改并也含有一个会破坏代码运行的缺陷的时候会发生什么呢？

这个问题的答案是 *Cargo.lock* 文件，它在第一次运行`cargo build`时被创建并位于 *guessing_game* 目录。当第一次构建项目时，Cargo 计算出所有符合要求的依赖版本并接着写入 *Cargo.lock* 文件中。当将来构建项目时，Cargo 发现 *Cargo.lock* 存在就会使用这里指定的版本，而不是重新进行所有版本的计算。这使得你拥有了一个自动的可重现的构建。换句话说，项目会继续使用`0.3.14`直到你显式升级，多亏了 *Cargo.lock* 文件。我们将会在这个文件编写全部的代码。

#### 更新 crate 到一个新版本

当你**确实**需要升级 crate 时，Cargo 提供了另一个命令，`update`，他会：

1. 忽略 *Cargo.lock* 文件并计算出所有符合 *Cargo.toml* 中规格的最新版本。
2. 如果成功了，Cargo 会把这些版本写入 *Cargo.lock* 文件。

不过，Cargo 默认只会寻找大于`0.3.0`而小于`0.4.0`的版本。如果`rand` crate 发布了两个新版本，`0.3.15`和`0.4.0`，在运行`cargo update`时会出现如下内容：

```sh
$ cargo update
    Updating registry `https://github.com/rust-lang/crates.io-index`
    Updating rand v0.3.14 -> v0.3.15
```

这时，值得注意的是 *Cargo.lock* 文件中的一个改变，`rand` crate 现在使用的版本是`0.3.15`。

如果想要使用`0.4.0`版本的`rand`或是任何`0.4.x`系列的版本，必须像这样更新 *Cargo.toml* 文件：

```toml
[dependencies]

rand = "0.4.0"
```

下一次运行`cargo build`时，Cargo 会更新 registry 中可用的 crate 并根据你指定新版本重新计算`rand`的要求。

第十四章会讲到[Cargo][doccargo]<!-- ignore --> 和[它的生态系统][doccratesio]<!-- ignore -->的更多内容，不过目前你只需要了解这么多。Cargo 使得复用库文件变得非常容易，所以 Rustacean 们能够通过组合很多包来编写出更轻巧的项目。

[doccargo]: http://doc.crates.io
[doccratesio]: http://doc.crates.io/crates-io.html

### 生成一个随机数

让我们开始**使用**`rand`。下一步是更新 *src/main.rs*，如列表 2-3：

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate rand;

use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

<figcaption>

Listing 2-3: Code changes needed in order to generate a random number

</figcaption>
</figure>

我们在顶部增加一行`extern crate rand;`来让 Rust 知道我们要使用外部依赖。这也会调用相应的`use rand`，所以现在可以使用`rand::`前缀来调用`rand`中的任何内容。

接下来，我们增加了另一行`use`：`use rand::Rng`。`Rng`是一个定义了随机数生成器应实现方法的 trait，如果要使用这些方法的话这个 trait 必须在作用域中。第十章会详细介绍 trait。

另外，中间还新增加了两行。`rand::thread_rng`函数会提供具体会使用的随机数生成器：它位于当前执行线程本地并从操作系统获取 seed。接下来，调用随机数生成器的`gen_range`方法。这个方法由我们使用`use rand::Rng`语句引入到作用域的`Rng` trait 定义。`gen_range`方法获取两个数作为参数并生成一个两者之间的随机数。它包含下限但不包含上限，所以需要指定`1`和`101`来请求一个`1`和`100`之间的数。

并不仅仅能够知道该 use 哪个 trait 和该从 crate 中调用哪个方法。如何使用 crate 的说明在每个 crate 的文档中。Cargo 另一个很棒的功能是可以运行`cargo doc --open`命令来构建所有本地依赖提供的文档并在浏览器中打开。例如，如果你对`rand` crate 中的其他功能感兴趣，运行`cargo doc --open`并点击左侧导航栏的`rand`。

新增加的第二行代码打印出了秘密数字。这在开发程序时很有用，因为我们可以去测试它，不过在最终版本我们会删掉它。游戏一开始就打印出结果就没什么可玩的了！

尝试运行程序几次：

```sh
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4
$ cargo run
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

你应该能得到不同的随机数，同时他们应该都是在 1 和 100 之间的。干得漂亮！

## 比较猜测与秘密数字

现在有了用户输入和一个随机数，我们可以比较他们。这个步骤如列表 2-4：

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less    => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal   => println!("You win!"),
    }
}
```

<figcaption>

Listing 2-4: Handling the possible return values of comparing two numbers

</figcaption>
</figure>

新代码的第一行是另一个`use`，从标准库引入了一个叫做`std::cmp::Ordering`的类型到作用域。`Ordering`是另一个枚举，像`Result`一样，不过`Ordering`的成员是`Less`、`Greater`和`Equal`。这是你比较两个值时可能出现三种结果。

接着在底部的五行新代码使用了`Ordering`类型：

```rust,ignore
match guess.cmp(&secret_number) {
    Ordering::Less    => println!("Too small!"),
    Ordering::Greater => println!("Too big!"),
    Ordering::Equal   => println!("You win!"),
}
```

`cmp`方法比较两个值并可以在任何可比较的值上调用。它获取一个任何你想要比较的值的引用：这里是把`guess`与`secret_number`做比较。`cmp`返回一个使用`use`语句引入作用域的`Ordering`枚举的成员。我们使用一个[`match`][match]<!-- ignore -->表达式根据对`guess`和`secret_number`中的值调用`cmp`后返回的哪个`Ordering`枚举成员来决定接下来干什么。

[match]: ch06-02-match.html

一个`match`表达式由 **分支（arms）** 构成。一个分支包含一个 **模式**（*pattern*）和代码，这些代码在`match`表达式开头给出的值符合分支的模式时将被执行。Rust 获取提供给`match`的值并挨个检查每个分支的模式。`match`结构和模式是 Rust 中非常强大的功能，它帮助你体现代码可能遇到的多种情形并帮助你处理全部的可能。这些功能将分别在第六章和第十九章详细介绍。

让我们看看一个使用这里的`match`表达式会发生什么的例子。假设用户猜了 50，这时随机生成的秘密数字是 38。当代码比较 50 与 38 时，`cmp`方法会返回`Ordering::Greater`，因为 50 比 38 要大。`Ordering::Greater`是`match`表达式得到的值。它检查第一个分支的模式，`Ordering::Less`，不过值`Ordering::Greater`并不匹配`Ordering::Less`。所以它忽略了这个分支的代码并移动到下一个分支。下一个分支的模式，`Ordering::Greater`，**正确**匹配了`Ordering::Greater`！这个分支关联的代码会被执行并在屏幕打印出`Too big!`。`match`表达式就此终止，因为在这个特定场景下没有检查最后一个分支的必要。

然而，列表 2-4 的代码并不能编译，尝试一下：

```sh
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
error[E0308]: mismatched types
  --> src/main.rs:23:21
   |
23 |     match guess.cmp(&secret_number) {
   |                     ^^^^^^^^^^^^^^ expected struct `std::string::String`, found integral variable
   |
   = note: expected type `&std::string::String`
   = note:    found type `&{integer}`

error: aborting due to previous error
Could not compile `guessing_game`.
```

错误的核心表明这里有**不匹配的类型**（*mismatched types*）。Rust 拥有一个静态强类型系统。不过，它也有类型推断。当我们写出`let guess = String::new()`时，Rust 能够推断出`guess`应该是一个`String`，并不需要我们写出类型。另一方面，`secret_number`，是一个数字类型。一些数字类型拥有 1 到 100 之间的值：`i32`，一个 32 位的数字；`u32`，一个 32 位无符号数字；`i64`，一个 64 位数字；等等。Rust 默认使用`i32`，所以`secret_number`的类型就是它，除非增加类型信息或任何能让 Rust 推断出不同数值类型的信息。这里错误的原因是 Rust 不会比较字符串类型和数字类型。

最终我们想要把程序从输入中读取到的`String`转换为一个真正的数字类型，这样好与秘密数字向比较。可以通过在`main`函数体中增加如下两行代码来实现：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    let guess: u32 = guess.trim().parse()
        .expect("Please type a number!");

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less    => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal   => println!("You win!"),
    }
}
```

这两行代码是：

```rust,ignore
let guess: u32 = guess.trim().parse()
    .expect("Please type a number!");
```

这里创建了一个叫做`guess`的变量。不过等等，难道这个程序不是已经有了一个叫做`guess`的变量了吗？确实如此，不过 Rust 允许我们通过**覆盖**（*shadow*） 用一个新值来覆盖`guess`之前的值。这个功能经常用在类似需要把一个值从一种类型转换到另一种类型的场景。shadowing 允许我们复用`guess`变量的名字而不是强迫我们创建两个不同变量，比如`guess_str`和`guess`。（第三章会介绍 shadowing 的更多细节。）

`guess`被绑定到`guess.trim().parse()`表达式。表达式中的`guess`对应包含输入的`String`类型的原始`guess`。`String`实例的`trim`方法会消除字符串开头和结尾的空白。`u32`只能包含数字字符。不过用户必须输入回车键才能让`read_line`返回。当用户按下回车键时，会在字符串中增加一个换行（newline）字符。例如，如果用户输入 5 并回车，`guess`看起来像这样：`5\n`。`\n`代表“换行”，回车键。`trim`方法消除`\n`，只留下`5`。

[字符串的`parse`方法][parse]<!-- ignore -->解析一个字符串成某个数字。因为这个方法可以解析多种数字类型，需要告诉 Rust 我们需要的具体的数字类型，这里通过`let guess: u32`指定。`guess`后面的冒号（`:`）告诉 Rust 我们指明了变量的类型。Rust 有一些内建的数字类型；这里的`u32`是一个无符号的 32 位整型。它是一个好的较小正整数的默认类型。第三章会讲到其他数字类型。另外，例子程序中的`u32`注解和与`secret_number`的比较意味着 Rust 会推断`secret_number`应该是也是`u32`类型。现在可以使用相同类型比较两个值了！

[parse]: ../std/primitive.str.html#method.parse

`parse`调用容易产生错误。例如，如果字符串包含`A👍%`，就无法将其转换为一个数字。因为它可能失败，`parse`方法返回一个`Result`类型，非常像之前在 XX 页“使用`Result`类型来处理潜在的错误”部分讨论的`read_line`方法。这里再次类似的使用`expect`方法处理这个`Result`类型。如果`parse`因为不能从字符串生成一个数字而返回一个`Err`的`Result`成员时，`expect`会使游戏崩溃并打印提供给它的信息。如果`parse`能成功地将字符串转换为一个数字，它会返回`Result`的`Ok`成员，同时`expect`会返回`Ok`中我们需要的数字。

现在让我们运行程序！

```sh
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

漂亮！即便是在猜测之前添加了空格，程序依然能判断出用户猜测了 76。多运行程序几次来检验不同类型输入的相应行为：猜一个正确的数字，猜一个过大的数字和猜一个过小的数字。

现在游戏已经大体上能玩了，不过用户只能猜一次。增加一个循环来改变它吧！

## 使用循环来允许多次猜测

`loop`关键字提供了一个无限循环。增加它后给了用户多次猜测的机会：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = guess.trim().parse()
            .expect("Please type a number!");

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => println!("You win!"),
        }
    }
}
```

如上所示，我们将提示用户猜测之后的所有内容放入了循环。确保这些代码多缩进了四个空格，并再次运行程序。注意这里有一个新问题，因为程序忠实地执行了我们要求它做的：永远地请求另一个猜测！看起来用户没法退出啊！

用户总是可以使用`Ctrl-C`快捷键来终止程序。不过这里还有另一个逃离这个贪得无厌的怪物的方法，就是在 XX 页“比较猜测”部分提到的`parse`：如果用户输入一个非数字回答，程序会崩溃。用户可以利用这一点来退出，如下所示：

```sh
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit
thread 'main' panicked at 'Please type a number!: ParseIntError { kind: InvalidDigit }', src/libcore/result.rs:785
note: Run with `RUST_BACKTRACE=1` for a backtrace.
error: Process didn't exit successfully: `target/debug/guess` (exit code: 101)
```

输入`quit`就会退出程序，同时其他任何非数字输入也一样。然而，毫不夸张的说这是不理想的。我们想要当猜测正确的数字时游戏能自动退出。

### 猜测正确后退出

让我们增加一个`break`来在用户胜利时退出游戏：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = guess.trim().parse()
            .expect("Please type a number!");

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

通过在`You win!`之后增加一行`break`，程序在用户猜对了神秘数字后会退出循环。退出循环也就意味着退出程序，因为循环是`main`的最后一部分。

### 处理无效输入

为了进一步改善游戏性，而不是在用户输入非数字时崩溃，需要让游戏忽略非数字从而用户可以继续猜测。可以通过修改`guess`从`String`转化为`u32`那部分代码来实现：

```rust,ignore
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

从`expect`调用切换到`expect`语句是如何从遇到错误就崩溃到真正处理错误的常用手段。记住`parse`返回一个`Result`类型，而`Result`是一个拥有`Ok`或`Err`两个成员的枚举。在这里使用`match`表达式，就像之前处理`cmp`方法返回的`Ordering`一样。

如果`parse`能够成功的将字符串转换为一个数字，它会返回一个包含结果数字`Ok`值。这个`Ok`值会匹配第一个分支的模式，这时`match`表达式仅仅返回`parse`产生的`Ok`值之中的`num`值。这个数字会最终如期变成新创建的`guess`变量。

如果`parse`*不*能将字符串转换为一个数字，它会返回一个包含更多错误信息的`Err`值。`Err`值不能匹配第一个`match`分支的`Ok(num)`模式，但是会匹配第二个分支的`Err(_)`模式。`_`是一个包罗万象的值；在这个例子中，我们想要匹配所有`Err`值，不管其中有何种信息。所以程序会执行第二个分支的代码，`continue`，这意味着进入`loop`的下一次循环并请求另一个猜测。这样程序就有效地忽略了`parse`可能遇到的所有错误！

现在万事俱备（只欠东风）了。运行`cargo run`来尝试一下：

```sh
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

太棒了！再有最后一个小的修改，就能完成猜猜看游戏了：还记得程序依然会打印出秘密数字。这在测试时还好，但会毁了游戏性。删掉打印秘密数字的`println!`。列表 2-5 为最终代码：

<figure>
<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

<figcaption>

Listing 2-5: Complete code of the guessing game

</figcaption>
</figure>

## 总结一下，

此时此刻，你顺利完成了猜猜看游戏！恭喜！

这是一个通过动手实践的方式想你介绍许多 Rust 新知识的项目：`let`、`match`、方法、关联函数，使用外部 crate，等等。接下来的几章，我们将会详细学习这些概念。第三章涉及到大部分编程语言都有的概念，比如变量、数据类型和函数，以及如何在 Rust 中使用他们。第四章探索所有权（ownership），这是一个 Rust 同其他语言都不相同的功能。第五章讨论结构体和方法语法，而第六章侧重解释枚举。