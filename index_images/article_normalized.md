# 40秒陣亡：當Google第一名親手把後門遞給你

## 摘要

一台剛拆箱兩天的MacBook Air M5，在敲下一行「官方網域」複製來的安裝指令後，四十秒內被攻陷。奇怪的是，那個網域完全真實，連防毒引擎當下都判定乾淨。真正被攻破的不是防火牆，而是「搜尋排名第一」與「網址看起來對」這兩件事在人心中建立起來的信任。這起案例背後，是一整條2026年上半年正在多國同步發生的攻擊產業鏈：SEO投毒、Claude與ChatGPT分享連結遭冒用、macOS Keychain竊密軟體大量繁殖。本文拆解攻擊如何運作、對應到哪些國際資安框架、目前有哪些已知變種，並給出一份給重度依賴AI協作開發者（vibe coding者）的具體防護清單。

## 兩天心血，四十秒歸零

新機到貨的興奮感通常只夠維持幾天。把常用軟體一一裝好，最後只剩一項：Claude Code。順手打開瀏覽器，搜尋「claude code 安裝」，第一筆結果的網域寫著`claude.ai`，一個每天都在用、閉著眼睛也會信任的名字。點進去，頁面排版正常，甚至還留著一行看起來像官方教學的安裝指令。複製，貼上終端機，按下Enter。

沒有跳出警告，沒有防毒攔截，只有macOS的Keychain視窗一次又一次跳出來要密碼。那個瞬間，警覺神經才終於啟動——低頭一看網址列，那串字元是Base64編碼，不是正常的路徑。斷網、關機、進入Recovery模式抹除整顆磁碟，這一連串動作在四十秒內全部完成。

如果只看結果，這像是一次教科書等級的事件應變：快、狠、準。但真正值得深究的，是「為什麼一個資安意識足夠敏銳、平常也儘量不讓瀏覽器記住密碼的使用者，還是在防毒軟體毫無反應的情況下，把手伸向了那行指令」。答案跟技術漏洞無關，跟人心的信任機制有關。

## VirusTotal為什麼測不出來

把那段Base64解碼之後的網址送去VirusTotal檢測，結果是乾淨的。這不是防毒引擎失靈，而是攻擊者根本沒有偽造任何東西。

傳統的釣魚攻擊仰賴假網域：把`claude.ai`拼成`claude-ai.co`或`ciaude.ai`，靠視覺相似度矇混。這類手法有天生的破綻——網域信譽資料庫看得到、瀏覽器警告攔得住、使用者只要多看一眼網址列就能識破。但這起事件裡，網址列上寫的貨真價實就是`claude.ai`。攻擊者用的不是偽造，而是**借用**：把一段包著惡意安裝指令的對話，用Claude官方的分享連結（Share Link）功能發布出去。網域是真的，伺服器是官方的，SSL憑證正確無誤，唯獨對話內容是攻擊者自己編寫的。

再往上一層看，這個分享連結能被搜尋到，靠的是SEO投毒（SEO Poisoning）與惡意廣告（Malvertising）——花錢買廣告或操作搜尋演算法，讓一個「看起來像官方安裝教學」的頁面擠進Google搜尋結果第一名，排在真正的官方文件前面。查證顯示，直到隔天，這個偽造頁面依然穩坐搜尋結果榜首。

這暴露出現行資安防護模型的一個結構性死角：幾乎所有信任判斷都建立在「網域是否合法」這個單一維度上，無論是使用者的直覺、瀏覽器的憑證檢查，還是威脅情報系統的網域黑名單。但當攻擊者不再需要偽造網域，而是直接寄生在合法服務的使用者生成內容（分享連結、共享對話、公開協作頁面）裡，這套以網域為核心的信任模型就出現了系統性破口。2026年上半年，Trend Micro、Push Security、Bitdefender、Malwarebytes、Huntress等多家獨立威脅情報廠商，分別記錄到幾乎相同的手法被用來冒充Claude Code與ChatGPT的分享頁面——這不是單一事件，而是一種被驗證有效、正在擴散的攻擊模式。

<!--MERMAID_KILLCHAIN-->

## 一條指令背後的三段式劫持

把整起事件攤開來看，其實是三個階段緊密咬合的鏈條。

