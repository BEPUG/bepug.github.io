---
layout: single
title: "Shorten-the-Code 2026 PSCONF-EU Contest"
date: 2026-06-04
permalink: /stc-psconfeu2026
author_profile: false
---

## 🏆 PSCONFEU Contest — BEPUG

> *Welcome to our official contest page!*

---

## 👋 Welcome!

You've opened this page — which means you're accepting the challenge. **Thank you!** 🎉

---

## 🙏 Special Thanks

First and foremost, we want to give a big thank you to **PSCONFEU** for allowing us to organize a contest again this year!

---

## 🎯 The Challenge: Shorten-the-Code

This year we are once again hosting a **Shorten-the-Code** contest — but this time with an extra level of difficulty!

As you will notice, we are playing into the **10th Anniversary of PSCONFEU** 🎂  
We want to bring the statistics of this milestone to life... in PowerShell code.

> *We hope that, just like every year, you will find this contest a fun and exciting experience!*

---

## 📋 What do you need to do?

Your mission is clear:

- ✂️ **Shorten the existing code** without changing the output
- 📊 The code displays statistics around the **10 editions of PSCONFEU**
- 🏅 The **3 shortest and correct** solutions win!

---

## ⏰ Deadline

| | Date & Time |
|---|---|
| 🟢 **Start** | Monday *(Live moment)* |
| 🔴 **End** | Wednesday **3rd of June 2026 — 18:00 CET** |

> ⚠️ Submissions received after the deadline will **not** be considered.

---

## 📜 Rules

> ‼️ **Please read the rules carefully before participating!**

| | Rule |
|---|---|
| ✅ | Output must be **IDENTICAL** to the original |
| ✅ | **No external modules** allowed |
| ✅ | The code must run on both **PS5 and PS7** |
| ✅ | All outputs must still be present — **exact result** as the start code |
| ❌ | **No shortening of URLs** is allowed |
| 🏆 | **Top 3 shortest** solutions win! |
| 🎲 | In case of a tie, we'll use `Get-Random` to determine the winner |
| 📧 | Mail your shortened solution to **[bepug@bepug.be](mailto:bepug@bepug.be)**: |
| ➡️ | *Do not forget to send your name and your country within the mail.*|

---

## 🏁 Results

At the end of the **10th edition of PSCONFEU**, we will announce and share the results with all participants.

Stay tuned for the announcement! 👀

---

## 💬 Good Luck!

> *Enjoy the contest and good luck to everyone!*  
> — **The BEPUG Team** 🇧🇪⚡

---

## 🚀 The Challenge Code

> ⬇️ **This is the code you need to shorten — good luck!**  
> The output must remain **identical** to the original.

```powershell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8

$psconfeuEditions = @(
    [PSCustomObject]@{ Year = 2016; City = 'Hannover';  Country = 'Germany';        Notes = 'Inaugural European event';             APICode = 'DEU' }
    [PSCustomObject]@{ Year = 2017; City = 'Hannover';  Country = 'Germany';        Notes = '-';                                    APICode = 'DEU' }
    [PSCustomObject]@{ Year = 2018; City = 'Hannover';  Country = 'Germany';        Notes = 'Core repositories archived on GitHub'; APICode = 'DEU' }
    [PSCustomObject]@{ Year = 2019; City = 'Hannover';  Country = 'Germany';        Notes = 'Held at Hannover Congress Centrum';    APICode = 'DEU' }
    [PSCustomObject]@{ Year = 2020; City = 'Online';    Country = 'N/A';            Notes = 'Online due to COVID-19';               APICode = $null }
    [PSCustomObject]@{ Year = 2021; City = 'No Conf';   Country = 'N/A';            Notes = 'No conference in 2021';                APICode = $null }
    [PSCustomObject]@{ Year = 2022; City = 'Vienna';    Country = 'Austria';        Notes = 'First in-person event post-COVID';     APICode = 'AUT' }
    [PSCustomObject]@{ Year = 2023; City = 'Prague';    Country = 'Czech Republic'; Notes = '-';                                    APICode = 'CZE' }
    [PSCustomObject]@{ Year = 2024; City = 'Antwerp';   Country = 'Belgium';        Notes = 'Core files hosted on PSConfEU GitHub'; APICode = 'BEL' }
    [PSCustomObject]@{ Year = 2025; City = 'Malmo';     Country = 'Sweden';         Notes = 'Held in late June';                    APICode = 'SWE' }
    [PSCustomObject]@{ Year = 2026; City = 'Wiesbaden'; Country = 'Germany';        Notes = '10th Anniversary Edition';             APICode = 'DEU' }
)

# Get unique API codes - skip Online and No Conf
$uniqueAPICodes = $psconfeuEditions | 
    Where-Object { $null -ne $_.APICode } | 
    Select-Object -ExpandProperty APICode | 
    Select-Object -Unique

# Fetch country details via REST API
$countryDetails = [System.Collections.Generic.List[PSCustomObject]]::new()

foreach ($code in $uniqueAPICodes) {
    Write-Host "Fetching: $code..." -ForegroundColor Cyan
    $apiData = Invoke-RestMethod -Uri "https://restcountries.com/v3.1/alpha/$code"

    $currencyCode = $apiData.currencies.PSObject.Properties.Name -join ', '
    $currencyName = ($apiData.currencies.PSObject.Properties.Name | ForEach-Object {
        "$_ ($($apiData.currencies.$_.name))"
    }) -join ', '

    $countryDetails.Add([PSCustomObject]@{
        APICode      = [string]$code
        Capital      = [string]$apiData.capital
        Population   = [int]$apiData.population
        Area_km2     = [int][math]::Round($apiData.area, 0)
        CurrencyCode = [string]$currencyCode
        CurrencyName = [string]$currencyName
        Languages    = [string]($apiData.languages.PSObject.Properties.Value -join ' / ')
    })
}

# Build full report
$fullReport = [System.Collections.Generic.List[PSCustomObject]]::new()

foreach ($edition in $psconfeuEditions) {
    $details = $null
    foreach ($detail in $countryDetails) {
        if ($detail.APICode -eq $edition.APICode) {
            $details = $detail
        }
    }

    $fullReport.Add([PSCustomObject]@{
        Year         = [int]$edition.Year
        City         = [string]$edition.City
        Country      = [string]$edition.Country
        Notes        = [string]$edition.Notes
        Capital      = if ($details) { [string]$details.Capital      } else { 'N/A' }
        Population   = if ($details) { [int]$details.Population      } else { 0     }
        Area_km2     = if ($details) { [int]$details.Area_km2        } else { 0     }
        CurrencyCode = if ($details) { [string]$details.CurrencyCode } else { 'N/A' }
        CurrencyName = if ($details) { [string]$details.CurrencyName } else { 'N/A' }
        Languages    = if ($details) { [string]$details.Languages    } else { 'N/A' }
    })
}

# Display Full Report
Write-Output ""
Write-Output "======================================================"
Write-Output "         PSConfEU - ALL EDITIONS REPORT               "
Write-Output "======================================================"
$fullReport | Format-Table -AutoSize -Property `
    @{N='Year';       E={$_.Year};         Width=6  },
    @{N='City';       E={$_.City};         Width=12 },
    @{N='Country';    E={$_.Country};      Width=16 },
    @{N='Capital';    E={$_.Capital};      Width=12 },
    @{N='Population'; E={$_.Population};   Width=12 },
    @{N='Area_km2';   E={$_.Area_km2};     Width=10 },
    @{N='Currency';   E={$_.CurrencyCode}; Width=10 }

