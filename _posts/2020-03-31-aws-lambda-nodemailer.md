---
layout: post
title: "Vue.js와 AWS Lambda, Nodemailer 로 이메일 전송 폼 만들기"
author: bluestragglr
date: 2020-03-31 22:55
tags: [aws, Lambda]
image: 'https://blog.nodemailer.com/wp-content/uploads/2017/01/cropped-nm_logo_1000x680.png'
---


> Vue.js와 AWS Lambda, Nodemailer 로 이메일 전송 폼 만들기 by 신성환(github.com/blueStragglr)

---

글의 목적: 이 글을 읽고 나면 온라인에서 폼을 작성해서 submit하면 메일로 도착하는 컴포넌트를 개발하는 과정과 방법에 대해서 알게 됩니다. 서버리스 구축하는 내용은 Vue.js에 무관하게 다른 프로젝트에도 사용하실 수 있습니다.

만들 수 있는 것: 

![최종 결과물](/assets/posts/blueStragglr/aws-nodemailer/Email-Example-final.gif)

사용하는 것들: 프론트엔드 프레임워크 Vue.js, npm(`node-mailer`, `nodemailer-smtp-transport`), 구글계정, AWS계정, AWS Lambda, AWS API Gateway 

## 작업 순서

1. 프론트엔드 (Vue.js)
   1. 폼 만들기
   2. axios로 post하기
2. 백엔드(?) (AWS Lambda)
   1. Lambda 함수 생성
   2. API Gateway로 POST 요청 받아줄 준비하기
   3. CORS & POST body 파싱하기
   4. Node Mailer로 메일 보내기
      1. 이메일을 보낼 SMTP(Simple Mail Transfer Protocol) 설정이 된 계정 정보 등록. (위 코드에서는 Gmail 사용.)
      2. 이메일을 보낼 타겟 설정
      3. Nodemailer 관련 모듈 불러오기
      4. SMTP 설정하기
      5. POST body 파싱
      6. 메일 컨텐츠 구성
      7. 메일 발송 및 response
3. 프론트엔드 (Vue.js)
   1. 비동기 인터랙션 UI 구성하기

## 들어가며

### 람다가 뭐에요? 나는 그냥 메일만 보내고 싶은데 왜 그런걸 써야하죠? 😡

이메일을 보내는 과정이 생각보다 까다롭기 때문에 별도의 서버(혹은 비스무리한것)가 돌아가야 한답니다. 이메일 형태에 맞는 데이터도 생성할 수 있어야하고, SMTP 서비스에서 계정 인증도 받아야해요. 계정 인증이 특히 중요한게, 계정 인증 없이 이메일을 보낼 수 있도록 하면 스팸메일 폭탄을 쏟아내는 것도 가능하기 때문에 이메일을 보낼 수 없답니다.

람다는 서버 없이 코드를 실행하고 실행한 만큼에 대한 금액만 청구되도록 할 수 있는 아마존 웹 서비스입니다. Node.js 등의 기본 환경 위에서 별도의 서버 관리 없이 "함수(기능)"를 실행할 수 있도록 구현된 서비스입니다. 이메일 보내기라는 하나의 기능을 위해서 서버를 만드는 것은 부담스러운 일입니다. 

### EC2로 하면 안되나요?

결론부터 말하자면 됩니다! 사용한 라이브러리인 nodemailer는 node.js 기반이고, 로컬 서버를 만들어 구동해도 잘 돌아가고 EC2 서버에서 구동해도 잘 돌아갑니다.

람다를 사용한 것은 효율 측면에서의 문제입니다. 사용 빈도가 굉장히 높고 실시간으로 반응해야 하는 API이거나 DB에 데이터를 안정적으로 적재해야 한다면 EC2 인스턴스에 express를 설치해서 구동하는 쪽이 유리합니다. 하지만 이메일을 전송하는 폼의 경우, 일반적인 시나리오에서 호출 빈도가 높은 편도 아니고, DB에도 적재할 데이터가 없습니다.

반대로, 호출될때만 사용되는 람다는 굉장히 적합하죠. 비용 측면에서도 훨씬 절약할 수 있고, 지구를 위해(?) 컴퓨팅 파워를 아낄수도 있습니다 🌱

## 작업 내용

### 1. 프론트엔드로 폼부터 만들자!

