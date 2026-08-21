# 參與本專案

## 開發環境

見 [README](README.md#快速開始)。

## 分支與 Commit

所有變更必須透過 Pull Request 進入 `main`,**不允許直接 push**。

### 分支命名

```
<type>/<issue編號>-<英文簡述>

feat/5-task-list-api
fix/23-timezone-bug
refactor/31-extract-hook
docs/10-fix-tech-stack
chore/2-init-laravel
```

### Commit 訊息

依循 [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <動詞開頭的英文描述>

feat(task): add status filter to task list API
fix(task): correct overdue calculation for timezone
test(task): add cases for invalid status value
docs(api): update POST /api/tasks response example
```

可用的 type:`feat` `fix` `refactor` `perf` `test` `docs` `chore`

## Pull Request

1. 一張票一個 PR,PR 內文寫 `Closes #<票號>`
2. 填寫 PR 範本的自我審查清單
3. CI 必須全綠
4. 使用 **Squash and merge**

## 程式碼規範

### 後端

- 所有輸入必須經過 FormRequest 驗證
- 查詢必須限制在當前使用者範圍內
- 使用 `with()` 避免 N+1 查詢
- 每支 API 至少兩個測試:正常情況 + 錯誤情況
- **改動 API 必須同步更新 `docs/api-spec.md`**

### 前端

- 每個資料抓取都要處理 loading / error / empty 三種狀態
- 只依照 `docs/api-spec.md` 實作,不參考後端程式碼
- 手機版必須測試過

## 開票

使用 Issue 範本,不允許開空白 Issue。

驗收條件必須**具體、可驗證**,並涵蓋錯誤情況。
估時超過 2 天(L)的票必須拆分。
