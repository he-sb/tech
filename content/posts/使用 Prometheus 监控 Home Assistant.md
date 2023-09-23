+++
title = "使用 Prometheus 监控 Home Assistant"
description = " "
date = "2023-09-24T01:29:02+08:00"
toc = true
categories = ["奇技淫巧"]
tags = ["homelab","docker","prometheus","hass","grafana"]
slug = "hass-prometheus"
draft = false
+++

为了方便叙述，本文将使用 hass 指代 Home Assistant.

首先稍微解释一下为什么不用 hass 自己的 dashboard 来展示数据：

1. 已经搭建了 PLG (Prometheus + Loki +Grafana)  监控系统，对于数据的监控和告警，统一在 Grafana 内配置和查看比较高效，不需要在 hass 再维护一套单独的 dashboard
2. hass 仅负责家中智能 IoT 设备的【操作 + 自动化】，数据监控交给 PLG 来做，贯彻单一职责原则，避免在单个系统内增加不必要的复杂度

关于 hass 和 PLG 的搭建，由于内容较多，俺打算后续单独撰文分别记录（先挖个大坑），本文仅记录如何将 hass 的数据暴露给 Prometheus.

## 00. 配置 hass 暴露监控指标

编辑 hass 的配置文件 `configuration.yaml`（如果你和俺一样是用 docker 安装 hass 的话，这个文件在容器内的 `/config` 路径下），在其中新增以下内容后重启 hass 即可：

```yaml
# 启用 prometheus api
prometheus:
```

注意这两行内容均是顶格书写的，不要有任何缩进或空格。

重启完成后，进入 hass 后台，点击左下角的用户名，翻到页尾，创建一个长期访问令牌，创建成功后记得先复制出来备用，这个令牌仅会在创建成功时展示一次，如果忘了复制，就只能删掉重新创建一个了。

一般来说现在可以去配置 Prometheus 抓取数据了，想要验证的话，可以使用 cURL：

```shell
curl -X GET \
  https://ha.he-sb.home/api/prometheus \
  -H 'Authorization: Bearer <刚复制的令牌>'
```

## 01. 配置 Prometheus 抓取 hass 的数据

编辑 Prometheus 的配置文件，加入抓取 hass 的配置：

```yaml
  - job_name: 'hass'
    scrape_interval: 60s
    metrics_path: /api/prometheus
    # Long-Lived Access Token
    authorization:
      credentials: "此处粘贴刚复制的令牌"
    scheme: https
    tls_config:
      ca_file: "he-sb.home.crt"
    static_configs:
      - targets:
        - ha.he-sb.home
```

注意这里俺使用了 https 协议，自定义的内网域名，以及对应的自签名证书——因为俺的 hass 容器没有暴露端口，通过部署在同一台机器上且在同一个 docker 网络的 Traefik 容器反代 + 内网自建 DNS 服务，来提供内网的 https 访问，且 Prometheus 是使用 docker 部署在内网另外一台机器。

如果你也是类似的配置（比如使用 nginx 或者 caddy 反代等），那么需要注意：

1. `scheme` 需要为 `https`
2. `ca_file` 是自签名证书的文件路径
  - 如果你和俺一样是用 docker 部署 Prometheus 的话，需要注意这个路径是容器内路径，不是宿主机的
4. `ha.he-sb.home` 替换为你自己的域名

如果你的 hass 部署方式比较简单，那么需要按需修改上方的配置：

1. 如果没有配置 https 访问，那么需要删除 `scheme` 和 `tls_config` 部分
2. 如果配置了 https 访问，但不是自签名证书（比如公网 DDNS 域名），那么只需要删掉 `tls_config` 部分
3. 如果啥也没配置，直接通过【IP】或【IP + 端口】方式访问 hass 的，那么不仅需要删除 `scheme` 和 `tls_config` 部分，`ha.he-sb.home` 也要替换为 hass 所在机器的 IP + 端口

配置修改完成后，重启 Prometheus 使改动生效（如果你的 Prometheus 开启了 `--web.enable-lifecycle` 参数，那么直接在内网任意机器使用 POST 方式调用 Prometheus 的 `/-/reload` api 也可以重载配置文件，不需要重启），此时查看 Prometheus 后台的【Status - Targets】页面，应该能看到 hass 已经能被正确地抓取了。

## 02. 配置 Grafana 监控面板

俺暂时只配置了 4 个指标的监控：

- 空调实时功率
- 空调的月度用电量
- PDU 插排的实时功率
- PDU 插排的月度用电量

可以直接 import 下方的 json 内容，看一下实际效果再做调整即可。

<details>
<summary>
点击展开
</summary>

