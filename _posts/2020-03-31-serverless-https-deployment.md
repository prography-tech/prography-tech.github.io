---
layout: post
title: "S3, ACM, CloudFront, Route53으로 서버리스 프로젝트 https 배포하기"
author: bluestragglr
date: 2020-03-31 22:18
tags: [aws, CloudFront, Route53]
image: 'https://user-images.githubusercontent.com/44422495/78035000-77809380-73a3-11ea-9138-ff90cc95a393.jpg'
---


> S3, ACM, CloudFront, Route53으로 서버리스 프로젝트 https 배포하기 by 신성환(github.com/blueStragglr)



## 포스트 소개

#### 어떤 글인가요?

- 아직 AWS에 익숙하지 않은 프론트엔드 개발자가
- 처음부터 차근차근 따라해서
- 자신이 소유한 도메인에 https를 붙여 배포하는

과정을 상세히 작성한 글입니다. 별도의 백엔드가 존재하지 않는 서버리스 앱을 AWS를 통해서 배포하고 싶은데 아직 인프라와 친하지 않아 막막함을 느낄 (저같은)분들이 부담없이 시작해 볼 수 있도록, 삽질했던 것들을 복기하고 정리해 보았습니다.

#### 무엇을 할 수 있나요?

AWS의 다양한 서비스(S3, ACM, CloudFront, Route53)와 본인이 소유한 도메인을 이용해서 https로 페이지를 배포할 수 있습니다.

#### 무엇을 준비해야 하나요?

- AWS 계정 (비용 발생합니다!)
- 배포할 프로젝트 (Production build)
- 사용할 도메인

#### 어떤 순서로 진행되나요?

1. 배포할 프로젝트 준비
   1. 프로젝트 구현 및 배포용 빌드
   2. S3 버킷에 올리기
2. 도메인 등록 및 호스팅 설정
   1. 도메인 구입 및 등록
   2. ACM으로 SSL 등록
   3. CloudFront를 통한 배포 등록
   4. Route53을 통한 도메인 연결

---



## 배포할 프로젝트 준비

1. 프로젝트 구현 및 배포용 빌드

본인이 배포하고 싶은 프로젝트를 준비합니다. 프로젝트 자체의 구현은 본 글에서 다루는 내용이 아니므로 배포를 위한 빌드가 완료되었다고 가정하고 넘어가겠습니다.

> **처음 배포하는 뷰린이를 위한 사족**
>
> yarn build나 npm run build를 실행하시면 프로젝트 루트에 dist라는 폴더가 생깁니다. 폴더 안에 index.html을 포함해서 여러 파일들이 있는데, 이 파일을 전부 S3 버킷에 올릴거랍니다.

2. S3 버킷에 올리기

S3는 아마존에서 제공하는 Simple Storage Service입니다. 단순히 파일을 저장하는 공간으로 사용할 수도 있지만, 기본으로 제공되는 옵션 중에 [속성 > 정적 웹 사이트 호스팅]을 이용해서 배포용 빌드 파일을 호스팅 할 수도 있습니다. 아래 순서를 따라서 S3에 빌드한 프로젝트 파일들을 업로드 해 봅시다.

- 버킷 생성

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled.png)

우선 S3 버킷을 생성해 줍시다. S3를 저장소로 사용하는 경우라면 퍼블릭 액세스는 막아두는것이 맞겠지만, 저희는  빌드한 프로젝트를 업로드해서 공개할 예정이니 모든 퍼블릭 액세스를 오픈해줍니다. 

- 파일 업로드 및 권한설정

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 1.png)

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 2.png)

생성된 버킷을 찾아 들어가면 위의 화면이 나옵니다. 업로드를 누르면 나오는 창에 파일을 전부 끌어다 넣습니다. 폴더를 통채로 넣는 것이 아니라, `index.html` 이 루트 경로에 자리할 수 있도록 모든 파일을 긁어다 넣습니다.

'다음' 버튼을 눌러 이것저것 설정할 수 있긴 한데, 따로 설정이 필요한 부분이 없으니 그냥 좌하단의 '업로드' 버튼을 눌러 줍니다. 

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 3.png)

