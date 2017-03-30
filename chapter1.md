# URL Shortener 설계하기

우리가 만는들 프로젝트 URL Shortener 라고 하며, 긴URL 주소를 짧게 줄여주는 서비스입니다  
Google URL Shortener \([https://goo.gl/](https://goo.gl/)\) 또는 Bitly \([https://bitly.com/](https://bitly.com/)\) 가 대표적인 서비스이며, 주로 블로그나 Social Service 에서 글을 작성할 때 긴 URL주소를 줄여서 쓰기 위해 많이 사용되고 있습니다. 또한 이 링크를 통해 URL 클릭 수를 통계 내는데 사용하기도 하여 많은 기업에서 자체 구축하여 사용하는 경우도 있습니다.

### 필요한 기능 설계하기

URL Shortener 는 긴 URL 주소를 입력 받아, 이 URL에 대응되는 키 값을 부여하여 DB에 저장합니다. 그리고, 키값을 서버에 요청하면, 서버는 DB에서 URL 을 찾아 페이지 이동시켜주면 될 것입니다.

긴 URL을 줄이는 가장 쉬운 방법은 입력 받은 URL 마다 고유한 일련번호\(Sequence Number\)를 부여 하는 것이다. 그리고 웹 브라우저에서 이 일련번호를 서버로 보내면, 서버는 원래 긴 URL 을 찾아서 redirect 해주게 된다.

&lt; **URL마다 일련번호 부여** &gt;

![](/images/func01.png)

하지만 숫자로 점차 증가하는 일련번호는 URL이 급격하게 많아지면 자리수가 빠르게 늘어날 것입니다. 그래서 간단한 알고리즘을 구현한 함수를 통해 자리수가 최대한 천천히 늘어나도록 해야 할 것입니다.

간단한 알고리즘은 전단사함수\(Bijection\)를 구현한 것으로, 숫자만으로 구성된 일련번호를 대/소문자와 숫자의 조합으로 변경하는 것이다.

```js
var alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
var base = alphabet.length;

function encode(num){
  var encoded = '';
  while (num){
    var remainder = num % base;
    num = Math.floor(num / base);
    encoded = alphabet[remainder].toString() + encoded;
  }
  return encoded;
}

function decode(str){
  var decoded = 0;
  while (str){
    var index = alphabet.indexOf(str[0]);
    var power = str.length - 1;
    decoded += index * (Math.pow(base, power));
    str = str.substring(1);
  }
  return decoded;
}
```

위의 JavaScript 를 실행하고 아래와 같이 테스트 해 봅니다. \(Chrome 브라우져의 'Inspect' 의 console 탭에서 실행 테스트 해볼 수 있습니다.\)

```
> encode(2147483647);
"cvuMLb"
> decode('cvuMLb');
2147483647
```

실행하면, 2147483647 -&gt; cvuMLb으로 출력 결과를 확인 할 수 있습니다.  
입력된 10진수 숫자를 62진수로 변환되는 것과 같습니다. 만약 자리 수를 더 줄이기 위해서는 ALPHABET 변수에 알파벳과 숫자 외에도 ‘-‘, ‘\_’ 등 다른 문자도 추가하면 될 것이다.

&lt; **URL마다 일련번호 생성하고 Bijection 함수로 변환 &gt;**

![](/images/func02.png)

