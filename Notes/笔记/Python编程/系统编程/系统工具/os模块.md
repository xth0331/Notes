# 介绍OS模块

`os`模块是两大核心系统模块中较大的那个。它包含了在C程序和shell脚本中经常用到的所有操作系统调用。它的调用设计目录、进程和shell变量等。准确地说，该模块提供了*POSIX*工具，操作系统调用的跨平台移植标准，以及不依赖平台的目录处理工具，如内嵌模块`os.path`便携的脚本通常无需改动即可在其他平台上运行。某些平台，os还包括了专属该平台的额外工具。不过总的来说，只要技术上可行，os都能做到跨平台。



## os模块中的工具

os模块中常用的一些工具：

| 任务               | 工具                                                         |
| ------------------ | ------------------------------------------------------------ |
| Shell变量          | `os.enviros`                                                 |
| 运行程序           | `os.system`,  `os.popen`,  `os.execv`,  `os.swawnv`          |
| 派生               | `os.fork`,  `os.pipe`,  `os.waitpid`, `os.kill`              |
| 文件描述符，文件锁 | `os.open`, `os.read` , `os.write`                            |
| 文件处理           | `os.remove`, `os.rename`, `os.mkfifo`, `os.mkdir`, `os.rmdir` |
| 管理工具           | `os.getcwd`, `os.chdir`, `os.chmod`, `os.getpid`, `os.listdir`, `os.access` |
| 移植工具           | `os.sep` , `os.pathsep`, `os.curdir`, `os.path.split`, `os.path.join` |
| 路径名工具         | `os.path.exists('path')`, `os.path.isdir('path')`, `os.path.getsize('path')` |

---------------------------

如果交互地查看这个模块的属性， 你会得到一大串名称， 他们会因Python的版本和运行平台而异，所以你必须了解每一个名称的意义才能很好地利用他们：

