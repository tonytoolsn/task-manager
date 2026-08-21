# GitHub 專案設定指南

> 從零建立一個符合團隊規格的 repo。
> 下次開新專案時,照這份文件從頭跑一次即可。
>
> 本文件基於 task-manager 專案的實際設定過程,
> 包含踩到的坑與解法。最後更新:2026-08-20

---

## 事前決定

### Public 還是 Private?

**建議 Public**,除非有商業機密。

| | Public | Private（免費方案） |
|---|---|---|
| Rulesets（分支保護） | ✅ | ❌ **需付費** |
| GitHub Actions | ✅ 無限免費 | ⚠️ 每月 2000 分鐘 |
| 當作品集 | ✅ | ❌ |

> **踩坑紀錄**:本專案原先設為 Private,建好 ruleset 後出現黃色警告
> 「Your rulesets won't be enforced on this private repository until you
> move to GitHub Team organization account」——規則建了但不生效。
> 改成 Public 後**立即生效,不需重新設定**。

**改成 Public 前務必確認**沒有把敏感資料 commit 進去過:

```bash
git log --all --oneline -- '*.env' '*.env.local' '*.pem' '*credentials*'
```

沒有輸出才安全。有輸出的話,即使現在刪掉,檔案仍留在 git 歷史裡。

---

## 一、Branch Protection（分支保護）

**路徑**:Settings → Branches → **Add branch ruleset**

> 旁邊的 "Add classic branch protection rule" 是舊版做法,GitHub 正在淘汰,不要用。

### 基本設定

| 欄位 | 值 | 備註 |
|---|---|---|
| Ruleset Name | `main protection` | |
| **Enforcement status** | **`Active`** | ⚠️ 預設是 Disabled,**最常漏掉的一步** |
| Bypass list | **空的** | ⚠️ 把自己加進去 = 規則失效 |
| Target branches | Include default branch | |

### Rules 勾選

| 規則 | 勾? | 說明 |
|---|---|---|
| Restrict deletions | ✅ | 防止 main 被誤刪 |
| **Require a pull request before merging** | ✅ | 核心 |
| └ Required approvals | **`0`** | ⚠️ 一人專案設 1 會**永遠卡死** |
| └ Require conversation resolution | ✅ | 自我 review 留的 comment 要處理完才能 merge |
| └ Require approval of most recent push | ❌ | 要求「他人」批准,一人專案會卡死 |
| └ Allowed merge methods | **只留 `Squash`** | 見下方說明 |
| **Block force pushes** | ✅ | 防止 `git push --force` 洗掉歷史 |
| Require status checks to pass | ⏳ | **CI 建好後再回來開** |
| Require code scanning / quality / coverage | ❌ | 個人專案不需要 |
| Automatically request Copilot code review | ❌ | 見下方說明 |

### 為什麼 merge method 只留 Squash

一張票的分支上通常有五六個 commit(`wip`、`修錯字`、`再試一次`)。

| 方法 | main 上留下 |
|---|---|
| Merge commit | 全部 commit + 一個合併紀錄 |
| **Squash** | **一張票 = 一個 commit** |
| Rebase | 全部 commit 攤平成一直線 |

Squash 的實際好處:**你在分支上可以放心亂 commit**,反正最後會壓成一個。
30 張票之後,`git log` 是一份乾淨的功能清單,可直接拿來寫 Sprint Review。

### 為什麼不開 Copilot code review

自我 review 是訓練判斷力的核心環節。AI 先列好清單,你會變成照清單改,
而不是自己看出問題。**建議先自己審三個 Sprint**,之後再開來當第二意見。

### 驗證設定是否生效

```bash
echo "test" >> README.md
git add . && git commit -m "test: verify branch protection"
git push
```

**要被拒絕才算成功**:

```
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: - Changes must be made through a pull request.
```

然後退掉測試 commit:

```bash
git reset --hard HEAD~1
```

---

## 二、Labels（標籤）

**路徑**:Issues → Labels → New label

先刪掉 GitHub 預設的九個(bug、documentation、enhancement、duplicate、
good first issue、help wanted、invalid、question、wontfix),避免和新標籤重複。

### 命名原則:前綴分類法

`pri:high` 而不是 `high`。理由:

1. GitHub 標籤篩選支援前綴搜尋,打 `pri:` 只列出優先級標籤
2. 每個前綴回答**一個獨立問題**,維度不重疊
3. 標籤超過 15 個時,沒有前綴會變成一團混亂

| 前綴 | 回答的問題 | 必填? |
|---|---|---|
| `type:` | 這是什麼工作? | ✅ 每張票一個 |
| `pri:` | 多重要? | ✅ 每張票一個 |
| `area:` | 動到哪一塊? | 選填,可多個 |
| `status:` | 為什麼沒進度? | 選填,只在異常時 |
| `hat:` | 戴哪頂帽子?(一人團隊專用) | 選填 |

### 完整標籤清單(參考用)

> ⚠️ **不要一次全建。** 起始只建標記為「**起始**」的 11 個。
> 其餘等「何時建」欄位描述的訊號出現時再補。理由見下方「為什麼分批建」。

#### `type:` — 這是什麼工作(必填,每張票一個)

| 標籤 | 色碼 | 定義 | 何時建 |
|---|---|---|---|
| `type:feat` | `0E8A16` | 新功能 | **起始** |
| `type:fix` | `D73A4A` | 修 bug | **起始** |
| `type:refactor` | `1D76DB` | 改結構,行為不變 | **起始** |
| `type:docs` | `C5DEF5` | 文件 | **起始** |
| `type:chore` | `CFD3D7` | 環境、設定、套件 | **起始** |
| `type:spike` | `5319E7` | 純研究,產出是知道怎麼做 | **起始** |
| `type:test` | `BFD4F2` | 補測試 | 開始補既有功能的測試時 |
| `type:perf` | `006B75` | 效能優化 | 真的遇到「好慢」的問題時 |

