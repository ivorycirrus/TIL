---
layout: post
title: "C#에서 Singleton 디자인패턴을 구현하는 방법"
tags: [c#, design pattern, singleton]
comments: true
---

싱글톤(Singleton)패턴은 애플리케이션을 실행한 이후 객체의 인스턴스가 단 한개만 생성하고자 하는 상황에서 자주 사용하는 디자인 패턴중 하나이다. C# 코드를 작성하면서 기존 습관적으로 사용하던 코드를 좀 더 우아하게 작성할 수 있는 방법이 있어 기록 해 둔다.

# 기존에 하던 방법

일반적인 싱클톤 패턴은 클래스 안에 다음 3개의 구성요소를 가진다.

* private 생성자
* private static 인스턴스 객체
* public static 의 객체반환 함수

따라서 코드로 작성하면 대강 이런 모양새가 나온다.

```csharp
namespace Sample
{
    public sealed class SomeClass
    {
        //private 생성자 
        private SomeClass() { }
        //private static 인스턴스 객체
        private static SomeClass _instance = null;
        //public static 의 객체반환 함수
        public static SomeClass Instance { get {
                if (_instance == null) _instance = new SomeClass();
                return _instance;
            }
        }

        // 그 외 멤버변수 및 함수
        // ....
    }
}
```

# 조금 더 나은 방법

.Net 4버전 이상에서 제공하는 ```System.Lazy<T>```를 이용하면 객체 생성 부분을 좀 더 우아하게 처리 할 수 있다.

```csharp
using System;

namespace Sample
{
    public sealed class SomeClass2
    {
        //private 생성자 
        private SomeClass2() { }
        //private static 인스턴스 객체
        private static readonly Lazy<SomeClass2> _instance = new Lazy<SomeClass2> (() => new SomeClass2());
        //public static 의 객체반환 함수
        public static SomeClass2 Instance { get { return _instance.Value; } }
    }
}
```

# 참고자료

아래 링크는 C#의 싱글톤 패턴의 다양한 구현체를 소개하고 실험한 블로그이다.

* https://csharpindepth.com/Articles/Singleton
