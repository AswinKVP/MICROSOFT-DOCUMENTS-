---
title: "Azure Active Directory (Azure AD) authentication for SQL Server overview"
description: Learn about Azure Active Directory authentication support for SQL Server
ms.date: "05/24/2022"
ms.prod: sql
ms.technology: security
ms.reviewer: vanto
ms.topic: conceptual
ms.custom:
- event-tier1-build-2022
author: GithubMirek
ms.author: mireks
monikerRange: ">=sql-server-ver15||>= sql-server-linux-ver16"
---

# Azure Active Directory authentication for SQL Server

[!INCLUDE [SQL Server 2022](../../../includes/applies-to-version/sqlserver2022.md)]

SQL Server 2022 introduces support for [Azure Active Directory (Azure AD) authentication](/azure/active-directory/authentication/overview-authentication).

## Overview

You can now connect to SQL Server using the following authentication methods using Azure AD identities:

- Azure Active Directory Password
- Azure Active Directory Integrated
- Azure Active Directory Universal with Multi-Factor Authentication
- Azure Active Directory access token

The current authentication modes, such as [SQL authentication and Windows authentication](/sql/relational-databases/security/choose-an-authentication-mode) remain unchanged.

As a central authentication repository used by Azure, Azure AD allows you to store objects such as users, groups, or service principals as identities. Azure AD also allows you to use those identities to authenticate with different Azure services. Azure AD authentication is supported for Azure SQL Database, Azure SQL Managed Instance, Azure Synapse Analytics, and SQL Server. For more information, see [Use Azure Active Directory authentication](/azure/azure-sql/database/authentication-aad-overview) and [Configure and manage Azure AD authentication with Azure SQL](/azure/azure-sql/database/authentication-aad-configure).

If your Windows Server Active Directory is federated with Azure AD, users can authenticate with SQL Server using their Windows credentials, either as a Windows logins or an Azure AD login. Azure AD doesn't support all AD features, such as service accounts or complex networking forest architecture that is supported for Windows Server Active Directory.

> [!IMPORTANT]
> Azure AD authentication is only supported for SQL Server running in the mixed mode authentication (allowing SQL Server and Windows authentication mode). If only Windows authentication mode is chosen, Azure AD authentication is not supported.

## Connecting SQL Server to Azure to use Azure AD

For SQL Server to communicate with Azure, both SQL Server and the Windows host it runs on must be registered with [Azure Arc](/sql/sql-server/azure-arc/overview). To do this, you'll need to install the [Azure Arc Agent](/azure/azure-arc/servers/overview) and [SQL Server extension (WindowsAgent.SqlServer)](/sql/sql-server/azure-arc/overview). This will facilitate SQL Server’s  communication with Azure.

To get started, see [Connect your SQL Server to Azure Arc](/sql/sql-server/azure-arc/connect).

## Authentication methods

The following authentication methods are available when connecting to SQL Server using Azure AD authentication.

### Azure Active Directory Password

Allows specifying the username and password to the client and driver, but this is disabled on many tenants for security reasons. When possible, we recommend avoiding this as it requires sending passwords over the network. These connections are encrypted, but it's best practice to never send them in the first place.

### Azure Active Directory Integrated

When the Windows domain is synchronized with Azure AD, and a user is logged into the Windows domain, the user's Windows credentials are used for Azure AD authentication.

### Azure Active Directory Universal with Multi-Factor Authentication

This is the standard interactive method with multi-factor authentication option for Azure AD accounts. This will work in most scenarios.

### Azure Active Directory access token

Some non-GUI clients such as [Invoke-sqlcmd](/powershell/module/sqlserver/invoke-sqlcmd) allow providing an access token. The scope or audience of the access token must be `https://database.windows.net/`.

## Remarks

- Only SQL Server 2022 on-premises with a Windows Operating System is supported for Azure AD authentication. SQL Servers on Azure Virtual Machines aren't supported for Azure AD authentication.
- To connect SQL Server to Azure Arc, the Azure AD account needs the following permissions:
  - Member of the *Azure Connected Machine Onboarding* group or *Contributor* role in the resource group.
  - Member of the *Azure Connected Machine Resource Administrator* role in the resource group.
  - Member of the *Reader* role in the resource group.

## See also

- [Tutorial: Set up Azure Active Directory authentication for SQL Server](azure-ad-authentication-sql-server-setup-tutorial.md)
- [Azure Active Directory (Azure AD) authentication](/azure/active-directory/authentication/overview-authentication)
- [Linked server for SQL Server with Azure Active Director authentication](azure-ad-authentication-sql-server-linked-server.md)

## Next steps

[Connect your SQL Server to Azure Arc](/sql/sql-server/azure-arc/connect)