본 작업의 대부분은 서버 작업입니다. 프론트에서는 정말로 폼을 만들고, post method로 작업을 요청하고, 응답이 올 때 까지 사람들이 버튼을 연타하지 못하도록 막아주면 됩니다. 

1. 폼 만들기

`<input>`에  `v-model` 로 데이터를 바인딩해줍니다.

    <input v-model="YOUR_DATA_MODEL"/>
    <!-- ...(폼 여러개) -->

Placeholder, CSS 등은 편의에 맞게 작성하시면 됩니다. 실제로는 코드가 훨씬 많지만 본문의 주제와 어긋나서 과감하게 쳐냈습니다. 

2. Axios로 post하기

우선 Axios를 설치합니다.

    $ yarn add axios
    # or, $ npm install axios

그 다음, 곧 생성할 AWS API쪽으로 Post할 준비를 합니다. 

    axios.post('API_FROM_AWS_API_GATEWAY', JSON.stringify(YOUR_DATA))
              .then((res) => {
                console.log(res)
              })
              .catch((e) => {
                console.log(e)
              })

### 2. Lambda 함수 생성

1. 생성하기

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled.png)

함수를 생성해 봅시다. 이름은 본인이 알아들을 수 있는 정도면 되고, 런타임(언어)은 노드 선택해 주시면 됩니다. 10.x도 동작하긴 할텐데 12.x도 동작에 문제가 없으니 유지보수기간의 안정성을 위해 최신 버전을 선택해 줍시다. 템플릿은 선택하지 않습니다. 

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 1.png)

인스턴스와는 다르게 별다른 설정 없이 뿅 하고 생깁니다. 스크롤해서 내려 보면 함수 코드 영역이 있습니다. 우상단의 테스트를 눌러서 테스트 인풋 세트를 만들고 만든 함수를 테스트 해 봅시다. 

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 2.png)

테스트케이스를 작성하고 실행하면 handler의 인자인 event가 해당 케이스의 JSON을 받습니다. 테스트 수행을 통해 함수 안에 console.log 등을 찍어 확인해 보는 것이 가능합니다.

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 3.png)

짐작하셨겠지만, exports.handler는 람다가 트리거되면 호출되는 함수입니다. POST 등으로 인자를 전달받아 수행하는 것도 가능하구요. 아래에서 POST를 통해 전달받은 요청을 활용하는 방법을 좀 더 자세하게 알아봅시다. 

2. API Gateway로 받아줄 준비하기

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 4.png)

람다의 함수는 "트리거" 되었을 때 실행됩니다. 메일링 서비스는 form을 작성한 후, submit 될 때 axios를 통해 POST method를 수행하도록 작성하였습니다. POST를 받아주기 위해서 API Gateway를 통해 API를 생성해 봅시다. 

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 5.png)

보안은 일단 만든 다음 생각 해 보도록 하고, 우선은 열린 API로 만들어 줍니다. 번역이 이상해서 '열기' 로 번역되어 있는데, 해당 옵션이 보안 없이 open된 API를 생성하는 옵션입니다.

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 6.png)

Designer 박스에서 API 게이트웨이를 클릭하면 아래에 생성한 API 이름이 나타납니다. 눌러서 들어가면

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 7.png)

이와 같이 메인 화면에서 호출을 위한 base URL을 알 수 있습니다. 좌측 탭의 '통합'에 들어가면 만든 함수의 이름으로 API가 생성되어 있습니다. 

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 8.png)

이제 이렇게 주소창에 직접 입력해서 response를 받아볼 수 있습니다! 당연하지만 Axios로 요청 보내서 response 받는것도 가능합니다. 

    postTest () {
      axios.post('https://YOUR_API_ADDRESS.amazonaws.com/tester',
    		JSON.stringify({
        name: 'blueStragglr',
        message: 'hello'
      }))
        .then((res) => {
          console.log(res)
        })
        .catch((e) => {
          console.log(e)
        })
    },

3. CORS & POST body 파싱하기

이제 람다에 연결한 API에 body를 갖는 POST request를 전송하고 람다 내에서 값을 파싱해 봅시다. 앞서 테스트해본 것 처럼, event는 전달받은 인자를 통째로 들고 있습니다. JSON.stringify된 객체를 string으로 가지고 있다는 뜻이죠. 

위와 같이 프론트 코드를 작성한 뒤, 로컬호스트에서 버튼을 누르고 Response를 기대하면 아래와 같은 창을 만나게 됩니다.

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 9.png)

