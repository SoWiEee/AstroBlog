---
title: TeamT5 Security Camp 2026
published: 2026-01-18
description: ''
image: 'teamt5.png'
tags: [InfoSec]
category: 'Note'
draft: false 
lang: ''
---

# Introduction

寒假參加了由 TeamT5 舉辦的資安培訓營（需通過前測篩選）。這幾天的課程並非零散的工具教學，而是非常有系統性地串聯起資安攻防的各個面向。這五天的培訓內容大致如下：

- 資安事件處理剖析（Forensic）：學習如何從混亂的現場找出駭客的蛛絲馬跡
- 初探威脅情資的奧秘（CTI）：不只是看病毒，更要追蹤背後的攻擊組織
- 漏洞挖掘的深入體驗（IoT, ARM）：挑戰嵌入式裝置與硬體模擬
- 與系統底層一日邂逅（Linux）：透過遊戲外掛製作，學習動態分析與偵測規則
- 解鎖資安職涯的多重視角

# Day 01

先從資安事件處理 (Incident Response, IR) 的全貌開始。知道 IR 不僅僅是技術上的止血，更包含了與客戶的溝通藝術以及撰寫專業的事件報告。在技術層面，重點在於建立處理資安事件的 mindset。

在技術實作方面，課程設計採用了非常啟發性的「問題導向教學」。講師在分析 Windows 案例前，先拋出幾個關鍵問題，讓我們帶著疑問去操作 Lab。對於習慣打 CTF Forensic 的人來說應該不陌生，但更強調正規的 SOP。我們利用數位鑑識工具，搭配講師準備的鑑識懶人包，學習如何從 Windows 的活動紀錄（如 Event Logs, Registry, Prefetch 等）中抽絲剝繭。

1. 利用數位鑑識工具檢視 Windows 系統留下的蛛絲馬跡（程式活動紀錄）
2. 搭配講師提供的工具懶人包，遵循邏輯流程抽絲剝繭
3. 最終找到 Root Cause，還原事件真相

雖然對 Windows 複雜的系統機制還不算非常熟悉，但透過系統本身留下的程式執行紀錄，搭配鑑識工具的輔助，我們最終能還原攻擊者的入侵路徑，並找出真正的 Root Cause。這不只是找 Flag，而是一場還原真相的推理過程。

🍽️ 午餐是[十一雞](https://maps.app.goo.gl/1KSadRfrKR7iNVfg9)

# Day 02

第二天的課程進入了威脅情資（Cyber Threat Intelligence, CTI）的領域。過去我對 VirusTotal 的認知僅停留在「掃描檔案有沒有毒」的 Sandbox，但透過講師的解析，我才發現它其實是一個巨大的情資關聯資料庫，能幫助我們追查惡意程式的變種與來源。

在 Lab 環節，我們深入 Windows PE (Portable Executable) 檔案的靜態分析。使用了 PE Bear、Detect it Easy (DiE) 等工具來解析檔案結構，並學習辨識常用的 Windows API，藉此推測惡意程式可能的行為。講師也大方分享了許多 IDA plugin，這對於提升逆向工程的效率非常有幫助。

這天的精華在於**歸因**(Attribution)。我們不只要分析這隻程式「做了什麼」，還要試著從程式碼特徵、字串或行為模式中，回推這可能是哪一個駭客組織（Adversary）的手筆。這讓我了解到，資安防禦不只是被動擋下攻擊，更要主動了解對手是誰，才能知己知彼。

🍽️ 午餐是[札等伍參餐盒](https://maps.app.goo.gl/erachGqPSUorJHLU9)

# Day 03

兩位講師看起來都做過一段時間的 IoT 研究，感覺超可靠。上午是教漏洞研究的方法論(Methodology)，從如何進行前期 Survey、取得 Firmware，到最後挖掘出漏洞，建立了一套完整的攻擊鏈思維。

下午的實作則是充滿挑戰的「模擬戰」。由於 IoT 設備通常運行在 ARM 或 MIPS 架構上，我們無法直接在筆電執行，因此需要依賴 QEMU 和 Unicorn 這類模擬器。一開始可能會用 binwalk 對韌體進行解包與初步分析，了解其檔案結構。接著進入最困難也最有趣的環節——環境模擬。

在模擬過程中，往往會因為缺乏硬體周邊或環境變數而導致程式崩潰。這時就需要搭配 IDA 進行逆向分析，運用一些 Patch 技巧（IDA tricks）來「修復」程式，讓它能順利跑起來以便進行更多研究。這個過程非常考驗對組合語言與系統架構的理解，雖然燒腦，但成功讓韌體跑起來的那一刻非常有成就感。

🍽️ 午餐是[費洛獴](https://maps.app.goo.gl/QzAaTMJU6tznm9JR8)

# Day 04

第四天的主題非常有趣，是從「遊戲外掛」的角度切入系統底層分析。我們探討了前測中的遊戲外掛題目，學習使用 Linux 版本的 Cheat Engine —— PINCE 來進行記憶體行為分析。透過動態追蹤數值變化，並搭配 IDA 的靜態逆向，我們練習如何 Hook 遊戲函式來改變遊戲邏輯。

課程後半段介紹了目前業界主流的偵測規則撰寫。針對靜態檔案，我們學習撰寫 YARA rule，透過定義 Hex Signature 或字串特徵來掃描惡意檔案；而針對動態行為，則學習了 Sigma rule，這是透過監控 Process 的行為模式來進行偵測。

🍽️ 午餐是[烘米餐廚](https://maps.app.goo.gl/w1hnNT3k5gPDmKDw5)

剩下等 1/22 完訓～