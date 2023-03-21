---
layout: post
---

Ngrok is a service which gives you random url whenever you start a new tunnel.
[Clodflare tunnel](https://www.cloudflare.com/products/tunnel/) is a free
service that you can use as ngrok alternative.

You can create a temporary tunnel with random url
```
cloudflared tunnel --url http://localhost:3000
```
and it will be available under .trycloudflare.com like
<https://request-composer-pools-requirements.trycloudflare.com>

For long living tunnels which uses the same url you should use
<https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/>

```
cloudflared tunnel login
# when you authorize it will create ~/.cloudflared/cert.pem

# list all tunnels created on web
cloudflared tunnel list

# create tunnel and config.json file
cloudflared tunnel create mytunnel
# Tunnel credentials written to /home/dule/.cloudflared/asd....asd.json


cat > ~/.cloudflared/config.yaml << 'HERE_DOC'
url: http://localhost:8000
tunnel: mytunnel
credentials-file: ~/.cloudflared/asd....asd.json
HERE_DOC

# create DNS CNAME record to route to tunnel
cloudflared tunnel route dns mytunnel mytunnel.trk.in.rs

# start the tunnel
cloudflared tunnel run mytunnel

# check tunnel status
cloudflared tunnel info mytunnel
```

If you install as a service than you need to stop that service
```
systemctl status cloudflared
```
