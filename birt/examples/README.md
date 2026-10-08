# birt 的範例

每個檔案是一個獨立的 R 程式，可以從頭執行到尾：

```sh
Rscript ex01_quickstart.R
```

| 檔案 | 內容 | 額外需要 | 執行時間* |
|---|---|---|---|
| [ex01_quickstart.R](ex01_quickstart.R) | 快速開始：作答加作答時間的聯合模型，用 ELGM（預設）配適 | | 24 秒 |
| [ex02_models_compare.R](ex02_models_compare.R) | 模型比較：ELGM 的 log 邊際概似，以及最大概似的 AIC／BIC | | 2 分鐘 |
| [ex03_latent_regression.R](ex03_latent_regression.R) | 潛在迴歸：speed 與能力的共變項，以及能力 → speed 的路徑 | JAGS（選用段落） | 2 分鐘 |
| [ex04_quantile.R](ex04_quantile.R) | 分配模型的分位數效果：SHASH 殘差、尺度調節、`quantiles()`、`quantile_test()` 與同時信賴帶 | JAGS（選用段落） | 4 分鐘 |
| [ex05_tables_equations.R](ex05_tables_equations.R) | 摘要表是物件、模型寫成式子、計算寫成步驟；輸出 CSV、LaTeX 與 RTMB 程式 | | 13 秒 |
| [ex06_ordinal_mnlfa.R](ex06_ordinal_mnlfa.R) | Likert 量表加一個總作答時間：調節的等級反應模型（MNLFA）、`hgrm()`、DIF 篩選 | JAGS（選用段落） | 9 分鐘 |
| [ex07_post_hoc.R](ex07_post_hoc.R) | 用已配適的模型算其他量（不必重新配適） | JAGS（選用段落） | 52 秒 |
| [ex08_rtmb.R](ex08_rtmb.R) | RTMB 引擎：同一個邊際概似上的 ELGM 與最大概似，以及最大概似工具（分數檢定、影響函數、DIF 樹、RTMB 程式） | `partykit`（DIF 樹） | 1.5 分鐘 |
| [ex09_correlation_vs_cross.R](ex09_correlation_vs_cross.R) | van der Linden 的相關模型等於一個共同的交叉關係（Cholesky） | | 1.3 分鐘 |

\* 在作者的電腦上（Apple silicon，R 4.5.3，含 JAGS）。沒有裝 JAGS 時，標「選用段落」的部分會自動跳過。

這些檔案也隨套件安裝：`system.file("examples", package = "birt")`。