CORS 세팅이 되어있지 않아 막힌 것입니다. 😭API Gateway 설정에서 CORS와 관련된 설정을 조작할 수 있습니다.

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 10.png)

API Gateway > CORS에서 허용되는 오리진 값에 로컬호스트를 추가 해 줍니다. 레퍼런스 문서에서는 'https://~~' 처럼 스트링 형태로 입력하고 있는데, 그냥 입력하면 됩니다. 마지막에 `/`를 붙이면 origin을 제대로 인식하지 못하니 주의하세요! (localhost:8080까지가 origin이고, `/` 는 루트 경로를 가르키는 URI입니다.)

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 11.png)

그러면 이제 response를 받게 됩니다! body에 (인코딩된 무언가로 보이는) 이상한게 들어가 있습니다. 이 response를 받은 람다의 코드는 이렇습니다.

    exports.handler = async (event) => {
        // TODO implement
        const response = {
            statusCode: 200,
            body: JSON.stringify(event),
        };
        return response;
    };

event를 바로 stringify 해서 보내는 바람에 이상한 값이 들어간 모양입니다. Axios POST로 보낸 값의 body는 `base64` 로 인코딩 된 값이며, 제대로 읽기 위해서는 디코딩 해 주면 됩니다. 

    exports.handler = async (event) => {
        // TODO implement
        
      const base64body = JSON.stringify(event.body)
      const body = JSON.parse(Buffer.from(base64body, 'base64').toString('utf8'))
      
        const response = {
            statusCode: 200,
            body: JSON.stringify(body),
        };
        return response;
    };

가운데 두 줄을 추가하면 body만을 가져올 수 있습니다. Response는 아래와 같이 정상적으로 변합니다. 

![](/assets/posts/blueStragglr/aws-nodemailer/Untitled 12.png)

이제 람다를 사용할 모든 준비가 완료되었습니다! 늘 그렇지만 환경설정이 가장 힘든 일인 것 같습니다. 보안 등은 나중에 업데이트 해야겠지만, 우선 의도대로 동작하는것에 감사하자구요 🦔

4. Nodemailer로 메일 보내기

