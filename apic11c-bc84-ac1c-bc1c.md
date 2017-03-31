# API 서버 개발

### MongoDB Schema

MongoDB 에 접속하고 명령어를 실행하기 위하여 MongoDB 용 모듈중 하나인 mongoose 를 설치 합니다.

```
$ npm install --save mongoose
```

이제 mongoose 모듈을 사용하여 MongoDB 의 document 인 `sequences` 와 `urls` 의 스키마 정의를 합니다.

`models.js` 파일을 아래와 같이 작성합니다.

```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var SequencesSchema = Schema({
    _id: { type: String, required: true },
    seq: { type: Number, default: 0 }
});

var sequences = mongoose.model('sequences', SequencesSchema);

var UrlsSchema = new Schema({
  _id: {type: Number, index: true},
  url: String,
  created_at: Date
});

UrlsSchema.pre('save', function(next){
  var self = this;
  sequences.findOneAndUpdate({_id: 'url_count'}, {$inc: {seq: 1} }, {upsert: true}, function(error, result) {
    console.log(result);
    if (error) return next(error);
    self.created_at = new Date();
    self._id = result.seq;
    next();
  });
});

var urls = mongoose.model('urls', UrlsSchema);

module.exports = urls;
```

mongoose 모듈은 mongoDB의 findAndModify 이름의 함수 대신에, 용도에 맞는 다양한 이름의 함수를 제공하고 있습니다. 본 프로젝트에서는 조회 후 Update 하는 findOneAndUpdate 를 사용했습니다.

자세한 설명은 [http://mongoosejs.com/docs/api.html\#query\_Query-findOneAndUpdate](https://www.gitbook.com/book/john-kim/training-nodejs-2/edit#) 를 참조합니다.

### Bijective 유틸

`bijective.js` 파일을 아래와 같이 작성합니다.

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

module.exports.encode = encode;
module.exports.decode = decode;
```

### 서버 코드

최종 완성된 `server.js` 파일은 아래와 같습니다.

```js
var express = require('express');
var mongoose = require('mongoose');
var bijective = require('./bijective.js');
var Urls = require('./models');

mongoose.connect('mongodb://localhost/url-shortener');

var app = express();
app.use(express.static('public'));

app.get('/url/:longUrl', function(req, res){

  var shortUrl = '';

  Urls.findOne({url: req.params.longUrl}, function (err, doc){
    if (doc){
      res.send({'key': bijective.encode(doc._id)});
    } else {

      var newUrl = Urls({
        url: req.params.longUrl
      });

      newUrl.save(function(err) {
        if (err) console.log(err);

        res.send({'key': bijective.encode(newUrl._id)});
      });
    }

  });

});

app.get('/:key', function(req, res){

  var id = bijective.decode(req.params.key);

  Urls.findOne({_id: id}, function (err, doc){
    if (doc) {
      res.redirect(doc.url);
    } else {
      res.redirect("/");
    }
  });

});


app.listen(3000, function () {
  console.log('Example app listening on port 3000!')
});

```

이제 실행해 보면서 동작 여부를 확인 합니다.

```
$ node server.js
```



