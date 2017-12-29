title: CI搭建和脚本自动化
photos:
  - null
date: 2016-03-02 17:39:11
tags: iOS
---

## install

```shell
brew cask install jenkins 
```


## lanuch

```shell
jenkins
```

## widget

系统管理->管理插件，在`可选插件`中搜索安装以下插件

* git

* GitLab Plugin 、Gitlab Hook Plugin

* Credentials Plugin 、Keychains and Provisioning Profiles Management  这个可以不装，只用写好配置文件即可

* Post-Build Script Plug-in  


## shell scrpit

```
fir build_ipa -B 'develop' -C Release './' -w -S 'jiuxian' -n jiuxian_appstore
```

```
xcodebuild  clean -configuration inHouse
xcodebuild -workspace "jiuxian.xcworkspace" -scheme jiuxian -configuration inHouse -archivePath build/jiuxian.xcarchive archive
xcodebuild  -exportArchive -exportFormat IPA -archivePath build/jiuxian.xcarchive -exportPath build/jiuxian.
```

## automation
* fastlane
* shell
* nodejs

