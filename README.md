# xray nginx  配置文件

## 📌 使用说明

- `your.domain.com` → 你的主域名(建议开启CDN)
  
- `reality.a.com` → 你的 Reality 伪装域名
  
 > ⚠️ **重要：使用前，请删除所有 `//` 后面的注释，否则配置会报错！**
  

### **🛠 参数生成**

```
在服务器执行

生成 UUID：  
xray uuid  

生成 VLESSENC配置 具体介绍见发布页 [![VLESSENC](https://img.shields.io/badge/VLESSENC-%E5%8F%91%E5%B8%83%E9%A1%B5%E9%9D%A2-green.svg)](https://github.com/XTLS/Xray-core/pull/5067)
先执行 xray x25519 获取PrivateKey 和 Password (PublicKey)
再执行 xray mlkem768 获取 Seed 和 Client
服务端 "decryption": "mlkem768x25519plus.native.600s.PrivateKey.Seed"
客户端 "encryption": "mlkem768x25519plus.native.0rtt.Password.Client"

生成 Reality 密钥：  
xray x25519  

生成 shortId：  
openssl rand -hex 8
```


## Disclaimer/免责声明

This project is for educational and research purposes only.

Users must comply with all applicable laws and regulations in their respective jurisdictions when using this project. The author is not responsible for any misuse or illegal activities.

Do not use this project for any unlawful purposes.

This project does not provide, promote, or imply any specific usage scenarios. Users are solely responsible for how they use this project.
