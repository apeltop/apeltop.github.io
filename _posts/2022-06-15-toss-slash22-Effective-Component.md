---
title: Toss SLASH22 Effective Component 지속 가능한 성장과 컴포넌트 간단 리뷰
author: sunshine@ptokos.com
categories: [toss, conference]
---

카카오톡을 사용하다 toss SLASH22 광고를 보았는데 세션 소개가 굉장히 흥미로워 사전 신청을 하였습니다. 

[유튜브](https://www.youtube.com/watch?v=ftFHZwyUN38&list=PL1DJtS1Hv1PiGXmgruP1_gM2TSvQiOsFL)에 각 세션 별로 영상이 올라와 있습니다.

여러 세션 중 [Effective Component 지속 가능한 성장과 컴포넌트](https://www.youtube.com/watch?v=fR8tsJ2r7Eg) 세션을 간단 리뷰해보려 합니다.

발표는 토스 Frontend Developer 이신 한재엽님께서 발표하셨습니다.
![Alt text](/assets/img/toss-slash22/effective-component-1.png)

해당 세션은 무엇보다 친절한 세션이였던 것 같습니다. 친절하다는 것은 설명과 예제가 훌륭하다는 것입니다.

대략 22분 발표로 짧은 시간의 발표는 아니라 많은 내용이 담겨있는 세션입니다. 그 중 인상 깊었던 몇 부분만 뽑아보려합니다.

초반에 부끄러운?! 코드라고 소개해주신 코드입니다.
![Alt text](/assets/img/toss-slash22/effective-component-2.png)

위 코드를 보면 Item component 에 item 이라는 props 를 넘겨주고 있습니다. 이 경우 item 이 무엇을 의미하는지 파악하기가 어렵습니다. 
이러한 비슷한 맥락에서 더스틴 보즈웰, 트레버 파우커가 쓴 『읽기 좋은 코드가 좋은 코드다(The Art of Readable code)』 에서 시간을 나타낼 때 ms, s, m, h 등 단위를 변수명 뒤에 적어야한다는 것이랑 비슷한 맥락이라고 느껴졌습니다.

이러한 경우는 해당 코드를 작성할 때는 머릿속에 다 그려져있기 때문에 문제 의식을 못 느끼기가 너무 쉬워 의지적으로 노력해야하는 부분인 것 같습니다.


지속 가능한 컴포넌트를 위한 큰 틀을 적어주신 것 같습니다. 
![Alt text](/assets/img/toss-slash22/effective-component-3.png)

Headless 에 관하여 한재엽님께서 설명을 잘 해주셔서 제대로 이해하고 싶으시다면 영상을 참고해주시면 좋을 것 같습니다.

간단히 요약하면 데이터와 화면을 분리하자는 것입니다. 
유용한 방법 중 하나는 hooks 를 화면에서 불러와 사용하는 것 입니다. 세션에서는 캘린더의 예제로 설명해주셨습니다.
데이터와 화면을 분리하면 얻어지는 것은 모듈화입니다. 모듈화가 된다는 것은 지속 가능한 코드라고도 볼 수 있지 않을까 싶습니다.

아래 이미지에 해당하는 부분은 특히 인상 깊었던 부분입니다. 그 이유는 저도 이러한 코드의 난잡함을 겪는데 hooks 로 깔끔하게 처리할 방법을 생각을 못 했었기 때문입니다.
이렇게 난잡함을 줄여가야 지속 가능한 성장이 될 수 있을 것 같다는 생각이 듭니다.(가끔은 새로 만드는게 빠르다고 느낄 때가 많으니까요..ㅎ)
![Alt text](/assets/img/toss-slash22/effective-component-4.png)

이마를 한 대 치고갈 코드였습니다.
![Alt text](/assets/img/toss-slash22/effective-component-5.gif)


아래 이미지에 해당하는 내용을 들었을 때는 좀 생각이 필요했습니다. 
![Alt text](/assets/img/toss-slash22/effective-component-6.png)

저는 엄청 단순한 component 가 아니라면 value 가 아니라 어떤 value 인지, onChange 는 어떤게 변경된 것인지를 변수명에 포함하는 것을 선호합니다.
더스틴 보즈웰, 트레버 파우커가 쓴 『읽기 좋은 코드가 좋은 코드다(The Art of Readable code)』 에서 간단한 함수에 반환이 될 값을 담은 변수명은 result 가 좋다고 설명하지만
다른 클린 코드 관련 책 중에는 result 변수명을 리펙토링의 대상으로 보기도 합니다. 이처럼 개인의 성향이 양보할 수 없는 지점이 있겠지만 제가 해당 내용을 들었을 때는
너무 component 를 복잡하게 사용해서 변수명을 명확하게 기술했어야하나 싶은 생각이 들었습니다. 그래서 component 를 단일 책임에 가깝도록 코드를 다시 봐야겠다고 생각이 들었습니다.



![Alt text](/assets/img/toss-slash22/effective-component-7.png)
저의 경우 react 를 사용하다보면 component 를 별 생각없이 쪼개고 보는 경향이 있습니다. 발표 자료에 적혀있는 것처럼 
component 를 분리했을 때 복잡도와 재사용 가능성에 대해서 고민해보고 불편함을 느끼고 개선해야겠다는 반성이 슬며시 올라왔습니다.


![Alt text](/assets/img/toss-slash22/effective-component-8.png)
좋은 코드, 좋은 컴포넌트는 나를 넘어 회사에 직접적인 영향을 미친다는 것은 개발자의 큰 장점인 것 같습니다.
물론 큰 책임이 따르지만요..! 이러한 이유로 끊임없이 발전하고 주변의 피드백을 듣고 성장해 나가야하는 것이 개발자의 특권인 것 같습니다.(성장통이 있는거같습니다만..ㅎ)


어떻게 보면 짧은 영상이지만 많은 깨달음과 생각을 그리고 한재엽님을 보며 저도 멋있는 개발자가 되고 싶다는 생각이 마구마구 피어올랐습니다.



