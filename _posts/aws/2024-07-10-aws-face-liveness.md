---
title: AWS Face Liveness 예제 따라하기
author: sunshine@ptokos.com
categories: [ AWS ]
---

## 목표

화면에 얼굴을 위치시키고 **Start video check** 를 누른다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-1.png)

**Move Closer** 가 나오며 얼굴을 화면에 가까이 위치시킨다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-2.png)

잠시 기다리면 **User is live** 또는 **User is not live** 가 나온다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-3.png)

## Face Liveness 예제 만들어보기

우선 AWS Blog 에 있는 예제는 2023년 8월 31일에 포스팅된 예제이다.

[Detect real users with AWS Amplify and Face Liveness](https://aws.amazon.com/blogs/mobile/detect-real-users-with-aws-amplify-and-face-liveness/)

제목을 통해 알 수 있듯 [Amplify](https://aws.amazon.com/amplify/) 라는 서비스를 사용하여 Face Liveness 를 구현하는 예제이다.

[Amplify](https://aws.amazon.com/amplify/) 는 웹 & 모바일 애플리케이션을 쉽게 개발할 수 있도록 도와주는 서비스이다.
프론트엔드, 백엔드 기능을 통합적으로 제공한다.

예제가 있음에도 불구하고 글을 따로 쓰는 이유는 코드가 변경된 부분이 있고 예제를 그대로 따라하다보면 **조그마한 에러**들을 만나기 때문이다.
빨리 만들어서 테스트를 하고 싶은 분들을 위해 이 글을 쓰게 되었다.

### Setup App

**create-next-app** 를 사용하여 프로젝트를 생성한다.

```bash
npx create-next-app@latest
```

문서에서는 npm 을 사용하는데 필자는 yarn 을 좋아하기에 **yarn** 으로 진행한다.

@aws-amplify/cli 를 글로벌로 설치한다.

```bash
yarn global add @aws-amplify/cli
```

필요한 패키지를 설치한다.

```bash
yarn add @aws-amplify/ui-react-liveness @aws-amplify/ui-react aws-amplify
```

init 통해 사용하여 amplify 를 **초기화**한다.

```bash
amplify init
```

설정 마지막에 AWS Profile 를 선택해야하는데 **Profile** 이 없으면 생성해야한다.
아래 명령어를 통해 생성하면 된다.

굉장히 친절하게 설명이 나오니 따라하면 된다.
각 단계에 링크를 주어 따라할 수 있도록 한다.

```bash
amplify configure
```

[Set up Amplify CLI](https://docs.amplify.aws/gen1/javascript/tools/cli/start/set-up-cli/#configure-the-amplify-cli) 참고

**amplify add auth** 를 통해 인증을 추가한다. 아래 중에서 하나를 선택하라고 나오는데 첫번째 선택인 default 를 선택한다.

- [x] Apply default configuration with Social Provider (Federation)
- [ ] Walkthrough all the auth configurations
- [ ] Create or update Cognito user pool groups
- [ ] Create or update Admin queries API

```bash
amplify add auth
```

**push** 를 통해 배포한다.

```bash
amplify push
```

여기까지 진행한 후 AWS Console 을 통해 확인해보자.

생성한 **Region** 을 선택한 후 **Amplify** 를 들어가면 생성한 App 이 보인다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-4.png)

해당 앱을 클릭해서 들어가보면 dev 브랜치로 생성된 것을 확인할 수 있다.
**dev** 를 클릭해서 더 자세히 확인해보자.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-5.png)

**Authentication** 탭을 클릭하고 **View in Cognito** 를 클릭해보자

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-6.png)

그럼 아래 예제를 다 따라한 후 실습할 때 이메일 인증을 받도록 하는데 해당 정보를 볼 수 있다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-7.png)

#### IAM 권한 설정

**IAM > Roles** 를 들어가보면 아래와 같이 3개의 role 이 생성된 것을 확인할 수 있다.
이 중에서 **authRole** 로 끝나는 role 을 클릭한다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-8.png)

**Add Permission** 을 클릭하고 **Create inline policy** 를 클릭한다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-9.png)

