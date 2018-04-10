---
title: 도커 컨테이너 끼리 통신
tags:
  - DevOps
  - docker
date: 2018-01-13 21:18:05
---

# [](#이슈 "이슈")이슈

## [](#현상 "현상")현상

*   도커 컨테이너 끼리의 통신이 연결되지 않는다.

## [](#상황 "상황")상황

*   컨테이너 A에서 컨테이너 B의 API를 호출하고 싶다.
*   컨테이너 A는 localhost 3000 포트로 설정되어 있다.
*   컨테이너 B는 localhost 4000 포트로 설정되어 있다.
*   컨테이너 A에서 localhost 4000 포트로 Request를 보내면 응답하지 않는다.

## [](#원인 "원인")원인

*   도커 컨테이너가 자신의 localhost가 다른 의미를 갖게 된다.
*   도커 컨테이너의 IP를 살펴보면, `172.17.0.X` 같은 별도의 로컬 IP를 갖게 된다.
*   도커 컨테이너는 기본적으로 도커 네트워크 **bridge**에 속한다.
*   따라서 도커 컨테이너에서 localhost의 의미는 자기 자신의 컨테이너 IP를 가르키게 되버린다. (내 컴퓨터의 IP가 아닌!!)<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">&gt; docker run -itd --name=container1 busybox</span>
<span class="line">&gt; docker run -itd --name=container2 busybox</span>
<span class="line">&gt; docker network inspect bridge</span>
<span class="line"></span>
<span class="line">...</span>
<span class="line">&quot;Containers&quot;: &#123;</span>
<span class="line">  &quot;0f65120b94a56c26124322ce272b74657b46ad12176caa79a576c660b7c01864&quot;: &#123;</span>
<span class="line">      &quot;Name&quot;: &quot;container1&quot;,</span>
<span class="line">      &quot;EndpointID&quot;: &quot;f71fd32ada7721d1412d0ceca764a6890e8902eaad1fd9bbfde39030ff717b89&quot;,</span>
<span class="line">      &quot;MacAddress&quot;: &quot;02:42:ac:11:00:02&quot;,</span>
<span class="line">      &quot;IPv4Address&quot;: &quot;172.17.0.2/16&quot;,</span>
<span class="line">      &quot;IPv6Address&quot;: &quot;&quot;</span>
<span class="line">  &#125;,</span>
<span class="line">  &quot;13bf17f4b7fec846a71c15efdf3d7264ed289ff2f7b70c258ae4faaf567f6f5c&quot;: &#123;</span>
<span class="line">      &quot;Name&quot;: &quot;container2&quot;,</span>
<span class="line">      &quot;EndpointID&quot;: &quot;a3379ded18d90c0e9d607982cda7b93a87951b0c042eb59a0c76fe59f31ce8fa&quot;,</span>
<span class="line">      &quot;MacAddress&quot;: &quot;02:42:ac:11:00:03&quot;,</span>
<span class="line">      &quot;IPv4Address&quot;: &quot;172.17.0.3/16&quot;,</span>
<span class="line">      &quot;IPv6Address&quot;: &quot;&quot;</span>
<span class="line">  &#125;</span>
<span class="line">&#125;,</span>
<span class="line">...</span>
</pre></td></tr></table></figure>

# [](#해결법 "해결법")해결법

## [](#도커-컨테이너-IP로-통신하기 "도커 컨테이너 IP로 통신하기")도커 컨테이너 IP로 통신하기

*   도커 컨테이너끼리 기본적으로 bridge라는 동일한 도커 네트워크에 속해 있으므로 IP를 입력하여 통신할 수 있다.
<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line">&gt; docker attach container1`</span>
<span class="line">&gt; ping 172.17.0.3</span>
<span class="line"></span>
<span class="line">PING 172.17.0.3 (172.17.0.3): 56 data bytes</span>
<span class="line">64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.140 ms</span>
<span class="line">64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.116 ms</span>
<span class="line">64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.159 ms</span>
</pre></td></tr></table></figure>
*   문제점은 도커 컨테이너가 재시작 되면 이 IP가 변경될 가능성이 있다.

