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

## 数据类型



### 数据类型分类

值类型：基本数据类型是Go语言实际的原子，复合数据类型是由不同的方式组合基本类型构造出来的数据类型，如：数组，slice，map，结构体

```go
整型    int8,uint               // 基础类型之数字类型
浮点型  float32，float64        // 基础类型之数字类型
复数                           // 基础类型之数字类型
布尔型  bool                   // 基础类型，只能存true/false，占据1个字节，不能转换为整型，0和1也不能转换为布尔
字符串  string                 // 基础类型
数组                           // 复合类型 
结构体  struct                 // 复合类型
```

引用类型：即保存的是对程序中一个变量或状态的间接引用，对其修改将影响所有引用的拷贝

```go
指针    *
切片    slice
字典    map
函数    func
管道    chan
接口    interface
```

Go没有字符型，可以使用`byte`来保存单个单词。

### 零值机制

Go变量初始化会自带默认值，不像其他语言为空，列出各数据类型的的0值：

```go
int     0
int8    0
int32   0
int64   0
uint    0x0
rune    0           //rune的实际类型是 int32
byte    0x0         // byte的实际类型是 uint8
float32 0           //长度为 4 byte
float64 0           //长度为 8 byte
bool    false
string  ""
```

### 格式化输出

常用格式化输出：

```go
%%			%字面量
%b			二进制整数值，基数为2，或者是一个科学记数法表示的指数为2的浮点数
%c			该值对应的unicode字符
%d			十进制数值，基数为10
%e			科学记数法e表示的浮点或者复数
%E			科学记数法E表示的浮点或者附属
%f			标准计数法表示的浮点或者附属
%o			8进制度
%p			十六进制表示的一个地址值
%s			输出字符串或字节数组
%T			输出值的类型，注意int32和int是两种不同的类型，编译器不会自动转换，需要类型转换。
%v			值的默认格式表示
%+v			类似%v，但输出结构体时会添加字段名
%#v			值的Go语法表示
%t			单词true或false
%q			该值对应的单引号括起来的go语法字符字面值，必要时会采用安全的转义表示
%x			表示为十六进制，使用a-f
%X			表示为十六进制，使用A-F
%U			表示为Unicode格式：U+1234，等价于"U+%04X"  
```

示例：

```go
package main

import (
	"fmt"
)

func main() {
	type User struct {
		Name string
		Age  int
	}
	user := User{
		"test",
		1024,
	}
	fmt.Printf("%%\n")                   // %
	fmt.Printf("%b\n", 16)               // 10000
	fmt.Printf("%c\n", 65)               // A
	fmt.Printf("%c\n", 0x4f60)           // 你
	fmt.Printf("%U\n", '你')              // U+4f60
	fmt.Printf("%x\n", '你')              // 4f60
	fmt.Printf("%X\n", '你')              // 4F60
	fmt.Printf("%d\n", 'A')              // 65
	fmt.Printf("%t\n", 1 > 2)            // false
	fmt.Printf("%e\n", 4396.7777777)     // 4.396778e+03 默认精度6位
	fmt.Printf("%20.3e\n", 4396.7777777) //            4.397e+03 设置宽度20,精度3,宽度一般用于对齐
	fmt.Printf("%E\n", 4396.7777777)     // 4.396778E+03
	fmt.Printf("%f\n", 4396.7777777)     // 4396.777778
	fmt.Printf("%o\n", 16)               // 20
	fmt.Printf("%p\n", []int{1})         // 0xc000016110
	fmt.Printf("Hello %s\n", "World")    // Hello World
	fmt.Printf("Hello %q\n", "World")    // Hello "World"
	fmt.Printf("%T\n", 3.0)              // float64
	fmt.Printf("%v\n", user)             // {test 1024}
	fmt.Printf("%+v\n", user)            // {Name:overnote Age:1}
	fmt.Printf("%#v\n", user)            // main.User{Name:"test", Age:1024}
}
```



## 流程控制

### 条件语句

#### 判断语句`if`

`if`判断：

```go
// 初始化与判断写在一起： if a := 10; a == 10
if i == '3' {
}
```

`if`的特殊写法：

```go
if err := Connect(); err != nil {    // err != nil 才是判断表达式
}
```

#### 分支语句`switch`

示例：

```go
switch num {
    case 1:						// case 中可以是表达式
   		fmt.Println("111")
    case 2:
    	fmt.Println("2222")
    default:
    	fmt.Println("000")
}
```

贴士：

- Go保留了`break`，用来跳出`switch`语句。
- Go也提供`fallthrough`，代表不跳出`switch`，后面的语句无条件执行。

### 循环语句

#### `for`循环

