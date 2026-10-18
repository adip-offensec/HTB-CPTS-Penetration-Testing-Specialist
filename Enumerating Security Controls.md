# Enumerating Security Controls

## Windows Defender

### Checking the Status of Defender with Get-MpComputerStatus

```powershell-session
PS C:\htb> Get-MpComputerStatus
```

## AppLocker

### **Using Get-AppLockerPolicy cmdlet**

```powershell-session
PS C:\htb> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

## PowerShell Constrained Language Mode

### Enumerating Language Mode

```powershell-session
PS C:\htb> $ExecutionContext.SessionState.LanguageMode
```

## LAPS

> The Microsoft [Local Administrator Password Solution (LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899) is used to randomize and rotate local administrator passwords on Windows hosts and prevent lateral movement. We can enumerate what domain users can read the LAPS password set for machines with LAPS installed and what machines do not have LAPS installed. The [LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit) greatly facilitates this with several functions. One is parsing `ExtendedRights` for all computers with LAPS enabled. This will show groups specifically delegated to read LAPS passwords, which are often users in protected groups. An account that has joined a computer to a domain receives `All Extended Rights` over that host, and this right gives the account the ability to read passwords. Enumeration may show a user account that can read the LAPS password on a host. This can help us target specific AD users who can read LAPS passwords.

* <https://github.com/leoloobeek/LAPSToolkit>

### Using Find-LAPSDelegatedGroups

```powershell-session
PS C:\htb> Find-LAPSDelegatedGroups
```

The `Find-AdmPwdExtendedRights` checks the rights on each computer with LAPS enabled for any groups with read access and users with "All Extended Rights." Users with "All Extended Rights" can read LAPS passwords and may be less protected than users in delegated groups, so this is worth checking for.

### **Using Find-AdmPwdExtendedRights**

```powershell-session
PS C:\htb> Find-AdmPwdExtendedRights
```

> We can use the `Get-LAPSComputers` function to search for computers that have LAPS enabled when passwords expire, and even the randomized passwords in cleartext if our user has access.

### Using Get-LAPSComputers

```powershell-session
PS C:\htb> Get-LAPSComputers
```
