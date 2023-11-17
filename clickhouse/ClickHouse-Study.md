### 熔断机制
1. 根据时间周期的累计熔断机制 (users文件内引用)
```xml
<quotas>
    <default> <!-- 自定义名称 -->
        <interval>
            <duration>3600</duration><!-- 时间周期 单位：秒 -->
            <queries>0</queries>
            <errors>0</errors>
            <result_rows>0</result_rows>
            <read_rows>0</read_rows>
            <execution_time>0</execution_time>
        </interval>
    </default>
    <limit_1>
        <interval>
            <duration>3600</duration>
            <queries>100</queries>
            <errors>100</errors>
            <result_rows>100</result_rows>
            <read_rows>2000</read_rows>
            <execution_time>3600</execution_time>
        </interval>
    </limit_1>
</quotas>
```
2. 根据单次查询的用量熔断(用户profile处定义)
```agsl
max_partitions_per_insert_block
max_rows_to_group_by
max_memory_usage
//等查看官网
```

### 数据备份
1. 小体量数据
```shell
#导出
clickhouse-client --query="SELECT * FROM test_backup" > /chbase/test_backup.tsv
#导入
cat /chbase/test_backup.tsv | clickhouse-client --query "INSERT INTO
test_backup FORMAT TSV"
```
2. 快照表备份
```sql
CREATE TABLE test_backup_0206 AS test_backup
INSERT INTO TABLE test_backup_0206 SELECT * FROM test_backup
# 考虑容灾将数据备份在不同的远程节点上
INSERT INTO TABLE test_backup_0206 SELECT * FROM remote('ch5.nauu.com:9000','default', 'test_backup', 'default')
```
3. 按分区备份
```sql
# 使用FREEZE备份
ALTER TABLE tb_name FREEZE PARTITION partition_expr
#保存到/shadow/N子目录下，是对原始文件的硬链接，恢复使用ATTACH语句装载
```
```sql
#使用FETCH备份
ALTER TABLE tb_name FETCH PARTITION partition_id FROM zk_path
ALTER TABLE test_fetch FETCH PARTITION 2019 FROM
    '/clickhouse/tables/01/test_fetch'
#保存目录（会下载数据到本地）    
data/default/test_fetch/detached 

#两者均不会备份元空间数据，需要用户单独备份
```