---
layout: single
title: '프론트엔드 UI↔️ business 로직 분리'
categories: business
tags: frontend
author_profile: false
sidebar:
  nav: 'docs'
# search: false 내가 검색이 원치 않는 글이면...
---

## 프롤로그 🎬

비즈니스 로직, UI는 분리가 되어야 코드읽기도, 유지보수할때도 비즈니스 로직쪽 코드만 수정, 컴포넌트쪽만 수정하는 것이 유리하다, 비즈니스 로직만 보아도 무슨 기능인지 이해할 수 있다, 여러가지 이점들을 가질 수 있다는 글을 읽었다.

이 부분에 대해서는 동의를 한다. 하지만 회사코드에서는 이런 흐름으로 작성된 코드가 없다. 그래도 일관되게 적용이 되어있지 않은것은 다행이다.

💡 맨날 이와 같은 방식(구분없이 한 컴포넌트내부에 모든 로직이 들어가있는)으로 작성을 하는 것은 개인적으로 발전도 없고, 항상 같은 사고에 갇혀있게 되는 것이라고 생각이 들었다. 같은 팀원들에게도 이렇게 작성하는 방식도 있다 라는 것을 공유하고 싶었다. 모두가 예라고 할때 누구는 또 아니라고 하는 사람이 있어야 같이 발전할 수 있지 않을까 라는 생각도 들었다. (무작정 아니라고 하는 건 답이 없지만, 충분한 의견이 있다면 그 의견도 들어는 봐야한다고 생각한다.)

아직은 비즈니스 로직이건, Adapter 패턴이건 자세하게 공부를 더 해야겠지만, 일단 컴포넌트쪽에 들어가 있는 로직을 관심에 맞게 분리하는 과정으로 각 관심사에 맞게 코드를 나눌 수 있을 것이라 판단하여, 이번에 새로운 feature 개발에 적용을 해보았다.

## 기능 분석 🧐 (구독 추가 페이지)

![기획화면](/images/2024-11-10-sixteenth/feature.png)

- 인플루언서 데이터 조회
- 인플루언서 구독 추가
- 인플루언서 구독 해제
- 인플루언서 알람 추가
- 인플루언서 알람 해제
- 우측 Input으로 인플루언서 검색

👉🏻 코드부터 치지말고, 어떻게 작성할지 시간을 갖는게 더 중요하다.

👉🏻 백엔드에서 항상 화면에 필요한 데이터를 모두 내려주는 것은 아니다. 그들도 그들이 유지보수하기 나은 방식으로 코드를 작성한다. 항상 프론트엔드 입맛대로 데이터를 내려주지 않는다.

## 백엔드 API 분석

- 구독중이지 않은 인플루언서들 조회 GET

  ```json
  {
    "id": 0,
    "category": "influencer",
    "name": "string",
    "description": "string",
    "sourceType": "twitter",
    "profileImage": "string",
    "pushTopic": "string"
  }
  ```

- 구독 추가 `POST`
  - 성공시 응답 data Schema

```json
{
  "sourceId": 0,
  "source": {
    "id": 0,
    "category": "influencer",
    "name": "string",
    "description": "string",
    "sourceType": "twitter",
    "profileImage": "string",
    "pushTopic": "string"
  },
  "isAlert": true
}
```

- 알람 추가 `PATCH`
  - 성공시 응답 data Schema

```json
{
  "sourceId": 0,
  "source": {
    "id": 0,
    "category": "influencer",
    "name": "string",
    "description": "string",
    "sourceType": "twitter",
    "profileImage": "string",
    "pushTopic": "string"
  },
  "isAlert": true
}
```

## 추가 기획

- 구독 추가 페이지에서 각 인플루언서 별 구독 추가 및 해제를 할 수 있다. 대신 구독추가를 했다고해서 목록이 사라지지 않고 그자리에 남아있어야한다 ⇒ `refetch`를 해서 구독하지 않은 인플루언서 조회를 하면안된다
- 카테고리를 이동하게 되면 구독 추가하면, 구독 추가를 했던 인플루언서는 사라져도된다.
- 알람 역시 구독추가와 같이 동작해야한다.

## 첫 느낌 🤖

