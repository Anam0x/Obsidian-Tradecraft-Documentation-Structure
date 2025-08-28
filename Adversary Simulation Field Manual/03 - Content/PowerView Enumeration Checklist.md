---
aliases:
  - PowerView Cheat Sheet
  - PowerView Checklist
tags:
  - ✅Playbook
primary categories:
  - "[[Penetration Test]]"
  - "[[Red Team]]"
secondary categories:
  - "[[Active Directory]]"
  - "[[Post-Exploitation]]"
  - "[[Domain Enumeration]]"
type: Playbook
---
# [[PowerView Enumeration Checklist]]

***
## Overview

This playbook provides a structured approach to enumerating _Active Directory_ (*AD*) environments using `PowerView.ps1`, a flexible PowerShell-based utility for domain enumeration. It is intended for post-compromise scenarios where the operator has access to a domain-joined host and aims to map the domain’s logical structure, user accounts, group memberships, trust relationships, and policy configurations.

```ad-info
These commands were written to be executed from the *Mythic* command and control (C2) framework.  To execute commands whose output is being truncated, it's recommended to submit `powerpick $FormatEnumerationLimit = -1; <powerview-command>` to the *Apollo* command interface.
```

## Steps

### Setup

* [ ] Set `$FormatEnumerationLimit` to display all modules without truncation

```powershell
$FormatEnumerationLimit = -1
```

* [ ] Create a PowerShell credential object and pass to PowerView function(s)

```powershell
$password = ConvertTo-SecureString "password" -AsPlainText -Force

$credential = New-Object System.Management.Automation.PSCredential ("domain\user", $password)

Get-DomainUser -Credential $credential
```

### Domains and Forests

* [ ] What do we know about the current/specified forest(s)?
	* [ ] What domain(s) is/are members of the forest (`Domains`)?
	* [ ] Is the `ForestModeLevel` attribute set to a value less than `7` (represents the minimum Windows Server version supported for domain controllers in the forest and available AD features)?
	* [ ] What is the root domain of the forest (`RootDomain`)?
* [ ] What do we know about the current/specified domain(s)?
	* [ ] Which domain is the parent of the current/specified domain(s) (`Parent`)?
	* [ ] Which domain(s) is/are the children of the current/specified domain(s)?
	* [ ] Is the `DomainModeLevel` attribute set to a value less than `7` (represents the minimum Windows Server version supported for domain controllers in the forest and available AD features)?
	* [ ] What is the root domain of the domain (`RootDomain`)?

```powershell
Get-Forest -Forest "forest.local" | Select Name,Domains,GlobalCatalogs,ForestModeLevel,RootDomain | fl

Get-Domain -Domain "domain.forest.local" | Select Name,Forest,DomainControllers,Children,DomainModeLevel,Parent | fl
```

* [ ] What are the observable domain trusts in the current/specified forest(s)?
	* [ ] What is/are the domain name(s) (`Name`)? 
	* [ ] What is the DC for each domain (`DomainControllers`)?
* [ ] What is the DC of the current/specified domain(s)?
	* [ ] What is the operating system (OS) of each DC (`OSVersion`)?
	* [ ] What is each DC's fully qualified domain name (FQDN) and IP address (`Name`, `IPAddress`)?

```powershell
Get-ForestDomain -Forest "forest.local" | Select Forest,Name,DomainControllers,Children,DomainModeLevel,Parent | fl

Get-DomainController -Domain "domain.forest.local" | Select Forest,Name,DomainController,Domain,OSVersion,Roles,IPAddress | fl
```

* [ ] What are the forest trusts for the current/specified forest(s)?
	* [ ] What forest(s) is/are trusted (`SourceName`, `TargetName`, `TrustType`)? 
	* [ ] What is the direction of the forest trust(s) (`TrustDirection`)?
		* [ ] `Bidirectional` (two-way trust)
		* [ ] `InBound` (`SourceName` forest trusted by `TargetName` forest)
		* [ ] `OutBound` (`SourceName` forest trusts `TargetName` forest)
	* [ ] What are the TopLevelNames (TLNs) associated with domains within a forest (`TopLevelNames`)?
	* [ ] Are any TLNs excluded from the forest trust (`ExcludedTopLevelNames`)?
	* [ ] What are the domains associated with the trusted forest (`TrustedDomainInformation`)?
