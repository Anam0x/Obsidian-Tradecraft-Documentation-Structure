---
aliases:
  - PowerView Cheat Sheet
tags:
  - ✅
primary categories:
  - "[[01 - Penetration Test]]"
  - "[[01 - Red Team]]"
secondary categories:
  - "[[02 - Active Directory]]"
  - "[[02 - Post-Exploitation]]"
  - "[[02 - Domain Enumeration]]"
type: Playbook
---
# [[PowerView Enumeration Checklist]]

***
## Overview

This playbook provides a structured approach to enumerating _Active Directory_ (*AD*)[^1] environments using `PowerView.ps1`[^2], a flexible PowerShell-based utility for domain enumeration. It is intended for post-compromise scenarios where the operator has access to a domain-joined host and aims to map the domain’s logical structure, user accounts, group memberships, trust relationships, and policy configurations.

> *Note*:
> These commands were written to be executed from the *Mythic*[^3] command and control (C2) framework.  To execute commands whose output is being truncated, it's recommended to submit `powerpick $FormatEnumerationLimit = -1; <powerview-command>` to the *Apollo*[^4] command interface.

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

### Trust Relationships

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
	* [ ] What is the operating system (OS) of each DC?
	* [ ] What is each DC's fully qualified domain name (FQDN)?

```powershell
Get-ForestDomain -Forest "forest.local" | Select Forest,Name,DomainControllers,Children,DomainModeLevel,Parent | fl

Get-DomainController -Domain "domain.forest.local" | Select Forest,Name,DomainController,Domain,OSVersion,Roles,IPAddress | fl
```

* [ ] What are the forest trusts for the current/specified forest(s)?
	* [ ] What forest(s) is/are trusted (`SourceName`, `TargetName`, `TrustType`)? 
	* [ ] What is the direction of the forest trust(s) (`TrustDirection`)?
		* [ ] `Bidirectional` (i.e., is it a two-way trust?)
		* [ ] `InBound` (i.e., is the `SourceName` forest trusted by the `TargetName` forest?)
		* [ ] `OutBound` (i.e., does the `SourceName` forest trust the `TargetName` forest?)
	* [ ] What are the TopLevelNames (TLNs) associated with domains within a forest (`TopLevelNames`)?
	* [ ] Are any TLNs excluded from the forest trust (`ExcludedTopLevelNames`)?
	* [ ] What are the domains associated with the trusted forest (`TrustedDomainInformation`)?
* [ ] What are the domain trusts for the current/specified domain(s)?
	* [ ] What domain(s) is/are trusted (`SourceName`, `TargetName`, `TrustType`)?
	* [ ] What is the direction of the domain trust(s) (`TrustDirection`)?
		* [ ] `Bidirectional` (i.e., is it a two-way trust?)
		* [ ] `InBound` (i.e., is the `SourceName` domain trusted by the `TargetName` domain?)
		* [ ] `OutBound` (i.e., does the `SourceName` domain trust the `TargetName` domain?)
	* [ ] What values are included in the `TrustAttributes` attribute?
		* [ ] `WITHIN_FOREST` (i.e., are both domains in the same forest?)
		* [ ] `FILTER_SIDS` (i.e., is SID filtering enabled so that golden/diamond ticket forgery is mitigated?)
		* [ ] `FOREST_TRANSITIVE` (i.e., is the trust between two root domains of forests?)
		* [ ] `TREAT_AS_EXTERNAL` (i.e., is SID history enabled so that the forest trust is treated as an external trust but with the transitivity of normal forest trusts?)
* [ ] Recursively map all domain trusts for the current domain

```powershell
Get-ForestTrust -Forest "forest.local" | Select SourceName,TargetName,TrustType,TrustDirection,TopLevelNames,ExcludedTopLevelNames,TrustedDomainInformation

Get-DomainTrust -Domain "domain.forest.local" | Select SourceName,TargetName,TrustType,TrustDirection,TrustAttributes

Get-DomainTrustMapping | Select SourceName,TargetName,TrustType,TrustDirection,TrustAttributes
```

### Organizational Units (OUs)

* [ ] What computer objects are members of the default OUs in the current/specified domain(s)?
	* [ ] `Domain Controllers`
