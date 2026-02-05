---
title: Building an mini EDR!
published: 2026-01-27
description: ''
image: ''
tags: [InfoSec]
category: 'Note'
draft: true 
lang: ''
---

# Introduction

由於之前參加 TeamT5 Camp 的時候做了一個跟惡意程式偵測相關的專案，過程中看了不少關於系統偵測、監控機制的東西。但我發現國內好像比較少有完整的文章在介紹怎麼「從零開始」寫一個 EDR（Endpoint Detection and Response），大部分都是在講怎麼分析 Malware 本身，而不是怎麼寫那個負責抓 Malware 的程式。所以就寫一篇文章整理起來，想知道細節的話可以看底下的 References。

# What you will learn

在這篇文章中，我們不會談太深的逆向工程，而是專注在系統架構面。你會了解一個 EDR 如何透過 Sysmon 和 ETW 收集資料，怎麼設計 JSON 規則引擎來過濾威脅，以及如何利用現成的開源工具來做記憶體掃描。最後，我們也會稍微聊一下大魔王關卡——Kernel Driver 的開發與 IOCTL 通訊。

# Telemetry：眼睛是怎麼看到東西的？

要偵測是否有惡意行為，我們需要 Windows 上的一些活動紀錄才能進行判別，微軟有提供一個系統工具 [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) 可以很方便地收集一些系統事件，而不需要用自行 Hook Windows API，直接訂閱 Sysmon 產生的事件就好。從 Sysmon 的文件可以看到有 ProcessCreate、CreateRemoteThread、ProcessTampering 等等的事件，Sysmon 都幫你正規化好了，省下超多工！

雖然 Sysmon 非常易用，但想要更周全地監聽事件，會建議使用 Windows 原生支援的 ETW (Event Tracing for Windows)，雖然寫起來比較麻煩，但這是邁向專業 EDR 的必經之路。

#  Detection & Correlation：大腦的判斷邏輯

把資料收進來後，總不能全部都寫死在 C++ 程式碼裡，那樣改規則會瘋掉。比較好的設計是弄一個 Data-driven 的 JSON 規則引擎。你的規則檔裡只要定義好欄位路徑（例如 proc.image 或 target.image）和比對方式（equals_any、regex_any 之類的），程式負責讀 JSON 就好。

但單一事件通常誤報率很高，這時候就需要引入**關聯分析 (Correlation)**。EDR 需要有狀態的概念，例如：單純有人讀取 `lsass.exe` 可能還好，但如果是「高權限的 Process Access」緊接著發生「CreateRemoteThread」，那這個 Sequence 就非常可疑，很有可能是注入攻擊，這就是 Correlation Engine 該做的事。

# Post-Exploitation Scanning：當行為分析不夠用時

有些高階的攻擊手法（像是 Process Hollowing 或 Reflective DLL Injection）光看事件日誌可能看不出破綻，這時候就需要記憶體掃描。但這是相當吃資源和性能的操作，一般來說在需要的時候才會去掃描，只有當前面的規則引擎覺得某個 PID 很這可疑的時候，才去觸發深層掃描。

記憶體掃描的話目前有開源專案 PE-sieve 和 HollowsHunter，這些工具能幫你比對記憶體中的 PE Header 和磁碟上的是不是長得不一樣，或是抓出被替換掉的 Payload，甚至你也可以整合 YARA 來掃描特定的特徵碼。這樣做既能保持效能，又能有深度的偵測能力。

# Kernel Driver：前往 Ring 0 的世界

等到 User Mode 玩得差不多了，最後的大魔王就是 Kernel Driver。這時候你就需要透過 WDK 寫一個 KMDF Driver 來監控。在 Kernel 層，我們會用 `PsSetCreateProcessNotifyRoutineEx` 來監控行程建立，或是用 `ObRegisterCallbacks` 來審核 Handle 的權限。這些 Kernel 事件可以透過 IOCTL 的方式傳回給 User Mode 的 Agent 處理。更進階一點，你甚至可以在 Driver 層實作**Enforcement**，例如當不明的程式試圖獲取受保護 Process 的 Handle 時，直接在 Kernel 把它擋下來或是剝奪它的寫入權限，這就是主動防禦的雛形了。


# References

- [為美好的 Windows 獻上 ETW (zeze)](https://ithelp.ithome.com.tw/articles/10279093)

