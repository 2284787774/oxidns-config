# oxidns-config
oxidns配置 支持国内外分流、自动更新规则、无DNS泄露、UI可视化

[OxiDNS](https://github.com/svenshi/oxidns)

## ✨ 核心特性

* **🛡️ 全面广告过滤**：深度集成 Geosite 广告规则与 AdGuard 规则，自动将广告和追踪域名拉入“黑洞” (Blackhole)。
* **🌍 精准 Geo 分流 (GeoIP / Geosite)**：根据 `geosite_cn`、`geosite_no_cn` 和 `geoip_cn`，智能判断域名和 IP，自动将国内请求分发给国内 DNS，海外请求分发给海外 DNS。
* **📍 智能 ECS (EDNS Client Subnet)**：支持向海外上游发送指定的子网 IP（如香港 IP `203.142.100.0/24`），获取对海外 CDN 最友好的解析结果。
* **⚙️ 自定义灰名单/白名单**：支持通过 `ecscn` 和 `ecs_noncn` 强制指定特定域名的解析路由，解决部分规则遗漏或误判的问题。
* **🔄 规则全自动更新**：内置定时任务 (Cron)，每 24 小时自动从 GitHub 下载最新的 GFW 列表、广告规则、GeoIP 和 Geosite 文件，并实现**热重载**（不中断服务）。

---

自定义域名路由

本配置内置了两个强大的自定义域名规则集，专门用于手动微调路由走向：

* **`ecscn` (强制国内解析)**：如果您发现某个国内网站被误判走了海外 DNS 导致加载慢，可以将其域名加入 `ecscn`。匹配到此列表的域名将**无条件**通过国内 DNS（腾讯/阿里）解析。
* **`ecs_noncn` (强制海外解析)**：如果您发现某个海外网站或服务（如 Github 某子域名）使用了国内节点导致无法访问，将其加入 `ecs_noncn`。匹配后将**无条件**通过海外 DNS (Google/NextDNS) 解析。

💡 **如何添加域名？**
您**不需要**手动修改配置文件或重启服务。直接访问本地 Web 管理界面：
👉 **[http://127.0.0.1:9199](https://www.google.com/search?q=http://127.0.0.1:9199)**
在 Web 界面中找到 `ecscn` 或 `ecs_noncn` 规则集，直接输入想要指定的域名（如 `oxidns.org`）并保存应用即可。

---

### 流程简述：

1. **安全与加速**：请求进来先过限流器，然后查 Hosts 和广告名单。一旦命中，马上返回，不浪费网络资源。
2. **高速缓存**：拦截器放行后，查本地高速缓存，如果有记录直接秒开。
3. **特权路由 (ecscn/noncn)**：检查用户在 WebUI 自定义的强制名单，优先级最高。
4. **智能分流**：根据 Geosite 判断域名归属地，各回各家。
5. **兜底纠错**：对于冷门域名，同时发给海外安全 DNS（ControlD/Quad9等），并通过 GeoIP 验证返回的 IP 是否被污染，确保拿到最干净的结果。


<img width="1720" height="909" alt="image" src="https://github.com/user-attachments/assets/803b5efe-5612-4e29-93b4-1e27131babc9" />