* [ ] What are the domain trusts for the current/specified domain(s)?
	* [ ] What domain(s) is/are trusted (`SourceName`, `TargetName`, `TrustType`)?
	* [ ] What is the direction of the domain trust(s) (`TrustDirection`)?
		* [ ] `Bidirectional` (two-way trust)
		* [ ] `InBound` (`SourceName` domain trusted by `TargetName` domain)
		* [ ] `OutBound` (`SourceName` domain trusts `TargetName` domain)
	* [ ] What values are included in the `TrustAttributes` attribute?
		* [ ] `WITHIN_FOREST` (both domains in the same forest)
		* [ ] `FILTER_SIDS` (security identifier (SID) filtering enabled; golden/diamond ticket forgery mitigated)
		* [ ] `FOREST_TRANSITIVE` (trust between two root domains of forests)
		* [ ] `TREAT_AS_EXTERNAL` (SID history enabled; forest trust is treated as an external trust but with the transitivity of normal forest trusts)
* [ ] Recursively map all domain trusts for the current domain

```powershell
Get-ForestTrust -Forest "forest.local" | Select SourceName,TargetName,TrustType,TrustDirection,TopLevelNames,ExcludedTopLevelNames,TrustedDomainInformation | fl

Get-DomainTrust -Domain "domain.forest.local" | Select SourceName,TargetName,TrustType,TrustDirection,TrustAttributes | fl

Get-DomainTrustMapping | Select SourceName,TargetName,TrustType,TrustDirection,TrustAttributes | fl
```

* [ ] What is the global catalog for the current/specified forest(s)?

```powershell
Get-ForestGlobalCatalog -Forest "forest.local" | Select Forest,OSVersion,Domain,IPAddress,Name | fl
```

### Organizational Units (OUs)

* [ ] What computer objects are members of the default OUs in the current/specified domain(s)?
	* [ ] `Domain Controllers`
* [ ] What are the non-default OUs in the domain?
	* [ ] Does the name imply a special function or service (e.g., MS SQL servers, file servers, workstations)?

```powershell
Get-DomainOU -Properties Name,DistinguishedName | Where-Object {$_.Name -eq "Domain Controllers"} | fl

Get-DomainOU -Properties Name,DistinguishedName | Where-Object {$_.Name -ne "Domain Controllers"} | fl 
```

* [ ] What are the computer objects in a given OU?

```powershell
Get-DomainComputer -Properties DnsHostName,OperatingSystem,OperatingSystemVersion,LastLogonTimestamp,UserAccountControl -SearchBase "ldap://OU=Domain Controllers,DC=forest,DC=local" | fl
```

### Group Policy Objects (GPOs)

* [ ] What are the password policy settings of the `Default Domain Policy` GPO of the current/specified domain?
	* [ ] What is the minimum password age (`MinimumPasswordAge`)? Maximum (`MaximumPasswordAge`)?
	* [ ] What is the minimum password length (`MinimumPasswordLength`)?
	* [ ] Is the "password complexity requirements" policy setting enabled and set to `1` (`PasswordComplexity`)?
	* [ ] What is the domain lockout policy (`LockoutBadCount`)?
	* [ ] How many previous passwords are stored in AD (`PasswordHistorySize`)?
	* [ ] Should passwords be stored using reversible encryption (`ClearTextPassword`)?
* [ ] What are the Kerberos ticket policy settings of the `Default Domain Policy` GPO of the current/specified domain?
	* [ ] What is the maximum lifetime for a ticket granting ticket (TGT) (`MaxTicketAge`)?
	* [ ] What is the maximum lifetime for a TGT renewal (`MaxRenewAge`)?
	* [ ] What is the maximum lifetime for a ticket granting service (TGS) (`MaxServiceAge`)?
	* [ ] What is the maximum tolerance for computer clock synchronization (`MaxClockSkew`)?
	* [ ] Does the Kerberos Key Distribution Center (KDC) validate each TGS request against the requesting account's access rights (`TicketValidateClient`)?  

```powershell
(Get-DomainPolicy -Domain domain.forest.local)."SystemAccess" | Select MinimumPasswordAge,MaximumPasswordAge,MinimumPasswordLength,PasswordComplexity,PasswordHistorySize,LockoutBadCount,ClearTextPassword | fl

(Get-DomainPolicy -Domain domain.forest.local)."KerberosPolicy" | Select MaxTicketAge,MaxRenewAge,MaxServiceAge,MaxClockSkew,TicketValidateClient | fl
```

