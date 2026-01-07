# Shodan

https://www.shodan.io/

# Basic Search Filters
---
**port:** `Search by specific port`  
**net:** `Search based on an IP/CIDR`  
**hostname:** `Locate devices by hostname`  
**os:** `Search by Operating System`  
**city:** `Locate devices by city`  
**country:** `Locate devices by country`  
**geo:** `Locate devices by coordinates`  
**org:** `Search by organization`  
**before/after:** `Timeframe delimiter`  
**hash:** `Search based on banner hash`  
**has_screenshot:true** `Filter search based on a screenshot being present`  
**title:** `Search based on text within the title`

# Examples
---
Webcamxp instances in the US  
`webcamxp country:"US"`

Cisco devices in New York  
`cisco city:"New York"`

Unsecured Linksys Webcams with screenshots in the search query  
`title:"+tm01+" has_Screenshot:true`