```python
import os
dir(os)
['CLD_CONTINUED', 'CLD_DUMPED', 'CLD_EXITED', 'CLD_TRAPPED', 'DirEntry', 'EX_CANTCREAT', 'EX_CONFIG', 'EX_DATAERR', 'EX_IOERR', 'EX_NOHOST', 'EX_NOINPUT', 'EX_NOPERM', 'EX_NOUSER', 'EX_OK', 'EX_OSERR', 'EX_OSFILE', 'EX_PROTOCOL', 'EX_SOFTWARE', 'EX_TEMPFAIL', 'EX_UNAVAILABLE', 'EX_USAGE', 'F_LOCK', 'F_OK', 'F_TEST', 'F_TLOCK', 'F_ULOCK', 'GRND_NONBLOCK', 'GRND_RANDOM', 'MutableMapping', 'NGROUPS_MAX', 'O_ACCMODE', 'O_APPEND', 'O_ASYNC', 'O_CLOEXEC', 'O_CREAT', 'O_DIRECT', 'O_DIRECTORY', 'O_DSYNC', 'O_EXCL', 'O_LARGEFILE', 'O_NDELAY', 'O_NOATIME', 'O_NOCTTY', 'O_NOFOLLOW', 'O_NONBLOCK', 'O_PATH', 'O_RDONLY', 'O_RDWR', 'O_RSYNC', 'O_SYNC', 'O_TMPFILE', 'O_TRUNC', 'O_WRONLY', 'POSIX_FADV_DONTNEED', 'POSIX_FADV_NOREUSE', 'POSIX_FADV_NORMAL', 'POSIX_FADV_RANDOM', 'POSIX_FADV_SEQUENTIAL', 'POSIX_FADV_WILLNEED', 'PRIO_PGRP', 'PRIO_PROCESS', 'PRIO_USER', 'P_ALL', 'P_NOWAIT', 'P_NOWAITO', 'P_PGID', 'P_PID', 'P_WAIT', 'PathLike', 'RTLD_DEEPBIND', 'RTLD_GLOBAL', 'RTLD_LAZY', 'RTLD_LOCAL', 'RTLD_NODELETE', 'RTLD_NOLOAD', 'RTLD_NOW', 'R_OK', 'SCHED_BATCH', 'SCHED_FIFO', 'SCHED_IDLE', 'SCHED_OTHER', 'SCHED_RESET_ON_FORK', 'SCHED_RR', 'SEEK_CUR', 'SEEK_DATA', 'SEEK_END', 'SEEK_HOLE', 'SEEK_SET', 'ST_APPEND', 'ST_MANDLOCK', 'ST_NOATIME', 'ST_NODEV', 'ST_NODIRATIME', 'ST_NOEXEC', 'ST_NOSUID', 'ST_RDONLY', 'ST_RELATIME', 'ST_SYNCHRONOUS', 'ST_WRITE', 'TMP_MAX', 'WCONTINUED', 'WCOREDUMP', 'WEXITED', 'WEXITSTATUS', 'WIFCONTINUED', 'WIFEXITED', 'WIFSIGNALED', 'WIFSTOPPED', 'WNOHANG', 'WNOWAIT', 'WSTOPPED', 'WSTOPSIG', 'WTERMSIG', 'WUNTRACED', 'W_OK', 'XATTR_CREATE', 'XATTR_REPLACE', 'XATTR_SIZE_MAX', 'X_OK', '_Environ', '__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '_execvpe', '_exists', '_exit', '_fspath', '_fwalk', '_get_exports_list', '_putenv', '_spawnvef', '_unsetenv', '_wrap_close', 'abc', 'abort', 'access', 'altsep', 'chdir', 'chmod', 'chown', 'chroot', 'close', 'closerange', 'confstr', 'confstr_names', 'cpu_count', 'ctermid', 'curdir', 'defpath', 'device_encoding', 'devnull', 'dup', 'dup2', 'environ', 'environb', 'errno', 'error', 'execl', 'execle', 'execlp', 'execlpe', 'execv', 'execve', 'execvp', 'execvpe', 'extsep', 'fchdir', 'fchmod', 'fchown', 'fdatasync', 'fdopen', 'fork', 'forkpty', 'fpathconf', 'fsdecode', 'fsencode', 'fspath', 'fstat', 'fstatvfs', 'fsync', 'ftruncate', 'fwalk', 'get_blocking', 'get_exec_path', 'get_inheritable', 'get_terminal_size', 'getcwd', 'getcwdb', 'getegid', 'getenv', 'getenvb', 'geteuid', 'getgid', 'getgrouplist', 'getgroups', 'getloadavg', 'getlogin', 'getpgid', 'getpgrp', 'getpid', 'getppid', 'getpriority', 'getrandom', 'getresgid', 'getresuid', 'getsid', 'getuid', 'getxattr', 'initgroups', 'isatty', 'kill', 'killpg', 'lchown', 'linesep', 'link', 'listdir', 'listxattr', 'lockf', 'lseek', 'lstat', 'major', 'makedev', 'makedirs', 'minor', 'mkdir', 'mkfifo', 'mknod', 'name', 'nice', 'open', 'openpty', 'pardir', 'path', 'pathconf', 'pathconf_names', 'pathsep', 'pipe', 'pipe2', 'popen', 'posix_fadvise', 'posix_fallocate', 'pread', 'putenv', 'pwrite', 'read', 'readlink', 'readv', 'remove', 'removedirs', 'removexattr', 'rename', 'renames', 'replace', 'rmdir', 'scandir', 'sched_get_priority_max', 'sched_get_priority_min', 'sched_getaffinity', 'sched_getparam', 'sched_getscheduler', 'sched_param', 'sched_rr_get_interval', 'sched_setaffinity', 'sched_setparam', 'sched_setscheduler', 'sched_yield', 'sendfile', 'sep', 'set_blocking', 'set_inheritable', 'setegid', 'seteuid', 'setgid', 'setgroups', 'setpgid', 'setpgrp', 'setpriority', 'setregid', 'setresgid', 'setresuid', 'setreuid', 'setsid', 'setuid', 'setxattr', 'spawnl', 'spawnle', 'spawnlp', 'spawnlpe', 'spawnv', 'spawnve', 'spawnvp', 'spawnvpe', 'st', 'stat', 'stat_float_times', 'stat_result', 'statvfs', 'statvfs_result', 'strerror', 'supports_bytes_environ', 'supports_dir_fd', 'supports_effective_ids', 'supports_fd', 'supports_follow_symlinks', 'symlink', 'sync', 'sys', 'sysconf', 'sysconf_names', 'system', 'tcgetpgrp', 'tcsetpgrp', 'terminal_size', 'times', 'times_result', 'truncate', 'ttyname', 'umask', 'uname', 'uname_result', 'unlink', 'unsetenv', 'urandom', 'utime', 'wait', 'wait3', 'wait4', 'waitid', 'waitid_result', 'waitpid', 'walk', 'write', 'writev']

```

