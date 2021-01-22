[toc]

## 1. gjson

### 1.1.1. 简介

之前我们介绍过gojsonq，可以方便地从一个 JSON 串中读取值。同时它也支持各种查询、汇总统计等功能。今天我们再介绍一个类似的库gjson。在上一篇文章Go 每日一库之 buntdb中我们介绍过 JSON 索引，内部实现其实就是使用gjson这个库。gjson实际上是get + json的缩写，用于读取 JSON 串，同样的还有一个sjson（set + json）库用来设置 JSON 串。

### 1.1.2. 快速使用

先安装：

```
    $ go get github.com/tidwall/gjson
```

后使用：

```go
package main

import (
  "fmt"

  "github.com/tidwall/gjson"
)

func main() {
  json := `{"name":{"first":"www.topgoer.com","last":"dj"},"age":18}`
  lastName := gjson.Get(json, "name.last")
  fmt.Println("last name:", lastName.String())

  age := gjson.Get(json, "age")
  fmt.Println("age:", age.Int())
}
```

使用很简单，只需要传入 JSON 串和要读取的键路径即可。注意一点细节，因为gjson.Get()函数实际上返回的是gjson.Result类型，我们要调用其相应的方法进行转换对应的类型。如上面的String()和Int()方法。

使用很简单，只需要传入 JSON 串和要读取的键路径即可。注意一点细节，因为gjson.Get()函数实际上返回的是gjson.Result类型，我们要调用其相应的方法进行转换对应的类型。如上面的String()和Int()方法。

### 1.1.3. 键路径

键路径实际上是以.分隔的一系列键。gjson支持在键中包含通配符`*`和?，`*`匹配任意多个字符，?匹配单个字符，例如`ca*`可以匹配cat/cate/cake等以ca开头的键，ca?只能匹配cat/cap等以ca开头且后面只有一个字符的键。

数组使用键名 + . + 索引（索引从 0 开始）的方式读取元素，如果键pets对应的值是一个数组，那么pets.0读取数组的第一个元素，pets.1读取第二个元素。

数组长度使用`**`键名 + . + #`**`获取，例如`pets.#`返回数组pets的长度。

如果键名中出现.，那么需要使用\进行转义。

```go
package main

const json = `
{
  "name":{"first":"Tom", "last": "Anderson"},
  "age": 37,
  "children": ["Sara", "Alex", "Jack"],
  "fav.movie": "Dear Hunter",
  "friends": [
    {"first": "Dale", "last":"Murphy", "age": 44, "nets": ["ig", "fb", "tw"]},
    {"first": "Roger", "last": "Craig", "age": 68, "nets": ["fb", "tw"]},
    {"first": "Jane", "last": "Murphy", "age": 47, "nets": ["ig", "tw"]}
  ]
}
`

func main() {
  fmt.Println("last name:", gjson.Get(json, "name.last"))
  fmt.Println("age:", gjson.Get(json, "age"))
  fmt.Println("children:", gjson.Get(json, "children"))
  fmt.Println("children count:", gjson.Get(json, "children.#"))
  fmt.Println("second child:", gjson.Get(json, "children.1"))
  fmt.Println("third child*:", gjson.Get(json, "child*.2"))
  fmt.Println("first c?ild:", gjson.Get(json, "c?ildren.0"))
  fmt.Println("fav.moive", gjson.Get(json, `fav.\moive`))
  fmt.Println("first name of friends:", gjson.Get(json, "friends.#.first"))
  fmt.Println("last name of second friend:", gjson.Get(json, "friends.1.last"))
}
```

前 3 个比较简单，就不赘述了。看后面几个：

- `children.#`：返回数组children的长度；
- `children.1`：读取数组children的第 2 个元素（注意索引从 0 开始）；
- `child*.2`：首先`child*`匹配children，.2读取第 3 个元素；
- `c?ildren.0`：`c?ildren`匹配到children，.0读取第一个元素；
- `fav.\moive`：因为键名中含有.，故需要\转义；
- `friends.#.first`：如果数组后#后还有内容，则以后面的路径读取数组中的每个元素，返回一个新的数组。所以该查询返回的数组所有friends的first字段组成；
- `friends.1.last`：读取friends第 2 个元素的last字段。

