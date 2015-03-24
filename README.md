# iOS 开发流程笔记

- [x] 证书知识及准备工作

- [ ] 几种开发者帐号区别

- [ ] 真机调试流程

- [ ] 内测发布流程

- [ ] App store 上架流程

## 证书知识及准备工作

- 什么是证书?

由 apple 官方颁发, 用以证明开发者身份的特殊文件, 在 iOS 开发中主要用于代码签名, 保障 iOS 生态的健康安全, 分为开发者证书和发布者证书

- 什么时候需要证书?

只有在本机模拟器调试时无需代码签名, 当 App 需要在真机运行和发布时需要使用相应证书进行签名

- 证书如何获得?

首先需要拥有相应权限的开发者帐号, 通过在本地生成配对的密钥, 向 provisioning portal 提交公钥后换取, 后续证书在使用时会验证本地私钥

- 如何对代码进行签名?

在 xcode 中, 使用描述文件(provision profile 包含调试者证书, 授权设备清单, 应用ID), 在 `Build Settings` 中选择存于 `Keychain Access` 中的证书文件设置调试和发布任务时的代码签名

- 我生成的私钥如何共享给团队成员?

在 `Keychain Access` 中找到导入的证书, 右击导出为包含私钥的 Personal Information Exchange(.p12)文件(导出时可以创建密码), 团队成员再导入 `p12` 证书后就完整包含了证书和私钥

### 证书换取流程

`CSR` -> `cer` -> `p12`

### 证书的需求情况

- 真机调试

  + 描述文件(Provisioning Profiles)

  + 开发者证书(ios_development.cer)

- 发布

  + 描述文件(Provisioning Profiles)

  + 可用于发布的开发者证书(ios_distribution.cer)

- 消息推送后端服务

  + apns 证书

### 证书及相关文件介绍

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

`cer` 证书跟签名绑定只能在签名机器上使用, 如果要迁移机器需要导出为 `p12` 文件

##### 生成方法

在 [开发者中心](https://developer.apple.com/devcenter/ios/index.action) "certificates" 面板中添加 `certificate` 并上传刚刚生成的 `CSR` 文件, 获取 `ios_development.cer`

#### apns 证书(Apple Push Notification Service)

用于服务端消息推送, 类似 ssl 证书使用, 和 App 端的开发打包没有关系

##### 生成方法

在 [开发者中心](https://developer.apple.com/devcenter/ios/index.action) "Identifiers" 面板中添加 `App ID` 并上传刚刚生成的 `CSR` 文件, 获取 `aps_production.cer`

#### p12 证书

`p12` 证书实际是包含了 `cer` 证书及私钥信息, 可以分发给团队成员

##### 生成方法

在 **Keychain Access** 中找到已经导入的 `cer` 证书, 点右键导出为 `p12` 格式

#### 描述文件(Provisioning Profiles)

包含 `certificate` `appID` `devices id` 的文件用于在 xcode 调试打包时提供授权的配置信息

##### 生成方法

- 在 [开发者中心](https://developer.apple.com/devcenter/ios/index.action) "Provisioning Profiles" 面板中添加 `iOS Provisioning Profiles` 并上传刚刚生成的 `CSR` 文件, 获取 `.mobileprovision` 文件

- 在 xcode 登录开发者帐号后可以连接开发者中心获取

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

关键点:

- 个人帐号可以真机调试, 发布 appstore, 每年 最多为 100台设备分发
- 公司帐号和个人帐号类似, 只有这两种帐号可以发布 appstore, 主要特权是可以添加多个开发者子账号, 但只允许主账号提交, 发布等操作, 在协同开发时比较灵活, 可以各自管理授权设备等
- 企业帐号**无法用于 appstore 发布**, 但可以不通过 appstore 发布任意 iphone 都可以安装的应用
- 大学帐号不能发布 appstore, 主要拥有真机调试的权限

## 真机调试流程

@TODO

帐号
登录 -> iOS Team Provisioning Profile
不登录 -> 导入证书+描述

## 内测发布流程

@TODO
- ad-hoc
- in-house
- TestFlight
- 越狱