#### `pri:` — 多重要(必填,每張票一個)

| 標籤 | 色碼 | 定義 | 何時建 |
|---|---|---|---|
| `pri:critical` | `B60205` | 線上壞了,或沒它系統不能用 | **起始** |
| `pri:high` | `D93F0B` | 核心體驗,缺了很痛 | **起始** |
| `pri:medium` | `FBCA04` | 有比較好 | **起始** |
| `pri:low` | `C2E0C6` | 想到再做 | **起始** |

優先級**可以隨時改變**。今天的 low,可能因為你實際用了系統之後變成 high。

#### `area:` — 動到哪一塊(選填,可多個)

**全部統一 `BFDADC`。** 這是刻意的 —— area 是輔助資訊,
不該搶走 `type:` 和 `pri:` 的視覺注意力。

| 標籤 | 涵蓋範圍 | 何時建 |
|---|---|---|
| `area:api` | Controller、路由、Resource | 程式碼多到會搞混時 |
| `area:db` | Migration、Model、查詢效能 | 同上 |
| `area:ui` | React 元件、樣式 | 前端開始動工時 |
| `area:auth` | 登入、權限、Sanctum | 做登入時 |
| `area:infra` | CI/CD、部署、環境變數 | 同上 |

#### `status:` — 為什麼沒進度(選填,只在異常時貼)

| 標籤 | 色碼 | 定義 | 何時建 |
|---|---|---|---|
| `status:blocked` | `000000` | 等外部條件,不能動 | **起始** |
| `status:needs-info` | `D876E3` | 需求不清,要先釐清 | 第一次看不懂自己的票時 |
| `status:wontfix` | `FFFFFF` | 決定不做,關票前貼 | 第一次決定砍掉某功能時 |

> `status:wontfix` 用白色是刻意的 —— 它在看板上幾乎看不見,
> 因為那張票即將消失。

#### `hat:` — 戴哪頂帽子(一人團隊專用,選填)

**全部統一 `E4E669`**(淡黃)。一般公司沒有這組,這是模擬多角色的專屬設計。

| 標籤 | 何時建 |
|---|---|
| `hat:pm` | 出現不同帽子的票時(約 Sprint 2) |
| `hat:design` | 同上 |
| `hat:backend` | 同上 |
| `hat:frontend` | 同上 |
| `hat:devops` | 同上 |

建了之後,Retro 時可以篩出「這個 Sprint 前端做了幾張、後端做了幾張」,
是很有參考價值的數據。

### 為什麼分批建

**標籤的成本不是建立,是使用。**

每張票開的時候,你要在腦中掃過所有標籤決定貼哪些。
11 個標籤時這件事花三秒;25 個標籤時花二十秒,而且你會開始猶豫
「這張算 `chore` 還是 `infra`?」

**猶豫本身就是成本。** 更糟的是,猶豫久了你會開始亂貼,標籤系統就失效了。

判斷標準:**一個從來沒被用來篩選過的標籤,就沒有存在價值。**

### 「該建了」的訊號

不是時間到了,是**你感受到具體的痛**:

| 訊號 | 該建的標籤 |
|---|---|
| 「這個 Sprint 我到底花多少時間在前端?」 | `hat:` 組 |
| 「上次改資料庫的那幾張票在哪?」 | `area:` 組 |
| 「這張票我自己都看不懂當初在寫什麼」 | `status:needs-info` |
| 「這功能決定不做了,但票留著提醒自己為什麼」 | `status:wontfix` |
| 「這支 API 好慢,要專門處理」 | `type:perf` |
| 「這功能沒測試,心裡毛毛的」 | `type:test` |

**痛出現之前建的標籤,你不會用**,因為你不知道它要解決什麼問題。

### 一張票貼幾個

**一個 `type:` + 一個 `pri:`,最多再加一個 `area:`。**
超過四個標籤就失去視覺篩選的意義。

### 為什麼 type 要和 commit type 對齊

```
Issue #5  標籤 type:feat
    ↓
分支      feat/5-task-list-api
    ↓
Commit    feat(task): add status filter to task list API
```

票、分支、commit 用同一套詞(Conventional Commits),
腦中只需要一套分類,不用轉換。

---

## 三、Milestones（里程碑）

**路徑**:Issues → Milestones → New milestone

### 為什麼期數用 Milestone 而不是標籤

舊做法 `P0` / `P1` / `P2` 有個根本問題:**它同時想表達「多重要」和「什麼時候做」**,
這兩件事會衝突。做到第五期時,`P5` 到底是「第五期」還是「第五重要」?

拆開之後:

- **優先級** → `pri:` 標籤,**隨時可以改變**
- **期數** → Milestone,**數量無限**,可設截止日、有進度條、完成可封存

兩者自由組合:`Milestone 3` + `pri:critical` = 第三期裡最重要的票。

### 正式專案的三種設計方式

實話說,**Milestone 在真實團隊裡沒有統一標準**,有些公司甚至完全不用。
以下是三種主流設計,依專案性質選一種。

#### 一、按版本(產品導向)

```
v0.1.0 · MVP
v0.2.0 · 專案分類與登入
v1.0.0 · 公開發布
v1.1.0 · 儀表板
```

**適合**:有明確發布節奏的產品、開源專案、SaaS
**特徵**:一個 Milestone 對應一次 release,關閉 Milestone = 發版。
開源專案幾乎都用這套(Laravel、React、Vue)。
**問題**:版本號要遵守 Semantic Versioning,不然會亂。
若採持續部署(每天上線),版本的概念會變模糊。

#### 二、按時間(Sprint 導向)

```
Sprint 12（8/21 - 9/03）
Sprint 13（9/04 - 9/17）
Sprint 14（9/18 - 10/01）
```