그 다음, 업로드 된 파일을 퍼블릭으로 설정 해 줍니다. 

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 4.png)

마지막으로, [속성 > 정적 웹 사이트 호스팅]으로 이동하여 "이 버킷을 사용하여 웹 사이트를 호스팅합니다" 규칙을 설정 해 줍니다. 인덱스 문서에는 빌드된 파일의 인덱스 파일(보통 index.html)을 입력해 줍니다. 

여기까지 잘 되었는지 확인하기 위해, 엔드포인트라고 표시된 주소를 클릭해서 만든 페이지에 접근할 수 있는지 확인 해 주세요! 권한 에러 등이 뜨는 경우 권한 탭에서 퍼블릭 설정이 잘 되어있는지 확인 해 보시고, 잘 되어 있다면 잠시만 기다렸다 다시 시도해 보세요. 아무리 기다려도 안된다면 버킷을 한 번 삭제했다 다시 만들어 보는것도 괜찮습니다. 

### 도메인 등록 및 호스팅 설정

1. 도메인 구매 및 가져오기

연결하고 싶은 도메인이 있다면 해당 도메인을 구매하여야 합니다. 저는 가비아에서 도메인을 구매하였습니다. 

> **왜 도메인을 돈주고 사야 하나요?**
>  저는 처음에 도메인을 구매할 때 왜 도메인을 구매해야 하는지 잘 이해하지 못했습니다. 먼저 등록하는 사람이 우선권을 갖는 것이라면, 중간에 왜 대행업체가 껴서 수수료를 떼가야 하는지 잘 이해하지 못했죠. 이유는 등록 신청 과정 자체가 까다롭고 신청 자격 또한 얻기가 까다롭기 때문입니다. 글로벌하게 주소를 사용하기 위해서는 ICANN(국제인터넷주소관리기구)에 주소를 등록하여야 합니다. 해당 기관에서 DNS 등 여러 인터넷 환경을 관리하기 때문에 수수료가 발생하기도 하고, 등록 절차 또한 직접 수행하지 못하기 때문에 수수료가 발생하는 것으로 보입니다.

가비아를 기준으로 [My 가비아 > 전체 > 관리툴] 을 누르시면 아래와 같은 화면이 나옵니다.

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 5.png)

해당 화면에서 네임서버 설정을 통해 AWS에서 해당 도메인을 사용할 수 있도록 등록할 수 있습니다. 

가비아 창은 닫지 말고, 새 창을 통해 AWS 콘솔로 돌아와 봅시다. [서비스 >Route 53]에 들어오면 아래와 같은 대시보드가 있습니다.

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 6.png)

좌상단의 호스팅 영역 생성을 누르면 오른쪽에 이런 폼이 등장합니다.

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 7.png)

도메인 이름에 구매한 도메인을 작성합니다. 설명은 따로 작성하지 않아도 되고, 저희의 목적은 퍼블릭한 배포이니 유형은 퍼블릭 호스팅 영역을 그대로 놔둡니다. 

그러면 호스팅 영역 하나가 생성됩니다. 눌러서 들어가면 아래와 같이 두 개의 레코드가 있습니다. 레코드는 도메인에 접근했을 때 취할 행동 정도로 생각해 두시면 편할 것 같습니다. 

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 8.png)

여기서 유형이 NS인 (네임서버) 값들을 가비아에 등록할 것입니다. 아까 끄지 않고 남겨둔 가비아 창에서 네임서버 설정을 누르면 아래와 같은 창이 나옵니다. 아래의 입력에 네임서버에 있는 각 값들을 한줄씩 옮깁니다. 이 때, 맨 뒤에 있는 `.` 은 지우고 넣어 줍니다. 

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 9.png)

좋습니다! 여기까지 오셨으면 Route53에 구매하신 도메인 연결이 마무리 된 것입니다. 

2. ACM으로 SSL(인증서) 등록하기

https를 사용하기 위해서는 인증서가 필요합니다. AWS에서 제공하는 Amazon Certification Manager에서 인증서를 발급받을 수 있습니다. Route53에 도메인을 등록한 상황이라면 아주 쉽게 인증서를 발급받을 수 있습니다. 

