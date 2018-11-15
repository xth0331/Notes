# shell环境变量

shell变量有时称之为环境变量，Python脚本可以通过一个类似Python字典的对象`os.environ`来访问它们，其中在该对象里每项（entry）对应一个shell的变量设置。shell变量独立于Python系统，通常在你的系统启动、startup文件或控制面板中设置，他能为程序提供系统级的配置。

现在你应该很熟悉这例子： Python使用shell变量PYTHONPATH模块搜索路径来加载模块。在操作系统中配置后，每次Python程序运行都可以使用它。也可以在某些程序里设置shell变量，然后传递给其他程序。由于他们的值可以被子程序访问，因此也可以被用作简易的进程通信。

## 获取shell变量

在Python中，外部的shell环境被当作一个简单的预置对象。在`os.environ`中索引shell变量（如，`os.environ['USER']`）,类似Unix shell在变量名前添加一个`$`,(比如，`$USER`),在DOS上使用`%`（`%USER%`），在C语言里里调用`getenv("USER")`。我们启动一个交互会话来进行演示：

```python
import os 
os.environ.keys()

list(os.environ.keys())

os.environ['SHELL']						# linux下获取shell变量
os.environ['TEMP']						# Windows下获取TEMP变量
```



在Windows下，keys方法可以得到所有变量名的迭代子，通过变量名如TEMP可以搜索出它的值，Linux也一样，不同的是在Python启动时有一些其他的变量被预设。我们已经接触过`PYTHONPATH`，所以以它为例：

```python
import os 
os.environ['PYTHONPATH']

for srcdir in os,environ['PYTHONPATH'].split(os.pathsep):
    print(srcdir)
    
import sys 
sys.path[:3]
```



`PYTHONPATH`是一个包含多个路径的字符串，路径之间以路径分隔隔开。为了分割路径，我们传给字符串函数split一个分隔符`os.pathsep`，它会自动适配目标平台，`sys.path`是运行期的实际模块搜索路径，它会合并`PYTHONPATH`的值并附到当前路径后面。

## 修改shell变量

`os.environ`对象支持像普通字典一样的键索引以及赋值功能。对于字典，赋值会改变对应的键值：

```python
import os 
os.environ['TEMP'] = r'c:\temp'
os.environ['TEMP']
'c:\temp'
```

然而，还存在别的影响：在最新版本的Python中，赋值给`os.environ`的键值将自动被导出到应用的其他部分。即赋值将同时改变Python程序中的`os.environ`对象，以及该进程对应的shell环境变量。Python程序、所有链入的C模块，所有该Python进程派生的子程序都可以看到新的赋值。

在内部，对`os.environ`的键赋值将会调用`os.putenv`，它负责改变Python解释器外部的shell变量。我们用一组脚本来演示shell变量的设置和读取，

```python
import os 
print('setenv...', end=' ')	
print(os.environ['USER'])		# 输出当前shell的变量值

os.environ['USER'] = 'Brian'	# 在后台运行 os.putenv
os.system('python echoenv.py')

os.environ['USER'] = 'Arthur'	# 传递更新到衍生程序
os.system('pytho;n echoenv.py')	# 链接的C语言库模块

os.environ['USER'] = input('?')
print(os.popen('python echoenv.py').read())
```

脚本setenv.py简单地修改shell变量USER。然后派生另外一个脚本进程读取该变量值，

------------------

**echoenv.py**

```python
import os 
print('echoenv...',end=' ')
print('Hello,' os.environ['USER'])
```

总而言之，一个子进程始终从它的父进程那里集成环境设置。子程序是由如下方式启动的程序：在Unix下`os.spawnv`、`os.fork/exec`，或者所有平台下的`os.popen`、`os.system`、subprocess。他们在启动时都会获得父进程的环境变量。

从更广泛的视角来看，像这样在启动程序前设置shell变量，是一种给程序传递信息的方式，比如，一个Python配置脚本在启动另外一个脚本程序前，可以修改`PYTHONPATH`变量以包含某种特定的目录。由于shell变量会传递给子进程，因而被启动程序的`sys.path`将会包含该目录。

## shell变量要点：父进程、putenv和getenv

在Python的最顶层程序退出后，USER变量会变回初始值，赋给`os.environ`的键值被传到解释器外部，然后向下传给子进程；然而它永远不会向上传递到父进程（包括系统shell）。这个规划不是Python缺陷，在调用`putenv`库的C程序中同样如此。

如果Python脚本处在你的应用顶层，这不会有什么问题，然而要谨记，在你的程序中对shell所做的设置只对程序本身以及它所衍生的子程序有效，如果想让你的设置在Python退出后仍然生效，你可以通过平台相关的扩展来实现。

另一个微妙之处：在当前实现中，对`os.environ`的修改会自动调用`os.putenv`，后者将调用C库里的`putenv`把该设置导出到Python链接的C库里。然而，虽然对`os.environ`因此，与`os.putenv`相比，更推荐使用`os.environ`映射接口。

同时要注意，环境设置是在程序启动时一次性载入`os.environ`，而非实时读取。因此，当程序启动后，底层的C库对环境设置的改动不会反应到`os.environ`上。如今，Python集中`os.getenv`调用，然而在绝大多数平台中，它也只是简单地转换对`os.environ`的读取，而不是通过C库的`getenv`接口的调用实现。大多数应用，尤其是纯Python代码不必介意于此，在没有`putenv`底层接口的平台上，可以将`os.environ`作为参数传递给启动程序来启动子程序。