# SampleModel – Power BI Semantic Model Documentation

> *Auto-generated documentation.*

## Table of Contents
1. [Overview](#overview)
2. [Table Relationships](#table-relationships)
3. [Tables and Columns](#tables-and-columns)
4. [Measures](#measures)
5. [Row-Level Security](#row-level-security)
6. [Data Sources](#data-sources)

## Overview

The **SampleModel** is a data model designed to analyze Marvel comic character data. It features a classic star schema with a central fact table (MarvelFact) connected to three dimension tables for date, eye color, and character alignment attributes. The model enables comprehensive analysis of character appearances across different time periods and characteristics.

| Aspect | Count |
|--------|-------|
| Tables | 6 |
| Relationships | 3 |
| Measures | 4 |

### Model Structure

- **Fact Table**: MarvelFact (contains character appearance records)
- **Dimension Tables**: DateDim, EyeColorDim, AlignmentDim
- **Source Tables**: MarvelSource (hidden, web-based CSV), DataTypes (utility table)

---

## Table Relationships

The model uses a standard star schema pattern with left-outer joins between the fact table and dimensions:

````mermaid
erDiagram
    MARVELFACT ||--o| DATEDIM : "DateID"
    MARVELFACT ||--o| EYECOLORDIM : "EyeID"
    MARVELFACT ||--o| ALIGNMENTDIM : "AlignmentID"
    
    DATEDIM {
        int64 DateID PK
        datetime Date
        string DateKey
        int64 MonthYearSort
        string "Month Year"
        int64 Year
    }
    
    MARVELFACT {
        int64 ID
        string Name
        int64 Appearances
        int64 DateID FK
        int64 EyeID FK
        int64 AlignmentID FK
        datetime "Test Change"
    }
    
    EYECOLORDIM {
        int64 EyeID PK
        string EyeKey
        string "Eye Color"
    }
    
    ALIGNMENTDIM {
        int64 AlignmentID PK
        string Alignment
        string AlignmentKey
    }
````

---

## Tables and Columns

### MarvelFact

The central fact table containing Marvel character appearance records.

| Column | Data Type | Description | Summarization |
|--------|-----------|-------------|----------------|
| ID | int64 | Character identifier | Count |
| Name | string | Character name | None |
| Appearances | int64 | Number of appearances | Sum |
| DateID | int64 | Foreign key to DateDim | None |
| EyeID | int64 | Foreign key to EyeColorDim | None |
| AlignmentID | int64 | Foreign key to AlignmentDim | None |
| Test Change | datetime | Calculated field for testing | None |

**Data Source**: Merged from MarvelSource with dimension lookups based on first appearance date, eye color, and alignment attributes.

### DateDim

Time dimension table providing date hierarchy for character analysis.

| Column | Data Type | Description | Summarization |
|--------|-----------|-------------|----------------|
| DateID | int64 | Primary key (unique date identifier) | None |
| Date | datetime | Calendar date | None |
| DateKey | string | Text representation of date | None |
| MonthYearSort | int64 | Sortable month-year integer (YYYYMM) | None |
| Month Year | string | Formatted month-year display (MMM-YY) | None |
| Year | int64 | Calendar year | None |

**Data Source**: Derived from unique first appearance dates in MarvelSource, expanded to monthly granularity from earliest date to current date.

### EyeColorDim

Dimension table containing Marvel character eye color attributes.

| Column | Data Type | Description | Summarization |
|--------|-----------|-------------|----------------|
| EyeID | int64 | Primary key (eye color identifier) | None |
| EyeKey | string | Source eye color code | None |
| Eye Color | string | Friendly eye color name | None |

**Data Source**: Distinct eye color values extracted from MarvelSource with standardized naming.

### AlignmentDim

Dimension table containing Marvel character alignment classifications.

| Column | Data Type | Description | Summarization |
|--------|-----------|-------------|----------------|
| AlignmentID | int64 | Primary key (alignment identifier) | None |
| Alignment | string | Friendly alignment name | None |
| AlignmentKey | string | Source alignment code | None |

**Data Source**: Distinct alignment values from MarvelSource, split and standardized with extensive transformations including sorting and null handling.

### MarvelSource

Hidden source table containing raw Marvel character data loaded from web CSV.

| Column | Data Type | Hidden | Description |
|--------|-----------|--------|-------------|
| page_id | int64 | Yes | Marvel wikia page identifier |
| name | string | Yes | Character name |
| urlslug | string | Yes | URL slug |
| ID | string | Yes | Character ID |
| ALIGN | string | Yes | Alignment code |
| EYE | string | Yes | Eye color code |
| HAIR | string | Yes | Hair color code |
| SEX | string | Yes | Character sex |
| GSM | string | Yes | GSM indicator |
| ALIVE | string | Yes | Alive/deceased status |
| APPEARANCES | int64 | Yes | Appearance count |
| FIRST APPEARANCE | string | Yes | First appearance info |
| Year | int64 | Yes | First appearance year |

**Data Source**: CSV file loaded from GitHub: `https://raw.githubusercontent.com/fivethirtyeight/data/master/comic-characters/marvel-wikia-data.csv`

### DataTypes

Utility table (structure not detailed in model).

---

## Measures

### Number of Characters

**Business Logic**: Counts the total number of distinct Marvel characters in the dataset.

**DAX Code**:
```dax
COUNTROWS('MarvelFact')
```

**Format**: Integer (0 decimal places)

**Use Case**: Total character count across the model.

---

### Number of Characters Title By Date

**Business Logic**: Displays a dynamic message indicating the number of characters that first appeared during the selected date period.

**DAX Code**:
```dax
"Number of Characters that first appeared " & [Date Filter]
```

**Format**: Text

**Use Case**: Narrative measure providing context about character debuts relative to selected dates.

---

### Rank of Appearances

**Business Logic**: Ranks each alignment group by total character appearances, allowing comparison of which character alignments have the most appearances.

**DAX Code**:
```dax
RANKX (
    ALLEXCEPT ( 'MarvelFact', AlignmentDim[Alignment] ),
    CALCULATE ( SUM ( 'MarvelFact'[Appearances] ) )
)
```

**Format**: Integer (0 decimal places)

**Use Case**: Comparative analysis of character appearance volumes by alignment type.

---

### Running Total of Character Appearances

**Business Logic**: Calculates a cumulative count of character appearances up to the current date context, showing growth over time.

**DAX Code**:
```dax
CALCULATE (COUNTROWS ( 'MarvelFact' ),
    FILTER (ALL ( 'MarvelFact' ),
        'MarvelFact'[DateID] <= MAX ( 'DateDim'[DateID] ))
)
```

**Format**: Number (#,0)

**Use Case**: Time-series analysis of cumulative character introduction trends.

---

### Date Filter

**Business Logic**: Provides a dynamic text label showing the date range context—either a specific month-year or the overall data range.

**DAX Code**:
```dax
VAR MinDate =
    FORMAT ( MIN ( 'DateDim'[Date] ), "MMMM, YYYY" )
VAR MaxDate =
    FORMAT ( MAX ( 'DateDim'[Date] ), "MMMM, YYYY" )
VAR SelDate =
    SELECTEDVALUE ( 'DateDim'[Date], "X" )
RETURN
    IF (
        SelDate = "X",
        "between " & MinDate & " and " & MaxDate,
        "for the month of " & FORMAT ( SelDate, "MMMM, YYYY" )
    )
```

**Format**: Text

**Use Case**: Dynamic date range labeling in reports and visualizations.

---

## Row-Level Security

No row-level security (RLS) rules are currently defined in this model. All data is visible to all users without row-level filtering.

---

## Data Sources

### Primary Data Source

**Source**: Remote CSV File
- **URL**: `https://raw.githubusercontent.com/fivethirtyeight/data/master/comic-characters/marvel-wikia-data.csv`
- **Load Method**: Web.Contents() function in Power Query
- **Format**: Comma-separated values (CSV)
- **Encoding**: UTF-8
- **Refresh**: Dynamic (refreshed on model refresh)

### Data Flow

1. **MarvelSource** loads raw Marvel character data from the GitHub CSV endpoint
2. **DateDim** is derived by extracting unique first appearance dates and expanding them to monthly granularity
3. **EyeColorDim** is created by deduplicating and standardizing eye color values
4. **AlignmentDim** is built by processing alignment codes with detailed transformation logic
5. **MarvelFact** is constructed by joining MarvelSource with all three dimensions on their respective keys

### Data Quality Notes

- Missing or null eye color values are replaced with "Not Available"
- Missing or null alignment values are replaced with "Not Available"
- First appearance dates without valid parsing default to "Jan-0001"
- The DateDim includes all months from the earliest character debut to the current date

---

*Documentation generated automatically from TMDL semantic model definition.*
