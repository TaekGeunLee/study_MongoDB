<p>
find()의 두 번째 인자로 프로젝션 값을 할당하여<br />
쿼리로 조회한 도큐먼트들 중 어느 필드만 보이게 할 것인지 설정할 수 있다.<br />
이번 레퍼토리에서는 프로젝션 쿼리에 대해 기록해본다.    
</p>

### 점 연산자(.)를 이용한 객체 접근

<p>
쿼리를 공부할 때<br />
점 연산자(.)로 필드에 접근한 뒤 큰따옴표(")로 감싸주는 것으로<br />
객체의 필드값에 접근할 수 있었다. 이것은 프로젝션도 마찬가지이다.    
</p>


```javascript
> var student = db.student

> student.find({}, {"name.first" : true})
{ "_id" : ObjectId("5f0db0c32ed65e074c7507db"), "name" : { "first" : "철수" } }
{ "_id" : ObjectId("5f0db0c32ed65e074c7507dc"), "name" : { "first" : "영희" } }
{ "_id" : ObjectId("5f0db0c32ed65e074c7507dd"), "name" : { "first" : "수영" } }
{ "_id" : ObjectId("5f0db0c32ed65e074c7507de"), "name" : { "first" : "희영" } }
```

### 배열에 접근하기

<p>
그렇다면 프로젝션을 배열에 적용할려면 어떻게 해야할까?<br />
배열을 위한 프로젝션 연산자들이 존재한다.<br />
제일 먼저 $slice에 대해 알아보자.    
</p>

```javascript
> inven.find(
    { tags : {$all : ["yellow"] } }, 
    { tags : {$slice : 1} }
)
{ "_id" : ObjectId("5f0db14a2ed65e074c7507e5"), "item" : "USBmemory", "qty" : 45, "tags" : [ "blue" ] }
{ "_id" : ObjectId("5f0db14a2ed65e074c7507e6"), "item" : "sweetBasil", "qty" : 1, "tags" : [ "yellow" ] }
```

<p>
find()의 프로젝션 인자를 살펴보자.<br />
배열의 맨 첫 째 원소부터 n개의 원소까지 보이게 할려면, 저렇게 쓰면 된다.<br />
예를 들어서 3개까지 살펴보고 싶으면 $slice : 3. 이런식으로 말이다.<br />
음수 값으로 쓰면 가장 마지막 원소부터 센다.   
</p>


```javascript
> inven.find(
    { tags : "red" }, 
    { item : true, "tags.$" : true }
)
```
<p>
프로젝션으로 $elemMatch도 사용할 수 있다.<br />
기존 쿼리에서 사용하는 것과 똑같다. (완전 똑같다는 건 아닐지도.)    
</p>

### $

<p>
마지막으로 연산자 $에 대해 알아보자.<br />
실습을 위해 다음 컬렉션을 추가한다.    
</p>

```javascript
var avo = db.avocadoCollection
avo.insertMany([
    {"name" : "oliveBasic", "array" : [ 0, 1, 2 ], "VIP" : false, "status" : true },
    {"name" : "aliceblue", "array" : [ 0, 2, 4, 3 ], "VIP" : true, "status" : true },
    {"name" : "Redgreen", "array" : [ 1, 3, 5, 100 ], "VIP" : false, "status" : true },
    {"name" : "Basilgreen", "array" : [ 0, 1, 2, 3 ], "VIP" : true },
    {"name" : "LeafGreen", "array" : [ "a", 0 ] }
])
```

<p>
$는, 쿼리 조건을 만족하는 데이터들 중 첫 번째 값을 반환한다.<br />
여기서 조회한 데이터는 배열을 의미하며, 쿼리 인자로 연산자를 썻을 경우에만 적용된다.<br />
직접 사용해보면 다음과 같다.    
</p>

```javascript
> avo.find(
    { array : {$elemMatch : { $lte : 3}} }, 
    { name : true, "array.$" : true }
)
{ "_id" : ObjectId("5f0ef9520083d8c210a22d4d"), "name" : "oliveBasic", "array" : [ 0 ] }
{ "_id" : ObjectId("5f0ef9520083d8c210a22d4e"), "name" : "aliceblue", "array" : [ 0 ] }
{ "_id" : ObjectId("5f0ef9520083d8c210a22d4f"), "name" : "Redgreen", "array" : [ 1 ] }
{ "_id" : ObjectId("5f0ef9520083d8c210a22d50"), "name" : "Basilgreen", "array" : [ 0 ] }
{ "_id" : ObjectId("5f0ef9520083d8c210a22d51"), "name" : "LeafGreen", "array" : [ 0 ] }
```
<p>
쿼리로 3 이하의 원소를 지닌 배열을 값으로 가진 array를 가진 도큐먼트들을 조회했다.<br />
어떤 도큐먼트는 조건을 만족하는 복수의 원소들을 지니고 있다.<br />
이때 프로젝션으로 $ 연산자를 쓰니 조회한 원소들중 첫 번째 값을 출력한 것이다.   
</p>

<p>
이해가 안된다면 name : Redgreen 를 지닌 도큐먼트를 살펴보자.<br />
array 필드에 적용한 쿼리 조건에 해당하는 원소는 1, 3이다.<br />
이 중에 첫 번째 원소만 보이게 했으니 1이 출력이 되는 것이다.   
</p>




