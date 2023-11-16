
- docker-compose -f elasticsearch2.yml  up -d


http://localhost:9200

http://localhost:9200/_cat/nodes?v&pretty
http://localhost:9200/_cat/indices?v&pretty

http://localhost:5601/app/integrations/browse


文件日志挂载路径：./docker_data/es02/logs



### kibana

http://localhost:5601/app/dev_tools#/console

## 创建索引模板，不同版本的elasticsearch语法差异较大
curl -XPUT "http://es01:9200/_template/cx_customer_address_template" -H 'Content-Type: application/json' -d'{"order": 1,"template": "cx_customer_address_*","settings": {   "index.max_ngram_diff":10,"index": {"routing": {"allocation": {"total_shards_per_node": "1"}},"analysis": {"analyzer": {"ngram_analyzer": {"tokenizer": "ngram_tokenizer"},"ngram_analyzer_addr": {"tokenizer": "ngram_tokenizer_addr"}},"tokenizer": {"ngram_tokenizer": {"type": "ngram","min_gram": "1","max_gram": "11"},"ngram_tokenizer_addr": {"type": "ngram","min_gram": "2","max_gram": "12"}}},"number_of_shards": "5","number_of_replicas": "1"}},"mappings": {"properties": {"address": {"analyzer": "ngram_analyzer_addr","type": "text"},"phone": {"analyzer": "ngram_analyzer","type": "text"},"companyname": {"analyzer": "ngram_analyzer","type": "text"},"telphone": {"analyzer": "ngram_analyzer","type": "text"},"contacts": {"analyzer": "ngram_analyzer","type": "text"}}},"aliases": {}}'

## 根据模板创建索引
curl -XPUT 'http://localhost:9200/cx_customer_address_1'

curl -XDELETE 'http://localhost:9200/cx_customer_address_1'

## 注意同一记录不能换多行，会有失败报错
curl -XPUT "http://es01:9200/cx_customer_address_1/_bulk" -H 'Content-Type: application/json' -d'
{"index":{"_id":123134}}
{	"addressid":"123134",	"address":"四川省西昌市礼州镇",	"companyname":"科技有限公司",	"contacts":"成家",	"phone":"15112361234",	"telphone":"15112361239"}
'