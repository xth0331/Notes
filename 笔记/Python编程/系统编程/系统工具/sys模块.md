# sys模块介绍

## 平台和版本

和大多数模块一样，`sys`既包括富含信息的名称，也包括完成具体工作的函数。例如，他的属性可以告诉我们底层操作系统名称，当前计算机可容纳的最大的**“原生”**整型（*虽然python3.X版本中整型的长度可以是任意长的*），异机正在运行我们代码的Python解释器的版本号：

```python
import sys
sys.platform,sys.maxsize,sys.version
('win32', 9223372036854775807, '3.6.4 (v3.6.4:d48eceb, Dec 19 2017, 06:54:40) [MSC v.1900 64 bit (AMD64)]')
if sys.platform[:3] == 'win':
    print('Hello Windows')
...
Hello Windows
```

如果你的代码确定要在不同的机器上表现出不同的行为，只需向刚才那样测试`sys.platform`字符串；虽然Python的大部分功能是跨平台的， 但也可以把不可抑制的工具封装在`if`语句测试中，就像上面一样，比方说没等下我们会看到一些程序的运行和底层控制台交互工具可能因平台而异。只需测试`sys.platform`，就可以胃运行你的脚本的计算机挑选合适的工具。

## 模块搜索路径

`sys`模块使得我们可以在Python程序内部或者交互地查看模块搜索路径。`sys.path`是一个由目录名称字符串组成的列表，每个目录名称字符串代表正在运行Python解释器的真正的搜索路径。模块导入时，Python会自左向右扫描列表，在列表中的每个目录下搜索模块文件。因此，在这里你可以验证设定的搜索路径是否正确。

`sys.path`列表在解释器启动时根据`PYTHONPATH`设置进行初始化，你电脑中的Python目录下的所有`.pyh`路径文件的内容，以及系统默认设置。其实如果你交互地查看`sys.path`，就会注意到相当多的目录并不在你的`PYTHONPATH`中：`sys.path`还包含了一个代表脚本主目录的指示器和一组标准库目录，因安装而异。

```python
import sys
sys.path
```

`sys.path`可以用程序进行更改。脚本可以借助多种列表操作来设置搜索路径，比如`append`、`extend`、`insert`、`pop`、`remove`和`del`，以便把所需的源目录全部包括进来。无罗你如何更改，Python在导入时总是使用当前`sys.path`设置：

```python
import sys
sys.path.append(r'C:\mydir')
```

设置shell变量PYTHONPATH的另一个办法时向上面这样直接更改`sys.path`，不过这不是永久性的。对`sys.path`的更改只维持到Python结束时，而且每次启动新的Python程序或会话时，都必须重新设置。

## 已加载模块表

`sys`模块还包含嵌入解释器的钩子。例如，`sys.modules`是个字典，你的Python会话或程序（准确说是进程）所导入的每个模块在其中都有一个`name:module`项:

```python
import sys
sys.modules
list(sys.modules.key())
sys.modules['sys']
```

我们可以使用这个钩子来编写程序，让程序显示或者处理某个程序加载的所有模块（对`sys.modules`的键列表进行迭代即可）。

另外，借助解释器的钩子，可以通过`sys.getrefcount`来查看对象的引用次数，而Python可执行程序的内置模块名称则在`sys.builtin_module_names`中列出。

## 异常的详细信息

通过sys模块中的另一些属性为我们提供最近抛出的Python异常的所有相关信息。如果想以更一般的方式来处理异常，这一点就变得非常方便。例如，`sys.exc_info`函数会返回一个元组，其中含有最近异常的类型、值和追踪对象，在Python使用的所有基于类的异常模型中，前两个异常模型对应着最近的异常所属的类及其实例：

```python
try:
    raise IndexError
execpt:
    print(sys.exc_info())
```



可以利用这些信息来格式化显示我们自己的错误信息，将其显示在GUI弹出窗口或HTML网页中（默认设置下，未能捕获的异常将终止程序并显示一条Python错误消息）。这个调用返回的前两项直接打印时显示具有一定格式的字符串，而第三项是追踪对象，可以用标准模块`traceback`处理：

```python
import traceback, sys
def grail(x):
    raise TypeError('already got one')
try:
    grail('arthur')
except:
    exc_info = sys.exc_info()
    print(exc_info[0])
    print(exc_info[1])
    traceback.print_tb(exc_info[2])
```



> traceback模块还可以将消息格式化为字符串并将其到想到特定文档对象

## sys模块到处的其他工具

sys模块还导出其他一些常用工具，例如：

- 显示为由字符串组成的列表的命令行参数，称为`sys.argv`。
- 标准流，包括`sys.stdin`、`sys.stdout`和`sys.stderr`。
- 程序可以通过调用`sys.exit`强制退出。