처음에 이거 백엔드 API가 잘못된거 아닌가??라는 생각이 들었다. 구독중인지 아닌지에 대한 여부를 나타내는 프로퍼티가 빠져있는게 아닌가? 그러기에는 api 목적 자체가 “구독중이지 않은 인플루언서들 조회” 였다.

그리고 알람에 대해서도, 구독중이지 않은 인플루언서 조회를 했을때 data **Schema** 어디에도 알람 등록을 했는지 아닌지에 대한 정보는 담겨져 있지 않다.

```json
{
  "id": 0,
  "category": "influencer",
  "name": "string",
  "description": "string",
  "sourceType": "twitter",
  "profileImage": "string",
  "pushTopic": "string"
}
```

하지만 곰곰이 생각해보면, 구독중이지 않은 인플루언서들은 기본적으로 알람도 추가되지 않는다는 점을 고려하면, 목적에 맞게 데이터가 설계되있는 것을 볼 수 있다.

👉🏻 기능 구현보다는 어떻게 로직을 분리했는지에 초점을 맞춰보았다.

## 기존과는 다른 디렉토리 구조

![기획화면](/images/2024-11-10-sixteenth/directory.png)

👍🏻 간단하게 각 디렉토리 및 파일들의 역할을 나열해 보았다.

- domain : 리액트와 상관없는 코드들.
  - `subscribe-viewmodel.ts` : 리액트와는 상관없는 비즈니스 로직 및 유저 액션로직이 위치한 파일이다.
    - 예시) input으로 검색했을때 검색 조건에 맞는 데이터 반환
    - _구독하는 상태 업데이트_
    - _알람 상태 업데이트_
  - `subScribeAdapter.ts`: 백엔드에서 받은 데이터를 실제 화면에 보여주기 위한 데이터로 초기에 변환하는 함수가 존재.
- hooks
  - `useSubscribe.ts` : 백엔드에서 데이터를 호출하고, domain에 있는 기능들(유저액션)을 바탕으로 화면에 필요한 데이터로 바꿔서 return 하는 역할
  - `useSocialMutation.ts` : 구독 추가하거나, 알람 상태 업데이트를 하기위한 api 선언부를 각각 react-query의 useMutation훅으로 한번 감싸 내보내는 역할
- page/component
  - 우리가 보통 작성하는 컴포넌트와 페이지

## Domain 디렉토리 코드

👍🏻 Class 문법을 조금 더 효율적으로 사용해보고 싶었는데(내부 객체 인스턴스화 시켜), 그러지는 못한점 참고!

1. `subScribeAdapter.ts` : 정말 간단하게 백엔드에서 받은 데이터를 rawData(날것의) 인자로 받아 클라이언트로직에 쓸 수 있는 데이터로 정제한다.
   - **선언 코드 🧑🏼‍💻**
     ```tsx
     export class SubScribeAdapter {
       static fromSubscribe(rawData: SocialSource[]): ClientSocialSubscribed[] {
         const clientData = rawData.map((item) => ({
           ...item,
           isSubScribed: false,
           isAlert: false,
         }));
         return clientData;
       }
     }
     ```
   - **실제 사용 코드** (`react-query`의 select 옵션에서 애초에 클라이언트 데이터로 변환후 반환한다.)
     ```tsx
     // hook/useSubscribe.ts
     const { data: unSubscribedData } = useQuery(
       [...EN_GET_LIVE_LIST_TYPE('add'), category],
       () =>
         getUnsubscribedInfluencerSocialList({
           category,
           excludeSubscriptions: true,
         }),
       {
         enabled: !!category,
         select: (data) => SubScribeAdapter.fromSubscribe(data),
         suspense: true,
       }
     );
     ```
