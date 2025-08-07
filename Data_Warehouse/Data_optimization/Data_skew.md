# 数据倾斜总结

## 一、数据倾斜定义
数据倾斜多发生在Reducer端，指表中数据分布不均衡，导致部分Worker计算时间远长于其他Worker的现象。实际生产中数据偏斜符合“二八定律”（如20%用户贡献80%数据），会导致作业进度停滞在99%。

## 二、数据倾斜判断（MaxCompute）
1. 在Fuxi Jobs中按运行时间（Latency）降序排列，选择耗时最长的Job Stage；
2. 在Fuxi Instance of Fuxi Stage中按Latency降序排列，选择运行时长远超平均的任务；
3. 查看任务输出日志（StdOut），定位作业执行图；
4. 根据执行图中的Key信息，定位导致倾斜的SQL代码片段。

## 三、数据倾斜主要原因及解决方法
### 1. Join倾斜（最常见）
**场景**：大表与小表/中表Join、Join热值长尾。
**解决方法**：
- **手动切分热值**：过滤热点值记录，先MapJoin再MergeJoin，最后合并结果；
- **设置SkewJoin参数**：`set odps.sql.skewjoin=true;`；
- **SkewJoin Hint**：使用`/*+ skewJoin(<table_name>[(<column1_name>[,<column2_name>,...])][((<value11>,<value12>)[,(<value21>,<value22>)...])]*/`；
- **倍数表取模相等Join**：利用倍数表优化。

**MapJoin注意事项**：
- 小表加载内存后总大小≤512MB（可通过`SET odps.sql.mapjoin.memory.max=2048;`调整，最大8192MB）；
- 支持小表为子查询，需引用别名；
- 支持不等值连接或OR条件，可通过`mapjoin on 1=1`实现笛卡尔积（注意数据膨胀）；
- 最多支持128张小表；
- 外连接限制：左外连接左表为大表，右外连接右表为大表，不支持全外连接。

### 2. GroupBy倾斜
**解决方法**：
- **参数调优**：`SET odps.sql.groupby.skewindata=true;`；
- **两阶段聚合**：在分区字段拼接随机数，分两阶段聚合。

### 3. Count(Distinct)倾斜
**解决方法**：
- 通用两阶段聚合：partition字段值拼接随机数；
- 类似两阶段聚合：先按`ds+shop_id`分组，再使用`count(distinct)`。

### 4. ROW_NUMBER(TopN)倾斜
**解决方法**：
- **SQL两阶段聚合**：增加随机列或拼接随机数作为分区参数；
- **UDAF两阶段聚合**：通过最小堆队列优先的UDAF调优。

### 5. 动态分区倾斜
动态分区指插入数据时不指定分区值，由Select子句列提供分区值（运行前未知分区）。
**解决方法**：
- **参数配置优化**：调整相关参数；
- **裁剪优化**：查找记录数多的分区，裁剪后单独插入。



