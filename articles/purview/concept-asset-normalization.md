---
title: Asset normalization
description: Learn how Azure Purview prevents duplicate assets in your data map through asset normalization
author: djpmsft
ms.author: daperlov
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 07/23/2021
ms.custom: ignite-fall-2021
---

# Asset normalization

When ingesting assets into the Azure Purview data map, different sources updating the same data asset may send similar, but slightly different qualified names. While these qualified names represent the same asset, slight differences such as an extra character or different capitalization may cause these assets on the surface to appear different. To avoid storing duplicate entries and causing confusion when consuming the data catalog, Azure Purview applies normalization during ingestion to ensure all fully qualified names of the same entity type are in the same format.

For example, you scan in an Azure Blob with the qualified name `https://myaccount.file.core.windows.net/myshare/folderA/folderB/my-file.parquet`. This blob is also consumed by an Azure Data Factory pipeline which will then add lineage information to the asset. The ADF pipeline may be configured to read the file as `https://myAccount.file.core.windows.net//myshare/folderA/folderB/my-file.parquet`. While the qualified name is different, this ADF pipeline is consuming the same piece of data. Normalization ensures that all the metadata from both Azure Blob Storage and Azure Data Factory is visible on a single asset, `https://myaccount.file.core.windows.net/myshare/folderA/folderB/my-file.parquet`.

## Normalization rules

Below are the normalization rules applied by Azure Purview.

### Encode curly brackets
Applies to: All Assets

Before: `https://myaccount.file.core.windows.net/myshare/{folderA}/folder{B/`

After: 	`https://myaccount.file.core.windows.net/myshare/%7BfolderA%7D/folder%7BB/`

### Trim section spaces
Applies to: Azure Blob, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure Data Factory, Azure SQL Database, Azure SQL Database Managed Instance, Azure SQL pool, Azure Cosmos DB, Azure Cognitive Search, Azure Data Explorer, Azure Data Share, Amazon S3

Before: `https://myaccount.file.core.windows.net/myshare/  folder A/folderB   /`

After: 	`https://myaccount.file.core.windows.net/myshare/folder A/folderB/`

### Remove hostname spaces
Applies to: Azure Blob, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure SQL Database, Azure SQL Database Managed Instance, Azure SQL pool, Azure Cosmos DB, Azure Cognitive Search, Azure Data Explorer, Azure Data Share, Amazon S3

Before: `https://myaccount .file. core.win dows. net/myshare/folderA/folderB/`

After:	`https://myaccount.file.core.windows.net/myshare/folderA/folderB/`

### Remove square brackets 
Applies to: Azure SQL Database, Azure SQL Database Managed Instance, Azure SQL pool

Before: `mssql://foo.database.windows.net/[bar]/dbo/[foo bar]`

After: 	`mssql://foo.database.windows.net/bar/dbo/foo%20bar`

> [!NOTE]
> Spaces between two square brackets will be encoded

### Lowercase scheme
Applies to: Azure Blob, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure SQL Database, Azure SQL Database Managed Instance, Azure SQL pool, Azure Cosmos DB, Azure Cognitive Search, Azure Data Explorer, Amazon S3

Before: `HTTPS://myaccount.file.core.windows.net/myshare/folderA/folderB/`

After: 	`https://myaccount.file.core.windows.net/myshare/folderA/folderB/`

### Lowercase hostname
Applies to: Azure Blob, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure SQL Database, Azure SQL Database Managed Instance, Azure SQL pool, Azure Cosmos DB, Azure Cognitive Search, Azure Data Explorer, Amazon S3

Before: `https://myAccount.file.Core.Windows.net/myshare/folderA/folderB/`

After: 	`https://myaccount.file.core.windows.net/myshare/folderA/folderB/`

### Lowercase file extension
Applies to: Azure Blob, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Amazon S3

Before: `https://myAccount.file.core.windows.net/myshare/folderA/data.TXT`

After: 	`https://myaccount.file.core.windows.net/myshare/folderA/data.txt`

### Remove duplicate slash
Applies to: Azure Blob, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure Data Factory, Azure SQL Database, Azure SQL Database Managed Instance, Azure SQL pool, Azure Cosmos DB, Azure Cognitive Search, Azure Data Explorer, Azure Data Share, Amazon S3

Before: `https://myAccount.file.core.windows.net//myshare/folderA////folderB/`

After: 	`https://myaccount.file.core.windows.net/myshare/folderA/folderB/`

### Lowercase ADF sections
Applies to: Azure Data Factory

Before: `/subscriptions/01234567-abcd-9876-0000-ba9876543210/resourceGroups/fooBar/providers/Microsoft.DataFactory/factories/fooFactory/pipelines/barPipeline/activities/barFoo`

After: 	`/subscriptions/01234567-abcd-9876-0000-ba9876543210/resourceGroups/foobar/providers/microsoft.datafactory/factories/foofactory/pipelines/barpipeline/activities/barfoo`

### Convert to ADL scheme
Applies to: Azure Data Lake Storage Gen1

Before: `https://mystore.azuredatalakestore.net/folderA/folderB/abc.csv`

After: 	`adl://mystore.azuredatalakestore.net/folderA/folderB/abc.csv`

## Next steps

[Scan in an Azure Blob Storage](register-scan-azure-blob-storage-source.md) account into the Azure Purview data map. 
