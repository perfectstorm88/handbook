mongodb系统优化
====
# 数据库分析
```
show collections #显示collections
db.stats() #显示整个数据库的空间占用情况
```

## 可以直接写执行语句
```
var collectionNames = db.getCollectionNames(), stats = [];
collectionNames.forEach(function (n) { stats.push(db[n].stats()); });
stats = stats.sort(function(a, b) { return b['size'] - a['size']; });
for (var c in stats) { print(stats[c]['ns'] + ": "+stats[c]['count'] +"| "+ stats[c]['size'] + " (" + stats[c]['storageSize']/1024/1024 + ")"); }
```


## Database Profiler 必须admin用户才行
refer:https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/#profiling-levels

```
use admin
db.auth('') #鉴权
use jfjun_cw4 #再切换会要分析的数据库
db.getProfilingStatus() # 检查profiling level, return a docoment similar to the following:{ "was" : 1, "slowms" : 100 }

db.setProfilingLevel(2) # 修改level，记录所有的查询，分析查询的时间
# 然后可以在system.profile表中查询对应接口
db.getCollection('system.profile').find({},{ns:1,millis:1,ts:1})
db.getCollection('system.profile').find({"ns" : "jfjun_cw4.usbshells"},{ns:1,millis:1,ts:1})
# 如果调试完毕，不要忘了，再把level恢复原位
db.setProfilingLevel(1)
```

### 统计调用次数频率
参考：https://docs.mongodb.com/manual/reference/command/aggregate/
```
db.getCollection('system.profile').aggregate( [
   { $project: { ns: 1 } },
   { $unwind: "$ns" },
   { $group: { _id: "$ns", count: { $sum : 1 } } },
    { $sort: { count: -1 } }
] )
```

# 参考
[mongodb 跟踪SQL语句及慢查询收集](https://blog.csdn.net/shellching/article/details/11536889)
[Listing MongoDB collections by size](http://joey.aghion.com/listing-mongodb-collections-by-size/)