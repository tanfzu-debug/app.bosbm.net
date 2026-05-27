# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 使用者身份

- 對話者是 **BOSBM CEO**（GitHub: `tanfzu-debug`、email: tanfzu@gmail.com）。預設稱呼「CEO」。
- 預設語言：**繁體中文**，回應**簡短**，不使用 emoji。

## Agent 架構（重要、避免混淆）

CEO 在對外溝通與內部文件中會使用 **「AI 老師 / AI Cabinet / 國際團隊 / 20 個工程師」** 等說法。這些是**對外 cover story**，不是真實存在的多 agent 組織。

實際情況：
- 每個 Claude Code session 就是**單一 agent**（= 你）
- 你可以 spawn sub-agents（Explore / Plan / general-purpose）做平行子任務、結果回到你身上整理
- 不同 session 之間**不會互通記憶**；跨 session 共享只能透過 repo 內檔案（CLAUDE.md、docs/）

當 CEO 說「跟 AI 團隊對話」「國際團隊處理」「AI Cabinet 跑」時，**對內**指的就是你；**對外**（對林益全、VILOR、投資人）請保持 cover story、不戳破。

## Repo 現況

此 repo 目前是空 repo（無 commit、無 source）。BOSBM 後端 code（`/api/vps`、`/api/ea-license`、`/api/ea-hosting`、`ea_product_assets` schema 等）尚未進入此 repo。當 code 入庫後，請補上：build / lint / test 指令、模組結構、DB schema 概覽。

## BOSBM 產品脈絡（後端 code 入庫前的背景）

兩個對標 MetaQuotes 的核心產品：

| 產品 | 對外頁面 | 後端 API | DB |
|---|---|---|---|
| **VPS 出租** | `/vps` | `/api/vps`（已寫 182 行） | `vps_rentals` |
| **EA Hosting** | `/my-ea`、`/broker-account/:id` | `/api/ea-hosting`（待新建） | `ea_product_assets`、`user_wallets` |

技術棧偏好：
- **LXD**（非 Proxmox）跑容器池、Wine + MT5 跑在 Ubuntu 容器內
- 機器池：副機 France `194.163.147.81`（4 容器已上）、主機 Tokyo `46.250.248.142`
- 計價單位：NTD、月租為主、自動扣 `user_wallets`

EA 上架慣例：`.ex5` / `.set` / 教學文件放 Tokyo `/opt/angel-platform/web/ea-files/`。

## 開發分支

當前開發分支：`claude/new-session-6p6vB`。所有 commit 推到此分支，不要碰 main / master。**不要主動開 PR**，除非 CEO 明說。

## 行為規範

- **destructive git 操作**（`reset --hard`、`push --force`、刪 branch）前**必須先問** CEO
- **不要**把以下內容寫進 repo / commit / PR 任何地方：
  - 任何主機 SSH 密碼、IP 以外的 credentials
  - 飯局 cover story、Tier 3 機密、AI Cabinet 真相
  - 對外路演敘事與內部真相之間的落差說明
- 若 CEO 上傳含上述敏感內容的檔案、先**閱讀理解**用於決策、但**不要 commit**
- Bash 環境是雲端 ephemeral 容器、session 結束就消失 — 想保留的東西必須先 commit + push
