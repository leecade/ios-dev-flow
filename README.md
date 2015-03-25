# iOS 开发流程笔记

- [x] 证书知识及准备工作
- [x] 几种开发者帐号区别
- [x] 真机调试流程
- [x] 内测发布流程
- [ ] Appstore 上架流程

## 目录导航

- [证书知识及准备工作](#证书知识及准备工作)
  - [基础问题答疑](#基础问题答疑)
    - [什么是证书](#什么是证书)
    - [什么时候需要证书](#什么时候需要证书)
    - [证书如何获得](#证书如何获得)
    - [如何对代码进行签名](#如何对代码进行签名)
    - [我生成的私钥如何共享给团队成员](#我生成的私钥如何共享给团队成员)
  - [各流程中证书的需求情况](#各流程中证书的需求情况)
    - [模拟器调试](#模拟器调试)
    - [真机调试](#真机调试)
    - [打包和发布](#打包和发布)
    - [消息推送后端服务](#消息推送后端服务)
  - [开发中常见的证书及相关文件介绍](#开发中常见的证书及相关文件介绍)
    - [CSR(certificate request) 文件](#csrcertificate-request-%E6%96%87%E4%BB%B6)
    - [开发者证书](#开发者证书)
    - [apns(Apple Push Notification Service) 证书](#apnsapple-push-notification-service-%E8%AF%81%E4%B9%A6)
    - [p12(Personal Information Exchange) 证书](#p12personal-information-exchange-%E8%AF%81%E4%B9%A6)
    - [描述文件(Provisioning Profiles)](##%E6%8F%8F%E8%BF%B0%E6%96%87%E4%BB%B6provisioning-profiles)
  - [附录1: 开发准备相关的网址](#%E9%99%84%E5%BD%951-%E5%BC%80%E5%8F%91%E5%87%86%E5%A4%87%E7%9B%B8%E5%85%B3%E7%9A%84%E7%BD%91%E5%9D%80)
- [几种开发者帐号区别](#几种开发者帐号区别)
  - [关键区别](#关键区别)
- [真机调试流程](#真机调试流程)
  - [基本概念](##%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5)
  - [条件和流程](#条件和流程)
- [内测发布流程](#内测发布流程)
  - [基本概念](#%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5-1)
  - [实现条件](#实现条件)
  - [几种常见的分发途径](#几种常见的分发途径)
  - [附录2: 常见分发渠道及工具地址](#%E9%99%84%E5%BD%952-%E5%B8%B8%E8%A7%81%E5%88%86%E5%8F%91%E6%B8%A0%E9%81%93%E5%8F%8A%E5%B7%A5%E5%85%B7%E5%9C%B0%E5%9D%80)
- [Appstore 上架流程](#appstore-%E4%B8%8A%E6%9E%B6%E6%B5%81%E7%A8%8B)
  - [附录3: App store最新审核标准(2015.3)](#%E9%99%84%E5%BD%953-app-store%E6%9C%80%E6%96%B0%E5%AE%A1%E6%A0%B8%E6%A0%87%E5%87%8620153)

## 证书知识及准备工作

### 基础问题答疑

#### 什么是证书

由 apple 官方颁发, 用以证明开发者身份的特殊文件, 在 iOS 开发中主要用于代码签名, 保障 iOS 生态的健康安全, 分为开发者证书和发布者证书

#### 什么时候需要证书

只有在本机模拟器调试时无需代码签名, 当 App 需要在真机运行和发布时需要使用相应证书进行签名

#### 证书如何获得

首先需要拥有相应权限的开发者帐号, 通过在本地生成配对的密钥, 向 [provisioning portal](https://developer.apple.com/ios/manage/overview/index.action) 提交公钥后换取, 后续证书在使用时会验证本地私钥

#### 如何对代码进行签名

在 xcode 中, 使用描述文件(provision profile 包含调试者证书, 授权设备清单, 应用ID), 在 `Build Settings` 中选择存于 `Keychain Access` 中的证书文件设置调试和发布任务时的代码签名

#### 我生成的私钥如何共享给团队成员

在 `Keychain Access` 中找到导入的证书, 右击导出为包含私钥的 Personal Information Exchange(.p12)文件(导出时可以创建密码), 团队成员再导入 `p12` 证书后就完整包含了证书和私钥

### 各流程中证书的需求情况

#### 模拟器调试

不需要

#### 真机调试

- 描述文件(Provisioning Profiles)

- 开发者证书(ios_development.cer)

#### 打包和发布

- 描述文件(Provisioning Profiles)

- 可用于发布的开发者证书(ios_distribution.cer)

#### 消息推送后端服务

- apns 证书

### 开发中常见的证书及相关文件介绍

#### CSR(certificate request) 文件

用于换取证书的公钥文件, 实际是在本地基于 `RSA` 加密得到配对的密钥, 私钥存于 `Keychain Access` 用于签名, 公钥作为换取证书的凭证

##### 生成方法

- OSX 系统自带的 **Keychain Access**

  + 选择 "Request a Certificate From a Certificate Authority…"

  + 输入 email 等信息后保存为 `.certSigningRequest` 文件

- 命令行下使用 **openssl** 生成

```bash
$ openssl genrsa -out private.key 2048
$ openssl req -new -sha256 -key private.key -out my.certSigningRequest
```

#### 开发者证书

由 apple 官方颁发, 用来证明开发者资格的证书文件, 分开发(ios_development.cer)和发布(ios_distribution.cer)两种

`cer` 证书跟开发机(私钥)绑定只能在拥有私钥的机器上使用, 如果要迁移机器需要导出为 `p12` 文件

##### 生成方法

在 [开发者中心](https://developer.apple.com/devcenter/ios/index.action) "certificates" 面板中添加 `certificate` 并上传刚刚生成的 `CSR` 文件, 获取 `ios_development.cer`

#### apns(Apple Push Notification Service) 证书

用于服务端消息推送, 类似 ssl 证书使用, 和 App 端的开发打包没有关系

##### 生成方法

在 [开发者中心](https://developer.apple.com/devcenter/ios/index.action) "Identifiers" 面板中添加 `App ID` 并上传刚刚生成的 `CSR` 文件, 获取 `aps_production.cer`

#### p12(Personal Information Exchange) 证书

`p12` 证书实际是包含了 `cer` 证书及私钥信息, 可以分发给团队成员

##### 生成方法

在 **Keychain Access** 中找到已经导入的 `cer` 证书, 点右键导出为 `p12` 格式

#### 描述文件(Provisioning Profiles)

包含 `certificate` `appID` `devices id` 的文件用于在 xcode 调试打包时提供授权的配置信息

##### 生成方法

- 在 [开发者中心](https://developer.apple.com/devcenter/ios/index.action) "Provisioning Profiles" 面板中添加 `iOS Provisioning Profiles` 并上传刚刚生成的 `CSR` 文件, 获取 `.mobileprovision` 文件

- 在 xcode 登录开发者帐号后可以连接开发者中心获取

### 附录1: 开发准备相关的网址

> 开发者中心
> https://developer.apple.com/devcenter/ios/index.action
> 
> iOS 描述管理(配置证书、描述文件、推送服务)
> https://developer.apple.com/ios/manage/overview/index.action
> 
> 切换团队(在 web 界面上死活没有找到)
> https://developer.apple.com/account/selectTeam.action
> 
> iOS 上架 Appstore
> http://itunesconnect.apple.com/

## 几种开发者帐号区别

> 详见: https://developer.apple.com/programs/start/ios/

- [个人(individual)](https://developer.apple.com/programs/ios/) **$99**/year
- [公司(company)](https://developer.apple.com/programs/ios/) **$99**/year
- [企业(enterprise)](https://developer.apple.com/programs/ios/enterprise/) **$299**/year
- [大学(University)](https://developer.apple.com/programs/start/university/) **free**

### 关键区别

- 个人帐号可以真机调试, 发布 appstore, 每年 最多为 100台设备分发
- 公司帐号和个人帐号类似, 只有这两种帐号可以发布 appstore, 主要特权是可以添加多个开发者子账号, 但只允许主账号提交, 发布等操作, 在协同开发时比较灵活, 可以各自管理授权设备等
- 企业帐号**无法用于 appstore 发布**, 但可以不通过 appstore 发布任意 iphone 都可以安装的应用
- 大学帐号不能发布 appstore, 主要拥有真机调试的权限

## 真机调试流程

### 基本概念

真机调试指 mac 连上 iphone, xcode 可以直接以这台 iphone 设备为 build target, 能在 iphone 里执行编译结果

### 条件和流程

分为拥有独立开发者帐号(也包括公司帐号或企业帐号成员)和共享开发者帐号两种情况

#### 拥有独立开发者帐号

- 1. 在 [provisioning portal](https://developer.apple.com/ios/manage/overview/index.action) 新建应用, 配置授权设备等
- 2. 开发机上导入证书
- 3. 在 xcode 上登录开发者帐号, 不需要准备描述文件, xcode 会自动生成(如果是公司帐号可以自动生成 `iOS Team Provisioning Profile`)

#### 共享开发者帐号

如果无法在 xcode 登录一个开发者帐号, 也可以通过他人对你手机和应用 id 的授权, 得到 `.mobileprovision` 描述文件再导入其含私钥的证书(`p12`) 即可, 具体步骤如下:

- 1. 获得手机的 `udid` (可以连上 mac, 在 itunes 中查看)
- 2. 告知对方 `udid` (用以设备授权) 和 应用 id
- 3. 得到对方生成的证书和描述文件后, 先导入 `p12` 证书, 再双击 `mobileprovision` 文件
- 4. 连接手机, 在 xcode 中选择 build target 为已连接的手机

> 对刚入门的个人开发者而言, 可以在淘宝搜 `iOS真机调试` 花几元购买一份授权, 包含(`p12` 证书 和 `.mobileprovision` 描述文件)

## 内测发布流程

### 基本概念

当 App 开发进行到一定程度, 需要更多的人参与测试, 需要谋求一种方式方便应用能安装进更多的设备中

### 实现条件

进行内测发布主要的关键点是:

- 1. 是如何将应用打包为 `.ipa`

xcode6 以后, 个人/公司帐号无法对应用打包为 `.ipa`, 要么用 xcode5 打包要么拥有企业帐号级别的授权

- 2. 设备需不需要授权

个人/公司帐号权限只有在 `TestFlight` / 越狱渠道下完成不授权安装; 企业帐号授权可以在 `ad-hoc` / `in-house` 渠道下分发, 完成不授权设备安装

### 几种常见的分发途径

- ad-hoc

打包时必须在登录企业帐号(或其成员)并已导入证书和描述文件的情况下, 任何用户(未授权)都可以在手机上用浏览器访问一个 url(例: itms-services://?action=download-manifest&url=https://example.com/manifest.plist) 完成安装

最大的问题是安装量有 100 的上限, 无法作为一个量很大的分发渠道

- in-house

针对企业内部用户进行分发, 相比 `ad-hoc` 无安装量上限

> iOS 8.1.3 开始不能企业证书 Iresign 方式重新签名的应用无法安装
> https://support.apple.com/en-us/HT204245

- TestFlight

仅支持 **iOS8.0** 以上, 不需要对设备 `udid` 进行授权, 适合个人 / 公司开发者, 在应用发布前可以开启 TestFlight Beta 测试并添加测试者的 iTunes Connect 帐号, 需要待测用户拥有 iTunes Connect 帐号并在设备安装 `TestFlight` 客户端

这种方式非常便于推送应用更新和收集测试信息

- 导出 ipa 包, 越狱安装

如果测试设备都越狱了, 这种方式非常灵活简单, 只有能导出 ipa 包就能通过 [itools](http://www.itools.cn/) 等第三方工具安装

### 附录2: 常见分发渠道及工具地址

> fir-第三方应用托管平台
> http://fir.im/
> 
> TestFlight
> https://developer.apple.com/testflight/
> 
> Agile-百度内部 ios 分发测试平台
> http://agile.baidu.com
> 
> fir-分发相关工具
> http://fir.im/dev/tools
> 
> itools
> http://www.itools.cn/

## Appstore 上架流程

@TODO

### 附录3: App store最新审核标准(2015.3)

> [App store最新审核标准(2015.3) 中文翻译](Appstore最新审核标准_2015-3.md)

> [App store最新审核标准(2015.3) 英文原版](https://developer.apple.com/app-store/review/guidelines)
