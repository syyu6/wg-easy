# WireGuard Easy

[![Build & Publish Docker Image to Docker Hub](https://github.com/wg-easy/wg-easy/actions/workflows/deploy.yml/badge.svg?branch=production)](https://github.com/wg-easy/wg-easy/actions/workflows/deploy.yml)
[![Lint](https://github.com/wg-easy/wg-easy/actions/workflows/lint.yml/badge.svg?branch=master)](https://github.com/wg-easy/wg-easy/actions/workflows/lint.yml)
![Docker](https://img.shields.io/docker/pulls/weejewel/wg-easy.svg)
[![Sponsor](https://img.shields.io/github/sponsors/weejewel)](https://github.com/sponsors/WeeJeWel)
![GitHub Stars](https://img.shields.io/github/stars/wg-easy/wg-easy)

You have found the easiest way to install & manage WireGuard on any Linux host!

<p align="center">
  <img src="./assets/screenshot.png" width="802" />
</p>

## 更新日志

### 2024-03-27

 - 合并上游分支代码

### 2024-03-07

 - 新增 `WG_SERVER_ALLOWED_IPS`, 定义允许路由转发的网段
 - 新增 `WG_SERVER_ROUTE_IP`, 定义转发指向的路由IP
  ```
  WG_SERVER_ALLOWED_IPS = 192.168.2.0/24
  WG_SERVER_ROUTE_IP = 192.168.2.2
  ```

### 使用场景

公网vps部署的wg-endpoint，通过wg接入，可直连本地NAS同网段的家庭网络。

client.name(客户端名称) 以 `NAS` / `LAN-` 开头，将自动添加路由规则和AllowedIPs

本地NAS-wg配置增加路由规则

```
PostUp = iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j SNAT --to-source 192.168.2.2(NAS-IP)
PostDown = iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -j SNAT --to-source 192.168.2.2(NAS-IP)
```

vps上设置允许 AllowedIPs

```
AllowedIPs = 10.8.0.0/24, 192.168.2.0/24
```

并且vps和nas上都要开启IP转发

```
sudo sysctl -w net.ipv4.ip_forward=1
```

### 示例: docker-run启动

```
docker run -d \
  --name=wg-easy \
  -e WG_HOST=xxxxxxx \
  -e PASSWORD=EWQ!@# \
  -e WG_PERSISTENT_KEEPALIVE=25 \
  -e LANG=chs \
  -e WG_DEFAULT_DNS=8.8.8.8 \
  -e WG_DEFAULT_ADDRESS=10.8.0.x \
  -e WG_ALLOWED_IPS=10.8.0.0/24,192.168.2.0/24 \ # NAS网段
  -e WG_SERVER_ALLOWED_IPS=192.168.2.0/24 \ # NAS网段
  -e WG_SERVER_ROUTE_IP=192.168.2.2 \ # NAS-IP
  -v /root/wg-easy:/etc/wireguard \
  -p 51820:51820/udp \
  -p 51821:51821/tcp \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  --restart unless-stopped \
  ghcr.io/syyu6/wg-easy:11.2
  
```


## Features

* All-in-one: WireGuard + Web UI.
* Easy installation, simple to use.
* List, create, edit, delete, enable & disable clients.
* Show a client's QR code.
* Download a client's configuration file.
* Statistics for which clients are connected.
* Tx/Rx charts for each connected client.
* Gravatar support.
* Automatic Light / Dark Mode
* Multilanguage Support
* UI_TRAFFIC_STATS (default off)

## Requirements

* A host with a kernel that supports WireGuard (all modern kernels).
* A host with Docker installed.

## Installation

### 1. Install Docker

If you haven't installed Docker yet, install it by running:

```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $(whoami)
exit
```

And log in again.

### 2. Run WireGuard Easy

To automatically install & run wg-easy, simply run:

```
  docker run -d \  --name=wg-easy \
  -e LANG=de \
  -e WG_HOST=<b>🚨YOUR_SERVER_IP</b> \
  -e PASSWORD=<b>🚨YOUR_ADMIN_PASSWORD</b> \
  -v ~/.wg-easy:/etc/wireguard \
  -p 51820:51820/udp \
  -p 51821:51821/tcp \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  --restart unless-stopped \
  ghcr.io/wg-easy/wg-easy
```

> 💡 Replace `YOUR_SERVER_IP` with your WAN IP, or a Dynamic DNS hostname.
>
> 💡 Replace `YOUR_ADMIN_PASSWORD` with a password to log in on the Web UI.

The Web UI will now be available on `http://0.0.0.0:51821`.

> 💡 Your configuration files will be saved in `~/.wg-easy`

WireGuard Easy can be launched with Docker Compose as well - just download
[`docker-compose.yml`](docker-compose.yml), make necessary adjustments and
execute `docker compose up --detach`.

### 3. Sponsor

Are you enjoying this project? [Buy Emile a beer!](https://github.com/sponsors/WeeJeWel) 🍻

## Options

These options can be configured by setting environment variables using `-e KEY="VALUE"` in the `docker run` command.

| Env | Default | Example | Description |
| - | - | - | - |
| `PORT` | `51821` | `6789` | TCP port for Web UI. |
| `WEBUI_HOST` | `0.0.0.0` | `localhost` | IP address web UI binds to. |
| `PASSWORD` | - | `foobar123` | When set, requires a password when logging in to the Web UI. |
| `WG_HOST` | - | `vpn.myserver.com` | The public hostname of your VPN server. |
| `WG_DEVICE` | `eth0` | `ens6f0` | Ethernet device the wireguard traffic should be forwarded through. |
| `WG_PORT` | `51820` | `12345` | The public UDP port of your VPN server. WireGuard will always listen on 51820 inside the Docker container. |
| `WG_MTU` | `null` | `1420` | The MTU the clients will use. Server uses default WG MTU. |
| `WG_PERSISTENT_KEEPALIVE` | `0` | `25` | Value in seconds to keep the "connection" open. If this value is 0, then connections won't be kept alive. |
| `WG_DEFAULT_ADDRESS` | `10.8.0.0` | `10.6.0.0` | Clients IP address range. |
| `WG_DEFAULT_ADDRESS_RANGE` | `24` | `32` | Value to define CIDR Range. If not defined fallback to `24`
| `WG_DEFAULT_DNS` | `1.1.1.1` | `8.8.8.8, 8.8.4.4` | DNS server clients will use. If set to blank value, clients will not use any DNS. |
| `WG_ALLOWED_IPS` | `0.0.0.0/0, ::/0` | `192.168.15.0/24, 10.0.1.0/24` | Allowed IPs clients will use. |
| `WG_PRE_UP` | `...` | - | See [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L19) for the default value. |
| `WG_POST_UP` | `...` | `iptables ...` | See [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L20) for the default value. |
| `WG_PRE_DOWN` | `...` | - | See [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L27) for the default value. |
| `WG_POST_DOWN` | `...` | `iptables ...` | See [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L28) for the default value. |
| `LANG` | `en` | `de` | Web UI language (Supports: en, ua, ru, tr, no, pl, fr, de, ca, es, ko, vi, nl, is, pt, chs, cht, it, th, hi). |
| `UI_TRAFFIC_STATS` | `false` | `true` | Enable detailed RX / TX client stats in Web UI |

> If you change `WG_PORT`, make sure to also change the exposed port.

## Updating

To update to the latest version, simply run:

```bash
docker stop wg-easy
docker rm wg-easy
docker pull ghcr.io/wg-easy/wg-easy
```

And then run the `docker run -d \ ...` command above again.

With Docker Compose WireGuard Easy can be updated with a single command:
`docker compose up --detach --pull always` (if an image tag is specified in the
Compose file and it is not `latest`, make sure that it is changed to the desired
one; by default it is omitted and
[defaults to `latest`](https://docs.docker.com/engine/reference/run/#image-references)). \
The WireGuared Easy container will be automatically recreated if a newer image
was pulled.

## Common Use Cases

* [Using WireGuard-Easy with Pi-Hole](https://github.com/wg-easy/wg-easy/wiki/Using-WireGuard-Easy-with-Pi-Hole)
* [Using WireGuard-Easy with nginx/SSL](https://github.com/wg-easy/wg-easy/wiki/Using-WireGuard-Easy-with-nginx-SSL)

For less common or specific edge-case scenarios, please refer to the detailed information provided in the [Wiki](https://github.com/wg-easy/wg-easy/wiki).
