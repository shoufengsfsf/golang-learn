# 流程控制

## 1 选择结构

### 1.1 if语句

#### 1.1.1 if

```go
    var a int = 3
    if a == 3 { //条件表达式没有括号
        fmt.Println("a==3")
    }
 
    //支持一个初始化表达式, 初始化字句和条件表达式直接需要用分号分隔
    if b := 3; b == 3 {
        fmt.Println("b==3")
    }
```

#### 1.1.2 if ... else

```go
    if a := 3; a == 4 {
        fmt.Println("a==4")
    } else { //左大括号必须和条件语句或else在同一行
        fmt.Println("a!=4")
    }
```

#### 1.1.3 if ... else if ... else

```go
    if a := 3; a > 3 {
        fmt.Println("a>3")
    } else if a < 3 {
        fmt.Println("a<3")
    } else if a == 3 {
        fmt.Println("a==3")
    } else {
        fmt.Println("error")
    }
```

### 1.2 switch语句

Go里面switch默认相当于每个case最后带有break，匹配成功后不会自动向下执行其他case，而是跳出整个switch, 但是可以使用fallthrough强制执行后面的case代码：

```go
    var score int = 90
 
    switch score {
    case 90:
        fmt.Println("优秀")
        //fallthrough
    case 80:
        fmt.Println("良好")
        //fallthrough
    case 50, 60, 70:
        fmt.Println("一般")
        //fallthrough
    default:
        fmt.Println("差")
    }
 
```

可以使用任何类型或表达式作为条件语句：

```go
    //1
    switch s1 := 90; s1 { //初始化语句;条件
    case 90:
        fmt.Println("优秀")
    case 80:
        fmt.Println("良好")
    default:
        fmt.Println("一般")
    }
 
    //2
    var s2 int = 90
    switch { //这里没有写条件
    case s2 >= 90: //这里写判断语句
        fmt.Println("优秀")
    case s2 >= 80:
        fmt.Println("良好")
    default:
        fmt.Println("一般")
    }
 
    //3
    switch s3 := 90; { //只有初始化语句，没有条件
    case s3 >= 90: //这里写判断语句
        fmt.Println("优秀")
    case s3 >= 80:
        fmt.Println("良好")
    default:
        fmt.Println("一般")
    }
```

## 2 循环语句

### 2.1 for

```go
    var i, sum int
 
    for i = 1; i <= 100; i++ {
        sum += i
    }
    fmt.Println("sum = ", sum)
```

### 2.2 range

关键字 range 会返回两个值，第一个返回值是元素的数组下标，第二个返回值是元素的值：

```go
    s := "abc"
    for i := range s { //支持 string/array/slice/map。
        fmt.Printf("%c\n", s[i])
    }

    for _, c := range s { // 忽略 index
        fmt.Printf("%c\n", c)
    }

    for i, c := range s {
        fmt.Printf("%d, %c\n", i, c)
    }

```

## 3 跳转语句

### 3.1 break和continue

在循环里面有两个关键操作break和continue，break操作是跳出当前循环，continue是跳过本次循环。

```go
    for i := 0; i < 5; i++ {
        if 2 == i {
            //break    //break操作是跳出当前循环
            continue //continue是跳过本次循环
        }
        fmt.Println(i)
    }
```

 

注意：break可⽤于for、switch、select，⽽continue仅能⽤于for循环。

### 3.2 goto

用goto跳转到必须在当前函数内定义的标签：

```go
func main() {
    for i := 0; i < 5; i++ {
        for {
            fmt.Println(i)
            goto LABEL //跳转到标签LABEL，从标签处，执行代码
        }
    }
 
    fmt.Println("this is test")
 
LABEL:
    fmt.Println("it is over")
}
```

