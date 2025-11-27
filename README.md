# Aradas L&L Fields and Variables Documentation

## Table of Contents
- [Overview](#overview)
- [QID Texts](#qid-texts)
  - [Configuration](#configuration)
  - [Materials and Non-Alphanumeric Options](#materials-and-non-alphanumeric-options)
  - [Alphanumeric Option Values](#alphanumeric-option-values)
  - [Data Sources](#data-sources)
  - [Multi-Language Support](#multi-language-support)
  - [Data Storage](#data-storage)
- [Custom SQL Query Variables](#custom-sql-query-variables)
  - [Query Configuration](#query-configuration)
  - [Query Definitions](#query-definitions)
  - [Query Requirements](#query-requirements)
  - [Data Storage](#data-storage-1)
- [Standard Fields & Variables](#standard-fields--variables)
- [Model Variables](#model-variables)
- [PDG Price Document Groups](#pdg-price-document-groups)
- [Related Features](#related-features)
  - [Transportation](#transportation)
  - [Profile Optimization Remnants](#profile-optimization-remnants)

---

## Overview

This document describes the fields and variables system used in Aradas List & Label (L&L) reporting.

**Test Case**: [View Sample Document](https://stg.prefweb.com/Aradas/PrefWeb/SalesDocuments/Edit?number=101599&version=1)

---

## QID Texts

The QID feature enables custom formatting of `SalesDocItem` descriptions by grouping, sorting, and formatting base materials and P-Options values. These descriptions are organized into blocks defined by `DesAutoOrden` start and end indexes, with each block mapped to specific L&L fields for flexible printout configurations.

### Configuration

**Module**: `SalesModule`  
**Events**: `Sales_DefineVariablesAndFields`, `Sales_SetFields`  
**Conditions**: Global variable `QIDEnabled = 1`  
**Settings Tables**: `dbo.UniwaveQID`, `dbo.Uniwave_QIDNumericOptions`

### Materials and Non-Alphanumeric Options

Settings are stored in `dbo.UniwaveQID`:

```sql
SELECT [RowId]           -- Unique identifier
      ,[BlockName]       -- L&L variable name (QID.<BlockName>)
      ,[BlockDesc]       -- Description displayed in UI
      ,[PrintFlag]       -- Print flag: 1=Yes, 0=No (always 1 in Aradas)
      ,[BlockType]       -- Type: 0=Materials, 1=Options
      ,[BlockStart]      -- Starting DesAuto order
      ,[BlockEnd]        -- Ending DesAuto order
      ,[BlockFields]     -- Display mode: 0=hidden, 1=separate lines, 2=TBD, 3=grouped with commas
      ,[BlockOption]     -- Text separator: 0=newline, 1=space, 2=comma
      ,[FieldsOption]    -- Not used in Aradas
      ,[GlassDimensions] -- Not used in Aradas
FROM [dbo].[UniwaveQID]
```

**Example Configuration**:

| BlockName     | BlockDesc            | BlockType | BlockStart | BlockEnd | BlockFields | BlockOption |
|---------------|---------------------|-----------|------------|----------|-------------|-------------|
| GlassUnits    | Glass Units         | 0         | 1500       | 1799     | 3           | 0           |
| Threshold     | MainProfileThreshold| 0         | 650        | 655      | 0           | 2           |
| GlazingBeads  | Glazing beads       | 0         | 3000       | 3499     | 1           | 2           |
| MountingSides | Mounting sides      | 0         | 7501       | 7504     | 2           | 2           |

### Alphanumeric Option Values

Settings are stored in `dbo.Uniwave_QIDNumericOptions`:

```sql
SELECT [RowId]
      ,[OptionName]
      ,[BlockOrden]
      ,[Settings]
FROM [dbo].[Uniwave_QIDNumericOptions]
```

**Example Configuration**:

| OptionName                | BlockOrden | Settings |
|---------------------------|------------|----------|
| Model Color               | 1840       | 1        |
| Frame Color               | 1841       | 1        |
| Sash Color                | 1842       | 1        |
| Mullion Color             | 1843       | 1        |
| Manual Glass Description  | 1690       | 1        |

### Data Sources

Data is extracted from:
- Material details (references, descriptions, colors, dimensions)
- Options and option values
- Field numbers and other data from `Interop.PrefSales.SalesDocItem.DescriptiveXML`

### Multi-Language Support

The system supports multiple languages by:
- Using the sales document's primary language
- Accessing additional languages via `Interop.PrefSales.SalesDoc.AdditionalLanguages`
- Querying translated values from `dbo.LanguageContent` table

### Data Storage

Generated descriptions are cached in `dbo.Uniwave_ContenidoPAFDescriptions`:

```sql
SELECT [Number]
      ,[Version]
      ,[Position]
      ,[LanguageId]
      ,[QIDRowId]
      ,[Description]
FROM [dbo].[Uniwave_ContenidoPAFDescriptions]
```

Data is generated and stored during the `Sales_DefineVariablesAndFields` event. The DesAuto text, ordering, and translations are cached to minimize database queries.

---

## Custom SQL Query Variables

This feature allows custom variables to be defined through user-specified SQL queries.

**Module**: `SalesModule`  
**Events**: `Sales_DefineVariablesAndFields`, `Sales_SetVariables`  
**Example Variables**: `User.PhonePersonalMobile`, `User.Fax`  
**Settings Tables**: `Uniwave_CustomReportsQuerys`, `Uniwave_CustomReportsQueryList`

### Query Configuration

```sql
SELECT [RowId]
      ,[Name]        -- Query name
      ,[Kind]        -- Document type (e.g., SalesDoc)
      ,[Document]    -- Print template layout name
      ,[QueryRowId]  -- Reference to query definition
      ,[Type]        -- 1=Variables, 2=Fields
      ,[SortOrder]
      ,[Drawing]
      ,[XmlSource]
FROM [dbo].[Uniwave_CustomReportsQuerys]
WHERE Kind = 'SalesDoc' AND [Name] = '00 - Commercial Document'
```

### Query Definitions

```sql
SELECT [RowId]       -- Unique query identifier
      ,[Name]        -- Query name
      ,[Kind]        -- 1=Variables, 2=Fields
      ,[Body]        -- SQL query body
      ,[SortOrder]
      ,[Type]
      ,[Namespace]
FROM [dbo].[Uniwave_CustomReportsQueryList]
```

**Example Queries**:

| Name           | Body |
|----------------|------|
| Transport      | `SELECT SUM(Amount) [Transport.Amount], (SELECT ISNULL([dbo].[Uniwave_GetTransportDistr]({Number},{Version}),N'No')) [Transport.PreSetting] FROM dbo.vwSalesDetail WHERE System=N'Transport' AND Number={Number} AND Version={Version}` |
| Transport Type | `SELECT ISNULL([dbo].[Uniwave_GetTransportType]({Number},{Version}),N'NoPresettings') [TransportType.PreSetting]` |

### Query Requirements

**Variables Queries** must return:
- `Number` column
- `Version` column
- Additional columns become available as variables

**Fields Queries** must return:
- `Number` column
- `Version` column
- `Orden` column
- Additional columns become available as fields

### Data Storage

Fields and variables are **not** stored persistently. They are defined and set only during event execution.

---

## Standard Fields & Variables

Aradas includes hardcoded fields used in sales reports, such as `TEXTO` and `ProductType`. Variables are extracted directly from `Interop.PrefSales.SalesDoc` and `Interop.PrefSales.SalesDocItem` properties.

**Module**: `SalesModule`  
**Events**: `Sales_DefineVariablesAndFields`, `Sales_SetFields`  
**Settings**: None (hardcoded)  
**Data Storage**: Not stored; calculated during event execution

> **Note**: All variables and fields are calculated per individual `SalesDocItem`. No aggregations or relationships to other items or documents are included.

---

## Model Variables

> **Future Enhancement**: This feature could be replaced by kernel functionality. Consider implementing a flag system to mark required Model Variables as available for Sales printouts.

---

## PDG Price Document Groups

> **Status**: Not currently used in Aradas.

---

## Related Features

### Transportation

This specialized feature automatically adds a `SalesDocItem` to the end of each `SalesDoc` for transportation calculations. The system calculates packing (pallets) and transport pricing based on presetting parameters selected in the user menu.

**Module**: `SalesModule`  
**Events**: `SalesDoc_BeforeSave`, `SalesDoc_Validate`  
**Settings**: Custom database tables

#### Calculation Process

1. **User Presettings Selection**  
   Users configure transportation parameters through the menu interface.
	![Presettings](https://github.com/mjokubauskas/aradas-l-and-l/blob/uni/images/presettings.png)

2. **Position Creation**  
   A new line item named "LDM" (Loading Meters) is automatically added to the sales document.
   ![Position](https://github.com/mjokubauskas/aradas-l-and-l/blob/uni/images/ldm-position.png)

3. **Pricing Calculation**  
   The system performs complex calculations based on `SalesDocItem` properties including:
   - Width
   - Height
   - Thickness
   - Weight
   - Other relevant dimensions

> **Execution Timing**: Calculation runs once per sales document during the `SalesDoc_BeforeSave` event, **not** when individual items are added. This occurs as the final step before printing quotes or orders.

#### Validation

The validation process checks the sales document and outputs errors to the **Validation Messages** section.
![Validation](https://github.com/mjokubauskas/aradas-l-and-l/blob/uni/images/validation.png)

> **Execution Timing**: Validation runs during the `SalesDoc_Validate` event, **not** when individual items are added. This occurs as the final step before printing.

**Data Storage**: Stored as a `SalesDocItem` within the `SalesDoc`.

---

### Profile Optimization Remnants

**Status**: Not currently implemented, but included in the Aradas roadmap.

**Purpose**: Calculate remnant costs in sales documents using `Preference.Customization.SignedRetales.Retales.CalculoRetales`.

---

## Additional Resources

For questions or support, please contact the Aradas development team.

---

## Document Information

**Version**: 1.0  
**Last Updated**: November 2024  
**Maintained By**: Aradas Development Team