Go仅支持`for`一种循环，但可以应对多种场景:

```go
// 传统的for循环
for init; condition; post{
}

// for 循环简化
var i int 
for ; ; i++ {
    if(i > 10){
        break;
    }
}

// 类似while循环
for codition {
    
}

// 死循环
for{
    
}

// for range: 一般用于遍历数组、切片、字符串、map、管道
for k, v := range []int{1,2,3} {
    
}
```

#### 跳出循环

常用的跳出循环关键字：

- `break`用于函数内跳转跳转当前`for`、`seitch`、`select`语句的执行。
- `continue`用于跳出`for`循环的本次迭代。
- `goto`可以退出多层循环

`break`跳出循环案例(`continue`同下):

```go
OuterLoopP:
for i := 0; i < 2; i++ {
    for j := 0; j < 5; j++{
        switch j{
        	case 2:
            	fmt.Println(i,j)
            	break OuterLoop
            case 3:
                 fmt.Println(i,j)
            	 break OuterLoop
        }
    }
}
```

`goto`跳出多重循环案例：

```go
for x := 0; x < 10; x++ {
    for y := 0; y < 10; x++ {
        if y == 2 {
            goto breakHere
        }
    }
}
breakHere:
	fmt.Println("break")
```

> `goto`也可以用统一错误处理。

```go
if err != nil {
    goto onExit
}
onExit:
	fmt.Println(err)
	exitProcess()
```

## 运算符

**运算符汇总**

```go
算术运算符：	+	-	*	/	%	++	--	
关系运算符：	==	!=	<=	>=	<	>	
逻辑运算符：	!	&&	||
位运算：		&（按位与）	|（按位或）	^（按位取反）	<<（左移）	>>（右移）
赋值运算符：	=	+=	-=	*=	/=	%=	<<=	>>=	&=	^=	|=
其他运算符：	&（取地址）	*（取指针值） <-（Go Channel相关运算符）
```

**自增、自减**

Go中只有*后--*和*后++*，且自增自减不能用于表达式中，只能独立使用：

```go
a = i++ 	// 错误语法
if i++ > 0 {} // 错误语法
i++           // 正确语法
```

**位运算**

```go
& 按位与，参与运算的两个数二进制位向与：同时为1，结果为1，否则0。
| 按位或，参与运算的两个数二进制位向或：有一个为1，结果为1，否则0。
^ 按位异或，二进位不同，结果为1，否则为0。
<< 按位左移，二进位左移若干位，高位丢弃，低位补0，左移n位其实就是乘以2的n次方。
>> 按位右移，二进位右移若干位，右移n位其实就是除以2的n次方。
```

### 优先级



| `→`      | `→`  | `→`   | `→`  | `→`   | `→`   | `→`   | `→`  | `→`  | `→`  | `→`  | `→`  |      |
| -------- | ---- | ----- | ---- | ----- | ----- | ----- | ---- | ---- | ---- | ---- | ---- | ---- |
| 赋值运算 | `|=` | `^=`  | `&=` | `<<=` | `>>=` | `%==` | `/=` | `*=` | `-=` | `+=` | `=`  | `↓`  |
| 逻辑运算 | `||` | `&&`  |      |       |       |       |      |      |      |      |      | `↓`  |
| 位运算   | `|`  | `^`   | `&`  |       |       |       |      |      |      |      |      | `↓`  |
| 关系运算 | `==` | `!=`  | `<`  | `<==` | `>`   | `>=`  |      |      |      |      |      | `↓`  |
| 位移运算 | `<<` | `>>>` |      |       |       |       |      |      |      |      |      | `↓`  |
| 算数运算 | `%`  | `/`   | `*`  | `-`   | `+`   | `++`  | `--` |      |      |      |      | `↓`  |

### 进制转换

#### 常见进制

- 二进制： 只有0和1，Go中不能直接使用二进制表示整数。
- 八进制：0-7，以数字0开头。
- 十进制：0-9。
- 十六进制：0-9，A-F，以0x开头，A-F及X不区分大小写。

#### 任意进制转化为十进制

**二进制转十进制：**

> 从最低位开始，每个为位乘以2（位数-1）次方然后求和。

*1011（二进制）转换为十进制：*
$$
1*2^0 + 1*2^1 + 0*2^2 + 1*2^3 = 11
$$

**八进制转十进制：**

> 从最低位开始， 每个位上乘以8（位数-1）次方然后求和。

*0123*（八进制）转换为十进制：
$$
3*8^0 + 2*8^1 + 1*8^2 + 0*8^3 = 83
$$

#### 十进制转其他进制

**十进制转二进制：**

> 不断除以2，至0为止，余数倒过来即可。

