
# Nest 초기세팅

[Documentation | NestJS - A progressive Node.js framework](https://docs.nestjs.com/)

nest공식문서에 있는 순서 그대로 진행할 것이다~

vscode에서 빈 폴더(디렉토리) 를 만들어서 터미널을 켜준다 그 뒤에

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/75025776-8a44-4b44-bd70-00802a81673f/Untitled.png)

1. npm i -g @nestjs/cli

→ nest js설치 명령어

1. nest new project-name

→ nestjs는 프레임위크인 만큼 정해진 틀이 있다 그 틀을 가져오는 명령어이다

→ project-name에 프로젝트이름을 임의로 작성해준다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/78b2eb54-79eb-4078-a2a6-daec5e2f2d65/Untitled.png)

nest new project-name을 진행했다면,

밑에서 두번째줄 코드처럼 어떤 package manager를 사용할지 나오는데 npm을 선택해준다!

→ npm을 선택하는 이유는 공식문서에서도 npm으로 설명을 하기 때문에!

설치가 완료되면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b5e4e45f-b7f2-4603-9352-b9ce6f83310c/Untitled.png)

위와같은 프레임들이 나온다!

src와 test안에 파일들을 보면

|`app.controller.ts`|단일 경로가 있는 기본 컨트롤러.||
|---|---|---|
|`app.controller.spec.ts`|컨트롤러에 대한 단위 테스트입니다.||
|`app.module.ts`|응용 프로그램의 루트 모듈입니다.||
|`app.service.ts`|단일 메서드를 사용하는 기본 서비스입니다.||
|`main.ts`|핵심 기능 `NestFactory`을 사용하여 Nest 애플리케이션 인스턴스를 생성하는 애플리케이션의 엔트리 파일입니다.||

spec.ts는 controller를 test하기 위한 것이고

test 폴더 안에있는것 또한 테스트를 위한 폴더이다!

우리가 주로 진행하게 될 곳은 src안이 될것이다!

## Nest에서의 Controller

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ebb6a0bd-4cc3-447c-9293-9a3263332042/Untitled.png)

express의 route감성이다

express에서 사용한

```tsx
router.get('/' , ~~~~ )
```

요것을 nest에서는

```tsx
@Get('/') === @Get() // 이 2개는 같다
getHello():string {
	return ~~~~~~~~
}
```

이런식으로 사용한다!

여기서 **@를 데코레이터** 라고 한다!

데코레이터는 함수나 클래스에 기능을 첨가해주는것이다!

→ 기능을 첨가해준다! ( 재사용성 극대화 )

→ 데코레이터는 아래줄과 공간이 있으면 안된다 ( 공간 있으면 error 발생 )

```tsx
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()

// error 발생!!!!!! <- 데코레이터에서는 빈공간이 있으면 안된다!

  getHello(): string {
    return this.appService.getHello();
  }
}
```

→ getHello() 함수에다가 기능을 첨가한다고 생각하면 된다!

위 코드에서 appService는 같은 디렉토리에 있는 app.service.ts 를 의미한다

@Controller()의 인자 안에 경로를 넣어주면 우리가 진행했던 express layered pattern에 index.js처럼 엔드포인트의 대분류를 지정할 수 있다.

→ @Controller(’class’)

@get() 인자 안에 경로를 넣어주면 우리가 진행했떤 express layered pattern에 route.js 처럼 엔드포엔트에 소분류를 지정할 수 있다.

→ @Get(’/record’)

getHello의 우리가 express의 layered pattern에서 사용한것 처럼 ( req , res ) 느낌을 사용할 수 있는데

```tsx
import { Body, Controller, Get, Param, Req } from '@nestjs/common';
import { AppService } from './app.service';

@Controller('/say') //index
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get('/hello/:id') // productRoute
  getHello(@Req() req: Request, @Body() body, @Param() param): string {
    console.log(req);
    console.log(body);
    console.log(param);
    return this.appService.getHello();
  }
}
```

