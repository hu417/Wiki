免费注册SSL证书

# 使用acme.sh
1. 登录免费证书网站：https://freessl.cn/
2. 输入：bbse.top,*.bbse.top
3. 选择"亚洲诚信trustasia"
4. 点击"创建免费的ssl证书"
5. ACME域名配置
  5.1. 域名,输入"bbse.top,*.bbse.top",点击下一步
  5.2. DCV配置
```bash
bbse.top
  主机记录: _acme-challenge
  记录类型: CNAME
  记录值: tqi3rsvpr7nid5q564kg.dcv2.httpsauto.com
*.bbse.top
  主机记录: _acme-challenge
  记录类型: CNAME
  记录值: tqi3rsvpr7nid5q564kg.dcv2.httpsauto.com

```
  5.3 阿里云域名解析
    https://dns.console.aliyun.com/#/dns/domainList
    在相关域名后面点击解析设置，再点击添加记录，将DCV配置添加
  5.4 DCV配置-配置完成立即检查
  5.5 部署
    5.5.1 使用acme.sh部署
```bash
acme.sh --issue -d bbse.top -d *.bbse.top  --dns dns_dp --server https://acme.freessl.cn/v2/DV90/directory/xwjmrdo1xxj9c05tqx3e
```
6. 部署acme
 6.1 申请证书
```bash
// 下载acme.sh脚本：
git clone https://github.com/acmesh-official/acme.sh.git

// 进入文件夹
cd acme.sh

alias acme.sh=~/.acme.sh/acme.sh
acme.sh -v
acme.sh --upgrade

// 初次创建
acme.sh --issue -d bbse.top -d *.bbse.top  --dns dns_dp --server https://acme.freessl.cn/v2/DV90/directory/xwjmrdo1xxj9c05tqx3e

// 重新申请
acme.sh --renew --issue -d bbse.top -d *.bbse.top  --dns dns_dp --server https://acme.freessl.cn/v2/DV90/directory/xwjmrdo1xxj9c05tqx3e

// 命令执行成功后会在/root/.acme.sh/bbse.top_ecc/生成相关证书文件

```
 6.2 配置文件，复制文件，强制重启nginx
```bash
// bbse.top.key、fullchain.cer文件复制到/etc/nginx/ssl/fullchain.cer和/etc/nginx/ssl/bbse.top.key
acme.sh  --installcert  -d  *.bbse.top  --key-file   /etc/nginx/ssl/bbse.top.key --fullchain-file /etc/nginx/ssl/fullchain.cer --reloadcmd  "service nginx force-reload"

// nginx配置
// vim /etc/nginx/nginx.conf


server {
    listen 80;
    server_name www.bbse.top bbse.top;
    rewrite ^(.*)$ https://${server_name}$1 permanent; 
}

server {
        listen 443;
        server_name  www.bbse.top bbse.top;  # 域名
        ssl          on;                                      # 开启ssl
        ssl_certificate  /etc/nginx/ssl/fullchain.cer;        # 证书目录
        ssl_certificate_key  /etc/nginx/ssl/bbse.top.key;        # 证书目录
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;


        location / {
            root /home/xxx;
            index index.html index.htm;
        location /static {
            alias /home/xx/static; # 静态资源路径
            }
        }
    }

// 重启nginx
nginx -s reload

```
