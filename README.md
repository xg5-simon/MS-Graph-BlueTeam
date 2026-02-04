# MS-Graph-BlueTeam

[Microsoft Graph](https://developer.microsoft.com/en-us/graph) PowerShell SDK commands and workflows for Blue Teamers

## Microsoft Graph PowerShell SDK

### Requirements

1. Install the [Microsoft Graph PowerShell SDK](https://learn.microsoft.com/en-us/powershell/microsoftgraph/installation?view=graph-powershell-1.0).
2. Requisite licenses for certain API endpoints, i.e., Microsoft Defender for Threat Intelligence.

### Installation

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser
```

For threat intelligence features, you may also need the beta module:
```powershell
Install-Module Microsoft.Graph.Beta -Scope CurrentUser
```

### Command Discovery

To find available Microsoft Graph PowerShell cmdlets, use:
```powershell
# Find commands related to security
Get-Command -Module Microsoft.Graph.Security

# Discover cmdlet equivalents
Find-MgGraphCommand -Command 'threat'
```

### Microsoft Defender for Threat Intelligence

#### Connect and configure MDTI Scope

```powershell
Connect-MgGraph -Scopes "ThreatIntelligence.Read.All"
```

#### Whois

```powershell
Get-MgSecurityThreatIntelligenceHostWhois -HostId "contoso.com"
```

##### Convert MDTI WHOIS to a MISP Object

```PowerShell
function ConvertTo-MispObject($whoisData) {
    $mispObject = @{
        "name" = "whois"
        "meta-category" = "network"
        "description" = "Whois record for a domain"
        "template_uuid" = "688c46fb-5edb-40a3-8273-1af7923e2215"
        "template_version" = "4"
        "Attribute" = @(
            @{ "type" = "datetime"; "object_relation" = "expiration-date"; "value" = $whoisData.ExpirationDateTime }
            @{ "type" = "datetime"; "object_relation" = "creation-date"; "value" = $whoisData.RegistrationDateTime }
            @{ "type" = "text"; "object_relation" = "registrar"; "value" = $whoisData.Registrar.Organization }
            @{ "type" = "text"; "object_relation" = "registrant"; "value" = $whoisData.Registrant.Organization }
            @{ "type" = "text"; "object_relation" = "registrant-phone"; "value" = $whoisData.Registrant.Telephone }
            @{ "type" = "email"; "object_relation" = "registrant-email"; "value" = $whoisData.Registrant.Email }
            @{ "type" = "text"; "object_relation" = "registrant-address"; "value" = $whoisData.Registrant.Address.Street }
            @{ "type" = "text"; "object_relation" = "domain-status"; "value" = $whoisData.DomainStatus }
            @{ "type" = "text"; "object_relation" = "whois-server"; "value" = $whoisData.WhoisServer }
            @{ "type" = "text"; "object_relation" = "raw-record"; "value" = $whoisData.RawWhoisText }
        )
    }

    for ($i = 0; $i -lt $whoisData.Nameservers.Count; $i++) {
        $mispObject.Attribute += @{ "type" = "hostname"; "object_relation" = "nameserver-$($i + 1)"; "value" = $whoisData.Nameservers[$i].Host.Id }
    }

    return $mispObject
}

$whoisData = Get-MgSecurityThreatIntelligenceHostWhois -HostId "contoso.com"

$mispObject = ConvertTo-MispObject -whoisData $whoisData

# Print the MISP object
$mispObject | ConvertTo-Json -Depth 10
```

#### Domain Reputation

```powershell
Get-MgSecurityThreatIntelligenceHostReputation -HostId "contoso.com"
```

#### Reverse DNS

```powershell
Get-MgSecurityThreatIntelligenceHostPassiveDnsReverse -HostId "contoso.com"
```

#### View results with JSON Crack in VSCode

##### Pre-requisites

[JSON Crack VSCode Extension](https://marketplace.visualstudio.com/items?itemName=AykutSarac.jsoncrack-vscode)

```powershell
Get-MgSecurityThreatIntelligenceHostWhois -HostId "contoso.com" | ConvertTo-Json -Depth 10 | Out-File "contoso_whois.json"; code "contoso_whois.json"
```

```bash
CTRL + SHIFT + p
menubar: Enable JSON Crack visualization
ENTER
```

![image](https://github.com/xg5-simon/MS-Graph-BlueTeam/assets/8979648/9c906db3-cb8c-4627-9193-9c2acbb7bfb0)

#### Search Threat Intel Articles

Search articles for a keyword, provide a count and only return the article ID and Title.

```powershell
Get-MgSecurityThreatIntelligenceArticle -Filter "contains(title,'Confluence')" -Select "id,title" -CountVariable resultCount
Write-Host "Found $resultCount articles"
```

Return the results as a formatted table:

```powershell
Get-MgSecurityThreatIntelligenceArticle -Filter "contains(title,'Confluence')" -Select "id,title" | Format-Table -AutoSize
```

#### List indicators for a Threat Intel Article

```powershell
Get-MgSecurityThreatIntelligenceArticleIndicator -ArticleId "<id>"
```

#### Render a Threat Intel Article as Markdown in the terminal (PowerShell)

```PowerShell
$articleObject = Get-MgSecurityThreatIntelligenceArticle -ArticleId "<id>"
$articleObject.Body.Content | Show-Markdown
```

### M365 Defender

#### Connect and configure M365D Scope

```powershell
Connect-MgGraph -Scopes "ThreatHunting.Read.All"
```

#### Run an Advanced Hunting Query

```powershell
$Query = @{
    Query = 'DeviceProcessEvents | where InitiatingProcessFileName =~ "powershell.exe" | project Timestamp, FileName, InitiatingProcessFileName | order by Timestamp desc | limit 2'
}

Invoke-MgSecurityRunHuntingQuery -BodyParameter $Query
```

#### Run an Advanced Hunting Query from the Microsoft Sentinel Github Repo

```PowerShell
Import-Module powershell-yaml

$yamlContent = Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/Hunting%20Queries/Microsoft%20365%20Defender/Troubleshooting/Connectivity%20Failures%20by%20Device.yaml"
$parsedYaml = ConvertFrom-Yaml $yamlContent.Content

$Query = @{
    Query = $parsedYaml.query
}

Invoke-MgSecurityRunHuntingQuery -BodyParameter $Query
```

or save the results as a PowerShell object:

```PowerShell
$result = Invoke-MgSecurityRunHuntingQuery -BodyParameter $Query
$result.Results
```
