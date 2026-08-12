# Google Play Store Analytics Dashboard — ElevanceSkills Internship

## 📌 Project Overview

This project extends the **Google Play Store Analytics** training project completed earlier in the internship. All 6 internship tasks are implemented as additional interactive dashboards and analytics features **on the same dataset and codebase** used during training — no new dataset or unrelated project has been introduced, per internship instructions.

The training project involved cleaning and merging the Google Play Store apps dataset with user review sentiment data. This internship submission builds 6 new visualizations on top of that same cleaned dataset, each with its own filtering logic, time-based visibility rules, and (where relevant) multilingual category labels.

**Domain:** Data Analytics
**Tools:** Python, Pandas, Plotly, Matplotlib, Jupyter Notebook
**Dataset:** Kaggle Google Play Store Dataset (`Play Store Data.csv`, `User Reviews.csv`)

---

## 🔗 Live Project / Links

- **GitHub Repository:** https://github.com/Ankit-pro908/google-playstore-analytics-dashboard
- **Live Website / Hosted Dashboard:** https://ankit-pro908.github.io/google-playstore-analytics-dashboard/

---

## 📂 Dataset

| File | Description |
|---|---|
| `data/Play Store Data.csv` | App-level metadata — category, rating, size, installs, price, reviews, last updated, Android version, content rating |
| `data/User Reviews.csv` | User review-level sentiment data — sentiment polarity and subjectivity per review |

### Repository Structure

data/          -> datasets
outputs/       -> generated charts
dashboards/    -> training dashboards
internship.ipynb -> internship implementation
index.html     -> hosted dashboard homepage
style.css      -> dashboard styling
README.md      -> project documentation

---
---

## 🧹 Data Cleaning Summary

Performed once, upfront, in `internship.ipynb` (Cells 1–24), shared across all 6 tasks:

- Removed duplicate app entries (`drop_duplicates(subset="App")`)
- Converted `Size` (e.g., `"19M"`, `"512k"`) into a numeric `Size_MB` column
- Converted `Rating` to numeric, dropped invalid/missing rows
- Cleaned `Installs` (removed `,` and `+`, converted to numeric)
- Converted `Reviews` to numeric
- Standardized `Category` to uppercase for consistent filtering
- Cleaned and averaged `Sentiment_Subjectivity` per app from the reviews dataset, merged into the main dataframe (`merged_df`)
- Parsed `Last Updated` into a proper datetime column
- Parsed `Android Ver` into a numeric minimum-version column (`Android_Ver_Num`)
- Cleaned `Price` into a numeric column and computed a `Revenue` proxy (`Price × Installs`)

---

## Task 1 — Bubble Chart

**Requirement:** Relationship between app size and rating, bubble size = installs, filtered by rating, category, reviews, sentiment, install count, and app name; Game category highlighted in pink; Beauty/Business/Dating translated to Hindi/Tamil/German; visible only 5–7 PM IST.

**Filters applied:**
- Rating > 3.5
- Reviews > 500
- Installs > 50,000
- Sentiment Subjectivity > 0.5
- Category in: Game, Beauty, Business, Comics, Communication, Dating, Entertainment, Social, Events
- App name does not contain the letter "S"

**Approach:**
- Plotly bubble chart (`px.scatter`), X = Size_MB, Y = Rating, bubble size = Installs
- Game category highlighted in pink via a derived `Color` column
- Beauty → Hindi (सौंदर्य), Business → Tamil (வணிகம்), Dating → German (Verabredung), shown via a separate `Category_Display` column so the underlying English `Category` stays intact for filtering/coloring logic
- Time-gated to render only between 5–7 PM IST; outside this window, a message is shown instead

**Post-review fix:** Initial version scaled bubble sizes purely linearly, causing low-install apps (e.g. Communication, Social, lower-rated Game apps) to render as barely-visible single pixels. Fixed by capping the maximum bubble size (`size_max=40`) and enforcing a visible minimum bubble size (`sizemin=4`) so every data point remains distinguishable regardless of install count.

**Output:** `outputs/task1_bubble_chart.html`

**Status:** ✅ Fully compliant with all stated filters. Bubble scaling issue identified in review resolved.