우리가 express의 controller에 getHello(req,res) ⇒ 이렇게 작성한것 처럼

getHello(@Req() req: Request, @Body() body, @Param() param): string

요렇게 작성한다 ( 원하는 함수의 매개변수에 요청과 요청에서 받는걸 선언해주는 감성 )

위코드로 통신한다면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a10da58b-8ae1-416c-88f3-a149f24380fa/Untitled.png)

응답에 Hello World 나오고

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/00684b6e-1f54-4e7d-8052-b5fe63246b4b/Untitled.png)

콘솔도 정확하게 나왔다 여기서 body param 같은 변수의 값은 객체로 노출된다

→ controller는 express의 layered pattern의 route + controller기능을 담당하는것 같다

## Nest에서 Service

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b184e3a7-283f-46f7-a469-83f74e14e601/Untitled.png)

이런 모양으로 되어있다.

app.service.ts에서는 서비스 로직이 실행되는 곳이다

여기서 return을 사용하는것은

express에서 우리가 res.status(200).json({}) 이렇게 보내준것처럼 동작한다

## Nest의 대략적인 진행 순서

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b184e3a7-283f-46f7-a469-83f74e14e601/Untitled.png)

service에서 return을 하면 이 Appservice 클래스의 getHello() 함수를 실행한

→ controller로 가서

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c92f5ba3-1601-4a18-a8cf-de615a6ea9f4/Untitled.png)

그 controller에서 controller를 가지고 있는

→app.module.ts로 이동하고

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/654d5e9d-1062-4015-9558-50eb35d17a75/Untitled.png)

이 모듈이 main.ts의

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3f7a4250-d141-4714-9d09-7f69285cc7d3/Untitled.png)

NestFactory로 오게된다.

## Nest서버 구동 방법

package.json을 보게되면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8366dfbc-12b7-44c4-8595-cab0f70ed362/Untitled.png)

요런 내용이 있다 우리가 express에서 사용했던것과 동일한 script들이 자동으로 생성되어 있다.

여기서 우리가 서버구동을 위해 사용해야 할것은

start:dev이다 ! nest를 시작하고 nodemon처럼 서버에 내용이 바뀌면 자동으로 서버 재실행? 같은 걸 하는 것으로 추측된다 하하

터미널에 **npm run start:dev**를 하여 서버를 실행해보자

→ 여기서 주의 사항은 nest new project-name하면서 정해줬던 project-name의 디렉토리 안에 nest 파일들 이 있으므로 여기로 이동해서 npm run start:dev를 실행해준다!

그 이후 postman에서

get으로 요청을 보내면~

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/191e66ba-f435-456e-9445-0700ca4ddaba/Untitled.png)

hello world 응답이 온다!

여기서 hello world는 어디서 온걸까

Nest 동작을 다시 복습해보자!

요청이 main.ts로 들어온다 → main.ts에 NestFactory는 AppModule을 사용하므로 app.module.ts로 이동 →여기서 코드는 아래에서 위로 실행되므로 controller로 이동한다 → 컨트롤러에서 @Get() 데코레이터 정의된 곳을 보면 appService의 getHello() 를 실행하므로 Service로 이동→ service에 정의된 getHello() 에 return

‘hello world’ 가 나오게 된다 ( 여기서 아까 말했던 res.status … 를 사용하지 않고 return으로 가능하다)

main.ts → app.module.ts → app.controller.ts → app.service.ts 요청이 이런식으로 가서

app.service.ts → app.controller.ts → app.module.ts → main.ts 이 방식으로 응답이 나간다!

# Eslint 와 Prettier

Vs code에서 Nest에 사용되는 확장(extension) 이 있다.

1. Eslint : JS / TS 의 코드를 검사하는 역할을 합니다!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c75c9efe-d75f-43d8-b92c-70decb8df0f3/Untitled.png)

Nest new project-name 을 하면 자동으로 생기는 것중에 해당 파일이다! 해당파일에 들어가면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b947e017-48d9-4b02-a8c7-7b5c5edd1906/Untitled.png)

