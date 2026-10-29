# 0.1.0 與 0.2.0 對照

這份文件比較 GitHub 上的 0.1.0 和目前的 0.2.0，說明兩版差在哪裡。

| | 0.1.0 | 0.2.0 |
|---|---|---|
| birt | GitHub `main`（a4d97a4） | `refactor` 分支（1b10ffa） |
| birtRcpp | GitHub `main`（eb74eb8） | `refactor` 分支（705e38c） |
| birtExamples | GitHub `main`（fb6a82d） | `refactor` 分支 |

0.2.0 還沒有推上 GitHub。要把舊程式改成新寫法，請看[新舊介面對照表](MIGRATION.md)。這份文件講的是整體差異，以及兩版實際跑出來的樣子。

文中「實際輸出」都是 2026-10-29 在同一台電腦上跑的：兩個版本都從原始碼乾淨安裝，用同一段程式和同一份資料。

## 1. 重點

- 估計結果沒有變。同一份資料、同一個模型，兩版算出的數值相同。兩個套件的 C++ 程式（`src/`）完全沒有改。
- birtRcpp 改了寫法：給資料，再用 `model =` 選模型。舊寫法還能用，但會警告。
- birt 的模型語法和 `birt()` 的引數都沒有改。改的是輸出、錯誤訊息和文件，另外新增 `ipcr()`。
- 兩個套件印出的數字都改成小數 3 位，回傳的表格改成 tibble。物件裡存的仍是完整數值。
- 常見的寫錯，0.1.0 常常只說哪裡不對，有時還會安靜地算錯。0.2.0 會說哪裡錯、該怎麼寫。
- 兩個教學最前面都加了給完全新手的第 0 章。

## 2. 估計結果沒有變

| 檢查 | 0.1.0 | 0.2.0 |
|---|---|---|
| birtRcpp：`ecm()` cross 模型的 logLik（300 人 × 10 題） | −5041.192 | −5041.192 |
| birt：預設 ELGM 的 log marginal likelihood（300 人 × 8 題） | −3668.94 | −3668.94 |
| birt：最大概似的 −2logL（Likert 題的例子） | 2202.704558 | 2202.704558 |

- 兩個套件都加了回歸測試：重構前先存下每個內建模型的結果，0.2.0 的每次測試都和它比對。
  - birt 比對 ELGM、最大概似（AGHQ）、JAGS。
  - birtRcpp 比對五種內建模型，新、舊兩種寫法都比。固定種子下，Gibbs 抽出的樣本和以前完全相同。
- 有一個例外會讓資料不同：`sim_rtirt()`、`sim_hgrm()` 的預設種子改了（第 5.3 節）。寫 `seed = 1` 就和 0.1.0 一樣。

## 3. 規模

| | birt 0.1.0 | birt 0.2.0 | birtRcpp 0.1.0 | birtRcpp 0.2.0 |
|---|---|---|---|---|
| commit 數（0.1.0 之後） | | 43 | | 46 |
| 改動的檔案 | | 95 個（+7991／−2749 行） | | 95 個（+8477／−2510 行） |
| R 程式行數 | 6857 | 10603 | 7539 | 11185 |
| 測試區塊（`test_that`） | 70 | 94 | 97 | 131 |
| 匯出的函數 | 36 | 37 | 113 | 115 |
| 說明頁 | 35 | 38 | 52 | 57 |

- 0.1.0 匯出的函數一個都沒有拿掉。新增的函數有 birt 的 `ipcr()`，以及 birtRcpp 的 `ecm_control()`、`gibbs_control()`。
- 0.2.0 兩個套件的 R CMD check `--as-cran` 都是 Status: OK。
- 0.2.0 的測試結果：
  - birt：CRAN 模式 601 項通過，完整測試 930 項通過。
  - birtRcpp：完整測試 1973 項通過。

## 4. birtRcpp

### 4.1 寫法