第一階段是**信任劫持**：攻擊者在Claude.ai建立一段對話，把惡意安裝指令寫進去，產生分享連結，再透過SEO投毒或購買廣告把它推上搜尋結果前列。使用者搜尋安裝方式，看到熟悉的網域，點進去，複製貼上——這個動作本身就是攻擊鏈裡「使用者執行」的關鍵一步，在資安框架裡有專門的名稱與編號可以對應。

第二階段是**混淆與繞過**：貼上的指令並非明文，而是經過Base64編碼包裝，一般人肉眼難以判讀內容，自然也就跳過了「先看內容再執行」這道最基本的防線。指令執行後，會先解碼出真正的下載位址，再取得後續腳本，整個過程刻意避開瀏覽器的下載掃描與macOS Gatekeeper的簽章驗證，因為使用者是在終端機裡手動貼上執行，而不是雙擊一個被隔離標記的下載檔案。

第三階段是**憑證洗劫**：腳本開始有系統地搜刮macOS Keychain裡的帳號密碼、主流瀏覽器記憶的密碼與Cookie、開發者慣用的設定檔（`~/.ssh`、`~/.aws`、`~/.kube`等）、通訊軟體資料，甚至針對加密貨幣錢包應用程式（如Ledger、Trezor）進行置換攻擊，把使用者導向偽造介面騙取私鑰，最後把整包資料打包上傳。

這三個階段可以逐一對應到MITRE ATT&CK這套全球通用的攻擊技術分類框架，讓「這次到底發生了什麼」從一段驚魂敘述，變成一張可以被複製、被偵測、被防禦的技術地圖。

<!--MERMAID_ATTCK-->

| ATT&CK戰術 | Technique | ID |
|---|---|---|
| Resource Development | SEO Poisoning | T1608.006 |
| Resource Development | Malvertising | T1583.008 |
| Execution | Malicious Copy and Paste | T1204.004 |
| Execution | Unix Shell | T1059.004 |
| Defense Evasion | Deobfuscate/Decode Files or Information | T1140 |
| Credential Access | Credentials from Password Stores: Keychain | T1555.001 |
| Credential Access | Unsecured Credentials In Files | T1552.001 |
| Credential Access | Steal Web Session Cookie | T1539 |
| Collection | Data from Local System | T1005 |
| Exfiltration | Exfiltration Over Web Service | T1567 |

事後的處置動作同樣能對號入座：斷網、關機、抹除磁碟、變更密碼、檢查登入裝置——這一連串反射動作，精準對應到NIST事件應變框架（SP 800-61）裡的「圍堵」與「根除」步驟。換句話說，這名使用者在毫無準備的情況下，憑直覺做出了教科書等級的應變，但如果一開始就有柵欄擋在指令執行之前，這整套應變流程根本不必上場。

## 這不是特例，是一整條產業鏈

最容易讓人低估這起事件的想法是「運氣不好，遇到針對性攻擊」。查證結果恰恰相反。

過去幾個月，至少十起獨立記錄的活動，手法與這起案例高度重疊：有攻擊者複製Claude Code的官方安裝頁面再投放Google廣告；有人偽造ChatGPT與Grok的對話介面誘導受害者執行指令，執行前還會用系統指令靜默驗證使用者輸入的密碼是否正確，確保竊來的密碼真實可用；有專門針對macOS系統工具的偽裝頁面，以「終端機貼上一行指令」為誘餌，繞過蘋果原生的Gatekeeper簽章檢查。這套誘導使用者手動貼上指令執行的手法，在威脅情報圈已經有了專屬名稱——ClickFix。

而背後扛起竊密工作的惡意程式，也早已形成一個活躍的地下市場。Atomic macOS Stealer在2025年一度佔了近四成的macOS威脅偵測量，能解開Keychain、掃遍瀏覽器與加密錢包；它的分支Poseidon與重製版Odyssey鎖定上百種瀏覽器擴充功能；Cthulhu用Go語言撰寫，直接在Telegram上以訂閱服務的形式出租給其他犯罪者使用；Banshee則專門把竊得的資料回傳到境外伺服器。這些名字聽起來像是獨立的孤例，實際上彼此借用程式碼、共享銷售通路，是一整條分工明確的犯罪供應鏈。