**十进制转八进制：**

> 不断除以8，至0为止，余数倒过来即可。

**十进制转十六进制：**

> 不断除以16，至0为止，余数倒过来即可。

#### 其他进制转换

- 二进制转换八进制：将二进制数从低位开始，每三位一组，转换成八进制数即可
- 二进制转十六进制：将二进制数从低位开始，每四位一组，转换成十六进制数即可
- 八进制转换二进制：将八进制数每1位转换成一个3位的二进制数（首位0除外）
- 十六进制转二进制：将十六进制每1位转换成对应的一个4位的二进制数即可

### 运算原理

计算机常见的术语：

- bit：比特，代表1个二进制位，一个位只能是0或者1
- Byte：字节，代表8个二进制位，计算机中存储的最小单元是字节
- WORD：双字节，即2个字节，16位
- DWORD：两个WORD，即4个字节，32位

一些常用单位：

- 1b：1bit，1位
- 1Kb：1024bit，即1024位
- 1Mb：1024*1024bit
- 1B：1Byte，1字节，8位
- 1KB：1024B
- 1MB：1024K

对于有符号数而言，二进制的最高为是符号位：0表示正数，1表示负数，比如 1在二进制中：

```
1  二进制位：0000  0001
-1 二进制位：1000  0001
```

正数的原码、反码、补码都一样，负数的反码=原码符号位不变，其他位取反，补码是反码+1

```
         1              -1
原码  0000  0001        1000  0001
反码  0000  0001        1111  1110
补码  0000  0001        1111  1111
```

常见理解：

- 0的反码补码都是0
- 计算机中是以补码形式运算的



## 数值类型

### 整数

整数类型有无符号（如int）和带符号（如uint）两种，这两种类型的长度相同，但具体长度取决于不同的编译器实现。

`int8`、`int16` 、`int32`和`int64`四种有符号整数类型，分别对应8、16、32、64bit大小的有符号整数，同样uint8，uint16、unit32和uint64对应四中无符号整数类型。

有符号类型：

```go
int     32位系统占4字节（与int32范围一样），64位系统占8个节（与int64范围一样）
int8    占据1字节   范围 -128 ~ 127
int16   占据2字节   范围 -2(15次方) ~ 2（15次方）-1
int32   占据4字节   范围 -2(31次方) ~ 2（31次方）-1
int64   占据8字节   范围 -2(63次方) ~ 2（63次方）-1
rune	int32的别称
```

无符号类型：

```go
uint	32位系统占4字节（与uint32范围一样），64位系统占8字节（与uint64范围一样）     
uint8   占据1字节   范围 0 ~ 255
uint16  占据2字节   范围 0 ~ 2（16次方）-1
uint32  占据4字节   范围 0 ~ 2（32次方）-1
uint64  占据8字节   范围 0 ~ 2（64次方）-1
byte	uint8的别称
```

注意：

- 上述类型的变量由于是不通类型，不允许相互赋值或操作。
- Go默认的整型是int。
- 查看数据所占据的字节数方法：`unsafe.Sizeof()`。



### 浮点类型

#### 浮点类型的分类

```go
float32 单精度  占据4字节   范围 -3.403E38 ~ 3.403E38    (math.MaxFloat32)
float64 双精度  占据8字节   范围 -1.798E208 ~ 1.798E308  (math.MaxFloat64)
```

可以看出：

- 浮点是有符号的，浮点数在机器中村次方形式是：浮点数=符号位+指数位+尾数位

- 浮点型的范围是固定的，不受操作系统影响。

- `.512`这样可以识别为`0.512`。

- 科学计数法：

  - $$
    5.12E2 =5.12 * 10^2
    $$

  - $$
    5.12E-2 = 5.12 /10^2
    $$

#### 精度损失

`float32`可以提供大约6个十进制数的精度，`float64`大约可以提供15个十进制数的精度（一般选`float64`）

```go
var num1 float32 = -123.0000901
var num2 float64 = -123.0000901
fmt.Println("num1=", num1)				// -123.00009
fmt.Println("num2=", num2)				// -123.0000901
```

#### 浮点数判断相等

使用 `==`判断浮点数是不可行的，以下是替代方案:

```go
func isEqual(f1,f2,p float64) bool {
    // p 为用户自定义精度，如：0.00001
    return math.Abs(f1-f2) < p
}
```

### 复数

Go中复数默认类型是`complex128`（64位实数+64位虚数）。如果需要小一些，也有`complex64`（32位实数+32位虚数）。复数形式为`RE + Imi`，其中`RE`是实数部分，`IM`是虚数部分，最后的`i`是虚数单位。

如下所示：

