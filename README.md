# 40秒陣亡：當Google第一名親手把後門遞給你

macOS Claude Code SEO 詐騙攻擊事件深度分析。

## 線上閱讀

啟用 GitHub Pages 後，網頁版位於：<https://lushinshang.github.io/1150711_Fake_URL/>

## 內容概述

拆解一起真實發生的攻擊案例：攻擊者透過 SEO 投毒與 Google 廣告，讓一個偽裝成 Claude Code 安裝教學、實為 `claude.ai` 分享連結（Share Link）的頁面登上搜尋結果第一名。頁面內夾帶 Base64 混淆的一鍵安裝指令，受害者於 macOS 終端機執行後觸發 Keychain 竊密，四十秒內斷網、關機、抹除磁碟止血。

文章涵蓋：

- 攻擊鏈拆解與 MITRE ATT&CK 技術對應
- 2026 年上半年同類 SEO 投毒／malvertising／Claude 與 ChatGPT 分享連結濫用案例彙整
- macOS infostealer 家族（Atomic macOS Stealer、Poseidon、Cthulhu、Banshee 等）
- 官方機構（FBI、FTC、CISA、NIST）警示與框架對應
- 給重度使用 AI 協作（vibe coding）開發者的具體防護清單

原始事件當事人之姓名、社群個人檔案連結、含人臉照片皆已去識別化處理。

## 檔案結構

```
index.html              主頁面（自包含 HTML，內嵌 CSS/JS）
index_images/
  summary-banner.png        摘要資訊圖（桌面 16:9）
  summary-banner-mobile.png 摘要資訊圖（手機 9:16）
```

## 技術說明

- 響應式版面：桌面／手機均適配，Mermaid 攻擊鏈圖表依節點數與裝置寬度動態切換橫式／直式排列
- 圖表與摘要圖片皆支援點擊放大（lightbox）
- 外部 CDN 資源（highlight.js、mermaid.js）皆加上 Subresource Integrity（SRI）雜湊
- 含 Open Graph／Twitter Card 標籤，供社群分享預覽
