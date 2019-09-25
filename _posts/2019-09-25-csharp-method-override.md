---
layout: post
title: "C#의 method override 와 virtual 키워드"
tags: [csharp, oop, inheritance]
comments: true
---

C#에서 자식클래스가 해당 메소드를 오버라이드 할 수 있도록 허용하기 위해서는 해당 메소드를 abstract 또는 virtual로 선언 해 줘야 한다. 물론 두 키워드를 붙이지 않아도 자식 클래서으세 재정의 해서 사용 할 수 있지만, 명시적으로 override 키워드를 사용하는데 제약이 있으며 그 동작 또한 다르다.

# abstract 와 virtual 의 차이 

메소드 선언시 사용하는 한정자 가운데 abstract 와 virtual는 모두 자식 클래스에서 해당 메소드를 오버라이드 하는 것을 허용한다는 의미로 사용 된다. 하지만 그 용법에 있어서 다소 차이가 있다.

둘의 가장 명확한 차이라면 abstract 메소드는 구현체(body)가 없지만 virtual은 구현체가 있다.

abstract 메소드는 구현체가 없으므로 당연히 abstract 클래스 안에서만 사용가능하다. 또한 해당 메소드를 호출하기 위해서는 자식클래스에서 반드시 해당 메소드를 구현해 줘야 한다. 반면에 virtual 메소드는 아무 동작도 하지 않거나 ```NotImplementedException```를 던지거나 할지라도 어쨋든 구현체가 있다. 따라서 어떤 클래스에서건 사용가능하고, 자식클래스에서 구현하지 않더라도 해당 메소드 및 클래스를 사용하는데 지장이 없다.

따라서 해당 클래스는 기본기능을 모두 갖춘 완성품인데 추가기능을 제공하기 위해서는 이런이런식으로 사용하세요 라고 가이드를 주고자 할 경우 ```virtual``` 키워드가 적합할 것이다. 더 나아가 자식클래스가 해당 메소드를 반드시 구현해서 사용하도록 제약을 주고 싶은 일종의 아키텍쳐나 스펙 정의 목적이라면 ```abstract```를 사용 하는 것을 고려 해 봐야 한다.

# override 시 virtual 유무에 따른 차이

결과적으로 다형성 측면에서 자식 클래스의 인스턴스를 부모클래스로 캐스팅해서 사용 할 경우 차이가 발생한다. ```virtual``` 키워드는 ```override```키워드와 함께 사용시 부모클래스로 자식클래스의 인스턴스를 캐스팅해서 사용 할 지라도 부모의 메서드가 아닌 실제 최정족으로 생성된 인스턴스의 메소드를 호출하도록 지시한다.

아래의 예제에서 Other 메소드는 명시적으로 오버라이드를 선언하지 않은 메소드로, 부모 클래스로 캐스팅 할 경우 부모 클래스 타입의 메소드가 호출된다. 반면 Another 메소드는 virtual 과  override 를 명시적으로 지정하여 부모 클래스로 캐스팅 시에도 최종적으로 오버라이드 된 메소드가 호출됨을 볼 수 있다.

```csharp
using System;

namespace Sample
{
    class Parent
    {
        public string Some() => "parent some";
        public virtual string Other() => "parent other";
        public virtual string Another() => "parent another";
    }

    class Child : Parent
    {
        public string Some() => "child some";

        // [Error] 'override' keyword is acceptable for abstract and virtual methods
        //public override string Some() => "child some";

        public string Other() => "child other";

        public override string Another() => "child another";
    }

    class Program
    {
        static void Main(string[] args)
        {
            Parent p = new Child();

            Console.WriteLine(p.Some());    // < Parent called
            Console.WriteLine(p.Other());   // < Parent called
            Console.WriteLine(p.Another()); // < Child called

            Console.ReadLine();
        }
    }
}
```

# 참고링크
* https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/keywords/virtual
* https://slaner.tistory.com/160
