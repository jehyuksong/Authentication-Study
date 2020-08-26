# Authentication-Study

### 보안,인증 공부를 단계별로 저장하기 위해서 만든 저장소

### Level 1 ~ Level 6 까지의 보안 수준을 정리했습니다.
### 프로젝트 완성작과는 살짝 다를 수가 있으니 각각 Commit마다 레벨별로 완성 파일을 정리해놨으니 필요하다면 참고해주세요!

# 오늘 한 일🔥

- 인증, 암호화, 로그인, 회원가입, Passport, OAuth(구글로그인) 에 대해서 공부하고 실습을 해봤다.

---

# 오늘 느낀 점🔥

- 레벨 4 단계의 보안수준까지 배웠을 때는 마냥 흥미롭고 너무 재미있었는데 `passport`가 나올 때부터 '음......... 머리가 살짝 아프네.....?' 하다가 `OAuth`가 나오면서부터 머리가 터질 뻔했다.~~(살짝 터진 것 같다.)~~
- 그래도 너무 흥미롭고 너무나 궁금했던 주제라서 너무 기분좋고 배울 수 있다는 자체로 너무 좋았다.
로그인과 회원관리가 어떻게 구현이 되는건지가 너무 막연하고 궁금했던 주제라서 그런 것 같다.

---

# 인증, 암호화

- 개인 데이터를 저장하고 엑세스를 제한하기 위해서 필요하다.
- 다양한 수준의 보안이 있다.
- 서비스와 이용자들을 전부 위험에 빠뜨리고 서비스가 망할 수도 있을 만큼 치명적이고 중요하다.

---

# Level 1 - 아이디, 비밀번호 데이터베이스 저장

- 단순히 사용자의 계정을 만들고 이메일과 비밀번호를 저장하는 것이다.

```jsx
app.post("/login", function (req, res) {
  const username = req.body.username;
  const password = req.body.password;

  User.findOne({ email: username }, function (err, foundUser) {
    if (err) {
      console.log(err);
    } else {
      if (foundUser.password === password) {
        res.render("secrets");
      }
    }
  });
});
```

- 이러한 방법으로 데이터베이스에 등록된 정보와 로그인하는 정보가 일치한다면 로그인이 되게끔 하는 기본적인 방법이다.
- 하지만 이 방법을 이용하면 데이터베이스에서 그대로 아이디와 비밀번호가 노출되게 된다. 보안이 매우 취약한 방법이라고 볼 수 있다.

---

# Level 2 - 암호화

- 무언가를 뒤섞고 바꾸는 것을 암호화라고 한다.
- 메시지를 뒤섞는 방법과 뒤섞인 메시지를 해제할 수 있는 키가 필요하다.

```jsx
$ npm i mongoose-encryption          // 암호화 모듈을 설치한다.
$ const encrypt = require("mongoose-encryption");
```

```jsx
const userSchema = new mongoose.Schema({
  email: String,
  password: String,
});
```

- 만든 데이터베이스 스키마를 `new mongoose.Schema()` 를 사용해서 새로운 몽구스 스키마 객체로 만든다.

```jsx
// secret을 정의하고
const secret = "Thisisourlittlesecret.";

//userSchema 에 암호화 플러그인을 추가한다. password 만이 암호화 되게끔 설정한다.
userSchema.plugin(encrypt, { secret: secret, encryptedFields: ["password"] });
```

- 이제 새로운 이메일과 비밀번호를 등록하거나 로그인할 때 알아서 암호화되고 알아서 해독이 된다.
데이터베이스에서 확인할 때도 굉장히 복잡한 문구로 나오는 것을 볼 수 있다.
ex > `"_ct" : { "$binary" : "YRdE1yDBN15wQadhPU8X9c+Oshc/f72VzdgHIAPAQsmap9gs/UNTARtm8FwVOxPtmQ==", "$type" : "00" },` 이처럼 말이다.
- 하지만 이 또한 문제가 있다. 자바스크립트 코드를 확인하는 것은 어렵지가 않은데, `secret` 인 `Thisisourlittlesecert.` 을 확인한다면 설정한 모든 패스워드를 해독할 수가 있기 때문이다.