```go
var t complex128
t = 2.1 + 3.14i
t1 = complex(2.1,3.14) // 结果同上
fmt.Println(real(t))   // 实部：2.1
fmt.Println(imag(t))   // 虚部：3.14
```

### NaN非数

Go中的`NaN`非数：

```go
var z float64
// 输出 "0 -0 +Inf -Inf NaN"
fmt.Println(z, -z 1/z, -1/z, z/z)
```

注意：

- 函数`math.IsNaN`用于测试一个数是否是非数`NaN`。
- 函数`math.NaN`则返回非数对应的值。
- 虽然可以用`math.NaN`来表示一个非法的结果，但是测试一个结果是否为非数`NaNs`则是充满了风险的，因为`NaN`和任何数都是不相等的。

```go
nan := math.NaN()
// "false false false"
fmt.Println(nan == nan, nan < nan, nan > nan)
```

## 字符串

### 字符

Golang中没有专门的字符类型，如果要存储单个字符（字母），一般使用`byte`来存储，且使用单引号包裹。

```go
var c1 byte = 'a'
var c2 byte = '0'
fmt.Println("c1=", c1)					//输出 97   
fmt.Println("c2=", c2)					//输出48
fmt.Printf("c1=%c,c2=%c\n", c1, c2)	    //输出原值 a 0

//var c3 byte = '北'
//fmt.Printf("c3=%c", c3)					// 溢出错误:overflows byte
```

> - 字符类型可以用`%d`打印为整型
> - 如果我们保存的字符在ASCII表，比如[0-1, a-z,A-Z..]直接可以保存到`byte`
> - 如果我们保存的字符对应值大于255，这是我们可以考虑使用`int`类型
> - 如果我们需要按照字符的方式输出，这是我们需要格式化，`fmt.Printf("%c", c1)`
> - 字符可以和整型进行运算

### 字符串

传统字符串是由字符组成的，而Go的字符串是由单个字节连接起来的，即Go字符串是遗传固定长度的字符连接起来的字符序列。

字符串在Go语言中是基本类型，内容在初始化后不能改变。

Go中的字符串都是采用UTF-8字符集编码，使用一对双引号或反引号定义。反引号可以额外解析换行，即其没有字符转义功能。

```go
var str1 string
st1 = "Hello "
str2 := " World!"

fmt.Println(str1[0])		// 输出字符串第一个字符 72
fmt.Println(len(str1))		// 输出长度 6
fmt.Println(str1 + str2)	// 输出不带空格的

// 字符串不可变，编译报错： connot assign to 因为 str1[0] = 'c'
```

由于Go中的字符串不可直接改变，可以使用下列两种方式进行修改：

方式一：通过转换为字节数组`[]byte`类型，构造一个临时字符串

```go
str := "hello"

strTemp := []byte(str)
fmt.Println("strTemp=", strTemp)	// [104 101 108 108 111]

strTemp[0] = 'c'
strResult := string(strTemp)
fmt.Println("strResult=", strResult)	// strResult= cello
```

方式二： 使用切片

```go
str := "hello"	
str = "c" + str[1:]	// 1: 表示从第1位开始到最后
```

Go和Java等语言一样，字符串默认不可变，这样保证了线程安全，大家使用的都是只读对象，无需加锁，且能很方便的共享内存，不必使用写时复制。

### 字符串常用操作

#### len()函数与字符串遍历

`len()`函数是go语言的内建函数，可以用来获取字符串、切片、通道等长度。

```go
package main

import (
	"fmt"
    "unicode/utf8"
)

func main() {
    
    str1 := "Hello World!"
    str2 := "你好，"
    
    fmt.Println(len(str1))	// 11
    fmt.Println(len(str2))  // 9
    fmt.Println(utf8.RuneCountInString(str2))  // 3
}
```

第一个函数输出11很容易理解，第二个函数却输入了9，这是因为Go的字符串都是以UTF-8格式保存，每个中文占据3个字节。Go中计算UTF-8字符串格式的长度应该使用`utf9.RuneCountInString`。

字符串遍历方式一：使用字节数组，注意每个中文在UTF-8中占据3个字节。

```go
str := "hello"
for i := 0; i<len(str); i++ {
    fmt.Println(i, str[i])
}
```

字符串遍历方式二：range关键字只是第一种方式的简写。

```go
str := "hello"
for i, ch := range str {
    fmt.Println(i, ch)
}
```

> 由于上述`len()`函数本身原因，Unicode字符遍历需要使用range。

#### string()函数类型转换

go的内建函数`string()`可以将其他类型转变为字符串类型：

```go
num := 12
fmt.Printf("%T \n", string(num))
```