| 要做的事 | 0.1.0 | 0.2.0 |
|---|---|---|
| ECM 估計 | `ecm(rtirt_cross(dat))` | `ecm(dat, model = "cross")` |
| Gibbs 抽樣 | `gibbs(rtirt_cross(dat), n_iter = 2000)` | `gibbs(dat, model = "cross", n_iter = 2000)` |
| 從 ECM 結果開始抽樣 | `gibbs(fit, init = "ecm")` | `gibbs(dat, model = "cross", init = fit)` |
| 分位數模型 | `gibbs(rtirt_cross(dat, quantile = 0.9))` | `gibbs(dat, model = "cross", quantile = 0.9)` |
| 很少改的設定 | `ecm(m, nodes = 61)` | `ecm(dat, model = "cross", control = ecm_control(nodes = 61))` |
| 估計前寫出模型 | `equations(rtirt_cross(dat))` | `equations(dat, model = "cross")` |
| 比較兩個模型 | 自己算 `2 * (logLik(f1) - logLik(f0))`、`pchisq()` | `anova(f0, f1)` |

模型的名字是 `"irt"`、`"null"`、`"latreg"`、`"latent"`、`"cross"`，不分大小寫。

0.1.0 沒有新寫法。用 0.1.0 跑 `ecm(dat, model = "cross")` 會得到：

```
Error: model must be a model object, e.g. rtirt_cross(data), or a block_model()
```

### 4.2 已不建議使用

下面這些還能用，測試也保留，但每個 R session 會警告一次。設 `options(birtRcpp.deprecation = "always")` 可以每次都看到警告。

- 建構函數 `mlirt()`、`rtirt_null()`、`rtirt_latreg()`、`rtirt_latent()`、`rtirt_cross()`
- `gibbs(fit, init = "ecm")`
- 積木 `block_model()`、`step_gibbs()`、`step_mala()`、`step_shift()`
- `vi()`、`check_vi()`、`mml()`

底層積木 `draw_*()`、`irt_part()`、`normal_part()` 和 Julia 版的別名不在其中。

### 4.3 同一個模型，兩版印出來的樣子

0.1.0：

```
rtirt_cross 
  300 persons x 10 items
  maximum marginal likelihood (ECM, 41 Gauss-Hermite nodes): 15 iterations, 0.2 s
  logLik -5041.192, npar 51, AIC 10184.4, BIC 10373.3
  Reliability: ability 0.782, speed 0.973
Use summary(), coef(), scores(), estimates(), fit_indices(); gibbs(fit, init = "ecm") to sample from here.
```

0.2.0：

```
model = "cross" (rtirt_cross)
  300 persons x 10 items
  maximum marginal likelihood (ECM, 41 Gauss-Hermite nodes): 15 iterations, 0.2 s
  logLik -5041.192, npar 51, AIC 10184.385, BIC 10373.278
  Reliability: ability 0.782, speed 0.973
Use summary(), coef(), scores(), estimates(), fit_indices(); gibbs(data, model = ..., init = fit) to sample from here.
```

`estimates(fit)` 的前 4 列，0.1.0：

```
  parameter       est        se        z      q025     q975 sig
1     a[q1] 1.0714760 0.1974079 5.427727 0.7467200 1.537472   *
2     a[q2] 0.8358899 0.1718874 4.863008 0.5586156 1.250792   *
```

0.2.0：

```
# A tibble: 4 x 7
parameter    est     se  z   q025   q975  sig
a[q1]      1.071  0.197     0.747  1.537
a[q2]      0.836  0.172     0.559  1.251
```

差別有三處：
- 數字印到小數 3 位，`est` 取出來仍是 1.0714760。
- 0.1.0 每個鑑別度 a 都有星號。a 的區間在 log 尺度上算出，永遠不含 0，所以星號沒有意義，連 a = 0.003 也有星號。0.2.0 只替 0 有意義的參數（b、beta、rho、相關）標星號和 z 值。
- 回傳的是 tibble。

### 4.4 輸出的其他改變

