# tutorial01-实现REPL

## 1\. 如何安装我们的单元测试环境？

-   检查我们是否有安装`ruby`。  
    输入命令`ruby -v`，我们将得到

```
ruby 2.7.0p0 (2019-12-25 revision 647ee6f091) [x86_64-linux-gnu] 
```

如果没有安装使用`sudo apt install ruby`或对应操作系统的包管理工具安装即可。

![image-20240619231741309](image/image-20240619231741309.png)

-   检查我们是否有安装`rspec`  
    输入命令`rspec -v`，我们将得到

```
RSpec 3.10
  - rspec-core 3.10.2
  - rspec-expectations 3.10.2
  - rspec-mocks 3.10.3
  - rspec-support 3.10.3 
```

同理如果没有安装则使用`sudo gem install rspec`安装即可。若提示没有`gem`则使用`sudo apt install gem`便可完成安装。

![image-20240619231851438](image/image-20240619231851438.png)

==发现问题==

安装rspec，终端没反应

![image-20240620184725541](image/image-20240620184725541.png)

使用系统提示方法安装（**==此部分错误，见解决gem安装慢或卡住==**）

apt install ruby-rspec-core

![image-20240620184825275](image/image-20240620184825275.png)

`ruby`和`rspec`版本基本未对本单元测试造成影响，如果存在问题，可提出。

至此我们的测试环境便已经安装完成了。

### 1.1 ruby和rspec介绍

待补充

## 2\. 如何编写和使用我们的第一个单元测试？

让我们在目录下创建`db_test.rb`文件。

```ruby
describe 'database' do
  def run_script(commands)
    raw_output = nil
    IO.popen("./db", "r+") do |pipe|
      commands.each do |command|
        pipe.puts command
      end

      pipe.close_write

      # Read entire output
      raw_output = pipe.gets(nil)
    end
    raw_output.split("\n")
  end

  it 'test exit and unrecognized command' do
    result = run_script([
      "hello world",
      "HELLO WORLD",
      ".exit",
    ])
    expect(result).to match_array([
      "db > Unrecognized command: hello world",
      "db > Unrecognized command: HELLO WORLD",
      "db > Bye!",
    ])
  end
end
```

让我们来执行它看看，输入`rspec spec db_test.rb`

```
LoadError:
  cannot load such file ... 
```

显然是会失败的，因为我们甚至还没有在目录下编译创建我们的`db`文件。 现在让我们从最简单交互窗口`REPL`开始。

### 补充 解释rb文件

代码是一个使用 RSpec 框架编写的测试脚本，用于测试一个名为 `db` 的数据库程序。以下是对这段代码的详细解释：

**整体结构**

```
describe 'database' do
  ...
end

```

- `describe 'database' do ... end` 是一个 RSpec 代码块，用于定义一个测试组（描述块），描述与数据库相关的测试。

### `run_script` 方法

```
def run_script(commands)
  raw_output = nil
  IO.popen("./db", "r+") do |pipe|
    commands.each do |command|
      pipe.puts command
    end

    pipe.close_write

    # Read entire output
    raw_output = pipe.gets(nil)
  end
  raw_output.split("\n")
end

```

- `run_script` 方法定义了一个辅助函数，用于运行一系列命令并捕获程序的输出。
- `IO.popen("./db", "r+")` 启动一个子进程，运行 `./db` 程序，允许读写操作。
- `commands.each do |command| pipe.puts command end` 将每个命令写入管道，发送到 `./db` 程序。
- `pipe.close_write` 关闭写入端，表示所有命令已经发送完毕。
- `raw_output = pipe.gets(nil)` 读取程序的所有输出，并存储在 `raw_output` 变量中。
- `raw_output.split("\n")` 将输出按行分割成数组，并返回该数组。

### `it 'test exit and unrecognized command' do ... end`

```C++
it 'test exit and unrecognized command' do
  result = run_script([
    "hello world",
    "HELLO WORLD",
    ".exit",
  ])
  expect(result).to match_array([
    "db > Unrecognized command: hello world",
    "db > Unrecognized command: HELLO WORLD",
    "db > Bye!",
  ])
end

```

- `it 'test exit and unrecognized command' do ... end` 定义了一个测试用例，描述了它要测试的行为（退出和未识别的命令）。

- `result = run_script([ ... ])` 调用 `run_script` 方法，传入一系列命令，并将结果存储在 `result` 变量中。

- ```
  expect(result).to match_array([ ... ])
  ```

   使用 RSpec 的 

  ```
  expect
  ```

   方法来定义预期结果。

  - `result` 应该匹配一个数组，数组中的每个元素对应程序的输出。
  - 数组包含以下预期输出：
    - `"db > Unrecognized command: hello world"`：未识别的命令 `hello world` 的输出。
    - `"db > Unrecognized command: HELLO WORLD"`：未识别的命令 `HELLO WORLD` 的输出。
    - `"db > Bye!"`：退出命令 `.exit` 的输出。

### 总结

这段 RSpec 代码测试了 `db` 程序的两个功能：

1. 当输入未识别的命令时，程序应输出相应的错误消息。
2. 当输入 `.exit` 命令时，程序应正确退出并输出告别消息。

通过 `run_script` 方法，测试用例能够模拟用户输入命令并捕获程序的输出，从而验证程序的行为是否符合预期。



## 3\. REPL是什么？

“读取-求值-输出”循环 _**(英語：Read-Eval-Print Loop，简称REPL)**_，也被称做交互式顶层构件 _**(英語：interactive toplevel)**_，是一个简单的，交互式的编程环境。