* What are the settings of the `Default Domain Controllers Policy` GPO?
	* What are the user privilege assignment rights on the domain controller?

```powershell
Get-DomainPolicy -Policy DC | Select -Expand PrivilegeRights | fl
```

* [ ] What are the non-default GPOs in the domain?
	* [ ] Does the name imply a special function or service (e.g., AV/EDR products, AppLocker, LAPS, logon scripts or scheduled tasks) (`displayname`)?

```powershell
Get-DomainGPO -Properties DisplayName,DistinguishedName | Where-Object {$_.DisplayName -ne "Default Domain Policy" -and $_.DisplayName -ne "Default Domain Controllers Policy"} | fl
```

* [ ] Do any GPOs modify local group membership via Restricted Groups or Group Policy Preferences?
	* [ ] What is the GPO (`GPODisplayName`)?
	* [ ] How was group delegation configured (`GPOType`)?
	* [ ] What is/are the affected domain group(s) (`GroupName`)?
* [ ] What are the computer objects in the domain where a specific principal is a member of a local group through GPO correlation?
	* [ ] What is the GPO (`ObjectName`, `ObjectDN`, `ObjectSID`, `GPODisplayName`)?
	* [ ] How was group delegation configured (`GPOType`)?
	* [ ] What is/are the affected computer object(s)/OU(s) (`ContainerName`, `ComputerName`)?
* [ ] What principal(s) is/are a member of a local group through GPO correlation on a specified computer object in the domain?
	* [ ] What is the GPO (`ObjectName`, `ObjectDN`, `ObjectSID`, `GPODisplayName`)?
	* [ ] How was group delegation configured (`GPOType`)?
	* [ ] What is/are the affected computer object(s) (`ComputerName`)?

```powershell
Get-DomainGPOLocalGroup | Select GPODisplayName,GPOType,GroupName,GroupSID,GroupMembers | fl

Get-DomainGPOUserLocalGroupMapping | Select ObjectName,ObjectDN,ObjectSID,GPODisplayName,GPOType,ContainerName,ComputerName | fl

powerpick Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity host.domain.forest.local | Select ObjectName,ObjectDN,ObjectSID,GPODisplayName,GPOType | fl
```

* [ ] Do any principal(s) have some kind of modification/control rights over a GPO?
* [ ] Do any principal(s) have the ability to create new GPOs?
	* [ ] If any principal(s) can create new GPOs, can other principal(s) link a GPO to an OU?

```powershell
Get-DomainObjectAcl -LDAPFilter '(objectCategory=groupPolicyContainer)' -ResolveGUIDs | ? { ($_.ActiveDirectoryRights -match 'WriteProperty|GenericAll|GenericWrite|WriteDacl|WriteOwner')} | fl

Get-DomainObjectAcl -LDAPFilter '(CN=Policies,CN=System,DC=forest,DC=local)' -ResolveGUIDs | ? { ($_.ActiveDirectoryRights -match 'CreateChild')} | fl
Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ObjectAceType -eq "GP-Link" -and $_.ActiveDirectoryRights -match "WriteProperty" } | fl
```

### Computers

* [ ] What computers are in the current/specified domain?
	* [ ] What is the FQDN of each computer (`DnsHostName`)?
	* [ ] What is the OS and OS version of each computer (`OperatingSystem`, `OperatingSystemVersion`)?
	* [ ] What is the last logon timestamp of each computer (`LastLogonTimestamp`)?
	* [ ] Does the `UserAccountControl` attribute include unusual or risky flags?
		* [ ] `PASSWD_NOTREQD` (empty passwords permitted)
		* [ ] `DONT_EXPIRE_PASSWD` (password never expires)
		* [ ] `TRUSTED_FOR_DELEGATION` (principal configured for unconstrained delegation)
		* [ ] `NOT_DELEGATED` (principal protected from Kerberos delegation)
		* [ ] `USE_DES_KEY_ONLY` (Data Encryption Standard (DES)-only keys)
		* [ ] `DONT_REQ_PREAUTH` (ASREP roastable)
* [ ] What GPOs are applied to the specified computer object?

