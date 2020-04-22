## jsoniter



```go
import (
    "fmt"
    "github.com/json-iterator/go"   // 引入
    "os"
    "strings"
)

type ColorGroup struct {
    ID      int
    Name    string
    Colors  []string
}

type Animal struct {
    Name    string
    Order   string
}

func main() {
    // ================= 序列化 =====================
    group := ColorGroup{
        ID:     1,
        Name:   "Reds",
        Colors: []string{"Crimson", "Red", "Ruby", "Maroon"},
    }
    b, err := jsoniter.Marshal(group)
    bb, err :=  jsoniter.MarshalIndent(group, "", " ")
    if err != nil{
        fmt.Println("error: ", err)
    }
    os.Stdout.Write(b)
    fmt.Println()
    os.Stdout.Write(bb)
    fmt.Println()

    // ===================  Deconde 解码 =================
    jsoniter.NewDecoder(os.Stdin).Decode(&group)
    fmt.Println(group)

    //encoder := jsoniter.NewEncoder(os.Stdout)
    //encoder.SetEscapeHTML(true)
    //encoder.Encode(bb)
    //fmt.Println(string(bb))

    // =================== 反序列化 =======================
    var jsonBlob = []byte(`[
        {"Name": "Platypus", "Order": "Monotremata"},
        {"Name": "Quoll",    "Order": "Dasyuromorphia"}
    ]`)
    var animals []Animal
    if err := jsoniter.Unmarshal(jsonBlob, &animals); err != nil{
        fmt.Println("error: ", err)
    }

    fmt.Printf("the unmarshal is  %+v", animals)

    // ======================= 流式 ========================
    fmt.Println()

    // 序列化
    stream := jsoniter.ConfigFastest.BorrowStream(nil)
    defer jsoniter.ConfigFastest.ReturnStream(stream)
    stream.WriteVal(group)
    if stream.Error != nil{
        fmt.Println("error: ", stream.Error)
    }
    os.Stdout.Write(stream.Buffer())

    fmt.Println()
    // 反序列化
    iter := jsoniter.ConfigFastest.BorrowIterator(jsonBlob)
    defer jsoniter.ConfigFastest.ReturnIterator(iter)
    iter.ReadVal(&animals)
    if iter.Error != nil{
        fmt.Println("error： ", iter.Error)
    }
    fmt.Printf("%+v", animals)

    fmt.Println()
    // ====================其他操作===================
    // get
    val := []byte(`{"ID":1,"Name":"Reds","Colors":
{"c":"Crimson","r":"Red","rb":"Ruby","m":"Maroon","tests":["tests_1","tests_2","tests_3","tests_4"]}}`)
    fmt.Println(jsoniter.Get(val, "Colors").ToString())
    fmt.Println("the result is " , jsoniter.Get(val, "Colors","tests",0).ToString())
    // fmt.Println(jsoniter.Get(val, "colors", 0).ToString())

    fmt.Println()
    hello := MyKey("hello")
    output, _ := jsoniter.Marshal(map[*MyKey]string{&hello: "world"})
    fmt.Println(string(output))

    obj := map[*MyKey]string{}
    jsoniter.Unmarshal(output, &obj)
    for k, v := range obj{
        fmt.Println(*k," = ", v)
    }

}
// 自定义类型
// 序列化： 需要实现MarshellText
type MyKey string

func (m *MyKey) MarshalText() ([]byte, error){
    // return []byte(string(*m)) , nil  // 针对序列化的内容不做任何调整
    return []byte(strings.Replace(string(*m), "h","H",-1)), nil
}

func(m *MyKey) UnmarshalText(text []byte) error{
    *m = MyKey(text[:])  // 针对text不做处理
    return nil
}
```



```go
// 初始化大文件
func init() {
    ioutil.WriteFile("large-file.json", []byte(`[{
  "person": {
    "id": "d50887ca-a6ce-4e59-b89f-14f0b5d03b03",
    "name": {
      "fullName": "Leonid Bugaev",
      "givenName": "Leonid",
      "familyName": "Bugaev"
    },
    "email": "leonsbox@gmail.com",
    "gender": "male",
    "location": "Saint Petersburg, Saint Petersburg, RU",
    "geo": {
      "city": "Saint Petersburg",
      "state": "Saint Petersburg",
      "country": "Russia",
      "lat": 59.9342802,
      "lng": 30.3350986
    },
    "bio": "Senior engineer at Granify.com",
    "site": "http://flickfaver.com",
    "avatar": "https://d1ts43dypk8bqh.cloudfront.net/v1/avatars/d50887ca-a6ce-4e59-b89f-14f0b5d03b03",
    "employment": {
      "name": "www.latera.ru",
      "title": "Software Engineer",
      "domain": "gmail.com"
    },
    "facebook": {
      "handle": "leonid.bugaev"
    },
    "github": {
      "handle": "buger",
      "id": 14009,
      "avatar": "https://avatars.githubusercontent.com/u/14009?v=3",
      "company": "Granify",
      "blog": "http://leonsbox.com",
      "followers": 95,
      "following": 10
    },
    "twitter": {
      "handle": "flickfaver",
      "id": 77004410,
      "bio": null,
      "followers": 2,
      "following": 1,
      "statuses": 5,
      "favorites": 0,
      "location": "",
      "site": "http://flickfaver.com",
      "avatar": null
    },
    "linkedin": {
      "handle": "in/leonidbugaev"
    },
    "googleplus": {
      "handle": null
    },
    "angellist": {
      "handle": "leonid-bugaev",
      "id": 61541,
      "bio": "Senior engineer at Granify.com",
      "blog": "http://buger.github.com",
      "site": "http://buger.github.com",
      "followers": 41,
      "avatar": "https://d1qb2nb5cznatu.cloudfront.net/users/61541-medium_jpg?1405474390"
    },
    "klout": {
      "handle": null,
      "score": null
    },
    "foursquare": {
      "handle": null
    },
    "aboutme": {
      "handle": "leonid.bugaev",
      "bio": null,
      "avatar": null
    },
    "gravatar": {
      "handle": "buger",
      "urls": [
      ],
      "avatar": "http://1.gravatar.com/avatar/f7c8edd577d13b8930d5522f28123510",
      "avatars": [
        {
          "url": "http://1.gravatar.com/avatar/f7c8edd577d13b8930d5522f28123510",
          "type": "thumbnail"
        }
      ]
    },
    "fuzzy": false
  },
  "company": "hello"
}]`), 0666)
}

/*
200000        8886 ns/op        4336 B/op          6 allocs/op
50000        34244 ns/op        6744 B/op         14 allocs/op
*/
// 解析json大文件
func Benchmark_jsoniter_large_file(b *testing.B) {
    b.ReportAllocs()
    for n := 0; n < b.N; n++ {
        file, _ := os.Open("large-file.json")
        iter := jsoniter.Parse(jsoniter.ConfigDefault, file, 4096)
        count := 0
        iter.ReadArrayCB(func(iter *jsoniter.Iterator) bool {
            // Skip() is strict by default, use --tags jsoniter-sloppy to skip without validation
            iter.Skip()
            count++
            return true
        })
        file.Close()
        if iter.Error != nil {
            b.Error(iter.Error)
        }
    }
}
// 反序列化文件内容
func Benchmark_json_large_file(b *testing.B) {
    b.ReportAllocs()
    for n := 0; n < b.N; n++ {
        file, _ := os.Open("large-file.json")
        bytes, _ := ioutil.ReadAll(file)
        file.Close()
        result := []struct{}{}
        err := json.Unmarshal(bytes, &result)
        if err != nil {
            b.Error(err)
        }
    }
}
```

