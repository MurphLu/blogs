这个世界是懒人创造的。

人懒了就会发明各种各样的工具，或者寻找各种各样能够给自己偷懒机会的工具，当然我还停留在使用各种工具的阶段。或者可能不是因为懒，只是为了我们的生活更方便。

回到开发上，传统打包提测的过程比较恶心，尤其是iOS一个工程可能有不同的bundle ID，使用各种各样的配置环境，各种各样的证书打各种各样的包来给测试人员使用，然后会发现，在打包的时候要注意好多东西，各种配置，证书，bundle ID，如果项目复杂的话就会可能打出来的包并不是测试人员要的包，或者哪个地方设置有问题就还得重新打一遍，作为开发人员的我们很头疼的好吧。

为了打包提测整个流程更方便，还有就是作为开发人员哪的我们把重点放在开发上而不是打包上，各种各样的自动打包构建提测工具就应运而生，我们本文提到的fastlane就是一种，集各种强大功能于一身，几乎可以解决我们打包提测整个流程的所有事情。

###安装

这个就不多说了，参见官方文档就好了[fastlane setup](https://docs.fastlane.tools/getting-started/ios/setup/)

### 初始化

官方文档给出了两种语言初始化方法
默认的Ruby的，以及Swift的，由于Swift现在还是beta版本，所以我们还是选取Ruby版本
打开终端，cd到项目根目录

```
fastlane init
```

执行完成之后会发现我们项目根目录多出了两个文件：
fastlane 文件夹——文件夹中包括Appfile（存储配置项，例如bundle ID，itunes_connect_id等），Fasefile（打包脚本）
Gemfile（Ruby的包管理工具，类似于iOS开发中的CocoaPods） 文本文件

```
fastlane action
```

fastlane提供了好多的action，包括证书的创建管理（cert），provisioning profile的创建管理（sigh， match），自动截图，打测试包上传到TestFilght，打线上包上传到App Store，通过各种各样的插件或者action打各种各样的包，打包之前之后进行各种操作等等。。。。由于我这里用到的东西比较少，更多的功能可以自行挖掘[fastlane actions](https://docs.fastlane.tools/actions/)

###fastlane Plugins

fastlane通过各种插件可以拓展更多的功能[available plugins](https://docs.fastlane.tools/plugins/available-plugins/)

```
fastlane search_plugins [pluginName] #通过该命令查找插件
```

```
fastlane add_plugin [pluginName] # 通过该命令安装插件
```

###fastlane CocoaPods

fastlane默认支持CocoaPods，只需要在打包脚本中添加一行`cocoapods`就OK了，但是实际使用的时候可能会有问题，这个问题貌似跟Ruby使用Gem作为包管理方式有关，直接这样写得时候回提示找不到`Pod`，需要在项目根目录中我们做初始化的时候生成的`Gemfile`做一下配置

```Ruby
source "https://gems.ruby-china.org/"
gem "fastlane"
#配置cocoaPods，并指定版本
gem 'cocoapods', '1.3.1'

#引用Pluginfile，我们安装的插件都会在Pluginfile中引入
plugins_path = File.join(File.dirname(__FILE__), 'fastlane', 'Pluginfile')
eval_gemfile(plugins_path) if File.exist?(plugins_path)
```

添加完这一行再次运行就OK了。我们在使用CocoaPods的时候也可以传入一些参数，具体请参见官方文档[CocoaPods](https://docs.fastlane.tools/actions/cocoapods/#cocoapods)。

###打包脚本

由于我们的App测试和发布都是使用的企业账号，所以fastlane并没有企业打包的默认配置，我们使用faselane推荐的[gym](https://docs.fastlane.tools/actions/gym/#gym)来进行打包
我们的App测试开发环境由于推送的种种问题，只能是通过设置两个Bundle ID的方式来实现两种环境的区分，所以，我们在打包之前需要做的事情有：
- 更换Bundle ID
- 更换打包描述文件
- 可能还有版本号的变更
基于此，我们在每次打包之前需要替换掉工程中的Bundle ID，以及使用的描述文件。
经过测试，描述文件只有在使用sigh（本文中只测试了sigh），才能正常动态替换掉描述文件，否则需要手动在Xcode中该成指定的描述文件才可以，所以，我们使用了[sigh](https://docs.fastlane.tools/actions/sigh/#sigh)来管理描述文件。
```
#为了不让我们的项目目录太乱，所以工程无关的文件我们就都输出到指定的目录，这样也方便统一管理
# --adhoc 指定倒出那种描述文件
fastlane sigh -o “~/PP/” --adhoc
```
命令执行过程中会让你输入App ID Userame, Password
选择team， 输入项目Bundle ID，然后就会自动查找并下载安装描述文件，

这样之后就可以在打包之前自动更换描述文件了。

实际使用中发现仅仅使用`update_project_provisioning `这个方法是不行的，应为它的功能仅为替换描述文件，在设置App签名的时候需要指定是否为自动，描述文件，证书，teamID等，如果仅仅是替换描述文件的话是有问题的。
经过在官方文档查询又发现了另外一个方法[automatic_code_signing](https://docs.fastlane.tools/actions/automatic_code_signing/#automatic_code_signing)，这个可以在打包之前为我们详细的指定签名需要的一系列内容

接下来是打包脚本：

```
# 平台，安卓还是iOS
default_platform(:ios) 

# iOS进行的操作
platform :ios do
  desc "Description of what the lane does"
  # 在执行lane之前进行的操作，例如我们在每次打包之前都执行一次pod install
  before_all do
    # faselane中pod install的操作
  	cocoapods
  end
  
  # 打包的lane操作，我们可以配置多个lane来打不同环境的包
  lane :build_uat do |op|
  
    # 打包之前更新bundle ID
  	update_info_plist(
  		plist_path:"./projectName/SupportFiles/Info.plist",
  		app_identifier:"com.example.id"
  	)
        
    # 打包之前跟新指定配置的描述文件
  	update_project_provisioning(
        # 之前有sigh下载的描述文件存储路径
  		profile:"./InHouse_com.example.id.mobileprovision",
                
        # 打包配置，Debug，Release
  		build_configuration:"Debug"
  	)

      automatic_code_signing(
        # 工程文件所在路径
        path:"project.xcodeproj",
        # 是否使用自动签名，这里如果是打包的话应该一般都为false吧，默认也是false
        use_automatic_signing:false,
        # 打包的team ID， 也就是打包使用的证书中的team ID，这个如果不知道是什么的话可以在xCode中设置好签名用的描述文件后到xcodeproj下的pbxproj文件中搜索“DEVELOPMENT_TEAM”，它的值就是了
        team_id:"teamID",
        # 这个就不用说了，需要修改的targets
        targets:"target_name",
        # 用哪种方式打包“iPhone Develop”还是“iPhone Distribution”
        code_sign_identity:"iPhone Distribution",
        # 描述文件名称， 也就是使用哪个描述文件打包
        profile_name:"profile_name"
    )

  	currentTime = Time.new.strftime("%Y-%m-%d-%H-%M")
  	outputDirectory = "./uat/#{currentTime}"
  	logDirectory = "#{outputDirectory}/fastlanelog"
  	gym(
        # 打包方式，enterprise, adhoc,appstore,development
  		export_method: "enterprise",
  		scheme: "scheme",
        # pod 生成的workspace文件
  		workspace:"project.xcworkspace",
        # 输出文件夹
  		output_directory: outputDirectory,
        # 输出包名称
  		output_name:"app.ipa",

        # 打包前是否clean
  		clean:true,
  		silent:true,
        # 打包的配置 Debug Release
  		configuration:"Debug",
        # 打包日志输出文件夹
  		buildlog_path:logDirectory,
        # 打包证书
  		codesigning_identity:'iPhone Distribution: .........',
        # Xcode 9 默认不允许访问钥匙串的内容,必须要设置此项才可以，运行过程可能会提示是否允许访问钥匙串，需要输入电脑密码
  		export_xcargs: "-allowProvisioningUpdates",
        # 导出选项
  		export_options:{ 
            # 打包导出时可选描述文件 "bundleID"=>"描述文件名称"
  			provisioningProfiles: {
                "com.example.id" => "InHouse_com.example.id",
            },

            # inHouse发布时需要的manifest.plist文件中需要配置的项
            manifest: {
		    	appURL: "https://example.com/app.ipa",
		    	displayImageURL: "https://example.com/app.png",
		    	fullSizeImageURL: "https://example.com/app.png",
		  	},
        }
  	)
  end
  
  # 当lane执行完成之后进行哪些操作
  after_all do |lane|

  end

  error do |lane, exception|
  end

end
```

实际使用过程中打包的时候发现会失败，大概报的是这么一个错，里面的Each是一个pod库
```
Code Signing Error: Each does not support provisioning profiles. Each does not support provisioning profiles, but provisioning profile NO_SIGNING/ has been manually specified. Set the provisioning profile value to "Automatic" in the build settings editor.
```
大概就是pod的库不需要签名，但是默认却给他进行签名了，这样就会有问题，然后在fastlane的github的[issue](https://github.com/fastlane/fastlane/issues/10543)中找到了这样的解决办法：
```Ruby
# 将下面代码添加到我们的podfile文件中
post_install do |installer| installer.pods_project.build_configurations.each do |config| config.build_settings['PROVISIONING_PROFILE_SPECIFIER'] = '' end end
```
具体的解释是这样的：
![image.png](http://upload-images.jianshu.io/upload_images/1648999-dc9257a8acc3d584.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后，在我的podfile中贴上了上面那段代码然后就神奇的好了。

之后就在终端中cd到我们的工程跟目录执行
```
fastlane build_uat(lane_name)
```
然后就该干嘛干嘛去了，不用再等待Xcode漫长的打包过程
然后我们可以将faselane与Jenkins等等工具结合起来，打包的时候只需要打开网页点下按钮，其他的都让程序自动帮我们打出各种包，上传到各种地方或者直接发布，通过各种方式告诉测试人员，而且程序做这些事情要比人更靠谱，毕竟不会出现低级错误。

今天尝试了一下通过插件把蒲公英或fir搞进来，最后发现两个插件最后都没能成功运行，还在继续折腾中，等好了继续更新

几乎把我遇到的坑都一一列举了，如果有其他问题请留言，如果那里表达有问题希望大神指正~~
