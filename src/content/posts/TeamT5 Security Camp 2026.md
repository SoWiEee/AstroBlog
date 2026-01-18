---
title: TeamT5 Security Camp 2026
published: 2026-01-18
description: ''
image: ''
tags: [InfoSec]
category: 'Note'
draft: false 
lang: ''
---

# Introduction

寒假去參加 TeamT5 舉辦的資安培訓營（有前測要通過），有系統性地學到不少東西，這五天的培訓內容大致如下：

- 資安事件處理剖析（Forensic）
- 初探威脅情資的奧秘
- 漏洞挖掘的深入體驗（IoT, ARM）
- 與系統底層一日邂逅（Linux）
- 解鎖資安職涯的多重視角

# Day 01

一開始會介紹 IR 在做什麼，並且除了技術面之外還要學會與客戶溝通，也大致介紹怎麼寫事件報告。然後會提一下資安事件的發生，並建立處理資安事件的正確觀念與心態。講師有準備兩個 Windows 的案例給大家分析，在開始介紹前有先提出幾個問題讓大家記著，這其實是挺有效的教學方式，讓學員能帶著問題聽課，讓學習效果更好。Lab 的部分對 CTF 打 Forensic 的應該有點熟悉，主要是用一些數位鑑識工具，然後遵循大致的流程、搭配簡報準備的工具懶人包，最後找到 root cause。

雖然還是沒有很熟悉，但大概就是 Windows 系統本身會有程式活動的紀錄，根據這些紀錄搭配數位鑑識工具就能抽絲剝繭，釐清資安事件的發生。

午餐是[十一雞](https://maps.app.goo.gl/1KSadRfrKR7iNVfg9)

# Day 02

介紹什麼是威脅情資 (Threat Intelligence)，並且知道 VirusTotal 不只是 sandbox，還可以追查情資相關的有用資訊。然後 Lab 有用一些 Windows 上的分析工具 PE Bear, Detect it Easy 等等來分析一支 PE，並認識常用的 Windows API，還推薦不少好用的 IDA plugin，接著從惡意程式中找到一些資訊來回推可能是什麼樣的 Adversary。

午餐是[札等伍參餐盒](https://maps.app.goo.gl/erachGqPSUorJHLU9)

# Day 03

兩位講師看起來都做過一段時間的 IoT 研究，感覺超可靠。上午是教漏洞研究的方法論，像是如何 survey、拿到 firmware、挖掘漏洞。
下午場是帶實作，可以用 qemu, Unicorn 之類的模擬。首先會用 binwalk 之類的工具對機器有初步的認識，接著模擬的時候遇到一些狀況要怎麼修，可能會搭配 IDA 之類的（學到不少 IDA trick）把東西修到能順利執行。過程中不只考驗工具的熟悉度，也要有不少經驗才能有效率地分析。

午餐是[費洛獴](https://maps.app.goo.gl/QzAaTMJU6tznm9JR8)

# Day 04

主要是講前測的遊戲外掛，可以用 Linux 版的作弊引擎 PINCE 來做行為分析，並搭配 IDA 逆向來做 hook。接著是怎麼偵測惡意程式，像是用 yara rule 來檢查 file signature、用 sigma rule 檢查 process signature，並且實作看看。

午餐是[烘米餐廚](https://maps.app.goo.gl/w1hnNT3k5gPDmKDw5)

剩下等 1/22 完訓～