# birtExamples

R 套件 **birt** 與 **birtRcpp** 的教學和範例程式（繁體中文）。

Tutorials and example scripts for the R packages **birt** and **birtRcpp** (in Traditional Chinese).

| 套件 | 做什麼 | 主要估計法 |
|---|---|---|
| birt（BIvariate Response Time Modeling with 'lavaan'-Style Syntax） | 用 lavaan 風格的語法寫 IRT 與作答時間模型，交給 RTMB 或 JAGS 配適 | ELGM（近似貝氏）；同一引擎也給最大概似 |
| birtRcpp（BIvariate Response Time Modeling with 'Rcpp'） | 博士論文的現成模型（作答 + 作答時間），用 `ecm(dat, model = "cross")` 這樣的寫法選模型；適合教學，能對照推導 | ECM；從 ECM 的解出發的 Gibbs |

## 教學

在 GitHub 上直接點 `.md` 檔就能讀。`.qmd` 是原始檔，可以用 Quarto 自己重新產生。
`.md` 用的是下面的格式（GitHub 的標題錨點、pipe 表格、`$` 數學式）；在 `.qmd` 的 `format:` 底下加上這一段，再執行 `quarto render 檔名.qmd --to commonmark+pipe_tables+tex_math_dollars --toc`：

```yaml
  commonmark+pipe_tables+tex_math_dollars:
    from: markdown+gfm_auto_identifiers
    wrap: none
    toc-depth: 2
```

Quarto 把 callout 寫成 `<div>` 包住的引用；這裡的 `.md` 再把它們改成 GitHub 的提示框（`> [!NOTE]`、`> [!WARNING]`）。

| 教學 | 內容 | 讀 |
|---|---|---|
| birt 使用教學 | 語法、ELGM、模型比較、潛在迴歸、分位數與訊息量、最大概似工具、調節 GRM、JAGS 引擎 | [birt/tutorial/birt_tutorial.md](birt/tutorial/birt_tutorial.md) |
| birtRcpp 入門 | 用現成的模型分析作答與作答時間：先用 ECM，再用 Gibbs | [birtRcpp/tutorial/birtRcpp_intro.md](birtRcpp/tutorial/birtRcpp_intro.md) |
| birtRcpp 積木（已不建議使用） | 用 `block_model()` 宣告模型，用 ECM、Gibbs、CAVI 估計。`block_model()` 與 `vi()` 還能用，但不再加新功能 | [birtRcpp/tutorial/birtRcpp_blocks.md](birtRcpp/tutorial/birtRcpp_blocks.md) |

同樣的教學也有單一檔案的網頁版，放在 [`docs/`](docs/)：下載後用瀏覽器打開，或在 repo 設定裡把 GitHub Pages 指到 `docs/` 資料夾。

建議的讀法：

1. 想用語法寫模型：讀《birt 使用教學》。
2. 想用現成的作答時間模型、要快：讀《birtRcpp 入門》。
3. 想看估計器怎麼由完全條件分配組成：讀完《birtRcpp 入門》再讀《birtRcpp 積木》（宣告式的 `block_model()` 已不建議使用；用匯出的積木函數自己寫抽樣器，見範例 ex07–ex12）。

## 從舊寫法改過來

2026-10-28 起，birtRcpp 改用 `ecm(dat, model = "cross")`、`gibbs(dat, model = "cross", init = fit)` 這樣的寫法，兩個套件的表格都改成 tibble。舊寫法大多還能用（會出現「已不建議使用」的警告）；新舊寫法逐項對照見 [MIGRATION.md](MIGRATION.md)。

## 範例

每個範例是一個獨立的 R 檔，可以直接執行，例如：

```sh
Rscript birt/examples/ex01_quickstart.R
```

- [birt 的範例](birt/examples/)（9 個）
- [birtRcpp 的範例](birtRcpp/examples/)（13 個）

兩個資料夾的 README 列出每個範例做什麼、需要哪些額外套件，以及在作者電腦上的執行時間。

## 安裝

需要 R 4.1 以上。

```r
install.packages(c("birt", "birtRcpp"))            # CRAN 上架之後

# GitHub 上的版本
# install.packages("remotes")
remotes::install_github("jiewenTsai/birt")
remotes::install_github("jiewenTsai/birtRcpp")     # 需要 C++ 編譯器
```

套件原始碼：[jiewenTsai/birt](https://github.com/jiewenTsai/birt)、[jiewenTsai/birtRcpp](https://github.com/jiewenTsai/birtRcpp)。

選用的套件與程式：

| 用途 | 需要 |
|---|---|
| birt 的 JAGS 引擎（spike-and-slab、ALD 分位數） | [JAGS](https://mcmc-jags.sourceforge.io/) 4.3 以上，以及 `install.packages(c("R2jags", "rjags"))` |
| 分數檢定 `score_test()` | `strucchange`、`sandwich`（`ipcr()` 不需要額外套件；已不建議使用的 `dif_tree()` 另需 `partykit`） |
| birtRcpp 範例 11–13 的資料 | `LNIRT` |

## 版本

這裡的輸出是用下列版本產生的（2026-10-28；birtRcpp 入門在 2026-10-29 重新產生）：

- R 4.5.3（macOS，aarch64）、Quarto 1.10.19
- birt 0.1.0、birtRcpp 0.1.0
- RTMB 2.0、lavaan 0.7.2、JAGS 4.3.2、R2jags 0.8.9、Rcpp 1.1.1、RcppArmadillo 15.2.4.1、dplyr 1.2.0

模擬資料都有設定亂數種子，但 JAGS 與 Gibbs 的結果在不同平台上可能有很小的差異；執行時間會隨電腦不同。

## 作者與授權

Jie-Wen Tsai <tsai.jiewen@gmail.com>

GPL-3，與兩個套件相同（見 [LICENSE](LICENSE)）。
