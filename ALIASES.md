# Aliases

The Microsoft Graph CLI commands can become quite lenghty so here is an example alias configuration for PowerShell and Bash. 

## PowerShell

1. Open $PROFILE with an editor, e.g., `code $PROFILE`
2. Add the code snippet below to your Powershell $PROFILE
3. From PowerShell run `. $PROFILE` to reload your PowerShell profile

```PowerShell
function Get-ThreatIntelPassiveDnsReverse {
    param(
        [Parameter(Mandatory=$true)]
        [string]$hostId
    )
    Write-Host "Running Get-ThreatIntelPassiveDnsReverse for $hostId"
    mgc security threat-intelligence hosts passive-dns-reverse list --host-id $hostId
}

function Get-ThreatIntelWhois {
    param(
        [Parameter(Mandatory=$true)]
        [string]$hostId
    )
    Write-Host "Running Get-ThreatIntelWhois for $hostId"
    mgc security threat-intelligence hosts whois get --host-id $hostId
}

function Get-ThreatIntelHostReputation {
    param(
        [Parameter(Mandatory=$true)]
        [string]$hostId
    )
    Write-Host "Running Get-ThreatIntelHostReputation for $hostId"
    mgc security threat-intelligence hosts reputation get --host-id $hostId
}

# Alias
Set-Alias -Name tipdnsr -Value Get-ThreatIntelPassiveDnsReverse
Set-Alias -Name tiwhois -Value Get-ThreatIntelWhois
Set-Alias -Name tireputation -Value Get-ThreatIntelHostReputation
```

## Bash

```bash
```