# Country Statistics - skip N/A
$physicalEditions = @($fullReport | Where-Object { $_.Country -ne 'N/A' })
$grouped          = $physicalEditions | Group-Object -Property Country

$countryStats = @()
foreach ($group in $grouped) {
    $uniqueCities = @($group.Group | 
                      Select-Object -ExpandProperty City | 
                      Select-Object -Unique)
    
    $countryStats += [PSCustomObject]@{
        Country      = [string]$group.Name
        Editions     = [int]$group.Count
        Cities       = [string]($uniqueCities -join ', ')
        Population   = [int]$group.Group[0].Population
        Area_km2     = [int]$group.Group[0].Area_km2
        CurrencyCode = [string]$group.Group[0].CurrencyCode
        CurrencyName = [string]$group.Group[0].CurrencyName
        Languages    = [string]$group.Group[0].Languages
    }
}

# Sort Editions descending, Country ascending - identical output in PS5 and PS7!
$countryStats = $countryStats | Sort-Object -Property @{Expression='Editions'; Descending=$true}, @{Expression='Country'; Descending=$false}

Write-Output ""
Write-Output "======================================================"
Write-Output "         PSConfEU - COUNTRY STATISTICS                "
Write-Output "======================================================"
$countryStats | Format-Table -AutoSize -Property `
    @{N='Country';    E={$_.Country};      Width=16 },
    @{N='Editions';   E={$_.Editions};     Width=10 },
    @{N='Cities';     E={$_.Cities};       Width=24 },
    @{N='Population'; E={$_.Population};   Width=12 },
    @{N='Area_km2';   E={$_.Area_km2};     Width=10 },
    @{N='Currency';   E={$_.CurrencyCode}; Width=10 }

Write-Output ""
Write-Output "======================================================"
Write-Output "         PSConfEU - LANGUAGE & CURRENCY DETAILS       "
Write-Output "======================================================"
$countryStats | Format-Table -AutoSize -Property `
    @{N='Country';      E={$_.Country};      Width=16 },
    @{N='Currency';     E={$_.CurrencyCode}; Width=10 },
    @{N='CurrencyName'; E={$_.CurrencyName}; Width=22 },
    @{N='Languages';    E={$_.Languages};    Width=50 }

Write-Output ""
Write-Output "======================================================"
Write-Output "         PSConfEU - EDITION NOTES                     "
Write-Output "======================================================"
$fullReport | Format-Table -AutoSize -Property `
    @{N='Year';    E={$_.Year};    Width=6  },
    @{N='City';    E={$_.City};    Width=12 },
    @{N='Country'; E={$_.Country}; Width=16 },
    @{N='Notes';   E={$_.Notes};   Width=50 }

# Summary
$mostVisited = $countryStats[0]
$onlineCount = @($fullReport | Where-Object { $_.City -eq 'Online'  }).Count
$noConfCount = @($fullReport | Where-Object { $_.City -eq 'No Conf' }).Count

Write-Output ""
Write-Output "======================================================"
Write-Output "         PSConfEU - SUMMARY                           "
Write-Output "======================================================"
Write-Output "Most Visited Country : $($mostVisited.Country) ($($mostVisited.Editions) editions)"
Write-Output "Cities Visited       : $($mostVisited.Cities)"
Write-Output "Physical Editions    : $($physicalEditions.Count)"
Write-Output "Online Editions      : $onlineCount"
Write-Output "No Conference        : $noConfCount"
Write-Output "Total Editions       : $($fullReport.Count)"
Write-Output "Unique Countries     : $($countryStats.Count)"
Write-Output "======================================================"
```

---

> 💡 **Tip:** Copy the code above, shorten it in your editor, and verify that the output is **identical** before submitting!

---
