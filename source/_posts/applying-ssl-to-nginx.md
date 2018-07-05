---
title: Nginx에 SSL 적용하기
tags:
  - Web Fundamental
  - nginx
  - SSL
date: 2018-01-13 21:18:04
---

## [](#Let’s-Encrypt "Let’s Encrypt")Let’s Encrypt

*   Let’s Encrypt는 무료로 TLS/SSL 증명서를 제공해주고 설치해주는 인증기관(Certificate Authority)이다.
*   letsencrypt는 무료 TSL/SSL 증명서를 발급하기 위한 절차(process)를 간단하게 해주는 도구이다.
*   여기서 letsencrypt를 이용하여 SSL 증명서를 발급받고 자동으로 갱신하는 방법을 살펴보자.

## [](#시작하기-전 "시작하기 전")시작하기 전

*   도메인이 매핑되어 있는 nginx가 설치되어 있는 ubuntu
*   증명서를 발급할 때 도메인 이름으로 유효성 검증을 거친다.
*   서브 도메인에 매핑하고 싶으면 A record에 등록된 서브 도메인 이어야 한다.

## [](#letsencrypt-client-설치하기 "letsencrypt client 설치하기")letsencrypt client 설치하기

*   apt 패키지를 업데이트 하고 letsencrypt를 설치해보자.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line">&gt; sudo apt-get update</span>
<span class="line">&gt; sudo apt-get install letsencrypt</span>
</pre></td></tr></table></figure>

## [](#SSL-증명서-얻기 "SSL 증명서 얻기")SSL 증명서 얻기

*   플러그인 사용하기

        *   letsencrypt는 다양한 플러그인을 이용하여 SSL 인증서를 얻을 수 있다.
    *   이 플러그인들은 서버에 인증서를 발급할지 여부만 따지므로 인증자(authenticators)라고도 한다.
    *   여기서는 Webroot를 이용한다.
    *   Webroot는 ./well-known 폴더에 특수한 파일을 놓고 작동한다.
    *   따라서 nginx 설정을 변경하여 ./well-known 경로에 접근할 수 있도록 해준다.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">location ~ /.well-known &#123;</span>
<span class="line">    allow all;</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>

*   nginx 설정을 변경하였으니 설정이 맞게 되었는지 확인하고 설정을 적용하기 위해 nginx를 재시작을 해준다.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line">&gt; nginx -t</span>
<span class="line">&gt; systemctl restart nginx</span>
</pre></td></tr></table></figure>

*   letsencrypt 명령어를 이용하여 인증서를 만들어 주자.

        *   `&gt; letsencrypt -a webroot --webroot-path=/var/www/html -d blackpost.xyz`
    *   -a: 우리가 사용할 plugin 이름
    *   –webroot-path: /.well-known의 경로
    *   -d: 도메인 이름
    *   중간에 prompt가 나타나서 이메일과 동의 여부를 묻는다.

                *   이메일은 key 분실 시 필요한 정보이다.
        *   letsencrypt로 4개의 .pem 파일이 만들어 진다.

                        *   cert.pem: 도메인 인증서
            *   chain.pem: letsencrypt 연결 인증서
            *   fullchain.pem: cert.pem과 chain.pem이 결합된 인증서
            *   privkey.pem: 인증서의 private key
            *   /etc/letsencrypt/live/domain_name 경로에 링크가 되어 만들어 진다.
*   보안 수준을 올리기 위해 strong diffie-hellman 그룹을 만들어 준다.

        *   `openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048`
    *   /etc/ssl/certs/dhparam.pem 경로에 strong DH 그룹이 만들어진다.

## [](#nginx에-TSL-SSL-설정하기 "nginx에 TSL/SSL 설정하기")nginx에 TSL/SSL 설정하기

*   SSL 키와 인증서를 가르키는 설정 파일 만들기

        *   `nano /etc/nginx/snippets/ssl-blackpost.xyz.conf`
    *   snippets 폴더에 conf 파일을 아래와 같은 내용으로 생성해준다.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line">ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;</span>
<span class="line">ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;</span>
</pre></td></tr></table></figure>

*   Strong encryption 세팅하기

        *   `nano /etc/nginx/snippets/ssl-params.conf`
    *   ssl_dhparam 세팅 추가

                *   앞서 만들어준 strong diffie-hellman 그룹 경로 추가
    *   HTTP Strict Transport Security에 preload 기능 제거

                *   실수나 잘못 설정되어 있는 경우 광범위한 결과(?)를 초래할 수 있다.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line">add_header Strict-Transport-Security &quot;max-age=63072000; includeSubdomains&quot;;</span>
<span class="line">ssl_dhparam /etc/ssl/certs/dhparam.pem;</span>
</pre></td></tr></table></figure>

*   Nginx config 수정

        *   nginx config 파일을 연다.

                *   `nano /etc/nginx/sites-available/default`
    *   default에서 http request를 https request로 redirect를 시키기 위한 설정을 추가한다.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line">server &#123;</span>
<span class="line">    listen 80 default_server;</span>
<span class="line">    listen [::]:80 default_server;</span>
<span class="line">    server_name example.com www.example.com;</span>
<span class="line">    return 301 https://$server_name$request_uri;</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>

*   바로 아래 https redirect 될 경우 적용할 server block을 만들어 준다.

        *   443 port를 사용하며 ssl 인증서를 사용하며, http2도 지원해준다.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line">server &#123;</span>
<span class="line">    # SSL configuration</span>
<span class="line">    listen 443 ssl http2 default_server;</span>
<span class="line">    listen [::]:443 ssl http2 default_server;</span>
<span class="line">    include snippets/ssl-example.com.conf;</span>
<span class="line">    include snippets/ssl-params.conf;</span>
<span class="line">. . .</span>
</pre></td></tr></table></figure>

*   수정한 설정을 적용하기 위해 nginx 설정을 확인하고 재시작한다.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line">&gt; nginx -t</span>
<span class="line">&gt; systemctl restart nginx</span>
</pre></td></tr></table></figure>

*   [https://www.ssllabs.com/ssltest/analyze.html?d=example.com](https://www.ssllabs.com/ssltest/analyze.html?d=example.com) 로 접속하여 적용한 SSL 레포트를 확인해 볼 수 있으며 A+가 나와야 한다.

## [](#인증서-자동-갱신 "인증서 자동 갱신")인증서 자동 갱신

*   인증서를 만들면 90 동안 유효기간이나 여유 있게 60일 정도 되면 갱신하는 게 좋다.
*   letsencrypt는 renew 커맨드를 제공해주며 인증서를 갱신해 준다.
*   만약 만기일이 30일 이하면 인증서를 갱신하나 그렇지 않으면, 만기일만 확인하고 끝난다.
*   실질적인 갱신 방법은 crontab에 갱신 작업을 등록하는 것이다.

        *   월요일에 한 번씩 갱신하도록 crontab에 작업을 추가하자.<figure class="highlight plain"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">&gt; crontab -e</span>
<span class="line">30 2 * * 1 /usr/bin/letsencrypt renew &gt;&gt; /var/log/le-renew.log</span>
<span class="line">35 2 * * 1 /bin/systemctl reload nginx</span>
</pre></td></tr></table></figure>

## [](#참고 "참고")참고

*   [https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)
*   port 80으로 설정되어야 letsencrypt certonly command가 에러 없이 실행된다.