*   언제든지 변경될 가능성이 생기기 때문에 웹 어플리케이션에 다이나믹한 IP를 설정하기엔 어려움이 있다.

## [](#도커-컨테이너-이름으로-통신하기 "도커 컨테이너 이름으로 통신하기")도커 컨테이너 이름으로 통신하기

*   도커 컨테이너 이름을 명시해서 통신하는 방법이 있다.
*   그러기 위해서 컨테이너를 시작할 때 `--link` 옵션을 사용한다.
<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
</pre></td><td class="code"><pre><span class="line">&gt; docker run --link container2:container2 -itd --name=container1 busybox</span>
<span class="line">&gt; docker attach container1</span>
<span class="line">&gt; ping container2</span>
<span class="line"></span>
<span class="line">PING container2 (172.21.0.3): 56 data bytes</span>
<span class="line">64 bytes from 172.21.0.3: seq=0 ttl=64 time=0.268 ms</span>
<span class="line">64 bytes from 172.21.0.3: seq=1 ttl=64 time=0.102 ms</span>
<span class="line">64 bytes from 172.21.0.3: seq=2 ttl=64 time=0.163 ms</span>
</pre></td></tr></table></figure>
*   `--link` 옵션을 추가하면 뭐가 달라질까?

    *   옵션 내용이 `/etc/hosts`에 반영된다.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">&gt; docker attach container1</span>
<span class="line">&gt; cat /etc/hosts</span>
<span class="line"></span>
<span class="line">127.0.0.1       localhost</span>
<span class="line">::1     localhost ip6-localhost ip6-loopback</span>
<span class="line">fe00::0 ip6-localnet</span>
<span class="line">ff00::0 ip6-mcastprefix</span>
<span class="line">ff02::1 ip6-allnodes</span>
<span class="line">ff02::2 ip6-allrouters</span>
<span class="line">172.17.0.2      container2 d759e0e85689</span>
<span class="line">172.17.0.3      87c4059eb3b6</span>
</pre></td></tr></table></figure>

## [](#알고보니-레거시-옵션 "알고보니 레거시 옵션")알고보니 레거시 옵션

*   `--link` 옵션은 현재 레거시 옵션으로 분류되어 있으며 나중에 언제라도 없어질 수 있다.
*   도커 문서에서는 직접 네트워크를 정의하여 추가하는 방법을 제안한다.

## [](#유저-정의-네트워크 "유저 정의 네트워크")유저 정의 네트워크

*   bridge 네트워크 같은 도커 네트워크를 사용자가 직접 정의할 수 있다.
*   장점은 컨테이너 이름을 IP로 풀어주는 DNS가 자동으로 세팅된 다는 점이다.

        *   즉, 컨테이너 이름으로 통신이 가능하다는 것
*   네트워크 정의한 후 도커 컨테이너 포함 시키기
<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">&gt; docker network create my-network</span>
<span class="line">&gt; docker network connect my-network container1</span>
<span class="line">&gt; docker network connect my-network container2</span>
</pre></td></tr></table></figure>
*   포함 시킨 후 container1에서 ping을 실행해보자
<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line">&gt; docker attach container1</span>
<span class="line">&gt; ping container2</span>
<span class="line"></span>
<span class="line">PING container2 (172.21.0.3): 56 data bytes</span>
<span class="line">64 bytes from 172.21.0.3: seq=0 ttl=64 time=0.268 ms</span>
<span class="line">64 bytes from 172.21.0.3: seq=1 ttl=64 time=0.102 ms</span>
<span class="line">64 bytes from 172.21.0.3: seq=2 ttl=64 time=0.163 ms</span>
</pre></td></tr></table></figure>