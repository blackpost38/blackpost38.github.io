---
title: Array.sort 부적절한 예제
tags:
  - Web Fundamental
  - javascript
date: 2018-01-13 21:18:04
---

`Array.sort`는 파라미터로 **compareFunction**을 넘겨줄 수 있다.

compareFunction은 0, 음수 또는 양수를 리턴하면서 배열의 순서를 정해준다.

MDN에서 compareFunction 예제로 `function (a, b) { return a - b; }`를 봤다.

내가 이것을 이용한다고 아래와 같이 올바르지 않은 compareFunction을 작성한 적이 있다.
<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">function (a, b) &#123; return a &gt; b; &#125;</span>
</pre></td></tr></table></figure>

위는 앞서 compareFunction을 설명했던 것과 달리 오로지 `true` 혹은 `false`를 반환한다.

따라서 이 boolean 값은 compareFunction의 리턴값으로 적절하지 못하다.

**문제는 이것이 알파벳 순 혹은 Ascending 형식으로 크롬에서 동작한다. 그리고 IE11에선 안된다.**

어떤 브라우저는 되고 안되고 해서 디버깅하기 까다로운 부분이므로 주의해야 한다.
<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line">// Chrome</span>
<span class="line">[&apos;sub0&apos;, &apos;sub2&apos;, &apos;sub1&apos;].sort((a, b) =&gt; a &gt; b);  // [&apos;sub0&apos;, &apos;sub1&apos;, &apos;sub2&apos;]</span>
<span class="line"></span>
<span class="line">// IE11</span>
<span class="line">[&apos;sub0&apos;, &apos;sub2&apos;, &apos;sub1&apos;].sort((a, b) =&gt; a &gt; b);  // [&apos;sub0&apos;, &apos;sub2&apos;, &apos;sub1&apos;]</span>
</pre></td></tr></table></figure>