**JSON 탭**을 클릭하고 아래와 같이 JSON 을 입력한다.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "rekognition:DetectFaces",
        "rekognition:CompareFaces"
      ],
      "Resource": "*"
    }
  ]
}
```

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-10.png)

여기까지 진행하면 기본적인 환경 설정은 끝났다.

### 백엔드 구성

```bash
amplify add api
```

add api 를 입력하면 GraphQL 과 REST 를 선택할 수 있는데 **REST** 를 선택한다.

```bash
? Please select from one of the below mentioned services: REST
```

label 은 **liveness** 를 입력한다.

```bash
Provide a friendly name for your resource to be used as a label for this category in the project: liveness
```

path 는 **/session** 을 입력한다.

```bash
Provide a path (e.g., /book/{isbn}): /session
```

위 설정은 아래에 있을 코드에서 이와 같이 사용된다.

```typescript
const {response, cancel} = get({
    apiName: "liveness", path: "/session/get",
    options: {
        queryParams: {
            sessionId: sessionId!.sessionId,
        },
    }
});
```

이어지는 설정에서는 **Lambda** 를 선택, **NodeJS** 를 선택하면 된다.

함수 템플릿은 **Serverless ExpressJS function** 을 선택한다.

마지막 2개 설정은 아래와 같이 설정한다.

```bash
? Do you want to configure advanced settings? No
? Do you want to edit the local lambda function now? Yes
```

**@aws-sdk/client-rekognition** 패키지 추가한다.

```bash
yarn add @aws-sdk/client-rekognition
```

위에 add api 를 통해 amplify 폴더가 생성되었을 것이다.
**amplify/backend/function/liveness/src** 안에 **utils** 폴더를 만들고 **getRekognitionClient.js 파일을 생성**한다.

간단한 코드지만 2가지가 중요하다.

1. ES6 문법을 사용하면 안된다.(ES6가 습관이 들어서 import 로 적었다가 에러를 만나게 되었다.)
2. region 이 한정적이다.

region 이 한정적이라는 뜻은 rekognition 을 사용할 수 있는 region 이 제한적이라는 뜻이다.
현재 2024년 7월 10일 기준으로는 4개의 region 에서만 사용할 수 있다.

- US East (N. Virginia)
- US West (Oregon)
- Europe (Ireland)
- **Asia Pacific (Tokyo)**

그래서 아래 코드에서 도쿄인 **ap-northeast-1** 을 사용하였다.

```javascript
const {Rekognition} = require("@aws-sdk/client-rekognition");

const getRekognitionClient = () => {
    const rekognitionClient = new Rekognition({
        region: 'ap-northeast-1'
    });

    return rekognitionClient;
};

module.exports = getRekognitionClient;
```

[참고 Face LiveNess FAQ](https://docs.aws.amazon.com/rekognition/latest/dg/face-liveness-faq.html)

**amplify/backend/function/liveness/src/app.js** 파일을 열어서 아래 코드를 추가해준다.

```javascript
const getRekognitionClient = require("./utils/getRekognitionClient");
```

샘플 코드로 GET, POST 등이 있을텐데 샘플 코드를 삭제해도 좋고 혹 그대로 두고 싶다면 **샘플 코드보다 위에 예제 코드를 배치**해야한다.
이유는 샘플 코드에 GET /session 이 있기 때문에 /session/get 이 아래에 배치되어있으면 /session/get 요청에 샘플 코드가 실행된다.

추가된 API 는 2가지인데 **세션을 만드는 것**과 **세션 결과**를 가져오는 것이다.
여기 말하는 세션이란 얼굴 인증 세션을 말한다.

```javascript
app.get("/session/create", async function (req, res) {
    const rekognition = getRekognitionClient();
    const response = await rekognition.createFaceLivenessSession({});

    return res.status(200).json({
        sessionId: response.SessionId,
    });
});

app.get("/session/get", async function (req, res) {
    const rekognition = getRekognitionClient();
    const response = await rekognition.getFaceLivenessSessionResults({
        SessionId: req.query.sessionId,
    });

    const isLive = response.Confidence > 90;

    res.status(200).json({isLive});
});
```

**amplify/backend/function/liveness/custom-policies.json** 파일을 열어 아래 json 추가해준다.

amplify 를 통해 생성된 role 이 있는데 이 role 에 rekognition 을 사용할 수 있는 권한을 추가해주는 것이다.
이런 부분이 amplify 가 가진 편리한 기능이다.

```json
[
  {
    "Action": [
      "rekognition:CreateFaceLivenessSession",
      "rekognition:GetFaceLivenessSessionResults"
    ],
    "Resource": [
      "*"
    ]
  }
]
```

push 를 통해 배포한다.

```bash
amplify push
```

### 프론트엔드 구성

원문 예제를 보면 Nest.js 가 예전 버전이라서 **_app.js** 를 사용하고 있다. 그러나 현재 Next.js 는 page.tsx 를 사용하고 있다.
이런 부분들이 몇 개 있어 이 글을 다시 쓰는 이유이기도 하다.

#### AuthWrapper.tsx 생성

혹 Next.js 생성 시 js 로 만들었다면 타입스크립트 문법만 제거하면 된다.

```tsx
'use client'