**適合**:跑 Scrum 的團隊、內部系統、外包專案
**特徵**:固定週期,時間到就關閉,沒做完的票移到下一期。
重點是**追蹤產能** —— 三個 Sprint 之後會知道「兩週大概能做幾點」。
**問題**:名稱沒有語意,三個月後看到「Sprint 12」不知道那期在做什麼。

#### 三、按主題(功能導向)

```
使用者認證
任務核心功能
專案管理
資料視覺化
```

**適合**:中大型專案、多人分工、需向非技術人員報告進度時
**特徵**:名稱有意義,被問「登入做完沒」打開 Milestone 就有進度條。
**問題**:沒有時間概念,容易無限延期。

### 業界更常見的:混合式

多數成熟團隊是三個維度塞進一個名稱:

```
v1.2.0 · 儀表板與統計（截止 10/15）
   ↑          ↑              ↑
  版本       主題           時間
```

版本可追溯、名稱有語意、又有時間壓力。

**然後 Sprint 不用 Milestone 管**,改用 GitHub Projects 的
**Iteration 自訂欄位** —— 那是專門為 Sprint 設計的,可設定週期自動輪替。

| 要回答的問題 | 用什麼 |
|---|---|
| 這批功能什麼時候發布? | **Milestone** |
| 這張票在哪個 Sprint 做? | **Projects 的 Iteration 欄位** |

**這兩個是不同維度。** 一個 Milestone 可能橫跨三個 Sprint。

### 一個 Milestone 該裝多少票

業界經驗值:**15 ~ 40 張**。

| 數量 | 問題 |
|---|---|
| < 10 張 | 太細碎,統計意義不大,不如用標籤 |
| 15 ~ 40 張 | ✅ 合適 |
| > 50 張 | 太大,進度條長期停在 30% 會讓人麻木,該拆 |

> 本專案 M1 只有 9 張,偏少但可接受 —— 第一個 Milestone
> 以「跑通流程」為優先。

### 什麼時候不該用 Milestone

有些團隊完全不用,理由是:

- 採**持續交付**,每張票做完就上線,沒有「一批」的概念
- 用外部工具(Jira、Linear)管理,GitHub 只放程式碼
- 團隊太小,看板就夠用

**判斷標準:如果你從來不點開 Milestone 頁面,它對你就沒有價值。**

### 撰寫格式

```
Title:       M1 · MVP 可用
Due date:    2026-09-03
Description: 任務與專案的 CRUD 能透過 API 完整運作,並部署到測試環境。
             不含前端、不含登入。
```

**Description 一定要寫「不含什麼」** —— 這是範圍控制的書面依據。
當你在期中想「順便把登入做一下」時,回來看這一行,它會擋住你。

真實團隊裡這叫 **out of scope**,是每份規格文件的標準段落。

### 本專案的演進計畫

| 階段 | 命名方式 | 理由 |
|---|---|---|
| M1 | `M1 · MVP 可用` | 先跑通流程,不糾結命名 |
| M2 起 | 改用**混合式**:`v0.2.0 · 專案分類與登入` | 見下方 |

從 M2 改成混合式的三個理由:

1. **版本號讓你之後能寫 CHANGELOG** —— 可展示專案的演進歷程,是好的履歷素材
2. **主題名稱讓你三個月後看得懂**
3. **截止日提供時間壓力**,防止無限延期

至於 Sprint,**等 Milestone 開始橫跨多個兩週週期時**,再去 Projects
加 Iteration 欄位。現階段 M1 剛好等於 Sprint 1,不需要維護兩套。

### 一次只建一個

M2、M3 等做完 M1 再建。提前規劃的里程碑,內容幾乎一定會變。

---

## 四、Projects（看板）

**路徑**:Projects → New project → 選 **Board**

| 欄位 | 值 |
|---|---|
| Project name | `Task Manager Sprint` |
| Import items from repository | ✅ 勾選(之後開的 Issue 自動進看板) |

### 五個欄位

預設只有三欄,需手動新增 `Backlog` 和 `Review`:

```
Backlog → Todo → In Progress → Review → Done
```

| 欄位 | 中文 | 進入條件 | 離開條件 |
|---|---|---|---|
| Backlog | 需求池 | 任何時候想到的新點子 | 下次 Planning 被挑中 |
| Todo | 本期待辦 | Planning 時挑入,之後不再增加 | 開始動手 |
| In Progress | 進行中 | 開了分支 | 開了 PR |
| Review | 待審查 | PR 已開 | 自我 review 完成並 merge |
| Done | 已完成 | PR merged | 不再移動 |

> 找不到新增欄位的按鈕時:右上角 `⋯` → Settings → Status → 編輯選項。
> GitHub 的看板欄位實際上是 `Status` 欄位的選項值。

### 怎麼決定要幾欄

#### 核心原則:每一欄代表「等待的原因不同」

看板的欄位**不是分類,是狀態轉移**。一張票從左走到右,
每次跨欄都代表「一件事完成了,現在卡在下一件事上」。

判斷該不該加一欄,只問這個問題:

> **票停在這一欄時,是在等什麼?這個「等」跟隔壁欄一樣嗎?**

一樣就合併,不一樣才分開。

**用本專案的五欄驗證:**

| 欄位 | 在等什麼 |
|---|---|
| Backlog | 等被挑中(等**決策**) |
| Todo | 等有人開始(等**人力**) |
| In Progress | 等寫完(等**工作**) |
| Review | 等審查(等**眼睛**) |
| Done | 不等了 |

五個不同的「等」,所以五欄成立。

> 反例:若加上 `Coding` 和 `Testing` 兩欄 —— 這兩者都是
> 「等我把它做完」,屬於同一種等待,應該合併成 `In Progress`。

#### 為什麼多數團隊落在 5~7 欄

| 欄數 | 問題 |
|---|---|
| < 4 欄 | **看不出瓶頸**。票全堆在 `In Progress`,不知道卡在寫程式還是審查 |
| 5~7 欄 | ✅ 實務上的甜蜜點 |
| > 8 欄 | 每張票要拖七次,拖到第三次就開始忘。且很多票會「卡在兩欄中間」 |

