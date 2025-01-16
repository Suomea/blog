索引的基本操作
查询索引
```
SHOW INDEX FROM table_name;
```

添加索引
```
ALTER TABLE table_name ADD INDEX index_name (column1, column2);
```

删除索引
```
ALTER TABLE table_name DROP INDEX index_name;
```

InnoDB 存储引擎支持以下几种常见的索引：
- B+ 树索引。
- 全文索引。
- 哈希索引。

什么是哈希索引，优劣。

Explain 解释。

调优的流程。

如果索引命中了，查询仍然很慢。

