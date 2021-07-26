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





