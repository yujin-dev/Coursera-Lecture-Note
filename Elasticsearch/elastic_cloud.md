## Elastic Cloud Endpoint
endpoint는 아래와 같은 형식이다.

```
https://CLUSTER_ID.REGION.CLOUD_PLATFORM.DOMAIN:PORT
```

`GET /_cluster/health` 을 실행하여 cluster에 대한 정보를 알 수 있다.
위 형식에서 `CLUSTER_ID`는 `cluster_name` 값을 의미한다.


출처: 
https://www.elastic.co/guide/en/cloud-enterprise/2.3/ece-getting-started-connect.html
