---
title: "Character Encoding"
category: Math & Bit
slug: character-encoding
subtitle: 一個字元怎麼變成 bytes，UTF-8 跟 UTF-16 差在哪
date: 2026-07-17T12:00:00
---

# Character Encoding

打開 console 打三行：

```javascript
"a".length    // 1
"中".length   // 1
"😀".length   // 2
```

同樣是「長度」，a 跟中都是 1，emoji 卻是 2。把這三個字存成檔案，長度又變了：a 是 1 byte、中是 3 bytes、😀 是 4 bytes。

為什麼同一個「數長度」，數出來全不一樣？因為長度可以指三種不同的東西：字元、碼位、bytes。這三層分清楚，UTF-8 跟 UTF-16 就通了。

---

## 三層：字元、碼位、bytes

**字元**：人眼看到的一個符號。`中`、`a`、`😀` 各是一個字元。

**碼位（code point）**：Unicode 給每個字元一個唯一號碼。

```
a   → U+0061   （十進位 97）
中  → U+4E2D   （十進位 20013）
😀  → U+1F600  （十進位 128512）
```

Unicode 是一張全世界統一的號碼表，只管「哪個字元是幾號」，不管「怎麼存」。

**bytes**：號碼要存進記憶體、寫進檔案、送上網路，得變成一串 0 到 255 的 bytes。這一步叫**編碼（encoding）**。UTF-8、UTF-16 就是兩種編碼規則：同一個碼位，編出來的 bytes 不一樣。

先講一個最常見的誤會：**Unicode 不是編碼**。Unicode 是號碼表，一個字元配一個號碼；UTF-8 / UTF-16 才是「把號碼轉成 bytes」的方法。前者管編號，後者管存法，是兩件事。

```
字元 中
  ↓ Unicode 查表
碼位 U+4E2D
  ↓ 選一種編碼
UTF-8  → [228, 184, 173]   3 bytes
UTF-16 → [78, 45]          2 bytes
```

同一個中，兩種編碼存出不同的 bytes。這就是為什麼「長度」會有好幾個答案。

---

## UTF-8：以 byte 為單位，變長

UTF-8 一個字元用 1 到 4 bytes，看碼位大小決定：

```
碼位範圍              bytes   例子
U+0000 ~ U+007F       1       a-z、0-9（就是 ASCII）
U+0080 ~ U+07FF       2       é、希臘、西里爾、阿拉伯
U+0800 ~ U+FFFF       3       大多數中日韓漢字
U+10000 ~ U+10FFFF    4       emoji、罕見漢字
```

「變長」的意思是**不同字元用不同數量的 bytes**，但每個字元的長度都是確定的：a 固定 1 byte、中固定 3 bytes，只是彼此不一樣。對照定長編碼（像 UCS-2 每個字元硬性 2 bytes），UTF-8 讓常用的英文只佔 1 byte，省下空間。

長度不一，解碼的人怎麼知道一個字元到哪裡結束？UTF-8 把「我有幾 bytes」寫進第一個 byte 開頭的幾個 bit：

```
0xxxxxxx                              1 byte  （開頭 0）
110xxxxx 10xxxxxx                     2 bytes （開頭 110）
1110xxxx 10xxxxxx 10xxxxxx            3 bytes （開頭 1110）
11110xxx 10xxxxxx 10xxxxxx 10xxxxxx   4 bytes （開頭 11110）
```

第一個 byte 開頭有幾個 `1`，就代表這個字元共幾 bytes；後面接續的 bytes 都以 `10` 開頭。實際看 `"a中"` 的 byte 流：

```
a  → 01100001                     開頭 0    → 讀 1 byte
中 → 11100100 10111000 10101101   開頭 1110 → 讀 3 bytes
```

解碼器從左到右：看到 `0...` 就拿 1 個 byte，看到 `1110...` 就拿 3 個。每個字元自帶長度標記，所以長短不一也切得乾淨。這才是「變長」能運作的關鍵：不是長度模糊，是每個字元自己宣告佔幾格。

UTF-8 **跟 ASCII（早期的英文字元編碼）完全相容**。英文字母、數字、標點的碼位都在 U+007F 以內，用 1 byte 存，跟幾十年前的 ASCII 一模一樣，舊系統、舊檔案不用改就能讀。