---

## Task 2 — Choropleth Map

**Requirement:** Global installs by category on a world map, top 5 categories (excluding those starting with A/C/G/S), highlight categories exceeding 1M installs, visible only 6–8 PM IST.

**Filters applied:**
- Category does not start with A, C, G, or S
- Top 5 categories by total installs

**⚠️ Data limitation — no country-level install data:**
The dataset has no `Country` column or any per-country breakdown of installs. A choropleth's entire purpose is to show geographic variation, so a workaround was required.

**Approach used:** Installs were distributed across countries using **population share as a weighting proxy** (`category total installs × country population / world population`), using `px.data.gapminder()` for population figures. This is a documented estimation technique, not real per-country data — clearly disclosed both in the notebook and directly on the chart itself (title + on-chart annotation), so it cannot be mistaken for real geographic data at a glance.

**Post-review fixes applied:**
1. **Highlight logic corrected** — The original condition (`installs > 1,000,000`) was always true for all 5 categories shown, since the top-5-by-installs selection guarantees each exceeds 1M by definition, making the highlight visually meaningless (every frame identical). Fixed by changing the threshold to installs ≥ 80% of the group's maximum, so highlighting now varies across categories instead of applying uniformly.
2. **Highlight visibility fixed** — Previously, the >1M highlight only appeared in hover data, not as a visible map element. Fixed by adding visible marker border styling (red border for highlighted categories, gray for others) directly on the map shapes.
3. **Disclosure clarity improved** — Estimation methodology is now disclosed directly on the chart itself (title + on-chart annotation), not just in a separate markdown note, so it can't be missed.

**⚠️ Open concern (pending mentor decision):** Review feedback raised a more fundamental objection — that a choropleth map shouldn't present *any* estimated/proxy per-country data, regardless of disclosure, since the dataset has no real geographic install breakdown. This is a data-availability limitation, not a code bug, and cannot be fully resolved without a different dataset or a different chart type for this task. Awaiting mentor guidance on whether to retain the current (disclosed, estimated) choropleth, or replace it with a category-only bar chart/table that avoids the geographic-accuracy question entirely.

**Insight:** Even as an estimate, the map consistently shows India and China with the highest projected installs — plausible given these are among the largest Android user bases globally.

**Output:** `outputs/task2_choropleth.html`

**Status:** ⚠️ Partially addressed — highlight logic, highlight visibility, and disclosure clarity all fixed; core data-availability objection still open, pending mentor decision.

---

## Task 3 — Time Series Line Chart

**Requirement:** Installs trend over time by category, shaded growth regions (>20% MoM), filtered by category prefix, app name, reviews; Beauty/Business/Dating translated; visible only 6–9 PM IST.

**Filters applied:**
- Reviews > 500
- Category starts with E, C, or B
- App name does not start with X, Y, or Z
- App name does not contain the letter "S"

**⚠️ Data limitation — no install-history log:**
The dataset only has a single `Last Updated` date per app — there is no record of installs at different points in time. `Last Updated` month was used as a time-axis proxy, documented clearly in the notebook.

**Post-review fix — chart readability:** One category (Communication) spiking to several billion installs in late 2018 compressed every other category into a flat line near zero on a linear axis, making the chart uninformative. Fixed by switching the Y-axis to a **logarithmic scale** (`update_yaxes(type="log")`), so all 8 categories remain visible and comparable regardless of scale differences.

**Consequence:** A sharp spike still appears near mid-2018 even on the log scale — this reflects a clustering artifact (most apps were last updated shortly before the dataset was scraped), not real install growth. This is explicitly noted below the chart.

**Contradiction noted:** The task also asks to translate "Dating" into German, but Dating starts with "D" and is excluded by the E/C/B category filter — so the German label can never actually appear on this chart. The translation mapping is kept in code for completeness but flagged as unreachable.

**Insight:** Entertainment and Communication show the most updates around 2018, suggesting the most active developer maintenance in these fast-moving categories.

**Output:** `outputs/task3_timeseries.html`

