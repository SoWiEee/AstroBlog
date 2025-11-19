---
title: Modern OpenGL Environment Build
published: 2025-11-17
description: ''
image: ''
tags: [CS]
category: 'Note'
draft: false 
lang: ''
---

由於 LearnOpenGL 上的資料都是舊版 (3.3) 的，所以撰寫一篇現代版本 (4.5) 的環境建置教學。

# OpenGL

* 通常在學校的計算機圖學課程會使用這個東西來實作程式
* 本身是由 Khronos 組織制定及維護的規範，是一個 spec
* 早期是立即渲染模式，但是效率太低，從 3.2 之後鼓勵開發者在 Core-profile 下進行開發
* 由於全域狀態的存在，早期 OpenGL 程式碼的正確性難以審核，迫使程式設計師不斷考慮 API 指令模糊不清的 scope
* 在 OpenGL 4.5 之後解決或緩解了許多此類問題

# Environment Setup

這篇文章會教你建置環境到繪製出三角形，這邊使用 Visual Studio 2022 作為 IDE，並搭配 GLAD, GLFW 來建置，此外我們也會常常需要做數學運算，這邊也順便下載 GLM。

1. 到 [GLFW](https://www.glfw.org/download.html) 網站找到自己的 OS 及 Arch，就能下載預編譯好的函式庫 `glfw3.lib`
2. 到 [GLAD](https://glad.dav1d.de/) 網站選擇開發語言 (C/C++)、OpenGL 版本 (4.5)、使用 Core Profile，勾選 Generate a loader 之後把壓縮檔 `glad.zip` 下載下來
3. 打開 VS 建立一個空白專案，整理剛剛下載好的檔案在 include 目錄中，結果如下：

```
include/
│
├── glad/
│   ├── include/
│   └── src/
├── glfw/
│   ├── include/
│   └── lib-vcc2022/
└── glm
    └── glm/
```

4. 這邊不使用 CMake 來管理，而是用 Visual Studio 內建的管理工具。首先到專案＞屬性

- VC++目錄＞包含目錄＞加入 `glm/`
- C/C++＞一般＞加入 `glfw\include`、`glad\include`、`glm`
- Linker＞其他函數庫目錄＞加入 `glfw\lib-vc2022`
- Linker＞輸入＞加入 `glfw3.lib`、`opengl32.lib`

5. 這樣就完成基本的環境建置了，執行起來後應該會動 XD

# Debug/Profiling Tool

以前寫 C/C++ 的時候有 gcc, perf 可以用，對於圖學程式也是有相應的工具。

## RenderDoc

* [網站在這](https://renderdoc.org/)
* [使用說明](https://renderdoc.org/docs/getting_started/quick_start.html)


## Nvidia Nsight Graphic

* N 卡專用的 debugger, profiler
* [使用說明](https://docs.nvidia.com/nsight-graphics/UserGuide/index.html)


