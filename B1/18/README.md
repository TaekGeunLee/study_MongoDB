시간이 계속 지나면서 MongoDB가 버전 업이 되면서
지원하는 파이프라인 스테이지는 늘어나지고 있다.
여기서는 주로 사용하는 스테이지 이외에 응용스러운 스테이지들을
다룰 것이다.

스테이지는 업데이트 될 때 마다 계속 늘어나고 있으므로
전부 암기하는 식으로 공부하지는 말자.
필요한 상황이 생길때 마다 API 공식 문서를 참고해서 공부하도록 하자.


```javascript
var rate = db.rating

{ "_id" : 1, "rating" : 1, "user_id" : 2 }
{ "_id" : 2, "rating" : 2, "user_id" : 3 }
{ "_id" : 3, "rating" : 3, "user_id" : 4 }
{ "_id" : 4, "rating" : 3, "user_id" : 1 }
{ "_id" : 5, "rating" : 4, "user_id" : 5 }
{ "_id" : 6, "rating" : 4, "user_id" : 8 }
{ "_id" : 7, "rating" : 5, "user_id" : 9 }
{ "_id" : 8, "rating" : 5, "user_id" : 10 }

rate.aggregate([
   { $bucket: {
       groupBy : "$rating", 
       boundaries : [1, 2, 3], 
       default : "outing", 
       output : {
           sum : {$sum: 1}, 
           user_ids : {$push: "$user_id"}
       }
    }}
])

rate.aggregate([
    { 
        $facet: {
            "categorizedByRating": [{$group: {_id: "$rating", count: {$sum: 1}}}], 
            "categorizedRyRating(auto)": [{$bucketAuto: {groupBy: "$_id", buckets: 5}}]
        }
    }  
])

rate.aggregate([
    { $sample: { size: 4} }
])

- 2 이상 3 미만
- 3 이상 5 미만

[1, 2, 3]

- 1 이상 2 미만 : 2
- 2 이상 3 미만 : 3

/*
- $bucket
: 주요 스테이지 $group과 마찬가지로 도큐먼트들을 그룹핑 한다.
  특정 범위를 지정해서 그룹핑 한다는 것이 차이점. 정수 보다는 실수 값을
  그룹핑 하는데 적절하다.
  
- $facet
: 각각 집계 연산을 완료한 도큐먼트들을 개별 필드를 생성하여 담는다.
  예를 들어 하나의 필드에는 $group, 하나의 필드에는 $project를 담아서 출력하는 식으로
  운용하는 것 이다.
  
- $sample
: 지정한 수 만큼 해당 컬렉션에서 랜덤으로 도큐먼트들을 추출한다.

- $lookup
: SQL문의 JOIN문과 비슷한 역할의 스테이지. 합치려는 서로 다른 컬렉션들 중 
  도큐먼트 내에 같은 값을 지닌 필드들이 각각 지니고 있을 때, 그 필드들을 매개체로
  컬렉션을 합친다.
*/
```
