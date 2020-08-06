# 准备工作
你需要拥有一个自己的**域名**，并**已经将域名解析至你的服务器**    
# 配置环境
硬件 : 内存 ≧ 512M 储存 ≧ 5G | 64位系统			

软件 : Debian 9/10 && Ubuntu 16/18/20
# 配置内容 
- 升级并安装必要软件   
```bash
apt update && apt -y install socat wget vim
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安装脚本 
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 申请证书 （请将 **your_domain.com** 改为你的域名）  
```bash
acme.sh --issue --standalone -d your_domain.com -k ec-256
mkdir /etc/trojan-go
acme.sh --installcert -d your_domain.com --fullchain-file /etc/trojan-go/server.pem --key-file /etc/trojan-go/server.key --ecc
```
- 安装 Docker && Nginx && Trojan     
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/trojan-go
docker pull containrrr/watchtower
```
- 修改 Trojan-go 配置
```bash
vim /etc/trojan-go/config.json
```
<details>
<summary>配置1 ：不使用CDN</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改为你设定的密码
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.pem",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改为你的域名
        "fallback_port": 3000 
    }
}
```
</details>

<details>
<summary>配置2 ：使用CDN，且你信任你的CDN提供商</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改为你设定的密码
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.pem",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改为你的域名
        "fallback_port": 3000 
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path",  #修改为你设定的路径
    "host": "your_domain.com"   #修改为你的域名
    }
}
```
</details>  

<details>
<summary>配置3 ：使用CDN，但你不信任你的CDN提供商（例如国内厂商的CDN）</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改为你设定的密码
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.pem",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改为你的域名
        "fallback_port": 3000 
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path",  #修改为你设定的路径
    "host": "your_domain.com"   #修改为你的域名
    },
    "shadowsocks": {
    "enabled": true,
    "method": "AES-128-GCM",
    "password": "password1"   #修改为另一个密码，请勿与上方密码一致
  }
}
```
</details>

- 修改 Nginx 配置  
```bash
mkdir /etc/nginx && mkdir /etc/nginx/conf.d
vim /etc/nginx/conf.d/default.conf
```
**复制下列配置**  
```bash
server {
    listen 127.0.0.1:80;
    server_name your_domain.com;   #修改为你的域名
    location / {
        proxy_pass https://your_proxy.com;   #修改为你想伪装的网站域名，例如 https://unsplash.com/  
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k;  
    }
}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;  #修改为你服务器的 IP地址
    return 301 https://your_domain.com$request_uri;   #修改为你的域名
}
server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
server {
	listen 127.0.0.1:3000;
	server_name _;
	return 400;
}
```
- 启动服务  
```bash
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
docker run --network host --name trojan-go -v /etc/trojan-go:/etc/trojan-go --restart=always -d teddysun/trojan-go
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- 开启 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
## 更新软件
使用这种配置方式后，**watchtower**会自动监测并更新软件，你无需手动更新

## 客户端的使用 
安卓平台 ：[点击下载](https://github.com/charlieethan/firewall-proxy/releases/download/V0.7.7/Igniter-Go-v0.7.7.apk)         

Windows && Linux && MacOS : [Qv2ray 下载](https://github.com/Qv2ray/Qv2ray/releases)      

支持 Trojan-Go 的插件 : [插件下载](https://github.com/Qv2ray/QvPlugin-NaiveProxy/releases)      

插件的用法 : [文档](https://qv2ray.net/plugins/usage.html) 


**移动版推荐配置如下 ：**		
<details>
<summary>对应配置1</summary>

```bash
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_domain",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "verify": true,
	"verify_hostname": true,
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "firefox"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    }
}
```
</details>

<details>
<summary>对应配置2</summary>

```bash
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_domain",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "verify": true,
	"verify_hostname": true,
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "firefox"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path", 
    "hostname": "your_domain.com"  
    }
}
```
</details>

<details>
<summary>对应配置3</summary>

```bash
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_domain",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "verify": true,
	"verify_hostname": true,
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "firefox"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path", 
    "hostname": "your_domain.com"  
    },
    "shadowsocks": {
    "enabled": true,
    "method": "AES-128-GCM",
    "password": "password1" 
  }
}
```
</details>