```powershell
Get-DomainComputer -Properties Name,DnsHostName,DistinguishedName,ObjectSID,SamAccountName,OperatingSystem,OperatingSystemVersion,LastLogonTimestamp,UserAccountControl | fl

Get-DomainGPO -ComputerIdentity "host.domain.forest.com" -Properties DisplayName,DistinguishedName | fl
```

* [ ] Is the `UserAccountControl` attribute set to `TRUSTED_FOR_DELEGATION` (principal configured for unconstrained delegation)?
	* [ ] If a computer object is configured for unconstrained delegation, are there any domain users that aren't marked as sensitive (principal permitted for delegation)?
	* [ ] If a computer object is configured for unconstrained delegation, are there any domain users currently logged in to that server (principal TGT cached on that server)?
 * [ ] Is the `msds-AllowedToDelegateTo` attribute not null (principal configured for constrained delegation)?
	* [ ] Which SPN(s) is/are the computer(s) trusted to delegate to (`msds-AllowedToDelegateTo`)?
	* [ ] Do the SPN(s) specify port numbers (unable to spoof TGS service via S4U)?
	* [ ] If a computer object is configured for constrained delegation, are there any domain users that aren't marked as sensitive (principal permitted for delegation)?
 * [ ] Do any principals have AD rights facilitating write access to any computer object's `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute (current/specified principal can conduct resource-based constrained delegation (RBCD) attack against the computer object)?
	* [ ] If a computer object is configured for RBCD delegation, are there any domain users that aren't marked as sensitive (principal permitted for delegation)?

```powershell
Get-DomainComputer -Unconstrained -Properties Name,DnsHostName,DistinguishedName,ObjectSID,UserAccountControl | fl
Get-DomainUser -AllowDelegation -AdminCount -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSID,SamAccountName,AdminCount,MemberOf,UserAccountControl | fl
Find-DomainUserLocation -ComputerUnconstrained -ShowAll | fl
Find-DomainUserLocation -ComputerUnconstrained -UserAdminCount -UserAllowDelegation | fl

Get-DomainComputer -TrustedToAuth -Properties Name,DnsHostName,DistinguishedName,ObjectSID,UserAccountControl,msds-AllowedToDelegateTo | fl
Get-DomainUser -AllowDelegation -AdminCount -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSID,SamAccountName,AdminCount,MemberOf,UserAccountControl | fl

powerpick Get-DomainComputer | Get-ObjectAcl -ResolveGUIDs | ForEach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.Value) -Force; $_} | Where-Object { $_.ActiveDirectoryRights -match "WriteProperty|GenericWrite|GenericAll|WriteDacl" } | Select AceType,ObjectDN,ActiveDirectoryRights,ObjectSID,SecurityIdentifier,AccessMask,AceQualifier,Identity | fl
Get-DomainUser -AllowDelegation -AdminCount -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSID,SamAccountName,AdminCount,MemberOf,UserAccountControl | fl
```

### Users

* [ ] Are the expected default users present in current/specified domain?
	* [ ] Is the account stale (`lastlogontimestamp`)?
	* [ ] Is the `Description` attribute populated? Does it reveal credentials or hints?
	* [ ] Is the user protected (`AdminCount`)?
	* [ ] What groups is the user a direct member of (`MemberOf`)?
	* [ ] Does the `UserAccountControl` attribute include unusual or risky flags?
		* [ ] `ACCOUNTDISABLE` (principal enabled/disabled)
		* [ ] `PASSWD_NOTREQD` (empty passwords permitted)
		* [ ] `DONT_EXPIRE_PASSWD` (password never expires)
		* [ ] `TRUSTED_TO_AUTH_FOR_DELEGATION` (principal configured for constrained delegation)
		* [ ] `NOT_DELEGATED` (principal protected from Kerberos delegation)
		* [ ] `USE_DES_KEY_ONLY` (principal restricted to only using DES encryption type for keys)
		* [ ] `DONT_REQ_PREAUTH` (principal is vulnerable to ASREP roasting)

```powershell
Get-DomainUser -Identity Administrator,Guest,KRBTGT -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSid,SamAccountName,AdminCount,MemberOf,UserAccountControl | fl
```