#### 真實團隊的常見配置

**有 QA 的團隊(6~7 欄)**
```
Backlog → Todo → In Progress → Code Review → QA → Ready to Deploy → Done
```
多出的 `QA` 和 `Ready to Deploy` 成立,是因為**不同人在等** ——
QA 等測試人員,Ready to Deploy 等發布窗口。

**有設計流程的團隊**
```
Backlog → Design → Design Review → Dev Ready → In Progress → Code Review → Done
```
`Dev Ready` 是關鍵的一欄:設計做完了,工程師還沒開始。
**這一欄的長度會告訴你是設計跑太快,還是工程跑太慢。**

**純持續交付的團隊(4 欄,極簡)**
```
Todo → Doing → Review → Done
```
沒有 Backlog(用 label 管)、沒有部署欄(merge 就自動上線)。

#### 進階做法:欄內再分「做」與「等」

成熟團隊常見這種寫法:

```
In Progress ┃ Review          ┃ Done
 Doing │ Done ┃ Doing │ Done  ┃
```

每個階段內部再切「正在做」和「做完了在等下一關」。

**理由**:「我寫完了但還沒開 PR」和「我還在寫」是不同狀態。
前者是**在等下游**,後者是**在做**。
分開之後才看得出瓶頸 —— 若 `In Progress / Done` 一直積壓,
代表 review 太慢,不是開發太慢。

> **對一人團隊太細**,因為下游也是你,不會有「等下游」的問題。

#### 一人團隊的特殊性:Review 欄的意義不同

真實團隊裡,票停在 `Review` 是**等別人有空看**。
一人團隊的 `Review` 是**等時間過去**(隔天再看)。

**所以 Review 欄的長度不會反映瓶頸,只反映「今天開了幾個 PR」。**

Retro 時不要用 Review 欄的積壓量判斷問題。該看的是:

| 觀察對象 | 代表什麼 |
|---|---|
| **In Progress 的停留時間** | 一張票待超過估時的兩倍 → 估錯了或卡住了 |
| **Backlog 的成長速度** | 一直變長 → 期中一直冒新點子,範圍控制有問題 |

#### 「該加欄」的訊號

| 訊號 | 該加的欄 |
|---|---|
| 開始有部署窗口(不是 merge 就上線) | `Ready to Deploy` |
| 前端票出現,而且要等後端 API | `Blocked`(或改用 label) |
| 開始畫線框稿,設計與開發分離 | `Design` + `Dev Ready` |
| 一張票 merge 後還要手動驗證 | `Verifying` |

**痛出現之前加的欄,你不會用**,而且會增加每張票的拖拉次數。

#### 反面案例

實務上看過的一個團隊看板:

```
Ideas → Backlog → Refined → Ready → Todo → In Progress →
Blocked → Code Review → Changes Requested → QA → UAT →
Ready to Deploy → Deployed → Verified → Done
```

**15 欄。** 結果是沒有人維護,票全部停在 `In Progress`,
因為拖到後面太麻煩。

**看板的價值來自「它是準的」。**
一個沒人更新的精細看板,遠不如一個大家都在用的粗糙看板。

### WIP limit

`In Progress` 欄位標題 → `⋯` → **Set limit** → `1`

一個人同時只有一雙手。開三個功能的結果是三個都做到一半、都跑不起來,
而且出事時分不清是哪個改動造成的。

超過時欄位標題會變紅,不會真的阻止你,但那個紅色就是提醒。

### 建議加的自訂欄位:Estimate

**路徑**:右上角 `⋯` → Settings → Fields → New field → Single select

```
XS = 半天以內
S  = 1 天
M  = 2 天
L  = 3 天以上 → 這張票該拆了
```

**「估到 L 就要拆」是很有用的紀律。**
估到 L 通常不是因為工作量大,而是**你還沒想清楚**。
當你能把一張 L 拆成三張 S,代表你已經想過執行細節了。
拆不出來 → 開一張 `type:spike` 的研究票,不要硬幹。

---

### 正式專案的四種估計方式

這是敏捷圈爭議最大的題目之一,沒有標準答案。

#### 一、Story Points(費氏數列)— 最主流

```
1, 2, 3, 5, 8, 13, 21
```

**估的不是時間,是「相對複雜度」。**
一張 2 點的票不代表 2 小時或 2 天,只代表「大約是 1 點那張的兩倍難」。

**為什麼用費氏數列而不是 1~10**:數字越大,估計越不準。
你分得出 1 和 2 的差別,但分不出 8 和 9 的差別。
費氏數列的間隔越後面越大,強迫你在不確定時往上跳一級。

**怎麼開始**:挑一張大家都懂的中等難度票當基準,定為 3 點,
之後所有票都跟它比。

**Velocity(速度)**:跑三個 Sprint 後會知道「一期大概能做 30 點」,
之後 Planning 就照這個數字抓票量。**這是 Story Points 存在的唯一目的。**

| 優點 | 缺點 |
|---|---|
| 避開「幾天做完」的政治壓力 | 概念抽象,新手要練很久 |
| 相對估計比絕對估計準 | 易被誤用成「1 點 = 半天」,退化成工時 |
| 有 Velocity 可預測產能 | 跨團隊不能比較(A 隊的 5 點 ≠ B 隊的 5 點) |

#### 二、T-shirt Size — 輕量派

```
XS, S, M, L, XL
```

概念與 Story Points 相同,但**刻意不用數字**,
避免有人拿去加總或換算成工時。

**適合**:早期規劃、尚未細拆的 epic、小團隊、一人專案

許多團隊採兩層做法:epic 用 T-shirt size 粗估,
拆成 task 之後才用 Story Points 細估。

#### 三、Ideal Hours / Days — 務實派

直接估「不受干擾的話要幾小時」。

**適合**:外包計費、需向客戶報價、工作內容高度可預測的團隊