---

### 환경 변수를 사용하여 비밀을 유지하는 방법 - dotenv

```jsx
$ npm i dotenv            // dotenv 모듈을 설치한다.
```

```jsx
require("dotenv").config();           // 상수로 정의하지 않고 사용하는데
																			// 이를 코드 가장 위에 놓는 것이 매우 중요하다.
```

```jsx
// 루트 폴더에 .env 파일을 만들고 연다.
// 아까 위에서 저장했던
const secret = "Thisisourlittlesecret.";
// 이 부분을 서버파일에서 삭제하고 .env 안에 아래와 같이 수정해서 넣는다.
SECERT=Thisisourlittlesecret.
// 자바스크립트 파일이 아니기 때문에 const도 필요없고, 따옴표, 세미콜론도 필요없다.
// 또한 암묵적으로 = 사이에는 공백이 없게 쓰는 것이 룰이다.
// 마지막 . 은 코드의 일부이기 때문에 남겨둔다.
```

```jsx
// 그리고 위에서 사용했던
userSchema.plugin(encrypt, { secret: secret, encryptedFields: ["password"] });
// 이 부분을 아래와 같이 수정한다.
userSchema.plugin(encrypt, { secret: process.env.SECRET, encryptedFields: ["password"] });
```

- `.env` 안에 저장된 내용들을 사용할 때는 `process.env` 에 이어서 참조할 수 있다.
- `.env` **파일은 공개되면 안되기 때문에** `.gitignore` **에 추가해주는 것이 매우 중요하다.**
- **Github에는 `commit history`가 남기 때문에 처음 `commit`을 하기 전에 `암호화`해야 할 것들을 설정해주고 `.gitignore`까지 정리를 하고 올려야 한다.**

---

# Level 3 - 해싱

- 이 전의 방법들은 가장 큰 문제점은 **암호화**를 위한 **암호화 키**가 필요하다는 것이다.
- **암호화 키가 어떤 방법으로든 공개가 되면 얼마든지 모든 정보가 노출될 수 있다.**
- 이런 문제점들을 **해싱**으로 해결할 수 있다.
- 비밀번호를 입력한다고 가정했을 때 이를 **해시 함수라는 것을 사용하여 암호를 해시로 변환**하고 해당 해시를 우리 데이터베이스에 저장한다.
- `해시`는 **암호를 해시로 바꾸는 것을 쉽고 빠르게** 하지만 **해시를 다시 암호로 바꾸는 것은 거의 불가능**하다고 볼 수 있기에 좋은 방법이다.
- 만약 암호를 해시로 바꾸는 것이 1ms가 걸리고 해시를 다시 암호로 바꾸는 데 2년이 걸린다고 한다면 해커는 시도하지 않을 것이다.

---

### 그렇다면 어떻게 암호가 맞는지 확인을 할까?

- 비밀번호를 등록할 때 **해시함수를 이용해서 데이터베이스에 해시로 저장**을 한다.
- 그리고 로그인을 할 때 **입력한 비밀번호를 해시함수를 이용해서 해시**로 만들고 **데이터베이스에 저장된 해시와 비교**를 해서 동일하다면 인증이 되는 형태이다.
- 이 프로세스의 어느 시점에서도 비밀번호를 일반 텍스트로 저장하지 않는다.

---

### 해시 사용하기

```jsx
$ npm i install md5             // md5 모듈 설치

const md5 = require('md5')      // md5 모듈 불러오기
```

```jsx
// 이전에 사용했던 mongoose-encryption에 관련된 내용들은 전부 지워준다.
```

```jsx
// 회원가입을 하는 코드를 아래와 같이 변경해준다.
app.post("/register", function (req, res) {
  const newUser = new User({
    email: req.body.username,
    password: md5(req.body.password),          // md5로 감싸준다.
  });
```

