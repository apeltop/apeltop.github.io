---
title: Sunsetting “expo publish” and Classic Updates 번역
author: sunshine@ptokos.com
categories: [book]
---

이 글은 expo 공식 블로그의 글을 번역한 글입니다.

원문 [https://blog.expo.dev/sunsetting-expo-publish-and-classic-updates-6cb9cd295378](https://blog.expo.dev/sunsetting-expo-publish-and-classic-updates-6cb9cd295378)입니다.

## Sunsetting “expo publish” and Classic Updates
> "expo publish" 와 Classic Updates 를 중단합니다.

Publishing Classic Updates will be supported until 2024. Existing apps will keep working and receive updates published before the cutoff.
> Classic Updates 는 2024년까지 지원됩니다.
> 기존 앱은 계속 작동하며 종료 전에 게시된 업데이트를 받습니다.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*A8yh5x_em-SHaf1Cp6PFNg.png)

### Introduction
Since the early days, Expo has offered an updates service that allows you to bundle your Javascript code and assets and quickly send them to your users,
now called the Classic Updates service.
Hundreds of thousands of developers have send billions of updates with Classic Updates.

> 초강기부터 Expo 는 Javascript 코드와 assets 을 번들로 묶어 사용자에게 빠르게 제공하는 업데이트 서비스를 제공해왔습니다.
> 수십만 명의 개발자는 Classic Updates 를 통해 수십억의 업데이트를 전달했습니다.

Over time, we found several ways we needed to improve how updates works.
Engineering teams working on la rge apps each have their own release process and often outgrow the limited flexibility of deployments offered by Classic Updates.
So, we set out to build a service that can handle high volumes of updates backed by scalable infrastructure. And, of course, a service that delivers updates even faster.

> 시간이 지남에 따라, 우리는 업데이트 작동 방식를 어떻게 개선해야하는지 몇 가지 방법을 찾았습니다.
> 대규모 앱을 작업하는 엔지니어링 팀은 각각 릴리즈 프로세스를 가지고 있으며 종종 Class Updates 에서 제공하는 개발의 유연성을 넘어섰습니다.
> 그래서 우리는 확장 가능한 인프라로 대량의 업데이트를 처리할 수 있는 서비스를 구축했습니다.
> 그리고 물론 업데이트를 훨씬 더 빠르게 제공합니다.

We built our new update service, called EAS Update, to serve those needs above and more.
EAS Update opened to customer preview in December 2021 and became generally available to all Expo developers with a free tier in August 2022.
Like other cloud services, EAS Update is a paid service and scales with customers' usage.

> 우리는 새로운 업데이트 서비스를 구축했습니다.
> 이 서비스는 EAS Update 라고 불리며 위 요구사항들을 충족합니다.
> EAS Update 는 2021년 12월에 고객 프리뷰로 오픈되었으며 2022년 8월에 모든 Expo 개발자가 일반적으로 사용할 수 있게 free tier 로 제공되고있습니다.
> 다른 클라우드 서비스처럼 EAS Update 는 유료 서비스이고 고객의 사용량에 따라 달라집니다.

EAS Update will be our main area of focus for updates-related work,
along with the `expo-updates` module for Expo and React Native apps.
To support this focus, the Classic Updates Service will sunset over the next year.
Read on to Learn about EAS Update and the migration plans for Classic Updates.

> EAS Update 는 업데이트 관련 작업과 Expo 및 React Native 앱용 `expo-updates` 모듈이 주된 관심사가 될 것입니다.
> 이러한 관심사를 위해 Classic Updates 는 내년에 종료될 것입니다.
> EAS Update 와 Class Updates 의 마이그레이션 계획을 알아보려면 더 읽어보세요.

### New Features: A Standard Protocol and EAS Update
#### The Expo Updates Protocol

