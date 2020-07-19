맵-리듀스(map-reduce)는 자바스크립트 엔진, 즉 쿼리 처리기 측에서 집계하는 방식으로, 
그리 빠르지도 않으면서 느리지도 않은 처리 속도를 가졌다.

핵심은 대상 도큐먼트 들을 묶고(mapping), 묶은 도큐먼트들을 처리(Reduce)하는 것이다.
작업 과정을 도식화하면 다음과 같다.

<img src="https://github.com/TaekGeunLee/study_MongoDB/blob/master/readmeImg/B1_16-3.png" alt="B1_16-3" />

어디까지나 이들은 필수 과정에 속한다.
추가로 선택적으로 거치는 과정도 있지만 천천히 기록해보는 것으로 한다.

### 맵-리듀스 사용해보기 : 필수 단계
맵-리듀스 방식을 쓸 시 
반드시 거쳐야 하는 최소 단계만 체험해보자.
선택 옵션 단계는 후의 구절에서 기록할 것이다.

실습을 위해 다음 컬렉션을 추가한다.

```javascript
var rate = db.rating
rate.insertMany([
    { _id: 1, rating: 1, user_id: 2 },
    { _id: 2, rating: 2, user_id: 3 },
    { _id: 3, rating: 3, user_id: 4 },
    { _id: 4, rating: 3, user_id: 1 },
    { _id: 5, rating: 4, user_id: 5 },
    { _id: 6, rating: 4, user_id: 8 },
    { _id: 7, rating: 5, user_id: 9 },
    { _id: 8, rating: 5, user_id: 10 },
    { _id: 9, rating: 5, user_id: 11 },
    { _id: 10, rating: 5, user_id: 12 }
])
```

맵-리듀스 방식을 쓰기 위해
mapReduce()라는 함수를 쓰게 된다.
이 때 3가지 인자로 mapper, reducer, outer를 받게 된다.
각 인자에 대한 값들을 작성한 뒤에 대입해보자.
먼저 mapper 함수이다.

```javascript
var mapper = function() {
    emit(this.rating, this.user_id)
}
```
mapper 함수 내부에 emit() 함수를 씀 으로써
다음 단계인 reduer로 mapping한 도큐먼트 값들을 넘긴다.
emit()의 첫 번째 인자로는 어떤 필드를 기준으로 묶을지, 
두 번째 인자로는 다음 단계로 넘길 값들을 작성한다.

그림으로 표현하면
다음과 같이 mapping 되는 것이다.

<img src="https://github.com/TaekGeunLee/study_MongoDB/blob/master/readmeImg/B1_16-1.png" alt="B1_16-1" />

이제 reducer 함수를 작성해보자.

```javascript
var reducer = function(key, values) {
    return values.length
}
```
이번엔 함수 리턴 값을 다음 단계인 outer로 넘겨준다.
함수 인자인 key는 mapping한 기준값을, values에는 배열의 형태로 각각의 값들을 담고 있다.
values의 인덱스 길이(length)를 리턴하고 있음을 관찰하고 있는데,
이 때 outer 함수로 넘어가는 값의 형태를 그림으로 표현하면 다음과 같다.

<img src="https://github.com/TaekGeunLee/study_MongoDB/blob/master/readmeImg/B1_16-2.png" alt="B1_16-2" />

마지막으로 outer에 해당되는 값을 작성해보자.
컬렉션으로 저장할지, 콘솔에 출력할 지에 따라 형식이 달라지는데,
일단은 콘솔에 출력만 해보는 걸로 하겠다.

```javascript
{ out : {inline : 1} }
```
이제 mapReduce() 함수를 작성하고 실행시켜보면 
다음과 같이 된다.

```javascript
> rate.mapReduce(mapper, reducer, { out : {inline : 1} })
{
        "results" : [
                {
                        "_id" : 1,
                        "value" : 2
                },
                {
                        "_id" : 2,
                        "value" : 3
                },
                {
                        "_id" : 3,
                        "value" : 2
                },
                {
                        "_id" : 4,
                        "value" : 2
                },
                {
                        "_id" : 5,
                        "value" : 4
                }
        ],
        "timeMillis" : 55,
        "counts" : {
                "input" : 10,
                "emit" : 10,
                "reduce" : 3,
                "output" : 5
        },
        "ok" : 1
}
```
이렇게 출력되면 성공이다.
result 필드 값을 유심히 살펴보면 _id : 1, 2에 해당되는 값이
의도한대로 출력되지 않았다.(의도한 값: 1)

### 맵-리듀스 작동 방식

이전 구절에서 왜 엉뚱한 값이 나왔는지 알기 위해서는
맵-리듀스의 작동 방식에서 생길 수 있는 상황을 알 필요가 있다.
해당 교재에서는 2가지의 상황을 소개하고 있다.

#### 단일 원소가 담긴 맵핑 값에 대한 처리

먼저 첫 번째 상황이다.
mapping한 값 내에 하나의 도큐먼트만 존재할 경우, 
MongoDB에서는 해당 그룹에 대해서는 연산 처리를 하지 않는다.
이전 구절에서 실습했던 컬렉션에 대한 집계 처리를 그림으로 나타내보면
다음과 같다.

<img src="https://github.com/TaekGeunLee/study_MongoDB/blob/master/readmeImg/B1_16-4.png" alt="B1_16-4" />

이걸 통해 알 수 있는 것은 
mapping 이후에 넘겨진 값과 reducing 이후에 리턴되는 값의 데이터 형식이
서로 달라질 수 있음을 의미한다.
따라서 되도록이면 집계 처리 과정에서 각 단계마다 리턴되는 데이터 형식이
달라지는 경우는 피해야 한다.

