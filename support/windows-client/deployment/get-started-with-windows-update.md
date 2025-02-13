---
title: Get started with Windows Update
description: An overview of learning resources for Windows Update, including documents on architecture, log files, and common errors.
ms.date: 08/19/2022
ms.prod: windows-client
author: aczechowski
ms.author: aaroncz
manager: dcscontentpm
ms.topic: troubleshooting
ms.technology: windows-client-deployment
ms.custom: sap:servicing, csstroubleshoot
ms.reviewer: dougeby
audience: itpro
localization_priority: medium
---

# Get started with Windows Update

With the release of Windows 10, we moved the update model to the Unified Update Platform. Unified Update Platform (UUP) is a single publishing, hosting, scan and download model for all types of OS updates, client devices for all Windows-based operating systems, for everything from monthly quality updates to new feature updates.

_Applies to:_ &nbsp; Windows 10

Use the following information to get started with Windows Update:

- Understand the UUP architecture
- Understand [how Windows Update works](how-windows-update-works.md)
- Find [Windows Update log files](windows-update-log-files.md)
- Learn how to [troubleshoot Windows Update](windows-update-issues-troubleshooting.md)
- Review [common Windows Update errors](common-windows-update-errors.md) and check out the [error code reference](/windows/deployment/update/windows-update-error-reference)
- Review [other resources](additional-resources-for-windows-update.md) to help you use Windows Update
- Review [Windows IT Pro Blog](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/bg-p/Windows10Blog) section of Microsoft Blogs.

## Unified Update Platform (UUP) architecture

To understand the changes to the Windows Update architecture that UUP introduces let's start with some new key terms.

:::image type="content" source="media/get-started-with-windows-update/update-terminology.png" alt-text="Screenshot of Windows Update terminology." border="false":::

- Update UI – The user interface to initiate Windows Update check and history. Available under **Settings** > **Update & Security** > **Windows Update**.
- Update Session Orchestrator (USO) - A Windows OS component that orchestrates the sequence of downloading and installing various update types from Windows Update.  

   Update types

  - OS Feature updates
  - OS Security updates
  - Device drivers
  - Defender definition updates

   > [!NOTE]
   > Other types of updates, like Office desktop updates, are installed if the user opts into Microsoft Update.
   >
   > Store apps aren't installed by USO, today they are separate.

- Windows Update Client/ UpdateAgent - The component running on your PC. It's essentially a DLL that is downloaded to the device when an update is applicable. It surfaces the APIs needed to perform an update, including those needed to generate a list of payloads to download, as well as starts stage and commit operations. It provides a unified interface that abstracts away the underlying update technologies from the caller.  
- Windows Update Arbiter handle - Code that is included in the UpdateAgent binary. The arbiter gathers information about the device, and uses the CompDB(s) to output an action list. It's responsible for determining the final "composition state" of your device, and which payloads (like ESDs or packages) are needed to get your device up to date.
- Deployment Arbiter- A deployment manager that calls different installers. For example, CBS.

Additional components include the following items:

- CompDB – A generic term to refer to the XML describing information about target build composition, available diff packages, and conditional rules.
- Action List – The payload and additional information needed to perform an update. The action list is consumed by the UpdateAgent, as well as other installers to determine what payload to download. It's also consumed by the "Install Agent" to determine what actions need to be taken, such as installing or removing packages.  
