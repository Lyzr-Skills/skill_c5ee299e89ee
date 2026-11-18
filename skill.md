---
name: outlier_skill
description: >-
  Use this skill whenever a planner asks about the Process Outlier Correction page in the Demand Analyst V6 / Demand Analysis V6 Anaplan model, or needs help with an outlier, spike, dip, manual correction, corrected history, why a week was flagged, how correction settings resolve, or what a page control or alert means. It explains the page and its model lineage using the cataloged modules below, including configuring the model, correcting demand history, classifying and segmenting demand, and managing analyst workflows through alerts and UX controls. Prefer this skill even when the user asks in ordinary planner language rather than module names.
compatibility: Requires Anaplan catalog tools for fresh model checks; read-only by design.
---

# Process Outlier Correction — Demand Analysis V6

## Purpose and safety

Explain the Process Outlier Correction page in the **Demand Analysis V6** model in planner language. Ground explanations in the cataloged module and line-item metadata here. The user requested read-only analysis: inspect metadata and read data only when useful; **do not write cells, create list members, launch imports/processes, or otherwise change model data**.

This skill documents the model structure, not a live page snapshot. Never present catalog metadata as proof of current values, alert counts, active selections, or a particular product/week's status. For a live answer, bind the correct model and use read-only cell inspection. If the user asks to change a setting or correct demand, explain the route and ask for explicit authorization before considering any write; this skill itself remains read-only.

## Model identification and evidence

The target model is **Demand Analysis V6**, in workspace **Agentic Enterprise - Supply Chain**. The cataloged target model id is `B990328F440946D2811A082426C51EAC`; workspace id is `3deff6878f494b89b0a171500228b68d`. Reconfirm with the model catalog/context tools in a fresh session before any model-scoped call. Do not silently use the previously bound PMGT inventory model or the separate production model named `5.2. Demand Analysis`.

The model is weekly-grain for the relevant correction series. `OCA200 - Outlier Explainability`, `OCA000 - Outlier Correction Calculations`, and `SHC100 - Statistics and History Correction` include Time at Week grain. The requested modules were inspected with `catalog_line_items(detail="full")`; use the module map below as the model-backed reference. Where a line item is a section header, it organizes a group rather than holding a planner value. Where catalog metadata returns formulas or notes, prefer those over assumptions from a line-item name.

## Planner-facing explanation

The page supports **correcting demand history**: it computes statistical reference values and limits, identifies outlying history, can automatically replace eligible unexplained spikes/dips in the system-corrected series, and then allows a **manual correction** to take precedence in final corrected history. Corrected history is not the same as source/selected history. It is an output series that may combine unchanged history, system correction, promotion treatment, and an analyst-entered correction.

Use **outlier**, **spike**, and **dip** plainly:
- A spike is demand above the applicable upper limit.
- A dip is demand below the applicable lower limit.
- A week between limits is not outside-limit outlier history under this flagging test.
- A detected breach is not necessarily replaced: correction must be enabled and valid, and event explanations / promotion rules may affect whether the system corrects it.

Do not describe a demand-sensing breach as automatically identical to the correction flag. The `SHC100` correction breach flags and `OCA200.Outlier`'s cataloged demand-sensing breach composition are distinct lines, even though both use upper/lower breach concepts.

## Three settings layers: GL, Edit, EF

Explain the three layers in this order:

