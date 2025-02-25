---
title: "Create a Format File (SQL Server)"
description: When you bulk import or export a SQL Server table, a format file allows writing data files with little editing or reading data files from other programs.
author: rwestMSFT
ms.author: randolphwest
ms.date: 08/29/2022
ms.prod: sql
ms.technology: data-movement
ms.topic: conceptual
helpviewer_keywords:
  - "format files [SQL Server], creating"
monikerRange: ">=aps-pdw-2016||=azuresqldb-current||>=sql-server-2016||>=sql-server-linux-2017||=azuresqldb-mi-current"
---
# Create a Format File (SQL Server)

[!INCLUDE[SQL Server Azure SQL Database Synapse Analytics PDW ](../../includes/applies-to-version/sql-asdb-asdbmi-asa-pdw.md)]

This article describes how to use the [bcp utility](../../tools/bcp-utility.md) to create a format file for a particular table. The format file is based on the data-type option specified (**-n**, **-c**, **-w**,or **-N**) and the table or view delimiters.

When you bulk import into a [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] table or bulk export data from a table, you can use a format file to a flexible system for writing data files that requires little or no editing to comply with other data formats or to read data files from other software programs.

[!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] support two types of format file: non-XML format and XML format. The non-XML format is the original format that is supported by earlier versions of [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)].

Generally, XML and non-XML format files are interchangeable. However, we recommend that you use the XML syntax for new format files because they provide several advantages over non-XML format files.

> [!NOTE]  
> The version of the **bcp** utility (Bcp.exe) used to read a format file must be the same as, or later than the version used to create the format file. For example, [!INCLUDE[ssSQL11](../../includes/sssql11-md.md)] **bcp** can read a version 10.0 format file, which is generated by [!INCLUDE[ssKatmai](../../includes/sskatmai-md.md)] **bcp**, but [!INCLUDE[ssKatmai](../../includes/sskatmai-md.md)] **bcp** cannot read a version 11.0 format file, which is generated by [!INCLUDE[ssSQL11](../../includes/sssql11-md.md)] **bcp**.

> [!NOTE]  
> This syntax, including bulk insert, is not supported in Azure Synapse Analytics. [!INCLUDE[Use ADF or PolyBase instead of Synapse Bulk Insert](../../includes/paragraph-content/bulk-insert-synapse.md)]

## Create a Non-XML format file

To use a **bcp** command to create a format file, specify the **format** argument and use `nul` instead of a data-file path. The **format** option also requires the **-f** option, such as: `bcp _table_or_view_ format nul -f_format_file_name_`.

> [!NOTE]  
> To distinguish a non-XML format file, we recommend that you use .fmt as the file name extension, for example, `MyTable.fmt`.

