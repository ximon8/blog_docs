```json
{
    "date":"2022.08.26 18:53",
    "tags": ["golang"],
    "description": "Uber go编程规范第一部分：编码和命名",
    "author": "ximon"
}
```


### 1.包名

- 全部小写，没有大写或下划线
- 简短而简洁
- 不使用复数。如tool，而不是tools
- 尽量不要使用信息量不足，很泛泛的名字，如common

**eg**

1.1 最后一个词不是包名

```
import (
  "net/http"

  client "example.com/client-go"  
  trace "example.com/trace/v2" // 最后一个元素是v2
)
```

1.2 最后一个词有冲突时

```
// 不推荐写法
import (
  "fmt"
  "os"

  nettrace "golang.net/x/trace" // 没有冲突，不使用别名
)

// 有冲突时，再起别名
import (
  "fmt"
  "os"
  "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

### 2.函数名

    使用驼峰命名，不要使用下划线。如UberGo，而不是uber_go


### 3.变量和常量

3.1 未导出变量应该带前缀'_'

    对于未导出的全局(包内)vars和consts， 前面加上前缀_，用来明确表示它们是全局符号。错误类型的变量例外，错误类型变量应以err开头。

**eg**

```
// 包内变量
const (
_defaultPort = 8080
_defaultUser = "user"
)
```

3.2 相似声明放一组

**eg**

```
// 导入包
import (
  "a"
  "b"
)
// 常量
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)
```

3.3 本地变量声明

    如果将变量明确设置为某个值，则应使用短变量声明形式 (:=)。但是，在某些情况下，var 使用关键字时默认值会更清晰。例如，声明空切片。

**eg**

```
func f(list []int) string {
  s := "foo"
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }

  return s
}
```

3.4 缩小变量作用域

    如果有可能，尽量缩小变量作用范围。除非它与 减少嵌套的规则冲突。

**eg**

```
// 反面示例
err := ioutil.WriteFile(name, data, 0644)
if err != nil {
 return err
}

// 推荐写法
if err := ioutil.WriteFile(name, data, 0644); err != nil {
 return err
}
```

### 4.import 分组

    导入应该分为两组：标准库、其他库,用空行分开

**eg**

```
import (
  "fmt" // 标准库 
  "os" // 标准库

  "go.uber.org/atomic" // 其他库 
  "golang.org/x/sync/errgroup" // 其他库 
)
```

### 5.defer使用

    使用 defer 释放资源，诸如文件和锁。

### 6. 枚举从1开始

    在 Go 中引入枚举的标准方法是声明一个自定义类型和一个使用了 iota 的 const 组。由于变量的默认值为 0，因此通常应以非零值开头枚举。

**eg**

```
const (
  Add Operation = iota + 1
  Subtract
  Multiply
)
```

### 7.不要使用panic

    在生产环境中运行的代码必须避免出现 panic。panic 是 级联失败 的主要根源 。如果发生错误，该函数必须返回错误，并允许调用方决定如何处理它。

### 8.避免使用init()

    尽可能避免使用init()。当init()是不可避免或可取的，代码应先尝试：

- 避免依赖于其他init()函数的顺序或副作用。虽然init()顺序是明确的，但代码可以更改， 因此init()函数之间的关系可能会使代码变得脆弱和容易出错。
- 避免访问或操作全局或环境状态，如机器信息、环境变量、工作目录、程序参数/输入等。
- 避免I/O，包括文件系统、网络和系统调用。

### 9.流程优化

9.1 减少嵌套

    代码应尽可能先处理错误情况、特殊情况并尽早返回，依次来减少嵌套。

9.2 不必要的else

**eg**

```
// 反面示例
var a int
if b {
  a = 100
} else {
  a = 10
}

// 推荐写法
a := 10
if b {
  a = 100
}
```

### 10.字符串

10.1 使用``，避免转义

**eg**

```
// 反面示例
wantError := "unknown name:\"test\""

//推荐写法
wantError := `unknown error:"test"`
```

10.2 格式化字符串

    如果你在函数外声明Printf-style 函数的格式字符串，请将其设置为const常量。

```
// 反面示例
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)

// 推荐写法
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```
