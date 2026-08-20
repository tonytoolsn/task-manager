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