* [ ] What are the non-default users in the domain?
	* [ ] Does the name imply a special function (e.g., service account, employee, administrative)?
	* [ ] Is the account stale (`LastLogonTimestamp`)?
	* [ ] Is the `Description` attribute populated? Does it reveal credentials or hints?
	* [ ] Is the user protected (`AdminCount`)?
	* [ ] What groups is the user a direct member of (`MemberOf`)?
	* [ ] Does the `UserAccountControl` attribute include unusual or risky flags?
		* [ ] `ACCOUNTDISABLE` (principal disabled)
		* [ ] `PASSWD_NOTREQD` (empty passwords permitted)
		* [ ] `DONT_EXPIRE_PASSWD` (password never expires)
		* [ ] `TRUSTED_TO_AUTH_FOR_DELEGATION` (principal configured for constrained delegation)
		* [ ] `NOT_DELEGATED` (principal protected from Kerberos delegation)
		* [ ] `USE_DES_KEY_ONLY` (DES-only keys)
		* [ ] `DONT_REQ_PREAUTH` (ASREP roastable)

```powershell
Get-DomainUser -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSid,SamAccountName,AdminCount,MemberOf,UserAccountControl | Where-Object { $_.name -ne "Administrator" -and $_.name -ne "Guest" -and $_.name -ne "KRBTGT" } | fl
```

* [ ] Which users changed their password less than one year ago?
* [ ] Which users have the `UserPassword` attribute set?
* [ ] Which users have the `SidHistory` attribute set (SIDs from previous domain migrations/merges recorded for this user)?

```powershell
$Date = (Get-Date).AddYears(-1).ToFileTime()
Get-DomainUser -LDAPFilter "(pwdlastset<=$Date)" -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSid,SamAccountName,AdminCount,MemberOf,UserAccountControl,PwdLastSet | fl

Get-DomainUser -LDAPFilter '(userPassword=*)' -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSid,SamAccountName,AdminCount,MemberOf,UserAccountControl,UserPassword | ForEach-Object {Add-Member -InputObject $_ NoteProperty 'Password' "$([System.Text.Encoding]::ASCII.GetString($_.userPassword))" -PassThru} | fl

Get-DomainUser -LDAPFilter '(sIDHistory=*)' -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSid,SamAccountName,AdminCount,MemberOf,UserAccountControl | fl
```

* [ ] Which users are enabled?
* [ ] Which users are disabled?

```powershell
Get-DomainUser -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSid,SamAccountName,AdminCount,MemberOf,UserAccountControl -LDAPFilter "(!userAccountControl:1.2.840.113556.1.4.803:=2)" | fl
Get-DomainUser -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSid,SamAccountName,AdminCount,MemberOf,UserAccountControl -UACFilter NOT_ACCOUNTDISABLE | fl

Get-DomainUser -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSid,SamAccountName,AdminCount,MemberOf,UserAccountControl -LDAPFilter "(userAccountControl:1.2.840.113556.1.4.803:=2)" | fl
Get-DomainUser -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSid,SamAccountName,AdminCount,MemberOf,UserAccountControl -UACFilter ACCOUNTDISABLE | fl
```

* [ ] What nested groups is the current/specified principal(s) effectively a member of (recursing 'up' using `TokenGroups`)? 

```powershell
Get-DomainGroup -MemberIdentity "John Smith" -Properties SamAccountName,ObjectSid,Name,Description,DistinguishedName,Member | fl
```

 * [ ] Is the `msds-AllowedToDelegateTo` attribute not null (principal configured for constrained delegation)?
	* [ ] Which SPN(s) is/are the user(s) trusted to delegate to (`msds-AllowedToDelegateTo`)?
	* [ ] Do the SPN(s) specify port numbers?

```powershell
Get-DomainUser -TrustedToAuth -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSid,SamAccountName,AdminCount,MemberOf,UserAccountControl,msds-AllowedToDelegateTo | fl
```

* [ ] Is the `ServicePrincipalName` attribute non-null (principal vulnerable to kerberoasting)?
	* [ ] What is the Service Principal Name (SPN) associated with the service account (`ServicePrincipalName`)?
* [ ] Does the `UserAccountControl` attribute include `DONT_REQ_PREAUTH` (principal vulnerable to ASREP roasting)?

```powershell
Get-DomainUser -SPN -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSID,SamAccountName,AdminCount,MemberOf,UserAccountControl,ServicePrincipalName | fl

Get-DomainUser -KerberosPreauthNotRequired -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSID,SamAccountName,AdminCount,MemberOf,UserAccountControl | fl
Get-DomainUser -UACFilter DONT_REQ_PREAUTH -Properties LastLogonTimestamp,Description,DistinguishedName,Name,ObjectSID,SamAccountName,AdminCount,MemberOf,UserAccountControl | fl
```

