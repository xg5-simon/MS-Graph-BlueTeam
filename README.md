# MS-Graph-BlueTeam

[Microsoft Graph](https://developer.microsoft.com/en-us/graph) CLI commands and workflows for Blue Teamers

## MS Graph CLI

### Requirements

1. Install the [MS Graph CLI](https://learn.microsoft.com/en-us/graph/cli/installation?tabs=windows) toolkit.
2. Requisite licenses for certain API endpoints, i.e., Microsoft Defender for Threat Intelligence.

### Microsoft Defender for Threat Intelligence

#### Login and configure MDTI Scope

```bash
mgc login --strategy DeviceCode --scopes ThreatIntelligence.Read.All
```

#### Whois

```bash
mgc security threat-intelligence hosts whois get --host-id contoso.com
```

##### Convert MDTI WHOIS to a MISP Object

```PowerShell
function ConvertTo-MispObject($whoisJson) {
    $mispObject = @{
        "name" = "whois"
        "meta-category" = "network"
        "description" = "Whois record for a domain"
        "template_uuid" = "688c46fb-5edb-40a3-8273-1af7923e2215"
        "template_version" = "4"
        "Attribute" = @(
            @{ "type" = "datetime"; "object_relation" = "expiration-date"; "value" = $whoisJson.expirationDateTime }
            @{ "type" = "datetime"; "object_relation" = "creation-date"; "value" = $whoisJson.registrationDateTime }
            @{ "type" = "text"; "object_relation" = "registrar"; "value" = $whoisJson.registrar.organization }
            @{ "type" = "text"; "object_relation" = "registrant"; "value" = $whoisJson.registrant.organization }
            @{ "type" = "text"; "object_relation" = "registrant-phone"; "value" = $whoisJson.registrant.telephone }
            @{ "type" = "email"; "object_relation" = "registrant-email"; "value" = $whoisJson.registrant.email }
            @{ "type" = "text"; "object_relation" = "registrant-address"; "value" = $whoisJson.registrant.address.street }
            @{ "type" = "text"; "object_relation" = "domain-status"; "value" = $whoisJson.domainStatus }
            @{ "type" = "text"; "object_relation" = "whois-server"; "value" = $whoisJson.whoisServer }
            @{ "type" = "text"; "object_relation" = "raw-record"; "value" = $whoisJson.rawWhoisText }
        )
    }

    for ($i = 0; $i -lt $whoisJson.nameservers.Count; $i++) {
        $mispObject.Attribute += @{ "type" = "hostname"; "object_relation" = "nameserver-$($i + 1)"; "value" = $whoisJson.nameservers[$i].host.id }
    }

    return $mispObject
}

$whoisJson = mgc security threat-intelligence hosts whois get --host-id contoso.com | ConvertFrom-Json

$mispObject = ConvertTo-MispObject -whoisJson $whoisJson

# Print the MISP object
$mispObject | ConvertTo-Json -Depth 10
```

#### Domain Reputation

```bash
mgc security threat-intelligence hosts reputation get --host-id contoso.com
```

#### Reverse DNS

```bash
mgc security threat-intelligence hosts passive-dns-reverse list --host-id contoso.com
```

#### View results with JSON Crackin VSCode

##### Pre-requisites

[JSON Crack VSCode Extension](https://marketplace.visualstudio.com/items?itemName=AykutSarac.jsoncrack-vscode)

```bash
mgc security threat-intelligence hosts whois get --host-id contoso.com > contoso_whois.json; code contoso_whois.json
```

```bash
CTRL + SHIFT + p
menubar: Enable JSON Crack visualization
ENTER
```

![image](https://github.com/xg5-simon/MS-Graph-BlueTeam/assets/8979648/9c906db3-cb8c-4627-9193-9c2acbb7bfb0)

#### Search Threat Intel Articles

Search articles for a keyword, provide a count and only return the article ID and Title.

```bash
mgc security threat-intelligence articles list --select title --search "Confluence" --count "true"
```

Return the results as table

```bash
mgc security threat-intelligence articles list --search "Confluence" --count "true" --output TABLE
```

#### List indicators for a Threat Intel Article

```bash
mgc security threat-intelligence articles indicators list --article-id <id>
```

#### Render a Threat Intel Article as Markdown in the terminal (PowerShell)

```PowerShell
$articleObject = mgc security threat-intelligence articles get --article-id "<id>" | ConvertFrom-Json
$articleObject.body.content | Show-Markdown
```

### M365 Defender

#### Login and configure M365D Scope

```bash
mgc login --strategy DeviceCode --scopes ThreatHunting.Read.All
```

#### Run an Advanced Hunting Query

```bash
mgc security microsoft-graph-security-run-hunting-query post --body '{"Query":"DeviceProcessEvents | where InitiatingProcessFileName =~ \"powershell.exe\" | project Timestamp, FileName, InitiatingProcessFileName | order by Timestamp desc | limit 2"}'
```

#### Run an Advanced Hunting Query from the Microsoft Sentinel Github Repo

```PowerShell
Import-Module powershell-yaml

$jsonObject = ConvertTo-Json -InputObject @{ Query = (ConvertFrom-Yaml (Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/Hunting%20Queries/Microsoft%20365%20Defender/Troubleshooting/Connectivity%20Failures%20by%20Device.yaml")).query }

mgc security microsoft-graph-security-run-hunting-query post --body $jsonObject
```
or save the results as a PSCustom Object

```PowerShell
$result = mgc security microsoft-graph-security-run-hunting-query post --body $jsonObject | ConvertFrom-Json
$result.results
```
