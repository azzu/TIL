# AWS Cloudwatch OpenSearch 연결시 로그내용에 따라 인덱스를 구분하는 방법
## 목표
AWS Cloudwath에 들어오는 로그의 특정 케이스(ERROR?)를 OpenSearch로 보내고자 할때   
로그그룹별로 1개의 등록만 가능하다.   
그러나, 로그의 내용에 따라 구분하여 OpenSearch로 보내어 집계하려면   
별개의 index를 생성하여야 한다.

## 환경
AWS Cloudwatch -> 로그 그룹에서 AWS OpenSearch Service 구독필터가 등록되어 있는 상태

## 변경내용
AWS OpenSearch Service가 등록되어있다면, AWS Lambda에 함수 등록되어 있을 것이다.   
```LogsToElasticsearch_{AWS_OpenSearchService_도메인_이름}```으로 등록됨.   
해당 함수의 코드에서 ```function transform()```을 찾아, ```indexName``` 변수의 값을   
생성해 주는 부분을 아래와 같이 변경한다.   
```javascript
var indexName = [
    'cwl-' + logEvent.message.split(' ')[0].toLowerCase() + '-' + timestamp.getUTCFullYear(),              // year
    ('0' + (timestamp.getUTCMonth() + 1)).slice(-2),  // month
    ('0' + timestamp.getUTCDate()).slice(-2)          // day
].join('.');
```
위의 경우 ```cwl-abc-YYYY.MM.DD```의 형태로 index 이름을 생성하여 저장하게 된다.   
```logEvent```는 Cloudwatch에서 전달해주는 data이다.