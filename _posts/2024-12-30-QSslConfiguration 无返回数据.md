---
layout: post
title: "QSslConfiguration 无返回数据"
categories: Qt
tags: Qt
author: August
typora-root-url: ..
---



- [1. 问题排查](#1-问题排查)
  - [1.1. 没有正确初始化 SSL 环境](#11-没有正确初始化-ssl-环境)
    - [1.1.1. 检查点](#111-检查点)
  - [1.2. QSslConfiguration 对象未正确获取](#12-qsslconfiguration-对象未正确获取)
    - [1.2.1. 检查点](#121-检查点)
  - [1.3. 服务器未启用 SSL/TLS](#13-服务器未启用-ssltls)
    - [1.3.1. 检查点](#131-检查点)
  - [1.4. 证书问题](#14-证书问题)
    - [1.4.1. 检查点](#141-检查点)
    - [1.4.2. 解决方案](#142-解决方案)
  - [1.5. Qt 版本问题](#15-qt-版本问题)
    - [1.5.1. 检查点](#151-检查点)
    - [1.5.2. 解决方案](#152-解决方案)
- [2. 测试函数](#2-测试函数)



该文主要介绍 `QSslConfiguration` 无返回数据问题。



# QSslConfiguration 无返回数据



## 1. 问题排查

### 1.1. 没有正确初始化 SSL 环境

确保 `SSL` 库正确加载。`Qt` 通常使用 `OpenSSL` 作为 `SSL` 支持。如果 `OpenSSL` 库缺失或版本不匹配，`QSslConfiguration` 可能无法正常工作。

#### 1.1.1. 检查点

- 确认 `OpenSSL` 库（如 `libssl.so` 和 `libcrypto.so`）在系统中已安装。
- 如果在 `Windows` 系统上，请确保相关 `DLL`（如 `libssl-1_1.dll` 和 `libcrypto-1_1.dll`）与应用程序放在同一目录。

> 跟着教程，一般就是没有将 `libssl-1_1.dll` 和 `libcrypto-1_1.dll` 放到程序运行目录导致获取不到数据



### 1.2. QSslConfiguration 对象未正确获取

如果试图从 `QSslSocket` 或 `QNetworkRequest` 获取 `QSslConfiguration`，确保相关对象已建立 `SSL` 连接。

#### 1.2.1. 检查点

- `QSslSocket` 的 `connectToHostEncrypted` 是否成功执行。
- `QNetworkReply` 是否返回了有效的 `SSL` 配置信息。



### 1.3. 服务器未启用 SSL/TLS

如果服务器未正确配置 `SSL/TLS`，`QSslConfiguration` 将无法获取 `SSL` 会话信息。

#### 1.3.1. 检查点

- 确保服务器支持 `SSL/TLS`，并提供正确的证书。
- 使用工具（如 `curl` 或 `openssl s_client`）手动验证服务器 `SSL` 状态。



### 1.4. 证书问题

服务器的证书可能无效或无法被识别。

#### 1.4.1. 检查点

- 确保证书链完整，根证书受信任。
- 如果使用自签名证书，必须明确信任。

#### 1.4.2. 解决方案

设置忽略证书错误（调试时使用，正式环境不推荐）



### 1.5. Qt 版本问题

某些版本的 `Qt` 在特定平台上可能有已知问题，导致 `SSL` 配置无法正确获取。

#### 1.5.1. 检查点

- 确认使用的是支持 `SSL` 的 `Qt` 版本（如 Qt 5.15 或 Qt 6）。
- 检查是否需要修复或更新。

#### 1.5.2. 解决方案

- 更新到最新的 `Qt` 小版本。
- 如果可能，尝试在其他系统环境中运行以排除平台问题。



## 2. 测试函数

```cpp
#include <QSslSocket>
#include <QSslConfiguration>
#include <QDebug>

// 检查是否支持 SSL
void checkSslSupport() {
    qDebug() << "SSL support:" << QSslSocket::supportsSsl();
    qDebug() << "SSL library version:" << QSslSocket::sslLibraryVersionString();
    qDebug() << "SSL build version:" << QSslSocket::sslLibraryBuildVersionString();
}

// 测试连接到指定主机并获取 SSL 配置
void testSslConnection(const QString &host, int port) {
    QSslSocket sslSocket;
    sslSocket.connectToHostEncrypted(host, port);

    if (sslSocket.waitForEncrypted(5000)) { // 等待加密连接成功
        qDebug() << "Connected to" << host << "on port" << port;
        QSslConfiguration sslConfig = sslSocket.sslConfiguration();
        qDebug() << "Cipher in use:" << sslConfig.cipher().name();
        qDebug() << "Protocol in use:" << sslConfig.protocolString();
        qDebug() << "Peer certificate:" << sslConfig.peerCertificate().toText();
    } else {
        qDebug() << "Failed to connect or encrypt:" << sslSocket.errorString();
    }
}

// 设置默认 SSL 配置（调试用，忽略证书验证）
void setDefaultSslConfiguration() {
    QSslConfiguration sslConfig = QSslConfiguration::defaultConfiguration();
    sslConfig.setPeerVerifyMode(QSslSocket::VerifyNone); // 忽略证书验证
    QSslConfiguration::setDefaultConfiguration(sslConfig);
    qDebug() << "Default SSL configuration updated to ignore certificate errors.";
}

// 使用 OpenSSL 验证主机证书（外部工具辅助）
void verifyServerCertificate(const QString &host, int port) {
    qDebug() << "Use the following command to manually verify the server certificate:";
    qDebug() << QString("openssl s_client -connect %1:%2").arg(host).arg(port);
}

// 主函数，调用测试逻辑
int main(int argc, char *argv[]) {
    QCoreApplication app(argc, argv);

    qDebug() << "Checking SSL support:";
    checkSslSupport();

    QString host = "www.baidu.com";
    int port = 443;

    qDebug() << "\nTesting SSL connection to" << host << "on port" << port << ":";
    testSslConnection(host, port);

    qDebug() << "\nSetting default SSL configuration to ignore certificate errors (debug only):";
    setDefaultSslConfiguration();

    qDebug() << "\nManually verify server certificate with OpenSSL:";
    verifyServerCertificate(host, port);

    return 0;
}
```



# 参考