* [ ] What are the non-default OUs in the domain?
	* [ ] Does the name imply a special function or service (e.g., MS SQL servers, file servers, workstations)?

```powershell
Get-DomainOU -Properties name,distinguishedname | Where-Object {$_.name -eq "Domain Controllers"} | fl

Get-DomainOU -Properties name,distinguishedname | Where-Object {$_.name -ne "Domain Controllers"} | fl 
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
	* [ ] What is the domain lockout policy (`LockoutBadConunt)?
	* [ ] How many previous passwords are stored in AD (`PasswordHistorySize`)?
	* [ ] Should passwords be stored using reversible encryption (`ClearTextPassword`)?
* [ ] What are the Kerberos ticket policy settings of the `Default Domain Policy` GPO of the current/specified domain?
	* [ ] What is the maximum lifetime for a ticket granting ticket (TGT) (`MaxTicketAge`)?
	* [ ] What is the maximum lifetime for a TGT renewal (`MaxRenewAge`)?
	* [ ] What is the maximum lifetime for a ticket granting service (TGS) (`MaxServiceAge`)?
	* [ ] What is the maximum tolerance for computer clock synchronization (`MaxClockSkew`)?
	* [ ] Does the Kerberos Key Distribution Center (KDC) validate each TGS request against the requesting account's access rights (`TicketValidateClient`)?  

```powershell
(Get-DomainPolicy -Domain domain.forest.local)."SystemAccess" | Select MinimumPasswordAge,MaximumPasswordAge,MinimumPasswordLength,PasswordComplexity,PasswordHistorySize,LockoutBadCount,ClearTextPassword

(Get-DomainPolicy -Domain domain.forest.local)."KerberosPolicy" | Select MaxTicketAge,MaxRenewAge,MaxServiceAge,MaxClockSkew,TicketValidateClient
```

* What are the settings of the `Default Domain Controllers Policy` GPO?
	* What are the user privilege assignment rights on the domain controller?

```powershell
Get-DomainPolicy -Policy DC | Select -Expand PrivilegeRights | fl
```

* [ ] What are the non-default GPOs in the domain?
	* [ ] Does the name imply a special function or service (e.g., AV/EDR products, AppLocker, LAPS, logon scripts or scheduled tasks)?

```powershell
Get-DomainGPO -Properties displayname,distinguishedname | Where-Object {$_.DisplayName -ne "Default Domain Policy" -and $_.DisplayName -ne "Default Domain Controllers Policy"} | fl
```

* [ ] Do any GPOs modify local group membership via Restricted Groups or Group Policy Preferences?
	* [ ] What is the GPO (`GPODisplayName`)?
	* [ ] How was group delegation configured (`GPOType`)?
	* [ ] What is/are the affected domain group(s) (`GroupName`)?
* [ ] What are the computer objects in the domain where a specific user/group is a member of a local group through GPO correlation?
	* [ ] What is the GPO (`ObjectName`, `ObjectDN`, `ObjectSID`, `GPODisplayName`)?
	* [ ] How was group delegation configured (`GPOType`)?
	* [ ] What is/are the affected computer object(s)/OU(s) (`ContainerName`, `ComputerName`)?
* [ ] What user/group is a member of a local group through GPO correlation on a specified computer object in the domain?
	* [ ] What is the GPO (`ObjectName`, `ObjectDN`, `ObjectSID`, `GPODisplayName`)?
	* [ ] How was group delegation configured (`GPOType`)?
	* [ ] What is/are the affected computer object(s) (`ComputerName`)?

```powershell
Get-DomainGPOLocalGroup | Select GPODisplayName, GPOType,GroupName,GroupSID,GroupMembers | fl

Get-DomainGPOUserLocalGroupMapping | Select ObjectName,ObjectDN,ObjectSID,GPODisplayName,GPOType,ContainerName,ComputerName | fl

powerpick Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity host.domain.forest.local | Select ObjectName,ObjectDN,ObjectSID,GPODisplayName,GPOType | fl
```

* [ ] Do any of the non-default GPOs introduce additional security risks to the affected OU?
	* [ ] Does the GPO contain startup/shutdown scripts?
	* [ ] Does the GPO include User Rights Assignment (e.g., `SeDebugPrivilege`)?
	* [ ] Are any credentials stored in Group Policy Preferences (e.g., `cpassword` field)?
	* [ ] Are security options disabled (e.g., SMB signing, NTLM restrictions)?

