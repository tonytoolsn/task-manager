# Learning Log

> 只記「卡超過 15 分鐘」的東西。小事不記。
> 格式:情境 → 原因 → 解法 → 延伸。
> 只寫解法的話,三個月後你會看不懂為什麼要這樣做。

---

## 範例（做完第一則就把這個刪掉）

### Eloquent 的 N+1 問題

**情境**
`GET /api/tasks` 回傳 20 筆任務,結果跑了 21 次 SQL 查詢。

**原因**
每筆任務取 `$task->project` 時,Eloquent 都額外查一次資料庫。

**解法**
```php
Task::with('project')->get();
```

**延伸**
Laravel Debugbar 可以看到每個請求的 SQL 次數。
開發環境可以用 `Model::preventLazyLoading()` 讓它直接報錯。

**日期**:2026-08-__ · 花了 40 分鐘

---

## 部署不是「把 repo 複製到伺服器」

**情境**
疑惑 docs/ 會不會上正式機。

**釐清**
部署是「跑 build,把產物放上去」。
docs/ 不是任何建置流程的輸入,前端部署完全不會碰到它。
Laravel 沒有 build 步驟,docs/ 確實會在伺服器上,
但在 public/ 之外,外界存取不到,只多佔幾百 KB。

**唯一要處理的場景**
Docker 的 COPY . . 會複製所有東西,需要 .dockerignore。

**真正該排除的**
.env、.git/、node_modules/、storage/logs/
其中 .git/ 最危險——含完整提交歷史,
包含曾經誤 commit 又刪掉的內容。

**日期**:2026-08-20

---

