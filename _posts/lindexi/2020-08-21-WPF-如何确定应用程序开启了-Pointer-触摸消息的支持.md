---
title: "WPF 如何确定应用程序开启了 Pointer 触摸消息的支持"
author: lindexi
date: 2020-12-3 16:22:26 +0800
CreateTime: 2020/8/21 8:43:29
categories: WPF
---

因为 WPF 在开启 Pointer 和没有开启的基础表现几乎相同，因此从业务层很难了解到当前是否开启了 Pointer 消息。本文从开发者的角度，通过 Windows 消息判断当前是否开启 Pointer 支持

<!--more-->


<!-- CreateTime:2020/8/21 8:43:29 -->



在 [win10 支持默认把触摸提升 Pointer 消息](https://blog.lindexi.com/post/win10-%E6%94%AF%E6%8C%81%E9%BB%98%E8%AE%A4%E6%8A%8A%E8%A7%A6%E6%91%B8%E6%8F%90%E5%8D%87-Pointer-%E6%B6%88%E6%81%AF.html) 告诉大家如何在 Win10 下让 WPF 在 .NET 4.7 和以上框架支持 Pointer 消息

那么如何确定这个 WPF 程序我写对了，开启了 Pointer 消息？可以通过监听 Window 消息，如果能收到 Pointer 的消息，那么算开启成功

不需要在用户端判断，用户端只需要判断 运行的系统是 `Windows 10 Creators Update` 1703 10.0.15063 就可以。因此本文更多是给开发端，开发的时候通过此方法可以确定是否开启了 Pointer 消息

在 [WPF 添加窗口消息钩子方法](https://blog.lindexi.com/post/WPF-%E6%B7%BB%E5%8A%A0%E7%AA%97%E5%8F%A3%E6%B6%88%E6%81%AF%E9%92%A9%E5%AD%90%E6%96%B9%E6%B3%95.html) 这篇博客告诉大家如何拿到窗口的消息

在这个基础上，尝试在拿到消息判断是否 Pointer 消息，如果能收到 Pointer 消息，那么证明代码没写错

```csharp
       public MainWindow()
        {
            InitializeComponent();

            SourceInitialized += OnSourceInitialized;
        }

        private void OnSourceInitialized(object sender, EventArgs e)
        {
            var windowInteropHelper = new WindowInteropHelper(this);
            var hwnd = windowInteropHelper.Handle;

            HwndSource source = HwndSource.FromHwnd(hwnd);
            source.AddHook(Hook);
        }

        private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
        {
            const int WM_POINTERDOWN = 0x0246;

            if (msg == WM_POINTERDOWN)
            {
                // 开启了 Pointer 消息
                Debugger.Break();
            }


            return IntPtr.Zero;
        }
```

如果能进入 `msg == WM_POINTERDOWN` 那么就是收到 Pointer 消息了

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/a91924abc9c0254edfb8bb567a66c6e796d3a7dd/KemjawyecawDurbahelal) 欢迎小伙伴访问