→ 이런 코드가 나오는데 Eslint의 검사에 대한 규칙을 정해놓는 파일이다!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fe6a67aa-bbbc-4b03-919e-979c9447ec83/Untitled.png)

eslintrc.js안에 rule객체에 no-console : warn을 넣어주면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cad2bfb6-5bf4-4fd5-b5cb-598f5744437b/Untitled.png)

console.log 입력시 해당 사진처럼 주의 ( 노란줄 ) 이 나온다!

해당기능을 사용하는 규칙은 공식문서에 나와있습니다

[Rules - ESLint - Pluggable JavaScript Linter](https://eslint.org/docs/latest/rules/)

만약 Eslint가 적용이 안될경우가 있다 이때는

.vscode 디렉토리를 만든 다음에

settings.json파일을 넣어준다

settings.json 파일은

# setting.json

```tsx
{
  "git.ignoreLimitWarning": true,
  "eslint.codeAction.showDocumentation": {
    "enable": true
  },
  "eslint.alwaysShowStatus": true,
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",

  "eslint.workingDirectories": [
    {
      "directory": "typeorm-project",
      "changeProcessCWD": true
    },
    {
      "directory": "chattings",
      "changeProcessCWD": true
    },
    {
      "directory": "04 NestJS 개요 및 객체지향 디자인 패턴",
      "changeProcessCWD": true
    },
    {
      "directory": "tsts",
      "changeProcessCWD": true
    },
    {
      "directory": "04 NestJS 개요 및 객체지향 디자인 패턴/05, 06",
      "changeProcessCWD": true
    },
    {
      "directory": "05 NestJS 실전 프로젝트/12",
      "changeProcessCWD": true
    },
    {
      "directory": "04 NestJS 개요 및 객체지향 디자인 패턴/07",
      "changeProcessCWD": true
    },
    {
      "directory": "04 NestJS 개요 및 객체지향 디자인 패턴/07",
      "changeProcessCWD": true
    },
    {
      "directory": "05 NestJS 실전 프로젝트/02, 03",
      "changeProcessCWD": true
    },
    {
      "directory": "05 NestJS 실전 프로젝트/04",
      "changeProcessCWD": true
    },
    {
      "directory": "05 NestJS 실전 프로젝트/05",
      "changeProcessCWD": true
    },
    {
      "directory": "05 NestJS 실전 프로젝트/06",
      "changeProcessCWD": true
    },
    {
      "directory": "05 NestJS 실전 프로젝트/07, 08, 09",
      "changeProcessCWD": true
    },
    {
      "directory": "05 NestJS 실전 프로젝트/10",
      "changeProcessCWD": true
    },
    {
      "directory": "05 NestJS 실전 프로젝트/13",
      "changeProcessCWD": true
    },
    {
      "directory": "06 프로젝트 배포와 서버 운영",
      "changeProcessCWD": true
    },
    {
      "directory": "07 실시간 랜덤 채팅 웹 앱 개발/02",
      "changeProcessCWD": true
    },
    {
      "directory": "07 실시간 랜덤 채팅 웹 앱 개발/03",
      "changeProcessCWD": true
    },
    {
      "directory": "07 실시간 랜덤 채팅 웹 앱 개발/04",
      "changeProcessCWD": true
    },
    {
      "directory": "07 실시간 랜덤 채팅 웹 앱 개발/05",
      "changeProcessCWD": true
    },
    {
      "directory": "07 실시간 랜덤 채팅 웹 앱 개발/06",
      "changeProcessCWD": true
    },
    {
      "directory": "07 실시간 랜덤 채팅 웹 앱 개발/07",
      "changeProcessCWD": true
    },
    {
      "directory": "pre-dev",
      "changeProcessCWD": true
    },
    {
      "directory": "nestjs-aws-s3",
      "changeProcessCWD": true
    }
  ]
}
```

여기에 있다

1. Prettier

→ 코드를 이쁘게 정리해주는 확장입니다!

nest 를 생성할시 .prettierrc 파일이 생기는데, prettier를 어떻게 이쁘게 할지 정해주는 옵션같은 파일이비낟

예를 들면 ‘singleQuote’ 라는 객체를 true로하면 string은 전부 작은 따옴표로 바꿔주고

singleQuote 가 false일 경우 string은 전부 큰 따옴표로 바꿔주게 됩니다!

# package.json 필요사항 설명

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/97b36893-a155-4c59-96e9-90f5bb22a12a/Untitled.png)

