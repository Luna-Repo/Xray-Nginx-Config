# Xray + Nginx All in 443 配置示例

一个基于 Nginx SNI 分流的 Xray 多协议共存配置示例。

支持：

* VLESS + XHTTP
* VLESS + REALITY
* TLS 1.3 / TLS 1.2
* VLESS Encryption（ML-KEM-768 + X25519）
* Nginx SNI 分流，共用 443 端口

## 📌 配置说明

本配置通过 Nginx 监听 443 端口，首先根据 TLS SNI 进行流量分流，然后根据 HTTP 路径将 XHTTP 请求转发至 Xray。

示例：

```
                     Client
                       |
                       |
                    TCP 443
                       |
                    Nginx
                       |
          +------------+------------+
          |                         |
 reality.a.com                 your.domain.com
          |                         |
          |                         |
 Xray REALITY                 HTTPS + XHTTP
 127.0.0.1:10002              127.0.0.1:10000
                                    |
                                    |
                              Xray VLESS
                              127.0.0.1:10001
```

流量处理逻辑：

* `reality.a.com`

  * 转发至 Xray REALITY 服务

* `your.domain.com`

  * 提供正常 HTTPS 网站
  * 指定路径转发至 Xray XHTTP

* 其他未匹配 SNI

  * 直接丢弃

## 🧩 配置文件

完整配置文件位于 `config` 分支：

### Xray 配置

[xray.jsonc](https://github.com/Luna-Repo/Xray-Nginx-Config/blob/config/xray.jsonc)

### Nginx 配置

[nginx.conf](https://github.com/Luna-Repo/Xray-Nginx-Config/blob/config/nginx.conf)

> 注意：
>
> * Xray 配置使用 JSONC 格式。
> * 使用前请删除 `//` 后面的注释，否则 JSON 解析会失败。
> * Nginx 配置支持 `#` 注释。

## 🛠 参数生成

在服务器执行以下命令生成所需参数。

### 生成 UUID

```bash
xray uuid
```

### 生成 VLESS Encryption 参数

VLESS Encryption 使用 ML-KEM-768 + X25519 混合加密。

执行 
```bash
xray x25519
```
获取PrivateKey 和 Password (PublicKey)

执行
```bash
xray mlkem768
```
获取 Seed 和 Client

配置按以下格式填写：

* 服务端 ```"decryption": "mlkem768x25519plus.native.600s.PrivateKey.Seed"```
* 客户端 ```"encryption": "mlkem768x25519plus.native.0rtt.Password.Client"```

详细说明：

[![VLESSENC](https://img.shields.io/badge/VLESSENC-%E5%8F%91%E5%B8%83%E9%A1%B5%E9%9D%A2-green.svg)](https://github.com/XTLS/Xray-core/pull/5067)

### 生成 REALITY 密钥

```bash
xray x25519
```

### 生成 REALITY shortId

```bash
openssl rand -hex 8
```

## ⚙️ 部署前需要修改

配置文件中的以下参数需要替换：

| 参数                   | 说明              |
| -------------------- | --------------- |
| `your.domain.com`    | 主域名             |
| `reality.a.com`      | Reality 伪装域名    |
| `uuid`               | Xray 用户 UUID    |
| `PrivateKey`         | Reality 私钥      |
| `shortId`            | Reality shortId |

## 🔐 安全说明

本配置默认采用以下安全策略：

* Xray 服务仅监听 `127.0.0.1`
* Nginx 作为统一 TLS 入口
* 非指定 SNI 请求直接丢弃
* TLS 使用现代加密套件
* 支持后量子混合密钥交换

注意：

* ML-KEM 后量子混合密钥交换需要较新的 OpenSSL / Nginx 支持。
* 部分旧设备可能不支持 PQC，可根据环境调整 TLS 参数。
* 不建议不了解 TLS 工作机制时随意修改 cipher suite 和密钥交换曲线。

**“如有错误或改进建议，欢迎指正，本配置已在个人环境中测试可用。”** ✅

## 📂 项目结构

```
main branch
   ├── README.md
   └── LICENSE
config branch
    ├── xray.jsonc
    └── nginx.conf
```


**“如有错误或改进建议，欢迎指正，本配置已在个人环境中测试可用。”** ✅

## 📂 项目结构

```
main branch
   ├── README.md
   └── LICENSE
config branch
    ├── xray.jsonc
    └── nginx.conf
```

## License

[![](https://licensebuttons.net/l/by-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-sa/4.0/deed.zh) This project is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0).

## Disclaimer/免责声明

This project is for educational and research purposes only.

Users must comply with all applicable laws and regulations in their respective jurisdictions when using this project. The author is not responsible for any misuse or illegal activities.

Do not use this project for any unlawful purposes.

This project does not provide, promote, or imply any specific usage scenarios. Users are solely responsible for how they use this project.