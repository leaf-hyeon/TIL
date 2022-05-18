# 몽고DB 스키마 설계

몽고DB 스키마를 설계하기 위해서는 도큐먼트들의 관계를 먼저 파악할 필요가 있다.

또한 관계의 **양적**인 면도 살펴봐야 한다.



### 일대다 관계

---



#### One to Few

```javascript
> db.person.findOne()
{
  name: 'Kate Monster',
  ssn: '123-456-7890',
  addresses : [
     { street: '123 Sesame St', city: 'Anytown', cc: 'USA' },
     { street: '123 Avenue Q', city: 'New York', cc: 'USA' }
  ]
}
```

사람과 사람이 현재 거주하고 있는 주소는 일대다 관계이며 양적인 면에서는 one to few 관계이다.

이때 주소는 **도큐먼트에 임베딩**하는 좋은 사용 사례이다.

이 디자인의 장점은 사람이 거주하고 있는 주소 정보를 얻기 위해서 별도의 쿼리 필요없이 한번의 쿼리로 가져올수 있다는것이 장점이다.

하지만 단점으로는 포함된 주소 정보를 독립적으로 액세스 할수 없다는것이 단점이다. 그러나 이 예제에서는 보통 주소 정보가 필요할때는 사람에 대한 정보도 같이 가져오지 주소 자체를 독립적으로 가져오지는 않기 때문에  도큐먼트에 임베딩 하는 좋은 사례로 보인다.



#### One to Many

```javascript
> db.parts.findOne()
{
    _id : ObjectID('AAAA'),
    partno : '123-aff-456',
    name : '#4 grommet',
    qty: 94,
    cost: 0.94,
    price: 3.99
```

```javascript
> db.products.findOne()
{
    name : 'left-handed smoke shifter',
    manufacturer : 'Acme Corp',
    catalog_number: 1234,
    parts : [     // array of references to Part documents
        ObjectID('AAAA'),    // reference to the #4 grommet above
        ObjectID('F17C'),    // reference to a different Part
        ObjectID('D2AA'),
        // etc
    ]
```

제품과 부품은 일대다 관계이다. 여기서 부품이 수백개~수천개 정도 된다면 One to Many 관계로 볼수 있다.

여기서는 One to few 관계와 달리 도큐먼트 사이의 관계를 표현하기 위해 도큐먼트를 임베딩하여 표현하지 않고 부모 객체가 자식 객체의 ObjectID를 배열에 배치하여 관계를 표현하였다.

이 디자인의 장점은 다음과 같다.

- 도큐먼트의 크기 제한(16MB)를 효율적이게 사용할수 있다.
- 제품 도큐먼트 조회시 임베딩 방식보다 용량이 상대적으로 작아서 몽고DB 서버의 버퍼캐시를 효율적으로 이용할수 있다.
- 제품과 부품 도큐먼트 각각을 독립적으로 조회할수 있다.

단점으로는 제품의 부품에 대한 세부정보를 얻기 위해서는 두번의 쿼리를 수행해야 한다. 또한 제품에 대한 부품을 추가할시에 제품 컬렉션과 부품 컬렉션 각각 insert를 해야 하기 때문에 두번의 커맨드가 필요하다.



#### One to Squillions

```javascript
> db.hosts.findOne()
{
    _id : ObjectID('AAAB'),
    name : 'goofy.example.com',
    ipaddr : '127.66.66.66'
}

>db.logmsg.findOne()
{
    time : ISODate("2014-03-28T09:42:41.382Z"),
    message : 'cpu is on fire!',
    host: ObjectID('AAAB')       // Reference to the Host document
}
```

호스트와 호스트에서 발생하는 로그 메세지의 관계는 일대다 관계이다. 여기서 발생하는 로그 메세지의 수가 굉장히 많을때 One to Squillions 관계로 볼수 있다.

