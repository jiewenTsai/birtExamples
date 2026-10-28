# 新舊介面對照表

2026-10-28 的重構之後，birt 與 birtRcpp 的寫法有哪些地方不一樣。舊寫法大多還能用，只是會出現「已不建議使用」的警告；改成新寫法就不會再看到警告。

- birtRcpp：**改最多**。現成模型改成「給資料、用 `model =` 選模型」，很少改的設定收進 `ecm_control()`、`gibbs_control()`。
- birt：**語法不變**。回傳的表格改成 tibble；新增 `ipcr()`，取代已不建議使用的 `dif_tree()`；錯誤訊息的呼叫回到原本的函數名稱。

## 1. birtRcpp：選模型與估計

| 舊寫法 | 新寫法 |
|---|---|
| `fit <- ecm(rtirt_cross(dat))` | `fit <- ecm(dat, model = "cross")` |
| `ecm(rtirt_null(dat))` | `ecm(dat, model = "null")` |
| `ecm(rtirt_latreg(dat))` | `ecm(dat, model = "latreg")` |
| `ecm(rtirt_latent(dat))` | `ecm(dat, model = "latent")` |
| `ecm(mlirt(dat))` | `ecm(dat, model = "irt")` |
| `gibbs(rtirt_cross(dat), n_iter = 2000)` | `gibbs(dat, model = "cross", n_iter = 2000)` |
| `gibbs(fit, init = "ecm")`（`fit` 是 `ecm()` 的結果） | `gibbs(dat, model = "cross", init = fit)` |
| `gibbs(rtirt_cross(dat, quantile = 0.9))` | `gibbs(dat, model = "cross", quantile = 0.9)` |
| `gibbs(rtirt_latent(dat, quantile = c(0.1, 0.5, 0.9)))` | `gibbs(dat, model = "latent", quantile = c(0.1, 0.5, 0.9))` |
| `rtirt_null(dat, speed_var = "fixed", itemtype = "1pl")` | `ecm(dat, model = "null", speed_var = "fixed", itemtype = "1pl")`（`gibbs()` 同） |

說明：

- 模型的名字：`"irt"`（只有作答）、`"null"`、`"latreg"`、`"latent"`、`"cross"`。
- `sim_data()`、`sim_para()` 的模型名字**沒有改**：只有作答的模型仍叫 `"mlirt"`。所以模擬時寫 `sim_data(cond, sim_para(cond, "mlirt"), "mlirt")`，估計時寫 `ecm(dat, model = "irt")`。
- `cov =` 選共變項（`NULL` 是資料裡全部的共變項）；`ecm()` 與 `gibbs()` 都有。
- `intercept =`（作答迴歸加截距）只在 `gibbs()`。
- 分位數模型（`quantile =`）只能用 `gibbs()`：`ecm(dat, model = "cross", quantile = 0.9)` 會報錯，說要改用 `gibbs()`。
- `init = fit`：`fit` 必須是**同一份資料、同一個模型**（`speed_var`、`itemtype` 也相同）的 `ecm()` 結果，否則報錯。分位數模型不能用 `init = fit`，從隨機起點開始。
- `init = "ecm"` 仍可用：`gibbs()` 先自己跑一次 ECM，再從那裡出發。

## 2. birtRcpp：很少改的設定

這些引數從 `ecm()`、`gibbs()` 移到 `control =`。

| 舊寫法 | 新寫法 |
|---|---|
| `ecm(m, nodes = 61)` | `ecm(dat, model = "cross", control = ecm_control(nodes = 61))` |
| `ecm(m, max_iter = 5000, tol = 1e-8)` | `ecm(..., control = ecm_control(max_iter = 5000, tol = 1e-8))` |
| `ecm(m, accelerate = FALSE)` | `ecm(..., control = ecm_control(accelerate = FALSE))` |
| `ecm(m, verbose = TRUE)` | `ecm(..., control = ecm_control(verbose = TRUE))` |
| `gibbs(m, n_burnin = 500, n_thin = 2)` | `gibbs(..., control = gibbs_control(n_burnin = 500, n_thin = 2))` |
| `gibbs(m, collapse = TRUE)` | `gibbs(..., control = gibbs_control(collapse = TRUE))` |
| `gibbs(m, verbose = FALSE, fit_indices = FALSE)` | `gibbs(..., control = gibbs_control(verbose = FALSE, fit_indices = FALSE))` |

留在 `ecm()` 本身的：`model`、`itemtype`、`speed_var`、`cov`、`se`、`prior`（`"ml"`／`"map"`）、`priors`。
留在 `gibbs()` 本身的：`model`、`itemtype`、`speed_var`、`quantile`、`cov`、`intercept`、`priors`、`n_iter`、`n_chain`、`seed`、`init`。