```json
{
  "__inputs": [
    {
      "name": "DS_PROMETHEUS",
      "label": "Prometheus",
      "description": "",
      "type": "datasource",
      "pluginId": "prometheus",
      "pluginName": "Prometheus"
    }
  ],
  "__elements": {},
  "__requires": [
    {
      "type": "grafana",
      "id": "grafana",
      "name": "Grafana",
      "version": "10.1.1"
    },
    {
      "type": "datasource",
      "id": "prometheus",
      "name": "Prometheus",
      "version": "1.0.0"
    },
    {
      "type": "panel",
      "id": "stat",
      "name": "Stat",
      "version": ""
    },
    {
      "type": "panel",
      "id": "timeseries",
      "name": "Time series",
      "version": ""
    }
  ],
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "fiscalYearStartMonth": 0,
  "graphTooltip": 0,
  "id": null,
  "links": [],
  "liveNow": false,
  "panels": [
    {
      "datasource": {
        "type": "prometheus",
        "uid": "${DS_PROMETHEUS}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          },
          "unit": "watt"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 3,
        "x": 0,
        "y": 0
      },
      "id": 2,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "10.1.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "${DS_PROMETHEUS}"
          },
          "editorMode": "code",
          "expr": "homeassistant_sensor_power_w{entity=\"sensor.lumi_mcn02_f582_electric_power\",friendly_name=\"客厅空调 功率\"}",
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "客厅空调 功率",
      "transparent": true,
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "${DS_PROMETHEUS}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 100
              },
              {
                "color": "red",
                "value": 150
              }
            ]
          },
          "unit": "watt"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 3,
        "x": 3,
        "y": 0
      },
      "id": 1,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "10.1.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "${DS_PROMETHEUS}"
          },
          "editorMode": "code",
          "expr": "homeassistant_sensor_power_w{entity=\"sensor.cuco_v3_682d_electric_power\",friendly_name=\"PDU 功率\"}",
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "PDU 功率",
      "transparent": true,
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "${DS_PROMETHEUS}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "decimals": 1,
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "#EAB839",
                "value": 50
              }
            ]
          },
          "unit": "kwatth"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 3,
        "x": 6,
        "y": 0
      },
      "id": 4,
      "options": {
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "10.1.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "${DS_PROMETHEUS}"
          },
          "editorMode": "code",
          "expr": "homeassistant_sensor_energy_kwh{entity=\"sensor.lumi_mcn02_f582_power_cost_month\",friendly_name=\"客厅空调 空调 power_cost_month\"}",
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "月用电量 - 客厅空调",
      "transparent": true,
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "${DS_PROMETHEUS}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "decimals": 1,
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "#EAB839",
                "value": 70
              }
            ]
          },
          "unit": "kwatth"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 3,
        "x": 9,
        "y": 0
      },
      "id": 3,
      "options": {
        "colorMode": "value",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "10.1.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "${DS_PROMETHEUS}"
          },
          "editorMode": "code",
          "expr": "homeassistant_sensor_energy_kwh{entity=\"sensor.cuco_v3_682d_power_cost_month\",friendly_name=\"PDU Switch power_cost_month\"}",
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "月用电量 - PDU",
      "transparent": true,
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "${DS_PROMETHEUS}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "watt"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 0,
        "y": 7
      },
      "id": 5,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "${DS_PROMETHEUS}"
          },
          "editorMode": "code",
          "expr": "homeassistant_sensor_power_w{entity=\"sensor.cuco_v3_682d_electric_power\",friendly_name=\"PDU 功率\"}",
          "instant": false,
          "legendFormat": "PDU",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "PDU 功率",
      "transparent": true,
      "type": "timeseries"
    }
  ],
  "refresh": "1m",
  "schemaVersion": 38,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "",
  "title": "hass",
  "uid": "b24eaa87-7074-4897-9986-fd99e6a83290",
  "version": 3,
  "weekStart": ""
}
```

</details>

## 03. 进阶配置

### 过滤部分实体的指标输出

默认情况下，hass 会暴露所有实体的指标，如果有部分实体不希望暴露给 Prometheus，那么可以修改 hass 的配置文件，增加 `filter` 配置，过滤掉一部分实体：

```yaml
# 启用 prometheus api
prometheus:
  filter:
    exclude_entities:
      - light.kitchen_light
    exclude_entity_globs:
      - binary_sensor.*_occupancy
    include_entities:
      - light.kitchen_light
    include_entity_globs:
      - binary_sensor.*_occupancy
```

稍微解释下：

- `exclude_entities`
  - 需要过滤的实体 ID
  - 仅过滤【完全匹配】的实体
- `exclude_entity_globs`
  - 使用通配符匹配，过滤匹配到的实体 ID
  - 仅过滤【符合规则】的实体
- `include_entities`
  - 需要暴露的实体 ID
  - 仅暴露【完全匹配】的实体
- `include_entity_globs`
  - 使用通配符匹配，暴露匹配到的实体 ID
  - 仅暴露【符合规则】的实体

**注意，如果配置了 `include` 规则，那么所有未命中的实体都【不会】被输出。**

详细的配置和匹配规则请参考 [官方文档](https://www.home-assistant.io/integrations/prometheus/) ，本文不再赘述了。

---

*参考链接：*
1. [prometheus获取homeassistant数据](https://www.bboy.app/2023/04/13/prometheus%E8%8E%B7%E5%8F%96homeassistant%E6%95%B0%E6%8D%AE/)
2. [Prometheus - Home Assistant](https://www.home-assistant.io/integrations/prometheus/)
