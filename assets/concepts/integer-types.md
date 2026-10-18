---
title: "Integer Types"
category: Math & Bit
slug: integer-types
subtitle: int 多大、範圍多少、為什麼各語言不一樣
date: 2026-07-19T12:00:00
updated: 2026-07-29
---

# Integer Types

寫 Java 的時候，`int` 一定是 32 bits、範圍固定。

換到 C，同一個 `int` 可能 16 bits 也可能 32 bits；`long` 在 Linux 是 8 bytes、在 Windows 卻是 4 bytes。

到了 Python，一個 int 想多大有多大，永遠不會溢位。

「一個 int 幾 bytes」這個問題，沒有跨語言的統一答案。哪些是固定的、哪些看語言、哪些看平台，得分開講。

---

## 固定的部分

完全固定、不用擔心平台的：

- **`1 byte = 8 bits`**。現代硬體的事實標準。只有讀網路規格看到 octet（專指剛好 8 bits）、或碰到古董硬體才有例外。
- **bit 是最小單位**，只有 0 或 1；一個 byte 是 8 個 bit，能表示 0 到 255。

其他的（int 幾 bytes、範圍多少）就開始看語言跟平台。

---

## 32 bits 的範圍

最常見的 int 是 32 bits（4 bytes）。範圍看有號還是無號：

| | 範圍 | 實際數字 | 大約 |
|---|---|---|---|
| signed（有號，可存負數） | $-2^{31}$ ~ $2^{31}-1$ | -2,147,483,648 ~ 2,147,483,647 | ±21 億 |
| unsigned（無號，只有非負） | $0$ ~ $2^{32}-1$ | 0 ~ 4,294,967,295 | 42 億 |

為什麼有號的正數少一格？因為 0 佔掉一格，最高位又拿去當正負號。同一組 bit，unsigned 全給正數，signed 分一半給負數。細節見 [Two's Complement](/concept/twos-complement)。

---

## 各語言的 int

| 語言 | `int` 大小 | 範圍固定嗎 | 備註 |
|---|---|---|---|
| Java | 32 bits | 固定 | spec 保證跨平台，`long` = 64 bits，沒有 unsigned 型別 |
| C# | 32 bits | 固定 | 大小同 Java，但有 unsigned 型別（`uint`、`ulong`） |
| C / C++ | 至少 16 bits，通常 32 bits | **看平台** | 標準只保證 $\ge 16$ bits，`long` 更麻煩（見下） |
| Go | 32 或 64 bits | **看平台** | 64 bits 機器上是 64 bits；要固定用 `int32`/`int64` |
| Rust | 沒有叫 `int` 的型別 | 明寫 | 一律 `i32`/`i64`/`u32`；`isize`/`usize` 是平台字長 |
| Python | 任意精度 | 無上限 | int 會自動長大，永遠不溢位 |
| JavaScript | 沒有 int | Float64 | 數字都是 64 bits 浮點，安全整數到 $2^{53}-1$ |

規律：1990 年代設計、當時要跨機器統一的語言（Java、C#）把 int 釘死在 32 bits；更早的 C 為了貼近硬體，讓 int 跟著平台走。晚近的兩個走不同路：Rust 乾脆不提供叫 `int` 的型別，強制每次明寫位寬；Go 提供 `int32`/`int64` 讓你明寫，但預設的 `int` 還是跟平台走，這點跟 C 一樣。

---

## C 的 `long` trap

同樣是 64 bits 作業系統，`long` 的大小卻不一樣：

```
Linux / macOS（LP64 模型）    long = 64 bits（8 bytes）
Windows（LLP64 模型）         long = 32 bits（4 bytes）
                            （只有 long long 跟指標才 64 bits）
```

所以一份 C code，`long x = 5000000000;`（50 億）在 Linux 塞得下、在 Windows 會溢位。這是跨平台移植很常見的 bug。要固定大小，別用 `long`，用 `int64_t`。

---

## 溢位

整數會溢位，因為它有**固定寬度**。32 bits signed int 加到 $2^{31}-1$，再加 1 會繞回 $-2^{31}$（最小的負數）。這不是隨機亂跳，是 two's complement 的固定寬度環繞，見 [Two's Complement](/concept/twos-complement)。（各語言態度不同：Java 明定就是環繞；C 把 signed 溢位列為 undefined behavior，實務上通常環繞，但編譯器有權假設它不會發生。）

JavaScript 有個特別版本的 trap。它沒有 int，數字都是 Float64，安全整數到 $2^{53}-1$（約 $9 \times 10^{15}$），照理很大。但 **bitwise 運算（`&`、`|`、`<<`、`>>`）會先把數字轉成 32 bits 有號整數**（ToInt32），所以 `(a + b) >> 1` 這種寫法會在 $2^{31}$ 就溢位，就算原本的 Number 夠大也沒用。

溢位的樣子：`a + b` 加到 $2^{31}$（2,147,483,648）時，ToInt32 把它塞進 32 bits，bit 31 被解讀成符號位，數值繞回 $-2^{31}$；接著 `>> 1` 是保留符號的右移，得到約 $-10.7$ 億。mid 變成負數，拿去當 array index 直接出錯。這正是 binary search 算 mid 常見的 trap，見 [Bit Operators](/concept/operator)。

---

## 要固定大小，用明寫位寬的型別

跨平台的 code 別用會變的 `int`/`long`，用明寫位寬的：

| 語言 | 明寫位寬的型別 | 備註 |
|---|---|---|
| C / C++ | `int32_t`、`int64_t`、`uint32_t` | 在 `<stdint.h>` |
| Rust | `i32`、`i64`、`u32` | 本來就強制明寫 |
| Go | `int32`、`int64` | `int` 才是看平台的 |

LeetCode 是單一環境，C/C++ 的 `int` 就是 32 bits，不用擔心平台差異。但 Go 別大意：LeetCode 跑在 64 bits 機器上，Go 的 `int` 是 64 bits。逐 bit 重組答案的題目（例如 [137 Single Number II](/problem/single-number-ii)）只組低 32 位，負數答案要自己 `int(int32(x))` 截斷回去，不然 -2 會變成 4294967294。寫會部署到多平台的 code 同理，固定位寬型別能省掉一整類「這台對、那台錯」的 bug。

---

## 總結

| 問題 | 答案 |
|---|---|
| 1 byte 幾 bit | 固定 8 |
| int 幾 bytes | 通常 4（32 bits），但看語言、看平台 |
| 32 bits signed 範圍 | $-2^{31}$ ~ $2^{31}-1$（約 ±21 億） |
| 32 bits unsigned 範圍 | $0$ ~ $2^{32}-1$（約 42 億） |
| Java / C# 的 int | 固定 32 bits |
| C 的 int / long | 看平台，long 還分 Linux 64 / Windows 32 |
| Python 的 int | 任意精度，不溢位 |
| JS 的數字 | Float64，但 bitwise 先轉 32 bits |
| 要固定大小 | 用 `int32_t` / `i64` / `int64` 這種明寫位寬的 |

相關：[Two's Complement](/concept/twos-complement)（負數與溢位）、[Bit Operators](/concept/operator)（bitwise 與 32 bits 轉換）、[Character Encoding](/concept/character-encoding)（byte 與 bit）。