이때는 One to Many 관계처럼 부모 도큐먼트에 자식 도큐먼트 ObjectID를 배열에 배치한다면 도큐먼트의 크기 제한(16MB)를 충분히 넘을수도 있기 때문에 자식 도큐먼트에 부모 도큐먼트 ObjectID를 배열에 배치해야 한다.



#### 정리

**고려할 사항**

- 일대다 관계에서는 다쪽에 있는 객체가 독립적으로 존재할 필요가 있는가?
- 관계는 어떠한가? One to Few, One to Many, One to Squillions ?

**3가지 팁**

- One to few 관계 일때 다쪽의 객체가 독립적으로 존재할 필요가 없다면 임베딩 한다.
- One to many 관계라면 One쪽에서 many쪽에 대한 reference를 저장한다.
- One to Squillions 관계라면 Squillions쪽에 One의 reference를 저장한다.



### 양방향 참조와 비정규화

---

#### 양방향 참조

프로젝트 팀원이 있고 작업이 있을때 이들 관계는 일대다 관계이며 One to Many로 볼수 있다. 

어플리케이션에서 프로젝트 팀원이 작업한 목록을 보여주는 기능이 있을때  이때 스키마 디자인은 다음과 같을것이다.

```javascript
db.person.findOne()
{
    _id: ObjectID("AAF1"),
    name: "Kate Monster",
    tasks [     // array of references to Task documents
        ObjectID("ADF9"), 
        ObjectID("AE02"),
        ObjectID("AE73") 
        // etc
    ]
}
```

즉, 부모 객체가 자식 객체를 referencing 하고 있는 형태이다.

여기서 추가적으로 어플리케이션이 전체 작업의 목록을 보여줘야 하고 그 작업을 진행한 팀원을 보여주는 기능이 있을때 현재 스키마 설계에서는 person 컬렉션의 tasks 필드 인덱스 없이 쿼리하기에는 성능적으로 매우 느리다.

인덱스 없이 더 효율적인 스키마 구조를 만들기 위해서는 자식 도큐먼트에서 부모 도큐먼트에 대한 reference를 추가하는 방법이 있다. 구조는 다음과 같다.

```javascript
db.tasks.findOne()
{
    _id: ObjectID("ADF9"), 
    description: "Write lesson plan",
    due_date:  ISODate("2014-04-01"),
    owner: ObjectID("AAF1")     // Reference to Person document
}
```

부모 도큐먼트에 대한 reference로 **owner** 필드가 추가 된것을 볼수 있다.

단방향 참조에 비해 양방향 참조의 단점으로는 기존에 작업에 할당되었던 프로젝트 팀원을 다른 팀원에게 (새롭게 합류한 팀원이 아닌) 재할당 해야 한다면 업데이트가 두번 일어난다는것이다. 또한 도큐먼트간에는 atomic한 



#### 비정규화

제품을 조회할때 부품의 이름도 항상 같이 필요하고 부품의 이름이 자주 변경되지 않는다면 이때가 비정규화의 좋은 사용 사례이다.

비정규화 스키마 디자인은 아래와 같다.

```javascript
> db.products.findOne()
{
    name : 'left-handed smoke shifter',
    manufacturer : 'Acme Corp',
    catalog_number: 1234,
    parts : [
        { id : ObjectID('AAAA'), name : '#4 grommet' },         // Part name is denormalized
        { id: ObjectID('F17C'), name : 'fan blade assembly' },
        { id: ObjectID('D2AA'), name : 'power switch' },
        // etc
    ]
}
```

제품 도큐먼트의 parts 배열에 name 필드 (부품 이름) 가 추가된것을 확인 할수 있다. 

부품의 다른 정보가 필요하다면 추가적인 쿼리가 필요하다. 하지만 이름만 필요하다면 한번의 쿼리만으로 확인 가능하기 때문에 읽기에 효율적이다.

그러나 부품의 이름이 변경 된다면 제품 컬렉션과 부품 컬렉션 각각 업데이트가 필요하기 때문에 비효율적이다. 따라서 어플리케이션의 쓰기에 대한 읽기의 비율을 고려하여 비정규화 해야 한다.