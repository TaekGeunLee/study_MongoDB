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

<img src="<img src="https://github.com/TaekGeunLee/study_MongoDB/blob/master/readmeImg/B1_16-2.png" alt="B1_16-2" />

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
