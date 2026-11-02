# loon-sub-config

把 `https://flink.dagrate.eu.org/7a2df4fc2672`（Clash / mihomo YAML，114 个 ss 节点 + 9 个策略组 + 35 万条规则）
转换成 Loon 能直接吃的格式。

原订阅是 Clash YAML，Loon 完全不认 YAML，所以无论客户端怎么魔改都一样 —— 必须转换。

## 文件

| 文件 | 大小 | 用途 |
|---|---|---|
| `loon-nodes.conf` | 15 KB | **纯节点订阅**（Sub-Store 官方转换器产出），Loon「订阅」直接添加，最省事 |
| `loon-light.conf` | 560 KB | 完整配置：9 个策略组 + 1.3 万条内联规则（不含广告拦截） |
| `loon-lite.conf` | 880 KB | 完整配置：含广告拦截，3 个大规则集走外链 |
| `loon-fixed.conf` | 13 MB | 完整配置：35 万条规则全部内联，规则顺序与原 Clash 配置逐条一致 |

三个 `.conf` 都带 `#!MANAGED-CONFIG <原订阅地址> interval=86400`，Loon 里会自动显示为托管订阅。

## Loon 导入

纯节点（推荐先试这个）：
```
https://raw.githubusercontent.com/spaoye/loon-sub-config/main/loon-nodes.conf
```

完整配置（含分流规则）：
```
https://raw.githubusercontent.com/spaoye/loon-sub-config/main/loon-lite.conf
```

国内直连不通 `raw.githubusercontent.com` 时，把域名换成 jsDelivr：
```
https://cdn.jsdelivr.net/gh/spaoye/loon-sub-config@main/loon-lite.conf
```

## 转换细节

- 协议：`ss` → Loon `Shadowsocks`，`chacha20-ietf-poly1305`，`udp=true` 保留
- 策略组：`select` 组全部保留，成员按原顺序
- 规则：`RULE-SET` 展开成内联规则，顺序与 Clash 的 `rules:` 完全一致（顺序错会导致分流结果不同）
- `IP-CIDR,...,no-resolve` 统一补 `no-resolve`
- 结尾 `MATCH` → `FINAL`，原 `GEOIP,CN,DIRECT` 保留
- 广告拦截（BanAD / BanAdobe）用 Clash 语法，转成 Loon 的 `DOMAIN` / `DOMAIN-SUFFIX` 后等价

## 重新生成

见 `/var/minis/workspace/loonfix/`：`build_both.py`（整份配置）与 Sub-Store 引擎脚本（节点订阅）。