### Groups

* [ ] Are the expected default groups in the domain?
* [ ] Who are the effective members of the "high-value" domain groups?
	* [ ] `Domain Admins`
	* [ ] `Enterprise Admins`
	* [ ] `Schema Admins`
	* [ ] `Account Operators`
	* [ ] `Backup Operators`
	* [ ] `Print Operators`
	* [ ] `Server Operators`
	* [ ] `DnsAdmins`
	* [ ] `Group Policy Creator Owners`
	* [ ] `Protected Users`
	* [ ] `Read-only Domain Controllers`

```powershell
"Administrators","Users","Guests","Domain Admins","Domain Users","Domain Guests","Account Operators","Server Operators","Print Operators","Backup Operators","Replicator","Incoming Forest Trust Builders","Pre-Windows 2000 Compatible Access","Enterprise Admins","Schema Admins","Cert Publishers","DnsAdmins","DnsUpdateProxy","Group Policy Creator Owners","Read-only Domain Controllers","Denied RODC Password Replication Group","Allowed RODC Password Replication Group","Cloneable Domain Controllers","Protected Users","Key Admins","Enterprise Key Admins" | ForEach-Object { Get-DomainGroup -Properties description,name,distinguishedname -Identity $_ } | fl

"Domain Admins","Enterprise Admins","Schema Admins","Account Operators","Backup Operators","Print Operators","Server Operators","DnsAdmins","Group Policy Creator Owners","Protected Users","Read-only Domain Controllers" | ForEach-Object { Get-DomainGroupMember -Identity $_ -Recurse } | Select GroupDomain,GroupName,GroupDistinguishedName,MemberDomain,MemberName,MemberDistinguishedName,MemberObjectClass,MemberSID | fl
```

* [ ] What are the local groups of the current/specified host(s)?
* [ ] Who are the members of the "high value" local groups of the current host?
	* [ ] `Administrators`
	* [ ] `Remote Desktop Users`
	* [ ] `Remote Management Users`
	* [ ] Any non-default local group(s)
* [ ] Who are the members of the "high value" local groups of the specified host(s)?
	* [ ] `Administrators`
	* [ ] `Remote Desktop Users`
	* [ ] `Remote Management Users`
	* [ ] Any non-default local group(s)

```powershell
Get-NetLocalGroup -Method API | Select ComputerName,GroupName,Comment | fl
Get-NetLocalGroup -Method WinNT | Select ComputerName,GroupName,Comment | fl
Get-NetLocalGroup -Method API -ComputerName host.domain.forest.local | Select ComputerName,GroupName,Comment | fl
Get-NetLocalGroup -Method WinNT -ComputerName host.domain.forest.local | Select ComputerName,GroupName,Comment | fl

"Administrators","Remote Desktop Users","Remote Management Users" | ForEach-Object { Get-NetLocalGroupMember -Method API -GroupName $_ } | Select ComputerName,GroupName,MemberName,SID,IsGroup,IsDomain | fl
"Administrators","Remote Desktop Users","Remote Management Users" | ForEach-Object { Get-NetLocalGroupMember -Method WinNT -GroupName $_ } | Select ComputerName,GroupName,MemberName,SID,IsGroup,IsDomain | fl

"Administrators","Remote Desktop Users","Remote Management Users" | ForEach-Object { Get-NetLocalGroupMember -Method API -GroupName $_ -ComputerName host.domain.forest.local } | Select ComputerName,GroupName,MemberName,SID,IsGroup,IsDomain | fl
"Administrators","Remote Desktop Users","Remote Management Users" | ForEach-Object { Get-NetLocalGroupMember -Method WinNT -GroupName $_ -ComputerName host.domain.forest.local } | Select ComputerName,GroupName,MemberName,SID,IsGroup,IsDomain | fl
```

* [ ] What are the non-default groups in the domain?
	* [ ] Does the name imply a special function (e.g., administrative access to a set of hosts, service access, geography)?
	* [ ] Is the group protected (`AdminCount`)?

