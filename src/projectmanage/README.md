# 工程管理

在实际的开发工作中，直接调用编译器进行编译和链接的场景是少而又少，因为在工程中不

会简单到只有一个源代码文件，且源文件之间会有相互的依赖关系。如果这样一个文件一个文件逐步编译，那不亚于一场灾难。 Go语言的设计者作为行业老将，自然不会忽略这一点。早期Go语言使用makefile作为临时方案，到了Go 1发布时引入了强大无比的Go命令行工具。

Go命令行工具的革命性之处在于彻底消除了工程文件的概念，完全用目录结构和包名来推导工程结构和构建顺序。针对只有一个源文件的情况讨论工程管理看起来会比较多余，因为这可以直接用go run和go build搞定。下面我们将用一个更接近现实的虚拟项目来展示Go语言的基本工程管理方法。

## 1 工作区

### 1.1 工作区介绍

Go代码必须放在工作区中。工作区其实就是一个对应于特定工程的目录，它应包含3个子目录：src目录、pkg目录和bin目录。

l src目录：用于以代码包的形式组织并保存Go源码文件。（比如：.go .c .h .s等）

l pkg目录：用于存放经由go install命令构建安装后的代码包（包含Go库源码文件）的“.a”归档文件。

l bin目录：与pkg目录类似，在通过go install命令完成安装后，保存由Go命令源码文件生成的可执行文件。

目录src用于包含所有的源代码，是Go命令行工具一个强制的规则，而pkg和bin则无需手动创建，如果必要Go命令行工具在构建过程中会自动创建这些目录。 

需要特别注意的是，只有当环境变量GOPATH中只包含一个工作区的目录路径时，go install命令才会把命令源码安装到当前工作区的bin目录下。若环境变量GOPATH中包含多个工作区的目录路径，像这样执行go install命令就会失效，此时必须设置环境变量GOBIN。

### 1.2 GOPATH设置

为了能够构建这个工程，需要先把所需工程的根目录加入到环境变量GOPATH中。否则，即使处于同一工作目录(工作区)，代码之间也无法通过绝对代码包路径完成调用。

在实际开发环境中，工作目录往往有多个。这些工作目录的目录路径都需要添加至GOPATH。当有多个目录时，请注意分隔符，多个目录的时候Windows是分号，Linux系统是冒号，当有多个GOPATH时，默认会将go get的内容放在第一个目录下。

## 2 包

所有 Go 语言的程序都会组织成若干组文件，每组文件被称为一个包。这样每个包的代码都可以作为很小的复用单元，被其他项目引用。

一个包的源代码保存在一个或多个以.go为文件后缀名的源文件中，通常一个包所在目录路径的后缀是包的导入路径。

### 2.1 自定义包

对于一个较大的应用程序，我们应该将它的功能性分隔成逻辑的单元，分别在不同的包里实现。我们创建的的自定义包最好放在GOPATH的src目录下（或者GOPATH src的某个子目录）。

在Go语言中，代码包中的源码文件名可以是任意的。但是，这些任意名称的源码文件都必须以包声明语句作为文件中的第一行，每个包都对应一个独立的名字空间：
package calc

包中成员以名称⾸字母⼤⼩写决定访问权限：

l public: ⾸字母⼤写，可被包外访问

l private: ⾸字母⼩写，仅包内成员可以访问



**注意：**同一个目录下不能定义不同的package。

### 2.2 main包

在 Go 语言里，命名为 main 的包具有特殊的含义。 Go 语言的编译程序会试图把这种名字的包编译为二进制可执行文件。所有用 Go 语言编译的可执行程序都必须有一个名叫 main 的包。一个可执行程序有且仅有一个 main 包。

当编译器发现某个包的名字为 main 时，它一定也会发现名为 main()的函数，否则不会创建可执行文件。 main()函数是程序的入口，所以，如果没有这个函数，程序就没有办法开始执行。程序编译时，会使用声明 main 包的代码所在的目录的目录名作为二进制可执行文件的文件名。

### 2.3 main函数和init函数

Go里面有两个保留的函数：init函数（能够应用于所有的package）和main函数（只能应用于package main）。这两个函数在定义时不能有任何的参数和返回值。虽然一个package里面可以写任意多个init函数，但这无论是对于可读性还是以后的可维护性来说，我们都强烈建议用户在一个package中每个文件只写一个init函数。

Go程序会自动调用init()和main()，所以你不需要在任何地方调用这两个函数。每个package中的init函数都是可选的，但package main就必须包含一个main函数。

