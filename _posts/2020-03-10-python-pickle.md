---
layout: post
title: "Python에서 Dictionary 데이터를 파일로 저장하기 (pickle)"
tags: [python, pickle]
comments: true
---

Python 개발을 하다보면 key-value 유형의 데이터는 흔히 Dictionary 자료형을 사용한다. 배열을 경우 numpy의 save, load 함수를 이용해 파일로 저장하고 불러오는 방식을 많이 사용했엇다. Dictionary의 경우에는 json 이나 xml형식으로 저장하고 불러오는 방법을 고민했엇으나 자료의 형변환도 번거롭고, 데이터의 크기가 큰 경우 그 불편함은 더할 수 밖에 없다.

뭔가 간편하게 Dictionary 자료형을 파일로 저장하고 불러올 수 있는 방법이 없을까 고민하던 차에 pickle 라는 라이브러리를 발견했다.

# pickle 은...
다음은 python 설명서의 내용중 pickle에대한 설명의 일부를 발췌한 것이다.

* pickle 모듈은 파이썬 객체 구조의 직렬화와 역 직렬화를 위한 바이너리 프로토콜을 구현한다.
* 파이썬이 marshal 이라 불리는 좀 더 원시적인 직렬화 모듈을 가지고 있지만, 일반적으로 pickle을 이용해서 파이썬 객체를 직렬화를 수행해야 한다. marshal 은 주로 파이썬의 .pyc 파일을 지원하기 위해 존재하는 것.
* JSON은 텍스트 직렬화 형식(유니코드 텍스트를 출력하지만, 대개는 utf-8 으로 인코딩됩니다)인 반면, pickle은 바이너리 직렬화 형식입니다.
* pickle은 JSON으로 표현할 수 없는 python의 사용자정의 데이터형을 표현할 수 있다.

# 데이터 저장과 불러오기
```python
import pickle

user = {'name':'Andrew K. Johnson', 'score': 199, 'location':[38.189323, 127.3495672]}

# save data
with open('user.pickle','wb') as fw:
    pickle.dump(user, fw)

# load data
with open('user.pickle', 'rb') as fr:
    user_loaded = pickle.load(fr)

# show data
print(user_loaded)
# {'name':'Andrew K. Johnson', 'score': 199, 'location':[38.189323, 127.3495672]}
```

# 참고
* https://docs.python.org/ko/3/library/pickle.html