---
title: "C# dotnet 使用 OpenXml 解析 PPT 里面的视频"
author: lindexi
date: 2020-12-3 20:27:50 +0800
CreateTime: 2020/3/13 19:09:15
categories: dotnet C#
---

本文告诉大家如何从 PPTX 文件里面解析出视频

<!--more-->


<!-- CreateTime:2020/3/13 19:09:15 -->



我期望看到本文的小伙伴是了解 OpenXML 的，如果想要解析 Office 的文档，我推荐使用使用 OpenXML SDK 这个开源的库，更多入门级博客请看 [C# dotnet 使用 OpenXml 解析 PPT 文件](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-PPT-%E6%96%87%E4%BB%B6.html)

我做了一个简单的 PPT 文件，这个文件里面只有一页，这一页上面有一个视频。做这个文件的作用是方便调试，本文将从这个文件里面拿到视频

```csharp
            using (var presentationDocument =
                DocumentFormat.OpenXml.Packaging.PresentationDocument.Open(@"小视频.pptx", false))
            {
                var presentationPart = presentationDocument.PresentationPart;
                var slidePart = presentationPart.SlideParts.FirstOrDefault();

                // 忽略代码
            }
```

上面代码是打开解析文件，我拿到第一页，而获取页面的元素需要了解一点是 PPT 将所有元素存放 ShapeTree 而视频是不存在元素的，在 PPT 里面用 Picture 存放视频

```xml
<p:pic>
    <p:nvpicpr>
        <p:cnvpr id="4" name="视频">
            <a:hlinkclick action="ppaction://media" r:id="">
            </a:hlinkclick>
        </p:cnvpr>
        <p:cnvpicpr>
            <a:piclocks nochangeaspect="1">
            </a:piclocks>
        </p:cnvpicpr>
        <p:nvpr>
            <a:videofile r:link="rId2">
            </a:videofile>
            <p:extlst>
                <p:ext uri="{DAA4B4D4-6D71-4841-9C94-3DE7FCFB9230}">
                    <p14:media r:embed="rId1" xmlns:p14="http://schemas.microsoft.com/office/powerpoint/2010/main">
                    </p14:media>
                </p:ext>
            </p:extlst>
        </p:nvpr>
    </p:nvpicpr>
</p:pic>
```

如上面代码，这就是视频元素其实也就是 Picture 元素，可以在 picture.NonVisualPictureProperties.ApplicationNonVisualDrawingProperties 找到 VideoFromFile 解析视频

```csharp
                var picture = slidePart.Slide.CommonSlideData.ShapeTree.OfType<Picture>().FirstOrDefault();
                var videoFromFile = picture.NonVisualPictureProperties
                    .ApplicationNonVisualDrawingProperties
                    .GetFirstChild<VideoFromFile>();
```

而视频用的是 `r:link` 拿到对应的资源，在 PPT 里面，用 GetPartById 获取 ChildrenRelationshipParts 的资源，用 GetReferenceRelationship 拿到 ReferenceRelationshipList 的资源

在 PPT 里面的视频放在 ReferenceRelationshipList 使用下面代码拿到

```csharp
           var openXmlPart = (DataPartReferenceRelationship) slidePart.GetReferenceRelationship(videoFromFile.Link.Value);
```

通过 GetStream 方法可以拿到压缩包里面的文件，此时不需要解压缩，新建文件写入就可以

```csharp
                var stream = openXmlPart.DataPart.GetStream();
                var file = System.IO.Path.Combine(System.IO.Path.GetTempPath(), "林德熙是逗比.mp4");
                File.WriteAllBytes(file, ReadAllBytes(stream));
```

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8551ad78455f7e56e2f1cafa66d6ae62d7a94995/GairhajelkewaiHeyerjeaginu) 欢迎小伙伴访问

