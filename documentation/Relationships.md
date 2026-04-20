# Relationships – Power BI Semantic Model Documentation

> *Auto-generated documentation.*

## Table of Contents
1. [Overview](#overview)
2. [Table Relationships](#table-relationships)
3. [Tables and Columns](#tables-and-columns)
4. [Measures](#measures)
5. [Row-Level Security](#row-level-security)
6. [Data Sources](#data-sources)

## Overview

The **Relationships** model is a demonstration semantic model designed to showcase different relationship configurations and cardinality patterns in Power BI. It serves as a reference implementation for testing various relationship scenarios, including one-to-many, one-to-one relationships, and inactive relationships.

| Aspect | Count |
|--------|-------|
| Tables | 5 |
| Relationships | 4 |
| Active Relationships | 3 |
| Inactive Relationships | 1 |
| Measures | 0 |

### Model Purpose

This model is used to explore and demonstrate:
- Standard one-to-many relationships between fact and dimension tables
- One-to-one relationships with bidirectional filtering
- Inactive relationships for alternative join paths
- Snowflake schema patterns with multiple filter paths

---

## Table Relationships

The model demonstrates multiple relationship patterns with varying cardinality and directionality:

````mermaid
erDiagram
    "Fact Table" ||--o| "Dimension Table" : "Active"
    "Fact Table 2" ||--o| "Dimension Table" : "Inactive"
    OneToOne ||--|| "Dimension Table" : "Bidirectional"
    "Fact Table" ||--o| "Snowflake Filter" : "Active"
    
    "Dimension Table" {
        string Test PK
    }
    
    "Fact Table" {
        string "Test ID" FK
    }
    
    "Fact Table 2" {
        string "Test ID" FK
    }
    
    OneToOne {
        string Test FK
    }
    
    "Snowflake Filter" {
        string Test FK
    }
````

### Relationship Details

| From Table | From Column | To Table | To Column | Type | Cardinality | Directionality | Status |
|-----------|-------------|----------|-----------|------|-------------|------------------|--------|
| Fact Table | Test ID | Dimension Table | Test | Standard | Many-to-One | One Direction | Active |
| Fact Table 2 | Test ID | Dimension Table | Test | Standard | Many-to-One | One Direction | **Inactive** |
| OneToOne | Test | Dimension Table | Test | One-to-One | One-to-One | Both Directions | Active |
| Fact Table | Test ID | Snowflake Filter | Test | Standard | Many-to-One | One Direction | Active |

---

## Tables and Columns

### Dimension Table

The primary dimension table used as a target for multiple relationship paths.

| Column | Data Type | Description | Key | Summarization |
|--------|-----------|-------------|-----|----------------|
| Test | string | Dimension attribute | Primary | None |

**Data Source**: Inline embedded table (3 rows of test data)

### Fact Table

A standard fact table with a many-to-one relationship to the Dimension Table.

| Column | Data Type | Description | Key | Summarization |
|--------|-----------|-------------|-----|----------------|
| Test ID | string | Foreign key to Dimension Table | Foreign | None |

**Data Source**: Inline embedded table (2 rows of test data)

**Relationships**:
- One-to-many with Dimension Table (Active)
- One-to-many with Snowflake Filter (Active)

### Fact Table 2

A secondary fact table demonstrating inactive relationships.

| Column | Data Type | Description | Key | Summarization |
|--------|-----------|-------------|-----|----------------|
| Test ID | string | Foreign key to Dimension Table | Foreign | None |

**Data Source**: Inline embedded table (2 rows of test data)

**Relationships**:
- One-to-many with Dimension Table (Inactive - alternative path for analysis)

### OneToOne

A dimension table demonstrating one-to-one relationships with bidirectional filtering.

| Column | Data Type | Description | Key | Summarization |
|--------|-----------|-------------|-----|----------------|
| Test | string | Dimension attribute | Foreign | None |

**Data Source**: Inline embedded table (3 rows of test data)

**Relationships**:
- One-to-one with Dimension Table (Active, bidirectional filtering enabled)

### Snowflake Filter

A detail dimension in a snowflake schema pattern, connected through the Fact Table.

| Column | Data Type | Description | Key | Summarization |
|--------|-----------|-------------|-----|----------------|
| Test | string | Dimension attribute | Foreign | None |

**Data Source**: Inline embedded table (1 row of test data)

**Relationships**:
- One-to-many with Fact Table (Active, enables snowflake filtering)

---

## Measures

No measures are currently defined in this model. It serves as a relationship structure demonstration without business calculations.

---

## Row-Level Security

No row-level security (RLS) rules are currently defined in this model. All data is visible to all users without row-level filtering.

---

## Data Sources

### Data Source Type

All tables in this model use **inline embedded data sources** defined directly in Power Query M expressions. Data is not connected to external databases or files.

### Data Source Descriptions

#### Dimension Table
```
Source: Inline JSON embedded in Power Query
Format: Table with Test column (text)
Rows: 3
Encoding: Base64 compressed JSON
```

#### Fact Table
```
Source: Inline JSON embedded in Power Query
Format: Table with Test ID column (text)
Rows: 2
Encoding: Base64 compressed JSON
```

#### Fact Table 2
```
Source: Inline JSON embedded in Power Query
Format: Table with Test ID column (text)
Rows: 2
Encoding: Base64 compressed JSON
```

#### OneToOne
```
Source: Inline JSON embedded in Power Query
Format: Table with Test column (text)
Rows: 3
Encoding: Base64 compressed JSON
```

#### Snowflake Filter
```
Source: Inline JSON embedded in Power Query
Format: Table with Test column (text)
Rows: 1
Encoding: Base64 compressed JSON
```

### Loading Mechanism

All tables use the following Power Query pattern:
1. **Source**: Embedded JSON data compressed with Base64 and DEFLATE compression
2. **Decompression**: Binary.Decompress() with Compression.Deflate
3. **Parsing**: Json.Document() to convert binary to table
4. **Type Transformation**: Table.TransformColumnTypes() to enforce data types

This approach ensures lightweight, self-contained test data without external dependencies.

---

## Usage Notes

This model is intended for:
- Testing Power BI relationship configurations
- Demonstrating different cardinality and directionality options
- Validating cross-filter behavior
- Understanding active vs. inactive relationship paths

**Note**: The inactive relationship between Fact Table 2 and Dimension Table can be used via `USERELATIONSHIP()` in DAX calculations when alternative analytical paths are needed.

---

*Documentation generated automatically from TMDL semantic model definition.*
