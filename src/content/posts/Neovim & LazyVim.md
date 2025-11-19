---
title: Neovim & LazyVim
published: 2025-07-26
description: ''
image: ''
tags: []
category: ''
draft: false 
lang: ''
---

# Introduction

Neovim 是 Vim 的發行版，提供了額外的功能；而 Lzayvim 提供一系列的功能，把 Neovim 進化成一個 IDE。

# Installtion

1. 去 Neovim github 下載
2. 可以去 `~/.config/nvim/init.vim` 查看 Neovim 的設定檔
3. 去 Lazyvim github 下載，或者 `git clone https://github.com/LazyVim/starter ~/.config/nvim`
4. 輸入指令 `nvim` 就能看到預設的介面了


## Requirement

* Neovim 版本 >= 0.9.0
* Git 版本 >= 2.19.0
* 一個 Nerd font (optional)
* GCC -> nvim-treesitter

## Directory

```
~/.config/nvim
├── lua/                    -- 所有 Lua 設定檔的核心目錄
│   ├── config/             -- 存放 Neovim 的核心設定，與外掛程式無關
│   │   ├── autocmds.lua    -- 自動化指令
│   │   ├── keymaps.lua     -- 全域鍵盤快捷鍵
│   │   ├── lazy.lua        -- LazyVim 本身的設定
│   │   └── options.lua     -- Neovim 的編輯器選項 (set ...)
│   └── plugins/            -- 存放所有外掛程式的設定（"規格"）
│       ├── spec1.lua       -- 一個外掛程式的設定檔
│       ├── ** -- (代表可以有更多檔案和子目錄)
│       └── spec2.lua       -- 另一個外掛程式的設定檔
└── init.lua                -- Neovim 的主要進入點
```

# Settings

介紹一下預設會安裝甚麼插件：

* 主題(Theme)：
  * `tokyonight.nvim`：預設的深色主題，非常美觀
  * `catppuccin/nvim`：另一套流行的主題，可自行啟用
* 狀態列(Statusline)：`lualine.nvim` 在視窗底部提供一個漂亮且資訊豐富的狀態列 (顯示模式、檔名、Git 分支等)
* 圖示(Icons)：`nvim-web-devicons` 在檔案總管、狀態列等地方顯示檔案類型的圖示
* 介面增強：
  * `noice.nvim`：將 Neovim 的命令和訊息介面變得更現代、更美觀。
  * `nvim-notify`：提供更漂亮的訊息通知系統。
  * `dressing.nvim`：美化 vim.ui.input() 和 vim.ui.select() 的介面
* 模糊搜尋(Fuzzy Finder)：`telescope.nvim` 是 LazyVim 的核心之一。可以讓您快速模糊搜尋檔案、專案中的文字、Git 提交紀錄、命令歷史等等。預設的 Space f f 就是用它來找檔案。
* 檔案總管(File Tree)：`neo-tree.filesystem.js` 是一個現代化的檔案總管，類似 VS Code 的側邊欄。可以用 Ctrl + n 開啟
* LSP (語言伺服器協定)
  * `nvim-lspconfig`：設定 LSP 的基礎。
  * `mason.nvim`：自動化管理 LSP 伺服器、DAP (除錯器)、Linter (語法檢查) 和 Formatter (格式化) 的安裝。
* 自動補全 (Autocomplete)：`nvim-cmp` 是功能強大的自動補全引擎
* 語法高亮 (Syntax Highlighting)：`nvim-treesitter` 提供更快速、更精確的語法高亮和程式碼結構分析。
  * `flash.nvim`：增強版的游標跳轉工具，可以在畫面內快速跳到指定字元
  * `comment.nvim`：方便地註解/取消註解程式碼 (預設 g c)
  * `nvim-autopairs`：自動補全成對的括號、引號等
* Git 狀態：`gitsigns.nvim` 在行號旁邊顯示該行的 Git 狀態 (新增、修改、刪除)
* 外掛程式管理器：`lazy.nvim` 是 LazyVim 的基礎，用來管理所有外掛程式的載入
* 快捷鍵提示：`which-key.nvim` 當您按下一個前導鍵 (例如 Space) 後，會彈出一個視窗提示您接下來可以按哪些鍵

## 設定方式

:::important[Principle]
不要修改 LazyVim 的核心檔案，而是建立自己的設定檔來覆蓋或擴充它！
:::

* 所有的自訂設定都應該放在 `~/.config/nvim/lua/plugins/` 目錄下

### 一、新增外掛程式





### 二、修改現有外掛程式的設定



### 三、調整 Neovim 本身的選項



## Useful Command

* `:Lazy`：開啟 Lazy.nvim 的管理介面，可以在這裡查看外掛程式狀態、更新、清理等
* `:LazyExtras`：LazyVim 提供的一個方便工具，可以用來快速啟用額外的功能模組 (例如對 rust, python, docker 等語言或工具的支援)
* `:checkhealth`：檢查 Neovim 和外掛程式的健康狀態，是除錯的好幫手


## Color & Theme

* `lua/plugins/core.lua`

```
return {
  {
    "LazyVim/LazyVim",
    opts = {
      colorscheme = "catppuccin",
    }
  }
}
```



# tmux