```powershell
Get-DomainGroup -Properties SamAccountName,ObjectSID,CN,Name,Description,DistinguishedName,AdminCount | Where-Object { $_.SamAccountName -notin @("Administrators","Users","Guests","Domain Admins","Domain Users","Domain Guests","Account Operators","Server Operators","Print Operators","Backup Operators","Replicator","Incoming Forest Trust Builders","Pre-Windows 2000 Compatible Access","Enterprise Admins","Schema Admins","Cert Publishers","DnsAdmins","DnsUpdateProxy","Group Policy Creator Owners","Read-only Domain Controllers","Denied RODC Password Replication Group","Allowed RODC Password Replication Group","Cloneable Domain Controllers","Protected Users","Key Admins","Enterprise Key Admins") } | fl
```

* [ ] What nested groups is the current/specified group(s) effectively a member of (recursing 'up' using `tokenGroups`)? 
* [ ] Who are the effective members of the current/specified group(s)?
* [ ] Do any groups of the current/specified domain(s) contain members from foreign domains?

```powershell
Get-DomainGroup -MemberIdentity "Domain Admins" -Properties SamAccountName,ObjectSID,Name,Description,DistinguishedName,Member | fl

Get-DomainGroupMember -Identity "Domain Admins" -Recurse | Select GroupDomain,GroupName,GroupDistinguishedName,MemberDomain,MemberName,MemberDistinguishedName,MemberObjectClass,MemberSID | fl

Get-DomainForeignGroupMember -Domain domain.forest.local -Properties GroupDomain,GroupName,GroupDistinguishedName,MemberDomain,MemberName,MemberDistinguishedName | fl
```

* [ ] Are there any hosts in the current/specified domain(s) where the current user has local administrative access?

```powershell
Find-LocalAdminAccess | fl
Find-LocalAdminAccess -ComputerDomain domain.forest.local | fl
```

### Session Enumeration

* [ ] Are there any active SMB sessions on the current/specified host(s)?
* [ ] Are there any active interactive logon sessions on the current/specified host(s)?
* [ ] Are there any active logon sessions on the current/specified host(s) based on enumeration of remote registry keys?

```powershell
Get-NetSession | fl
Get-NetSession -ComputerName host.domain.forest.local | fl

Get-NetLoggedon | fl
Get-NetLoggedon -ComputerName host.domain.forest.local | fl

Get-RegLoggedOn | fl
Get-RegLoggedOn -ComputerName host.domain.forest.local | fl
```

* [ ] Is/are there any host(s) where the specified user is logged on?
* [ ] Is/are there any host(s) where a member of the specified group is logged on?

```powershell
Find-DomainUserLocation -UserIdentity "John Smith" | fl

Find-DomainUserLocation -UserGroupIdentity "Domain Admins" | fl
```
### Data Exfiltration

* [ ] Are there domain shares?
* [ ] Are there accessible domain shares?
* [ ] Are there accessible domain shares that match a given substring(s) filter?
	* [ ] Microsoft Word (`*.doc*`)
	* [ ] Microsoft Excel (`*.xls*`)
	* [ ] Microsoft PowerPoint (`*.ppt*`)
	* [ ] Comma Separated Values (`*.csv`)
	* [ ] Portable Document Format (`*.pdf`)
	* [ ] Text (`*.txt`)
	* [ ] Images (`*.png`, `*.jp*`, `*.gif`)

```powershell
Find-DomainShare | fl

Find-DomainShare -CheckShareAccess | fl

Find-InterestingDomainShareFile -Include *.doc*, *.xls*, *.ppt*, *.csv, *.pdf, *.txt, *.png, *.jp*, *.gif | fl
```

***

## Resources

| Hyperlink                                                                                                                                | Info                                                         |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| [HarmJ0y CheatSheets - PowerView](https://github.com/HarmJ0y/CheatSheets/blob/master/PowerView.pdf)                                      | PowerView 3.0 cheat sheet by PowerSploit co-creator          |
| [Active Directory - Enumeration: PowerView CheatSheet](https://zflemingg1.gitbook.io/undergrad-tutorials/powerview/powerview-cheatsheet) | Zach Fleming's (AKA _zflemingg1_) AD enumeration cheat sheet |
| [HarmJ0y GitHub Gist - PowerView 3.0 Tricks](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)                           | PowerView 3.0 sample commands by PowerSploit co-creator      |
| [PowerSploit - Recon](https://powersploit.readthedocs.io/en/latest/Recon/)                                                               | PowerView documentation                                      |

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