而且**英文省空間**。純英文文件每個字元 1 byte，沒有浪費。網頁的標籤（`<div>`、`href`）、程式碼、JSON 的 key 大多是英文，UTF-8 幾乎最省。

代價是漢字要 3 bytes，比 UTF-16 的 2 bytes 多一個。但網路上英文跟符號佔多數，整體還是 UTF-8 省。所以 HTML5 預設 UTF-8，HTTP、JSON、大多數檔案都用它。

<details>
<summary>UTF-8 的身世（為什麼是 4 bytes）</summary>

UTF-8 是 1992 年 Ken Thompson 跟 Rob Pike 設計的（比 UTF-16 晚），據說是在紐澤西一家小餐館的餐墊紙上畫出來的。它一開始就是變長，而且原始版本能長到 **6 bytes**（涵蓋 31 bits），比現在還寬。

現在看到的「最多 4 bytes」是 2003 年 RFC 3629 才砍下來的，砍的理由是**對齊 UTF-16 的範圍**（到 U+10FFFF 為止）。所以這個 4 bytes 上限不是 UTF-8 自己的設計，是 UTF-16 借給它的：UTF-16 賭固定寬度賭輸、補丁變長；UTF-8 沒賭、本來更寬，最後反而被 UTF-16 的天花板框住，兩邊才停在同一個 U+10FFFF。

</details>

---

## UTF-16：以 16-bit 為單位，2 或 4 bytes

UTF-16 的單位是 16 bit（2 bytes），叫一個 **code unit**。

```
碼位範圍              單位數   bytes
U+0000 ~ U+FFFF       1        2      BMP 常用區
U+10000 ~ U+10FFFF    2        4      surrogate pair（代理對）
```

**BMP（Basic Multilingual Plane，基本多文種平面）** 是 U+0000 到 U+FFFF 這 65536 個號碼，涵蓋幾乎所有日常文字，包含全部常用漢字。這一區一個字元就是一個單位、2 bytes。

超出 BMP 的（emoji、罕見字），一個單位裝不下，得用**兩個**單位拼，這叫 surrogate pair。所以 `😀` 佔 2 個單位、4 bytes。

UTF-16 為什麼存在？早期大家以為 16 bit（65536 個號碼）就夠裝下全世界的字。當時 Java、Windows、JavaScript 都照這個假設設計，字串內部用固定 2 bytes 一格，`index` 存取很整齊。後來 Unicode 塞不下、擴到一百多萬個號碼，才補上 surrogate pair 這個「兩格拼一個」的機制。所以 UTF-16 是歷史包袱下的產物：本來想固定寬度，最後還是變長了。

---

## 誰選 UTF-8、誰選 UTF-16

分兩種場合：**存放跟傳輸** vs **程式語言內部**。

**對外（檔案、網路）幾乎都 UTF-8**：

```
HTML5 / HTTP / JSON / 大多數文字檔  → UTF-8
```

**程式語言內部**就分兩派：

| 語言 / 系統 | 字串內部 | 備註 |
|---|---|---|
| JavaScript | UTF-16 | `length`、`charCodeAt` 都是 UTF-16 單位 |
| Java | UTF-16 | `char` 就是 16-bit |
| C# / .NET | UTF-16 | |
| Windows API | UTF-16 | 「wide char」那套 |
| Go | UTF-8 | 字串就是 UTF-8 的 byte 序列 |
| Rust | UTF-8 | `String` 保證是合法 UTF-8 |

<details>
<summary>Python 的情況（比較特別）</summary>

Python 3 的 `str` 是 Unicode，但內部表示會**依內容自動選**最省的一種（純 ASCII 用 1 byte、BMP 內用 2 bytes、超出才用 4 bytes）。所以 Python 裡 `len("😀")` 得到 1，不像 JS 得到 2。它把「一個字元算一個」做對了，代價是內部實作複雜。[GUESS：Python 內部彈性表示的細節是我記憶中的，本 session 沒撈官方 doc 對過]

</details>

規律：**UTF-16 陣營大多是 1990 年代設計、當時賭 16 bit 夠用的語言**（Java、JavaScript、C#、Windows）。**UTF-8 陣營大多是後來設計的**（Go、Rust），直接選了對外一致、又省空間的 UTF-8。

---

## JavaScript：對內 UTF-16、對外 UTF-8

