# birtRcpp 的範例

每個檔案是一個獨立的 R 程式，可以從頭執行到尾：

```sh
Rscript ex01_quickstart.R
```

## 現成的模型

| 檔案 | 內容 | 額外需要 | 執行時間* |
|---|---|---|---|
| [ex01_quickstart.R](ex01_quickstart.R) | 從資料框到交叉關係模型的表：先用 ECM（估計值、標準誤、AIC／BIC），再從 ECM 的解出發做 Gibbs | | 5 秒 |
| [ex02_models.R](ex02_models.R) | 五個現成模型，用 ECM（最大邊際概似）配適模擬資料 | | 2 秒 |
| [ex03_quantiles.R](ex03_quantiles.R) | 分位數模型：每個分位數一個模型（ALD 工作概似），只能用 Gibbs；平均數模型用 ECM 當參照 | | 29 秒 |
| [ex04_ecm.R](ex04_ecm.R) | ECM：最大邊際概似、小樣本用 MAP，以及從 ECM 的解出發的 Gibbs 鏈 | | 4 秒 |
| [ex05_compare.R](ex05_compare.R) | 比較潛在結構：AIC／BIC／LR 檢定（ECM）與邊際 LOOIC（Gibbs） | `loo`（LOOIC） | 12 秒 |
| [ex06_tables_equations.R](ex06_tables_equations.R) | 取出摘要表、把模型寫成論文用的式子 | | 2 秒 |

**ex03 的注意事項：** 在右偏的模擬資料上，q = .9 的 ALD 工作後驗是多峰的，所以會出現 R-hat 警告（約 19），q = .9 的 ρ 也和其他分位數差很多。這是範例刻意展示的情況，檔案裡的註解有說明：先看 `convergence()`，偏態資料上不要解讀極端分位數。

## 用積木自己寫

| 檔案 | 內容 | 額外需要 | 執行時間* |
|---|---|---|---|
| [ex07_blocks.R](ex07_blocks.R) | 用積木在 R 裡寫一個抽樣器：`gibbs(model = "cross")` 的 Gibbs，一步一步寫 | | 2 秒 |
| [ex08_testlet.R](ex08_testlet.R) | 題組模型（Bradlow, Wainer & Wang, 1999） | | 14 秒 |
| [ex09_mirt_rt.R](ex09_mirt_rt.R) | 補償型多向度 IRT 加作答時間（Man, Harring, Jiao & Zhan, 2019） | | 25 秒 |
| [ex10_conditional_dependence.R](ex10_conditional_dependence.R) | 作答時間與作答的條件相依（Bolsinova, De Boeck & Tijmstra, 2017） | | 9 秒 |
| [ex11_lnirt.R](ex11_lnirt.R) | LNIRT 的聯合模型，在證照考試資料上重現 Fox & Marianti (2017) 的表 2 | `LNIRT`（資料） | 1.7 分鐘 |
| [ex12_variable_speed.R](ex12_variable_speed.R) | 作答中途改變的速度（Fox & Marianti, 2016），用 Amsterdam Chess Test 資料 | `LNIRT`（資料） | 28 秒 |
| [ex13_preknowledge.R](ex13_preknowledge.R) | 用作答時間偵測事先看過題目（Sinharay, 2020 的表 2） | `LNIRT`（資料） | 3 秒 |

\* 在作者的電腦上（Apple silicon，R 4.5.3）。

這些檔案也隨套件安裝：`system.file("examples", package = "birtRcpp")`。ex12 會從那裡讀 ex11 的抽樣器，所以要先裝好 birtRcpp。