**問題**:沒有人有「不受干擾的一天」。開會、被問問題、環境出包,
實際產能大約是估計值的 **50~60%**。這個落差會造成持續性的「延遲」,
士氣很差 —— **這正是 Story Points 被發明出來的原因**。

#### 四、#NoEstimates — 反對派

**主張**:估計花的時間不如拿去做事,而且估了也不準。

**替代做法**:把票都拆到差不多大小(一天內能做完),
然後單純數「這期做了幾張票」。統計上一樣能預測產能,
但省下估點會議。

**適合**:成熟團隊、持續交付、票拆得很細的專案。
在小型團隊和獨立開發者中越來越流行。

---

### Planning Poker:多人團隊的標準做法

1. PM 念一張票
2. 每個人**同時**出牌(避免被資深者影響)
3. 數字差距大的人各自說明理由
4. 再出一次,直到收斂

**重點不是那個數字,是討論的過程。**

當後端出 3、前端出 13,代表兩人對這張票的理解完全不同。
把這個落差抓出來,才是 Planning Poker 真正的價值 ——
**它是一個溝通工具,偽裝成估計工具。**

---

### 估計的兩條紀律

#### 一、估過的票,做完後不准回頭改估點

很多人會想「我估 3 點,結果做了 8 點的工,改一下比較準確」——**不行**。

這樣 Velocity 會永遠完美,你也永遠學不到自己的偏差。
**保留錯誤的估計,才有校準的依據。**

#### 二、範圍變了要開新票,不是改估點

唯一該處理的情況是:**期中發現這張票的範圍根本不對。**

> 例:「CI 與部署」估 M,做的時候發現部署平台不支援 SQLite,
> 還要多做資料庫遷移。

**正確做法不是改估點,而是把多出來的工作開成一張新票。**
原票維持 M,新票獨立估 —— 這樣兩張票的紀錄都是誠實的。

---

### 本專案採用:T-shirt Size + 記錄實際花費

一人專案拿不到 Story Points 的最大價值(團隊對齊),所以選 T-shirt Size:

1. 判斷「這張比那張大」很容易,不需學抽象概念
2. 要練的是**校準自己的直覺**,不是跟別人對齊
3. **「實際花費」那一欄才是重點**

三個 Sprint 之後,`learning-log.md` 裡應該要出現這種東西:

```
估 S（1天)的票,實際平均 1.8 天
估 M（2天)的票,實際平均 4 天 ← 票越大,偏差越大
凡是牽涉「部署」「時區」「第三方服務」的票,一律乘 2.5
```

**這才是估時能力 —— 不是估得準,是知道自己會怎麼估錯。**

---

### 補充:Projects 的進階設計(參考用)

> ⚠️ **現階段唯一需要做的是「開啟兩條自動化」**(見本節最後)。
> 其餘內容等「該加了的訊號」出現時再回來看。

#### Project 的切分依據是「團隊」,不是「專案」

**Project 的本質是排程工具,不是分類工具。**
它要回答的是:「這群人接下來要做什麼、順序如何、做得完嗎?」

而「這群人」的邊界就是團隊。同一批人做的所有事,
都該在同一個看板上排隊 —— 否則無法在 A 專案的票和 B 專案的票之間比優先級。

| 情況 | 結構 |
|---|---|
| 一個團隊,一個專案 | `Project: Task Manager`(本專案) |
| 一個團隊,多個專案 | `Project: Platform Team` ← 跨 repo 收票 |
| 一個專案,多個團隊 | `Backend Team` / `Frontend Team` / `Mobile Team` |

> GitHub Projects **可以跨 repo 收票**。同一團隊維護三個 repo 時,
> 按 repo 切成三個 Project,每天要開三個看板,
> 而且無法回答「我們這期總共要做多少事」。

#### 判斷該不該開第二個 Project

只問一個問題:**這兩批工作,會不會在同一個時間段搶同一個人的時間?**

- **會搶** → 同一個 Project(否則無法排優先級)
- **不會搶** → 可以分開

> 對一人團隊而言,未來的第二個專案**會搶你的時間**,
> 所以嚴格來說應該併入同一個 Project(跨 repo 收票),
> 而不是另開一個。等真的有第二個專案時再搬,批次加入很容易。

#### 四個維度,四個工具,不要混用

| 要回答的問題 | 用什麼 |
|---|---|
| 這是哪個專案的程式碼? | **Repo** |
| 這批功能什麼時候發布? | **Milestone** |
| 這是什麼類型的工作? | **Label** |
| 這群人接下來做什麼? | **Project** |

**混用就是日後會亂掉的原因。**

#### 三種切法的取捨

**一、按團隊(傳統)**

```
Backend Team / Frontend Team / Platform Team / Design
```

適合 20 人以上、團隊工作內容差異大的組織。
**問題**:一個功能常橫跨前後端,同一件事被拆成兩張票、放在兩個看板,
進度對不起來。

**二、按產品線**

```
Web App / Mobile App / Admin Console / Public API
```

適合一家公司有多個獨立產品、各自有發布節奏。
**問題**:共用元件與基礎設施的票不知道放哪,
通常要再開一個 `Shared`,然後那個 Project 會變成雜物間。

**三、單一 Project + 多個 View（現代主流)**

**同一份資料,不同的呈現方式。** 這是 GitHub Projects v2 的設計哲學。

**關鍵理解:View 不是複製資料,是同一批票的不同鏡頭。**
在 Board 上把票拖到 Done,Table view 裡的 Status 也會同步更新。

#### 為什麼多 View 優於多 Project

**一、不同角色關心不同事情**

| 角色 | 想看什麼 | 用哪個 View |
|---|---|---|
| 工程師 | 我今天做什麼 | My tasks(篩 assignee) |
| Tech Lead | 誰卡住了 | Board(看 In Progress 停多久) |
| PM | 這期進度如何 | This sprint(篩 Iteration) |
| 主管 | 什麼時候能上線 | Roadmap(時間軸) |
| QA | 有哪些 bug 待修 | Bugs(篩 `type:fix`) |