运行结果：

```
    last name: Anderson
    age: 37
    children: ["Sara", "Alex", "Jack"]
    children count: 3
    second child: Alex
    third child*: Jack
    first c?ild: Sara
    fave.moive
    first name of friends: ["Dale","Roger","Jane"]
    last name of second friend: Craig
```

对于数组，gjson还支持按条件查询元素，#(条件)返回第一个满足条件的元素，#(条件)#返回所有满足条件的元素。括号内的条件可以有==、!=、<、<=、>、>=，还有简单的模式匹配%（符合某个模式），!%（不符合某个模式）：

```
    fmt.Println(gjson.Get(json, `friends.#(last="Murphy").first`))
    fmt.Println(gjson.Get(json, `friends.#(last="Murphy")#.first`))
    fmt.Println(gjson.Get(json, "friends.#(age>45)#.last"))
    fmt.Println(gjson.Get(json, `friends.#(first%"D*").last`))
    fmt.Println(gjson.Get(json, `friends.#(first!%"D*").last`))
    fmt.Println(gjson.Get(json, `friends.#(nets.#(=="fb"))#.first`))
```

还是使用上面的 JSON 串。

- `friends.#(last="Murphy").first`：`friends.#(last="Murphy")`返回数组friends中第一个last为Murphy的元素，.first表示取出该元素的first字段返回；
- `friends.#(last="Murphy")#.first`：`friends.#(last="Murphy")#`返回数组friends中所有的last为Murphy的元素，然后读取它们的first字段放在一个数组中返回。注意与上面一个的区别；
- `friends.#(age>45)#.last`：`friends.#(age>45)#`返回数组friends中所有年龄大于 45 的元素，然后读取它们的last字段返回；
- `friends.#(first%"D*").last`：`friends.#(first%"D*")`返回数组friends中第一个first字段满足模式D*的元素，取出其last字段返回；
- `friends.#(first!%"D*").last`：```friends.#(first!%"D*")`返回数组friends中第一个first字段`**`不`**`满足模式`D*`的元素，读取其last`字段返回；
- `friends.#(nets.#(=="fb"))#.first`：这是个嵌套条件，`friends.#(nets.#(=="fb"))#`返回数组friends的元素的nets字段中有fb的所有元素，然后取出first字段返回。

运行结果：

```
    Dale
    ["Dale","Jane"]
    ["Craig","Murphy"]
    Murphy
    Craig
    ["Dale","Roger"]
```

### 1.1.4. 修饰符

修饰符是gjson提供的非常强大的功能，和键路径搭配使用。gjson提供了一些内置的修饰符：

- `@reverse`：翻转一个数组；
- `@ugly`：移除 JSON 中的所有空白符；
- `@pretty`：使 JSON 更易用阅读；
- `@this`：返回当前的元素，可以用来返回根元素；
- `@valid`：校验 JSON 的合法性；
- `@flatten`：数组平坦化，即将["a", ["b", "c"]]转为["a","b","c"]；
- `@join`：将多个对象合并到一个对象中。

修饰符的语法和管道类似，以`|`分隔键路径和分隔符。

```go
const json = `{
  "name":{"first":"Tom", "last": "Anderson"},
  "age": 37,
  "children": ["Sara", "Alex", "Jack"],
  "fav.movie": "Dear Hunter",
  "friends": [
    {"first": "Dale", "last":"Murphy", "age": 44, "nets": ["ig", "fb", "tw"]},
    {"first": "Roger", "last": "Craig", "age": 68, "nets": ["fb", "tw"]},
    {"first": "Jane", "last": "Murphy", "age": 47, "nets": ["ig", "tw"]}
  ]
}`

func main() {
  fmt.Println(gjson.Get(json, "children|@reverse"))
  fmt.Println(gjson.Get(json, "children|@reverse|0"))
  fmt.Println(gjson.Get(json, "friends|@ugly"))
  fmt.Println(gjson.Get(json, "friends|@pretty"))
  fmt.Println(gjson.Get(json, "@this"))

  nestedJSON := `{"nested": ["one", "two", ["three", "four"]]}`
  fmt.Println(gjson.Get(nestedJSON, "nested|@flatten"))

  userJSON := `{"info":[{"name":"dj", "age":18},{"phone":"123456789","email":"dj@example.com"}]}`
  fmt.Println(gjson.Get(userJSON, "info|@join"))
}
```