JS 字串內部是 UTF-16，但跟外界交換 bytes 時用 UTF-8。常見的字串方法，幾乎都在 UTF-16 那一層。

**`length` 數的是 UTF-16 單位，不是字元**：

```javascript
"中".length        // 1   一個單位
"😀".length        // 2   代理對，兩個單位
"a中😀".length     // 4   = 1 + 1 + 2
```

要數**真正的字元數**，用展開或 `Array.from`。字串的 iterator 認得代理對，會把兩個單位合成一個字元：

```javascript
[..."a中😀"].length         // 3   人眼看到的字元數
Array.from("a中😀").length  // 3   同上
```

**字元跟碼的來回轉，有兩對方法**：

```
UTF-16 單位（0 ~ 65535）        char ↔ 碼
  ch.charCodeAt(i)             字元 → 碼
  String.fromCharCode(n)       碼 → 字元

完整碼位（含 emoji）             char ↔ 碼
  ch.codePointAt(i)            字元 → 碼
  String.fromCodePoint(n)      碼 → 字元
```

```javascript
"中".charCodeAt(0)         // 20013
String.fromCharCode(20013) // "中"

"😀".charCodeAt(0)         // 55357   只拿到半個代理，沒意義
"😀".codePointAt(0)        // 128512  完整碼位
```

規則：a-z、常用漢字都在 BMP 內，`charCodeAt` 那對就夠。資料可能有 emoji 或罕見字，就得用 `codePointAt` 那對，不然 `charCodeAt`、逐字元迴圈全會裂在代理對上。（兩個 `from...` 都是 `String` 的靜態方法，不掛在字串實例上。）

**進出 UTF-8 bytes 用 `TextEncoder` / `TextDecoder`**：

```javascript
new TextEncoder().encode("中")
// Uint8Array [228, 184, 173]   ← 轉成 UTF-8，3 bytes

new TextDecoder().decode(new Uint8Array([228, 184, 173]))
// "中"                          ← UTF-8 bytes 轉回字串
```

`TextEncoder` 永遠吐 UTF-8。這就是「對外 UTF-8」那條線：把字串變成真正的 bytes，都會經過它。

---

## 真正摸到 bytes：Uint8Array 跟 TypedArray 家族

`TextEncoder` 回傳的 `Uint8Array` 是什麼？它是 JS 用來裝**原始 bytes** 的容器。

底層是 `ArrayBuffer`：一塊固定大小的原始記憶體，就是一排 bytes，本身沒有型別。不能直接讀它，要透過一個「視角」。**TypedArray** 就是各種視角，決定「幾個 byte 算一格、怎麼解讀」：

| 型別 | 一格幾 bytes | 範圍 |
|---|---|---|
| `Uint8Array` | 1 | 0 ~ 255 |
| `Int8Array` | 1 | -128 ~ 127 |
| `Uint16Array` | 2 | 0 ~ 65535 |
| `Int16Array` | 2 | -32768 ~ 32767 |
| `Uint32Array` | 4 | 0 ~ 4294967295 |
| `Int32Array` | 4 | -2147483648 ~ 2147483647 |
| `Float32Array` | 4 | 單精度浮點 |
| `Float64Array` | 8 | 雙精度浮點（JS 的 number 就是這個） |
| `BigInt64Array` | 8 | 64-bit 整數 |

開頭有 `U` 的（`Uint`）是 unsigned，只存 0 跟正數；沒 `U` 的（`Int`）是 signed，能存負數。同樣的 bits，一個把範圍全給正數，一個分一半給負數。

同一塊 `ArrayBuffer`，用不同 TypedArray 看，byte 怎麼分組就不一樣。同一排 8 個 byte，各種 view 切法對照：

```
原始 8 bytes：   b0 b1 b2 b3 b4 b5 b6 b7

Uint8Array   →  [b0][b1][b2][b3][b4][b5][b6][b7]   8 格，每格 1 byte
Uint16Array  →  [b0 b1][b2 b3][b4 b5][b6 b7]       4 格，每格 2 bytes
Uint32Array  →  [b0 b1 b2 b3][b4 b5 b6 b7]         2 格，每格 4 bytes
Float64Array →  [b0 b1 b2 b3 b4 b5 b6 b7]          1 格，8 bytes
```

同樣是 `b0 b1` 這兩個 byte：`Uint8Array` 當成兩個獨立的數看，`Uint16Array` 把它們合成一個數。底層 bytes 沒變，變的是「幾個 byte 算一格」。