1. **GL (global)** — `SHC000 - Statistics and History Correction Parameters` exposes `GL Correct From`, `GL Outlier Magnitude`, `GL Correct What`, `GL History Correction Adjustment`, `GL Period by Period`, `GL Correct Promotion Outliers?`, and `GL Correction of promotion outliers`. These are sourced from `PRM120 - Global Parameters: Effective`, which resolves the global input/default settings. GL is the common fallback across combinations.
2. **Edit (leaf/user override)** — in the same `SHC000` module, `Correct From`, `Outlier Magnitude`, `Correct What`, `Correct To`, `Period by Period`, `Correct Promotion Outliers?`, and `Correction of promotion outliers` are editable at product-customer leaf combination. The edit cells have validity access logic. A populated edit value generally takes precedence over its GL counterpart. Blank means use the applicable fallback; it does not mean zero/disabled unless the actual formula says that.
3. **EF (effective)** — `SHC000` calculates the final resolved values used by correction logic: `EF Correct From`, `EF Outlier Magnitude`, `EF Correct What`, `EF Correct To`, `EF Period by Period`, `EF Correct Promotion Outliers?`, and `EF Correction of promotion outliers`. EF combines eligible demand-class/seasonality-specific parameters, leaf edits, and global settings as the formulas prescribe, and applies validity/effectivity/history gates and special substitutions.

Important precedence nuance: for correction basis/type/target and associated class-specific values, the `CALCEF` intermediate lines use the demand-class-specific `DC ...` value when it is present and `HDA220 - Strength of Seasonality Parameters`.`Use Seasonality & Demand Classification for Hist Cor` is enabled; otherwise the fallback is user Edit when populated, then GL. `EF Correct From` can turn correction Off when history is insufficient or the current period is not effective, and substitutes the configured trend-seasonal fallback if decomposition cannot run. `EF Correct To` can likewise substitute the configured trend-seasonal target. `EF Outlier Magnitude` uses a leaf `Outlier Magnitude` when nonzero, otherwise eligible demand-class magnitude, otherwise GL magnitude, and is multiplied by the valid-combination counter. Explain this nuanced resolution rather than claiming Edit always wins in every scenario.

`PRM100` is the global input/configuration layer. `PRM120` is the effective resolved global layer (using defaults where inputs are blank/zero as defined per line). `HDA230` provides five demand-classification × seasonality-specific history-correction controls; `SHC000` consumes those settings into the calculation-ready global, leaf-edit, and effective layers.

## How a week gets flagged

Explain the correction flag step-by-step, without inventing a numerical cutoff:

1. The combination must be valid and the week must be within the relevant effective history window. `SHC000.ISEnoughHistoryforCorrection` checks effective history periods against `PRM120.Min Periods History`; `SHC100.ISValidforCorrection` requires correction basis on, enough history, and an effective history period.
2. The effective correction basis chooses a reference series: mean, linear trend, or trend-seasonal decomposition (with configured substitution when decomposition cannot be used). `SHC100` calculates the corresponding reference and dispersion measure.
3. The effective `Outlier Magnitude` scales dispersion to form control limits. For Mean, the formulas are Mean ± (Standard Deviation × magnitude); for Trend, Trend ± (RMSE Trend × magnitude); for Trend/Seasonal, Trend Seasonal ± (RMSE Residual × magnitude). The week-level formulas are in `SHC100.Upper Limit` and `Lower Limit`.
4. `SHC100.ISUpperLimitBreached` is true when valid history is **greater than** Upper Limit; this is a spike. `ISLowerLimitBreached` is true when valid history is **less than** Lower Limit; this is a dip.
5. `ISOutsideEffectiveLimits` combines the upper/lower breach according to effective `Correct What` (spikes, dips, or both). Therefore only configured directions count. The correction `ISCorrectionNeeded` then gates this breach with a non-Off basis and the selected period-by-period/global correction logic.
6. Explainability and promotions are another check. `OCA200 - Outlier Explainability` accepts weekly numerical explanations such as MOQ, Customer Won/Lost, Marketing Campaign, Promotion, Stockout, Price Change, competitor events, and free-text `Manual Explaination`. `Explained Outlier?` is true if any supported reason is entered. `SHC100.Explained Outlier?` and `Unexplained Outlier?` carry that explanation state into correction logic. Promotion periods may use the separate promotion-outlier flag/method.
7. When correction is required, `System Corrected History (U)` chooses the effective correction target (e.g. mean, trend, limit, or trend-seasonal), with special promotion logic. `Manual Correction (U)` is an editable value; `Corrected History (U)` uses it when nonzero, otherwise the system-corrected series.

