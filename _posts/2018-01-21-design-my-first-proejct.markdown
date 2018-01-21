---
layout: post
title:  "첫 스프링 프로젝트.. 어떻게 설계할까?"
date:   2018-01-21
description: "설계에 대한 고민은 항상 하기 마련이예요. 좋은 설계를 위해 꾸준히 고민하고 리팩터링 한다는게 좋은 것 같아요."
---

<p class="intro"><span class="dropcap">설</span>계에 대한 고민은 항상 하기 마련이예요. 좋은 설계를 위해 꾸준히 고민하고 리팩터링 한다는게 좋은 것 같아요.</p>


취업 준비할 때 만들었던 포트폴리오를 내리고 깃 블로그를 시작하며, 한주의 고민을 주말에 포스팅하자는 목표를 세웠다. </br>
이번주의 고민은 첫 스프링 프로젝트의 구조 설계였다.

한주 동안 팀원들과 함께 DB를 설계했고, 프로젝트 내부 구조 설계를 어떻게 할지에 대해 많은 고민을 했다.

구글링을 통해 여러 좋은 자료를 찾아보며 일반적인 스프링 계층을 살펴보았다.


    Presentation Layer
	  View : jsp, html, js etc..
	  비지니스 로직이나 퍼시스턴트 계층에서 처리하는 일을 직접 수행하거나
	  각 계층의 컴포넌트와 직접적인 통신이 있어서는 안됨

    Control Layer
	  Presentation Layer와 비지니스 로직을 분리 (다리 역할)
	  어떤 요청이 들어왔을 때 어떤 로직이 처리되야 하는지 결정
	  UI 검증, 요청 및 응답 전달, 로직에서 던져진 예외 처리, 도메인 모델과 뷰 연결만 수행

    Busniess Logic Layer
	  핵심 업무 로직의 구현과 관련 데이터의 적합성 검증 등
	  DB 트랜젝션 처리, 다른 계층가 통신을 위한 인터페이스 제공

    Persistence Layer
	  데이터의 처리를 담당 (CRUD연산), Query문 관리

    Domain Model Layer
    DAO...

위와 같은 계층 구조를 보며 간단히 패키지를 나누는 작업부터 시작했다.</br>
(물론 주말에 혼자 하는거라 팀 회의를 통해 바뀔 것 같지만..)

먼저, Controller, Service, Repository, Domain 4개의 Package로 나누고 각 Package에 들어갈 Class들을 고민했다.

Domain에는 DAO 처럼 DB Table 객체를 넣고, Repository에는 Mapper를, Controller에는 View와 Business logic의 연결 고리를, Service에는 Business logic을 넣고자했다.

여러 블로그를 읽다보니 Service에서 Transaction도 처리한다고 하니 구현할 때 참고해야겠다.

4개의 Package로 나누어 간단하게 REST 통신 및 DB 접근을 할 수 있는 틀만 만들어 놓고, 팀원들과 회의를 통해 자세한 구조를 결정할 예정이다.



참조 </br>
[웹 애플리케이션의 계층]:    http://egloos.zum.com/mt1716/v/9291203
[첫 Java 프로젝트의 생생한 후기]: http://woowabros.github.io/experience/2016/08/02/first_java_project.html