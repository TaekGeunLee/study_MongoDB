<p>집계 처리 방식중 하나인 파이프라인 방식을 공부해보자.<br />
입력/출력 식으로 다음 과정에 작업물을 전달하는 식으로 처리가 된다.<br />
(하나의 과정을 스테이지 라고 명명하겠다.)</p>

<img src="https://github.com/TaekGeunLee/study_MongoDB/blob/master/readmeImg/B1_17-1.png" alt="B1_17-1" />

<p>이전 맵 리듀스에서는 JS로 집계 연산을 처리했지만,<br />
여기서는 쿼리마냥 연산자로 작업을 진행한다.<br />
연산자로 처리한다는 것은, MongoDB 내에서 집계 연산을 처리하는 것이기에
집계 처리 속도가 빠르다.<br />그러나 관련 연산자들도 엄청 많을 뿐 더러
타 방식들에 비해 제약이 있다.
</p>

### 기본 스테이지

<p>집계 파이프라인 방식에서 존재하는 모든 스테이지의 종류는 많다.<br />
그 중에 기본적으로 쓰이는 스테이지만 추려내면 다음과 같다.
</p>

<ul>
    <li>$project : 어떤 필드를 숨기고, 어떤 필드를 새로 만들지 정함</li>
    <li>$group : _id 값으로 지정한 내용이 같은 도큐먼트끼리 그룹화 됨</li>
    <li>$match : 도큐먼트를 필터링. find()와 비슷.</li>
    <li>$unwind : 입력 도큐먼트에서 배열 필드를 분해하여 각 요소에 대한 도큐먼트로 분리하여 출력.</li>
    <li>$out : 집계 처리의 결과를 컬렉션에 기록.</li>
</ul>

<p>하나 하나씩 살펴보자. 실습을 위해 다음 컬렉션을 추가한다.</p>

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

### $project

<p>어떻게 필드를 보이게 할지, 숨기게 할지에 대한 과정을 처리하는 스테이지이다.<br />
다음과 같은 구문을 작성해본다.</p>

```javascript
rate.aggregate([
    { $project: {_id: 0, rating: 1} }
])
```

<p>파이프라인 방식을 쓸려면 명령어 .aggregate()를 쓴다.<br />
인자로 배열 값을 받으며, 각 배열은 원소로 객체들을 지닌다.<br />
각 객체들은 스테이지를 의미하고, 각 스테이지 마다 필요한 연산자를 사용한다.<br />
위의 명령어를 실행하면 다음과 같은 결과가 출력된다.</p>

```javascript
{ "rating" : 1 }
{ "rating" : 2 }
{ "rating" : 3 }
{ "rating" : 3 }
{ "rating" : 4 }
{ "rating" : 4 }
{ "rating" : 5 }
{ "rating" : 5 }
```

<p>_id 필드가 감춰지고, rating 필드만 보이게 되었음을 관찰할 수 있다.<br />
user_id 필드에는 아무것도 입히지 않았는데 감춰진 것을 보면 프로젝션의 동작 원리와 비슷하다는 것을 알 수 있다.<br />
_id 필드를 제외하고, 모든 필드의 값은 반드시 하나로 통일되야 함을 유념하자.</p>

<p>$project로는 새로운 필드를 만들 수 있다. 다음과 같이 작성해본다.</p>


```javascript
> rate.aggregate([
    { $project: {_id: 0, rating: 1, hello: "hello world!"}}
])
{ "rating" : 1, "hello" : "hello world!" }
{ "rating" : 2, "hello" : "hello world!" }
{ "rating" : 3, "hello" : "hello world!" }
{ "rating" : 3, "hello" : "hello world!" }
{ "rating" : 4, "hello" : "hello world!" }
{ "rating" : 4, "hello" : "hello world!" }
{ "rating" : 5, "hello" : "hello world!" }
{ "rating" : 5, "hello" : "hello world!" }
```
<p>보시다싶히 모든 도큐먼트에 hello 필드가 생성된 채로 출력되었다.<br />
물론, 실제로 필드가 추가되었다는 의미는 전혀 아니다.</p>

### $group