| 設定 | 預設 |
|---|---|
| `ecm_control()` | `max_iter = 1000`、`tol = 1e-10`、`nodes = 41`、`accelerate = TRUE`、`verbose = FALSE` |
| `gibbs_control()` | `n_burnin = NULL`（`n_iter` 的一半）、`n_thin = 1`、`collapse = FALSE`、`fit_indices = TRUE`、`verbose = TRUE` |

## 3. birtRcpp：估計之前就能寫出模型

| 舊寫法 | 新寫法 |
|---|---|
| `equations(rtirt_cross(dat))` | `equations(dat, model = "cross")` |
| `equations(rtirt_latent(dat, quantile = 0.25))` | `equations(dat, model = "latent", quantile = 0.25)` |
| `algorithm(rtirt_cross(dat))` | `algorithm(dat, model = "cross")`（`method = "ecm"` 或 `"gibbs"`） |

估計之後照舊：`equations(fit)`、`algorithm(fit)`。

## 4. birtRcpp：已不建議使用

還能用、測試也保留，但不再加功能。每個函數在一個 R session 裡只警告一次；要每次都看到，設 `options(birtRcpp.deprecation = "always")`。

| 函數 | 改用 |
|---|---|
| `mlirt()`、`rtirt_null()`、`rtirt_latreg()`、`rtirt_latent()`、`rtirt_cross()` | `ecm(dat, model = ...)`、`gibbs(dat, model = ...)` |
| `gibbs(fit, init = "ecm")` | `gibbs(dat, model = ..., init = fit)` |
| `block_model()`、`step_gibbs()`、`step_mala()`、`step_shift()` | 現成模型（`ecm()`、`gibbs()` 的 `model =`） |
| `vi()` | `ecm()`（估計值）或 `gibbs()`（後驗分配） |
| `check_vi()` | `check_ecm()` 或 `check_sampler()` |
| `mml()` | `ecm(dat, model = ...)` |

不在廢止範圍：底層積木 `draw_*()`、`irt_part()`、`normal_part()` 等仍是正式函數。Julia 版的別名（`setCond()`、`GibbsRtIrtCross()` 等）也照舊。

## 5. 兩個套件：表格改成 tibble

| 函數 | 舊 | 新 |
|---|---|---|
| birtRcpp：`estimates()`、`scores()`、`fit_indices()`、`compare_para()` | data.frame | tibble |
| birtRcpp：`summary(fit)` 的各表（`s$items` 等） | data.frame | tibble（印法不變） |
| birt：`estimates()`、`scores()`、`fit_indices()`、`convergence()`、`compare()` | data.frame | tibble |
| birt：`summary(fit)` 的各表 | data.frame | tibble（印法不變） |
| 兩個套件：`as.data.frame(summary(fit))` | data.frame | data.frame（不變） |
| `score_test()`、`ci_test()`、`person_fit()`、`cond_reliability()` 等有自己 class 的結果表 | data.frame | data.frame（不變） |

數值完全相同，只有印出來的樣子不同。要注意的地方：

- tibble 預設只印 3 位有效數字。要看 logLik、AIC 的小數，先設 `options(pillar.sigfig = 7)`。
- tibble 預設只印前 10 列（超過 20 列時）。要看全部，寫 `print(estimates(fit), n = Inf)`。
- 取一欄用 `e$est` 或 `e[["est"]]`，得到向量。`e[, "est"]` 在 tibble 會得到一欄的 tibble，不是向量。
- tibble 沒有 row names。子集之後列號從 1 重新編，不再是原本的列號。
- 要舊的樣子：`as.data.frame(estimates(fit))`。

## 6. birt：`dif_tree()` 改用 `ipcr()`

| 舊寫法 | 新寫法 |
|---|---|
| `dif_tree(model, data, partition = z)`（需要 partykit） | `ml <- birt(model, data, control = rtmb_control(method = "aghq"))`，再 `ipcr(ml, z)` |

- `dif_tree()` 已不建議使用：還能用，每個 session 警告一次（`options(birt.deprecation = "always")` 每次都警告），之後的版本會移除。
- `ipcr()`（個人參數貢獻迴歸；Arnold 等人，2020）把每個人對每個估計值的貢獻，對所有共變項一起做迴歸：一次檢查所有參數、可以同時放連續與類別共變項，只要配適一次。
- 和 `score_test()` 一樣只接受最大概似配適。參數在內部尺度（log 負荷量、log SD、atanh 相關）。
- 斜率會往 0 縮（一步近似），所以用來篩選；效果大小請配適加上調節的模型（例如 `E(y3) ~ x2`）。
- 回傳的 `coefficients` 是 tibble；只看顯著的斜率：`dplyr::filter(ipcr(ml, z)$coefficients, p_holm < 0.05)`。

## 7. birt：錯誤訊息

birt 的語法、`birt()` 的引數都沒有改。重構時把檢查拆成小函數，錯誤訊息一度顯示小函數的名稱（例如 `Error in check_path_alias(ld, factors)`）；已改回原本的函數（`Error in parse_model(model)`），和重構前逐字相同。
