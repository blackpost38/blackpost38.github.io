---
title: React 컴포넌트 가독성 높이기
tags:
  - MERN
  - react
date: 2018-01-13 21:18:04
---

어느 정도 시간이 생겨 회사 솔루션에서 사용되는 React 컴포넌트 일부를 리팩토링 하기로 했다.

리팩토링으로 해결하고자 하는 큰 목적은 **유지보수를 쉽게**하는 것이었다.

_그럼 어떻게 리팩토링을 할까? 어떻게 하면 유지보수가 쉬워질까?_

결국 React 컴포넌트 뿐만 아니라 모든 소스 코드들이 그렇듯 **가독성**이 좋아지면 유지보수하기 쉬워지는 것 같다.

유지보수의 대부분이 에러 잡고, 기능 변경일 것이다.

이러한 업무를 맡으면 남이 짜놓은 소스 코드를 읽는 것은 피할 수 없다.

만약 내가 읽는 소스 코드가 복잡하면 절망감이 먼저 들며, 이해하는데 시간이 많이 걸린다.

그렇다면 어떻게 하면 React 컴포넌트의 가독성이 높아질까?

모르겠다. 그 방법을 알면 진작에 써먹었을 것이다. (젠장…)

_그렇다면 현재 React 컴포넌트의 가독성을 떨어뜨리는 것들은 무엇인가?_

차라리 이러한 요인을 제거해서 가독성을 높여 보자.

**React 컴포넌트에서 코드를 복잡하게 만드는 요인 중 하나는 너무 긴 JSX 코드인 것 같다.**

JSX 템플릿 코드가 많아지다 보니, 거기에 필요한 데이터 또는 이벤트 핸들러가 많아지면서 코드가 복잡해지는 것 같다.
<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
<span class="line">11</span>
<span class="line">12</span>
<span class="line">13</span>
<span class="line">14</span>
<span class="line">15</span>
<span class="line">16</span>
<span class="line">17</span>
<span class="line">18</span>
</pre></td><td class="code"><pre><span class="line">onClickFirst = () =&gt; &#123; /* ... */ &#125;</span>
<span class="line">onClickSecond = () =&gt; &#123; /* ... */ &#125;</span>
<span class="line">onClickThird = () =&gt; &#123; /* ... */ &#125;</span>
<span class="line"></span>
<span class="line">render () &#123;</span>
<span class="line">  const someTransformedDataFirst = /* some logic that transforms data */</span>
<span class="line">  const someTransformedDataSecond = /* some logic that transforms data */</span>
<span class="line">  const someTransformedDataThird = /* some logic that transforms data */</span>
<span class="line"></span>
<span class="line">  return (</span>
<span class="line">    &lt;div&gt;</span>
<span class="line">      &lt;div onClick=&#123;this.onClickFirst&#125;&gt;&#123;someTransformedDataFirst&#125;&lt;/div&gt;</span>
<span class="line">      &lt;div onClick=&#123;this.onClickSecond&#125;&gt;&#123;someTransformedDataSecond&#125;&lt;/div&gt;</span>
<span class="line">      &lt;div onClick=&#123;this.onClickThird&#125;&gt;&#123;someTransformedDataThird&#125;&lt;/div&gt;</span>
<span class="line">      /* ... */</span>
<span class="line">    &lt;/div&gt;</span>
<span class="line">  );</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>

**그리고 다른 요인은 데이터를 JSX에 표시하기 위해서 변형하는 로직이다.**

이러한 로직이 `render` 메서드에 있다보니, 정작 React 컴포넌트가 어떤 형태의 UI인지 눈에 들어 오지 않고 스크롤 압박만 준다.
<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
<span class="line">11</span>
<span class="line">12</span>
<span class="line">13</span>
<span class="line">14</span>
<span class="line">15</span>
<span class="line">16</span>
<span class="line">17</span>
<span class="line">18</span>
<span class="line">19</span>
<span class="line">20</span>
<span class="line">21</span>
</pre></td><td class="code"><pre><span class="line">render () &#123;</span>
<span class="line">  const someData = this.props.someData;</span>
<span class="line">  const transformedData = /*</span>
<span class="line">    some complex logic that transforms data</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">  */</span>
<span class="line"></span>
<span class="line">  return (</span>
<span class="line">    &lt;div&gt;&#123;transformedData&#125;&lt;/div&gt;</span>
<span class="line">  );</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>