nest에서 프로젝트를 만들면 생성되는 package.json에서 dependencies 를 살펴보고자 한다!

→ nest 작업하는데 유용한 느낌으로

@nestjs가 앞에 붙은 3개는 nestjs의 기본 라이브러리들이다!

reflect-metadata는 우리가 사용해봤던 데코레이터(@)를 사용하려 하는 라이브러리다

rimraf : rm -rf 명령어를 사용할 수 있게 하는 라이브러리

rxjs : 비동기 및 이벤트 기반의 프로그래밍을 사용하기위한 라이브러리

# Providers 와 의존성 주입

express에서 router를 보게되면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/38f8939d-10d7-4b80-ac7b-4798d370fc48/Untitled.png)

파일에서 함수를 불러와서 사용을 하였다 그런데 Nest는 그렇지 않다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c1c9d8f4-ecdf-4e1d-b90b-1e7caea3511a/Untitled.png)

클래스 내부에서 생성자로 초기화 하고 그걸 그대로 사용하였다.

app.controller.ts 에 보게되면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d845ac95-9428-4dfa-bf88-9d639aade466/Untitled.png)

해당 클래스에 생성자를 보게되면 우리가 알고 있는 생성자와 모양이 다르다

우리는 클래스에 생성자에 주로

```tsx
export class AppController {
	constructor(){
		this.appService = appService
	}
}
```

주로 이렇게 사용한다.

왜 이미지처럼 사용하는 걸까?

AppController는 ( controller ) 는 HTTP 요청을 처리하고 보다 복잡한 작업을 공급자(provider)에게 위임해야 합니다!

공급자 모듈 provider는 JavaScript의 Class 입니다!

→ app.module.ts 를 보면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea80b98a-4f3c-4230-830c-98c573e5e4fa/Untitled.png)

provider에 AppService가 들어있는것을 알 수 있다!

app.module.ts에서 controllers와 providers를 정해줌으로서

controller의 보다 복잡한 과정을 provider인 AppService로 위임함을 정해준다!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7fa0aefc-97cf-4ba8-9114-0358ad067888/Untitled.png)

provider로 사용할 경우에는 class위에 데코레이터로 @Injectable을 붙여줘야 한다( injectable : 주입 가능한 )

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d845ac95-9428-4dfa-bf88-9d639aade466/Untitled.png)

constructor에 this.~~로 사용할수도 있지만 위의 설명처럼 controller의 복잡한 과정을 provider에게 위임하기위해 위 사진처럼 사용한다

이를 의존성주입(종속성주입 DI : Dependency Injection ) 이라고 한다

의존성 주입의 간단한 설명을 보자

_**의존대상 B가 변하면, 그것이 A에 영향을 미친다.**_

- _이일민, 토비의 스프링 3.1, 에이콘(2012), p113_

→ B의 기능이 추가 또는 변경되거나 형식이 바뀌면 그 영향이 A에 미치는 걸 의미한다!

→ 그 의존관계를 외부에서 결정하고 주입하는 것이 **DI(의존관계 주입)**이다.

아래 블로그 설명이 좀 쉽게 말해준것 같아서 링크 남긴다… ( 화이팅 물론 나부터 )

[의존관계 주입(Dependency Injection) 쉽게 이해하기](https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/)

# Module과 캡슐화

module & 캡슐화을 알아보기 앞서

nest에서 module controller service를 만드는 명령어를 알아볼게요우

