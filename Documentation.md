# 📄 Documentation — Global Employee Analytics Dashboard

## 1. Project Aim / Goal

The company's HR team wanted a single, interactive view of its global workforce to answer recurring questions about headcount, pay, demographics, and performance — and specifically to compare its **India** and **New Zealand** offices side by side. The goal of this project was to design and build a Power BI dashboard that:

- Consolidates raw HR records into a clean, analysis-ready data model.
- Surfaces core workforce KPIs at a glance.
- Lets HR stakeholders slice the data by country, department, gender, join date, and name — without needing to write a single query.
- Answers ten specific business questions defined by stakeholders (see `**[Questions.png](https://github.com/DRasool7013/-Global-Employee-Analytics-Dashboard/blob/main/Questions.png)**).

---

## 2. Dataset Description

**Source file:** **[hr-data.xlsx](Documentation.md)**
**Sheet:** `Data`
**Records:** 183 employees

| Column | Type | Notes |
|---|---|---|
| Name | Text | Employee full name |
| Gender | Text | Male, Female, Other |
| Age | Whole number | Range: 19–46 |
| Rating | Text | Poor, Very Poor, Average, Above Average, Exceptional |
| Date Joined | Date | Range: 07-May-2020 to 29-Apr-2023 |
| Department | Text | HR, Sales, Procurement, Finance, Website |
| Salary | Currency (number) | Annual salary in USD |
| Country | Text | IND (India), NZ (New Zealand) |

---

## 3. Data Cleaning & Preparation Process

Performed in **Power Query Editor** before loading into the model:

1. **Import** — Connected to `hr-data.xlsx`, loaded the `Data` sheet.
2. **Type checks** — Verified each column's data type (Age and Salary as whole numbers, Date Joined as Date, remaining fields as Text) and corrected any mis-typed columns.
3. **Trim/Clean text** — Applied `Text.Trim` / `Text.Clean` to Name, Department, and Country to remove stray whitespace from the source file.
4. **Duplicate/blank check** — Checked for duplicate employee names and blank rows; none required removal beyond the header row.
5. **Calculated columns added post-load:**
   - `Age (bins)` — a grouped/binned version of Age (created via Power BI's **New Group** on the Age column) used to drive the age-spread histogram.
   - `First Letter` — `= LEFT(Data[Name], 1)`, used to power the alphabet-based employee filter.
6. **Custom sort table (`field order`)** — Because Rating is a text field with a natural but non-alphabetical order (Poor → Very Poor → Average → Above Average → Exceptional), a small two-column lookup table was created:

   | Rating | Order |
   |---|---|
   | Poor | 1 |
   | Very Poor | 2 |
   | Average | 3 |
   | Above Average | 4 |
   | Exceptional | 5 |

   This table was related to `Data[Rating]` (one-to-many, `field order[Rating]` → `Data[Rating]`), and the `Data[Rating]` column's **Sort by Column** property was set to `field order[Order]` so every visual using Rating displays in the correct logical sequence rather than alphabetically.

---

## 4. Data Model

- **Tables:** `Data` (main table) and `field order` (lookup table for Rating sort order).
- **Relationship:** `field order[Rating]` (1) → `Data[Rating]` (*), i.e., one row in `field order` maps to many rows in `Data`.
- This is a lightweight version of a snowflake/star pattern — a single fact-like table with one supporting dimension table for sort logic. See `Physical_Model.png` for the relationship diagram.

---

## 5. Measures (DAX)

Core measures created for the KPI cards and charts (see `table___measures.png` for the full field list):

```DAX
Head Count = COUNTROWS(Data)

Average Salary = AVERAGE(Data[Salary])

Max Salary = MAX(Data[Salary])

Min Salary = MIN(Data[Salary])
```

These measures respond dynamically to whichever slicers (Country, Department, Gender, Date Joined, First Letter) are currently applied, which is what allows the same four cards to show different numbers for India vs. New Zealand on the scorecard page.

---

## 6. Slicers — Design & Setup

| Slicer | Field | Style | Purpose |
|---|---|---|---|
| Country | `Data[Country]` | Tile | Switch/compare IND vs NZ |
| Department | `Data[Department]` | Tile | Filter to one or more departments |
| Gender | `Data[Gender]` | Tile | Filter by gender |
| Date Joined | `Data[Date Joined]` | Between (date slider) | Filter by hire date range |
| First Letter | `Data[First Letter]` | Dropdown | Jump to employees whose name starts with a given letter |

**Setup steps:**

1. Click an empty area of the report canvas.
2. In the **Fields** pane, drag the target field onto the canvas — Power BI creates a table by default; change the visual type to **Slicer** in the **Visualizations** pane.
3. Under **Format → Slicer settings → Options**, choose the appropriate style (`Tile`, `Dropdown`, or `Between` for dates).
4. Position and resize consistently along the top of the page.
5. Use **View → Sync Slicers** to sync each slicer across both report pages, so filtering on one page carries over to the other.
6. Set **Single select** vs **multi-select** per business need (e.g., Country/Department typically allow multi-select; Date Joined uses a range).

---

## 7. Business Questions & How the Dashboard Answers Them

| # | Question | Visual / Feature Used |
|---|---|---|
| 1 | How many people are there in each department? | Pie chart — Head Count by Department |
| 2 | Gender distribution by department | Stacked bar chart — Gender × Department |
| 3 | Age spread of staff | Histogram on `Age (bins)` |
| 4 | Min / max / average salary in each department | Column chart with Min/Max/Average Salary measures by Department |
| 5 | Top earners in each country | Table (Name, Gender, Salary) sorted descending, filtered by Country slicer |
| 6 | Performance spread (sort by column) | Bar chart on Rating, custom-sorted via `field order` table |
| 7 | Company growth trend | Line chart of Head Count over `Date Joined` (cumulative hires) |
| 8 | Employee filter (by starting letter) | Dropdown slicer on `First Letter` |
| 9 | Performance vs. Salary relationship | Scatter chart — Rating (or Order) vs. Salary |
| 10 | India vs. New Zealand quick scorecard | Side-by-side KPI + pie chart + table layout, one panel per country |

---

## 8. Final Result

The finished report (**[global-employee-analytics-dashboard.pbix](https://github.com/DRasool7013/-Global-Employee-Analytics-Dashboard/blob/main/global-employee-analytics-dashboard.pbix)**) is a two-section dashboard:

- A **scorecard page** placing India and New Zealand panels side by side, each with Head Count, Average Salary, a department pie chart, and a top-earners table (see **[Global-Employee-analytis-dashboard.png](https://github.com/DRasool7013/-Global-Employee-Analytics-Dashboard/blob/main/Global-Employee-analytis-dashboard.png)**).
- Supporting analysis visuals (histogram, salary-by-department chart, performance spread, growth trend, and scatter plot) that answer the remaining business questions and can be filtered using the shared slicer set.

**Headline numbers:** India — 92 employees, $77K average salary. New Zealand — 91 employees, $77K average salary. Both offices show a similar departmental split, with Website and Procurement as the largest functions (~29–30% of headcount each) and HR the smallest (~4%).

---

## 9. Key Insights

- Headcount and average salary are nearly identical between India and New Zealand, suggesting balanced staffing investment across both offices.
- Website and Procurement are the two largest departments in both countries, together accounting for roughly 60% of headcount.
- HR is consistently the smallest department (~4%) in both locations.
- The custom Rating sort (via the `field order` table) was necessary because Power BI's default alphabetical sort would have scrambled the natural Poor → Exceptional performance scale.

---

## 10. Tools & Skills Demonstrated

- Power Query data cleaning and shaping
- Data modeling and table relationships (including a lookup table purely for custom sort order)
- DAX measure writing
- Interactive slicer design and cross-page slicer syncing
- Dashboard/report design for executive-style comparison views
- Git/GitHub version control and repository publishing
