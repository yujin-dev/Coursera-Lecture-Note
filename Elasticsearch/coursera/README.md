## ELK 개요
- REST API( RPC pattern )
- DynomicDB
- distributed indexing in full-text search engine

Elasticsearch : distributed NoSQL databases  
Logstash : ingests streams of activity data  
Kibana : visualization  

### 기본 구조
```python
queryurl = "http://{elasticsearch address}"
body = json.dumps( {"query": {"match_all": {}}})
reponse = requests.post(queryurl, data =body)
```