處理文字用 `Uint8Array`（一格一個 byte，剛好對上 UTF-8）。要在同一塊 buffer 裡混讀不同型別、或控制 byte 順序（大端小端），用 `DataView`。這套是 JS 碰二進位資料的底層：讀檔案、處理圖片、`WebSocket`、`fetch` 的 `arrayBuffer()` 都會用到。

一條線串起來：

```
字串 "中"
  ↓ TextEncoder（編成 UTF-8）
Uint8Array [228, 184, 173]
  ↓ 底層就是
ArrayBuffer（3 bytes 原始記憶體）
```

字串在 UTF-16 那層，`Uint8Array` 在 bytes 那層，`TextEncoder` 負責兩層之間的轉換。

---

## 什麼時候會踩到

這些不是冷知識，寫 code 常撞上：

**算字數別用 `.length`。** 表單限 280 字、暱稱限 20 字，用 `.length` 檢查，使用者打一個 emoji 就被算成 2（中文倒還好，BMP 內算 1）。要數人眼看到的字數，用 `[...str].length`。

```javascript
"héllo 😀".length         // 8   UTF-16 單位（emoji 算 2）
[..."héllo 😀"].length    // 7   真正字元數
```

**存 emoji 進 MySQL 要用 `utf8mb4`。** MySQL 有個歷史坑：它的 `utf8` 其實只支援到 3 bytes（BMP），塞 4 bytes 的 emoji 會報錯或被截成亂碼。真正完整的 UTF-8 在 MySQL 叫 `utf8mb4`（mb4 = maximum bytes 4）。看到「emoji 存不進資料庫」十之八九是這個。

**按 byte 切字串會切壞字元。** 有些限制是「最多 100 bytes」（資料庫欄位、封包大小）。直接抓前 100 bytes，可能剛好切在一個中文字（3 bytes）或 emoji（4 bytes）的中間，尾巴變亂碼。要嘛按字元切，要嘛用 `TextEncoder` 先算好 byte 邊界。

**讀檔案亂碼，通常是編碼猜錯。** 一個 Big5 存的舊檔用 UTF-8 讀，中文全變亂碼；反過來也是。bytes 本身沒錯，是「用錯編碼表去解讀」。指定對的 encoding 就正常。

**網頁開頭要 `<meta charset="utf-8">`。** 不宣告，瀏覽器只能猜編碼，猜錯中文就變亂碼。這行是明白告訴瀏覽器「這頁用 UTF-8 解」。

**要真實 byte 大小，用 `TextEncoder`。** 算一段文字佔多少空間（存 DB、傳輸上限），`.length` 給的是 UTF-16 單位數，不是 bytes。

```javascript
const s = "中文 emoji 😀";
s.length                                // UTF-16 單位數
new TextEncoder().encode(s).length      // 真正的 UTF-8 byte 數
```

共通點：**只要牽涉到「存進哪裡、傳去哪裡、佔多少空間、算幾個字」，就要分清楚現在數的是字元、UTF-16 單位、還是 bytes。** 三個是不同的數字。

---

## 總結

| 概念 | 一句話 |
|---|---|
| Unicode | 號碼表，一個字元配一個碼位，不管怎麼存 |
| 編碼（encoding） | 把碼位轉成 bytes 的方法 |
| UTF-8 | 論 byte，1~4 bytes，ASCII 相容，英文最省，網路檔案用 |
| UTF-16 | 論 16-bit 單位，2 或 4 bytes，JS / Java 內部用 |
| JS `.length` | 數 UTF-16 單位，不是字元；emoji 算 2 |
| `charCodeAt` / `codePointAt` | 前者給 UTF-16 單位（emoji 裂半），後者給完整碼位 |
| `TextEncoder` | 字串 → UTF-8 的 `Uint8Array` |
| `Uint8Array` | 裝原始 bytes 的容器，底層是 `ArrayBuffer` |

分界點記兩個數字：**碼位 65536（0x10000）** 以上要兩個 UTF-16 單位（emoji 就在這）；**一個 byte 永遠 8 bits、0~255**。

相關：[Two's Complement](/concept/twos-complement) 講 bytes 怎麼存負數，[Bit Operators](/concept/operator) 講 bit 層的操作。
