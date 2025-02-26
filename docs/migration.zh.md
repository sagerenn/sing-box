---
icon: material/arrange-bring-forward
---

## 1.11.0

### 迁移旧的特殊出站到规则动作

旧的特殊出站已被弃用，且可以被规则动作替代。

!!! info "参考"

    [规则动作](/zh/configuration/route/rule_action/) /
    [Block](/zh/configuration/outbound/block/) / 
    [DNS](/zh/configuration/outbound/dns)

=== "Block"

    === ":material-card-remove: 弃用的"
    
        ```json
        {
          "outbounds": [
            {
              "type": "block",
              "tag": "block"
            }
          ],
          "route": {
            "rules": [
              {
                ...,
                
                "outbound": "block"
              }
            ]
          }
        }
        ```

    === ":material-card-multiple: 新的"
    
        ```json
        {
          "route": {
            "rules": [
              {
                ...,
                
                "action": "reject"
              }
            ]
          }
        }
        ```

=== "DNS"

    === ":material-card-remove: 弃用的"
    
        ```json
        {
          "inbound": [
            {
              ...,
              
              "sniff": true
            }
          ],
          "outbounds": [
            {
              "tag": "dns",
              "type": "dns"
            }
          ],
          "route": {
            "rules": [
              {
                "protocol": "dns",
                "outbound": "dns"
              }
            ]
          }
        }
        ```
    
    === ":material-card-multiple: 新的"
    
        ```json
        {
          "route": {
            "rules": [
              {
                "action": "sniff"
              },
              {
                "protocol": "dns",
                "action": "hijack-dns"
              }
            ]
          }
        }
        ```

### 迁移旧的入站字段到规则动作

入站选项已被弃用，且可以被规则动作替代。

!!! info "参考"

    [监听字段](/zh/configuration/shared/listen/) /
    [规则](/zh/configuration/route/rule/) /
    [规则动作](/zh/configuration/route/rule_action/) /
    [DNS 规则](/zh/configuration/dns/rule/) /
    [DNS 规则动作](/zh/configuration/dns/rule_action/)

=== ":material-card-remove: 弃用的"

    ```json
    {
      "inbounds": [
        {
          "type": "mixed",
          "sniff": true,
          "sniff_timeout": "1s",
          "domain_strategy": "prefer_ipv4"
        }
      ]
    }
    ```

=== ":material-card-multiple: New"

    ```json
    {
      "inbounds": [
        {
          "type": "mixed",
          "tag": "in"
        }
      ],
      "route": {
        "rules": [
          {
            "inbound": "in",
            "action": "resolve",
            "strategy": "prefer_ipv4"
          },
          {
            "inbound": "in",
            "action": "sniff",
            "timeout": "1s"
          }
        ]
      }
    }
    ```

## 1.10.0

### TUN 地址字段已合并

`inet4_address` 和 `inet6_address` 已合并为 `address`，
`inet4_route_address` 和 `inet6_route_address` 已合并为 `route_address`，
`inet4_route_exclude_address` 和 `inet6_route_exclude_address` 已合并为 `route_exclude_address`。

旧字段已废弃，且将在 sing-box 1.11.0 中移除。

!!! info "参考"

    [TUN](/zh/configuration/inbound/tun/)

=== ":material-card-remove: 弃用的"

    ```json
    {
      "inbounds": [
        {
          "type": "tun",
          "inet4_address": "172.19.0.1/30",
          "inet6_address": "fdfe:dcba:9876::1/126",
          "inet4_route_address": [
            "0.0.0.0/1",
            "128.0.0.0/1"
          ],
          "inet6_route_address": [
            "::/1",
            "8000::/1"
          ],
          "inet4_route_exclude_address": [
            "192.168.0.0/16"
          ],
          "inet6_route_exclude_address": [
            "fc00::/7"
          ]
        }
      ]
    }
    ```

=== ":material-card-multiple: 新的"

    ```json
    {
      "inbounds": [
        {
          "type": "tun",
          "address": [
            "172.19.0.1/30",
            "fdfe:dcba:9876::1/126"
          ],
          "route_address": [
            "0.0.0.0/1",
            "128.0.0.0/1",
            "::/1",
            "8000::/1"
          ],
          "route_exclude_address": [
            "192.168.0.0/16",
            "fc00::/7"
          ]
        }
      ]
    }
    ```

## 1.9.5

### Apple 平台客户端的 Bundle Identifier 更新

由于我们旧的苹果开发者账户存在问题，我们只能通过更新 Bundle Identifiers
来重新上架 sing-box 应用， 这意味着数据不会自动继承。

对于 iOS，您需要自行备份旧的数据（如果您仍然可以访问）；  
对于 Apple tvOS，您需要从 iPhone 或 iPad 重新导入配置或者手动创建；  
对于 macOS，您可以使用以下命令迁移数据文件夹：

```bash
cd ~/Library/Group\ Containers && \ 
  mv group.io.nekohasekai.sfa group.io.nekohasekai.sfavt
```