面對這樣的態勢，美國聯邦調查局（FBI）、聯邦貿易委員會（FTC）與網路安全暨基礎設施安全局（CISA）都已各自發出正式警示，內容直指搜尋廣告詐騙、假冒AI工具軟體廣告、以及軟體供應鏈的信任驗證問題——這代表相關單位已經把這類攻擊視為一種需要制度性應對的公眾風險，而不是零星個案。

## 給重度依賴AI協作者的防護清單

如果日常工作習慣是用口語快速對AI下指令、講求速度與流暢（也就是所謂的vibe coding），觀念上的警覺其實遠遠不夠，因為警覺會在疲憊、趕時間、身處陌生環境時失靈——四十秒的災情就是明證。真正能撐住的，是預先建好、不依賴當下判斷力的技術柵欄。

第一層是執行環境隔離。任何不明來源的一鍵安裝指令，原則上都不該在裝有正式憑證的工作機上直接執行，而是先丟進一台乾淨的虛擬機或可拋棄的容器裡跑一次。Claude Code官方本身也提供沙盒機制，透過檔案系統隔離與網路對外連線的白名單管控，大幅降低誤觸權限的機會；使用開發容器時，也不該把主機的SSH金鑰目錄整個掛載進去。

第二層是指令執行前的強制檢查習慣。任何`curl`接管線直接執行的安裝方式，養成先把內容存下來看過一遍的習慣，而不是讓終端機直接吞下去；看到經過Base64或十六進位編碼的網址與內容，先解碼再判斷；更重要的是隨時區分，眼前這段指令究竟是從官方文件複製來的，還是從某個分享頁面上臨時冒出來的——這正是本次事件唯一的破口所在。

第三層是憑證與金鑰隔離。系統層級的密碼管理工具與瀏覽器記憶密碼功能能少用就少用，一旦真的被植入竊密程式，損害範圍才會被限制在最小範圍；像SSH金鑰、雲端服務憑證這類高價值目標，更適合搬到需要額外驗證才能解鎖的位置存放。

第四層是AI代理人本身的權限最小化。當手上同時串接多個MCP工具、讓AI能夠直接操作日誌系統、防火牆、虛擬化平台時，每一個工具的授權範圍都值得重新檢視一次是否收得夠緊——這正是國際上針對LLM與自主代理人風險框架裡，反覆被提及的「過度授權」問題。同時，任何一台新機器，裝設端點偵測與防護軟體的優先順序都該排在其他工具之前，而不是等出事後才發現保護傘還沒撐開。

第五層是網路層防護。端點防毒軟體雖然不是萬靈丹，但在多起同類案例裡確實攔下過第一階段的惡意腳本下載；搭配防火牆或DNS層過濾，擋掉已知的竊密程式對外連線位址，能在攻擊鏈條的更早一段就切斷它。

## 總結

這起事件真正的教訓，不是「要小心不明連結」這種老生常談，而是揭露了一個更根本的問題：當攻擊者放棄偽造網域，轉而寄生在合法平台的分享機制裡，傳統上依賴「網址看起來對不對」的直覺防線，已經徹底失效。SEO投毒讓假頁面登上搜尋榜首、共享連結讓網域信譽形同虛設、Base64混淆讓內容審查失去意義——三層手法疊加起來，足以讓一名資安意識充分的使用者，在四十秒內就從新機開箱走到全機抹除。這不是孤例，而是2026年上半年一整條已被多家威脅情報機構獨立證實的攻擊產業鏈的其中一環。真正能夠對抗這種攻擊的，從來不是事後更快的反應速度，而是預先架好、不依賴當下判斷力的技術柵欄——沙盒隔離、憑證分離、權限最小化，把信任的成本，從「相信一眼看到的網址」，轉移到「不管網址是誰，先假設指令有問題」。

## 參考連結彙整

### MITRE ATT&CK 官方技術對應

- SEO Poisoning (T1608.006): https://attack.mitre.org/techniques/T1608/006/
- Malvertising (T1583.008): https://attack.mitre.org/techniques/T1583/008/
- Malicious Copy and Paste (T1204.004): https://attack.mitre.org/techniques/T1204/004/
- Unix Shell (T1059.004): https://attack.mitre.org/techniques/T1059/004/
- Deobfuscate/Decode Files or Information (T1140): https://attack.mitre.org/techniques/T1140/
- Credentials from Password Stores: Keychain (T1555.001): https://attack.mitre.org/techniques/T1555/001/
- Unsecured Credentials In Files (T1552.001): https://attack.mitre.org/techniques/T1552/001/
- Steal Web Session Cookie (T1539): https://attack.mitre.org/techniques/T1539/
- Data from Local System (T1005): https://attack.mitre.org/techniques/T1005/
- Exfiltration Over Web Service (T1567): https://attack.mitre.org/techniques/T1567/

