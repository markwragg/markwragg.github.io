---
layout: post
title: Get Transaction Sensor URL settings from PRTG with Powershell
image: "/content/images/2016/09/PRTG-logo.jpg"
date: '2016-09-01 21:31:39'
tags:
- powershell
- prtg
---

If you're looking for a way to interrogate multiple HTTP Transaction Sensors in your PRTG monitoring installation using a script then this blog post is for you.

I was interested in auditing our monitoring sensors to check they were correctly configured, in particular our Transaction Sensors which visit a series of URLs in turn to simulate a login and check for keyword values on certain pages. We had an issue where (due to sensor cloning) some of these sensors were pointing to the wrong URLs and we needed to understand how extensive this problem was.

![](/content/images/2016/09/tip-of-the-iceberg-90839.jpg)

PRTG has fairly detailed [API documentation](https://www.paessler.com/manuals/prtg/application_programming_interface_api_definition), but doing the above turned out to be surprisingly challenging, so i'm detailing it here in case it should help someone else in the future. 

The below method allows you to return a list of objects (i'm filtering for sensors) in JSON which match a certain filter (i'm getting PRTG to filter by type for the HTTP Transaction sensors). This gets me the list of sensors I want to query:

```
/api/table.json?content=sensors&output=json&columns=objid,probe,group,device,sensor,status&count=10000&filter_type=HTTPTransaction
```

The API also includes a method for querying an object setting, which works on an individual basis by passing the ID of the object and the **name** of the property you want to return:

```
/api/getobjectproperty.htm?id={object ID}&name={property name}
```

So far so good.. unfortunately the Paessler documentation of the property names is pretty light. It directs you to look at the edit page for the setting you want to query and check the name for the input box. For some settings this is fairly derivable, the object name is usually "name" and you can get the tags with "tags" for example. However determining the names of the Transaction Sensor URL step properties (such as URL):

![](/content/images/2016/09/TransactionURL-2.png)

Ultimately required guesswork. The answer turned out to be that the URL property is named:

> **HTTPURL{n}** 

i.e HTTPURL1, HTTPURL2, HTTPURL3, HTTPURL4, HTTPURL5 .. HTTPURL10.

The API only permits you to query a single property of a single object at a time, so I used Powershell to loop through the settings that I wanted to query.

Having worked out the above property name, I googled it to see if anyone had come before me. They seemingly had not, but I did come across a [malware analysis site that had analysed the PRTG core executable](https://malwr.com/analysis/OGNhNjRlZTI4NmI4NGZlNGJjM2U2M2Y1ZWIwZjhkNzQ/) and as a result had a page of string values, on which was HTTPURL1. This led me to the following additional list, which I tested against the HTTP Transaction sensor and confirmed are valid for that sensor type:

**Transaction Sensor Property Names**

cookies
httpauthentication
httpmethod1
httpmethod2
httpmethod3
httpmethod4
httpmethod5
httpmethod6
httpmethod7
httpmethod8
httpmethod9
httpmethod10
httppassword
httpurl1
httpurl2
httpurl3
httpurl4
httpurl5
httpurl6
httpurl7
httpurl8
httpurl9
httpurl10
httpuser
includemaynot1
includemaynot2
includemaynot3
includemaynot4
includemaynot5
includemaynot6
includemaynot7
includemaynot8
includemaynot9
includemaynot10
includemust1
includemust2
includemust3
includemust4
includemust5
includemust6
includemust7
includemust8
includemust9
includemust10
maxdownload
name
parenttags
postdata1
postdata2
postdata3
postdata4
postdata5
postdata6
postdata7
postdata8
postdata9
postdata10
priority
proxypassword
proxyport
proxyuser
steptimeout
tags
timeout

*-- Note: this is not an exhaustive list. Password properties return stars rather than the actual password (as does the priority property but I think that's intentional :) ).*

I wanted to find a property for knowing whether a transaction step is enabled or disabled (this is an option from step 2 onward) but unfortunately have not been able to find one.

For the regular HTTP sensor types, the names above without numbers are likely all valid.

**The script**

Returning to the matter at hand, the full script for auditing the URL settings of my Transaction sensors is below. The script works as follows:

1. Uses the PRTG API to get a filtered list of all sensor objects which match a certain type (as defined in the Param block) in JSON and converts this to a Powershell object. Note that whatever you set as `$SensorType` is sent via the query string so be careful with characters (such as spaces) that may need to be URL encoded.
2. Loops through the list of servers and gets the URL property for all 10 URLs steps (note that it checks them all whether they are all populated or not and it does not check or return whether the steps are enabled). Through the loop it adds each URL to the sensor object.
3. The final complete object (the original sensor settings and the 10 new columns with the URL details) are piped to a CSV file (you could modify where the output is redirected to whatever suits your needs). 

I've also got a bit of a thing for `Write-Progress` at the moment and this was a legitimate opportunity to use nested progress bars to report progress:

![](/content/images/2016/09/get-prtgsensordetails.png)

Here's the full script for anyone looking for a similar solution:

<script src="https://gist.github.com/markwragg/7640aeab480f2f4e4b278908d108c1ef.js"></script>

# Get Auto Acknowledge setting for all Ping sensors from PRTG with Powershell

While browsing Paessler's Knowledge Base I came across [a post from someone looking to extract the auto acknowledge setting](https://kb.paessler.com/en/topic/71062-extract-auto-acknowledge-setting-from-all-sensors-via-api) (which appears to be an option on Ping sensors only) from all sensors in PRTG using the API. As this was fundamentally a similar problem as my script above, I made a quick modified version of it to achieve this result, which you can find below:

<script src="https://gist.github.com/markwragg/02a91560f46210cf67ca5c810681f6ae.js"></script>