[서비스 > Certification Manager]로 이동하면 이렇게 생긴 화면이 반겨줍니다. 여기서 인증서 프로비저닝을 눌러 인증서를 발급하면 됩니다. 

**그런데 잠깐!** 아주 중요한 것이 있습니다. 저희는 CloudFront를 이용해서 배포를 진행할 것이기 때문에, 서비스 지역을 꼭 "N. Virginia (버지니아 북부)"로 지정하고 키를 만들어야 합니다. CloudFront에서는 N. Virginia에 생성된 키만 사용할 수 있습니다. 

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 10.png)

이제 시작해 봅시다. 시작하기를 눌러서 공인 인증서 요청을 선택하고

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 11.png)

도메인 이름을 써 줍시다. 

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 12.png)

이 때, 나중에 my-prefix.my-url.com(ex. www.my-url.com, [admin.my-url.com](http://admin.my-url.com) 등)과 my-url.com(www 없는 주소) 모두에서 접속이 원활하게 이루어질 수 있도록 하고 싶다면 "이 인증서에 다른 이름 추가" 버튼을 눌러서 위와 같이 두 개의 도메인을 입력 해 줍니다. 

이제 도메인의 소유권을 인증해야 합니다. 앞 단계에서 Route53에 도메인을 등록 해 두었으니, DNS 검증을 선택해 줍니다. 

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 13.png)

이제 완료하면 아래와 같은 창이 뜹니다. (상태는 주황색 대기중으로 표시됩니다.)

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 14.png)

저는 이미 생성을 완료해서 좌하단의 "Route53에서 레코드 생성" 버튼이 비활성화 되어있지만, 키를 만드신 직후라면 활성화 되어 있을 것입니다. 누르면 작은 팝업이 뜨는데, 해당 팝업에서 DNS 인증을 위한 레코드를 등록할 수 있습니다.

앞서 선택한 DNS 인증이 해당 레코드 등록을 통해서 이루어지는 것입니다. 적절한 이름-유형-값 쌍을 DNS에 등록함으로써 SSL 발급이 완료되며 이후 CloudFront에서 해당 키를 선택해 인증할 수 있습니다. 

3. CloudFront에서 S3와 SSL 연결하여 호스팅하기

이제 거의 다 왔습니다! 조금만 더 힘내봅시다. 

이제 S3에서 정적 호스팅하고 있는 페이지를 CloudFront를 통해 뿌려줄 차례입니다. CloudFront는 AWS에서 제공하는 CDN(Contents Delivery Network)입니다. 컨텐츠를 네트워크 상에 좀 더 효율적으로 배포하기 위한 수단으로 생각하면 좋습니다. 지금 우리는 https 배포를 위해 ACM을 통해 만든 SSL 인증서를 이용하기 위해서 사용하는 것이기도 합니다. 

CloudFront에 처음 들어가면 이런 화면이 반겨줍니다. 아직 CloudFront 콘솔은 한국어를 지원하지 않고 있습니다. 

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 15.png)

Create Distribution을 선택해서 새 배포 설정을 만듭니다.

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 16.png)

Web으로 만들어 줍니다. 밑에 노란색 어그로는 2020년 연말 이후로는 RTMP를 지원하지 않는다는 소리인데 저희랑 관련이 없으니 지나쳐 줍시다.

Get Started를 누르면 바로 숨막히는 양의 설정창이 저희를 반겨줍니다. 겁먹지 말고 하나씩 해 봅시다.

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 17.png)

Origin Domain Name - 클릭하면 사용 가능한 도메인이 주루룩 뜹니다. 생성해둔 S3 버킷이 있을겁니다. 배포할 버킷을 선택합시다.

Origin ID - 위에 두 개 쓰면 자동으로 채워집니다.

Viewer Protocol Policy - HTTP로 들어온 것도 HTTPS로 리다이렉트 하도록 설정해 줍니다.

이제 나머지는 그냥 다 놔두고 쭉쭉 여기(Distribution Settings)까지 내려옵니다.

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 18.png)

