持久存储
====

riot 引擎支持将搜索数据存入硬盘，并在当机重启动时从硬盘恢复数据。使用持久存储只需设置EngineInitOptions中的三个选项：

```go
type EngineInitOptions struct {
  // 略过其他选项

  // 是否使用持久数据库，以及数据库文件保存的目录和裂分数目
  UseStorage bool
  StorageFolder string
  StorageShards int
  StorageEngine string
}
```

当UseStorage为true时使用持久存储：

1. 在引擎启动时（engine.Init函数），引擎从StorageFolder指定的目录中读取
文档索引数据，重新计算索引表并给排序器注入排序数据。如果分词器的代码
或者词典有变化，这些变化会体现在启动后的引擎索引表中。
2. 在调用engine.IndexDocument时，引擎将索引数据写入到StorageFolder指定
的目录中。
3. StorageShards定义了数据库裂分数目，默认为8。为了得到最好的性能，请调整这个参数使得每个裂分文件小于100M。
4. 在调用engine.RemoveDocument删除一个文档后，该文档会从持久存储中剔除，下次启动
引擎时不会载入该文档。


### 必须注意事项

一、如果排序器使用[自定义评分字段](/docs/zh/custom_scoring_criteria.md)，那么该类型必须在gob中注册，比如在左边的例子中需要在调用engine.Init前加入：
```
gob.Register(MyScoringFields{})
```
否则程序会崩溃。

二、在引擎退出时请使用engine.Close()来关闭数据库，如果数据库未关闭，数据库文件会被锁定，
这会导致引擎重启失败。解锁的方法是，进入StorageFolder指定的目录，删除所有以"."开头的文件即可。

### 性能测试

[benchmark.go](/examples/benchmark.go)程序可用来测试持久存储的读写速度：

```go
go run benchmark.go --num_repeat_text 1 --use_persistent
```

不使用持久存储的结果：

```
加入的索引总数3711375
建立索引花费时间 3.159781129s
建立索引速度每秒添加 1.174567 百万个索引
搜索平均响应时间 0.051146982400000006 毫秒
搜索吞吐量每秒 78205.98229466612 次查询
```

使用持久存储：

```
建立索引花费时间 13.068458511s
建立索引速度每秒添加 0.283995 百万个索引
搜索平均响应时间 0.05819595780000001 毫秒
搜索吞吐量每秒 68733.29611219149 次查询
从持久存储加入的索引总数3711375
从持久存储建立索引花费时间 6.406528378999999
从持久存储建立索引速度每秒添加 0.579311 百万个索引
```

可以看出，和不使用持久存储相比：

- [持久存储](#%E6%8C%81%E4%B9%85%E5%AD%98%E5%82%A8)
        - [必须注意事项](#%E5%BF%85%E9%A1%BB%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)
        - [性能测试](#%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95)
