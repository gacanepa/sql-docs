---
title: SQL Data Discovery & Classification | Microsoft Docs
description: SQL Data Discovery & Classification
documentationcenter: ''
ms.reviewer: vanto
ms.assetid: 89c2a155-c2fb-4b67-bc19-9b4e03c6d3bc
ms.service: sql-database
ms.prod_service: sql-database,sql
ms.custom: security
ms.topic: conceptual
ms.date: 02/22/2022
ms.author: matripathy
author: Madhumitatripathy
---
# SQL Data Discovery and Classification
[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

Data Discovery & Classification adds capabilities for **discovering**, **classifying**, **labeling** & **reporting** the sensitive data in your databasesc. This can be done via T-SQL or using [SQL Server Management Studio (SSMS)](../../ssms/download-sql-server-management-studio-ssms.md) .
Discovering and classifying your most sensitive data (business, financial, healthcare, etc.) can play a pivotal role in your organizational information protection stature. It can serve as infrastructure for:
* Helping meet data privacy standards.
* Monitoring access to databases/columns containing highly sensitive data.

> [!NOTE]
> Data Discovery & Classification is **supported for SQL Server 2012 and later, and can be used with [SSMS 17.5](../../ssms/download-sql-server-management-studio-ssms.md) or later**. For Azure SQL Database, see [Azure SQL Database Data Discovery & Classification](/azure/sql-database/sql-database-data-discovery-and-classification/).

## <a id="Overview"></a>Overview
Data Discovery & Classification forms a new information-protection paradigm for SQL Database, SQL Managed Instance, and Azure Synapse, aimed at protecting the data and not just the database. Currently it supports the following capabilities:

* **Discovery & recommendations** - The classification engine scans your database and identifies columns containing potentially sensitive data. It then provides you an easy way to review and apply the appropriate classification recommendations, as well as to manually classify columns.
* **Labeling** - Sensitivity classification labels can be persistently tagged on columns.
* **Visibility** - The database classification state can be viewed in a detailed report that can be printed/exported to be used for compliance & auditing purposes, as well as other needs.

## <a id="Discovering-classifying-labeling-sensitive-columns"></a>Discovering, classifying & labeling sensitive columns
The following section describes the steps for discovering, classifying, and labeling columns containing sensitive data in your database, as well as viewing the current classification state of your database and exporting reports.

The classification includes two metadata attributes:
* Labels - The main classification attributes, used to define the sensitivity level of the data stored in the column.  
* Information Types - Provide additional granularity into the type of data stored in the column.

**To classify your SQL Server database:**

1. In SQL Server Management Studio (SSMS) connect to the SQL Server.

2. In the SSMS Object Explorer, right click on the database that you would like to classify and choose **Tasks** > **Data Discovery and Classification** > **Classify Data...**.

   ![Screenshot showing the SSMS Object Explorer with Tasks > Data Discovery and Classification > Classify Data... selected.][0]

3. The classification engine scans your database for columns containing potentially sensitive data and provides a list of **recommended column classifications**:

    * To view the list of recommended column classifications, click on the recommendations notification box at the top or the recommendations panel at the bottom of the window:

        ![Screenshot showing the notification that says We have found 39 columns with classification recommendations. Click here to view them.][2]

        ![Screenshot showing the notification that says 39 columns with classification recommendations (click to view).][3]

    * Review the list of recommendations:
        * To accept a recommendation for a specific column, check the checkbox in the left column of the relevant row. You can also mark *all recommendations* as accepted by checking the checkbox in the recommendations table header.

        * You can also change the recommended Information Type and Sensitivity Label using the drop down boxes.        

        ![Screenshot showing the list of recommendations.][4]

    * To apply the selected recommendations, click on the blue **Accept selected recommendations** button.

        ![Screenshot of the Accept selected recommendations button.][5]

4. To display the classified columns, select appropriate **schema** and corresponding **table** from the drop down, then click **Load Columns**.

   :::image type="content" source="media/sql-data-discovery-and-classification/data-classification-load-columns.png" alt-text="screenshot of SSMS data classification loading classified columns.":::

5. You can also **manually classify** columns as an alternative, or in addition, to the recommendation-based classification:

    * Click on **Add classification** in the top menu of the window.

        ![Screenshot showing the top menu with the Add classification option called out.][6]

    * In the context window that opens, enter the column name that you want to classify, information type and the sensitivity label. Schema and table are selected based on the entries in the main page.

        ![Screenshot showing the Add Classification context window.][7]

    - If you want to add classification for all the unclassified columns for a specific table in a single attempt, then select **All Unclassified** in the **Column** drop down of **Add Classification** page.
    
        :::image type="content" source="media/sql-data-discovery-and-classification/data-classification-all-unclassified-column-selection.png" alt-text="screenshot of SSMS data classification selecting all unclassified columns":::

6. To complete your classification and persistently label (tag) the database columns with the new classification metadata, click on **Save** in the top menu of the window.

    ![Screenshot showing the top menu with the Save option called out.][8]


7. To generate a report with a full summary of the database classification state, click on **View Report** in the top menu of the window. (You can also generate a report using SSMS. Right click on the database where you would like to generate the report, and choose **Tasks** > **Data Discovery and Classification** > **Generate Report...**)

    ![Screenshot showing the top menu with the View Report option called out.][9]

    ![Screenshot showing the SQL Data Classification Report.][10]

## <a id="Manage-information-protection-policy-with-SSMS"></a>Manage information protection policy with SSMS

You can manage the information protection policy using [SSMS 18.4](../../ssms/download-sql-server-management-studio-ssms.md) or later:

1. In SQL Server Management Studio (SSMS) connect to the SQL Server.

2. In the SSMS Object Explorer, right click on one of your databases and choose **Tasks** > **Data Discovery and Classification**.

   The following menu options allow you to manage the information protection policy:

* **Set Information Protection Policy File**: uses the information protection policy as defined in the selected JSON file. (See the default [Information Protection Policy File](https://github.com/Azure-Samples/sql-data-classification/blob/main/sql_information_protection_default.json))

* **Export Information Protection Policy**: exports the information protection policy to a JSON file.

* **Reset Information Protection Policy**: resets the information protection policy to the default information protection policy.

> [!IMPORTANT]
> Information protection policy file is not stored in the SQL Server.
> SSMS uses a default information protection policy. If an information protection policy customized fails, SSMS cannot use the default policy. Data classification fails. To resolve, click **Reset Information Protection Policy** to use the default policy and re-enable data classification.

## <a id="sAccessing-the-classification-metadata"></a>Accessing the classification metadata

SQL Server 2019 introduces [`sys.sensitivity_classifications`](../system-catalog-views/sys-sensitivity-classifications-transact-sql.md) system catalog view. This view returns information types and sensitivity labels.

On SQL Server 2019 instances, query `sys.sensitivity_classifications` to review all classified columns with their corresponding classifications. For example: 

```sql
SELECT 
    schema_name(O.schema_id) AS schema_name,
    O.NAME AS table_name,
    C.NAME AS column_name,
    information_type,
	label,
	rank,
	rank_desc
FROM sys.sensitivity_classifications sc
    JOIN sys.objects O
    ON  sc.major_id = O.object_id
	JOIN sys.columns C 
    ON  sc.major_id = C.object_id  AND sc.minor_id = C.column_id
```

Prior to SQL Server 2019, the classification metadata for information types and sensitivity labels is in the following Extended Properties: 

* `sys_information_type_name`
* `sys_sensitivity_label_name`

For instances of SQL Server 2017 and prior, the following example returns all classified columns with their corresponding classifications:

```sql
SELECT
    schema_name(O.schema_id) AS schema_name,
    O.NAME AS table_name,
    C.NAME AS column_name,
    information_type,
    sensitivity_label 
FROM
    (
        SELECT
            IT.major_id,
            IT.minor_id,
            IT.information_type,
            L.sensitivity_label 
        FROM
        (
            SELECT
                major_id,
                minor_id,
                value AS information_type 
            FROM sys.extended_properties 
            WHERE NAME = 'sys_information_type_name'
        ) IT 
        FULL OUTER JOIN
        (
            SELECT
                major_id,
                minor_id,
                value AS sensitivity_label 
            FROM sys.extended_properties 
            WHERE NAME = 'sys_sensitivity_label_name'
        ) L 
        ON IT.major_id = L.major_id AND IT.minor_id = L.minor_id
    ) EP
    JOIN sys.objects O
    ON  EP.major_id = O.object_id 
    JOIN sys.columns C 
    ON  EP.major_id = C.object_id AND EP.minor_id = C.column_id
```

## <a id="Permissions"></a>Permissions

On SQL Server 2019 instances, viewing classification requires **VIEW ANY SENSITIVITY CLASSIFICATION** permission. For more information, see [Metadata Visibility Configuration](./metadata-visibility-configuration.md).

Prior to SQL Server 2019, the metadata can be accessed using the Extended Properties catalog view [`sys.extended_properties`](../system-catalog-views/extended-properties-catalog-views-sys-extended-properties.md).

Managing classification requires ALTER ANY SENSITIVITY CLASSIFICATION permission. The ALTER ANY SENSITIVITY CLASSIFICATION is implied by the database permission ALTER, or by the server permission CONTROL SERVER.

## <a id="subheading-5"></a>Manage classifications

# [T-SQL](#tab/t-sql)
You can use T-SQL to add/remove column classifications, as well as retrieve all classifications for the entire database.

- Add/update the classification of one or more columns: [ADD SENSITIVITY CLASSIFICATION](../../t-sql/statements/add-sensitivity-classification-transact-sql.md)
- Remove the classification from one or more columns: [DROP SENSITIVITY CLASSIFICATION](../../t-sql/statements/drop-sensitivity-classification-transact-sql.md)

# [PowerShell Cmdlet](#tab/sql-powelshell)
You can use PowerShell Cmdlet to add/remove column classifications, as well as retrieve all classifications and get recommendations for the entire database.

- [Get-SqlSensitivityClassification](/powershell/module/sqlserver/Get-SqlSensitivityClassification)
- [Get-SqlSensitivityRecommendations](/powershell/module/sqlserver/Get-SqlSensitivityRecommendations)
- [Set-SqlSensitivityClassification](/powershell/module/sqlserver/Set-SqlSensitivityClassification)
- [Remove-SqlSensitivityClassification](/powershell/module/sqlserver/Remove-SqlSensitivityClassification)

## <a id="Next-steps"></a>Next steps

For Azure SQL Database, see [Azure SQL Database Data Discovery & Classification](/azure/azure-sql/database/data-discovery-and-classification-overview).

Consider protecting your sensitive columns by applying column level security mechanisms:

* [Dynamic Data Masking](./dynamic-data-masking.md) for obfuscating sensitive columns in use.
* [Always Encrypted](./encryption/always-encrypted-database-engine.md) for encrypting sensitive columns at rest.

<!--Anchors-->
[SQL Data Discovery & Classification overview]: #subheading-1
[Discovering, classifying & labeling sensitive columns]: #subheading-2
[Manage information protection policy with SSMS]: #subheading-3
[Accessing the classification metadata]: #subheading-4
[Manage classifications]: #subheading-5
[Next Steps]: #subheading-6

<!--Image references-->

[0]: ./media/sql-data-discovery-and-classification/0-data-classification-explorer.png
[2]: ./media/sql-data-discovery-and-classification/2-recommendations-notification-box.png
[3]: ./media/sql-data-discovery-and-classification/3-recommendations-panel.png
[4]: ./media/sql-data-discovery-and-classification/4-recommendations.png
[5]: ./media/sql-data-discovery-and-classification/5-accept-recommendations-button.png
[6]: ./media/sql-data-discovery-and-classification/6-add-classification-button.png
[7]: ./media/sql-data-discovery-and-classification/7-manual-classification.png
[8]: ./media/sql-data-discovery-and-classification/8-save.png
[9]: ./media/sql-data-discovery-and-classification/9-view-report.png
[10]: ./media/sql-data-discovery-and-classification/10-report.png