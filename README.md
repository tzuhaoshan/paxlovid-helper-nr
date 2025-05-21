# 無 Ritonavir 補發版 README-nr.md

```markdown
# Paxlovid 給藥異常處理工具（無 Ritonavir 補發版）

## 功能說明

這是一個純靜態的 HTML/JavaScript 工具，協助藥師快速判斷與處理 Paxlovid 給藥異常情況，特別優化處理藥局無法補發 Ritonavir 的情境。

## 判斷邏輯與計算原則

### 基本設定

#### 腎功能分類與總劑次
- **正常腎功能**：早晚各服用 2N+1R，共 10 劑次
- **中度腎功能不全**：早晚各服用 1N+1R，共 10 劑次
- **重度腎功能不全/血液透析**：僅早上服用，第 1 劑 2N+1R，後 4 劑 1N+1R，共 5 劑次

#### 偏差量計算
```javascript
missing_n = max(0, should_n - actual_n)  // 漏服 N 的顆數
missing_r = max(0, should_r - actual_r)  // 漏服 R 的顆數
extra_n = max(0, actual_n - should_n)    // 多服 N 的顆數
extra_r = max(0, actual_r - should_r)    // 多服 R 的顆數
剩餘療程計算
javascriptremDoses = totalDoses - currentDoseNum   // 剩餘劑次
remN = 根據腎功能計算剩餘 N 錠總數      // 剩餘 N 錠數
remR = remDoses                          // 剩餘 R 錠數（每劑 1 顆）
1. 處理建議原則
處理建議依照以下優先順序進行判斷：
A. 漏服 Ritonavir (missing_r > 0)

≤ 8 小時：建議「補完整劑量（should_n N+should_r R），下一劑依原時程」
> 8 小時：建議「跳過該劑，下一劑依原時程」

B. 僅漏服 Nirmatrelvir (missing_n > 0 且 missing_r == 0)

≤ 8 小時：建議「補吃 missing_n 顆 N，下一劑依原時程」
> 8 小時：建議「跳過該錠，下一劑依原時程」

C. 多服完整劑量 (extra_n ≥ should_n 且 extra_r ≥ should_r)

建議「跳過下一劑並監測臨床反應」

D. 其他多服 (extra_n > 0 或 extra_r > 0)

建議「監測臨床反應，下一劑依原時程」

E. 無異常

建議「無異常，依原時程給藥」

2. 藥局補發提示原則（無 Ritonavir 補發版特殊邏輯）
A. 漏服 Ritonavir 且 ≤ 8 小時

補發實際已吃的 Nirmatrelvir 錠數 (actual_n)
Ritonavir 由病人現場補吃，不需補發

B. 多服 Ritonavir 的特殊處理

剩餘劑次足夠：建議最後 extra_r 劑各少服用 1 顆 Ritonavir

如同時多服 Nirmatrelvir：仍需補發 Nirmatrelvir
提示訊息：「藥局無多餘 Ritonavir 可補發，建議最後 X 劑各少服用 1 顆 Ritonavir」


剩餘劑次不足：建議病人諮詢醫師

提示訊息：「藥局無多餘 Ritonavir 可補發且剩餘劑次不足，請諮詢醫師」



C. 只多服 Nirmatrelvir

正常補發 Nirmatrelvir

D. 完整多服

不補發任何藥品

E. 其他狀況

不進行任何補領

3. 護理餘藥提示原則
A. 觸發條件
僅在「錯誤劑次漏服超過 8 小時」且該次需要「跳過本劑」的情況下，才會提示護理人員回收餘藥。
B. 回收計算
javascriptreturn_n = max(0, should_n - actual_n)  // 漏服 N 的顆數（護理需回收）
return_r = max(0, should_r - actual_r)  // 漏服 R 的顆數（護理需回收）
C. 回收判斷

如果某一項計算結果 ≤ 0，表示不需回收該藥品
如有需要回收的藥品，提示「需回收 [藥品名稱] [數量] 錠，請將多餘藥品送回藥局」
如無需回收，提示「無餘藥需處理」或「無餘藥需處理（病人已服完或多服）」

特殊情境處理示例
例 1：多服 2 顆 Ritonavir，剩餘 5 劑

建議：藥局無多餘 Ritonavir 可補發，建議最後 2 劑各少服用 1 顆 Ritonavir

例 2：多服 2 顆 Ritonavir 且多服 1 顆 Nirmatrelvir，剩餘 5 劑

建議：需補發 1 顆 Nirmatrelvir。藥局無多餘 Ritonavir 可補發，建議最後 2 劑各少服用 1 顆 Ritonavir

例 3：多服 3 顆 Ritonavir，剩餘 2 劑（剩餘劑次不足）

建議：藥局無多餘 Ritonavir 可補發且剩餘劑次不足，請諮詢醫師

使用方法

選擇病人的「腎功能狀況」
輸入「給藥錯誤的劑次」
輸入「距離給藥錯誤的時間（小時）」
輸入「實際吃 Nirmatrelvir 錠數」
輸入「實際吃 Ritonavir 錠數」
系統自動計算並顯示：

處理建議
藥局補發提示（含 Ritonavir 不足特殊處理）
護理餘藥提示
詳細分析（包含理論應服用量、偏差量分析、剩餘療程）



版本資訊

無 Ritonavir 補發版：Version 1.0.5 (nr) - 適用於藥局無法補發 Ritonavir 的特殊情境
另有「標準版」：Version 1.0.5 - 適用於一般藥局補發情境

應用場景
本版本特別適用於以下情境：

藥局庫存 Ritonavir 不足
需要在療程中動態調整 Ritonavir 服用量
處理 Ritonavir 多服但無法補發的特殊情況

開發資訊
本工具由 AFTERNOON 團隊開發，採用純前端技術實現，不需要後端支援。
© 2025 AFTERNOON