For information about the structure and fields of non-XML format files, see [Non-XML Format Files &#40;SQL Server&#41;](../../relational-databases/import-export/non-xml-format-files-sql-server.md).

### Examples

This section contains the following examples that show how to use **bcp** commands to create a non-XML format file. The examples use the `HumanResources.Department` table in the [!INCLUDE[ssSampleDBobject](../../includes/sssampledbobject-md.md)] sample database. The `HumanResources.Department` table contains four columns: `DepartmentID`, `Name`, `GroupName`, and `ModifiedDate`.

#### A. Create a non-XML format file for native data

The following example creates an XML format file, `Department-n.xml`, for the [!INCLUDE[ssSampleDBnormal](../../includes/sssampledbnormal-md.md)] `HumanResources.Department` table. The format file uses native data types. The contents of the generated format file are presented after the command.

The **bcp** command contains the following qualifiers.

|Qualifiers|Description|  
|----------------|-----------------|  
|**formatnul-f** _format_file_|Specifies the non-XML format file.|  
|**-n**|Specifies native data types.|  
|**-T**|Specifies that the **bcp** utility connects to [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] with a trusted connection using integrated security. If **-T** is not specified, you must specify **-U** and **-P** to successfully log in.|

At the Windows command prompt, enter the following `bcp` command:

```cmd
bcp AdventureWorks2012.HumanResources.Department format nul -T -n -f Department-n.fmt
```

The generated format file, `Department-n.fmt`, contains the following information:

```output
12.0  
4  
1  SQLSMALLINT   0       2       ""   1     DepartmentID         ""  
2  SQLNCHAR      2       100     ""   2     Name                 SQL_Latin1_General_CP1_CI_AS  
3  SQLNCHAR      2       100     ""   3     GroupName            SQL_Latin1_General_CP1_CI_AS  
4  SQLDATETIME   0       8       ""   4     ModifiedDate         ""
```

For more information, see [Non-XML Format Files &#40;SQL Server&#41;](../../relational-databases/import-export/non-xml-format-files-sql-server.md).

#### B. Create a non-XML format file for character data

The following example creates an XML format file, `Department.fmt`, for the [!INCLUDE[ssSampleDBnormal](../../includes/sssampledbnormal-md.md)] `HumanResources.Department` table. The format file uses character data formats and a non-default field terminator (`,`). The contents of the generated format file are presented after the command.

The **bcp** command contains the following qualifiers.

|Qualifiers|Description|  
|----------------|-----------------|  
|**formatnul-f** _format_file_|Specifies a non-XML format file.|  
|**-c**|Specifies character data.|  
|**-T**|Specifies that the **bcp** utility connects to [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] with a trusted connection using integrated security. If **-T** is not specified, you must specify **-U** and **-P** to successfully log in.|

At the Windows command prompt, enter the following `bcp` command:

```cmd
bcp AdventureWorks2012.HumanResources.Department format nul -c -f Department-c.fmt -T
```

The generated format file, `Department-c.fmt`, contains the following information:

```output
12.0  
4  
1  SQLCHAR       0       7       "\t"     1     DepartmentID            ""  
2  SQLCHAR       0       100     "\t"     2     Name                    SQL_Latin1_General_CP1_CI_AS  
3  SQLCHAR       0       100     "\t"     3     GroupName               SQL_Latin1_General_CP1_CI_AS  
4  SQLCHAR       0       24      "\r\n"   4     ModifiedDate            ""
```

For more information, see [Non-XML Format Files &#40;SQL Server&#41;](../../relational-databases/import-export/non-xml-format-files-sql-server.md).

#### C. Create a non-XML format file for Unicode native data

To create a non-XML format file for Unicode native data for the `HumanResources.Department` table, use the following command:

```cmd
bcp AdventureWorks2012.HumanResources.Department format nul -T -N -f Department-n.fmt
```

For more information about how to use Unicode native data, see [Use Unicode Native Format to Import or Export Data &#40;SQL Server&#41;](../../relational-databases/import-export/use-unicode-native-format-to-import-or-export-data-sql-server.md).

#### D. Create a non-XML format file For Unicode character data

To create a non-XML format file for Unicode character data for the `HumanResources.Department` table that uses default terminators, use the following command:

```cmd
bcp AdventureWorks2012.HumanResources.Department format nul -T -w -f Department-w.fmt
```

For more information about how to use Unicode character data, see [Use Unicode Character Format to Import or Export Data &#40;SQL Server&#41;](../../relational-databases/import-export/use-unicode-character-format-to-import-or-export-data-sql-server.md).

#### F. Use a format file with the code page option

If you create a format file using the bcp command (that is, by using `bcp format`), information about the collation/code page will be written in the format file.

The following example format file for a table with 5 columns includes the collation.

```output
13.0  
5  
1  SQLCHAR         0       0       "**\t**"         1     c_0          Cyrillic_General_CS_AS  
2  SQLCHAR         0       0       "**\t**"         2     c_1          Cyrillic_General_CS_AS  
3  SQLCHAR         0       3000    "**\t**"         3     c_2          Cyrillic_General_CS_AS  
4  SQLCHAR         0       5       "**\t**"         4     c_3          ""  
5  SQLCHAR         0       41      "!!!\r\r\n"      5     c_4          ""

```

If you try to import data into [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] using `bcp in -c -C65001 -f format_file` ..." or "`BULK INSERT`/`OPENROWSET` ... `FORMATFILE='format_file' CODEPAGE=65001` ...", information about the collation/code page will have priority over 65001 option.

Therefore, if you generate a  format file, you must manually delete the collation info from the generated format file before you start importing data back into [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)].

The following is an example of the format file without the collation info.

```output
13.0  
5  
1  SQLCHAR         0       0       "**\t**"         1     c_0              ""  
2  SQLCHAR         0       0       "**\t**"         2     c_1              ""  
3  SQLCHAR         0       3000    "**\t**"         3     c_2              ""  
4  SQLCHAR         0       5       "**\t**"         4     c_3              ""  
5  SQLCHAR         0       41      "!!!\r\r\n"      5     c_4              ""
```

## Create an XML Format File

To use a **bcp** command to create a format file, specify the **format** argument and use **nul** instead of a data-file path. The **format** option always requires the **-f** option, and to create an XML format file, you must also specify the **-x** option, such as `bcp _table_or_view_ format nul -f _format_file_name_ -x`

> [!NOTE]  
> To distinguish an XML format file, we recommend that you use .xml as the file name extension, for example, MyTable.xml.

For information about the structure and fields of XML format files, see [XML Format Files &#40;SQL Server&#41;](../../relational-databases/import-export/xml-format-files-sql-server.md).

### Examples

This section contains the following examples that show how to use **bcp** commands to create an XML format file. The examples use the `HumanResources.Department` table in the [!INCLUDE[ssSampleDBobject](../../includes/sssampledbobject-md.md)] sample database. The `HumanResources.Department` table contains four columns: `DepartmentID`, `Name`, `GroupName`, and `ModifiedDate`.

> [!NOTE]  
>  [!INCLUDE[ssSampleDBdesc](../../includes/sssampledbdesc-md.md)]

#### A. Create an XML format file for character data

The following example creates an XML format file, `Department.xml`, for the [!INCLUDE[ssSampleDBnormal](../../includes/sssampledbnormal-md.md)]`HumanResources.Department` table. The format file uses character data formats and a non-default field terminator (`,`). The contents of the generated format file are presented after the command.

The **bcp** command contains the following qualifiers.

|Qualifiers|Description|  
|----------------|-----------------|  
|**formatnul-f** _format_file_ **-x**|Specifies the XML format file.|  
|**-c**|Specifies character data.|  
|**-t** `,`|Specifies a comma (**,**) as the field terminator.<br /><br /> Note: If the data file uses the default field terminator (`\t`), the **-t** switch is unnecessary.|  
|**-T**|Specifies that the **bcp** utility connects to [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] with a trusted connection using integrated security. If **-T** is not specified, you must specify **-U** and **-P** to successfully log in.|

At the Windows command prompt, enter the following `bcp` command:

```cmd
bcp AdventureWorks2012.HumanResources.Department format nul -c -x -f Department-c.xml -t, -T
```

The generated format file, `Department-c.xml`, contains the following XML elements:

```xml
<?xml version="1.0"?>  
<BCPFORMAT xmlns="https://schemas.microsoft.com/sqlserver/2004/bulkload/format" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">  
<RECORD>  
  <FIELD ID="1" xsi:type="CharTerm" TERMINATOR="," MAX_LENGTH="7"/>  
  <FIELD ID="2" xsi:type="CharTerm" TERMINATOR="," MAX_LENGTH="100" COLLATION="SQL_Latin1_General_CP1_CI_AS"/>  
  <FIELD ID="3" xsi:type="CharTerm" TERMINATOR="," MAX_LENGTH="100" COLLATION="SQL_Latin1_General_CP1_CI_AS"/>  
  <FIELD ID="4" xsi:type="CharTerm" TERMINATOR="\r\n" MAX_LENGTH="24"/>  
</RECORD>  
<ROW>  
  <COLUMN SOURCE="1" NAME="DepartmentID" xsi:type="SQLSMALLINT"/>  
  <COLUMN SOURCE="2" NAME="Name" xsi:type="SQLNVARCHAR"/>  
  <COLUMN SOURCE="3" NAME="GroupName" xsi:type="SQLNVARCHAR"/>  
  <COLUMN SOURCE="4" NAME="ModifiedDate" xsi:type="SQLDATETIME"/>  
</ROW>  
</BCPFORMAT>
```

For information about the syntax of this format file, see [XML Format Files &#40;SQL Server&#41;](../../relational-databases/import-export/xml-format-files-sql-server.md). For information about character data, see [Use Character Format to Import or Export Data &#40;SQL Server&#41;](../../relational-databases/import-export/use-character-format-to-import-or-export-data-sql-server.md).

#### B. Create an XML format file for native data

The following example creates an XML format file, `Department-n.xml`, for the `HumanResources.Department` table. The format file uses native data types. The contents of the generated format file are presented after the command.

The **bcp** command contains the following qualifiers.

|Qualifiers|Description|  
|----------------|-----------------|  
|**formatnul-f** _format_file_ **-x**|Specifies the XML format file.|  
|**-n**|Specifies native data types.|  
|**-T**|Specifies that the **bcp** utility connects to [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] with a trusted connection using integrated security. If **-T** is not specified, you must specify **-U** and **-P** to successfully log in.|

At the Windows command prompt, enter the following `bcp` command:

```cmd
bcp AdventureWorks2012.HumanResources.Department format nul -x -f Department-n.xml -n -T
```

The generated format file, `Department-n.xml`, contains the following XML elements:

```xml
<?xml version="1.0"?>  
<BCPFORMAT xmlns="https://schemas.microsoft.com/sqlserver/2004/bulkload/format" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">  
<RECORD>  
  <FIELD ID="1" xsi:type="NativeFixed" LENGTH="2"/>  
  <FIELD ID="2" xsi:type="NCharPrefix" PREFIX_LENGTH="2" MAX_LENGTH="100" COLLATION="SQL_Latin1_General_CP1_CI_AS"/>  
  <FIELD ID="3" xsi:type="NCharPrefix" PREFIX_LENGTH="2" MAX_LENGTH="100" COLLATION="SQL_Latin1_General_CP1_CI_AS"/>  
  <FIELD ID="4" xsi:type="NativeFixed" LENGTH="8"/>  
</RECORD>  
<ROW>  
  <COLUMN SOURCE="1" NAME="DepartmentID" xsi:type="SQLSMALLINT"/>  
  <COLUMN SOURCE="2" NAME="Name" xsi:type="SQLNVARCHAR"/>  
  <COLUMN SOURCE="3" NAME="GroupName" xsi:type="SQLNVARCHAR"/>  
  <COLUMN SOURCE="4" NAME="ModifiedDate" xsi:type="SQLDATETIME"/>  
</ROW>  
</BCPFORMAT>
```

For information about the syntax of this format file, see [XML Format Files &#40;SQL Server&#41;](../../relational-databases/import-export/xml-format-files-sql-server.md). For information about how to use native data, see [Use Native Format to Import or Export Data &#40;SQL Server&#41;](../../relational-databases/import-export/use-native-format-to-import-or-export-data-sql-server.md).

## Map data fields to table columns

As created by **bcp**, a format file describes all the table columns in order. You can modify a format file to rearrange or omit table rows. This lets you customize a format file to a data file whose fields do not map directly to the table columns. For more information, see the following topics:

- [Use a Format File to Skip a Table Column &#40;SQL Server&#41;](../../relational-databases/import-export/use-a-format-file-to-skip-a-table-column-sql-server.md)

- [Use a Format File to Skip a Data Field &#40;SQL Server&#41;](../../relational-databases/import-export/use-a-format-file-to-skip-a-data-field-sql-server.md)

- [Use a Format File to Map Table Columns to Data-File Fields &#40;SQL Server&#41;](../../relational-databases/import-export/use-a-format-file-to-map-table-columns-to-data-file-fields-sql-server.md)

## Next steps

- [bcp Utility](../../tools/bcp-utility.md)
- [Use a Format File to Map Table Columns to Data-File Fields &#40;SQL Server&#41;](../../relational-databases/import-export/use-a-format-file-to-map-table-columns-to-data-file-fields-sql-server.md)
- [Use a Format File to Skip a Table Column &#40;SQL Server&#41;](../../relational-databases/import-export/use-a-format-file-to-skip-a-table-column-sql-server.md)
- [Use a Format File to Skip a Data Field &#40;SQL Server&#41;](../../relational-databases/import-export/use-a-format-file-to-skip-a-data-field-sql-server.md)
- [Non-XML Format Files &#40;SQL Server&#41;](../../relational-databases/import-export/non-xml-format-files-sql-server.md)
- [XML Format Files &#40;SQL Server&#41;](../../relational-databases/import-export/xml-format-files-sql-server.md)