2. `subscribe-viewmodel.ts` : 클라이언트 데이터를 바탕으로 여러가지 로직을 수행한다.

   - **선언 코드 🧑🏼‍💻**

     ```tsx
     export default class SubScribeViewModel {
       // 검색 기능
       static findSubscribe(
         value: string,
         subscribeArray?: ClientSocialSubscribed[]
       ) {
         if (value.trim().length === 0) return subscribeArray;

         const searchString = value.toLowerCase();

         return subscribeArray?.filter(
           (item) =>
             item.originalName.toLowerCase().includes(searchString) ||
             item.originalDescription?.toLowerCase().includes(searchString)
         );
       }
       // 구독하고 있는 갯수
       static getSubscribesCount(
         subscribeArray?: ClientSocialSubscribed[]
       ): number {
         return subscribeArray?.length || 0;
       }

       // 구독하는 상태 업데이트
       static updateSubScribe(
         sourceId: number,
         subscribeArray?: ClientSocialSubscribed[]
       ) {
         return subscribeArray?.map((item) => ({
           ...item,
           isSubScribed:
             item.id === sourceId ? !item.isSubScribed : item.isSubScribed,
         }));
       }

       // 구독하는 상태 삭제
       static deleteSubScribe(
         sourceId: number,
         subscribeArray?: ClientSocialSubscribed[]
       ) {
         return subscribeArray?.map((item) => ({
           ...item,
           isSubScribed:
             item.id === sourceId ? !item.isSubScribed : item.isSubScribed,
           isAlert: item.id === sourceId ? false : item.isAlert,
         }));
       }

       // 알람 상태 업데이트
       static updateAlarm(
         sourceId: number,
         subscribeArray?: ClientSocialSubscribed[]
       ) {
         return subscribeArray?.map((item) => ({
           ...item,
           isAlert: item.id === sourceId ? !item.isAlert : item.isAlert,
         }));
       }
     }
     ```

👉🏻 실제 UI를 렌더링하는 컴포넌트 내부에서는 실제 로직은 몰라도된다. 함수들만 이벤트 핸들러에 부착하여 사용될 수 있다. 함수 이름을 조금 더 자세하게 작성한다면 대략적으로 이벤트 핸들러에 붙은 함수를 보고 어떤 기능을 하는 함수인치 예측할 수 있다.

## 느낀점 🔥

개발자는 데이터 중심으로 사고를 해야된다는 것을 다시 한번 느꼈다. 그리고 리액트와 리액트 바깥에 대해 구분하는 방법에 대해 고민해 보았다. 개인적으로는 리액트 바깥에서의 로직을 명확하게 분리하는게 이상적일 것이지 않을까라고 생각한다. **리액트 여야하는것과 리액트 일필요가 없는것을 구분하는 것이다.**

**모든 것을 리액트 내부에서만 작성해야 할 필요는 없다.** 만약 내가 Vue.js로 화면을 그려야한다면? 🧐 Svelete라면???🧐(이 core 로직과 라이브러리 로직을 정말 잘 분리한 라이브러리가 [Tanstack Query](https://tanstack.com/query/latest)이다.) **리액트는 정말 단지 UI Renderer 일뿐이다.** UI만 조건에 맞게 그려주는 역할을 하면된다. (처음에 `.tsx` 파일이 아닌 `.ts` 파일들에 로직들이 적혀져있으면 불안했던게 생각난다..)

👉🏻 그래서 앞으로 추구해보려는 방향은 선 비즈니스 로직 / 후 UI 보통의 개발 역순으로 개발을 하는 것이다.
조금 더 서비스 로직에 집중을 할 수도 있고, 데이터 중심으로 사고를 할 수 있으며, 진짜 프로그래밍을 하기 위함이다.

그리고 미래의 노파심 때문에 하는 말이지만, 무조건 적인 분리가 좋다고 할 수 없는건 “응집도”라는 개념때문이다. 우리가 유지보수하기 좋은 설계로 가기 위해, “높은 응집도, 낮은 결합도”를 이야기한다.

이건 정말 이상적인 기준이다. 그래서 우리는 **적절히 높은 응집도, 적절히 낮은 결합도**를 위한 프로그래밍을 해나가야한다.

개인적인 의견으로 결합도를 낮추는 방향이 조금 더 치우치면 코드가 좀 더 유연해지며, 유지보수 하기 편하지 않을까? 라는 의견이다.

## Reference 📚

- https://martinfowler.com/articles/modularizing-react-apps.html
- [쩌는 프론트엔드 관심사 분리](https://jamsilcrops-library.notion.site/bbaae5b49a37421ea7248103db8d247b)
- [쩌는 프론트엔드 관심사 분리 예시](https://jamsilcrops-library.notion.site/fcba3c9f8cb64019994f04f163abcbde)
