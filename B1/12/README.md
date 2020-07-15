<p>다음은 문자열에 관련된 연산자이다.<br />
이전에 배운 논리 연산자 보다는 이해하기 쉬울 것이다.</p>

<p>정규표현식을 쓰는 $regex과 일반 연산자인 $text가 있다.<br />
현재 시점에서 정규표현식을 모르기 때문에 일단 전자는 기록하지 않겠다.</p>

### $text

<p>실습을 위해 다음 도큐먼트들을 생성한다.</p>

```javascript
> var store = db.store
> store.insertMany([
...     {_id: 1, name: "Java Hut", description: "Coffee and cakes"},
...     {_id: 2, name: "Burger Buns", description: "Gourmet hamburgers"},
...     {_id: 3, name: "Coffee Shopping", description: "Just coffee"},
...     {_id: 4, name: "Clothes Clothes Clothes", description: "Discount clothing"},
...     {_id: 5, name: "영희의 옷", description: "옷이 펄럭거리다"},
...     {_id: 6, name: "달리는 철수", description: "철수는 인도인"},
... ])
{ "acknowledged" : true, "insertedIds" : [ 1, 2, 3, 4, 5, 6 ] }
```

<p>
이제 연산자를 사용해보자. 문자열을 사용할 때에는 제일 먼저 <b>인덱스(index)</b>를 만들어놔야 한다.<br />
자료구조로 이루어져 있으며, 문자열을 조회하는데 지원을 해주지만,
정작 MongoDB 내에서는 필수로 생성을 해야 하는 요소이다.<br />
아래의 명령어를 입력한다.    
</p>

```javascript
> store.createIndex( {name: "text", description: "text"})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
```
<p>
인덱스의 생성을 완료했다. 그럼 이제 문자열을 조회해보자.    
</p>

```javascript
> store.find({ $text : {$search : "java burger"} })
{ "_id" : 2, "name" : "Burger Buns", "description" : "Gourmet hamburgers" }
{ "_id" : 1, "name" : "Java Hut", "description" : "Coffee and cakes" }
```
<p>
"burger"와 "java"가 포함된 문자열을 찾아낸다.(필드와 대소문자에 구별하지 않는다.)<br />
언급하지 않은 $search는, 검색하려는 문자열을 담는 기본적인 연산자이다.<br />
띄어쓰기를 포함하면서까지 문자열을을 찾기 위해서는 이스케이핑의 과정이 필요하다.    
</p>

```javascript
> store.find({ $text : {$search : "\"coffee and\""} })
{ "_id" : 1, "name" : "Java Hut", "description" : "Coffee and cakes" }
```

<p>
이스케이핑 할 문자 뒤에 큰따옴표(")를 쓰는 것을 기억하자.<br />
개발에서 공통적으로 해당하는 사항이다.    
</p>

<p>
$text 프로퍼티의 값으로 연산자를 더 넣어 옵션을 걸 수 있는데, <br />
언어($language)를 필터링 하거나, $대소문자($caseSensitive)를 구별하게 할 수 있다.
언어의 경우, 전체 언어를 지원하지는 않는다. 한국어도 거기에 포함된다.
</p>

### $text의 제약점
<p>$regex과 비교하면 많은 제약점이 있다.</p>

<p>첫 번째로 한글 문자열은 조회가 안된다는 점이다.<br />
$text가 따로 지원하는 언어만 조회가 가능한데, 한글은 지원하지 않는다는 것이다.
</p>

<p>
두 번째로 문자열 인덱스를 만들어야 한다는 점이다.<br />
$regex는 따로 인덱스를 만들 필요가 없되, $text는 문자열을 새로 만들어 줘야 한다.   
</p>

<p>$regex를 쓰기 위해 따로 정규표현식을 공부할 필요가 있을 것 같다.</p>