```powershell
????
```

* [ ] Do any users/groups have some kind of modification/control rights over a GPO?
* [ ] Do any users/groups have the ability to create new GPOs?

```powershell

```

### Computers

* [ ] What computers are in the current/specified domain?
	* [ ] What is the FQDN of each computer (`dnshostname`)?
	* [ ] What is the OS and OS version of each computer (`operatingsystem`, `operatingsystemversion`)?
	* [ ] What is the last logon timestamp of each computer (`lastlogon`)?
	* [ ] Does the `usercccountcontrol` attribute include unusual or risky flags?
		* [ ] `PASSWD_NOTREQD` (i.e., are empty passwords permitted?)
		* [ ] `DONT_EXPIRE_PASSWD` (i.e., does the password never expire?)
		* [ ] `TRUSTED_FOR_DELEGATION` (i.e., is the principal configured for unconstrained delegation?)
		* [ ] `NOT_DELEGATED` (i.e., is the principal protected from Kerberos delegation?)
		* [ ] `USE_DES_KEY_ONLY` (i.e., is the principal restricted to only using Data Encryption Standard (DES) encryption type for keys?)
		* [ ] `DONT_REQ_PREAUTH` (i.e., is the principal vulnerable to ASREP roasting?)
* [ ] What GPOs are applied to the specified computer object?

```powershell
Get-DomainComputer -Properties name,dnshostname,distinguishedname,objectsid,samaccountname,operatingsystem,operatingsystemversion,lastlogon,useraccountcontrol | fl


Get-DomainGPO -ComputerIdentity "host.domain.forest.com" -Properties displayname,distinguishedname | fl
```

* [ ] Is the `UserAccountControl` attribute set to `TRUSTED_FOR_DELEGATION` (i.e., is it configured for unconstrained delegation)?
* [ ] Is the *msds-allowedtodelegateto* attribute not null (i.e., is it configured for constrained delegation)?
	* [ ] Which SPN(s) is/are the computer trusted to delegate to?
	* [ ] Do the SPN(s) specify port numbers?
* [ ] Does the current/specified user identity have AD rights facilitating write access to any computer object's `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute (i.e., is the current/specified user able to conduct a resource-based constrained delegation (RBCD) attack against the computer object)?

```powershell
Get-DomainComputer -Unconstrained -Properties name,dnshostname,distinguishedname,objectsid,useraccountcontrol | fl

Get-DomainUser -TrustedToAuth -Properties distinguishedname,name,objectsid,samaccountname,msds-allowedtodelegateto,useraccountcontrol

powerpick Get-DomainComputer | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Where-Object { $_.ActiveDirectoryRights -match "WriteProperty|GenericWrite|GenericAll|WriteDacl" } | Select AceType,ObjectDN,ActiveDirectoryRights,ObjectSID,SecurityIdentifier,AccessMask,AceQualifier,Identity
```

### Users

* [ ] Are the expected default users in the current/specified domain(s)?
	* [ ] Is the account stale (`lastlogontimestamp`)?
	* [ ] Is the `description` field populated? Does it reveal credentials or hints?
	* [ ] Is the `adminCount` attribute set to `1` (i.e., protected user)?
	* [ ] What groups is the user a direct member of (`memberof`)?
	* [ ] Does the `usercccountcontrol` attribute include unusual or risky flags?
		* [ ] `ACCOUNTDISABLE` (i.e., is the principal enabled/disabled?)
		* [ ] `PASSWD_NOTREQD` (i.e., are empty passwords permitted?)
		* [ ] `DONT_EXPIRE_PASSWD` (i.e., does the password never expire?)
		* [ ] `TRUSTED_TO_AUTH_FOR_DELEGATION` (i.e., is the principal configured for constrained delegation?)
		* [ ] `NOT_DELEGATED` (i.e., is the principal protected from Kerberos delegation?)
		* [ ] `USE_DES_KEY_ONLY` (i.e., is the principal restricted to only using Data Encryption Standard (DES) encryption type for keys?)
		* [ ] `DONT_REQ_PREAUTH` (i.e., is the principal vulnerable to ASREP roasting?)