Be precise: the cataloged `OCA200.Outlier` formula is `SHC100.DemandSensingUpperBreach + SHC100.DemandSensingLowerBreach`, while `SHC100.ISOutsideEffectiveLimits` is the correction's configurable direction-aware combined breach flag. When asked why a particular week appears in an OCA200 view, inspect the exact line and week context rather than substituting one flag for the other.

## Module and line-item map

Descriptions below summarize the full line-item catalogs relevant to the page. Section headings are listed to explain workbook/page organization. Abbreviations are retained where they are native model terminology. If asked for an exhaustive raw blueprint, fetch the catalog again and paginate; this map is a semantic guide, not a substitute for every property in every Blueprint row.

### PRM100 - Global Parameters: Input
Global administrator configuration inputs, mostly undimensioned. It contains section headers and editable values; blank or zero values often request the default, depending on the individual formula. Main groups and planner-relevant lines:
- **Edit Parameters** (header) groups model-wide settings.
- **List Maintenance**: `Creation Threshold %` controls automatic creation threshold during list reconciliation/imports.
- **User List Configuration**: `Email Separator` and `Name First` control user naming/email construction.
- **Chart Visibility**: `View Years History`, `View Years Future`, and `View Period Type` configure global chart horizon and display grain.
- **Hierarchy Filters**: `Customer Filter` and `Product Filter` define default global hierarchy scope.
- **Product Effectivity**: `Product Effective Basis` determines when products become effective.
- **History Correction / Outlier Identification**: `Correct Promotion Outliers?`, `Correction of promotion outliers`, `Correct From`, `Outlier Magnitude`, `Correct What`, `Correct To`, `Period by Period`, `Min Periods History`, `Substitute From Trend Seasonal`, and `Substitute To Trend Seasonal`. These govern whether promotion outliers are treated, the reference/baseline, magnitude sensitivity, which direction(s) are in scope, target value, correction scope, minimum history, and seasonal fallback.
- **ABC/XYZ** inputs select active segmentation and update basis, product/customer calculation periods and data source, C-VAR method, formatting, etc.; these configure classification/segmentation rather than being weekly correction edits.
- **Input Demand Classification** includes `Average Demand Interval Cut-Off` and `C-VAR Cut-Off`, the thresholds that shape smooth/intermittent/erratic/lumpy pattern classification.
- **Demand Sensing** includes `Number of Periods to Analyze`, `Alert: Percentage of Outlier Periods (%)`, portfolio alert thresholds, and alert review thresholds. These govern demand-sensing alert scope and monitoring.
- **Seasonal Indices**, rounding, units/exchange rates, analyst comparison, algorithm history thresholds, trend windows (`T1`, `T2`, `T3`, standard trend), forecast-accuracy windows, and classification window start dates are also present. They are global configuration, not the per-week correction result.

### FLT102 - Filter Settings Demand - User Selection
User- and page-specific time-view controls, dimensioned by `FL003 - Pages for Time Filters #` and `Users`:
- `Time View Settings` selects a view; `EF Time View Settings` resolves user choice/default and drives selected range; `Edit Time View Settings?` gates user-editable controls.
- Inputs `View Years History`, `View Years Future`, `View Period Type`, `History Only`, `Future Only`, and `Table: Totals Only` configure period display. `Periods History` / `Periods Future` derive counts from years and period type.
- `Current Period`, `Current FY`, `Period Number`, and FY/rolling-year start/end calculations anchor the view; `Start Period` and `End Period` calculate time boundaries; `EF Start Period`/`EF End Period` provide final bounds.
- `EF View Years History`, `EF View Years Future`, `EF View Period Type`, `EF History Only`, `EF Future Only`, `EF Periods History`, `EF Periods Future`, and `EF Table: Totals Only` are resolved settings. `ISGlobalViewSelected` identifies global/default view selection; `Page Item` links settings to page context.
- Headings (`Edit View Settings`, `Effective View Settings`, `Calculations`, `Booleans`) group controls. This module configures the time view, not the statistical outlier test itself.