import {withAuthenticator} from "@aws-amplify/ui-react";
import {ReactNode} from "react";
import {Amplify} from "aws-amplify";
import awsExports from "@/aws-exports";

Amplify.configure(awsExports);

function AuthWrapper({children}: { children: ReactNode }) {
    return <>{children} < />;
}

export default withAuthenticator(AuthWrapper);

```

#### LiveNessQuickStart.tsx 생성

원문 예제에서와 코드가 좀 다른데 이유는 아래와 같은 코드는 현재 사용할 수 있다.
패키지가 **업데이트** 되면서 **코드가 변경**되었기 때문이다.

```javascript
import {API} from "aws-amplify";
```

지금은 아래와 같은 코드를 사용해야한다.

```javascript
import {get} from "@aws-amplify/api";
```

```tsx
import {useEffect, useState} from "react";
import {Loader, Heading} from "@aws-amplify/ui-react";
import {FaceLivenessDetector,} from "@aws-amplify/ui-react-liveness";
import {get} from "@aws-amplify/api";


export function LivenessQuickStart() {
    const [loading, setLoading] = useState<boolean>(true);
    const [sessionId, setSessionId] = useState<{
        sessionId: string;
    } | null>(null);
    const [success, setSuccess] = useState('');

    useEffect(() => {
        const fetchCreateLiveness = async () => {
            const {response, cancel} = get({
                apiName: "liveness", path: "/session/create",
            });
            const result = await response;
            const data: {
                sessionId: string;
            } = await result.body.json() as any;
            setSessionId(data);
            setLoading(false);
        };

        fetchCreateLiveness();
    }, []);

    const handleAnalysisComplete = async () => {

        const {response, cancel} = get({
            apiName: "liveness", path: "/session/get",
            options: {
                queryParams: {
                    sessionId: sessionId!.sessionId,
                },
            }
        });

        const data: any = await (await response).body.json();

        if (data.isLive) {
            setSuccess("User is live");
            console.log("live");
        } else {
            setSuccess("User is not live");
            console.log("not live");
        }
    };

    const handleError = (error: any) => {
        console.log("got error", error);
    };

    return (
        <>
            {loading ? (
                <Loader/>
            ) : (
                <>
                    <FaceLivenessDetector
                        sessionId={sessionId?.sessionId ?? "1"}
                        region="ap-northeast-1"
                        onAnalysisComplete={handleAnalysisComplete}
                        onError={handleError}
                    />
                    <Heading level={2}>{success}</Heading>
                </>
            )}
        </>
    );
}

```

#### LivenessPage.tsx 생성

```tsx
import {Button, useAuthenticator, View} from "@aws-amplify/ui-react";
import {LivenessQuickStart} from "@/app/components/LivenessQuickStart";

export default function LivenessPage() {
    const {signOut} = useAuthenticator((context) => [context.signOut]);
    return (
        <View width="600px" margin="0 auto">
            <LivenessQuickStart/>
            <Button onClick={signOut} variation="warning">
                Sign Out
            </Button>
        </View>
    );
}
```

#### page.tsx 수정

```tsx
'use client';

import "@aws-amplify/ui-react/styles.css";
import LivenessPage from "./components/LivenessPage";
import AuthWrapper from "./components/AuthWrapper";


export default function Home() {
    return (
        <AuthWrapper>
            <LivenessPage/>
        </AuthWrapper>
    );
}
```

## 테스트

처음으로 페이지를 열면 아래와 같이 로그인 화면이 나온다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-11.png)

Create Account 탭에서 정보를 입력하고 **Create Account** 를 누른다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-12.png)

그럼 이메일 인증 화면이 나오고 **verification code** 를 입력하면 가입이 된다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-13.png)

가입한 정보로 로그인을 하고 브라우저에서 웹 캠 권한을 허용해주면 아래와 같이 사용할 수 있다.

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-1.png)

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-2.png)

![AWS Face Liveness 예제 따라하기](/assets/img/aws/aws-face-liveness-3.png)

휴대폰에 사진을 보여주어서 인증을 시도해보면 **User is not live** 가 나온다.




