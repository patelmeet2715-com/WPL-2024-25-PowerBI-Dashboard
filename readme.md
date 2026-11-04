# 🏆 Premier League (WPL) 2024–25 Analytics Dashboard — Power BI

**Author:** Meet Patel  
**Dashboard File:** [meet\_patel\_WPL.pbix](https://www.google.com/search?q=./meet_patel_WPL.pbix)  
**Tools Used:** Power BI Desktop, Power Query (M), DAX, Star Schema Data Modeling

---

## 📌 Project Overview

An end-to-end sports & financial analytics Power BI solution analyzing the **2024–25 Premier League season** across **380 fixtures**, **20 clubs**, and **562 players (577 club stints)**. The project transforms 8 raw datasets through Power Query into a 10-table **Star Schema** (`7 Dimension Tables` + `3 Fact Tables`) joined via Surrogate Keys, powered by **36 DAX measures** inside `MEETs Measure` and presented across a **4-page interactive dashboard** with custom iconography and page navigation.

---

## 🛠️ Data Engineering & Power Query Pipeline

1. **Text Cleaning & Trimming (`Text.Trim` & `Text.Clean`)**: Removed leading/trailing whitespace and non-printable characters across all club, player, venue, nation, and referee fields.
2. **Entity Standardization & Encoding Fixes**:
   - Standardized referee variations (`"Andrew Madley"` → `"Andy Madley"`, reducing 22 raw strings to 21 distinct officials).
   - Fixed UTF-8 mojibake and reconciled 14 player naming variations across `player_stats`, `player_possession_stats`, and `player_salaries`.
3. **Missing Value Imputation**:
   - Imputed 4 missing stadium attendance records in `fixtures.csv` using each venue's season mean attendance.
   - Populated missing nationality, age, and birth year fields for 3 U-18 academy debutants (`Jake Evans`, `Mateus Mane`, `Jeremy Monga`).
4. **Surrogate Key Generation (`Table.AddIndexColumn`)**: Created integer Primary Keys (`TeamKey`, `VenueKey`, `RefereeKey`, `DateKey`, `PlayerKey`, `PositionKey`, `NationKey`) and merged them into the 3 Fact tables (`Table.NestedJoin`).

---

## 🗂️ Star Schema Architecture (7 Dimensions + 3 Facts)

| Table Name | Table Type | Grain & Row Count | Primary / Foreign Keys | Key Custom Columns (`fx` / Power Query) |
| :--- | :--- | :--- | :--- | :--- |
| **`Dim_Team`** | Dimension | 20 Clubs | `TeamKey` (PK, `1..20`) | `Qualification_Tier`, `Squad_Age_Profile` |
| **`Dim_Venue`** | Dimension | 20 Stadiums | `VenueKey` (PK, `1..20`) | `Capacity_Tier`, `Attendance_Spread` |
| **`Dim_Referee`** | Dimension | 21 Referees | `RefereeKey` (PK, `1..21`) | `Referee_Panel_Tier` |
| **`Dim_Date`** | Dimension | 109 Matchdays | `DateKey` (PK, `YYYYMMDD`) | `Month_Name`, `Month_Year_Label`, `Quarter`, `Fixture_Window`, `Season_Phase` |
| **`Dim_Player`** | Dimension | 562 Unique Players | `PlayerKey` (PK, `1..562`) | `Age_Bracket`, `Wage_Bracket`, `Transfer_Movement` |
| **`Dim_Position`** | Dimension | 10 Tactical Roles | `PositionKey` (PK, `1..10`) | `Position_Group`, `Tactical_Role`, `Role_Type` |
| **`Dim_Nation`** | Dimension | 65 Nationalities | `NationKey` (PK, `1..65`) | `Homegrown_Region`, `Confederation` |
| **`Fact_Fixtures`** | Fact | 380 Matches | `FixtureKey` (PK), `DateKey`, `VenueKey`, `RefereeKey`, `HomeTeamKey`, `AwayTeamKey` | `Total_Goals`, `Goal_Margin`, `Home_Points`, `Away_Points`, `Match_Result`, `Match_Excitement_Tier` |
| **`Fact_Team_Stats`** | Fact | 20 Club Seasons | `TeamStatKey` (PK), `TeamKey` (FK) | `Matches_Played`, `Goal_Difference`, `xG_Differential`, `Possession_Style` |
| **`Fact_Player_Stats`** | Fact | 577 Player-Club Stints | `PlayerStatKey` (PK), `PlayerKey`, `TeamKey`, `PositionKey`, `NationKey` | `Goals_Plus_Assists`, `Total_Cards`, `Non_Penalty_Goals`, `Goal_Contribution_Tier` |

---

## 📐 Key DAX Measures (`MEETs Measure`)

| Measure Name | DAX Formula | Description |
| :--- | :--- | :--- |
| **`Total League Goals (Fixtures)`** | `SUM(Fact_Fixtures[Total_Goals])` | Total goals scored across all 380 matches (`1,115` goals) |
| **`Avg Goals per Match`** | `DIVIDE([Total League Goals (Fixtures)], [Total Matches (Fixtures)], 0)` | League-wide scoring average (`2.93` goals/match) |
| **`Avg Attendance per Match`** | `AVERAGE(Fact_Fixtures[Attendance])` | Mean stadium attendance (`40,418` fans/match) |
| **`Home Win Rate %`** | `DIVIDE(CALCULATE(COUNTROWS(Fact_Fixtures), Fact_Fixtures[Match_Result] = "Home Win"), [Total Matches (Fixtures)], 0)` | Percentage of fixtures won by the home side (`40.8%`) |
| **`Team xG Overperformance`** | `[Player Scored Goals (Team)] - [Team Expected Goals (xG)]` | Clinical finishing differential vs. expected goals |
| **`Total Goal Contributions (G+A)`** | `SUM(Fact_Player_Stats[Goals_Plus_Assists])` | Combined player goals and assists (`1,886` G+A) |
| **`Take-On Success Rate %`** | `DIVIDE(SUM(Fact_Player_Stats[Successful_Take_Ons]), SUM(Fact_Player_Stats[Attempted_Take_Ons]), 0)` | Dribble completion rate (`44.5%`) |
| **`Total Club Annual Payroll (£M)`** | `DIVIDE(SUM(Fact_Team_Stats[Annual_Payroll_GBP]), 1000000, 0)` | Official club wage expenditure (`£1,755.55M` total) |
| **`Payroll Cost per League Point (£M)`** | `DIVIDE([Total Club Annual Payroll (£M)], [Total League Points], 0)` | Wage efficiency per point earned (`£1.66M/pt` league mean) |
| **`Payroll Cost per Team Goal (£M)`** | `DIVIDE([Total Club Annual Payroll (£M)], [Team Goals For (Standings)], 0)` | Wage efficiency per goal scored (`£1.57M/goal` league mean) |

---

## 📊 Report Pages & Key Insights

1. **Page 1 — Standings & Match Overview**:
   - **Liverpool (84 pts, +45 GD, 86 GF)** clinched the title ahead of **Arsenal (74 pts, +35 GD)**, **Manchester City (71 pts)**, and **Chelsea (69 pts)**.
   - Across 380 fixtures, clubs scored **1,115 goals (2.93/match)** with **40.8% Home Wins (155)**, **34.7% Away Wins (132)**, and **24.5% Draws (93)**.
2. **Page 2 — Club Tactical & Possession**:
   - **Manchester City (61.6% possession, 28,022 touches)** and **Liverpool (57.7% possession, 1,924 progressive passes)** led territorial control, while **Nottingham Forest (65 pts on 41.2% possession)** excelled in direct counter-attacking play.
3. **Page 3 — Player Scoring & Progression**:
   - **Mohamed Salah** led the Golden Boot and Playmaker charts with **47 G+A (29 Goals, 18 Assists)**, followed by **Alexander Isak (29 G+A: 23G, 6A)**, **Bryan Mbeumo (27 G+A: 20G, 7A)**, and **Erling Haaland (25 G+A: 22G, 3A)**.
4. **Page 4 — Wage Bill & Financial ROI**:
   - Total league payroll reached **£1,755.55M/yr**, led by **Manchester City (£222.98M)**, **Manchester United (£189.36M)**, **Chelsea (£175.45M)**, and **Arsenal (£174.93M)**.
   - **Ipswich Town (£0.51M/pt)**, **Bournemouth (£0.91M/pt)**, **Nottingham Forest (£0.92M/pt)**, and **Brentford (£0.93M/pt)** delivered the best points-per-pound efficiency.

---

## 🚀 How to Open & Use

1. Clone or download this repository.
2. Open [meet\_patel\_WPL.pbix](https://www.google.com/search?q=./meet_patel_WPL.pbix) in **Microsoft Power BI Desktop**.
3. Use the **left sidebar icon buttons** (`Ctrl + Click` in Power BI Desktop) to navigate across all 4 pages, or open **Home → Transform data** to inspect the 18 Power Query M pipelines.
