# 基本操作备忘

初始化Go项目

```bash
go mod init github.com/lix7/go-example 
```

for是唯一的循环语句

```go
// infinite
for {

}

// while-like
for checkCondition {

}

// range slice会输出一个[索引, 元素]的二元组
for _, item := range os.Args[1:] {

}
// 如果只需要索引，不需要元素
for i := range os.Args[1:] {

}

// range map会输出[key, value]二元组
for key, value := range make(map[string]int) {
    
}

// 标准
for i := 1; i < 10; i++ {
    
}

```

变量初始化

```go
// 函数内
s := "func var"

// 包级变量
var s string = "package var"
```

变量赋值可以这样

```go
// swap
i, j = j, i
```

类型断言

```go
v, ok := x.(T)
```

对于每个类型T，都有一个对应的类型转换操作`T(x)`将值x转换为类型T。如果两个类型具有相同的底层类型或二者都是指向相同底层类型变量的未命名指针类型，则二者是可以相互转换的

```go
import "fmt"

func main() {
	type a int
	type b int
	var va a = 1
	var vb b = 2
	vbb := a(vb)
	fmt.Println(va, vb, vbb)
}
```

`rune`和`int32`是同义词，常常用于指明一个值是unicode码点，统计字符串字符数量需要使用utf8方法，`len()`只统计了字符串的字节数量

```go
	s := "abc测试"
	for _, c := range s {
		fmt.Printf("%d-%c-%c ", c, c, rune(c))
	}
	fmt.Println()
	fmt.Println("bytes count: ", len(s))
	fmt.Println("runes count: ", utf8.RuneCountInString(s))
    
    // 97-a-a 98-b-b 99-c-c 27979-测-测 35797-试-试 
    // bytes count:  9
    // runes count:  5
```

使用反引号包装原生字符串和多行字符串

```go
	text := `
this is multiple lines of code
	let's review it
	printf \n not work here
	`

	fmt.Print(text)
```

# Go 潜规则

- main包用来定义一个可独立执行的程序，main函数为程序入口

- 测试文件文件名以Test为前缀，例如 TestAdd()

- internal 包限制外部 import

- go不允许无用的变量存在，否则会编译错误，必须显式丢弃 `_`

- 实践中，我们应当使用前两种形式，使用显式的初始化来说明初始化变量的重要性，使用隐式的初始化来表明初始化变量不重要。

- 实体的名字首字母是否为大写决定了其是否是外部可见的

- 两个指针当且仅当指向同一个变量或者两者都是 nil 的情况下才相等

```go
var x, y int
fmt.Println(&x == &x, &x == &y, &x == nil) // true false false
```

- go中`fmt.Printf("%s", v)`使用的方法为 `String()`

- go文件可以包含一个`func init(){}`函数，该函数在文件初始化时有编译器调用执行

- 一个文件夹（不包括子文件夹）会被归类为一个go package，该package下所有的go文件中导出的实体（字段、方法）都可以直接通过`包名.实体`访问。这就限制了同一个包下不同的Go文件中不能包含相同名字的变量或方法

- 字符串第 $i$ 个字节不一定是第 $i$ 个字符，因为 `utf-8` 编码中一个字符可能对应1、2、3、4个字节

- go的参数传递为值传递，即使是slice也是进行了一次拷贝的值传递。可以使用slice指针类型作为参数避免拷贝开销。