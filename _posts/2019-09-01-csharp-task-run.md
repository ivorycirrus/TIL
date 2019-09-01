---
layout: post
title: "C# Task.Run 사용하기"
tags: [c#, thread]
comments: true
---

.Net Framework 4.5에서부터 스레드를 만드는 방법으로 ```Task.Run``` 메소드를 지원한다. 이는 그 이전까지 사용하던 ```Task.Factory.StartNew```를 보다 가볍게 사용 할 목적으로 만들었다고 한다. 새로운 메소드를 제공한다고 StartNew를 폐기한다는 것은 아니며 간편하게 사용하기 위한 추가적인 메소드를 제공한다고 한다.

결론부터 말하지면, 스레드 내부에서 관리가 필요한 자식 스레드를 생성하지 않고, 백그라운드에서 가볍게 처리해야 할 스레드가 필요하다면 가급적 ```Thread.Run```을 사용하는 것이 편리 할 것이다.

# Task.Run 과 Task.Factory.StartNew

```Task.Run(MathodA)```은 기본적으로 다음과 동일하다.

```csharp
Task.Factory.StartNew(
    MathodA
    , CancellationToken.None
    , TaskCreationOptions.DenyChildAttach
    , TaskScheduler.Default
);
```

이는 ```Task.Factory.StartNew(MathodA)```가 다음의 기본 실행옵션을 갖는 것과 차이가 있다.

```csharp
Task.Factory.StartNew(
    MathodA
    , CancellationToken.None
    , TaskCreationOptions.None
    , TaskScheduler.Current
);
```

# 어떻게 차이가 난다는 건가?

## 1. DenyChildAttach

아래 코드에서 ```parentTask.Wait()```는 내부의 childTask에 영향을 주지 않는다. 스레드 내부에서 생성한 하위태스크의 관리가 필요한 경우 상태관리를 하기가 불편할 것이다.

```csharp
// ref :  https://stackoverflow.com/a/55949173
var parentTask = Task.Run(() =>
{
    var childTask = new Task(() =>
    {
        Thread.Sleep(10000);
        Console.WriteLine("Child task finished.");
    }, TaskCreationOptions.AttachedToParent);
    childTask.Start();

    Console.WriteLine("Parent task finished.");
});

parentTask.Wait();
Console.WriteLine("Main thread finished.");
```

## 2. TaskScheduler.Default 와 TaskScheduler.Current

```Task.Factory.StartNew```의 기본 스케쥴러는 Current이다. UI스레드에서 생성한 Task는 작업 스케쥴러를 UI스레드와 같이 사용 할 수 있으며, 이는 백그라운드에서 업데이트가 잦은 동작을 할 경우 UI 업데이트를 멈추게 할 수도 있을 것이다. 

따라서 기본값으로 백그라운드 스레드를 생성한다면 Task.Run 이 좀 더 좋은 선택이 될 수 있다.

## 3. 그 외에도..

```Task.Factory.StartNew```는 async delegate를 이해하지 못한다거나, ```ThrowUnobservedTaskExceptions``` 설정에 따라 스레드 내부에서 발생한 예외를 통지받지 목하는 경우도 있다고 한다.

그래서 결국은 꼭 필요한 상황이 아니라면 ```Task.Factory.StartNew```를 사용하지 말자는 의견을 볼 수 있었다.


# 참고링크

* MSDN : https://docs.microsoft.com/ko-kr/dotnet/api/system.threading.tasks.task.run?view=netframework-4.8
* Stackoverflow : https://stackoverflow.com/questions/38423472/what-is-the-difference-between-task-run-and-task-factory-startnew
* Stephen Cleary's Blog : https://blog.stephencleary.com/2013/08/startnew-is-dangerous.html
* MS Israel Blog : http://blogs.microsoft.co.il/bnaya/2017/01/24/task-run-vs-task-factory-startnew-part-1/
* Dissecting the Code : https://sergeyteplyakov.github.io/Blog/async/2019/05/21/The-Dangers-of-Task.Factory.StartNew.html