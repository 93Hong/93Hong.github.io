---
layout: post
title:  "Transaction과 Auto Increment"
date:   2018-01-29
comments: true
---

<p class="intro"><span class="dropcap">Y</span>ou should never depend on the numeric features of autogenerated keys.</p>

오늘은 메일 서비스를 구현하면서 맞닥뜨린 문제를 포스팅한다.
메일 서비스에서 메일 보내기를 구현할 때, 한 메일에 대해서 5개의 Table에 insert를 해야했다.
각 Table에 insert를 하는 도중 에러가 발생한다면 insert된 값들을 rollback 해야하므로 transaction을 찾아보았다.
다행히 Spring에서 @Transactional anotation으로 쉽게 transcation 기능을 제공받을 수 있었고 별 문제없이(?) 진행됬다.

Transaction을 걸었고 5개의 Table에서 하나의 key를 PK로 사용하게 구현하였고 IO에서 이점을 얻기위해 Table을 수평분할했다.
단순히 수평분할이라 key를 AUTO_INCREMENT로 설정한다면 5개의 Table이 문제없이 항상 같은 row에 대해서 같은 PK 값을 가질 것이라는 생각을 했다.

Transaction이 잘 구현되었는지 테스트를 하는 도중에 AUTO_INCREMENT를 여러 Table에서 PK로 사용하는 방법의 문제점을 깨달았다.
3개의 Table에 insert 후 4번 째 Table에 insert가 실패했을 때, transaction은 정상적으로 동작해서 앞서 insert한 Table의 값은 rollback 되었지만, AUTO_INCREMENT 값은 이미 증가한 상태여서 PK가 5개의 Table에서 다른 값을 가지는 상황이 발생했다.

단순 오류라 생각하고 API 문서를 찾아보았지만, Transaction은 AUTO_INCREMENT로 증가한 값의 rollback을 보장하지 않는다는 것이였다.
물론 타당한 이유가 있었고 이 문제를 해결하기 위해 다른 방법을 찾아야했다.

그렇다면 왜 Transaction은 AUTO_INCREMENT의 rollback을 보장하지 않을까?

예시를 들어보자.
한 프로그램이 실행되면서 100이라는 AUTO_INCREMENT key값을 DB에 남겼다.
다음 프로그램은 101이라는 값을 남겼고 성공적으로 종료되었다.
이런 상황에서 만약 100을 남긴 프로그램이 rollback 되었다면, 어떤 상황이 벌어질까?
완벽한 Transaction을 위해 사라진 100을 채우기 위해 101의 값들을 모두 100으로 당겨야할까?
만약 당겨야 한다면, 101을 FK로 참조하는 다른 Table의 값들은 어떻게 될까? 또 첫 프로그램의 결과 값을 100이라는 값으로 가져오려는 프로그램은 어떻게 처리해야할까?

이런 이유로 오라클 등에서는 AUTO_INCREMENT로 변화한 값을 Transaction시 복구하지 않는다.

그렇다면 여러 Table이 같은 값을 PK로 가지면서 Transaction을 구현하려면 어떻게 해야할까?

서버에서 static 변수를 가지면서 insert가 발생할 때마다 증가시키는 방법 등 여러 해결방안을 생각해보다가 간단한 방법을 발견했다.
바로 처음 Table에 insert 할 때 AUTO_INCREMENT로 증가한 PK값을 가져와서 다른 Table에 insert 할 때 사용하는 방법이다.
LAST_INSERT_ID는 가장 최근에 성공적으로 수행된 insert 구문의 첫 번째 AUTO_INCREMENT column을 가져온다.
이 값을 이용하면 Transaction과 여러 Table에서 PK FK를 안정적으로 유지할 수 있다.



[LAST_INSERT_ID() - insert](http://blog.leocat.kr/notes/2017/07/22/mysql-ibatis-fetch-pk-after-insert)

[MySQL AUTO_INCREMENT does not ROLLBACK](https://stackoverflow.com/questions/449346/mysql-auto-increment-does-not-rollback)
