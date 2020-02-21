# Go基础语法

## 关键字

```go
if      for     func    case        struct      import               
go      type    chan    defer       default     package
map     const   else    break       select      interface
var     goto    range   return      switch      continue     fallthrough       
```

## 保留字

```go
内建常量：  
        true        false       iota        nil
内建类型：  
        int         int8        int16       int32       int64
        uint        uint8       uint16      uint32      uint64      uintptr
        float32     float64 
        complex128  complex64
bool：      
        byte        rune        string 	    error
内建函数：   
        make        delete      complex     panic       append      copy    
        close       len         cap	        real        imag        new   	    recover
```

## 变量

### 单变量声明

```go
var a int 						// 声明一个变量，默认值。
var b = 1					      // 声明并初始化，推导类型。
c := 2						      // 初始化，并自动推导。
```

- `:=`定义变量只能在函数内部使用，var定义全局变量。
- 声明未使用的变量在编译阶段会报错：`not used`。
- 标识符以字母或下划线开头，大小写敏感。
- 推荐驼峰命名

### 多变量声明

```go
var a, b string
var a1, b2 string = "是", "去"
var c3, d4 int = 3, 4
c, d := 1, 2
var(
	e int
	f bool
)
```

### 变量值互换

```go
m, n = n, m			// 变量值互换
temp, _ = m, n		// 匿名变量：变量值互换，且丢弃变量n
```

### 丢弃变量

`_`是一个特殊的变量名，任何赋予它的值都会被丢弃。该变量不占用命名空间，也不会分配内存。

```go
_, b := 55, 88		// 值35赋予变量b， 同时丢弃34
```



### `:=`声明的注意事项

正确示例：

```go
in, err := os.Open(file)
out, err := os.Create(file)		// err已经在上方定义，此处为err的赋值操作
```

如果这样，则编译不通过：

```go
in, err := os.Open(file)
in, err := os.Create(file)     // 即:=必须确保至少有一个变量是用于声明
```

`:=`仅对已经在同级词法域声明过的变量才和赋值操作语句等价，如果变量是在外部词法域声明，那么`:=`将会在当前词法域重新声明新的变量。

### 多数据分组书写

`Go`可以使用该方法声明多个数据：

```go
const(
	i = 100
	pi = 3.14
    prefix = "Go_"
)
var(
	i int
	pi float32
	prefix string
)
```

## 关键字`iota`

`iota`声明初始值为0，每行递增1：

```go
const (
    a = iota    	        // 0
    b =	iota 		        // 1        
    c = iota 		        // 2
)

const (
    d = iota    	 // 0
    e 				// 1        
    f 				// 2
)

//如果iota在同一行，则值都一样
const (
    g = iota    	            // 0
    h,i,j = iota,iota,iota      // 1,1,1
    // k = 3                    // 此处不能定义缺省常量，会编译错误	
)
```

