> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [dlonng.com](https://dlonng.com/posts/makefile)

> Linux 高级编程 - Makefile 基础语法 版权声明：本文为 DLonng 原创文章，可以随意转载，但必须在明确位置注明出处！Makefile 简介 Makefile 是一个管理项目的配置文件......

> 版权声明：本文为 DLonng 原创文章，可以随意转载，但必须在明确位置注明出处！

Makefile 简介
-----------

`Makefile` 是一个管理项目的配置文件，它主要有 2 个作用：

1.  组织工程文件，编译成复杂的程序
2.  安装及卸载程序

Makefile 是被 `make` 这个程序执行的，当执行 `make` 时，如果发现当前目录下有 `Makefile` 或者 `makefile` 文件，那么 `make` 命令就会根据这个文件的内容来找到相关的文件，然后去执行编译，链接，安装，卸载等操作，这就实现了使用 `make + Makefile` 来管理项目的功能。

这篇文章主要是介绍编写 Makefile 的基础语法，帮助你能够编写和理解一般的 Makefile 文件。

如何系统的学习 Makefile
----------------

想要系统的学习 Makefile，需要 2 个条件：

1.  Makefile 官网帮助文档：[Makefile](https://www.gnu.org/software/make/manual/make.html#Introduction)
2.  大量练习编写 Makefile：想要系统学习，必须自己练习

因为 `make` 是 `GNU` 开发的开源软件，所以官网的资料可以说是最好的，这也是开源的力量，这篇文章只是带你入门，想要精通还要自己努力。

make 的工作流程
----------

在学习 Makefile 语法之前，我们先来学习下 `make` 是如何解析 Makefile 文件的，主要分为下面 4 个步骤：

1.  在当前目录下查找 Makefile 或者 makefile 的文件
2.  找到之后，解析并得到最终要生成的目标文件
3.  根据时间戳生成目标文件
4.  递归去寻找其他目标文件的依赖文件，并递归生成

只需要了解这个步骤即可，这是为帮助你理解执行 `make` 后发生了什么事情，清楚自己在做些什么。

第一个 Makefile
------------

我们先写一个最简单的 Makefile 来熟悉**编译，执行，清理，安装，卸载** `Hello World` 的操作，以此来熟悉通用的过程。

### 编写 hello.c

这个程序打印 `Hello World`，非常简单：

```
#include <stdio.h>

int main(void) {
	printf("Hello World!\n");
	return 0;
}


```

### 编写 Makefile

这个 Makefile 其中包含的就是基本的 `gcc` 编译指令和一些 `shell` 指令：

```
hello:
	gcc hello.c -o hello

clean:
	rm ./hello

install:
	cp ./hello /usr/local/bin/hello

uninstall:
	rm /usr/local/bin/hello


```

**注意：使用 `Table` 键来缩进，否则会出现语法错误**！

### 编译

现在有了 Makefile，我们就可以直接在终端键入 `make` 来编译啦：

```
make

# make 之后打印的信息
gcc hello.c -o hello


```

我们看到只有一条打印信息，就是我们写的那条编译指令，联想到我们手动编译别人的软件时，我们也是键入 `make` 然后屏幕就被一大串信息刷屏了，其实那些信息跟这个本质上是一样，也是 Makefile 中的指令。

### 执行

执行 `./hello`：

```
./hello

# 结果
Hello World!


```

可见这种方法更加简单了，不仅如此，还可以很容易清理可执行文件。

### 清理

还记得我们在 Makefile 中写了一个 `clean:` 吗，它下面还有一条删除文件的指令，我们在终端键入：

```
make clean

# make clean 打印的信息
rm ./hello


```

可以看到 `make clean` 帮助我们删除了 `hello`，是不是特别方便，以后再也不用敲那一大串 `gcc` 编译命令和手动删除文件了，效率瞬间又提升了。

### 安装

还记得使用 `sudo make install` 来安装程序吗？我们也在终端键入下面的命令：

```
sudo make install

# 提示输入 root 密码
[sudo] password for orange: 
cp ./hello /usr/local/bin/hello

# 直接执行 hello
hello

# 输出
Hello World!


```

可以看出**安装的过程其实就是拷贝程序的过程**。

### 卸载

卸载使用 `sudo make uninstall`，我们卸载 `hello`：

```
# 卸载
sudo make uninstall

rm /usr/local/bin/hello

# 再次执行 hello，提示没找到即已经删除
bash: /usr/local/bin/hello: No such file or directory


```

至此，我们已经了解使用一个简单的 Makefile 来编译，清理，安装，卸载程序的例子，下面就来学习一些常用的 Makefile 语法吧。 　　　

Makefile 基础语法
-------------

先来看看 Makefile 的编写规则。

### Makefile 编写规则

**Makefile 由若干条规则组成**，规则格式如下：

```
目标（target）: 依赖（prerequisites）
	命令（command）


```

例如：

```
main : main.o
	gcc main.o -o main

main.o : main.c
	gcc -c main.c -o main.o


```

其中 `main` 是最后生成的目标文件，`main.o` 是依赖文件，`gcc main.o -o main` 是编译命令，要注意的是 `main.o` 是由 `main.c` 生成的，所以还需要一条编译 `main.c` 生成 `main.o` 的规则，这些依赖关系规则共同组成了最后的 Makefile。

### Makefile 变量

Makefile 的变量分为 3 类：

1.  用户自定义变量
2.  预定义变量
3.  环境变量

#### 1. 用户自定义变量

定义格式如下：

使用 `$(VAR_NAME)`来引用变量，比如：

```
file_name = hello.c
hello:
	gcc $(file_name) -o hello


```

#### 2. 预定义变量

Makefile 常见的预定义变量有下面这些：

*   AR：库文件维护程序，默认为 `ar`
*   AS：汇编程序，默认为 `as`
*   CC：C 编译器，默认为 `cc`
*   CXX：C++ 编译器，默认为 `g++`
*   ARFLAGS：库文件维护程序选项，无默认值
*   ASFLAGS：汇编程序选项，无默认值
*   CFLAGS：C 编译器选项，无默认值
*   CXXFLAGS：C++ 编译器选项，无默认值

我们可以直接使用或者重新定义这些变量，来看个使用 `CFLAGS` 和 `CC` 的例子：

```
CFLAGS = -g
CC = gcc

hello:
	$(CC) $(CFLAGS) hello.c -o hello


```

这句编译指令就相当于 `gcc -g hello.c -o hello`，我们增加了 `[-g]` 编译参数，来看看实际 `make` 的效果：

```
make

# make 后的效果
gcc -g hello.c -o hello


```

#### 3. 环境变量

Makefile 常用的环境变量有下面这些：

*   `$*`：不包含扩展名的目标文件名称
*   `$<`：第一个依赖文件名称
*   `$?`：所有时间戳比目标文件晚的依赖文件
*   `$@`：目标文件完整名称
*   `$^`：所有不重复的依赖文件

这里我们以 `$^` 和 `$@` 为例：

```
hello : hello.o 
	gcc $^ -o $@

hello.o : hello.c
	gcc -c hello.c


```

看看 `make` 的结果：

```
make
# 一共执行了 2 条指令
gcc -c hello.c

gcc hello.o -o hello


```

可以发现 `$^ = hello.o`，`$@ = hello`，符合这两个预定义变量的定义。

### Makefile 伪目标

先来看一个伪目标：

```
.PHONY: install
install:
	cp hello /usr/local/bin/hello


```

伪目标 `install` 表示即使当前文件夹内有 `install` 这个文件，但是 `make` 执行的仍然是 `Makefile` 内部的 `install`，不会使用外部的文件，相当于进行了一个内部封装。

### Makefile 包含

一个 Makefile 可以包含另一个 Makefile，需要使用 `include` 指令，例如：

这里我们包含了当前目录下 `dir` 下的 Makefile 文件。

### Makefile 嵌套

在大型的项目中，我们经常需要一个 Makefile 来嵌套调用另一个目录下的 Makefile，这是可以使用下面的指令：

```
submake:
	cd dir && gcc hello.c -o hello


```

意思是**先进入 `dir`，之后执行后面的编译指令**。例如我们在 `dir` 目录下新建 `hello.c`，在上一级目录下使用上面的命令作为 Makefile，我们来 `make`：

```
make

# 结果
cd dir && gcc hello.c -o hello


```

### Makefile 条件判断

在 Makefile 中也可以进行条件判断，格式如下：

注意：**前面不能用 `table` 分隔**，来看一个例子：

```
CC = gcc

hello:  
ifeq ($(CC),gcc)
	gcc hello.c -o $@
else
	gcc hello.c -o hello2
endif


```

意思很容易理解，当 `CC = gcc` 时编译的输出文件名为 `hello`，否则为 `hello2`，我们看看 `make` 的结果：

```
# CC = gcc
make
gcc hello.c -o hello

# CC != gcc
make
gcc hello.c -o hello2


```

可以看到输出文件名是不同的，说明判断是有效的。

### Makeifle 管理命令

Makefile 有几个管理命令需要了解：

*   `[-C dir]`：读入指定目录下面的 Makefile
*   `[-f file]`: 读入当前目录下的 `file` 文件为 Makefile
*   `[-i]`：忽略所有命令执行错误
*   `[-l dir]`：指定被包含的 Makefile 所在的目录

我们这里介绍下前面两个：

比如，我们在 `dir` 目录下放了 `hello.c` 和 `Makefile` 2 个文件，使用下面的 `shell` 命令来在**上一级目录编译**：

```
make -C dir/

# make 结果
make: Entering directory '/home/orange/Desktop/Makefile/hello/dir'
gcc hello.c -o hello2
make: Leaving directory '/home/orange/Desktop/Makefile/hello/dir'


```

可以看到 `make` 先进入这个 `dir` 目录，之后编译，然后退出。

我们可以使用 `[-f]` 指定一个文件作为 Makefile，例如：

```
make -f cdeveloper.mk

gcc hello.c -o hello


```

`cdeveloper.mk` 内容就是普通 Makefile 的内容，只是名字变了而已，当然名字你可以随意改，但是**内容必须符合 Makefile 语法**。

结语
--

通过这篇博客，我们学习了 Makefile 的基础语法，如果需要更加系统的学习建议你看 [GNU Makefile 的官方文档](https://www.gnu.org/software/make/manual/make.html#Introduction) 来学习，这是最好最权威的学习资料，看这些英文文档，不仅能够学到知识，还能锻炼英文水平，这是再好不过了事了，不过你需要有毅力，不能怕困难。

最后，谢谢你的阅读，我们下次再见 :)