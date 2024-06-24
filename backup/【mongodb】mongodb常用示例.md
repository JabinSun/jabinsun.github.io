```sql
-- test集合创建索引
db.test.createIndex( { "eId_idx": 1 }, { unique: true } );

-- test集合查找数据
db.test.find({eId: 'xxx'});

-- 查询mongodb缓存限制和当前使用内存大小
db.serverStatus().wiredTiger.cache["maximum bytes configured"];
db.serverStatus().wiredTiger.cache["bytes currently in the cache"];
```