Nodemailer로 메일을 보내는 예제는 온라인에 많이 있습니다. 제가 작성한 코드도 대부분 공식 문서([https://nodemailer.com/about/](https://nodemailer.com/about/)) 등의 레퍼런스에서 가져온 코드입니다. 우선 전체 코드입니다. 

    // index.js
    const sesAccessKey = 'YOUR_GMAIL_ID'
    const sesSecretKey = 'YOUR_GMAIL_PASSWORD'
    
    exports.handler = function (event, context, callback) {
    
      const sendTarget = 'SEND_TO'
    
      const nodemailer = require('nodemailer');
      const smtpTransport = require('nodemailer-smtp-transport');
    
      const transporter = nodemailer.createTransport(smtpTransport({
        service: 'Gmail',
        port: 465,
        auth: {
          user: sesAccessKey,
          pass: sesSecretKey
        }
      }));
    
      const base64body = JSON.stringify(event.body)
      const body = JSON.parse(Buffer.from(base64body, 'base64').toString('utf8'))
      const data = {
        name: body.name,
        email: body.email,
        organization: body.organization,
        title: body.title,
        inquiry: body.inquiry,
      }
    
      const mailOptions = {
        from: sesAccessKey,
        to: sendTarget,
        subject: data.title,
        html: `<p>Name: ${data.name}</p>
              <p>Email: ${data.email}</p>
              <p>Organization: ${data.organization}</p>
              <br/><br/>
              <p>------ Message ------</p>
              <p>${data.inquiry}</p>`
      };
    
      transporter.sendMail(mailOptions, function (error, info) {
        if (error) {
          const response = {
            statusCode: 500,
            body: JSON.stringify({
              error: error.message,
            }),
          };
          callback(null, response);
        }
        const response = {
          statusCode: 200,
          body: JSON.stringify({
            message: `Email sent successfully!`,
            data: event
          }),
        };
        callback(null, response);
      });
    }

전체적으로 코드의 구조를 구분하자면 이렇습니다.

1. 이메일을 보낼 SMTP(Simple Mail Transfer Protocol) 설정이 된 계정 정보 등록. (위 코드에서는 Gmail 사용.)
2. 이메일을 보낼 타겟 설정
3. Nodemailer 관련 모듈 불러오기
4. SMTP 설정하기
5. POST body 파싱
6. 메일 컨텐츠 구성
7. 메일 발송 및 response

각각의 코드를 간단하게 살펴봅시다.

    // 1. 이메일 SMTP 설정된 계정 정보 등록
    const sesAccessKey = 'YOUR_GMAIL_ID'
    const sesSecretKey = 'YOUR_GMAIL_PASSWORD'

사용할 Gmail 아이디와 비밀번호를 등록합니다. 이메일 전송자를 입력받은 폼의 이메일로 할 수 있으면 좋겠지만 악용의 여지가 있어 그렇게 설정할 수는 없습니다. 여기서 등록한 이메일 계정으로 메일이 발송됩니다.

    // 2. 이메일 보낼 타겟 설정
    const sendTarget = 'SEND_TO'

해당 함수에서 작성된 이메일을 수신할 주소를 작성합니다. 아래에서 다시 호출해 사용합니다. 명시적인 사용을 위해 별도로 변수를 설정했습니다. 

    // 3. Nodemailer 관련 모듈 불러오기 
    const nodemailer = require('nodemailer');
    const smtpTransport = require('nodemailer-smtp-transport');

Nodemailer는 자바스크립트 기반 SMTP 메일 전송 모듈입니다. 가져온 모듈을 사용해서 바로 다음 단계인 SMTP 구성을 진행해줍니다. 

    // 4. SMTP 설정
    const transporter = nodemailer.createTransport(smtpTransport({
      service: 'Gmail',
      port: 465,
      auth: {
        user: sesAccessKey,
        pass: sesSecretKey
      }
    }));

모듈을 불러와서 사용할 메일링 서비스와 포트, 인증 정보를 바인딩합니다. Nodemailer에 사용할 수 있는 서비스 목록은 공식 페이지에 잘 정리되어 있습니다. 

    // 5. POST body 파싱
      const base64body = JSON.stringify(event.body)
      const body = JSON.parse(Buffer.from(base64body, 'base64').toString('utf8'))
      const data = {
        name: body.name,
        email: body.email,
        organization: body.organization,
        title: body.title,
        inquiry: body.inquiry,
      }

람다에서 디버깅하는 것이 어려워 가장 많은 시간을 쓴 파트입니다. `JSON.stringify` 하여 전달된 body는 API Gateway 등을 거치면서 `base64` 인코딩 되어 들어옵니다. 앞에서 다룬 내용대로, 적당히 파싱하여 body를 가져와 원하는 형태로 담습니다. 참고로, POST에서 보낸 데이터의 형태는 다음과 같습니다.

    // 5-1. Sended data from Front-end
    inquiryInput: {
            name: '홍길동',
            organization: '소속',
            email: 'example@gmail.com',
            title: '웹 메일링 폼',
            inquiry: '본문...'
          },
    
    ...
    axios.post('https://[YOUR_API_ADDR].amazonaws.com/[YOUR_API_NAME]', JSON.stringify(this.inquiryInput))
              .then((res) => {
                // DO SOMETHING
              }).catch((e) => {
    ...
    
    // 6. 메일 컨텐츠 구성
      const mailOptions = {
        from: sesAccessKey,
        to: sendTarget,
        subject: data.title,
        html: `<p>Name: ${data.name}</p>
              <p>Email: ${data.email}</p>
              <p>Organization: ${data.organization}</p>
              <br/><br/>
              <p>------ Message ------</p>
              <p>${data.inquiry}</p>`
      };

이제 메일 내용을 작성합니다. 앞에서 선언한 메일 타겟을 작성해 줍니다. `from` attribute가 있긴 한데 SMTP를 인증한 이메일만 사용할 수 있기 때문에 다른 값을 작성하더라도 앞서 인증한 주소로 발송되므로 그냥 앞서 등록한 이메일을 바인딩 해 줍니다. (바인딩 하지 않는 경우에는 뒤의 SMTP 객체가 정상적으로 생성되지 않습니다!)

`html` Attribute 안에는 작성할 메일 본문을 적어 줍니다. 회신할 email을 알아야 하기 때문에 본문에 해당 내용들을 포함시켰습니다. 

    // 7. 발송!
    transporter.sendMail(mailOptions, function (error, info) {
      if (error) {
        const response = {
          statusCode: 500,
          body: JSON.stringify({
            error: error.message,
          }),
        };
        callback(null, response);
      }
      const response = {
        statusCode: 200,
        body: JSON.stringify({
          message: `Email sent successfully!`,
          data: event
        }),
      };
      callback(null, response);
    });

알에서 생성한 `transporter` 객체의 메소드를 이용해 메일을 발송합니다. SMTP 설정인 mailOptions이 문제없이 구성되어 과정이 수행된 경우에는 `200` response 를 생성하고, 그렇지 않은 경우 `500` error와 error message를 반환합니다. 

수고하셨습니다! 메일 전송을 위한 Lambda&API Gateway 구성이 종료되었습니다. 이제 다시 프론트로 돌아가서 간단한 비동기 UI작업을 해 주면 모든 것이 마무리됩니다. 

### 3. 프론트엔드 마무리

이제 프론트엔드 버튼에 Axios를 수행하도록 `@click` 을 달아주면 기능이 잘 수행됩니다. 그런데 뭔가 조금 허전합니다. 버튼을 눌러도 변화가 없고, 완료된 후에도 이게 된건지 안된건지 잘 모르겠다는 생각이 듭니다.

인터랙션을 개선하기 위해서 아래와 같은 항목들을 개발했습니다.

- Form validation 및 입력되지 않은 폼 확인 및 표시
- 눌렀을 때, respond가 돌아오기 전까지 동작하지 않도록 버튼 비활성화 및 UI 변경
- 눌렀을 때 버튼 바로 위에 같은 크기의 Toaster 표시 및 자동 제거
- Respond 대기 중에 버튼 텍스트 바꾸기
- Toaster가 표시되고 있을 때 동작하지 않도록 하기
- 기타 트랜지션 및 스타일링

최종적으로 완성한 결과물은 아래와 같습니다.

![](/assets/posts/blueStragglr/aws-nodemailer/webmail-form.gif)

Input의 validity를 확인하고, 이메일 발송중이거나 sent 토스터가 떠있지 않다면 메일을 전송할 수 있도록 하였습니다. 유저가 열심히 버튼을 연타하거나 하는 등의 행동을 방지하기 위해서이며, 이메일이 전송중이라는 것을 명확하게 보여주기 위한 UI의 로직을 구성하기 위해서 사용한 것이기도 합니다. 

    // 버튼이 클릭되었을 때 수행
    mailTo () {
          if (this.checkInputValidity() && !this.mailingInProgress && !this.mailSent) {
            this.mailingInProgress = true
            axios.post('YOUR_API_ADDRESS.amazonaws.com/seesoMailer', JSON.stringify(this.inquiryInput))
              .then(() => {
                this.mailingInProgress = false
                this.responseText = 'Email sent succeeded!'
                this.mailSent = true
              })
              .catch((e) => {
                this.mailingInProgress = false
                this.responseText = e + '. Please try later.'
                this.mailSent = false
              })
          }
        }

스타일링의 경우 대부분 클래스 바인딩을 통해서 조율하였습니다. Input validating은 각각의 form에 대해 조건을 걸 수 있도록 함수를 작성했고, Toaster의 경우에는 따로 Vue 컴포넌트를 개발하였습니다. 스타일링이나 컴포넌트 제작에 대한 내용은 글의 범위를 벗어나므로 따로 다루지 않겠습니다.

### 마치며 & 사족

여기까지가 Vue.js와 Nodemailer, AWS Lambda를 이용해 서버리스 웹메일 폼을 만드는 방법이었습니다! 사실 본 글을 쓰게 된 이유는 React로 위에서 소개한 일련의 과정을 수행하는 영문 포스트를 보고 Vue 포스트도 있으면 좋겠다 + 한글로 된 포스트가 있으면 도움이 되겠다 하는 마음이 생겨서였습니다. 

저는 Lambda 사용이 처음이라 POST request의 body를 가져오는 부분에서 가장 많이 삽질을 한 것 같습니다. 사실 이메일 폼을 만만하게 보고 3시간이면 뚝딱일줄 알았는데 하루 온종일 시간을 써서 개발하게 되었었습니다. 제 글을 보고 누군가의 삽질 스트레스가 조금이라도 줄어들 수 있길 바랍니다. 

## References

[Nodemailer](https://nodemailer.com/about/)

[Sending email with Nodemailer using a lambda - Edward Beazer Blog](https://www.edwardbeazer.com/sending-email-using-nodemailer-using-a-lambda/)

[CORS](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/how-to-cors.html)