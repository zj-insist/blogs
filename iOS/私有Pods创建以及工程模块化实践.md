# 私有Pods创建以工程模块化实践  

最近，项目中需要做一个独立性的基础模块，目的是要配置成第三方库类似的使用模式，没有业务依赖，但是引用便捷。当前，iOS开发中通常都是使用CocoaPods维护项目中的第三方库，之前也曾接触过使用Pods管理项目子模块的项目，但当时对CocoaPods都是似懂非懂，没有深入的研究。现在刚好借助这个机会了解Pods的整个运行机制。  

## 私有Pods创建以及维护  

私有库的创建CocoaPods的官网有相当明确的文档，网上也有相当多的博客整理，总结之后大致可以分为以下几个步骤：  

1.创建与添加私有Specs Repo  
2.创建私有Pods工程文件  
3.对私有Pods的Podspec文件进行配置  
4.推送私有Pods的Podspec文件至私有Repo  
5.使用与维护  

#### 1. 创建与添加私有Specs Repo  

关于什么是Specs Repo，可以简单的理解为配置文件仓库。Pods的Search、Insatll、update等操作都依赖这个Specs Repo。官方提供了 `pod repo list` 用于查看当前Cocoapods下的所有Specs Repo，输入命令可以看到以下内容：  

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

私有库进行部分打码…我之前添加过两个repo，通常状态下只有一个master的repo，而这个master正是pods官方支持的所有第三方库的repo。三个repo都有一个相同的path前缀，我们切换到.cocoapods文件夹下使用tree命令查看这个目录的文件结构，关于tree命令的相关内容，可以点击[这里](http://yijiebuyi.com/blog/c0defa3a47d16e675d58195adc35514b.html)查看。

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

截去master中过长的内容，可以看到其中主要包含每个repo中包含的库和其对应版本的配置文件。由于在安装pods的时候我们便已经将master clone到本地，因此我们可以对pods中支持的库进行常规的pods操作。那么要建立私有库的步骤就很明显了：创建自己的repo->clone这个repo->添加私有库的配置文件到这个repo。这样，就能像操作常规的第三方库一样操作我们自己创建的私有库了。  

创建自己的repo首先需要一个git仓库，如何创建根据自身需求，这里作为例子使用GitHub；然后添加这个仓库为repo，使用如下命令：  

```zsh  
  # pod repo add [Private Repo Name] [GitHub HTTPS clone URL]
  ➜ pod repo add QSPrivateRepo https://github.com/zj-insist/PrivatePodsRepo.git
```  

repo的名称并不需要跟git仓库中的一样，可以使用自定义名称。repo仓库并不需要所有项目的参与者都在本地添加，其主要作用是为了更新私有库的版本号以供使用者更新。如果只有一个人维护其中私有库版本，则只需维护者添加这个repo，参与这个repo的push工作，使用者只需在podfile中进行相关配置便可正常使用。  


#### 2. 创建私有Pods工程文件  

这一步，最简单的方法便是创建一个文件夹，然后把工程文件放入这个文件夹，最后，创建一个远程git仓库，管理这个工程。至此，便完成了第二步。  

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

最后，额外的关注一下Example工程的Podfile文件，内容如下：  

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

因为在Podfile中使用本地路径引入的第三方库，所以第三方库以Development Pods的形式引入项目，两者的主要区别是Development Pods中的代码是可以更改的，而Pods中的内容更改是被锁定的。根据命名和特性也很明显，Development Pods主要用于第三方或私有库的开发，Pods则主要是管理第三方库。  

创建好目录和私有库必要的文件，编写好私有库功能，然后将私有库中的文件推送到git，这个git便是要在podsepc文件中配置的第三方库的项目地址；此外，还要给项目添加一个tag，用于标示私有库的版本，这个tag需要和podspec中配置的version对应。CocoaPod在管理私有库时，会根据podsepc文件中的version获取项目对应tag位置的版本，以此达到版本控制的目的。如果在podspec中配置的版本在项目中没有对应的tag，则podspec文件的验证会无法通过。  
简单来说，修改私有库上传git后，需要给这个上传的版本打上一个tag，同步的，要更新这个私有库中podsepc文件中的version为这个tag的值。    

#### 3. 配置Podspec文件  

完成了私有库功能的编写，剩下就是把创建的私有库集成到CocoaPods。CocoaPods管理第三方库主要是依赖podspec文件中的配置，这一步要做的便是在私有库目录下创建podspec文件并按照官方定义的规范编写。pods中同样也集成了生成podspec文件的功能，使用如下命令创建podspec文件：  

```zsh
pod spec create [PodsName] [GitHub HTTPS clone URL]
```  

其中Name便是使用中私有库或者第三方库的名字，GitHub中的内容为选填，此处输入一个私有库的git地址，CocoaPods会自动将项目中的部分信息填充到需要配置的podspec文件，使用命令生成模板文件：  

```zsh
pod spec create test https://github.com/zj-insist/QSPrivatePodsLibrary.git
```

从[这个](https://github.com/zj-insist/QSPrivatePodsLibrary)项目地址生成名为test的podspec文件，删掉部分非必要配置，生成的模板文件如下：  

```ruby
Pod::Spec.new do |s|
  s.name         = "test"         #名称
  s.version      = "0.1.1"        #版本号
  s.summary      = "私有pods测试"  #简介
  #项目描述
  s.description  = <<-DESC       
                  这里填充项目描述
                   DESC

  s.homepage     = "https://github.com/zj-insist/QSPrivatePodsLibrary"  #主页

  #license相关信息
  s.license      = "MIT (example)"
  # s.license      = { :type => "MIT", :file => "FILE_LICENSE" }

  #作者相关信息
  s.author             = { "Jie_ZJ" => "email@address.com" }
  # s.authors            = { "Jie_ZJ" => "email@address.com" }

  #支持系统及版本相关信息
  s.platform     = :ios

  #项目地址及引用版本
  s.source       = { :git => "https://github.com/zj-insist/QSPrivatePodsLibrary.git", :tag => "#{s.version}" }

  #文件及资源文件配置
  s.source_files  = "Classes", "Classes/**/*.{h,m}"
  # s.resources = "Resources/*.png"

  #动态库以及静态库配置
  s.framework  = "SomeFramework"
  # s.frameworks = "SomeFramework", "AnotherFramework"
  s.library   = "iconv"
  # s.libraries = "iconv", "xml2"

  #项目相关配置，编译环境、第三方库依赖等
  s.requires_arc = true
  s.dependency "JSONKit", "~> 1.4"

end
```   
[官方文档](https://guides.cocoapods.org/syntax/podspec.html#group_root_specification)中有更多更详细的说明，文档目录中还很友好的使用`R`标示必填内容：  

![Cocoapods-require-parameters](./Images/Cocoapods-require-parameters.png)  

根据官方文档编写好podspec文件，在上传之前还需要通过验证，否则无法完成上传，切换到包含podspec文件的目录，使用命令：  

```zsh
pod spec lint --allow-warnings  
#以下命令也可验证podspec文件，但是只适用于本地podspec文件的验证，不适用于推送到远端的podspec验证，关于区别会在之后说明
#pod lib lint --allow-warnings  
```  

`--allow-warnings`参数可有可无，但是验证的标准似乎比较严格，很多对项目毫无影响的问题也会造成验证不通过，如果没有对项目有影响的警告信息，建议可以使用该参数通过验证。验证通过后，会有如下提示信息：  

```zsh
 -> QSTestPodsLibrary (0.1.1)
    - WARN  | summary: The summary is not meaningful.
    - WARN  | description: The description is shorter than the summary.

Analyzed 1 podspec.

QSTestPodsLibrary.podspec passed validation.
```  

验证通过后，一个可使用CocoaPods管理的私有库便创建完成。  

#### 4. 共享私有库

完成私有库的创建后，便要实现私有库的共享。这时，便要使用在第一步创建的私有repo，这个repo便是存储私有库信息的仓库，在命令行推送已经通过验证的podspec文件到repo：  

```zsh
pod repo push [Location RepoName] [Podspec Name] --allow-warnings
```  

这里需要注意的是，push的repo名字是在本地创建的名字，pod会从本地的名字索引git地址，如果此处名字和本地不匹配，则推送不成功，这也是管理私有repo需要把这个repo clone到本地的原因。添加`--allow-warnings`参数的原因和验证时一致，此处不赘述。  

推送成功后，便可以在添加过包含这个私有库的repo中使用pods的查询操作查询这个库：  

```zsh
QSPrivatePodsLibrary git:(master) pod repo push QSPrivateRepo QSTestPodsLibrary.podspec

Validating spec
 -> QSTestPodsLibrary (0.1.1)
    - WARN  | summary: The summary is not meaningful.
    - WARN  | description: The description is shorter than the summary.

-> QSTestPodsLibrary (0.1.1)
   A short description of QSTestPodsLibrary.
   pod 'QSTestPodsLibrary', '~> 0.1.1'
   - Homepage: https://github.com/zj-insist/QSPrivatePodsLibrary
   - Source:   https://github.com/zj-insist/QSPrivatePodsLibrary.git
   - Versions: 0.1.1, 0.1.0 [QSPrivateRepo repo]
```  

至此，一个私有的CocoaPod仓库创建完成，只要在Podfile中进行一些配置，便可像使用第三方仓库一样使用我们自己的私有库。  

#### 5. 私有库的使用和维护  

如果上面的四步都顺利通过，则进入了私有库的分发过程。在Podfile编写好要引用的库，然后执行pod install会提示找不到我们创建的私有库，这是因为CocoaPod在搜寻私有库时会有一个source，而这个source默认为官方的源，我们创建的私有库没有添加到这个源，所以提示找不到。所以此时需要在Podfile文件开头添加我们创建的私有repo地址：  

```ruby
source 'https://github.com/zj-insist/PrivatePodsRepo.git'
```  

再运行，发现还是无法执行，提示官方源中的第三方库找不到，这是因为修改了源的地址为我们的私有repo，所以还要再添加一次官方的源：  

```ruby   
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/zj-insist/PrivatePodsRepo.git'
```  

这样，就可以像管理第三方库一样管理自己的私有库了。在私有库更新后，执行pod update同样可以完成私有库在引入项目中的更新。  

私有库的更新需要以下步骤：  
1. 在更新私有库代码后上传git，提交一个tag，然后同步的更新podspec中的version，确保这个version和tag一致。
2. 在本地验证更新后的podspec文件
3. 将验证通过后的podspec文件上传私有repo  

删除一私有库不需要借助CocoaPods，只需要像操作普通的工程一样，切换到要删除的私有库目录，然后删除这个私有库的目录及内部配置文件，最后更新这个私有库的改动，这样就完成了私有库的删除工作。当然，这个操作只是删除了私有库在repo中的配置信息，并没有对私有库的内容做改动，想添加回来再次push私有库的podspec文件到私有repo即可。  

第一部分只介绍了创建一个私有库的基本操作，还有比如使用子模块、创建framework以及静态库等技巧，有兴趣的可以更深入的了解一下。  

## 私有pods创建过程中的部分坑  

虽说根据教程创建私有库也就四步工作，但实际实践起来还是有很多坑的，遇到的坑做如下记录：  

1.tag移动对私有库的影响   

可能是对tag的理解不够深入，导致了一系列奇怪的问题，因为我使用sourceTree这种GUI的git管理工具，每次移动标签都会报标签已存在的错误，所以有些问题也不能确保就是移动标签造成的，很可能是我的标签没有移动到正确的位置。  

首先，一个比较明显的问题，发布一个新版本后，不可以通过修改tag的位置更新私有库的内容。简单来说，发版后，这个tag就没卵用了，即使我这时候删除这个tag，私有库依然可以正常被引用。结合验证时需要项目中存在与version对应的tag号，否则不能通过验证分析，猜测CocoaPods内部存储了这个tag对应的commit编号，在上传完成后，这个编号固定，我们修改或者删除tag并不会对这个编号造成影响。tag只是对外的一种标示，真正使用的是commit编号。但是，我并没有在私有repo中找到存储commit编号的记录，所以以上为不负责猜测，有时间可以研究一下CocoaPods的实现。     

之后，还有一个出现过一次的问题，具体是，我把tag打在了commit编号为A的位置，然后验证podsepc发现有问题不通过，于是修改私有库再提交，这个commit编号为B。这时我不想重新打个tag，就把tag移动到B，之后提交验证，而这个时候验证的报错是我已经修改过的问题，就是验证的还是A位置的记录。具体原因不明，但是更新tag号可以解决这个问题，估计问题还是跟CocoaPods匹配tag和version的内部实现有关系。  

2.私有库依赖其他库的问题  

在私有库中引入第三方库头文件时，使用""而不是<>。  

当私有库依赖另一个私有库时，验证私有库时需要使用如下命令：  
```zsh
pod spec lint --sources='[Private Repo Sources]'
```  

具体可参考[这里](https://stackoverflow.com/questions/27303475/cocoapods-unable-to-find-a-specification-for-privatespec-depended-upon-by-pr)。  

3.pod lib lint和pod spec lint命令的区别  

pod lib lint只从本地验证podspec文件的合法性，而pod spec lint则是从本地和远端都进行验证  

根据需求使用两个命令，如果要上传远端，则需要使用pod spec lint  

4.私有库资源文件的引入方式  

具体可以参考[Cocoapods 使用私有库中遇到的坑](http://www.jianshu.com/p/1e5927eeb341)中的第6点，需要额外强调的是，如果使用xcassets存放图片资源，则在Podfile文件中不要包含`use_frameworks!`。  

## 工程模块化实践  