```javascript
var mapper = function() {
    emit(this.rating, 1)
}
```
이런 상황을 피하기 위해 mapping 그룹 내 값으로 1를 보내도록 했다.
reducer의 목적이 counting 이므로, 단일 그룹에 대한 처리를 안해도
원하는 목적을 이룰 수 있다.

#### reducer 단계 : 멱등원적(Idempotent) 성질

만약 mapping 해야할 도큐먼트의 수가 너무 많다면,
MongoDB는 나누어서 mapping 처리를 한다.

각 나누어진 mapping 그룹들은 개별로 reduce 처리를 수행하다
후반부에 하나로 값을 합친다.
그림으로 나타내면 다음과 같다.

<img src="https://github.com/TaekGeunLee/study_MongoDB/blob/master/readmeImg/B1_16-5.png" alt="B1_16-5" />

원래는 value 값으로 4가 출력되야 하는데 3이 출력되었다.
reduce한 각 값들을 하나로 합치는 과정에서 문제가 생긴 것이다.

value 필드 값이 최종 value 값의 배열 원소로 들어가버려
문제가 생긴 것이다. 

```javascript
reducer(5, [2, 1, 1]) // 문제가 생긴 값
```
문제를 해결하기 위해 reducer 함수를 다음과 같이 변경한다.

```javascript
var reducer = function(key, values) {
    var count = 0;
    values.forEach(function(value) {
        count += value;
    })
    return count;
}
```

이런 식으로 함수 내에서 연산 처리의 결과 값을 완료한 뒤
다른 변수에 담아 리턴하는 식으로 reducer 함수를 설계하였다.

이와 같이 데이터를 몇 번이든 쪼개어서 함수를 호출하더라도 같은 결과가 나오게 하는 특성을
멱등원적 특성이라고 한다.
reducer 함수를 설계할 때에는 해당 특성을 지키도록 한다.

### 맵-리듀스 사용해보기 : 선택 단계

이전 구절에서 필수로 거쳐야 하는 처리 단계에 대해 다루었었다.
이번에는 선택적으로 선택할 수 있는 옵션 단계에 대해 기록해보자.

크게 다음과 같이 분류할 수 있다.

<ul>
   <li>사전 작업</li>    
   <li>마무리 작업</li>
</ul>

#### 선택 단계 : 사전 작업

mapping 이전, 즉 자바스크립트 엔진이 집계 연산을 하기 이전에
기본 틀을 깔아 놓는 것이다. (맵-리듀스는 엔진 내에서 처리되니까..)
3가지 단계가 있다.

<ul>
    <li>
    <b>guery</b>
    <p>query를 걸어 원하는 조건에 맞는 도큐먼트만 걸러낸다.<br />데이터 조회할 때 쓰는 그 query이다.</p>
    </li>
    <li>
    <b>sort</b>
    <p>재정렬 한다. 1은 오름차순, -1은 내림차순.</p>
    </li>
    <li>
    <b>limit</b>
    <p>설정한 값의 수의 도큐먼트만 넘긴다.<br />예를 들어 10으로 지정했다면 10개의 도큐먼트만 넘기는 것이다.</p>
    </li>
</ul>

선택 사항이기 때문에 저 3단계를 전부 쓸 필요는 없다.
위에서 아래 순으로 단계가 진행된다.

연산할 필요가 없는 도큐먼트들을 걸러내기 때문에, 만약 처리할 도큐먼트 중 잉여 데이터가 많다면
JS 엔진에 부담을 덜 수 있다.

#### 선택 단계 : 마무리 작업

reduce 처리를 완료한 이후에 out 단계로 넘어가기 전에 
마무리 연산을 진행하는 단계이다. Finalize 단계라고도 불린다.

함수를 작성할 때 reducer 함수와 비슷하다.
다만 reducer가 여러 개의 값을 연산할 수 있는 것에 비해
Finalizer는 단 한번만 연산할 수 있는 것이 차이이다.

#### 직접 사용해보기

직접 선택 단계 까지 거쳐서 집계 연산을 집행해보자.

Rating 컬렉션에서 _id 값이 6 이하 이면서 가장 큰 4개의 도큐먼트를 rating 별로 분류한 다음, 
각 도큐먼트의 개수(count), user_id 값들의 합(user_id_sum)과 평균(user_id_avg)을 구해보자.

```javascript
var mapper = function() {
    emit(this.rating, { count: 1, user_id_sum: this.user_id })
}

var reducer = function(key, values) {
    var counter = 0
    var sum = 0
                                     
    values.forEach(function(x) {
        counter += x.count
        sum += x.user_id_sum
    })
    return {count: counter, user_id_sum: sum}
}

var finalizer = function(key, value) {
    value.user_id_avg = value.user_id_sum / value.count
    return value
}

rate.mapReduce(mapper, reducer, {
    out: "mapReduceRating", 
    query: {_id: {$lte: 6}}, 
    sort: {_id: -1}, 
    limit: 4, 
    finalize: finalizer
});
```
코드를 작성하면 다음과 같이 된다.
각 함수에 대한 설명은 하지 않았지만
JS 스킬의 센스가 있다면 충분히 이해할 수 있다.
결과로 "mapReduceRating" 이라는 이름의 컬렉션이 db내에 담겨진다.
생성된 컬렉션을 조회해보면 다음과 같은 결과를 출력한다.

참고로 이전 구절에서 만들었던 컬렉션과는 다르다.

```javascript
> db.mapReduceRating.find()
{ "_id" : 3, "value" : { "count" : 2, "user_id_sum" : 5, "user_id_avg" : 2.5 } }
{ "_id" : 4, "value" : { "count" : 2, "user_id_sum" : 13, "user_id_avg" : 6.5 } }
```

집계 연산 결과를 콘솔에도 한번 출력해보자.






