# 私有Pods创建以工程模块化实践  

最近，项目中需要做一个独立性的基础模块，目的是要配置成第三方库类似的使用模式，没有业务依赖，但是引用便捷。当前，iOS开发中通常都是使用CocoaPods维护项目中的第三方库，之前也曾接触过使用Pods管理项目子模块的项目，但当时对CocoaPods都是似懂非懂，没有深入的研究。现在刚好借助这个机会了解Pods的整个运行机制。  

## 私有Pods创建以及维护  

私有库的创建CocoaPods的官网有相当明确的文档，网上也有相当多的博客整理，总结之后大致可以分为以下几个步骤：  

1.创建与添加私有Specs Repo  
2.创建私有Pods工程文件  
3.对私有Pods的Podspec文件进行配置  
4.推送私有Pods的Podspec文件至私有Repo  
5.使用与维护  

#### 1.创建与添加私有Specs Repo  

关于什么是Specs Repo，可以简单的理解为配置文件仓库。Pods的Search、Insatll、update等操作都依赖这个Specs Repo。官方提供了 `pod repo list` 用于查看当前Cocoapods下的所有Specs Repo，输入命令可以看到以下内容：  

```zsh   
master
- Type: git (master)
- URL:  https://github.com/CocoaPods/Specs.git
- Path: /Users/apple/.cocoapods/repos/master

QSPrivateRepo
- Type: git (master)
- URL:  https://github.com/zj-insist/PrivatePodsRepo.git
- Path: /Users/apple/.cocoapods/repos/QSPrivateRepo

XGTCPrivatePodsRepo
- Type: git (master)
- URL:  ****
- Path: /Users/apple/.cocoapods/repos/XGTCPrivatePodsRepo

3 repos
```  

私有库进行部分打码…我之前添加过两个repo，通常状态下只有一个master的repo，而这个master正是pods官方支持的所有第三方库的repo。三个repo都有一个相同的path前缀，我们切换到.cocoapods文件夹下使用tree命令查看这个目录的文件结构，关于tree命令的相关内容，可以点击[这里](http://yijiebuyi.com/blog/c0defa3a47d16e675d58195adc35514b.html)查看。

```zsh  
➜  .cocoapods tree -L 5
.
└── repos
    ├── QSPrivateRepo
    │   ├── QSTestPodsLibrary
    │   │   ├── 0.1.0
    │   │   │   └── QSTestPodsLibrary.podspec
    │   │   └── 0.1.1
    │   │       └── QSTestPodsLibrary.podspec
    │   └── README.md
    ├── XGTCPrivatePodsRepo
    │   ├── README.md
    │   └── XGJSBridgeLibrary
    │       ├── 0.1.1
    │       │   └── XGJSBridgeLibrary.podspec
    │       ├── 0.1.2
    │       │   └── XGJSBridgeLibrary.podspec
    │       ├── 0.1.3
    │       │   └── XGJSBridgeLibrary.podspec
    │       ├── 0.1.4
    │       │   └── XGJSBridgeLibrary.podspec
    │       └── 0.1.5
    │           └── XGJSBridgeLibrary.podspec
    └── master
        ├── CocoaPods-version.yml
        ├── README.md
        └── Specs
```  

截去master中过长的内容，可以看到其中主要包含每个repo中包含的库和其对应版本的配置文件。由于在安装pods的时候我们便已经将master clone到本地，因此我们可以对pods中支持的库进行常规的pods操作。那么要建立私有库的步骤就很明显了：创建自己的repo->clone这个repo->添加私有库的配置文件到这个repo。这样，就能像操作常规的第三方库一样操作我们自己创建的私有库了。  

创建自己的repo首先需要一个git仓库，如何创建根据自身需求，这里作为例子使用GitHub；然后添加这个仓库为repo，使用如下命令：  

```zsh  
  # pod repo add [Private Repo Name] [GitHub HTTPS clone URL]
  ➜ pod repo add QSPrivateRepo https://github.com/zj-insist/PrivatePodsRepo.git
```  

repo的名称并不需要跟git仓库中的一样，可以使用自定义名称。repo仓库并不需要所有项目的参与者都在本地添加，其主要作用是为了更新私有库的版本号以供使用者更新。如果只有一个人维护其中私有库版本，则只需维护者添加这个repo，参与这个repo的push工作，使用者只需在podfile中进行相关配置便可正常使用。  


#### 2.创建私有Pods工程文件  

这一步，最简单的方法便是创建一个文件夹，然后把工程文件放入这个文件夹，最后，创建一个远程git仓库，管理这个工程。至此，便完成了第二步。  

此外，pods中也集成了创建pods项目的命令，并帮助我们完成了相关配置文件以及example工程的创建。切换目录到要创建工程的文件夹，然后运行如下命令：  

```zsh   
 # pod lib create [Private Pods Name]
 ➜ pod lib create QSPrivateLibrary
```   

之后在终端中，会被询问如下问题：  

```zsh
What language do you want to use?? [ Swift / ObjC ]
 > ObjC

Would you like to include a demo application with your library? [ Yes / No ]
 > Yes

Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > None

Would you like to do view based testing? [ Yes / No ]
 > No

What is your class prefix?
 > QS

```  

创建的项目，目录结构如下：  

```zsh   
.
├── Example
│   ├── Podfile
│   ├── Podfile.lock
│   ├── Pods
│   ├── QSPrivatePodsLibrary
│   ├── QSPrivatePodsLibrary.xcodeproj
│   ├── QSPrivatePodsLibrary.xcworkspace
│   └── Tests
├── LICENSE
├── QSPrivatePodsLibrary
│   ├── Assets
│   └── Classes
├── QSPrivatePodsLibrary.podspec
├── README.md
└── _Pods.xcodeproj -> Example/Pods/Pods.xcodeproj
```   

由于我在创建中选择生成example工程，系统为我创建了一个Demo工程，此外需要关注的便是QSPrivatePodsLibrary文件夹，以及QSPrivatePodsLibrary.podspec文件。前者是我们私有库的库文件包括代码文件以及资源文件，除了系统自动生成的文件夹，我们可以根据自身需求创建文件夹；而后者则是重点需要关注的pods的索引配置文件，pods的工作都是依赖这个文件，具体内容会在下一节讲到。  

最后，额外的关注一下Example工程的Podfile文件，内容如下：  

```ruby  
use_frameworks!

target 'QSPrivatePodsLibrary_Example' do
  pod 'QSPrivatePodsLibrary', :path => '../'

  target 'QSPrivatePodsLibrary_Tests' do
    inherit! :search_paths
  end
end
```  

然后再看一下在Xcode工程下的目录结构：  
![目录结构](./Images/private-pods-file-tree.png)  

可以看到，因为Podfile文件中没有使用常规