```powershell
Get-DomainUser -Identity Administrator,Guest,KRBTGT -Properties lastlogontimestamp,description,distinguishedname,name,objectsid,samaccountname,admincount,memberof,useraccountcontrol
```

* [ ] What are the non-default users in the domain?
	* [ ] Does the name imply a special function (e.g., service account, employee, administrative)?
	* [ ] Is the account stale (`lastlogontimestamp`)?
	* [ ] Is the `description` field populated? Does it reveal credentials or hints?
	* [ ] Is the `adminCount` attribute set to `1` (i.e., protected user)?
	* [ ] What groups is the user a direct member of (`memberof`)?
	* [ ] Does the `usercccountcontrol` attribute include unusual or risky flags?
		* [ ] `ACCOUNTDISABLE` (i.e., is the principal enabled/disabled?)
		* [ ] `PASSWD_NOTREQD` (i.e., are empty passwords permitted?)
		* [ ] `DONT_EXPIRE_PASSWD` (i.e., does the password never expire?)
		* [ ] `TRUSTED_TO_AUTH_FOR_DELEGATION` (i.e., is the principal configured for constrained delegation?)
		* [ ] `NOT_DELEGATED` (i.e., is the principal protected from Kerberos delegation?)
		* [ ] `USE_DES_KEY_ONLY` (i.e., is the principal restricted to only using Data Encryption Standard (DES) encryption type for keys?)
		* [ ] `DONT_REQ_PREAUTH` (i.e., is the principal vulnerable to ASREP roasting?)

```powershell
Get-DomainUser -Properties lastlogontimestamp,description,distinguishedname,name,objectsid,samaccountname,admincount,memberof,useraccountcontrol | Where-Object { $_.name -ne "Administrator" -and $_.name -ne "Guest" -and $_.name -ne "KRBTGT" } | fl
```

* [ ] What nested groups is the current/specified user(s) effectively a member of (recursing 'up' using `tokenGroups`)? 

```powershell
Get-DomainGroup -MemberIdentity "John Smith" -Properties samaccountname,objectsid,name,description,distinguishedname,member | fl
```

* [ ] Is the `serviceprincipalname` attribute non-null (i.e., is it kerberoastable)?
	* [ ] What is the Service Principal Name (SPN) associated with the service account (`serviceprincipalname`)?
* [ ] Does the `useraccountcontrol` attribute include `DONT_REQ_PREAUTH` (i.e., is the principal vulnerable to ASREP roasting?)?

```powershell
Get-DomainUser -SPN -Properties lastlogontimestamp,description,distinguishedname,name,objectsid,samaccountname,admincount,memberof,useraccountcontrol,serviceprincipalname | fl

Get-DomainUser -KerberosPreauthNotRequired -Properties lastlogontimestamp,description,distinguishedname,name,objectsid,samaccountname,admincount,memberof,useraccountcontrol | fl

Get-DomainUser -UACFilter DONT_REQ_PREAUTH -Properties lastlogontimestamp,description,distinguishedname,name,objectsid,samaccountname,admincount,memberof,useraccountcontrol | fl
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
"Administrators","Users","Guests","Domain Admins","Domain Users","Domain Guests","Account Operators","Server Operators","Print Operators","Backup Operators","Replicator","Incoming Forest Trust Builders","Pre–Windows 2000 Compatible Access","Enterprise Admins","Schema Admins","Cert Publishers","DnsAdmins","DnsUpdateProxy","Group Policy Creator Owners","Read-only Domain Controllers","Denied RODC Password Replication Group","Allowed RODC Password Replication Group","Cloneable Domain Controllers","Protected Users","Key Admins","Enterprise Key Admins" | ForEach-Object { Get-DomainGroup -Properties description,name,distinguishedname -Identity $_ } | fl

"Domain Admins","Enterprise Admins","Schema Admins","Account Operators","Backup Operators","Print Operators","Server Operators","DnsAdmins","Group Policy Creator Owners","Protected Users","Read-only Domain Controllers" | ForEach-Object { Get-DomainGroupMember -Identity $_ -Recurse } | Select GroupDomain,GroupName,GroupDistinguishedName,MemberDomain,MemberName,MemberDistinguishedName,MemberObjectClass,MemberSID
```

