---
title: 데이터를 Redux store 대신 React state에 저장해도 될까요?
tags:
  - MERN
  - react
  - redux
date: 2018-01-13 21:18:04
---

> 아래 있는 이전 글을 [react korea](https://www.facebook.com/groups/1413798668880301?id=1850080511918779)에 공유하고 만족할 만한 피드백을 받았다.
> 여기에 피드백을 받고 생각이 바뀐 부분이 있어서 다시 정리하고자 한다. -원본은 멘 아래에 있다.-

## [](#모든-데이터를-Redux-store에-관리하려는-이유 "모든 데이터를 Redux store에 관리하려는 이유")모든 데이터를 Redux store에 관리하려는 이유

처음 data를 Redux store에 관리할지 React state에 관리할지에 대한 내 생각은 아래와 같았다.
> _데이터를 Redux store에 관리하는 것이 기본이다._
> _단, 어플리케이션 전반적으로 영향을 끼치지 않는 데이터는 React state로 두어도 괜찮다_

이유는 사용자 행동을 action으로 하나하나 정의하고 로깅으로 남겨두면, **사용자 행동을 추적할 수 있어 디버깅이 편할 것 같았다.**

그래서 사용자의 action에 영향을 받는 모든 데이터를 Redux store로 관리하는 것이 기본이라고 생각했다.

## [](#모든-데이터를-Redux-store에-관리하면-생기는-위험 "모든 데이터를 Redux store에 관리하면 생기는 위험")모든 데이터를 Redux store에 관리하면 생기는 위험

그런데 이러한 방식이 **오히려 데이터 변경 이력을 지저분하게 만들 수 있다**는 피드백을 받았다.

데이터 변경 이력에 대한 피드백을 곰곰이 생각해보니, 고객사에 가서 log를 뒤지던 나의 모습이 떠올랐다.

유의미한 log를 찾기 위해 무의미한 log를 수없이 필터링한 것이 기억이 났다.

이와 비슷하게 너무 많은 action이 정말로 중요한 action을 가릴 것 같은 생각이 들었다.

또한, **퍼포먼스에 문제가 생길 수 있다**는 의견도 받았다.

예를 들어, 폼을 입력하는 것까지 Redux로 관리하면 부하가 상당할 수도 있다는 내용이었다.

퍼포먼스와 관련된 내용은 [Redux 문서](http://redux.js.org/docs/faq/Performance.html#wont-calling-all-my-reducers-for-each-action-be-slow)에서 아래와 같이 안내하고 있어 별로 걱정하지 않았다.

1.  JavaScript engine의 충분히 좋은 성능
2.  action이 dispatch 될 때 대부분 reducer는 default를 실행하므로 실질적으로 실행되는 코드는 적다는 것이었다.
3.  만약 문제가 되면 패키지(redux-ignore)를 사용해서 특정 action에 특정 reducer만 작동하게끔 만들 수도 있다는 내용이었다.

결국 문제가 되면 패키지를 이용해서 reducer를 커스터마이징 하라는 건데, 생각보다 많은 사람이 그 문제를 겪고 있는 것 같다.

그래서 action, reducer를 하나하나 정의해서 퍼포먼스 문제를 일으킬 가능성을 높히고, 커스터마이징 하는 수고까지 하기엔 비용이 굉장히 크다고 판단했다.

## [](#결론 "결론")결론

따라서 다시 정리한 내 생각은 아래와 같다.

_되도록이면 지역적으로 사용하는 데이터는 React state로 관리하자._

생각 살짝 바꾸는게 이리 힘들 줄이야
> [Q&amp;A](http://redux.js.org/docs/faq/OrganizingState.html#do-i-have-to-put-all-my-state-into-redux-should-i-ever-use-reacts-setstate)에 이와 같은 주제를 다룬 내용이 있다.

* * *
> 여기서 부터 원본이다.

Redux store에 데이터를 추가하는 일은 단순하면서도 귀찮은 작업이다.

Action, Reducer 그리고 Action을 dispatch 하는 곳 모두 수정해야 하기 때문이다.

때문에 번거로운 작업에서 벗어나기 위해 React state의 유혹을 받는다.

하지만 규모가 큰 React state를 관리하기 위해 Redux store를 사용하지 않던가…!

_나는 Redux store 중에서 React state로 보관해도 문제가 없는 데이터에 대해 생각했다._

_그리고 고민 끝에 지역적으로 사용하는 데이터는 React state로 사용해도 되겠다는 판단을 내렸다._
> 중요한 것은 local state는 React state로 관리해야 한다가 아닌, 관리해도 된다는 것이다.

여기서 지역적으로 사용한다는 것은 하나 또는 일부 컴포넌트 내에서만 사용하는 데이터이다.

비유하자면 데이터를 변수라고 가정하면, 지역 변수는 React state가 될 것이며, 전역 변수는 Redux Store가 될 것이다.

이 데이터의 특징은 Redux store로 굳이 관리하지 않아도, 어플리케이션 전반에 걸쳐 사용 되지 않고 일부에만 사용된다.

예를 들어, checkbox의 checked 데이터나 modal의 show/off 데이터는 해당 컴포넌트에만 사용된다.

따라서 이러한 데이터들은 React state로 관리해도 무방하다.

validation check가 필요한 input 컴포넌트의 데이터도 전반적으로 사용되는 것은 아니므로 React state로 관리해도 된다.