- 회원가입을 다시 해보면 데이터베이스에 해시의 형태로 암호가 저장되는 것을 확인할 수 있다.
- 여기서 **매우 중요한 포인트**는 **동일한 문자열에서 생성된 해시는 항상 동일**하다는 것이다.

```jsx
// 예를 들어
console.log(md5(12345));    // 7cfdd07889b3295d6a550914ab35e068
```

- 위와 같이 `12345` 에 대한 **해시값**은 언제나 `7cfdd07889b3295d6a550914ab35e068` 이다.
- 이래서 비밀번호를 어렵게 만들라고 하는건가보다. 쉬운 것들은 미리 해시함수를 생각해서 대입해볼 수가 있으니까.

---

### 로그인할 때의 비밀번호도 해시함수로 바꿔야한다.

```jsx
app.post("/login", function (req, res) {
  const username = req.body.username;
  const password = md5(req.body.password);       // 해시값으로 바꿈

  User.findOne({ email: username }, function (err, foundUser) {
    if (err) {
      console.log(err);
    } else {
      if (foundUser.password === password) {
        res.render("secrets");
      }
    }
  });
});
```

- 이제 등록할 때의 비밀번호 해시값과 로그인할 때의 비밀번호 해시값의 비교를 통해 인증이 이루어진다.
- **이 단계까지 되면 대부분의 웹사이트 보안 수준이라고 볼 수 있다.**

---

# 해킹

- 해킹을 하기 위한 목적은 아니지만 해킹이 어떤 식으로 진행되는 지를 알면 웹사이트를 만들 때 보안을 어떻게 유지해야 하는 지에 대해서 알 수 있다.

### 왜 대기업들도 해킹을 당할까?

