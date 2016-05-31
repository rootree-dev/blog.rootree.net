---
layout: post
title: Node.js로 5분안에 봇 만들기
description: 사람에 따라서 50분이 걸릴지도 모릅니다.
image: bot5min-og.png
---

제목이 조금 자극적이지만 준비만 되어있다면 약간 과장을 섞어 5분이면 봇 하나를 만들 수 있다. ~~그렇다고 어디가서 5분이면 만들어요! 라고 말하지는 말자~~

아래의 예제는 [messenger-bot](https://github.com/remixz/messenger-bot)의 Readme 예제를 사용하였고 [Github](https://github.com/rootree-dev/bot-5min)에 Example 소스코드를 올려두었다.

# 준비사항

시작하기 앞서 아래의 것들이 준비되어야만 5분정도 걸린다. 참고로 Mac 터미널 환경에서 작업하였으나 윈도우도 크게 문제가 되지 않을 것이다. ~~윈도우 시러요~~

- Git
- Node.js (여기서는 v6.2.0을 사용했다) [설치법 참고](https://nodejs.org/en/download/)
- 무료 서버 사용을 위한 Heroku 세팅 [여기 참고](https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction)
- 복사 붙여넣기 마인드

# Heroku 앱을 만들기

![Create Heroku App](/public/bot5min-heroku-create-app.png)

먼저 [Heroku Dashboard](https://dashboard.heroku.com/apps)에 가서 앱의 이름을 넣고 Create App을 누른다. 여기서는 `bot-5min`으로 정했다.

# 페이스북 세팅하기(1)

페이스북에서 우측상단의 화살표를 클릭하여 페이지를 생성한다. 아래 이미지 대로 하면 된다.

![facebook setting #1](/public/bot5min-fb1.png)

![facebook setting #2](/public/bot5min-fb2.png)

여기서 messenger옆의 Get started를 누르고 또 같은 버튼을 누른다.

![facebook setting #3](/public/bot5min-fb3.png)

Setting에 가서 App secret 아래의 show버튼을 누른 후 어딘가에 복사해둔다.

![facebook setting #4](/public/bot5min-fb4.png)

좌측에 보면 Messenger밑에 Setting을 눌러 Page Access Token을 생성하고 어딘가에 저장해둔다.

# 코드 생성하기

잠깐 페이스북 세팅 페이지를 놔둔채 코드를 짜보도록 하자. 폴더를 하나 만들고 다음과 같은 명령어를 입력한다.

```
$ mkdir bot-5min
$ cd bot-5min
$ git init
$ heroku git:remote -a <Heroku app 이름>
```

그 다음 각 파일을 만들고 코드를 복사 붙여넣기하고 약간의 수정을 한다.

*app.js*

```
const http = require('http')
const Bot = require('messenger-bot')
const process = require('process')

let bot = new Bot({
  token: '<여기에 아까 저장해둔 Page access token을 넣는다>',
  verify: 'helloworld',
  app_secret: '<아까 저장해둔 App secret token을 넣는다>'
})

bot.on('error', (err) => {
  console.log(err.message)
})

bot.on('message', (payload, reply) => {
  let text = payload.message.text

  bot.getProfile(payload.sender.id, (err, profile) => {
    if (err) throw err

    reply({ text }, (err) => {
      if (err) throw err

      console.log(`${profile.first_name} ${profile.last_name}: ${text}`)
    })
  })
})

http.createServer(bot.middleware()).listen(process.env.PORT)
console.log('서버 뜸')
```

*package.json*

```
{
  "name": "bot-5min",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node app.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "messenger-bot": "^2.3.0"
  },
  "engines": {
    "node": "6.2.0"
  }
}
```

# 배포하기

이 부분은 Heroku Dashboard에서 App 선택을 하고 Deploy 메뉴를 누르면 나오는 내용이다. 준비사항에서 Heroku Toolbelt를 설치해두었다는 가정하에 아래를 진행하였다.

```
$ heroku login
```

이메일과 비밀번호를 적고 아래의 내용을 진행한다. (당연하게도 `bot-5min` 대신 당신의 앱 이름을 넣어야한다..)

```
$ git add .
$ git commit -am "make komerica great!"
$ git push heroku master
```

잠시 기다린후 자신의 Heroku url에 접속해보자. `{"status": "ok"}`라고 나온다면 성공!

```
https://<Heroku App 이름>.herokuapp.com/_status
```

# 페이스북 세팅하기(2)

이제 페이스북을 조금 더 세팅하면 끝난다. 아까전 페이스북 앱설정 페이지로 돌아가서 Messenger setting으로 들어간 후 Webhooks를 아래처럼 *자신의 Heroku app url을 넣어* 설정하고 아까 만든 페이스북 페이지가 Webhook을 subscribe 하게 한다. 참고로 Verify token은 코드를 복붙했다면 `helloworld`다. 

![facebook setting #5](/public/bot5min-fb5.png)

![facebook setting #6](/public/bot5min-fb6.png)

# 끝

세팅이 끝났다면 이제 만들어둔 페이지에 메세지를 보내본다. 참고로 App review를 받기전까지는 개발자로 등록되지 않은 사람들은 페이지에 메세지를 보내도 답변을 받을 수 없다.

![Oops](/public/bot5min-final.png)

더 발전을 시키고자 하면 [messenger-bot](https://github.com/remixz/messenger-bot) 패키지와 [Messenger platform](https://developers.facebook.com/docs/messenger-platform/implementation) 문서를 읽으면 가능하다.

# 후기

봇 만드는 시간보다 블로그 글을 쓰는 시간이 훨씬 더 오래 걸렸다(..)
