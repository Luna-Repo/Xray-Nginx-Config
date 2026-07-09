# xray nginx all in 443 配置示例

## 📌 使用说明

- `your.domain.com` → 你的主域名(建议开启CDN)
  
- `reality.a.com` → 你的 Reality 伪装域名
  
  ## 🧩 Xray 配置
  
  > ⚠️ **重要：复制使用前，请删除所有 `//` 后面的注释，否则配置会报错！**
[![Xray](https://img.shields.io/badge/XRAY-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E9%93%BE%E6%8E%A5-blue.svg)](https://github.com/Luna-Repo/Xray-Nginx-Config/blob/config/xray.jsonc)
```
{
  "log": {
    "loglevel": "none"
  },
  "dns": {
    "queryStrategy": "UseIP",
    "servers": [
      {
        "address": "https://1.1.1.1/dns-query",
        "skipFallback": false
      },
      {
        "address": "1.0.0.1",
        "skipFallback": true
      }
    ]
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 10001,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "email": "user@example.com",
            "flow" : "xtls-rprx-vision",
            "id": "uuid",      //执行"xray uuid"生成
            "level": 0
          }
        ],
        "decryption": "mlkem768x25519plus.native.600s.X25519-PrivateKey.ML-KEM-768-Seed."  //执行"xray x25519"生成X25519-PrivateKey 执行"xray mlkem768" 生成ML-KEM-768-Seed 
      },
      "streamSettings": {
        "network": "xhttp",
        "xhttpSettings": {
          "host": "",
          "mode": "auto",    //这里建议设置为auto兼容3种模式
          "path": "/xhttp-path"
        }
      }
    },
    {
      "listen": "127.0.0.1",
      "port": 10002,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "flow": "xtls-rprx-vision",
            "id": "uuid"     //执行"xray uuid"生成，也可使用上面的uuid
          }
        ],
        "decryption": "none"
      },
      "sniffing": {
        "destOverride": [
          "http",
          "tls",
          "quic"
        ],
        "enabled": true,
        "routeOnly": true
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "dest": "reality.a.com:443",  //请填入你的伪装域名
          "privateKey": "PrivateKey",  //执行"xray x25519"生成
          "serverNames": [
            "reality.a.com"
          ],
          "shortIds": [
            "yourShortIds"  //执行"openssl rand -hex 8"生成
          ]
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "tag": "block"
    }
  ],
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      {
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "block",
        "type": "field"
      },
      {
        "ip": [
          "geoip:cn"
        ],
        "outboundTag": "block",
        "type": "field"
      },
      {
        "domain": [
          "geosite:category-ads-all"
        ],
        "outboundTag": "block",
        "type": "field"
      }
    ]
  }
}
```

### **🛠 参数生成**
[![VLESSENC](https://img.shields.io/badge/VLESSENC-%E5%8F%91%E5%B8%83%E9%A1%B5%E9%9D%A2-green.svg)](https://github.com/XTLS/Xray-core/pull/5067)
```
在服务器执行

生成 UUID：  
xray uuid  

生成 VLESSENC配置 详细配置见VLESSENC发布页面
先执行 xray x25519 获取PrivateKey 和 Password (PublicKey)
再执行 xray mlkem768 获取 Seed 和 Client
服务端 "decryption": "mlkem768x25519plus.native.600s.PrivateKey.Seed"
客户端 "encryption": "mlkem768x25519plus.native.0rtt.Password.Client"

生成 Reality 密钥：  
xray x25519  

生成 shortId：  
openssl rand -hex 8
```

## 🌐 Nginx 配置

> ✅ nginx 支持 `#` 注释，不需要删除注释内容**
[![Nginx](https://img.shields.io/badge/NGINX-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E9%93%BE%E6%8E%A5-blue.svg)](https://github.com/Luna-Repo/Xray-Nginx-Config/blob/config/Nginx.conf)
```
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
    # multi_accept on;
}

#基于SNI分流
stream {
    map $ssl_preread_server_name $backend {
        reality.a.com reality;    #reality伪装域名
        your.domain.com web_xray; #填入你的域名
        default drop;
    }

    upstream reality {
        server 127.0.0.1:10002; #reality端口
    }

    upstream web_xray {
        server 127.0.0.1:10000; #web_xray端口
    }

    upstream drop {
        server 0.0.0.0:1; #丢弃数据包
    }

    server {
        listen 443 reuseport;#监听443tcp
        proxy_pass $backend;
        ssl_preread on;
    }
}

#HTTP 及 HTTPS 主体配置

http {
    # 基本设置
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    server_tokens off;

    # 日志设置
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log warn;

    # 1. HTTP 默认站
  
    server {
        listen 80;
        listen [::]:80;
        server_name your.domain.com;    #填入你的域名

        # 强制跳转到 HTTPS
        return 301 https://$host$request_uri;
    }

    # 2. 域名 HTTPS 配置：
   
    server {
        listen 127.0.0.1:10000 ssl http2;
        listen [::1]:10000 ssl http2;
        server_name your.domain.com;#填入你的域名

        #填入你证书路径
        ssl_certificate /home/admin/cert/domain.crt;
        #填入你证书私钥路径
        ssl_certificate_key /home/admin/cert/domain.key;


        #OCSP装订 如果你的证书不支持OCSP请不要开启 不确定是/否支持建议关闭
        #ssl_stapling on;
        #ssl_stapling_verify on;
        #resolver 8.8.8.8 1.1.1.1 valid=300s;
        #resolver_timeout 5s;

        ssl_protocols TLSv1.3 TLSv1.2;

        #不建议在未理解各协议层级作用范围的情况下随意修改 TLS1.2/1.3 cipher suite 或 ecdh_curve 顺序，否则可能导致：
        # TLS 1.2 安全性下降
        # ChaCha20 优先策略失效
        # PQC hybrid key exchange 回退
        # 兼容性下降或性能异常

        #TLS1.2 cipher
        #配置1全部AEAD套件，旧设备兼容性较差，但安全性较好
        #ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
        #配置2允许AES-CBC套件，旧设备兼容性较好，但安全性稍差
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384;
        
        #对于无AES-IN的设备进行CHACHA20优化 建议保持开启
        ssl_conf_command Options PrioritizeChaCha;

        #TLS1.3 cipher 
        ssl_conf_command Ciphersuites TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384;       
        
        #服务器cipher优先 建议开启
        ssl_prefer_server_ciphers on;

        #session 缓存 与 tickets 开启安全性下降，但性能提升，关闭性能下降，但安全性提升. 适用于低流量 / 单机 / 安全优先场景，高并发建议开启 session cache
        #ssl_session_timeout 30m;
        #ssl_session_cache shared:SessionCache:10m; #大约 40000 sessions
        ssl_session_cache off;
        ssl_session_tickets off;
        
    
        #X25519MLKEM768 SecP256r1MLKEM768 后量子PQ混合密钥交换，需要较新 OpenSSL / Nginx 支持
        ssl_ecdh_curve X25519MLKEM768:X25519:SecP256r1MLKEM768:prime256v1:X448:secp384r1:secp521r1;

        #如果你的环境不支持 ML-KEM，可以降级为 
        #ssl_ecdh_curve X25519:prime256v1:secp384r1:secp521r1;


        # HSTS (HTTP Strict Transport Security)
        # 推荐启用以保障安全性，但请理解其不可逆影响（浏览器缓存期间，当前设置时间为6个月）
        #
        # 注意：
        # 1. 一旦设置 max-age，在有效期内无法通过 HTTP 取消
        # 2. includeSubDomains 会影响所有子域名
        # 3. 不建议在未完全确认 HTTPS 覆盖前启用 preload
        
        add_header Strict-Transport-Security "max-age=15552000" always;
        
        root /home/admin/webpage;  #填入你的网页文件路径
        index index.html;
        
        #  3. xray配置部分                                               
        
        #下方有两种写法，第一种只可以使用xhttp packet-up模式
        #第二种可以使用xhttp所有模式
        #如果你决定使用其中一种配置，请删除另一种配置示例
        #vless-xhttp 示例配置1
        location /xhttp-path {
            proxy_pass http://127.0.0.1:10001;# 填入你xray监听的地址和端口
            proxy_http_version 1.1;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_redirect off;
        }
        #vless-xhttp 示例配置2
        location /xhttp-path {
            grpc_buffer_size 16k;
            grpc_connect_timeout 60s;
            grpc_read_timeout 3600s;
            grpc_send_timeout 3600s;
            grpc_socket_keepalive on;
            grpc_pass grpc://127.0.0.1:10001; # 填入你xray监听的地址和端口
            grpc_set_header Host $host;
            grpc_set_header X-Real-IP $remote_addr;
            grpc_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size 0;
            proxy_redirect off;
        }

       # 其他所有路径 → 尝试找静态文件，找不到返回 主页面

       location / {
        try_files $uri $uri/ /index.html;
       }
    }
}
```

##### 🧠 原理解析

```
数据包发往服务器443端口----->nginx监听443端口
                               |
                               |
                   sni是否为reality伪装域名or你的域名-----否---->丢弃数据包
                        |                 |
                        |                 |                   
 转发本地10002端口<-reality伪装域名       你的域名
 交由xray处理流量                          |
                                          |
                                   转发本地10000端口
                                          |
                                          |
   返回index.html<---否------是否为xhttp设定路径"/xhttp-path"
                                          |
                                          是
                                          |
                                          |
                                转发至本地10001端口
                                交由xray处理流量
```

**“如有错误或改进建议，欢迎指正，本配置已在个人环境中测试可用。”** ✅

## License

[![](https://licensebuttons.net/l/by-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-sa/4.0/deed.zh) This project is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0).

## Disclaimer/免责声明

This project is for educational and research purposes only.

Users must comply with all applicable laws and regulations in their respective jurisdictions when using this project. The author is not responsible for any misuse or illegal activities.

Do not use this project for any unlawful purposes.

This project does not provide, promote, or imply any specific usage scenarios. Users are solely responsible for how they use this project.
