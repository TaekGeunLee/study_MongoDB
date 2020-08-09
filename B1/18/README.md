<p>
시간이 계속 지나면서 MongoDB가 버전 업이 되면서
지원하는 파이프라인 스테이지는 늘어나지고 있다.<br />
여기서는 주로 사용하는 스테이지 이외에 응용스러운 스테이지들을
다룰 것이다.    
</p>

<p>
스테이지는 업데이트 될 때 마다 계속 늘어나고 있으므로
전부 암기하는 식으로 공부하지는 말자.<br />
필요한 상황이 생길때 마다 API 공식 문서를 참고해서 공부하도록 하자.    
</p>

<p>실습을 위해 다음 컬렉션을 사용한다.</p>


```javascript
> var rate = db.rating
> rate.find()

{ "_id" : 1, "rating" : 1, "user_id" : 2 }
{ "_id" : 2, "rating" : 2, "user_id" : 3 }
{ "_id" : 3, "rating" : 3, "user_id" : 4 }
{ "_id" : 4, "rating" : 3, "user_id" : 1 }
{ "_id" : 5, "rating" : 4, "user_id" : 5 }
{ "_id" : 6, "rating" : 4, "user_id" : 8 }
{ "_id" : 7, "rating" : 5, "user_id" : 9 }
{ "_id" : 8, "rating" : 5, "user_id" : 10 }
```

### $bucket
<p>
주요 스테이지 $group과 마찬가지로 도큐먼트들을 그룹핑 한다.<br />
특정 범위를 지정해서 그룹핑 한다는 것이 차이점.<br />정수 보다는 실수 값을
그룹핑 하는데 적절하다.   
</p>

```javascript
> rate.aggregate([
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

{ "_id" : 1, "sum" : 1, "user_ids" : [ 2 ] }
{ "_id" : 2, "sum" : 1, "user_ids" : [ 3 ] }
{ "_id" : "outing", "sum" : 6, "user_ids" : [ 4, 1, 5, 8, 9, 10 ] }
```
<p>$bucket 스테이지의 값으로 객체를 받고 있다.</p>

<p>
boundaries 필드로 그룹핑할 범위를 정한다.<br />
값으로 배열을 가지는데, 저것은 1 이상 2 미만, 2 이상 3 미만 씩 그룹핑 하겠다는 뜻이다.<br />
범위에 포함되지 않은 나머지 도큐먼트들은 default 필드 값의 이름의 컬렉션에 담을 수 있다.<br />
이는 선택 값이다.    
</p>

### $facet
<p>
각각 집계 연산을 완료한 도큐먼트들을 개별 필드를 생성하여 담는다.<br />
예를 들어 하나의 필드에는 $group, 하나의 필드에는 $project를 담아서 출력하는 식으로
운용하는 것 이다.   
</p>

```javascript
> rate.aggregate([
    { 
        $facet: {
            "categorizedByRating": [{$group: {_id: "$rating", count: {$sum: 1}}}], 
            "categorizedRyRating(auto)": [{$bucketAuto: {groupBy: "$_id", buckets: 5}}]
        }
    }  
])

{ 
    "categorizedByRating" : [ 
        { "_id" : 1, "count" : 1 }, 
        { "_id" : 2, "count" : 1 }, 
        { "_id" : 5, "count" : 2 }, 
        { "_id" : 3, "count" : 2 }, 
        { "_id" : 4, "count" : 2 } 
    ], 
    "categorizedRyRating(auto)" : [ 
        { "_id" : { "min" : 1, "max" : 3 }, "count" : 2 }, 
        { "_id" : { "min" : 3, "max" : 5 }, "count" : 2 }, 
        { "_id" : { "min" : 5, "max" : 7 }, "count" : 2 }, 
        { "_id" : { "min" : 7, "max" : 8 }, "count" : 2 } 
    ] 
}
```
<p>
categorizedByRating 필드에는 $group 스테이지의 연산 결과를,<br /> 
categorizedRyRating(auto) 필드에는 $bucketAuto 스테이지의 결과를 담아서
출력하고 있는 것을 관찰할 수 있다.    
</p>

### $sample
<p>지정한 수 만큼 해당 컬렉션에서 랜덤으로 도큐먼트들을 추출한다.</p>

```javascript
> rate.aggregate([
    { $sample: { size: 4} }
])

{ "_id" : 1, "rating" : 1, "user_id" : 2 }
{ "_id" : 3, "rating" : 3, "user_id" : 4 }
{ "_id" : 2, "rating" : 2, "user_id" : 3 }
{ "_id" : 6, "rating" : 4, "user_id" : 8 }
```

