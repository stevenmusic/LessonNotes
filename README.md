# LessonNotes 琴課小簿

音樂老師與家長共用的一本聯絡簿。這堂課練到哪裡、下週要帶什麼譜、家裡練得順不順,
寫在同一頁上,不必再靠訊息翻來翻去。

開啟方式:用瀏覽器打開 `index.html`(單一檔案,沒有建置步驟、沒有相依套件,
只有 Google Fonts 與 Firebase 兩個 CDN)。

線上版:<https://stevenmusic.github.io/LessonNotes/>

## 目前這一版

**只有署名(Google 登入)。** 署名之後這一頁會顯示你的名字、聯絡信箱與簿本編號
(Firebase 的 `displayName` / `email` / `uid`),以及一顆「闔上簿本」。
課堂紀錄、家裡的練習、上課日與堂數還在寫,首頁上以「接下來要裝訂的幾頁」列著。

## 設定

`index.html` 裡的 `firebaseConfig` 六個欄位預設是 `PASTE_HERE`。
沒填完的時候整頁會換成「簿本還沒裝訂完成」,連 Firebase 都不會初始化 ——
與其讓使用者按下去收到 `auth/invalid-api-key`,不如直接說還沒設定好。

1. Firebase 主控台建一個專案,加一個 Web App,把它給的六個值貼進 `firebaseConfig`
2. Authentication → Sign-in method → 開啟 **Google**
3. Authentication → Settings → Authorized domains 加入 `stevenmusic.github.io`
   (本機測試再加 `localhost`)

`apiKey` 放在前端是 Firebase 的正常用法,它不是密鑰,擋門的是上面第 3 步的網域清單
與之後的安全規則。

## 技術

- 單檔 HTML,inline CSS/JS,CDN-only,沒有建置步驟(與其他五個工具同一套部署方式)
- Firebase v10 modular ESM,直接吃 gstatic 的 CDN
- 用**動態** `import()` 載 Firebase,不是頂層 `import`。頂層 import 抓不到的時候
  整個模組都不會執行,連語言與主題切換都會一起死掉;改成動態載入之後,
  CDN 連不上只會關掉署名入口,這一頁其他部分照常能看能切
- 主題與語言在載 Firebase **之前**先套用,不然存了英文偏好的使用者會先看到一整頁中文

## 版面與配色

版面(頂欄、品牌名字級、按鈕、圓形語言/主題鈕、頁尾、斷點、`data-i18n` 那一套)
與 ScrollScore / SightScore / HarmonyMap / LoudMaster / VinylVault 逐項相同。

配色用的是**同一組語意變數換一套值**,跟 HarmonyMap 的淺色主題同一個做法,
版面與元件一行都不用改寫:

| 變數 | 深色(其他五個工具的預設) | 紙張色(這一頁的預設) |
| --- | --- | --- |
| `--bg` | `#0C0A07` | `#F4ECDA` |
| `--panel` | `#161209` | `#EFE0C4` |
| `--ivory`(字) | `#F4ECDA` | `#3A2F22` |
| `--gold`(字與圖示) | `#C9A24B` | `#856418` |
| `--gold-edge`(只畫線) | `#C9A24B` | `#C9A24B` |

金色在紙張色底下拆成兩個角色:`#C9A24B` 在 `#F4ECDA` 上只有約 1.9:1,
當文字讀不動,所以文字與圖示壓深成 `#856418`(約 4.5:1),
原本的 `#C9A24B` 留給框線與內頁橫線這種不用讀的地方。

**紙張色是這一頁的預設,深色是整個工具箱的預設。** 兩者共用 `sm_theme` 這把
localStorage key(與其他五個工具同一把),所以共用的是「使用者的選擇」,
不是「還沒選之前的預設」——在任何一個工具切過主題,六個工具都跟著走。

## 文案

寫成紙本聯絡簿的口氣:**署名**、**翻開**、**闔上簿本**、**簿本編號**。
不要出現「連線成功」「使用者編號」「認證失敗」這類系統用語 ——
這一頁是給家長看的。Firebase 的錯誤代碼一律不顯示在畫面上,
`errKeyFor()` 把它們對應成一句人話(那扇窗被關上了、瀏覽器把那扇窗擋住了……),
對不到的走 `errGeneric`。

## 中英雙語

只維護一份 DOM,靠 `data-i18n` / `data-i18n-attr` / `data-i18n-html` + `applyLanguage()`
換字。動態寫進畫面的字(扉頁欄位、錯誤紙條、按鈕的忙碌狀態)不吃 `data-i18n`,
`applyLanguage()` 結尾要重畫一次。`sm_lang` 與其他五個工具共用。

**英文介面不附中文對照。** 新增文案時 `zh` 與 `en` 兩邊都是必填。

## 已知踩過的坑

- `hidden` 屬性靠的是瀏覽器預設樣式表的 `display:none`,任何一條寫了 `display` 的
  class 規則都會蓋過它(`.slip` 是 `display:flex`)。樣式表裡要有
  `[hidden]{display:none !important}`,不然空的錯誤紙條會一直留在畫面上
- 主題要在 `<body>` 開頭用一小段 classic script 先定下來。等到模組腳本才套用的話,
  預設紙張色的這一頁會先閃一下深色。那一段的判斷要跟 `themeIsLight()` 一模一樣
- 署名成功後不要自己換畫面,交給 `onAuthStateChanged` ——
  兩邊都寫會出現換兩次的閃動
