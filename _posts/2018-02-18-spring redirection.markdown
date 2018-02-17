---
layout: post
title:  "Spring Redirection"
date:   2018-02-18
comments: true
---

<p class="intro"><span class="dropcap">오</span>늘의 포스팅은 스프링에서 Redirection을 구현하며 겪었던 여러 이슈들에 관한 포스팅이다.</p>

메일 서비스를 개발하면서 메일 전송의 결과를 다른 페이지로 Redirection을 해야 했다. 성공적으로 메일을 받은 사람과 그렇지 못한 사람을 보여주기 위해 object도 담아서 보내야 했고, 스프링에서 RedirectView를 찾아 구현 할 수 있었다.

RedirectView를 사용할 때 RedirectAttributes를 사용하여 object를 담을 수 있었는데, addAttribute가 아닌 addFlashAttribute를 사용하면 object가 URL에 보이지 않게 보낼 수 있었다. (Redirection은 보통 POST/REDIRECTION/GET으로 진행되는데, URL에 object가 보이지 않는다는 사실이 신기했다. 찾아본 결과 addFlashAttribute를 사용하면 session을 사용하여 attribute를 저장하고 redirection 후 바로 사라진다고 한다.)

RedirectView와 addFlashAttribute를 사용하여 구현한 Controller는 다음과 같다.

<figure>
	<img src="{{ '/assets/img/2018-02-18-spring redirection_1.jpg' }}" alt="">
	<figcaption>Fig1. - 초기 redirect controller</figcaption>
</figure>

RedirectView의 reference를 보면, 처음 코드에서 사용한 빈 생성자 말고도 여러 파라미터를 가진 생성자를 볼 수 있다.

<figure>
	<img src="{{ '/assets/img/2018-02-18-spring redirection_2.jpg' }}" alt="">
	<figcaption>Fig2. - RedirectView code</figcaption>
</figure>

객체의 생성과 소멸 주기는 짧을수록 좋다는 기본을 인지하고, RedirectView의 reference를 참고하여 수정한 코드는 다음과 같다.

<figure>
	<img src="{{ '/assets/img/2018-02-18-spring redirection_3.jpg' }}" alt="">
	<figcaption>Fig3. - 수정 redirect controller</figcaption>
</figure>

물론 RedirectView를 사용하지 않고 redirection을 알려주는 prefix를 붙여 modelAndView로 redirection을 구현할 수 있다. return type이 RedirectView라면, 해당 controller는 항상 redirection을 수행하기 때문에 prefix를 사용하여 modelAndView를 return하는 controller가 더 나은 방법이다.

<figure>
	<img src="{{ '/assets/img/2018-02-18-spring redirection_5.jpg' }}" alt="">
	<figcaption>Fig3. - redirection using prefix</figcaption>
</figure>

또한 RedirectAttributes.addFlashAttribute 처럼 URL에 보이지 않게 하려면 @ModelAttribute("flashAttribute")를 사용하면 된다.


[A Guide To Spring Redirects](http://www.baeldung.com/spring-redirect-and-forward)

[Spring RedirectAttributes: addAttribute() vs addFlashAttribute()](https://stackoverflow.com/questions/14470111/spring-redirectattributes-addattribute-vs-addflashattribute)

[RedirectView Reference](https://docs.spring.io/spring/docs/5.0.3.RELEASE/javadoc-api/)