[Documentation | NestJS - A progressive Node.js framework](https://docs.nestjs.com/cli/usages)

공식문서 명령어사이트!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea37ef79-9755-45fc-8c45-353e441a80c5/Untitled.png)

해당형식으로 nest를 만들수 있다

- nest g mo cats

→ g는 generate의 약어

→ mo는 module의 약어

→ cats는 생성될 요소의 이름

- nest g co cats

→ co는 controller의 약어

- nest g s cats

→ s는 service의 약어

이렇게하면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e206b396-b3cd-43dd-9351-2c136f7aa924/Untitled.png)

이렇게 생긴다

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d63dd949-d725-43e4-b891-5d64ee9b3f9c/Untitled.png)

cats.module.ts를 보면 controllers / providers 에 자동으로 CatController / CatService가 추가되었다!

그럼 이전에 배웠던 controller에서 의존성 주입 및 라우팅을 해보도록보도 하자~

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/77edce2c-ce02-4dd5-bd03-2faa36cfceac/Untitled.png)

이렇게 진행하고 나면 의존성 주입이 완료되었다!

이제 app.module.ts를 보자

```tsx
@Module({
  imports: [CatsModule, UsersModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

그럼 처음에 만들었던 app.modules.ts에

자동으로 imports가 생기면서 해당 모듈들이 들어간다

→ 여기서 우리가 알 수 있는점은 여러 모듈들을 만들면 결국 main으로 다 모인다는것이다!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e31212c-6aa8-4db5-a7c6-0caa8386a144/Untitled.png)

이렇게 되면 app.module.ts 와 cats.module.ts가 연결된 것이다

( app ↔ cats 연결 )

여기서 imports된 것 들을 사용하고 싶으면 일단 controllerc에 의존성 주입을 이용하여 사용할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/600ee966-6106-4263-8976-a4eb034bffba/Untitled.png)

이런식으로 생성자의 인자에 추가로 사용할 수 있는데

모듈은 기본적으로 provider를 캡슐화 하여 사용하는데 캡슐화 하지 않으면 사용할 수가 없다

이때 해야할 일은

cat.module.ts 에서

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/71bbcb2a-017b-478c-8111-7d655881ff97/Untitled.png)

exports에 외부에서 사용하고 싶은 모듈(CatService)을 넣어서 사용해야한다!

## 캡슐화

- 객체지향의 4요소
    
    ## 객체지향
    
    ### 객체지향의 4대 특성
    
    ### 1. 캡슐화
    
    - 객체의 속성(Variable)을 보호하기 위해 사용
        - 컴퓨터 전원을 켜기 위해 메인보드에 전기신호를 직접 주는 것이 아닌 외부 케이스의 전원 버튼을 이용함
    - Method 설계
        - 속성이 선언 되었으나 상태를 변경하는 method가 없다면 잘못 선언 된 속성이다.
        - 실물 객체가 가진 기능을 모두 제공해야 한다.
        - 각각의 Method는 서로 관련성이 있어야 한다
            - 등록/해지, 생성/삭제 등
        - 객체 안의 Method는 객체 안의 속성을 처리해야 한다
            - 다른 객체를 전달받아 해당 다른 객체에 정의 된 속성을 직접 처리해서는 안 된다
            - 단, Method 실행에 필요한 값들은 객체의 형태가 아닌 매개변수의 형태로 전달 되어져야 한다.
        - Getter/Setter Method
            - 외부에서 내부 속성에 직접 접근해서는 안됨
        - CRUD Method
        - Business Logic Method
        - 객체 생명 주기 처리 Method
            - destroy(), disconnect(), qeit() 등 소멸에 대한 method
        - 객체 영구성 관리 Method
            - 영구성(유효성) 속성에 대한 변경이 필요한 경우 외부에서는 접근이 불가하도록 private으로 선언하면, 내부의 다른 Method를 통해 사용 되도록 한다.
        - Method의 속성은 반드시 1개에 속할 필요는 없고, 여러 속성에 해달 될 수 있다.
    - 장점
        - 객체지향의 패러다임 중 하나인 추상화 제공
            - 단순 호출만으로 해당 기능을 실행할 수 있고 이를 통해 객체 단위 프로그램 설계 가능
        - 재 사용성 향상
            - 한 객체에 관련된 속성 및 Method는 모두 캡슐화의 형태로 제공 되기에 모듈성과 응집도가 높아짐
            - 객체의 경우 단일 객체에만 영향을 주기 때문에 전체 프로그램에 영향을 주지 않아 재 사용성이 높음
        - 유지보수의 효율성 향상
    - 무결성 보장
        - Getter/Setter를 제외 하고는 public method는 입력된 매개변수를 Validation을 한 후에 실행하는 것을 기본으로 함
        - Validation을 통해 객체의 값을 변경 할 때 유효성을 가질 수 있음
    
    ### 2.상속
    
    - 객체지향에서의 상속은 하위로 내려 갈수록 구체화 되는 것을 의미
    - 상속의 효과
        - 프로그램 구조에 대한 이해도 향상
            - 최상위 클래스의 구조를 보고 하위 클래스의 동작을 이해 할 수 있음
        - 재사용성 향상
            - 상속을 이용하여 해당 클래스에 필요한 속성 및 메소드를 모두 정의하지 않고 상속을 받아서 사용 할 수 있다
        - 확장성 향상
            - 일관된 형태의 객체를 추가 할 수 있어 간단하게 프로그램 확장 가능
        - 유지보수성 향상
            - 각 객체마다 일관된 형태를 지니기 때문
    
    ### 3. 다형성
    
    - 하나의 개체가 여러 형태로 변화하는 것을 의미
        - Overriding을 통해 가능
    - 같은 이름의 Method를 이용해 각자의 개체에 맞는 여러 형태의 다른 기능을 구현 할 수 있음
    
    ### 4. 추상화
    
    - Modeling을 의미
    - 구체적으로 공통적인 부분, 또는 특정 특성을 분리해 재조합하는 부분
    - 다형성, 상속 모두 추상화에 속함
    
    ---
    
    ### 객체지향 5원칙 SOLID
    
    ### 1. SRP (단일 책임의 원칙 : Single Responsibility Principle)
    
    - 결합도(Coupling)은 낮추고 응집도(Cohesion)은 높여야 한다
        - **결합도**
            - 모듈(클래스)간의 상호 의존 정도를 나타내는 지표
            - 결합도가 낮으면 상호 의존성이 줄어서 재사용 및 유지보수 유리
        - **응집도**
            - 하나의 모듈 내부에 존재하는 구성 요소들의 기능적 관련성
            - 응집도가 높은 모듈은 하나의 책임에 집중하고 독립성이 높아져 재사용 및 유지보수 유리
    - 상속, 오버라이딩을 사용
    
    ### 2. OCP (개방 폐쇄의 원칙 : Open Close Principle)
    
    - 자신의 확장에는 열려 있고, 주변의 변화에 대해서는 닫혀 있어야 한다.
    - 상위 클래스 또는 인터페이스를 중간에 두어 자신의 변화에 대해서는 폐쇄하고 인터페이스는 외부의 변화에 대해 확장성을 개방한다.
        - JDBC, Mybatis, Hibernate 등, 그리고 JAVA에서는 Stream(Input,Output)에서 찾아볼 수 있다.
    
    ### 3. LSP (리스코브 치환의 원칙 : The Liskov Subsitution Principle)
    
    - 하위 타입은 언제나 자신의 기반인 상위 타입으로 교체 할 수 있어야 한다.
    - 예시 : 상위 : 자동차/운송수단 - 하위:아반테,그렌져
    
    ### 4. ISP (인터페이스 분리의 원칙 : Interface Segregation Principle)
    
    - 클라이언트는 자신이 사용하지 않는 메서드에 의존 관계를 맺으면 안된다.
        - 인터페이스로 각 기능을 분리하여 사용하는 메서드에만 의존 관계를 맺는다.
    - 프로젝트 요구사항과 설계에 따라 SRP(단일 책임 원칙) 또는 ISP(인터페이스분리원칙)을 선택한다.
    
    ### DIP (의존성 역전의 원칙 : Dependency Inversion Principle)
    
    - 자신보다 변하기 쉬운 것에 의존하지 말아야 한다.
        - 개발폐쇄 원칙에서도 살펴본 내용
            
            ```
            - SOLID는 객체 지향 4대 특성에 기반하기에 유사한 모양을 지님
            ```
            

# 미들웨어

→ express에서의 미들웨어와 완전히 동일하다!

→ router에서 controller가기전 실행하는 느낌

→ (req , res , next ) 3개의 매개변수가 있으면 미들웨어로 인식한다!

- 미들웨어 만드는 방법

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/423ae277-2127-4612-8b95-6a266513d930/Untitled.png)

위처럼 nest g middleware라고 명령어를 입력하면 어떤 미들웨어를 할지 정하는 곳이 나오는데

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1499b812-690c-415a-8886-8d6830e4ff27/Untitled.png)

일단 logger를 만들어 보겠습니다! logger를 만들면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2f538c3f-c1d2-4403-b7c3-21f9e1edad16/Untitled.png)

이런 ts파일이 생성됩니다!

express와 동일하게 미들웨어를 사용하는데 use

가 사용되었고 next() 도 볼수 있어요~

현재 req res next 전부 타입이 기본으로 되어있는데 이걸 알맞게 바꾸면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bdab2222-7cd4-4b60-81e0-c20cdda85c9a/Untitled.png)

이렇게 바꿀수 있따

이제 실행되는지 확인해 볼까요?

일단 app.module.ts에서

```tsx
himport { Module, MiddlewareConsumer, NestModule } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UsersController } from './users/users.controller';
import { UsersService } from './users/users.service';
import { UsersModule } from './users/users.module';
import { LoggerMiddleware } from './logger/logger.middleware';

