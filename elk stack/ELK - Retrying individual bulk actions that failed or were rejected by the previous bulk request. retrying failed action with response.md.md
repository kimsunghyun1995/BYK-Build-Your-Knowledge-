## 개요

6/11 09:17:33 이후부터 poker , classic, client 인덱스에 로그가 안남고 있는 걸 확인

logstash 로그 확인 결과, 아래와 같은 로그가 발생 중

```
[2023-06-12T16:00:13,910][INFO ][logstash.outputs.elasticsearch][main]{중략....} Retrying individual bulk actions that failed or were rejected by the previous bulk request.{중략....} retrying failed action with response code: 403 ({"type"=>"cluster_block_exception", "reason"=>"index [shrink-index-2023.06] blocked by: [FORBIDDEN/8/index write (api)];"})
```

로그 그대로 shrink-index에 데이터을 write하는걸 blocked 당하고 있다는 건데, 6월 달 로그가 벌써 shrink에 들어간 점이 이상하였다.

## 원인

확인해보니 classic, client의 ILM인 client-index에 WARM 상태에 들어가는 것이 10일로 설정되어 있었다.

[##_Image|kage@bDPCZM/btsjYth29PK/bkc5pUSaoWiDuO8rBPDVz0/img.png|CDM|1.3|{"originWidth":963,"originHeight":404,"style":"alignCenter"}_##]

WARM 상태 즉 **shirnk 된 index에는 data를 write 할 수 없었다.**

6월달 데이터만 문제가 된 이유는 아래와 같다.

기존에는 매일 로그를 남겼기 때문에 ILM에서 10일이 지나면 WARM 상태로 변하도록 설정하였다.

하지만, 최근 매일 남기던 로그를 한달로 남기기로 하였는데, 한달 단위로 저장되는 1개의 인덱스가 10일이 지나니 WARM 상태로 변화하여, Block 하고 있었다.

## 해결

ILM 설정 중 10일이 지나면 WARM 상태로 되는 설정을 disable 하였다.

여기서 문제는 이미 shirnk로 되어버린 index이다.

elastic에서는 index에 대한 모든 설정은 생성된 이후에서는 변경할 수 없는 규칙이 있다.

그렇기 때문에, 문제가 되는 index를 reindex하여, 다른 index로 데이터를 옮기고, delete하였다.

  
**결과적으로 6월 11일 이전 데이터는 \`backup\_index-2023.06\`에 있고,**

이후 데이터는 `index-2023.06`에 있다.