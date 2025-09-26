# Technical Assessment: SQL to Python Performance Optimization

## Context

Your financial BI dashboard contains a dropdown list populated by this SQL query: `SELECT * FROM [Altair].[AllocationDatesUniverse]('xxxxx.xxxx')`. The query is called on every dashboard refresh and page change. The `DM_RiskData` table contains around **300 million rows** and performance is unacceptable. 

The dropdown list contains **reporting dates that are fixed** for the day and **only updated once daily with new data**.

> **Your task**: Develop a Python API endpoint to replace this SQL function and optimize performance.

## Current Implementation

### Function call that's too slow:
```sql
SELECT * FROM [Altair].[AllocationDatesUniverse]('xxxxx.xxxx'')
-- Returns: 546 rows
```

### Table size:
```sql
SELECT COUNT(*) FROM DM_RiskData
-- Result: 277,997,117 rows
```

### Main function:
```sql
CREATE FUNCTION [Altair].[AllocationDatesUniverse]
(
    @userid VARCHAR(30)
)
RETURNS TABLE
AS
RETURN (
    WITH RiskMapClientFund AS (
        SELECT SecurityId, SecurityKey, SHOW_POSITIONS
        FROM [SecureLayer].fnRiskMapClientFund(@UserId)
    )
    SELECT DISTINCT ReportingDate AS REPORTING_DATE, 
                   'Y' AS verified,
                   CONVERT(VARCHAR(10), ReportingDate, 120) AS charDate
    FROM dbo.vw_RiskData_ReportingDate d
    JOIN RiskMapClientFund AS n
        ON d.SecurityKeyParent = n.SecurityKey
);
```

### Used:
```sql
CREATE VIEW [dbo].[vw_RiskData_ReportingDate]  
AS
SELECT SecurityKeyParent, ReportingDate, COUNT_BIG(*) [Count]
FROM [dbo].[DM_RiskData]
WHERE [RiskEngineId] = 1
GROUP BY SecurityKeyParent, ReportingDate
```

```sql
CREATE FUNCTION [SecureLayer].[fnRiskMapClientFund] (@UserId VARCHAR(30))
RETURNS TABLE WITH SCHEMABINDING
AS
RETURN
(
    SELECT SecurityId, SecurityKey, SHOW_POSITIONS
    FROM SecureLayer.MapClientFund WITH (NOLOCK)
    WHERE USER_IDC = @UserId AND SHOW_POSITIONS = 1
);
```

Table structure and indexes:
```sql
-- Primary Key
PRIMARY KEY (RiskEngineId, secpath, ReportingDate)
WITH (data_compression = page)
ON psReportingDate (ReportingDate)

-- Index 1: 
CREATE INDEX IDX_DM_RiskDataSwitch_ReportingDate
ON dbo.DM_RiskData (RiskEngineId, ReportingDate)
ON psReportingDate (ReportingDate)

-- Index 2: 
CREATE INDEX IDX_DM_RiskData_SecurityKeyParent
ON dbo.DM_RiskData (RiskEngineId, ReportingDate, SecurityKeyParent) 
INCLUDE (ChildKey, SecurityKeyDirectInvestment, UnproxiedSecurityKey)
ON psReportingDate (ReportingDate)

-- Index 3:
CREATE INDEX IX_DM_RiskData_RiskEngineId_SecurityKeyParent
ON dbo.DM_RiskData (ReportingDate, RiskEngineId, SecurityKeyParent)
ON psReportingDate (ReportingDate)
```

## Expected Output

546 available reporting dates for the user:

| REPORTING_DATE | verified | charDate   |
|---------------|----------|------------|
| 2025-08-05    | Y        | 2025-08-05 |
| 2025-08-06    | Y        | 2025-08-06 |
| ...           | ...      | ...        |

## Assignment 

**Develop a Python API endpoint** that replaces this SQL function with dramatically better performance.

### What to address:

#### 1. **Analysis** 
What does this SQL code do? What are the performance bottlenecks?

#### 2. **API Development Strategy** 
- How will you implement the Python API endpoint?
- What's your approach to reproduce the logic efficiently?
- How will you handle the 277M row dataset?
- What technologies and frameworks will you use?

#### 3. **Architecture & Implementation** 
- API endpoint design and structure
- Performance optimization approach