### PRM120 - Global Parameters: Effective
Resolved, model-wide settings. It generally takes `PRM100` input when set and otherwise a `PRM110` default (some settings have their own zero/blank rules or minimum/clamp logic). It contains headings and effective values across configuration domains. Correction-relevant outputs include `Correct From`, `Outlier Magnitude`, `Correct What`, `Correct To`, `Period By Period`, `Correct Promotion Outliers?`, `Correction of promotion outliers`, `Min Periods History`, `Substitute From Trend Seasonal`, and `Substitute To Trend Seasonal`. Demand sensing includes effective analysis period count (capped to available history), percentage-of-outlier-period alert threshold, portfolio alert bounds, and reviewed-alert bounds. Other effective groups resolve hierarchy filters, product effectivity, ABC/XYZ, demand classification thresholds, seasonal indices/53rd-week handling, algorithm history thresholds, and trend/FA windows. Use this module to explain the active **global default**, not a product/customer-specific final EF result.

### HDA230 - History Correction Parameters
Dimensioned by `PL240 - Seasonal or Non-Seasonal` × `SML004 - Demand Classification`; contains five controls for each combination:
- `Correct From`: reference basis for detection.
- `Outlier Magnitude`: deviation threshold multiplier.
- `Correct What`: direction/type to correct (spikes, dips, or both according to the selected list item).
- `Correct To`: replacement/adjustment target.
- `DC Period by Period`: per-period versus broader-window correction choice.
These are the demand-classification/seasonality-specific layer. `SHC000` looks them up using the seasonality assignment and effective demand classification when the “Use Seasonality & Demand Classification for Hist Cor” switch is enabled; do not assume they unconditionally override GL/Edit.

### SHC000 - Statistics and History Correction Parameters
Dimensioned by customer and product forecast lists. This module owns the settings-resolution ladder and eligibility gates:
- **Global (GL)**: `GL Correct From`, `GL Outlier Magnitude`, `GL Correct What`, `GL History Correction Adjustment`, `GL Period by Period`, promotion correction flag/method. They pass through PRM120.
- **Demand Classification (DC)**: `DC Correction From`, `DC Outlier Magnitude`, `DC Correct What`, `DC Correct To`, `DC Period by Period`; these look up HDA230 using seasonality and demand classification.
- **Edit**: leaf-level editable `Correct From`, `Outlier Magnitude`, `Correct What`, `Correct To`, `Period by Period`, `Correct Promotion Outliers?`, and `Correction of promotion outliers`. Combination validity drives access.
- **Calculated intermediate**: `CALCEF Correct From` and `CALCEF Correct To` combine eligible DC, Edit, and GL selections before fallback substitutions. `EF ...` outputs provide final basis, magnitude, direction, target, period-by-period, and promotion settings.
- **Control/diagnostics**: `ISEnoughHistoryforCorrection`, `ISNotEnoughHistoryforCorrection`, `ISEffectiveCurrentPeriod`, `ISNotEffectiveCurrentPeriod`, `ISCorrectionBasisOn/Off`, basis/type/target booleans, period-by-period booleans, and trend-seasonal-no-decomposition indicators gate calculations and explain fallback.
- **Alerts**: `IND History Correction Alert Text`, `History Correction Alert Text`, and demand-sensing alert counterparts indicate why calculations may be unavailable (not effective, insufficient history, insufficient decomposition history, or possible correction). Numeric indicators map to display text.
- **Period-by-period numeric indicators** support calculation/formatting. `NumberofProductEffectivePeriods` counts available effective periods. Section headings are organizational only.

