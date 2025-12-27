---
publishDate: 2025-12-26T06:30:00Z
title: 添加he.net免費的ipv6 TunnelBroker
excerpt: he.net免費ipv6 Tunnelbroker配置指南.
image: https://img.siu5.dpdns.org/file/dypics/1762312515289_1751052421281.jpg?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1651&q=80
category: Documentation
tags:
  - ipv6
  - tunnelbroker
  - free
metadata:
  canonical: https://toolsblog.pages.dev/add-henet-free-ipv6
---

## 添加he.net免費的ipv6 TunnelBroker

在 Debian 或 Ubuntu 系統中，透過 HE.net (Tunnelbroker) 配置 IPv6 隧道是一個非常穩定的方案。使用 /etc/network/interfaces.d/ 目錄可以讓你的網路配置模組化，方便管理。

以下是分步操作指南：

第一步：獲取 Tunnelbroker 信息
在開始之前，請登錄你的 Tunnelbroker 帳戶，點擊你的隧道並獲取以下關鍵信息：

Server IPv4 Address (HE 端服務器地址)

Client IPv4 Address (你的公網 IPv4，如果是 NAT 後的環境則為內網 IP)

Client IPv6 Address (通常以 ::2/64 結尾)

第二步：創建配置文件
使用 sudo 權限創建一個名為 he-ipv6 的文件：

Bash

sudo nano /etc/network/interfaces.d/he-ipv6
第三步：寫入配置內容
將以下內容複製並粘貼到文件中，請務必將括號內的值替換為你從 HE 官網獲取的實際數據：

Plaintext

auto he-ipv6
iface he-ipv6 inet6 v4tunnel
    # 你的公網 IPv4 地址 (如果你在路由器後，請填寫本機內網 IP)
    address <Your_Client_IPv6_Address>
    netmask 64
    
    # Tunnelbroker 的服務器端 IPv4 地址
    endpoint <Server_IPv4_Address>
    
    # 本機 IPv4 地址 (與 address 下方的邏輯一致)
    local <Your_Local_IPv4_Address>
    
    # 設置默認路由
    gateway <Server_IPv6_Address_without_slash>
    
    # 啟動後自動設置 TTL
    up ip link set mtu 1480 dev he-ipv6
注意： gateway 通常是 Client IPv6 Address 去掉末尾 /64 後把最後一位數字改為 1。例如你的地址是 2001:470:1f10:xxx::2/64，網關通常是 2001:470:1f10:xxx::1。

第四步：啟動隧道接口
保存並退出（在 nano 中按 Ctrl+O 然後 Enter，再按 Ctrl+X）。

接著，使用以下命令啟用該接口：

Bash

sudo ifup he-ipv6
第五步：驗證連接
運行以下命令檢查 IPv6 是否已成功分配並連通：

查看接口狀態：

Bash

ip -6 addr show he-ipv6
測試 Ping：

Bash

ping6 google.com -c 4
常見問題小貼士
防火牆： 確保你的防火牆（如 ufw 或 iptables）允許 Protocol 41 (IPv6-in-IPv4 封裝)。

動態 IP： 如果你的公網 IPv4 是動態的，你需要定期向 HE 更新你的端點 IP，否則隧道會失效。

MTU 設置： 默認 1480 通常最穩定，如果網絡不通，可以嘗試降至 1472 或 1280。