<p>맵 리듀스에서 mapper 함수로 도큐먼트들을 묶는 것을 익힌 적이 있었다.<br />
$group은 바로 그룹화의 역할을 담당한다.<br />
그룹화는 집계 연산에서 핵심적으로 다루는 과정임을 유의하자.</p>

```javascript
> rate.aggregate([
    { $group: {_id: "$rating", count: {$sum: 1}} }
])
{ "_id" : 2, "count" : 1 }
{ "_id" : 3, "count" : 2 }
{ "_id" : 5, "count" : 2 }
{ "_id" : 1, "count" : 1 }
{ "_id" : 4, "count" : 2 }
```
<p>_id 필드의 값을 기준으로 그룹핑 한 것을 확인할 수 있다.<br />
명령어의 count 필드 값에 처음보는 연산자가 보인다.<br />
$sum은 필드 값으로 받은 숫자만큼 합산 하라는 의미로, <br />
$sum: 1은 그룹핑된 도큐먼트당 1를 덧셈. 이라고 해석할 수 있을 것이다.</p>

<p>이처럼 파이프라인 방식에서는 그룹핑을 하면서 필요한 연산을 진행할 수 있다.<br />
$group 스테이지에서 쓰는 연산자들을 추려내면 다음과 같다.
</p>

<ul>
    <li>$first: 그룹의 첫 번째 값을 반환.</li>
    <li>$last: 그룹의 마지막 값을 반환.</li>
    <li>$max: 그룹의 해당 필드의 최댓값을 반환.</li>
    <li>$min: 그룹의 해당 필드의 최솟값을 반환.</li>
    <li>$avg: 그룹의 해당 필드의 평균을 반환.</li>
    <li>$sum: 그룹의 해당 필드의 합산값을 반환.</li>
    <li>$push: 그룹에서 해당 필드의 모든 값을 배열에 넣어서 반환한다. 중복을 제거하지 않는다.</li>
    <li>$addToSet: $push와 역할이 비슷하다. 차이점이라면 반환하는 배열에 중복된 요소가 없다는 것 정도.</li>
</ul>

### $match

<p>find()와 같이 지정한 조건에 맞는 도큐먼트들을 조회한다.<br />
쿼리를 써서 조건을 거는 것도 완전 같다.</p>


```javascript
rate.aggregate([
    { $match: {rating: {$gte:4}} }, 
    { $group: {_id: "$rating", user_ids: {$push: "$user_id"}} }
])
{ "_id" : 5, "user_ids" : [ 9, 10 ] }
{ "_id" : 4, "user_ids" : [ 5, 8 ] }
```

<p>$group 스테이지의 $push 연산자를 쓴 뒤의 결과가 어떻게 출력되었는지 관찰하자.</p>

### $unwind

<p>도큐먼트 내에 필드 값으로 지니고 있는 배열의 요소들을 별개의 도큐먼트로 분리한다.<br />
$match 스테이지 단락에서 사용했던 예제에서 $unwind 스테이지를 추가해보겠다.</p>


```javascript
> rate.aggregate([
    { $match: {rating: {$gte:4}} }, 
    { $group: {_id: "$rating", user_ids: {$push: "$user_id"}} }, 
    { $unwind: "$user_ids" }
])
{ "_id" : 4, "user_ids" : 5 }
{ "_id" : 4, "user_ids" : 8 }
{ "_id" : 5, "user_ids" : 9 }
{ "_id" : 5, "user_ids" : 10 }
```
<p>$unwind 스테이지의 값으로 이전 과정에서 배열로 묶어서 담겨진 user_ids를 넣었다.<br />
그랬더니 배열의 각 원소들을 별개의 도큐먼트로 분리하는 것을 확인할 수 있다.<br />
집계 처리에서 사용하는 실제 데이터는 앞에 $를 붙임으로써 출력 결과로 띄워지는 필드와
구분짓는 것으로 보인다.</p>

<p>$unwind에는 또 옵션 값을 부여할 수 있다.<br />
여러 옵션 값이 있지만, 배열 인덱스 필드를 부여 해보자.</p>


