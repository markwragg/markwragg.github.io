---
title: Perform Currency Conversions with PowerShell

header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2024/Currency-Converter.jpeg"
  teaser: "/content/images/2024/Currency-Converter.jpeg"
date: '2024-02-08 13:30:00'
tags:
- powershell
- api
- module
- currency
---

I've recently been working on a [PowerShell module for exploring Azure costs](https://wragg.io/monitor-and-manage-your-azure-cloud-costs-with-powershell/) and while doing so added some functionality to allow the costs to be converted between different currencies. It occurred to me that this functionality would be useful as a module of its own, and when I searched around I didn't find too many recent examples for the same. As such I've now developed and published a module in the PowerShell Gallery and on GitHub called [CurrencyConverter](https://github.com/markwragg/PowerShell-CurrencyConverter/tree/main).

> https://github.com/markwragg/PowerShell-CurrencyConverter/

The module is essentially a PowerShell wrapper for the currency conversion API that is provided by [ExchangeRate-API](https://www.exchangerate-api.com/). They provide an open API, which requires no registration or API key to use. This was helpful for my other module as I didn't want the user to have to take a dependency on registering for a service for it to work. By default the Currency Converter module uses the open API, but if you have registered and wish to use an API key, you can provide one via the `-APIKey` parameters of `Convert-Currency` and `Get-ExchangeRate`. Note that with the open API, the rates refresh once every 24 hours. This is also true for the free tier of the registered API, but you are less likely to be rate limited. If you take one of their paid plans then the currency conversion rates refresh more frequently.

The Currency Converter module is published in the [PowerShell Gallery](https://www.powershellgallery.com/packages/CurrencyConverter/), so can be installed by running:

```powershell
Install-Module CurrencyConverter
```

## Usage

This module provides the following cmdlets:

Cmdlet               | Description
-------------------- | ---------------------------------------------------------------------------------------------------------
Convert-Currency     | Converts a decimal value between two specified currencies.
Format-Currency      | Formats a value as a string with the currency symbol for a specified country.
Get-Currency         | Returns the list of supported currency codes along with their currency name and country code.
Get-ExchangeRate     | Returns all exchange rates for a specified currency, or a specified exchange rate between two currencies.

### Convert-Currency

To perform a simple currency conversion, execute:

```powershell
Convert-Currency -Value 100 -From USD -To GBP
```

To use the registered API, provide your key via the `-APIKey` parameter:

```powershell
Convert-Currency -Value 100 -From USD -To GBP -ApiKey yourapikeystringhere
```

> See here for a list of supported currencies: https://www.exchangerate-api.com/docs/supported-currencies. Alternatively you can use the `Get-Currency` cmdlet.

After querying the API, the response is saved within the module directory, as a JSON file. If you query the API for the same currency again, the file will be read. It contains a date field for when the values next refresh. If the current date is earlier than this value, then the file is used. If it is after this date, the API is queried again and the file updated. As a result this module will only query the API once a day, per currency.

You can alternatively provide the value to be converted via the pipeline:

```powershell
100 | Convert-Currency -From EUR -To CAD
```
```plaintext
79.201100
```

And this means that you can pipe multiple values in to perform a series of conversions:

```powershell
100,200,250 | Convert-Currency -From USD -To JPY
```
```plaintext
14805.315400
29610.630800
37013.288500
```

### Format-Currency

The value returned by `Convert-Currency` is always a decimal, to ensure we keep the result as precise as possible, and in case you want to do further calculations. If you want to present the result (such as in a report) as a currency value, then you can use the `Format-Currency` cmdlet. This returns a string value, rounded to two decimal places (by default) and with the corresponding currency symbol added. Obviously you can't then treat this result as a decimal value, so you probably want to ensure this cmdlet is used last in your pipeline.

```powershell
100 | Format-Currency -Currency USD
```
```plaintext
$100.00
```

You can specify a different number of decimal places to round to, via the `-Decimals` parameter:

```powershell
Format-Currency -Value 123.4567890 -Currency GBP -Decimals 4
```
```plaintext
£123.4568
```

You can also perform a currency conversion at the same time as performing the format:

```powershell
Format-Currency -Value 100 -Currency GBP -ConvertTo USD
```
```plaintext
$125.85
```

For some currencies it may be convention to put the currency symbol after the value. This can be achieved by specifying the `-SymbolAtEnd` switch parameter:

```powershell
Format-Currency -Value 100 -Currency GBP -SymbolAtEnd
```
```plaintext
125.85£
```

`Format-Currency` also supports pipeline input, so you can simply pipe the result of `Convert-Currency` to `Format-Currency`:

```powershell
Convert-Currency -Value 100 -From USD -To EUR | Format-Currency -Currency EUR
```
```plaintext
€93.04
```

### Get-Currency

The currency code inputs for the previous two cmdlets have a `validateset` attribute, so if you try to use an unsupported code an error is returned with the list of codes supported. If you're not sure what the code is for a particular currency or country, you can explore them with the `Get-Currency` cmdlet. Executing this on its own returns all of the supported currency codes along with their currency name and country:

```powershell
Get-Currency
```
```plaintext
Code Name                          Country
---- ----                          -------
AED  UAE Dirham                    United Arab Emirates
AFN  Afghan Afghani                Afghanistan
ALL  Albanian Lek                  Albania
AMD  Armenian Dram                 Armenia
ANG  Netherlands Antillian Guilder Netherlands Antilles
AOA  Angolan Kwanza                Angola
ARS  Argentine Peso                Argentina
AUD  Australian Dollar             Australia
AWG  Aruban Florin                 Aruba
AZN  Azerbaijani Manat             Azerbaijan
BAM  Bosnia and Herzegovina Mark   Bosnia and Herzegovina
...
```

> Note that for all European Countries that use the Euro, "European Union" is listed as the country. I appreciate this isn't entirely accurate and possibly politically controversial :).

With `Get-Currency` you can also specify a specific code (this requires an exact match):

```powershell
Get-Currency -Currency GBP
```

Or you can filter by currency or country name (this will return results for partial matches):

```powershell
Get-Currency -Name 'Pound'
```
```plaintext
Code Name                   Country
---- ----                   -------
EGP  Egyptian Pound         Egypt
FKP  Falkland Islands Pound Falkland Islands
GBP  Pound Sterling         United Kingdom
GGP  Guernsey Pound         Guernsey
GIP  Gibraltar Pound        Gibraltar
IMP  Manx Pound             Isle of Man
JEP  Jersey Pound           Jersey
LBP  Lebanese Pound         Lebanon
SDG  Sudanese Pound         Sudan
SHP  Saint Helena Pound     Saint Helena
SSP  South Sudanese Pound   South Sudan
SYP  Syrian Pound           Syria
```

```powershell
Get-Currency -Country 'United'
```
```plaintext
Code Name                 Country
---- ----                 -------
AED  UAE Dirham           United Arab Emirates
GBP  Pound Sterling       United Kingdom
USD  United States Dollar United States
```

### Get-ExchangeRate

Finally there is the `Get-ExchangeRate` cmdlet. You can use this to return the full result from the API as a PowerShell object, which includes the fields that show when the next update will be available and all of the supported rates for a specified currency:

```powershell
Get-ExchangeRate -Currency USD
```

To use the registered API, provide your key via the `-APIKey` parameter:

```powershell
Get-ExchangeRate -Currency GBP -ApiKey yourapikeystringhere
```

When an API key is provided, the v6 API is invoked. This returns the rates as a property called 'conversion_rates'. The Open API returns the property as 'rates'.

```plaintext
result                : success
provider              : https://www.exchangerate-api.com
documentation         : https://www.exchangerate-api.com/docs/free
terms_of_use          : https://www.exchangerate-api.com/terms
time_last_update_unix : 1707350551
time_last_update_utc  : Thu, 08 Feb 2024 00:02:31 +0000
time_next_update_unix : 1707437661
time_next_update_utc  : Fri, 09 Feb 2024 00:14:21 +0000
time_eol_unix         : 0
base_code             : USD
rates                 : @{USD=1; AED=3.6725; AFN=73.755496; ALL=96.818553; AMD=405.868756; ANG=1.79; AOA=839.476403; ARS=830.15;
                        AUD=1.533245; AWG=1.79; AZN=1.699659; BAM=1.81591; BBD=2; BDT=109.768633; BGN=1.81581; BHD=0.376;
                        BIF=2850.388516; BMD=1; BND=1.343391; BOB=6.934197; BRL=4.962119; BSD=1; BTN=83.001945; BWP=13.679849;
                        BYN=3.245101; BZD=2; CAD=1.346345; CDF=2728.475301; CHF=0.873457; CLP=947.575519; CNY=7.19952;
                        COP=3954.758146; CRC=517.23293; CUP=24; CVE=102.376655; CZK=23.176359; DJF=177.721; DKK=6.92457;
                        DOP=58.566572; DZD=134.658699; EGP=30.904297; ERN=15; ETB=56.700184; EUR=0.928449; FJD=2.247047;
                        FKP=0.792018; FOK=6.92461; GBP=0.792011; GEL=2.659816; GGP=0.792018; GHS=12.422748; GIP=0.792018;
                        GMD=66.562902; GNF=8588.188018; GTQ=7.814905; GYD=209.536717; HKD=7.820255; HNL=24.66876; HRK=6.995483;
                        HTG=131.989675; HUF=360.172059; IDR=15650.579046; ILS=3.652044; IMP=0.792018; INR=83.001968;
                        IQD=1310.842088; IRR=42080.433475; ISK=137.663905; JEP=0.792018; JMD=156.245808; JOD=0.709;
                        JPY=148.053154; KES=160.352692; KGS=89.414602; KHR=4091.328437; KID=1.533294; KMF=456.772434;
                        KRW=1326.84486; KWD=0.307938; KYD=0.833333; KZT=452.274884; LAK=20736.087575; LBP=15000; LKR=312.628935;
                        LRD=192.145847; LSL=18.897808; LYD=4.844127; MAD=10.061229; MDL=17.827961; MGA=4536.616958;
                        MKD=57.312368; MMK=2099.744651; MNT=3412.288092; MOP=8.054862; MRU=39.584855; MUR=45.384067;
                        MVR=15.453447; MWK=1689.264206; MXN=17.05608; MYR=4.761416; MZN=63.917225; NAD=18.897808;
                        NGN=1407.528785; NIO=36.672231; NOK=10.577722; NPR=132.803112; NZD=1.636807; OMR=0.384497; PAB=1;
                        PEN=3.861833; PGK=3.749532; PHP=56.01365; PKR=279.042023; PLN=4.033029; PYG=7296.21721; QAR=3.64;
                        RON=4.620779; RSD=108.801814; RUB=91.346239; RWF=1284.744312; SAR=3.75; SBD=8.488406; SCR=13.291546;
                        SDG=544.79719; SEK=10.475851; SGD=1.343391; SHP=0.792018; SLE=22.587694; SLL=22587.694431;
                        SOS=571.785619; SRD=36.611265; SSP=1102.449099; STN=22.747273; SYP=12950.775091; SZL=18.897808;
                        THB=35.595367; TJS=10.947348; TMT=3.500938; TND=3.129049; TOP=2.371485; TRY=30.62091; TTD=6.777386;
                        TVD=1.533294; TWD=31.341578; TZS=2541.212285; UAH=37.600041; UGX=3817.518963; UYU=39.148736;
                        UZS=12479.514661; VES=36.2375; VND=24413.241561; VUV=120.508933; WST=2.745778; XAF=609.029912; XCD=2.7;
                        XDR=0.754059; XOF=609.029912; XPF=110.795003; YER=250.331823; ZAR=18.897825; ZMW=27.029301;
                        ZWL=10811.587531}
```

If you want to return just the exchange rates, you can execute `Get-ExchangeRate` and specify the `-Rates` parameter:

```powershell
Get-ExchangeRate -Currency GBP -Rates
```
```plaintext
GBP : 1
AED : 4.636888
AFN : 92.99538
ALL : 122.070196
AMD : 510.608906
ANG : 2.260049
AOA : 1067.044213
ARS : 1048.144979
AUD : 1.935836
AWG : 2.260049
AZN : 2.145246
BAM : 2.292681
BBD : 2.525194
BDT : 138.549806
...
```

If you just want to return a specific exchange rate, you can execute `Get-ExchangeRate` and specify `-From` and `-To`. This will get you the rate of exchange, instead of performing a conversion:

```powershell
Get-ExchangeRate -From GBP -To USD
```
```plaintext
1.262609
```

`Get-ExchangeRate` also supports pipeline input, so if you want to get the exchange rate details for a set of currencies you can execute:

```powershell
'USD','GBP','EUR' | Get-ExchangeRate
```

Finally, if for some reason you wanted to query the API for every currency (resulting in the module caching every currencies exchange rate to disk), you could do the following (although beware this might trigger rate limiting for the API, as it will make 161 queries in quick succession):

```powershell
(Get-Currency).Code | Get-ExchangeRate
```

I hope you found this useful. If you experience any issues or can think of improvements that could be made, please see here for how to request or make a contribution:

- https://github.com/markwragg/PowerShell-CurrencyConverter/blob/main/CONTRIBUTING.md