用「多 Project」的做法,這五個人要開五個分頁,還要自己在腦中拼起來。

**二、篩選比分割更有彈性**

需求會變。分割成多個 Project 之後切法就固定了,要改就得搬票;
多 View 只要新增一個 View,兩分鐘的事。

#### 成熟團隊的欄位設計

| 欄位 | 類型 | 用途 |
|---|---|---|
| `Status` | Single select | 看板欄位 |
| `Priority` | Single select | Critical / High / Medium / Low |
| `Estimate` | Single select 或 Number | XS/S/M/L 或 Story Points |
| `Iteration` | **Iteration** | Sprint 週期,GitHub 自動輪替 |
| `Area` | Single select | api / db / ui / auth / infra |
| `Assignee` | 內建 | 誰負責 |
| `Start date` | Date | Roadmap view 用 |
| `Target date` | Date | Roadmap view 用 |

> `Iteration` 是**專門的欄位類型**,不是 Single select。
> 設定週期後 GitHub 會自動產生 Sprint 1、Sprint 2…,
> 並知道「現在是哪一期」,可篩選 `iteration:@current`。

#### 常見的六個 View

**1. Board — 日常工作台**
```
Layout:   Board
Group by: Status
Filter:   iteration:@current
Sort:     Priority
```
**只顯示當期的票。** 這個 filter 最重要 ——
沒有它,Backlog 的一百張票會淹沒看板。

**2. Table — 批次編輯**
```
Layout: Table
Filter: （無)
```
Planning 時用,可一次選十張票批次設定 Iteration 或 Priority。

**3. My work — 個人視角**
```
Layout:   Board
Filter:   assignee:@me is:open
Group by: Status
```

**4. Roadmap — 對外報告**
```
Layout:   Roadmap
Group by: Milestone
Dates:    Start date → Target date
```
給非技術主管看的。Roadmap view 能自動畫出甘特圖,
是 GitHub Projects 最被低估的功能。

**5. Blocked — 卡住的票**
```
Layout: Table
Filter: label:status:blocked
```
每天站會看這個。**一張票在此待超過三天,是需要升級處理的訊號。**

**6. Backlog grooming — 需求池整理**
```
Layout: Table
Filter: status:Backlog
Sort:   Priority desc
```
每兩週整理一次,該做的往上提,該砍的關掉。

#### ✅ 現在就該做:開啟自動化

**路徑**:Project → 右上角 `⋯` → **Settings** → **Workflows**

| 觸發 | 動作 | 現在開? |
|---|---|---|
| Item added to project | Status = Backlog | 選用 |
| **Item closed** | **Status = Done** | ✅ **開** |
| **Pull request merged** | **Status = Done** | ✅ **開** |
| Code changes requested | Status = In Progress | 選用 |
| Auto-add to project | 符合條件的新 Issue 自動加入 | 選用 |

**「PR merged → Status = Done」一定要開。**
否則每次 merge 完還要手動拖卡片,拖三次就會忘,看板開始失真。

**看板失真之後,你會停止相信它,然後停止使用它。**
自動化不是為了省時間,是為了**保持資料可信**。

#### 常見錯誤:開太多 Project

「這個 epic 比較大,單獨開一個 Project 吧」——
三個月後你有十二個 Project,沒人知道某張票在哪,搜尋要跨看板找。

**原則:一個團隊一個 Project,用 View 切視角,用 Milestone 切批次。**
只有在**兩批人幾乎不互相溝通**時,才值得開第二個 Project。

#### 「該加了」的訊號

| 訊號 | 該加的東西 |
|---|---|
| 票超過 30 張,Board 開始要捲動 | Board 加 `iteration:@current` filter |
| 開始有 Backlog 積壓 | Backlog grooming(Table view) |
| 開始跑第二個 Milestone | Roadmap view |
| 出現卡住超過三天的票 | Blocked view |
| Milestone 橫跨多個兩週週期 | `Iteration` 欄位 |

跟標籤同一個道理:**痛出現之前建的東西,你不會用。**

---

## 五、範本檔案

### 正式團隊的完整清單

```
.github/
├── ISSUE_TEMPLATE/
│   ├── config.yml              範本選單的設定
│   ├── feature.yml
│   ├── bug.yml
│   ├── tech-debt.yml
│   └── spike.yml
├── PULL_REQUEST_TEMPLATE/      多個 PR 範本時才用資料夾
│   ├── default.md
│   ├── hotfix.md
│   └── release.md
├── workflows/                  CI/CD
├── CODEOWNERS                  誰該 review 哪些檔案
├── dependabot.yml              自動更新套件
└── FUNDING.yml                 贊助連結(開源專案)

根目錄:
├── README.md                   最重要,但最常被忽略
├── CONTRIBUTING.md             怎麼參與這個專案
├── CODE_OF_CONDUCT.md          行為準則(開源必備)
├── SECURITY.md                 資安漏洞回報管道
├── CHANGELOG.md                版本變更紀錄
└── LICENSE
```

### 哪些對一人專案有意義

| 檔案 | 一人專案 | 理由 |
|---|---|---|
| `README.md` | ✅ **必要** | 見下方 |
| `ISSUE_TEMPLATE/config.yml` | ✅ **必要** | 禁止開空白 Issue |
| `ISSUE_TEMPLATE/*.yml` | ✅ 建議升級 | 必填欄位可強制紀律 |
| `dependabot.yml` | ✅ 值得 | 練習 review 別人的 PR |
| `CHANGELOG.md` | ⏳ M2 之後 | 有版本號才有意義 |
| `CONTRIBUTING.md` | ⚠️ 選用 | 對自己的規範備忘 |
| `CODEOWNERS` | ❌ | 只有你,沒有「誰該 review」的問題 |
| `SECURITY.md` | ❌ | 沒有外部使用者回報漏洞 |
| `CODE_OF_CONDUCT.md` | ❌ | 沒有貢獻者 |
| `PULL_REQUEST_TEMPLATE/`(資料夾) | ❌ | 單一 `.md` 就夠,等有 hotfix 流程再說 |

