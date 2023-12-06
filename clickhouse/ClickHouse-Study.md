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

### 服务监控
1. 系统表
```sql
SELECT * FROM system.metrics LIMIT 5
SELECT event, value FROM system.events LIMIT 5
SELECT * FROM system.asynchronous_metrics LIMIT 5
```
2. 查询日志
```xml

<query_log>
    <database>system</database>
    <table>query_log</table>
    <!--            <table>query_thread_log</table>-->
    <!--            <table>part_log</table>-->
    <!--            <table>text_log</table>-->
    <!--            <table>metric_log</table>-->
    <partition_by>toYYYYMM(event_date)</partition_by>
    <!--            <!—刷新周期&ndash;&gt;-->
    <flush_interval_milliseconds>7500</flush_interval_milliseconds>
</query_log>
       
```
```sql
 SELECT type,concat(substr(query,1,20),'...')query,read_rows,
        query_duration_ms AS duration FROM system.query_log LIMIT 6
```

## 建表相关
```sql

CREATE TABLE IF NOT EXISTS hits_v1_1 ENGINE = Memory AS SELECT * FROM hits_v1

CREATE TABLE partition_v1 ( 
 ID String,
 URL String,
 EventTime Date
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventTime) 
ORDER BY ID

INSERT INTO partition_v1 VALUES 
('A000','www.nauu.com', '2019-05-01'),
('A001','www.brunce.com', '2019-06-02')

SELECT table,partition,path from system.parts WHERE table = 'partition_v1'
ALTER TABLE testcol_v1 ADD COLUMN OS String DEFAULT 'mac'
ALTER TABLE testcol_v1 COMMENT COLUMN ID '主键ID'
ALTER TABLE testcol_v1 DROP COLUMN URL
ALTER TABLE testcol_v1 MODIFY COLUMN IP IPv4  #修改字段类型
RENAME TABLE default.testcol_v1 TO db_test.testcol_v2  #移动数据库只能在单节点内移动
#复制分区
ALTER TABLE B REPLACE PARTITION partition_expr FROM A #表结构相同，分区键相同
# 重置分区列数据
ALTER TABLE partition_v2 CLEAR COLUMN URL in PARTITION 201908
#卸载装载分区
#数据并没有删除而是转移到当期表数据所在目录的detached目录
# ATTACH装载
ALTER TABLE partition_v2 DETACH PARTITION 201908
```

- 数据的删除与修改
```sql
ALTER TABLE partition_v2 DELETE WHERE ID = 'A003'
```