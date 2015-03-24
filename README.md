# 证书及准备工作

## 证书换取流程

`CSR` -> `cer` -> `p12`

## 证书的需求情况

- 真机调试

  + 描述文件(Provisioning Profiles)

  + 开发者证书(ios_development.cer)

- 发布

  + 描述文件(Provisioning Profiles)

  + 可用于发布的开发者证书(ios_distribution.cer)

- 消息推送后端服务

  + apns 证书

## 证书及相关文件介绍

### CSR(certificate request) 文件

用于换取证书的本地私钥文件, 实际是基于 `RSA` 加密得到的当前机器的私钥凭证

#### 生成方法

- OSX 系统自带的 **Keychain Access**

  + 选择 "Request a Certificate From a Certificate Authority…"

  + 输入 email 等信息后保存为 `.certSigningRequest` 文件

- 命令行下使用 **openssl** 生成

```bash
$ openssl genrsa -out private.key 2048
$ openssl req -new -sha256 -key private.key -out my.certSigningRequest
```

### 开发者证书

由 apple 官方颁发, 用来证明开发者资格的证书文件, 分开发(ios_development.cer)和发布(ios_distribution.cer)两种

`cer` 证书跟签名绑定只能在签名机器上使用, 如果要迁移机器需要导出为 `p12` 文件

#### 生成方法

在 [开发者中心](https://developer.apple.com/devcenter/ios/index.action) "certificates" 面板中添加 `certificate` 并上传刚刚生成的 `CSR` 文件, 获取 `ios_development.cer`

### apns 证书(Apple Push Notification Service)

用于服务端消息推送, 类似 ssl 证书使用, 和 App 端的开发打包没有关系

#### 生成方法

在 [开发者中心](https://developer.apple.com/devcenter/ios/index.action) "Identifiers" 面板中添加 `App ID` 并上传刚刚生成的 `CSR` 文件, 获取 `aps_production.cer`

### p12 证书

`p12` 证书实际是包含了 `cer` 证书及私钥信息, 可以分发给团队成员

#### 生成方法

在 **Keychain Access** 中找到已经导入的 `cer` 证书, 点右键导出为 `p12` 格式

### 描述文件(Provisioning Profiles)

包含 `certificate` `appID` `devices id` 的文件用于在 xcode 调试打包时提供授权的配置信息

#### 生成方法

- 在 [开发者中心](https://developer.apple.com/devcenter/ios/index.action) "Provisioning Profiles" 面板中添加 `iOS Provisioning Profiles` 并上传刚刚生成的 `CSR` 文件, 获取 `.mobileprovision` 文件

- 在 xcode 登录开发者帐号后可以连接开发者中心获取
