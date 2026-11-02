# loon-sub-config

把 `https://flink.dagrate.eu.org/7a2df4fc2672`（Clash / mihomo YAML：114 个 Shadowsocks 节点 + 9 个策略组 + 原配置 35 万条规则）
转换成 Loon 能直接读取的格式。

原订阅是 Clash YAML。Loon 不支持 YAML 订阅，不论客户端怎么魔改都一样 —— FlClash 能读是因为它内核就是 mihomo。
所以这里用转换解决，Loon 侧一行不用改。

## 文件

| 文件 | 大小 | 规则数 | 用途 |
|---|---|---|---|
| `loon-nodes.conf` | 15 KB | — | 纯节点订阅（Sub-Store 官方转换器产出）。Loon「订阅」里添加即用，规则走 Loon 自己的配置 |
| `loon-mini.conf` | 200 KB | 3,149 | 完整配置：9 个策略组 + 常用分流（局域网 / 苹果 / 微软 / Telegram / OpenAI / Claude / Gemini / PikPak / Steam / 流媒体）+ GEOIP CN 直连 |
| `loon-no-ads.conf` | 5.2 MB | 148,945 | 完整配置：`loon-mini` + 全部国内域名 / 代理域名列表，不含广告拦截 |
| `loon-fixed.conf` | 13.3 MB | 351,544 | 完整配置：与原 Clash 订阅逐条等价（含广告拦截） |

三个 `.conf` 都带 `#!MANAGED-CONFIG <原订阅地址> interval=86400`，Loon 里会显示为托管订阅，
以后原订阅更新只需在 Loon 里点刷新（节点会更新，规则保持转换后的版本）。

## Loon 导入

节点订阅（最省事，先试这个）：
```
https://raw.githubusercontent.com/spaoye/loon-sub-config/main/loon-nodes.conf
```

完整配置（自带策略组和分流）：
```
https://raw.githubusercontent.com/spaoye/loon-sub-config/main/loon-mini.conf
```

`raw.githubusercontent.com` 直连不通时换 jsDelivr（同样内容，CDN）：
```
https://cdn.jsdelivr.net/gh/spaoye/loon-sub-config@main/loon-mini.conf
```

## 转换细节

- `ss` → `shadowsocks`，`chacha20-ietf-poly1305`，`udp=true` 原样保留
- 9 个策略组全部保留：`select` 类型，成员顺序与原配置一致
- `RULE-SET` 展开为内联规则，**顺序与 Clash 的 `rules:` 严格一致**（顺序变了分流结果就会变）
  - `behavior: ipcidr` 的集合（ChinaCIDR）→ `IP-CIDR,x/y,no-resolve`
  - `behavior: domain` 的集合（BanAD / ChinaClassical / ProxyClassical）→ `DOMAIN-SUFFIX,x`
  - `behavior: classical`（blackmatrix7 系列）→ 逐条转换
- `IP-CIDR,...,no-resolve` 统一补 `no-resolve`
- 等值去重（保留首次出现位置）
- `MATCH` → `FINAL`，`GEOIP,CN,DIRECT` 保留
- 丢弃 Loon 不支持的：`PROCESS-NAME` 以外的进程规则、`IP-ASN` 保留、`USER-AGENT` 保留

## 自己复现

- 整份配置：`/var/minis/workspace/loonfix/build_loon.py`（`MANAGED=<订阅地址> python3 build_loon.py`）
- 纯节点订阅：Sub-Store 转换引擎 `sub-store-convert`（npm），`loadProducer('Loon')` 直接产出 Loon 节点格式

## 想要自动化（Sub-Store 自建）

不想每次手动转，可以在自己的服务器上跑 Sub-Store：添加原 Clash 订阅作为单条订阅，
新建组合订阅，输出选 **Loon**，得到一个固定 URL 直接塞进 Loon。
```
docker run -d --name sub-store -p 3001:3000 -v $PWD/sub-store:/opt/app/data \
  xream/sub-store:latest
```
或纯 node：`node sub-store.min.js`，浏览器开 `http://IP:3001` 添加订阅。