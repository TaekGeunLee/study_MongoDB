## 비교 연산자
<p>
비교 연산자들을 하나 하나 살펴보자.<br />
아래에 소개되는 연산자들은 전부 비교 연산자이다.   
</p>


### $eq와 $ne

<p>
$eq는 주어진 값과 일치하는 도큐먼트를 조회하는 연산자 이다. (equal)<br />
$ne는 그 반대로, 주어진 값을 제외한 모든 도큐먼트를 조회한다. (not equal)   
</p>

```javascript
> var avo = db.avocadoCollection
> avo.find(
    { name : {$eq : "basilGreen"} })
```

<p>
이것만 보면 별 볼일 없어 보이겠지만, 위의 명렁어는 아래의 명령어와 동일한 동작을 취한다.    
</p>

```javascript
> avo.find({name : "basilGreen"})
```

<p>
그렇다면 뭐하러 연산자를 쓰는가.. 싶겠지만 논리 연산자로 여러 개의 쿼리들을 묶을 때에는 연산자가 꼭 필요하다.    
</p>

### $in과 $nin

<p>이들은 배열에 관련된 연산자이다.</p>
<p>$in은 해당 배열 내에 속한 값을 포함하고 있는 도큐먼트들을 조회한다.<br />
$nin은 그 반대 개념이다.</p>
<p>예제를 살펴보면 다음과 같다.</p>

```javascript
> avo.find({ array : {$in : [0, 1]} })
{ "_id" : ObjectId("5ef46b42db1f8d6e902de697"), "name" : "oliveBasic", "array" : [ 0, 1, 2 ], "VIP" : false, "status" : true }
{ "_id" : ObjectId("5ef46b42db1f8d6e902de698"), "name" : "aliceblue", "array" : [ 0, 2, 4, 3 ], "VIP" : true, "status" : true }
{ "_id" : ObjectId("5ef46b77db1f8d6e902de699"), "name" : "Redgreen", "array" : [ 1, 3, 5, 100 ], "VIP" : false, "status" : true }
{ "_id" : ObjectId("5efc9e2a7cbd7dd824dce476"), "name" : "Basilgreen", "array" : [ 0, 1, 2, 3 ], "VIP" : true }
{ "_id" : ObjectId("5efc9e2a7cbd7dd824dce477"), "name" : "LeafGreen", "array" : [ "a", 0 ] }
```

<p>array 프로퍼티의 배열에 0, 1를 포함하고 있는 도큐먼트를 조회했다.<br />
중요한 것은, <b>0 또는 1을 포함하고 있는 것</b>들을 조회한다는 점이다.</p>


### 나머지 비교 연산자들

<p>이상($gt : greater then), 이하($lt : less then), 미만($lte : less then equal), 초과($gte : greater then equal)에 해당하는 연산자들이다.<br />쓰는 방법은 이미 이전 디렉토리에서 다뤄서 특별히 볼 것이 없지만, 얘네들은 서로 다른 데이터 타입들 까지 비교한다. JS 자체의 특성이긴 하다만, MongoDB 내에서만 적용되는 룰이 존재한다.</p>

<p>물론 지금으로써는 다른 데이터 타입 간의 비교를 마주칠 일은 없을 것이라고 판단된다.</p>

## 논리 연산자
<p>
논리 연산자들을 살펴보자.<br />
아래의 소개되는 연산자들은 전부 논리 연산자이다.    
</p>

### $not
<p>
$not을 제외한 모든 연산자($and, $or, $nor)들은 사용 형식이 독특하다.<br />
제일 먼저 $not을 살펴보자.    
</p>

<p>
find()의 인자로 가지는 쿼리에 정 반대인 도큐먼트들을 가져온다.<br />
프로그래밍에서의 not 연산자와 역할이 똑같다. 아래 예시를 보자.
</p>


```javascript
> var avo = db.avocadoCollection
> avo.find({ name : {$not : {$eq:'blue'}}} )
```
<p>
name 필드의 값이 'blue'가 아닌 값들을 조회한다.<br />
만약 가장 안쪽의 연산자로 비교 연산자를 썼을 경우 그 반대 결과로 출력되는 것을 알 수 있을 것이다.<br />
예를 들어 $lte 가 $gt의 결과로 출력되는 것 처럼 말이다.
</p>


### $and, $or, $nor
<p>
이들을 사용하는 방법을 알아보자. 기본적인 형식은 다음과 같다.    
</p>

```javascript
{ <논리 연산자> : [{실행할 쿼리 1}, {실행할 쿼리 2}] }
```

<p>
가장 바깥쪽 프로퍼티로 논리 연산자를 두고 있으며, 그 값으로 배열을 담고 있다.<br />
각 배열들은 쿼리에 해당되는 객체들을 담는다. 실전에 써보면 다음과 같다.
</p>

```javascript
{ $and : [{population : {$gte:300000, $lte:500000}}, {city_or_province : {$eq:"부산"}}] })
```

<p>
인구(population)이 300000 이상, 500000 이하(첫 번째 조건) 그리고(and) 도시(city_or_province)가 부산(두 번째 조건)인
도큐먼트 들을 조회하는 명령어이다.    
</p>

<p>
괄호를 많이 써서 가독성이 많이 떨어진다만, 
이전 레퍼토리를 열심히 작성했었다면, 요령을 이미 파악 했을 것이다.   
</p>

<p>
첫 번째 조건을 관찰했을 때, '어? 그럼 저기에도 $and를 써야 하지 않아?' 라고 생각을 했었을 것이다.    
</p>

```javascript
{population : {$gte:300000, $lte:500000}
```

<p>
저렇게 입력해도 $and를 쓴 것과 같은 결과를 도출할 수 있다. 객체 내에 여러 개의 연산자를 씀 으로써 논리 연산자를 적용할 대상을 늘릴 수도 있을 것이다.<br />
그렇기에, 보통 어렵지 않은 조건이라면 $and 보다는 $or, $nor를 자주 쓰는 편이다. $and는 좀 더 복잡한 조건이 필요할 때 많이 쓴다. 
</p>
