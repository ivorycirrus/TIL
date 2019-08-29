---
layout: post
title: "C# 로그 라이브러리 NLog 시작하기"
tags: [c#, lolgger, NLog]
comments: true
---

C# 프로그램을 개발하면서 로그를 콘솔에 단순히 표시하는 것 이외에 파일이나 또는 원격지의 저장소로 전송할 필요가 생겼다. 직접 로그저장을 위한 모듈을 만들기보다는 간단하게 사용 할 수 있는 로그 라이므러리를 찾아보니 Log4Net와 NLog라는 훌륭한 라이브러리가 있었다. 둘 모두 훌륭한 오픈소스 로그 라이브러리임에 분명하지만, NLog쪽이 처음 접하는데 있어 사용법이 간단하고 원하는 기능이 모두 들어있어서 사용을 결정했다.


# NLog 설치

개발하고 있는 프로젝트에서 NuGet 패키지를 검색해서 설치하면 된다.

로그를 출력할 모든 프로젝트에 각각 NuGet패키지 관리자에서 검색해서 설치해줘야 하며, 환경설정을 위해 가급적 솔루션의 시작프로젝트에는 반드시 설치해주도록 한다.

# 환경설정

## 환경설정 방법

NLog의 환경설정은 다음 3가지가 있다.

* 실행 어셈블리와 같은 경로에 NLog.config 라는 이름의 파일로 XML형식으로 설정하는 방식
* 실행 바이너리의 config 파일 (App.config) 안에 ```<nlog>```태그를 추가하는 방식
* 프로그램 안에서 C#코드로 설정하는 방식

설정이나 관리가 편리한 방식은 두번째방식이다. 코드로 작성하는 부분은 가독성이 떨어지고, NLog.config파일에 설정하는 것은 프로그램 빌드시마다 또는 설치시마다 NLog.config를 일일히 관리해 줘야 한다. 프로그램의 기본 설정파일안에 설정정보를 추가하는 두번째 방식이 개발 및 배포시 관리의 편의성 및 가독성 측면에서 가장 간단하고 편리했다.

## 환경설정 샘플 (App.config 안에 하는 방법)
App.config 또는 web.config 안에 설정을 두는 경우 두 가지를 고려해야 한다.
* ```<configSectrions>```안의 설정정보는 ```<configuration>```대그의 가장 윗쪽에 나와야 된다. 기존에 뭔가 설정항목이 있는 경우 ```<section>```태그의 내용만 추가한다.
* ```<nlog>```태그 안쪽이 실제 NLog의 설정을 기록하는 영역이다.

```xml
<configuration>
  <configSections>
    <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog"/>
  </configSections>
  
  <!-- 설정항목들 뭔가 쭉 있을거고... -->
  
  <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <targets>
        <target name="logfile" xsi:type="File" fileName="file.txt" />
        <target name="logconsole" xsi:type="Console" />
    </targets>

    <rules>
        <logger name="*" minlevel="Info" writeTo="logconsole" />
        <logger name="*" minlevel="Debug" writeTo="logfile" />
    </rules>
</nlog>>
</configuration>
```

## 환경설정 항목

환경설정은 크게 Target, Rule, latyout을 지정한다.

* Target : 어디에 로그를 저장할 건지 로그를 저장하는 대상이다. [NLog 문서에는 현재 84가지의 Target 지원목록](https://nlog-project.org/config/?tab=targets)이 있다. 위 설정에서는 파일과 콘솔에 출력하도록 설정되어 있다.
* Rule : 로그의 수준별 출력정책을 설정하는 곳이다. 로그는 낮은 레벨부터 Trace - Debug - Info - Warn - Error - Fatal 순으로 레벨이 부여되어 있다. writeTo는 해당 범위의 로그를 출력할 Target를 지정하는 부분이며 복수의 Target를 지정하려면 쉼표 ( , )로 구분해서 지정하면 된다.
* Layout : 로그의 출력형식이다. 위의 기본설정예시에는 없지만, target 태그에 layout 속성으로 로그의 출력형식을 지정할 수 있다.[ ]사용가능한 옵션은 여기 NLog 문서를 참고](https://nlog-project.org/config/?tab=layout-renderers) 하자.

## File Target 의 기간별/최대용량별 로그 아카이빙

File Tatget는 기간별로 아카이브를 하거나, 최대 용량을 지정해서 아카이빙을 할 수 있다. 환경설정의 target 태그에 아카이빙 관련 속성을 추가해서 선언할 수 있다.

아래는 일자별 최대용량제한을 가진 파일로그 아카이빙을 설정하는 예시이다.

```xml
<targets>
    <target name="file" xsi:type="File"
        layout="${longdate} ${logger} ${message}${exception:format=ToString}" 
        fileName="${basedir}/logs/logfile.txt" 
        maxArchiveFiles="4"
        archiveAboveSize="10240"
        archiveEvery="Day" />
</targets>
```


# 프로그램 구현

환경설정이 끝났으면, 프로그램 안에서 로그를 출력하는 방법은 매우 간단하다.


로그를 기록할 클래스에 멤버변수로 로거 인스턴스를 선언하고, 원하는 위치에서 로그를 출력하면 된다.


```csharp
public class Program
{
    // 로거 인스턴서 선언
    private static readonly NLog.Logger Logger = NLog.LogManager.GetCurrentClassLogger();

    public void SomeFunctionWhatYouWrote()
    {
        try
        {
            // 로그레벨별 로그 출력
            Logger.Debug("Debug log");
            Logger.Info("Info log");
            Logger.Warn("Warn Log");

            // 문자열 포맷
            Logger.Warn("Warn log with formatting : {0}", "something formatted text");
        }
        catch (Exception ex)
        {
            // exception객체 출력 
            Logger.Error(ex, "Goodbye cruel world");
        }
    }
} 
```

# 참고 링크

* 홈페이지 : https://nlog-project.org/
* GitHub : https://github.com/NLog/NLog