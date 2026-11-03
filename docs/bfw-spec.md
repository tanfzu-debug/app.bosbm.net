# BFW — BOSBM Software Firewall 技術規格

> 狀態：草案 v0.1
> 定位：**純軟體防火牆**。跑在標準 x86-64 Linux 上，不做自有硬體、不碰 PCB / EMC / ODM。
> 第一批落地場景：BOSBM VPS 出租客戶的雲端防火牆加值服務。

---

## 1. 產品定位

| 項目 | 決策 |
|---|---|
| 形態 | 軟體套件（deb package + 可選 OS image） |
| 執行環境 | Debian 12/13 bare metal、KVM/VM、LXD 容器（容器模式功能受限，見 §7） |
| 不做 | 自有硬體、ASIC、晶片級加速 |
| 商業模式 | 訂閱授權（每節點月租）＋ VPS 加值綁定，計價 NTD，自動扣 `user_wallets` |
| 目標客群 | 中小企業辦公室出口、台商海外據點、BOSBM 自有 VPS 租戶 |

### 兩種銷售 SKU

- **BFW-Cloud**：部署在 BOSBM 節點（France / Tokyo），客戶不需硬體，用 WireGuard 把辦公室流量導上來過濾。上架最快。
- **BFW-OnPrem**：授權 + 安裝腳本，客戶自備主機（推薦規格見 §6）。

---

## 2. 架構分層

```
┌──────────────────────────────────────────────┐
│  管理層  bfw-ui (React)  ←→  bfw-api (Go)     │
├──────────────────────────────────────────────┤
│  控制層  bfw-core：設定模型 → 規則編譯 → 套用   │
│          設定版本控制 / rollback / HA 同步     │
├──────────────────────────────────────────────┤
│  資料層  nftables（L3/L4 + NAT）              │
│          Suricata（IDS/IPS, AF_PACKET or NFQ） │
│          WireGuard / strongSwan（VPN）        │
│          Unbound + RPZ（DNS 過濾）             │
├──────────────────────────────────────────────┤
│  觀測層  nflog → vector → ClickHouse → 報表    │
└──────────────────────────────────────────────┘
```

**核心設計原則**：控制層永遠不直接下指令改核心狀態。所有變更都是
「改設定 → 編譯成完整 nftables ruleset → 原子 `nft -f` 套用 → 驗證 → 成功則 commit，失敗則自動 rollback」。
這是 pfSense / OPNsense 的做法，也是唯一能避免半套規則把客戶鎖在外面的方式。

---

## 3. 技術選型與理由

| 層 | 選擇 | 理由 | 替代方案 |
|---|---|---|---|
| 封包過濾 | **nftables** | Debian 原生、atomic replace、單一 ruleset 檔好做版本控制 | iptables（舊）、XDP/eBPF（v2 再上） |
| IPS | **Suricata 7** | 多執行緒、支援 ET Open 免費規則、JSON(EVE) 日誌好接 | Snort3（單執行緒較弱） |
| VPN | **WireGuard** 主、**strongSwan** 備 | WG 效能與設定複雜度都勝；IPsec 只為了對接老設備 | OpenVPN（效能差，不列入） |
| DNS 過濾 | **Unbound + RPZ** | 輕量、黑名單可熱更新 | Pi-hole（不適合多租戶） |
| API | **Go** | 單一靜態 binary、好打包、無 runtime 依賴 | FastAPI（跟現有 `/api/vps` 一致，但多 Python 依賴） |
| UI | **React + Tailwind** | 跟 app.bosbm.net 前端同棧 | — |
| 設定儲存 | **SQLite（節點本地）+ PostgreSQL（中控）** | 節點斷線仍可運作 | etcd（過重） |
| 日誌 | nflog/EVE → **Vector** → **ClickHouse** | ClickHouse 壓縮比與查詢速度適合流量日誌 | Elasticsearch（吃資源） |
| HA | **keepalived (VRRP)** + 設定單向同步 | 成熟、不需共享儲存 | — |

### 授權地雷（重要）

- **OPNsense** = BSD-2，可閉源貼牌 → 若要抄架構，看它。
- **VyOS** = GPL，閉源商用有風險 → **不要 fork**。
- **Suricata** = GPL-2，但我們是「呼叫執行檔 + 讀它的日誌」，不 link 它的程式庫，不會傳染。
- **ET Open 規則** 可商用；ET Pro 要付費。
- **nftables (libnftnl)** = GPL，同樣只透過 `nft` 指令與檔案互動，不直接 link。

結論：只要「不 fork GPL 專案、不 link GPL 程式庫，只以子程序方式驅動」，我們的 `bfw-core` / `bfw-api` / `bfw-ui` 可以保持專有閉源。

---

## 4. 模組結構（規劃）

```
bfw/
├── cmd/
│   ├── bfw-api/            # HTTP API + WebSocket（即時流量）
│   ├── bfw-core/           # 規則編譯器 + 套用器 daemon
│   └── bfwctl/             # CLI（給客戶 SSH 進去救援用）
├── internal/
│   ├── model/              # 設定資料模型（zone/rule/nat/vpn/alias）
│   ├── compile/
│   │   ├── nft.go          # model → nftables ruleset 文字
│   │   ├── suricata.go     # model → suricata.yaml + 自訂規則
│   │   ├── wg.go           # model → wg0.conf
│   │   └── unbound.go      # model → RPZ zone 檔
│   ├── apply/              # atomic apply + health check + rollback
│   ├── state/              # SQLite，設定版本（每次變更一個 revision）
│   ├── telemetry/          # conntrack 取樣、介面統計、EVE 解析
│   ├── ha/                 # VRRP 狀態、設定同步
│   └── license/            # 節點授權驗證（離線寬限 7 天）
├── web/                    # React 管理介面
├── packaging/
│   ├── debian/             # .deb 打包
│   └── image/              # 可選的安裝 ISO（buildroot/preseed）
└── docs/
```

