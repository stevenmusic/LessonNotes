# LessonNotes 專案規則

## 技術棧
- 單檔 HTML,inline CSS/JS,CDN-only,沒有建置步驟
  (跟 ScrollScore / SightScore / HarmonyMap / LoudMaster 同一套部署方式)
- Firebase v10 modular ESM,從 gstatic 的 CDN 動態載入
- 沒有任何相依套件,只有 Google Fonts 與 Firebase 兩個外部來源

## 絕不能做的事
- 不要把單檔拆成多檔案,除非我明確要求
- 不要把 Firebase 改回頂層 `import`。頂層 import 抓不到時整個模組都不會執行,
  連語言與主題切換都會一起死掉。維持動態 `import()` + try/catch,
  載不到只關掉署名入口
- 不要在 `applyTheme()` / `applyLanguage()` 之前 await Firebase。
  順序反過來的話,存了英文偏好的使用者會先看到一整頁中文
- 不要拿掉 `<body>` 開頭那段 classic script。這一頁預設紙張色,
  等到模組腳本才套主題會先閃一下深色。它的判斷要跟 `themeIsLight()` 一模一樣,
  兩邊要一起改
- 不要拿掉 `[hidden]{display:none !important}`。`.slip` 是 `display:flex`,
  會蓋掉 `hidden` 屬性,少了這條會留下一張空的錯誤紙條(踩過)
- 頂欄品牌名的 CSS 與 ScrollScore 逐項相同(字型、clamp 字級、字距、各斷點),
  改之前先確認兩邊還是一致
- 圖示一律用 ScrollScore 那一套線稿 SVG(24 格線、`stroke-width:2`、圓端點、
  `stroke:currentColor`)。唯一的例外是登入鈕上 Google 官方的四色 G,
  那是品牌規範要求的,不要改色也不要換成線稿
- Firebase 的錯誤代碼不要顯示在畫面上。一律經過 `errKeyFor()` 換成一句人話,
  對不到的走 `errGeneric`

## 響應式斷點(與其他四個專案共用,不要自己多開)
```
max-width:600 / 380 / 340     pointer:coarse     prefers-reduced-motion:reduce
```
這一組是五個專案共用的,任何一邊要動都要一起看。需要「隨尺寸連續變化」的東西
一律用 clamp 算,不要為它新增斷點。

## 字型與字級
- Google Fonts 那一行與其他四個工具**逐字相同**:
  `Noto+Serif+TC:wght@600;700&family=Noto+Sans+TC:wght@400;500;700`
- `--fs-*` 代幣(2xl 26 / xl 19 / lg 16 / md 14 / sm 13 / xs 12 / 2xs 11)也相同
- 標題用 Serif、內文用 Sans
- **字級也是字型的一部分**。頁面標題是 `clamp(24px, 7vw, 32px)`,照 LoudMaster 的值。
  這裡原本開到 40px,比任何一個工具最大的 serif 都大,並排看就像換了一套字(踩過)

## 配色
同一組語意變數換一套值,版面與元件一行都不用改寫(跟 HarmonyMap 的淺色主題同一個做法)。

- 紙張色是**這一頁的**預設,深色是**整個工具箱的**預設
- 兩者共用 `sm_theme` 這把 localStorage key,所以共用的是「使用者的選擇」,
  不是「還沒選之前的預設」。在任何一個工具切過主題,五個工具都跟著走
- 金色在紙張底下拆成兩個角色:`#C9A24B` 在 `#F4ECDA` 上只有約 1.9:1,當文字讀不動,
  所以文字與圖示用 `--gold` = `#856418`(約 4.5:1),框線與內頁橫線用
  `--gold-edge` = `#C9A24B`。**新增元件時要挑對變數**,不要一律用 `--gold`

## 文案
寫成紙本聯絡簿的口氣:**署名**、**翻開**、**闔上簿本**、**簿本編號**。
不要出現「連線成功」「使用者編號」「認證失敗」「登入」這類系統用語 ——
這一頁是給家長看的,不是給工程師看的。

## 中英雙語
- 只維護一份 DOM,靠 `data-i18n` / `data-i18n-attr` / `data-i18n-html` +
  `applyLanguage()` 換字
- 動態寫進畫面的字(扉頁欄位、錯誤紙條、按鈕的忙碌狀態)不吃 `data-i18n`,
  `applyLanguage()` 結尾要重畫一次
- **英文介面不附中文對照**。新增文案時 `zh` 與 `en` 兩邊都是必填
- `sm_lang` 與其他四個工具共用

## 設定與部署
- `firebaseConfig` 六個欄位只要有一個是 `PASTE_HERE`,整頁換成「簿本還沒裝訂完成」,
  連 Firebase 都不初始化
- `apiKey` 不是密鑰,部署出去的原始碼裡本來就看得到。擋門的是 Authentication 的
  已授權網域清單(`stevenmusic.github.io` —— 只填主機名稱,不要協定也不要路徑)
- GitHub Pages 從 `main` 的根目錄部署。改完推上 main 之後要確認
  Actions 有跑出一筆 `pages build and deployment`;沒跑出來的話線上還是舊版,
  跟瀏覽器快取無關(踩過)

## 改完要確認的事
1. 兩種主題 × 兩種語言各看一遍,英文版畫面上不能有中文(語言鈕的「中」除外)
2. 320px 寬不能出現橫向捲軸
3. 四種狀態都要看:未設定、未署名、已署名、CDN 連不上
4. CDN 連不上之後切一次語言,署名鈕必須還是停用的(`unavailable` 旗標,踩過)
