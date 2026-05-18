---
layout: single
title: "Shorten-the-Code 2026 PSCONFEU Contest"
date: 2026-06-04
permalink: /stc-psconfeu2026
author_profile: false
---

## Welcome to our contest

If you open this page you commit to our great contest for PSCONFEU 2026.  
Enjoy the contest!

## Contest Rules
Here comes the rules

*   ✅ Output must be IDENTICAL
*   ✅ No external modules
*   ✅ PowerShell 5.1 or higher


## The Code

```powershell

$psconfeuEditions = @(
    [PSCustomObject]@{ Year = 2014; City = 'Amsterdam';  Country = 'Netherlands';    APIName = 'Netherlands'    }
    [PSCustomObject]@{ Year = 2015; City = 'Stockholm';  Country = 'Sweden';         APIName = 'Sweden'         }
    [PSCustomObject]@{ Year = 2016; City = 'Hannover';   Country = 'Germany';        APIName = 'Germany'        }
    [PSCustomObject]@{ Year = 2017; City = 'Vienna';     Country = 'Austria';        APIName = 'Austria'        }
    [PSCustomObject]@{ Year = 2018; City = 'Prague';     Country = 'Czech Republic'; APIName = 'Czech Republic' }
    [PSCustomObject]@{ Year = 2019; City = 'Hannover';   Country = 'Germany';        APIName = 'Germany'        }
    [PSCustomObject]@{ Year = 2020; City = 'Online';     Country = 'Online';         APIName = $null            }
    [PSCustomObject]@{ Year = 2021; City = 'Online';     Country = 'Online';         APIName = $null            }
    [PSCustomObject]@{ Year = 2022; City = 'Online';     Country = 'Online';         APIName = $null            }
    [PSCustomObject]@{ Year = 2023; City = 'Edinburgh';  Country = 'Scotland/UK';    APIName = 'United Kingdom' }
    [PSCustomObject]@{ Year = 2024; City = 'Vienna';     Country = 'Austria';        APIName = 'Austria'        }
    [PSCustomObject]@{ Year = 2025; City = 'Wiesbaden';  Country = 'Germany';        APIName = 'Germany'        }
)

# Get unique API names (skip Online)
$uniqueAPINames = $psconfeuEditions | 
    Where-Object { $_.APIName -ne $null } | 
    Select-Object -ExpandProperty APIName | 
    Select-Object -Unique

# Fetch country details from REST API
$countryDetails = [System.Collections.Generic.List[PSCustomObject]]::new()

foreach ($apiName in $uniqueAPINames) {
    Write-Host "Fetching: $apiName..." -ForegroundColor Cyan
    
    $searchName = $apiName.Replace(' ','%20')
    $apiResult  = Invoke-RestMethod -Uri "https://restcountries.com/v3.1/name/$searchName"
    $apiData    = $apiResult | Select-Object -First 1

    $countryDetails.Add([PSCustomObject]@{
        APIName    = $apiName
        Capital    = $apiData.capital[0]
        Population = $apiData.population
        Area_km2   = $apiData.area
        Flag       = $apiData.flag
        Currency   = ($apiData.currencies.PSObject.Properties.Name)[0]
        Languages  = ($apiData.languages.PSObject.Properties.Value -join ' / ')
    })
}

# Build full report
$fullReport = [System.Collections.Generic.List[PSCustomObject]]::new()

foreach ($edition in $psconfeuEditions) {
    $details = $countryDetails | 
        Where-Object { $_.APIName -eq $edition.APIName } | 
        Select-Object -First 1

    $fullReport.Add([PSCustomObject]@{
        Year       = $edition.Year
        City       = $edition.City
        Country    = $edition.Country
        Flag       = if ($details) { $details.Flag       } else { '🌐'  }
        Population = if ($details) { $details.Population } else { 0     }
        Area_km2   = if ($details) { $details.Area_km2   } else { 0     }
        Currency   = if ($details) { $details.Currency   } else { 'N/A' }
        Languages  = if ($details) { $details.Languages  } else { 'N/A' }
    })
}

# Display Full Report
Write-Output ""
Write-Output "╔══════════════════════════════════════════════════════╗"
Write-Output "║        PSConfEU - ALL EDITIONS REPORT               ║"
Write-Output "╚══════════════════════════════════════════════════════╝"
$fullReport | Format-Table Year, City, Country, Flag, Population, Currency -AutoSize

# Country Statistics
$countryStats = $fullReport | 
    Where-Object { $_.Country -ne 'Online' } |
    Group-Object -Property Country |
    ForEach-Object {
        [PSCustomObject]@{
            Country    = $_.Name
            Editions   = $_.Count
            Cities     = ($_.Group.City | Select-Object -Unique) -join ', '
            Flag       = $_.Group.Flag | Select-Object -First 1
            Population = $_.Group.Population | Select-Object -First 1
        }
    } | Sort-Object -Property Editions -Descending

Write-Output "╔══════════════════════════════════════════════════════╗"
Write-Output "║        PSConfEU - COUNTRY STATISTICS                ║"
Write-Output "╚══════════════════════════════════════════════════════╝"
$countryStats | Format-Table Country, Flag, Editions, Cities, Population -AutoSize

# Summary
$mostVisited = $countryStats | Select-Object -First 1
Write-Output "🏆 Most Visited : $($mostVisited.Flag) $($mostVisited.Country) ($($mostVisited.Editions) editions)"
Write-Output "📍 Cities       : $($mostVisited.Cities)"
Write-Output ""
Write-Output "📊 Physical Editions : $(($fullReport | Where-Object {$_.Country -ne 'Online'}).Count)"
Write-Output "💻 Online Editions   : $(($fullReport | Where-Object {$_.Country -eq 'Online'}).Count)"
Write-Output "📅 Total Editions    : $($fullReport.Count)"
```
