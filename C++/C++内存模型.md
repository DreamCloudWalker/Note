C++内存区域划分（栈区，堆区，代码区，全局区（静态区，常量区，字符串区））

* 默认构造函数执行时，会在栈区开辟空间，暴露地址（this）；
* 静态区没有this区分；



编译命令看内存模型：

```shell
g++ -fdump-class-hierarchy xxx.cpp -o xxx
```

执行上面的命令会吧.cpp文件编译成.class文件

但要注意g++版本。上面的命令是在gcc 8.0版本之前。如果是gcc 8.0之后，要用

```shell
g++ -fdump-lang-class xxx.cpp
```

如果是在mac平台，比如g++ --version打印： Apple clang version 14.0.3 (clang-1403.0.22.14.1)

`Apple clang` 是 Apple 提供的 Clang 编译器，通常其支持的选项和功能与标准 GCC 编译器有所不同。`-fdump-class-hierarchy` 是 `g++` (GNU C++) 编译器的一个特有选项，用于生成类的层次结构信息。

在你的情况下，由于你正在使用的是 `Apple clang version 14.0.3`，这实际上是 Clang 而不是 GNU `g++`。Clang 不支持 `-fdump-class-hierarchy` 选项。因此，你不能在使用 Clang 的情况下获得这样的输出。

Clang 在类层次结构方面的替代方案

如果你想获取关于类层次结构的信息，Clang 提供了一些其他的工具和选项：

1. **使用 Clang 的静态分析工具**：例如，Clang 的 AST (抽象语法树) 工具可以帮助分析和检查代码结构。可以使用工具如 `clang-query` 或 `clang-tidy`。

2. **使用 `clang -Xclang -ast-dump`**：这个选项可以获取类及其成员的抽象语法树视图。例如：

   ```
   clang++ -Xclang -ast-dump -fsyntax-only example.cpp
   ```

   这条命令会输出 AST 信息，包括类的结构。

3. **使用 `libclang`**：如果你需要程序化地访问源代码结构，可以考虑使用 `libclang`，它提供了 C API 来解析 C/C++ 代码。