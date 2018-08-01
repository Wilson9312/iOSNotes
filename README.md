>随着多次改版和时间的积累，工程越来越大，而此次接入某行SDK后，工程剧增几十M，让我意识到是时候给 app 瘦身了！<br>
经过两天的实践，ipa 从 `70+M` 缩减到了 `50+M` ！


### 一、🚮 资源瘦身
#### 1.  🗑删除未使用的资源

- 🔨工具 [LSUnusedResources](https://github.com/tinymind/LSUnusedResources)

- ⚠️注意1：对于使用 imageName_%d 这种方式使用的图片，为了防止误删，建议勾上

![忽略相似名称](https://user-gold-cdn.xitu.io/2018/8/1/164f4c5914538a5e?w=718&h=70&f=png&s=30117)

>使用时我对比了下勾选和不勾选 Ignore similar name 时的搜索结果，对可能会误删的图片进行逐个确定，意外的防止了一次误删！<br>
图片的名称是这样的，符合 tag_%d 格式：

![tag_%d 格式举例](https://user-gold-cdn.xitu.io/2018/8/1/164f408b6ea08e6c?w=318&h=360&f=png&s=56189)
>然鹅，图片的引用方式是这样的：
`[self.bg_Ima setImage:[UIImage imageNamed:type]];` type动态获取。<br>
误打误撞防止了一次误删，哈哈。

<br>

- ⚠️注意2：不要误删某些SDK里的资源文件，我们的项目中可能未使用过，但谁知道他们呢！
如下图，还不少咧，差点就误删了！

![SDK内的资源文件](https://user-gold-cdn.xitu.io/2018/8/1/164f40bfc15d1d28?w=368&h=486&f=png&s=110217)

- 👨‍💻‍搜索结果：
由于产品经理经常换，导致积压了很多不适用的图片。。。正好这次一锅端了吧！

![删除未使用的资源结果](https://user-gold-cdn.xitu.io/2018/8/1/164f40df18658481?w=702&h=62&f=png&s=23108)

- 🥇结果：工程文件夹小了12.4M

#### 🗑 2.删除重复的资源

- 🔨 工具：[fdupes](https://github.com/adrianlopezroche/fdupes)
- ⏳使用 `HomeBrew` 下载：`brew install fdupes`
- ⚙ 开始使用：`fdupes -r .../Assets.xcassets`

- ⚠️注意：防止 AppIcon 误删

>一个用于 `40 3x` 一个用于 `60 2x` 结果一样，用的是同一张图，这种情况不应该删

![AppIcon 防止误删](https://user-gold-cdn.xitu.io/2018/8/1/164f415b4f7849d3?w=492&h=492&f=png&s=91595)

- 🥉结果：在这个项目中效果不显著，只删除了一张图片。
由于第一步已经删除挺多文件相同、但名字不同、而又不用的图片，在这一步中只删除了一张文件不同、名字相同的图片。

#### 3. 🏞图片无损压缩

- 🔨工具：[ImageOptim](https://imageoptim.com/mac)

- ⚙开始使用：
直接把 工程文件夹 或 `Assets.xcassets` 文件夹拖进去，工具会自动提取图片，进行无损压缩，最后覆盖原图片。

- 🥇结果：
🎉恭喜你节省了 `20.86%` 的空间，超越了 `80%` 的玩家~

![图片无损压缩结果](https://user-gold-cdn.xitu.io/2018/8/1/164f41913cc970d8?w=788&h=52&f=png&s=26861)

#### 4. 📦图片管理方式
- 在项目中添加文件夹存放，适合比较大，但不常使用的图片，通过`-imageWithContentsOfFile:`加载
- 使用 `Assets.xcassets` 管理，适合较小且频繁使用的图片。它会把里边的所有 png 格式的图片压缩成一个`Assets.car`文件，压缩比率比其他方式管理图片要高，大大减少图片体积。

#### 5. 📌其他
- 音视频压缩
- 非必要资源从服务器下载再缓存

<br>

### 二. 🔨利用 linkmap 筛选需要优化的二进制包(可执行文件)

> 二进制包是由各种代码文件、静态库和动态库经过编译后生成的可执行文件。

- 🔨工具：[LinkMap](https://github.com/huanxsd/LinkMap)

- ⚙开始使用：
Xcode -> project -> Build Settings -> Link Map

![Link Map File Setting](https://user-gold-cdn.xitu.io/2018/8/1/164f42833fea612d?w=1396&h=386&f=png&s=86140)

编译运行后，使用工具分析 Link Map File：

![Link Map File 分析结果](https://user-gold-cdn.xitu.io/2018/8/1/164f4303289dcc04?w=512&h=464&f=png&s=71405)

结果一目了然，根据类、库等的大小排序，可以进行针对性优化。

### 三、💼第三方静态库瘦身

#### 1.👨‍💻‍基础知识了解
- arm64：iPhone6s | iphone6s plus｜iPhone6｜ iPhone6 plus｜iPhone5S | iPad Air｜ iPad mini2(iPad mini with Retina Display)
- armv7s：iPhone5｜iPhone5C｜iPad4(iPad with Retina Display)
- armv7：iPhone4｜iPhone4S｜iPad｜iPad2｜iPad3(The New iPad)｜iPad mini｜iPod Touch 3G｜iPod Touch4

- 模拟器32位处理器测试需要i386架构，i386是针对intel通用微处理器32位处理器
- 模拟器64位处理器测试需要x86_64架构，x86_64是针对x86架构的64位处理器
- 真机32位处理器需要armv7,或者armv7s架构，
- 真机64位处理器需要arm64架构

> `armv7` 可以兼容 `armv7s`，打包时 `Xcode` 是不会将模拟器框架打进去的，所以只需要去掉 `armv7s` 即可既达到瘦身效果，又不影响开发和生产。


#### 2. 🔨瘦身的工具：lipo
> `lipo` 可以用来移除 `fat binary` 中不被支持的或者多余的可执行代码，达到瘦身目的。<br>
`lipo` 不会改变程序执行逻辑，仅仅只是文件的大小瘦身。

#### 3. ⚙开始使用

- 无需优化的静态库：

![无需优化的静态库信息](https://user-gold-cdn.xitu.io/2018/8/1/164f43b3265cb5e7?w=1094&h=66&f=png&s=42643)

- 需要优化的静态库：

![需要优化的静态库信息](https://user-gold-cdn.xitu.io/2018/8/1/164f43c33b0cbead?w=1108&h=70&f=png&s=33594)

- 去掉 armv7s 的指令：
```
lipo MESDK -thin armv7 -output MESDK-armv7
lipo MESDK -thin arm64 -output MESDK-arm64
lipo -create MESDK-armv7 MESDK-arm64 -output MESDK-device
```

- 🥇结果：

![第三方静态库瘦身结果](https://user-gold-cdn.xitu.io/2018/8/1/164f43f6bb64e1fe?w=1374&h=422&f=png&s=119235)
 
 
### 四. 📰iOS Thinning

> iOS9 发布了 iOS Thinning，具体介绍了编译器的三项 app 瘦身技术：App 切割(App Slicing)、位源码(Bitcode)和按需加载资源(On Demand Resources)

#### 1. 🔪App Slicing 切割

- 使用 `Assets.xcassets` 管理图片资源
- 适用于 `iOS9` 以后，确切是 `iOS9.0.2` 以后。(这个版本修复了 App Thinning 失效的 bug)
- `itunes` 根据用户设备类型和屏幕分辨率的不同分发定制的下载包，开发者只需要把完整应用包的内容上传到 `iTunes Connect` 即可(编译器自动完成中间的处理)。


#### 2、🔩 Bitcode 位源码

- ⚙使用 Bitcode：
Xcode -> project -> Build Settings -> Enable Bitcode -> Yes

> Bitcode 是程序编译过程中的中间码，介于 LLVM 编译器的前端语言和后端语言(汇编或机器语言)之间。
打包时 Xcode 会将程序编译成 Bitcode 中间码；用户下载时 App store 会再根据具体设备和芯片平台优化二进制文件、减小安装包大小，将 Bitcode 编译成相应的汇编指令以及翻译为机器码，最终成为可执行的64位或32位程序。
<br><br>
Bitcode 的存在，使苹果可以在 LLVM 架构的上面发明新的前端语言，以及在 LLVM 架构的下面支持新的 CPU (后端)指令输出 
-> 
新的前端语言写出来的程序可以兼容旧的设备；已经上架的 App 可以在搭载新的 CPU 的设备运行，而避免了发布新版本的麻烦。
<br><br>
不同设备对 Bitcode 的使用：iOS 可选的，watchOS 必须的，Mac OS 不支持的。
注意：如果我们的工程需要支持 Bitcode，则所有引入的第三方库都要支持 Bitcode。

#### 3、 🔨按需加载资源 On Demand Resources

功能：支持在 App 首次安装后再下载其他资源。

如何开启：
Xcode -> project -> Build Settings -> Enable On Demand Resources -> Yes

应用：游戏开发中，某些游戏关卡的资源可以在玩家付费后再下载；在玩家通过低级关卡并不再去玩时，可以将对应的资源删除以便节省空间。

### 五、🗑 删除无用代码和第三方库

由于 obc 的动态特性，即使是那些没有 import 使用的类，都会在 build 后被编译进可执行文件(build 后观察 linkmap 能看到相关文件信息)；

对于第三方库的依赖和滥用，会导致很多第三发库利用率低或者随着需求的改变压根就没用到，删除这些不用的库；

因此删除这些无用的类和库也是缩小项目大小的有效途径之一。

### 六、⚙ 编译选项设置
- 见思维导图总结

### 最终瘦身结果：
![瘦身结果](https://user-gold-cdn.xitu.io/2018/8/1/164f42bb9dae6b5e?w=1038&h=108&f=png&s=32390)

### 总结：

![思维导图总结](https://user-gold-cdn.xitu.io/2018/8/1/164f47c5dd7a06a2?w=2382&h=2598&f=png&s=623917)