---

### README.md — 最該優先補的檔案

**這是整個 repo 最重要的檔案。** 三個月後有人(包括你自己)
打開這個 repo,第一眼就是它。

必要段落:

| 段落 | 內容 |
|---|---|
| 一句話簡介 | 這是什麼、給誰用 |
| 技術棧 | 表格列出各層技術 |
| **快速開始** | 從零跑起來的完整指令 |
| 專案結構 | 資料夾對照 |
| 文件索引 | 連到 `docs/` 底下各文件 |
| 開發流程 | 分支、commit、PR 規則摘要 |

**「快速開始」是重點。** 三個月後換電腦、或要在另一台機器上跑,
這段就是救命的。而且寫這段會逼你檢查:
**我的專案真的能從零裝起來嗎?**

> 很多人的專案裝不起來,因為某個關鍵步驟只存在於作者的記憶裡。

---

### Issue 範本:YAML 表單 vs Markdown

`.md` 是舊做法,GitHub 現已支援 **YAML 表單**,差異很大。

| | Markdown（`.md`) | YAML（`.yml`) |
|---|---|---|
| 呈現 | 一個文字框,內含預填內容 | 真正的表單欄位 |
| **必填** | ❌ 無法強制 | ✅ **沒填不能送出** |
| 下拉選單 | ❌ | ✅ |
| 自動加標籤 / 標題前綴 | ✅ | ✅ |

**建議用 YAML,理由:它把紀律變成系統強制。**

Markdown 範本可以整段刪掉 —— 趕時間時你一定會這麼做,
然後那張票三週後你就看不懂了。YAML 的必填欄位擋得住。

> 這是**把紀律變成系統強制的少數機會之一**,
> 跟 branch protection 屬於同一類價值。

YAML 範例(節錄):

```yaml
name: 功能
title: "[feat] "
labels: ["type:feat"]
body:
  - type: textarea
    id: acceptance
    attributes:
      label: 驗收條件
      value: |
        - [ ] 
        - [ ] 有對應的測試
    validations:
      required: true

  - type: dropdown
    id: estimate
    attributes:
      label: 估時
      options:
        - XS - 半天以內
        - S - 1 天
        - M - 2 天
        - L - 3 天以上（該拆了）
    validations:
      required: true
```

#### 四種 Issue 範本

| 範本 | 什麼時候用 | 自動加的標籤 |
|---|---|---|
| 功能 | 新增功能 | `type:feat` |
| Bug | 有東西壞了 | `type:fix` |
| 技術債 | 重構、補測試、清理 | `type:refactor` |
| 研究 | 純研究,產出是知道怎麼做 | `type:spike` |

**「研究」為什麼需要獨立範本:**
Spike 沒有功能產出,但它是真實的工作。它的範本有兩個特殊欄位 ——
**時間盒**(超過就停止,用當下已知資訊做決定)與
**結論**(研究完回填,同步寫進 `learning-log.md`)。

有了這張票,你就不會覺得「查了半天資料」的那段時間是浪費掉的。

---

### config.yml — 控制範本選單

```yaml
blank_issues_enabled: false

contact_links:
  - name: 開發流程文件
    url: https://github.com/<你的帳號>/<repo>/blob/main/docs/一人團隊操作手冊.md
    about: 開票前先確認驗收條件該怎麼寫
```

**`blank_issues_enabled: false` 是關鍵。**

它禁止開空白 Issue,**強迫每張票都走範本**。
不然趕時間時你會點「Open a blank issue」隨手寫兩行,
那張票三週後就變成謎題。

---

### dependabot.yml

```yaml
version: 2
updates:
  - package-ecosystem: "composer"
    directory: "/task-api"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 3
    commit-message:
      prefix: "chore(deps)"
    labels: ["type:chore"]
```

**`open-pull-requests-limit` 是必要的。** 不設限制的話,
第一次啟用可能一次開二十個 PR,直接淹沒看板。

**對一人專案的實際價值**:你會定期收到「有套件可更新」的 PR,
這是練習「**review 一個我沒寫的 PR**」的好機會 ——
也是真實工作裡最常見的日常任務之一。

**啟用時機**:等 `composer.json` / `package.json` 真的存在之後。
現在放上去它會找不到目錄。

---

### CHANGELOG.md 的撰寫紀律

**寫給「使用者」看,不是給工程師看。**

```
✅ 可以依專案篩選任務
❌ 新增 TaskController@index 的 project_id 參數
```

分類只用六種:`Added` `Changed` `Deprecated` `Removed` `Fixed` `Security`。

開發中的變更先放 `[Unreleased]`,發版時才移到新版本區塊下。

---

### 現階段的建議

**只做兩件事:**

1. **寫 `README.md`** —— 唯一「不寫會有實際損失」的
2. **加 `config.yml` 的 `blank_issues_enabled: false`** —— 三行,但強制走範本

其餘等訊號出現:

| 訊號 | 該加的檔案 |
|---|---|
| 發現自己開票開始偷懶跳欄位 | Issue 範本升級 YAML |
| `composer.json` / `package.json` 已存在 | `dependabot.yml` |
| 開始用版本號(M2) | `CHANGELOG.md` |
| 出現緊急修正線上問題的流程 | `PULL_REQUEST_TEMPLATE/hotfix.md` |
| 有第二位協作者 | `CODEOWNERS` `CONTRIBUTING.md` |

---


放在 repo 根目錄的 `.github/`:

```
.github/
├── pull_request_template.md
└── ISSUE_TEMPLATE/
    ├── feature.md
    └── bug.md
```