## 4\. 怎么实现一个简单REPL？

首先我们直接启动一个无限循环，就像一个shell一样。之后我们`print_prompt()`意味着打印提示符。接着虽然std::string被大家诟病许久，但是作为我们简易数据库的使用也暂且足矣。我们每次重复创建`input_line`这个string对象，同时通过`std::getline`这个函数从`std::cin`标准输入中获取到所需信息（以换行符做分割，即回车标志着新语句）

`.exit`意味着退出当前REPL交互，即成功退出，其他情况即输出未知命令。

```C++
void DB::print_prompt()
{
    std::cout << "db > ";
}

bool DB::parse_meta_command(std::string command)
{
    if (command == ".exit")
    {
        std::cout << "Bye!" << std::endl;
        exit(EXIT_SUCCESS);
    }
    else
    {
        std::cout << "Unrecognized command: " << command << std::endl;
        return true;
    }
    return false;
}
void DB::start()
{
    while (true)
    {
        print_prompt();
        
        std::string input_line;
        std::getline(std::cin, input_line);

        if (parse_meta_command(input_line))
        {
            continue;
        }
    }
}
```

### 测试

让我们来测试一下，首先编译我们的文件`g++ db.cpp -o db`，再输入`rspec spec db_test.rb`



```ruby
.

Finished in 0.00504 seconds (files took 0.07621 seconds to load)
1 example, 0 failures
```

结果显然，恭喜大家通过了所创建的单元测试。

#### 实际操作

![image-20240620191957403](image/image-20240620191957403.png)

![image-20240620192331190](image/image-20240620192331190.png)

报错

![image-20240620193138783](image/image-20240620193138783.png)

repec没安装好，重新安装，见 解决gem安装慢 

测试成功

![image-20240620200247176](image/image-20240620200247176.png)

## 3\. 总结

作为教程第一章，实现REPL是非常基础的。由于我们使用了`std::string`，故省去了很多`c_str`内存管理以及长度等一系列繁琐的事情。同时基于我们的测试进行开发，作为教程来说可以让大家更加清楚该朝着哪个方向前进。下一章我们将对输入的命令进行更进一步的分析。

## 4.完整代码

db.cpp

```C++
#include <iostream>
#include <string>

class DB
{
public:
    void start();
    void print_prompt();

    bool parse_meta_command(std::string command);
};

void DB::print_prompt()
{
    std::cout << "db > ";
}

bool DB::parse_meta_command(std::string command)
{
    if (command == ".exit")
    {
        std::cout << "Bye!" << std::endl;
        exit(EXIT_SUCCESS);
    }
    else
    {
        std::cout << "Unrecognized command: " << command << std::endl;
        return true;
    }
    return false;
}
void DB::start()
{
    while (true)
    {
        print_prompt();
        
        std::string input_line;
        std::getline(std::cin, input_line);

        if (parse_meta_command(input_line))
        {
            continue;
        }
    }
}

int main(int argc, char const *argv[])
{
    DB db;
    db.start();
    return 0;
}
```

补充：ubuntu下安装C++环境部分命令

```bash
sudo apt update
```

![image-20240620190303422](image/image-20240620190303422.png)

安装常用的构建工具和编译器，例如 `build-essential` 包，其中包括 `gcc`、`g++` 和其他一些开发工具：

```bash
sudo apt install build-essential
```



安装完成后，可以验证 `g++` 编译器是否安装成功：

```bash
g++ --version
g++ (Ubuntu 10.3.0-1ubuntu1~20.04) 10.3.0
```

如果你需要调试工具，可以安装 `gdb`：

```bash
sudo apt install gdb
```

安装代码编辑器（可选）

根据你的偏好，你可以安装各种代码编辑器，例如 `vim`、`emacs`、`VS Code` 等：

```bash
sudo apt install vim
sudo apt install vim
```

**Visual Studio Code**：

1. 下载并安装 Microsoft 的 GPG 密钥和存储库：

   ```bash
   sudo apt update
   sudo apt install software-properties-common apt-transport-https wget
   wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | sudo apt-key add -
   sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
   ```

2. 安装 VS Code：

   ```bash
   sudo apt update
   sudo apt install code
   ```

### 编写和运行简单的 C++ 程序

你可以使用你喜欢的编辑器创建一个简单的 C++ 程序来测试编译器。例如，创建一个名为 `hello.cpp` 的文件，并写入以下内容：

```C++
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

然后编译并运行该程序：

```
g++ hello.cpp -o hello
./hello
```

你应该会看到输出：

```
Hello, World!
```

通过这些步骤，你应该已经在 Ubuntu 下成功安装并配置了 C++ 开发环境。



如果用vscode远程调试的话，需要下载C++扩展工具和cmake等。

### gem安装速度慢问题

![image-20240620195123764](image/image-20240620195123764.png)

### 解决gem安装慢或卡住

- 使用梯子直接安装。
- 将gem的源改为国内镜像。

替换源是最简单的方法

```
# 替换源
$ gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
# 查看替换后的源地址
$ gem sources -l
https://gems.ruby-china.com
# 确保只有 gems.ruby-china.com

```

![image-20240620195954083](image/image-20240620195954083.png)



```
gem sources --remove https://rubygems.org/
gem sources -a http://ruby.taobao.org/
gem sources -l

gem sources --add http://ruby.taobao.org/ --remove https://rubygems.org/
```

gem install rspec安装好了 

![image-20240620200107191](image/image-20240620200107191.png)