### 同類攻擊案例與威脅情報

- InstallFix（假冒Claude Code安裝頁）— Push Security: https://pushsecurity.com/blog/installfix
- 假冒Claude Code Google Ads(Amatera/ACR Stealer)— Bitdefender Labs: https://www.bitdefender.com/en-us/blog/labs/fake-claude-code-google-ads-malware
- Fake Claude Code Install Pages — Malwarebytes: https://www.malwarebytes.com/blog/news/2026/03/fake-claude-code-install-pages-hit-windows-and-mac-users-with-infostealers
- MacSync Stealer 投放鏈 — CloudSEK: https://www.cloudsek.com/blog/macsync-stealer-seo-poisoning-and-clickfix-based-macos-malware-delivery-chain
- AI-Poisoning（假冒ChatGPT/Grok對話頁）— Huntress: https://www.huntress.com/blog/amos-stealer-chatgpt-grok-ai-trust
- Google Ads + claude.ai共享聊天投放MacSync — CyberSecurityNews: https://cybersecuritynews.com/macos-malware-leverages-google-ads/
- LLMShare：濫用ChatGPT/Claude分享功能 — Push Security: https://pushsecurity.com/blog/llmshare-malvertising-campaign
- ChatGPT Share Links假故障頁 — BleepingComputer: https://www.bleepingcomputer.com/news/security/chatgpt-share-links-abused-to-host-fake-outage-pages-to-deliver-malware/
- ClickFix假冒macOS系統工具 — Microsoft Security Blog: https://www.microsoft.com/en-us/security/blog/2026/05/06/clickfix-campaign-uses-fake-macos-utilities-lures-deliver-infostealers/
- 200+惡意Google Ads冒充知名軟體 — Cybernews: https://cybernews.com/security/hackers-spread-mac-infostealer-using-google-ads/
- Trend Micro:Claude.ai Shared Chat被濫用： https://www.trendmicro.com/en_us/research/26/f/claudeai-shared-chat-abused-in-malvertising.html

### macOS Infostealer 家族

- Atomic macOS Stealer(AMOS)— Darktrace: https://www.darktrace.com/blog/atomic-stealer-darktraces-investigation-of-a-growing-macos-threat
- Poseidon Stealer — Red Canary: https://redcanary.com/blog/threat-intelligence/atomic-odyssey-poseidon-stealers/
- Cthulhu Stealer — Unit 42: https://unit42.paloaltonetworks.com/macos-stealers-growing/
- Banshee Stealer — Intego: https://www.intego.com/mac-security-blog/mac-stealer-malware-market-surges-with-banshee-and-new-threat-actors/

### 官方規範與框架

- FBI IC3 PSA260618（惡意流量分配系統）: https://www.ic3.gov/PSA/2026/PSA260618
- FBI IC3 PSA221221（搜尋廣告冒充品牌）: https://www.ic3.gov/PSA/2022/PSA221221
- FTC消費者警示（假AI/軟體廣告）: https://consumer.ftc.gov/consumer-alerts/2023/04/ads-fake-ai-other-software-spread-malicious-software
- CISA供應鏈攻擊防禦： https://www.cisa.gov/resources-tools/resources/defending-against-software-supply-chain-attacks
- NIST SP 800-61 Rev.3（事件應變）: https://csrc.nist.gov/pubs/sp/800/61/r3/final
- NIST Cybersecurity Framework 2.0: https://www.nist.gov/cyberframework
- Claude Code官方安裝文件： https://code.claude.com/docs/en/setup
- Claude Code Devcontainer文件： https://code.claude.com/docs/en/devcontainer
- Anthropic Claude Code Sandboxing工程部落格： https://www.anthropic.com/engineering/claude-code-sandboxing
- OWASP Top 10 for Agentic Applications 2026: https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/
- OWASP Top 10 for LLM Applications: https://genai.owasp.org/llm-top-10/
