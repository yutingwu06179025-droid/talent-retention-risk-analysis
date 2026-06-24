# 39.8% vs 16.1%：被平均值掩蓋的離職熱點
### 人才保留風險量化分析｜以加權衝擊模型取代傳統平均分析

**作者**：吳育庭　|　**工具**：Python (pandas) · Power BI (DAX) · SQL 邏輯
**資料來源**：[IBM HR Analytics Employee Attrition Dataset](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset)（公開資料集，1,470 名員工、34 個變數）

---

## 業務背景與問題陳述

某跨國科技製造公司在台員工約 1,470 人。HR 部門近期觀察到整體離職率為 **16.1%**，雖未明顯偏離產業平均水準，但前線部門主管反映特定崗位的人才流失感受遠比報表呈現的更為嚴重，且資深員工的離職案例有上升趨勢。傳統的平均值報表無法協助管理層辨識具體成因與優先處理對象。

本專案旨在回答三個問題：

1. 哪些員工特徵組合的離職風險被整體平均值低估？
2. 其背後驅動因子為何？
3. 管理層可採取哪些具體且可量化追蹤的行動，優先降低高風險群體的流失？

---

## 分析方法

不直接對資料跑探索式分析，而是先建立假設樹，再逐一用資料驗證：

| 假設線 | 對應變數 |
|---|---|
| 薪酬公平性 | MonthlyIncome、PercentSalaryHike |
| 工作負荷與生活平衡 | OverTime、WorkLifeBalance |
| 職涯發展停滯 | YearsSinceLastPromotion、JobLevel |
| 管理關係 | YearsWithCurrManager、RelationshipSatisfaction |

驗證流程：① 資料清洗與完整性檢查（Python / pandas）→ ② 單變數風險差異分析 → ③ 複合風險分群驗證 → ④ DAX 加權風險模型建構（Power BI）→ ⑤ 轉譯為可執行建議與 KPI 監測指標。

---

## 關鍵發現

**1. Sales Representative 的離職率為全公司平均的 2.5 倍，是整體報表完全無法看見的高風險職位**
- 全公司整體離職率：16.1%；Sales Representative 職位離職率：**39.8%**（n = 83）
- Manager、Research Director 等資深職位離職率僅 2.5%–4.9%，顯示風險高度集中於特定崗位，而非全面性問題

**2. 加班是單一最強的離職驅動因子，且屬於改善成本相對低的「Quick Win」**
- 有加班員工的離職率為 **30.5%**，無加班員工僅 10.4%，差距近 3 倍

**3. 「加班 + 升遷停滯」的複合風險群體，是傳統平均值報表最容易低估的族群**
- 加班且 3 年以上未獲升遷的員工，離職率達 **28.0%**；對照組（無加班、近期有升遷）僅 11.1%
- 低工作滿意度且薪資低於全公司中位數的員工，離職率達 **26.7%**，為平均值的 1.7 倍

---

## 加權離職風險模型（DAX 邏輯）

不以單一變數判斷風險，而是將四項風險因子依據其單變數影響力的相對大小設定權重，組合成單一風險分數：

```
Risk_OverTime         = IF(OverTime = "Yes", 1, 0)
Risk_PromotionStale   = IF(YearsSinceLastPromotion >= 3, 1, 0)
Risk_LowSatisfaction  = IF(JobSatisfaction <= 2, 1, 0)
Risk_LowIncome        = IF(MonthlyIncome < MEDIAN(全公司MonthlyIncome), 1, 0)

Weighted_Risk_Score =
    Risk_OverTime        * 0.35 +
    Risk_PromotionStale  * 0.25 +
    Risk_LowSatisfaction * 0.25 +
    Risk_LowIncome       * 0.15
```

權重設定依據：先以單變數分析觀察各因子造成的離職率差異幅度，差異幅度越大者賦予越高權重（加班的單變數影響最大，故權重最高）。

---

## 建議行動與時間軸

| 時間軸 | 行動方案 | 對應發現 |
|---|---|---|
| **Quick Win**（0–3 個月） | 針對 Sales Representative 及高加班部門檢視人力配置與排班政策 | 發現 1、2 |
| **中期**（3–12 個月） | 建立明確升遷時間表與內部薪酬檢視機制，鎖定資深、高績效但長期未獲升遷之員工 | 發現 3 |
| **策略性**（12 個月以上） | 導入結構化離職面談問卷，持續累積資料以支持縱貫分析與因果推論 | 方法論限制 |

---

## 預估財務效益

以 Sales Representative 職位為例：若離職率自 39.8% 降至接近全公司平均水準（約 16%），預估每年可減少約 20 人次離職。參考業界估算之單次離職總成本（招募、訓練、生產力空窗）約為年薪之 0.5–1.5 倍，初步估算每年可節省離職相關成本約 31 萬至 93 萬元（以本資料集薪資單位計算，實際應用於真實客戶時需以當地薪資水準與離職成本結構重新校準）。

---

## 方法論限制

- 本分析為橫斷面資料，僅能呈現相關性與風險排序，無法做因果推論
- 建議客戶導入縱貫資料追蹤（定期離職面談、長期追蹤同一群員工），以驗證行動方案之長期成效
- 財務效益估算為示意性計算，實際導入真實客戶專案時，須以該公司實際薪資結構、招募成本與產業基準重新校準

---

## 檔案結構

```
├── README.md                          # 本檔案
├── executive_summary.pdf              # 一頁式高層摘要
├── data_cleaning_and_analysis.ipynb   # Python 資料清洗與分群驗證
├── dashboard/
│   └── talent_retention_dashboard.pbix  # Power BI 互動儀表板
└── data/
    └── cleaned_hr_employee_attrition.csv
```
