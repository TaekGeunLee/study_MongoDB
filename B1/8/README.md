### deleteOne(), deleteMany()

<p>제거 명령어는 정말 쉽다.
기존의 수정 명령어와 사용 방법이 정말 비슷하다.<br />
예제를 살펴보자.</p>

```javascript
> var berry = db.berryCollection

> berry.deleteOne(
... {name:"Basilgreen"})
{ "acknowledged" : true, "deletedCount" : 1 }

> berry.find()
{ "_id" : ObjectId("5ef4654edb1f8d6e902de695"), "name" : "Salmon-light", "status" : 5, "VIP" : true }
{ "_id" : ObjectId("5ef4654edb1f8d6e902de696"), "name" : "Cup-cake", "status" : 100, "VIP" : true }
{ "_id" : ObjectId("5ef46938062e73623a423ef6"), "name" : "Crimson", "status" : 7 }
```
<p>
여러 개의 document를 제거하는 deleteMany() 도
위와 같이 쓰면 된다. 두 번째 인자로 옵션값도 있지만 이것은 생략하도록 하겠다.<br />
컬렉션 그 자체를 지울려면 drop() 명령어를 쓰면 된다.    
</p>

<p>
이처럼, 가장 기초적인 수정하고 제거, 읽는 명령어에는 공통적으로
쿼리(query)가 쓰인다.<br />
쿼리 다루는 방법을 잘 알고 있으면 CRUD 명령어를 자유자재로 다룰 수 있다.
</p>