### 整合現有後端

中控面接在既有 `app.bosbm.net`：
- 新增 `/api/firewall` — 節點註冊、授權簽發、設定下發、告警彙整
- DB 新增 `firewall_nodes`、`firewall_revisions`、`firewall_alerts`
- 計費沿用 `user_wallets` 月扣模式，與 `vps_rentals` 同一套扣款排程

---

## 5. 功能範圍（MVP vs v2）

### MVP（必須有，否則賣不出去）
- [ ] Zone-based 防火牆規則（WAN/LAN/DMZ/VPN），可排序、可註解、可暫時停用
- [ ] NAT：SNAT/masquerade、DNAT port forward、1:1 NAT
- [ ] 別名物件（Alias）：IP / 網段 / 連接埠群組，規則引用
- [ ] WireGuard site-to-site 與 client（含 QR code 設定檔）
- [ ] DHCP server + 靜態租約
- [ ] DNS 過濾（RPZ 黑名單，含廣告/惡意網域清單自動更新）
- [ ] Suricata IDS 模式（先只告警，不阻擋）
- [ ] 流量日誌與 Top Talkers 報表
- [ ] 設定備份/還原、revision rollback
- [ ] 管理員帳號 + 2FA（TOTP）、操作稽核日誌

### v2
- [ ] Suricata IPS 模式（NFQUEUE 內聯阻擋）
- [ ] 應用層識別（TLS SNI、HTTP Host、QUIC 偵測）
- [ ] 使用者身分綁定（LDAP / AD 整合）
- [ ] HA 主備切換
- [ ] 多租戶中控儀表板（MSP 模式，給代理商賣）
- [ ] XDP/eBPF 快速路徑（10G+ 才需要）
- [ ] GeoIP 封鎖、威脅情資訂閱

### 明確不做
- 不做 SSL 中間人解密（法務與信任成本過高，企業客戶也抗拒）
- 不做沙箱動態分析（需要自建雲端沙箱，成本結構不合）
- 不做 Windows 版主機防火牆（完全不同產品）

---

## 6. 硬體建議規格（給 OnPrem 客戶，我們不出貨硬體）

| 等級 | 使用者數 | 吞吐目標 | 規格 |
|---|---|---|---|
| S | ≤ 25 | 500 Mbps | 4 核 x86、8 GB RAM、64 GB SSD、Intel i226 雙網口 |
| M | ≤ 150 | 2 Gbps | 8 核、16 GB RAM、256 GB NVMe、Intel X710 四網口 |
| L | ≤ 500 | 10 Gbps | 16 核、32 GB RAM、512 GB NVMe、X710/E810 |

網卡**只建議 Intel**。Realtek 在高 PPS 下驅動掉包，會變成支援惡夢。
開 Suricata IPS 時吞吐約打對折，報價頁要寫清楚。

---

## 7. LXD 容器模式的限制（我們現有機器池）

現有池子是 LXD（France 副機 4 容器已上、Tokyo 主機）。非特權容器無法完整控制 netfilter。

- **可行**：WireGuard 終結點、DNS 過濾、Suricata IDS（被動監聽）、API/UI
- **需要特權容器或 VM**：nftables 完整 NAT/mangle、conntrack 調參、XDP
- **決策**：BFW-Cloud 的資料平面跑在 **KVM VM 或特權 LXD 容器**，管理面可繼續放普通容器。
  部署前要先在 France 副機實測一台特權容器，確認 nftables 可用。

---

## 8. 里程碑

| 階段 | 內容 | 預估 |
|---|---|---|
| M0 | 技術驗證：特權容器 nftables + WireGuard + Suricata 跑通 | 1 週 |
| M1 | `bfw-core` 規則編譯器 + atomic apply + rollback | 3 週 |
| M2 | `bfw-api` + SQLite 設定模型 + revision | 3 週 |
| M3 | React 管理介面（規則/NAT/VPN/儀表板） | 4 週 |
| M4 | 日誌管線 + 報表 + 告警 | 2 週 |
| M5 | 打包 .deb、授權機制、`/api/firewall` 中控對接 | 3 週 |
| M6 | 內部試跑（自用一個月）→ 首批客戶 pilot | 4 週 |

合計約 **5 個月**到可收費的 MVP。單人（我）開發節奏，不含認證與行銷。

---

## 9. 風險與現實評估

1. **規則庫是持續成本**，不是一次性開發。ET Open 免費但更新慢；真要打企業單，威脅情資要編預算。
2. **企業採購會問認證**：ISO 27001、共同供應契約、資安署資格。這些不是寫 code 能解決的，要另外排。
3. **競品是 FortiGate / PA / Sophos**，他們在「硬體 + 簽章庫 + 全球支援」有結構優勢。
   我們的切入點只能是：**價格、在地支援、與 BOSBM VPS／EA 服務綑綁**，不要正面打功能表。
4. **支援負擔**：防火牆設錯會讓客戶整間公司斷網。要先把 rollback、救援 CLI、遠端支援通道做紮實，再賣第二家。
5. **MVP 之前不要對外承諾 IPS**。IDS（告警）和 IPS（阻擋）的支援成本差一個量級。

---

## 10. 下一步

M0 技術驗證要先做，才知道架構站不站得住。需要 CEO 確認：
1. SKU 優先序：先做 BFW-Cloud（無硬體、綁現有 VPS 客戶）還是 BFW-OnPrem？
2. 是否同意在 France 副機開一台特權容器做 nftables 實測？
3. 首批 pilot 客戶來源（現有 VPS 租戶？外部？）
