<p>
다음은 배열 연산자에 대해 기록해보겠다.<br />
실은 연산자 없이 find&#40;&#41; 만으로도 배열을 지니고 있는 도큐먼트 들을 조회할 수 있다.<br />
실습을 위해 다음 컬렉션을 추가한다.   
</p>

```javascript
var inven = db.inventory
inven.insertMany([
    { item : "journal", qty : 25, tags : ["blank", "red"] },
    { item : "notebook", qty : 50, tags : ["red", "blank"] }, 
    { item : "paper", qty : 100, tags : ["red", "blank"] },
    { item : "planner", qty : 75, tags : null},
    { item : "postcard", qty : 45, tags : ["blank", "red", "blue"] },
    { item : "smartTV", qty : 40, tags : ["blue"] },
    { item : "USBmemory", qty : 45, tags : ["blue", "yellow"] },
    { item : "sweetBasil", qty : 1, tags : ["yellow", "red", "blank"] }
])
```

<p>
tags 값으로 &#34;blue&#34;를 지니고 있는 도큐먼트를 조회할려면 다음과 같이 명령어를 작성하면 된다.    
</p>

```javascript
> inven.find({ tags : "blue" }, { qty : false })
{ "_id" : ObjectId("5f096f98b2b8c049b821517d"), "item" : "postcard", "tags" : [ "blank", "red", "blue" ] }
{ "_id" : ObjectId("5f096f98b2b8c049b821517e"), "item" : "smartTV", "tags" : [ "blue" ] }
{ "_id" : ObjectId("5f096f98b2b8c049b821517f"), "item" : "USBmemory", "tags" : [ "blue", "yellow" ] }
```
<p>
이런 식으로 tags 값으로 blue를 지니고 있는 도큐먼트들이 조회된다.<br />
그렇다면 두 개 이상의 값을 검색한다면 어떻게 될까?    
</p>

```javascript
> inven.find( { tags : ["red", "blank"] }, { qty : false } )
{ "_id" : ObjectId("5f096f98b2b8c049b821517a"), "item" : "notebook", "tags" : [ "red", "blank" ] }
{ "_id" : ObjectId("5f096f98b2b8c049b821517b"), "item" : "paper", "tags" : [ "red", "blank" ] }
```

<p>
처음엔 &#34;red&#34;, &#34;blank&#34;를 지니는 도큐먼트라면 전부 조회되겠지? 라고 생각했지만,<br />
순서대로 &#34;red&#34;와 &#34;blank&#34;만을 가지고 있는 도큐먼트들만 검색되었다.   
</p>

<p>
MongoDB에서도 <b>배열은 순서를 가진 여러 개의 인덱스를 포함하는 값</b> 임을 기억하고 있어야 한다.<br />
처음 JS를 공부할 때 객체와 배열의 차이점을 공부했던 시절이 떠오른다.<br />
만약 tags 값으로 &#91;&#34;red&#34;, &#34;blank&#34;&#93;, &#34;blue&#34;를 가지고 있는 도큐먼트가 있었다면, 그것도 조회되었을 것이다.<br />
find&#40;&#41; 명령어로 &#91;&#34;red&#34;, &#34;blank&#34;&#93;를 조회했었으니까 말이다.<br />
배열 자체를 검색할 땐, 순서도 고려함을 기억하고 있자.    
</p>


### $elemMatch
<p>
본격적으로 배열 연산자를 사용해보자.<br />
배열를 다루기 때문에, 배열이 없는 도큐먼트에는 아무 소용이 없다.<br />
다음 명령어를 입력해보자.   
</p>

```javascript
> inven.find({ tags : {$elemMatch : {$eq : "red", $eq : "blank"}} }, { qty : false })
{ "_id" : ObjectId("5f097422b2b8c049b8215182"), "item" : "journal", "tags" : [ "blank", "red" ] }
{ "_id" : ObjectId("5f097422b2b8c049b8215183"), "item" : "notebook", "tags" : [ "red", "blank" ] }
{ "_id" : ObjectId("5f097422b2b8c049b8215184"), "item" : "paper", "tags" : [ "red", "blank" ] }
{ "_id" : ObjectId("5f097422b2b8c049b8215186"), "item" : "postcard", "tags" : [ "blank", "red", "blue" ] }
{ "_id" : ObjectId("5f097422b2b8c049b8215189"), "item" : "sweetBasil", "tags" : [ "yellow", "red", "blank" ] }
```

<p>
이전에 find&#40;&#41;만으로 탐색 했던 것 과는 달리<br />
순서에 관계 없이 해당 값들을 지니고 있는 도큐먼트가 조회되는 것을 관찰할 수 있다.    
</p>

<p>
이렇게 &#36;elemMatch 연산자는, 순서에 관계없이 필드 값으로 지닌 모든 조건들을 만족하는 원소들을 지닌 배열을 출력하는 역할을 담당한다.    
</p>

### $elemMatch : 예제 A
<p>
알아둬야 할 예제가 있다.<br />
실습을 위해 다음과 같은 도큐먼트를 추가한다.   
</p>

