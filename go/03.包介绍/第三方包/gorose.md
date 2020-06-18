# gorose - 使用orm的方式快捷操作数据库

[项目地址](https://github.com/gohouse/gorose)

## 引入

- go.mod

```
require github.com/gohouse/gorose/v2 v2.1.5
```

```go
import (
  "github.com/gohouse/gorose/v2"
	
  // 需引入指定数据库的底层驱动
  _ "github.com/go-sql-driver/mysql"    // mysql
)
```

### 各数据库的 DSN协议

```go
// mysql
dsn = fmt.Sprintf("%s:%s@tcp(%s)/%s?charset=utf8&parseTime=true", username, password, host, database)

// clickhouse
dsn = "tcp://host1:9000?username=user&password=qwerty&database=clicks&read_timeout=10&write_timeout=20&alt_hosts=host2:9000,host3:9000"

// sqlite3
dsn = "test.db?cache=shared&mode=memory"
```

## 基本示例

### 创建连接池

```go
DB, err := gorose.Open(&gorose.Config{Driver: "sqlite3", Dsn: "./db.sqlite"})
```

### 插入数据

```go
orm := DB.NewOrm()
cnt, err = orm.Table(h.todoTable).Data(map[string]interface{}{"jobid":jobid}).Insert()

// -------------------------------
// 一条数据
var data = map[string]interface{}{"age":17, "job":"it3"}
// insert into user (age, job) values (17, 'it3')
db.Table(&user).Data(data).Insert()

// 多条数据
var multi_data = []map[string]interface{}{ {"age":17, "job":"it3"},{"age":17, "job":"it4"} }
// insert into user (age, job) values (17, 'it3') (17, 'it4')
db.Table("user").Data(multi_data).Insert()
```

### 更新数据

```go
orm := DB.NewOrm()
cnt, err = orm.Table(h.timeTable).Data(map[string]interface{}{"time": c.NewCheckpoint}).Where("tag", h.timeKey).Update()

// Update() 后会自动回收链接

// -----------------------------------------------------------
// update user set age=17, job='ite3' where (id=1) or (age>30)
db.Table("user").
	Data(map[string]interface{}{"age":17, "job":"it3"}).
    Where("id", 1).
    OrWhere("age",">",30).
    Update()

```



### 查询数据

```go
orm := DB.NewOrm()

if err := orm.Table(h.jobTable).					// 设置 table 名称，这里也可以是对应的结构体，但要实现结构 .Table()
    Fields("jobid", "end", "updated").
    WhereNotRegexp("jobid", ".*\\..*").
    Where("state", "!=", "RUNNING").
    Where("end", "!=", "1970-01-01 00:00:01").
    Where("updated", ">=", checkpoint).
    OrderBy("updated").
    Select(); err != nil{
    return err
}

// log.Printf("sql:%s", orm.LastSql())

rows, err := orm.Get()
if err != nil {
    return err
}

if len(rows) > 0{
    for _, row := range rows{
        jobid   := row["jobid"].(string)
        end     := row["end"].(time.Time)
        updated := row["updated"].(time.Time)
      
        // do something you needed
    }

    return nil
}
```

### 删除数据

```go
_, err = orm.Table(h.historyTable).Where("jobid", jobid).Delete()
```


### 事务

```go
func (c*Context) CommitTodoJobids(jobids []string) error {

    var err error

    h := c.h

    orm := h.DB2.NewOrm()

    orm.Begin()

    // remove todos from Todo table
    for _, jobid := range c.Todos {
        _, err = orm.Table(h.todoTable).Where("jobid", jobid).Delete()
        if err != nil {
            goto ErrRet
        }
    }

    // insert todos to history table
    for _, jobid := range c.Todos {
        _, err = orm.Table(h.historyTable).Fields("jobid").Insert(map[string]interface{}{"jobid" : jobid})
        if err != nil {
            if strings.Contains(err.Error(), "Duplicate entry"){
                err = nil
            } else {
                goto ErrRet
            }
        }
    }

    err = orm.Commit()
    if err != nil {
        goto ErrRet
    }

    return nil

ErrRet:
    orm.Rollback()

    return err

}
```

## 详细

### 查询时扫描入结构体

```go
// struct类型
type Users struct {
    Uid   int64  `gorose:"uid"`
    Name  string `gorose:"name"`
    Age   int64  `gorose:"age"`
    State int64  `gorose:"-"`       // - 意为忽略
}

// 绑定表名
func (u Users) TableName() string {
 	return "users"
}

// struct类型的一条数据, 这里默认会加上limit 1, 节省空间
{
  var u Users
  // var u = Users{}
  err := db.Table(&u).Select()
}

// struct类型的多条数据
{
  var u []Users
  // var u = []Users{}
  err := db.Table(&u).Limit(5).Select()
}
```

#### 设置解析的tagname和ignore

```go
engin,err:=gorose.Open()
engin.TagName("orm").IgnoreName("ignore")

// 若作了上述配置，那么结构体的定义应该如下
type Users struct {
    Uid   int64  `orm:"uid"`
    Name  string `orm:"name"`
    Age   int64  `orm:"age"`
    State int64  `orm:"ignore"`
}
```

### 查询时存入map

```go
// 一条数据的map类型
{
  type userMap gorose.Map
  func (u userMap) TableName() string {
    return "users"
  }

  var u = userMap{}   					
  err := db.Table(&u).Select()  // map类型的一条数据, 这里默认会加上limit 1, 节省空间
}

```

```go
{
  // 多条数据的map类型
  // 这里的多条数据, 只能预先定义为 userMap2, 不能像 struct 的多条那样直接复用[]Users
  // 因为需要获取对应的 TableName(), 当然, 如果名字刚好是表名,可以不添加`TableName()`
  type userMap2 []gorose.Map
  func (u userMap2) TableName() string {
    return "users"
  }

  var u = userMap2{}               // 注意: 这里是map类型, 必须初始化
  err := db.Table(&u).Limit(5).Select()
}
```

### 直接返回map

```go
// map类型的一条数据, 这里默认会加上limit 1, 节省空间
res,err := db.Table("users").First()

// map类型的多条数据
res,err := db.Table("users").Limit(5).Get()
fmt.Println(res)
```

### orm API

#### `Fields()`和`AddFields()`

```go
db.Table("users").Fields("uid").AddFields("name,age").First()
```

> Fields()会替换为最新的字段, AddFields()则会追加最新字段, 只有当传入得table为string和map类型时有效,传入struct时该方法无效,因为会自动解析struct自身的字段

#### `Distinct()`

```go
db.Table("users").Distinct().First()
```

#### `Where`...

```go
// select * from users where id=1
db.Table(&user).Where("id", 1).Select()
// select * from users where id>1
db.Table(&user).Where("id", ">", 1).Select()
// select * from users where id>1 and name like "%fizz%"
db.Table(&user).Where("id", ">", 1).Where("name","like","%fizz%").Select()
// select * from users where id>1 or name like "%fizz%"
db.Table(&user).Where("id", ">", 1).OrWhere("name","like","%fizz%").Select()
// select * from users where job is null
db.Table(&users).WhereNull("job").Select()
// select * from users where job is not null
db.Table(&users).WhereNotNull("job").Select()
// select * from users where age in (17,18)
db.Table(&users).WhereIn("age",[]interface{}{17, 18}).Select()
// select * from users where age between 17 and 20
db.Table(&users).WhereBetween("age",[]interface{}{17, 18}).Select()
// select * from users where uid=1 and age=18 limit 1
db.Table("users").Where(gorose.Data{"uid":1, "age":18}).First()
```

#### 嵌套where

```go
// SELECT  * FROM users  
//     WHERE  uid > 1 
//         and ( name = 'fizz' 
//             or ( name = 'fizz2' 
//                 and ( name = 'fizz3' or website like 'fizzday%')
//                 )
//             ) 
//     and job = 'it' LIMIT 1
User := db.Table("users")
User.Where("uid", ">", 1).Where(func() {
		User.Where("name", "fizz").OrWhere(func() {
			User.Where("name", "fizz2").Where(func() {
				User.Where("name", "fizz3").OrWhere("website", "like", "fizzday%")
			})
		})
	}).Where("job", "it").First()
```

#### `Join`...

```go
// 普通示例  
// select * from user inner join card on user.id=card.user_id limit 10
db.Table("user")
    .Join("card","user.id","=","card.user_id")
    .Limit(10)
    .Get()
  
// 左链接  
// select * from user a left join card b on a.id=b.user_id limit 1
db.Table("user a")
    .LeftJoin("card b on a.id = b.user_id")
    .First()

//  RightJoin => right join
// CrossJoin => cross join
// UnionJoin => union join
```

默认 `join` 是 `inner join`

#### Order / OrderBy

```go
// select * from users order by id desc, job asc
db.Table(&user).Order("id desc, job asc").Select()
```

Order 等同于 orderby

#### Group / GroupBy

```go
// select * from users group by job
db.Table(&user).Group("job").Select()
```

Group 等同于 GroupBy

#### Limit 和 Offset

```go
// select * from users limit 10 offset 10
db.Table(&user).Limit(10).Offset(10).Select()
```

#### Having

```go
// select * from users group by job having age>17
db.Table(&user).Group("job").Having("age>17").Select()
```

#### Value 获取一个字段的单一值

```go
// 获取id=1的job
db.Table(&user).Value("job")	// it
```

#### Pluck 获取表的一列值

```go
// 获取id=1的job
db.Table(&user).Pluck("job")	// ["id", "drawer"]
// 可以指定第二个参数, 第二个参数的值, 做为第一个字段值得key, 这两个参数都必须为表的字段
db.Table(&user).Pluck("job", "name")	// {"fizz":id", "fizz2":"drawer"}
```

#### Sum / Count / Avg / Max / Min 聚合查询

```go
// select sum(age) from users
db.Table(&user).Sum("age")
db.Table(&user).Count()
db.Table(&user).Count("*")
db.Table(&user).Avg("age")
db.Table(&user).Max("age")
db.Table(&user).Min("age")
```

#### Chunk 数据分片 大量数据批量处理 (累积处理)

```go
` 当需要操作大量数据的时候, 一次性取出再操作, 不太合理, 就可以使用chunk方法  
  chunk的第一个参数是指定一次操作的数据量, 根据业务量, 取100条或者1000条都可以  
  chunk的第二个参数是一个回调方法, 用于书写正常的数据处理逻辑  
  目的是做到, 无感知处理大量数据  
  实现原理是, 每一次操作, 自动记录当前的操作位置, 下一次重复取数据的时候, 从当前位置开始取
`

User := db.Table("users")
User.Fields("id, name").Where("id",">",2).Chunk(2, func(data []gorose.Data) error {
  // for _,item := range data {
  // 	   fmt.Println(item)
  // }
  fmt.Println(data)

  // 这里不要忘记返回错误或nil
  return nil
})

// 打印结果:  
// map[id:3 name:gorose]
// map[id:4 name:fizzday]
// map[id:5 name:fizz3]
// map[id:6 name:gohouse]
[map[id:3 name:gorose] map[name:fizzday id:4]]
[map[id:5 name:fizz3] map[id:6 name:gohouse]]
```

#### Loop 数据分片 大量数据批量处理 (从头处理)

```go
   ` 类似 chunk 方法, 实现原理是, 每一次操作, 都是从头开始取数据
   原因: 当我们更改数据时, 更改的结果可能作为where条件会影响我们取数据的结果,所以, 可以使用Loop`

	User := db.Table("users")
    User.Fields("id, name").Where("id",">",2).Loop(2, func(data []gorose.Data) error {
        // for _,item := range data {
        // 	   fmt.Println(item)
        // }
        // 这里执行update / delete  等操作
        
        // 这里不要忘记返回错误或nil
        return nil
    })
```

#### 获取sql记录

- 直接构建sql而不执行

```go
// select 语句
db.Table().BuildSql() // 或者 xxx.BuildSql("select")
// insert
db.Table().Data().BuildSql("insert") 
// update, delete
// xxx.BuildSql("update")
// xxx.BuildSql("delete")
```

- 执行过程中获取sql

```go
// 要先新建会话
db := DB()
db.Table().xxx()
// 获取最后一条sql
fmt.Println(db.LastSql())
```

### 事务

#### 标准事务

```go
db := engin.NewOrm()
// 开始事务
db.Begin()

res,err := db.Table("user").Where("id", 1).Data(data).Update()
if (res == 0 || err!=nil) {
    // 回滚事务
    db.Rollback()
}

res2,err2 := db.Table("user").Data(data).Insert()
if (res2 == 0 || err2!=nil) {
    // 回滚事务
    db.Rollback()
}
// 提交事务
db.Commit()
```

#### 一键事务

```go
db:= DB()
err := db.Transaction(
    // 第一个业务
    func(db IOrm) error {
        _,err := db.Where("uid",1).Update(&Data{"name":"gorose2"})
        if err!=nil {
            return err
        }
        _,err = db.Insert(&Data{"name":"gorose2"})
        if err!=nil {
            return err
        }
        return nil
    }, 
    // 第二个业务
    func(db IOrm) error {
        _,err := db.Where("uid",1).Delete()
        if err!=nil {
            return err
        }
        _,err = db.Insert(&Data{"name":"gorose2"})
        if err!=nil {
            return err
        }
        return nil
    }
)
```

内部闭包会在返回错误时自动回滚, 会在没有错误时自动提交事务

**特别说明:**

1. 传入的数据如果是struct绑定数据, 则默认忽略字段类型默认值.
   如果想更新的话, 则可以使用 `ExtraCols(args ...string)`,如:`xx.ExtraCols("field","field2").Update(&Data{xxx})`

### 原生查询

直接返回结果集

```go
// 原生sql
res,err := DB().Query("select * from users where uid>? limit 2", 1)
fmt.Println(res)
affected_rows,err := DB().Execute("delete from users where uid=?", 1)
fmt.Println(affected_rows, err)
```

也可以绑定查询结果到给定对象

```go
session := engin.NewSession()

var u = make(map[string]interface{})
session := engin.NewSession()
// 这里Bind()是为了存放结果的, 如果你使用的是NewOrm()初始化,则可以直接使用 NewOrm().Table().Query()
_,err := session.Bind(&u).Query("select * from users where uid=? limit 2", 1)
```

更改操作

```go
session := engin.NewSession()

session.Execute("insert into users(name,age) values(?,?)(?,?)", "gorose",18,"fizzday",19)
session.Execute("update users set name=? where uid=?","gorose",1)
session.Execute("delete from users where uid=?", 1)

// 事务, 用法跟orm的事务类似
session.Transaction()

session.LastSql()
```