Price Class - 퍼포먼스에 비례해서 가격이 오릅니다. 실시간성이 크게 중요한 서비스가 아니라면 싼걸로 하고, 인터랙션 실시간성이 높다면 지금 선택된 옵션을 골라서 돈을 좀 씁시다.

Alternate Domain Names - 해당 Distribution을 통해 사용할 도메인을 등록 해 줍시다. 저는 bluestragglr.com과 www.bluestragglr.com을 모두 하나의 버킷을 가르키도록 사용하기 위해 위와 같이 작성하였습니다. 

SSL Certificate - Custom SSL Certificate를 선택하면 ACM에서 설정한 인증서를 불러올 수 있습니다. 

마지막으로 Default Root Object를 설정해 줍니다. 

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 19.png)

Default Root Object - S3버킷 호스팅의 디폴트 오브젝트인 index.html을 입력해 줍니다

수고하셨습니다! 이제 CloudFront 셋업도 끝났습니다. CloudFront는 반영되는데 시간이 좀 걸려서 Status 뱅뱅이가 좀 오래 돕니다. 5~30분 정도 도는 것으로 알고 있습니다. 여유롭게 물 한잔 떠오고 다음 단계로 넘어갑시다.

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 20.png)

여기까지 제대로 진행되었는지 확인하려면 "Domain Name"을 주소창에 붙여넣기 했을 때 배포한 것이 제대로 표시되는지 확인해 보면 됩니다. 이 과정에서 Access Denied 에러가 굉장히 빈번하게 발생하는데, 아래와 같은 것들을 체크 해 보면 대부분 해결됩니다.

- S3 버킷이 퍼블릭으로 설정되어 있나요?
- S3 버킷 내의 각 객체가 퍼블릭으로 설정되어 있나요?
- CloudFront의 Default Root Object가 index.html로 설정되어 있나요?

4. Route53으로 도메인 연결하기

수고 많으셨습니다. 여기까지 오셨다면 CloudFront를 통해서 SSL 인증서가 붙은 S3 버킷 정적 호스팅의 배포 설정이 준비된 상태입니다. 마지막으로 도메인을 배포 설정에 연결해 주면 목표를 이룰 수 있습니다.

Route53으로 대시보드에서 사용할 도메인 이름을 눌러 호스팅 영역 페이지로 진입합니다. 

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 21.png)

좌상단의 "레코드 세트 생성"을 눌러서 두 개의 레코드를 추가할 것입니다. 하나는 www.my-url.com으로 들어오는 연결에 대한 것이고, 하나는 www 없이 my-url.com으로 들어오는 연결에 대한 것입니다.

www.my-url.com으로 들어오는 것

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 22.png)

이름에 www를 붙이고 별칭은 아니오, 값에는 CloudFront에 있는 도메인 값을 붙여넣습니다.

www 없이 my-url.com으로 들어오는 것

![](/assets/posts/blueStragglr/s3-acm-cloudfront-route53//Untitled 23.png)

이름을 입력하지 않고 별칭은 예, 별칭 대상은 아까 만든 CloudFront 배포를 선택해 줍니다.

CloudFront 배포 별칭은 두 개의 다른 도메인 레코드가 참조할 수 없습니다. 따라서 두 개 모두 A 레코드로 CloudFront 배포를 선택하려고 하면 두 번째로 생성하는 레코드에는 해당 대상이 표시되지 않습니다. 반대로, 두 경우 모두 CNAME을 직접 입력하려고 한다면 최상위 도메인(www가 없는 도메인)에 CNAME을 연결하는 것이 불가능하다는 메세지가 표시됩니다. 따라서 위와 같이 두 개의 레코드를 각각의 방식으로 설정하게 되면 www를 쓴 유저와 그렇지 않은 유저의 요청을 모두 적절하게 처리해 줄수 있습니다. 

이제 모두 끝났습니다! 주소를 입력해 들어가 보면 설정한 도메인으로 훌륭하게 https가 붙은 주소가 동작하고 있는 것을 확인할 수 있습니다!