**Status:** ✅ Uses `Last Updated` as a documented time proxy. Log-scale fix applied for readability. Minor unreachable-translation note included.

---

## Task 4 — Stacked Area Chart

**Requirement:** Cumulative installs over time by category, increasing color intensity for months with >25% MoM growth, filtered by rating, app name, category prefix, reviews, size; Travel & Local/Productivity/Photography translated to French/Spanish/Japanese; visible only 4–6 PM IST.

**Filters applied:**
- Rating ≥ 4.2
- App name contains no digits
- Category starts with T or P
- Reviews > 1,000
- Size between 20–80 MB

**⚠️ Two limitations addressed:**

1. **Time proxy:** Same `Last Updated`-based limitation as Task 3.
2. **Plotly cannot vary opacity within a single continuous stacked band.** Since the task explicitly requires "increasing color intensity" for high-growth months *within* a category's band, **this chart was built in Matplotlib instead of Plotly** — the only static chart in the dashboard, used specifically to satisfy this requirement literally (via per-segment `fill_between()` opacity) rather than approximating it with a marker overlay.

**Trade-off:** Since segments are drawn individually to allow per-segment opacity, category bands overlap rather than stack cleanly like a standard stacked area chart — smaller categories (Parenting, Personalization) can be visually dominated by larger ones (Photography).

**Font handling:** Japanese category labels (写真 for Photography) require a CJK-capable font, since Matplotlib's default font doesn't include Japanese glyphs. A font-detection fallback (Yu Gothic / MS Gothic / Meiryo / Noto Sans CJK JP) is applied before rendering.

**Insight:** Photography shows the most concentrated high-opacity (significant-growth) segments, particularly near 2018 — suggesting installs were driven by a small number of sharp update clusters rather than steady growth.

**Output:** `outputs/task4_stacked_area.png`

**Status:** ✅ Passed review. Uses documented time proxy + Matplotlib (mentor-approved) to satisfy the opacity requirement literally.

---

## Task 5 — Grouped Bar Chart

**Requirement:** Average rating vs total reviews for top 10 categories by installs, filtered by rating, size, and January last-update; visible only 3–5 PM IST.

**Filters applied:**
- Size ≥ 10 MB
- `Last Updated` month = January (any year)
- Average rating ≥ 4.0 (applied after the above filters)
- Top 10 categories by total installs

**Approach:** Dual-axis grouped bar chart (Average Rating on primary axis, Total Reviews on secondary axis), built using `make_subplots(secondary_y=True)` with explicit `offsetgroup` values on each trace — required because combining `barmode="group"` with a secondary y-axis via a simple layout dict causes Plotly to render fused/overlapping bars instead of properly grouped ones, with both axes clearly visible on independent scales.

**Output:** `outputs/task5_grouped_bar.html`

**Status:** ✅ No data limitations. Dual-axis rendering verified with fully separated, independently-scaled bars. Filters applied before aggregation/ranking for internal consistency — confirmed with mentor.

---

## Task 6 — Dual-Axis Chart (Free vs Paid)

**Requirement:** Average installs and revenue for free vs. paid apps in the top 3 categories, filtered by installs, revenue, Android version, size, content rating, app name length; visible only 1–2 PM IST.

**Filters applied:**
- Installs ≥ 10,000
- Android version > 4.0
- Size > 15 MB
- Content Rating = "Everyone"
- App name ≤ 30 characters
- Revenue ≥ $10,000 (**applied to Paid apps only** — see below)

**⚠️ Two limitations addressed:**

1. **No real Revenue column.** Computed as a standard proxy: `Revenue = Price × Installs`.
2. **Revenue filter conflict resolved:** Free apps always have $0 revenue by definition (Price = 0). Applying the $10,000 revenue filter to *all* apps would eliminate every free app, making a free-vs-paid comparison impossible. The filter was therefore applied to Paid apps only, while Free apps only need to pass the installs filter — confirmed acceptable with mentor.

**Approach:** Same dual-axis grouped bar technique as Task 5 (`make_subplots` + `offsetgroup`) to avoid the fused-bar rendering issue.

**Output:** `outputs/task6_dual_axis.html`

