---
publishDate: 2025-12-26T00:00:00Z
title: Batch generate vmess ipv6 nodes
excerpt: Using free ipv6 tunnel from he.net ipv6 tunnelbroker,xray batch generate so many vmess nodes
image: https://img.siu5.dpdns.org/file/dypics/1762312516047_1750697289092.jpg?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1651&q=80
category: Documentation
tags:
  - astro
  - tailwind css
  - front-end
metadata:
  canonical: https://astrowind.vercel.app/astrowind-template-in-depth
---

## Batch generate vmess nodes 批量生成vmess ipv6節點

前置警告與準備

VPS 要求：必須擁有獨立的公網 IPv4 地址（不能是 NAT），且防火牆允許 ICMP (Ping)，否則 Tunnelbroker 無法建立連線。

HE.net 帳號：你需要先在 Hurricane Electric Tunnelbroker 註冊並建立一個 Tunnel。

TOS 風險：短時間內產生大量連接或用於濫用可能導致 HE.net 封鎖帳號或 VPS 商家暫停服務。

第一步：配置 Tunnelbroker (HE.net)
登入 Tunnelbroker。

點擊 "Create Regular Tunnel"。

在 IPv4 Endpoint 輸入你 VPS 的公網 IPv4。

選擇一個離你 VPS 最近的伺服器節點，點擊 Create。

在 Tunnel 詳情頁面，找到 "Routed /64" 這一欄（例如 2001:470:1f0a:123::/64）。這就是我們要用的 IP 池。

以下是針對 60001-60100 端口，每個端口綁定一個專屬 IPv6 地址的完整配置方案。

⚠️ 核心先決條件 (AnyIP 設定)
您必須確認您的 VPS 已經啟用了 AnyIP 設定，將整個 /64 路由塊指向本機迴環接口，這是 Xray sendThrough 功能能正常運作的關鍵。

如果尚未設置，請執行以下命令（需替換為您的 /64 網段，例如 2001:db8:1234:5678::/64）：

Bash

# 確保您的 /64 網段已正確路由到本機
sudo ip route add local 您的/64網段 dev lo
步驟一：系統資源優化 (建議執行)
雖然只有 100 個端口，但解除文件句柄限制是一個好的習慣，以確保 Xray 在大量 I/O 操作下穩定運行。

修改 /etc/security/limits.conf 在文件末尾添加以下內容（或確保數值夠高，例如 1000000）：

Bash

* soft nofile 50000
* hard nofile 50000
root soft nofile 50000
root hard nofile 50000
修改 /etc/sysctl.conf 確保文件句柄的最大數量足夠：

Bash

fs.file-max = 1000000
生效配置

Bash

sysctl -p
# 建議重啟 VPS 以確保 limits.conf 生效
reboot
步驟二：生成 Xray 配置腳本
我們使用 Python 腳本生成 config.json，將端口 60001 綁定到 ::1，60002 綁定到 ::2，以此類推直到 60100 綁定到 ::100。

創建腳本 gen_xray_bind.py：

Python

import json
import base64

# ==================== 請替換以下變數 ====================
UUID = "YOUR_UUID_HERE"          # 替換為您的 UUID
PREFIX = "2001:470:xxxx:xxxx"    # 替換為您的 Routed /64 前綴 (例如: 2001:db8:1234:5678)
VPS_IPV4 = "YOUR_VPS_IP"         # 替換為您的 VPS 公網 IPv4 (用於生成分享連結)
# =========================================================

# --- 配置參數 ---
START_PORT = 60001
NODE_COUNT = 100
CONFIG_PATH = "/usr/local/etc/xray/config.json"
# ----------------

config = {
    "log": {"loglevel": "warning"},
    "inbounds": [],
    "outbounds": [],
    "routing": {
        "rules": []
    }
}

vmess_links = []

print(f"Generating configuration for {NODE_COUNT} ports ({START_PORT} - {START_PORT + NODE_COUNT - 1})...")

for i in range(1, NODE_COUNT + 1):
    port = START_PORT + i - 1
    ip_suffix = i
    bind_ip = f"{PREFIX}::{ip_suffix}"
    
    tag_in = f"in_{ip_suffix}"
    tag_out = f"out_{ip_suffix}"

    # 1. Inbound: 監聽特定端口
    config["inbounds"].append({
        "tag": tag_in,
        "port": port,
        "protocol": "vmess",
        "settings": {
            "clients": [{"id": UUID, "alterId": 0}]
        },
        "streamSettings": {"network": "tcp"}
    })

    # 2. Outbound: 指定出口 IPv6 (sendThrough)
    config["outbounds"].append({
        "tag": tag_out,
        "protocol": "freedom",
        "sendThrough": bind_ip, # 核心：指定每個端口的出口 IP
        "settings": {}
    })

    # 3. Routing: 強制將 Inbound 流量導向對應的 Outbound
    config["routing"]["rules"].append({
        "type": "field",
        "inboundTag": [tag_in],
        "outboundTag": tag_out
    })

    # 4. 生成 VMess 連結
    vmess_json = {
        "v": "2",
        "ps": f"Port{port}_IPv6::{ip_suffix}",
        "add": VPS_IPV4,
        "port": port,
        "id": UUID,
        "aid": 0,
        "net": "tcp",
        "type": "none",
        "tls": ""
    }
    b64_str = base64.b64encode(json.dumps(vmess_json).encode()).decode()
    vmess_links.append(f"vmess://{b64_str}")

# 添加默認的兜底 Outbound
config["outbounds"].append({"protocol": "freedom", "tag": "default"})

# 寫入配置文件
with open(CONFIG_PATH, "w") as f:
    json.dump(config, f, indent=2)

# 寫入連結文件
with open("bound_vmess_links.txt", "w") as f:
    f.write("\n".join(vmess_links))

print(f"✅ 配置已成功生成並保存到 {CONFIG_PATH}")
print(f"🔗 VMess 連結已保存到 bound_vmess_links.txt")
步驟三：執行腳本與部署
停止 Xray

Bash

systemctl stop xray
執行 Python 腳本 請先將腳本中的 UUID、PREFIX 和 VPS_IPV4 替換為您的實際值。

Bash

python3 gen_xray_bind.py
（如果您的系統沒有 python3 命令，請嘗試 python）

覆蓋 Systemd 服務文件 (重要) 編輯 /etc/systemd/system/xray.service，在 [Service] 下確保有以下配置：

Ini, TOML

# 允許綁定非本地 IP (這是 AnyIP 成功運作的關鍵)

AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE

重新加載和啟動 Xray

Bash

systemctl daemon-reload
systemctl start xray
systemctl status xray
如果 Xray 成功啟動，則配置完成。

步驟四：驗證結果
打開腳本生成的 bound_vmess_links.txt 文件。

第一個連結：對應端口 60001，其出口 IP 應為 PREFIX::1。

第一百個連結：對應端口 60100，其出口 IP 應為 PREFIX::100。

將這些連結導入您的客戶端，並測試連接：

連接到端口 60001 的節點。

訪問 https://test-ipv6.com 或 https://ip.sb。

確認顯示的 IPv6 地址是 PREFIX::1。

切換到其他端口（例如 60050），驗證出口 IPv6 是否正確變為 PREFIX::50。

這樣，您就實現了每個固定端口對應一個固定 IPv6 出口地址的需求。
