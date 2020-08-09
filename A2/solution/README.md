## 1번 : 정답
문제 : 강릉시(county) 교차로 내에서 일어난 총 사고 숫자를 출력해라. (by_road_type)

### my solution

```javascript
var road = db.by_road_type
        
road.find(
    { county: "강릉시" }, 
    { county: 1, "교차로내.accident_count": 1 }
)
```

## 4번 : 정답
문제 : 전국의 "차대사람" 사고 중에서 사망자수가 0회 이거나 중상자수가 0회인 지역을 출력해라.

### my solution

```javascript
var type = db.by_type
        
type.find(
    { type: "차대사람", $or: [{death_toll: 0}, {heavy_injury: 0}] }, 
    { city_or_province: 1, county: 1 }
)
```

## 8번 : 정답
문제 : 서울시에서 한달에 200회 이상 교통사고가 일어난 적이 있는 지역을 출력해라.

### my solution

```javascript
var month = db.by_month
        
month.find(
    { city_or_province: "서울", "month_data.accident_count": {$gte: 200} }, 
    { county: 1 }
)
```

## 9번 : 오답
문제 : 1월에 중상자 수가 0명이고, 2월에 사망자 수가 0명인 지역을 출력해라.

### my solution

```javascript
var month = db.by_month

month.find(
    { $and: [
            {"month_data.month": "01월", "month_data.heavy_injury": 0}, 
            {"month_data.month": "02월", "month_data.death_toll": 0}] }, 
    { city_or_province: 1, county: 1 }
)
```

### solution
<p>
해당 문제는 배열 내에서 하나 이상의 조건에 부합하는 도큐먼트를 조회할 것을 요구하고 있다.<br />
$elemMatch는 배열 내에서 여러 조건에 맞는 도큐먼트를 조회할 수 있는 연산자이다.<br />
내 풀이 처럼 써도 맞지 않아? 라고 생각했지만 배열을 대상으로는 통하지 않는 것 같다..($and를 쓰는 건 맞았다!)<br />
어쨌든 $elemMatch를 써서 문제를 풀어보도록 하자. 또는 $all를 쓰는 것도 고려해볼 수 있다.
까먹었으면 <a href="https://github.com/TaekGeunLee/study_MongoDB/tree/master/B1/13">여기</a>를 참고할 것.
</p>


