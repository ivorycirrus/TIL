---
layout: post
title: "C#의 Assembly 버전 충돌 문제"
tags: [csharp, assembly, ziparchive]
comments: true
---

C#에서 ```System.IO.Compression```에 있는 ```ZipArchive```클래스를 사용하고자 할때, 해당 클래스를 찾지 못해 빌드 오류가 발생하는 경우가 있다. 분명히 참조목록에 ```System.IO.Compression 4.1.2.0```을 추가했음에도 해당 클래스를 참조하지 못해서 오류가 발생했다. 이는 프로젝트의 기본 패키지인 ```System.IO```에 ```System.IO.Compression```이 존재하지만, ZipArchive 관련 클래스가 없어서 발생하는 오류이다.

# 문제의 코드

문제가 발생한 상황은 다음과 같다.

```System.IO``` 와 ```System.IO.Compression```를 둘 다 사용하는 모듈이다.

이 때, ```ZipArchive```를 사용하기 위해서는 참조관리자의 어셈블리 참조메뉴에서 ```System.IO.Compression 4.1.2.0```를 추가해야 한다. 하지만 참조를 추가하고, 프로젝트의 참조 목록에 해당 네임스페이스가 보이더라도 빌드 오류가 발생하는 경우가 있다.

```csharp
using System.IO;
using System.IO.Compression;

namespace ClassLibrary1
{
    public class Class1
    {
        public static byte[] ZipData(string rootDir)
        {
            using (var ms = new MemoryStream())
            {
                using (var archive = new ZipArchive(ms, ZipArchiveMode.Create, true))
                {
                    foreach (var file in new DirectoryInfo(rootDir).GetFiles())
                    {
                        var zipEntity = archive.CreateEntry(file.Name);
                        using (var entryStream = zipEntity.Open())
                        using (var data = File.OpenRead(Path.Combine(rootDir, file.Name)))
                        {
                            data.CopyTo(entryStream);
                        }
                    }
                }

                ms.Seek(0, SeekOrigin.Begin);
                return ms.ToArray();
            }
        }
    }
}
```


# 문제의 해결

해당 코드 빌드시 ```System.IO.Compression```의 경우 모두 4.1.2.0 버전의 라이브러리를 참조하도록 지정하면 된다.

이는 ```app.config```파일에서 지정 할 수 있으며, 프로젝트 마다 별도로 관리하고 있으니 빌드 오류가 발생하는 프로젝트의 설정파일에 적용 하면 된다.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="System.IO.Compression"/>
        <bindingRedirect oldVersion="0.0.0.0-4.1.2.0" newVersion="4.1.2.0" />
      </dependentAssembly>
    </assemblyBinding>
  </runtime>
</configuration>
```

혹시 Asp.Net과 같은 웹 프로젝트의 경우 ```project.json```에 유사한 설정을 추가해서 해결 할 수 있다고 한다.

# 참고링크

* https://stackoverflow.com/questions/54145504/cant-find-system-io-compression-assembly
* https://stackoverflow.com/questions/39642990/system-io-compression-dll-missing-and-possible-version-conflict