### SHC100 - Statistics and History Correction
Weekly calculation engine, dimensioned by Time, customer forecast list, and product forecast list. Key line-item families:
- **Input/reference series**: `Chained History (U)` is the post-chaining source history in units. `Selected History (U)` is baseline pre-correction history. `One` supports counts.
- **Statistics**: `Calc Mean (U)` / `Mean (U)`; deviations (`x - x bar`, `x-x bar2`), sums, `Standard Deviation`, and `C-Var (Mean)`; regression inputs (`xy`, `x2`, sums, `CALC1m/CALC2m`, `m`, `CALC1b/CALC2b`, `b`, `Trend (U)`, regression equation); trend error (`x-xbar2 (Trend)`, `Calc RMSE (Trend)`, `RMSE (Trend)`, `C-Var (Trend)`); Trend Seasonal (U) and Residual RMSE references. These calculate baseline and dispersion for limits.
- **Validity and limits**: `ISValidforCorrection` gates by valid combination, active basis, sufficient history, and effective period. `Upper Limit` / `Lower Limit` use effective basis, magnitude, and dispersion. `ISUpperLimitBreached`, `ISLowerLimitBreached`, `ISOutsideEffectiveLimits`, period-by-period variants, `CALC1ISCorrectionNeeded`, `CALC2ISCorrectionNeeded`, and `ISCorrectionNeeded` express breach and correction conditions.
- **Correction outputs**: `ISPromotionPeriod`, `System Corrected Promotion History (U)`, `System Corrected History (U)`, editable `Manual Correction (U)`, and final `Corrected History (U)`. `Corrected History (R)` converts units to revenue. Manual correction is used when nonzero; otherwise final history follows system correction. `Explained Outlier?` / `Unexplained Outlier?` carry rationale state.
- **Diagnostics/graph/KPIs**: corrected mean/deviation/trend/statistics and correction-window counts; DemandSensing upper/lower breach counts; `CountOutsideEffectiveLimits`; graph-only series; period limit range; optional exports/KPI outputs. Section headings group the families.

### OCA000 - Outlier Correction Calculations
Calculation support for the outlier-analysis summary, weekly grain, product/customer context:
- `Product` and `Customer` expose hierarchy codes; `Last Effective Index`, `First Effective Index`, `ISProdEffective (HISTFUT)`, `Prod Effective Period Count (HISTFUT)`, and `Number of Effective History Periods` delimit/count the effective FA-trend window.
- `xy`, `x2`, `sum of x`, `sum of y`, `sum of xy`, `sum of x2`, `CALC1m`, `CALC2m`, `m`, `CALC1b`, `CALC2b`, and `b` form a least-squares regression; `Trend (U)` is fitted trend; `Calc Mean (U)`, `Mean FA`, `Mean Bias`, and `Mean Target FA` provide summary metrics. This is supporting calculation metadata, not the page's weekly correction-flag source.

### OCA200 - Outlier Explainability
Weekly product-customer-period explainability and visual context:
- `Outlier` is cataloged as the sum of `SHC100.DemandSensingUpperBreach` and `DemandSensingLowerBreach` for the period.
- Editable numeric reason counts: `MOQ`, `Customer Won / Lost`, `Marketing Campaign`, `Promotion`, `Stockout`, `Price Change`, `Competitor Promotion`, `Competitor Stockout`, `Competitor Pricing Change`, `Competitor Marketing Campaign`; free-text `Manual Explaination` captures other causes.
- `Explained Outlier?` is true if any reason count is nonzero or free-text explanation is present; `No of Explained Outliers` converts this to a count for aggregation.
- `Upper Limit`, `Lower Limit`, and `History` show thresholds and corrected-history context. These make a flagged history point explainable; they do not independently prove automatic correction occurred.

