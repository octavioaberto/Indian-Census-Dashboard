https://github.com/octavioaberto/Indian-Census-Dashboard/releases

# Indian Census Dashboard — State Trends, Literacy & Employment

[![Releases](https://img.shields.io/badge/Releases-v1.0-blue?style=for-the-badge&logo=github)](https://github.com/octavioaberto/Indian-Census-Dashboard/releases) [![Power BI](https://img.shields.io/badge/PowerBI-Report-yellow?style=for-the-badge&logo=power-bi)](https://powerbi.microsoft.com/) [![SQL](https://img.shields.io/badge/SQL-Scripts-lightgrey?style=for-the-badge&logo=mysql)](https://github.com/octavioaberto/Indian-Census-Dashboard)

🇮🇳 📊 Interactive Power BI report that analyzes Indian Census data from 1991, 2001 and 2011 at the state level. It covers population, age structure, literacy, employment sectors, housing and key ratios. The report blends cleaned CSVs, SQL scripts and a Power BI .pbix file. Use the Releases page to get the report and supporting files.

![India map](https://upload.wikimedia.org/wikipedia/commons/thumb/7/76/India_-_States_and_Union_Territories.svg/1200px-India_-_States_and_Union_Territories.svg.png)

## Table of contents
- About
- Key features
- Data sources and structure
- Sample visuals
- How to run (download and execute)
- Data cleaning and prep
- SQL snippets
- Power BI tips and measures
- Folder layout
- Contributing
- License

## About
This repo hosts an interactive dashboard and data artifacts for state-level analysis of the Indian Census across three decennial rounds: 1991, 2001 and 2011. The analysis focuses on:
- Demographics and population change
- Literacy and education gaps
- Workforce distribution and unemployment proxies
- Housing and household amenities
- Ratios and derived indicators for trend analysis

The work supports researchers, planners and analysts who need a compact, reproducible view of long-term state trends.

## Key features
- Multi-year comparatives by state and union territory.
- Demographics: total population, sex ratio, age bands.
- Literacy: overall and gendered literacy, effective literacy rate.
- Employment: sector split (agriculture, industry, services), work participation rate.
- Housing: household size, electrification, drinking water, latrine access.
- Map, trend, and bar chart pages for quick insight.
- Export-ready visuals for reports or presentations.
- Data cleaning scripts in SQL and Excel templates for repeatable prep.

## Data sources and structure
Primary source: Indian Census tables for 1991, 2001 and 2011 (state-level aggregates).
Included in the repo:
- Raw CSV exports from official tables (converted from PDF/Excel as needed).
- Cleaned CSVs with standard state codes and consistent column names.
- SQL scripts to import and transform CSVs into a relational staging schema.
- Power BI report (.pbix) with modeled tables, measures and visuals.

Common fields
- StateCode, StateName, Year
- TotalPopulation, MalePopulation, FemalePopulation
- LiteracyTotal, LiteracyMale, LiteracyFemale
- WorkersTotal, MainWorkers, Cultivators, AgriculturalLaborers, HouseholdWorkers, Others
- Households, HouseholdSize, HasElectricity, HasLatrine, PotableWater

## Sample visuals
- Choropleth map: population density and literacy by state.
- Trend lines: population, literacy and workforce share across years.
- Stacked bars: workforce by sector for a chosen year.
- Scatter: literacy rate vs. work participation with bubble for population.
- Small multiples: state time series for selected indicators.

Images below are examples of layout and purpose. Use them for design reference.

![Power BI example](https://cdn.pixabay.com/photo/2017/04/05/01/55/analysis-2203001_1280.jpg)

## How to run (download and execute)
You must download and open the release asset. Go to the Releases page, download the bundle, and execute the main report file.

Steps
1. Visit the releases page: https://github.com/octavioaberto/Indian-Census-Dashboard/releases
2. Download the release asset named like:
   - Indian-Census-Dashboard-v1.0.pbix (Power BI Desktop file)
   - data-cleaned.zip or census-data-1991-2001-2011.csv (data bundle)
   - sql-scripts.zip (optional)
3. Open the .pbix file in Power BI Desktop (version compatible with the file).
4. If the release includes a data bundle, extract it to the same folder and use the Manage Connections dialog to point the report to local CSVs, or import the cleaned CSVs via Get Data > Text/CSV.
5. If the release includes SQL scripts, run them against a local database to recreate the staging tables and then connect the report to that database.

The release file must be downloaded and executed to run the report locally. The .pbix file will open in Power BI Desktop. If the report uses DirectQuery, configure the connection string before viewing real-time visuals.

## Data cleaning and prep
Prep steps used for this project
- Standardize state names and codes across years.
- Unpivot and reshape year-specific tables into a tall format (Year column).
- Harmonize classification categories (e.g., workforce sectors).
- Derive indicators: sex ratio, literacy rate, work participation rate, household size.
- Flag missing values and set imputation rules only when needed.

Excel approach
- Use a master lookup sheet for states and codes.
- Use Power Query to merge, pivot/unpivot and clean formats.
- Save final outputs as UTF-8 CSV files.

SQL approach
- Load raw CSVs into staging tables.
- Apply transformations using SQL CREATE TABLE AS SELECT or INSERT...SELECT.
- Build dimension tables (StateDim, YearDim) and fact tables (CensusFact).

## SQL snippets
Create a staging table (Postgres/MySQL syntax)
```sql
CREATE TABLE staging_census_raw (
  state_code VARCHAR(10),
  state_name VARCHAR(100),
  census_year INT,
  indicator VARCHAR(100),
  value NUMERIC
);
```

Aggregate to a wide layout
```sql
CREATE TABLE census_wide AS
SELECT
  state_code,
  state_name,
  census_year,
  MAX(CASE WHEN indicator = 'TotalPopulation' THEN value END) AS total_population,
  MAX(CASE WHEN indicator = 'MalePopulation' THEN value END) AS male_population,
  MAX(CASE WHEN indicator = 'FemalePopulation' THEN value END) AS female_population,
  MAX(CASE WHEN indicator = 'Literate' THEN value END) AS literate_total
FROM staging_census_raw
GROUP BY state_code, state_name, census_year;
```

Compute derived indicators
```sql
ALTER TABLE census_wide ADD COLUMN sex_ratio NUMERIC;
ALTER TABLE census_wide ADD COLUMN literacy_rate NUMERIC;

UPDATE census_wide
SET
  sex_ratio = (female_population * 1000.0) / male_population,
  literacy_rate = (literate_total * 100.0) / total_population;
```

## Power BI notes and DAX examples
Model
- Use StateDim and YearDim for filters.
- Keep census facts numeric and indexed.
- Use relationships by StateCode + Year.

Sample DAX measures
Total Population
```dax
Total Population = SUM(CensusFact[TotalPopulation])
```

Literacy Rate (measure)
```dax
Literacy Rate = DIVIDE(SUM(CensusFact[LiterateTotal]), SUM(CensusFact[TotalPopulation]), 0)
```

Workforce Share by Sector
```dax
Work in Agriculture % = DIVIDE(SUM(CensusFact[AgricultureWorkers]), SUM(CensusFact[WorkersTotal]), 0)
```

Use Bookmarks and Drillthrough
- Provide pre-set bookmarks for key states or indicators.
- Use Drillthrough from map to state-level trends.

Performance tips
- Reduce model size by removing unused columns.
- Use numeric keys for joins.
- Use aggregated tables for large visuals.

## Folder layout
A recommended layout for the repo and releases
- /data/raw/ — unmodified CSVs or downloads
- /data/cleaned/ — cleaned CSVs ready for modeling
- /sql/ — scripts for staging and transforms
- /pbix/ — the Power BI report file(s)
- /docs/ — screenshots, design notes, mapping files
- /images/ — static images used in README and docs

## Contributing
- Fork the repo and create a topic branch.
- Create issues for bugs or data errors.
- Submit pull requests with clear descriptions and minimal diffs.
- Add tests for script outputs where applicable.
- Document any new data sources or assumptions.

Code style
- Keep SQL modular and idempotent.
- Use Power Query steps with descriptive names.
- Keep CSV column names stable across commits.

## Releases
Download and execute the release asset from the Releases page. The asset will contain the Power BI report and supplemental files you need to run the dashboard locally.

Releases: https://github.com/octavioaberto/Indian-Census-Dashboard/releases

If the file includes installable scripts or a .pbix file, download the file and execute/open it as described in the How to run section.

## SEO and topics
This project uses the following topics for discovery:
dashboard, data, data-analysis, data-visualization, datacleaning, datapreparation, datapreprocessing, ms-excel-data-analytics, powerbi, sql

## License
MIT License

---

Repository images and icons above use public CDN or Wikimedia sources.