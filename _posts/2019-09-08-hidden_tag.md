---
layout: post
title: "Flask WTForm의 Hidden_tag()"
tags: [Web, python, security]
comments: true
---
## Intro 
웹에서 form은 사용자와 웹 어플리케이션 간의 데이터 전송의 기능을 담당한다. 일반적으로 사용자로부터 입력을 받아서 서버에서는 해당 입력을 받아서 적당히 처리하여, 다시 사용자에게 결과를 반환한다.  
하지만 모든 사용자가 반드시 프로그래머가 의도한 대로 입력을 넣는다는 보장은 없다. 경우에 따라서는 특수문자가 입력이 될 수 있고, 이를 적절하게 필터링하지 않는다면 다양한 웹 취약점에 노출 될 수 있다  
(ex. SQLi, XSS, CSRF etc ...)


## CSRF
Cross Site Request Forgery의 약어로 사이트 간 요청 위조를 의미한다. 단어가 XSS(Cross Site Sript)와 비슷해보이지만 둘은 다른 웹 취약점이다.  
CSRF는 사용자가 자신의 의지와는 무관하게 서버에 공격자가 의도한 행위(변경, 삭제 등)을 요청하는 공격이다.      

![csrf_attack](/images/post/2019-09-08-csrf.png) 

CSRF의 공격과정은 다음과 같이 진행된다.  
1. 공격자는 공격할 웹 서버(Target)에 악성 스크립트가 포함된 글을 게시한다.
2. 사용자는 해당 스크립트가 포함된 글을 읽기 위해, 서버로 게시글 열람 요청을 보낸다.  
3. 게시물을 읽은 사용자의 권한으로 공격자가 원하는 행위의 공격이 시도된다.  
4. 공격자가 원하는 결과를 서버로부터 받는다.  

CSRF가 발생되는 이유는 웹사이트 별로 사용자별로 할당되는 토큰이 예측 가능할 때 발생한다. 사이트 별로 요청하는 토큰이 다르지만, 이를 예측할 수 있다면, 공격자는 예측된 토큰을 가지고 요청 메세지를 변조하는 것이 가능하기 때문이다.

## Flask의 WTForm, hidden_tag()
CSRF를 막는 방법은 여러가지가 있다.그 중 하나는 토큰을 사용하는 방법이다. 사용자의 세션에 임의의 난수값(=token)을 저장하고, 요청을 할 때마다, 해당 token을 같이 보낸다. 요청을 받은 서버는 해당 토큰을 기반으로 요청과 데이터를 검증한다. Flask에서는 WTForm을 이용하여, 폼의 유효성을 검증하고 CSRF를 막기위한 다양한 기능을 지원한다. 그 중 flask-wtf라는 모듈은 암호화 키를 이용하여 토큰을 생성하고, 해당 토큰을 이용하여 폼 데이터와 요청을 검증하여 CSRF 공격을 방어한다.  
하지만 이렇게 생성된 token을 유출 할 수 있다면, 유출된 token 값을 이용하여, CSRF 공격이 가능하다. 따라서 token을 보낼 때에는 token 값이 유출되지 않도록 해야한다.   
flask-wtf의 hidden_tag()는 생성된 token을 hidden field를 이용하여 사용자가 submit할 때 같이 보낼 수 있도록 전송한다.  
hidden_tag()를 이용할 경우, 웹 페이지 상에 token이 노출되지 않은 채로 token을 보낼 수 있으므로, CSRF 방어에 도움이 된다.

ref.  
* flask-wtf doc : https://flask-wtf.readthedocs.io/en/stable/
* wtform doc : https://wtforms.readthedocs.io/en/stable/
* CSRF attack : https://hack-cracker.tistory.com/253