* [ ] What are the local groups of the current host?
* [ ] What are the local groups of the specified host(s)?
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
Get-NetLocalGroup -Method <API-or-WinNT> | Select ComputerName,GroupName,Comment

Get-NetLocalGroup -Method <API-or-WinNT> -ComputerName host.domain.forest.local | Select ComputerName,GroupName,Comment

"Administrators","Remote Desktop Users","Remote Management Users" | ForEach-Object { Get-NetLocalGroupMember -Method <API-or-WinNT> -GroupName $_ } | Select ComputerName,GroupName,MemberName,SID,IsGroup,IsDomain

"Administrators","Remote Desktop Users","Remote Management Users" | ForEach-Object { Get-NetLocalGroupMember -Method <API-or-WinNT> -GroupName $_ -ComputerName host.domain.forest.local } | Select ComputerName,GroupName,MemberName,SID,IsGroup,IsDomain
```

* [ ] What are the non-default groups in the domain?
	* [ ] Does the name imply a special function (e.g., administrative access to a set of hosts, service access, geography)?
	* [ ] Is the `adminCount` attribute set to `1` (i.e., protected group)?

```powershell
Get-DomainGroup -Properties samaccountname,objectsid,cn,name,description,distinguishedname,adminCount | Where-Object { $_.samaccountname -notin @("Administrators","Users","Guests","Domain Admins","Domain Users","Domain Guests","Account Operators","Server Operators","Print Operators","Backup Operators","Replicator","Incoming Forest Trust Builders","Pre–Windows 2000 Compatible Access","Enterprise Admins","Schema Admins","Cert Publishers","DnsAdmins","DnsUpdateProxy","Group Policy Creator Owners","Read-only Domain Controllers","Denied RODC Password Replication Group","Allowed RODC Password Replication Group","Cloneable Domain Controllers","Protected Users","Key Admins","Enterprise Key Admins") }
```

* [ ] What nested groups is the current/specified group(s) effectively a member of (recursing 'up' using `tokenGroups`)? 
* [ ] Who are the effective members of the current/specified group(s)?
* [ ] Do any groups of the current/specified domain(s) contain members from foreign domains?

```powershell
Get-DomainGroup -MemberIdentity "Domain Admins" -Properties samaccountname,objectsid,name,description,distinguishedname,member | fl

Get-DomainGroupMember -Identity "Domain Admins" -Recurse | Select GroupDomain,GroupName,GroupDistinguishedName,MemberDomain,MemberName,MemberDistinguishedName,MemberObjectClass,MemberSID | fl

Get-DomainForeignGroupMember -Domain domain.forest.local -Properties GroupDomain,GroupName,GroupDistinguishedName,MemberDomain,MemberName,MemberDistinguishedName | fl
```

### Session Enumeration

* [ ] 

```powershell

```

### Data Exfiltration

* [ ] Are there domain shares?
* [ ] Are there accessible domain shares?
* [ ] Are there accessible domain shares that match a given substring(s) filter?

```powershell
Find-DomainShare

Find-DomainShare -CheckShareAccess

Find-InterestingDomainShareFile -Include *.doc*, *.xls*, *.csv, *.ppt*, *.pdf, *.txt, *.png, *.jp*
```

***
## Resources:

| Hyperlink                                                                                                                                | Info                                                                       |
| ---------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [HarmJ0y CheatSheets - PowerView](https://github.com/HarmJ0y/CheatSheets/blob/master/PowerView.pdf)                                      | PowerView 3.0 Cheat Sheet by PowerSploit Co-creator                        |
| [Active Directory - Enumeration: PowerView CheatSheet](https://zflemingg1.gitbook.io/undergrad-tutorials/powerview/powerview-cheatsheet) | Zach Fleming's (AKA _zflemingg1_) Active Directory Enumeration Cheat Sheet |
| [HarmJ0y GitHub Gist - PowerView 3.0 Tricks](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)                           | PowerView 3.0 Sample Commands by PowerSploit Co-creator                    |
| [PowerSploit - Recon](https://powersploit.readthedocs.io/en/latest/Recon/)                                                               | Official PowerSploit Documentation for PowerView                           |

***

[^1]: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview
[^2]: https://powersploit.readthedocs.io/en/latest/Recon/
[^3]: https://github.com/its-a-feature/Mythic
[^4]: https://github.com/MythicAgents/Apollo

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
