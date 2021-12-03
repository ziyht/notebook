## pprof



### 引入

```go
import _ "net/http/pprof"

r.HandleFunc("/debug/pprof/", pprof.Index)
r.HandleFunc("/debug/pprof/cmdline", pprof.Cmdline)
r.HandleFunc("/debug/pprof/profile", pprof.Profile)
r.HandleFunc("/debug/pprof/symbol", pprof.Symbol)
r.HandleFunc("/debug/pprof/trace", pprof.Trace)
```



### 图像

安装 graphviz

#### 方案1

```
go tool pprof qos0:9333/debug/pprof/profile
```

注意上述方案会生成的相关的性能文件，可供`方案2`使用

上述命令执行后，会进入命令行交互模式：

* top 10 - 输出前10个占用最高的 

* web - 生成图像（需要graphviz支持，所以需要安装）

#### 方案2

```
go tool pprof -http localhost:4001  C:\\Users\\EDZ\\pprof\\pprof.qosdbadaptor.samples.cpu.002.pb.gz
```

> 上述文件可由 方案1 获取

```
- VIEW 看各种视图
  - Top：主要看占用内存当排名信息
  - Graph：主要看调用关系图，并且通过框框的粗细、颜色当深浅、线的实虚、以及数字信息包括执行时间和占比
  - Flame Graph：火焰图，要看宽度和深度，heap 中宽度表明内存占用大小，
  - Peek
  - Source
  - Disassemble
- SAMPLE：采样信息，包括申请对象、申请空间、占用对象、占用空间的信息
- REFINE：可以精细化视图中的信息
  - Focus：聚焦在选中元素的上下游元素
  - Ignore：忽略选中当元素，包含其后继元素
  - Hide：隐藏选中当元素，但不会隐藏其后继元素
  - Show：只显示选中的元素，不包含后继元素
  - Show from：从选中当某一个元素开始，只列出其后继元素
- CONFIG：能将当前已精细化的页面保存起来
```

