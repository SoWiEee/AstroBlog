---
title: Flare-On 2024 Writeup
published: 2025-08-07
description: ''
image: ''
tags: [CTF, InfoSec]
category: 'Writeup'
draft: true 
lang: ''
---

# Introduction

今年想打打看 Flare-on 來加強自己的逆向能力，所以這篇是紀錄 2024 考古題的 writeup。


# Challenge 1 - frog

打開檔案會看到 `frog.exe`、`frog.py`，這題有附使用說明，使用 `python frog.py` 來啟動遊戲，記得要裝 pygame。接著會看到視窗如下：


可以使用 W, A, S, D 移動青蛙，但由於沒有能到達終點的路，顯然需要做一些調整才能到達終點。

查看 `frog.py` 可以看到方塊是寫死的，可以註解程式碼來把方塊移除，然後就能走到終點。


# Challenge 2 - Checksum

可以直接執行 `checksum.exe`，會要你回答幾題數學題目，最後輸入 Checksum。

經過 IDA decompile 之後可以發現是用 Golang 寫的，算是勉強可讀。那來看看 `main()` 在做甚麼，可以搜尋 Checksum 字串來找到之後是怎麼做檢查的：


在 `main.a()` 是接受 ChaCha20 解密資料產生的 SHA256 hash，接著和 xor_key_FlareOn2024( FlareOn2024），然後進行 base64 編碼之後比對是否吻合。

```txt
v20 = checksum_xored_array;
checksum_xored_base64 = encoding_base64__ptr_Encoding_EncodeToString(
                        runtime_bss,
                        checksum_xored_array,
                        a2,
                        a2,
                        a5,
                        (_DWORD)v10,
                        v11,
                        v12,
                        v13,
                        v23,
                        v24,
                        v25);
if ( v20 == 88 )
    return runtime_memequal(
            checksum_xored_base64,
            "cQoFRQErX1YAVw1zVQdFUSxfAQNRBXUNAxBSe15QCVRVJ1pQEwd/WFBUAlElCFBFUnlaB1ULByRdBEFdfVtWVA==",
            88LL);
else
    return 0LL;
```

可以寫個腳本來推算出原本的 checksum 為 `7fd7dd1d0e959f74c133c13abb740b9faa61ab06bd0ecd177645e93b1e3825dd`

最後可以去看 `C:\Users\[CURRENT_USER]\AppData\Local\REAL_FLAREON_FLAG.JPG`，flag 印在上面了。

# Challenge 3 - aray

題目是一個 `aray.yara` 的檔案，我完全沒看過 yara rule 真實的樣貌，只知道裡面可以寫一些規則來判別惡意程式。這邊可以看到在 `chndition:` 下面有一堆擠在一起的判斷式，我們可以把它攤開來看總共有 500 多個 condition，以下只節錄一部分（猜測只會用到一部分）。

首先是檔案大小要是 85 bytes，並且 MD5 hash 要是 b7dc94ca98aa58dabb5404541c812db2，此外可以過濾掉大於小於這種判斷，只留下有**精確判斷式**的部分，這樣只要專心處理剩下的 38 個判斷式就好。

```
filesize == 85 
hash.md5(0, filesize) == "b7dc94ca98aa58dabb5404541c812db2" 
uint8(58) + 25 == 122  
uint32(52) ^ 425706662 == 1495724241 
uint32(17) - 323157430 == 1412131772 
hash.crc32(8, 2) == 0x61089c5c 
hash.crc32(34, 2) == 0x5888fc1b 
uint8(36) + 4 == 72 
uint8(27) ^ 21 == 40 
uint32(59) ^ 512952669 == 1908304943 
uint8(65) - 29 == 70 
uint8(45) ^ 9 == 104 
uint32(28) - 419186860 == 959764852 
uint8(74) + 11 == 116 
hash.crc32(63, 2) == 0x66715919 
hash.sha256(14, 2) == "403d5f23d149670348b147a15eeb7010914701a7e99aad2e43f90cfa0325c76f" 
hash.sha256(56, 2) == "593f2d04aab251f60c9e4b8bbc1e05a34e920980ec08351a18459b2bc7dbf2f6" 
uint8(75) - 30 == 86 
uint32(66) ^ 310886682 == 849718389 
uint32(10) + 383041523 == 2448764514 
uint32(37) + 367943707 == 1228527996 
uint8(2) + 11 == 119 
hash.md5(0, 2) == "89484b14b36a8d5329426a3d944d2983" 
uint32(46) - 412326611 == 1503714457 
hash.crc32(78, 2) == 0x7cab8d64 
uint8(7) - 15 == 82 
uint32(70) + 349203301 == 2034162376 
hash.md5(76, 2) == "f98ed07a4d5f50f7de1410d905f1477f" 
uint32(80) - 473886976 == 69677856 
uint32(3) ^ 298697263 == 2108416586 
uint8(21) - 21 == 94 
uint8(16) ^ 7 == 115 
uint32(41) + 404880684 == 1699114335 
hash.md5(50, 2) == "657dae0913ee12be6fb2a6f687aae1c7" 
uint8(26) - 7 == 25 
hash.md5(32, 2) == "738a656e8e8ec272ca17cd51e12f558b" 
uint8(84) + 3 == 128 
uint8(23) & 128 == 0 
```


# Challenge 4 - Meme Maker 3000


可以看到是一個經過混淆的 js 檔案，可以直接丟[解混淆的工具](https://github.com/relative/synchrony)，結果如下：

```javascript
const a0c = [
    'When you find a buffer overflow in legacy code',
    'Reverse Engineer',
    'When you decompile the obfuscated code and it makes perfect sense',
    'Me after a week of reverse engineering',
    'When your decompiler crashes',
    "It's not a bug, it'a a feature",
    "Security 'Expert'",
    'AI',
    "That's great, but can you hack it?",
    'When your code compiles for the first time',
    "If it ain't broke, break it",
    "Reading someone else's code",
    'EDR',
    'This is fine',
    'FLARE On',
    "It's always DNS",
    'strings.exe',
    "Don't click on that.",
    'When you find the perfect 0-day exploit',
    'Security through obscurity',
    'Instant Coffee',
    'H@x0r',
    'Malware',
    '$1,000,000',
    'IDA Pro',
    'Security Expert',
  ],
  a0d = {
    doge1: [
      ['75%', '25%'],
      ['75%', '82%'],
    ],
    boy_friend0: [
      ['75%', '25%'],
      ['40%', '60%'],
      ['70%', '70%'],
    ],
    draw: [['30%', '30%']],
    drake: [
      ['10%', '75%'],
      ['55%', '75%'],
    ],
    two_buttons: [
      ['10%', '15%'],
      ['2%', '60%'],
    ],
    success: [['75%', '50%']],
    disaster: [['5%', '50%']],
    aliens: [['5%', '50%']],
  },
  a0e = {
    'doge1.png':
        'data:image/png;base64, iVBORw0KGgoA...',
    'draw.jpg':
        'data:image/jpeg;base64, /9j/4AAQSkZJRgABAQA...',
    'drake.jpg':
        'data:image/jpeg;base64, /9j/4AAQSkZJRgABAQAAAQA...',
    'two_buttons.jpg':
        'data:image/jpeg;base64, /9j/4AAQSkZJRgABAQ...',
    'fish.jpg':
        'data:binary/red; base64, TVqQAAMAAAAE...',
    'boy_friend0.jpg':
        'data:image/jpeg;base64, /9j/4AAQSkZJRgABAQA...',
    'success.jpg':
        'data:image/jpeg;base64, /9j/4AAQSkZJRgAB...',
    'disaster.jpg':
        'data:image/jpeg;base64, /9j/4AAQSkZJRgAB...',
    'aliens.jpg':
        'data:image/jpeg;base64, /9j/4AAQSkZJRgABA...'
    }
function a0f() {
  document.getElementById('caption1').hidden = true
  document.getElementById('caption2').hidden = true
  document.getElementById('caption3').hidden = true
  const a = document.getElementById('meme-template')
  var b = a.value.split('.')[0]
  a0d[b].forEach(function (c, d) {
    var e = document.getElementById('caption' + (d + 1))
    e.hidden = false
    e.style.top = a0d[b][d][0]
    e.style.left = a0d[b][d][1]
    e.textContent = a0c[Math.floor(Math.random() * (a0c.length - 1))]
  })
}
a0f()
const a0g = document.getElementById('meme-image'),
  a0h = document.getElementById('meme-container'),
  a0i = document.getElementById('remake'),
  a0j = document.getElementById('meme-template')
a0g.src = a0e[a0j.value]
a0j.addEventListener('change', () => {
  a0g.src = a0e[a0j.value]
  a0g.alt = a0j.value
  a0f()
})
a0i.addEventListener('click', () => {
  a0f()
})
function a0k() {
  const a = a0g.alt.split('/').pop()
  if (a !== Object.keys(a0e)[5]) {
    return
  }
  const b = a0l.textContent,
    c = a0m.textContent,
    d = a0n.textContent
  if (
    a0c.indexOf(b) == 14 &&
    a0c.indexOf(c) == a0c.length - 1 &&
    a0c.indexOf(d) == 22
  ) {
    var e = new Date().getTime()
    while (new Date().getTime() < e + 3000) {}
    var f =
      d[3] +
      'h' +
      a[10] +
      b[2] +
      a[3] +
      c[5] +
      c[c.length - 1] +
      '5' +
      a[3] +
      '4' +
      a[3] +
      c[2] +
      c[4] +
      c[3] +
      '3' +
      d[2] +
      a[3] +
      'j4' +
      a0c[1][2] +
      d[4] +
      '5' +
      c[2] +
      d[5] +
      '1' +
      c[11] +
      '7' +
      a0c[21][1] +
      b.replace(' ', '-') +
      a[11] +
      a0c[4].substring(12, 15)
    f = f.toLowerCase()
    alert(atob('Q29uZ3JhdHVsYXRpb25zISBIZXJlIHlvdSBnbzog') + f)
  }
}
const a0l = document.getElementById('caption1'),
  a0m = document.getElementById('caption2'),
  a0n = document.getElementById('caption3')
a0l.addEventListener('keyup', () => {
  a0k()
})
a0m.addEventListener('keyup', () => {
  a0k()
})
a0n.addEventListener('keyup', () => {
  a0k()
})
```

可以發現 `a0k()` 可能是用來檢查 condition 並顯示 flag 的函數。



# Challenge 5 - sshd

題目給了一個 tar 檔，可以用 WinRAR 打開翻一翻

在 ssh_container.tar\.\var\lib\systemd\coredump 底下可以看到一個 coredump 的檔案，接著可以用 pwngdb 來 trace



