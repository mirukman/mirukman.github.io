---
title: Ubuntu MySQL 한글 에러 해결
author: devbooja
date: 2020-10-03 17:00:00 +0800
categories: [DB, MySQL]
tags: [mysql한글]
---

Ubuntu 18.04 버전에서 MySQL 설치 후 테이블에 한글 데이터 입력이 되지 않는 문제

![한글에러](https://devbooja.github.io/images/db/mysql/mysql-hangul-error/insert-error.png){: width="100%" height="100%"}


<br>

이 문제는 mysql의 character-set이 UTF8로 잡혀있지 않아서 생기는 문제이다.


먼저 mysql에 접속 후 아래 명령어를 쳐보자

~~~ shell
show variables like 'c%';
~~~

![캐릭터셋](https://devbooja.github.io/images/db/mysql/mysql-hangul-error/mysql-character-variables.png){: width="100%" height="100%"}

'latin'으로 되어있는 설정들이 몇 보인다. 이 부분들을 utf8로 변경해주어야한다.

<br>

### 해결방법 ###

MySQL 설정 파일에 character-set 설정들을 추가해주어야한다.

오래된 블로그 글들에는 /etc/mysql/my.cnf 파일에 설정들을 추가하라는 글들이 많이 보이지만, MySQL 5.x 버전부터는 설정 파일들의 구조가 바뀌어 my.cnf 파일에 설정 내용들이 없다. 물론 여기에다 추가해도 되지만 바뀐 구조에 맞게 변경해보자.

설정 파일 및 폴더 수만해도 여러가지인데, MySQL 서버가 실행될 때 읽어들이는 설정파일들 중 한 곳에 추가하면 된다.

나같은 경우는 **/etc/mysql/conf.d/mysql.cnf**에 아래 내용들을 추가했다.

<br>
~~~ vim
[mysql]
default-character-set=utf8

[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci
init_connect = set collation_connection = utf8_general_ci
init_connect = set names utf8

[mysqld_safe]
default-character-set=utf8

[client]
default-character-set=utf8

[mysqldump]
default-character-set=utf8
~~~
<br>

그리고 shell에서 다음 명령어를 입력해서 mysql을 재시작 해주자.

~~~ shell
sudo systemctl restart mysql
~~~

이후 다시 mysql에 접속해서 변경된 캐릭터셋을 확인해보면 아래 그림과 같이 'latin'이던 캐릭터셋들이 'utf8'로 변경된 것을 확인할 수 있다.

![캐릭터셋](https://devbooja.github.io/images/db/mysql/mysql-hangul-error/changed-character-set.png){: width="100%" height="100%"}


이후 다시 mysql에서 데이터 조회, 입력 등을 하니 문제가 없다.