**Status:** ✅ Passed review. Uses a documented revenue proxy; free/paid filter logic confirmed with mentor.

---

## ⚠️ Known Limitations & Methodology Notes (Summary)

| Limitation | Affected Tasks | How it was handled |
|---|---|---|
| Bubble sizes collapsing to invisible pixels for low-install apps | Task 1 | Applied `size_max` cap and `sizemin` floor for consistent visibility |
| No country-level install data | Task 2 | Population-weighted estimation using `px.data.gapminder()`, disclosed on-chart |
| Flat, uniform highlight logic (always true for top-5) | Task 2 | Fixed: relative threshold (≥80% of group max) + visible map border styling |
| Choropleth built on estimated (not real) per-country data | Task 2 | Disclosed on-chart; open objection to using estimated data on a map — pending mentor decision |
| Dominant category compressing others on linear scale | Task 3 | Switched Y-axis to logarithmic scale |
| No install-history log (only `Last Updated`) | Tasks 3, 4 | `Last Updated` month used as a documented time-axis proxy |
| Plotly can't vary opacity within one stacked band | Task 4 | Rebuilt in Matplotlib with per-segment `fill_between()` opacity |
| No real Revenue column | Task 6 | `Price × Installs` used as a standard proxy |
| Free apps always have $0 revenue | Task 6 | Revenue filter applied to Paid apps only |
| Dating→German translation unreachable | Task 3 | Category filter (E/C/B) excludes Dating; mapping kept but flagged as unused |

All of the above are also documented inline as markdown cells directly above/below their respective charts in `internship.ipynb`.

---

## 🕒 Time-Gated Visibility

Per task requirements, each chart is only rendered within a specific IST time window; outside that window, the notebook prints a message instead of the chart. This is implemented using `pytz` and `datetime.now()` in each chart's cell.

| Task | Visible Window (IST) |
|---|---|
| 1 — Bubble Chart | 5 PM – 7 PM |
| 2 — Choropleth | 6 PM – 8 PM |
| 3 — Time Series | 6 PM – 9 PM |
| 4 — Stacked Area | 4 PM – 6 PM |
| 5 — Grouped Bar | 3 PM – 5 PM |
| 6 — Dual-Axis | 1 PM – 2 PM |

---

## 🛠️ How to Run

### Local Execution

1. Clone this repository and ensure `data/Play Store Data.csv` and `data/User Reviews.csv` are present in the `data/` folder.
2. Install dependencies:
```bash
   pip install pandas numpy plotly matplotlib pytz nltk scikit-learn
```
3. Open `internship.ipynb` in Jupyter Notebook or VS Code.
4. Run all cells in order (**Kernel → Restart & Run All** recommended to avoid cell-order dependency issues).
5. Each visualization follows the internship-specified IST time window restrictions implemented within the notebook logic. Outside the designated time window, the notebook displays an informational message instead of rendering the chart.
6. Generated outputs are saved to the `outputs/` folder:
   - `.html` files for Plotly visualizations
   - `.png` file for the Matplotlib visualization (Task 4)

### Time Window Enforcement

The internship requirements specify that each visualization must only be accessible during a particular IST time window.

Since GitHub Pages is a static hosting platform and cannot execute Python code after deployment, the hosted dashboard implements additional browser-side validation using JavaScript and the Asia/Kolkata timezone. This ensures:

- Dashboard access is checked dynamically in real time.
- Users can only open a dashboard during its allowed IST time window.
- Outside the allowed time window, an informational message is displayed.
- The original notebook implementation remains unchanged and continues to satisfy the internship requirements.

This approach preserves the intended behavior of all six tasks in both the notebook environment and the hosted dashboard.

---

## 🧰 Tech Stack

**Data Processing:** Python, Pandas, NumPy
**Visualization:** Plotly (Tasks 1, 2, 3, 5, 6), Matplotlib (Task 4)
**NLP & Sentiment Analysis:** NLTK (VADER Sentiment Analyzer)
**Time Window Handling:** pytz, datetime, JavaScript (Hosted Dashboard Time Validation)
**Development Environment:** Jupyter Notebook, VS Code
**Deployment:** GitHub, GitHub Pages