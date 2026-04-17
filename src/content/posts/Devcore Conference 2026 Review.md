---
title: Devcore Conference 2026 Review
published: 2026-03-28
description: ''
image: ''
tags: [InfoSec]
category: 'Note'
draft: false 
lang: ''
---

# Introduction

去年忘記報了，今年來湊熱鬧 @@

## 紅隊的 AI 視界：攻防演練中的 LLM

講者是紅隊演練的隊長，大概是說最近和企業合作有發現引入了 LLM 的服務，但除了 LLM 本身的安全議題如 Prompt Injection, Jailbreak 之外，和其相連的元件也可能成為攻擊面。例如像是網站上的 AI 客服機器人，有輸入框的話一樣會有傳統 Injection 的風險。此外如果員工會透過機敏資訊來和企業內部的 LLM 聊天，對話紀錄也是可被竊取的 credential

## 什麼！原來連 Wi-Fi 不需要密碼！？

- WEP, WPA 又老又不安全
- WPA2 最多人用 (75%)、WPA3 目前最新最安全
- WPA-WPA3 驗證是使用 EAPoL 進行驗證和金鑰交換，而未驗證的使用者只能傳送 EAPoL
- WiFi Driver 和廠商基於 protocol 的客製化協議都是相當大的攻擊面
- 高安全性的密碼不一定能夠保護到你的 AP
- 最好定期追蹤設備韌體的更新
- 網段隔離，減少任意 Data Frame 注入帶來的影響

## 獵捕到偵測的最後一哩路

- 大量的資料只有少部分會偵測
- 確定安全 -> 需要調查的資料 -> 確定被偵測
- 資料用於調查：識別因素和 root cause、威脅搜尋和溯源、感染案例等 -> 請求偵測/白名單
- 偵測 red team 送入的記憶體 exploit，會用 hased-assembly 進行比對
- 善加利用低威脅的偵測
- 資源耗損作戰
- 增加垃圾代碼
- 從流程上做偵測 (storyline)
    - group 是一組 processes 的集合，是追蹤和偵測的基本單位
    - 威脅實際上是一個 group 

## Turning Browser Features into Exploits

- 瀏覽器核心的 feature，會永遠存在
- 有 2 requests 要送出，但只有 1 socket 可用，優先級看 port > scheme > host
- 可以從用起來理所當然的函式實作細節進行研究，可能會發現奇怪的東西

## Playing Cat and Mouse with WAF: the React2Shell Vercel CTF

 - Vercel 為了緩解 React 大洞而開的 bug bounty，一個洞 $5000
 - 感覺像在打網站漏洞，是 "exploit -> patch" 的循環

# Afterword

太酷了看到一堆不同年齡層的人，第一場提到目前企業引入 AI/RAG 等等的元件本身有可能被利用，但也不能忽視網站本身的漏洞。也知道 WiFi 協議的規範，但怎麼實作也是很重要的工程。總之收穫了很多觀念和攻擊思維，往後再打比賽或看漏洞可能也會用到。