- `print()` 和 `summary()` 的數字都是小數 3 位。人數、題數、自由度、ESS 不帶小數，p 值小於 0.001 時印 `<0.001`。
- `print(gibbs_fit)` 的收斂警告和 `convergence()` 用同一個標準：每個參數 ESS > 400 且 R̂ < 1.05。只有一條鏈時，會說明算不出 R̂。0.1.0 只看 R̂。
- `print(ecm_fit)` 在估計值或標準誤極端時會警告，並建議 `prior = "map"` 或 `gibbs()`。例如 40 人時 b = −34.139、SE 4104.678。0.1.0 不警告。
- `p <- precis(fit)`、`s <- summary(qs)` 指定給變數時不再印出。
- `gibbs()` 一開始抽樣就印一行，每條鏈跑完再印一行，附預估還要多久。0.1.0 要等第一條鏈跑完才有輸出。
- 新增：
  - `anova()`：比較同一份資料的模型。ECM 結果給概似比檢定，Gibbs 結果給 WAIC 和 LOOIC。
  - `plot(fit, "structural")`：結構參數的估計值與區間。
  - `as.data.frame(fit)`：給 ggplot2 用。
  - `algorithm(fit, detail = FALSE)`：只列步驟，不列公式。

`anova()` 的實際輸出（0.2.0）：

```
                           npar     logLik        AIC        BIC    Chisq  Df  Pr(>Chisq)
ecm(dat, model = "null")     42  -5156.048  10396.097  10551.656
ecm(dat, model = "cross")    51  -5041.192  10184.385  10373.278  229.712   9      <0.001  ***
```

### 4.5 寫錯時的訊息

0.1.0 一欄都是實際跑 0.1.0 得到的。寫「沒有這個寫法」的，是新寫法才有的情況。0.2.0 一欄取自實際執行、NEWS 和測試。

| 寫錯的地方 | 0.1.0 | 0.2.0 |
|---|---|---|
| 長格式資料（每人每題一列）傳給 `input_data()` | 不報錯，變成「3000 人 × 1 題」 | 報錯，並給 `tidyr::pivot_wider()` 的轉換寫法 |
| `id` 有重複 | 不提醒 | 警告是哪個 id 重複 |
| 共變項是文字或因子 | 只說 `cov: non-numeric columns: grade` | 另外說明怎麼做虛擬編碼（`model.matrix(~ grade, data)[, -1]`） |
| 作答寫成 `"correct"`／`"wrong"` | 只說 `resp: non-numeric columns: q1, q2, ...` | 列出找到的值，並給轉成 0/1 的寫法 |
| 時間的單位是毫秒 | 不提醒 | 中位數超過 1000 時警告 |
| `ecm(dat, model = "cross", nodes = 61)` | 沒有這個寫法 | `nodes is a setting of ecm_control(), not an argument of ecm(): use ecm(dat, model = "cross", control = ecm_control(nodes = 61))` |
| 引數拼錯（`niter`） | 舊寫法：照樣跑完，再警告 `unused arguments: niter` | `unknown argument niter; did you mean n_iter?`，估計前就停下 |
| `model = "null"` 加 `quantile = 0.9` | 沒有這個寫法 | 報錯，說明只有 latent 和 cross 有分位數版本 |
| 模型名字拼錯（`"crosss"`） | 沒有這個寫法 | `unknown model "crosss"; did you mean "cross"?` |
| `init = fit` 的資料或模型不同 | 沒有這個寫法 | 說明差在哪裡，例如人數不同、模型不同 |
| `ecm_control()` 交給 `gibbs()` | 沒有這個設定物件 | 說明這是哪個函數的設定 |
| `gibbs(dat, prior = "map")` | 沒有這個寫法 | 說明 `prior =` 是 `ecm()` 的設定 |

### 4.6 文件

- README 從「該從哪裡開始」寫起，附「你是……就讀……」的對照表。安裝時加 `build_vignettes = TRUE`，並說明 `vi` 遮蔽 `utils::vi` 的訊息是什麼意思。
- 入門教學：
  - 第 0 章是給完全新手的導入：資料長什麼樣子、IRT 和作答時間的白話說明、安裝、R 基本功、第一個完整分析。
  - 新增一節「從 CSV 檔到報告」：長格式、毫秒、缺失的時間、類別共變項。
  - 附錄 B 改用 `ecm()` 的結果檢查。
- 說明頁和訊息裡的舊寫法都改成新寫法。

