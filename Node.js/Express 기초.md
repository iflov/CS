## Express의 require()와 app

require는 express module을 만들고, express application 을 만든다.

app은 전통적으로 이름지어진 객체이다.

이 객체는 http 요청/미들웨어 구성/ HTML views 랜더링 / 템플릿 엔진 일시적으로 주기억장치에 기억시키기(registering)/응용 프로그램의 동작 방식(예: 환경 모드, 경로 정의의 대소문자 구분 여부 등)을 제어하는 응용 프로그램 설정 수정 등의 기능이 있다.

## require()

`require()`모듈은 Node의 기능 을 사용하여 다른 코드로 가져올 수 있는 JavaScript 라이브러리/파일입니다 . _Express_ 자체는 우리가 _Express_ 애플리케이션에서 사용하는 미들웨어 및 데이터베이스 라이브러리와 마찬가지로 모듈입니다.

아래 코드는 _Express_ 프레임워크를 예로 사용하여 이름별로 모듈을 가져오는 방법을 보여줍니다. 먼저 `require()`모듈의 이름을 문자열( `'express'`)로 지정하고 반환된 객체를 호출하여 [Express 애플리케이션](https://expressjs.com/en/4x/api.html#app) 을 생성하는 함수를 호출합니다 . 그런 다음 응용 프로그램 객체의 속성과 기능에 액세스할 수 있습니다.

```jsx
var express = require('express');
var app = express();
```

## module.export

한 번에 하나의 속성을 빌드하는 대신 전체 개체를 하나의 할당으로 내보내려면 `module.exports`을 이용해 할당합니다(이 작업을 수행하여 내보내기 개체의 루트를 생성자 또는 다른 함수로 만들 수도 있음).

## app.listen(3000,()⇒{})

'3000' 포트에서 서버를 시작하고 콘솔에 로그 주석을 인쇄합니다. 서버가 실행 중일 `localhost:3000`때 브라우저에서 로 이동하여 반환된 예제 응답을 볼 수 있습니다.

## route

route는 HTTP 동사(GET, POST, PUT, DELETE 등), URL 경로/패턴 및 해당 패턴을 처리하기 위해 호출되는 함수를 연결하는 익스프레스 코드의 섹션이다.

express.Router() 미들웨어를 사용하면 사이트의 특정 부분에 대한 경로 처리기를 함께 그룹화하고 공통 경로 접두사를 사용하여 액세스할 수 있습니다. 도서관 관련 루트는 모두 "카탈로그" 모듈에 보관하고, 사용자 계정이나 기타 기능을 처리하기 위한 루트를 추가하면 별도로 그룹화할 수 있습니다.

## orm을 사용하는 이유

ORM"(Object Relational Mapper)을 통해 데이터베이스에 간접적으로 액세스하는 것입니다. 이 접근 방식에서는 데이터를 "객체" 또는 "모델"로 정의하고 ORM은 이를 기본 데이터베이스 형식에 매핑합니다. 이 접근 방식은 개발자가 계속해서 데이터베이스 의미가 아닌 JavaScript 개체의 관점에서 생각할 수 있고 들어오는 데이터의 유효성 검사 및 검사를 수행할 수 있는 분명한 장소가 있다는 이점이 있습니다.

# app.use / app.method

# **🎯 Node.js Middleware란?**

- 미들웨어는 Express.js 동작의 핵심이다.
- HTTP 요청과 응답 사이에서 단계별 동작을 수행해주는 함수이다.

## **📝 미들웨어 동작 원리**

![](https://blog.kakaocdn.net/dn/dYDqh6/btrmYckPRpO/zePmyk55RKMoSX5WTQWgqK/img.jpg)

- Express.js의 미들웨어는 HTTP 요청이 들어온 순간부터 **순차적으로** 시작이 된다.
- 미들웨어는 **HTTP 요청과 응답 객체를 처리하거나, 다음 미들웨어를 실행**할 수 있다.
- **HTTP 응답이 마무리될 때까지** 미들웨어 동작 사이클이 실행된다.

## **📝 Route Handler**

- Route Handler도 미들웨어의 한 종류이다. **라우팅 함수(get, post, put, delete 등)에 적용된** 미들웨어이다.
- 일반적인 미들웨어와는 다르게 **path parameter를 사용**할 수 있다.

## **📝 미들웨어의 작성법**

- req, res, next를 가진 함수를 작성하면 해당 함수는 미들웨어로 동작할 수 있다.
- `req` : HTTP **요청을** 처리하는 객체
- `res` : HTTP **응답을** 처리하는 객체
- `next` : 다음 미들웨어를 실행하는 함수

```jsx
const logger = (req, res, next) => { 
	 console.log(`Request ${req.path}`);  
		next();
};

const auth = (req, res, next) => {
  if (!isAdmin(req)) {    
		next(new Error("Not Authorized"));
    return;  }  
		next();
};
```

- req, res, next를 인자로 갖는 함수를 작성하면 미들웨어가 된다.
- req, res 객체를 통해 HTTP 요청과 응답을 처리하거나 next 함수를 통해 다음 미들웨어를 호출해야 한다.
- **next() 함수가 호출되지 않으면 미들웨어 사이클이 멈추기 때문에 주의해야 된다.**
- 미들웨어는 적용되는 위치에 따라서 애플리케이션 미들웨어, 라우터 미들웨어, 오류처리 미들웨어로 분류가 가능하다.
- 필요한 동작 방식에 따라 미들웨어를 적용할 위치를 결정해야 된다.

## **📝 애플리케이션 미들웨어**

```jsx
app.use((req, res, next) => {
  console.log(`Request ${req.path}`);  
	next();// 1
});

app.use(auth);// 2
app.get("/", (req, res, next) => {
  res.send("Hello Express");// 3});
```

- use나 http method 함수를 사용하여 미들웨어를 연결할 수 있다.
- 미들웨어를 모든 요청에 공통적으로 적용하기 위한 방법이다.
- http 요청이 들어온 순간부터 **적용된 순서대로 동작한다.**

## **📝 라우터 미들웨어**

```jsx
router.use(auth);// 3
router.get("/", (req, res, next) => {
  res.send("Hello Router");});// 4
//-----------------------------------------
app.use((req, res, next) => {
  console.log(`Request ${req.path}`);  
	next();// 1
});
app.use("/admin", router);// 2
```

- router 객체에 미들웨어가 적용되는 것 외에는 애플리케이션 미들웨어와 사용 방법은 동일하다.
- 특정 경로의 라우팅에만 미들웨어를 적용하기 위한 방법이다.
- app 객체에 라우터가 적용된 이후로 순서대로 동작한다.

## **📝 미들웨어 서브스택**

```jsx
app.use(middleware1, middlware2, ...);

app.use("/admin", auth, adminRouter);

app.get("/", logger, (req, res, next) => {
  res.send("Hello Express");
});
```

- 여러 개의 미들웨어를 동시에 적용할 수 있다.
- 주로 한 개의 경로에 특정해서 미들웨어를 적용하기 위해 사용한다.
- 전달된 인자의 순서 순으로 동작한다.

## **📝 오류처리 미들웨어**

- 일반적으로 가장 마지막에 위치한다.
- err, req, res, next 네 가지 인자를 가지며, 앞선 미들웨어에서 next 함수에 인자가 전달되면 실행된다.

```jsx
app.use((req, res, next) => 
	{  if (!isAdmin(req)) {
    next(new Error("Not Authorized")); // 1 중간은 건너뛰고 마지막 오류처리 미들웨어로 실행된다.    
		return;  }
  next();});
app.get("/", (req, res, next) => {
  res.send("Hello Express");
});

app.use((err, req, res, next) => { // 2, err 인자  
res.send("Error Occurred");});
```

- `next(new Error());`
- 가장 아래 적용된 err, req, res, next를 인자로 갖는 함수가 오류처리 미들웨어이다.
- 이전에 적용된 미들웨어 중 next에 인자를 넘기는 경우 중간 미들웨어들은 뛰어넘고 오류처리 미들웨어가 바로 실행된다.

## **📝 함수형 미들웨어**

- 하나의 미들웨어를 작성하고, **작동 모드를 선택하면서 사용**하고 싶을 경우 미들웨어를 함수형으로 작성하여 사용할 수 있다.
- Ex) API별로 사용자의 권한을 다르게 제한하고 싶은 경우. 사용자 권한을 제한하는 미들웨어를 1개만 작성하고 사용자의 권한을 미들웨어의 인자로 넘기면서 체크할 수 있도록 작성할 수 있다.

```jsx
const auth = (memberType) => {
  return (req, res, next) => {// 미들웨어 함수 리턴
    if (!checkMember(req, memberType)) {
      next(new Error(`member not ${memberType}`));
      return;
    }
    next();
  };
};

app.use("/admin", auth("admin"), adminRouter);
app.use("/users", auth("member"), userRouter);
```

- auth 함수는 **미들웨어 함수를 반환하는 함수이다.**
- auth **함수 실행 시 미들웨어의 동작이 결정**되는 방식이다.
- 일반적으로 **동일한 로직에 설정값만 다르게** 미들웨어를 사용하고 싶을 경우에 활용된다.

## **🏷 요약**

- 미들웨어는 **HTTP 요청과 응답 사이**에서 동작하는 **함수**이다.
- **req, res, next**를 인자로 갖는 함수는 미들웨어로 동작할 수 있다.
- **app 혹은 router 객체에 연결**해서 사용 가능하다.
- **next에 인자**를 넘기는 경우 **오류처리 미들웨어**가 실행된다.
- 미들웨어에 값을 설정하고 싶은 경우는 **함수형 미들웨어**로 작성 가능하다.