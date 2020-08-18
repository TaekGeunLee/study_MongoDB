## 1번
MongoDB의 특징인 샤딩과 복제에 대해 설명해라.
#### my answer
샤딩(shading)
: 데이터를 분산, 잘게 쪼개는 행위를 의미합니다.
  DB 내에 입출력되는 데이터의 양이 무거울 경우 데이터를 샤딩 시킴으로써
  DB 동작 속도를 향상 시킵니다.
 
- 복제(Replicate)
: 구축한 DB를 새로 더 만들어두는 행위입니다.
  DB가 손상되는 등의 문제가 생겼을 경우 미리 복제해둔 DB로
  연결을 옮김으로써 언제나 사용할 수 있는 상태를 유지합니다.

MongoDB에서는 샤딩과 복제를 가장 중요하게 여긴다고 판단됩니다.
#### reference
<ul>
    <li>서적
        <ul>
        <li>맛있는 MongoDB : (7~8p), (317~353p)</li>    
        </ul>
    </li>
    <li>포스팅 글/API</li>
    <li>영상</li>
</ul>

## 2번
NoSQL이 무엇인지에 대해 설명하고, RDBMS과의 차이점을 구술해라.
#### my answer
NoSQL은 보통 테이블(표) 형태의 릴레이션을 구성해서 쓰이는 RDBMS 형태가
아닌 다른 형태의 데이터 저장 기술을 뜻합니다.

- RDBMS에서 CRUD 작업을 진행할 때 사용하는 SQL문을 사용하지 않습니다.
  NoSQL의 일종 중 하나인 MongoDB는 DML의 역할과 비슷한 걸로 query를
  사용합니다.

- 스키마를 정의하지 않습니다. 덕분에 데이터 구조의 제약을 받지 않아
  입출력이 유동적입니다. 다만 구조, 형식을 정해놓지 않기 때문에 일관성이
   떨어집니다.

- JOIN과 트랙잭션을 지원하지 않습니다.
  완전히 그 기능들을 사용할 수 없다는 말은 아니지만, 약간의
  제약이 따릅니다.
#### reference
<ul>
    <li>서적
        <ul>
        <li>맛있는 MongoDB : 5~10p</li>
        <li>Node.js 교과서 : 298~299p</li>    
        </ul>
    </li>
    <li>포스팅 글/API</li>
    <li>영상</li>
</ul>
  
## 3번
도큐먼트는 BSON 형식으로 구성되어 있다. JSON과의 차이점은 뭔가?
#### my answer
BSON(Binary JSON)은 JSON 파일을 기반으로 이진화된 형식의
파일로, 형식은 JSON과 많이 유사합니다.
JSON이 브라우저 간의 비동기 통신에 정보 교환을 목적으로 쓰는 형식이라면,
BSON은 컴퓨터 교환 형식으로 데이터 저장 및 네트워크 전송에 주로 사용되는
형식입니다. 
출처 : https://www.educba.com/json-vs-bson/


## 4번
데이터 타입에 관해 묻겠다. Objectid 타입에 대해 설명해봐라.
#### my answer
컬렉션 내에 도큐먼트를 생성할 때 마다 자동으로 생성되는 값으로,
각 도큐먼트 마다 중복된 값이 아닌 고유한 값을 가지게 된다.
RDBMS로 치면 기본키의 역할을 담당한다.

생성된 날짜, 기기 id, 프로세스 id, 카운터 값을 할당 받아 이들의 조합으로
항상 고유한 값을 가질 수 있도록 한다.


## 5번
MongoDB의 가장 중요한 특징을 대봐라.
#### my answer
- RDBMS와 같은 전통적인 DB 보다 빠른 속도와 확장성을 지님.
- (SQL문의 역할을 담당하는) 다양한 종류의 쿼리들
- 스키마를 만들지 않아도 되어 유동적인 데이터 모델을 지님.
#### reference
<ul>
    <li>서적
        <ul>
        <li>맛있는 MongoDB : 5~10p</li>
        <li>Node.js 교과서 : 298~299p</li>    
        </ul>
    </li>
    <li>포스팅 글/API
        <ul>
        <li><a href="https://www.tutorialsjar.com/key-features-of-mongodb/">tutorialsjar_key features of MongoDB</a></li>   
        </ul>
    </li>
    <li>영상</li>
</ul>

## 6번
NoSQL의 종류를 나열하라. MongoDB는 어느 타입의 NoSQL인가?

## 7번
어떤 언어에서 MongoDB를 사용할 수 있는가?

## 8번
집계 연산(aggregate)이 무엇인가? MongoDB에서 사용할 수 있는 집계 연산의 방법을 나열하시오.
#### reference
<ul>
    <li>서적
        <ul>
        <li>맛있는 MongoDB : 126~132p</li>    
        </ul>
    </li>
    <li>포스팅 글/API</li>
    <li>영상</li>
</ul>

## 9번
MongoDB 아키텍쳐를 간단하게 그려보시오.

## 10번
MongoDB에서 트랜잭션과 락킹(locking)을 하는 방법이 있을까?

## 11번
스토리지 엔진(Storage Engine)이 무엇인가?MongoDB에서 사용하는 2개의 엔진은 무엇일까?