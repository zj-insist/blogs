## MacOS Tips  

#### 1.坐标系  

MacOS 开发的坐标系与 iOS 不同，原点在左下角，而 iOS 则是在左上角。此外，覆盖 NSView 的如下方法：  

```objc
- (BOOL)isFlipped;
```  

可以使坐标系原点变成左上角。  

![20181127154329817729132.jpg](https://qs-image-bucket.oss-cn-hangzhou.aliyuncs.com/20181127154329817729132.jpg)  

#### 2.NSWindow的使用问题  

使用代码创建的 NSWindow ，第一次弹出没有问题，但是第二次弹出会出现 crash 问题，原因是关闭 NSWindow 后，系统会默认销毁这个对象，第二次访问会出现野指针问题。  

解决方法是使用 NSWindowController 管理 NSWindow，这样 NSWindow 就不会被系统自动释放。