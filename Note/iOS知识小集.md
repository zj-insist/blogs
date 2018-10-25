# iOS 知识小集

## [Tip 32](https://mp.weixin.qq.com/s/ny0wnRZZr1Xr7UCeJ4xZLQ)

1、利用 GCD 信号量将异步方法改造为同步方法

> 发起连接请求之前，创建一个初始值为 0 的信号量，在方法返回之前请求该信号量（P 操作），同时，在连接请求的结果回调中释放该信号量（V 操作）。这样，发起连接请求后，由于信号量初始值为 0，会阻塞在请求信号量的地方，连接请求回调回来之后，释放信号量，刚刚的信号量请求被满足，方法得以返回。

**文章**  
[iOS 界面渲染流程分析](https://mp.weixin.qq.com/s/-0k0CXn046_4549OcbH4JQ)  
[iOS Memory Deep Dive](https://mp.weixin.qq.com/s/WQ7rrTJm-cn3Cb6e_zZ4cA)

## [Tip 31](https://mp.weixin.qq.com/s/RwtVeKg5_s5QaAK9wBa_gA)

1、iPhone 屏幕分辨率终极指南
![屏幕](https://qs-image-bucket.oss-cn-hangzhou.aliyuncs.com/20181019153991779851403.jpg)
[链接](https://www.paintcodeapp.com/news/ultimate-guide-to-iphone-resolutions)

2、使用 IBDesignable + IBInspectable 实时更新 UI

**文章**  
[iPhone 分辨率终极指南以及检测 iPhone X 设备的几种方式](https://mp.weixin.qq.com/s/Ik2zBox3_w0jwfVuQUJAUw)

## [Tip 30](https://mp.weixin.qq.com/s/0fd73SUL1D25Ds_qKU84sA)

**文章**  
[单一职责原则在 iOS 中的应用](https://mp.weixin.qq.com/s/owzQOIXbvwYBTBvfrMybnw)

## [Tip 29](https://mp.weixin.qq.com/s/s5crnjMG-fCAfWVZ3q83yg)

1、几个第三方框架关于线程锁的封装小技巧

- YYCache

![20181025154044846134728.jpg](https://qs-image-bucket.oss-cn-hangzhou.aliyuncs.com/20181025154044846134728.jpg)

- SDWebImage

![20181025154044848831538.jpg](https://qs-image-bucket.oss-cn-hangzhou.aliyuncs.com/20181025154044848831538.jpg)

- YYWebImage

![20181025154044850630180.jpg](https://qs-image-bucket.oss-cn-hangzhou.aliyuncs.com/20181025154044850630180.jpg)

## [Tip 27](https://mp.weixin.qq.com/s/s5crnjMG-fCAfWVZ3q83yg)

1、利用 Storyboard Reference 给 storyboard 瘦身

2、

**文章**  
[黑幕背后的 Code Signing](https://mp.weixin.qq.com/s/Ml9UteoAB0KrWLU9dPht6g)