> **踩坑紀錄**:範本檔案本身也要透過 PR 才能進 main(因為 branch protection)。
> 所以**第一個 PR 開的時候,範本還不會生效**——那個 PR 的描述要手動貼。
> 從第二個 PR 開始才會自動帶出。

### Issue 範本的關鍵欄位

```markdown
## 驗收條件
- [ ] 具體、可驗證
- [ ] 錯誤情況也要寫
- [ ] 有對應的測試
```

**驗收條件是一張票最重要的部分。** 沒有它,你永遠不知道一張票什麼時候算做完。

### 估時與實際花費

範本裡同時有「估時」和「實際花費」兩欄。完成時務必回填實際值。

**三個 Sprint 之後,你會發現自己有固定的估時偏差**(多數人低估 1.5~2 倍)。
知道自己的倍率之後,估時才會準——這是真實工作裡很值錢的能力。

---

## 六、PR 流程

### 完整循環

```
git checkout main
git pull
git checkout -b <type>/<issue>-<描述>
   ↓
（寫程式,分支上可隨意 commit)
   ↓
git push -u origin <branch>
   ↓
開 PR → 內文寫 Closes #N → 設 Assignee、Labels
   ↓
【放到隔天】← 整個流程最有價值的一步
   ↓
Files changed → 逐檔勾 Viewed → Submit review
   ↓
Squash and merge → Delete branch
   ↓
git checkout main && git pull && git branch -d <branch>
```

### 分支命名

```
feat/5-task-list-api        新功能
fix/23-timezone-bug         修 bug
refactor/31-extract-hook    重構
docs/10-fix-tech-stack      文件
chore/2-init-laravel        環境設定
```

格式:`<type>/<issue編號>-<英文簡述>`

**票號一定要對得上**,不然三個月後看 git log 找不到對應的票。

### Commit 訊息(Conventional Commits)

```
feat(task): add status filter to task list API
fix(task): correct overdue calculation for timezone
test(task): add cases for invalid status value
docs(api): update POST /api/tasks response example
refactor(ui): extract TaskCard component
chore(deps): bump laravel to 11.9
```

格式:`type(scope): 動詞開頭的英文描述`

### Closes #N 的作用

PR 內文寫 `Closes #5`,merge 時 GitHub 會**自動關閉 #5 那張 Issue**,
並在 Issue 上留下連結。省下手動關票的步驟,也建立了雙向追溯。

支援的關鍵字:`Closes` / `Fixes` / `Resolves`,大小寫皆可。

---

## 七、已知限制與繞法

### GitHub 不允許 approve 自己的 PR

```
Pull request authors can't approve their own pull requests.
```

**這是 GitHub 硬性規定,無法透過設定關閉。**

繞法:Submit review 時選 **`Comment`**。一樣留下正式審查紀錄與時間戳,
只是沒有綠色的 Approved 徽章。

**這也是為什麼 Required approvals 必須設 0** ——設 1 的話,
一人專案的 PR 會永遠無法 merge。

### merge 後的 PR 無法再修改

merge 完的 PR 是關閉狀態,不能再加東西進去。這是 Git 的設計,不是設定問題。

review 時發現的小問題,如果不值得擋住整個 PR,做法是:
**記在 review comment 裡 → merge → 另開一張 follow-up 票修掉。**

這在真實團隊裡每天都在發生,是正常流程。

### Co-authored-by 誤判

squash merge 時,Extended description 可能自動出現:

```
Co-authored-by: Tony Wu <tony.wu@example.com>
```

原因:git 設定的 email 與 GitHub 帳號的 email 不同,GitHub 誤判成兩位作者。

刪掉那一行,並修正 git 設定:

```bash
git config user.email "你的GitHub帳號email"
git config user.name "你的GitHub帳號名稱"
```

> 不加 `--global` 只影響當前專案。
> **順帶影響**:commit 的 email 若與 GitHub 帳號對不上,
> contribution graph(綠色方格)不會亮。

---

## 新專案快速檢查清單

```
□ 建立 repo,設為 Public
□ 確認沒有敏感資料在 git 歷史裡
□ Settings → Branches → Add branch ruleset
   □ Enforcement status = Active
   □ Bypass list 空的
   □ Required approvals = 0
   □ Require conversation resolution ✅
   □ Allowed merge methods 只留 Squash
   □ Block force pushes ✅
□ 驗證 push 到 main 會被拒絕
□ 刪除預設 labels,建立**起始 11 個** labels（其餘等訊號出現再補)
□ 建立第一個 milestone（含 due date 與「不含什麼」;命名方式見第三章)
□ 建立 Board,五欄,In Progress 設 WIP limit = 1
□ 加 Estimate 自訂欄位（XS/S/M/L;估計方式的取捨見第四章)
□ **開啟 Workflows 自動化**:Item closed → Done、PR merged → Done
□ 放入 .github/ 範本檔案（走 PR 流程)
□ **撰寫 README.md**（含「快速開始」的完整指令)
□ `config.yml` 設 `blank_issues_enabled: false`
□ 設定 git user.email / user.name
□ 開第一批票,設 Labels + Milestone + Project + Assignee
```

---

## 之後要回來補的

| 時機 | 動作 |
|---|---|
| CI 建好並跑過一次後 | 回 Ruleset 開啟 **Require status checks to pass** |
| 出現前端的票時 | 建立 `hat:` 標籤組 |
| 程式碼變多、容易搞混時 | 建立 `area:` 標籤組 |
| 自我 review 進行三個 Sprint 後 | 考慮開啟 Copilot code review 當第二意見 |
| 三個 Sprint 之後 | 統計估時偏差倍率,寫進 `learning-log.md` |
| 團隊超過一人時 | 考慮改用 Story Points + Planning Poker |
| M1 完成後 | 建立 M2(改用混合式命名 `v0.2.0 · 主題`),封存 M1 |
| Milestone 開始橫跨多個兩週週期時 | Projects 加 **Iteration** 自訂欄位管理 Sprint |