### $lookup
<p>
SQL문의 JOIN문과 비슷한 역할의 스테이지. 합치려는 서로 다른 컬렉션들 중<br />
도큐먼트 내에 같은 값을 지닌 필드들이 각각 지니고 있을 때,<br />그 필드들을 매개체로
컬렉션을 합친다.   
</p>

<p>
<a href="https://github.com/TaekGeunLee/study_MongoDB/tree/master/A2/referList">이전에 쿼리를 공부할 때 쓰던 컬렉션</a>들을 예제로 삼아서 살펴보도록 하자.<br />car_accident 데이터베이스 내의 두 개의 컬렉션, area, by_month 이들을 합쳐보자.<br />
상황을 그림으로 표현해보면 다음과 같다.   
</p>

<img src="https://github.com/TaekGeunLee/study_MongoDB/blob/master/readmeImg/B1_18-1.png" alt="B1_18-1" />

<p>
MySQL이나 MS access 등으로 RDBMS를 공부해본 경험이 있다면 이해할 수 있을 것이다.<br />
이제 코드를 작성해보면 다음과 같다.   
</p>

```javascript
> use car_accident
> var area = db.area
> area.aggregate([
    { $lookup: {
        from: "by_month", 
        localField: "_id", 
        foreignField: "area_id", 
        as: "area_data"
    } }, 
    { $limit: 1 }
])

{ 
    "_id" : ObjectId("5c88f9f70da47a850775277c"), 
    "city_or_province" : "서울", 
    "county" : "은평구", 
    "population" : 491476, 
    "area_data" : [ { "_id" : ObjectId("5c8909190da47a8507756079"), "city_or_province" : "서울", "county" : "은평구", "area_id" : ObjectId("5c88f9f70da47a850775277c"), "month_data" : [ { "month" : "01월", "accident_count" : 76, "death_toll" : 1, "casualties" : 102, "heavy_injury" : 33, "light_injury" : 66, "injury_report" : 3 }, { "month" : "02월", "accident_count" : 83, "death_toll" : 2, "casualties" : 118, "heavy_injury" : 36, "light_injury" : 70, "injury_report" : 12 }, { "month" : "03월", "accident_count" : 88, "death_toll" : 0, "casualties" : 126, "heavy_injury" : 29, "light_injury" : 70, "injury_report" : 27 }, { "month" : "04월", "accident_count" : 94, "death_toll" : 2, "casualties" : 118, "heavy_injury" : 35, "light_injury" : 79, "injury_report" : 4 }, { "month" : "05월", "accident_count" : 108, "death_toll" : 3, "casualties" : 141, "heavy_injury" : 40, "light_injury" : 86, "injury_report" : 15 }, { "month" : "06월", "accident_count" : 108, "death_toll" : 0, "casualties" : 131, "heavy_injury" : 36, "light_injury" : 84, "injury_report" : 11 }, { "month" : "07월", "accident_count" : 103, "death_toll" : 2, "casualties" : 138, "heavy_injury" : 36, "light_injury" : 90, "injury_report" : 12 }, { "month" : "08월", "accident_count" : 98, "death_toll" : 0, "casualties" : 150, "heavy_injury" : 36, "light_injury" : 99, "injury_report" : 15 }, { "month" : "09월", "accident_count" : 95, "death_toll" : 1, "casualties" : 133, "heavy_injury" : 36, "light_injury" : 87, "injury_report" : 10 }, { "month" : "10월", "accident_count" : 94, "death_toll" : 1, "casualties" : 126, "heavy_injury" : 42, "light_injury" : 69, "injury_report" : 15 }, { "month" : "11월", "accident_count" : 74, "death_toll" : 1, "casualties" : 106, "heavy_injury" : 37, "light_injury" : 59, "injury_report" : 10 }, { "month" : "12월", "accident_count" : 100, "death_toll" : 1, "casualties" : 129, "heavy_injury" : 37, "light_injury" : 80, "injury_report" : 12 } ] } ] 
}
```
<p>
area에 by_month를 join하기 위해<br />
area의 _id 필드와 by_month의 area_id 필드를 합쳐 area_data 필드로 합쳤다.<br />
합칠려는 각 대상 필드의 값이 서로 같으면 합치는 것이 가능하다.<br />
JOIN 당하는 대상과 입히는 대상을 서로 바꾸면 집계 연산 결과는 얼마든지 달라질 것이다.    
</p>

<p>$lookup 스테이지의 각 프로퍼티가 어떤 역할을 지니는지 관찰해보도록 해보자.</p>