每个包可以包含任意多个 init 函数，这些函数都会在程序执行开始的时候被调用。所有被
编译器发现的 init 函数都会安排在 main 函数之前执行。 init 函数用在设置包、初始化变量或者其他要在程序运行前优先完成的引导工作。

程序的初始化和执行都起始于main包。如果main包还导入了其它的包，那么就会在编译时将它们依次导入。
有时一个包会被多个包同时导入，那么它只会被导入一次（例如很多包可能都会用到fmt包，但它只会被导入一次，因为没有必要导入多次）。

当一个包被导入时，如果该包还导入了其它的包，那么会先将其它包导入进来，然后再对这些包中的包级常量和变量进行初始化，接着执行init函数（如果有的话），依次类推。等所有被导入的包都加载完毕了，就会开始对main包中的包级常量和变量进行初始化，然后执行main包中的init函数（如果存在的话），最后执行main函数。下图详细地解释了整个执行过程：

![2.3.init](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5o5nc7oj30bj0543zc.jpg)

示例代码目录结构：![2018-01-01_204724](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5ojqqhrj302z01la9x.jpg)

**main.go**示例代码如下：

```go
// main.go
package main
 
import (
    "fmt"
    "test"
)
 
func main() {
    fmt.Println("main.go main() is called")
 
    test.Test()
}
```

**test.go**示例代码如下：

```go
//test.go
package test
 
import "fmt"
 
func init() {
    fmt.Println("test.go init() is called")
}
 
func Test() {
    fmt.Println("test.go Test() is called")
}
```

运行结果：

![2018-01-01_204638](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5ph45qkj304e01jjrd.jpg)

### 2.4 导入包

导入包需要使用关键字import，它会告诉编译器你想引用该位置的包内的代码。包的路径可以是相对路径，也可以是绝对路径。

```go
//方法1
import "calc"
import "fmt"
 
//方法2
import (
    "calc"
    "fmt"
)
```

标准库中的包会在安装 Go 的位置找到。 Go 开发者创建的包会在 GOPATH 环境变量指定的目录里查找。GOPATH 指定的这些目录就是开发者的个人工作空间。

如果编译器查遍 GOPATH 也没有找到要导入的包，那么在试图对程序执行 run 或者 build
的时候就会出错。

注意：如果导入包之后，未调用其中的函数或者类型将会报出编译错误。

#### 2.4.1 点操作

```go
import (
    //这个点操作的含义是这个包导入之后在你调用这个包的函数时，可以省略前缀的包名
    . "fmt"
)
 
func main() {
    Println("hello go")
}
```

#### 2.4.2 别名操作

在导⼊时，可指定包成员访问⽅式，⽐如对包重命名，以避免同名冲突：

```go
import (
    io "fmt" //fmt改为为io
)
 
func main() {
    io.Println("hello go") //通过io别名调用
}
```

#### 2.4.3 _操作

有时，用户可能需要导入一个包，但是不需要引用这个包的标识符。在这种情况，可以使用空白标识符_来重命名这个导入：

```go
import (
    _ "fmt"
)
```

_操作其实是引入该包，而不直接使用包里面的函数，而是调用了该包里面的init函数。

## 3 测试案例

### 3.1 测试代码

![无标题_副本](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5ucl2kcj30bj06rmy7.jpg)

calc.go代码如下：

```go
package calc
 
func Add(a, b int) int { //加
    return a + b
}
 
func Minus(a, b int) int { //减
    return a - b
}
 
func Multiply(a, b int) int { //乘
    return a * b
}
 
func Divide(a, b int) int { //除
    return a / b
}
```

main.go代码如下：

```go
package main
 
import (
    "calc"
    "fmt"
)
 
func main() {
    a := calc.Add(1, 2)
    fmt.Println("a = ", a)
}
```

### 3.2 GOPATH设置

#### 3.2.1 windows

![1](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5wj95k2j30bj066t9t.jpg)

![2](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5wnbslfj30bj058mxj.jpg)

![3](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5wtzwc4j309f04yaax.jpg)

#### 3.2.2 linux

![无标题_副本](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5x39stpj30bj074q54.jpg)

### 3.3 编译运行程序

![2017-09-24_182223_副本](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5xich3nj309s044gm8.jpg)

### 3.4 go install的使用

设置环境变量GOBIN：

![2017-09-24_182825_副本](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5xwvtwoj30bj05qtar.jpg)

在源码目录，敲go install:

![2017-09-24_182910_副本](https://tva1.sinaimg.cn/large/007S8ZIlly1gfo5ycfqnhj309109s0uo.jpg)