- 같은 해시의 정보가 같다는 것은 같은 비밀번호를 사용하고 있다는 뜻이다.
- 123456,qwerty,111111 등의 자주 쓰는 해시 테이블을 만들고 이 해시값들로 비밀번호를 조회할 수가 있다.
- 모든 문자,숫자,특수기호 등의 경우의 수를 다 돌아가면서 해시 테이블을 만들 수도 있고, 자주 사용되는 우선순위가 높은 비밀번호들의 해시테이블을 이용해서 만든다. 이 해시테이블로 계산을 돌려보는 것이다.
- 일반적인 알파벳 6글자의 비밀번호를 해독하는데 일반 컴퓨터 성능으로도 3초면 해독할 수 있다.
하지만 알파벳 12글자의 비밀번호라면 일반 컴퓨터로는 31년, 꽤 빠른 GPU라면 2년의 시간이 걸린다.
- 또한 숫자와 특수문자를 섞었을때 6글자라면 일반 컴퓨터로 2시간, 꽤 빠른 컴퓨터로 6분이면 해독할 수 있다. 여전히 오래 걸리지않는다.
- **그렇기 때문에 가장 보안 수준이 높은 것은 우선은 긴 비밀번호이다. 수학적 공식에 의해 만들어진 원리를 생각하면 숫자가 하나가 늘어날 때마다 경우의 수는 기하급수적으로 늘어난다.**
- [http://password-checker.online-domain-tools.com/](http://password-checker.online-domain-tools.com/) 비밀번호를 해독하는데 얼마나 걸리는지에 대한 조회를 해볼 수 있는 사이트이다. 재미삼아 해볼 수 있다.

---

# Level 4 - 해싱 , 솔트

- 해시 테이블 크랙을 방지할 수 있는 방법이다.

### Salt(솔트)

- 비밀번호를 생성할 때 `123456` 이라고 입력을 했다면 `123456` + `임의의 값` 을 더해서 해시값으로 바꿔주는 것을 의미한다.
- 사용자가 지정한 암호가 아무리 간단하더라도 salt가 더 복잡하게 만들어줄 수 있다.
- 또한, **동일한 문자열의 비밀번호**라도 **임의의 솔트**가 더해짐으로 인해 모두 **다른 해시값**을 가질 수가 있다.
- 데이터베이스에는 솔트와 해시값이 저장되게 되는 것이다.

### Salt Rounds

- 솔트를 얼마나 사용하느냐에 따른 정의
- `입력한 비밀번호 + 솔트 -> 해시값1`   을 만든다. (솔트라운드1)
- `해시값1 + 솔트 -> 해시값2` 을 만든다. (솔트라운드2)
- `해시값3 + 솔트 -> 해시값3` 를 만든다. (솔트라운드3)
- 무어의 법칙에 의해서 컴퓨터의 성능,속도가 매년 빨라지고 있기 때문에 솔트라운드도 올릴 필요가 있다.

### Bcrypt

- 해싱과 솔트를 해주는 모듈이다.

---

# Bcrypt 사용하기

- Bcrypt를 사용하기 위해서는 node 안정적인 버전이 필요하다.

```jsx
// nvm 을 설치해준다.
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

- 터미널을 껐다가 켜야한다.

```jsx
$ nvm install 12.18.3          // node 안정화 버전으로 바꿔준다.
$ npm i bcrypt                 // bcrypt 를 설치해준다.
```

```jsx
const bcrypt = require("bcrypt");    // bcrypt
const saltRounds = 10;           // 솔트라운드를 설정해준다.
```

- 2020년 기준으로는 솔트라운드 10이면 적당하지만, 시간이 지남에 따라 올라갈 것이다.
하지만 **솔트라운드가 올라감에 따라서 컴퓨터가 처리하는 시간도 오래 걸림**을 인지해야 한다.

---

### Bscrypt 함수로 비밀번호 등록하기

```jsx
app.post("/register", function (req, res) {

	// 첫번째 인자(req.body.password)를 saltRounds 만큼 돌리고 hash로 반환한다.
  bcrypt.hash(req.body.password, saltRounds, function (err, hash) {
    const newUser = new User({
      email: req.body.username,
      password: hash,              // 반환된 hash를 입력해줌
    });

    newUser.save(function (err) {
      if (err) {
        console.log(err);
      } else {
        res.render("secrets");
      }
    });
  });
});
```

- 이제 새로운 정보를 입력하고 회원가입을 하고 나면 데이터베이스에서 결과값으로 나온 매우 복잡한 해시값을 확인할 수 있다.

---

### 로그인할 때 비밀번호 비교하기

```jsx
app.post("/login", function (req, res) {
  const username = req.body.username;
  const password = req.body.password;

  User.findOne({ email: username }, function (err, foundUser) {
    if (err) {
      console.log(err);
    } else {
      if (foundUser) {
        bcrypt.compare(password, foundUser.password, function (err, result) {
          if (result === true) {
            res.render("secrets");
          }
        });
      }
    }
  });
});
```

- 입력한 `password = req.body.password` 로 hash를 돌린 값이 데이터베이스에 입력되어있는 `foundUser.password` 와 비교한  `result` 를 제공하는 함수를 사용해서 그것이 `true` 라면 비밀번호가 일치한다는 것이다.
- 즉 `첫번째 매개변수의 값 === 데이터베이스의 값 ? result = true : false` 인것이다.

---

# Level 5 - 쿠키, 세션

### 쿠키

- 브라우저가 서버에 요청을 보낸다.
- 서버가 브라우저에 응답과 함께 HTTP 헤더에 쿠키를 넣어 보낸다.
- 브라우저가 같은 요청을 할 때 HTTP 헤더에 쿠키를 넣어서 요청한다.
- 서버가 이전 기록을 확인하고 변경해야 될 부분이 있으면 쿠키를 업데이트하고 다시 HTTP 헤더에 쿠키를 넣어서 응답한다.

---

### 세션

- 쿠키를 기반으로 하지만, 정보를 브라우저가 아닌 서버가 관리한다.
- 클라이언트를 구분하기 위해 고유한 세션 ID를 부여하고 웹브라우저가 서버에 접속해서 브라우저를 종료할 때까지 인증을 유지한다.
- 보안 면에서 쿠키보다 더 좋지만, 사용자가 많아지면 서버 메모리를 많이 차지한다.
- 로그인처럼 보안상 중요한 작업을 수행할 때 많이 사용한다.

---

### 쿠키와 세션의 차이

- 세션도 쿠키를 사용하지만, **정보가 저장되는 위치**가 **쿠키는 브라우저**, **세션은 서버측**이다.
- 세션이 세션id만 저장하고 서버에서 처리하기 때문에 **보안**은 더 좋지만, 서버의 처리가 필요한 만큼 **요청속도**가 더 느리다.
- `쿠키`는 **브라우저를 종료해도 남아있을 수 있고**, 만료기간을 넉넉하게 잡으면 쿠키를 삭제할 때까지 유지되지만, `세션`은 만료기간을 정할 수 있는 건 같지만, **브라우저를 종료하면 기간과 상관없이 삭제**된다.

---

### Passport.js를 통해 쿠키 및 세션 추가하기

```jsx
// passport     passport-local       passport-local-mongoose     express-session
$ npm i passport passport-local passport-local-mongoose express-session

```

- 4개의 모듈을 설치한다.

```jsx
const session = require("express-session");
const passport = require("passport");
const passportLocalMongoose = require("passport-local-mongoose");

// 3줄의 코드로 모듈을 불러온다.
```

```jsx
// 사용 설정을 해준다.
app.use(
  session({
    secret: "Our little secret.",
    resave: false,
    saveUninitialized: false,
  })
);

app.use(passport.initialize());
app.use(passport.session());
```

```jsx
// 스키마 아래에 넣어준다.
userSchema.plugin(passportLocalMongoose);  // db 플러그인설정을 해준다. 
																					 // 스키마는 자바스크립트 객체여야한다.
```

```jsx
// 모델 아래에 넣어준다.
passport.use(User.createStrategy());

passport.serializeUser(User.serializeUser());       // 쿠키를 만드는 것
passport.deserializeUser(User.deserializeUser());   // 쿠키를 불러오는 것
```

- 이제 서버를 실행해줬을 때 에러가 발생한다면 mongoose.connect 아래에 `mongoose.set("useCreateIndex", true);` 를 입력해준다.

---

### 회원가입

```jsx
app.get("/secrets", function (req, res) {
  if (req.isAuthenticated()) {
    res.render("secrets");
  } else {
    res.redirect("/login");
  }
});

app.post("/register", function (req, res) {
  User.register({ username: req.body.username }, req.body.password, function (err, user) {
    if (err) {
      console.log(err);
      res.redirect("/register");
    } else {
      passport.authenticate("local")(req, res, function () {
        res.redirect("/secrets");
      });
    }
  });
});
```

### 로그인

```jsx
app.post("/login", function (req, res) {
  const user = new User({
    username: req.body.username,
    password: req.body.password,
  });

  req.login(user, function (err) {
    if (err) {
      console.log(err);
    } else {
      passport.authenticate("local")(req, res, function () {
        res.redirect("/secrets");
      });
    }
  });
});
```

- 위와 같이 만들어주고 새로 회원 등록을 하고 데이터베이스를 들어가보면 **해시와 솔트값까지 자동으로 만들어진다.**
- 로그인을 해보면 쿠키가 추가되어서 서버가 다시 시작되거나 브라우저를 종료하기 전까지 로그인 상태를 유지한다.

---

# Level 6 - OAuth(Open Authorisation)

- 타사 웹사이트의 정보에 액세스하는 것
- Google, Facebook의 친구, 이메일 정보등이 있지만 `인증`의 측면에서 봤을 때의 이점은 **암호를 안전하게 관리하는 작업을 이러한 회사에 위임**하는 것이다.
- 사용자에게 로그인할 때마다 Facebook, Google, Kakao, Naver 등에 로그인을 하도록 요청하면 이러한 기업들에서 자체 보안 방법을 사용하여 인증한다.
- 인증이 완료되면 인증완료를 알려주고, 우리의 책임과  단계가 굉장히 간략해진다.

---

### 왜 OAuth를 사용하는가?

- **세분화된 수준의 액세스 권한을 부여한다.** 예를 들어 내 앱에는 Facebook 프로필과 이메일 주소만 필요하다고 말할 수도 있다. 혹은 친구 목록만을 원할 수도 있다.
- **읽기 전용 또는 읽기 및 쓰기 액세스를 허용한다.** 예를 들어 WordPress가 이 사용자의 계정으로 Facebook에 게시할 수 있도록 하는 경우가 있다. 그럴 때 엑세스 요청을 한다.
- **액세스 요청을 하는 사용자가 어느 시점에서든 액세스를 취소할 수 있다.** 예를 들어 페이스북, 구글과 연동된 게임 등을 할 때 엑세스 권한을 계정에 부여하거나 해제할 수 있는 것이다.

---

### OAuth 사용 단계

1. Facebook, Twitter, Google에 우리가 사용을 하겠다고 등록을 해야한다. 우리는 그럼 앱ID를 얻는다.
2. 사용자가 우리 사이트를 방문해서 인증을 원할 때 페이스북으로 로그인, 구글로 로그인 등의 옵션을 제공받는다.
3. 옵션을 클릭하면 페이스북, 구글 웹사이트로 이동되면서 로그인하고 인증을 받는다.
4. 그 이후에 웹사이트에서 요청하는 권한을 검토한다. 프로필과 이메일을 원하는 등의 권한.
5. 권한을 부여한 뒤에 우리 사이트로 Facebook이 인증토큰을 보내주고 이를 통해 로그인이 된다.
6. 만약 여기서 **액세스토큰을 받으면 데이터베이스에 저장까지 가능하다. 이것은 나중에 정보를 요청하려는 경우에 사용할 수 있는 토큰**이다. 액세스 토큰이 인증토큰보다 훨씬 오래 유효한다.
- 인증코드 - 1회 입장권, 성공적으로 로그인한 사용자를 인증하는데 필요,
- 액세스토큰 - 1년 프리 패스 와 같은 개념, 여기에 저장된 정보에 액세스하는데 사용

---

### Google OAuth 사용하기

```jsx
$ npm i passport-google-oauth20      // google oauth 2.0 을 다운받는다.
```

```jsx
https://console.developers.google.com/apis   // 새 프로젝트 만들기
```

- 새 프로젝트를 만들고 **OAuth 동의 화면 구성**을 완료한다.
- `사용자 인증 정보` - `OAuth 클라이언트 ID 만들기`를 누른다.
- `승인된 자바스크립트 출처` 에 `[http://localhost:3000](http://localhost:3000)` 을 입력하고,
`승인된 리디렉션 URI` 에 `[http://localhost:3000/auth/google/secrets](http://localhost:3000/auth/google/secrets)` 를 입력한다.
- 완료가 되면 `클라이언트ID와 PW` 를 알려준다. 그 값을 아래와 같이 **.env** 에 저장해둔다.

```jsx
CLIENT_ID=클라이언트ID
CLIENT_SECRET=클라이언트PW
```

```jsx
// Google OAuth를 불러온다.
const GoogleStrategy = require("passport-google-oauth20").Strategy;
```

```jsx
$ npm i mongoose-findorcreate            // 이후 코드에서 사용할 기능을 위해 설치한다.

const findOrCreate = require("mongoose-findorcreate");    // 모듈을 불러오고

userSchema.plugin(findOrCreate);        // 플러그인 사용을 정의한다.
```

---

```jsx
passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.CLIENT_ID,
      clientSecret: process.env.CLIENT_SECRET,
      callbackURL: "http://localhost:3000/auth/google/secrets",
      userProfileURL: "https://www.googleapis.com/oauth2/v3/userinfo",
    },
    function (accessToken, refreshToken, profile, cb) {
      console.log(profile);

      User.findOrCreate({ googleId: profile.id }, function (err, user) {
        return cb(err, user);
      });
    }
  )
);
```

```jsx
   <div class="col-sm-4">
      <div class="card social-block">
        <div class="card-body">
          <a class="btn btn-block" href="/auth/google" role="button">
            <i class="fab fa-google"></i>
            Sign Up with Google
          </a>
        </div>
      </div>
    </div>
```

```jsx
   <div class="col-sm-4">
      <div class="card social-block">
        <div class="card-body">
          <a class="btn btn-block" href="/auth/google" role="button">
            <i class="fab fa-google"></i>
            Sign Up with Google
          </a>
        </div>
      </div>
    </div> 
```

- 프론트엔드에 이와 같이 구글로그인이 가능하게 해주는 버튼을 생성한다.
- 또한, 인터넷에 `Social Buttons` 라고 검색하면 소셜로그인 버튼에 대한 부트스트랩 디자인을 제공한다.

```jsx
app.get("/auth/google", passport.authenticate("google", { scope: ["profile"] }));
```

- `/auth/google` 에 대한 GET요청 설정을 해준다.
- 이제 웹사이트에서 `Sign Up with Google` 버튼을 누르면 google 로그인 화면이 뜨지만 아직 로그인이 되지는 않고 에러가 뜬다. 아직 사용자를 인증한 후 사용자를 리디렉션하는 곳이 추가되지 않았기 때문이다.

```jsx
app.get("/auth/google/secrets", passport.authenticate("google", { failureRedirect: "/login" }), function (req, res) {
  // Successful authentication, redirect home.
  res.redirect("/secrets");
});
```

- 리디렉션될  URI에 대한 정보를 설정해준다.

```jsx
passport.serializeUser(function (user, done) {
  done(null, user.id);
});
passport.deserializeUser(function (id, done) {
  User.findById(id, function (err, user) {
    done(err, user);
  });
});
```

- 아까 위에서 직렬화, 역직렬화 해주던 코드를 위와 같이 바꿔준다.
- 이게 `구글로그인` 을 하면 로그인이 되면서 데이터베이스는 오직 `앱ID` 만이 저장된다.
- 하지만 여러분 구글로그인을 해보면 계속 데이터베이스에 생성이 되는 것을 볼때 이는 구글ID와 연결되지 않는 것을 확인할 수 있다. 그렇기에 코드를 수정해야한다.

```jsx
const userSchema = new mongoose.Schema({
  email: String,
  password: String,
  googleId: String,      // googleId = Profile.id이다.
});
```

- 만들어둔 스키마에 `googleId: String` 을 추가해주면서 불러온 정보의 `Profile.id` 값을 데이터베이스에 저장한다.
- 이제 로그인할 때마다 유저정보에 데이터베이스가 계속 생성되지 않고, 로그인 기록이 있으면 데이터베이스에서 `googleId` 값을 비교해서 로그인된다.
- **이로써 우리는 그들의 비밀번호를 알고 관리할 필요도 없으며, 수 많은 구글의 엔지니어들이 그들의 정보를 보호하기 위해 노력해줄 것이다. 우리는 그저 가져온 ID값만 알면서 인증을 진행하고 우리가 필요할 때 검색하는 것만 하면 된다.**

---

### req.user

- `passport` 를 이용할 때 post 요청을 보낸 사람이 누군인지 `req.user` 를 통해서 쉽게 알 수 있다.

### 사용자가 올린 글을 사용자의 데이터베이스에 저장을 하려면?

```jsx
const userSchema = new mongoose.Schema({
  email: String,
  password: String,
  googleId: String,
  secret: String,
});
```

- 우선 스키마에 `secret` 이라는 값을 추가해서 스키마를 수정해줬다.

```jsx
app.post("/submit", function (req, res) {
  const submittedSecret = req.body.secret;

  console.log(req.user.id);
  User.findById(req.user.id, function (err, foundUser) {
    if (err) {
      console.log(err);
    } else {
      if (foundUser) {
        foundUser.secret = submittedSecret;
        foundUser.save(function () {
          res.redirect("/secrets");
        });
      }
    }
  });
});
```

- 텍스트를 입력을 해서 올리면 foundUser의 secret에 글이 저장된다.

---
