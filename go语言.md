## 包
package main

## 导入
import (
	"fmt"
	"math/rand"
)

fmt 打印。等于c的printf scanf

## exported-names
导出名
一般大写字母 例如 *math.Pi*

## 函数
```go

package main

import "fmt"

func add(x int, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}
```
**变量名 之后才是类型**
原因：在复杂调用时 清晰的知道时哪个变量，参见续

## 函数（续）
当连续两个或多个函数的已命名形参类型相同时，除最后一个类型以外，其它都可以省略。
```go
package main

import "fmt"

func add(x, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}

```

## 指针
指针是证明规则的例外。请注意，例如，在数组和切片中，Go 的类型语法将括号放在类型的左侧，但表达式语法将它们放在表达式的右侧：

var a []int 
x = a[1]
为熟悉起见，Go 的指针使用 C 中的 * 符号，但我们无法对指针类型进行类似的反转。因此指针是这样工作的

var p *int 
x = *p

## 多值返回
函数可以返回任意数量的返回值。

swap 函数返回了两个字符串。
```go

package main

import "fmt"

func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```
## 命名返回值
Go 的返回值可被命名，它们会被视作定义在函数顶部的变量。

返回值的名称应当具有一定的意义，它可以作为文档使用。

没有参数的 return 语句返回已命名的返回值。也就是 直接 返回。

直接返回语句应当仅用在下面这样的短函数中。在长的函数中它们会影响代码的可读性。


```go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(9))
}

返回值 4 5
```

## 变量
变量
var 语句用于声明一个变量列表，跟函数的参数列表一样，类型在最后。

就像在这个例子中看到的一样，var 语句可以出现在包或函数级别。
```go
package main

import "fmt"

var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
0 false false false


```

## 变量初始化 

变量的初始化
变量声明可以包含初始值，每个变量对应一个。

如果初始化值已存在，则可以省略类型；变量会从初始值中获得类型。
```go
package main

import "fmt"

var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}

1 2 true false no!

```

## 短变量声明
在函数中，简洁赋值语句 := 可在类型明确的地方代替 var 声明。

函数外的每个语句都必须以关键字开始（var, func 等等），因此 := 结构不能在函数外使用。

```go
package main

import "fmt"

func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"

	fmt.Println(i, j, k, c, python, java)
}

1 2 3 true false no!

```

## 基本类型
Go 的基本类型有

bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // uint8 的别名

rune // int32 的别名
    // 表示一个 Unicode 码点

float32 float64

complex64 complex128

**其中 u代表 unsigned**

```go

```

## 类型转换
表达式 T(v) 将值 v 转换为类型 T。

一些关于数值的转换：

var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
或者，更加简单的形式：

i := 42
f := float64(i)
u := uint(f)
与 C 不同的是，Go 在不同类型的项之间赋值时需要显式转换。

```go

```

## 类型推导
类似python
会根据精度自动选择
仅限于函数内


```go

```

## 常量
常量的声明与变量类似，只不过是使用 const 关键字。

常量可以是字符、字符串、布尔值或数值。

常量不能用 := 语法声明。

```go
const (
	// 将 1 左移 100 位来创建一个非常大的数字
	// 即这个数的二进制是 1 后面跟着 100 个 0
	Big = 1 << 100
	// 再往右移 99 位，即 Small = 1 << 1，或者说 Small = 2
	Small = Big >> 99
)

```

##

```go

```

##

```go

```

##

```go

```





