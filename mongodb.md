# MongoDB 에 데이터 관리하기

### MongoDB 의 특징

MongoDB는 필드Key와 값 를 갖는 JSON\(JavaScript Object Notation\)형태로 데이터를 저장합니다. MongoDB에서는 바이너리 파일도 함께 저장이 가능하며, 이것을 BSON\(binary and JSON Document\) 라고 합니다.

![](https://lh6.googleusercontent.com/rtSnNzYJvrpUYKtcbEZwlrlIQSIpOfPvwIMHh-o-RInXi1Ml-s0d_hlrY9laTM8dgBn6Q0t7f_ikEFbVbAvqnNc4Kp9LjvWZJNJ-mvzboU8VzfMoJtduCnoBWsAcBYVOfMwmIfgFhUhrCtTi)

그리고, 모든 Document들을 Collection에 저장됩니다. Collection은 Document들의 그룹이며, Document의 필드 별로 인덱스Index를 생성할 수 있는 구조로 되어 있습니다.

MongoDB의 모든 Document에는 `_id` 라는 특수한 타입의 필드가 존재하며, 중복되지 않는 유일한 값이 저장됩니다. `_id` 필드는 항상 인덱싱\(indexing\)되며, Document 생성시 명시 하지 않으면 자동으로 12byte 값으로 만들어지고, 필요에 따라서는 명시적으로 값을 부여하여 생성할 수도 있습니다.

URL Shortener 에서는 `_id` 필드의 값을 자동생성 하도록 하지 않고, 직접 값을 넣어서 사용할 것입니다.

![](https://lh5.googleusercontent.com/Kif4fH73kW6ZQSn4M_MoLVehDwUsIk8Rl2yWbC6bcHTaeqRcLNTmeG16XcxGCerXXHWL7Dtfo5tXVSMwyBGljvKxLal-zDHzkZsqZ0y8JurXFokB-d7AdyzmdrQklo80nQyEai-51UWTo_6G)

RDB와 MongoDB를 비교하자면, Table을 Collection으로, row 데이터를 document로 비교해볼 수 있겠습니다.

MongoDB의 Query는 SQL 과 전혀 다르며, Script 언어와 비슷하게 작성하도록 지원하고 있습니다. 예제로, 'users' Collection 에서 ‘age’ 필드가 18 보다 큰 Document 들을 조회하고, ‘age’ 필드로 오름차순으로 정렬한다고 한다면 아래 그림과 같습니다.

![](https://lh6.googleusercontent.com/1sDF87iRA1MRQDwVojapY4r6bmm3zxbPI39rwK52uJ02uGsa59aUjhdDoF8BzmWOYWgvSnpQjAfIIedD3r-tMyQjrmsXJlELY1nuYdIQhq7nxz4gt1tPrS6e-yDL8_yn0hBxjYa5n0_wJVOW)

insert와 update 도 마찬가지이며, 아래 예제를 참고할 수 있겠습니다.

![](https://lh6.googleusercontent.com/ogEM-_ccsINKrRCcBeqH1Zjg6GxI8YQ57GJoZyW8hqcUZrozQHu9t-qe0iIMYNEH3Ku9MgdSGGpY0bUg8MwdP5JnLBEgJAtEevxbJnLQH_31abfotffjlNDTU3OxkItRP2jkte02ia2fGCKS)

그리고, MongoDB는 미리 Collection을 생성해둘 필요가 없으며, Document를 처음 insert할 때 Collection도 신규 생성되게 됩니다.

### 일련번호 Collection

MongoDB는 일련번호를 생성하는 기능을 제공하지는 않습니다. 그러므로, 일련번호를 구하기 위한 Collection과 Document구조를 정의하여 직접 일련번호를 구하는 조회 쿼리를 만들어 보도록 합니다.

**일련번호 Collection \(sequences\)**![](/images/mongo01.png)일련번호 Collection인 ‘sequences’ 에는 URL Shortener 를 위한 현재 일련번호\(‘seq’ 필드\)가 포함된 Document 하나만 저장합니다. 새로운 일련번호를 생성하기 위해서는, ‘seq’ 필드 값을 1씩 증가시켜서 조회해오면 될 것입니다. 이를 구현하기 위해서 findAndModify 함수를 사용합니다.

```js
> db.sequences.findAndModify({
    query: { _id: "urlShortener" },
    update: { $inc: { seq: 1 } },
    upsert: true
  })
```

findAndModify 는 조회조건에 맞는 document 를 찾아서 수정해 주는 기능을 제공합니다. 그리고 실행 결과로 조회조건으로 찾은 Document 를 반환합니다.

작성된 쿼리를 보면, ‘\_id’ 필드가 ‘urlShortener’ 인 Document를 찾고, 이 Document의 ‘seq’ 필드 값에 1을 증가시켜 다시 저장합니다. 그리고, 그 결과 Document를 반환할 것입니다.

upsert 가 true 인 경우 조회조건에 맞는 document 가 없을 때 새로운 Document를 생성할 것입니다.  
가장 처음 실행될 경우, {‘\_id’: ‘urlShortener’, ‘seq’: 1} 의 Document가 ‘sequences’ Collection 에 생성 될 것입니다.

위의 명령을 직접 MongoDB 에서 실행해보고, ‘seq’ 필드 값이 1씩 증가되는지 직접 확인해 봅시다.

findAndModify함수에 대한 자세한 설명은 MongoDB Document 를 참조합니다.

\(http://docs.mongodb.org/manual/reference/method/db.collection.findAndModify\)

### URL 저장 Collection

URL Shortener는 앞장의  '필요한 기능'에서 설명한대로, 앞에서 생성한 일련번호를 그대로 사용하지 않고, Bijection 알고리즘을 통해 문자가 포함된 짧은 코드로 변환한 코드 값이 URL과 함께 MongoDB에 저장되도록 할 것입니다. 코드 값을 ‘\_id’ 필드 값으로 저장하면, 자동으로 인덱싱 되어 코드 값으로 검색할 때 빠르게 Document를 찾아 올 수 있을 것입니다.

**URL 저장 Collection \(urls\)**![](/images/mongo02.png)

insert 메소드를 사용하여 ‘urls’ Collection 에 코드 값과 URL로 구성된 Document를 신규 저장하는 예제는 다음과 같습니다.

```js
> db.urls.insert( { _id: "A1", url: "http://long.url.example.com/abcdefg" } )
```

그리고, 코드 값으로 원래 URL을 가지고 있는 Document를 조회해 오는 예제는 다음과 같습니다.

```js
> db.urls.findOne({ _id: "A1"});
```



