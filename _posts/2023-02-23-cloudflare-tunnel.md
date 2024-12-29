---
layout: post
---

Ngrok is a service which gives you random url whenever you start a new tunnel.
[Clodflare tunnel](https://www.cloudflare.com/products/tunnel/) is a free
service that you can use as ngrok alternative.

Install https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/
```
# macOS
brew install cloudflared
# ubuntu
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb && dpkg -i cloudflared-linux-amd64.deb
```
and login
```
cloudflared tunnel login
# this will create and download ~/.cloudflared/cert.pem
# you can read pem file in future so you know on which account you have access
openssl x509 -in cert.pem -noout -ext subjectAltName
X509v3 Subject Alternative Name: 
    DNS:*.my-domain.com, DNS:my-domain.com
```

You can create a temporary tunnel with random url
```
cloudflared tunnel --url http://localhost:3000
```
and it will be available under .trycloudflare.com like
<https://request-composer-pools-requirements.trycloudflare.com>

For long living tunnels which uses the same url you should use local
<https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/>
or remotelly managed tunnels

## Locally managed tunnels

```
# list all tunnels created previously using cloudflared cli on web
cloudflared tunnel list

# create tunnel on cloudflare site and local config.json file
cloudflared tunnel create mytunnel
# Tunnel credentials written to /home/dule/.cloudflared/asd....asd.json

# create config file that uses this credentials and tunnel name
cat > ~/.cloudflared/config.yaml << 'HERE_DOC'
url: http://localhost:3000
tunnel: mytunnel
# home shortcut ~/.cloudflared/ will not work
credentials-file: /home/dule/.cloudflared/asd....asd.json
HERE_DOC

# create DNS CNAME record for mytunnel with value asd...asd.cfargotunnel.com
cloudflared tunnel route dns mytunnel mytunnel.trk.in.rs

# start the tunnel, default is to read cert from .cloudflared/cert.pem and .cloudflared/config.yml
cloudflared tunnel run mytunnel
# but you can define using params for credentials json and config yml
cloudflared --credentials-file doc/cloudflared/f77e7809-5f2b-497e-9086-f13c1ad8b222.json --config doc/cloudflared/config.yml tunnel run mytunnel

# check tunnel status
cloudflared tunnel info mytunnel
```

For multiple domains you just need to add same CNAME records to other
subdomains.

To block by IP address, go to Security > WAF and create rule with "IP Source
Address" "is not in" {my ip address} And "Hostname" "equals"
"mytunnel.trk.in.rs" than "Block".

If you install as a service than you need to stop that service
```
systemctl status cloudflared
```

If you need another account you can use `origincert` param
https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/configure-tunnels/tunnel-run-parameters/#origincert
```
cloudflared --origincert doc/cloudflared/cert.pem tunnel list
cloudflared --origincert doc/cloudflared/cert.pem tunnel create
# this will create json in the same folder as pem doc/cloudflared/asd.json
```
and `config` param to configure port
https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/configure-tunnels/tunnel-run-parameters/#config
```
cloudflared --origincert doc/cloudflared/cert.pem --config doc/cloudflared/config.yml tunnel run mytunnel
```

To debug use
```
cloudflared --loglevel debug tunnel run
```

## Remotelly managed tunnels

After install
```
sudo cloudflared service install eyJh....

Installing cloudflared client as a system launch daemon. cloudflared client
will run at boot
2024-04-12T05:14:09Z INF Outputs are logged to
/Library/Logs/com.cloudflare.cloudflared.err.log
/Library/Logs/com.cloudflare.cloudflared.out.log
```

#  Cloudflare DNS

If you need direct ssh port 22 access than you need to disable Proxied and use
DNS only proxy status.

Other solution is to tunnel ssh as http service

```
# config.yml
ingress:
  - hostname: ssh-my-app.trk.in.rs
    service: ssh://localhost:22
```
and use cloudflared access

```
Host ssh-my-app.trk.in.rs
  ProxyCommand $(brew --prefix)/bin/cloudflared access ssh --hostname %h
```

Similarly for mysql connection you can use tcp service
```
# config.yml
ingress:
  - hostname: mysql-my-app.trk.in.rs
    service: tcp://localhost:3306
```
and from remote you can connect using
```
cloudflared access tcp --hostname mysql-my-app.trk.in.rs --url localhost:3306

```
