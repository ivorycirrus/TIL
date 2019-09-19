---
layout: post
title: "Github Pages의 Liquid tag 관련 빌드 오류"
tags: [jekyll, liquid tag]
comments: true
---

지난 zsh 환경설정 포스트를 작성하면서 Unknown tag error 라는 jekyll 빌드 오류가 발생했었다. Github Pages 는 [jekyll](https://jekyllrb-ko.github.io/) 기반의 블로그 엔진을 지원하는데, 이 때 인지할 수 없는 Liquid tag가 있다며 오류가 발생했었다. 

오류 메세지는 다음과 같다.

> Subject: Page build failed
> The page build failed with the following error:
> The tag `fake_tag` in `index.html` is not a recognized Liquid tag.

# 그래서 Liquid Tag 라는게 뭔가?

[jykell](https://jekyllrb-ko.github.io/)은 Ruby 언어 기반으로 만들어졌는데, 여기에서 템플릿을 표현하기위한 언어로 사용하는 것이 [Liquid Tag](https://help.shopify.com/en/themes/liquid/tags) 라고 한다. Liquid Tag는 {% raw %}```{%``` 와 ```%}```로{% endraw %}이루어진 태그로 그 안에 프ㅜ로그래밍 언어로 템플릿을 생성 할 수 있다.

공교롭게도, 이전 포스트의 git log의 템플릿을 지정하는 부분에 {% raw %}```{%``` 와 ```%}```로{% endraw %} 가 포함되어 있어서 발생한 오류였다.

소스코드 템플릿으로 감싸여진 곳이라도, 화면에 렌더링 되기 전에 템플릿 엔진이 먼저 파싱을 하면서, 해당 부분이 샘플코드에 해당하는 텍스트로 처리해야될 부분임을 인지하지 못한 것이다.

# 해결 방법은?

해당 부분은 Liquid Tag가 아닌 일반 텍스트 라고 지정해 주면 된다.

즉 샘플 코드가 있는 부분 아래/위로 raw 와 endraw 라는 Liquid Tag로 감싸준다. 그러면 jekyll 빌드시 해당 영역은 템플릿 엔진도 raw contents 라고 인지해서 태그 파싱을 하지 않으므로 정상적으로 빌드 및 표시다 괴는 것을 확인 할 수 있었다.

# 참고 링크

* https://help.github.com/en/articles/page-build-failed-unknown-tag-error
* https://help.shopify.com/en/themes/liquid/
* https://help.shopify.com/en/themes/liquid/tags/theme-tags#raw