With EAS Update, we introduced a [standardized updates manifest and asset protocol](https://docs.expo.dev/technical-specs/expo-updates-0),
called the Expo Updates protocol. The protocol specification ensures changes that affect clients and servers are backwards-compatible.
It also reduces vendor lock-in and strengthens the updates ecosystem,
since any developer can write an updates server that conforms to the protocol on their own infrastructure.
Our implementation of the updates server that runs on Expo infrastructure is called EAS Update.

> EAS Update 를 도입하며, 우리는 Expo Updates 프로토콜이라 불리는 [표준화된 업데이트 매니페스트와 asset 프로토콜](https://docs.expo.dev/technical-specs/expo-updates-0)을 도입했습니다.
> 이 프로토콜은 클라이언트와 서버에 영향을 주는 변경 사항이 이전 버전과 호환되도록 보장합니다.
> 이는 공급 업체에 대한 의존도(*vendor lock-in*) 을 줄이고 업데이트 환경을 강화합니다,
> 또한 모든 개발자가 자체 인프라에서 프로토콜을 준수하는 업데이트 서버를 만들 수 있기 때문입니다.
> Expo 인프라에서 실행되는 업데이트 서버의 구현을 EAS Update 라고 부릅니다.

#### Introducing EAS Update
EAS Update is a production-ready service that delivers updates to your users. 
You'll get a ton of features out of box:

- Flexible [deployment strategies](https://docs.expo.dev/eas-update/deployment-patterns/), which give you more control over sending updates to users
- The ability to preview updates in [development builds](https://docs.expo.dev/development/introduction/) of your app, marking reviewing PRs from teammates faster than ever before
- A global CDN that caches assets at the edge, near you users
- Brotli compression, which boasts 14% smaller JavaScript file sizes than Gzip, so that your users will get updates faster
- Web dashboards that display your updates and how they relate to your builds
- A free tier for getting started, and [on-demand pricing](https://expo.dev/pricing) that scales with your usage

To learn more, check out our [announcement blog post](https://blog.expo.dev/eas-update-is-now-generally-available-6532d25e750).

> EAS Update 는 사용자에게 업데이트를 제공하는 프로덕션 지원 서비스입니다.
> 수많은 기능을 즉시 사용할 수 있습니다.
>
> - 사용자에게 업데이트를 보내는 것을 보다 효과적으로 제어할 수 있는 유연한 [배포 전략](https://docs.expo.dev/eas-update/deployment-patterns/)
> - 앱의 [development 빌드](https://docs.expo.dev/development/introduction/)에서 미리 업데이트를 볼 수 있는 기능, 팀원들로부터 PR 리뷰를 더 빠르게 검토 할 수 있습니다.
> - 사용자 근처의 edge 에서 asset 을 캐싱하는 글로벌 CDN
> - 자바스크립트 파일 크기가 Gzip 보다 14% 작은 Brotli 압축, 사용자가 업데이트를 더 빨리 받을 수 있습니다.
> - 업데이트와 빌드들에 얼마나 의존하고 있는지 볼 수 있는 웹 대쉬보드
> - 사용량에 따라 [가격이 확장되는 on-demand](https://expo.dev/pricing) 와 시작하기 위한 프리 티어
>
> 더 자세한 내용은 [공지 볼로그 게시물](https://blog.expo.dev/eas-update-is-now-generally-available-6532d25e750)을 확인해주세요. 


### Sunsetting plan for `expo publish` and Classic Updates
**If you're currently using Classic Updates (that is, using the `expo publish` command with the legacy version of Expo CLI),
you'll be affected by this change.**
Starting with SDK 50, releasing in late 2023, 
the `expo-updates` library will no longer support Classic Updates,
and apps built with latest SDK will need to migrate to either EAS Update or their own updates service.
You will still be able to use Classic Updates in builds that use SDK 49 or earlier.

> 만약 Classic Updates (Expo CLI 의 레거시 버전과 함께 `expo publish` 명령을 사용)를 사용중이라면 이 변경에 영향을 받을 것입니다.
> SDK 50 부터 시작하여 2023 말에 출시되는 `expo-updates` 라이브러리는 Classic Updates 를 더 이상 지원하지 않습니다.
> 최신 SDK 로 빌드된 앱은 EAS Update 또는 자체 업데이트 서비스로 마이그레이션해야 합니다.
> Classic Update 는 SDK 49 또는 이전 버전을 사용하는 빌드에서는 여전히 사용할 수 있습니다.

In over a year, after February 12, 2024, it will no longer be possible to publish new Classic Updates with `expo publish`.
Existing apps will continue to receive updates that were published prior to the cutoff and are actively used.
An update will be considered "actively used" as long as it is downloaded at least once every six months.
Inactive updates will be scheduled for deletion and, in the unlikely event an end user app checks for an update,
will run normally as if the update hadn't been published. It's important to us **this change affects only how you'll publish updates and won't affect your users.**

> 2024년 2월 12일 이후 1년 이상 지난 후에는 `expo publish`로 새로운 Classic Updates 를 더 이상 게시할 수 없습니다. 
> 기존 앱은 중단 이전에 게시되어 활발하게 사용되는 업데이트를 계속 받습니다.
> 업데이트는 적어도 6개월에 한 번 다운로드가 되는 한 "활성 사용(_actively used_)"으로 간주됩니다.
> 비활성 업데이트는 삭제될 예정이며 드문 경우지만 사용자 앱이 업데이트를 확인하면 업데이트가 게시되지 않은 것처럼 정상적으로 실행됩니다.
> 이것은 업데이트를 게시하는 방법에만 영향을 미치고 사용자에게는 영향을 미치지 않는다는 것이 중요합니다.

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*s06_d26eLHvwPj6UVGnu3A.png)

To update your app, you will need to create a new build with `expo-updates@0.13.0` or newer and use an updates service that implements the Expo Updates protocol (described below)

> 앱을 업데이트하려면 `expo-updates@0.13.0` 또는 상위 버전으로 새로운 빌드를 만들어야하며 Expo Updates 프로토콜을 구현한 업데이트 서비스를 사용해야합니다. (아래에 기술되어있습니다)

### How to migrate
There are two ways to use the new Expo Updates protocol after Classic Updates if no longer supported:
- Migrate to EAS Update - follow [our guide](https://docs.expo.dev/eas-update/migrate-to-eas-update/). We recommend this since it's the fastest way to start using the new Expo Updates protocol and is managed by Expo.
- Create and host your own updates service. We created an [example updates server](https://docs.expo.dev/distribution/custom-updates-server/) which you can use to get started. You may want to pursue this route if you prefer to run your own server hardware.

> Classic Updates 가 더 이상 지원되지 않을 때 Expo Updates protocol 을 사용하는 2가지 방법이 있습니다.
> - EAS Update 로 마이그레이션 - [우리의 가이드](https://docs.expo.dev/eas-update/migrate-to-eas-update/)를 따라주세요. 새로운 Expo Updates protocol 를 사용하는 가장 빠른 방법이며 Expo 에서 관리하기에 우리는 이 방법을 추천합니다.
> - 자체 업데이트 서버스를 만들고 호스팅. 우리는 당신이 시작할 수 있도록 [예제 업데이트 서버](https://docs.expo.dev/distribution/custom-updates-server/)를 만들었습니다. 자체 하드웨어 서버에서 실행하려는 경우 이 방법을 추구할 수 있습니다.

### Existing apps will keep working
One of the most important parts of this announcement is that **this change will not break end users' apps**.
Existing apps will continue to receive Classic Updates that have already been published and are actively used.
Additionally, the `expo-updates` library is robust and already handles scenarios when apps are unable to check for of download an update for any reason.

> 이 공지에서 가장 중요한 부분 중 하나는 이 변경이 사용자의 앱을 중단하지 않는다는 것입니다.
> 기존 앱은 이미 게시되어 활발히 사용되는 Classic Updates 를 계속 받습니다.
> 추가적으로 `expo-updates` 라이브러리는 강력하며 이미 앱이 어떤 이유로 업데이트를 확인하거나 다운로드할 수 없는 시나리오를 다룰 수 있습니다.

### Request for Feedback
We understand mandatory migrations interrupt your roadmap and create work that you otherwise wouldn't need to do.
To reduce time pressure and feelings of urgency, we're making this significant announcement after EAS Update has been available to customers for over a year,
and with more than one more year of advance notice before `expo publish` can no longer be used to publish Classic Updates.

> 우리는 강제적인 마이그레이션으로 당신의 로드맵을 방해하고 그렇지 않으면 수행할 필요가 없는 작업을 생성한다는 것을 이해합니다.
> 시간 압박과 긴급함을 줄이기 위해 EAS Update 를 1년 넘게 고객에게 제공하고 `expo publish` 로 Classic Updates 를 게시할 수 없게 되기 1년 이상 전에 사전 공지로 이 중요한 공지를 합니다.

Please reach out in the next month (e.g. on the [forums](https://forums.expo.dev/) or [Discord](https://chat.expo.dev/)) if this timeline will cause problems for you.
We can't promise to accommodate every request, but would like to hear about scenarios we might have overlooked.

> 이 타임라인으로 인해 문제가 발생할 경우 다음 달에 (예: [포럼](https://forums.expo.dev/) 또는 [Discord](https://chat.expo.dev/)) 연락해주세요.
> 모든 요청을 수용한다고 약속할 수는 없지만 못 보고 넘어간 시나리오에 대해서 듣고 싶습니다.