### OCA500 - Compound Alerts
Configuration by `LIS: OCA100 - Outlier Correction Alerts` for compound alert scoring:
- `Input Points` gives a KPI's raw weight; `KPI Usage` selects aggregation; `Points Total` totals weights by evaluation aggregation; `Rebalanced Points` normalizes active weights to a 100-point basis.
- `Low is Good` tells the alert calculation whether lower KPI values indicate good performance. These determine weighted compound alert contribution, not the week's statistical upper/lower test.

### OCA520 - RAG Thresholds
Dimensioned by alert KPI × KPI evaluation category:
- Inputs `Threshold Start Input` and `Range Width` configure performance bands; `Low is Good` sets direction; `DCA Start Input` and `Cumulate 1=Good, 2=Neutral, 3=Bad` help calibrate/order categories.
- `Threshold End` and `Threshold Start` compute band edges; `EF Lower` is effectively the lower edge (as named in the catalog this line item is `Threshold Start`; do not invent an EF output not cataloged). `Points for Category`, `Previous Categories Cumulation`, and `To Points` allocate/accumulate priority points by RAG category.
RAG thresholds classify KPI performance for compound priority. They are not the outlier magnitude or correction limits.

### OCA530 - Priority to Include to Review
- `Levels to Include in To Review` is the per-priority setting controlling which levels enter the review queue.
- `Include?` compares a product-customer pair's `OCA100.Priority (Compound Alerts)` to a priority level and requires that level's inclusion setting.
- The `To Review` heading groups the controls. OCA100 combines this automated inclusion with manual override and clear-review actions.

### UXC210 - Outlier Analysis ABC Filter Input
Per-user filter inputs for product and customer ABC categories: `Apply Product Filter?` / `Apply Customer Filter?`; selection checks determine whether any category is selected; `Selected Items Product` / `Selected Items ABC Customer` expose selected category items. An empty selection is all-pass (as indicated by checks/formulas/notes). This is a filter, not a correction setting.

### UXC220 - Outlier Analysis XYZ Filter Input
Per-user XYZ-category filter: `Apply Filter?` for a category, `XYZ Product Selection Check` to detect a selection, and `Selected Items` to pass selected XYZ categories to the list-filter output. No selection is treated as all-pass.

### UXC230 - Outlier Analysis Demand Classification Filter Input
Per-user demand-classification filter: `Apply Filter?`, `Selection Check`, and `Selected Items` pass selected demand classes to `UXC250`; selection logic supports all-pass when no categories are chosen.

### UXC240 - Outlier Analysis Priority Filter Input
Per-user review-priority filter: `Apply Filter?`, `Selection Check`, and `Selected Items` for `PL210 - Alert Priority`. It scopes which priority tiers are visible in the review list.

### UXC260 - Outlier Analysis Value Filter Input
Per-user numeric range controls across `UXL200 - UX Controls Outlier Analysis List`: `Show Values Above`, `Show Values Below`, and `Apply Filter?`. `HasInput?` checks whether either bound is populated; `Filter Check` requires both enabled and an input; `Filter Check Any` collapses to per-user active status. `Filter Settings` is a section heading. This supports value-based filtering of the analysis table, not statistical breach limits.

### UXC270 - Outlier Analysis Value Filter Output
This is the user-specific product/customer/value-list mask/collection output driven by UXC260 controls and the values mapped for the outlier analysis list. Use it to explain how value filtering is propagated; do not call it an outlier calculation.

### UXC280 - Outlier Analysis Value CF Input
Per-value-list conditional-formatting inputs: `Apply CF?`, `Lower Threshold`, `Higher Threshold`, and computed `EF Lower Threshold` (clamps the effective lower value to the higher threshold when entered in reverse). `CF Lower Threshold>Higher Threshold` flags the invalid ordering. These color-formatting thresholds are distinct from SHC100 statistical outlier limits.

### UXC290 - Outlier Analysis Value CF Output
A mask/collection and threshold-comparison output that supplies conditional-formatting indicators for mapped outlier-analysis values. `UXC280` controls whether and how values are color-coded; these visual indicators are not correction decisions.