`children|@reverse`先读取数组children，然后使用修饰符`@reverse`翻转之后返回，输出：

```
    ["Jack","Alex","Sara"]
```

`children|@reverse|0`在上面翻转的基础上读取第一个元素，即原数组的最后一个元素，输出：

```
    Jack
```

`friends|@ugly`移除friends数组中的所有空白字符，返回一行长长的字符串：

```
    [{"first":"Dale","last":"Murphy","age":44,"nets":["ig","fb","tw"]},{"first":"Roger","last":"Craig","age":68,"nets":["fb","tw"]},{"first":"Jane","last":"Murphy","age":47,"nets":["ig","tw"]}]
```

`friends|@pretty`格式化friends数组，使之更易读：

```
[
  {
    "first": "Dale",
    "last": "Murphy",
    "age": 44,
    "nets": ["ig", "fb", "tw"]
  },
  {
    "first": "Roger",
    "last": "Craig",
    "age": 68,
    "nets": ["fb", "tw"]
  },
  {
    "first": "Jane",
    "last": "Murphy",
    "age": 47,
    "nets": ["ig", "tw"]
  }
]
```

`@this`返回原始的 JSON 串。

`@flatten`将数组nested的内层数组平坦到外层后返回，即将所有内层数组的元素依次添加到外层数组后面并移除内层数组，输出：

```
    ["one","two","three", "four"]
```

`@join`将一个数组中的各个对象合并到一个中，例子中将数组中存放的部分个人信息合并成一个对象返回：

```
    {"name":"dj","age":18,"phone":"123456789","email":"dj@example.com"}
```

### 1.1.5. 修饰符参数

修饰符还可以有参数，通过在修饰符后加:后跟参数。如果我们在格式化 JSON 串时，想要对键进行排序，那么可以使用@pretty修饰符的sortKeys参数。我们还是拿上面的 JSON 数据举例：

```
    fmt.Println(gjson.Get(json, `friends|@pretty:{"sortKeys":true}`))
```

最终按键名顺序输出 JSON 串：

```
[
  {
    "age": 44,
    "first": "Dale",
    "last": "Murphy",
    "nets": ["ig", "fb", "tw"]
  },
  {
    "age": 68,
    "first": "Roger",
    "last": "Craig",
    "nets": ["fb", "tw"]
  },
  {
    "age": 47,
    "first": "Jane",
    "last": "Murphy",
    "nets": ["ig", "tw"]
  }
]
```

当然还可以指定每行缩进indent（默认两个空格），每行开头字符串prefix（默认为空串）和一行最多显示字符数width（默认 80 字符）。下面在每行前增加两个空格：

```
    fmt.Println(gjson.Get(json, `friends|@pretty:{"sortKeys":true,"prefix":"  "}`))
```

### 1.1.6. 自定义修饰符

如此强大的功当然要支持自定义！gjson使用AddModifier()添加一个修饰符，传入一个名字和类型为func(json arg string) string的处理函数。处理函数接受待处理的 JSON 值和修饰符参数，返回处理后的结果。下面编写一个转换大小写的修饰符：

```go
func main() {
  gjson.AddModifier("case", func(json, arg string) string {
    if arg == "upper" {
      return strings.ToUpper(json)
    }

    if arg == "lower" {
      return strings.ToLower(json)
    }

    return json
  })

  const json = `{"children": ["Sara", "Alex", "Jack"]}`
  fmt.Println(gjson.Get(json, "children|@case:upper"))
  fmt.Println(gjson.Get(json, "children|@case:lower"))
}
```

