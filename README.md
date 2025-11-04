# sing-box 配置
适用于 sing-box 1.12.x 版本的客户端配置。

```json
{
    "log": {
        "level": "info"
    },
    "dns": {
        "servers": [
            {
                "type": "https",
                "tag": "google",
                "domain_resolver": "dns-resolver",
                "server": "dns.google"
            },
            {
                "type": "https",
                "tag": "alidns",
                "domain_resolver": "dns-resolver",
                "server": "dns.alidns.com"
            },
            {
                "type": "fakeip",
                "tag": "fakeip",
                "inet4_range": "198.18.0.0/15",
                "inet6_range": "fc00::/18"
            },
            {
                "type": "hosts",
                "tag": "dns-resolver",
                "predefined": {
                    "dns.alidns.com": [
                        "223.5.5.5",
                        "223.6.6.6",
                        "2400:3200::1",
                        "2400:3200:baba::1"
                    ],
                    "dns.google": [
                        "8.8.8.8",
                        "8.8.4.4",
                        "2001:4860:4860::8888",
                        "2001:4860:4860::8844"
                    ]
                }
            }
        ],
        "rules": [
            {
                "clash_mode": "全局",
                "server": "fakeip"
            },
            {
                "clash_mode": "直连",
                "server": "alidns"
            },
            {
                "query_type": [
                    "HTTPS",
                    "PTR",
                    "SVCB"
                ],
                "action": "reject"
            },
            {
                "query_type": [
                    "A",
                    "AAAA"
                ],
                "server": "fakeip",
                "rewrite_ttl": 1
            },
            {
                "rule_set": "geosite-geolocation-cn",
                "server": "alidns"
            },
            {
                "type": "logical",
                "mode": "and",
                "rules": [
                    {
                        "rule_set": "geosite-geolocation-!cn",
                        "invert": true
                    },
                    {
                        "rule_set": "geoip-cn"
                    }
                ],
                "server": "google",
                "client_subnet": "1.0.1.0/24"
            }
        ],
        "strategy": "prefer_ipv6",
        "independent_cache": true
    },
    "inbounds": [
        {
            "type": "tun",
            "tag": "tun-in",
            "address": [
                "172.19.0.1/30",
                "fdfe:dcba:9876::1/126"
            ],
            "auto_route": true,
            "strict_route": true,
            "route_exclude_address": [
                "10.0.0.0/8",
                "100.64.0.0/10",
                "169.254.0.0/16",
                "172.16.0.0/12",
                "192.0.0.0/24",
                "192.168.0.0/16",
                "224.0.0.0/4",
                "240.0.0.0/4",
                "fc00::/7",
                "fe80::/10",
                "ff01::/16",
                "ff02::/16",
                "ff03::/16",
                "ff04::/16",
                "ff05::/16"
            ],
            "stack": "gvisor",
            "platform": {
                "http_proxy": {
                    "enabled": true,
                    "server": "127.0.0.1",
                    "server_port": 2025
                }
            }
        },
        {
            "type": "mixed",
            "tag": "mixed-in",
            "listen": "127.0.0.1",
            "listen_port": 2025
        }
    ],
    "outbounds": [
        {
            "type": "selector",
            "tag": "手动选择",
            "outbounds": [
                "Node-1",
                "Node-2",
                "Node-3",
                "自动选择"
            ],
            "default": "自动选择",
            "interrupt_exist_connections": true
        },
        {
            "type": "urltest",
            "tag": "自动选择",
            "outbounds": [
                "Node-1",
                "Node-2",
                "Node-3"
            ],
            "interrupt_exist_connections": true
        },
        {
            "type": "direct",
            "tag": "direct-out"
        },
        {
            "type": "vless",
            "tag": "Node-1",
            "server": "node1.example.com",
            "server_port": 20251,
            "uuid": "f503b9b1-3b33-4b49-9565-6b4b49562018",
            "flow": "xtls-rprx-vision",
            "tls": {
                "enabled": true,
                "server_name": "example.com",
                "utls": {
                    "enabled": true
                },
                "reality": {
                    "enabled": true,
                    "public_key": "-SAs210sMfT1JACvmySGsAykjAdiDTCXWd92847wt3A",
                    "short_id": "248102e17320fde8"
                }
            }
        },
        {
            "type": "hysteria2",
            "tag": "Node-2",
            "server": "node2.example.com",
            "server_port": 20252,
            "up_mbps": 1000,
            "down_mbps": 1000,
            "password": "3h7&Fh2(@jkaHQd1",
            "tls": {
                "enabled": true,
                "server_name": "bing.com",
                "insecure": true,
                "alpn": "h3"
            }
        },
        {
            "type": "anytls",
            "tag": "Node-3",
            "server": "node3.example.com",
            "server_port": 20253,
            "password": "H892yb&29KdOJE_2a",
            "tls": {
                "enabled": true,
                "server_name": "node3.example.com"
            }
        }
    ],
    "route": {
        "rules": [
            {
                "action": "sniff"
            },
            {
                "type": "logical",
                "mode": "or",
                "rules": [
                    {
                        "protocol": "dns"
                    },
                    {
                        "port": 53
                    }
                ],
                "action": "hijack-dns"
            },
            {
                "ip_is_private": true,
                "outbound": "direct"
            },
            {
                "type": "logical",
                "mode": "or",
                "rules": [
                    {
                        "port": 853
                    },
                    {
                        "network": "udp",
                        "port": 443
                    },
                    {
                        "protocol":[
                            "quic",
                            "stun"
                        ]
                    },
                    {
                        "rule_set": "ads"
                    }
                ],
                "action": "reject"
            },
            {
                "clash_mode": "全局",
                "outbound": "手动选择"
            },
            {
                "clash_mode": "直连",
                "outbound": "direct-out"
            },
            {
                "rule_set": "geosite-geolocation-cn",
                "outbound": "direct-out"
            },
            {
                "type": "logical",
                "mode": "and",
                "rules": [
                    {
                        "rule_set": "geoip-cn"
                    },
                    {
                        "rule_set": "geosite-geolocation-!cn",
                        "invert": true
                    }
                ],
                "outbound": "direct-out"
            }
        ],
        "rule_set": [
            {
                "type": "remote",
                "tag": "ads",
                "url": "https://raw.githubusercontent.com/Moiudev/rule-set/main/ads.srs",
                "update_interval": "1h"
            },
            {
                "type": "remote",
                "tag": "geoip-cn",
                "url": "https://raw.githubusercontent.com/Moiudev/rule-set/main/geoip-cn.srs"
            },
            {
                "type": "remote",
                "tag": "geosite-geolocation-cn",
                "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/geolocation-cn.srs"
            },
            {
                "type": "remote",
                "tag": "geosite-geolocation-!cn",
                "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/geolocation-!cn.srs"
            }
        ],
        "final": "手动选择",
        "auto_detect_interface": true,
        "default_domain_resolver": "alidns"
    },
    "experimental": {
        "cache_file": {
            "enabled": true,
            "store_fakeip": true,
            "store_rdrc": true
        },
        "clash_api": {
            "default_mode": "规则"
        }
    }
}
```
