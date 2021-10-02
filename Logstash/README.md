# Logstash

## logstash.conf

### input
### filter
field를 추가할 수 있다.

#### filter pattern 가져오기
https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/legacy

### output

## 데이터 저장
```sh
$ head -n 2 ./weblog-sample.log | nc localhost 5044
```
`logstash.conf` 에 설정된 input port로 데이터를 보내면 output에 설정된 형식에 따라 나온다.

elasticsearch에서 확인하면 설정한 필드가 반영됨을 알 수 있다.
```sh
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "logstash",
        "_type" : "_doc",
        "_id" : "-VpUQXwBfVGz-kWbnvtX",
        "_score" : 1.0,
        "_source" : {
          "geoip" : {
            "timezone" : "Asia/Seoul",
            "latitude" : 37.5112,
            "ip" : "14.49.42.25",
            "country_name" : "South Korea",
            "country_code2" : "KR",
            "continent_code" : "AS",
            "country_code3" : "KR",
            "location" : {
              "lon" : 126.9741,
              "lat" : 37.5112
            },
            "longitude" : 126.9741
          },
          "@timestamp" : "2020-08-12T01:24:44.000Z",
          "port" : 56566,
          "clientip" : "14.49.42.25",
          "useragent" : {
            "patch" : "b1",
            "os" : "Windows",
            "major" : "3",
            "minor" : "6",
            "build" : "",
            "name" : "Firefox Beta",
            "os_name" : "Windows",
            "device" : "Other"
          },
          "message" : "14.49.42.25 - - [12/Aug/2020:01:24:44 +0000] \"GET /articles/ppp-over-ssh/ HTTP/1.1\" 200 18586 \"-\" \"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5\""
        }
      },
      {
        "_index" : "logstash",
        "_type" : "_doc",
        "_id" : "-lpWQXwBfVGz-kWbUPu2",
        "_score" : 1.0,
        "_source" : {
          "geoip" : {
            "timezone" : "Asia/Seoul",
            "latitude" : 37.5112,
            "ip" : "14.49.42.25",
            "country_name" : "South Korea",
            "country_code2" : "KR",
            "continent_code" : "AS",
            "country_code3" : "KR",
            "location" : {
              "lon" : 126.9741,
              "lat" : 37.5112
            },
            "longitude" : 126.9741
          },
          "@timestamp" : "2020-08-12T01:24:44.000Z",
          "port" : 56764,
          "clientip" : "14.49.42.25",
          "useragent" : {
            "patch" : "b1",
            "os" : "Windows",
            "major" : "3",
            "minor" : "6",
            "build" : "",
            "name" : "Firefox Beta",
            "os_name" : "Windows",
            "device" : "Other"
          },
          "message" : "14.49.42.25 - - [12/Aug/2020:01:24:44 +0000] \"GET /articles/ppp-over-ssh/ HTTP/1.1\" 200 18586 \"-\" \"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5\""
        }
      }
    ]
  }
}
```

## beat 활용

### filebeat

`filebeat.yml` 수정하여 로그 수집
- Elasticsearch Output으로 설정하면 message 필드만 저장됨
- Logstash Output으로 설정하여 직접 필드를 추가하여 저장할 수 있음