### UXC310 - Outlier Analysis Boolean Filter Input
Per-user boolean/Yes-No filter controls for `UXL300 - UX Controls Outlier Analysis Boolean`: `Yes/No?` is the selected entity; `EF Yes/No?` converts it to a boolean; `EF Yes/No? - Value` converts to 1/0; `HasInput?`, `Apply Filter?`, and `Filter Check` determine whether a selected boolean filter is active; `Filter Check Any` is the per-user active flag. `Inputs` and `Filter Settings` are section headings.

### UXC320 - Outlier Analysis Boolean Filter Output
The corresponding list/value mask output consumes UXC310 checks and the boolean-to-list mapping; `Check`, `Show?`, `Counter`, and `Collect` support filtering displayed product/customer combinations. It controls the UX result set rather than changing correction calculations.

## Alerts, priority, manual workflow, and UX controls

Distinguish three questions:
- **Was the week statistically outside limits?** Inspect `SHC100` validity, upper/lower breach and effective correction-type lines for the exact period and combination.
- **Why might it have breached / was it explained?** Inspect `OCA200` reason inputs, `Explained Outlier?`, limit lines, and history context. A planner explanation records a cause; it does not mean the underlying period was mathematically not a breach.
- **Will the combination enter the analyst queue?** Inspect `OCA100` compound priority, `OCA530` inclusion configuration, `To Review`, `Override To Review`, and `Clear Review`, together with user filters/UX controls.

`OCA100` summary examples from the full catalog include product/customer ABC, product XYZ, demand classification, Mean Target FA, Mean FA, their differences, Most Recent Period FA, Mean Bias, FA Trend, number of outliers and unexplained outliers, seasonal classification, numeric classification/formatting indicators, Points Scored, Priority (Compound Alerts), To Review and its manual override/clear actions, and user-specific final filtering. It is an analyst queue/summary, not a single per-week correction line.

The page's configurable UX path includes ABC, XYZ, demand classification, priority, numeric value, boolean filters, and conditional formatting. `OCA530` controls priority levels included by default; `OCA100.Override To Review` allows an analyst to add an item manually; `Clear Review` lets an analyst clear the queue state after review. Explain that filters change what the planner sees, not the source data or statistical correction formulas.

## Read-only investigation workflow for future questions

1. Confirm exact workspace/model using `catalog_models` / `set_model_context`; check `get_model_context` after binding.
2. Catalog the matching module(s) and all line items with `catalog_line_items(detail="full")` when an exact line item, format, notes, or formula matters. Paginate until `has_more=false`.
3. For current values, prefer `read_cells` with all dimensions accounted for. Slice all but one or two intended breakdown dimensions to a single member; use exact member names from catalog/list outputs. Relevant weekly modules use Time + product/customer lists; SHC100 additionally uses week and customer/product forecast dimensions.
4. For a single week, use the exact model week label, product, customer, and applicable forecast members. Check the model time metadata/anchors first if the user says current/last/recent; do not use real-world time as a substitute.
5. Trace flag lineage in order: `SHC000` settings/eligibility → `SHC100` limits and breach/test/correction lines → `OCA200` explanation context or `OCA100` summary/priority as needed.
6. Report which facts are from the blueprint and which are live values. If a required coordinate, period, or selected context is ambiguous or unavailable, ask rather than infer.
7. Keep every action read-only. Do not invoke `cell_write`, `create_list_item`, `run_import`, or `run_process` for this skill.

## Answer format

For planner questions, respond with:
1. **Short answer** in plain language.
2. **What the model checks** — cite exact module/line-item names and relevant settings/logic.
3. **What happens next** — system correction, explanation, manual correction, alert/priority, or filter behavior, as applicable.
4. **Limits/uncertainty** — clarify when catalog structure is known but the user's live value or the exact combination/week has not been inspected.

Use planner vocabulary before Anaplan internals. Avoid presenting a long blueprint dump unless the user explicitly asks for the full map.