## 1.9.0

### `domain_suffix` 行为更新

由于历史原因，sing-box 的 `domain_suffix` 规则匹配字面前缀，而不与其他项目相同。

sing-box 1.9.0 修改了 `domain_suffix` 的行为：如果规则值以 `.` 为前缀则行为不变，否则改为匹配 `(domain|.+\.domain)`。

### 对 Windows 上 `process_path` 格式的更新

sing-box 的 `process_path` 规则继承自Clash，
原始代码使用本地系统的路径格式（例如 `\Device\HarddiskVolume1\folder\program.exe`），
但是当设备有多个硬盘时，该 HarddiskVolume 系列号并不稳定。

sing-box 1.9.0 使 QueryFullProcessImageNameW 输出 Win32 路径（如 `C:\folder\program.exe`），
这将会破坏现有的 Windows `process_path` 用例。

## 1.8.0

### :material-close-box: 将缓存文件从 Clash API 迁移到独立选项

!!! info "参考"

    [Clash API](/zh/configuration/experimental/clash-api/) / 
    [Cache File](/zh/configuration/experimental/cache-file/)

=== ":material-card-remove: 弃用的"

    ```json
    {
      "experimental": {
        "clash_api": {
          "cache_file": "cache.db", // 默认值
          "cahce_id": "my_profile2",
          "store_mode": true,
          "store_selected": true,
          "store_fakeip": true
        }
      }
    }
    ```

=== ":material-card-multiple: 新的"

    ```json
    {
      "experimental"  : {
        "cache_file": {
          "enabled": true,
          "path": "cache.db", // 默认值
          "cache_id": "my_profile2",
          "store_fakeip": true
        }
      }
    }
    ```

### :material-checkbox-intermediate: 迁移 GeoIP 到规则集

!!! info "参考"

    [GeoIP](/zh/configuration/route/geoip/) / 
    [路由](/zh/configuration/route/) / 
    [路由规则](/zh/configuration/route/rule/) / 
    [DNS 规则](/zh/configuration/dns/rule/) / 
    [规则集](/zh/configuration/rule-set/)

!!! tip

    `sing-box geoip` 命令可以帮助您将自定义 GeoIP 转换为规则集。

=== ":material-card-remove: 弃用的"

    ```json
    {
      "route": {
        "rules": [
          {
            "geoip": "private",
            "outbound": "direct"
          },
          {
            "geoip": "cn",
            "outbound": "direct"
          },
          {
            "source_geoip": "cn",
            "outbound": "block"
          }
        ],
        "geoip": {
          "download_detour": "proxy"
        }
      }
    }
    ```

=== ":material-card-multiple: 新的"

    ```json
    {
      "route": {
        "rules": [
          {
            "ip_is_private": true,
            "outbound": "direct"
          },
          {
            "rule_set": "geoip-cn",
            "outbound": "direct"
          },
          {
            "rule_set": "geoip-us",
            "rule_set_ipcidr_match_source": true,
            "outbound": "block"
          }
        ],
        "rule_set": [
          {
            "tag": "geoip-cn",
            "type": "remote",
            "format": "binary",
            "url": "https://raw.githubusercontent.com/SagerNet/sing-geoip/rule-set/geoip-cn.srs",
            "download_detour": "proxy"
          },
          {
            "tag": "geoip-us",
            "type": "remote",
            "format": "binary",
            "url": "https://raw.githubusercontent.com/SagerNet/sing-geoip/rule-set/geoip-us.srs",
            "download_detour": "proxy"
          }
        ]
      },
      "experimental": {
        "cache_file": {
          "enabled": true // required to save rule-set cache
        }
      }
    }
    ```

### :material-checkbox-intermediate: 迁移 Geosite 到规则集

!!! info "参考"

    [Geosite](/zh/configuration/route/geosite/) / 
    [路由](/zh/configuration/route/) / 
    [路由规则](/zh/configuration/route/rule/) / 
    [DNS 规则](/zh/configuration/dns/rule/) / 
    [规则集](/zh/configuration/rule-set/)

!!! tip

    `sing-box geosite` 命令可以帮助您将自定义 Geosite 转换为规则集。

=== ":material-card-remove: 弃用的"

    ```json
    {
      "route": {
        "rules": [
          {
            "geosite": "cn",
            "outbound": "direct"
          }
        ],
        "geosite": {
          "download_detour": "proxy"
        }
      }
    }
    ```

=== ":material-card-multiple: 新的"

    ```json
    {
      "route": {
        "rules": [
          {
            "rule_set": "geosite-cn",
            "outbound": "direct"
          }
        ],
        "rule_set": [
          {
            "tag": "geosite-cn",
            "type": "remote",
            "format": "binary",
            "url": "https://raw.githubusercontent.com/SagerNet/sing-geosite/rule-set/geosite-cn.srs",
            "download_detour": "proxy"
          }
        ]
      },
      "experimental": {
        "cache_file": {
          "enabled": true // required to save rule-set cache
        }
      }
    }
    ```