```javascript
> rate.aggregate([
    { $match: {rating: {$gte:4}} }, 
    { $group: {_id: "$rating", user_ids: {$push: "$user_id"}} }, 
    { $unwind: {path: "$user_ids", includeArrayIndex: 'index' } }
])
{ "_id" : 5, "user_ids" : 9, "index" : NumberLong(0) }
{ "_id" : 5, "user_ids" : 10, "index" : NumberLong(1) }
{ "_id" : 4, "user_ids" : 5, "index" : NumberLong(0) }
{ "_id" : 4, "user_ids" : 8, "index" : NumberLong(1) }
```

<p>이전에 $unwind의 값으로 할당했던 것을 path 필드에 넣는 것을 관찰할 수 있다.<br />
그리고 includeArrayIndex의 값으로 새로이 만들 인덱스 필드의 이름을 기입한다.<br />
저런 식으로도 $unwind 스테이지를 작성할 수 있다. 정도로 이해하자.</p>

<p>인덱스 필드로 <a href="https://stackoverrun.com/ko/q/4677624">NumberLong</a> 데이터 타입으로 인덱스가 저장되었음을 확인했다.</p>

### $out

<p>쉽게 말해 마무리 작업으로 이해하면 된다.<br />
맵 리듀스에서는 outer 함수를 작성함과 동시에 결과를 출력하거나, 컬렉션으로 저장할 수 있었다.<br />
파이프라인의 $out은 후자에 속한다.</p>

```javascript
> rate.aggregate([
    { $match: {rating: {$gte:4}} }, 
    { $group: {_id: "$rating", user_ids: {$push: "$user_id"}} }, 
    { $out: "user_id_by_rating" }
])
```

<p>이후에 DB 내에 컬렉션이 추가됨을 확인할 수 있다.</p>

### 입력 도큐먼트 제어 스테이지

<p>
앞의 5개가 기본적으로 알아야할 스테이지라면<br />
이번 절에서 언급할 것은 연산 처리가 아닌, 도큐먼트들을 제어하는 스테이지이다.<br />
예를 들어 오름/내림 차순으로 정렬하거나 n개의 도큐먼트만 집계 연산에 사용하는 등..이 이에 해당된다.   
</p>

<p>예제를 살펴보자.</p>

```javascript
{ "_id" : 1, "rating" : 1, "user_id" : 2 }
{ "_id" : 2, "rating" : 2, "user_id" : 3 }
{ "_id" : 3, "rating" : 3, "user_id" : 4 }
{ "_id" : 4, "rating" : 3, "user_id" : 1 }
{ "_id" : 5, "rating" : 4, "user_id" : 5 }
{ "_id" : 6, "rating" : 4, "user_id" : 8 }
{ "_id" : 7, "rating" : 5, "user_id" : 9 }
{ "_id" : 8, "rating" : 5, "user_id" : 10 }

> rate.aggregate([
    { $sort: {user_id: -1} }, 
    { $limit: 5 }, 
    { $skip: 1 }
])

{ "_id" : 7, "rating" : 5, "user_id" : 9 }
{ "_id" : 6, "rating" : 4, "user_id" : 8 }
{ "_id" : 5, "rating" : 4, "user_id" : 5 }
{ "_id" : 3, "rating" : 3, "user_id" : 4 }
```
<p>
먼저 user_id 필드를 기준으로 내림차순으로 정렬($sort) 하고,<br /> 
5개의 도큐먼트만 추려낸 뒤($limit)에 하나의 도큐먼트를 건너뛰었다.($skip)<br />
이들도 파이프라인 연산자 이므로 위에서 아래 스테이지 순으로 처리된다.   
</p>

<p>
건너뛰었다는 것은 추려낸 도큐먼트 들 중 첫 번째 부터 추려내는 것이다.<br />
이전에 맵 리듀스에서 outer 함수를 작성할 때 옵션 값과 비슷하다.<br />
마지막으로 하나의 예제만 살펴보고 레퍼토리 작성을 마치도록 하겠다.
</p>

```javascript
> rate.aggregate(
    [ { $sort: {user_id: -1} } ], { allowDiskUse: true }
  )
```

<p>
$sort 스테이지를 사용할 때 100MB 이상의 데이터는 제어하지 못한다.<br />
이를 위해서는 allowDiskUse 옵션을 true로 설정해야 한다.    
</p>

