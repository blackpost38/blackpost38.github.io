---
title: unable to find a live medium containing a live file system 에러 메시지
tags:
  - DevOps
  - Ubuntu
date: 2018-01-20 20:20:54
---

# [](#이슈 "이슈")이슈

## [](#현상 "현상")현상

*   Ubuntu 16 설치 용 USB를 컴퓨터에 꽂고 인스톨을 하면 다음과 같은 에러 메시지가 뜬다.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">(initramfs) unable to find a live medium containing a live file system</span>
</pre></td></tr></table></figure>

## [](#원인 "원인")원인

*   다양해서 파악하기 힘들다. (참조에서 살펴보자)

# [](#해결법 "해결법")해결법

*   하나하나 원인이 될만한 것을 골라서 시도하는 수밖에 없는 것 같다.
*   그중에서 기존에 USB 3.0을 2.0으로 바꿔서 부팅 USB를 만드니 정상적으로 설치가 되었다.

        *   USB 포트에만 버전이 있는 줄 알았고, USB 자체에 버전이 있는 줄 몰랐다.
*   실제로 시도했던 것들

        *   우분투 설치 파일 이미지에 문제가 있다고 생각해서 다시 다운로드 받음
    *   USB 자체에 문제가 있다고 생각해서 다른 USB로도 만들어봄 (안타깝게도 둘 다 USB3.0이었다)
    *   부팅 디스크를 만드는 소프트웨어에 문제가 있다고 가정

# [](#참조 "참조")참조

*   [에러 메시지 원인과 해결책 1](https://forum.ubuntu-kr.org/viewtopic.php?t=27386)*   [에러 메시지 원인과 해결책 2](https://askubuntu.com/questions/15425/error-when-installing-unable-to-find-a-medium-containing-a-live-file-system)
*   [USB 2.0과 3.0 차이](http://tip.daum.net/question/89798349)
*   [Mac OS에서 Software로 Ubuntu 부팅 디스크 만들기](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-macos)
*   [Mac OS에서 CLI로 Ubuntu 부팅 디스크 만들기](http://sergeswin.com/1178)