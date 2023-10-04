# 개요

로그 수집 용도로 ELK를 사용하고 있었는데, 원래 보안 용도로 클라이언트에서 서버를 거쳐 로그를 수집하고 있었다.

로그 양이 많아지니, 서버의 부하가 몰려 성능 이슈가 발생할 수도 있었다.

방지를 위해, 클라에서 ELK로 바로 쏘도록 구조를 변경할 필요성을 느꼈고, VIP와 L4을 추가하도록 하였다.

원래 Elasticsearch는 자체 로드밸런싱 기능이 있어서, 1개의 머신으로 보내도 로드밸런싱을 하지만,

트래픽 양이 많을 때도, 제대로 작동할 지 몰라, 앞단에 로드밸런스를 놓기로 하였다.

# 문제

사내에서 ELK 구조는 머신 6대를 클러스터로 구성하여 사용하고 있다. 또한, 사용하는 L4 방식이 proxy 방식이다.

[##_Image|kage@ugqol/btsv91mxr4J/oyAgagYpVZt2of9FTrDYIk/img.png|CDM|1.3|{"originWidth":209,"originHeight":243,"style":"alignCenter"}_##]

아래는 `elasticsearch.yml`의 일부 내용이다.

```
bootstrap.memory_lock: true
network.host: 0.0.0.0
http.port: 9200
transport.port: 9300
```

위 내용처럼 세팅하니, 클러스터에 있는 머신들끼리 통신하는데 문제가 생겼다.

예를 들어, 로드밸런스 주소가 1.2.3.4 이고, A 머신이 1.1.1.0 B 머신이 2.2.2.0이라고 가정했을 때,

A 머신에서 B 머신으로 통신할 때, 로드밸런스를 거쳐서 가서 1.2.3.4의 로드밸런스 주소로 B머신의 source IP로 들어가,

통신 실패하여, 클러스터 형성을 할 수 없는 이슈가 있었다.

Elasticsearch에서는 위와 같은 문제를 방지하고자, 여러 옵션들을 이용할 수 있다.

옵션에 대한 정보는 공식 홈페이지에서 확인할 수 있다.

[https://www.elastic.co/guide/en/elasticsearch/reference/7.10/modules-network.html#common-network-settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/modules-network.html#common-network-settings)

 [Network settings | Elasticsearch Guide \[7.10\] | Elastic

Be careful with the network configuration! Never expose an unprotected node to the public internet.

www.elastic.co](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/modules-network.html#common-network-settings)

다시 돌아가, 위 이슈를 해결하기 위해서는 로드밸런스 주소가 아닌, 머신 자신의 주소로 통신을 해야한다.

마스터 노드는 `transport.publish_host`에 자기 자신의 주소를 넣으면 되고,

데이터 노드는 `transport.host`에 자기 자신의 주소를 넣으면 된다.