---
layout: post
title: GitLab Nginx에 vhost 설정하기
---

- 작성일 : 2018-07-27 (금)
- GitLab버전 : GitLab Community Edition 8.17.3 (빨리 업데이트 필요..)

기존에 사용하던 IDC와의 계약 해지로 인해 IDC에 있던 서버들을 모두 본사로 이전 또는 클라우드로 전환을 계획하였음.

본사의 서버실은 좁고, 시설이 제대로 갖춰져있지 않아 서버의 갯수를 최소한으로 사용하기 위해
회사 홈페이지, 회사 내부 서비스, GitLab 빌드서버를 GitLab이 설치되어 있는 서버 하나로 사용해야 하는 상황

기존 GitLab이 GitLab에 내장된 Nginx 서버를 사용하고 80포트를 사용하고 있기 때문에, 이전하는 사이트가 80포트를 사용하기 위해서는 vhost 설정을 해줘야 했다.

## html 폴더 생성

1. /var/www 경로로 이동하여 디렉터리를 생성한다.
```
[root@gitlab www]# cd /var/www
[root@gitlab www]# mkdir homepage
```

2. 생성한 디렉터리 내부에 logs와 public_html 디렉터리를 생성한다.
```
[root@gitlab www]# mkdir logs
[root@gitlab www]# mkdir public_html
```
3. public_html 폴더에 기존 사이트의 파일들을 이동

## vhost 설정

1. GitLab에 내장된 Nginx의 설정을 수정해야 하기때문에 해당 경로로 이동
```
[root@gitlab ~]# cd /var/opt/gitlab/nginx/conf
```
2. 해당 경로의 파일 목록을 보면 아래와 같은 설정파일들이 있는 것을 확인 할 수 있다.
```
[root@gitlab conf]# ll
-rw-r--r--. 1 root root 4504 Jul  5 17:53 gitlab-http.conf
-rw-r--r--. 1 root root 1496 Jul  5 17:44 nginx.conf
-rw-r--r--. 1 root root  201 May 11  2017 nginx-status.conf
```
3. vhost 설정을 위해 설정 파일 하나를 추가
```
[root@gitlab conf]# vi homepage-http.conf
```
4. 아래와 같이 작성
```
server
{
        listen *:80;
        server_name test.co.kr www.test.co.kr;

        access_log /var/www/homepage/logs/access.log;
        error_log /var/www/homepage/logs/error.log;

        root    /var/www/homepage/public_html;
}
```
- server_name : server_name으로 입력한 도메인으로 접속했을 때, 경로로 설정한 html을 보여줌
- access_log : access_log가 저장되는 위치
- error_log : error_log가 저장되는 위치
- root : html파일이 위치한 경로
5. nginx 설정 수정
```
[root@gitlab conf]# vi nginx.conf
```
6. 파일 하단에 추가한 설정파일 include (nginx-status.conf 하단에)  
```  
include /var/opt/gitlab/nginx/conf/gitlab-http.conf;  
include /var/opt/gitlab/nginx/conf/nginx-status.conf;  
  
include /var/opt/gitlab/nginx/conf/homepage-http.conf;  
```  
7. GitLab nginx 재구동
```
[root@gitlab conf]# gitlab-ctl nginx restart
```

### 문제점

위와 같이 재구동하고 해당 도메인으로 접속하면 문제 없이 잘 된다.
그러나.....

GitLab 자체를 재구동할경우 nginx.conf 파일이 다시 수정전으로 돌아갔다.

추가로 GitLab 설정파일도 수정을 해야한다.

1. GitLab 설정 파일 수정을 위해 아래 경로로 이동하여 gitlab.rb 파일 수정
```
[root@gitlab gitlab]# cd /etc/gitlab
[root@gitlab gitlab]# vi gitlab.rb
```
2. custom_nginx_config의 주석을 풀고 추가한 설정파일을 include해준다.
```
nginx['custom_nginx_config'] = "include /var/opt/gitlab/nginx/conf/homepage-http.conf;"
```
3. GitLab 재구동
```
[root@gitlab conf]# gitlab-ctl restart
```

위와 같이 수정하면 문제 없이 잘 되는 것을 확인할 수 있다.
