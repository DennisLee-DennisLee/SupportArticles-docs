---
title: Steps to disable the TLS 1.0 and 1.1 on the MBAM Servers and force the use of TLS 1.2
description: Describes the steps to disable the TLS 1.0 and 1.1 on the MBAM Servers and force the use of TLS 1.2.
ms.date: 09/21/2020
author: Deland-Han
ms.author: delhan
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika
ms.prod-support-area-path: Transport Layer Security (TLS)
ms.technology: ActiveDirectory
---
# Steps to disable the TLS 1.0 and 1.1 on the MBAM Servers and force the use of TLS 1.2

This article describes the steps to disable the TLS 1.0 and 1.1 on the MBAM Servers and force the use of TLS 1.2.

_Original product version:_ &nbsp; Windows 10 – all editions, Windows Server 2012 R2  
_Original KB number:_ &nbsp; 4558055

## Symptoms

Microsoft is planning to disable older TLS protocols, in preparation for disabling **Transport Layer Security (TLS) 1.0** and **1.1** by default. See [Plan for change: TLS 1.0 and TLS 1.1 soon to be disabled by default](https://blogs.windows.com/msedgedev/2020/03/31/tls-1-0-tls-1-1-schedule-update-edge-ie11/). 

For enterprise customers, this may require disabling TLS 1.0 and 1.1 in their environment for **Microsoft Bitlocker Administration and Monitoring (MBAM) Infrastructure**. 

## Resolution

The following are the steps to disable the TLS 1.0 and 1.1 on the MBAM Servers, and force the use of TLS 1.2.

1. Download and install the latest available version of Microsoft .NET Framework on all MBAM servers that are Web Servers running IIS roles, SQL Servers running SQL Server database Engine, and SQL Server Reporting Services.
    Refer to: [Microsoft .NET Framework 4.8 offline installer for Windows](https://support.microsoft.com/help/4503548/microsoft-net-framework-4-8-offline-installer-for-windows) 
2. Execute the PowerShell Scripts below. They're used to disable TLS 1.0 and 1.1 and force the use only TLS 1.2.
3. Reboot the servers, then test the MBAM web applications and confirm that the MBAM clients can communicate with the server to back up recovery information.
    
\<Tighten_DotNet.PS1>

```powershell
#Tighten up the .NET Framework
$NetRegistryPath = "HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319"
New-ItemProperty -Path $NetRegistryPath -Name "SchUseStrongCrypto" -Value "1" -PropertyType DWORD -Force | Out-Null
$NetRegistryPath = "HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319"
New-ItemProperty -Path $NetRegistryPath -Name "SchUseStrongCrypto" -Value "1" -PropertyType DWORD -Force | Out-Null
```

\<Force_TLS1.2.PS1>

```powershell
$ProtocolList       = @("SSL 2.0","SSL 3.0","TLS 1.0", "TLS 1.1", "TLS 1.2")
$ProtocolSubKeyList = @("Client", "Server")
$DisabledByDefault = "DisabledByDefault"
$Enabled = "Enabled"
$registryPath = "HKLM:\\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\"

foreach($Protocol in $ProtocolList)
{
    Write-Host " In 1st For loop"
foreach($key in $ProtocolSubKeyList)
{
$currentRegPath = $registryPath + $Protocol + "\" + $key
Write-Host " Current Registry Path $currentRegPath"

if(!(Test-Path $currentRegPath))
{
    Write-Host "creating the registry"
New-Item -Path $currentRegPath -Force | out-Null

}
if($Protocol -eq "TLS 1.2")
{
    Write-Host "Working for TLS 1.2"
New-ItemProperty -Path $currentRegPath -Name $DisabledByDefault -Value "0" -PropertyType DWORD -Force | Out-Null
New-ItemProperty -Path $currentRegPath -Name $Enabled -Value "1" -PropertyType DWORD -Force | Out-Null

}
else
{
    Write-Host "Working for other protocol"
New-ItemProperty -Path $currentRegPath -Name $DisabledByDefault -Value "1" -PropertyType DWORD -Force | Out-Null
New-ItemProperty -Path $currentRegPath -Name $Enabled -Value "0" -PropertyType DWORD -Force | Out-Null
}
}
}
Exit 0
```

## More information

[Transport Layer Security (TLS) registry settings](/windows-server/security/tls/tls-registry-settings)