除了以上这些，内嵌的`os.path`模块还能导出更多的工具，他们大多都涉及可移植地处理文档和目录名称：

```python
import os
dir(os.path)
['__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '_get_sep', '_joinrealpath', '_varprog', '_varprogb', 'abspath', 'altsep', 'basename', 'commonpath', 'commonprefix', 'curdir', 'defpath', 'devnull', 'dirname', 'exists', 'expanduser', 'expandvars', 'extsep', 'genericpath', 'getatime', 'getctime', 'getmtime', 'getsize', 'isabs', 'isdir', 'isfile', 'islink', 'ismount', 'join', 'lexists', 'normcase', 'normpath', 'os', 'pardir', 'pathsep', 'realpath', 'relpath', 'samefile', 'sameopenfile', 'samestat', 'sep', 'split', 'splitdrive', 'splitext', 'stat', 'supports_unicode_filenames', 'sys']

```

## 管理工具

和`sys`模块一样，os模块也代用一套提供信息、帮助管理的工具：

```python
import os
os.getpid
14539

os.chdir(r'C:\User')
os.getcwd()
'C:\User'
```



`os.getpid()`给出调用函数的进程的ID（这是系统为当前运行程序定义的唯一标识符，可用于进程控制和唯一命名），`os.fercwd`则返回当前的工作目录。当前的工作目录是你的脚本所打开的文档应当放置的位置，除非文件名里显式的包含了目录路径。

## 可移植的常量

`os`模块同事导出了一组用于简化跨平台编程的名称，包括与具体平台相关的路径和目录分隔符、父目录和当前目录指示器，以及底层计算机所采用的换行符。

```python
import os
os.pathsep, os.sep, os,pardir, os.curdir, os.linesep
(':', '/', '..', '.', '\n')				# linux
(';', '\\', '..', '.', '\r\n')				# win
```



<<<<<<< HEAD:Notes/笔记/Python编程/系统编程/系统工具/os模块.md
`os.sep`是Python底层运行平台所采用的目录组分隔符号。他在windows下自动预设为“\” ，POSIX则是“/” ,某些Mac则是“:”。类似的，os.pathsep提供用于在目录列表中分隔目录的字符，POSIX使用“:”，DOS和Windows使用“；”。



`os.sep`是Python底层运行平台所采用的目录组分隔符号。他在windows下自动预设为

"\\",POSIX计算机则是"/", 某些Mac上则使用”:“。类似地，`os.pathsep`提供用于在目录列表中分隔目录的字符，POSIX使用":",DOS和windows使用”;“。

当我们在脚本中拼装和分解这些系统相关字符串时，借助这些属性，可以充分实现脚本的可移植性。例如，虽然`dirpath`在windows中是*dir\dir*，在Linux中是*dir/dir* ,但`dir path.split(os.sep)`的调用可以准确无误地将与平台相关的目录名称分解为各个部分。在Windows中你通常可以在列出待打开的文件名时用斜杠代替反斜杠，但这些可移植的常量允许脚本在目录处理部分的代码不依赖平台。

