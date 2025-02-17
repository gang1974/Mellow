# Mellow
一个基于规则进行透明代理的 V2Ray 客户端，支持 Windows 和 macOS。

## 下载

https://github.com/eycorsican/Mellow/releases

## 特性
Mellow 可对所有应用、所有请求进行透明代理，不需要为每个应用或程序单独设置代理，它所支持的特性可以概括为：

|     | [Mellow](https://github.com/eycorsican/Mellow) | [Surge Mac](https://nssurge.com/) | [SSTap](https://www.sockscap64.com/sstap-enjoy-gaming-enjoy-sstap/) | [Proxifier](https://www.proxifier.com/) | [Outline](https://getoutline.org/) |
|:---:|:------:|:---------:|:-----:|:---------:|:-------:|
| 透明代理 | ✅ | ✅ | ✅ | ✅ | ✅ |
| TCP 代理 | ✅ | ✅ | ✅ | ✅ | ✅ |
| UDP 代理 | ✅ | ✅ | ✅ | | ✅ |
| IP 规则 | ✅ | ✅ | ✅ | ✅ |
| 域名规则 | ✅ | ✅ | | ✅ |
| 应用进程规则 | ✅ | ✅ | | ✅ |
| 端口规则 | ✅ | ✅ | | ✅ |
| MitM | | ✅ | | |
| URL Rewrite | | ✅ | | |
| 多个代理出口 | ✅ | ✅ | | ✅ |
| 负载均衡 | ✅ | ✅ | | |
| DNS 分流 | ✅ | ✅ | | |
| SOCKS | ✅ | ✅ | ✅ | ✅ |
| Shadowsocks | ✅ | ✅ | ✅ | | ✅ |
| VMess | ✅ | | | |
| WebSocket, mKCP, QUIC, HTTP/2 传输| ✅ | | | |
| Windows 支持 | ✅ | | ✅ | ✅ | ✅ |
| macOS 支持 | ✅ | ✅ | | ✅ | ✅ |

其它 V2Ray 所支持的功能也都是支持的，上面并没有全部列出。

## 构建
```sh
# macOS
yarn && yarn distmac

# Windows
yarn && yarn distwin
```

## 扩展功能配置方式

### 自动选择最优线路
可根据代理请求的 RTT，自动选择负载均衡组中最优线路来转发请求。

```json
"routing": {
    "balancers": [
        {
            "tag": "server_lb",
            "selector": [
                "server_1",
                "server_2"
            ],
            "strategy": "latency",
            "totalMeasures": 2,
            "interval": 300,
            "delay": 1,
            "timeout": 6,
            "tolerance": 300,
            "probeTarget": "tls:www.google.com:443",
            "probeContent": "HEAD / HTTP/1.1\r\n\r\n"
        }
    ]
}
```


## 应用进程规则
支持 `*` 和 `?` 通配符匹配，匹配内容为进程名称。

在 Windows 上，进程名称通常为 `xxx.exe`，例如 `chrome.exe`，在 Mellow 的 `Statistics` 中可方便查看。

在 macOS 上也可以通过 Mellow 的 `Statistics` 查看，也可以通过 `ps` 命令查看进程。

```json
"routing": {
    "rules": [
        {
            "app": [
                "git*",
                "chrome.exe"
            ],
            "type": "field",
            "outboundTag": "proxy"
        }
    ]
}
```

## 配置示例
<details><summary>cfg.json</summary>
<p>

```json
{
    "log": {
        "loglevel": "info"
    },
    "dns": {
        "hosts": {
            "localhost": "127.0.0.1"
        },
        "servers": [
            {
                "address": "8.8.8.8",
                "port": 53
            },
            {
                "address": "223.5.5.5",
                "port": 53,
                "domains": [
                    "geosite:cn"
                ]
            }
        ]
    },
    "outbounds": [
        {
            "protocol": "vmess",
            "settings": {},
            "tag": "economic_vps_1"
        },
        {
            "protocol": "vmess",
            "settings": {},
            "tag": "economic_vps_2"
        },
        {
            "protocol": "vmess",
            "settings": {},
            "tag": "bittorrent_vps_1"
        },
        {
            "protocol": "vmess",
            "settings": {},
            "tag": "expensive_vps_1"
        },
        {
            "protocol": "freedom",
            "settings": {},
            "tag": "direct"
        },
        {
            "settings": {},
            "protocol": "blackhole",
            "tag": "block"
        }
    ],
    "policy": {
        "levels": {
            "0": {
                "connIdle": 300,
                "downlinkOnly": 0,
                "uplinkOnly": 0,
                "handshake": 4
            }
        }
    },
    "routing": {
        "domainStrategy": "IPIfNonMatch",
        "balancers": [
            {
                "tag": "limited",
                "selector": [
                    "expensive_vps_1",
                    "economic_vps_1"
                ],
                "strategy": "latency",
                "totalMeasures": 2,
                "interval": 300,
                "delay": 1,
                "timeout": 6,
                "tolerance": 300,
                "probeTarget": "tls:www.google.com:443",
                "probeContent": "HEAD / HTTP/1.1\r\n\r\n"
            },
            {
                "tag": "bt",
                "selector": [
                    "bittorrent_vps_1"
                ]
            },
            {
                "tag": "nolimit",
                "selector": [
                    "economic_vps_1",
                    "economic_vps_2"
                ],
                "strategy": "latency",
                "totalMeasures": 2,
                "interval": 120
            }
        ],
        "rules": [
            {
                "domain": [
                    "domain:doubleclick.net"
                ],
                "type": "field",
                "outboundTag": "block"
            },
            {
                "type": "field",
                "ip": [
                    "1.1.1.1",
                    "9.9.9.9",
                    "8.8.8.8",
                    "8.8.4.4"
                ],
                "balancerTag": "limited"
            },
            {
                "app": [
                    "ssh",
                    "git",
                    "brew",
                    "Dropbox"
                ],
                "type": "field",
                "balancerTag": "limited"
            },
            {
                "type": "field",
                "domain": [
                    "geosite:cn"
                ],
                "outboundTag": "direct"
            },
            {
                "type": "field",
                "ip": [
                    "geoip:cn",
                    "geoip:private"
                ],
                "outboundTag": "direct"
            },
            {
                "type": "field",
                "app": [
                    "aria2c"
                ],
                "balancerTag": "bt"
            },
            {
                "type": "field",
                "domain": [
                    "googlevideo",
                    "dl.google.com",
                    "ytimg"
                ],
                "balancerTag": "nolimit"
            },
            {
                "type": "field",
                "domain": [
                    "domain:youtube.com",
                    "android",
                    "google",
                    "nyaa",
                    "git"
                ],
                "balancerTag": "limited"
            },
            {
                "ip": [
                    "0.0.0.0/0",
                    "::/0"
                ],
                "type": "field",
                "balancerTag": "nolimit"
            }
        ]
    }
}
```

</p>
</details>