@Module({
  imports: [UsersModule],
  controllers: [AppController, UsersController],
  providers: [AppService, UsersService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('*');
  }
}

```

이렇게 AppModule에 logger 미들웨어를 사용하게 해놓는다

console.log를 넣어보아요

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a645d174-ab71-464e-82c0-d9bd85670b75/Untitled.png)

요청의 ip와 요청의 엔드포인트를 알려주는 값들을 콘솔 찍어볼게요

postman으로 실행한다면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/28ec49d7-93ae-4a3e-8950-8e60724e8186/Untitled.png)

요렇게 잘 나오는걸 확인할 수 있다!

이제 테스트는 확인해 보았는데 nest에서의 logger를 사용하여 확인해보자

nestjs에서는 logger를 사용하는데

Logger 클래스에서 인스턴스를 생성해서 사용한다!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1a238f1b-6186-4b46-96f1-0f9870a8d20b/Untitled.png)

HTTP관련한 logger 인 logger 인스턴스를 생성해서

logger에 관련한 log()를 찍는다 그럼

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/78b5a1c9-c523-424b-89e2-036c05b0ccc9/Untitled.png)

이렇게 console.log처럼 나오는게 아니라 nest에 log로 노출된다

여기서 더 확장한다면 res를 보내는것을 끝냈을때의 log를 찍고싶다고 한다면~

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/31e6db53-7b38-4df2-ac99-656d303943f2/Untitled.png)

이런식으로 .on() 으로 핸들러를 등록하고 finish 가 되었을때 2번째 인자인 콜백함수를 실행하는 느낌으로도 logger를 활용할 수 있다!

# 예외처리

postman으로 우리가 등록하지 않은 엔드포인트 ( ‘ / ‘ , ‘/cats’ ) 을 제외한 엔드포인트로 요청을 보내면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/776fff94-7c6d-4240-a059-935971e84377/Untitled.png)

이런식으로 응답이 온다 이때 우리는 해당정보 말고 더 많은 정보를 넣고 싶을때도 있다.

이때 우리는 어떻게 해야하는지 알아보자~

일단 express에서 예외처리를 할때에는 우리는

```tsx
const error =  new Error('your request is strainge)
error.statusCode = 400;
throw error
```

이런식으로 진행했다 nest에서는

```tsx
throw new HttpException('your request is strange, 400)
```

이런식으로 진행하면 된다

테스트를위해 우리가 만들었던 cats.controller.ts 에서 get /cats에 해당 코드를 추가해보자

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/acb222a9-0c33-4f44-b9bc-d3826658e1e1/Untitled.png)

이렇게 추가한 뒤에 postman으로 쏴주면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5356e0ad-28b7-4634-b9be-3b69d37ed002/Untitled.png)

이런식으로 응답이 노출된다.

HttpException 은 아래처럼 커스텀하여 사용할 수도 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/739f0cc2-1efc-4bac-9fc3-52060607f8eb/Untitled.png)

그렇다면 postman에서는

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b44997c9-4d37-45e5-8e0f-684d5955cb49/Untitled.png)

요렇게 응답이 나온다!

이렇게 예외처리를 진행하려면 예외처리가 필요할때마다 사용해야하는데,

재사용성이 굉장히 떨어진다( 모든 endpoint마다 사용해야된다..)

그래서 재사용성을 고려한 코드를 작성해보자

일단 공식문서에서 나온것처럼 ts파일 하나 만든다

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/67063049-bdba-4f8d-854d-1de792f24332/Untitled.png)

이런 파일을 하나 만든다

보면 느낌이 Http 예외가 발생한다면 catch로 잡고 response와 request와 status를 가져와서 status에 맞는 statuscode와 json을 넘겨주는거 같다

이를 사용하는 법을 알아보자! 사용방법에는 두가지가 있다.

- 하나씩 넣어주기
- 전역으로 해버리기

일단 하나씩 넣어주는 방법을 알아보자

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/79290a6f-c875-4aa6-883f-c8f9b2141827/Untitled.png)

cats.controller.ts에 우리가 만들었던 HttpExceptionFilter를 import해오고 @UseFilters()라는 데코레이터를 추가해준다.

그럼 아래에 발생한 exception 을 usefilters로 가져가서 응답을 주게됩니다

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f40b73a9-5983-4ef6-a3cf-14706aa691ce/Untitled.png)

우리가 아까 만들었던 response.status(status).json({})이 응답으로 나오게됩니다!

그럼 여기서 exception filter에서의 json형태를 한번 바꿔보자

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7b6a4de7-dfa7-4f38-aab5-81ba78d58743/Untitled.png)

이런식으로 success: false, error:error 를 추가해준다

여기서 변수 error는 exception.getResponse()를 이용하여 우리가 설정했던 에러 메시지 ( exception test) 를 가지고 오는 방법이다.

이렇게 변경하고 postman을 사용하면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/52516d58-0c76-435e-b6da-7cfe29a09b50/Untitled.png)

이렇게 나온답니다!

그럼 전역으로 하는방법도 알아봐야 겠죠?

간단하게 생각한다면 각 메소드 만이 아니라

클래스에 사용하면 된다

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f01a2b01-2b6e-4b50-b64f-b15591712028/Untitled.png)

위처럼 @Controller 아래에 사용하면 된다.

여태 우리가 했던건 우리가 임의로 만든 예외처리였다면

엔드포인트가 없는 곳에 요청을 보내는 nest에서 잡는 에러는 어떻게 핸들링 하는지 확인해보자( 전역 감성 )

main.ts에서

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e0b9c1bc-fde0-4f71-a547-06b12a688a95/Untitled.png)

위처럼 app.useGlobalFilters(new HttpExceptionFilter());

이렇게 사용하면 된다 이제 404에러를 내보자

엔드포인트를 이상한거 하면 된다

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cabc9f8d-dfd9-458e-adbf-336c5bf18eaa/Untitled.png)

그럼 응답이 이렇게 노출되었다.

이때 예외를 401일때 404일때 나눠서 진행할 수 있다.