## 5. birt

### 5.1 新功能

- `ipcr()`：個人參數貢獻迴歸（Arnold 等人，2020）。配適一次最大概似模型，就能同時檢查所有參數對所有共變項的差異，連續和類別共變項都可以。0.1.0 沒有這個函數。
- `dif_tree()` 改為不建議使用，請改用 `ipcr()`。還能用，每個 session 警告一次。
- `plot()`：
  - 沒有分位數效果的模型，`plot(fit)` 會畫個人分數。0.1.0 只印一行訊息。
  - `plot(fit, type = "information")` 可以用在二元和連續指標上。

### 5.2 輸出

同一個預設 ELGM 模型，0.1.0 的 `print(fit)` 有這兩行：

```
    inner: AGHQ (k = 9) per person; outer: AGHQ, k = 9 on lsd2, atr, Gaussian in 32 directions (81 nodes)
Use summary() for the parameter table and quantiles() / plot() for the quantile effects.
```

0.2.0 對應的兩行：

```
    inner: AGHQ (k = 9) per person; outer: AGHQ, k = 9 on log sd(speed), atanh cor(ability, speed), Gaussian in 32 directions (81 nodes)
Use summary() for the parameter table, scores() for the person scores and plot() for a plot of them.
```

- 內部名稱 `lsd2, atr` 改成使用者看得懂的 `log sd(speed), atanh cor(ability, speed)`。
- 模型沒有分位數效果時，不再叫你去看分位數效果。
- 其他行和數值兩版相同。

`estimates(fit)` 的前 2 列，0.1.0：

```
      lhs op rhs         prior       est        sd      q025     q975
1 ability =~  y1 hier(ability) 0.9780937 0.1983888 0.6572386 1.455543
2 ability =~  y2 hier(ability) 1.5592434 0.2816668 1.0943247 2.221637
```

0.2.0：

```
# A tibble: 4 × 8
lhs      op  rhs  prior            est     se  lower  upper
ability  =~  y1   hier(ability)  0.978  0.198  0.657  1.456
ability  =~  y2   hier(ability)  1.559  0.282  1.094  2.222
```

- 欄名統一：ELGM、最大概似、JAGS 都是 `est`、`se`、`lower`、`upper`，和 `as.data.frame(summary(fit))` 相同。0.1.0 的 ELGM 用 `sd`、`q025`、`q975`，最大概似用 `ci.lower`、`ci.upper`。舊欄名仍可取用，每個 session 警告一次。
- 數字印到小數 3 位，物件裡是完整數值。Bayes factor 大於 1e5 時用科學記號，例如 `6.26e+11`。
- 回傳的表格是 tibble。
- ELGM 的參數表最後註明沒有 p 值，以及要用最大概似才有。
- 變異數的先驗寫成 `on SD: dt(0, 1, 3) T(0,)`，說清楚是放在 SD 上的先驗。
- `compare()` 回傳的 `BF_best` 改成數字（0.1.0 是文字），印出來的表不變。

### 5.3 預設值的改變

| | 0.1.0 | 0.2.0 |
|---|---|---|
| `sim_rtirt()`、`sim_hgrm()` 不給種子 | 預設 `seed = 1`，每次資料都一樣 | 預設 `seed = NULL`，跟著 session 的亂數走 |

實際檢查：連續兩次 `sim_rtirt(N = 100, K = 4)`，0.1.0 得到完全相同的資料，0.2.0 不同。0.1.0 的行為會讓 `set.seed()` 沒有作用，模擬研究的每次重複也都用同一份資料。要和 0.1.0 一樣的資料，寫 `seed = 1`。

### 5.4 寫錯時的訊息