_이 문제들의 공통점이 있다. 바로 `render` 메서드가 복잡하다는 것이다._

그러므로 `render` 메서드를 깔끔하게 만들면 가독성이 올라갈 것이다.

_그럼 `render` 메서드를 어떻게 깔끔하게 만들까?_

**JSX 코드가 너무 길어진다면 다른 컴포넌트로 추출해서 JSX 코드의 길이를 줄이는 방법이 있다.**

이것이 단지 코드 라인 수를 줄여 주는 것뿐만 아니라, 단순히 길었던 엘리먼트가 의미있는 컴포넌트로 대체되면서 가독성을 높여 준다.
<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
</pre></td><td class="code"><pre><span class="line">render () &#123;</span>
<span class="line">  return (</span>
<span class="line">    &lt;div&gt;</span>
<span class="line">      &lt;First someDataFirst=&#123;someDataFirst&#125; /&gt;</span>
<span class="line">      &lt;Second someDataFirst=&#123;someDataSecond&#125; /&gt;</span>
<span class="line">      &lt;Third someDataFirst=&#123;someDataThird&#125; /&gt;</span>
<span class="line">      /* ... */</span>
<span class="line">    &lt;/div&gt;</span>
<span class="line">  );</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>

파일 만들고, 새로운 컴포넌트 만들고, Props 옮겨주는 작업 귀찮은 거 알면서도… 그래도 그만큼 값어치 한다.

디버깅 할때 눈에 훨씬 잘들어 오고, 코드 읽기도 훨씬 편해진다. 한 번 수고하고 그 이후 유지보수 쉽게 가자.

**데이터를 변형하는 로직이 긴 경우엔 `render` 메서드 말고, `getter` 메서드 따로 선언해주고 그 안에 작성하자.**

그래야 `render`가 메인으로 하는 역할인 JSX 코드 리턴해주는 부분이 훨씬 눈에 잘 들어온다.

`getter` 메서드 로직은 `render` 메서드가 어떻게 생긴지 보고 난 후에 봐도 늦지 않는다.

만약, `getter` 메서드가 다른 컴포넌트에서도 쓰인다면 **selector**를 별도로 만들어 주는 것도 생각해볼만 하다.

또한, `render*` 메서드를 활용해서 data 변형과 JSX를 리턴하는 로직을 같이 추출할 수도 있다.
<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
<span class="line">11</span>
<span class="line">12</span>
<span class="line">13</span>
<span class="line">14</span>
<span class="line">15</span>
<span class="line">16</span>
<span class="line">17</span>
<span class="line">18</span>
<span class="line">19</span>
<span class="line">20</span>
<span class="line">21</span>
<span class="line">22</span>
<span class="line">23</span>
<span class="line">24</span>
<span class="line">25</span>
</pre></td><td class="code"><pre><span class="line">getTransformedData = () =&gt; &#123;</span>
<span class="line">  const transformedData = /*</span>
<span class="line">    some logic that transforms data</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">    .</span>
<span class="line">  */</span>
<span class="line">  return transformedData;</span>
<span class="line">&#125;</span>
<span class="line"></span>
<span class="line">render () &#123;</span>
<span class="line">  const transformedData = this.getTransformedData();</span>
<span class="line"></span>
<span class="line">  return (</span>
<span class="line">    &lt;div&gt;&#123;transformedData&#125;&lt;/div&gt;</span>
<span class="line">  );</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>

이 긴 내용을 요약하자면, React 컴포넌트의 가독성을 올리기 위해 `render` 메서드를 정리했다는 것이다.

그러면서 컴포넌트 내에 다른 메서드를 선언하여 로직을 추출하기도 했고, 아예 다른 컴포넌트를 만들어 추출하기도 했다.

끝으로 이 글에 대한 피드백이나, React 컴포넌트 가독성을 위해 고민했던 생각이나 다른 아티클이 있다면 댓글로 남겨주길 바란다.