另外请注意，`os.linesep`在这里使用的是\r\n,这个符号转义码在Windows下代表回车加换行惯例，在使用Python处理文本时一般不会注意这一点。



## 常见os.path工具

内嵌的`os.path`模块提供了一整套目录处理相关工具。举例来说，它提供的可移植函数可以用来检查文件类型（`isdir`,`isfile`等）、测试文件是否存在（`exists`)，以及通过文件名来获取文件的大小（`getsize`）:

```python
import os
os.path.isdir(r'C:\Users'), os.path.isfile(r'C:\Users')
(True, False)


os.path.isdir(r'C:\config.sys'), os.path.isfile(r'C:\config.sys')
(False, True) 

os.path.isdir('nonesuch'), os.path.isfile('nonesuch')
(False, False)

os.path.exists(r'c\Users\Xie')
False

os.path.exists(r'c:\Users\Default')
True

os.path.getsize(r'C:\autoexec.bat')
24
```



`os.path.isdir`和`os.path.isfile`调用可以告诉我们文件名是目录还是一个简单的文件。如果文件不存在，二者都会返回*False*(也就是说不存在即暗示否定的意思)。我们还能得到用于分割和合并目录路径字符串的函数，它们会自动应用Python所在底层平台的目录命名惯例：

```python
import os
os.path.split(r'C:\temp\data.txt')
('C:\\temp', 'data.txt')

os.path.join(r'C:\temp','output.txt')
('C:\\temp\\output.txt')

name = r'C:\temp\data.txt'		# windows路径风格
os.path.dirname(name), os.path.basename(name)
('C:\temp', 'data.txt')

name = '/home/xth/temp/data.txt'	# Unix风格路径
os.path.dirname(name), os.path.basename(name)
('/home/xth/temp', 'data.txt')

os.path.splitext(r'C:\example\python\demo.py')
('C:\\example\\python\\demo', '.py')


```

`os.path.split`将文件名从它的目录路径中剥离出来，`os.path.join`则将它们合并起来，这两个调用都是可移植的，都自动采用所在平台的路径命名惯例。为了简便起见`dirname`和`basename`则调用返回了`split`返回结果的前两项，`splitext`则剥离了文件的扩展名。微妙之处在于，使用字符串的`split`和`join`方法几乎可以起到`os.sep`相同的作用，只是有细微区别：

```python
import os 
os.sep
'\\'

pathname = r'C:\example\python\demo.py'
os.path.split(pathname)		# 分割文件和目录
('C:\\example\\python\\', 'demo.py')
pathname.split(os.sep)		# 在每个斜杠处做分割
('C', 'example', 'python', 'demo.py')

os.sep.join(pathname.split(os.sep))
'C:\\example\\python\\demo.py'

os.path.join(*pathname.split(os.sep))
'C:example\\python\\demo.py'
```



最后一个`join`调用要求传入参数（因此用了*），但因为Windows的驱动器句法的关系不插入第一个斜杠；如果这种区别影响到程序，可以转而使用前面的`str.join`方法。如果你的路径混用了
Unix和Windows分隔符，这时`normpath`调用显得格外好用：

```python
import os
mixed = 'C:\\temp\\public/files/index.html'
os.path.normpath(mixed)
'C:\\temp\\public\\files\\index.html'
print(os.path.normpath(r'C\trmp\\sub\.\file,ext'))
C:\temp\sub\file.ext
```

这个模块还有`abspath`调用，他能可移植地返回文件的完整目录路径名。它负责将当前目录添加为前缀以及处理..父目录句法等：

```python
import os 
os.getcwd()
'C:\\Users'

os.path.abspath('')			# 空字符串代表当前工作目录
'C:\\Users'

os.path.abspath('temp')		# 扩展为当前工作目录下的完整路径名
'C:\\Users\\temp'

os.path.abspath(r'demo\dev')
'C:\\Users\\demo\\dev'

```

