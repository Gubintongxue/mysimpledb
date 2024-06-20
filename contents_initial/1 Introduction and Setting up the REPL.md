## **第1部分 - 介绍和设置REPL**

作为一个Web开发人员，我每天在工作中使用关系型数据库，但它们对我来说是一个黑匣子。我有一些问题：

- 数据以什么格式保存？（在内存中和在磁盘上）
- 什么时候数据从内存移动到磁盘？
- 为什么每个表只能有一个主键？
- 如何回滚事务？
- 索引是如何格式化的？
- 何时以及如何进行全表扫描？
- 预处理语句以什么格式保存？

换句话说，数据库是如何工作的？

为了弄清楚这些问题，我将从头开始编写一个数据库。它的模型是SQLite，因为它设计得很小，比MySQL或PostgreSQL功能少，所以我可以更好地理解它。整个数据库存储在一个文件中！

**SQLite**

在他们的网站上有大量的SQLite内部文档，加上我有一本《SQLite数据库系统：设计和实现》的副本。

**下图为sqlite结构**

![img](image/arch1.gif)

Tokenizer（词法分析器）→Parser（解析器）→Code Generator（代码生成器）→Virtual Machine（虚拟机）

→B-Tree（B树）→Pager（页管理器）→OS Interface（操作系统接口）



一个查询通过一系列组件来检索或修改数据。前端由以下部分组成：

- 词法分析器
- 解析器
- 代码生成器

前端的输入是一个SQL查询。输出是sqlite虚拟机字节码（本质上是可以在数据库上操作的已编译程序）。

后端由以下部分组成：

- 虚拟机
- B树
- 页管理器
- 操作系统接口

虚拟机将前端生成的字节码作为指令。然后它可以对一个或多个表或索引执行操作，每个表或索引存储在一种称为B树的数据结构中。虚拟机本质上是一个基于字节码指令类型的开关语句。

每个B树由许多节点组成。每个节点的长度为一页。B树可以通过向页管理器发出命令从磁盘检索页面或将其保存回磁盘。

页管理器接收命令以读写数据页面。它负责在数据库文件中适当的偏移量处读取/写入。它还在内存中保留最近访问的页面缓存，并确定何时需要将这些页面写回磁盘。

操作系统接口是根据编译sqlite时针对的操作系统不同而变化的层。在本教程中，我不会支持多个平台。

千里之行，始于足下，所以让我们从更直接的内容开始：REPL。



## **制作一个简单的REPL**

当你从命令行启动sqlite时，它会启动一个读取-执行-打印循环（REPL）：

```shell
~ sqlite3
SQLite version 3.16.0 2016-11-04 19:09:39
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite> create table users (id int, username varchar(255));
sqlite> .tables
users
sqlite> .exit
~
```

为了实现这一点，我们的主函数将有一个无限循环，打印提示符，获取一行输入，然后处理这行输入：

```C++
int main(int argc, char* argv[]) {
    // 创建新的输入缓冲区
    InputBuffer* input_buffer = new_input_buffer();
    
    // 无限循环，处理用户输入
    while (true) {
        // 打印提示符
        print_prompt();
        // 读取输入
        read_input(input_buffer);

        // 如果输入是".exit"，则退出程序
        if (strcmp(input_buffer->buffer, ".exit") == 0) {
            close_input_buffer(input_buffer);
            exit(EXIT_SUCCESS);
        } else {
            // 处理未识别的命令
            printf("Unrecognized command '%s'.\n", input_buffer->buffer);
        }
    }
}

```

我们将定义`InputBuffer`作为一个小的封装，用于存储与`getline()`交互所需的状态。（稍后会详细解释）

```C++
typedef struct {
    // 缓冲区指针
    char* buffer;
    // 缓冲区长度
    size_t buffer_length;
    // 输入长度
    ssize_t input_length;
} InputBuffer;

// 创建新的输入缓冲区
InputBuffer* new_input_buffer() {
    InputBuffer* input_buffer = (InputBuffer*)malloc(sizeof(InputBuffer));
    input_buffer->buffer = NULL;
    input_buffer->buffer_length = 0;
    input_buffer->input_length = 0;

    return input_buffer;
}

```

