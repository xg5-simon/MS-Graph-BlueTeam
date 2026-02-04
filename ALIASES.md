# Aliases

The Microsoft Graph PowerShell SDK cmdlets can become quite lengthy, so here is an example alias configuration for PowerShell.

## PowerShell

1. Open $PROFILE with an editor, e.g., `code $PROFILE`
2. Add the code snippet below to your PowerShell $PROFILE
3. From PowerShell run `. $PROFILE` to reload your PowerShell profile

```PowerShell
function Get-ThreatIntelPassiveDnsReverse {
    param(
        [Parameter(Mandatory=$true)]
        [string]$HostId
    )
    Write-Host "Running Get-ThreatIntelPassiveDnsReverse for $HostId"
    Get-MgSecurityThreatIntelligenceHostPassiveDnsReverse -HostId $HostId
}

function Get-ThreatIntelWhois {
    param(
        [Parameter(Mandatory=$true)]
        [string]$HostId
    )
    Write-Host "Running Get-ThreatIntelWhois for $HostId"
    Get-MgSecurityThreatIntelligenceHostWhois -HostId $HostId
}

function Get-ThreatIntelHostReputation {
    param(
        [Parameter(Mandatory=$true)]
        [string]$HostId
    )
    Write-Host "Running Get-ThreatIntelHostReputation for $HostId"
    Get-MgSecurityThreatIntelligenceHostReputation -HostId $HostId
}

function Get-ThreatIntelArticle {
    param(
        [Parameter(Mandatory=$false)]
        [string]$SearchTerm,
        
        [Parameter(Mandatory=$false)]
        [string]$ArticleId
    )
    
    if ($ArticleId) {
        Write-Host "Getting specific threat intelligence article: $ArticleId"
        Get-MgSecurityThreatIntelligenceArticle -ArticleId $ArticleId
    }
    elseif ($SearchTerm) {
        Write-Host "Searching threat intelligence articles for: $SearchTerm"
        Get-MgSecurityThreatIntelligenceArticle -Filter "contains(title,'$SearchTerm')" -Select "id,title"
    }
    else {
        Write-Host "Getting all threat intelligence articles"
        Get-MgSecurityThreatIntelligenceArticle -Select "id,title"
    }
}

function Invoke-AdvancedHuntingQuery {
    param(
        [Parameter(Mandatory=$true)]
        [string]$Query
    )
    Write-Host "Running Advanced Hunting Query"
    $QueryBody = @{
        Query = $Query
    }
    Invoke-MgSecurityRunHuntingQuery -BodyParameter $QueryBody
}

# Aliases
Set-Alias -Name tipdnsr -Value Get-ThreatIntelPassiveDnsReverse
Set-Alias -Name tiwhois -Value Get-ThreatIntelWhois
Set-Alias -Name tireputation -Value Get-ThreatIntelHostReputation
Set-Alias -Name tiarticle -Value Get-ThreatIntelArticle
Set-Alias -Name huntingquery -Value Invoke-AdvancedHuntingQuery

# Connection helper
function Connect-MgThreatIntel {
    Write-Host "Connecting to Microsoft Graph with Threat Intelligence permissions..."
    Connect-MgGraph -Scopes "ThreatIntelligence.Read.All"
}

function Connect-MgDefender {
    Write-Host "Connecting to Microsoft Graph with M365 Defender permissions..."
    Connect-MgGraph -Scopes "ThreatHunting.Read.All"
}

Set-Alias -Name connectti -Value Connect-MgThreatIntel
Set-Alias -Name connectdefender -Value Connect-MgDefender
```

### Usage Examples

After loading the profile, you can use the simplified commands:

```powershell
# Connect to Graph
connectti

# Get WHOIS data
tiwhois "contoso.com"

# Get domain reputation
tireputation "contoso.com"

# Search threat intel articles
tiarticle -SearchTerm "Confluence"

# Run hunting query
huntingquery 'DeviceProcessEvents | where ProcessCommandLine contains "powershell" | limit 10'
```