| 寫錯的地方 | 0.1.0 | 0.2.0 |
|---|---|---|
| 因素名稱打錯：`abilty =~ ...` 之後寫 `ability ~~ speed` | `residual covariances between indicators are not supported` | 說 `ability` 可能是 `abilty` 打錯，並列出定義過的潛在變項 |
| `birt(model, d, method = "ML")` | `unused argument (method = "ML")` | 說明要寫 `control = rtmb_control(method = "ML")` |
| `rtmb_control(method = "ml")` | `'arg' should be one of "elgm", "aghq", "laplace"` | 接受，等於 `"aghq"`（最大概似） |
| 1–5 分的 Likert 題沒寫 `ordered =` | 不提醒，當成連續變數估計 | 警告，並給 `ordered = "y1"` 的寫法 |
| 警告裡的參數名稱 | 內部名稱，例如 `lpos` | 語法裡的名稱，例如 `log(ability =~ y1)` |
| 時間已經取過 log 又加 `log_rt = TRUE` | 不提醒，再取一次 log | 警告看起來已經取過 log |
| 時間 ≤ 0 | 只說 `response times must be positive seconds; check t1` | 說明有幾個、最小值多少，負值可能代表已經取過 log |
| `ipcr(fit, d)` 傳入整個資料框 | 沒有這個函數 | 報錯，並給 `d[c("x1", "x2")]` 的寫法 |
| lavaan 不接受的語法（`ability ~= y1`、`e(y3) ~ x2`） | R 的語法錯誤 `unexpected '='` | 指出是哪一行、錯在哪裡，並列出可用的運算子 |
| 共變項是因子或文字（`ability ~ gender`） | 只說 `covariates must be numeric without missing values: gender` | 給轉成 0/1 的寫法，缺失值另外報告 |
| speed 的負荷量都固定為 +1 | 不提醒 | 提醒這樣 speed 越高代表越慢 |
| 錯誤訊息開頭的呼叫 | birt 內部的函數，例如 `Error in build_spec(model, data, binary, family, ordered, itemtype)` | 你寫的呼叫，例如 `Error in birt("ability =~ Y1 + y2", d)` |

0.1.0 一欄是實際跑 0.1.0 得到的，只有「警告裡的參數名稱」一列取自學生審查的紀錄。0.2.0 一欄取自實際執行、NEWS 和測試。

### 5.5 文件

- README 從 birt 能回答的研究問題寫起，附「研究問題 → 函數」對照表，並說明預設的估計沒有 p 值、要怎麼得到。
- 使用教學：
  - 第 0 章是給完全新手的導入：安裝、R 的最少基礎、資料長什麼樣子、看懂錯誤訊息、能力和難度與鑑別度、作答時間、第一個完整分析、名詞表。
  - 開頭加了和 README 相同的對照表，以及最快得到第一個結果的路線。
- `?birt`、README 和教學對交叉負荷的規則，改成和程式實際的做法一致。

## 6. 沒有改的

- 兩個套件的 C++ 估計程式。
- birt 的模型語法（lavaan 寫法），以及 `birt()` 的引數。
- 支援的模型和估計方法。下面這些 0.1.0 不能做，0.2.0 仍然不能做：
  - birtRcpp 的 mixture 模型和 GRM。
  - birtRcpp 的 `ecm()` 估計分位數模型（birtRcpp 0.3.0 加入）。
  - birtRcpp 的 `input_data()` 接受缺失值。
- 中文 README（`README.zh-TW.md`）沒有同步更新，內容還是 0.1.0 的。

## 7. 升級時要注意

下面這些舊程式在 0.2.0 的結果或行為會不一樣：

1. `sim_rtirt()`、`sim_hgrm()` 沒給種子時，資料每次都不同。要重現舊資料，寫 `seed = 1`。
2. 表格是 tibble：
   - `e[, "est"]` 得到的是一欄的 tibble，不是向量。改寫成 `e$est`。
   - tibble 沒有 row names。
   - 要舊的樣子，用 `as.data.frame()`。
3. birt `estimates()` 的舊欄名（`sd`、`q025`、`ci.lower` 等）還能用，但會警告。
4. birtRcpp：
   - 多給或拼錯的引數，以前只警告，現在會報錯停下。
   - 長格式資料以前會安靜地算錯，現在會報錯。
5. 印出的數字只到小數 3 位。要看更多位數，直接取值，例如 `fit$ecm$logLik`、`estimates(fit)$est`。

## 8. 還沒處理

- 三個 `refactor` 分支都還沒推上 GitHub。