输出：

```
    ["SARA", "ALEX", "JACK"]
    ["sara", "alex", "jack"]
```

### 1.1.7. JSON 行

gjson提供..语法可以将多行数据看成一个数组，每行数据是一个元素：

```go
const json = `
{"name": "Gilbert", "age": 61}
{"name": "Alexa", "age": 34}
{"name": "May", "age": 57}
{"name": "Deloise", "age": 44}`

func main() {
  fmt.Println(gjson.Get(json, "..#"))
  fmt.Println(gjson.Get(json, "..1"))
  fmt.Println(gjson.Get(json, "..#.name"))
  fmt.Println(gjson.Get(json, `..#(name="May").age`))
}
```

- ..#：返回有多少行 JSON 数据；
- ..1：返回第一行，即{"name": "Gilbert", "age": 61}；
- ..#.name：#后再接路径，表示对数组中每个元素读取后面的路径，将读取到的值组成一个新数组返回；..#.name表示读取每一行中的name字段，最终返回["Gilbert","Alexa","May","Deloise"]；
- ..#(name="May").age：括号中的内容(name="May")表示条件，所以该条含义为取name为"May"的行中的age字段。

gjson还提供了遍历 JSON 行的方法：gjson.ForEachLine()，参数为 JSON 串和类型为func(line gjson.Result) bool的回调函数。回调返回false时遍历停止。下面代码读取输出每一行的name字段：

```go
gjson.ForEachLine(json, func(line gjson.Result) bool {
  fmt.Println("name:", gjson.Get(line.String(), "name"))
  return true
})
```

### 1.1.8. 遍历

上面我们介绍了遍历 JSON 行的方式，实际上gjson还提供了通用的遍历数组和对象的方式。gjson.Get()方法返回一个gjson.Result类型的对象，json.Result提供了ForEach()方法用于遍历。该方法接受一个类型为func (key, value gjson.Result) bool的回调函数。遍历对象时key和value分别为对象的键和值；遍历数组时，value为数组元素，key为空（不是索引）。回调返回false时，遍历停止。

```go
const json = `
{
  "name":"dj",
  "age":18,
  "pets": ["cat", "dog"],
  "contact": {
    "phone": "123456789",
    "email": "dj@example.com"
  }
}`

func main() {
  pets := gjson.Get(json, "pets")
  pets.ForEach(func(_, pet gjson.Result) bool {
    fmt.Println(pet)
    return true
  })

  contact := gjson.Get(json, "contact")
  contact.ForEach(func(key, value gjson.Result) bool {
    fmt.Println(key, value)
    return true
  })
}
```

### 1.1.9. 校验 JSON

调用gjson.Get()时，gjson假设我们传入的 JSON 串是合法的。如果 JSON 非法也不会panic，这时会返回不确定的结果：

```go
func main() {
  const json = `{"name":dj,age:18}`
  fmt.Println(gjson.Get(json, "name"))
}
```

上面 JSON 串是非法的，dj和age都没有加上双引号（实际上习惯了 Go 语言map的写法，很容易把 JSON 写成这样😭）。上面代码输出18，显然是错误的。我们可以使用gjson.Valid()检测 JSON 串是否合法：

```go
if !gjson.Valid(json) {
  fmt.Println("error")
} else {
  fmt.Println("ok")
}
```

### 1.1.10. 一次获取多个值

调用gjson.Get()一次只能读取一个值，多次调用又比较麻烦，gjson提供了GetMany()可以一次读取多个值，返回一个数组[]gjson.Result。

```go
const json = `
{
  "name":"dj",
  "age":18,
  "pets": ["cat", "dog"],
  "contact": {
    "phone": "123456789",
    "email": "dj@example.com"
  }
}`

func main() {
  results := gjson.GetMany(json, "name", "age", "pets.#", "contact.phone")
  for _, result := range results {
    fmt.Println(result)
  }
}
```

上面代码返回字段name、age、数组pets的长度和contact.phone字段。

文章转自：Go 每日一库 GitHub：https://github.com/darjun/go-daily-lib