```javascript
var number = db.number
number.insertMany([
    { item : "journal", qty : [8, 36] },
    { item : "crimison", qty : [1, 26] },
    { item : "journal", qty : [26, 1, 30] },
    { item : "journal", qty : [3, 2, 1, 60] },
    { item : "journal", qty : [10, 25, 60] }
])
```
<p>
이번엔 qty 값으로 10 미만이면서 25 초과인 도큐먼트들을 검색해보고자 한다.<br />
부등식을 떠올려보면 해당 조건들을 전부 만족하는 값들은 존재하지 않아서 아무 값도 조회되지 않겠지만, 
find&#40;&#41;만으로는 값들을 어떻게든 출력할 수 있다.<br />
다음 명령어를 입력해본다.    
</p>

```javascript
> number.find({ qty : {$lt : 10, $gt : 25} })
{ "_id" : ObjectId("5f097853b2b8c049b821518a"), "item" : "journal", "qty" : [ 8, 36 ] }
{ "_id" : ObjectId("5f097853b2b8c049b821518b"), "item" : "crimison", "qty" : [ 1, 26 ] }
{ "_id" : ObjectId("5f097853b2b8c049b821518c"), "item" : "journal", "qty" : [ 26, 1, 30 ] }
{ "_id" : ObjectId("5f097853b2b8c049b821518d"), "item" : "journal", "qty" : [ 3, 2, 1, 60 ] }
```
<p>
이럴수가! 도큐먼트가 조회됨을 확인할 수 있다.<br />
10 미만인 수를 가지고 있거나, 25를 초과한 수를 가지고 있는.. 으로 판단했기 때문에
저렇게 출력하는 것이다. 논리 게이트를 생각해보면 OR과 비슷하다고 할 수 있다.<br />
만약 해당 조건을 모두 만족시키도록 쿼리를 작성하려면 &#36;elemMatch를 써야 할 것이다.<br />
물론, 그 어떠한 값도 조회되지는 않겠지만 말이다.    
</p>

### $all

<p>
$all은 순서와 관계없이 값으로 받은 모든 요소들을 지닌다면, 해당 도큐먼트를 불러온다.<br />
입력 형식은 다음과 같다.
</p>

```javascript
db.collection.find(
    { $all : [<value1>, <value2>] }
)
```

<p>
잘 보면 논리 연산자 $and, $or, $nor 처럼 값으로 배열을 받는다는 것을 확인할 수 있다.<br />
그냥 뜻만 봐서는 $elemMatch의 역할과 큰 차이가 없어보인다.<br />
$elemMatch가 모든 <b>조건</b>을 만족하는 것을 조회하는 것 이라면<br /> 
$all은 값으로 지닌 모든 <b>원소</b>를 지닌 것들 조회한다 라는 것이 차이점이다.    
</p>

<p>직접 연산자를 사용해보면 다음과 같다.</p>

```javascript
> inven.find(
    { tags : {$all : ["blank", "red"]} }, 
    { qty : false}
)
{ "_id" : ObjectId("5f0c8aa64eed560b2d2216f6"), "item" : "journal", "tags" : [ "blank", "red" ] }
{ "_id" : ObjectId("5f0c8aa64eed560b2d2216f7"), "item" : "notebook", "tags" : [ "red", "blank" ] }
{ "_id" : ObjectId("5f0c8aa64eed560b2d2216f8"), "item" : "paper", "tags" : [ "red", "blank" ] }
{ "_id" : ObjectId("5f0c8aa64eed560b2d2216fa"), "item" : "postcard", "tags" : [ "blank", "red", "blue" ] }
{ "_id" : ObjectId("5f0c8aa64eed560b2d2216fd"), "item" : "sweetBasil", "tags" : [ "yellow", "red", "blank" ] }
```
<p>tags 필드 값으로 "blank", "red" 원소들을 지닌 도큐먼트를 조회하는 것을 알 수 있다.</p>

### $elemMatch와 $all의 조합

<p>
그렇다면 $all만으로<br /> 
"10 초과의 원소"와 "25 미만의 원소"를 지니고 있는 배열을 가진 도큐먼트를 조회할 수 있을까?   
</p>

<p>
조건이 붙어 있기 때문에 $all 만으로는 해당 상황을 해결할 수 없다.<br />
조건이 붙어 있는 원소들을 포함하는 배열을 탐색하려면,<br />
$elemMatch와 $all를 조합할 필요가 있다.    
</p>

```javascript
number.find(
    { qty : {$all : [{$elemMatch : {$gte:10}}, {$elemMatch : {$lte:25}}]}, item : "journal" }
)
{ "_id" : ObjectId("5f097853b2b8c049b821518a"), "item" : "journal", "qty" : [ 8, 36 ] }
{ "_id" : ObjectId("5f097853b2b8c049b821518c"), "item" : "journal", "qty" : [ 26, 1, 30 ] }
{ "_id" : ObjectId("5f097853b2b8c049b821518d"), "item" : "journal", "qty" : [ 3, 2, 1, 60 ] }
{ "_id" : ObjectId("5f097853b2b8c049b821518e"), "item" : "journal", "qty" : [ 10, 25, 60 ] }
```
<p>
쿼리 인자 객체의 첫 번째 필드(qty)를 잘 관찰해보자.<br />
각 조건을 만족하는 원소들을 지닌 도큐먼트들을 성공적으로 조회하는 것을 관찰할 수 있다.   
</p>

<p>여기까지 배열 연산자에 관한 기록이다.</p>

<p>
배열이 없는 도큐먼트엔 무용지물 이라는 것은<br />
배열 내의 원소를 탐색해야 할 때에는 해당 연산자를 사용해야 함을 알아두자.    
</p>




