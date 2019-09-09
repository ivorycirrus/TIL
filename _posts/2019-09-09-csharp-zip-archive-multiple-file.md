---
layout: post
title: "C# 에서 Zip 파일 압축하기"
tags: [c#, zip]
comments: true
---

어러개의 텍스트파일을 전송하고자 할 경우 파일을 하나로 압축해서 전송하면 전송 용량측면과 속도 측면에서 이점이 있을 수 있다. C#에서는 ```System.IP.Compression``` 네임스페이스 안에 Zip, GZip, Deflate 알고리즘을 지원하는 압축 API를 제공하고 있다. 그 중에서 Zip 형식의 파일을 생성해서 여러개의 파일을 하나의 zip파일로 압축하는 방법을 알아본다.

# 파일 압축

* 먼저 ```MemoryStream```에 모든 파일의 데이터를 압축한다.
* 그 다음으로 압축한 데이터를 파일에 기록한다.
* 로직을 단순화하고, 구현의 용이성 때문에 압축파일을 통으로 들고 있지만, 시스템 메모리가 부족할 경우 중간중간 디스크로 쓰는 방식으로 로직을 변경하는 편이 좋을 것이다.


```csharp
public ArchiveFiles(string zipFileName, string[] archiveFileList) {
    using (var ms = new MemoryStream())
    {
        // archive log files
        using (var archive = new ZipArchive(ms, ZipArchiveMode.Create, true))
        {
            foreach( var f in archiveFileList )
            {
                var zipEntity = archive.CreateEntry(f);

                using (var entryStream = zipEntity.Open())
                using (var readStream = File.OpenRead(f))
                {
                    readStream.CopyTo(entryStream);
                }
            }
        }

        // write archived data
        using (var fs = new FileStream(zipFileName, FileMode.Create))
        {
            ms.Seek(0, SeekOrigin.Begin);
            ms.CopyTo(fs);
        }
    }
}
```

# 참고 링크

* https://stackoverflow.com/questions/17232414/creating-a-zip-archive-in-memory-using-system-io-compression

